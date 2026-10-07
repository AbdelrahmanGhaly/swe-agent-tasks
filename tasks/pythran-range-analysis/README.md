# Sharpen integer range analysis in the pythran optimizer

| | |
|---|---|
| **Project** | [serge-sans-paille/pythran](https://github.com/serge-sans-paille/pythran) (BSD-3-Clause) |
| **Base commit** | [`4a19d7b9b8`](https://github.com/serge-sans-paille/pythran/tree/4a19d7b9b8ff9d47bbab2df847f365c542b23f00) |
| **Language** | Python |
| **Reference solution** | +733 / -117 lines across 4 files |
| **New tests** | 140 test functions, +1,124 lines in 1 test file |

## Problem

`RangeValues` reports an interval for each expression, and the optimizer rewrites code based on those intervals. Report for each operation the smallest interval holding every value it produces as its operands range over theirs. Build each interval only from the operand pairs the operation accepts. A zero divisor and a negative shift count contribute nothing rather than raising. An operation left with no acceptable pair is unbounded. Report an unbounded interval when an accepted pair may give something that is not whole, which covers true division and negative exponents. Keep an endpoint finite whenever the operands still fix it, and do not make an endpoint infinite merely because an operand has an infinite endpoint.

Python's integer arithmetic decides the values: floor division and remainder round toward negative infinity, and `&`, `|`, `^` and the complement work on two's complement numbers with no word size, so a negative operand carries endless leading one bits. `abs` reports the magnitudes its operand reaches.

The remainder follows the rules here instead of the smallest-interval law, since pinning it exactly costs work growing with the numbers involved. A positive divisor gives values from zero up to one below the largest. A negative divisor gives values from one above the smallest up to zero. A divisor range holding both signs gives both windows. A dividend of the divisor's sign caps its window on that side. When every divisor has greater magnitude than every dividend, a dividend sharing the divisor's sign comes through unchanged, and a dividend of the opposite sign is shifted by the divisor.

`and` and `or` report the operand they yield, not the span of both. `or` gives its left side's nonzero values, plus the right side's values when the left can be zero; `and` gives zero when the left can be zero, plus the right side's values when the left can be something else. A chain is read left to right, and an interval holding zero counts as able to be zero. `not` reports zero when its operand cannot be zero, one when its operand cannot be anything else, and both values when the operand can be either.

A comparison on a sum, a difference or a leading minus also narrows every variable inside it, on either side and at each link of a chain: translate the bound back through those steps onto each one, and ignore a translated bound whose low end exceeds its high end. A comparison naming a variable on its own keeps the behaviour it has today.

`RangeBasedSimplify` rewrites remainders, and each remainder rewrite must compute what Python computes and must happen whenever it is sound. A remainder becomes a comparison and a subtraction only when the dividend is not negative and cannot reach twice the smallest divisor. A rewrite may evaluate an operand more than once only when evaluating it has no effect. Everything the analysis already reports correctly keeps its values, and the package's doctests are part of the suite.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/serge-sans-paille/pythran repo && cd repo
git checkout 4a19d7b9b8ff9d47bbab2df847f365c542b23f00
cp ../Dockerfile . && git apply ../test.patch
docker build -t pythran-range-analysis .
docker run --rm pythran-range-analysis ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t pythran-range-analysis .
docker run --rm pythran-range-analysis ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm pythran-range-analysis ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
