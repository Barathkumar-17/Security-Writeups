# Bandit Level 23 → 24

**Date:** 05-09
**Level goal (site):** A program is running automatically at regular intervals.

Same wording as 21 → 22 and 22 → 23, but this is a step up — first level where the solution
requires **writing and placing a script**, not just reading one.

---

## Approach

Same starting point as always:

```bash
cat /etc/cron.d/cronjob_bandit24
cat /usr/bin/cronjob_bandit24.sh
```

The script (paraphrased) does roughly this:
- Runs every minute as **bandit24**
- Looks inside a specific directory (`/var/spool/bandit24/foo`)
- Runs **every script it finds there**, as bandit24
- Deletes each script immediately after running it, regardless of success or failure

That last part is the key detail: whatever the script does, it needs to happen and leave a
result behind in one shot, because the script itself won't survive to be inspected afterwards.

---

## Plan

Since anything dropped in `foo` gets executed as bandit24, the move is:

1. Write a script that copies bandit24's password to a location I can read
2. Give the script execute permissions
3. Drop it into `foo`
4. Wait for cron to pick it up (runs every minute)
5. Read the result

---

## The script

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/band1928/pass.txt
```

Straightforward copy, same pattern as levels 21–22. The only new consideration is that this
script has to be placed correctly and be executable, and the **destination has to be prepared
in advance** since bandit24 (not me) is the one running it.

---

## Things that had to be checked (and why)

**1. Does the output directory exist before the script runs?**
`/tmp/band1928` had to be created ahead of time. The script only redirects output into it —
it doesn't create the folder. If the folder isn't there, the write fails silently.

**2. Can bandit24 write into that directory?**
The folder was created as bandit23, but the script executing inside it runs as **bandit24**.
Checked with:
```bash
ls -ld /tmp/band1928
```
Result: `drwxrwxrwx` — world-writable, so bandit24 can create the file inside it.

Note on `ls -ld`: the `-d` flag matters here — without it, `ls` lists the *contents* of the
directory instead of the permissions of the directory itself. Easy mistake to make when
checking a folder vs. a file.

**3. Are the script's own permissions correct?**
Checked with:
```bash
ls -l /tmp/band1928/myscript.sh
```
Result: `-rwxrwxrwx` — the leading `-` (not `d`) confirms it's a file, and full rwx for
everyone means cron can execute it regardless of which user runs it.

**4. Once bandit24 creates `pass.txt`, can bandit23 read it back?**
Since the destination directory is world-writable/readable, yes — the file bandit24 creates
inside it inherits an accessible location.

---

## Solution

After placing the executable script in `/var/spool/bandit24/foo` and waiting under a minute
for cron to pick it up:

```bash
cat /tmp/band1928/pass.txt
```

Password retrieved.

---

## Concepts learned

**Permission bit reading (`ls -l` / `ls -ld`)**
First character tells you the type: `d` = directory, `-` = regular file.
Remaining nine characters are owner / group / other permissions (r/w/x each).
`-d` on `ls` is required to inspect a directory's own permissions rather than its contents.

**Why the output location has to be prepared in advance**
The executing script is deleted right after it runs — success or failure. There's no chance to
inspect it afterward or fix a missing directory mid-flight. Everything the script depends on
(destination folder, permissions) has to already be correct before dropping it in.

**Cron executing arbitrary scripts from a writable directory is a real privilege escalation vector**
Any process that runs as user X and executes files from a directory writable by user Y is a
direct path for Y to get code executed as X. This is the clearest example of that pattern so far
in Bandit — not just reading a leaked value, but getting arbitrary code run under a more
privileged account.

**Recurring theme across levels 21–24**
Every one of these levels has been some version of: "a privileged process does something on a
schedule — read what it does, exploit the gap between its permissions and what it exposes."

---

## Notes for next time

- When a script deletes itself after running, plan for that — nothing about its execution can be
  debugged after the fact, so all preconditions must be verified beforehand
- `ls -ld` vs `ls -l` on a directory is a distinction worth remembering permanently — checked it
  fresh this level, worth not re-deriving it every time
