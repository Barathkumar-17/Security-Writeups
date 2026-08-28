====================================
BANDIT LEVEL 0-4
====================================
Date: 27-08-2026
Time taken: ~45 min

------------------------------------
GOAL
------------------------------------
Navigate basic Linux file system and find hidden/tricky-named
files containing the password for the next level.

------------------------------------
LEVEL 1 - Dash-named file
------------------------------------
Found file named "-". Reading it directly failed.
Fix: cat ./-

------------------------------------
LEVEL 2 - Filename with spaces
------------------------------------
Filename had spaces in it. Used backslash to escape:
cat ./Spaces\ there\ file

------------------------------------
LEVEL 3 - Hidden file
------------------------------------
Used ls -a to reveal hidden (dot) files, then cat'd it.

------------------------------------
LEVEL 4 - Junk files, one real
------------------------------------
Multiple files with garbage content. Used:
file ./*
to find the one labeled ASCII text, then cat'd that one.

------------------------------------
PASSWORD FOR NEXT LEVEL
------------------------------------
[REDACTED - see local passwords.txt]
