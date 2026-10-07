# Add resolved module output to the YANG and YIN formats

| | |
|---|---|
| **Project** | [mbj4668/pyang](https://github.com/mbj4668/pyang) (ISC) |
| **Base commit** | [`ddd73d7e6e`](https://github.com/mbj4668/pyang/tree/ddd73d7e6ee62a1e127968a3b1b56351f41fe350) |
| **Language** | Python |
| **Reference solution** | +560 / -3 lines across 7 files |
| **New tests** | 61 test functions, +2,091 lines in 2 test files |

## Problem

Add `--yang-resolve` to the `yang` output format and `--yin-resolve` to the `yin` format. With the option set, the module is printed in resolved form: every `uses` is replaced by its nodes, no `grouping` statement is left, and augments of the module's own nodes are merged into them. Augments of other modules stay augments, with their content resolved the same way.

Compiled by itself with the same search path and `--features`, but without `--deviation-module`, the printed module must produce no errors or warnings, and give the same schema as the original compiled with the printing options. Nodes removed by a disabled feature or a not-supported deviation, and nodes other modules add, are not printed; deviated properties appear in their deviated form. The output imports what it uses and nothing else.

The same schema means the same nodes in order, choices and cases included, with the same config, mandatory, presence, status, units, keys, unique constraints, element counts and extension statements at any depth, including under a moved or combined `when`. Types reduce to the same built-in type with the same restrictions, defaults are the same values, leafrefs point at the same nodes, if-feature conditions are the same, and when and must expressions select the same nodes, compare against the same values and keep their grouping. Layout, quoting, the order of non-node statements and the names of moved definitions do not matter.

A moved statement keeps its meaning, and every prefix in the printed module (types, identities, features, extension keywords, paths, expressions) is read through its own imports. So a `default red`, a `type small` or the `'red'` of `derived-from(../paint, 'red')` written in a grouping of module `grp` names the identity or typedef of `grp`, even when defined there, and a deviation's `default gd:dark-red` names what `gd` means in the deviation module.

A `when` on a `uses` is evaluated from the node holding the `uses`, also on the nodes it brings in. Choices and cases are not steps in an expression: on a leaf in a case below container `sel`, `../on` selects `sel/on`. An augment's `when` and `if-feature`, including one inside a `uses`, apply to every node it adds, and conditions meeting on one node are combined into one `when` requiring all of them.

Input limits: YANG 1.1 only, no submodules, one revision per module. Deviation modules define no typedefs or identities; extensions define no data nodes. No union, leafref or instance-identifier type has a default, and an `input` or `output` emptied by pruning has no `must`. A `uses` or a merged augment that carries a `when` adds no action or notification. The module has no `deviation` of its own; no pruning removes a choice's default case; no two same-named nodes start on one line. Expressions call only XPath 1.0 and YANG functions, `current()` is never followed directly by a predicate, a union has at most two operands, and a `prefix:name` string only appears as the identity argument of `derived-from()` or `derived-from-or-self()`.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/mbj4668/pyang repo && cd repo
git checkout ddd73d7e6ee62a1e127968a3b1b56351f41fe350
cp ../Dockerfile . && git apply ../test.patch
docker build -t pyang-resolved-modules .
docker run --rm pyang-resolved-modules ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t pyang-resolved-modules .
docker run --rm pyang-resolved-modules ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm pyang-resolved-modules ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
