# Add weighted voting to raft quorums

| | |
|---|---|
| **Project** | [etcd-io/raft](https://github.com/etcd-io/raft) (Apache-2.0) |
| **Base commit** | [`638f42739b`](https://github.com/etcd-io/raft/tree/638f42739b77572b5708eb0d8a776a272ade71ed) |
| **Language** | Go |
| **Reference solution** | +231 / -12 lines across 8 files |
| **New tests** | +1,035 lines in 1 test file |

## Problem

Each voter has an equal vote in etcd raft. So any quorum will be greater than half of the voters. To allow for weights on voters we can add a `Config.VoterWeights map[uint64]uint64` to our config and `RawNode.SetVoterWeights(map[uint64]uint64) error` which will change the weights on the fly. If a voter is not found in the map their weight will be 1. A weight of 0 is considered an error. Just as if you gave NewRawNode a bad config value SetVoterWeights will return an error and make no changes.

Any group of voters who sum to greater than half of all the voters weight in the config is a quorum. Not equal to half just greater. Learners have no weight. This will apply to every quorum based decision raft makes. Joint configs will use weights on each side of the config and both sides will need a quorum. A leader with a weight of a quorum will respond to ReadIndex requests immediately similar to a leader that is the only voter.

Since weights will be associated with a node id they will always be applied to who ever is currently a voter.

Once you call SetVoterWeights your new weights take effect. If there is anything your new weights would cause to happen it will do so without receiving more information. Such as a leader committing entries (and replicating to its followers) and responding to read requests. Or a candidate/precandidate winning an election. If CheckQuorum is on, a leader will also look at which voters its last check found active, and if they are not a quorum with the new weights it steps down right away.

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
docker build -t raft-weighted-quorums .
docker run --rm raft-weighted-quorums ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t raft-weighted-quorums .
docker run --rm raft-weighted-quorums ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm raft-weighted-quorums ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
