# Preserve model structure through the HiGHS file handlers

| | |
|---|---|
| **Project** | [ERGO-Code/HiGHS](https://github.com/ERGO-Code/HiGHS) (MIT) |
| **Base commit** | [`73cac48c53`](https://github.com/ERGO-Code/HiGHS/tree/73cac48c5340d775a477087198611862559be250) |
| **Language** | C++ |
| **Reference solution** | +415 / -128 lines across 8 files |
| **New tests** | 56 test functions, +1,305 lines in 2 test files |

## Problem

Writing a model to a file and reading it back should give the same model. HiGHS does not manage that today, in either file format.

Make the LP writer keep each row as a single constraint under its own name, giving both bounds when a row has two of them and an infinite bound on each side when a row is free. The LP reader needs the matching form, so a constraint may carry a bound on each side, written ascending or descending. Both comparisons of such a constraint have to run the same way, and the lower bound may not exceed the upper one. A constraint that breaks either rule is a file error.

Rows with one bound, and rows whose bounds are equal, keep the form they have today. The bounds section keeps its present meaning, so an entry there still bounds a column and never becomes a row, and a two sided constraint on a single variable is still a row.

Every column survives too, in the position it had. A column that costs nothing and appears in no row is still part of the model, and the order of the columns is part of it as well. That holds for a column named like a value the format also uses, one called inf or 1 or free, anywhere the file names it, a quadratic term included, except in a bound entry of its own.

Integrality is part of what round trips. A column that is continuous, integer, semi-continuous or semi-integer must read back as the same kind, and the objective sense, offset and row name must survive as well. So must the constraint matrix itself.

The same guarantee holds for MPS files under either parser. The fixed format reader should honour `BV`, `LI`, `UI`, `SC` and `SI` alongside the types it knows, taking the value on an `SC` or `SI` entry as the column's upper bound, and reject an unrecognised type rather than passing over it, whether or not the column it names was declared. An integer column given any explicit bound keeps it rather than the default upper bound of one. A semi-continuous or semi-integer column keeps its kind and its bounds too, equal bounds and an infinite upper bound included, so what the file records for an infinite bound has to read back as infinite whatever value the reader treats as infinity.

A row bounded on neither side is written as an MPS `N` row. Both readers must follow the `keep_n_rows` option for the ones after the objective, whether it asks for them to be discarded, kept as rows that constrain nothing, or kept without their coefficients. A right hand side or range entry naming one of those rows is ignored: it must not bound a kept row, nor move the objective when the row is discarded.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/ERGO-Code/HiGHS repo && cd repo
git checkout 73cac48c5340d775a477087198611862559be250
cp ../Dockerfile . && git apply ../test.patch
docker build -t highs-model-roundtrip .
docker run --rm highs-model-roundtrip ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t highs-model-roundtrip .
docker run --rm highs-model-roundtrip ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm highs-model-roundtrip ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
