# groups

## What is it?
`groups` prints the **names of the groups** that a user belongs to. Without arguments, it shows the groups of the current user (you). With one or more user names, it shows the groups of those users, in the form `user : group group ...`. It is a very small command from GNU coreutils: it does the same as `id -Gn`, but with a simpler name. The first group in the list is the **primary group**, the others are the supplementary groups. It is the quickest way to check "is this user in the `docker` group?" or "why can't this user read the file?".

## Syntax
```bash
groups [USER]...
groups --help
groups --version
```
`groups` has no real options, apart from `--help` and `--version`.

## Visual Overview
> `groups` lists the groups a user belongs to. The first one is the primary group.

```mermaid
flowchart LR
    c1[groups ravi] --> o1[ravi : ravi sudo docker<br/>primary group first<br/>then the extra groups]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1 cmd
    class o1 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| *(none)* | Print the groups of the current user |
| `USER...` | Print the groups of each user, in the form `USER : GROUP GROUP ...` |
| `--help` | Show the help text |
| `--version` | Show the version |

### `groups` and `id`: what is the difference?
| | `groups` | `id` |
|-|----------|------|
| Output | Group **names** only | UID, GID and groups with **numbers and names** |
| Same as | `id -Gn` | `id` |
| Several users | `groups alice bob` prints one line for each user with a prefix | `id alice bob` (new versions) |
| Use it for | A quick look, human reading | Scripts (`id -u`, `id -g`), numbers |

## Usage Examples

Let's say we are on the server `web01`. Today is `2026-10-12`. We are logged in as `deploy`. These accounts and groups exist:

**Input file** (`/etc/passwd`):
```text
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
deploy:x:1001:1001:Deploy User,,,:/home/deploy:/bin/bash
alice:x:1002:1002:Alice Smith,,,:/home/alice:/bin/bash
bob:x:1003:1004:Bob Jones,,,:/home/bob:/bin/bash
```
**Input file** (`/etc/group`):
```text
root:x:0:
sudo:x:27:deploy,alice
www-data:x:33:deploy
docker:x:999:deploy
deploy:x:1001:
alice:x:1002:
devs:x:1004:alice
```
`bob` has the primary group `devs` (GID `1004`) and is not listed as a member. Each example starts again from this state.

### Example 1: The groups of the current user
**Command:**
```bash
groups
```
**Sample Output:**
```text
deploy sudo www-data docker
```
The first name, `deploy`, is the primary group.

### Example 2: The groups of another user
**Command:**
```bash
groups alice
```
**Sample Output:**
```text
alice : alice sudo devs
```
Notice the format. With a name, the output starts with `alice :`.

### Example 3: Several users
**Command:**
```bash
groups alice bob www-data
```
**Sample Output:**
```text
alice : alice sudo devs
bob : devs
www-data : www-data
```
`bob` is only in the group `devs`, which is his primary group (he has no private group `bob`).

### Example 4: A user who does not exist
**Command:**
```bash
groups nosuchuser; echo "exit code: $?"
```
**Sample Output:**
```text
groups: ‘nosuchuser’: no such user
exit code: 1
```

### Example 5: Test one group in a script
**Command:**
```bash
groups | grep -qw docker && echo "in docker group" || echo "not in docker group"
```
**Sample Output:**
```text
in docker group
```
`-w` matches whole words, so `docker-users` would not match. For exact tests `id -nG | tr ' ' '\n' | grep -qx docker` is safer.

### Example 6: One group on each line
**Command:**
```bash
groups | tr ' ' '\n'
```
**Sample Output:**
```text
deploy
sudo
www-data
docker
```

### Example 7: Count the groups
**Command:**
```bash
groups | wc -w
```
**Sample Output:**
```text
4
```

### Example 8: The same result with `id`
**Command:**
```bash
groups deploy
id -Gn deploy
```
**Sample Output:**
```text
deploy : deploy sudo www-data docker
deploy sudo www-data docker
```
`id -Gn` has no `deploy :` prefix. This is why it is better in scripts.

### Example 9: The groups of the running session and of the files
After you are added to a group, the running session does not know it. `groups` (no name) shows the session, `groups NAME` reads the files.

**Command:**
```bash
sudo usermod -aG devs deploy
groups
groups deploy
```
**Sample Output:**
```text
deploy sudo www-data docker
deploy : deploy sudo www-data docker devs
```
The session needs a new login (or `newgrp devs`) to get `devs`.

### Example 10: Check the version (`--version`)
**Command:**
```bash
groups --version | head -n 1
```
**Sample Output:**
```text
groups (GNU coreutils) 8.32
```

### Example 11: Show the help (`--help`)
**Command:**
```bash
groups --help | head -n 3
```
**Sample Output:**
```text
Usage: groups [OPTION]... [USERNAME]...
Print group memberships for each USERNAME or, if no USERNAME is specified, for
the current process (which may differ if the groups database has changed).
```

## Pitfalls / Gotchas
- **The output format is different with and without a name.** `groups` prints `deploy sudo docker`. `groups deploy` prints `deploy : deploy sudo docker`. Scripts should use `id -Gn` (it never has the prefix).
- **A new group needs a new login.** `groups` (without a name) shows the groups of the running process. `groups USER` reads the group files and shows the new groups at once. If they differ, the user needs a new session.
- **The primary group is the first.** If the primary group has no members entry in `/etc/group`, it is still shown, because it comes from `/etc/passwd`.
- **It shows names only,** with no numbers. If a group has no name on this server (for example in a container or after `groupdel`), you see the number only. Use `id` for the numbers.
- **No options.** `groups -n` and `groups -a` do not exist. Use `id` for more control.
- **Remote users (LDAP, SSSD, Active Directory)** are included, and the lookup can be slow. A large number of groups is possible.
- **Names with spaces** (from Active Directory, for example `domain users`) break the simple word splitting. Use `id -Gnz`.
- **`groups` on macOS and BSD** has a similar output but sometimes without the primary group. Check the output before you use it in a portable script.
- **It tells nothing about the rights.** A group name does not say what the group may do. Check the group on the files (`ls -l`), `sudoers`, and ACLs.

## DevOps Use Cases

### Use Case 1: Check why a user cannot use Docker
**Situation:** `alice` gets `permission denied while trying to connect to the Docker daemon socket`. Check her groups.

**Command:**
```bash
groups alice
ls -l /var/run/docker.sock
```
**Output:**
```text
alice : alice sudo devs
srw-rw---- 1 root docker 0 Oct 12 09:00 /var/run/docker.sock
```
`alice` is not in the group `docker`. Add her with `sudo usermod -aG docker alice`, and she must log in again.

### Use Case 2: A quick review of a new colleague's access
**Situation:** `bob` is new in the team. Show all his groups before you add more.

**Command:**
```bash
groups bob
```
**Output:**
```text
bob : devs
```

### Use Case 3: Compare the groups of two users
**Situation:** `bob` must get the same access as `alice`. Which groups does he lack?

**Command:**
```bash
diff <(id -Gn alice | tr ' ' '\n' | sort) <(id -Gn bob | tr ' ' '\n' | sort)
```
**Output:**
```text
1d0
< alice
3d1
< sudo
```
`alice` has `alice` (her private group) and `sudo`, which `bob` does not have.

### Use Case 4: A pre-check in a deployment script
**Situation:** A script must write to a directory that belongs to the group `www-data`. It checks the membership first, and gives a clear message.

**Command:**
```bash
if groups | grep -qw www-data; then
  echo "OK: $(whoami) can write to /var/www"
else
  echo "ERROR: $(whoami) is not in the group www-data"
  exit 1
fi
```
**Output:**
```text
OK: deploy can write to /var/www
```

### Use Case 5: List the members of an important group
**Situation:** Who can use `sudo`? `groups` shows the groups of a user. For the members of a group use `getent`.

**Command:**
```bash
getent group sudo
for u in deploy alice; do groups "$u"; done
```
**Output:**
```text
sudo:x:27:deploy,alice
deploy : deploy sudo www-data docker
alice : alice sudo devs
```

### Use Case 6: Check the identity of a service account
**Situation:** A web server must run with the group `www-data` and nothing else.

**Command:**
```bash
groups www-data
sudo -u www-data groups
```
**Output:**
```text
www-data : www-data
www-data
```

## Related Commands
- [`id`](id.md) - the same data with numbers, and better options for scripts (`id -Gn`)
- [`whoami`](whoami.md) - print only the user name
- [`who`](who.md) / [`w`](w.md) - who is logged in
- [`getent`](getent.md) - list the members of a group (`getent group NAME`)
- [`usermod`](usermod.md) - add a user to groups (`usermod -aG`)
- [`gpasswd`](gpasswd.md) - add or remove one group member
- [`groupadd`](groupadd.md) - create a group
- [`chgrp`](../permissions/chgrp.md) - change the group of a file
- [`ls`](../file-management/ls.md) - see the group of a file
- `newgrp` - start a shell that uses another group
