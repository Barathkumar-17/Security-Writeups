## Day 5 — RE track: compiler optimization levels

Goal: compare same C source compiled at different optimization levels in Ghidra.

Source: main() with an empty-effect for loop (i = 0; i < 5; i++), then return 0.

Build commands:
  gcc -O0 prog.c -o a.out     # no optimization
  gcc -O2 prog.c -o a.out     # aggressive optimization
  (-o = output filename, lowercase; -O = optimization level, capital. Different flags.)

### -O0 result
Decompile shows loop intact:
  for (local_c = 0; local_c < 5; local_c = local_c + 1) { }
  return 0;
Loop body empty but loop still present. Compiler translated source faithfully.

### -O2 result
Decompile collapses to:
  return 0;
Entire loop removed.

Disassembly of full main() = 3 instructions:
  ENDBR64
  XOR EAX,EAX     <- sets EAX to 0 (x XOR x = 0)
  RET
EAX holds the function return value, so this is literally "return 0".

### Takeaway
Optimizer proved the loop had no observable effect (nothing printed, result unused)
and legally deleted it. Binary reflects what the program DOES, not what was WRITTEN.
At -O2, decompiled output often won't map cleanly to any source — relevant when
reversing real-world / CTF binaries.

### Ghidra friction
- Auto-analysis prompt only fires on first open of a program. If missed:
  Analysis -> Auto Analyze... (shortcut: A), accept defaults, Analyze.
- Unanalyzed binary looks like ?? in the byte column + "No Function" in decompiler.
