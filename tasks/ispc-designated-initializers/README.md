# Add C99 designated initializers to ispc initializer lists

| | |
|---|---|
| **Project** | [ispc/ispc](https://github.com/ispc/ispc) (BSD-3-Clause) |
| **Base commit** | [`e99a37840c`](https://github.com/ispc/ispc/tree/e99a37840cd7d83c84e56e97a03eab6049b59fe7) |
| **Language** | C++ |
| **Reference solution** | +538 / -2 lines across 7 files |
| **New tests** | +3,081 lines in 48 test files |

## Problem

Initializer lists in ispc are positional only. Add C99 designators: `.name = value` picks a struct member and `[index] = value` picks an array or short vector element. A vector element can also be picked by the one-letter member names ispc already accepts for vectors (`x`, `y`, `z`, `w`, or `r`, `g`, `b`, `a`, or `u`, `v`). Designators chain, as in `.pos.y = 1` or `[2].name = x`, and designated and plain items mix freely. Designated lists must work everywhere ispc accepts an initializer list today, including `soa<N>` structs and arrays of them, under the same typing, storage and constness rules, and positional initializers and their diagnostics must not change.

Every initializer ispc accepts must produce the values a C99 compiler gives for it. An item without a designator initializes what comes right after the previous item's target at the same depth, moving outward once that part is used up, so in `{ .pos.x = 1, 2 }` the 2 goes to `pos.y`. If an element is initialized twice the later item wins: a later value or brace list replaces the whole element, while a designator reaching inside an element that an earlier brace list set only changes that part. Anything not initialized is zero, and pointers are null. When an item overwrites something an earlier item explicitly initialized, report a warning at the later item, as clang and gcc do.

An array dimension declared without a size gets one past the highest element initialized. ispc still has no brace elision: anywhere inside an initializer that uses designators, an item that lands on a struct, array or short vector must be a brace list, or it is a compile error, even when the item is a value of that type. In multi-dimensional arrays with no sizes, every sub-list that a later item does not replace must still imply the same length, and a mismatch is still an error.

Array indices are integer constant expressions, including enum values, local and global const variables with a constant initializer (braced or not) and template integer parameters; an unsigned index may be rejected. Report a compile error, never a crash, for: a member name the struct does not have (the message must name it); an index that is negative or past the end of a sized array or vector (the message must include the index); a non-constant, boolean or floating-point index; a vector member name that is not one of those letters or is past the vector's end; `.name` on something that is neither a struct nor a vector, or `[index]` on something that is not an array or vector; designators for any other type, such as `varying int`; and an item that would land past the end of a sized array, vector or struct.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/ispc/ispc repo && cd repo
git checkout e99a37840cd7d83c84e56e97a03eab6049b59fe7
cp ../Dockerfile . && git apply ../test.patch
docker build -t ispc-designated-initializers .
docker run --rm ispc-designated-initializers ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t ispc-designated-initializers .
docker run --rm ispc-designated-initializers ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm ispc-designated-initializers ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
