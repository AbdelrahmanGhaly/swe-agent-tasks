# Add conditional breakpoints to circuit simulations

| | |
|---|---|
| **Project** | [tilk/digitaljs](https://github.com/tilk/digitaljs) (BSD-2-Clause) |
| **Base commit** | [`a77a6b3a91`](https://github.com/tilk/digitaljs/tree/a77a6b3a916996e215e6c33c75ffe05dc6f55e3d) |
| **Language** | JavaScript |
| **Reference solution** | +309 / -0 lines across 6 files |
| **New tests** | 25 test functions, +619 lines in 3 test files |

## Problem

Add `addBreakpoint(condition)` and `removeBreakpoint(id)` to `HeadlessCircuit` and therefore to `Circuit`. `addBreakpoint` will return an id number. When the breakpoint is triggered the circuit will emit a `breakpoint` event with the id and what tick it was triggered on (after, the tick that just ran). If you remove an id that doesn't exist nothing happens.

Conditions are strings. You can specify a port on a device with `dev:port`. To access ports on subcircuits use the label of the subcircuit `sub.dev:port`. If no port is specified the name refers to the output of an Input device, the input of an Output device or the only output of any other device. `name[i]` will refer to bit i where bit 0 is the lowest bit. `name[i: j]` will refer to bits i through j.

You can compare a name or selection to a binary literal of the same size using `==` or `!=`. Example: `'b10x1`. x will match any value. If a bit on the signal is undefined it will not be equal to 0 or 1. `rose(name)` and `fell(name)` will be true if the name or selection is one bit and changed from 0 to 1 or 1 to 0 respectively since the last time the breakpoint was checked. This counts every check, even when `&&` or `||` skips it.

You can combine conditions with `!`, `&&`, `||` and (). They follow the normal order of operations. If the device or port does not exist, the name is ambiguous (like two devices sharing a label), the selection is out of bounds or the width is incorrect or there is a syntax error it will throw and nothing will be added.

A breakpoint is checked when it is added and after every tick that the simulation steps through, even ticks where nothing happens. The values it checks are at the end of that tick. A breakpoint will trigger after a tick if its condition is met and it wasn't met when it was last checked. If the simulation is running it will stop right after that tick and no more ticks run. If stepping it will continue to step. If multiple breakpoints trigger after the same tick they will trigger in the order they were added. This all works across all engines and even if the device is in a subcircuit that isn't being observed.

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
docker build -t digitaljs-conditional-breakpoints .
docker run --rm digitaljs-conditional-breakpoints ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t digitaljs-conditional-breakpoints .
docker run --rm digitaljs-conditional-breakpoints ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm digitaljs-conditional-breakpoints ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
