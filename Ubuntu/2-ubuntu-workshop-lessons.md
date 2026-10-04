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

---

## Lesson 5: GNOME Tweaks and Extension Manager

Ubuntu's desktop is **GNOME**. It hides most settings; these two apps unlock them.

```bash
sudo apt install gnome-tweaks gnome-shell-extension-manager
```

**GNOME Tweaks** – hidden settings: fonts and scaling, Caps Lock → Ctrl/Escape, window buttons, focus behaviour, startup apps, top-bar clock/battery details.

**Extension Manager** – adds features to the desktop. Browse tab → search → Install. Good first picks:

- Dash to Dock (macOS-style dock, better autohide)
- Clipboard Indicator (clipboard history)
- Tiling Shell or Pop Shell (automatic window tiling)
- Blur my Shell (cosmetic)
- Caffeine (prevent screen lock)

Caution: install a few at a time. If the desktop misbehaves, log out and back in. Extensions can break after major Ubuntu upgrades.

## Lesson 6: Screen brightness and colour (ThinkPad T460s, 2016)

**Brightness is mostly hardware.** The T460s Full HD panel is ~250–300 nits; a MacBook Pro is 500+. Software can't add light. Checks:

- `Fn+F6` to max; disable Automatic Screen Brightness in Settings → Power
- Power profile Balanced or Performance, not Power Saver
- Plug in; many laptops cap brightness on battery
- The panel uses 220 Hz PWM dimming; keep brightness high if eyes feel strained

**Colour (ICC profiles)**

- Settings → Color → select display → Add profile → import `.icc`
- Free source: Notebookcheck reviews publish measured ICC profiles. For the T460s FHD panel, the Core i5 (20F9003SGE) review. *Done – noticeable improvement.*
- Identify exact panel if needed:
  ```bash
  sudo apt install edid-decode
  cat /sys/class/drm/card*-eDP-1/edid | edid-decode | grep -iE "manufacturer|product|model"
  ```

**Calibrating with a Spyder**

- Quick: Settings → Color → Calibrate (built-in wizard)
- Proper: **DisplayCAL**, not in apt anymore; install via Flatpak:
  ```bash
  sudo apt install flatpak
  flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
  # log out and in
  flatpak install flathub net.displaycal.DisplayCAL
  sudo flatpak override --filesystem=home   # if it can't see folders
  ```
- Targets: 6500 K, gamma 2.2, brightness "as measured" (a cd/m² target lowers brightness). Takes 15–30 min.
- Flatpak = third packaging format (after Snap and Deb). Common on other distros.

**Hardware upgrade option (later)**

- The panel is replaceable; only the LCD sheet is swapped, not the lid/bezel/hinges.
- Community favourite: **Innolux N140HCG-GQ2** (400 nits, ~100% sRGB, low power, matte only). ~100 €.
- Sellers list it as T440–T490 compatible; no confirmed T460s install report found. Ask seller to pre-verify with machine type (20F9/20FA) and current panel FRU number.
- Beware counterfeit 250-nit panels and panels sold without mounting brackets.
- Lenovo Hardware Maintenance Manual has the step-by-step. ~1 hour.

## Lesson 7: Reading `free -h`

```
               total   used   free   shared  buff/cache  available
Mem:            11Gi  6.3Gi  995Mi   1.0Gi       5.2Gi       4.8Gi
Swap:          4.0Gi     0B  4.0Gi
```

- **total** – installed RAM (12 GB here; 4 soldered + 8 in the slot)
- **free** – unused RAM; Linux keeps this low on purpose
- **buff/cache** – file cache, handed back instantly when needed
- **available** – the real "how much can I still use"
- **Swap** – disk overflow; 0 B used is healthy

`sudo apt install htop` for a live view. `inxi -Fxz` for a full hardware summary.
