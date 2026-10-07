# Honor via-way turn restrictions in map creation and routing

| | |
|---|---|
| **Project** | [abrensch/brouter](https://github.com/abrensch/brouter) (MIT) |
| **Base commit** | [`2f4bcb80d2`](https://github.com/abrensch/brouter/tree/2f4bcb80d2e28342f282224bf2c6cf741b390553) |
| **Language** | Java |
| **Reference solution** | +580 / -14 lines across 17 files |
| **New tests** | 26 test functions, +792 lines in 2 test files |

## Problem

BRouter only understands turn restrictions whose via member is a single node. A `type=restriction` relation whose via members are ways, for example a `no_u_turn` across the short connector between two carriageways, is dropped while the segment files are built, so routes drive straight through it. Support these relations end to end: map creation has to keep them in the rd5 segments and the router has to obey them.

Such a relation has exactly one from way, exactly one to way and one or more via ways, listed in any order, that form a chain joining an end of the from way to an end of the to way. Relations that do not form such a chain are ignored. A chain may cross the borders between the 5x5 degree rd5 tiles. The restriction concerns a route that reaches the chain along the from way and then drives every via way in chain order to the far end of the chain. There, a `no_*` restriction forbids continuing onto the to way and an `only_*` restriction forbids continuing onto any other way. Leaving the chain before its end, driving it in the opposite direction, or entering it from any other way is not affected.

The `except` tag, the vehicle kind of the profile and the `considerTurnRestrictions` switch treat these restrictions exactly as they treat via-node restrictions. A restriction removes nothing but the forbidden maneuver: the router still returns the cheapest route that avoids it, including routes that use the from way, the via ways or the to way in a permitted way. Via-node restrictions keep working as they do today, also at a junction where a via-way restriction ends, and routes come out the same when the JVM runs with `-DdisableDirectWeaving=true`, which decodes segments through the micro-cache instead of weaving them straight into the routing graph. Profiles that switch on `inverseRouting` are out of scope.

Map creation must also apply turn restrictions no matter which directory the process runs in. Today one of the cutting stages looks for its intermediate restriction files relative to the working directory rather than next to the tile directories it was given, so map creation started anywhere else silently loses every turn restriction.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/abrensch/brouter repo && cd repo
git checkout 2f4bcb80d2e28342f282224bf2c6cf741b390553
cp ../Dockerfile . && git apply ../test.patch
docker build -t brouter-via-way-restrictions .
docker run --rm brouter-via-way-restrictions ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t brouter-via-way-restrictions .
docker run --rm brouter-via-way-restrictions ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm brouter-via-way-restrictions ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
