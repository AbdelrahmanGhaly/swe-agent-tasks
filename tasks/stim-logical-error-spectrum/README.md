# Add a command reporting the graphlike logical error spectrum

| | |
|---|---|
| **Project** | [quantumlib/Stim](https://github.com/quantumlib/Stim) (Apache-2.0) |
| **Base commit** | [`79ae4f118c`](https://github.com/quantumlib/Stim/tree/79ae4f118ca11c615d6d8de7c6eed7d189d3a6eb) |
| **Language** | C++ |
| **Reference solution** | +866 / -1 lines across 9 files |
| **New tests** | 58 test functions, +943 lines in 2 test files |

## Problem

Stim can already find one minimum weight undetectable logical error, but it cannot say how many of them there are, and it cannot say what each observable costs on its own. Add a `shortest_errors` command that reads a detector error model and writes that report.

An error mechanism takes part when its probability is not zero. A mechanism written with separators is split at them and each part is treated as a mechanism of its own. Repeated targets in the same part cancel in pairs, so a part carries only the detectors and observables it names an odd number of times. Parts mentioning more than two detectors are skipped. Parts mentioning the same detectors and flipping the same observables count once, however many times they appear and whatever tags they carry. A set of mechanisms is an undetectable logical error when every detector is mentioned an even number of times across the whole set and at least one observable is flipped an odd number of times. The weight of such a set is the number of mechanisms in it.

The report always starts with a `distance` line holding the smallest weight any undetectable logical error reaches, then a `multiplicity` line holding how many distinct sets reach that weight, then one line for each observable index in increasing order, named `L0`, `L1` and so on. Each of those lines holds two numbers: the smallest weight of an undetectable logical error that flips that observable, and how many distinct undetectable logical errors of that weight flip it. That second number is counted at the observable's own weight, which need not be the distance, and only errors flipping that observable are counted, so it is not the multiplicity.

An observable counts as flipped whether or not the same error also flips others. Where no such weight exists the line reads `none 0`, and a model with no undetectable logical error at all reports `distance none` together with `multiplicity 0`. The number of observable lines is the observable count of the model itself, so an index no mechanism mentions still gets a line.

Passing `--list` writes every minimum weight undetectable logical error after that report. Each one is preceded by a blank line and written as `error(1)` instructions, one mechanism per line, detector targets ahead of observable targets. Mechanisms within a set, and sets against each other, are both ordered by comparing detector indices first and then observable indices, always numerically. Nothing is written when the model has no undetectable logical error.

The model is read from `--in`, or from standard input when that is absent, and the report is written to `--out`, or to standard output when that is absent. Any other flag is rejected.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/quantumlib/Stim repo && cd repo
git checkout 79ae4f118ca11c615d6d8de7c6eed7d189d3a6eb
cp ../Dockerfile . && git apply ../test.patch
docker build -t stim-logical-error-spectrum .
docker run --rm stim-logical-error-spectrum ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t stim-logical-error-spectrum .
docker run --rm stim-logical-error-spectrum ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm stim-logical-error-spectrum ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
