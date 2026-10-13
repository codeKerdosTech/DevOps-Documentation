# modprobe

## What is it?
`modprobe` loads a kernel module into the running kernel, together with every module it depends on, or removes one with `-r`. It is the normal way to manage modules. Loading and removing need root.

## Syntax
```bash
modprobe [OPTIONS] MODULE [PARAMETER=VALUE]
```

## Visual Overview
> `modprobe` reads the dependency list built by `depmod`, loads each dependency first, then the module.

```mermaid
flowchart LR
    A[modprobe NAME] --> B[Read modules.dep]
    B --> C[Load dependencies first]
    C --> D[Load the module]
    D --> E[Module active in kernel]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    class A start
    class B,C,D proc
    class E ok
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
