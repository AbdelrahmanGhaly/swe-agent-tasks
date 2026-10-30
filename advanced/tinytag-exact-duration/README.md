# Add an exact duration mode for MPEG and FLAC files

| | |
|---|---|
| **Project** | [tinytag/tinytag](https://github.com/tinytag/tinytag) (MIT) |
| **Base commit** | [`ea5be2b7cf`](https://github.com/tinytag/tinytag/tree/ea5be2b7cf9ba51452bf01e42c6dfae04357e94a) |
| **Language** | Python |
| **Reference solution** | +379 / -5 lines across 2 files |
| **New tests** | +930 lines in 1 test file |

## Problem

tinytag can get the MPEG `duration` from a Xing or VBRI header or filesize. Its duration also includes the encoder delay and padding if there is a LAME header (both LAME and ffmpeg create one). FLAC has the count in STREAMINFO.

There should be a keyword arg `exact_duration` for `TinyTag.get()` with default of `False` which would return what we currently have. If set to `True` we want `duration = n / samplerate` for MPEG audio and native FLAC files. Where `n` is how many samples per channel ffmpeg 5.1 can decode from the file (running `ffmpeg -i FILE -f s16le -ac 1 -` will output `2 * n` bytes) or None if `n == 0`. Everything else should be left as it is.

Find a way to determine `n` from the file itself no external programs. Should work on files that are cut off at the end, possibly mid frame. Files that ffmpeg can't read, MPEG streams that change sample rate and any bytes between frames are out of scope.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/tinytag/tinytag repo && cd repo
git checkout ea5be2b7cf9ba51452bf01e42c6dfae04357e94a
cp ../Dockerfile . && git apply ../test.patch
docker build -t tinytag-exact-duration .
docker run --rm tinytag-exact-duration ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t tinytag-exact-duration .
docker run --rm tinytag-exact-duration ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm tinytag-exact-duration ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
