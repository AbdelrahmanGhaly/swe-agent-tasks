# Add cost-ordered enumeration of the expressions in an e-class

| | |
|---|---|
| **Project** | [egraphs-good/egg](https://github.com/egraphs-good/egg) (MIT) |
| **Base commit** | [`2f31b28e3f`](https://github.com/egraphs-good/egg/tree/2f31b28e3f9d78e02273b6c6d4201b5b0720b343) |
| **Language** | Rust |
| **Reference solution** | +406 / -0 lines across 3 files |
| **New tests** | 60 test functions, +738 lines in 1 test file |

## Problem

An `Extractor` can already pick the single cheapest expression an e-class represents. There is no way to ask for the second cheapest, or to walk a class looking for an expression of some particular shape. Add that.

Three new methods on `Extractor`, all taking `&mut self` because costing does. `exprs(&mut self, eclass: Id) -> Exprs<'_, CF, L, N>` returns a lazy iterator over the expressions the class represents, yielding `(CF::Cost, RecExpr<L>)`. `find_k_best(&mut self, eclass: Id, k: usize) -> Vec<(CF::Cost, RecExpr<L>)>` collects the first `k` of them, or all of them when the class holds fewer than `k`. `exprs_matching(&mut self, eclass: Id, pattern: &Pattern<L>) -> MatchingExprs<'_, CF, L, N>` yields that same sequence with only the expressions matching `pattern` kept, in the order `exprs` would have given them.

`Exprs` and `MatchingExprs` are public types implementing `Iterator`, so a caller can take a handful of results without paying to build the rest of the class. Both accept any `Id`, canonical or not. Once either has run out it keeps returning `None`, and enumerating leaves the e-graph itself unchanged.

An expression counts as one the class represents when no e-class repeats along a single root-to-leaf path through it. That is a per-path rule and not a per-expression one, so two sibling positions may draw on the same e-class. A class containing `(+ a a)` whose child class represents both `x` and `y` therefore yields `(+ x x)`, `(+ x y)`, `(+ y x)` and `(+ y y)`, because each child position chooses its subterm independently of the others. A class every one of whose expressions is cyclic yields nothing at all, and so does that class used as a child, though its siblings still contribute.

The cost paired with an expression is the cost function's `cost_rec` of that finished expression, so a subterm reached through two positions is paid for twice. Results come out in nondecreasing cost order, which makes the first one agree with `find_best`; expressions that tie on cost may come in any order. Each distinct expression is yielded at most once however many derivations of it exist, which starts to matter once unions let two e-nodes of one class build the same term.

Matching works on whole expressions rather than their subterms, and follows the usual pattern rules. A pattern node matches when the operator and the arity agree and every child matches. A variable matches any subterm, but a variable occurring more than once has to bind the same subterm at every occurrence, while two different variables are free to bind the same subterm.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/egraphs-good/egg repo && cd repo
git checkout 2f31b28e3f9d78e02273b6c6d4201b5b0720b343
cp ../Dockerfile . && git apply ../test.patch
docker build -t egg-eclass-enumeration .
docker run --rm egg-eclass-enumeration ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t egg-eclass-enumeration .
docker run --rm egg-eclass-enumeration ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm egg-eclass-enumeration ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
