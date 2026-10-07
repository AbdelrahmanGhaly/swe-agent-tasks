# Lay out vstruct C structures the way GCC does

| | |
|---|---|
| **Project** | [vivisect/vivisect](https://github.com/vivisect/vivisect) (Apache-2.0) |
| **Base commit** | [`66c93f0c1c`](https://github.com/vivisect/vivisect/tree/66c93f0c1caa949434b9b1ba8bb64c8bad0eb3d9) |
| **Language** | Python |
| **Reference solution** | +887 / -119 lines across 5 files |
| **New tests** | 183 test functions, +3,380 lines in 2 test files |

## Problem

Make vstruct.cparse structures (ctorFromCSource, vsFromCSource, CVStruct) match GCC: psize 4 is i386 (4-byte long and pointers, 8-byte types 4-aligned inside structures, bit-fields included), psize 8 is x86-64.

Members are aligned; aggregates align and round their size to their strictest member. Union members share the union's first bytes: a write, even through a nested member, changes only its bytes; the last write wins. New structures have zero padding; parsed ones re-emit their bytes.

A bit-field takes the next free bits unless that spans more of its type's alignment units than its type fills, then starts at the next unit; :0 rounds up to it, and unnamed bit-fields don't raise alignment. Bits fill from each byte's low end, low bits first, or high end, high bits first, under bigend. Bit-fields read and assign by name; signed ones (plain int and char too) sign-extend.

Accept any integer spelling, qualifiers, _Bool, float, double, typedefs, 4-byte enums, any pointers. Tags and enumerators work once defined, even inside another structure (adding no member) or pointer, array or function declaration; ctorFromCSource returns the last file-scope structure or union, named (vsGetTypeName) by tag, else typedef. Array sizes are integer constant expressions (octal too); sizeof takes expressions and file-scope objects, typed as C does: literal suffixes, integer promotion, usual arithmetic conversions (double over float over integers), elements (`*a`, `a[i]`) and members, int character and enumeration constants. Plain char arrays stay strings, others (signed and unsigned char too) index by element; an unsized last array is empty.

len, non-bit-field vsGetOffset paths, vsParse and vsEmit (fast or not), bigend byte order and makeStructure's pointer xrefs follow this layout. Workspace user structures use the workspace's pointer size and byte order (default 4-byte little-endian), even after reloading. Hand-written VStructs and VBitField are unchanged.

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
docker build -t vivisect-c-struct-layout .
docker run --rm vivisect-c-struct-layout ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t vivisect-c-struct-layout .
docker run --rm vivisect-c-struct-layout ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm vivisect-c-struct-layout ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
