# Add force and release to circuit simulations

| | |
|---|---|
| **Project** | [tilk/digitaljs](https://github.com/tilk/digitaljs) (BSD-2-Clause) |
| **Base commit** | [`a77a6b3a91`](https://github.com/tilk/digitaljs/tree/a77a6b3a916996e215e6c33c75ffe05dc6f55e3d) |
| **Language** | JavaScript |
| **Reference solution** | +117 / -10 lines across 6 files |
| **New tests** | +964 lines in 4 test files |

## Problem

Add force(cell,target,value,mask) and release(cell,target,mask) to HeadlessCircuit therefore Circuit. cell can be any device, even in subcircuits, target is a port. value and the optional mask are a Vector3vl the length of target with the 1s in the mask selecting which bits to use (defaults to all).

When you force a bit it stays that value until released. It adds to already forced bits (if a bit is forced again it takes the new value). Devices and monitors connected to a forced output bit see the value. Devices see the value on forced input bits. The forced device will operate normally under it. When released it will go to whatever value it should have.

This occurs on the current tick just like setInput. Monitors will show what is on the output now, but not changes masked by a force. This is true for all engines including the main thread of a worker circuit after synchronize(), where a release also shows changes the force hid.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/tilk/digitaljs repo && cd repo
git checkout a77a6b3a916996e215e6c33c75ffe05dc6f55e3d
cp ../Dockerfile . && git apply ../test.patch
docker build -t digitaljs-signal-forcing .
docker run --rm digitaljs-signal-forcing ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t digitaljs-signal-forcing .
docker run --rm digitaljs-signal-forcing ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm digitaljs-signal-forcing ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
