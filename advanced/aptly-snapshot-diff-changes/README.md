# Show changelog entries in aptly snapshot diff

| | |
|---|---|
| **Project** | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) (MIT) |
| **Base commit** | [`516fcc6bfd`](https://github.com/aptly-dev/aptly/tree/516fcc6bfd08c46e1cd35e5395a04e82bc3ea4fe) |
| **Language** | Go |
| **Reference solution** | +1,269 / -32 lines across 10 files |
| **New tests** | +1,214 lines in 1 test file |

## Problem

`aptly snapshot diff` should have a `-changes` option that will show you, under every package that exists in both snapshots but with different versions, what `dpkg-parsechangelog -l <changelog> --since <older version> -S Changes` would output for the newer package's changelog. Newer and older based on dpkg's version ordering. So the snapshot diff API would accept `changes=1` and return every such entry with a `Changes` field containing this text or empty string if nothing was shown.

The changelog for a binary package would be `usr/share/doc/<package>/changelog.Debian.gz` inside its .deb or `changelog.gz` in that directory if no debian one exists. If there is also a `changelog.Debian.<architecture>.gz` for the package's architecture in that directory its changelog is that file followed by the other one. Following any symlinks on the way. If those point to a `usr/share/doc/<name>` directory that the package does not include we take the file from the package named `<name>` in the same snapshot with the same version and either the same architecture or `all`.

For source packages their changelog is `debian/changelog` after `dpkg-source -x` extracted it. If a package does not have a changelog nothing is shown. We can assume changelogs are valid and source packages use one of format 1.0, 3.0 (quilt) or 3.0 (native) and quilt does not change `debian/changelog`. All of this has to happen inside aptly, without running any dpkg tools.

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
docker build -t aptly-snapshot-diff-changes .
docker run --rm aptly-snapshot-diff-changes ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t aptly-snapshot-diff-changes .
docker run --rm aptly-snapshot-diff-changes ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm aptly-snapshot-diff-changes ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
