# lsmod

## What is it?
`lsmod` lists the **kernel modules** that are loaded right now, with the size of each and which other modules use it. It prints a formatted view of `/proc/modules`.

## Syntax
```bash
lsmod
```

## Visual Overview
> `lsmod` lists the kernel modules that are loaded right now. `Used by` shows what depends on each module.

```mermaid
flowchart LR
    i1[Kernel<br/>loaded modules] -->|lsmod| o1[Module  Size  Used by<br/>e1000  126976  0<br/>nf_tables  299008  45]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class i1 inp
    class o1 out
```

## Options/Flags
`lsmod` takes no options. Use `grep`, `wc` and `sort` on its output.

| Column | Meaning |
|--------|---------|
| `Module` | Module name |
| `Size` | Memory used, in bytes |
| `Used by` | Use count, then the modules that depend on this one |

## Usage Examples

### Example 1: List loaded modules
**Command:**
```bash
lsmod | head -4
```
**Sample Output:**
```text
Module                  Size  Used by
ipt_REJECT             12288  9
nf_reject_ipv4         12288  1 ipt_REJECT
btrfs                1716224  0
```

### Example 2: Count loaded modules
**Command:**
```bash
lsmod | wc -l
```
**Sample Output:**
```text
40
```
Subtract one for the header line: 39 modules.

### Example 3: Is one module loaded
**Command:**
```bash
lsmod | grep -E '^(btrfs|xt_nat) '
```
**Sample Output:**
```text
btrfs                1716224  0
xt_nat                 12288  22
```

### Example 4: The raw source
**Command:**
```bash
head -1 /proc/modules
```
**Sample Output:**
```text
ipt_REJECT 12288 9 - Live 0x0000000000000000
```

## Pitfalls / Gotchas
- Features compiled **into** the kernel do not appear. Check `/sys/module/NAME` as well.
- `grep -w NAME` or `grep '^NAME '` avoids partial matches such as `nf_nat` inside `nf_nat_ipv4`.
- `lsmod` is read only and works for any user. Loading and unloading need root.

## DevOps Use Cases

### Use Case 1: Verify modules before starting Kubernetes or Docker
**Situation:** A node needs certain modules.

**Command:**
```bash
for m in xt_nat ipt_REJECT overlay br_netfilter; do
  grep -q "^$m " /proc/modules && echo "$m loaded" || echo "$m missing"
done
```
**Sample Output:**
```text
xt_nat loaded
ipt_REJECT loaded
overlay missing
br_netfilter loaded
```

### Use Case 2: Add the module count to a support ticket
**Situation:** A vendor asks for the kernel version, module count and taint state.

**Command:**
```bash
echo "kernel=$(uname -r) modules=$(($(lsmod | wc -l) - 1)) tainted=$(cat /proc/sys/kernel/tainted)"
```
**Sample Output:**
```text
kernel=6.18.54.1-microsoft-standard-WSL2 modules=39 tainted=0
```

## Related Commands
- [`modinfo`](modinfo.md) - details about one module
- [`modprobe`](modprobe.md) - load and unload modules
- [`dmesg`](dmesg.md) - kernel messages when modules load
- [Kernel support and modules](../os-internals/06-kernel-support-and-modules.md) - the concept page

---
<!-- nav -->
[← Previous: `lsblk`](lsblk.md) | [Learning path](../README.md) | [Next: `modinfo` →](modinfo.md)
