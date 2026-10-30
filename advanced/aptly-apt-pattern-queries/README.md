# Add apt pattern support to aptly package queries

| | |
|---|---|
| **Project** | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) (MIT) |
| **Base commit** | [`516fcc6bfd`](https://github.com/aptly-dev/aptly/tree/516fcc6bfd08c46e1cd35e5395a04e82bc3ea4fe) |
| **Language** | Go |
| **Reference solution** | +2,265 / -0 lines across 8 files |
| **New tests** | +1,468 lines in 1 test file |

## Problem

aptly queries for packages have their own language. So if you know apt you will have to convert things like `?name(...)`, `~nfoo` or `?depends(...)` yourself. It would be nice if aptly accepted apt patterns. If a package query (including mirror filter queries) begins with a `?` or `~` it should be treated as an apt pattern. All other queries behave as they do today.

The pattern should match exactly the package versions that apt 2.6 (in the image) will output for it with `apt list --all-versions` given that the packages being queried are the only source apt knows about, on a system that is amd64 and has all architectures the packages are available on enabled and no packages are installed. Since apt does not list source packages a pattern will never select one. If apt gives an error about the pattern the command should fail.

For simplicity assume no patterns will need release info (`?archive`, `?codename` or `?origin`). Also assume no package relationships will contain architecture qualifiers.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/aptly-dev/aptly repo && cd repo
git checkout 516fcc6bfd08c46e1cd35e5395a04e82bc3ea4fe
cp ../Dockerfile . && git apply ../test.patch
docker build -t aptly-apt-pattern-queries .
docker run --rm aptly-apt-pattern-queries ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t aptly-apt-pattern-queries .
docker run --rm aptly-apt-pattern-queries ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm aptly-apt-pattern-queries ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
