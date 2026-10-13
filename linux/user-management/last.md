# last

## What is it?
`last` shows the **history of logins and logouts**, newest first. It reads the binary log file `/var/log/wtmp`, which records every login (SSH, console, `su -`/`login` sessions), every logout, every **reboot** and every **shutdown** of the system, with the user, the terminal, the **remote address**, the time and the **duration** of the session. It is the first tool for questions like "who logged in last night?", "from which address?", "when did the server reboot?" and "was this account used?". It is part of the `util-linux` package (older systems: `sysvinit-tools`), on every distribution. The sibling `lastb` reads `/var/log/btmp` and shows the **failed** logins. `lastlog` shows the last login of each account.

## Syntax
```bash
last [OPTIONS] [USER...] [TTY...]
last reboot                  # the history of reboots
last -x shutdown             # shutdowns and runlevel changes
```
With names, `last` shows only the sessions of those users or terminals. The special names `reboot` and `shutdown` show system events. You may need `sudo` to read `/var/log/wtmp` (on many systems it is readable by everybody).

### The columns of the output
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
```
| Column | Meaning |
|--------|---------|
| 1 | The user name (`reboot` and `shutdown` for system events) |
| 2 | The terminal: `pts/N` (SSH or terminal window), `tty1` (the console), `system boot` |
| 3 | The remote host or IP address (for `reboot` rows: the **kernel version**) |
| 4 | The login time |
| 5 | The logout time, or `still logged in`, `still running`, `gone - no logout` (the session ended in an unclean way, for example a crash), `down` (the system went down) or `crash` |
| 6 | The duration of the session: `(hh:mm)` or `(d+hh:mm)` |

## Visual Overview
> `last` reads the binary file `/var/log/wtmp` from the end, pairs each login record with its logout record, and prints them from the newest to the oldest, with filters.

```mermaid
flowchart LR
    A[var log wtmp binary file] --> B[last]
    C[Filters user tty time] --> B
    B --> D[Read records from the end]
    D --> E[Pair login with logout]
    E --> F{Matches filters}
    F -->|Yes| G[Print the line]
    F -->|No| H[Skip]
    G --> I[Print the wtmp begins line]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    classDef alt fill:#FF9F43,stroke:#E67E22,stroke-width:2px,color:#fff
    class A,C start
    class B,D,E proc
    class F dec
    class G ok
    class H alt
    class I out
```

## Options/Flags

### How many lines, which file
| Flag | Description |
|------|-------------|
| `-n NUM` / `-NUM` / `--limit NUM` | Show only `NUM` lines (`last -5` is the same as `last -n 5`) |
| `-f FILE` / `--file FILE` | Read `FILE` instead of `/var/log/wtmp` (for example the rotated `/var/log/wtmp.1`) |
| `-x` / `--system` | Also show the **shutdown** and the **runlevel** changes |

### What the columns show
| Flag | Description |
|------|-------------|
| `-a` / `--hostlast` | Put the host name in the **last column** (easier to read when the name is long) |
| `-d` / `--dns` | Translate the IP address back into a **host name** (a DNS lookup) |
| `-i` / `--ip` | Show the IP address as **numbers** (not names) |
| `-R` / `--nohostname` | Do not show the host name column |
| `-w` / `--fullnames` | Show the full user and domain names (not cut to 8 characters) |
| `-F` / `--fulltimes` | Show **full** login and logout times with the seconds and the year |
| `--time-format FORMAT` | Change the time format: `notime`, `short` (the default), `full` or `iso` |

### Time filters
| Flag | Description |
|------|-------------|
| `-s TIME` / `--since TIME` | Show the records since `TIME` (`YYYY-MM-DD HH:MM:SS`, `yesterday`, `today`, `-1hour`...) |
| `-t TIME` / `--until TIME` | Show the records until `TIME` |
| `-p TIME` / `--present TIME` | Show who was **logged in at** `TIME` |

### Other
| Flag | Description |
|------|-------------|
| `-h` / `--help` | Show the help text |
| `-V` / `--version` | Show the version |

## Usage Examples

Let's say we are on the server `web01`. Now it is `2026-10-12 10:05`. The system was last rebooted on `Oct 5`, and the log file `/var/log/wtmp` begins on `Oct 3`. These sessions are in the file:

**Input (content of `/var/log/wtmp`)**:

| User | Terminal | From | Login | Logout | Duration |
|------|----------|------|-------|--------|----------|
| `deploy` | `pts/0` | `10.0.0.5` | Oct 12 09:45:10 | (still logged in) | |
| `alice` | `pts/1` | `10.0.0.9` | Oct 12 08:10:03 | Oct 12 09:30:41 | 1 h 20 min |
| `bob` | `pts/0` | `10.0.0.7` | Oct 11 14:02:20 | Oct 11 14:50:12 | 47 min |
| `deploy` | `pts/0` | `10.0.0.5` | Oct 11 09:00:05 | Oct 11 17:30:00 | 8 h 29 min |
| `reboot` | system boot | kernel `5.15.0-122-generic` | Oct 5 05:45 | (still running) | |
| `alice` | `tty1` | (console) | Oct 3 12:00 | Oct 3 12:30 | 30 min |
| `shutdown` | system down | kernel `5.15.0-122-generic` | Oct 5 05:40 | | |
| `reboot` | system boot | kernel `5.15.0-119-generic` | Oct 3 06:12 | Oct 5 05:40 | 1 d 23 h 28 min |

Each example starts again from this state.

### Example 1: The login history
**Command:**
```bash
last
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00 - 17:30  (08:29)
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
alice    tty1                          Sat Oct  3 12:00 - 12:30  (00:30)
reboot   system boot  5.15.0-119-gener Sat Oct  3 06:12 - 05:40 (1+23:28)

wtmp begins Sat Oct  3 06:12:01 2026
```
The last line shows how far back the data goes.

### Example 2: Only the last few entries (`-n`, `-NUM`)
**Command:**
```bash
last -n 3
last -3
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)

wtmp begins Sat Oct  3 06:12:01 2026
```
The two commands print the same result. The line `wtmp begins` is always shown.

### Example 3: The history of one user
**Command:**
```bash
last deploy
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00 - 17:30  (08:29)

wtmp begins Sat Oct  3 06:12:01 2026
```

### Example 4: The history of several users, or of one terminal
**Command:**
```bash
last alice bob
last tty1
```
**Sample Output:**
```text
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)
alice    tty1                          Sat Oct  3 12:00 - 12:30  (00:30)

wtmp begins Sat Oct  3 06:12:01 2026
alice    tty1                          Sat Oct  3 12:00 - 12:30  (00:30)

wtmp begins Sat Oct  3 06:12:01 2026
```
The terminal name can be written with or without `/dev/` (`tty1` or `/dev/tty1`).

### Example 5: When was the server rebooted?
`reboot` is a pseudo user.

**Command:**
```bash
last reboot
```
**Sample Output:**
```text
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
reboot   system boot  5.15.0-119-gener Sat Oct  3 06:12 - 05:40 (1+23:28)

wtmp begins Sat Oct  3 06:12:01 2026
```
The kernel version is shown in the third column. After the reboot on Oct 5 the kernel changed from `119` to `122`.

### Example 6: Shutdowns and runlevels (`-x`)
**Command:**
```bash
last -x shutdown reboot
```
**Sample Output:**
```text
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
shutdown system down  5.15.0-119-gener Mon Oct  5 05:40 - 05:45  (00:05)
reboot   system boot  5.15.0-119-gener Sat Oct  3 06:12 - 05:40 (1+23:28)

wtmp begins Sat Oct  3 06:12:01 2026
```
`-x` adds the `shutdown` records (and `runlevel` changes). With it you can see **how** a server went down: a regular `shutdown`, or a `crash` (no shutdown record, and the previous session ends with `crash`).

### Example 7: Full times with the year (`-F`)
**Command:**
```bash
last -F -n 3
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45:10 2026   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10:03 2026 - Mon Oct 12 09:30:41 2026  (01:20)
bob      pts/0        10.0.0.7         Sun Oct 11 14:02:20 2026 - Sun Oct 11 14:50:12 2026  (00:47)

wtmp begins Sat Oct  3 06:12:01 2026
```
It is the right form for an incident report, because the seconds and the year are shown.

### Example 8: ISO time format (`--time-format`)
**Command:**
```bash
last -n 2 --time-format iso
last -n 2 --time-format notime
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         2026-10-12T09:45:10+00:00   still logged in
alice    pts/1        10.0.0.9         2026-10-12T08:10:03+00:00 - 2026-10-12T09:30:41+00:00  (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
deploy   pts/0        10.0.0.5           still logged in
alice    pts/1        10.0.0.9         (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
```
`iso` is easy to parse in scripts. `notime` hides the times. Other formats are `short` and `full`.

### Example 9: Show the host in the last column (`-a`)
**Command:**
```bash
last -a -n 2
```
**Sample Output:**
```text
deploy   pts/0        Mon Oct 12 09:45   still logged in    10.0.0.5
alice    pts/1        Mon Oct 12 08:10 - 09:30  (01:20)     10.0.0.9

wtmp begins Sat Oct  3 06:12:01 2026
```
The address is moved to the end of the line. It is easy to copy, and works with `awk '{print $NF}'`.

### Example 10: Turn the address into a name (`-d`)
**Command:**
```bash
last -d -n 2
```
**Sample Output:**
```text
deploy   pts/0        workstation01    Mon Oct 12 09:45   still logged in
alice    pts/1        laptop-alice     Mon Oct 12 08:10 - 09:30  (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
```
`last` does a reverse DNS lookup. If DNS is slow, `-d` is slow.

### Example 11: Show numbers for the addresses (`-i`)
**Command:**
```bash
last -i -n 2
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
```
`-i` always prints addresses as numbers. For a login that has no address (a local console login), it prints `0.0.0.0`. It never does a DNS lookup, so it is fast.

### Example 12: Do not show the host column (`-R`)
**Command:**
```bash
last -R -n 3
```
**Sample Output:**
```text
deploy   pts/0        Mon Oct 12 09:45   still logged in
alice    pts/1        Mon Oct 12 08:10 - 09:30  (01:20)
bob      pts/0        Sun Oct 11 14:02 - 14:50  (00:47)

wtmp begins Sat Oct  3 06:12:01 2026
```

### Example 13: Long user names (`-w`)
By default the user name is cut to 8 characters. In a domain with names like `jsmith@example.com`, you lose information.

**Command:**
```bash
last -w -n 2 domainuser
```
**Sample Output:**
```text
jane.smith@corp.example.com pts/2 10.0.0.31 Mon Oct 12 07:55 - 08:20  (00:25)
jane.smith@corp.example.com pts/2 10.0.0.31 Sun Oct 11 17:02 - 17:40  (00:38)

wtmp begins Sat Oct  3 06:12:01 2026
```

### Example 14: Only since a time (`-s`)
**Command:**
```bash
last -s 2026-10-12
last -s yesterday -F -n 2
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10 - 09:30  (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
deploy   pts/0        10.0.0.5         Mon Oct 12 09:45:10 2026   still logged in
alice    pts/1        10.0.0.9         Mon Oct 12 08:10:03 2026 - Mon Oct 12 09:30:41 2026  (01:20)

wtmp begins Sat Oct  3 06:12:01 2026
```

### Example 15: Only until a time (`-t`)
**Command:**
```bash
last -t "2026-10-11 12:00" -n 3
```
**Sample Output:**
```text
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00 - 17:30  (08:29)
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
alice    tty1                          Sat Oct  3 12:00 - 12:30  (00:30)

wtmp begins Sat Oct  3 06:12:01 2026
```
`-t` shows the records that started **before** the given time (here `2026-10-11 12:00`), so the sessions of Oct 12 and the session of `bob` (14:02) are missing. The session of `deploy` that began on Oct 11 at 09:00 is shown, because it began before noon.

### Example 16: A time window
**Command:**
```bash
last -s "2026-10-11 08:00" -t "2026-10-11 18:00"
```
**Sample Output:**
```text
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00 - 17:30  (08:29)

wtmp begins Sat Oct  3 06:12:01 2026
```
All the sessions on Oct 11 between 08:00 and 18:00.

### Example 17: Who was logged in at a given time (`-p`)
**Command:**
```bash
last -p "2026-10-11 14:30"
```
**Sample Output:**
```text
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00 - 17:30  (08:29)
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running

wtmp begins Sat Oct  3 06:12:01 2026
```
At 14:30 on Oct 11, `bob` and `deploy` were logged in, and the system was running since Oct 5. `-p now` shows the present sessions.

### Example 18: An older (rotated) log (`-f`)
`logrotate` renames the log monthly. Read the old file with `-f`.

**Command:**
```bash
last -f /var/log/wtmp.1 -n 3
```
**Sample Output:**
```text
alice    pts/0        10.0.0.9         Wed Sep 30 16:12 - 17:01  (00:49)
deploy   pts/0        10.0.0.5         Wed Sep 30 09:05 - 18:00  (08:55)
bob      pts/1        10.0.0.7         Tue Sep 29 11:20 - 11:58  (00:38)

wtmp begins Sun Sep  6 06:05:11 2026
```

### Example 19: A session that was not closed properly
When a server crashes, the sessions have no logout record.

**Command:**
```bash
last -x -n 4 | head -n 4
```
**Sample Output:**
```text
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
deploy   pts/0        10.0.0.5         Sun Oct  4 21:10 - crash  (08:35)
```
`crash` means that the session ended with the crash of the system (no `shutdown` record before the next `reboot`). `gone - no logout` means that the session was cut off without a record.

### Example 20: Failed logins with `lastb` (the sister command)
`lastb` reads `/var/log/btmp`.

**Command:**
```bash
sudo lastb -n 3
```
**Sample Output:**
```text
admin    ssh:notty    203.0.113.77     Mon Oct 12 09:58 - 09:58  (00:00)
root     ssh:notty    203.0.113.77     Mon Oct 12 09:58 - 09:58  (00:00)
test     ssh:notty    198.51.100.14    Mon Oct 12 09:41 - 09:41  (00:00)

btmp begins Sat Oct  3 06:15:12 2026
```
These lines are failed SSH logins from the Internet (a typical password guessing attack).

### Example 21: File not readable
**Command:**
```bash
last -f /var/log/wtmp.2
```
**Sample Output:**
```text
last: cannot open /var/log/wtmp.2: No such file or directory
```

### Example 22: Check the version (`-V`)
**Command:**
```bash
last -V
```
**Sample Output:**
```text
last from util-linux 2.37.2
```

### Example 23: Show the help (`-h`)
**Command:**
```bash
last -h | head -n 4
```
**Sample Output:**
```text

Usage:
 last [options] [<username>...] [<tty>...]

```

## Pitfalls / Gotchas
- **The data may be gone.** The log rotates (`logrotate` with `monthly` and `rotate 1` on many systems), and tools or administrators can delete it. `last` shows only what `/var/log/wtmp` still holds (see `wtmp begins ...`). Read the old files with `-f` (and `zcat` for compressed ones).
- **An attacker can erase or change `wtmp`.** It is a plain file. Do not rely on it alone for a forensic investigation. Send the logs to a central log server (`rsyslog`, `journald` forwarding, `auditd`).
- **`last` shows logins, not what the user did.** For the commands use `auditd`, `sudo` logs, or shell history (which is easy to avoid).
- **SSH sessions without a terminal** (for example `ssh host command`, `scp`, `rsync`) are often **not** in `wtmp`, because no `pty` is created. Look at `/var/log/auth.log` (Debian) or `/var/log/secure` (RHEL), or `journalctl -u ssh`.
- **`su` and `sudo` are not logins.** They do not make a `wtmp` record. Use the auth log.
- **Times are in the local time zone of the server,** and the year is missing in the short format. A log across a year change can be confusing. Use `-F` or `--time-format iso`.
- **`last -d` can be very slow** when DNS is slow, or when the addresses have no reverse name. Use `-i` or no option.
- **The user name is cut to 8 characters** (except with `-w`). Two users with the same first 8 characters look the same.
- **"still logged in" can be wrong.** If a session was closed in an unclean way, `last` may show `still logged in` for a long time until the next reboot writes the correct record. Compare with `who` and `w`.
- **`last` with a user name that has never logged in** prints only the `wtmp begins` line. It does not tell you that the user does not exist.
- **Containers:** most containers have no `/var/log/wtmp`. The command fails with `cannot open /var/log/wtmp`. Use the log of the host.
- **On `systemd` hosts** `journalctl` and `loginctl list-sessions` show the sessions too, and the `wtmp` file is still written by `sshd` and `login`.
- **Time values with `-s` and `-t`** accept many forms (`yesterday`, `-2hours`, `today`), but not all versions. Test your form first, and use `YYYY-MM-DD HH:MM:SS` for scripts.

## DevOps Use Cases

### Use Case 1: Who was logged in during an incident?
**Situation:** The database went down on Oct 11 at 14:30. Find out who had a session at that time.

**Command:**
```bash
last -p "2026-10-11 14:30" -F | grep -v -E '^(reboot|wtmp|$)'
```
**Output:**
```text
bob      pts/0        10.0.0.7         Sun Oct 11 14:02:20 2026 - Sun Oct 11 14:50:12 2026  (00:47)
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00:05 2026 - Sun Oct 11 17:30:00 2026  (08:29)
```
Then look at `/var/log/auth.log` for `sudo` commands in that period.

### Use Case 2: When did the server last reboot? (monitoring)
**Situation:** A check script alerts when a server rebooted in the last 24 hours (maybe an unplanned reboot).

**Command:**
```bash
last -F reboot -n 1 | head -n 1
uptime -s
```
**Output:**
```text
reboot   system boot  5.15.0-122-generic Mon Oct  5 05:45:20 2026   still running
2026-10-05 05:45:19
```
`uptime -s` and `who -b` also show the boot time.

### Use Case 3: Find the logins from an unknown address
**Situation:** Security wants to know which addresses were used to log in, and how often.

**Command:**
```bash
last -i -w | awk 'NF>=5 && $3 ~ /^[0-9]+\./ {print $3}' | sort | uniq -c | sort -rn
```
**Output:**
```text
      3 10.0.0.5
      1 10.0.0.9
      1 10.0.0.7
```
An address that should not be there (for example one from outside the company network) is a finding.

### Use Case 4: Check that an account is really unused before you remove it
**Situation:** Before you delete `bob`, check when he last logged in.

**Command:**
```bash
last -n 1 bob
lastlog -u bob
```
**Output:**
```text
bob      pts/0        10.0.0.7         Sun Oct 11 14:02 - 14:50  (00:47)

wtmp begins Sat Oct  3 06:12:01 2026
Username         Port     From             Latest
bob              pts/0    10.0.0.7         Sun Oct 11 14:02:20 +0000 2026
```
`lastlog` is not limited by the rotation of `wtmp`.

### Use Case 5: A report of failed SSH logins (attack check)
**Situation:** See how many failed logins there were today, and from where.

**Command:**
```bash
sudo lastb -i | awk '{print $3}' | grep -E '^[0-9]' | sort | uniq -c | sort -rn | head -n 3
```
**Output:**
```text
   412 203.0.113.77
   130 198.51.100.14
    18 192.0.2.45
```
Block the addresses (firewall, `fail2ban`), and use SSH keys and `PasswordAuthentication no`.

### Use Case 6: Total time that a user was logged in
**Situation:** Billing for a shared server: add the hours of `deploy` on Oct 11.

**Command:**
```bash
last -F -s 2026-10-11 -t 2026-10-12 deploy | awk -F'[()]' '/\(/ {split($2,a,":"); t+=a[1]*60+a[2]} END{printf "%d h %d min\n", t/60, t%60}'
```
**Output:**
```text
8 h 29 min
```

### Use Case 7: A daily report by mail
**Situation:** A cron job mails the list of logins of the last day to the security team.

**Input file** (`/etc/cron.d/login-report`):
```text
0 7 * * * root last -F -s yesterday | mail -s "Logins on $(hostname)" security@example.com
```
**Command:**
```bash
last -F -s yesterday | head -n 3
```
**Output:**
```text
bob      pts/0        10.0.0.7         Sun Oct 11 14:02:20 2026 - Sun Oct 11 14:50:12 2026  (00:47)
deploy   pts/0        10.0.0.5         Sun Oct 11 09:00:05 2026 - Sun Oct 11 17:30:00 2026  (08:29)
```

### Use Case 8: Find a crash in the history
**Situation:** The server was unavailable for a while. Was it a regular shutdown, or a crash?

**Command:**
```bash
last -x reboot shutdown crash | head -n 4
```
**Output:**
```text
reboot   system boot  5.15.0-122-gener Mon Oct  5 05:45   still running
shutdown system down  5.15.0-119-gener Mon Oct  5 05:40 - 05:45  (00:05)
reboot   system boot  5.15.0-119-gener Sat Oct  3 06:12 - 05:40 (1+23:28)
```
A `shutdown` record before each `reboot` means a clean stop. A `reboot` without a `shutdown` before it means a crash or a power loss.

## Related Commands
- [`who`](who.md) - who is logged in **now**
- [`w`](w.md) - who is logged in now, and what they run
- [`id`](id.md) / [`whoami`](whoami.md) - your own identity
- [`passwd`](passwd.md) / [`chage`](chage.md) - check the state of an account
- [`usermod`](usermod.md) - lock an account that you find suspicious
- [`userdel`](userdel.md) - remove an unused account
- [`getent`](getent.md) - read the account databases
- [`tail`](../file-management/tail.md) - follow `/var/log/auth.log`
- [`journalctl`](../services-logging/journalctl.md) - read the `systemd` journal of logins
- `lastb` - failed logins (`/var/log/btmp`)
- `lastlog` - the last login of every account
- `uptime -s` / `who -b` - the boot time
- `ac` - total connect time of users
- `loginctl list-sessions` - the sessions of `systemd-logind`
