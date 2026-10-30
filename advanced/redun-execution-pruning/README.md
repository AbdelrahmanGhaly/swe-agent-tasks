# Add execution pruning to the redun repository database

| | |
|---|---|
| **Project** | [insitro/redun](https://github.com/insitro/redun) (Apache-2.0) |
| **Base commit** | [`49a299b223`](https://github.com/insitro/redun/tree/49a299b223bc345b999aaa40daa6876f105089e1) |
| **Language** | Python |
| **Reference solution** | +764 / -4 lines across 5 files |
| **New tests** | +1,399 lines in 1 test file |

## Problem

The redun repository will only accumulate data. Every time you run something it adds the execution and all of its jobs, callnodes, values and tags. It will never remove them. I propose adding a `RedunBackendDb.prune(execution_ids)` function to delete given executions and all associated data that only exists because of these executions. Records pulled in from other repositories are out of scope.

Once a prune has happened if there is no other execution that called the same taskcall, directly or through a cached call that made it, it will need to be recalculated the next time its used. That holds even when the taskcall's record has to stay because a remaining execution received its result as an argument, and also for tasks with `check_valid="shallow"`.

Some records are shared between executions using hashes. All records used by remaining executions, including their tags, should stay unchanged, and if you recalculate a remaining execution's workflow it will hit the cache and get the same result it would have before you deleted any executions.

You should not be able to tell that an execution ever existed once you prune it, and that includes every handle state it recorded, even a forked one that no call refers to. You can assume a task never passes the same handle more than once, that every handle fork has a non-empty key and that no task calls `merge_handles` or turns provenance off with `prov=False`. Pruning all executions should result in every table of the repository, and the value store if the backend has one, being empty except for records that don't belong to any execution. Except for cache entries that don't come from a finished call, like the `catch` cache. Those should not be affected by prune since they are not execution specific.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/insitro/redun repo && cd repo
git checkout 49a299b223bc345b999aaa40daa6876f105089e1
cp ../Dockerfile . && git apply ../test.patch
docker build -t redun-execution-pruning .
docker run --rm redun-execution-pruning ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t redun-execution-pruning .
docker run --rm redun-execution-pruning ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm redun-execution-pruning ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
