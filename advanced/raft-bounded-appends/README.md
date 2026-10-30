# Bound the log entries a raft Ready hands out for appending

| | |
|---|---|
| **Project** | [etcd-io/raft](https://github.com/etcd-io/raft) (Apache-2.0) |
| **Base commit** | [`638f42739b`](https://github.com/etcd-io/raft/tree/638f42739b77572b5708eb0d8a776a272ade71ed) |
| **Language** | Go |
| **Reference solution** | +243 / -50 lines across 5 files |
| **New tests** | +2,458 lines in 3 test files |

## Problem

You can now specify `MaxAppendSizePerReady uint64` in `Config`. If this value is 0 there is no maximum size. Each time we give you a Ready (or MsgStorageAppend if AsyncStorageWrites is true) we will try to give you as many entries from your log (in order) as possible up until this number of bytes.

The size of the entry is calculated the same as with MaxSizePerMsg. If a single entry is larger than this value it will be given to you by itself. Any other entries left in memory will be given to you on subsequent calls.

Your application should persist these entries when given to you. All other information a Ready/MsgStorageAppend gives you, including every message it asks you to send and every read state, can only depend on entries it has been given or ones before it. Messages that can not be sent without depending on future entries will not be dropped but will wait until those entries have been given to you. Vote and pre-vote responses and append rejections do not wait for them.

Leaders will not count their own entry towards commit until the entry has been persisted. Acknowledgements to a previous term leader or for snapshotted entries can be dropped. It will not be sent if a new leader took over and those entries have been overwritten before they were given to your application.

A node must be able to recover from storage at any point after your application has consumed a Ready/MsgStorageAppend.

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
docker build -t raft-bounded-appends .
docker run --rm raft-bounded-appends ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t raft-bounded-appends .
docker run --rm raft-bounded-appends ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm raft-bounded-appends ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
