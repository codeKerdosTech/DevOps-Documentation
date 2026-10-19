# Variables: Giving Your Script a Memory

> **Before this page:** you should have read [01 What is a shell script](01-what-is-a-shell-script.md) and be able to write a small script, make it executable and run it with `./script.sh`.
>
> **By the end of this page** your scripts will be able to remember values, reuse them, and be changed in one place instead of ten.

---

## 1. Start with a problem

In the last page you wrote `morning.sh`. Now imagine a script that checks a server called `web1`:

```bash
echo "Checking web1"
ping -c 1 web1
echo "Finished checking web1"
echo "Report for web1 saved"
```

The name `web1` appears **four times**. Next week your manager says: *"Use `web2` instead."*

You must find and change all four places. Miss one, and your script quietly checks the wrong server and reports on a different one. In a script with 200 lines this goes wrong all the time.

What you want is to write the name **once**, give it a label, and let every other line refer to the label. Change the label's content once, and the whole script follows.

**That is what a variable is.**

---

## 2. The big idea: a labelled jar

Think of a kitchen shelf with jars. Each jar has a **label** on the outside, such as `SUGAR`, and something **inside**, the sugar. When a recipe says "add the sugar", you do not need to know what is inside. You find the jar by its label and use what is in it. If tomorrow you fill the jar with brown sugar, the recipe does not change at all.

| Kitchen | Shell script |
|---------|--------------|
| A jar | A **variable** |
| The label on the jar | The variable's **name** (`server`) |
| What is inside the jar | The variable's **value** (`web1`) |
| Filling the jar | **Assigning**: `server=web1` |
| Taking the contents out to use them | **Reading**: `$server` |

A variable is simply **a name that holds a value** while your script runs.

```mermaid
flowchart LR
    A["Line 1 puts Asha in a jar labelled name"] --> B["The shell keeps the jar in memory"]
    B --> C["Line 2 says echo Hello and dollar name"]
    C --> D["The shell swaps dollar name for Asha"]
    D --> E["echo prints Hello Asha"]
    classDef start fill:#6C5CE7,stroke:#4834D4,stroke-width:2px,color:#fff
    classDef proc fill:#00B8D9,stroke:#0087A8,stroke-width:2px,color:#fff
    classDef ok fill:#2ECC71,stroke:#1E8449,stroke-width:2px,color:#fff
    classDef out fill:#FD79A8,stroke:#E84393,stroke-width:2px,color:#fff
    class A,B start
    class C,D proc
    class E out
```

---

## 3. Your first variable (do this with me)

Go to your practice folder (`cd ~/practice`) and create `greet.sh`.

Let's say we have this file `greet.sh`:

**Input file** (`greet.sh`):
```bash
name="Asha"
echo "Hello, $name"
echo "Welcome, $name. Nice to meet you, $name!"
```

**Command:**
```bash
bash greet.sh
```
**Sample Output:**
```text
Hello, Asha
Welcome, Asha. Nice to meet you, Asha!
```

Line 1 **fills the jar**: the name is `name` and the value is `Asha`.
Lines 2 and 3 **use the jar**. Writing `$name` means *"take out whatever is in the jar called `name`"*. The shell replaces `$name` with `Asha` **before** `echo` runs, so `echo` never sees the word `$name`, only `Asha`.

Now the useful part. Let's change the value and watch the same line print something new:

Let's say we have this file `greet2.sh`:

**Input file** (`greet2.sh`):
```bash
name="Asha"
echo "Hello, $name"
name="Ravi"
echo "Hello, $name"
```

**Command:**
```bash
bash greet2.sh
```
**Sample Output:**
```text
Hello, Asha
Hello, Ravi
```

The line `echo "Hello, $name"` is **identical** both times, yet it prints different results. The word *variable* simply means *its content can vary*. The script reads top to bottom (remember page 01), so each `echo` sees whatever the jar holds **at that moment**.

---

## 4. The two golden rules

Beginners break these two rules constantly. Learn them now and you will skip hours of confusion.

### Rule 1: Writing a value has NO spaces around `=`

Right:
```bash
name="Asha"
```
Wrong:
```bash
name = "Asha"
```

Let's see what the shell does with the wrong version.

Let's say we have this file `bad.sh`:

**Input file** (`bad.sh`):
```bash
name = "Asha"
echo "Hello, $name"
```

**Command:**
```bash
bash bad.sh
```
**Sample Output:**
```text
bad.sh: line 1: name: command not found
Hello, 
```

With spaces, the shell thinks `name` is the **name of a program** and `=` and `"Asha"` are things you are handing to it. There is no program called `name`, so you get *command not found*. The jar was never filled, so the second line prints an empty greeting.

> **Memory trick:** in the shell, the `=` must be glued to both sides, like `name=Asha`. No gaps at all.

### Rule 2: Writing uses the bare name, reading uses a `$`

- When you **put something in** the jar, write only the name: `city="Pune"`
- When you **take something out**, put a `$` in front: `$city`

Let's say we have this file `dollar.sh`:

**Input file** (`dollar.sh`):
```bash
city="Pune"
echo city
echo $city
```

**Command:**
```bash
bash dollar.sh
```
**Sample Output:**
```text
city
Pune
```

Without the `$`, `echo` just prints the word `city`. With the `$`, the shell opens the jar and prints what is inside. Think of `$` as the words **"the contents of"**: `$city` means *"the contents of city"*.

The opposite mistake is also common: writing `$` when filling.

Let's say we have this file `da.sh`:

**Input file** (`da.sh`):
```bash
$name="Asha"
```

**Command:**
```bash
bash da.sh
```
**Sample Output:**
```text
da.sh: line 1: =Asha: command not found
```

The shell first replaced the empty `$name` with nothing, and was left with `=Asha`, which it tried to run as a command. So: **bare name to write, `$name` to read.**

---

## 5. Order matters: fill the jar before you use it

A script runs top to bottom. If you read a jar **before** you fill it, you get an empty value. The shell does **not** complain.

Let's say we have this file `order.sh`:

**Input file** (`order.sh`):
```bash
echo "Hello, $user"
user="Asha"
echo "Hello, $user"
```

**Command:**
```bash
bash order.sh
```
**Sample Output:**
```text
Hello, 
Hello, Asha
```

The first line printed nothing after the comma because `user` was still empty at that point. Always set your variables **above** the lines that use them.

---

## 6. Always put quotes around `$name`

This is the habit that separates tidy scripts from dangerous ones. Look at what happens when a value contains a **space**.

Let's say we have this file `quotes.sh`, run inside an empty folder:

**Input file** (`quotes.sh`):
```bash
file="my notes.txt"
touch $file
ls
```

**Command:**
```bash
bash quotes.sh
```
**Sample Output:**
```text
my
notes.txt
```

We wanted **one** file called `my notes.txt`, but we got **two** files: `my` and `notes.txt`. Without quotes, the shell sees `touch my notes.txt`, splits the value at the space and passes two separate names.

Now the same script with quotes, in a fresh empty folder:

Let's say we have this file `quotes2.sh`:

**Input file** (`quotes2.sh`):
```bash
file="my notes.txt"
touch "$file"
ls
```

**Command:**
```bash
bash quotes2.sh
```
**Sample Output:**
```text
'my notes.txt'
```

One file, as intended. (When a file name contains a space, `ls` shows it inside quote marks to make that clear.)

The danger is bigger than it looks. Imagine `rm -rf $folder` where `folder` contains `old backups`: without quotes you could delete two completely different things. So the rule is simple:

> **Write `"$name"`, with double quotes, every time.**

### Double quotes versus single quotes

| Quotes | What happens to `$name` | Use when |
|--------|-------------------------|----------|
| `"double"` | The shell **opens the jar** and pastes the value | Almost always |
| `'single'` | Everything is kept **exactly as typed**, nothing is replaced | You really want a literal `$` |

Let's say we have this file `sq.sh`:

**Input file** (`sq.sh`):
```bash
name="Asha"
echo "double: Hello, $name"
echo 'single: Hello, $name'
```

**Command:**
```bash
bash sq.sh
```
**Sample Output:**
```text
double: Hello, Asha
single: Hello, $name
```

---

## 7. When the name runs into other letters: `${name}`

Suppose you want to print `mangojuice`, where `mango` comes from a variable.

Let's say we have this file `br.sh`:

**Input file** (`br.sh`):
```bash
fruit="mango"
echo "I like $fruitjuice"
echo "I like ${fruit}juice"
```

**Command:**
```bash
bash br.sh
```
**Sample Output:**
```text
I like 
I like mangojuice
```

In the first line the shell looked for a jar called `fruitjuice`, which does not exist, so it printed nothing. Curly braces `${fruit}` tell the shell exactly where the name ends. Use braces whenever letters, digits or underscores come directly after the variable.

---

## 8. Putting a command's answer into a variable

In page 01 you used `$(date +%A)` to paste a command's answer into a line. You can do exactly the same to **fill a jar with a command's answer**. This is one of the most useful things in scripting.

Let's say we have a folder `f` that holds three empty files `a`, `b` and `c`, and this file `cs.sh`:

**Input file** (`cs.sh`):
```bash
today=$(date +%d-%b-%Y)
count=$(ls f | wc -l)
echo "Date  : $today"
echo "Files : $count"
```

**Command:**
```bash
bash cs.sh
```
**Sample Output:**
```text
Date  : 20-Oct-2026
Files : 3
```

Read `$( ... )` as **"run this, and give me what it prints"**. The date and the file count are produced when the script runs, so tomorrow the same script prints tomorrow's date.

---

## 9. Variables that already exist

You do not need to create every variable yourself. Your system already has some ready to use:

| Variable | What it holds |
|----------|---------------|
| `$USER` | Your login name |
| `$HOME` | Your home folder |
| `$PWD` | The folder you are in right now |

Let's say we have this file `bi.sh`:

**Input file** (`bi.sh`):
```bash
echo "User   : $USER"
echo "Home   : $HOME"
echo "Folder : $PWD"
```

**Command:**
```bash
bash bi.sh
```
**Sample Output:**
```text
User   : devops
Home   : /home/devops
Folder : /home/devops/practice
```

Your own name and folders will appear instead. You have also already met one more: `$?`, the exit code, in page 01. In the next page you will meet `$1`, `$2` and friends, which carry values **into** your script from outside.

---

## 10. Variables can hold numbers too

A variable can hold a number, and the shell can do simple whole-number maths for you. The trick is to wrap the sum in `$(( ... ))`.

Let's say we have this file `ar.sh`:

**Input file** (`ar.sh`):
```bash
apples=5
more=3
total=$((apples + more))
echo "Total: $total"
total=$((total + 1))
echo "After one more: $total"
echo "Text, not math: $apples+$more"
echo "10 / 3 = $((10 / 3))"
```

**Command:**
```bash
bash ar.sh
```
**Sample Output:**
```text
Total: 8
After one more: 9
Text, not math: 5+3
10 / 3 = 3
```

Three things to notice:

- `$(( ... ))` means *"calculate this"*. Inside it you can write variable names without the `$`.
- Without `$(( ))` the shell treats everything as plain text, so `$apples+$more` printed `5+3`.
- The shell only does **whole numbers**. `10 / 3` gives `3`, not `3.33`. Decimal maths needs another tool, which you will meet much later.

The line `total=$((total + 1))` is how a script **counts**: take the old value, add one, store it back. You will use it a lot in loops.

---

## 11. Naming your variables

| Rule | Good | Bad |
|------|------|-----|
| Use letters, digits and `_` only | `backup_dir`, `server2` | `backup-dir`, `my var` |
| Do not start with a digit | `server2` | `2server` |
| **Capital letters matter** | `Name` and `name` are two different jars | |
| Pick a name that explains itself | `log_file` | `x`, `lf` |

Let's see the capital-letters rule in action.

Let's say we have this file `case.sh`:

**Input file** (`case.sh`):
```bash
Name="Asha"
name="Ravi"
echo "$Name and $name"
```

**Command:**
```bash
bash case.sh
```
**Sample Output:**
```text
Asha and Ravi
```

**A common style** that you will see in real scripts: use `lower_case` for everyday values, and `UPPER_CASE` for **settings** that stay the same through the whole script.

---

## 12. Upgrade `morning.sh`: settings at the top

Let's use everything on the script from page 01. We move the things that might change into variables at the **top**. Anyone who wants to adapt the script then edits only those first lines.

Let's say we now have this file `morning.sh`:

**Input file** (`morning.sh`):
```bash
#!/bin/bash
# morning.sh - settings live at the top, so they are easy to change

TEAM="Platform team"
DISK="/"
today=$(date +%d-%b-%Y)

echo "Good morning, $TEAM!"
echo "Folder : $PWD"
echo "Date   : $today"
echo "Disk   : $DISK"
df -h "$DISK" | tail -1
```

**Command:**
```bash
chmod +x morning.sh
./morning.sh
```
**Sample Output:**
```text
Good morning, Platform team!
Folder : /home/devops/practice
Date   : 20-Oct-2026
Disk   : /
/dev/sdd       1007G   16G  940G   2% /
```

To check a different disk, you change **one line** (`DISK="/var"`) and the script follows. Compare that with the `web1` problem at the start of this page. Notice also the quotes in `df -h "$DISK"`, as promised in section 6.

---

## 13. The silent mistake: a typo in a variable name

Look closely at this script.

Let's say we have this file `typo.sh`:

**Input file** (`typo.sh`):
```bash
server="web1"
echo "Deploying to $sever"
```

**Command:**
```bash
bash typo.sh
```
**Sample Output:**
```text
Deploying to 
```

The jar is called `server`, but we wrote `$sever`. The shell does **not** raise an error. It treats a missing jar as an empty value and carries on. In a real deploy script, an empty value can mean "deploy to nowhere" or even "delete the wrong folder". Later in this series you will learn a switch (`set -u`) that makes the shell shout in this situation. For now, **double-check your spelling**.

---

## 14. Common mistakes at a glance

| What you wrote | What goes wrong | Fix |
|----------------|-----------------|-----|
| `name = "Asha"` | `name: command not found` | No spaces: `name="Asha"` |
| `echo name` | Prints the word `name` | Add the dollar: `echo "$name"` |
| `$name="Asha"` | `=Asha: command not found` | No `$` when writing: `name="Asha"` |
| `touch $file` | A value with spaces splits into pieces | Quote it: `touch "$file"` |
| `echo "$fruitjuice"` | Shell looks for the wrong jar | Use braces: `"${fruit}juice"` |
| Using a variable above its assignment | It is empty | Set it first |
| `echo 'Hello $name'` | Prints `$name` literally | Use double quotes |
| Misspelled `$sever` | Silently empty | Check spelling |
| `1st_value=5` | Names cannot start with a digit | `first_value=5` |

---

## 15. Words to remember

| Word | Simple meaning |
|------|----------------|
| **Variable** | A name that holds a value while the script runs (a labelled jar) |
| **Assign** | Put a value in a variable: `name="Asha"` |
| **Expand** | The shell swapping `$name` for its value |
| **`$name`** | "The contents of `name`" |
| **`${name}`** | Same, but with a clear end to the name |
| **`$(command)`** | "Run the command and give me what it prints" |
| **`$(( ))`** | "Calculate this, whole numbers only" |
| **String** | Plain text, like `"Asha"` |

---

## 16. Try it yourself

1. **Introduce yourself.** Write `me.sh` with three variables: `name`, `role` and `city`. Print one sentence that uses all three, for example `Asha works as a DevOps engineer in Pune`.
2. **Break the rule.** Add spaces around one `=`, run it and read the error. Then fix it.
3. **Spot the bug.** What does this print? Run it to check, then fix it.
   ```bash
   app="shop"
   echo "Starting $appserver"
   ```
4. **Count something.** Write `count.sh` that starts a variable at `10`, adds `5`, then subtracts `3`, printing the value after each step.
5. **Build a tiny report.** Write `report.sh` with a `PROJECT` variable at the top, a `today` variable filled with `$(date +%F)`, and a number of files in the current folder filled with `$(ls | wc -l)`. Print all three in a neat sentence.

<details>
<summary>Answer for exercise 3 (open only after trying)</summary>

It prints `Starting ` with nothing after it, because there is no variable called `appserver`. The shell read `$appserver` as one long name. Use braces:

```bash
app="shop"
echo "Starting ${app}server"
```
</details>

---

## 17. Quick recap

- A **variable** is a labelled jar: a name that holds a value.
- **Write** with no spaces and no dollar: `name="Asha"`.
- **Read** with a dollar: `"$name"`.
- **Always quote** your variables: `"$name"`. Spaces in values will otherwise split into pieces.
- Use `${name}` when other letters follow the name.
- Fill a variable with a command's answer: `today=$(date)`.
- Do whole-number maths with `$(( ))`.
- Put changeable settings at the **top** of the script as variables.
- A misspelled variable is silently empty, so check your spelling.

**Next page:** [03 Input: arguments and `read`](03-arguments-and-input.md), where you will give your script different values every time you run it, without editing the file.
