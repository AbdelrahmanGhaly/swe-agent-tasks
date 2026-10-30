# Hand off leadership when a raft leader removes itself

| | |
|---|---|
| **Project** | [etcd-io/raft](https://github.com/etcd-io/raft) (Apache-2.0) |
| **Base commit** | [`638f42739b`](https://github.com/etcd-io/raft/tree/638f42739b77572b5708eb0d8a776a272ade71ed) |
| **Language** | Go |
| **Reference solution** | +57 / -8 lines across 2 files |
| **New tests** | +1,114 lines in 3 test files |

## Problem

If we remove a leaders vote by either removing it or setting it to a learner we want it to step down from being leader. Currently it will either remain leader or if StepDownOnRemoval is set it will step down immediately. The other voters will have no leader until one of them times out on an election. We can add TransferLeadershipOnRemoval bool to Config. If set when a leader removes itself from the configuration it will choose which voter to give leadership to. It will pick the voter that has the most of its log committed (lowest id in case of tie). It will not pick learners. The new leader should be able to take over without any node timing out on an election. And should be able to take over with all entries in the old leaders log. The leader will stop proposing while it gives leadership to another node. It will step down one election timeout after it applies the config change (ElectionTick ticks). Unless it hears about a new leader before then. This will happen with or without StepDownOnRemoval. If TransferLeadershipOnRemoval is not set and a leader removes itself it will act as it does now.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/etcd-io/raft repo && cd repo
git checkout 638f42739b77572b5708eb0d8a776a272ade71ed
cp ../Dockerfile . && git apply ../test.patch
docker build -t raft-removal-handoff .
docker run --rm raft-removal-handoff ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t raft-removal-handoff .
docker run --rm raft-removal-handoff ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm raft-removal-handoff ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
