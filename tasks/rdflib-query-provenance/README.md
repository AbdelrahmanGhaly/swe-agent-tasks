# Report what a SPARQL update changed and why

| | |
|---|---|
| **Project** | [RDFLib/rdflib](https://github.com/RDFLib/rdflib) (BSD-3-Clause) |
| **Base commit** | [`2dc7bc5ec4`](https://github.com/RDFLib/rdflib/tree/2dc7bc5ec4593e0659c40834adfc5b5f022b07e2) |
| **Language** | Python |
| **Reference solution** | +1,004 / -91 lines across 7 files |
| **New tests** | 154 test functions, +2,293 lines in 2 test files |

## Problem

Add provenance reporting to SPARQL updates. `graph.update(request, provenance=True)`, on a `Graph` or a `Dataset`, returns a report of what the request did. A bare `ConjunctiveGraph` is out of scope. The report is a mapping under `"added"` and `"removed"`, each pairing a quad with a frozenset of the quads that justified that change. Each quad names the graph holding it, and the default graph of a dataset answers to `urn:x-rdflib:default`. When `provenance=True`, the SPARQL update engine runs the request in place of a store-provided `update` or a processor the caller supplied, and the store ends where the same request run by that engine without the flag would leave it. The request may be text or prepared in advance. Without the flag a request keeps the route and the return value it has today.

The report is the difference between the store before the request and the store after it, including data a dataset clause leaves behind. Three cases leave the store as it was, so none appear: adding a statement already held, deleting one not held, and adding one the request later removes. If a quad changes more than once, only the reason for its final change is kept; reasons for changes that cancel are dropped. An operation asking for a change the store already reflects alters nothing and adds no reason of its own.

Explicit data in the request, and data read from a document it names, have empty justifications. So do quads removed by emptying, dropping or replacing a graph; only removal by matching carries witnesses. Data copied from another graph is justified by the quad it came from. Quads a request reads from a document to evaluate its WHERE clause name that document as their graph, so a statement two documents carry is justified by two quads. A property path match is outside that rule.

A change produced by a WHERE solution inherits that solution's matched quads. A successful condition also contributes the quads it matched, including when a `FILTER`, a `BIND` or a grouping used its answer. Where a condition could hold in more than one way, the report does not fix which match it names, and it does not fix whether or which quads a property path match contributes. Absence justifies nothing, so an optional part that found no match and a condition that did not hold contribute no quads of their own. Where a condition inside a discarded part held, the report does not fix whether its match is kept.

Combining solutions unions their quads, and when solutions with the same bindings collapse, the survivor carries the union of all of them. A solution computed from a group carries that group's union, and where several solutions account for one changed quad, its entry carries the union.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/RDFLib/rdflib repo && cd repo
git checkout 2dc7bc5ec4593e0659c40834adfc5b5f022b07e2
cp ../Dockerfile . && git apply ../test.patch
docker build -t rdflib-query-provenance .
docker run --rm rdflib-query-provenance ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t rdflib-query-provenance .
docker run --rm rdflib-query-provenance ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm rdflib-query-provenance ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
