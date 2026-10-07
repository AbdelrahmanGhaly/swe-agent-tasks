# Resolve pickups and solves in dependency order

| | |
|---|---|
| **Project** | [optiland/optiland](https://github.com/optiland/optiland) (MIT) |
| **Base commit** | [`d531321025`](https://github.com/optiland/optiland/tree/d531321025521622efe77c3b81611ebfb595646d) |
| **Language** | Python |
| **Reference solution** | +1,239 / -17 lines across 9 files |
| **New tests** | 125 test functions, +1,344 lines in 1 test file |

## Problem

A pickup copies a value between two places through a scale and offset; a solve drives a parameter until a paraxial condition holds. Either may read what the other writes.

`update()` applies each pickup and each solve once, each after everything it reads. When the call returns, every value a pickup settles equals its scale times its source plus its offset, and a marginal ray height or angle solve sits on its target. Independent pickups and solves keep the order they were added, pickups ahead of solves. Where two write one value the later addition wins, pickup and solve alike. Writing the optic out and reading it back keeps that answer. `optic.pickups.apply()` and `optic.solves.apply()` order their own lists the same way.

A pickup names a value by short name or path: `radius`, `surfaces[2].geometry.radius` and `surfaces.surfaces[2].geometry.radius` are one value, and `conic` and `material` alias their paths the same way. `material` is the glass filling the space after a surface. The same attribute on two optics is two values, so configurations are never interchangeable.

The space after a surface is a single value: reading it reads the distance to the next surface, and a solve that resizes it leaves every other space alone. A space's recorded size follows the surface it belongs to, so a later thickness write does not undo a solve, the space in front of the first surface included.

A ray height or ray angle solve is settled by the surfaces, spaces and glasses the ray crosses on the way in to the surface it is aimed at, never that surface's own radius or the space and glass it opens. Both are aimed at their own surface, and an angle solve also refracts into the glass beyond it. A height and an angle solve on one surface are not circular, nor are two height solves on different surfaces. A chief ray is aimed at the aperture stop, so a chief ray solve reads at least that far, though its own surface stays excluded just the same. Adding a pickup or a solve never raises. Anything that reads a value it also writes, directly or around a loop, is rejected with a `ValueError` when the prescription is resolved, and nothing is applied.

`optic.updater.resolution_order()` gives the pickups and solves in the order they run. `optic.updater.dependency_report()` pairs each of them, in that order, with a list of the earlier ones that write a value it reads or writes. `optic.updater.driven_values()` maps each value the prescription sets for itself, by name, to the pickup or solve that settles it, in that order. A value with several writers takes its last writer's place. A named surface property reads as `radius[2]`, `conic[2]`, `thickness[2]` or `material[2]`, anything else as its path. `MultiConfiguration.update_all()` updates every configuration after those its pickups read from. `configuration_order()` returns the configurations themselves, not their positions, keeping the order they were added when none reads another; a circle among them is rejected the same way.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/optiland/optiland repo && cd repo
git checkout d531321025521622efe77c3b81611ebfb595646d
cp ../Dockerfile . && git apply ../test.patch
docker build -t optiland-constraint-resolution .
docker run --rm optiland-constraint-resolution ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t optiland-constraint-resolution .
docker run --rm optiland-constraint-resolution ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm optiland-constraint-resolution ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
