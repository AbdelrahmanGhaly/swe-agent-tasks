# Add reusable templates for composing discrete Bayesian networks

| | |
|---|---|
| **Project** | [pgmpy/pgmpy](https://github.com/pgmpy/pgmpy) (MIT) |
| **Base commit** | [`e781b58a73`](https://github.com/pgmpy/pgmpy/tree/e781b58a736665755e1413f5a5e7aed1435fb8fb) |
| **Language** | Python |
| **Reference solution** | +547 / -0 lines across 5 files |
| **New tests** | 104 test functions, +761 lines in 1 test file |

## Problem

`NetworkTemplate` in `pgmpy.models` is a piece of a discrete Bayesian network, built from edges plus `boundary`, the nodes left open for variables supplied later. A boundary node has no parents and no CPD of its own. `boundary_states(node)` returns the states the CPDs of its children give it, and rejects a node off the boundary. Every other node is interior and carries a `TabularCPD` over its parents, added with `add_cpds` and returned by `get_cpds`.

`check_template()` returns True or raises. It rejects a boundary node with a parent, one no CPD mentions, and one whose children disagree about its states. It rejects an interior node with no CPD, one whose CPD evidence is not exactly its parents, one whose CPD leaves the states of a variable unnamed, one whose CPD gives a parent states that parent does not have, and one whose CPD is not a distribution. Every rejection here or below raises `ValueError`.

`NetworkTemplate.from_network(network, nodes)` builds a template whose interior is `nodes`, with their CPDs copied and every parent outside `nodes` as a boundary node. `nodes` must be a non-empty selection of variables of the network. `to_network(priors)` goes the other way and returns a `DiscreteBayesianNetwork`: `priors` maps exactly the boundary nodes, no more and no fewer, each to a `TabularCPD` over that node alone using the states the template gives it, which becomes that node's CPD.

`DiscreteBayesianNetwork.embed(template, prefix, bindings)` returns a copy of the network with the template added, leaving the original unchanged. Each embedded interior node is named `prefix.<node>`. Each boundary node is replaced by the variable `bindings` maps it to, so its template children take that variable as a parent. `bindings` maps exactly the boundary nodes, no more and no fewer, no two to the same variable, each to a network variable carrying exactly the states that node needs. A `prefix.<node>` name already in use is refused. The CPDs carry over unchanged, so the piece keeps the distribution it described. `compose(other, prefix, bindings)` does the same inside a template, binding to interior or boundary nodes and keeping the outer boundary.

`marginalize(nodes)` returns a copy of the template with those interior nodes summed out. The result must have those nodes gone, be a well formed template, and carry the joint distribution the template had over the variables left. How it gets factorized is left open: children that shared a removed node may gain edges among themselves. That holds even where a configuration of a child's parents has probability zero. A boundary node the result no longer uses drops out with it. A boundary node cannot be summed out, and neither can a node the template does not have.

`TabularCPD.rename(mapping)` returns a copy of the CPD with the variables in `mapping` renamed and the table left as it is, so every entry keeps the states it was written for. It rejects a mapping that would give two of the CPD's variables one name, or that names a variable the CPD does not have.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/pgmpy/pgmpy repo && cd repo
git checkout e781b58a736665755e1413f5a5e7aed1435fb8fb
cp ../Dockerfile . && git apply ../test.patch
docker build -t pgmpy-network-templates .
docker run --rm pgmpy-network-templates ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t pgmpy-network-templates .
docker run --rm pgmpy-network-templates ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm pgmpy-network-templates ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
