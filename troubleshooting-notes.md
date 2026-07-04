# Module 1 — Troubleshooting Notes
# System Basics (Linux Foundations)

**Environment:** V*****-VM (Ubuntu Server 25.04), Oracle VirtualBox on Asus laptop. NAT networking.
**Sessions:** module1_session01.log, module1_session02.log

---

## Issue 1 — Command syntax errors on first use of `ls` and `mkdir`

**Observed behavior:** Running `-la` and `-ls` without a leading command name returned `command not found`. Running `mkdir -Users` returned an unexpected argument error.

**Root cause:** Flags and options must follow a command name — they cannot be run standalone. `-la` is an option for `ls`, not a command itself. `-Users` was interpreted as a flag rather than a directory name because of the leading `-`.

**Resolution:** Correct syntax used: `ls -la` and `mkdir users`. Both executed successfully.

**Lesson:** Linux commands follow strict syntax: `command [options] [arguments]`. Options (flags) are meaningless without a command name preceding them. Directory names beginning with `-` are valid but require special handling.

---

## Issue 2 — `sudo` authentication failures during user creation

**Observed behavior:** First two `sudo` attempts during `adduser Management` returned `Authentication failed`. Third attempt succeeded.

**Root cause:** Password entered incorrectly on first two attempts. Not a system or configuration issue.

**Resolution:** Correct password entered on third attempt. Command succeeded.

**Lesson:** `sudo` caches credentials for a short window after a successful authentication — subsequent `sudo` commands in the same session may not re-prompt immediately.

---

## Issue 3 — `groupadd` failed with space in group name

**Observed behavior:** `sudo groupadd Line Employees` returned a usage error. The shell interpreted `Line` and `Employees` as two separate arguments.

**Root cause:** Linux group names cannot contain spaces. The shell splits arguments on whitespace by default — `Line Employees` was parsed as two separate tokens.

**Resolution:** Hyphen used instead: `sudo groupadd Line-Employees`. Command succeeded.

**Lesson:** Group names, usernames, and most identifiers in Linux cannot contain spaces. Use hyphens or underscores as word separators.

---

## Issue 4 — `usermod` flag syntax error (`-ag` instead of `-a -G`)

**Observed behavior:** `sudo usermod -ag Executives Management` returned: `usermod: -a flag is only allowed with the -G flag`.

**Root cause:** `-a` (append) and `-G` (supplementary groups) must be passed as separate flags: `-a -G`. Combined as `-ag`, the parser reads `-a` without a following `-G`, which is invalid.

**Resolution:** Correct syntax used: `sudo usermod -a -G Executives Management`. Verified with `id Management` showing correct group membership.

**Lesson:** Not all short flags can be combined. `-a -G` is a specific pair where `-a` modifies the behavior of `-G`. Combining as `-aG` works in some versions, but the safest form is `-a -G` with a space.

---

## Issue 5 — `chown` path error: `/users` vs `/home/admin/users`

**Observed behavior:** `sudo chown :Executives /users` returned `chown: cannot dereference '/users': No such file or directory`.

**Root cause:** The target directory `users` was created inside `/home/admin/`, not at the filesystem root `/`. The command was run with an absolute path pointing to `/users` which does not exist.

**Diagnosis:** `pwd` confirmed current location as `/home/admin`. The correct absolute path was `/home/admin/users`.

**Resolution:** Pivoted to `setfacl` for permission assignment instead. `sudo setfacl -m g:Executives:rwx /home/admin/users` executed successfully and confirmed with `getfacl`.

**Lesson:** Always verify the absolute path of a target before running commands that reference it. `pwd` is a fast sanity check. Path errors are one of the most common causes of "No such file or directory" errors.

---

## Issue 6 — `Permission denied` despite correct ACL on target directory (Key diagnostic scenario)

**Observed behavior:** After granting `Executives` group `rwx` ACL on `/home/admin/users`, switching to the Management user and running `cd /home/admin/users` still returned `Permission denied`.

**Root cause:** Linux enforces permissions at every directory level during path traversal. The ACL granted access to `/home/admin/users`, but the parent directory `/home/admin` had permissions `drwxr-x---` — no execute bit for users outside the `admin` group. The shell could not traverse `/home/admin` to reach `/users` regardless of what was set on the target.

**Diagnosis:** Independently identified that the parent directory was the blocker. Management attempted `sudo setfacl` to fix it while logged in as Management, which returned sudo's HAL-9000 response (`I'm afraid I can't do that`) — correctly identifying that Management lacked sudo privileges. Switched back to `admin` to apply the fix.

**Resolution:** `sudo setfacl -m g:Executives:--x /home/admin` — granted execute-only (traverse) permission on the parent directory. Execute-only was chosen deliberately: it allows passing through the directory to reach a known target, without granting read access (which would expose a directory listing of other files in `/home/admin`).

**Verification:** Switched to Management user, `cd /home/admin/users` succeeded, `touch test_file.txt` created a file, `ls -la` confirmed file created with correct ownership.

**Lesson:** ACLs and permissions on a target directory are irrelevant if a user cannot traverse the parent directories leading to it. Execute permission on a directory means "can pass through to a known target" — not "can run programs." Read permission means "can list contents." These are independent and can be granted separately. This is a foundational permission concept that comes up repeatedly in real sysadmin work.

---

## Issue 7 — `.bashrc` edits not reflected until `source` is run

**Observed behavior:** After editing `.bashrc` in `nano` and adding `MYVAR` and the `Hello` alias, running `echo $MYVAR` immediately after saving returned the old value (`test123`) rather than the updated value.

**Root cause:** `.bashrc` is only executed when a new shell session starts, or when explicitly sourced. Saving the file does not automatically reload it into the running shell's environment.

**Resolution:** `source ~/.bashrc` run after each edit. Variable and alias confirmed live immediately after sourcing.

**Lesson:** File edits to shell configuration files (`.bashrc`, `.bash_profile`, `.profile`) take effect in the current shell only after `source <file>` is run. New terminal sessions will pick up changes automatically. This is consistent behavior and not a bug.
