# Add database-generated columns to Piccolo tables

| | |
|---|---|
| **Project** | [piccolo-orm/piccolo](https://github.com/piccolo-orm/piccolo) (MIT) |
| **Base commit** | [`6ef3751426`](https://github.com/piccolo-orm/piccolo/tree/6ef3751426c0c6df82f40b712904bf91df98028c) |
| **Language** | Python |
| **Reference solution** | +780 / -61 lines across 16 files |
| **New tests** | 208 test functions, +2,389 lines in 1 test file |

## Problem

Add `Generated` to `piccolo.columns`: a column reads `Varchar(generated=Generated("UPPER({})", name))`. Its first argument is either SQL with curly brackets as placeholders followed by the columns filling them, or a Piccolo comparison such as `popularity > 100`, alone or combined with `&` and `|`. Anything else, a function call included, must use the placeholder form, even as a comparison operand. A comparison takes no columns after it; giving it any, or a placeholder count mismatch, is a `ValueError`. `stored` defaults to `True`.

Each placeholder becomes the column's `db_column_name` in double quotes, and so does a column used as a comparison operand or inside its list. Resolve those names when the SQL is rendered, after the table class is built, not when `Generated` is created. A generated expression cannot carry bind parameters, so values are inlined: numbers and booleans as they stand, strings, dates, times and UUIDs quoted with any quotes inside doubled, `None` as null. A comparison hands over the value Piccolo already converted; anything else, a NaN or an infinity included, is a `ValueError`.

It reaches the table definition as `GENERATED ALWAYS AS (<expression>) STORED`, or `VIRTUAL` when `stored` is `False`, with no `DEFAULT`, which a database refuses. Nothing else changes: a nullable one omits `NOT NULL`; placeholder-free SQL passes through unchanged. A generated primary key is a `ValueError` at creation.

The database owns the value, so Piccolo never sends it, fixture loads included: it is absent from inserts and updates, a constructor value is kept but not written, and an unset attribute on a new row is `None`, not the type's default. `save()` and `update_self()` bring back what the database calculated, even where it can't return values from a write; `ModelBuilder` leaves them alone. Reject explicit writes. Naming one in `update()`, `update_self()`, `save(columns=...)` on a new or existing row, `on_conflict()` values or `ModelBuilder` defaults, by column or by string, raises a `ValueError` naming it. Otherwise it is ordinary: selects, filters, ordering, objects and refreshes see it.

`alter().add_column()` adds one and `alter().drop_column()` removes one, though dropping a column named by a placeholder or comparison, generated or not, raises a `ValueError` naming both, pending additions included, in either order. Only `alter().add_column()` rejects a stored one on SQLite, raising a `ValueError` naming it; a table definition still creates one.

Auto migrations carry the expression across. Only a generated column's params gain a `Generated` key; its other params are unchanged. A migration file deep-copies it and records the copy as a `Generated` call holding one finished SQL string with the names substituted in and `stored` kept, plus the import, so no column objects or placeholders survive. A migration adds a column under its `db_column_name`. A newly added generated column keeps its expression, goes in after the columns a placeholder or comparison names and out before them when reversed. A changed or removed expression is an altered column, but running the migration raises a `ValueError` naming it: a database won't rewrite one in place.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/piccolo-orm/piccolo repo && cd repo
git checkout 6ef3751426c0c6df82f40b712904bf91df98028c
cp ../Dockerfile . && git apply ../test.patch
docker build -t piccolo-generated-columns .
docker run --rm piccolo-generated-columns ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t piccolo-generated-columns .
docker run --rm piccolo-generated-columns ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm piccolo-generated-columns ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
