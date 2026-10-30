# Read and write rpaths natively instead of calling patchelf

| | |
|---|---|
| **Project** | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) (MIT) |
| **Base commit** | [`07333c6e94`](https://github.com/linuxdeploy/linuxdeploy/tree/07333c6e942c7d71782b66be924c5d867f9dfdfc) |
| **Language** | C++ |
| **Reference solution** | +1,272 / -92 lines across 7 files |
| **New tests** | +2,462 lines in 2 test files |

## Problem

ElfFile::getRPath and ElfFile::setRPath both execute patchelf. patchelf has corrupted files for us in the past (#204). We should be reading and modifying the elf file directly. Not executing patchelf or any other external process.

getRPath should return the DT_RUNPATH string for the file, or the DT_RPATH string if it doesn't have one, or an empty string if it has neither.

setRPath(value) should allow glibc to load the file and use value as its search path. If there is a DT_RUNPATH or DT_RPATH it should retain that type and just update the string. If it has neither it should add a DT_RUNPATH. It should also work if you call setRPath on a file that you have already called setRPath on.

The rest of the file should remain unchanged: it should still load and execute the same way, with all other dynamic entries, names and section contents the same. It also shouldn't bloat the file. A program or library that reserves a lot of zero-initialized memory (a 64 MiB static array for example) shouldn't grow by anywhere near that much. These files will be stripped later (#323) so running strip on the resulting file should produce a file that can still load and execute the same way.

This should work on executable files and shared objects compiled for the architecture that linuxdeploy is being run on, including files with no section headers.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/linuxdeploy/linuxdeploy repo && cd repo
git checkout 07333c6e942c7d71782b66be924c5d867f9dfdfc
cp ../Dockerfile . && git apply ../test.patch
docker build -t linuxdeploy-native-rpath-editing .
docker run --rm linuxdeploy-native-rpath-editing ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t linuxdeploy-native-rpath-editing .
docker run --rm linuxdeploy-native-rpath-editing ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm linuxdeploy-native-rpath-editing ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
