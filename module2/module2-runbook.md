# Module 2 Runbook — System Troubleshooting

Environment: V*****-VM, Ubuntu Server 25.04, Oracle VirtualBox on Asus M16 (2023).
Source logs: `module2_session01.log`, `module2_session02.log`, `module2_session03.log`

---

## 2.1 — Process Management

**Problem:** Identify and correctly terminate a runaway/misbehaving process.

**Environment:** V*****-VM, single-user session, no prior processes of note.

**Commands Used:**
```bash
yes > /dev/null &
ps aux | grep yes
top
kill <PID>
kill -9 <PID>
ps aux | grep yes
```

**Observed Behavior:** `yes` pinned a single CPU core near 100% utilization while overall system CPU/memory remained healthy (multi-core VM). Plain `kill` produced a clean `Terminated` result. A second instance was cleanly terminated with `kill -9`, producing `Killed`. Both terminations were confirmed via a follow-up `ps aux | grep`.

**Root Cause:** N/A — deliberate, controlled scenario (not a real fault). Process identification and correct signal selection were the actual skills under test.

**Resolution:** Correct PID identified via `ps aux`/`top`; appropriate signal (`kill` first, `kill -9` for the second instance) applied.

**Verification:** `ps aux | grep yes` returned empty after each termination.

**Lessons Learned:** `kill` (SIGTERM) requests a graceful shutdown; `kill -9` (SIGKILL) forces immediate termination with no cleanup. Default to plain `kill` first — `-9` is appropriate only when a process doesn't respond, since it can leave corrupted state behind on anything mid-write to disk.

**Incident note:** This session included a genuine unplanned VM freeze following a second `yes` process — the desktop environment became unresponsive and required a full VM power-cycle to recover. Root cause was inconclusive; CPU/memory were not near exhaustion per the last captured `top` snapshot, ruling out simple resource contention from `yes` itself. Recovery was completed independently and is documented in `troubleshooting-notes.md`.

---

## 2.2 — Services

**Problem:** Diagnose and recover a systemd service that fails to start due to a configuration error.

**Environment:** V*****-VM, fresh `nginx` install.

**Commands Used:**
```bash
sudo apt update && sudo apt install nginx -y
systemctl status nginx
curl localhost
sudo nano /etc/nginx/nginx.conf   # semicolon removed from worker_connections directive
sudo systemctl restart nginx
sudo nginx -t
journalctl -u nginx -n 30 --no-pager
```

**Observed Behavior:** After removing a semicolon from the `worker_connections` directive, `systemctl restart nginx` failed. `nginx -t` reported a syntax error at the closing brace (`}`) on the line **after** the actual fault — not at the missing semicolon itself, since nginx's parser doesn't detect the problem until it fails to close the block correctly.

**Root Cause:** Missing statement terminator (`;`) on the `worker_connections` directive inside the `events {}` block.

**Resolution:** Reasoned backward from the reported error line to the actual fault line; restored the missing semicolon.

**Verification:** `nginx -t` returned "syntax is ok / test is successful"; `systemctl status nginx` returned to `active (running)`; `curl localhost` confirmed the service was functionally serving content, not just reporting healthy.

**Lessons Learned:** Config parser errors often point to where the parser gave up, not where the mistake actually is — reasoning backward from the reported line is a general skill, not nginx-specific. Verifying at two levels (service status vs. actual function) catches failures that "active" alone would miss. Most config-driven services support a test/check-config flag (`-t`, `--test`) — always validate before restarting a hand-edited config.

---

## 2.3 — Networking Troubleshooting

**Problem:** Diagnose a DNS resolution failure and distinguish it from a routing/connectivity failure.

**Environment:** V*****-VM.

**Commands Used:**
```bash
sudo tee /etc/resolv.conf <<< "nameserver <BAD_IP>"
ping -c 4 8.8.8.8
ping -c 4 google.com
dig
nslookup
ls -l /etc/resolv.conf
resolvectl status
sudo systemctl restart systemd-resolved
cat /etc/resolv.conf
```

**Observed Behavior:** `ping` to a known IP succeeded, ruling out routing/upstream failure. `ping` to a hostname failed with "Temporary failure in name resolution." `dig`/`nslookup` returned "connection refused," confirming the resolver itself was misconfigured rather than simply unreachable. `ls -l` confirmed `/etc/resolv.conf` remained a valid symlink to `systemd-resolved`'s managed file — the bad nameserver had been written *through* the symlink into the underlying file, not replacing the symlink itself.

**Root Cause:** `/etc/resolv.conf` (a `systemd-resolved`-managed symlink) had been overwritten with an invalid nameserver entry, out of sync with `systemd-resolved`'s actual internal DNS configuration.

**Resolution:** `sudo systemctl restart systemd-resolved` — regenerates the managed file from the service's real configuration rather than hand-editing the symlinked file directly.

**Verification:** Post-restart, `/etc/resolv.conf` returned to the `systemd-resolved`-managed stub (`127.0.0.53`), and `ping google.com` resolved and returned successfully. The scenario was independently re-broken and re-fixed a second time to confirm the fix wasn't coincidental.

**Lessons Learned:** The diagnostic ladder (ping IP → ping hostname → dig/nslookup) reliably isolates DNS failures from routing failures in three quick steps. `/etc/resolv.conf` on modern Ubuntu is typically a symlink to a `systemd-resolved`-managed file — direct edits get written through the symlink, and the correct fix is restarting the managing service, not re-editing the file by hand.

---

## 2.4 — Log Analysis

**Problem:** Diagnose a service failure using log evidence alone, with no live observation of the failure occurring.

**Environment:** V*****-VM, custom `widget.service` (a minimal heartbeat-logging systemd unit built for this scenario).

**Commands Used:**
```bash
sudo systemctl start widget.service
systemctl status widget.service
sudo kill -STOP <PID>
systemctl status widget.service
journalctl -u widget.service -n 20 --no-pager
ps aux | grep widget
sudo systemctl restart widget.service      # first fix attempt
sudo kill -STOP <PID>                       # re-run for correct-fix demonstration
kill -CONT <PID>                            # failed: Operation not permitted
sudo kill -CONT <PID>                       # succeeded
journalctl -u widget.service -f
```

**Observed Behavior:** After `kill -STOP`, `systemctl status` continued reporting `active (running)` with no visible change — a false-healthy status. `journalctl` was the only source that revealed the actual failure: heartbeat log entries stopped generating at the exact moment the signal was sent. `ps aux` confirmed the process in `T` (stopped) state.

**Root Cause:** Process suspended via `SIGSTOP`, not crashed or exited — a state invisible to `systemctl status` but fully visible in the log's activity gap.

**Resolution:** Initial fix (`systemctl restart`) worked but replaced the process entirely (new PID) rather than resuming the original — recognized as a functionally different fix from what the failure actually required. Re-ran the scenario and applied `kill -CONT` on the original PID, correctly recovering from an `Operation not permitted` error (signal requires root/owner privilege) by re-running with `sudo`.

**Verification:** `journalctl -u widget.service -f` showed heartbeat logging resume on the original PID, confirming the exact same process was restored rather than replaced.

**Lessons Learned:** Service status output can report "healthy" while the service does nothing useful — logs are often the only reliable evidence of actual function. Different failure modes require matching fixes: a frozen (stopped) process should be resumed (`SIGCONT`), not restarted, if preserving process state and continuity matters. Signal permissions follow standard Unix ownership rules — sending a signal to a root-owned process requires elevated privilege.
