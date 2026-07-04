# Module 1 — Lessons Learned
# System Basics (Linux Foundations)

**Environment:** V*****-VM (Ubuntu Server 25.04), Oracle VirtualBox on Asus laptop.
**Sessions:** module1_session01.log, module1_session02.log
**Completed:** 2026-06-29

---

## Lesson 1 — Command syntax is strict and consistent

Linux command syntax follows a fixed pattern: `command [options] [arguments]`. Options cannot be used standalone, flags must follow their command name, and arguments must match what the command expects. This isn't arbitrary — every Unix utility follows the same convention, which means the pattern learned here applies to every command encountered going forward.

**Applied to:** `ls -la`, `mkdir`, `usermod -a -G`, `groupadd`

---

## Lesson 2 — Read error messages literally before assuming something is broken

Every error encountered in this module contained the answer to why it failed. `chown: cannot dereference '/users': No such file or directory` said exactly what was wrong — the path didn't exist. `usermod: -a flag is only allowed with the -G flag` described the exact syntax problem. Getting in the habit of reading error output carefully before escalating to other diagnostic steps is the first and cheapest troubleshooting step.

---

## Lesson 3 — `pwd` is a fast, cheap sanity check

Using `pwd` to confirm location before running a path-dependent command caught and prevented at least one repeated error during this module. In production environments, running a destructive command against the wrong path can cause real damage. `pwd` before `rm`, `chown`, `chmod`, `mv`, or any command taking a path argument should become a reflex.

---

## Lesson 4 — Permissions are evaluated at every directory level during traversal

The most significant discovery in this module: an ACL correctly applied to a target directory is irrelevant if a user cannot traverse the parent directories leading to it. Access is not granted by having permission on the destination alone — the entire path must be traversable. This is a foundational Linux permission concept that applies equally to `chmod`/`chown` permissions and ACLs, and will come up again in every module involving multi-level directory structures.

**Specific scenario:** `Executives` group had `rwx` ACL on `/home/admin/users` but `Permission denied` because `/home/admin` had no execute bit for the group. Fix: `setfacl -m g:Executives:--x /home/admin` — traverse-only, no read, no write.

---

## Lesson 5 — Execute and read permissions on directories do different things

On a directory, these are independent:
- **Execute (`x`)** — allows traversal into the directory and access to specific items by name. Does not allow listing contents.
- **Read (`r`)** — allows listing contents (`ls`). Does not allow entering or accessing items inside without execute.
- **Write (`w`)** — allows creating, deleting, or renaming items within the directory.

Granting only `--x` (execute-only) on `/home/admin` was the correct minimal fix: it let the Executives group pass through to `/home/admin/users` without exposing what else lived in `/home/admin`. This principle — grant the minimum permission needed to accomplish the task — is the practical definition of least privilege, which underpins all of security operations.

---

## Lesson 6 — ACLs provide more granular control than `chmod`/`chown` alone

Standard Linux permissions (`chmod`/`chown`) operate on three buckets: owner, group, and others. ACLs (`setfacl`/`getfacl`) allow different permissions for multiple specific users and groups on the same object without restructuring ownership. Both tools were used in this module — `chmod 770` and `chown :Executives` for the `No_Grunts` directory, and `setfacl` for the `users` directory. Knowing which tool to reach for depends on the complexity of the access requirement.

---

## Lesson 7 — Shell environment is loaded once per session

`.bashrc` executes when a shell session starts. Changes to it do not take effect in a running shell until `source ~/.bashrc` is called explicitly. This means shell configuration changes are always safe to experiment with — they can be edited and sourced in a test shell without affecting other sessions, and reverted by editing the file again. This same principle applies to any file that configures environment behavior (`.bash_profile`, `.profile`, custom scripts).

---

## Lesson 8 — Traversal and routing are the same concept at different layers

The parent-directory traversal issue in this module (`--x` needed on `/home/admin` to reach `/home/admin/users`) is structurally identical to how network routing works — traffic destined for a specific host must pass through every hop in the path, and each hop has its own rules. This mental model recurs throughout the curriculum: in networking (routing tables, ACLs on router interfaces), in cloud (security group evaluation order, VPC routing), and in security operations (defense-in-depth assumes each layer independently enforces access). Recognizing the shared pattern across layers accelerates learning in every subsequent module.

---

## Resume Bullet (generated from this module)

> Built a multi-user Linux environment on Ubuntu Server with role-based access control using both standard permission bits (`chmod`/`chown`) and POSIX ACLs (`setfacl`/`getfacl`). Independently diagnosed a `Permission denied` failure caused by missing directory traversal permissions on a parent path — correctly identified the root cause, applied a minimal execute-only ACL fix, and verified resolution. Documented the full break→diagnose→fix→verify cycle in structured session logs.
