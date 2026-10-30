# Save the debug symbols of stripped files for GDB

| | |
|---|---|
| **Project** | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) (MIT) |
| **Base commit** | [`07333c6e94`](https://github.com/linuxdeploy/linuxdeploy/tree/07333c6e942c7d71782b66be924c5d867f9dfdfc) |
| **Language** | C++ |
| **Reference solution** | +994 / -2 lines across 6 files |
| **New tests** | +2,069 lines in 2 test files |

## Problem

linuxdeploy will remove debug symbols from elf files when it copies them into an appdir (#15). We should add a `--debug-symbols-dir <dir>` argument. Before linuxdeploy removes symbols from a file it should copy the files debug symbols to `<dir>/<files directory in the AppDir>/.debug/<name of file>.debug`. Then set the `.gnu_debuglink` in the stripped file to point at this file, replacing any existing one. If the file has a build id we should also create a relative symlink to this file at `<dir>/.build-id/<first 2 hex characters>/<rest>.debug`.

GDB 13 must then find the same debug information for the stripped file as for the file linuxdeploy copied it from, both with `<dir>` extracted over your appdir and with `debug-file-directory` set to `<dir>` and your appdir anywhere.

These debug symbols should be the ones that belong to the file: its own DWARF and symbol table. So if linuxdeploy does not copy a file because it is already at the destination the debug symbols should be the ones that belong to that file. If there is no DWARF but gdb would use a separate debug file for that file (using the files actual path, not the path of a symlink to it) then use those debug symbols. `/usr/lib/debug` should be used, or the `$LINUXDEPLOY_DEBUG_FILE_DIRECTORY` environment variable instead if it is set (colon separated list of directories).

If one of the saved debug files points to a dwz file via `.gnu_debugaltlink` we should be able to find that file the same way gdb would (path or build id). Copy that file once to `<dir>/.dwz/<its build id>.debug` and change the path in the debug file to be relative to the debug file. Files with no debug symbols and files that are not stripped by linuxdeploy, including a file `strip` fails on, will not get anything. If you run linuxdeploy again with the same appdir and dir everything should still work.

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
docker build -t linuxdeploy-split-debug-symbols .
docker run --rm linuxdeploy-split-debug-symbols ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t linuxdeploy-split-debug-symbols .
docker run --rm linuxdeploy-split-debug-symbols ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm linuxdeploy-split-debug-symbols ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
