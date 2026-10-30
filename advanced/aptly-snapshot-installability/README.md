# Add an installability check to aptly snapshot verify

| | |
|---|---|
| **Project** | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) (MIT) |
| **Base commit** | [`516fcc6bfd`](https://github.com/aptly-dev/aptly/tree/516fcc6bfd08c46e1cd35e5395a04e82bc3ea4fe) |
| **Language** | Go |
| **Reference solution** | +870 / -16 lines across 6 files |
| **New tests** | +1,825 lines in 1 test file |

## Problem

`aptly snapshot verify` will just show you what can't be met. Which means any snapshots that have packages that would not be able to be installed due to conflicts or unresolvable dependency tree's would pass. I want a `-installable` option that will tell you about these packages instead.

When this option is used every architecture that `verify` checks (specified by `-architectures` or the architectures of the binaries in the snapshot being verified if not specified) is treated as if it was the native architecture of a system that has all the other's marked as foreign. Packages can be installed from the verified snapshot and any other source snapshots you may pass. Only packages from the verified snapshot of the native architecture or `all` will be checked.

If dose3 7.0's `dose-distcheck` (provided by the image to diff against) says a package is installable with these packages using `--deb-native-arch` and `--deb-foreign-archs` to correspond and everything else default then the package is considered installable. This means versions are compared like `dpkg --compare-versions`. Every installation will also include an Essential package for each Essential package name. Assume no package relies on a case where dose3 reads a relationship differently from dpkg.

Will output `Uninstallable packages (N):` followed by one line per failure, prefixed with two spaces, in the format of `name_version_arch [native]` where native is the architecture whose check failed.

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
docker build -t aptly-snapshot-installability .
docker run --rm aptly-snapshot-installability ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t aptly-snapshot-installability .
docker run --rm aptly-snapshot-installability ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm aptly-snapshot-installability ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
