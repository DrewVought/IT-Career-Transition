# Module 2 Troubleshooting Notes — System Troubleshooting

Supplementary notes covering diagnostic reasoning and one genuine unplanned incident encountered during Module 2. See `runbook.md` for the formal per-objective documentation.

---

## Incident: VM Freeze During 2.1 (Session 1)

**Summary:** While testing process termination on a second `yes > /dev/null &` instance, the desktop environment (GNOME) on V*****-VM became unresponsive. The terminal froze, and subsequent attempts to regain access to a terminal session failed. The VM required a forced power-cycle to recover.

**Evidence reviewed:** The session log (captured via `script`) was checked directly rather than relying on recollection. It showed:
- A first `yes` process cleanly terminated via plain `kill` (`[1]+ Terminated`), with no anomalies.
- CPU and memory both well within normal range in the last `top` snapshot before the freeze (~1.3GB/5.2GB memory used; one core pinned by `yes`, aggregate CPU idle >90% on this multi-core VM).
- A second `yes` process started successfully (new PID assigned).
- The log then cuts off abruptly mid-stream, consistent with the VM being powered off before the `script` session could close normally — it does **not** contain any crash message, kernel panic, or OOM-killer output.

**Root cause:** Inconclusive. The evidence rules out resource exhaustion caused directly by `yes` — both CPU and memory had significant headroom. The freeze is more consistent with a GUI/desktop-environment issue or host-side resource contention than with the lightweight background process itself, but no conclusive single cause could be established from available evidence.

**Recovery:** VM was power-cycled and returned to a stable, healthy state. The interrupted session log was treated as closed (per policy, an unclosed `script` session isn't recoverable/extendable — this is expected `script` behavior, not a data-handling error), and work resumed in a new session log.

**Disposition:** Logged as-is rather than chased further once the VM was confirmed stable, on the judgment that continued investigation without new evidence would be speculation rather than diagnosis. Recognizing when to stop chasing an inconclusive root cause — rather than over-investing in a dead end — is treated here as a legitimate part of the troubleshooting skill being built, not a shortfall in the work.

---

## Diagnostic Reasoning Notes

**On `nginx -t` reporting the wrong line (2.2):** Config parsers that read sequentially (like nginx's) often can't detect a missing terminator until a later, structurally-dependent token fails — commonly the next closing brace. This means the reported error line is frequently downstream of the actual fault, not at it. General takeaway: when a parser error doesn't match what a plain read of that line would suggest, check upward from the reported line for the real cause, rather than editing the reported line itself.

**On the `/etc/resolv.conf` symlink (2.3):** The initial working hypothesis was that writing to `/etc/resolv.conf` would destroy the file's symlink status. Checking `ls -l /etc/resolv.conf` directly disproved this — the symlink was intact (`lrwx...`), meaning the write had gone through the symlink into the underlying `systemd-resolved`-managed file instead. This is a good example of why direct verification of a hypothesis matters even when the fix ultimately works: the fix (`systemctl restart systemd-resolved`) was correct either way, but the *reason* it was correct differs depending on which mechanism actually broke — and only checking, not assuming, resolved which one it was.

**On the false-healthy service status (2.4):** `systemctl status` reporting `active (running)` for a process that had been fully suspended (`SIGSTOP`) is a reminder that "the service is running" and "the service is doing its job" are two different claims. `journalctl`, by capturing the service's actual output over time, was the only tool in this scenario that exposed the gap between the two.
