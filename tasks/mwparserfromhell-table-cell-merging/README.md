# Add a table grid with cell merging and splitting

| | |
|---|---|
| **Project** | [earwig/mwparserfromhell](https://github.com/earwig/mwparserfromhell) (MIT) |
| **Base commit** | [`343bfee5f1`](https://github.com/earwig/mwparserfromhell/tree/343bfee5f1eb35ae9b9238df5f83d6b9a5638af1) |
| **Language** | Python |
| **Reference solution** | +434 / -1 lines across 5 files |
| **New tests** | 92 test functions, +941 lines in 1 test file |

## Problem

Add `Wikicode.get_tables()`, returning a `mwparserfromhell.table.Table` for each table, nested ones included, in start order. `Table(node)` accepts a `table` tag in wiki or HTML syntax, exposes it as `node`, and raises `ValueError` otherwise. Tag names ignore case. A table always reflects its tag's state.

A row is a `tr` directly under the table, or directly under a `thead`, `tbody` or `tfoot` that is directly under the table. A run of cells directly under the table or a group tag, broken only by whitespace, is also a row. Cells are the `td` and `th` tags directly in a row, except `|+` caption lines. Each group tag is a row group, as is each stretch of rows between them.

Place cells row by row. Each cell starts where the previous cell in its row ended and takes the first slot no earlier row covers. A span comes from the last attribute of that name, ignoring case and outer spaces. It is the number its value starts with after spaces, or one. A `colspan` of zero is one. A `rowspan` of zero reaches the last row of its group, and no span goes past it. A slot claimed twice belongs to the cell placed first.

`grid` lists rows, with the covering cell or `None` for each column up to the furthest used. `cells` is a flat list of the cells, row by row. `position(cell)` returns the `(row, column)` of a cell's first slot, and `span(cell)` the `(rows, columns)` it covers. Both raise `ValueError` for any node that is not a cell of the table.

`merge(top, left, bottom, right)` makes the top-left cell cover the region, removes the other cells there, and returns it. `split(row, column)` shrinks the cell covering that slot to its first slot, returning, in grid order, a new empty cell of its kind for each other slot where `grid` showed that cell. A slot or corner outside the grid raises `IndexError`. `ValueError` is raised if no cell covers the slot, the corners are reversed, a cell's span reaches both inside and outside the region, a cell to remove holds more than whitespace, or the region spans two groups. Neither error changes anything.

After either operation, the text must reparse to the updated grid and change only as follows. A span is written to the attribute it was read from, or added as ` rowspan="2"` when missing. A span of one removes every attribute of that name. A wiki cell gaining its first attribute also gains ` |` after it, and loses both with its last.

A new cell goes before the first cell further right in its row, or ends the row. It is `<td></td>` or `<th></th>` in HTML tables and a line of only `|` or `!` in wiki tables. A removed cell takes its line break with it, unless it ended a shared line; then the break stays with the cell before. A wiki cell left starting a shared line is written with `|`, or `!` for a header.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/earwig/mwparserfromhell repo && cd repo
git checkout 343bfee5f1eb35ae9b9238df5f83d6b9a5638af1
cp ../Dockerfile . && git apply ../test.patch
docker build -t mwparserfromhell-table-cell-merging .
docker run --rm mwparserfromhell-table-cell-merging ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t mwparserfromhell-table-cell-merging .
docker run --rm mwparserfromhell-table-cell-merging ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm mwparserfromhell-table-cell-merging ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
