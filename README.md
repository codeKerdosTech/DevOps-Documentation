# DevOps Documentation

Course notes and command references for teaching DevOps. Each tool has its own directory, and each command has its own Markdown file.

## Contents

| Tool | Directory |
|------|-----------|
| Linux | [linux/](linux/) |
| Shell scripting | [shell-script/](shell-script/) |
| Git | [git/](git/) |
| Docker | [docker/](docker/) |
| Ansible | [ansible/](ansible/) |
| Kubernetes | [kubernetes/](kubernetes/) |
| Terraform | [terraform/](terraform/) |
| Jenkins | [jenkins/](jenkins/) |
| AWS | [aws/](aws/) |

## Linux categories

| Category | Directory |
|----------|-----------|
| File management | [linux/file-management/](linux/file-management/) |
| Text processing | [linux/text-processing/](linux/text-processing/) |
| Process management | [linux/process-management/](linux/process-management/) |
| Networking | [linux/networking/](linux/networking/) |
| User management | [linux/user-management/](linux/user-management/) |
| Permissions | [linux/permissions/](linux/permissions/) |
| Package management | [linux/package-management/](linux/package-management/) |
| System info | [linux/system-info/](linux/system-info/) |

## Conventions

- One lowercase, hyphenated `.md` file per command, for example `linux/file-management/ls.md`.
- Group commands into sub-directories by category where a tool has many commands.
- Empty directories hold a `.gitkeep` file; delete it once the directory has real content.

## Command page template

Every command page uses these sections, in this order:

1. **Title**: the command name
2. **What it does**: one or two sentences
3. **Syntax**: the general form of the command
4. **Common options**: a table of options and descriptions
5. **Examples**: commands with a short explanation of each
6. **Practice exercise**: a small task for students
7. **Related commands**: links to other pages
