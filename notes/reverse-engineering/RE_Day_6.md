# RE Day 6: Early-Exit (break) Loop vs Compiler Optimization

## Goal
Test whether a `break` on a runtime-unknown condition survives `-O2` optimization,
building on the argc-based loop from the previous session.

## Attempt 1 — no printf inside loop
Wrote a loop with an early-exit `break` condition based on a `scanf`-read value, but
with no side effect (no printf) inside the loop body.

Compiled `-O0` and `-O2`, compared in Ghidra:
- `-O0`: full loop present, `break` condition folded into the loop's continue-condition
- `-O2`: loop **completely deleted** — nothing left but the `scanf` call and stack-check

## Key finding from attempt 1
A `break` alone is not enough to stop the compiler from removing a loop. Without an
observable side effect (I/O, memory write the compiler can't prove is dead, etc.), the
optimizer still eliminates the loop entirely — same root cause as the very first dead-loop
experiment (Day 5), just with a break condition added this time. The break didn't change
the outcome because the loop still had no visible effect on the outside world.

## Attempt 2 — printf added inside loop
Added `printf("%d", i);` inside the loop body, right before the increment. Recompiled.

### -O0 decompile
```c
for (local_14 = 0; local_14 < param_1 && local_14 != local_18; local_14++) {
    printf("%d", local_14);
}
```
Matches source almost exactly — no transformation at this level.

### -O2 decompile
```c
if (0 < param_1) {
    iVar1 = 0;
    do {
        if (local_24 == iVar1) break;
        __printf_chk(2, ..., iVar1);
        iVar1++;
    } while (param_1 != iVar1);
}
```

## What changed and why

**1. Loop survived this time.**
`printf` is a real side effect (writes to stdout) that the compiler cannot prove is
unnecessary, so it can no longer delete the loop outright.

**2. The `break` condition remained a genuine runtime branch.**
`if (local_24 == iVar1) break;` is preserved as-is. Since `local_24` comes from `scanf`
(unknown at compile time), the compiler cannot predict whether/when the break fires, so
it has no choice but to keep it as a real conditional jump.

**3. `for` became `do-while` with a leading guard.**
`-O0`'s `for (i = 0; i < param_1 && ...; i++)` became, at `-O2`:
```
if (0 < param_1) {
    i = 0;
    do { ... } while (param_1 != i);
}
```
This is a general `for`-to-`do-while` transformation — a `do-while` only checks its
condition *after* the first iteration, which is one less jump per loop pass than a
`for`/`while` that checks *before* every iteration. GCC does this whenever it can prove
the loop runs at least once (hence the `if (0 < param_1)` guard added up front to cover
the case where it doesn't). This is unrelated to the `break` — it happens on plain loops too.

**4. `printf` → `__printf_chk`.**
Same FORTIFY_SOURCE artifact observed on Day 6. `_FORTIFY_SOURCE` swaps certain
functions (`printf`, `memcpy`, `strcpy`, etc.) for hardened `_chk` versions that add a
runtime safety check before calling the real function. This only appears when
optimization is enabled (`-O1`+), since the fortification relies on compile-time buffer
analysis that `-O0` skips. Seeing `_chk`-suffixed functions in an unknown binary is a
quick signal that it was built with optimizations and hardening on.

## Answer to the exercise's core question
An early-exit `break` on a condition that depends on runtime input (not a compile-time
constant) cannot be optimized away by itself — it always survives as a real branch. What
*can* still be eliminated is the entire loop, if nothing inside it produces an observable
effect. The `break` protects nothing on its own; the side effect (printf) is what forces
the compiler to keep the loop and, by extension, the branch inside it.

## Status
Complete. Two-part experiment: no-effect loop with break (still deleted) vs.
side-effecting loop with break (fully preserved, including a general for-to-do-while
restructuring at -O2).
