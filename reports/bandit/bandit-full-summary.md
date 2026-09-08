# Bandit Wargame — Full Summary Report (Levels 0–32)

**Platform:** OverTheWire Bandit
**Status:** Complete (all 32 levels solved; Bandit ends at 32, no level 33)
**Format:** Condensed, grouped by technique/theme rather than level-by-level

---

## 1. File & Directory Discovery

Core Linux navigation skills used throughout, especially early levels.

| Technique | Command | Used for |
|---|---|---|
| Reading oddly-named files | `cat ./-`, escaping spaces with `\` | Filenames starting with `-` or containing spaces |
| Hidden files | `ls -a` | Dotfiles not shown by default |
| Identifying real file type | `file ./*` | Junk files where only one is real ASCII text |
| Search by size | `find . -size 1033c` | Locating a specific file by byte size |
| Search by owner/group | `find / -user X -group Y -size Nc 2>/dev/null` | Cross-filesystem search, permission errors suppressed |
| Search by permission | `find ! -executable`, `find -type f` | Filtering by executability or type (file/dir/link) |
| Directory permission check | `ls -ld` (vs `ls -l`) | `-d` shows the folder's own permissions, not its contents |

**Key lesson:** `ls -l` on a directory lists contents; `ls -ld` lists the directory itself — easy to mix up, worth remembering permanently.

---

## 2. Text Processing & Encoding

| Technique | Command | Used for |
|---|---|---|
| Pattern matching | `grep "word" file` | Finding a password near a known keyword |
| Binary → readable text | `strings file` (then pipe to `grep`) | Extracting text from binary-heavy files without missing matches |
| Duplicate/unique detection | `sort file \| uniq -u` | Finding the one line that appears only once (uniq only compares *adjacent* lines, so sort first) |
| Base64 | `base64 -d` | Decoding encoded content |
| ROT13 | `tr "A-Za-z" "N-ZA-Mn-za-m"` | Reversing a 13-character alphabet shift |
| Diffing files | `diff file1 file2` | Spotting the one changed line between two near-identical files |
| Compression chains | `file`, `gunzip`, `bunzip2`, `tar -xf` | Repeatedly identifying and decompressing a file disguised under the wrong extension, until reaching human-readable output |

**Key lesson:** file extensions can lie — always verify actual type with `file` before assuming how to open something.

---

## 3. SSH, Networking & Raw Protocols

| Technique | Command | Used for |
|---|---|---|
| Command-only SSH login | `ssh user@host -p port command` | Running one command without a full interactive session (used to dodge a `.bashrc` auto-logout) |
| Secure file transfer | `scp -P port user@host:path dest` | Moving a private key between users/levels when direct SSH wasn't possible |
| Key-based login | `ssh -i keyfile user@host -p port` | Logging in with a passkey instead of a password (key file permissions had to be locked down first) |
| Raw TCP connection | `nc host port` | Sending/receiving data manually over a raw socket |
| Listening server | `nc -lvp port` | Hosting a local listener to catch data from a setuid binary |
| Piping input into a listener | `echo password \| nc -lp port` | Feeding data to a service that expects input immediately on connect (no wait time) |
| Port + service scanning | `nmap -sV -p range host` | Identifying which open port ran the SSL service needed (not just which ports were open) |
| TLS/SSL handshake | `openssl s_client -host h -port p` | Connecting to a server that only speaks SSL/TLS, not plaintext |
| Fixing SSL parsing issue | `openssl s_client ... -quiet` | Preventing openssl from misinterpreting a password starting with a reserved letter (`k`) as an internal command |
| Brute-force via script | `for i in {0..9}{0..9}{0..9}{0..9}; do echo "pass $i"; done | nc host port > out.txt` | Testing all 10,000 possible 4-digit PINs against a listening service in one automated pass |

**Key lesson:** `nc` is single-shot and often expects input the instant a connection opens — timing and piping matter more than the command itself.

---

## 4. setuid Binaries & Privilege Boundaries

| Concept | Detail |
|---|---|
| What setuid does | A binary with the `s` bit set (e.g. `-rwsr-x---`) runs with the **file owner's** privileges, not the caller's — regardless of who executes it |
| How to spot one | Look for `s` in the owner-execute permission slot via `ls -l` |
| How it was used | Running a setuid binary with a command argument (e.g. `cat /etc/bandit_pass/bandit20`) to read a file the current user couldn't access directly |
| Recurring pattern | Levels 20 and 26 both used this — second encounter was recognized instantly, confirming the concept had stuck |

**Key lesson:** the goal with setuid binaries isn't "log in as the target user" — it's making the binary perform one privileged action on your behalf.

---

## 5. Cron Jobs & Scheduled-Process Exploitation

The clearest recurring theme across levels 21–24 — four levels, increasing difficulty, same root idea: *a privileged process runs on a schedule; the gap between what it does and what it exposes is the vulnerability.*

| Level | What made it different |
|---|---|
| 21→22 | Script copies a password to a **world-readable** temp file. Simplest case — just read the leaked copy. |
| 22→23 | Destination filename is **computed at runtime** from `whoami` via `md5sum`. Running the script yourself gives the wrong hash — you must simulate the *target user's* execution context by hand (`echo I am user bandit23 | md5sum | cut -d ' ' -f 1`). |
| 23→24 | No leak to read — you must **write and place your own script** in a directory cron scans, get it executed as the privileged user, and have it copy the password out. The script is deleted immediately after running, so every precondition (destination folder exists, correct permissions) must be set up *before* dropping the script. |
| Common thread | `/etc/bandit_pass/` file permissions were never actually broken in any of these — the privileged *process* is what leaks the data, not a permission flaw in the original file. |

**Key lesson:** reading code isn't enough — the real question is always "what does this do in the context where it actually runs," especially when scripts derive values like usernames or hostnames at runtime.

---

## 6. Restricted Shells & Pager/Editor Escapes

| Level | Restriction | Escape method |
|---|---|---|
| 25 | Login shell is a custom script that just runs `more` on a file, then exits | `more` has an internal `!command` escape — but only reachable if the file doesn't fit on one screen. Fixed by shrinking the terminal window so `more` paginates instead of dumping and exiting. |
| 26 | Same `more`-based shell, but `!command` was **disabled** in this build | Confirmed via harmless tests (`!id`, `!echo`) producing zero output — not a syntax problem, the feature itself was off. Pivoted to `more`'s `v` key, which opens the current file in **vim**. Inside vim, `:!bash` returned immediately (unstable), but `:set shell=/bin/bash` + `:shell` gave a proper persistent shell. |
| 32 | A shell that only accepts **uppercase** commands (`WELCOME TO THE UPPERCASE SHELL`) | Lowercase external commands were flatly rejected. `$0` — a shell **variable** holding the currently-running shell, not a typed lowercase command — bypassed the filter and dropped into a normal `$` shell. |

**Key lesson:** when one escape path is blocked, look for alternate interactive commands inside the same restricted program (a pager can lead to an editor, which has its own escape). When a filter blocks a category of input (e.g. lowercase), shell variables/built-ins often aren't caught by it because they're interpreted by the shell itself, not typed as external binaries.

**Methodology note:** in both 25 and 26, the first debugging step was confirming *whether input was even reaching the program* (spacebar test, harmless test commands) before assuming the exploit technique itself was wrong. Worth keeping as a general first move when an interactive escape fails silently.

---

## 7. Git-Based Levels (27–31)

Five consecutive levels, each hiding the password in a different part of a git repository — effectively a tour of "everywhere a secret can hide in git."

| Level | Where the password was hidden | Command that found it |
|---|---|---|
| 27 | Plainly in a file after cloning | `git clone ssh://...`, then read the file |
| 28 | Censored in the current file, but visible in an earlier commit | `git log` to find the commit, `git diff <commit>` to see the change |
| 29 | On a separate **remote branch** (`dev`), not `master` | `git branch -a` to list all branches, `git switch --detach remotes/origin/dev` |
| 30 | Hidden as a **git tag**, not in any branch or commit history | `git tag` to list tags, `git show secret` to read the tagged content |
| 31 | Required **writing to** the repo (create `key.txt`, commit, push) rather than reading | `git add -f` (needed to override `.gitignore`), commit, push |

**Full git-secret-hunting checklist built from this run:**
branches (`git branch -a`) → commit history (`git log -p`) → deleted files (`git log --diff-filter=D --summary`) → tags (`git tag`) → and don't skip opening files that *look* empty (`cat -A` rules out hidden characters).

**Key lesson:** secrets in real repos aren't only in the current `master` branch — attackers and pentesters alike need to check branches, full commit history, and tags, since leftover credentials commonly get left behind in exactly those spots.

---

## Overall Takeaways

1. **Permissions were rarely "broken"** — most levels exploited a *process* voluntarily exposing something (writing to `/tmp`, running with elevated rights, executing attacker-placed code) rather than a flaw in file permissions themselves. This is realistic: real privilege escalation is usually about gaps in process behavior, not cracked permissions.
2. **Context matters more than code.** Several levels (22→23 especially) only made sense once evaluated from the *target user's* execution context, not the solver's own.
3. **Systematic elimination beats guessing.** The most efficient debugging (levels 25, 26, 30) came from testing one variable at a time — confirm input reaches the program, confirm a technique produces *any* effect, then narrow down — rather than jumping between unrelated fixes.
4. **Live capture + batch conversion worked as a workflow.** Notes were captured in the moment per level; this report is the batch-converted, condensed version — consistent with the stated goal of avoiding write-up fatigue.
