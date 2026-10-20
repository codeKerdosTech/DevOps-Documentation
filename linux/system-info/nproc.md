# nproc

## What is it?
`nproc` prints **the number of processing units (CPUs) that are available to the current process**. It is a tiny command with one job: giving you a number that you can use in a script, for example `make -j"$(nproc)"` (compile on all the CPUs), or to set the number of worker processes. By default it respects the **CPU affinity** of the process (set with `taskset`, or by a cgroup cpuset), and the environment variable `OMP_NUM_THREADS`. With `--all` it shows the number of CPUs that are installed. It is part of GNU coreutils, on every Linux distribution.

## Syntax
```bash
nproc [OPTIONS]
```
`nproc` has no arguments.

### What `nproc` counts
| Command | Result |
|---------|--------|
| `nproc` | The CPUs that this process **may use** (affinity, cpuset, `OMP_NUM_THREADS`) |
| `nproc --all` | **All** the CPUs of the machine that are installed (`lscpu` shows the same number as `CPU(s)`) |
| `nproc --ignore=N` | The result of `nproc`, minus `N` (but at least 1) |

A "CPU" is a logical CPU (a hardware thread). A server with 2 cores and hyper-threading shows `4`.

## Visual Overview
> `nproc` prints the number of CPUs available to the current process. Build and CI scripts use it to choose how many parallel jobs to run.

```mermaid
flowchart LR
    c1[nproc] --> o1[8] --> c2[make -j8<br/>one job per CPU]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class c1,c2 cmd
    class o1 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `--all` | Print the number of **installed** CPUs, and not only those that the process may use |
| `--ignore=N` | Do not count `N` CPUs. The result is never lower than 1 |
| `--help` | Show the help text |
| `--version` | Show the version |

### Variables that change the result
| Variable | Effect |
|----------|--------|
| `OMP_NUM_THREADS` | If set, `nproc` prints the first number of the list (and `--all` ignores it) |
| `OMP_THREAD_LIMIT` | If set, it limits the number that `nproc` prints |

## Usage Examples

Let's say we are on **Ubuntu 22.04** on the server `web01` with 4 logical CPUs (2 cores x 2 threads). Today is `2026-10-12`. Each example starts again from this state.

### Example 1: The number of CPUs
**Command:**
```bash
nproc
```
**Sample Output:**
```text
4
```

### Example 2: All the installed CPUs (`--all`)
**Command:**
```bash
nproc --all
```
**Sample Output:**
```text
4
```
On a machine where nothing limits the process, both numbers are the same.

### Example 3: The difference when a process is limited (`taskset`)
`taskset` limits the process to some CPUs. `nproc` respects it.

**Command:**
```bash
taskset -c 0,1 nproc
taskset -c 0,1 nproc --all
```
**Sample Output:**
```text
2
4
```
`nproc` printed `2`: this process may use 2 CPUs. `--all` still shows `4`.

### Example 4: Keep some CPUs free (`--ignore`)
**Command:**
```bash
nproc --ignore=1
nproc --ignore=3
nproc --ignore=10
```
**Sample Output:**
```text
3
1
1
```
The result is never smaller than 1, even when you ignore more CPUs than there are.

### Example 5: `OMP_NUM_THREADS` changes the result
**Command:**
```bash
OMP_NUM_THREADS=2 nproc
OMP_NUM_THREADS=2 nproc --all
OMP_NUM_THREADS=8 nproc
```
**Sample Output:**
```text
2
4
8
```
`OMP_NUM_THREADS` is the variable of OpenMP programs. `nproc` takes its value, even if it is larger than the number of CPUs. `--all` ignores it.

### Example 6: Use it in a command (`make -j`)
**Command:**
```bash
make -j"$(nproc)"
make -j"$(nproc --ignore=1)"
```
**Sample Output:**
```text
make: Entering directory '/srv/build'
[ 25%] Building C object src/main.o
[ 50%] Building C object src/util.o
[100%] Linking C executable app
make: Leaving directory '/srv/build'
```
The second form keeps one CPU free for the rest of the system.

### Example 7: In a container with a CPU limit
In Docker, `--cpuset-cpus` limits the CPUs. `--cpus` (a quota) is **not** seen by `nproc`.

**Command:**
```bash
docker run --rm alpine nproc
docker run --rm --cpuset-cpus=0,1 alpine nproc
docker run --rm --cpus=1 alpine nproc
```
**Sample Output:**
```text
4
2
4
```
The quota `--cpus=1` still shows 4. The container may use 4 CPUs, but only for a total of 1 CPU of time.

### Example 8: Compare `nproc` with `lscpu`
**Command:**
```bash
nproc
lscpu | grep -E '^CPU\(s\):'
getconf _NPROCESSORS_ONLN
grep -c ^processor /proc/cpuinfo
```
**Sample Output:**
```text
4
CPU(s):                          4
4
4
```
All four ways give the same number here.

### Example 9: Check the version (`--version`)
**Command:**
```bash
nproc --version | head -n 1
```
**Sample Output:**
```text
nproc (GNU coreutils) 8.32
```

### Example 10: Show the help (`--help`)
**Command:**
```bash
nproc --help | head -n 3
```
**Sample Output:**
```text
Usage: nproc [OPTION]...
Print the number of processing units available to the current process,
which may be less than the number of online processors.
```

### Example 11: A wrong value
**Command:**
```bash
nproc --ignore=abc
```
**Sample Output:**
```text
nproc: invalid number: ‘abc’
Try 'nproc --help' for more information.
```

## Pitfalls / Gotchas
- **`nproc` does not see a CPU quota.** In Docker (`--cpus`), Kubernetes (`resources.limits.cpu`) and `systemd` (`CPUQuota=`), the quota limits the **time**, not the number of CPUs. `nproc` shows the number of CPUs of the host, and a program that starts that many threads gets throttled (slow, with high latency). Use the value of `/sys/fs/cgroup/cpu.max` (cgroup v2: `max 100000` or `200000 100000` = 2 CPUs), or the runtime settings of your language (`GOMAXPROCS`, the JVM `-XX:ActiveProcessorCount`, which new JVMs find by themselves).
- **It counts threads, not cores.** With hyper-threading, `nproc` is twice the number of cores. For CPU bound work, the number of physical cores is often better. See `lscpu`.
- **`OMP_NUM_THREADS` surprises.** A variable that is set in a profile file (`/etc/profile.d`) changes `nproc`, so the same command prints different numbers in different shells.
- **`make -j$(nproc)` can use too much memory.** Each compile job can need 1 to 2 GiB (C++, Rust). On a 4 GiB server, `-j4` can cause an OOM kill. Use `-j$(( $(free -g | awk '/Mem:/ {print $2}') / 2 ))` or limit it.
- **`--ignore` is for "leave something for the others",** not for calculations with 0 (the result is never below 1).
- **Offline CPUs are not counted** by `nproc`, but are in `lscpu` as `Off-line CPU(s) list`.
- **The result is evaluated when the command runs.** If the affinity changes later, the number is out of date.
- **BSD and macOS have no `nproc`.** Use `sysctl -n hw.ncpu` (macOS) or `getconf _NPROCESSORS_ONLN`, which works in most places, or `nproc` from GNU coreutils (`gnproc`).
- **Alpine (BusyBox) `nproc`** supports only the plain form (and `--all` in newer versions).

## DevOps Use Cases

### Use Case 1: Compile with all the CPUs
**Situation:** A build script must use all the CPUs, without a fixed number in the code.

**Command:**
```bash
make -j"$(nproc)" all
echo "built with $(nproc) jobs"
```
**Output:**
```text
built with 4 jobs
```

### Use Case 2: Set the number of workers of an application
**Situation:** Write the number of workers of `gunicorn` or `nginx` into a config at the start of the container.

**Command:**
```bash
workers=$(( $(nproc) * 2 + 1 ))
echo "workers = $workers" > /etc/myapp/workers.conf
cat /etc/myapp/workers.conf
```
**Output:**
```text
workers = 9
```
For `nginx` the line `worker_processes auto;` makes it find the number itself.

### Use Case 3: Run a task on all the CPUs in parallel (with `xargs`)
**Situation:** Compress 100 log files, one process on each CPU.

**Command:**
```bash
ls /var/log/myapp/*.log | xargs -P "$(nproc)" -n 1 gzip
ls /var/log/myapp/*.gz | wc -l
```
**Output:**
```text
100
```
`xargs -P` runs as many `gzip` processes at the same time as there are CPUs.

### Use Case 4: A load check that is based on the number of CPUs
**Situation:** The load average is high only if it is higher than the number of CPUs.

**Command:**
```bash
load=$(cut -d' ' -f1 /proc/loadavg)
awk -v l="$load" -v n="$(nproc)" 'BEGIN{ printf "load %.2f on %d CPUs: %s\n", l, n, (l>n ? "OVERLOADED" : "ok") }'
```
**Output:**
```text
load 1.02 on 4 CPUs: ok
```

### Use Case 5: A thread count that respects the container limit
**Situation:** A start script must choose a thread count that is correct in a pod with a CPU limit (a quota), where `nproc` shows the host.

**Command:**
```bash
if [ -r /sys/fs/cgroup/cpu.max ]; then
  read -r quota period < /sys/fs/cgroup/cpu.max
  if [ "$quota" != "max" ]; then
    limit=$(( (quota + period - 1) / period ))
  fi
fi
threads=${limit:-$(nproc)}
echo "using $threads threads"
```
**Output:**
```text
using 2 threads
```
The value `200000 100000` in `cpu.max` means a quota of 2 CPUs. Without a limit (`max 100000`), the script falls back to `nproc`.

### Use Case 6: Do a test with fewer CPUs
**Situation:** Check how an application runs when only 2 CPUs are available.

**Command:**
```bash
taskset -c 0,1 ./benchmark --threads "$(taskset -c 0,1 nproc)"
```
**Output:**
```text
threads: 2
throughput: 8120 req/s
```

### Use Case 7: Keep one CPU for the system during a backup
**Situation:** A compression job should not take all the CPUs.

**Command:**
```bash
tar -cf - /srv/data | xz -T"$(nproc --ignore=1)" > /backup/data.tar.xz
```
**Output:**
```text
```
`xz -T3` uses 3 threads on 4 CPUs, which leaves one CPU free for the services.

## Related Commands
- [`lscpu`](lscpu.md) - the CPU details: cores, threads, sockets, flags
- [`mpstat`](mpstat.md) - the load of each CPU
- [`uptime`](uptime.md) - the load average, to compare with `nproc`
- [`top`](../process-management/top.md) - the load of the CPUs (`1` key)
- [`free`](free.md) - the memory that limits `make -j`
- [`vmstat`](vmstat.md) - the CPU summary
- [`xargs`](../text-processing/xargs.md) - `xargs -P "$(nproc)"` runs jobs in parallel
- [`uname`](uname.md) - the architecture
- `taskset` - limit a process to some CPUs, and see the effect on `nproc`
- `getconf _NPROCESSORS_ONLN` - the portable way to get the number
- `parallel` - GNU parallel uses `nproc` as the default

---
<!-- nav -->
[← Previous: `mpstat`](mpstat.md) | [Learning path](../README.md) | [Next: `timedatectl` →](timedatectl.md)
