# Bandit Level 22 → 23

**Date:** 02-09
**Level goal (site):** A program is running automatically at regular intervals.

---

## Approach

Same wording as level 21 → 22, so same starting point — check the cron directory.

```bash
cat /etc/cron.d/cronjob_bandit23
```

```
@reboot bandit23 /usr/bin/cronjob_bandit23.sh
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh
```

Runs every minute as **bandit23**. Read the script it points to:

```bash
cat /usr/bin/cronjob_bandit23.sh
```

```bash
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

---

## Reading the script

Unlike the previous level, the destination filename isn't hardcoded — it's **computed at runtime**.

| Line | What it does |
|---|---|
| `myname=$(whoami)` | `$( )` runs a command and substitutes its output; stores the current username |
| `echo I am user $myname` | builds a string like `I am user bandit23` |
| `\| md5sum` | hashes that string |
| `\| cut -d ' ' -f 1` | `md5sum` outputs `<hash>  <filename>`; this splits on space and keeps field 1, the hash alone |
| `cat /etc/bandit_pass/$myname > /tmp/$mytarget` | copies that user's password into the computed filename |

So the temp filename is `md5("I am user <username>")`.

---

## The trap

The obvious move is to just run the script and see what happens. That fails.

`whoami` returns whoever is executing the script. Running it as **bandit22** produces
`md5("I am user bandit22")` — a different hash, pointing at a file that does not contain the
bandit23 password.

The script has to be evaluated as it behaves **when cron runs it as bandit23**, not as it
behaves when I run it.

---

## Solution

Compute the hash by hand with the target username substituted in:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

Then read the resulting file:

```bash
cat /tmp/<hash>
```

Password retrieved.

---

## Concepts learned

**Command substitution — `$( )`**
Runs the enclosed command and replaces the whole expression with its output.
`$(whoami)` becomes the username at the moment of execution.

**`cut` for field extraction**
- `-d ' '` sets the delimiter to a space
- `-f 1` selects the first field

Needed here because `md5sum` prints the hash *and* the input filename, and only the hash was wanted.

**The actual lesson: execution context determines behaviour**
The same script produces different results depending on which user runs it. Reading code is not
enough — the question is always *what does this do in the context where it actually runs*.

In privilege escalation work this means simulating the privileged context rather than executing
the script directly. Executing it yourself only tells you what it does with *your* permissions,
which is exactly the thing that isn't interesting.

**Recurring pattern (same as 21 → 22)**
A privileged process writing its output into a shared directory (`/tmp`) defeats the permissions
on the original file. The `/etc/bandit_pass/` permissions were never broken in either level.

---

## Notes for next time

- When a script derives paths or names from runtime values (`whoami`, `hostname`, `date`), always
  recompute those values for the target context by hand
- `md5sum` output format is `<hash><two spaces><filename>` — worth remembering when parsing it
