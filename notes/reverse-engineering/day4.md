# RE — Constant Folding at -O2

**Date:** 01-09
**Tool:** Ghidra
**Builds on:** Day 5 (`-O0` vs `-O2`, dead code elimination)

---

## Setup

```c
#include <stdio.h>

int main(void) {
    int sum = 0;
    for (int i = 0; i < 5; i++) {
        sum += i;
    }
    printf("%d\n", sum);
    return 0;
}
```

```bash
gcc -O0 -o out1 sum.c
gcc -O2 -o out2 sum.c
```

Question being tested: yesterday `-O2` deleted a loop that did nothing. What
happens when the loop actually produces a result?

---

## `-O0` result (out1)

Decompiler output is close to the original source:

```c
local_10 = 0;
for (local_c = 0; local_c < 5; local_c = local_c + 1) {
    local_10 = local_10 + local_c;
}
printf("%d", (ulong)local_10);
```

- `local_10` = `sum`, `local_c` = `i`
- Both are stack variables (`Stack[-0x10]`, `Stack[-0xc]`)
- Standard prologue present: `PUSH RBP` / `MOV RBP,RSP` / `SUB RSP,0x10`
- The loop exists as real code — counter, comparison, backward jump

---

## `-O2` result (out2)

`main` is nine instructions. No loop.

```asm
ENDBR64
SUB   RSP,0x8
MOV   EDX,0xa          ; <-- 10, the precomputed answer
MOV   EDI,0x2
XOR   EAX,EAX
LEA   RSI,[DAT_00102004]
CALL  __printf_chk
XOR   EAX,EAX
ADD   RSP,0x8
RET
```

Decompiled:

```c
__printf_chk(2, &DAT_00102004, 10);
```

`0xa` = 10 decimal. 0+1+2+3+4 = 10. The compiler ran the loop itself at compile
time and hardcoded the result.

No stack variables — nothing needed storing.

---

## Concept: constant folding vs dead code elimination

| | Day 5 | Day 6 |
|---|---|---|
| Did the loop have an effect? | No | Yes |
| What the optimizer did | Removed it entirely | Computed the result at compile time, removed the loop |
| Name | Dead code elimination | Constant folding |
| What survives in the binary | Nothing | The answer, as a literal |

In my own words: dead code elimination throws away work that changes nothing.
Constant folding does the work early — at compile time — and keeps only the
answer. Either way the loop is missing from the binary, but for different
reasons.

**Why this matters for RE:** a constant appearing with no visible source is
often a folded computation. The algorithm that produced it was in the source
but is not in the binary. Optimized code cannot always be traced back to the
original logic.

---

## Side observation: `printf` became `__printf_chk`

Not something I asked for. `-O2` enables `_FORTIFY_SOURCE`, which swaps certain
libc calls for hardened variants that perform buffer checks. The leading `2`
argument is the fortify level.

`_chk` suffixes in a binary are a compilation fingerprint — they say something
about how the target was built.

---

## Not yet done

Stretch version: take the loop bound from `argc` so the compiler cannot know
the answer ahead of time. Recompile at `-O2` and check whether the loop
reappears. Expectation: it should, because the input is only known at runtime.
