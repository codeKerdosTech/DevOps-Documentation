# who

## What is it?
`who` shows who is currently logged in to the system, with their terminal, login time and where they connected from.

## Syntax
```bash
who [OPTIONS]
who am i
```

## Visual Overview
> `who` lists the people logged in right now, from the session records in `/var/run/utmp`.

```mermaid
flowchart LR
    i1[Session records<br/>/var/run/utmp] -->|who| o1[ravi  pts/0  Jan 5 10:00  10.0.0.9<br/>amit  pts/1  Jan 5 10:20  10.0.0.7]
    i1 -->|who -b| o2[time of the last boot]
    i1 -->|who -q| o3[names and the user count]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class i1 inp
    class o1,o2,o3 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-H` | Print a heading line |
| `-b` | Show the time of the last system boot |
| `-q` | Quick: list user names and a count |
| `-u` | Show idle time for each session |
| `-a` | Show all information |
| `am i` | Show only the current terminal session |

## Usage Examples

Let's say these sessions are active on the server:

**Input file** (login sessions, as seen by the system):
```text
devops logged in on pts/0 at 2026-10-06 09:12 from 192.168.1.10
admin  logged in on pts/1 at 2026-10-06 09:45 from 10.0.0.5
```
We are the user `devops` on `pts/0`.

### Example 1: List logged in users
**Command:**
```bash
who
```
**Sample Output:**
```text
devops   pts/0        2026-10-06 09:12 (192.168.1.10)
admin    pts/1        2026-10-06 09:45 (10.0.0.5)
```

### Example 2: With headings (`-H`)
**Command:**
```bash
who -H
```
**Sample Output:**
```text
NAME     LINE         TIME             COMMENT
devops   pts/0        2026-10-06 09:12 (192.168.1.10)
admin    pts/1        2026-10-06 09:45 (10.0.0.5)
```

### Example 3: Last boot time (`-b`)
**Command:**
```bash
who -b
```
**Sample Output:**
```text
         system boot  2026-10-01 07:30
```

### Example 4: Quick count (`-q`)
**Command:**
```bash
who -q
```
**Sample Output:**
```text
devops admin
# users=2
```

### Example 5: Idle time (`-u`)
**Command:**
```bash
who -u
```
**Sample Output:**
```text
devops   pts/0        2026-10-06 09:12   .          2210 (192.168.1.10)
admin    pts/1        2026-10-06 09:45 00:25        2340 (10.0.0.5)
```
(`.` means active in the last minute, `00:25` means idle for 25 minutes; the number is the process ID.)

### Example 6: All information (`-a`)
**Command:**
```bash
who -a
```
**Sample Output:**
```text
           system boot  2026-10-01 07:30
           run-level 5  2026-10-01 07:30
LOGIN      tty1         2026-10-01 07:31               812 id=tty1
devops   + pts/0        2026-10-06 09:12   .          2210 (192.168.1.10)
admin    + pts/1        2026-10-06 09:45 00:25        2340 (10.0.0.5)
```

### Example 7: Only my session (`am i`)
**Command:**
```bash
who am i
```
**Sample Output:**
```text
devops   pts/0        2026-10-06 09:12 (192.168.1.10)
```

## Pitfalls / Gotchas
- `who` only shows interactive logins that are recorded in utmp. Containers and some cron or systemd sessions do not appear.
- `who am i` prints nothing when there is no terminal, for example in a cron job.
- `w` is similar but also shows what each user is running.

## Related Commands
- [`whoami`](whoami.md) - current effective user
- `w` - who is logged in and what they are doing
- `last` - history of past logins
- `id` - user and group IDs
