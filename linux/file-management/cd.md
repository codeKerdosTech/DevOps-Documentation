# cd

## What is it?
`cd` (change directory) moves your shell into another directory. It is a shell built-in, not a separate program.

## Syntax
```bash
cd [DIRECTORY]
cd -
cd ~
cd ..
```

## Visual Overview
> `cd` only moves your shell to another folder. Each form below starts from the same place: `/home/ravi/app`.

```mermaid
flowchart LR
    n1[You are in<br/>/home/ravi/app] -->|cd logs| o1[folder<br/>/home/ravi/app/logs]
    n1 -->|cd /var/log| o2[folder<br/>/var/log]
    n1 -->|cd ..| o3[parent folder<br/>/home/ravi]
    n1 -->|cd or cd ~| o4[your home folder<br/>/home/ravi]
    n1 -->|cd -| o5[the folder you were<br/>in before]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class n1 note
    class o1,o2,o3,o4,o5 out
```

## Options/Flags
| Argument / Flag | Description |
|-----------------|-------------|
| `DIRECTORY` | Absolute or relative path to move into |
| `..` | Go up one level to the parent directory |
| `~` or no argument | Go to your home directory |
| `-` | Go back to the previous directory |
| `-P` | Resolve symbolic links to the physical path |

## Usage Examples

Let's say we have this directory layout:

**Input file** (directory tree):
```text
/home/devops/
├── project/
│   └── logs/
└── releases -> /opt/releases/v2   (symbolic link)
```

### Example 1: Move into a directory
**Command:**
```bash
cd /home/devops/project
pwd
```
**Sample Output:**
```text
/home/devops/project
```

### Example 2: Relative path
**Command:**
```bash
cd logs
pwd
```
**Sample Output:**
```text
/home/devops/project/logs
```

### Example 3: Go up one level (`..`)
**Command:**
```bash
cd ..
pwd
```
**Sample Output:**
```text
/home/devops/project
```

### Example 4: Go home (`~`)
**Command:**
```bash
cd ~
pwd
```
**Sample Output:**
```text
/home/devops
```

### Example 5: Go back to the previous directory (`-`)
**Command:**
```bash
cd project
cd -
```
**Sample Output:**
```text
/home/devops
```

### Example 6: Resolve symlinks (`-P`)
**Command:**
```bash
cd -P /home/devops/releases
pwd
```
**Sample Output:**
```text
/opt/releases/v2
```

## Pitfalls / Gotchas
- `cd` is a shell built-in, so it cannot change the directory of the parent shell from a script. A script that runs `cd` only changes its own working directory.
- Paths with spaces need quotes: `cd "My Folder"`.
- `cd -` prints the directory it moves to.
- A failed `cd` in a script keeps running the next lines in the wrong directory. Use `cd /path || exit 1`.

## Related Commands
- [`pwd`](pwd.md) - print the current directory
- [`ls`](ls.md) - list directory contents
- `pushd` / `popd` - directory stack
