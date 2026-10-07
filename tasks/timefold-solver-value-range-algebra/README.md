# Add union, intersection and difference value ranges

| | |
|---|---|
| **Project** | [TimefoldAI/timefold-solver](https://github.com/TimefoldAI/timefold-solver) (Apache-2.0) |
| **Base commit** | [`9d8a59cf92`](https://github.com/TimefoldAI/timefold-solver/tree/9d8a59cf92d27bce8a425e117e446a42d2b7c51c) |
| **Language** | Java |
| **Reference solution** | +1,320 / -6 lines across 19 files |
| **New tests** | 98 test functions, +1,898 lines in 12 test files |

## Problem

`ValueRangeFactory` gains four new ways to build a `ValueRange`. `createCollectionValueRange(Collection<T> values)` wraps a collection in its iteration order, where a Set must be a SortedSet or a LinkedHashSet and no element may be null. `createUnionValueRange(ValueRange<T>... valueRanges)`, `createIntersectionValueRange(ValueRange<T>... valueRanges)` and `createDifferenceValueRange(ValueRange<T> valueRange, ValueRange<T> excludedValueRange)` compose existing ranges. A composition is an ordinary value range: it works as the result of a value range provider on the solution or on an entity, for basic and list variables, and its members may themselves be compositions. The union and intersection factories reject an empty member list, and every new factory method rejects a null argument, including a null member array or a null collection element, with an IllegalArgumentException.

A member decides membership with its own contains check. A member that allows null produces null among its values, and a composition carries that null like any other value it produces. The size of a composition is the number of values it produces, and get with an index from zero to size minus one returns the value at that position of the original iterator. 

An intersection produces the values of its first member, in that member's order, that every other member contains. A difference produces the values of its first member, in order, that the excluded range does not contain. A union produces the values of its first member, then the values of each later member that no earlier member contains, each member keeping its own order. Within one member, repeats of an equal value are produced once, at the first occurrence. The order of a range that the factory builds from bounds is ascending.

A composition stays as cheap as its members. When the members are int, long, BigInteger or BigDecimal ranges built from bounds, or collections combined with such ranges, size, get, contains, the original iterator and random selection are computed from the bounds, the increments and the collection values, so members holding billions of values are never visited. The random iterator never ends while the composition has values, selects every value with equal probability, and draws one bounded random index per value the way the built-in ranges do, so with the same random source it returns the values those ranges return for the same indexes.

A composition can be sorted exactly when every member can be sorted. The sorted range holds the same values reordered by the sorter, a null among them is never handed to the sorter and stays first, and a composition with a member that cannot be sorted rejects sorting with the same unsupported-operation error that member raises. Two compositions are equal, with equal hash codes, when they apply the same operation to equal members in the same order. The value counts and problem size statistics the solver reports reflect the composed ranges.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/TimefoldAI/timefold-solver repo && cd repo
git checkout 9d8a59cf92d27bce8a425e117e446a42d2b7c51c
cp ../Dockerfile . && git apply ../test.patch
docker build -t timefold-solver-value-range-algebra .
docker run --rm timefold-solver-value-range-algebra ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t timefold-solver-value-range-algebra .
docker run --rm timefold-solver-value-range-algebra ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm timefold-solver-value-range-algebra ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
