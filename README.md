# DevOps Documentation

Course notes and command references for teaching DevOps. Each tool has its own directory, and each command has its own Markdown file.

## Contents

| Tool | Directory |
|------|-----------|
| Linux | [linux/](linux/), start with the [learning path](linux/README.md) |
| Shell scripting | [shell-script/](shell-script/), start with the [series index](shell-script/README.md) |
| Git | [git/](git/) |
| Docker | [docker/](docker/) |
| Ansible | [ansible/](ansible/) |
| Kubernetes | [kubernetes/](kubernetes/) |
| Terraform | [terraform/](terraform/) |
| Jenkins | [jenkins/](jenkins/) |
| AWS | [aws/](aws/) |

## Linux categories

**New to Linux? Start with the [Linux learning path](linux/README.md).** It lists the folders in the order you should read them. The table below follows the same order.

| Step | Category | Directory |
|------|----------|-----------|
| 1 | Fundamentals (read first) | [linux/fundamentals/](linux/fundamentals/) |
| 2 | Shell basics | [linux/shell-basics/](linux/shell-basics/) |
| 3 | File management | [linux/file-management/](linux/file-management/) |
| 3 | Editors (nano, vim) | [linux/editors/](linux/editors/) |
| 4 | Text processing | [linux/text-processing/](linux/text-processing/) |
| 5 | Permissions | [linux/permissions/](linux/permissions/) |
| 5 | User management | [linux/user-management/](linux/user-management/) |
| 6 | Process management | [linux/process-management/](linux/process-management/) |
| 6 | System info | [linux/system-info/](linux/system-info/) |
| 7 | Archiving and compression | [linux/archiving/](linux/archiving/) |
| 7 | Package management | [linux/package-management/](linux/package-management/) |
| 7 | Services and logging | [linux/services-logging/](linux/services-logging/) |
| 7 | Scheduling (cron, crontab) | [linux/scheduling/](linux/scheduling/) |
| 8 | Networking | [linux/networking/](linux/networking/) |
| 9 | OS internals (kernel, processes, threads) | [linux/os-internals/](linux/os-internals/) |

## Conventions

- One lowercase, hyphenated `.md` file per command, for example `linux/file-management/ls.md`.
- Group commands into sub-directories by category where a tool has many commands.
- Empty directories hold a `.gitkeep` file; delete it once the directory has real content.

## Command page template

Every command page follows [_template.md](_template.md), with these sections in this order:

1. **Title**: the command name
2. **What is it?**: one or two sentences
3. **Syntax**: the general form of the command
4. **Visual Overview**: a Mermaid `flowchart LR` or `flowchart TD` diagram
5. **Options/Flags**: a table of options and descriptions
6. **Usage Examples**: every example shows the input file, the exact command and the exact output
7. **Sample Output**: what a typical run prints
8. **Pitfalls / Gotchas**: common mistakes
9. **DevOps Use Cases**: optional, only for commands with real DevOps relevance
10. **Related Commands**: links to other pages
