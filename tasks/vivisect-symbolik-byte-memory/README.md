# Make symbolik memory and register writes byte-accurate

| | |
|---|---|
| **Project** | [vivisect/vivisect](https://github.com/vivisect/vivisect) (Apache-2.0) |
| **Base commit** | [`66c93f0c1c`](https://github.com/vivisect/vivisect/tree/66c93f0c1caa949434b9b1ba8bb64c8bad0eb3d9) |
| **Language** | Python |
| **Reference solution** | +403 / -54 lines across 8 files |
| **New tests** | 144 test functions, +1,240 lines in 4 test files |

## Problem

Keep the CLI's symboliks command listing each path's memory writes and translator tests passing unchanged.

writeSymMemory(symaddr, symval, vals=None, symsize=None) and WriteMemory change only the bytes of their size (int or symbolik, default symval's width), cutting or zero extending the value, low byte first. Every emulator solves addresses as given with each call's vals, never updating them; equal solutions are the same memory. readSymMemory builds the read-size value from each byte's latest write; a read matching one write's start and size returns that value, fit to size. Unwritten bytes come from the whole read's plain access, Mem(symaddr, symsize), even on imports. If none of the read's bytes were written, it acts as today (import var or None). Reads above 32 bytes are unsupported.

Non-symbolik values (int, str), even with a symsize, and unknown sizes act as today: readable only at their exact address, by any read there, until a later sized write covers that address, and an unknown-size read sees only the write starting there. Only ints and discrete symboliks are known sizes (never via vals); a variable-size Mem is sized only by a symsize.

Mem.solve(emu=..., vals=...) solves that emulator's readSymMemory result with those vals, never a cached solve, even values with variables, following pointers in its address; a None result solves normally. Plain Mem solves fit their size. Snapshots are copied when taken and set. TrackingSymbolikEmulator gets this memory: a read with none of its bytes written acts and tracks as today; otherwise unwritten mapped bytes come from the workspace, even in an import slot, others from the whole read's Mem, and that value is tracked.

In i386 and amd64 translators, writing al, ah or ax changes only those bits; amd64 32-bit writes zero the upper half. Memory operand addresses use the full base register. movs, stos, lods and cmps use 1, 2, 4 or 8 bytes by instruction, operand size prefix or REX.W, stepping index registers by that size per the direction flag; stos stores and lods loads the matching accumulator part. Operand size prefixed push, pop and pushf move 2 bytes.

With rep, the emulator runs movs, stos or lods as many times as the count register holds when it reaches the instruction, each seeing the last; the count ends at 0, and returned effects list every repetition plus that reset. An unknown count runs once, unchanged.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/vivisect/vivisect repo && cd repo
git checkout 66c93f0c1caa949434b9b1ba8bb64c8bad0eb3d9
cp ../Dockerfile . && git apply ../test.patch
docker build -t vivisect-symbolik-byte-memory .
docker run --rm vivisect-symbolik-byte-memory ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t vivisect-symbolik-byte-memory .
docker run --rm vivisect-symbolik-byte-memory ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm vivisect-symbolik-byte-memory ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
