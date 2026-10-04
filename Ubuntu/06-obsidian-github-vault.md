# Lesson: Obsidian with a GitHub repository as storage

An Obsidian vault is just a folder of Markdown files. "Stored in GitHub" simply means that folder is a git repository.

## Part 1: Install Obsidian

Obsidian offers Linux downloads in several formats. Use the `.deb` (not the Snap).

1. obsidian.md → Download → Linux → **.deb**
2. Install it, either by double-clicking the file (App Center opens, click Install) or:
   ```bash
   sudo apt install ~/Downloads/obsidian_*_amd64.deb
   ```
3. Launch: press **Super** (Windows key), type `obsidian`, Enter.

Notes:
- The App Center shows "potentially unsafe" for any `.deb` you download yourself. It means "Ubuntu didn't verify this source, make sure you trust it." Downloaded over HTTPS from the official site → fine.
- Obsidian doesn't update via `apt`. It notifies you in-app; download the new `.deb` the same way.

## Part 2: Vault in a GitHub repo

Requires git and the SSH key from lesson 3.

1. On GitHub, create a new repository, e.g. `notes`. Make it **private**. Don't add a README.
2. Clone it where the vault should live:
   ```bash
   mkdir -p ~/Documents && cd ~/Documents
   git clone git@github.com:yourname/notes.git
   ```
3. In Obsidian: **Open folder as vault** → select `~/Documents/notes`.
4. Add a `.gitignore` so machine-specific files aren't committed:
   ```bash
   cd ~/Documents/notes
   printf ".obsidian/workspace.json\n.obsidian/workspace-mobile.json\n.trash/\n" > .gitignore
   ```
   - `workspace.json` – which panes/tabs are open; differs per machine and causes conflicts.
   - `.trash/` – Obsidian's deleted-notes folder.
5. First commit and push:
   ```bash
   git add .
   git commit -m "Initial vault"
   git push -u origin main
   ```

## Part 3: Automate commits with the Obsidian Git plugin

1. Obsidian → Settings → **Community plugins** → turn off Restricted mode.
2. Browse → search **Git** → install **Obsidian Git** → Enable.
3. In the plugin settings:
   - Auto backup interval: e.g. 10 minutes
   - Pull on startup: on
   - Push on backup: on

Your notes now commit and push themselves, and you get full version history for free.

## Cautions

- Never put secrets (passwords, API keys, tokens) in a vault that lives on GitHub, even a private one.
- Using the vault on a second machine: always let the plugin **pull before you type**, or you'll get merge conflicts inside your notes.
- If a conflict does happen, the plugin shows a conflict file; resolve it like any git conflict (keep the lines you want, delete the `<<<<<<<` / `=======` / `>>>>>>>` markers).
