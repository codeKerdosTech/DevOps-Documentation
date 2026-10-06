# pwd

## What is it?
`pwd` (print working directory) prints the absolute path of the directory you are currently in.

## Syntax
```bash
pwd [-L | -P]
```

## Visual Overview
> `pwd` reads the shell's current directory and prints it, either as you typed it (logical) or with symlinks resolved (physical).

```mermaid
flowchart LR
    A[Current directory] --> B{Option}
    B --> C[L logical path]
    B --> D[P physical path]
    C --> E[Print path]
    D --> E
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    classDef alt fill:#FF9F43,stroke:#E67E22,stroke-width:2px,color:#fff
    class A start
    class B dec
    class C,D proc
    class E out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-L` | Logical path: keep symbolic links as you navigated them (default) |
| `-P` | Physical path: resolve all symbolic links |

## Usage Examples

### Example 1: Print the current directory
**Command:**
```bash
cd /var/log
pwd
```
**Sample Output:**
```text
/var/log
```

### Example 2: Logical path with a symlink (`-L`)
Let's say we have a symbolic link `/home/devops/current` that points to `/opt/releases/v2`. This is the link as `ls -l` shows it:

**Input file** (`/home/devops/current`):
```text
lrwxrwxrwx 1 devops devops 15 Oct  6 10:00 /home/devops/current -> /opt/releases/v2
```

**Command:**
```bash
cd /home/devops/current
pwd -L
```
**Sample Output:**
```text
/home/devops/current
```

### Example 3: Physical path with a symlink (`-P`)
Using the same symlink `/home/devops/current` shown in Example 2.

**Command:**
```bash
cd /home/devops/current
pwd -P
```
**Sample Output:**
```text
/opt/releases/v2
```

## Pitfalls / Gotchas
- Plain `pwd` in bash is `-L`. Use `pwd -P` when you need the real location behind a symlink.
- `/bin/pwd` (external program) defaults to physical behavior on some systems, while the shell built-in defaults to logical.
- The `$PWD` variable holds the same value as `pwd -L`.

## Related Commands
- [`cd`](cd.md) - change directory
- [`ls`](ls.md) - list directory contents
- `readlink -f` - resolve any path to its real location
