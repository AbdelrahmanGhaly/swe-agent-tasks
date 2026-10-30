# Resolve ELF dependencies natively instead of running ldd

| | |
|---|---|
| **Project** | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) (MIT) |
| **Base commit** | [`07333c6e94`](https://github.com/linuxdeploy/linuxdeploy/tree/07333c6e942c7d71782b66be924c5d867f9dfdfc) |
| **Language** | C++ |
| **Reference solution** | +1,193 / -63 lines across 4 files |
| **New tests** | +1,988 lines in 2 test files |

## Problem

ElfFile::traceDynamicDependencies uses ldd to find what libs the file depends on by executing the file and parsing it's output (#8). We cant do this if ldd cant be executed. Also this will execute the file through the dynamic loader. We want to be able to find this out ourselves from the ELF file(s) and /etc/ld.so.cache as well as environment variables. No executing ldd or the dynamic loader or anything else. Should also not try to load the libs into our process.

Given a dynamic elf file that is made for the machine that linuxdeploy is being run on we expect it to return the same libs that glibc 2.36 ldd would return (we have this installed). For the canonical path of that file if we give it the same LD_LIBRARY_PATH and cwd we call with. Should return the same libs in the same order except for linux-vdso.so.1 and the dynamic loader. Paths should be absolute and lead to the same file that ldd returns they dont have to be the same path.

If ldd would say a lib could not be found we should throw a DependencyNotFoundError with the first lib that is not found that does not fit our exclude patterns. Same order that ldd returns them and name that ldd returns. We should continue to ignore any libs that are not found and fit our exclude patterns.

Can ignore glibc-hwcaps and legacy hwcap dirs. $LIB and $PLATFORM LD_PRELOAD and /etc/ld.so.preload. If a lib has DT_RPATH and DT_RUNPATH set. Anything ldd fails on.

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
docker build -t linuxdeploy-native-dependency-resolution .
docker run --rm linuxdeploy-native-dependency-resolution ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t linuxdeploy-native-dependency-resolution .
docker run --rm linuxdeploy-native-dependency-resolution ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm linuxdeploy-native-dependency-resolution ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
