# Bandit Level 29 -> 30

## Task
Git repo at ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo (port 2220).
Password same as bandit30-git.

## Steps taken
1. Cloned repo with git clone over ssh (port 2220)
2. Checked `git log --all` vs `git log` -> no difference, ruled out extra branches
3. Checked `git status` -> clean, nothing staged/unstaged
4. Checked `git log -p` -> only one commit, no diffs of interest
5. Checked `git log --diff-filter=D --summary` -> no deleted files
6. `ls -la` -> only .git and README.md present
7. Opened README.md -> appeared empty
8. `cat -A README.md` -> confirmed genuinely empty (no hidden chars, just EOL `$`)
9. Checked `git tag` -> found a tag named `secret`
10. `git show secret` -> revealed the password

## Key learning
- Passwords/secrets can hide in git **tags**, not just branches or commit history.
- Full checklist for git-based levels going forward:
  branches (`git branch -a`) -> commit history (`git log -p`) -> deleted files (`--diff-filter=D`) -> tags (`git tag`)
- Empty-looking files are worth double-checking with `cat -A` to rule out hidden chars before moving on.

## Status
Solved. Password obtained via `git show secret`.
