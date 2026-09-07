# Bandit Level 25

**Goal:** Log into bandit26 despite its non-standard login shell; figure out what that shell is and break out of it.

## Steps
1. Checked `/etc/passwd` — world-readable even though only root can write to it.
   - Found bandit26's shell set to `/usr/bin/showtext` instead of `/bin/bash`.
2. Read the `showtext` script (confirmed read-only, no write perms via `ls -l`).
   - Script runs `more` on `text.txt`, then exits.
3. `more` is interactive, not just a viewer — has its own internal commands.
   - `man more` → `!command` runs a shell command from inside `more`.
4. First attempt failed: SSH disconnected instantly, no prompt shown.
   - Cause: `text.txt` short enough to fit one screen → `more` displays fully and exits before input possible.
5. Fix: shrunk terminal window height (~5-10 lines) before connecting.
   - Forced `more` to treat content as overflowing → paused at `--More--` instead of exiting.
6. At `--More--` prompt, typed `!bash` → dropped into a real bash shell as bandit26.

## Root cause of "instant disconnect" bug
Terminal height, not exploit logic. Window size controls whether `more` pages or dumps-and-exits.

## Key concepts
- `/etc/passwd`: world-readable, lists every user's UID, home dir, login shell. Read ≠ write access — always check both.
- Restricted login shells limit a user to one task, but any interactive sub-command inside that program (pagers, editors, mail clients) can be a breakout vector.
- `more`'s `!command` (pager escape) — same idea applies to `less`, `vim`, `man`, etc.
- Terminal dimensions can be manipulated to force different code paths (paging vs non-paging).

## Methodology note
Initial failure (blank disconnect) traced by:
1. Confirming SSH command syntax was correct.
2. Checking if any output appeared before disconnect (banner only, no file text).
3. Reasoning about *why* `more` would exit without pausing → file length vs terminal height.

Lesson: when an interactive escape fails silently, check whether the interactive prompt is even being reached before assuming the escape technique itself is wrong.
