# Make u-root's ed behave like GNU ed

| | |
|---|---|
| **Project** | [u-root/u-root](https://github.com/u-root/u-root) (BSD-3-Clause) |
| **Base commit** | [`34ab302805`](https://github.com/u-root/u-root/tree/34ab3028053499999c2d8a00a02240fc6e6dbee7) |
| **Language** | Go |
| **Reference solution** | +3,488 / -2,308 lines across 13 files |
| **New tests** | +4,371 lines in 2 test files |

## Problem

`cmds/exp/ed` should act as a drop-in replacement for GNU ed. The doc says if it doesn't match the GNU ed man page it's a bug. For most edit scripts it will either print something different to stdout or create different files or return a different exit code.

I have GNU ed version 1.19 installed as `ed`. If I run u-root's ed with the args `ed -s FILE` where FILE is a readable file in the current directory and provide it an edit script via stdin, it should print the same thing to stdout and have the same exit code and all the files in that directory should be the same as if I ran GNU ed with those same arguments. This is true for all commands, addresses and regexes that GNU ed supports. This is also true whether you provide the edit script through a pipe or a file, and a script from a file stops at its first error.

All files and scripts are text that don't contain NUL characters. I'm using the C locale. My scripts never use shell escapes (the `!` command or file names beginning with `!`). You can run GNU ed and read its manual (`/usr/share/info/ed.info.gz`) to check how it behaves, but do not use GNU ed or any other program to generate your answer.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/u-root/u-root repo && cd repo
git checkout 34ab3028053499999c2d8a00a02240fc6e6dbee7
cp ../Dockerfile . && git apply ../test.patch
docker build -t u-root-gnu-ed .
docker run --rm u-root-gnu-ed ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t u-root-gnu-ed .
docker run --rm u-root-gnu-ed ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm u-root-gnu-ed ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
