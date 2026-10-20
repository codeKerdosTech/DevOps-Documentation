# modprobe

## What is it?
`modprobe` loads a kernel module into the running kernel, together with every module it depends on, or removes one with `-r`. It is the normal way to manage modules. Loading and removing need root.

## Syntax
```bash
modprobe [OPTIONS] MODULE [PARAMETER=VALUE]
```

## Visual Overview
> `modprobe` loads a module and, unlike `insmod`, loads everything it depends on first.

```mermaid
flowchart TD
    c1[modprobe name] --> i1[Read modules.dep<br/>to find the dependencies]
    i1 --> c2[Load the dependencies first]
    c2 --> o1[Load the module<br/>it is now active]
    c3[modprobe -r name] --> x1[Unload the module]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1,c2,c3 cmd
    class i1 inp
    class o1 out
    class x1 bad
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-r` | Remove the module (and unused dependencies) |
| `-n` | Dry run, do nothing |
| `-v` | Verbose, show the `insmod` and `rmmod` steps |
| `-c` | Show the merged configuration from `/etc/modprobe.d` |
| `-l`, `--show-depends` | List the dependencies that would be loaded |

## Usage Examples

### Example 1: Load a module that is already loaded
**Command:**
```bash
modprobe xt_nat
echo rc=$?
```
**Sample Output:**
```text
rc=0
```
Success prints nothing, and loading twice is harmless. Loading a new module needs root, use `sudo modprobe NAME`.

### Example 2: Dry run (`-n -v`)
**Command:**
```bash
modprobe -n -v ipt_REJECT
echo rc=$?
```
**Sample Output:**
```text
rc=0
```
The module is already loaded, so there are no steps to show.

### Example 3: A module that does not exist
**Command:**
```bash
modprobe nosuchmod
echo rc=$?
```
**Sample Output:**
```text
modprobe: FATAL: Module nosuchmod not found in directory /lib/modules/6.18.54.1-microsoft-standard-WSL2
rc=1
```

### Example 4: Compare with `insmod`
**Command:**
```bash
insmod /nonexist.ko
echo rc=$?
```
**Sample Output:**
```text
insmod: ERROR: could not load module /nonexist.ko: No such file or directory
rc=1
```
`insmod` loads one file and does not resolve dependencies. Prefer `modprobe`.

## Pitfalls / Gotchas
- Needs root to load or remove. Without it you get `Operation not permitted`.
- `modprobe -r` fails when the module is in use, see the `Used by` column of [`lsmod`](lsmod.md).
- A change made with `modprobe` is lost at reboot. List persistent modules in `/etc/modules-load.d/*.conf`.
- Options are persistent only when put in `/etc/modprobe.d/*.conf`, for example `options NAME key=value`.
- Containers cannot load modules, ask the host.
- `blacklist NAME` in `/etc/modprobe.d/` stops automatic loading but does not stop a manual `modprobe`.

## DevOps Use Cases

### Use Case 1: Check that a needed module is loaded or loadable
**Situation:** A bootstrap script must be sure that a module is available.

**Command:**
```bash
modprobe -n xt_nat && echo "xt_nat available"
modprobe -n nosuchmod 2>/dev/null || echo "nosuchmod not available"
```
**Sample Output:**
```text
xt_nat available
nosuchmod not available
```

## Related Commands
- [`lsmod`](lsmod.md) - loaded modules
- [`modinfo`](modinfo.md) - module details and dependencies
- [`dmesg`](dmesg.md) - kernel messages after loading
- [`sudo`](../user-management/sudo.md) - needed to load modules
- [Kernel support and modules](../os-internals/06-kernel-support-and-modules.md) - the concept page

---
<!-- nav -->
[← Previous: `modinfo`](modinfo.md) | [Learning path](../README.md) | [Next: `mount` →](mount.md)
