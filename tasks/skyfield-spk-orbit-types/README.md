# Load SPK type 5 and type 15 orbit segments

| | |
|---|---|
| **Project** | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) (MIT) |
| **Base commit** | [`31a6ed7584`](https://github.com/skyfielders/python-skyfield/tree/31a6ed7584cd09b6e48e07546a4b5921e1cd5bb4) |
| **Language** | Python |
| **Reference solution** | +149 / -22 lines across 2 files |
| **New tests** | 55 test functions, +817 lines in 3 test files |

## Problem

Skyfield refuses SPK segments of types 5 and 15. Load them.

New segments work wherever types 2 and 3 do, raise the same `EphemerisRangeError` outside their dates, and match SPICE within 1 cm and 1e-8 km/s for every time. Records use km, seconds, radians and TDB seconds past J2000; "propagate" means two-body motion.

Type 5 stores N states, their N epochs, and ends with GM and N. At or before the first epoch, or after the last, propagate that state. Otherwise propagate the states at the last epoch `t1` before `t` and the next, `t2`, and blend with `w=(1+cos(pi*(t-t1)/(t2-t1)))/2`: position `w*p1+(1-w)*p2`, velocity `w*v1+(1-w)*v2+w'*(p1-p2)`.

Type 15 stores 16 numbers: periapsis epoch, orbit pole, periapsis direction, semi-latus rectum `p`, eccentricity `e`, J2 flag, body pole, GM, J2 and equatorial radius `R`. Propagate the periapsis state: `p/(1+e)` along the periapsis direction, moving at `sqrt(GM/p)*(1+e)` along orbit pole x periapsis. If `e<1`, J2 is nonzero and `p/(1+e)>R`, take `u`, the signed true anomaly since the periapsis epoch with whole orbits, `c`, the poles' dot product, and `z=1.5*J2*u*(R/p)**2`. Rotate position and velocity about the orbit pole by `z*(2.5*c*c-0.5)` unless the flag is 1 or 3, then about the body pole by `-z*c` unless it is 2 or 3.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/skyfielders/python-skyfield repo && cd repo
git checkout 31a6ed7584cd09b6e48e07546a4b5921e1cd5bb4
cp ../Dockerfile . && git apply ../test.patch
docker build -t skyfield-spk-orbit-types .
docker run --rm skyfield-spk-orbit-types ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t skyfield-spk-orbit-types .
docker run --rm skyfield-spk-orbit-types ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm skyfield-spk-orbit-types ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
