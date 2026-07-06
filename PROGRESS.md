# PROGRESS

Single-glance status tracker. Update immediately after any objective is scored.
Statuses: ⬜ Not Started · 🟡 In Progress · 🟨 Needs Reinforcement · ✅ Complete

Last updated: 2026-07-05 (Module 2 complete)

---

## Module 0 — Project Standards

| Item | Status |
|---|---|
| Repo structure defined | ✅ Complete |
| Naming conventions defined | ✅ Complete |
| Governing document finalized | ✅ Complete |

---

## Module 1 — System Basics (Linux Foundations)
**Status: ✅ COMPLETE — 2026-06-29**

| Objective | Status | Notes |
|---|---|---|
| 1.1 Navigation & Filesystem | ✅ Complete | Absolute and relative paths demonstrated organically during 1.2 work. `pwd` used as genuine diagnostic step. |
| 1.2 Users, Groups, Permissions | ✅ Complete | ACL traversal issue independently diagnosed and resolved. `chmod`/`chown` demonstrated on `No_Grunts`. Grunt denied-access root cause correctly identified. Full break→diagnose→fix→verify loop completed. |
| 1.3 Shell Environment | ✅ Complete | `env`/`printenv` reviewed. `.bashrc` edited via `nano`, sourced, effect proven live. Learner independently observed "stale until sourced" behavior. |
| 1.4 Basic Networking Awareness | ✅ Complete | `ip a`, `ip route`, `ping -c 4 google.com` logged. Gateway (`<GATEWAY_IP>`), VM address (`<VM_IP>`), Tailscale overlay (`<TAILSCALE_IP>`) correctly identified. DNS resolution step explained correctly. |

---

## Module 2 — System Troubleshooting
**Status: ✅ COMPLETE — 2026-07-05**

| Objective | Status | Notes |
|---|---|---|
| 2.1 Process Management | ✅ Complete | `ps aux`/`top` used to identify a runaway `yes` process, ownership confirmed. Both plain `kill` (clean `Terminated`) and `kill -9` (`Killed`) demonstrated, each verified via follow-up `ps aux | grep`. Session 1 included a genuine unplanned VM freeze/power-cycle incident (root cause inconclusive — ruled out resource exhaustion from `yes` itself; desktop/GUI froze after starting a second instance) that was independently recovered from and logged as evidence in its own right. |
| 2.2 Services | ✅ Complete | Fresh nginx install, confirmed healthy (`active (running)`, `curl localhost`). Deliberately broke config (removed semicolon from `worker_connections` line). Diagnosed via `nginx -t` (which flagged the wrong line — the closing brace — requiring backward reasoning to the real fault) and cross-checked against `systemctl status`/`journalctl -u nginx`. Fixed root cause, verified via both `systemctl status` (service-level) and `curl localhost` (functional-level). |
| 2.3 Networking Troubleshooting | ✅ Complete | Corrupted `/etc/resolv.conf` via the symlinked path (initial attempt had a genuine typo — `namesaver` instead of `nameserver` — compounding the invalid IP). Full diagnostic ladder run correctly: `ping` to a known IP (routing confirmed healthy) → `ping` to hostname (DNS-specific failure confirmed) → `dig`/`nslookup` (connection refused, isolating the resolver). Traced root cause through the `/etc/resolv.conf` → `systemd-resolved` symlink relationship rather than assuming symlink breakage. Fixed via `systemctl restart systemd-resolved` (correct mechanism — regenerates the managed file rather than hand-editing it). Self-verified by deliberately re-breaking and re-fixing independently. |
| 2.4 Log Analysis | ✅ Complete | Custom `widget.service` built and confirmed healthy via heartbeat logging. Cold diagnosis scenario: process frozen via `kill -STOP` with no live observation — `systemctl status` showed a false-healthy `active (running)` while `journalctl` was the only evidence revealing heartbeats had actually stopped. Correctly identified `T` (stopped) process state via `ps aux`. Initial fix (`systemctl restart`) recognized as functionally different from the correct fix; re-ran the scenario and applied `kill -CONT` correctly (including recovering from an `Operation not permitted` error by adding `sudo`), resuming the original process without restarting it. Verified recovery via resumed heartbeats in `journalctl`. |

**Module 2 complete — multi-layer diagnosis, log-based root-causing, and independent troubleshooting all demonstrated across the four objectives, satisfying the module's completion bar.**

---

## Module 3 — System Administration

| Objective | Status |
|---|---|
| 3.1 Service Configuration | ⬜ Not Started |
| 3.2 Permissions/Policy Control | ⬜ Not Started |
| 3.3 Storage/Filesystems | ⬜ Not Started |
| 3.4 Multi-layer Debugging | ⬜ Not Started |

---

## Module 4 — Security Fundamentals

| Objective | Status |
|---|---|
| 4.1 Access Control Review | ⬜ Not Started |
| 4.2 Attack Surface Awareness | ⬜ Not Started |
| 4.3 Authentication vs. Authorization | ⬜ Not Started |
| 4.4 Controlled Security Failure | ⬜ Not Started |

---

## Module 5 — Networked Systems

| Objective | Status |
|---|---|
| 5.1 Lab Host Migration (VirtualBox → Proxmox) | ⬜ Not Started |
| 5.2 Network Segmentation (pfSense/OPNsense) | ⬜ Not Started |
| 5.3 Multi-system Communication | ⬜ Not Started |
| 5.4 Network Failure Analysis | ⬜ Not Started |
| 5.5 Distributed Service Awareness | ⬜ Not Started |

---

## Module 6 — Windows Server & Active Directory

| Objective | Status |
|---|---|
| 6.1 Windows Server Install & Config | ⬜ Not Started |
| 6.2 Active Directory Domain Setup | ⬜ Not Started |
| 6.3 Users, Groups & Group Policy | ⬜ Not Started |
| 6.4 Linux Machine Joined to Domain | ⬜ Not Started |
| 6.5 Remote Administration via RDP | ⬜ Not Started |

---

## Module 7 — Automation

| Objective | Status |
|---|---|
| 7.1 Advanced Bash Scripting | ⬜ Not Started |
| 7.2 Ansible | ⬜ Not Started |
| 7.3 Python for SysAdmin/Security | ⬜ Not Started |

---

## Module 8 — Containers

| Objective | Status |
|---|---|
| 8.1 Docker Fundamentals | ⬜ Not Started |
| 8.2 Docker Compose | ⬜ Not Started |
| 8.3 Deploy a Real Service | ⬜ Not Started |
| 8.4 Container Lifecycle Management | ⬜ Not Started |

---

## Module 9 — Cloud Fundamentals

| Objective | Status |
|---|---|
| 9.1 AWS — Compute (EC2) | ⬜ Not Started |
| 9.2 AWS — Networking (VPC/SGs) | ⬜ Not Started |
| 9.3 AWS — IAM | ⬜ Not Started |
| 9.4 AWS — Remote Administration | ⬜ Not Started |
| 9.5 Azure — Compute (VM) | ⬜ Not Started |
| 9.6 Azure — Networking (VNet/NSGs) | ⬜ Not Started |
| 9.7 Azure — IAM (RBAC) | ⬜ Not Started |
| 9.8 Azure — Cross-environment Integration | ⬜ Not Started |

---

## Module 10 — Security Operations

| Objective | Status |
|---|---|
| 10.1 Log Aggregation | ⬜ Not Started |
| 10.2 SIEM Basics | ⬜ Not Started |
| 10.3 Incident Response Simulation | ⬜ Not Started |
| 10.4 Vulnerability Scanning | ⬜ Not Started |
| 10.5 System Hardening | ⬜ Not Started |

---

## Parallel Certification Track

Runs alongside modules — not sequential, not prerequisites. Tracked separately.

| Certification | Status | Begin | Target Completion | Notes |
|---|---|---|---|---|
| CompTIA Security+ | ⬜ Not Started | During Module 4 | ~Module 5–6 | DoD 8570 compliant. Primary cert for security-adjacent roles. |
| AWS Cloud Practitioner | ⬜ Not Started | Early Module 9 | Within Module 9 | Foundational stepping stone to SAA. Fast to obtain. |
| AWS Solutions Architect Associate (SAA-C03) | ⬜ Not Started | Mid-Module 9 | ~Module 10 | The cert that changes offer numbers. Primary cloud credential. |
| CompTIA Linux+ | ⬜ Optional | After Module 3 | Flexible | Low additional study burden. Good if SysAdmin-primary path. |
| Azure Fundamentals (AZ-900) | ⬜ Optional | During Module 9 | Within Module 9 | Pairs naturally with Azure module content. Minimal extra effort. |
| CompTIA Network+ | ⬜ Optional | After Module 5 | Flexible | Lower priority given networking is covered operationally in the portfolio. |


| Objective | Status |
|---|---|
| 11.1 Multi-VM Linux Infrastructure | ⬜ Not Started |
| 11.2 Windows Domain Integration | ⬜ Not Started |
| 11.3 Containerized Service Deployment | ⬜ Not Started |
| 11.4 Cloud Component Integration | ⬜ Not Started |
| 11.5 Access Control Across Environment | ⬜ Not Started |
| 11.6 Monitoring & Logging | ⬜ Not Started |
| 11.7 Security Hardening & Documentation | ⬜ Not Started |
| 11.8 Independent Failure Recovery | ⬜ Not Started |tarted |
