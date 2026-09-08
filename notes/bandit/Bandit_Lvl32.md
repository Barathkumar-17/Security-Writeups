# Bandit Level 32 -> 33

## Task
"After all this git stuff, it's time for another escape." No further hints given.

## Steps taken
1. SSH'd into bandit32 (needed college VPN active for port 2220 access)
2. Landed in a restricted shell: "WELCOME TO THE UPPERCASE SHELL" with `>>` prompt
3. Tried lowercase command (`ls`) -> permission denied, confirms lowercase is fully blocked
4. Tried `$0` (shell variable holding currently running shell, no lowercase letters needed)
   -> dropped into a real `$` shell prompt, escaping the uppercase restriction
5. Ran `whoami` and `cat` on the password file -> got bandit33 password

## Key learning
- "Escape the shell" challenges often hinge on shell **variables** or **built-ins** rather than
  external commands -- `$0` works because it's interpreted by the shell itself, not typed as a
  lowercase external binary, so it slips past an uppercase-only filter.
- When a shell blocks a category of input (e.g. lowercase), look for ways to invoke a shell
  indirectly instead of fighting the filter directly.

## Status
Solved. Escaped restricted uppercase shell via `$0`, obtained bandit33 password.
