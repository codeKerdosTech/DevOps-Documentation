# mkdir

## What is it?
`mkdir` (make directory) creates one or more new directories.

## Syntax
```bash
mkdir [OPTIONS] DIRECTORY...
```

## Visual Overview
> `mkdir` creates a folder. Add `-p` to create every missing parent folder in one go.

```mermaid
flowchart TD
    c1[mkdir -p app/logs/2025] --> o1[app] --> o2[logs] --> o3[2025]
    c2[mkdir app/logs/2025<br/>without -p, app does not exist] --> x1[Error<br/>No such file or directory]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1,c2 cmd
    class o1,o2,o3 out
    class x1 bad
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-p` | Create parent directories as needed; no error if the directory already exists |
| `-v` | Print a message for each created directory |
| `-m MODE` | Set permissions (like `chmod`) on the new directory |

## Usage Examples

Let's say we are in an empty directory `/home/devops/work`:

**Input file** (`/home/devops/work`):
```text
(empty directory)
```

### Example 1: Create a directory
**Command:**
```bash
mkdir backups
ls
```
**Sample Output:**
```text
backups
```

### Example 2: Create several directories
**Command:**
```bash
mkdir dev staging prod
ls
```
**Sample Output:**
```text
backups  dev  prod  staging
```

### Example 3: Create nested directories (`-p`)
**Command:**
```bash
mkdir -p app/config/env
ls -R app
```
**Sample Output:**
```text
app:
config

app/config:
env

app/config/env:
```

### Example 4: Show what is created (`-v`)
**Command:**
```bash
mkdir -v logs
```
**Sample Output:**
```text
mkdir: created directory 'logs'
```

### Example 5: Set permissions (`-m`)
**Command:**
```bash
mkdir -m 700 private
ls -ld private
```
**Sample Output:**
```text
drwx------ 2 devops devops 4096 Oct  6 10:00 private
```

### Example 6: Error without `-p`
**Command:**
```bash
mkdir x/y/z
```
**Sample Output:**
```text
mkdir: cannot create directory 'x/y/z': No such file or directory
```

## Pitfalls / Gotchas
- Without `-p`, `mkdir` fails if the parent is missing or the directory already exists.
- `-p` hides errors for existing directories, which is useful in scripts because it makes them re-runnable.
- Quote names with spaces: `mkdir "my folder"`.
- Use brace expansion for many folders: `mkdir -p project/{src,test,docs}`.

## Try it yourself

Work in a practice folder so nothing real can be damaged: `mkdir -p ~/practice && cd ~/practice`. Try each task first, then open the answer.

**Task 1.** Create a folder named `project`.

<details>
<summary>Show the answer</summary>

```bash
mkdir project
ls
```

</details>

**Task 2.** Create `project/src/utils` in one command, even though `src` does not exist yet.

<details>
<summary>Show the answer</summary>

```bash
mkdir -p project/src/utils
```
Without `-p` this fails because the parent `src` is missing.

</details>

**Task 3.** Run `mkdir project` again. What happens? How do you make it harmless?

<details>
<summary>Show the answer</summary>

```bash
mkdir project
```
Output: `mkdir: cannot create directory 'project': File exists`.
With `mkdir -p project` there is no error.

</details>

## Related Commands
- [`rmdir`](rmdir.md) - remove empty directories
- [`rm`](rm.md) - remove files and directories
- [`touch`](touch.md) - create files
- [`ls`](ls.md) - list directories

---
<!-- nav -->
[← Previous: `tree`](tree.md) | [Learning path](../README.md) | [Next: `touch` →](touch.md)
