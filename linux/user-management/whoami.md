# whoami

## What is it?
`whoami` prints the user name of the effective user ID of the current process, which means the user the commands run as right now.

## Syntax
```bash
whoami [OPTION]
```

## Visual Overview
> `whoami` looks up the effective user ID of the shell and prints the matching user name.

```mermaid
flowchart LR
    A[Shell process] --> B[Effective UID]
    B --> C[Look up in passwd]
    C --> D[Print user name]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    classDef alt fill:#FF9F43,stroke:#E67E22,stroke-width:2px,color:#fff
    class A start
    class B,C proc
    class D ok
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `--help` | Show help and exit |
| `--version` | Show version and exit |

## Usage Examples

### Example 1: Show the current user
**Command:**
```bash
whoami
```
**Sample Output:**
```text
devops
```

### Example 2: Show the user when running with sudo
**Command:**
```bash
sudo whoami
```
**Sample Output:**
```text
root
```

### Example 3: Show help (`--help`)
**Command:**
```bash
whoami --help
```
**Sample Output:**
```text
Usage: whoami [OPTION]...
Print the user name associated with the current effective user ID.
Same as id -un.

      --help        display this help and exit
      --version     output version information and exit
```
(Extra lines with online help links are omitted.)

### Example 4: Show the version (`--version`)
**Command:**
```bash
whoami --version
```
**Sample Output:**
```text
whoami (GNU coreutils) 8.32
```
(The version number depends on your system, and licence lines are omitted.)

## Pitfalls / Gotchas
- `whoami` shows the effective user. After `su` or `sudo` it shows the new user, not the one who logged in. Use `logname` or `who am i` for the login user.
- `whoami` is the same as `id -un`.
- In containers it often prints `root`, even if you never logged in as root.

## Related Commands
- [`who`](who.md) - list logged in users
- [`sudo`](sudo.md) - run a command as another user
- `id` - show user and group IDs
- `logname` - show the login name
