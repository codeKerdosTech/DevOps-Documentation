# Regular Expressions

## What is it?
A **regular expression** (regex) is a short pattern that describes a set of text strings. Instead of searching for one fixed word, you describe a **shape**: "three digits", "a line that starts with `ERROR`", "four numbers separated by dots". Tools such as `grep`, `sed`, `awk`, `find -regex` and the bash test `[[ x =~ y ]]` use regexes to find, filter and rewrite text.

Do not confuse a regex with a **wildcard** (glob). A glob such as `*.log` matches **file names** and is expanded by the shell, see [Wildcards and globbing](06-wildcards-globbing.md). A regex matches **text inside lines**, and the same characters mean different things: in a glob `*` is "anything", in a regex `.*` is "anything" and `*` alone means "repeat the previous item".

## Syntax
```bash
grep 'PATTERN' FILE            # basic regex (BRE)
grep -E 'PATTERN' FILE         # extended regex (ERE)
grep -P 'PATTERN' FILE         # Perl regex (PCRE), GNU grep only
sed -E 's/PATTERN/REPLACEMENT/' FILE
awk '/PATTERN/ {print}' FILE
[[ $var =~ PATTERN ]]          # bash, extended regex
```

## Visual Overview
> A regular expression is a pattern. The tool tests every line against it and keeps the lines that match. Here `^ERROR` means a line that starts with ERROR.

```mermaid
flowchart LR
    i1[app.log<br/>ERROR disk full<br/>INFO started<br/>ERROR timeout] --> c1[grep ^ERROR app.log]
    c1 --> o1[ERROR disk full<br/>ERROR timeout]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class i1 inp
    class c1 cmd
    class o1 out
```

## Options/Flags
**Building blocks (work in both BRE and ERE)**

| Pattern | Matches | Example |
|---------|---------|---------|
| `abc` | The exact text `abc` | `ERROR` |
| `.` | Any one character | `c.t` matches `cat`, `cut` |
| `*` | Zero or more of the previous item | `colou*r` |
| `^` | Start of the line | `^2026` |
| `$` | End of the line | `login$` |
| `[abc]` | One character from the set | `c[aou]t` |
| `[a-z]` | One character from the range | `[0-9]` |
| `[^abc]` | One character **not** in the set | `[^0-9]` |
| `\` | Turn off the special meaning of the next character | `a\.b` |

**POSIX character classes** (used inside brackets)

| Class | Meaning |
|-------|---------|
| `[[:digit:]]` | A digit, `0` to `9` |
| `[[:alpha:]]` | A letter |
| `[[:alnum:]]` | A letter or digit |
| `[[:space:]]` | A space, tab or newline |
| `[[:upper:]]` / `[[:lower:]]` | Upper or lower case letter |
| `[[:punct:]]` | A punctuation character |

**BRE and ERE: the difference**

| Meaning | BRE (`grep`, `sed` default) | ERE (`grep -E`, `sed -E`, `awk`, `[[ =~ ]]`) |
|---------|-----------------------------|----------------------------------------------|
| One or more | `\+` | `+` |
| Zero or one | `\?` | `?` |
| Either / or | backslash then pipe | the pipe character |
| Group | `\( \)` | `( )` |
| Exactly `n`, `n` to `m` | `\{n\}`, `\{n,m\}` | `{n}`, `{n,m}` |
| Back reference | `\1` | `\1` |

In BRE the characters `+ ? | ( ) { }` are **plain text** unless you put a backslash before them. In ERE they are special. Use `-E` and you write less.

**Other common pieces**

| Item | Meaning | Where |
|------|---------|-------|
| `\b` / `\<` `\>` | Word boundary | GNU `grep`, `sed` |
| `\w` `\d` `\s` | Word, digit, space | `grep -P`, `\w` also in GNU `grep` |
| `(?<=text)` | Look behind | `grep -P` only |
| `\K` | Forget what matched so far | `grep -P` only |
| `-i` | Ignore case | `grep`, `sed -i` is different (in place) |
| `-o` | Print only the matching part | `grep` |
| `-w` | Match whole words only | `grep` |
| `-x` | The whole line must match | `grep` |
| `-F` | Fixed text, no regex | `grep` |
| `-v` | Invert, show lines that do **not** match | `grep` |
| `-c` | Count matching lines | `grep` |

For the examples let's say we have these two files:

**Input file** (`app.log`):
```text
2026-10-14 10:01:05 INFO  user=alice ip=10.0.0.5 action=login
2026-10-14 10:02:11 ERROR user=bob ip=10.0.0.9 action=upload code=500
2026-10-14 10:03:42 WARN  user=alice ip=192.168.1.20 action=download
2026-10-14 10:04:00 ERROR user=carol ip=10.0.0.7 action=login code=403
2026-10-14 10:05:30 INFO  user=dave ip=172.16.0.4 action=logout
```

**Input file** (`words.txt`):
```text
cat
cut
cart
colour
color
coat
abc123
ab12
```

## Usage Examples

### Example 1: Literal text
**Command:**
```bash
grep 'ERROR' app.log | cut -c1-40
```
**Sample Output:**
```text
2026-10-14 10:02:11 ERROR user=bob ip=10
2026-10-14 10:04:00 ERROR user=carol ip=
```
A pattern with no special characters is just a search for that text.

### Example 2: Start of line (`^`)
**Command:**
```bash
grep '^2026-10-14 10:0[1-2]' app.log | cut -c1-30
```
**Sample Output:**
```text
2026-10-14 10:01:05 INFO  user
2026-10-14 10:02:11 ERROR user
```
`^` anchors the match to the start, `[1-2]` accepts `1` or `2` for the last digit of the minute.

### Example 3: End of line (`$`)
**Command:**
```bash
grep 'login$' app.log | cut -c1-45
```
**Sample Output:**
```text
2026-10-14 10:01:05 INFO  user=alice ip=10.0.
```
The fourth line also contains `login`, but it ends with `code=403`, so it does not match.

### Example 4: Any single character (`.`)
**Command:**
```bash
grep -n 'c.t' words.txt
```
**Sample Output:**
```text
1:cat
2:cut
```
`cart` has two letters between `c` and `t`, and `coat` has two, so they do not match.

### Example 5: A set of characters (`[ ]`)
**Command:**
```bash
grep '^c[aou]t$' words.txt
```
**Sample Output:**
```text
cat
cut
```

### Example 6: Digits and negation (`[0-9]`, `-v`)
**Command:**
```bash
grep '[0-9]' words.txt
echo "---"
grep -v '[0-9]' words.txt | head -3
```
**Sample Output:**
```text
abc123
ab12
---
cat
cut
cart
```

### Example 7: Repeat zero or more times (`*`)
**Command:**
```bash
grep -x 'colou*r' words.txt
echo "---"
grep -x 'co*lou*r' words.txt
```
**Sample Output:**
```text
colour
color
---
colour
color
```
`u*` allows zero or more `u`. `-x` means the whole line must match. Both spellings match.

### Example 8: BRE needs a backslash for `+`
**Command:**
```bash
grep 'ab[0-9]\+' words.txt
grep 'ab[0-9]+' words.txt
echo rc=$?
```
**Sample Output:**
```text
ab12
rc=1
```
In BRE `\+` means "one or more". A plain `+` is just the plus sign, so the second command finds nothing and exits with `1`. Also note that `abc123` does not match, there is a `c` after `ab`.

### Example 9: ERE makes it cleaner (`grep -E`)
**Command:**
```bash
grep -E 'ab[0-9]+' words.txt
grep -E '^(cat|cut)$' words.txt
grep -E 'colou?r' words.txt
```
**Sample Output:**
```text
ab12
cat
cut
colour
color
```
`+` is one or more, `( | )` is either or, and `?` is zero or one.

### Example 10: Exact counts (`{n}`)
**Command:**
```bash
grep -E '^[a-z]{3}$' words.txt | tr '\n' ' '
echo
grep -E '[0-9]{3}' words.txt
```
**Sample Output:**
```text
cat cut 
abc123
```
Only three letter words match the first pattern. `{3}` means exactly three, `{2,4}` means two to four.

### Example 11: Extract IP addresses (`-o`)
**Command:**
```bash
grep -oE '[0-9]{1,3}(\.[0-9]{1,3}){3}' app.log
```
**Sample Output:**
```text
10.0.0.5
10.0.0.9
192.168.1.20
10.0.0.7
172.16.0.4
```
One to three digits, followed by three groups of a dot and one to three digits. `-o` prints only the matched part.

### Example 12: Extract a field and remove duplicates
**Command:**
```bash
grep -oE 'user=[a-z]+' app.log | sort -u
grep -oE 'code=[0-9]+' app.log
```
**Sample Output:**
```text
user=alice
user=bob
user=carol
user=dave
code=500
code=403
```

### Example 13: Whole words only (`-w`)
**Command:**
```bash
grep -w 'cat' words.txt
grep 'ca' words.txt | tr '\n' ' '
echo
```
**Sample Output:**
```text
cat
cat cart 
```
Without `-w`, `ca` also matches `cart`.

### Example 14: Case insensitive (`-i`) and counting (`-c`, `-vc`)
**Command:**
```bash
grep -i 'error' app.log | wc -l
grep -vc ERROR app.log
```
**Sample Output:**
```text
2
3
```

### Example 15: Back reference, a repeated word (`\1`)
Let's say we have this file `rep.txt`:

**Input file** (`rep.txt`):
```text
hello hello world
abc abc abc
no repeat
```

**Command:**
```bash
grep -E '(\w+) \1' rep.txt
```
**Sample Output:**
```text
hello hello world
abc abc abc
```
`(\w+)` captures a word and `\1` means "the same text again".

### Example 16: Replace with `sed` and capture groups
**Command:**
```bash
echo 'user=alice ip=10.0.0.5' | sed -E 's/ip=([0-9.]+)/IP[\1]/'
sed -E 's/[0-9]{1,3}(\.[0-9]{1,3}){3}/x.x.x.x/' app.log | head -1 | cut -c1-60
```
**Sample Output:**
```text
user=alice IP[10.0.0.5]
2026-10-14 10:01:05 INFO  user=alice ip=x.x.x.x action=login
```
`\1` in the replacement is the text caught by the first group. The second command hides the IP address, which is useful before sharing a log.

### Example 17: The same in BRE (`\( \)`)
**Command:**
```bash
echo 'John Smith' | sed 's/\(.*\) \(.*\)/\2, \1/'
```
**Sample Output:**
```text
Smith, John
```

### Example 18: Regex in `awk`
**Command:**
```bash
awk '/ERROR/ {print $4}' app.log
awk '$4 ~ /^user=(alice|bob)$/ {print $4, $1}' app.log | head -2
```
**Sample Output:**
```text
user=bob
user=carol
user=alice 2026-10-14
user=bob 2026-10-14
```
`/ERROR/` tests the whole line, `$4 ~ /…/` tests one field.

### Example 19: Regex in bash (`[[ =~ ]]`)
**Command:**
```bash
for v in 2026-10-14 14-10-2026 abc; do
  [[ $v =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]] && echo "$v matches" || echo "$v no match"
done
```
**Sample Output:**
```text
2026-10-14 matches
14-10-2026 no match
abc no match
```
Do **not** quote the pattern on the right side, a quoted pattern is matched as plain text.

### Example 20: Capture groups in bash (`BASH_REMATCH`)
**Command:**
```bash
if [[ "build-1.42.7" =~ ^build-([0-9]+)\.([0-9]+)\.([0-9]+)$ ]]; then
  echo "major=${BASH_REMATCH[1]} minor=${BASH_REMATCH[2]} patch=${BASH_REMATCH[3]}"
fi
```
**Sample Output:**
```text
major=1 minor=42 patch=7
```

### Example 21: Greedy matching
**Command:**
```bash
echo '<b>bold</b> and <i>it</i>' | sed -E 's/<.*>//'
echo '<b>bold</b> and <i>it</i>' | sed -E 's/<[^>]*>//g'
```
**Sample Output:**
```text

bold and it
```
`.*` is **greedy**, it takes as much as it can, so the first command removed everything from the first `<` to the last `>` and printed an empty line. `[^>]*` stops at the first `>`, which removes only the tags.

### Example 22: Escape special characters
**Command:**
```bash
echo 'price is $5.00 (net)' | grep -F '$5.00'
echo 'a.b' | grep 'a\.b'
echo 'axb' | grep 'a\.b'
echo rc=$?
```
**Sample Output:**
```text
price is $5.00 (net)
a.b
rc=1
```
`-F` searches for fixed text. `\.` means a real dot, so `axb` does not match.

### Example 23: Character classes (`[[:digit:]]`)
**Command:**
```bash
echo 'ab 12 _x' | grep -oE '[[:digit:]]+'
echo 'ab 12 _x' | grep -oE '[[:alpha:]]+' | tr '\n' ' '
echo
echo 'a  b' | grep -E '[[:space:]]{2}'
```
**Sample Output:**
```text
12
ab x 
a  b
```

### Example 24: Perl regex for look behind (`grep -P`)
**Command:**
```bash
echo 'ip=10.0.0.5 port=8080' | grep -oP '(?<=port=)\d+'
grep -oP 'code=\K[0-9]+' app.log
```
**Sample Output:**
```text
8080
500
403
```
`(?<=port=)` checks that `port=` comes before the number but does not include it. `\K` throws away what was matched before it. `-P` is not available on macOS `grep`.

### Example 25: Validate an email like string
**Command:**
```bash
printf 'a@b.com\nbad@\nx.y@z.org\n' | grep -E '^[[:alnum:]._-]+@[[:alnum:].-]+\.[a-z]{2,}$'
```
**Sample Output:**
```text
a@b.com
x.y@z.org
```
This is a quick filter, not a full email validator.

## Sample Output
Count the status codes in `app.log`:

```bash
grep -oE 'code=[0-9]+' app.log | sort | uniq -c
```
```text
      1 code=403
      1 code=500
```

## Pitfalls / Gotchas
- **Quote your pattern** with single quotes. Without quotes the shell expands `*`, `?` and `[ ]` as file name wildcards first.
- A regex is **not anchored** by default. `cat` also matches `concatenate`. Use `^`, `$`, `-w` or `-x`.
- `*` means "repeat the previous item", not "anything". Write `.*` for anything.
- `.` matches **any** character. To match a real dot write `\.`, otherwise `10.0.0.5` also matches `10x0y0z5`.
- BRE vs ERE: `grep 'a+'` looks for the text `a+`. Use `grep -E 'a+'` or `grep 'a\+'`.
- `grep -P` is a GNU extension, `sed -E` and `grep -E` work almost everywhere.
- Different tools use different flavors: `awk` is ERE, `sed` is BRE unless `-E`, `grep` is BRE unless `-E` or `-P`, `[[ =~ ]]` is ERE. A pattern that works in one may fail in another.
- Do not quote the right side of `[[ x =~ pattern ]]` or it becomes a plain string. Put the pattern in a variable if it has spaces: `re='a b'; [[ $x =~ $re ]]`.
- Greedy matching surprises: `.*` runs to the **last** possible match. Use `[^x]*` to stop at the first `x`.
- Locale matters: `[a-z]` may include capital letters in some locales. Use `[[:lower:]]` for safety.
- Regexes can be slow on huge input when they backtrack. Prefer simple patterns and `grep -F` for fixed text.
- A pattern like `[0-9]{1,3}(\.[0-9]{1,3}){3}` finds things that **look like** an IP, it also accepts `999.999.999.999`.

## DevOps Use Cases

### Use Case 1: Show only the error lines with their code
**Situation:** Find all failed requests and their status codes in one pass.

Using the `app.log` file shown above.

**Command:**
```bash
grep -E 'ERROR.*code=[0-9]+' app.log | grep -oE 'user=[a-z]+|code=[0-9]+' | paste - - | head -2
```
**Sample Output:**
```text
user=bob	code=500
user=carol	code=403
```

### Use Case 2: Mask IP addresses before sharing a log
**Situation:** Send a log to a vendor without leaking internal addresses.

Using the `app.log` file shown above.

**Command:**
```bash
sed -E 's/[0-9]{1,3}(\.[0-9]{1,3}){3}/x.x.x.x/' app.log | head -2 | cut -c1-60
```
**Sample Output:**
```text
2026-10-14 10:01:05 INFO  user=alice ip=x.x.x.x action=login
2026-10-14 10:02:11 ERROR user=bob ip=x.x.x.x action=upload 
```

### Use Case 3: Find the users that triggered 4xx or 5xx errors
**Situation:** Alert on every request whose status code starts with 4 or 5.

Using the `app.log` file shown above.

**Command:**
```bash
grep -E 'code=[45][0-9]{2}' app.log | grep -oE 'user=[a-z]+'
```
**Sample Output:**
```text
user=bob
user=carol
```

### Use Case 4: Check a version string in a CI script
**Situation:** A release job must refuse tags that do not look like `v1.2.3`.

**Command:**
```bash
for tag in v1.2.3 v1.2 1.2.3 v10.20.30; do
  [[ $tag =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]] && echo "$tag ok" || echo "$tag rejected"
done
```
**Sample Output:**
```text
v1.2.3 ok
v1.2 rejected
1.2.3 rejected
v10.20.30 ok
```

### Use Case 5: Count requests per user
**Situation:** Who is generating the most traffic?

Using the `app.log` file shown above.

**Command:**
```bash
grep -oE 'user=[a-z]+' app.log | sort | uniq -c | sort -rn | head -2
```
**Sample Output:**
```text
      2 user=alice
      1 user=dave
```
Users with equal counts (bob, carol and dave have one each) are listed in reverse name order by `sort -rn`, so `dave` comes first.

## Related Commands
- [Shortcuts and tab completion](11-shortcuts-and-tab-completion.md) - the previous topic
- [Wildcards and globbing](06-wildcards-globbing.md) - the file name patterns that regexes are often confused with
- [Quoting](07-quoting.md) - why patterns need single quotes
- [`grep`](../text-processing/grep.md) - search with regexes
- [`sed`](../text-processing/sed.md) - replace with regexes
- [`awk`](../text-processing/awk.md) - regexes on whole lines or fields
- [`sort`](../text-processing/sort.md) / [`uniq`](../text-processing/uniq.md) - count what a regex extracts
- [`find`](../file-management/find.md) - `-regex` matches whole paths
