# Fix block processing in oversample and add dynamic oversampling

| | |
|---|---|
| **Project** | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) (Apache-2.0) |
| **Base commit** | [`559584043a`](https://github.com/SamiPerttu/fundsp/tree/559584043aa3369e92d5aad12a5e3b003cfacfd5) |
| **Language** | Rust |
| **Reference solution** | +494 / -29 lines across 6 files |
| **New tests** | 64 test functions, +1,551 lines in 5 test files |

## Problem

Only ticks one sample at a time. When processed in blocks like `Wave::render` and `Net` will it produces no sound. If block size is not even it drops samples. Any node that does not have the same number of inputs and outputs will fail. You cannot set settings on it and have them take effect on the node within. Signal flow analysis thinks the node within is running at the rate of the outside.

Make it produce the same output when processed in blocks of any size less than or equal to 64 as it does when ticked. For all nodes. Output when ticked should remain the same. Settings set on it should pass through to the node within.

Analysis should report latency in samples of the outside rate. So the latency of the node within will be divided by 2. And the resample filters in `oversample` will add no latency. Frequency responses will include the resample filters. Meaning if you delay the reported response by the reported latency you get the true frequency response of the node at any sample rate.

Add `oversample_linear(node)` to all 3 preludes. Works like `oversample` but uses the linear phase halfband filter from the module. Will report the fixed delay of these filters as latency added to the node within. And the rest of the response will have no phase.

Add `OversampleUnit::new(factor: usize, unit: Box<dyn AudioUnit>)` and `OversampleUnit::linear(factor, unit)` to the `oversample` module. These will return an `OversampleUnit`. Which is an `AudioUnit` that will run `unit` at `factor` times the sample rate. Factor can only be 2, 4, 8 or 16. Otherwise it will panic. Oversampling by 2^k should be identical to using k `oversample`(s) or `oversample_linear`(s) around the same node. Producing the same samples with the same random phase anywhere in a graph. As well as having the same analysis and settings. Even after being reset, cloned or changing sample rate. Should not allocate memory when ticked or processed after `allocate`.

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
docker build -t fundsp-oversampling-units .
docker run --rm fundsp-oversampling-units ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t fundsp-oversampling-units .
docker run --rm fundsp-oversampling-units ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm fundsp-oversampling-units ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
