# Add .RECIPEPREFIX support to ckati and rkati

| | |
|---|---|
| **Project** | [google/kati](https://github.com/google/kati) (Apache-2.0) |
| **Base commit** | [`7a3ee413e1`](https://github.com/google/kati/tree/7a3ee413e1cfc230a597ac33034d2f478351e432) |
| **Language** | C++ and Rust |
| **Reference solution** | +1,250 / -216 lines across 34 files |
| **New tests** | +1,185 lines in 1 test file |

## Problem

ckati and rkati will only consider lines that begin with a tab as recipe lines. So if you have a makefile that changes .RECIPEPREFIX to something else it will either fail with missing separator or execute incorrect commands. Have both parse recipe lines the same way make 4.2.1 does. We install this as make and if there are any discrepancies between it and the docs use what make does.

The first char of the value of this variable will be used to start a recipe line. An empty value means tab. = means that the value will be whatever you put there pre expansion. Once you set a new value it will be used for all subsequent lines read by make. This includes lines inside conditionals that we take. Or if they are in a include or $(eval). Lines that begin with a tab while using a different prefix will be treated as normal makefile lines. Also if a recipe line is continued by \ the following prefix will not be included in the recipe. That is the prefix in effect when the rule's colon was read, even if the rest of that line changes it.

kati should behave the same as make in normal mode. Using the same method that run_test.go uses to compare them. If make succeeds or fails kati should too. If kati currently behaves differently without .RECIPEPREFIX then that's out of scope, like kati stopping on errors in recipes that make never runs.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/google/kati repo && cd repo
git checkout 7a3ee413e1cfc230a597ac33034d2f478351e432
cp ../Dockerfile . && git apply ../test.patch
docker build -t kati-recipe-prefix .
docker run --rm kati-recipe-prefix ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t kati-recipe-prefix .
docker run --rm kati-recipe-prefix ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm kati-recipe-prefix ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
