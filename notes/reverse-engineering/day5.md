# RE Day 5 — argc-based Loop: Constant Folding vs Side Effects

**Goal:** Write a loop driven by a runtime-unknown value (`argc`) and check whether it survives `-O2` optimization, building on earlier constant-folding observations (Day 5/6).

## Experiment 1: plain counter loop

```c
#include <stdio.h>
int main(int argc, char* argv[]){
    int num = 0;
    for (int i = 0; i < argc; i++)
        num++;
    printf("%d", num);
    return 0;
}
```

Compiled with:
```
gcc -O0 -o loop_O0 loop.c
gcc -O2 -o loop_O2 loop.c
```

### -O0 result
Loop present exactly as written:
```c
for (local_c = 0; local_c < param_1; local_c = local_c + 1) {
    local_10 = local_10 + 1;
}
printf("%d", local_10);
```

### -O2 result
Loop **eliminated entirely**, replaced with a single conditional assignment:
```c
iVar1 = 0;
if (-1 < param_1) {
    iVar1 = param_1;
}
printf(iVar1);
```

**Why:** the compiler proved that incrementing `num` by 1, `argc` times, always produces `num == argc` (guarded by an `argc >= 0` check since `argc` is signed). It didn't need to know the *value* of `argc` at compile time — it only needed to prove the *outcome* mathematically. This is **strength reduction / loop-to-arithmetic conversion**, a step beyond simple constant folding: the compiler can eliminate a loop even with a runtime-unknown bound, as long as the loop body is pure arithmetic with a provable closed-form result.

## Experiment 2: loop with printf inside (side effect)

```c
#include <stdio.h>
int main(int argc, char* argv[]){
    if (argc > 0) {
        for (int i = 0; i < argc; i++) {
            printf("%d", 0);
        }
    }
    return 0;
}
```

### -O2 result
Loop **survived**:
```c
if (0 < param_1) {
    iVar1 = 0;
    do {
        __printf_chk(2, &DAT_00102004, 0);
        iVar1 = iVar1 + 1;
    } while (param_1 != iVar1);
}
```

**Why:** `printf` is an observable side effect (writes to stdout). The compiler cannot replace N calls to `printf` with a single computation — it has no way to "fold" an I/O operation into pure math. So it must preserve the actual loop structure to execute the call the correct number of times.

## Key takeaway
The compiler removes a loop only when it can prove an equivalent final effect using simpler means:
- **Pure arithmetic loops** (counters, sums with no I/O) → reducible → often collapsed into a formula, even when the bound is a runtime value like `argc`.
- **Loops with side effects** (I/O, external memory writes, function calls with observable behavior) → not reducible → loop structure is preserved.

This refines earlier constant-folding notes (Day 5/6): the key variable isn't "is the value known at compile time" but "can the *effect* of the loop be proven regardless of the value." `argc` being unknown didn't stop elimination in Experiment 1 — it was the *pure arithmetic nature* of the loop body that made it foldable.

## Methodology notes (for report later)
- Initial prediction ("argc is unknown, so the loop should survive optimization") was reasonable but incomplete — corrected by direct comparison against Ghidra decompilation.
- Second experiment (adding printf) was designed specifically to isolate the variable — side effects vs pure computation — confirming the refined theory.
- Good habit: predict before viewing decompiled output, then compare prediction against evidence.
