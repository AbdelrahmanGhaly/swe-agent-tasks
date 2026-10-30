# Chain implicit rules in kati the way GNU make does

| | |
|---|---|
| **Project** | [google/kati](https://github.com/google/kati) (Apache-2.0) |
| **Base commit** | [`55ea1e26f4`](https://github.com/google/kati/tree/55ea1e26f4da6d52a4ddca8f36a130dc26120525) |
| **Language** | C++ and Rust |
| **Reference solution** | +1,561 / -144 lines across 8 files |
| **New tests** | +1,491 lines in 80 test files |

## Problem

kati will only use a pattern rule if all of its prerequisites either exist or have a rule. It will never use a second pattern rule to create a prerequisite. If you have a makefile that relies on these it will error out with No rule to make target (#245). Also .INTERMEDIATE, .SECONDARY and .PRECIOUS are not supported by kati. Have ckati and rkati support implicit rules and intermediates like make 4.2.1 does. This includes chaining pattern rules and supporting .INTERMEDIATE, .SECONDARY and .PRECIOUS. See make's documentation for more information about "Chains of Implicit Rules" and "Implicit Rule Search Algorithm".

make should be installed as make. When given the same makefile and goals kati should produce the same output as make (with or without -s). It should also pass/fail when make passes/fails and produce the same set of files. The output should be the same as what the testcases compare when comparing make and kati. Except that the order of the files after rm can be in any order.

Suffix rules, vpath, .SECONDEXPANSION and make's builtin rules are not supported. --ninja is also not supported.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/google/kati repo && cd repo
git checkout 55ea1e26f4da6d52a4ddca8f36a130dc26120525
cp ../Dockerfile . && git apply ../test.patch
docker build -t kati-implicit-rule-chains .
docker run --rm kati-implicit-rule-chains ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t kati-implicit-rule-chains .
docker run --rm kati-implicit-rule-chains ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm kati-implicit-rule-chains ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
