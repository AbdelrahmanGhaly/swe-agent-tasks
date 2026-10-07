# Make directory targets rebuild and cache like file targets

| | |
|---|---|
| **Project** | [SCons/scons](https://github.com/SCons/scons) (MIT) |
| **Base commit** | [`7afde3bd9f`](https://github.com/SCons/scons/tree/7afde3bd9f9930a10468f4199ecb86e686c45d3c) |
| **Language** | Python |
| **Reference solution** | +410 / -8 lines across 3 files |
| **New tests** | 131 test functions, +1,692 lines in 21 test files |

## Problem

When a builder's target is a directory, for example `env.Command(Dir('out'), 'in.txt', action)`, SCons only checks that the directory exists. Editing a source or changing the action leaves the old output in place, `--debug=explain` gives no reason, targets that use the directory never notice its files changing, and CacheDir never stores or restores it.

A directory built by a builder should be rebuilt when one of its sources, an explicit dependency, a dependency that a scanner finds for its sources (such as the files `DirScanner` lists in a source directory) or its build action changed since it was last built, or when it is missing, and `--debug=explain` should say why just as it does for a file. The files the builder wrote inside the directory are its output, not its dependencies, so changing or deleting them by hand does not by itself cause a rebuild.

A target that uses a built directory as a source should look at what the directory holds, the way it looks at a file source's contents: it is rebuilt when a file inside the directory was added, removed or changed since that target was last built, including by hand, and not when the directory was rebuilt but ended up with the same files.

With CacheDir enabled, a built directory is pushed as a whole tree and, when the same build is needed again, restored instead of running the builder, reported as "Retrieved `out' from cache", or as the command under `--cache-show`. A build is the same only when its sources, explicit dependencies, scanned dependencies and action are all the same.

A restore replaces what the directory held before with the pushed tree: the files with their contents and permission bits, empty subdirectories, and symbolic links kept as links. Files inside the directory that belong to other targets are left as those targets built them. Targets that read files inside a restored directory work in the same run, and the next run is up to date. `NoCache`, `--cache-readonly` and `--cache-force` treat directories the way they treat files.

File targets, directories that only exist because other targets are built inside them, and `scons -c`, which leaves directories in place, keep their current behavior.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/SCons/scons repo && cd repo
git checkout 7afde3bd9f9930a10468f4199ecb86e686c45d3c
cp ../Dockerfile . && git apply ../test.patch
docker build -t scons-cachedir-directory-targets .
docker run --rm scons-cachedir-directory-targets ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t scons-cachedir-directory-targets .
docker run --rm scons-cachedir-directory-targets ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm scons-cachedir-directory-targets ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
