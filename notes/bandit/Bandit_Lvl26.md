# Bandit Level 26

**Goal:** From bandit26, find a way to escalate to bandit27.

## Steps
1. Repeated the level 25 breakout method (`!bash` inside `more`) — but this time it silently failed.
   - Debugging process:
     - `!bash` → returned instantly to `--More--`, no shell.
     - `!/bin/sh` → same result.
     - `!id`, `!echo hello` → zero output, not even a flicker.
     - `!id > /tmp/out.txt` then `!cat /tmp/out.txt` → file empty/no output.
   - Conclusion: `!command` shell-escape was **disabled entirely** in this `more` build (not a syntax issue — confirmed keystrokes were reaching `more` fine via spacebar paging test).
2. Switched to a different `more` escape: pressed `v` at the `--More--` prompt.
   - Opened the file in **vim** (confirmed via `"~/text.txt" [readonly]` status line).
3. Used vim's shell-escape:
   - `:!bash` → ran but returned immediately ("Press ENTER or type command to continue"), not stable.
   - Fix: `:set shell=/bin/bash` then `:shell` → dropped into a **stable, real shell** as bandit26.
4. Explored home directory:
   ```
   ls -la
   ```
   Found:
   - `text.txt` (used for the shell trick)
   - `bandit27-do` — permissions `-rwsr-x---`, owner `bandit27`, group `bandit26`
5. The `s` in the owner-execute position = **setuid bit**. Binary runs with bandit27's privileges regardless of who executes it (same concept as level 20).
6. Ran the binary with no args → it printed usage instructions saying to pass it a command to run.
7. Used it to read bandit27's password directly, leveraging bandit27's own file access:
   ```
   ./bandit27-do cat /etc/bandit_pass/bandit27
   ```
   → got bandit27's password.

## Key concepts
- Not every pager/editor allows shell escapes — some builds disable `!command` for security. Always verify with a harmless test (`!id`) before assuming syntax is wrong.
- When one escape method is blocked, look for **alternate interactive commands** in the same restricted program (e.g. `more`'s `v` → opens vim → vim has its own shell escape).
- `vim`'s `:!cmd` can return immediately without persisting; `:set shell=...` + `:shell` gives a proper interactive shell instead.
- **Setuid binaries** run with the *file owner's* privileges, not the caller's — no password needed for the target user. The goal is to make the binary perform an action on your behalf (e.g. reading a file you can't access directly), not to "get the password" through login.

## Methodology note
- Initially assumed level 26 breakout would be identical to level 25 — wrong assumption caught early by testing with harmless commands (`!id`, `!echo`) instead of guessing at `bash` syntax issues.
- Systematic elimination order: confirm input reaches program (spacebar test) → confirm command escape produces any output at all → isolate whether it's shell-specific or command-execution-wide → pivot to alternate escape path once `!` confirmed fully disabled.
- Setuid re-encountered from level 20 — recognizing the `s` permission bit pattern was immediate the second time, showing the concept stuck.
