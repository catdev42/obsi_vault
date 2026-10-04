# Lesson: Setting up an SSH key for GitHub

## 1. Install and configure git

```bash
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## 2. Generate the key

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

- `-t ed25519` – modern key type (small, fast, secure)
- `-C` – a comment so you recognise the key later
- Accept the default location (`~/.ssh/id_ed25519`)
- Set a passphrase; it protects the key if the laptop is stolen

Creates two files:
- `~/.ssh/id_ed25519` – **private**, never share
- `~/.ssh/id_ed25519.pub` – **public**, this goes to GitHub

## 3. Load it into the SSH agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

The agent remembers the unlocked key so you don't retype the passphrase on every push. GNOME Keyring usually handles this automatically after first use.

## 4. Copy the public key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the whole line (starts with `ssh-ed25519`).

## 5. Add it to GitHub

GitHub → profile picture → Settings → **SSH and GPG keys** → New SSH key → paste → title it (e.g. "ThinkPad T460s") → save.

## 6. Test

```bash
ssh -T git@github.com
```

Type `yes` to trust GitHub's fingerprint the first time. Expected: "Hi yourname! You've successfully authenticated." The "does not provide shell access" line is normal.

## Afterwards

Clone with the SSH URL (`git@github.com:user/repo.git`), not HTTPS. No more password prompts.

Later: `~/.ssh/config` is where you add keys for other servers or a second GitHub account.
