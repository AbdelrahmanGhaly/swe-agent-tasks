# Fix flex input buffers in reflex --yy scanners

| | |
|---|---|
| **Project** | [Genivia/RE-flex](https://github.com/Genivia/RE-flex) (BSD-3-Clause) |
| **Base commit** | [`c06bb16a49`](https://github.com/Genivia/RE-flex/tree/c06bb16a49e79f4f123f4b31b6f53e5bfa884624) |
| **Language** | C++ |
| **Reference solution** | +391 / -68 lines across 9 files |
| **New tests** | +3,871 lines in 239 test files |

## Problem

`reflex --yy` is supposed to maximize compatibility with flex, but C programs that move their input between files and strings with flex's buffer functions break with it. The global `yylex()` resets the scanner to `yyin` on every call, `yy_scan_string()` and `yy_scan_bytes()` return NULL instead of the new buffer, and a buffer created before the first `yylex()` call crashes the program.

Built from the same spec and linked with the same `main()`, the `reflex --yy` scanner should give the same output as the flex 2.6.4 scanner (flex and gcc are installed). This covers the `YY_BUFFER_STATE` functions of flex's multiple input buffers together with `yyrestart()`, `yyin`, `yywrap()` and `YY_USER_INIT`. `yylineno` is left out of this, it still counts lines per buffer as it does today.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Genivia/RE-flex repo && cd repo
git checkout c06bb16a49e79f4f123f4b31b6f53e5bfa884624
cp ../Dockerfile . && git apply ../test.patch
docker build -t re-flex-yy-input-buffers .
docker run --rm re-flex-yy-input-buffers ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t re-flex-yy-input-buffers .
docker run --rm re-flex-yy-input-buffers ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm re-flex-yy-input-buffers ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
