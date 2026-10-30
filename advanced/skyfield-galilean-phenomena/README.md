# Add Galilean moon phenomena to Skyfield's almanac

| | |
|---|---|
| **Project** | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) (MIT) |
| **Base commit** | [`31a6ed7584`](https://github.com/skyfielders/python-skyfield/tree/31a6ed7584cd09b6e48e07546a4b5921e1cd5bb4) |
| **Language** | Python |
| **Reference solution** | +341 / -0 lines across 2 files |
| **New tests** | +767 lines in 1 test file |

## Problem

Add `skyfield.almanac.galilean_phenomena(eph, observer)`, where `observer` is a body such as Mars, Amalthea or `eph['earth'] + wgs84.latlon(...)`. Like the other almanac routines it returns a function of time for `find_discrete`. Counting Io, Europa, Ganymede and Callisto from 0, the function's value has bit `4*i + k` set while moon `i` is in state `k` for the observer, and its changes are accurate to 0.1 s.

Jupiter is an ellipsoid, radii 71492 km equatorial and 66854 km polar, north pole at RA 268.057, declination 64.495 degrees. The moons are balls of radii 1821.6, 1560.8, 2631.2 and 2410.3 km. Bodies are placed where they were when the light involved passed them, without aberration or deflection.

A moon's light comes evenly from the part of the disk it shows the observer where its surface faces the Sun. In state 0 the moon is occulted: Jupiter hides most of that light from the observer. In state 1 it is in transit: most of that light lies in front of Jupiter's disk. In state 2 it is eclipsed: most of that light would come from surface where Jupiter hides the Sun's center. In state 3 its shadow is in transit: the line from the Sun's center past the moon's center first strikes Jupiter where the observer sees.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/skyfielders/python-skyfield repo && cd repo
git checkout 31a6ed7584cd09b6e48e07546a4b5921e1cd5bb4
cp ../Dockerfile . && git apply ../test.patch
docker build -t skyfield-galilean-phenomena .
docker run --rm skyfield-galilean-phenomena ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t skyfield-galilean-phenomena .
docker run --rm skyfield-galilean-phenomena ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm skyfield-galilean-phenomena ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
