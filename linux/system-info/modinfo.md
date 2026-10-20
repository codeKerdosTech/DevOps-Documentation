# modinfo

## What is it?
`modinfo` shows information about a kernel module: its file, description, license, dependencies and parameters. It reads the module file, so it works for modules that are not loaded.

## Syntax
```bash
modinfo [OPTIONS] MODULE
```

## Visual Overview
> `modinfo` reads the details stored inside a module file without loading it.

```mermaid
flowchart LR
    c1[modinfo e1000] --> i1[Finds the module file<br/>for the running kernel] --> o1[filename<br/>description<br/>author<br/>depends<br/>parm options]
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
| `-F FIELD` | Print only one field (`filename`, `description`, `license`, `depends`, `parm`, `version`) |
| `-n` | Print only the file name |
| `-p` | Print only the parameters |
| `-k VERSION` | Use the modules of another kernel version |

## Usage Examples

### Example 1: Description (`-F description`)
**Command:**
```bash
modinfo -F description btrfs
```
**Sample Output:**
```text
B-Tree File System (BTRFS)
```

### Example 2: License (`-F license`)
**Command:**
```bash
modinfo -F license btrfs
```
**Sample Output:**
```text
GPL
```
`GPL` means open source. A `Proprietary` license taints the kernel.

### Example 3: Dependencies (`-F depends`)
**Command:**
```bash
modinfo -F depends btrfs
```
**Sample Output:**
```text
xor,raid6_pq
```

### Example 4: File name (`-F filename`, `-n`)
**Command:**
```bash
modinfo -F filename xt_nat
modinfo -n xt_nat
```
**Sample Output:**
```text
/lib/modules/6.18.54.1-microsoft-standard-WSL2/kernel/net/netfilter/xt_nat.ko
/lib/modules/6.18.54.1-microsoft-standard-WSL2/kernel/net/netfilter/xt_nat.ko
```

### Example 5: A short description of another module
**Command:**
```bash
modinfo -F description xt_nat
```
**Sample Output:**
```text
SNAT and DNAT targets support
```

## Pitfalls / Gotchas
- `modinfo` looks in `/lib/modules/$(uname -r)`. After a kernel update without reboot it can say `Module not found`.
- It does not tell you whether the module is loaded. Use [`lsmod`](lsmod.md).
- Modules built into the kernel have no `.ko` file and are not found.

## DevOps Use Cases

### Use Case 1: Check a vendor driver license before loading it
**Situation:** Security policy forbids proprietary kernel modules.

**Command:**
```bash
modinfo -F license btrfs
```
**Sample Output:**
```text
GPL
```

### Use Case 2: Find out what a module needs
**Situation:** `modprobe` fails and you want to know which dependency is missing.

**Command:**
```bash
modinfo -F depends btrfs
```
**Sample Output:**
```text
xor,raid6_pq
```

## Related Commands
- [`lsmod`](lsmod.md) - loaded modules
- [`modprobe`](modprobe.md) - load a module with its dependencies
- [`uname`](uname.md) - the kernel version whose modules are searched
- [Kernel support and modules](../os-internals/06-kernel-support-and-modules.md) - the concept page
