# Security-Writeups

A self-directed cybersecurity learning journal. This repo tracks my hands-on progress through a structured 6-month roadmap, with a primary focus on **reverse engineering** and a secondary focus on **penetration testing**.

Everything here is my own work — solved labs, raw notes, and polished write-ups — kept public as a running record of how I learn, not just what I solve. Failed attempts are kept on purpose, because methodology is the point.

---

## How this repo is organised

```
notes/       raw scratch notes, captured live during each session, committed as-is
reports/     polished pentest-format write-ups (scope, methodology, findings, remediation)
templates/   reusable write-up template
```

Both `notes/` and `reports/` are split by platform:

```
bandit/  portswigger/  picoctf/  reverse-engineering/
```

The workflow: notes are written live while working, then batch-converted into clean reports later. Raw first, polished second.

---

## Progress

### Month 1 — complete

**Main track — OverTheWire Bandit (levels 0–32, fully complete)**
A condensed, theme-grouped summary report is in `reports/bandit/`, covering:
file discovery, text processing, networking & SSH, setuid binaries, cron-job exploitation, restricted-shell escapes, and secrets hidden in git.

**Reverse engineering track — compiler behaviour in Ghidra**
Compiled my own C and studied how it changes under optimisation. Summary report in `reports/reverse-engineering/`, covering:
baseline workflow, dead code elimination, constant folding, strength reduction, side effects as optimisation barriers, control-flow transformation (for → do-while), and optimisation / hardening fingerprints.

### Month 2 — in progress

- **Main track:** PortSwigger Web Security Academy — SQL injection, XSS, authentication flaws (Burp Suite Community set up on Ubuntu).
- **RE track:** x86-64 assembly basics — reading and writing simple assembly.

---

## Full roadmap

Two parallel tracks over six months. Order matters more than exact dates.

| Month | Main track | Reverse engineering track |
|-------|-----------|---------------------------|
| 1 | Bandit / Linux basics | Compile own C, read it in Ghidra |
| 2 | PortSwigger: SQLi, XSS, auth | x86-64 assembly basics |
| 3 | PortSwigger: advanced topics | crackmes.one (easy) |
| 4 | picoCTF (all categories) | pwn.college (stack basics) |
| 5 | HackTheBox / TryHackMe + "Jr Penetration Tester" path | Buffer overflows + intro to ROP |
| 6 | Pick a specialisation | — |

**Long-term goal:** a role in the cybersecurity field, working toward OSCP.

---

## Environment

- Ubuntu (VirtualBox VM)
- Ghidra — reverse engineering
- Burp Suite Community — web security
- gcc — compilation experiments
