# clear

## What is it?
`clear` wipes the visible terminal screen and moves the prompt to the top. It does not delete your command history or any files.

## Syntax
```bash
clear [OPTIONS]
```

## Visual Overview
> `clear` empties the visible screen. `clear -x` keeps the scrollback so you can still scroll up to the old output.

```mermaid
flowchart LR
    i1[Screen full of old output] -->|clear| o1[Empty screen<br/>old output gone from scrollback]
    i1 -->|clear -x| o2[Empty screen<br/>scroll up still shows the old output]
    classDef inp fill:#DCEBFF,stroke:#2F6FDE,color:#10213F
    classDef cmd fill:#FFF0C2,stroke:#C98A00,color:#3A2A00
    classDef out fill:#D9F5E3,stroke:#1F9D55,color:#0B2E1A
    classDef bad fill:#FDE0E0,stroke:#D13B3B,color:#4A0F0F
    classDef note fill:#ECEFF3,stroke:#7B8794,color:#1F2933
    class i1 inp
    class o1,o2 out
```

## Options/Flags
| Flag | Description |
|------|-------------|
| `-x` | Do not clear the scrollback buffer |
| `-T TERM` | Use the given terminal type instead of `$TERM` |
| `-V` | Print the version and exit |

## Usage Examples

### Example 1: Clear the screen
**Command:**
```bash
ls
clear
```
**Sample Output:**
```text
(the screen is empty and the prompt is at the top)
```

### Example 2: Keep the scrollback (`-x`)
After `clear -x` you can still scroll up in the terminal to see earlier output.

**Command:**
```bash
echo "old output"
clear -x
```
**Sample Output:**
```text
(the screen is empty; "old output" is still visible when you scroll up)
```

### Example 3: Specify the terminal type (`-T`)
**Command:**
```bash
clear -T xterm-256color
```
**Sample Output:**
```text
(the screen is empty and the prompt is at the top)
```

### Example 4: Show the version (`-V`)
**Command:**
```bash
clear -V
```
**Sample Output:**
```text
ncurses 6.3.20211021
```
(The version depends on your system.)

### Example 5: Keyboard shortcut
Press `Ctrl+L` in bash for the same effect without typing the command.

## Pitfalls / Gotchas
- `clear` only hides output. Commands, history and files remain untouched.
- Older versions of `clear` also erase scrollback; the `-x` flag is not available everywhere.
- If `$TERM` is not set (for example in cron), `clear` prints `TERM environment variable not set.`
- To reset a broken terminal after printing binary data, use `reset` instead.

## Related Commands
- [`history`](history.md) - command history, which `clear` does not erase
- `reset` - fully reinitialize the terminal
- `tput` - control terminal features
- `Ctrl+L` - shell shortcut for clearing the screen

---
<!-- nav -->
[← Previous: `history`](history.md) | [Learning path](../README.md) | [Next: `alias` →](alias.md)
