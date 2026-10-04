# Ubuntu Workshop – Lessons So Far

## Lesson 1: Yes, there's an app store

It's called **App Center** (older versions call it "Ubuntu Software"). Look for the orange shopping-bag icon in your dock. Search, click Install, and it handles everything, including updates.

Ubuntu has two main ways apps are packaged:

- **Snap** – Ubuntu's own format. Apps are sandboxed and update automatically. The App Center shows these first.
- **Deb (APT)** – the classic Debian format, used by Ubuntu for decades. Often lighter and more "native."

Beginners don't need to care about the difference yet. If an app seems slow to start or behaves oddly, that's sometimes a Snap, and the deb version may be better.

## Lesson 2: Meet the terminal (gently)

Open it with `Ctrl+Alt+T`. Three essential commands:

```bash
sudo apt update      # refresh the list of available software
sudo apt upgrade     # install updates
sudo apt install gimp   # install an app
```

`sudo` means "do this as administrator." It asks for your password; nothing shows while you type it, which is normal.

### Common error: "are you root?"

```
Error: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
Error: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

You forgot `sudo`. Installing software changes system files, which only root may touch. Fix:

```bash
sudo apt install gimp
```

Tips:
- Press `Y` when asked to continue.
- `apt` works system-wide; it doesn't matter which folder you're in.
- Read error messages slowly. They usually tell you what's wrong.

## Lesson 3: Three best practices

1. **Update weekly.** Use the Software Updater pop-up, or `sudo apt update && sudo apt upgrade`.
2. **Don't paste commands you don't understand**, especially anything with `rm -rf` or `curl ... | bash`. Ask first.
3. **Stick to official sources**: App Center, APT, or an app's official website. Avoid random PPAs or `.deb` files from forums.

## Lesson 4: Making it fun

- Press the **Super key** (Windows key) and start typing to launch anything. Press it twice to see all apps.
- **Settings → Appearance** for dark mode and accent colours.
- Install **GNOME Tweaks** and **Extension Manager** to customise the desktop heavily (macOS-style dock, tiling windows, etc.).
- Learn one keyboard shortcut per day:
  - `Super+A` – show all apps
  - `Alt+Tab` – switch windows
  - `Ctrl+Alt+T` – open terminal

## Side lesson: Why not `curl ... | bash`?

`curl https://somewhere.com/install.sh | bash` downloads a script and runs it immediately, without looking at it. The problem is trust plus blindness.

**Why it's risky**

1. **You're executing unknown code.** With `sudo`, it can do anything: delete files, install a backdoor, add itself to startup.
2. **You can't review what you ran.** APT packages are cryptographically signed and you can see every file they touch. A piped script leaves no trace.
3. **The server can lie.** It can serve a clean script to browsers and a malicious one to `curl`, or detect that it's being piped and behave differently.
4. **Partial downloads.** If the connection drops, bash runs whatever arrived. A truncated `rm -rf /home/you/project` becomes `rm -rf /home/you`.

**The nuance**

Reputable tools (Rust, Homebrew, Node version managers) ship this way and people use them daily. The real rule is: *don't pipe to bash blindly.* The safer habit:

```bash
curl -o install.sh https://example.com/install.sh
less install.sh      # read it; press q to quit
bash install.sh      # run only if it looks sane
```

Thirty seconds of reading, and you keep a copy of exactly what was executed.

## Homework

- Install one app via the App Center.
- Install one app via the terminal.
- Run your first update.
- Decide what you want to do with your machine (programming, media, gaming, office work) so the next session can be tailored.
