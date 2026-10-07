# Iterate over array and JSON values in Pony queries

| | |
|---|---|
| **Project** | [ponyorm/pony](https://github.com/ponyorm/pony) (Apache-2.0) |
| **Base commit** | [`07136c853b`](https://github.com/ponyorm/pony/tree/07136c853b180a5987523877a471a90296a5e2d8) |
| **Language** | Python |
| **Reference solution** | +425 / -7 lines across 7 files |
| **New tests** | 62 test functions, +1,112 lines in 1 test file |

## Problem

A loop in a declarative query can only walk entities, entity collections and other queries. Let a loop also walk the elements of an array value (an IntArray, StrArray or FloatArray attribute, or an expression such as a slice of one) and of a JSON value, both in the main query and in generators nested inside it, such as the ones passed to count, sum, min, max, avg, group_concat and exists.

Array elements have the array's item type and support the same operations as ordinary int, str or float values. Elements of a JSON value are JSON values themselves: they can be compared with constants, used as conditions, subscripted, tested with in, iterated again and selected as Python values. Every element is produced, including repeated ones. A missing, null or empty collection produces nothing, and so does a JSON value that is not an array.

A JSON query parameter, a Json object or a dict, is walked the same way by any loop except the query's first, including a nested generator's loop: directly, inside enumerate(), zipped with another collection, or through keys(), values() and items(). As the source of the first loop it raises TypeError. A Python list or tuple is still not accepted as a loop source, including as an argument of enumerate() or zip(), and raises TranslationError as today.

`enumerate(collection)` produces (position, element) pairs with positions starting at zero, or at the integer constant given as start. `zip()` over two or more arrays or JSON arrays pairs their elements by position and stops at the end of the shortest one. For a JSON value, `keys()`, `values()` and `items()` walk the members of a JSON object and produce nothing for any other JSON value. When the number of loop variables does not match the source, the query raises TypeError.

These loops follow the rules Pony already applies when a loop walks an entity collection: rows of the result are distinct unless without_distinct() is used, and, as in every Pony query, order_by() turns that default off while distinct() asks for it back. Aggregates over elements behave exactly as they do over attribute values, except that sum, avg, min and max treat JSON elements as numbers and group_concat treats them as text. This holds for a nested generator, a generator passed directly to the function, the query method of the same name and a call inside the selected expression. Inside left_join(), a row whose collection produces nothing is still returned, with None for the loop variable. Queries built this way support filter(), first(), order_by(), grouping by an element and delete().

These loops are available with PostgreSQL and with SQLite when its library has the JSON1 functions. Creating a query that contains one raises NotImplementedError with any other provider, CockroachDB and Oracle included, and with SQLite when JSON1 is missing. Loops over every other kind of source keep their current behavior.

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
docker build -t pony-collection-iteration .
docker run --rm pony-collection-iteration ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t pony-collection-iteration .
docker run --rm pony-collection-iteration ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm pony-collection-iteration ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
