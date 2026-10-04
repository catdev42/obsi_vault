# Lesson: Renaming your Ubuntu user account

Three separate things carry your name. Renaming one does not change the others.

```
pol@pol-ThinkPad-T460s
 ↑         ↑
 user      hostname          (+ a display name shown on the login screen)
```

## 1. The account name (`pol` → `cat`)

Linux won't rename an account you're logged into. You need a second admin account.

1. Create a temporary admin user (when asked for "Full Name", that's just a label):
   ```bash
   sudo adduser temp
   sudo usermod -aG sudo temp
   groups temp          # confirm "sudo" is listed
   ```
2. Log out completely. Log in as `temp`.
3. In a terminal:
   ```bash
   sudo usermod -l cat pol            # rename the account
   sudo usermod -d /home/cat -m cat   # rename and move the home folder
   sudo groupmod -n cat pol           # rename the personal group
   ```
4. Log out. Log in as `cat`. Verify:
   ```bash
   whoami        # cat
   echo $HOME    # /home/cat
   ls ~          # your files
   ```
5. Remove the temporary user:
   ```bash
   sudo deluser --remove-home temp
   ```

## 2. The hostname (`pol-ThinkPad-T460s` → `cat-thinkpad`)

```bash
sudo hostnamectl set-hostname cat-thinkpad
```

Lowercase, numbers and hyphens only. Open a new terminal to see the change.

## 3. The display name ("Pol" → "Cat")

Settings → Users → click the name → edit, or:

```bash
sudo chfn -f "Cat" cat
```

Log out and in once more so everything picks up the new names.

## Useful checks

- `users` shows who is logged in *now*, not all accounts.
- List all real accounts:
  ```bash
  getent passwd | awk -F: '$3 >= 1000 {print $1}'
  ```
- Usernames are case-sensitive; `Cat` and `cat` are different accounts.
- `usermod -l` refuses to run if the new name already exists, so it fails safely.

## Caveats

- Some programs store the full path `/home/pol` in configs (often under `~/.config`). Most survive; fix stragglers with search-and-replace.
- Best done on a fresh machine, before cloning lots of repos.
- GitHub username is unrelated; change it on github.com.
