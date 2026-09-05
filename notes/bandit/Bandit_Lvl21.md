# Bandit Level 21 → 22

**Date:** 02-09
**Level goal (site):** A program is running automatically at regular intervals.

---

## Approach

"Automatically at regular intervals" on Linux points to **cron** — the scheduler daemon.
System-wide cron jobs live in `/etc/cron.d/`, so that was the first place to look.

```bash
cd /etc/cron.d
ls
```

Found a job named `cronjob_bandit22`.

---

## Reading the cron job

```bash
cat cronjob_bandit22
```

Output:

```
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

Breaking the second line down:

| Field | Meaning |
|---|---|
| `* * * * *` | min / hour / day-of-month / month / day-of-week — all wildcards, so **every minute** |
| `bandit22` | the job runs **as this user**, with that user's permissions |
| `/usr/bin/cronjob_bandit22.sh` | the script being executed |
| `&> /dev/null` | all output (stdout + stderr) discarded |

The `@reboot` line is the same script, just triggered once at boot instead of on a schedule.

Key observation: output is thrown away, so if the script produces anything useful it must be
writing it somewhere itself — not printing it.

---

## Reading the script

```bash
cat /usr/bin/cronjob_bandit22.sh
```

Output:

```bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Two lines:

1. `chmod 644` — sets the temp file to owner read/write, **everyone else read-only**
2. `cat /etc/bandit_pass/bandit22 > ...` — copies bandit22's password into that temp file

---

## Solution

```bash
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Password retrieved. Logged in as bandit22.

---

## Concepts learned

**cron**
- Daemon that runs commands on a schedule
- `/etc/cron.d/` holds system-wide job definitions
- Five-field time spec: minute, hour, day-of-month, month, day-of-week
- System crontabs (unlike user crontabs) include a **username field** specifying which user
  the command runs as

**The actual security lesson**
`/etc/bandit_pass/bandit22` is readable only by bandit22 — that permission was never broken.
The vulnerability is that a **privileged process voluntarily writes its secret to a
world-readable location**. The file permissions on the original were fine; the copy defeated them.

This is a real privilege-escalation pattern: look for scheduled or privileged processes that
drop output into shared directories like `/tmp`.

**Why `&> /dev/null` was a useful clue**
It confirmed the script wasn't meant to communicate through output, which redirected attention
to what it writes to disk instead.

---

## Notes for next time

- Worth checking `/etc/crontab` and `/var/spool/cron/` as well — cron jobs are not only in `/etc/cron.d/`
- `chmod 644` appearing in a script run by another user is itself a red flag worth grepping for
writing i
