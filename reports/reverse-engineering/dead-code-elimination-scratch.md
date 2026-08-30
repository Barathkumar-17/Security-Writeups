# RE Session 2 - Dead code elimination vs dynamic input

Date: 30-08
Repo path: notes/reverse-engineering/

## Goal

Session 1 showed Ghidra's decompiler deleting an if-branch when `x` was hardcoded.
Test: does the branch survive if `x` comes from user input instead?

## Setup

Source: `num_greater_10.c`
Change: `int x = 5;` -> `scanf("%d", &x);`

```
gcc -o test num_greater_10.c
```

Imported into Ghidra project "testing".
Auto-analysis did NOT prompt on open - ran manually via `Analysis -> Auto Analyze 'test'`, default options.

## Result

**Branch survived.** Both `if` and `else` present in the decompiler output.

Confirms: the compiler only deletes a branch if it can *prove* which way it goes.
With `scanf`, `x` is unknown at compile time, so both paths must be kept.

## Decompiler output (main)

```c
int local_14;              // this is x
long local_10;             // stack canary

local_10 = *(long *)(in_FS_OFFSET + 0x28);
printf("Enter number:");
__isoc23_scanf(&DAT_00102012, &local_14);
if (local_14 < 0xb) {
    puts("small");
}
else {
    puts("big");
}
if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
    __stack_chk_fail();
}
return 0;
```

## Compiler artefacts noticed

**1. Variable names gone**
`x` -> `local_14`, named from its stack offset. Ghidra has no access to source names.
But the type `int` was correctly inferred from how `scanf` used it.

**2. Comparison flipped**
Source: `x > 10`. Decompiled: `local_14 < 0xb` (11) with the branches swapped.
Same logic, different shape - the compiler picked whichever comparison was cheaper in assembly.

=> Decompiled code is **logically** equivalent to source, not textually identical.
Don't expect a 1:1 match when reading decompiler output.

**3. printf became puts**
`printf("big\n")` has no format specifiers, so gcc substituted the simpler `puts`,
and dropped the `\n` (puts appends its own newline).

**4. Stack canary present**

```c
local_10 = *(long *)(in_FS_OFFSET + 0x28);              // set on entry
if (local_10 != *(long *)(in_FS_OFFSET + 0x28))         // checked before return
    __stack_chk_fail();
```

A random value is placed on the stack at function entry and re-checked before return.
If it changed, something overwrote the stack -> abort.
gcc adds this automatically. It's anti-buffer-overflow protection.

Relevant later: Month 5 (buffer overflows) - this is the thing to get around.

**5. `__isoc23_scanf`**
Just glibc's internal versioned name for `scanf`. Not a different function.

## Import summary notes

- Ghidra auto-detected the architecture on its own: `x86:LE:64` (64-bit, little endian)
- Binary leaked the source filename: `ELF Source File [2]: num_greater_10.c`
  Compiled programs carry more metadata than expected.
- `[libc.so.6] -> not found in project` - harmless. Just means the C standard library
  wasn't imported alongside. External calls still resolve by name.

## Next session

- Compare this decompilation side by side with the session 1 binary (hardcoded `x`)
- Try compiling with `-O2` and see what else disappears
