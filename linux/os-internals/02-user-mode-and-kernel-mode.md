# User Mode and Kernel Mode

## What is it?
A modern CPU can run code at different **privilege levels**. Linux uses two of them:

- **Kernel mode** (also called supervisor mode, ring 0): the code can run every CPU instruction and touch all memory and hardware. Only the kernel and its drivers run here.
- **User mode** (ring 3): the code is restricted. It cannot access hardware directly and can only use its own memory. All normal programs run here: your shell, `ls`, `nginx`, `python`.

This split protects the system. A buggy program in user mode can crash itself, but it cannot corrupt the kernel or other programs.

## Syntax
There is no command to enter kernel mode. A program switches into it by making a **system call**, see [System calls](03-system-calls.md). You can observe the split with:

```bash
time COMMAND          # shows user time and sys (kernel) time
top                   # CPU line: us (user) and sy (system, kernel)
vmstat 1 2            # columns us, sy
```

## Visual Overview
> A program runs in user mode. When it needs a service it makes a system call, the CPU switches to kernel mode, the kernel does the work, and the CPU switches back.

```mermaid
flowchart LR
    A[Program runs in user mode] --> B[Needs a service such as reading a file]
    B --> C[System call instruction]
    C --> D[CPU switches to kernel mode]
    D --> E[Kernel checks the request and does the work]
    E --> F[CPU switches back to user mode]
    F --> A
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    class A start
    class B,C proc
    class D,E err
    class F ok
```

## Options/Flags
| | User mode | Kernel mode |
|---|-----------|-------------|
| CPU ring | Ring 3 | Ring 0 |
| Who runs here | Applications, shells, libraries | Kernel core, device drivers, kernel modules |
| Memory access | Only its own virtual memory | All memory, including all processes |
| Hardware access | Not allowed directly | Allowed |
| Privileged CPU instructions | Not allowed | Allowed |
| Crash result | The process is killed (for example `Segmentation fault`) | The whole system stops (**kernel panic** or "oops") |
| Time shown by `time` | `user` | `sys` |
| CPU line in `top` | `us` | `sy` |

| Event | What happens |
|-------|--------------|
| System call | A program asks the kernel for a service, a planned switch to kernel mode |
| Interrupt | Hardware (disk, network card, timer) signals the CPU, the kernel handler runs |
| Exception or fault | The program did something illegal (bad address, divide by zero), the kernel decides what to do |

The switch itself, called a **context switch** or **mode switch**, costs time. Programs that make millions of tiny system calls are slow.

Do not confuse the CPU privilege level with the Linux user `root`. `root` is a normal **user mode** process with extra permission checks removed. It still has to ask the kernel.

## Usage Examples

### Example 1: User mode cannot touch hardware memory
**Command:**
```bash
echo x > /dev/mem
```
**Sample Output:**
```text
bash: /dev/mem: Permission denied
```
Even the open request goes through the kernel, which refuses it.

### Example 2: A user mode crash does not harm the system
Let's say we have this file `seg.c`, a program that writes to address zero:

**Input file** (`seg.c`):
```c
int main(void){ int *p = 0; *p = 1; return 0; }
```

**Command:**
```bash
gcc -o seg seg.c
./seg
echo rc=$?
```
**Sample Output:**
```text
bash: line 9:  2529 Segmentation fault      ./seg
rc=139
```
The kernel noticed the illegal access, sent the process `SIGSEGV` and ended it. Exit code `139` is `128 + 11` (signal 11). Every other program kept running.

### Example 3: Time spent in user mode (`time`)
**Command:**
```bash
time python3 -c 'sum(i*i for i in range(3000000))'
```
**Sample Output:**
```text

real	0m0.528s
user	0m0.520s
sys	0m0.004s
```
This is pure calculation. Almost all the time is `user`, the process rarely needs the kernel.

### Example 4: Time spent in kernel mode, many system calls
**Command:**
```bash
time dd if=/dev/zero of=/dev/null bs=1 count=2000000 2>/dev/null
```
**Sample Output:**
```text

real	0m3.135s
user	0m0.614s
sys	0m2.530s
```
`bs=1` makes `dd` do 2 million tiny reads and 2 million tiny writes. Most of the time is `sys`, spent in the kernel.

### Example 5: The same work with fewer system calls
**Command:**
```bash
time dd if=/dev/zero of=/dev/null bs=1M count=2000 2>/dev/null
```
**Sample Output:**
```text

real	0m0.116s
user	0m0.004s
sys	0m0.112s
```
This moves 2000 MB, about 1000 times more data than Example 4, in 3 percent of the time, because it does only 2000 reads and writes. Bigger blocks mean fewer mode switches.

### Example 6: See the split on the whole machine (`top`)
**Command:**
```bash
top -bn1 | sed -n 3p
```
**Sample Output:**
```text
%Cpu(s): 25.1 us,  6.8 sy,  0.0 ni, 61.6 id,  2.7 wa,  0.0 hi,  3.8 si,  0.0 st
```
`us` is time in user mode, `sy` is time in kernel mode, `id` is idle, `wa` is waiting for disk, `hi` and `si` are interrupt handling.

### Example 7: `vmstat` shows it per second
**Command:**
```bash
vmstat 1 2
```
**Sample Output:**
```text
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 7  0      0 4526768  73140 2248996    0    0  3287   122  833 1012  6  9 81  4  0
 2  0      0 4465320  73164 2285012    0    0 29020   308 11148 14872  8  6 83  2  0
```
The first line is the average since boot, the second is the last second. `in` is interrupts per second and `cs` is context switches per second.

## Sample Output
How to read a `time` line:

```text
real  0m3.135s   wall clock time you waited
user  0m0.614s   CPU time in user mode
sys   0m2.530s   CPU time in kernel mode
```

## Pitfalls / Gotchas
- `real` can be smaller than `user + sys` on multi core machines, because several CPUs work at the same time.
- A high `sy` (system) percentage in `top` means programs or the kernel are busy with system calls, interrupts or I/O. Look with `strace -c` or `perf` for the cause.
- A high `wa` means the CPUs are idle waiting for slow disk or network, not busy.
- Being `root` does not mean running in kernel mode. Only code inside the kernel and loaded modules runs there.
- A bad **kernel module** runs in kernel mode and can crash the whole machine. Load modules only from trusted sources, see [Kernel modules](06-kernel-support-and-modules.md).
- Containers still use the host kernel. A container escape is a kernel level problem, not only a user mode one.

## DevOps Use Cases

### Use Case 1: Tell a CPU bound app from a syscall bound app
**Situation:** An app is slow. You must decide whether it is computing or constantly calling the kernel.

**Command:**
```bash
time python3 -c 'sum(i*i for i in range(3000000))'
time dd if=/dev/zero of=/dev/null bs=1 count=2000000 2>/dev/null
```
**Sample Output:**
```text

real	0m0.528s
user	0m0.520s
sys	0m0.004s

real	0m3.135s
user	0m0.614s
sys	0m2.530s
```
High `user` points to the code itself. High `sys` points to too many small reads, writes or locks. Increase buffer sizes, batch the I/O, or use [`strace -c`](03-system-calls.md) to find the busiest call.

### Use Case 2: Spot a kernel heavy host in monitoring
**Situation:** An alert says the CPU is at 90 percent. Is it the application or the kernel?

**Command:**
```bash
top -bn1 | sed -n 3p
```
**Sample Output:**
```text
%Cpu(s): 25.1 us,  6.8 sy,  0.0 ni, 61.6 id,  2.7 wa,  0.0 hi,  3.8 si,  0.0 st
```
Mostly `us`: look at the application. Mostly `sy` or `si`: look at network traffic, disk I/O or context switches. Mostly `wa`: look at the disk.

## Related Commands
- [The kernel and its responsibilities](01-kernel-and-responsibilities.md) - the previous page
- [System calls](03-system-calls.md) - the next page
- [`time`](../shell-basics/time.md) - user, sys and real time of a command
- [`top`](../process-management/top.md) - live `us` and `sy` percentages
- [`vmstat`](../system-info/vmstat.md) - `us`, `sy`, `in`, `cs` per second
- [`mpstat`](../system-info/mpstat.md) - the same per CPU
- [`dmesg`](../system-info/dmesg.md) - kernel faults and crashes
