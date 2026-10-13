# Wildcards and Globbing

## What is it?
A **wildcard** is a special character that stands for other characters in a file name. **Globbing** is the shell expanding a pattern such as `*.log` into the list of matching names **before** the command runs. The command itself never sees the `*`, it receives the finished list, for example `app.log error.log`.

## Syntax
```bash
command pattern
ls *.log
ls a?.txt
ls [ab]1.txt
ls {app,error}.log
```

## Visual Overview
> You type a pattern. Bash looks in the directory, replaces the pattern with every matching name, and only then starts the command.

```mermaid
flowchart LR
    A[You type ls star dot log] --> B[Bash reads the pattern]
    B --> C[Bash lists matching names]
    C --> D{Any match}
    D --> E[Replace pattern with the names]
    D --> F[Keep the pattern as typed]
    E --> G[Run ls app.log error.log]
    F --> G
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    class A start
    class B,C proc
    class D dec
    class E,F ok
    class G out
```

## Options/Flags
| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any number of characters, including none | `*.log` |
| `?` | Exactly one character | `a?.txt` |
| `[abc]` | One character from the set | `[ab]1.txt` |
| `[a-z]` | One character from the range | `[a-c]*.txt` |
| `[!abc]` | One character **not** in the set | `[!a]*.txt` |
| `{x,y}` | Brace expansion, builds several words (not a real wildcard) | `{app,error}.log` |
| `~` | Your home directory | `~/logs` |

A leading dot is not matched by `*` or `?`, so hidden files are skipped unless you write the dot yourself, for example `.*`.

For all examples let's say we have these files in the current directory (all empty) and one empty directory `logs`:

**Input file** (listing of the directory, from `ls -a`):
```text
.
..
.hidden
a1.txt
a10.txt
a2.txt
app.log
app.txt
b1.txt
error.log
logs
notes.md
```

## Usage Examples

### Example 1: Everything that ends in `.log` (`*`)
**Command:**
```bash
ls *.log
```
**Sample Output:**
```text
app.log
error.log
```

### Example 2: Exactly one character (`?`)
**Command:**
```bash
ls a?.txt
```
**Sample Output:**
```text
a1.txt
a2.txt
```
`a10.txt` is not listed, it has two characters between `a` and `.txt`.

### Example 3: Any length (`*`)
**Command:**
```bash
ls a*.txt
```
**Sample Output:**
```text
a1.txt
a10.txt
a2.txt
app.txt
```

### Example 4: A set of characters (`[ab]`)
**Command:**
```bash
ls [ab]1.txt
```
**Sample Output:**
```text
a1.txt
b1.txt
```

### Example 5: A range (`[a-b]`)
**Command:**
```bash
ls [a-b]1.txt
```
**Sample Output:**
```text
a1.txt
b1.txt
```

### Example 6: Not in the set (`[!a]`)
**Command:**
```bash
ls [!a]*.txt
```
**Sample Output:**
```text
b1.txt
```
Only `b1.txt` starts with something other than `a`.

### Example 7: Brace expansion (`{ }`)
**Command:**
```bash
ls {app,error}.log
echo file{1,2,3}.txt
```
**Sample Output:**
```text
app.log
error.log
file1.txt file2.txt file3.txt
```
Braces do not look at the disk. `echo` prints names even when the files do not exist.

### Example 8: Nested braces build a directory tree
**Command:**
```bash
mkdir -p dir/{a,b}/{x,y}
find dir -type d | sort
```
**Sample Output:**
```text
dir
dir/a
dir/a/x
dir/a/y
dir/b
dir/b/x
dir/b/y
```

### Example 9: Hidden files are not matched
**Command:**
```bash
ls -a | head -5
ls *hidden
```
**Sample Output:**
```text
.
..
.hidden
a1.txt
a10.txt
ls: cannot access '*hidden': No such file or directory
```
Write the dot to match a hidden file, for example `ls .hid*`.

### Example 10: A pattern that matches nothing
**Command:**
```bash
echo *.nomatch
```
**Sample Output:**
```text
*.nomatch
```
When nothing matches, bash passes the pattern through unchanged. A command like `ls *.nomatch` then complains that the file `*.nomatch` does not exist.

### Example 11: Home directory (`~`)
**Command:**
```bash
echo ~
```
**Sample Output:**
```text
/home/devops
```

### Example 12: Expansion of `*` shows the list the command receives
**Command:**
```bash
echo *.md *.log
```
**Sample Output:**
```text
notes.md app.log error.log
```

## Sample Output
See the exact words the command receives:

```bash
echo a?.txt
```
```text
a1.txt a2.txt
```

## Pitfalls / Gotchas
- Wildcards are expanded by the **shell**, not by `ls`, `rm` or `cp`. That is why `rm *.log` deletes the matched names without asking `rm` to search.
- `*` does not match `/`, so `*.log` does not look inside sub directories. Use `find . -name '*.log'` or `**` after `shopt -s globstar`.
- `*` does not match hidden files, so `rm *` leaves `.hidden` behind.
- Wildcards are not regular expressions. In a glob `*` means "anything", in a regex `.*` does.
- Always run `echo rm *.log` or `ls *.log` first to preview what a pattern matches. A typo like `rm * .log` (with a space) deletes everything.
- Quote the pattern when you want the program to see it, for example `find . -name '*.log'`. Without quotes the shell expands it first.
- Too many matches can fail with `Argument list too long`. Use `find ... -exec` or `xargs` instead.

## DevOps Use Cases

### Use Case 1: Count log files for a service
**Situation:** Check how many rotated logs exist before cleaning up.

**Command:**
```bash
ls *.log | wc -l
```
**Sample Output:**
```text
2
```

### Use Case 2: Back up a file with brace expansion
**Situation:** Make a copy of a config before editing it.

**Command:**
```bash
cp app.log{,.bak}
ls app.log*
```
**Sample Output:**
```text
app.log
app.log.bak
```
`app.log{,.bak}` expands to `app.log app.log.bak`.

### Use Case 3: Run a step on every log file
**Situation:** A script must process each log in a folder.

**Command:**
```bash
for f in *.log; do echo "processing $f"; done
```
**Sample Output:**
```text
processing app.log
processing error.log
```

## Related Commands
- [Quoting](07-quoting.md) - stop the shell from expanding a pattern
- [Paths](05-paths.md) - the previous topic
- [`ls`](../file-management/ls.md) - list files
- [`find`](../file-management/find.md) - search by name without shell limits
- [`rm`](../file-management/rm.md) - always preview a pattern before using it here
- [`xargs`](../text-processing/xargs.md) - handle very long file lists
- [`echo`](../shell-basics/echo.md) - preview what a pattern expands to
