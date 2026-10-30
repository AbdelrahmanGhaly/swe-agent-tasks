# Add GNU make's load directive and loaded-object API to ckati and rkati

| | |
|---|---|
| **Project** | [google/kati](https://github.com/google/kati) (Apache-2.0) |
| **Base commit** | [`55ea1e26f4`](https://github.com/google/kati/tree/55ea1e26f4da6d52a4ddca8f36a130dc26120525) |
| **Language** | C++ and Rust |
| **Reference solution** | +3,894 / -173 lines across 35 files |
| **New tests** | +1,150 lines in 1 test file |

## Problem

kati bails out with "missing separator" when it hits a `load` directive, so any makefile that uses functions from loaded objects wont work. Allow ckati and rkati to load objects just like make 4.2.1 does: the `load` and `-load` directives, the `.LOADED` variable and the c functions in makes `gnumake.h` (installed to /usr/local/include/gnumake.h). See make's documentation for "Loading Dynamic Objects". That make is installed as `make`, and where it and the documentation disagree we go with make.

Once an object is loaded, its functions work anywhere a make function does, even in a variable set before the load or through `$(call)`. Their arguments should be split and expanded just like make would (`X := $(f a\#b)` passes `a#b` to f), and gmk_expand and gmk_eval should work in the context of the expansion that called the function.

Running kati in regular mode should give the same output as make, compared the way run_test.go does, and succeed or fail when make does. Out of scope: remaking objects the way make remakes makefiles, loading an object that registers a function named like a make or kati function, using a function in the same expansion that loads or registers it, and anything kati already does different than make without loaded objects.

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
docker build -t kati-load-plugins .
docker run --rm kati-load-plugins ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t kati-load-plugins .
docker run --rm kati-load-plugins ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm kati-load-plugins ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
