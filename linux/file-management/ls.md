# ls

<!-- start-here -->
> **Start here.** On your first read, learn only these. The rest of the page is reference you can come back to.
>
> | Learn this first | What it does |
> |---|---|
> | `ls` | list the names in this folder |
> | `ls -l` | long list: permissions, owner, size, date |
> | `ls -a` | also show hidden files (names that start with a dot) |
> | `ls -lh` | long list with sizes like 4K or 2M instead of bytes |
> | `ls -lt` | newest files first |
<!-- /start-here -->

## What is it?
`ls` lists the files and directories inside a directory. It is the first command you use to see what is in a folder, check permissions, sizes and modification times.

## Syntax
```bash
ls [OPTIONS] [FILE or DIRECTORY]...
```

## Visual Overview
> Each column of `ls -l` answers one question about the file. Read the line from left to right.

```mermaid
flowchart LR
    c1[-rwxr-xr--  2  ravi dev<br/>4096  Jan 5 10:00  app.sh] --> o1[-rwxr-xr--<br/>type and permissions]
    c1 --> o2[2<br/>number of hard links]
    c1 --> o3[ravi and dev<br/>owner and group]
    c1 --> o4[4096<br/>size in bytes]
    c1 --> o5[Jan 5 10:00<br/>last modified]
    c1 --> o6[app.sh<br/>name]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1 cmd
    class o1,o2,o3,o4,o5,o6 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-l` | Long format: permissions, owner, size, modification time |
| `-a` | Show all entries including hidden files (names starting with `.`) |
| `-h` | Human readable sizes (use with `-l`) |
| `-t` | Sort by modification time, newest first |
| `-r` | Reverse the sort order |
| `-S` | Sort by file size, largest first |
| `-R` | List sub-directories recursively |
| `-d` | List the directory itself, not its contents |
| `-1` | One entry per line |

## Usage Examples

Let's say we have a directory `project/` with this content (name, size, last modified):

**Input file** (`project/` directory):
```text
.env          24 bytes     Oct  3 12:00   (hidden file)
app.py        2048 bytes   Oct  6 09:15
logs/         4096 bytes   Oct  6 11:40   (directory, contains app.log)
notes.txt     12 bytes     Oct  5 18:20
readme.md     1536 bytes   Oct  1 08:00
```

All examples below are run from inside `project/`, and the output assumes an `en_US.UTF-8` locale (case-insensitive sorting).

### Example 1: Basic listing
**Command:**
```bash
ls
```
**Sample Output:**
```text
app.py  logs  notes.txt  readme.md
```

### Example 2: Long format (`-l`)
**Command:**
```bash
ls -l
```
**Sample Output:**
```text
total 16
-rw-r--r-- 1 devops devops 2048 Oct  6 09:15 app.py
drwxr-xr-x 2 devops devops 4096 Oct  6 11:40 logs
-rw-r--r-- 1 devops devops   12 Oct  5 18:20 notes.txt
-rw-r--r-- 1 devops devops 1536 Oct  1 08:00 readme.md
```

### Example 3: Show hidden files (`-a`)
**Command:**
```bash
ls -a
```
**Sample Output:**
```text
.  ..  .env  app.py  logs  notes.txt  readme.md
```

### Example 4: Human readable sizes (`-h`)
**Command:**
```bash
ls -lh
```
**Sample Output:**
```text
total 16K
-rw-r--r-- 1 devops devops 2.0K Oct  6 09:15 app.py
drwxr-xr-x 2 devops devops 4.0K Oct  6 11:40 logs
-rw-r--r-- 1 devops devops   12 Oct  5 18:20 notes.txt
-rw-r--r-- 1 devops devops 1.5K Oct  1 08:00 readme.md
```

### Example 5: Newest first (`-t`)
**Command:**
```bash
ls -t
```
**Sample Output:**
```text
logs  app.py  notes.txt  readme.md
```

### Example 6: Reverse order (`-r`)
**Command:**
```bash
ls -rt
```
**Sample Output:**
```text
readme.md  notes.txt  app.py  logs
```

### Example 7: Largest first (`-S`)
**Command:**
```bash
ls -S
```
**Sample Output:**
```text
logs  app.py  readme.md  notes.txt
```

### Example 8: Recursive (`-R`)
Inside `logs/` there is one file, `app.log`.

**Command:**
```bash
ls -R
```
**Sample Output:**
```text
.:
app.py  logs  notes.txt  readme.md

./logs:
app.log
```

### Example 9: Directory itself (`-d`)
**Command:**
```bash
ls -ld logs
```
**Sample Output:**
```text
drwxr-xr-x 2 devops devops 4096 Oct  6 11:40 logs
```

### Example 10: One per line (`-1`)
**Command:**
```bash
ls -1
```
**Sample Output:**
```text
app.py
logs
notes.txt
readme.md
```

## Pitfalls / Gotchas
- Hidden files (starting with `.`) are not shown unless you use `-a`.
- `ls -l` on a directory name lists its contents; use `-d` to see the directory itself.
- Sorting depends on the locale. In the `C` locale uppercase names sort before lowercase names.
- Do not parse `ls` output in scripts, because file names with spaces or newlines break it. Use globs or `find` instead.
- Many distros alias `ls` to `ls --color=auto`. Colors disappear when output is piped.

## Try it yourself

Work in a practice folder so nothing real can be damaged: `mkdir -p ~/practice && cd ~/practice`. Try each task first, then open the answer.

**Task 1.** Let's say we set up this practice folder:
```bash
mkdir ls-lab && cd ls-lab
touch a.txt .hidden
echo hello > b.txt
```
List it normally.

<details>
<summary>Show the answer</summary>

```bash
ls
```
Output:
```
a.txt  b.txt
```
The hidden file is not shown.

</details>

**Task 2.** Show the hidden file too.

<details>
<summary>Show the answer</summary>

```bash
ls -a
```
Output:
```
.  ..  .hidden  a.txt  b.txt
```

</details>

**Task 3.** What size in bytes does `ls -l` show for `b.txt`?

<details>
<summary>Show the answer</summary>

```bash
ls -l b.txt
```
The size column is `6`: the word `hello` is 5 characters plus the new line.

</details>

## Related Commands
- [`cd`](cd.md) - change directory
- [`pwd`](pwd.md) - print the current directory
- [`mkdir`](mkdir.md) - create directories
- `find` - search files with conditions
- `tree` - show a directory as a tree

---
<!-- nav -->
[← Previous: `pwd`](pwd.md) | [Learning path](../README.md) | [Next: `cd` →](cd.md)
