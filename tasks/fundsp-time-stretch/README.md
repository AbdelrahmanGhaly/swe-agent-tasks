# Add time stretching to Wave and a variable-speed player

| | |
|---|---|
| **Project** | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) (Apache-2.0) |
| **Base commit** | [`559584043a`](https://github.com/SamiPerttu/fundsp/tree/559584043aa3369e92d5aad12a5e3b003cfacfd5) |
| **Language** | Rust |
| **Reference solution** | +470 / -0 lines across 7 files |
| **New tests** | 57 test functions, +1,464 lines in 1 test file |

## Problem

FunDSP only lets you play a `Wave` at it's original speed. Time stretching is needed. New length same pitch. `Wave::stretch(factor)` will produce a wave `round(len * factor)` long. Same sample rate and channels. A factor of 1 gives the same samples. Since all channels share one time, a sound that ends on multiple channels will end at the same time in the stretched wave.

Add `playwave_stretch(&wave, channel, loop_point)` to all 3 preludes. Its input is the speed. When speed is 1 and sample rates match you get the same samples. If speed is 0 you stay at that point but the sound around you keeps playing at it's original pitch. Negative speed plays the sound backwards from where you are, including the wave form.

`position()` is in the waves samples. It starts at 0 and moves by speed * waves sample rate / output sample rate each sample, even below 0. You hear nothing once you are 50ms past the start or end of the wave. With a loop point you loop from it to the end of the wave infinitely. You enter the loop when your position reaches the loop point. After that your position wraps from the end of the wave to the loop point and before the loop point to the end of the wave. At speed 1 you should receive the same samples as just looping.

The following should also apply when looping if the loop contains full cycles of a tone. Pitch should not change when varying speed and output sample rate. A constant tone from 100hz to 5khz that is less than 1/6th of the waves sample rate, overtones included, should not vary in frequency by more than 2% and volume by 1.5db over any 50ms. This does not include the first and last 50ms of changing speeds or output, what `Wave::stretch` produces from the first and last 50ms of a wave, or any silence around a wave. This holds for any factor from .25 to 4 and speed from -2 to 2.

When `Wave::stretch` stretches a wave out any click should be at least half it's original height within 50ms of where it would of been. And no greater than 1% of that height 100ms or more away. Including the first and last sample. Silence should remain silent.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/SamiPerttu/fundsp repo && cd repo
git checkout 559584043aa3369e92d5aad12a5e3b003cfacfd5
cp ../Dockerfile . && git apply ../test.patch
docker build -t fundsp-time-stretch .
docker run --rm fundsp-time-stretch ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t fundsp-time-stretch .
docker run --rm fundsp-time-stretch ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm fundsp-time-stretch ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
