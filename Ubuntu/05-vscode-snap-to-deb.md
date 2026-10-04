# Lesson: Replacing the Snap version of VS Code with Microsoft's `.deb`

## Why

Symptom: files or folders created in the terminal don't appear in VS Code's Explorer until you refresh manually.

Diagnosis:

```bash
cat /proc/sys/fs/inotify/max_user_watches   # 65536 → the watch limit is fine
touch ~/open-folder/test.txt                 # didn't appear → file watching is broken
snap list | grep code                        # printed a line → VS Code is the Snap version
```

Cause: the App Center installs VS Code as a **Snap**. Snaps run in a sandbox, and that sandbox is known to break VS Code's file watching. No setting fixes it; the packaging is the problem.

Fix: install Microsoft's own `.deb` package instead. Same program, different installation method, no sandbox.

## The steps

### Step 1: Close VS Code

Quit it completely.

### Step 2: Remove the Snap version

```bash
sudo snap remove code
```

Note: Snap VS Code kept its settings and extensions in `~/snap/code/`. The `.deb` version uses `~/.config/Code/` and won't see them. You'll reinstall extensions afterwards.

### Step 3: Add Microsoft's repository

```bash
sudo apt install -y wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
rm packages.microsoft.gpg
```

Line-by-line explanation below.

### Step 4: Install VS Code

```bash
sudo apt update
sudo apt install code
```

### Step 5: Test

Open VS Code (Super → type `code`), File → Open Folder, then in the terminal:

```bash
touch ~/that-folder/test.txt
```

It should appear instantly.

### Step 6: Reinstall extensions

Extensions icon (four squares) on the left → re-add what you had.

From now on VS Code updates with `sudo apt upgrade` like everything else.

---

## What Step 3 actually does, line by line

### Background: how `apt` trusts software

`apt` installs software from **repositories**: servers holding packages plus an index of what's available. Ubuntu's own repositories are configured out of the box. To install something from a third party like Microsoft, you tell `apt` about their repository.

But `apt` won't trust a repository blindly. Every repository **signs** its package index with a cryptographic key. `apt` needs the matching **public key** so it can verify the signature. Without the key, `apt` refuses to install anything from that source.

So adding a third-party repository is always two things:

1. Install the vendor's public key where `apt` can find it.
2. Tell `apt` the repository's address and which key goes with it.

That's all Step 3 is.

### Line 1

```bash
sudo apt install -y wget gpg
```

Installs two small tools needed for the next lines. `wget` downloads files from the web. `gpg` handles cryptographic keys. `-y` answers "yes" automatically to the install prompt. Both are usually already present; this just makes sure.

### Line 2

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
```

Downloads Microsoft's public key and converts it to the format `apt` wants.

- `wget -qO- URL` – download the URL. `-q` means quiet (no progress output). `-O-` means "write to standard output" (the terminal stream) instead of to a file, so it can be piped.
- `|` – the pipe. Sends the output of the left command into the right command.
- `gpg --dearmor` – the downloaded key is in "ASCII-armored" text form (a `.asc` file, readable text starting with `-----BEGIN PGP PUBLIC KEY BLOCK-----`). `--dearmor` converts it to the binary form `apt` expects.
- `> packages.microsoft.gpg` – write the result to a file in the current folder.

Why this is not like `curl | bash`: the download is piped into a *converter*, not into a shell. Nothing executes. The result is a data file.

### Line 3

```bash
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
```

Moves the key file to the system folder where `apt` looks for keys, with the correct ownership and permissions.

- `install` – despite the name, it's a copy command that also sets ownership and permissions in one step. Think of it as "copy and set up properly."
- `-D` – create the destination folder (`/etc/apt/keyrings/`) if it doesn't exist.
- `-o root -g root` – make the file owned by the `root` user and `root` group. System files should belong to root so ordinary users can't tamper with them.
- `-m 644` – permissions: owner can read and write (6), group can read (4), everyone else can read (4). Readable by all, writable only by root.
- Source file, then destination path.
- `sudo` is needed because `/etc/` is system territory.

`/etc/apt/keyrings/` is the modern standard location for third-party keys. Each vendor gets its own file, so one key can't vouch for another vendor's packages.

### Line 4

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```

Tells `apt` about the repository by writing one line into a config file.

The text between the quotes is a **source line**, which `apt` reads:

- `deb` – this is a repository of binary packages (as opposed to `deb-src`, source code).
- `[arch=amd64 ...]` – only for 64-bit Intel/AMD machines, which is what you have.
- `signed-by=/etc/apt/keyrings/packages.microsoft.gpg` – "verify this repository using *this specific key*." Ties Line 3 to Line 4. If the signature doesn't match this key, `apt` refuses.
- `https://packages.microsoft.com/repos/code` – the repository's address.
- `stable` – the release channel (as opposed to `insiders`, the beta builds).
- `main` – the section within the repository.

The rest of the line gets it into a file:

- `echo "..."` – print the text.
- `| sudo tee /etc/apt/sources.list.d/vscode.list` – `tee` writes its input to a file (and also echoes it to the screen, which is why you see the line printed once). It's used instead of `>` because `>` is handled by *your* shell, which doesn't have root permission; `sudo tee` runs the write as root.

`/etc/apt/sources.list.d/` is the folder for third-party repositories. One file per vendor. To remove Microsoft's repository later, delete `vscode.list` and the key file, and `apt` forgets it existed.

### Line 5

```bash
rm packages.microsoft.gpg
```

Deletes the temporary copy of the key left in your current folder. The real one is already in `/etc/apt/keyrings/`.

### Then Step 4

```bash
sudo apt update
```

`apt` re-reads all its source files, including the new `vscode.list`, contacts Microsoft's server, downloads the package index, and checks its signature against the key. If you see no errors, trust is established.

```bash
sudo apt install code
```

`apt` finds `code` in Microsoft's index and installs it, verifying each package along the way.

---

## The general pattern

Almost every third-party `apt` repository you'll ever add follows this exact shape: download key → place in `/etc/apt/keyrings/` → write a `signed-by=` source line → `apt update` → `apt install`. Once you recognise the shape, you can read any vendor's install instructions and know whether they're doing something standard or something odd.

Red flags in a vendor's instructions:

- `apt-key add` – the old, deprecated way; it made the key trust *every* repository, not just the vendor's. Still works but is being phased out.
- Piping the download into `sudo bash` or `sudo sh` – that's `curl | bash` again.
- A source line with `[trusted=yes]` – disables signature checking entirely. Never.
