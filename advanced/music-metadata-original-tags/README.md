# Read the original file's tags kept in WavPack and FLAC

| | |
|---|---|
| **Project** | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) (MIT) |
| **Base commit** | [`ef859cbfce`](https://github.com/Borewit/music-metadata/tree/ef859cbfce4f1b35040692c0aadcc43307267a16) |
| **Language** | TypeScript |
| **Reference solution** | +696 / -26 lines across 10 files |
| **New tests** | +620 lines in 1 test file, plus 58 binary fixtures |

## Problem

wavpack will keep the pre/post data of the file so that you can use wvunpack to extract a bit perfect copy of the original. `flac --keep-foreign-metadata` allows this to happen for wav/aiff, though `flac -d` writes some of the headers itself so its copy isn't always bit perfect. Since music metadata doesn't read these areas we lose any tags that may exist on the original. (INFO chunk & id3 chunk if wav or ID3 tag on a DSF).

I propose we add `includeOriginalTags` to `IOptions`. It would default to `false` and nothing would change. If set to `true` we would want `metadata.native` to contain all tag types and tags that music metadata returns for the original once extracted and parsed with the same options. After the current files tags. `common` would be mapped from the files tags and then the originals just as music metadata would map the originals. `format.tagTypes` would contain the additional tag types after the current files tag types. Nothing else would change in `format`.

`metadata.original` would also hold everything music metadata returns for the extracted original with the same options. If music metadata can't read the original, or it is W64/RF64, nothing from it would be added, not even `metadata.original`. Same if the parts of the original kept in the file add up to more than `maxMemoryAllocation`.

This should be possible for all parse functions and any original that music metadata supports parsing. Originals written to a pipe, or whose audio doesn't end on a whole frame, don't need to be supported. Currently have wavpack 5.6 flac 1.4.2 and ffmpeg 5.1 installed.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Borewit/music-metadata repo && cd repo
git checkout ef859cbfce4f1b35040692c0aadcc43307267a16
cp ../Dockerfile . && git apply ../test.patch
docker build -t music-metadata-original-tags .
docker run --rm music-metadata-original-tags ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t music-metadata-original-tags .
docker run --rm music-metadata-original-tags ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm music-metadata-original-tags ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
