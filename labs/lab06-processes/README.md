# Lab 6 — Live processes and memory pressure

Pair: Ibraeva Nursaya 
Machine: Lenovo IdeaPad Slim 5 16IRL8 (Windows 11, 16 GB RAM), VirtualBox
Guests: Ubuntu Server 26.04.1 LTS (`lab01-ubuntu`, 2 vCPU, RAM 3.3 GiB lowered to 2 GiB for Part 2, swap 2 GiB) · Windows 11 Enterprise evaluation (`lab02-windows`, 4 GB RAM)
Date: 14–15.11.2026 (times in the Ubuntu logs are guest time, UTC)

## Part 1 — One process, three tools

Process chosen: `python -m http.server 8000` (Python 3.12.10 on Windows, 3.14.4 on Ubuntu) — why: it runs with the same command on both guests, and the VirtualBox Guest Additions service is not installed on the Ubuntu guest (`pgrep -a VBoxService` printed nothing).

Windows guest: PID 9592, parent `powershell.exe` (12116). Ubuntu guest: PID 2077, parent `bash` (1594).

Commands:

```text
# Windows (PowerShell)
Get-Process python | Select-Object Id, ProcessName
Get-CimInstance Win32_Process -Filter "ProcessId=9592" | Select-Object ProcessId, ParentProcessId, CommandLine | Format-List
Get-Process -Id 12116 | Select-Object Id, ProcessName

# Ubuntu
python3 -m http.server 8000 &
ps -o pid,ppid,user,rss,vsz,nlwp,cmd -p 2077
ls /proc/2077/fd | wc -l
sudo ls -l /proc/2077/fd | head
ps -p $(ps -o ppid= -p 2077) -o pid,comm
```

Comparison table (values from the screenshots and text files in `evidence/`):

| | Task Manager (Details tab) | Process Explorer | htop / ps |
|---|---|---|---|
| PID | 9592 | 9592 | 2077 |
| Parent (name and PID) | — (not shown) | `powershell.exe`, 12116 (PPID column, Image tab) | `bash`, 1594 (PPID column, tree view) |
| "Memory" as shown by default | 828 K ("Memory (active private working set)") | no single "Memory" column; first memory columns are Private Bytes and Working Set | MEM% 0.6 (computed from RES) |
| Private memory | Commit size 13,120 K | Private Bytes 13,120 K (WS Private 11,316 K) | no such column |
| Working set / RES | Working set 12,000 K | Working Set 22,516 K | RES 22,512 KB (SHR 10,788 KB) |
| Virtual size | not available in Details | not added to the view | VIRT 35,404 KB (`ps` VSZ 35,404) |
| Handles / file descriptors | 156 | 156 | 4 (0–2 → `/dev/tty1`, 3 → a socket) |
| Threads | 1 | 1 | 1 (NLWP) |

Timing note: the Process Explorer screenshot was taken at 6:02 PM and the Task Manager one at 7:03 PM (Windows guest time), so the working-set numbers are not from the same moment. Private Bytes / Commit size (13,120 K) did not change between the two.

### Where the tools disagree

1. **"Memory" means different things.** Task Manager's default Memory column is the *active private working set* (828 K), a small part of the process. Process Explorer's Working Set (22,516 K) and htop's RES (22,512 KB) both count pages that are shared with other processes (htop shows 10,788 KB as SHR). The same process therefore looks like 828 K, 12,000 K or 22.5 MB depending on the tool. The Working Set also shrank between 6:02 PM and 7:03 PM (22,516 K → 12,000 K) because Windows trims the working set of an idle process, while its commit stayed the same.
2. **Same number, different names.** Task Manager "Commit size" and Process Explorer "Private Bytes" are the same value (13,120 K). htop has no such column; its VIRT (35,404 KB) is the whole address space, not memory in use.
3. **Handles are not file descriptors.** Windows counts every kernel object a process holds (sections, desktop, files, devices, …): 156. Linux lists only open files and sockets in `/proc/<PID>/fd`: 4.
4. **Parent process.** Task Manager does not show it at all. Process Explorer shows it (PPID, Image tab, command line) and so does htop (PPID column or tree view).

## Part 2 — Memory pressure

Guest RAM: 2 GiB (`free -h` shows 1.6 GiB total) · swap: 2 GiB (`/swap.img`). The VM snapshot `before-lab6` was taken before the experiment.

Steps:

```bash
# second terminal (tty2): recording
cd ~/lab06
{ echo "=== BEFORE $(date) ==="; free -h; swapon --show; } > memory-pressure.txt
while true; do echo "=== $(date +%T) ==="; free -h; sleep 3; done | tee -a memory-pressure.txt

# first terminal (tty1): load, one after another, with a marker line before each run
echo "### stress 70% start $(date +%T)" >> memory-pressure.txt
stress-ng --vm 2 --vm-bytes 70% --timeout 60s
stress-ng --vm 2 --vm-bytes 90% --timeout 60s
stress-ng --vm 2 --vm-bytes 120% --timeout 60s
stress-ng --vm 2 --vm-bytes 400% --timeout 60s

# kernel log
sudo journalctl -k --since "90 min ago" | grep -i -E -B3 -A8 "out of memory|oom-kill" > kernel-log-extract.txt
sudo journalctl -k | grep -iE "soft lockup|rcu_preempt|watchdog" > lockup.txt
```

The four runs:

| Run | stress-ng asked for | Result |
|---|---|---|
| 70% | 637 MB of 910 MB available | finished in 5 min 27 s instead of 60 s; kernel: `soft lockup - CPU#0 stuck for 264s` |
| 90% | 798 MB of 887 MB available | 1 min 2.5 s; `soft lockup` 22 s on both CPUs |
| 120% | 999 MB of 833 MB available | 1 min 9.2 s; `soft lockup` 42 s on both CPUs, later 36 s in `snapd` |
| 400% | 4.33 GB of 1.08 GB available | 1 min 5.7 s; **OOM killer at 19:58:37** |

Memory figures (`free -h`, from `evidence/memory-pressure.txt`):

| | Before | During (worst) | After |
|---|---|---|---|
| Available | 1.3 GiB | **15 MiB** (19:58:49) | 1.3 GiB (from 19:58:53 until the last sample, 20:19:02) |
| Swap used | 0 B | **2.0 GiB — swap full** (19:58:35) | 12 MiB (19:58:53), 11 MiB (20:19:02) |

During the 70–120% runs swap used peaked at only 391 MiB. I did not append a separate `AFTER` block; "After" is the first and the last sample of the recording loop after the kill.

What happened (times from `memory-pressure.txt`):

- 19:57:03 — marker line written; `stress-ng` for 400% was typed a little later and started at about 19:57:48.
- 19:57:50 — available drops from 1.2 GiB to 53 MiB, swap starts to grow (133 MiB).
- 19:57:50–19:58:35 — swap grows to 2.0 GiB (212 → 472 → 741 MiB → 1.1 → 1.4 → 1.7 → 1.9 → 2.0 GiB) while available stays at 45–54 MiB.
- 19:58:37 — the kernel kills a `stress-ng-vm` worker (see below).
- 19:58:42 — swap used falls from 2.0 GiB to 1.1 GiB, because the swapped pages of the killed process are released.
- 19:58:49 — lowest available memory (15 MiB), the second worker is still running.
- 19:58:53 — `stress-ng` has finished; available is 1.3 GiB again.

Responsiveness: the sampling loop writes every 3 seconds, but under pressure there are gaps of 6 s (19:57:44 → 19:57:50) and 7 s (19:58:35 → 19:58:42) — the guest was stalling. In the 70/90/120% runs the kernel reported `soft lockup` / `rcu_preempt kthread starved` messages (see `lockup.txt`), and the 70% run took 5 min 27 s instead of 60 s. `stress-ng` still reported `passed: 2: vm (2)` after the kill.

### What the kernel did

```text
Nov 14 19:58:37 lab01-ibraeva kernel: stress-ng-vm invoked oom-killer: gfp_mask=0x140cca(GFP_HIGHUSER_MOVABLE|__GFP_COMP), order=0, oom_score_adj=1000
Nov 14 19:58:37 lab01-ibraeva kernel: Out of memory: Killed process 4069 (stress-ng-vm) total-vm:2340188kB, anon-rss:733240kB, file-rss:520kB, shmem-rss:0kB, UID:1000 pgtables:3568kB oom_score_adj:1000
```

- **Which process:** the `stress-ng-vm` worker with PID 4069. The OOM killer was invoked by its sibling 4068 when that worker asked for a page and neither RAM nor swap had one.
- **Why that one:** the table the kernel prints before the kill shows two almost identical workers (4068 and 4069), both with `oom_score_adj` 1000 (stress-ng sets this on its workers on purpose, so they are the first victims). 4069 held a bit more memory: 183,440 pages in RAM plus 261,449 in swap, against 168,734 + 258,657 for 4068 (about 445k vs 427k pages), so it scored higher. Its `anon-rss` of 733,240 kB is about 716 MiB in RAM, and `total-vm` is 2.2 GiB.
- **When:** 19:58:37. In `memory-pressure.txt` swap reached 2.0 GiB at 19:58:35 — two seconds earlier — and fell back to 1.1 GiB at 19:58:42, which matches about 1 GiB of swapped pages (261,449 × 4 KiB) being freed.

### Windows

The Windows guest never ran low on memory (commit charge was about 50%), so the query returns nothing. Command and result (`evidence/eventlog-win.txt`):

```text
PS> Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='Microsoft-Windows-Resource-Exhaustion-Detector'} -MaxEvents 5 | Format-List TimeCreated, Id, Message
No events were found that match the specified selection criteria.
PS> Get-WinEvent -ListProvider Microsoft-Windows-Resource-Exhaustion-Detector | Select-Object Name

Name
----
Microsoft-Windows-Resource-Exhaustion-Detector
```

The second command shows that the provider exists, so the query is correct and the empty result means that no event was written. Windows does not kill processes like the Linux OOM killer: it warns when the commit charge gets close to the limit (RAM plus page file), allocations start to fail, and event 2004 from this provider names the processes that used the most memory. I did not try Testlimit.

## What did not work the first time

- **The guide's percentages gave no pressure.** `stress-ng` takes the percentage from the *available* memory (about 0.9 GiB), not from the whole RAM, so even 120% asked for only about 1 GB and swap stayed small (max 391 MiB). The OOM killer appeared only at 400%.
- **`soft lockup` on the Ubuntu guest.** The guest stalled several times: during `apt install` (`soft lockup - CPU#0 stuck for 123s`; I pressed Ctrl+C and `apt` printed `received signal 2`, so I ran `sudo apt install -y htop stress-ng` again), and in the 70% run (264 s). The messages are in `lockup.txt`. My assumption is that the guest did not get enough CPU time from the laptop; I did not prove it.
- **`winget install Python.Python.3.12` failed** with `Failed when searching source: msstore … 0x8a15005e : The server certificate did not match any of the expected values.` `winget install -e --id Python.Python.3.12 --source winget` worked. Right after the install `python` was still "not found" in the same PowerShell window; it works in a new window because PATH is only read at start.
- **A script block was not run:** my first `process-win.txt` contained the text of the commands instead of their output, because `{ … } | Out-File` needs `& { … }`. Windows PowerShell 5.1 also writes UTF-16 with `Out-File` by default, so for later files I used `-Encoding utf8`.
- **Typing mistakes in the guest console:** F-keys left `~` in front of a command (`Command '~echo' not found`), `grep -B3-A8` without a space gave `invalid context length argument` (and left an empty `kernel-log-extract.txt`), and one `journalctl` command was typed on the host, where it does not exist.

## Evidence

- [evidence/task-manager.png](evidence/task-manager.png) — Task Manager, Details tab, `python.exe` PID 9592 with Working set, Memory, Commit size, Handles, Threads
- [evidence/process-explorer.png](evidence/process-explorer.png) — Process Explorer: same process, PPID, Private Bytes, Working Set, WS Private, Handles, Threads and the handles pane
- [evidence/htop.png](evidence/htop.png) — htop on Ubuntu: PID 2077, PPID 1594, VIRT, RES, SHR
- [evidence/process-win.txt](evidence/process-win.txt) — PID, parent PID and command line of the Windows process as text
- [evidence/process-ps.txt](evidence/process-ps.txt) — `ps` and `/proc/2077/fd` output on Ubuntu as text
- [evidence/memory-pressure.txt](evidence/memory-pressure.txt) — `free -h` every 3 seconds with a marker before each `stress-ng` run
- [evidence/kernel-log-extract.txt](evidence/kernel-log-extract.txt) — the OOM killer lines from the kernel log with surrounding lines
- [evidence/lockup.txt](evidence/lockup.txt) — `soft lockup` / `rcu_preempt` messages from the kernel log
- [evidence/eventlog-win.txt](evidence/eventlog-win.txt) — the `Get-WinEvent` query and its empty result
