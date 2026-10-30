# Implement .ONESHELL and .SHELLFLAGS in ckati and rkati

| | |
|---|---|
| **Project** | [google/kati](https://github.com/google/kati) (Apache-2.0) |
| **Base commit** | [`55ea1e26f4`](https://github.com/google/kati/tree/55ea1e26f4da6d52a4ddca8f36a130dc26120525) |
| **Language** | C++ and Rust |
| **Reference solution** | +829 / -136 lines across 20 files |
| **New tests** | +730 lines in 1 test file |

## Problem

kati warns that it doesn't support `.ONESHELL` and runs each recipe line in its own shell. It never defines or reads `.SHELLFLAGS` either: recipes and `$(shell ...)` always get `-c`, or `-ec` after `.POSIX`. A recipe also always runs with the global `SHELL`, even when its target sets its own.

Make ckati and rkati support both the way GNU make 4.2.1 does. Its manual covers them under "Using One Shell" and "Choosing the Shell". That make is installed as `make`, and where it doesn't match its manual, follow make. As in make, a target's own `SHELL` and `.SHELLFLAGS`, target-specific and pattern-specific values included, apply to its recipe and to `$(shell ...)` calls in it.

In normal mode and with `--ninja`, kati should print the same output as make (compared the way `run_test.go` does), succeed or fail when make does, and leave the same files behind. Differences kati already has from make without these features are out of scope.

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
docker build -t kati-oneshell-shellflags .
docker run --rm kati-oneshell-shellflags ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t kati-oneshell-shellflags .
docker run --rm kati-oneshell-shellflags ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm kati-oneshell-shellflags ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
