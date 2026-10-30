# Report the decoded length of MP4 audio

| | |
|---|---|
| **Project** | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) (MIT) |
| **Base commit** | [`8e61c734be`](https://github.com/Borewit/music-metadata/tree/8e61c734beb6c3000b23e81bb3e3562de6bf0368) |
| **Language** | TypeScript |
| **Reference solution** | +1,100 / -15 lines across 6 files |
| **New tests** | +1,282 lines in 2 test files, plus 27 binary fixtures |

## Problem

music-metadata derives the duration from mp4 header info. The time includes the encoder delay that will not be played back by the player. This is noted in the mp4 as an edit list or the itunes `iTunSMPB` tag.

Have a `decodedDuration` option added to `IOptions`. This would be `false` by default so all values stay the same as they are now. If this is set to `true` and the file is mp4 (m4a m4b mov etc). Then `format.numberOfSamples` would equal the amount of samples ffmpeg 5.1 has when decoding (ffmpeg installed on my system) the first audio stream of the file. You can verify this by doing `ffmpeg -i FILE -map 0:a:0 -ac 1 -f s16le -` you should see that it outputs `numberOfSamples * 2` bytes. `format.duration` would then be `numberOfSamples / format.sampleRate`. Everything else stays the same for mp4 files. And obviously any other formats that aren't mp4.

We can get this information directly from the file. Do not execute any program. Any file that has the codecs of AAC-LC MP3 MP2 AC-3 E-AC-3 ALAC FLAC Opus PCM should work at any mp4 time scale. When reading with `parseStream` or `parseWebStream` it only needs to work if the moov comes before the audio. Fragmented files, any file that ffmpeg gives us an error on, AAC that was encoded using libfaac, tracks with more than one sample description or a QuickTime version 2 sound description, E-AC-3 frames with fewer than 6 audio blocks, multiple non-empty edits in an edit list, an edit list with only empty edits, edits with 0 duration, sample rates above 65535 Hz and an Opus track with only one packet will not be supported.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Borewit/music-metadata repo && cd repo
git checkout 8e61c734beb6c3000b23e81bb3e3562de6bf0368
cp ../Dockerfile . && git apply ../test.patch
docker build -t music-metadata-decoded-duration .
docker run --rm music-metadata-decoded-duration ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t music-metadata-decoded-duration .
docker run --rm music-metadata-decoded-duration ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm music-metadata-decoded-duration ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
