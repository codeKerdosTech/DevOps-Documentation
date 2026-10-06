# ping

## What is it?
`ping` sends ICMP echo request packets to a host and waits for echo replies. It tells you whether a host is reachable, how long the round trip takes and whether packets are being lost.

## Syntax
```bash
ping [OPTIONS] DESTINATION
```

## Visual Overview
> `ping` sends an ICMP echo request to the target. If the target answers, `ping` prints the round trip time. If not, the packet counts as lost. At the end it prints statistics.

```mermaid
flowchart LR
    A[ping HOST] --> B[Resolve name to IP]
    B --> C[Send ICMP echo request]
    C --> D{Reply received}
    D -->|Yes| E[Print bytes ttl and time]
    D -->|No| F[Request timeout or unreachable]
    E --> G[Next packet after interval]
    F --> G
    G --> H[Print loss and rtt statistics]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    classDef alt fill:#FF9F43,stroke:#E67E22,stroke-width:2px,color:#fff
    class A start
    class B,C proc
    class D dec
    class E ok
    class F err
    class G alt
    class H out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-c COUNT` | Stop after sending `COUNT` packets |
| `-i SECONDS` | Wait `SECONDS` between packets (default 1) |
| `-s SIZE` | Set the payload size in bytes (default 56) |
| `-t TTL` | Set the IP time to live |
| `-W SECONDS` | Time to wait for each reply |
| `-w SECONDS` | Stop after this many seconds in total (deadline) |
| `-q` | Quiet output: only the summary |
| `-4` / `-6` | Force IPv4 or IPv6 |
| `-I INTERFACE` | Send packets out of this interface or from this source address |
| `-M do` | Do not fragment the packet (used to find the path MTU) |

## Usage Examples

### Example 1: Send a fixed number of packets (`-c`)
**Command:**
```bash
ping -c 3 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.6 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=14.1 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 14.100/14.300/14.600/0.216 ms
```

### Example 2: Change the interval (`-i`)
**Command:**
```bash
ping -c 3 -i 0.5 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=14.4 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 14.000/14.233/14.400/0.170 ms
```

### Example 3: Change the packet size (`-s`)
**Command:**
```bash
ping -c 2 -s 1000 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 1000(1028) bytes of data.
1008 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.1 ms
1008 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=15.3 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 15.100/15.200/15.300/0.100 ms
```

### Example 4: Set the TTL (`-t`)
A TTL of 1 expires at the first router, so the router answers with an error.

**Command:**
```bash
ping -c 1 -t 1 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
From 192.168.1.1 icmp_seq=1 Time to live exceeded

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
```

### Example 5: Set the reply timeout (`-W`)
**Command:**
```bash
ping -c 2 -W 2 203.0.113.99
```
**Sample Output:**
```text
PING 203.0.113.99 (203.0.113.99) 56(84) bytes of data.

--- 203.0.113.99 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1024ms
```

### Example 6: Stop after a total deadline (`-w`)
**Command:**
```bash
ping -w 3 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=14.1 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 14.100/14.267/14.500/0.170 ms
```

### Example 7: Quiet output (`-q`)
**Command:**
```bash
ping -c 4 -q 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 14.000/14.250/14.500/0.180 ms
```

### Example 8: Force IPv4 or IPv6 (`-4`, `-6`)
**Command:**
```bash
ping -4 -c 1 localhost
ping -6 -c 1 localhost
```
**Sample Output:**
```text
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.040 ms

--- localhost ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.040/0.040/0.040/0.000 ms
PING localhost(localhost (::1)) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.036 ms

--- localhost ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms
```

### Example 9: Choose the outgoing interface (`-I`)
**Command:**
```bash
ping -c 2 -I eth1 10.0.2.1
```
**Sample Output:**
```text
PING 10.0.2.1 (10.0.2.1) from 10.0.2.15 eth1: 56(84) bytes of data.
64 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=0.412 ms
64 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=0.380 ms

--- 10.0.2.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.380/0.396/0.412/0.016 ms
```

### Example 10: Do not fragment (`-M do`)
Payload 1472 plus 28 bytes of headers equals 1500, so this fits a standard MTU. Payload 1473 does not.

**Command:**
```bash
ping -c 1 -M do -s 1472 8.8.8.8
ping -c 1 -M do -s 1473 8.8.8.8
```
**Sample Output:**
```text
PING 8.8.8.8 (8.8.8.8) 1472(1500) bytes of data.
1480 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=15.0 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 15.000/15.000/15.000/0.000 ms
PING 8.8.8.8 (8.8.8.8) 1473(1501) bytes of data.
ping: local error: message too long, mtu=1500

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
```

## Pitfalls / Gotchas
- Many servers and cloud firewalls (AWS security groups by default) drop ICMP. No reply does not always mean the host is down.
- Without `-c` or `-w`, `ping` runs forever on Linux. Press `Ctrl+C` to stop it.
- A name that fails with `Name or service not known` is a DNS problem, not a network problem. Ping the IP to tell them apart.
- Very small intervals (`-i` under 0.2) need root.
- A low TTL in the reply (for example 64) hints at the remote OS and distance. It is not a reliable fingerprint.

## DevOps Use Cases

### Use Case 1: Separate a DNS failure from a network failure
**Situation:** A service cannot reach `api.example.com`. Ping the name, then the IP.

**Command:**
```bash
ping -c 1 api.example.com
ping -c 1 203.0.113.10
```
**Output:**
```text
ping: api.example.com: Name or service not known
PING 203.0.113.10 (203.0.113.10) 56(84) bytes of data.
64 bytes from 203.0.113.10: icmp_seq=1 ttl=56 time=21.3 ms

--- 203.0.113.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 21.300/21.300/21.300/0.000 ms
```
The network path works. DNS is the problem.

### Use Case 2: Health check script using the exit code
**Situation:** A script must know whether a host is up. `ping` exits with 0 on success and non-zero on failure.

Let's say we have this file `check_host.sh`:

**Input file** (`check_host.sh`):
```bash
#!/bin/bash
HOST=$1
if ping -c 1 -W 2 "$HOST" > /dev/null 2>&1; then
    echo "$HOST is UP"
else
    echo "$HOST is DOWN"
fi
```
**Command:**
```bash
bash check_host.sh 10.0.1.5
bash check_host.sh 10.0.1.99
```
**Output:**
```text
10.0.1.5 is UP
10.0.1.99 is DOWN
```

### Use Case 3: Wait for a new server in a CI/CD pipeline
**Situation:** Terraform just created a VM. The pipeline must wait until it answers before running Ansible.

Let's say we have this file `wait_for_host.sh`:

**Input file** (`wait_for_host.sh`):
```bash
#!/bin/bash
HOST=$1
for i in $(seq 1 10); do
    if ping -c 1 -W 2 "$HOST" > /dev/null 2>&1; then
        echo "Host $HOST is reachable after $i attempt(s)"
        exit 0
    fi
    echo "Attempt $i failed, retrying in 5 seconds"
    sleep 5
done
echo "Host $HOST never came up"
exit 1
```
**Command:**
```bash
bash wait_for_host.sh 10.0.1.20
```
**Output:**
```text
Attempt 1 failed, retrying in 5 seconds
Attempt 2 failed, retrying in 5 seconds
Host 10.0.1.20 is reachable after 3 attempt(s)
```

### Use Case 4: Measure packet loss to a database server
**Situation:** The app is slow. Check for loss and jitter to the database.

**Command:**
```bash
ping -c 20 -q 10.0.2.30
```
**Output:**
```text
PING 10.0.2.30 (10.0.2.30) 56(84) bytes of data.

--- 10.0.2.30 ping statistics ---
20 packets transmitted, 17 received, 15% packet loss, time 19031ms
rtt min/avg/max/mdev = 0.421/3.882/48.113/10.940 ms
```
15 percent loss and a high `mdev` point to an unstable link.

### Use Case 5: Find the path MTU for a VPN or tunnel
**Situation:** Large transfers hang over a VPN. Find the largest packet that passes without fragmentation.

**Command:**
```bash
ping -c 1 -M do -s 1400 10.8.0.1
ping -c 1 -M do -s 1472 10.8.0.1
```
**Output:**
```text
PING 10.8.0.1 (10.8.0.1) 1400(1428) bytes of data.
1408 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=32.1 ms

--- 10.8.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 32.100/32.100/32.100/0.000 ms
PING 10.8.0.1 (10.8.0.1) 1472(1500) bytes of data.
ping: local error: message too long, mtu=1420

--- 10.8.0.1 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
```
The tunnel MTU is 1420. Set the interface MTU to match.

### Use Case 6: Check pod to pod connectivity in Kubernetes
**Situation:** Verify that one pod can reach another pod IP.

**Command:**
```bash
kubectl exec web-7d9f -- ping -c 2 10.244.1.15
```
**Output:**
```text
PING 10.244.1.15 (10.244.1.15) 56(84) bytes of data.
64 bytes from 10.244.1.15: icmp_seq=1 ttl=62 time=0.512 ms
64 bytes from 10.244.1.15: icmp_seq=2 ttl=62 time=0.447 ms

--- 10.244.1.15 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.447/0.479/0.512/0.032 ms
```

### Use Case 7: Sweep a subnet to find live hosts
**Situation:** Find which addresses in a small range respond.

**Command:**
```bash
for i in $(seq 1 5); do ping -c 1 -W 1 192.168.1.$i > /dev/null 2>&1 && echo "192.168.1.$i is up"; done
```
**Output:**
```text
192.168.1.1 is up
192.168.1.4 is up
```

### Use Case 8: Log latency with timestamps for later analysis
**Situation:** Record latency to a gateway every second for 3 seconds with the time of each reply.

**Command:**
```bash
ping -c 3 192.168.1.1 | while read line; do echo "$(date +%T) $line"; done
```
**Output:**
```text
10:15:01 PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
10:15:01 64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=1.02 ms
10:15:02 64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.98 ms
10:15:03 64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=1.05 ms
10:15:03
10:15:03 --- 192.168.1.1 ping statistics ---
10:15:03 3 packets transmitted, 3 received, 0% packet loss, time 2003ms
10:15:03 rtt min/avg/max/mdev = 0.980/1.017/1.050/0.029 ms
```

## Related Commands
- [`traceroute`](traceroute.md) - show each hop on the path
- [`ip`](ip.md) - inspect interfaces and routes
- [`nslookup`](nslookup.md) - check DNS resolution
- [`dig`](dig.md) - detailed DNS queries
- `mtr` - live ping and traceroute combined
