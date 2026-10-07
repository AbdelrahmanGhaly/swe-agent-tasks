# Add per-partition version election to version groups

| | |
|---|---|
| **Project** | [JamieMason/syncpack](https://github.com/JamieMason/syncpack) (MIT) |
| **Base commit** | [`958d30689a`](https://github.com/JamieMason/syncpack/tree/958d30689ac24b60623258630242330bd6d0264b) |
| **Language** | Rust |
| **Reference solution** | +734 / -55 lines across 25 files |
| **New tests** | 101 test functions, +2,920 lines in 119 test files |

## Problem

A version group that elects a preferred version compares every instance of a dependency, so a monorepo holding two majors reports the older as a mismatch. Add a `partitionBy` option to those groups. It divides a dependency's instances before the election, and each partition elects its own preferred version, so one partition's winner never marks another's as a mismatch. One exemption overrides this: a dependency that is itself a package in this monorepo is never partitioned, even when its version is comparable or a group's filters leave the package itself out; its instances compare exactly as today.

`partitionBy` takes `major`, `minor`, `releaseChannel`, or a list of semver ranges. A major partition is the specifier's major version, a minor partition its major and minor, a release channel the version's first prerelease identifier, or `stable` when absent. A list of ranges puts each instance in the first range it satisfies; one satisfying none is unpartitioned. A git url carries its version in a `#semver:` selector, even a shorthand such as `^1.2` or an exact `=1.2.3`. A fix that gives a git url a new version keeps its `#semver:` selector; an election writes the winner's as it stands. Not every specifier carries a comparable version, and `latest` never does however it is wrapped. Those instances are not partitioned either. They stay under the dependency's own name, reported exactly as today. The same applies under `minor` to a specifier pinning only a major, naming no single minor.

`syncpack json` reports each instance's partition under a new `partition` key: the partition's own key as a string, never an internal name, or null when unpartitioned. `syncpack list` heads each partition's row with the dependency name and partition key, so `zod` split on its major reads `zod 3` and `zod 4`. Partitions of one dependency are reported in version order, comparing digit runs as numbers so 9 precedes 10 and 5.9 precedes 5.10. Range partitions instead keep the order they were configured in. Unpartitioned instances come after that dependency's partitions.

A group may also set `maxPartitions`, the most partitions one dependency may split into. A dependency over that limit cannot be resolved automatically, so each of its instances, including unpartitioned ones, is reported `PartitionLimitExceeded` and keeps its specifier. Unpartitioned instances do not count.

Reject configuration syncpack cannot act on: an unknown `partitionBy` dimension, a range that does not parse, an empty range list, a `maxPartitions` of zero or without a `partitionBy`, and a `partitionBy` on a group that elects no preferred version. A `sameMinor` group elects only when given a `preferVersion`, then partitions like any other. A group without `partitionBy` keeps exactly the behaviour it has today, including how it compares `workspace:` specifiers, apart from the git selectors described above.

The published types and schema come from `npm/syncpack.ts`, which gains the same surface: `partitionBy` and `maxPartitions` on the groups that accept them, `partition` on the JSON output, and `PartitionLimitExceeded` among the status codes.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/JamieMason/syncpack repo && cd repo
git checkout 958d30689ac24b60623258630242330bd6d0264b
cp ../Dockerfile . && git apply ../test.patch
docker build -t syncpack-partitioned-version-election .
docker run --rm syncpack-partitioned-version-election ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t syncpack-partitioned-version-election .
docker run --rm syncpack-partitioned-version-election ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm syncpack-partitioned-version-election ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
