# Add logarithmic level units to the unit engine

| | |
|---|---|
| **Project** | [sharkdp/numbat](https://github.com/sharkdp/numbat) (Apache-2.0) |
| **Base commit** | [`49fccb1070`](https://github.com/sharkdp/numbat/tree/49fccb107047e2615c42f09f7c9d81bca1d64b92) |
| **Language** | Rust |
| **Reference solution** | +593 / -34 lines across 19 files |
| **New tests** | 152 test functions, +1,043 lines in 1 test file |

## Problem

Numbat has no way to write a unit that sits on a logarithmic scale. Decibels are the obvious example: 30 dBm is one milliwatt scaled by a thousand. Teach the unit engine about them and ship a prelude module of the common ones.

A unit definition may now carry a `@logarithmic(s)` decorator, or `@logarithmic(s, b)` to choose a logarithm base other than ten. Both arguments are positive number literals, in any base the language writes numbers in, and b may not be one. Only one may be attached. It applies only to a derived unit, so a base unit, a `let` or a function carrying it is an error, as is a scale or base outside those bounds, and a defining expression whose physical value is not positive.

A quantity in such a unit is a level. A level of x denotes the linear quantity `R * b^(x/s)`, where R is the defining expression, in the unit of R, so 10 dBm reads as 10 milliwatt. A metric prefix divides the scale: a decibel is a level on a scale ten times that of a bel.

A plain number in front of a logarithmic unit builds a level. Multiplying or dividing a level by a plain unitless number, and negating one, scale the level value and keep the unit. Everywhere else a level stands for its linear reading: addition, subtraction, comparison, equality, conversion to another unit, factorial and every function call work on linear values, so 10 dBm plus 10 dBm is 20 milliwatt. A level nested in a list or in a struct field is linearized with it.

Converting to a logarithmic unit is the inverse: the left hand side needs the same dimension and a positive value, and the result is the level `s * log(q / R)` in base b; zero or negative has to fail. Converting to a level ignores any magnitude on the right hand side, so `1 W -> 2 dBm` gives what `1 W -> dBm` gives; for ordinary units the display it produces today keeps working, as do the temperature pseudo-units.

A logarithmic unit may never appear inside a compound unit, so the power operator on a level, dividing anything by a level, and multiplying or dividing a level by anything but a plain unitless number are errors, and no unit may be defined from a level, though one defined from a level's linear reading is fine. Simplification must leave levels untouched and never rewrite an ordinary quantity into a level.

Add a `units::logarithmic` prelude module: the dimensionless `bel` at scale 1 with metric prefixes, so `decibel` and `centibel` follow; `neper` at scale 0.5 in base e; `dBm` and `dBW` for power against one milliwatt and one watt; `dBV` and `dBu` for voltage against one volt and 0.7745966692414834 volt; and `dBSPL` for pressure against 20 micropascal. The voltage and pressure units measure amplitudes and use scale 20, the power units 10.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/sharkdp/numbat repo && cd repo
git checkout 49fccb107047e2615c42f09f7c9d81bca1d64b92
cp ../Dockerfile . && git apply ../test.patch
docker build -t numbat-logarithmic-units .
docker run --rm numbat-logarithmic-units ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t numbat-logarithmic-units .
docker run --rm numbat-logarithmic-units ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm numbat-logarithmic-units ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
