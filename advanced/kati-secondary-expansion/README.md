# Add .SECONDEXPANSION support to ckati and rkati

| | |
|---|---|
| **Project** | [google/kati](https://github.com/google/kati) (Apache-2.0) |
| **Base commit** | [`55ea1e26f4`](https://github.com/google/kati/tree/55ea1e26f4da6d52a4ddca8f36a130dc26120525) |
| **Language** | C++ and Rust |
| **Reference solution** | +1,250 / -124 lines across 16 files |
| **New tests** | +746 lines in 1 test file |

## Problem

kati doesn't handle `.SECONDEXPANSION`. It just warns and reads a prerequisite like `$$(objs)` as a file name with a `$` in it, so makefiles that rely on it fail (the Linux kernel is one, see issue #247).

Make ckati and rkati expand prerequisite lists a second time the way GNU make 4.2.1 does. Its manual covers this under "Secondary Expansion". That make is installed as `make`, and where it doesn't match its manual, follow make. The second expansion sees a target's own target-specific variables and, for a pattern rule or a static pattern rule, the pattern-specific ones set for that pattern, but not variables the target would inherit from targets that depend on it. A target's `+=` adds to the value the target would see without it, and its `?=` does nothing if the variable already had a global value when that line was read.

In normal mode and with `--ninja`, kati should print the same output as make (compared the way `run_test.go` does), succeed or fail when make does, and leave the same files behind. Archive members, `$*` in rules without a pattern, and anything else kati already does differently from make without a second expansion are out of scope.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/google/kati repo && cd repo
git checkout 55ea1e26f4da6d52a4ddca8f36a130dc26120525
cp ../Dockerfile . && git apply ../test.patch
docker build -t kati-secondary-expansion .
docker run --rm kati-secondary-expansion ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t kati-secondary-expansion .
docker run --rm kati-secondary-expansion ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm kati-secondary-expansion ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
