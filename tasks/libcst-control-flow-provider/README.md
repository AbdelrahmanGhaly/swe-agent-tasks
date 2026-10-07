# Add a control flow provider for statement reachability

| | |
|---|---|
| **Project** | [Instagram/LibCST](https://github.com/Instagram/LibCST) |
| **Base commit** | [`d9a255843b`](https://github.com/Instagram/LibCST/tree/d9a255843b5cdbecc6834684d233bce1f2987f9d) |
| **Language** | Python |
| **Reference solution** | +407 / -0 lines across 5 files |
| **New tests** | 206 test functions, +2,127 lines in 2 test files |

## Problem

Add a `ControlFlowProvider` whose metadata for every statement and small statement has a `reachable`
boolean and an `exits` frozen set. The provider and `ExitKind` are exported from `libcst.metadata`.
`exits` is drawn from the five `ExitKind` members `FALL_THROUGH`, `RETURN`, `RAISE`, `BREAK` and
`CONTINUE`. Every statement carries the metadata whether or not it is reachable, and `exits`
describes the statement on its own, so an unreachable `raise` still reports `RAISE` and an
unreachable assignment still reports `FALL_THROUGH`.

A statement is reachable when control can arrive from the start of its block, so anything after a
statement that cannot fall through is not. A statement built out of other statements reports the
exits actually reached in order, so once one part cannot fall through the parts after it contribute
nothing. A loop absorbs its own body's breaks and continues, so a break belonging to a nested loop
does not let the outer loop fall through.

A test is constant when syntax alone fixes its truth value: `True`, `False`, `None`, numbers with an
optional sign, strings, collection literals, and `not` in front of any of those. A formatted string
counts only when it has no interpolations, since an interpolated one depends on its values, and the
same holds for each piece of an implicitly concatenated string. An empty
collection literal is
false, and one holding at least one element that is not an unpacking is true. A literal whose
elements are all unpackings, such as `[*c]`, is not constant, since what it unpacks may turn out to
be empty.

An `if` on a constant test runs only the branch it picks, and only that branch contributes exits. A
`while` on a constant true test completes only through a break, so its `else` cannot be reached; one
on a constant false test never runs its body; a `for` can always complete, since its iterable may be
empty.

Every `except` handler is reachable, and its exits join the guarded statement's. When a `finally`
body cannot fall through, its exits replace those of the statement it guards; when it can, its other
exits join them. A `match` falls through unless some case is irrefutable. Function and class bodies
are analyzed on their own, so their statements stay reachable even when the definition is not.

Also add `RemoveUnreachableCodeVisitor`, exported from `libcst.codemod.visitors`, and
`RemoveUnreachableCodeCommand` in `libcst.codemod.commands.remove_unreachable_code`, which is a
codemod command so the command line tooling lists it alongside the others. They delete
everything the provider marks unreachable, leave a `pass` in an emptied block, and drop the trailing
semicolon when trimming a semicolon-joined line. Whatever survives keeps the comments and spacing it
started with.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Instagram/LibCST repo && cd repo
git checkout d9a255843b5cdbecc6834684d233bce1f2987f9d
cp ../Dockerfile . && git apply ../test.patch
docker build -t libcst-control-flow-provider .
docker run --rm libcst-control-flow-provider ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t libcst-control-flow-provider .
docker run --rm libcst-control-flow-provider ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm libcst-control-flow-provider ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
