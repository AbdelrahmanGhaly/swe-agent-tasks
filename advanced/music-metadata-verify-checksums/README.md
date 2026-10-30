# Verify the checksums stored in audio files

| | |
|---|---|
| **Project** | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) (MIT) |
| **Base commit** | [`96f0828a60`](https://github.com/Borewit/music-metadata/tree/96f0828a60f4d6a8587986f334fce03daca92b63) |
| **Language** | TypeScript |
| **Reference solution** | +793 / -17 lines across 12 files |
| **New tests** | +1,193 lines in 2 test files, plus 10 binary fixtures |

## Problem

Music-metadata parses many formats that store checksums on there data. I would like a `verifyChecksums` option added to `IOptions`. Default to `false` which will not affect anything. If set to `true` then `metadata.quality.checksums` will be an object with a `{ checked, failed }` entry for each type of checksum that was checked in the file, failed being how many did not match the data. All parse functions should support this and it should not affect the output of `format`, `common` or `native` if any combination of options are used.

Here are the types and all checksums of that type in a file will be counted. `mpegFrame` will be the CRC of a frame that contains one in an MPEG audio file (MPEG 2.5 included), as defined in ISO 11172-3 and 13818-3. `lameTag` and `lameMusic` will be the CRCs found in the Info tag that is written by LAME version 3.90 and above. One covering the Info frame to the start of the tag CRC and the other covering the bytes after the Info frame to the length of the music in the tag. `ebml` will be a CRC-32 element that is found at the beginning of an Element directly under the Segment in a Matroska or WebM file.

Checksums will only be checked if the file contains all bytes it should cover. Free format MPEG, ADTS, junk data before/between frames, APEv2 and Lyrics3 tags after MPEG audio and unknown size EBML Elements will not be supported. I have ffmpeg 5.1 (`-err_detect crccheck`), lame and twolame installed.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Borewit/music-metadata repo && cd repo
git checkout 96f0828a60f4d6a8587986f334fce03daca92b63
cp ../Dockerfile . && git apply ../test.patch
docker build -t music-metadata-verify-checksums .
docker run --rm music-metadata-verify-checksums ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t music-metadata-verify-checksums .
docker run --rm music-metadata-verify-checksums ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm music-metadata-verify-checksums ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
