# Reverse Engineering Track — Month 1 Summary

**Repo:** `Security-Writeups` · **Path:** `reports/reverse-engineering/`
**Scope:** Month 1, RE track · Focus: how compiler optimization changes what a binary looks like in Ghidra
**Tools:** gcc (compile at `-O0` and `-O2`), Ghidra (primary)

---

## Overview

The whole month was one investigation: **compile the same C source at different optimization levels, then diff the results in Ghidra to see what the optimizer did.** The point was not to solve challenges but to build an eye for optimization artefacts — the fingerprints a real binary carries that don't map cleanly back to the source.

Baseline method, used every session:
1. Write a small C program.
2. Compile at `-O0` (no optimization) and `-O2` (aggressive).
3. Load both in Ghidra, compare decompilation and disassembly.
4. Predict first, then compare the prediction against the evidence.

---

## Theme 1 — Workflow & baseline

- Compile-both-then-diff is the standard loop. `-O0` gives a near-literal translation of the source; `-O2` rewrites, deletes, and restructures.
- Flag distinction that bit early: **`-o` (lowercase)** = output filename, **`-O` (capital)** = optimization level. Different flags.
- Ghidra friction: auto-analysis prompt only fires on the **first** open of a program. If missed → `Analysis → Auto Analyze` (shortcut `A`), accept defaults. An unanalyzed binary shows `??` in the byte column and "No Function" in the decompiler.

## Theme 2 — Dead code elimination

Source: `main()` with an empty-effect `for (i=0; i<5; i++) {}`, then `return 0;`

- `-O0`: loop present exactly as written, body empty but still there.
- `-O2`: loop gone. Whole `main` collapses to **3 instructions**:
  ```asm
  ENDBR64
  XOR EAX,EAX     ; x XOR x = 0 → zeroes the return register
  RET             ; EAX holds the return value → literally "return 0"
  ```
- **Takeaway:** absence in the binary ≠ absence in the source. The optimizer proved the loop had no observable effect and legally deleted it.

## Theme 3 — Constant folding

Source: `sum += i` for `i < 5`, then `printf("%d", sum)`. (0+1+2+3+4 = 10)

- `-O0`: real loop, stack vars (`sum`, `i`), standard prologue (`PUSH RBP` / `MOV RBP,RSP` / `SUB RSP,0x10`).
- `-O2`: 9 instructions, **no loop, no stack vars**. The answer is baked in as a literal:
  ```asm
  MOV EDX,0xa          ; 0xa = 10, the precomputed result
  ...
  CALL __printf_chk
  ```
  Decompiled: `__printf_chk(2, &DAT_00102004, 10);`
- The compiler **ran the loop itself at compile time** and kept only the answer.
- **Constant folding vs dead code elimination:** both remove the loop, for different reasons — dead code elimination throws away work that changes nothing; constant folding does the work early and keeps only the result.
- **Takeaway:** a constant with no visible source is often a folded computation. The algorithm was in the source but is not in the binary.

## Theme 4 — Strength reduction

Tested whether a runtime-unknown bound (a loop over `argc`) would force the loop to survive.

Source: `for (i=0; i<argc; i++) num++;`

- `-O2`: loop gone, replaced by a guarded assignment:
  ```c
  iVar1 = 0;
  if (-1 < param_1) { iVar1 = param_1; }   // param_1 = argc
  ```
- The compiler proved "increment `num` by 1, `argc` times" always equals `argc` — the `-1 < argc` guard is there because `argc` is signed. It never needed the *value* of `argc`, only the provable *outcome*.
- **Key reframe:** the deciding factor is **not** "is the value known at compile time" but "can the *effect* of the loop be proven regardless of the value." A pure-arithmetic loop is foldable even with a runtime bound.

## Theme 5 — Side effects as optimization barriers

The thing that actually keeps a loop alive is a **side effect** — something observable to the outside world.

- **`printf` inside the argc loop → survives `-O2`** (as a `do-while`). N calls to `printf` can't be folded into one computation, so the loop structure must stay.
- **`break` alone is not enough.** A loop with an early-exit `break` on a `scanf` value but *no* side effect was **still fully deleted** at `-O2` (only the `scanf` and stack-check remained) — same root cause as the very first dead-loop experiment.
- Add a `printf` inside that break loop and it survives; the break stays as a **real runtime branch**:
  ```c
  if (local_24 == iVar1) break;   // local_24 from scanf → unknown at compile time
  ```
  The compiler can't predict when the break fires, so it keeps it as a genuine conditional jump.
- **Takeaway:** the `break` protects nothing on its own. The side effect is what forces the compiler to keep the loop *and* the branch inside it.

## Theme 6 — Control-flow transformation

- At `-O2`, a `for` loop was restructured into a **`do-while` with a leading guard**:
  ```c
  if (0 < param_1) {           // guard for the zero-iteration case
      i = 0;
      do { ... } while (param_1 != i);
  }
  ```
- **Why:** a `do-while` checks its condition *after* the first iteration → one fewer jump per pass than a `for`/`while` that checks *before* every iteration. GCC does this whenever it can prove the loop runs at least once, adding the up-front guard to cover the case where it doesn't.
- This is a **general** transformation — independent of the `break`; it happens on plain loops too.
- **Takeaway:** loop *shape* in the disassembly won't match the source. Recognize the pattern, don't expect a 1:1 mapping.

## Theme 7 — Optimization & hardening fingerprints

- **`__printf_chk`** — replaces `printf` under `_FORTIFY_SOURCE`. The leading `2` argument is the fortify level. `_chk` variants add a runtime buffer check. They only appear at **`-O1`+**, because fortification relies on compile-time buffer analysis that `-O0` skips → seeing `_chk` = built with optimization *and* hardening.
- **`ENDBR64`** — control-flow-integrity (CET) landing pad at function entry.
- **`XOR EAX,EAX` / `RET`** — the compact idiom for "return 0".
- **Takeaway:** several symbols in a disassembly come from the compiler and hardening flags, not the programmer.

---

## Key principles carried forward

- **Diff-driven learning:** the `-O0` vs `-O2` comparison is where every insight came from.
- **Predict, then verify:** the first prediction ("argc is unknown, so the loop should survive") was reasonable but wrong — corrected by direct comparison against Ghidra.
- **Effect, not value:** a loop is removable when its *effect* is provable, whether or not its inputs are known at compile time.
- **Side effects are the anchor:** if code has no observable effect, assume the optimizer may erase it — a lone `break` won't save it.
- **Don't trust the shape:** loops become formulas, `for` becomes `do-while`, empty functions become three instructions.
- **Know the compiler's signature:** `__printf_chk`, `ENDBR64`, and the zero-return idiom are the compiler talking, not the author.

## Next up (Month 2, RE track)

- Begin **x86-64 assembly basics** — read the instructions directly instead of leaning on the decompiler.
- Carry the optimization-artefact awareness into hand-reading disassembly.
