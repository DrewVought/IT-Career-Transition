# Tier 1 Runbook — System Basics (Linux Foundations)

**Environment:** Vought-VM (Ubuntu Server 25.04), Oracle VirtualBox on Asus M16 (2023). NAT networking + Tailscale overlay.
**Source logs:** `tier1_session01.log`, `tier1_session02.log`

This runbook documents how to reproduce the environment state built during Tier 1.

---

## 1. Navigation & Filesystem

No persistent build artifact — demonstrated through live navigation:

```bash
cd /home/dvadmin/users      # absolute path
cd ..                       # relative, one level up
cd ..                       # relative, another level up
cd                           # bare cd — returns to invoking user's home
pwd                          # used to confirm location after a failed command
```

## 2. Users, Groups, Permissions

**Users created:**
```bash
sudo adduser Management
sudo adduser Grunt
```

**Groups created:**
```bash
sudo groupadd Executives
sudo groupadd Line-Employees
```

**Group membership assigned:**
```bash
sudo usermod -a -G Executives Management
sudo usermod -a -G Line-Employees Grunt
```
Verify with: `id Management`, `id Grunt`

**Restricted directory (ACL approach):**
```bash
sudo setfacl -m g:Executives:rwx /home/dvadmin/users
sudo setfacl -m g:Executives:--x /home/dvadmin      # required for traversal — see troubleshooting notes
```
Verify with: `getfacl /home/dvadmin/users`

**Restricted directory (chmod/chown approach):**
```bash
mkdir No_Grunts
sudo chown :Executives No_Grunts
sudo chmod 770 No_Grunts
```
Verify with: `ls -ld No_Grunts` → `drwxrwx--- dvadmin Executives`

## 3. Shell Environment

Added to `~/.bashrc` (via `nano ~/.bashrc`):
```bash
alias Hello='echo "Hello There - Obi-Wan Kenobi"'
export MYVAR="test123"
```

Apply changes to a live shell without opening a new terminal:
```bash
source ~/.bashrc
```

## 4. Basic Networking Awareness

```bash
ip a            # list interfaces and assigned addresses
ip route        # show routing table / identify default gateway
ping -c 4 google.com   # test reachability + implicit DNS resolution
```

**Observed on Vought-VM:**
- `enp0s3`: `10.0.2.15/24` (VM's own NAT address)
- `tailscale0`: `100.90.37.50/32` (Tailscale overlay)
- Default gateway: `10.0.2.2` via `enp0s3`
