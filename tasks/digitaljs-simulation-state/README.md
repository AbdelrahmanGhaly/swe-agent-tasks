# Add saving, restoring and live engine switching for circuit simulations

| | |
|---|---|
| **Project** | [tilk/digitaljs](https://github.com/tilk/digitaljs) (BSD-2-Clause) |
| **Base commit** | [`a77a6b3a91`](https://github.com/tilk/digitaljs/tree/a77a6b3a916996e215e6c33c75ffe05dc6f55e3d) |
| **Language** | JavaScript |
| **Reference solution** | +450 / -25 lines across 9 files |
| **New tests** | 44 test functions, +1,442 lines in 3 test files |

## Problem

We want `HeadlessCircuit` and by extension `Circuit` to store a state and resume from it. `saveState()` will resolve to a plain object with the current simulation state. `loadState(state)` will resolve once the state is loaded, and after a `synchronize()` the circuit is back at that state. State is not altered when saving and should survive `JSON.stringify` and `JSON.parse`. A state can restore the original circuit or be used on any identical circuit, whatever engines they use. A `state` option for the constructor creates a circuit that is at the given state straight away, without needing a `synchronize()`.

Same tick, same ports on all devices at all levels of subcircuits. It should have the same memory contents and automaton states. And produce the same port outputs from that point forward given the same inputs. It should not cause the circuit to simulate a tick or anything to be evaluated or triggered. Monitors keep firing and alarms trigger once the simulation gets to their tick. Even if a previous loadState skipped over an alarm a future loadState will trigger it.

States should be compatible with any circuit that has the same devices (id's at all levels), device types and port widths. If `loadState` is given a state that isn't compatible it will reject. Creating a circuit with a state option that doesn't match will throw. In both cases the circuit should be unchanged.

`switchEngine(engine, engineOptions)` will switch the circuit to use a different engine and resolve once complete. The circuit's state should stay the same. If it is simulating it should continue at the same interval if the engine allows it to run autonomously. It should keep calling any functions that use the circuits `postUpdateGates`. Any input that was set before switching should still work. Monitors and waits don't have to see an input change made right before switching. Any graph you observe using `observeGraph` should work.

Any monitor wait or alarm that was added/removed should continue to work after you switch. This includes an alarm due right after a pending evaluation, which should go off without stopping the simulation. Any ids returned by adding monitors should still work when using `unmonitor`, even for a once shot monitor that has only reported its current value. Once shot monitors should not re-trigger and no monitors should trigger from switching.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/tilk/digitaljs repo && cd repo
git checkout a77a6b3a916996e215e6c33c75ffe05dc6f55e3d
cp ../Dockerfile . && git apply ../test.patch
docker build -t digitaljs-simulation-state .
docker run --rm digitaljs-simulation-state ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t digitaljs-simulation-state .
docker run --rm digitaljs-simulation-state ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm digitaljs-simulation-state ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
