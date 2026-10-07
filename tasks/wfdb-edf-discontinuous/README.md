# Read discontinuous EDF+ files into variable-layout records

| | |
|---|---|
| **Project** | [MIT-LCP/wfdb-python](https://github.com/MIT-LCP/wfdb-python) (MIT) |
| **Base commit** | [`f627b5ff9d`](https://github.com/MIT-LCP/wfdb-python/tree/f627b5ff9dcfdb11d4c3150f9c1ebb47cfc3909d) |
| **Language** | Python |
| **Reference solution** | +517 / -246 lines across 3 files |
| **New tests** | 68 test functions, +1,332 lines in 2 test files |

## Problem

Make `read_edf` and `rdedfann` handle EDF+ files, even interrupted (EDF+D) ones. EDF+ signals labelled `EDF Annotations` are left out of records. Any EDF file's frame rate is the greatest common divisor of the other signals' samples per data record over its duration; `samps_per_frame` is each signal's multiple of it. When every signal has one sample per frame, records and segments keep digital samples in `d_signal` beside `p_signal`. Otherwise they keep `e_d_signal` and `e_p_signal`, and `p_signal` averages each frame. Physical values follow each signal's ranges as today; `init_value` holds each signal's first stored sample.

The first list of each data record's first annotation signal is the record's onset, not an annotation. Times are exact decimals as written. Frame 0 is the first record's onset, the base date and time is the header start plus that onset, and any time maps to its nearest frame, an exact half going to the later frame: at 100 per second, 0.145 s after frame 0 is frame 15 and 0.015 s before is frame -1; at 4/3 per second, 0.375 s after is frame 1.

EDF and EDF+C files give one `Record` of back-to-back data records. An EDF+D file always gives a variable-layout `MultiRecord`: a zero-length layout segment `<name>_layout` holding the signal specifications, then `<name>_0001` onward, one per run of records each starting on the frame where the last ends, and a null `~` segment (None) per gap. A record whose exact onset precedes the previous one's end raises, even if rounding hides it. With `header_only`, every EDF, EDF+C or EDF+D file with non-annotation signals gives its header dictionary, matching the full read.

`MultiRecord.wrsamp` writes only the header of a layout segment, whose signals have file name `~` and format `0`. Every None segment, whatever its name, is written as a `~` null segment, including those `rdrecord` leaves without selected signals. A record from `rdrecord(..., m2s=False, physical=False)` or `read_edf` can be written, expanded if its samples are, and read back with identical segment lengths, signal specifications, samples and single-record conversion.

`rdedfann` reads every annotation signal. Each other list has a non-negative onset, an optional duration and one annotation per non-empty UTF-8 text, spaces included, kept verbatim. With `info_only` it returns, in file order, the `onset_time` (HH:MM:SS.fff rounded to the millisecond, carrying into minutes and hours), `sample_num` (its frame, or None without other signals), `comment` and `duration` (None when absent) lists.

Otherwise a positive duration gives a `(text` note at the onset and a `text)` note at its end, any other annotation one note, ordered by time and at equal times instantaneous notes, openings, then closings, each group in file order. The notes (symbol `"`, at the frame rate) are returned as an `Annotation` with `record_only` or written to `<name>.atr` in the working directory, where only ASCII text must stay exact. With no notes, `record_only` gives an empty `Annotation` and writing raises, as does writing a note before frame 0 or either mode without other signals.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/MIT-LCP/wfdb-python repo && cd repo
git checkout f627b5ff9dcfdb11d4c3150f9c1ebb47cfc3909d
cp ../Dockerfile . && git apply ../test.patch
docker build -t wfdb-edf-discontinuous .
docker run --rm wfdb-edf-discontinuous ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t wfdb-edf-discontinuous .
docker run --rm wfdb-edf-discontinuous ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm wfdb-edf-discontinuous ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
