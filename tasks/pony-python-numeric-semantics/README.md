# Give query arithmetic Python's numeric semantics

| | |
|---|---|
| **Project** | [ponyorm/pony](https://github.com/ponyorm/pony) (Apache-2.0) |
| **Base commit** | [`07136c853b`](https://github.com/ponyorm/pony/tree/07136c853b180a5987523877a471a90296a5e2d8) |
| **Language** | Python |
| **Reference solution** | +536 / -21 lines across 7 files |
| **New tests** | 130 test functions, +1,343 lines in 1 test file |

## Problem

A query is written as a Python generator, so the expressions inside it are Python source, but the values that come back are whatever the database computed for them. The division operators are the clearest case: what a query returns for `x.qty // 3` or for `x.qty % 3` follows the database's rules rather than Python's, and `x.qty / 2` over an integer attribute never produces a fractional value at all.

Make a translated expression produce the value Python produces for it, with the type Python gives it. That covers true division, floor division and the remainder operator over every numeric attribute type the mapper supports, integers, floats, booleans and decimals alike. Decimal values follow Python's own decimal rules rather than the integer ones, so a decimal quotient drops its fraction toward zero and a decimal remainder takes the sign of the left operand.

The database still does the arithmetic, so a decimal, a float or a piece of text keeps its representation and rounding there: a decimal carries the precision of its column rather than Python's context, a float is divided and reduced by whatever floating-point operations that database offers, which at a rounding boundary need not land where CPython lands, and text that comes back empty is whatever that database stores. A whole number is the exception: however wide it grows it arrives as the Python integer it ought to be. `math.floor` of a float attribute holding 1e20 is 100000000000000000000, and a floor division or remainder over an attribute at the far end of a 64-bit column gives Python's value, not one clipped to that column.

It also covers expressions a query rejects today, each carrying the meaning the same expression has in Python. `round` with one argument, `math.floor`, `math.ceil` and `math.trunc` give back an integer. A string multiplied by an integer repeats it, in either order. `count` and `find`, each taking only the text to look for, report occurrences and a position the way the string methods of those names do, the case of searching for empty text included. `startswith` and `endswith` take a tuple of alternatives and hold when any member of it matches, while a list written out in the query is refused. `ljust`, `rjust` and `zfill` pad to a width, the first two also taking a fill character, which has to be a literal one character long.

A value has to be the same wherever its expression appears: in the selected result, in a condition, in an ordering key, inside an aggregate function, and in an expression built over a collection attribute.

Addition, subtraction, multiplication, exponentiation, comparison and every string operation a query already understands keep their present behavior, including the type each of them returns for boolean attributes. An expression built only from constants keeps the result it already gives, and a division or a remainder whose divisor is zero still leaves that row's value undefined.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/ponyorm/pony repo && cd repo
git checkout 07136c853b180a5987523877a471a90296a5e2d8
cp ../Dockerfile . && git apply ../test.patch
docker build -t pony-python-numeric-semantics .
docker run --rm pony-python-numeric-semantics ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t pony-python-numeric-semantics .
docker run --rm pony-python-numeric-semantics ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm pony-python-numeric-semantics ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
