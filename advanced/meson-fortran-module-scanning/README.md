# Fix meson's Fortran module scanning to match what gfortran does

| | |
|---|---|
| **Project** | [mesonbuild/meson](https://github.com/mesonbuild/meson) (Apache-2.0) |
| **Base commit** | [`be6841aab8`](https://github.com/mesonbuild/meson/tree/be6841aab8318b61b620c6c9375137570a01b89d) |
| **Language** | Python |
| **Reference solution** | +1,196 / -74 lines across 5 files |
| **New tests** | +1,621 lines in 1 test file |

## Problem

Meson will look at all targets sources when building fortran with ninja and find all the `.mod` and `.smod` files that each source produces and requires. It then adds this information to the targets dyndep file to ensure the correct build order. This fails on many valid use cases.

For the version of gfortran installed here we need to know what module files gfortran will produce when compiling each objects source and add them to that objects dyndep entry as outputs. And we need to know what module files gfortran will read when compiling each objects source that are produced by another source in the same target and add them to that objects dyndep entry as inputs. As well as any objects from linked targets that produce module files that the source reads. Nothing else should be listed.

This needs to be true for any combination of source type and layout. Including through `include` statements and gfortrans preprocessor that is used on some source files depending on the name of the file, with the `-D` and `-I` arguments meson passes to gfortran in the order they are passed in. And when ninja is run again after a file that a source includes is changed, every object whose source includes that file has to be rebuilt. All of this needs to be determined from the source and these arguments, without executing gfortran or any other program.

Sources that gfortran will fail on are not of interest to us, and neither are sources that declare a module with the name of an intrinsic module. Nor are function like macros, predefined macros, `#elifdef` and `#elifndef` lines, Hollerith constants, headers that gfortran only finds in its default include directories, other compiler arguments or `#if` lines with a number too big for a signed 64 bit integer. It doesn't matter whether the `.smod` file that gfortran produces for a module because it uses another module is listed. You can assume that the only module files gfortran can find are its own and the ones that the targets produce.

Tests are run with `python -m pytest` from the repo root.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/mesonbuild/meson repo && cd repo
git checkout be6841aab8318b61b620c6c9375137570a01b89d
cp ../Dockerfile . && git apply ../test.patch
docker build -t meson-fortran-module-scanning .
docker run --rm meson-fortran-module-scanning ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t meson-fortran-module-scanning .
docker run --rm meson-fortran-module-scanning ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm meson-fortran-module-scanning ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
