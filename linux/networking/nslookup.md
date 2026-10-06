# nslookup

## What is it?
`nslookup` (name server lookup) queries DNS to turn a host name into an IP address, or an IP address into a name. It is simpler than `dig` and works in two modes: a one line command and an interactive prompt.

## Syntax
```bash
nslookup [OPTIONS] NAME [SERVER]
```

## Visual Overview
> `nslookup` sends the name to a DNS server (the default resolver or the one you give) and prints the server it used followed by the answer. A non-authoritative answer means it came from a cache.

```mermaid
flowchart LR
    A[nslookup NAME] --> B{Server given}
    B -->|Yes| C[Ask that DNS server]
    B -->|No| D[Ask default resolver]
    C --> E{Record found}
    D --> E
    E -->|Yes| F[Print server and answer]
    E -->|No| G[Print NXDOMAIN error]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef dec fill:#FFC312,stroke:#E1A100,stroke-width:2px,color:#222
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef err fill:#FF4757,stroke:#C0392B,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    classDef alt fill:#FF9F43,stroke:#E67E22,stroke-width:2px,color:#fff
    class A start
    class B,E dec
    class C,D proc
    class F ok
    class G err
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `NAME` | Host name or IP address to look up |
| `SERVER` | DNS server to ask (second argument) |
| `-type=TYPE` | Record type: `A`, `AAAA`, `MX`, `NS`, `TXT`, `CNAME`, `SOA`, `PTR` |
| `-query=TYPE` | Same as `-type` |
| `-port=PORT` | Use a different DNS port |
| `-timeout=SECONDS` | Time to wait for a reply |
| `-retry=N` | Number of retries |
| `-debug` | Print the full DNS response |
| `-` (interactive) | Run `nslookup` with no name to open its prompt |

## Usage Examples

Let's say we have this file `lookup_cmds.txt`:

**Input file** (`lookup_cmds.txt`):
```text
set type=MX
example.org
exit
```

### Example 1: Look up a host name
**Command:**
```bash
nslookup example.com
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	example.com
Address: 93.184.216.34
```

### Example 2: Use a specific DNS server (`SERVER`)
**Command:**
```bash
nslookup example.com 8.8.8.8
```
**Sample Output:**
```text
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	example.com
Address: 93.184.216.34
```

### Example 3: Reverse lookup of an IP
**Command:**
```bash
nslookup 8.8.8.8
```
**Sample Output:**
```text
8.8.8.8.in-addr.arpa	name = dns.google.

Authoritative answers can be found from:
```

### Example 4: Mail servers (`-type=MX`)
**Command:**
```bash
nslookup -type=MX example.org
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	mail exchanger = 10 mail.example.org.
example.org	mail exchanger = 20 mail2.example.org.
```

### Example 5: Other record types (`-type`, `-query`)
**Command:**
```bash
nslookup -type=NS example.org
nslookup -query=TXT example.org
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	nameserver = ns1.example.org.
example.org	nameserver = ns2.example.org.

Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	text = "v=spf1 include:_spf.example.org ~all"
```

### Example 6: Alias records (`-type=CNAME`)
**Command:**
```bash
nslookup -type=CNAME www.example.com
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
www.example.com	canonical name = d111111abcdef8.cloudfront.net.
```

### Example 7: Use a different port (`-port`)
**Command:**
```bash
nslookup -port=5353 app.local 127.0.0.1
```
**Sample Output:**
```text
Server:		127.0.0.1
Address:	127.0.0.1#5353

Name:	app.local
Address: 10.0.1.15
```

### Example 8: Set timeout and retries (`-timeout`, `-retry`)
**Command:**
```bash
nslookup -timeout=2 -retry=1 example.com 203.0.113.99
```
**Sample Output:**
```text
;; connection timed out; no servers could be reached
```

### Example 9: Show the raw response (`-debug`)
**Command:**
```bash
nslookup -debug example.com
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

------------
    QUESTIONS:
	example.com, type = A, class = IN
    ANSWERS:
    ->  example.com
	internet address = 93.184.216.34
	ttl = 3600
------------
Non-authoritative answer:
Name:	example.com
Address: 93.184.216.34
```

### Example 10: Interactive mode
**Command:**
```bash
nslookup
```
**Sample Output:**
```text
> set type=MX
> example.org
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	mail exchanger = 10 mail.example.org.

> exit
```

### Example 11: Feed commands from a file
**Command:**
```bash
nslookup < lookup_cmds.txt
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	mail exchanger = 10 mail.example.org.
example.org	mail exchanger = 20 mail2.example.org.
```

### Example 12: A name that does not exist
**Command:**
```bash
nslookup nosuchhost.example.com
```
**Sample Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

** server can't find nosuchhost.example.com: NXDOMAIN
```

## Pitfalls / Gotchas
- `Non-authoritative answer` is normal. It only means the answer came from a cache and not directly from the zone owner.
- The first `Server:` and `Address:` lines show the resolver that answered, not the host you asked about. Do not confuse the two.
- `nslookup` ignores `/etc/hosts`, like `dig`. Use `getent hosts NAME` to see the system resolution.
- On many minimal containers `nslookup` and `dig` are missing. Install `dnsutils` (Debian, Ubuntu) or `bind-utils` (RHEL, CentOS).
- `nslookup` output is meant for humans and changes between versions. Use `dig +short` for scripts.

## DevOps Use Cases

### Use Case 1: Test DNS from inside a Kubernetes pod
**Situation:** A service name does not resolve inside the cluster. Test it from a pod.

**Command:**
```bash
kubectl exec -it web-7d9f -- nslookup db.default.svc.cluster.local
```
**Output:**
```text
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	db.default.svc.cluster.local
Address: 10.96.88.14
```

### Use Case 2: Check whether the problem is the resolver or the record
**Situation:** An internal name fails to resolve. Ask the default resolver, then a public one.

**Command:**
```bash
nslookup app.example.com
nslookup app.example.com 8.8.8.8
```
**Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

** server can't find app.example.com: NXDOMAIN

Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	app.example.com
Address: 203.0.113.60
```
The company resolver is missing the record, while the public resolver has it.

### Use Case 3: Verify mail routing for a domain
**Situation:** Confirm MX records before setting up a mail relay.

**Command:**
```bash
nslookup -type=MX example.org
```
**Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	mail exchanger = 10 mail.example.org.
example.org	mail exchanger = 20 mail2.example.org.
```

### Use Case 4: Identify a host from an IP found in logs
**Situation:** An unknown IP appears in the firewall log.

**Command:**
```bash
nslookup 203.0.113.77
```
**Output:**
```text
77.113.0.203.in-addr.arpa	name = scanner-77.badhosting.example.
```

### Use Case 5: Check a load balancer's backend IPs
**Situation:** A name should return several A records for round robin.

**Command:**
```bash
nslookup api.example.com
```
**Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	api.example.com
Address: 203.0.113.11
Name:	api.example.com
Address: 203.0.113.12
Name:	api.example.com
Address: 203.0.113.13
```

### Use Case 6: Find the authoritative name servers
**Situation:** Before asking the source of truth, find who it is.

**Command:**
```bash
nslookup -type=NS example.org
```
**Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	nameserver = ns1.example.org.
example.org	nameserver = ns2.example.org.
```

### Use Case 7: Quick DNS check script for a list of hosts
**Situation:** A pre-deploy check reports hosts that do not resolve.

Let's say we have this file `hosts.txt`:

**Input file** (`hosts.txt`):
```text
web01.example.com
db01.example.com
```
**Command:**
```bash
for h in $(cat hosts.txt); do nslookup $h > /dev/null 2>&1 && echo "$h OK" || echo "$h FAILED"; done
```
**Output:**
```text
web01.example.com OK
db01.example.com FAILED
```

### Use Case 8: Check SPF and domain verification TXT records
**Situation:** A cloud provider asks you to add a TXT record to prove domain ownership. Confirm it is live.

**Command:**
```bash
nslookup -type=TXT example.org
```
**Output:**
```text
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
example.org	text = "v=spf1 include:_spf.example.org ~all"
example.org	text = "cloud-verify=abc123def456"
```

## Related Commands
- [`dig`](dig.md) - detailed DNS queries, better for scripts
- [`ping`](ping.md) - test reachability of the resolved IP
- [`curl`](curl.md) - call the service after resolving it
- [`traceroute`](traceroute.md) - see the path to the IP
- `host` - short DNS lookups
