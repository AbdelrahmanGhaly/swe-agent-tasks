# Make the conditional C scanner read the headers GCC reads

| | |
|---|---|
| **Project** | [SCons/scons](https://github.com/SCons/scons) (MIT) |
| **Base commit** | [`37add80b36`](https://github.com/SCons/scons/tree/37add80b36b25f4ca13811de819a12c1c1648ec0) |
| **Language** | Python |
| **Reference solution** | +1,739 / -418 lines across 2 files |
| **New tests** | +2,162 lines in 1 test file |

## Problem

`SCons.Scanner.C.CConditionalScanner()` tries to mimic what the C preprocessor does to find header files in `SCons/cpp.py`. It does not do a good enough job to be trusted. It traces through macros one depth at a time with limited patterns, it can evaluate some `#if` statements using python code, it assumes `#include_next` is just another `#include`. It will not read a header file twice and it will modify its search path after finding a file in a `CPPPATH` path. It will fail to detect some header files and detect others that are not actually included.

Improve the preprocessor emulation to correctly find all headers for C source files that gcc version 12 installed on this system will pre-process with no errors and no missing headers. The expected result is the list of files, other than the source itself, that gcc reads when running `gcc -E -undef -nostdinc -x c` followed by a `-I` for each path in `CPPPATH` and any `-D`s added by SCons to it's command line for `CPPDEFINES` as passed by the shell. Exclude support for `__COUNTER__`, `_Pragma`, `__has_attribute`, `__has_c_attribute`, `__has_builtin`, gcc assert and built-in macros that evaluate to a file name, date or time. No external programs may be used to determine these header files.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/SCons/scons repo && cd repo
git checkout 37add80b36b25f4ca13811de819a12c1c1648ec0
cp ../Dockerfile . && git apply ../test.patch
docker build -t scons-gcc-include-scanning .
docker run --rm scons-gcc-include-scanning ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t scons-gcc-include-scanning .
docker run --rm scons-gcc-include-scanning ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm scons-gcc-include-scanning ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
