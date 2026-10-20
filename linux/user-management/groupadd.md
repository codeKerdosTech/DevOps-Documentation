# groupadd

## What is it?
`groupadd` creates a new **group**. A group is a list of users who share the same access rights to files, devices and commands. The command adds a line to `/etc/group` (and to `/etc/gshadow`) and picks a free group ID (GID). It is a low level program of the `shadow-utils` package, the same on all distributions, and not interactive, so it is the right command for scripts. A new group is empty. Add members with `usermod -aG`, `gpasswd -a` or `useradd -G`.

### `groupadd` and `addgroup`: the difference, and when to use which
There is a second command with a similar name, with a different behavior depending on the Linux family.

| | `groupadd` | `addgroup` |
|-|------------|------------|
| What it is | The low level program of the `shadow` package, on **all** distributions | **Debian, Ubuntu:** a friendly **Perl script** (the same script as `adduser`, in the package `adduser`) that creates groups. **RHEL, CentOS, Rocky, AlmaLinux, Fedora:** it **does not exist** (`addgroup: command not found`) |
| Interactive? | No | No, but it prints each step |
| Choose the GID | `-g GID` | `--gid GID` |
| System group | `-r` | `--system` |
| Group that already exists | Error (or silent success with `-f`) | Error (`The group 'x' already exists.`) |
| Add a user to a group | No (use `usermod -aG` or `gpasswd -a`) | Yes: `addgroup USER GROUP` (like `adduser USER GROUP`) |
| Name rules | `[a-z_][a-z0-9_-]*` (up to 32 characters) | `NAME_REGEX` in `/etc/adduser.conf` (can allow more) |
| Config files | `/etc/login.defs` (`GID_MIN`, `GID_MAX`, `SYS_GID_MIN`...) | `/etc/adduser.conf` (`FIRST_GID`, `LAST_GID`, `FIRST_SYSTEM_GID`, `LAST_SYSTEM_GID`) |
| Messages | Silent when it works | `Adding group 'devs' (GID 1005) ... Done.` |
| Good for | **Scripts and automation on every distribution** | **Debian and Ubuntu**, at the keyboard |

**When to use which:**
- **In scripts, Ansible, Docker or cloud-init, use `groupadd`** on every distribution. It is the same everywhere.
- **At the keyboard on Debian or Ubuntu, `addgroup` is nice,** because it prints what it did, and can add a user to a group.
- **On RHEL, Rocky, Alma, CentOS or Fedora, only `groupadd` exists.**
- **For removing a group:** `groupdel` on every system, and `delgroup` on Debian. See `groupdel`.

## Syntax
```bash
groupadd [OPTIONS] GROUP
```
`GROUP` is the name of the new group. It must start with a lower case letter or `_` and may contain lower case letters, digits, `_` and `-`. You need `root` rights.

## Visual Overview
> `groupadd` creates a new group by adding one line to `/etc/group`. The system picks the next free group number, the GID.

```mermaid
flowchart LR
    c1[groupadd devs] --> i1[Picks the next free GID<br/>for example 1005] --> o1[Adds the line<br/>devs:x:1005: to /etc/group]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1 cmd
    class i1 inp
    class o1 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-g GID` / `--gid` | Use this group ID. It must be unique, unless `-o` is used |
| `-o` / `--non-unique` | With `-g`, allow a GID that is already in use |
| `-r` / `--system` | Create a **system group**. The GID comes from the system range (`SYS_GID_MIN` to `SYS_GID_MAX`) |
| `-f` / `--force` | Do not fail if the group exists (exit with `0`). If the GID given with `-g` is in use, pick another one |
| `-p PASSWORD` / `--password` | Set the encrypted group password (a hash). Rarely used |
| `-K KEY=VALUE` / `--key` | Override a setting of `/etc/login.defs` for this call (`GID_MIN`, `GID_MAX`) |
| `-U USER,...` / `--users` | Add these users as members at once (newer `shadow-utils`, 4.14 and later) |
| `-R CHROOT_DIR` / `--root` | Apply the change in a chroot directory |
| `-P PREFIX_DIR` / `--prefix` | Use a prefix directory for the files (newer versions) |
| `-h` / `--help` | Show the help text |

### Exit status
| Code | Meaning |
|------|---------|
| `0` | Success |
| `2` | Invalid command syntax |
| `3` | Invalid argument to an option |
| `4` | GID not unique (when `-o` is not used) |
| `9` | Group name not unique |
| `10` | Cannot update the group file |

### The GID ranges
| | Debian, Ubuntu | RHEL, Rocky, Alma, Fedora |
|-|----------------|---------------------------|
| Normal groups | `1000` to `59999` | `1000` to `60000` |
| System groups (`-r`) | `100` to `999` | `201` to `999` |

## Usage Examples

Let's say we are on the server `web01` and work as `root` (or with `sudo`). Today is `2026-10-12`. These groups and users exist:

**Input file** (`/etc/group`):
```text
root:x:0:
sudo:x:27:deploy
www-data:x:33:
docker:x:999:deploy
deploy:x:1001:
alice:x:1002:
```
The next free GID is `1003`. Each example starts again from this state.

### Example 1: Create a group
**Command:**
```bash
sudo groupadd devs
getent group devs
```
**Sample Output:**
```text
devs:x:1003:
```
The group exists, and has the next free GID. It has no members.

### Example 2: Choose the GID (`-g`)
**Command:**
```bash
sudo groupadd -g 2500 ops
getent group ops
```
**Sample Output:**
```text
ops:x:2500:
```
Use a fixed GID when files on NFS or in a container volume must have the same group on all servers.

### Example 3: A GID that is already used
**Command:**
```bash
sudo groupadd -g 1001 ops
```
**Sample Output:**
```text
groupadd: GID '1001' already exists
```
The exit code is `4`.

### Example 4: Allow a duplicate GID (`-o`)
**Command:**
```bash
sudo groupadd -o -g 1001 ops
getent group 1001
```
**Sample Output:**
```text
deploy:x:1001:
ops:x:1001:
```
Two names have the same GID. They are the same group for file permissions. This is not good practice, but it is sometimes used to create an alias.

### Example 5: A system group (`-r`)
**Command:**
```bash
sudo groupadd -r myapp
getent group myapp
```
**Sample Output:**
```text
myapp:x:998:
```
The GID is in the system range (below `1000`). Use it for the groups of services and for group based access to files and devices.

### Example 6: A group that already exists
**Command:**
```bash
sudo groupadd docker; echo "exit code: $?"
```
**Sample Output:**
```text
groupadd: group 'docker' already exists
exit code: 9
```

### Example 7: Make it safe to run again (`-f`)
With `-f`, an existing group is not an error. This makes scripts idempotent.

**Command:**
```bash
sudo groupadd -f docker; echo "exit code: $?"
sudo groupadd -f devs; echo "exit code: $?"
```
**Sample Output:**
```text
exit code: 0
exit code: 0
```
The first group existed already, nothing was done. The second group was created.

### Example 8: `-f` with a GID that is taken
With `-f`, if the GID given with `-g` is in use, `groupadd` chooses another one.

**Command:**
```bash
sudo groupadd -f -g 1001 ops; echo "exit code: $?"
getent group ops
```
**Sample Output:**
```text
exit code: 0
ops:x:1003:
```

### Example 9: Set the group password (`-p`)
`-p` takes a hash. Group passwords are rarely used today.

**Command:**
```bash
hash=$(openssl passwd -6 'GroupPass1')
sudo groupadd -p "$hash" secret
sudo getent gshadow secret | cut -c1-24
```
**Sample Output:**
```text
secret:$6$f2vK1oQz$Hh8a1
```
With a password, a user who is not a member can use `newgrp secret` and type it. It is a weak method.

### Example 10: Change a setting for one call (`-K`)
Use a GID range for the groups of a project, without changing `/etc/login.defs`.

**Command:**
```bash
sudo groupadd -K GID_MIN=5000 -K GID_MAX=5999 project1
getent group project1
```
**Sample Output:**
```text
project1:x:5000:
```

### Example 11: Add members at creation (`-U`)
Newer versions of `shadow-utils` (4.14 and later, for example Fedora 39+) can add members when they create the group.

**Command:**
```bash
sudo groupadd -U alice,deploy devs
getent group devs
```
**Sample Output:**
```text
devs:x:1003:alice,deploy
```
On older systems (`groupadd: invalid option -- 'U'`), use `usermod -aG devs alice`.

### Example 12: Create a group in a chroot (`-R`)
**Command:**
```bash
sudo groupadd -R /mnt/newroot devs
grep devs /mnt/newroot/etc/group
```
**Sample Output:**
```text
devs:x:1000:
```

### Example 13: A bad group name
**Command:**
```bash
sudo groupadd "Bad Name"
sudo groupadd 1abc
```
**Sample Output:**
```text
groupadd: 'Bad Name' is not a valid group name
groupadd: '1abc' is not a valid group name
```
The exit code is `3`.

### Example 14: A bad GID
**Command:**
```bash
sudo groupadd -g abc devs
sudo groupadd -g -5 devs
```
**Sample Output:**
```text
groupadd: invalid group ID 'abc'
groupadd: invalid group ID '-5'
```

### Example 15: Not enough rights
**Command:**
```bash
groupadd devs
```
**Sample Output:**
```text
groupadd: Permission denied.
groupadd: cannot lock /etc/group; try again later.
```

### Example 16: Add members after the creation
A new group is empty. Add users with `usermod`, `gpasswd` or `useradd`.

**Command:**
```bash
sudo groupadd devs
sudo usermod -aG devs alice
sudo gpasswd -a deploy devs
getent group devs
```
**Sample Output:**
```text
Adding user deploy to group devs
devs:x:1003:alice,deploy
```

### Example 17: What `addgroup` does on Debian and Ubuntu
**Command:**
```bash
sudo addgroup devs
sudo addgroup --gid 2500 ops
sudo addgroup --system myapp
sudo addgroup alice devs
```
**Sample Output:**
```text
Adding group `devs' (GID 1003) ...
Done.
Adding group `ops' (GID 2500) ...
Done.
Adding group `myapp' (GID 998) ...
Done.
Adding user `alice' to group `devs' ...
Adding user alice to group devs
Done.
```
On RHEL and Rocky Linux: `sudo: addgroup: command not found`. Use `groupadd` (and `usermod -aG` for the last case).

### Example 18: Show the help (`-h`)
**Command:**
```bash
groupadd -h | head -n 4
```
**Sample Output:**
```text
Usage: groupadd [options] GROUP

Options:
  -f, --force                   exit successfully if the group already exists,
```

## Pitfalls / Gotchas
- **A new group is empty,** and it gives nobody any rights. Add members (`usermod -aG`), and set the group on files (`chgrp`) and the group permissions (`chmod g+rw`).
- **New memberships need a new login.** A user who is added to a group must log out and in again, or run `newgrp GROUP`.
- **Fixed GIDs help on shared storage.** NFS and container volumes use **numbers**. Create the same group with the same `-g` on all servers.
- **Never reuse a GID of a deleted group without a check.** Old files still have that number, and the new group gets access to them. Search with `find / -xdev -gid N`.
- **`-o` makes confusing aliases.** Use it only when you know why.
- **`-f` changes the meaning of `-g`.** With `-f`, a taken GID is silently replaced by another one. The script then continues with a different GID than you expected.
- **A user and a group can have the same name.** With the usual setup, every user has a private group. Creating a group with the name of a future user (`groupadd alice`) makes `useradd alice` fail or use the existing group (`useradd` prints a message and uses it only with `-g`).
- **Group names are case sensitive,** and should be lower case.
- **`groupadd` changes only the local files.** It does nothing for groups in LDAP or Active Directory.
- **`addgroup` is not portable.** It exists on Debian and Ubuntu, and does not exist on RHEL. Use `groupadd` in scripts.
- **On Debian, `addgroup USER GROUP` adds a user to a group,** while `groupadd` has no such form.

## DevOps Use Cases

### Use Case 1: Prepare a shared directory for a team
**Situation:** The team `devs` works in `/srv/shared`. Create the group, add the members, and prepare the directory with the setgid bit.

**Command:**
```bash
sudo groupadd -f devs
sudo usermod -aG devs alice
sudo usermod -aG devs deploy
sudo mkdir -p /srv/shared
sudo chgrp devs /srv/shared
sudo chmod 2775 /srv/shared
ls -ld /srv/shared
getent group devs
```
**Output:**
```text
drwxrwsr-x 2 root devs 4096 Oct 12 10:05 /srv/shared
devs:x:1003:alice,deploy
```

### Use Case 2: A group for a service, with a fixed GID
**Situation:** Several servers share an NFS volume. The group `appdata` must have the GID `3000` everywhere.

**Command:**
```bash
sudo groupadd -g 3000 appdata
getent group appdata
```
**Output:**
```text
appdata:x:3000:
```
On each server, the same command gives the same number, so the group on the files is the same.

### Use Case 3: An idempotent group in a provisioning script
**Situation:** A bootstrap script must create groups, and can be run again without errors.

**Command:**
```bash
for g in devs ops docker; do
  sudo groupadd -f "$g"
done
getent group devs ops docker | cut -d: -f1,3
```
**Output:**
```text
devs:1003
ops:1004
docker:999
```

### Use Case 4: Give a service access to a device or a socket by a group
**Situation:** A monitoring agent must read `/var/log/app` that is readable only by the group `logreaders`. Create the group, and let the agent user join.

**Command:**
```bash
sudo groupadd -r logreaders
sudo chgrp -R logreaders /var/log/app
sudo chmod -R g+rX /var/log/app
sudo usermod -aG logreaders monitoring
id monitoring
```
**Output:**
```text
uid=998(monitoring) gid=997(monitoring) groups=997(monitoring),996(logreaders)
```

### Use Case 5: Create the same groups in a Docker image
**Situation:** A container must have a group with a fixed GID, which is also used by the volume of the host.

**Input file** (`Dockerfile`):
```text
FROM ubuntu:22.04
RUN groupadd -g 2000 appgroup && useradd -m -u 10001 -g appgroup appuser
USER appuser
CMD ["id"]
```
**Command:**
```bash
docker build -t myapp . && docker run --rm myapp
```
**Output:**
```text
uid=10001(appuser) gid=2000(appgroup) groups=2000(appgroup)
```
On Alpine Linux use `addgroup -g 2000 appgroup`, which is the BusyBox form (`-g` for the GID, no long options).

### Use Case 6: Ansible
**Situation:** The same with infrastructure code. The module calls `groupadd` when needed.

**Input file** (`playbook.yml`):
```text
- name: Create the devs group
  ansible.builtin.group:
    name: devs
    gid: 2500
    state: present
```
**Command:**
```bash
ansible-playbook -i web01, playbook.yml
```
**Output:**
```text
TASK [Create the devs group] ***
changed: [web01]
```

### Use Case 7: Find a free GID
**Situation:** You must choose a GID that does not collide with the GIDs on other servers. Check the used ones, first.

**Command:**
```bash
getent group | cut -d: -f3 | sort -n | tail -n 3
```
**Output:**
```text
1001
1002
65534
```
The user group `nogroup` (`65534`) is the top. Choose a number in a range that your company reserves for its own groups (for example `3000` to `3999`).

## Related Commands
- [`addgroup`](groupadd.md) - the Debian friendly command (described above)
- [`groupmod`](groupmod.md) - rename a group or change its GID
- [`groupdel`](groupdel.md) - remove a group (on Debian also `delgroup`)
- [`gpasswd`](gpasswd.md) - add and remove members, set group admins
- [`usermod`](usermod.md) - `usermod -aG` adds a user to a group
- [`useradd`](useradd.md) - `useradd -G` puts a new user into groups
- [`adduser`](adduser.md) - `adduser --group` and `adduser USER GROUP` on Debian
- [`id`](id.md) / [`groups`](groups.md) - check the groups of a user
- [`getent`](getent.md) - read the group database (also LDAP)
- [`chgrp`](../permissions/chgrp.md) - set the group of a file
- [`chmod`](../permissions/chmod.md) - give the group its rights on files
- `newgrp` - use a group without a new login
