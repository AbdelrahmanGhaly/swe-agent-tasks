# Support planetodetic sites and local skies on other bodies

| | |
|---|---|
| **Project** | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) (MIT) |
| **Base commit** | [`31a6ed7584`](https://github.com/skyfielders/python-skyfield/tree/31a6ed7584cd09b6e48e07546a4b5921e1cd5bb4) |
| **Language** | Python |
| **Reference solution** | +313 / -57 lines across 5 files |
| **New tests** | 46 test functions, +1,327 lines in 4 test files |

## Problem

`PlanetaryConstants.build_latlon_degrees()` should allow any 3 positive `BODY<id>_RADII` (x,y,z of the frame) with planetodetic lat/lon. The lat and east lon define the outward surface normal `(cos(lat)cos(lon), cos(lat)sin(lon), sin(lat))` and the site will be located `elevation_m` from the surface point with that normal, in its direction. Spheres keep the current sites. Sites will have `latitude` and `longitude` and `elevation` added as a Distance.

PlanetaryConstants should have functions `latlon_of()`, `height_of()`, `subpoint_of()` and `planetodetic_position_of()` that take a `(frame, position)` similar to the `wgs84` versions (last one similar to `geographic_position_of()`) but will be measured from closest point on the surface. Longitude will be between -180 and 180 and height will be negative if within the body. Should accept arrays of times and throw a ValueError unless the position is centered on the frame's center. These should work for any location inside or outside the body no matter what order the axes are in. If several surface points are equally close, any can be used. Height will still be accurate.

These sites should work with `altaz()`, where the normal will be zenith and north will be the body's z axis projected onto the horizon. `hadec()` will measure declination from the equator of the body and hour angle from the meridian of the site. `frame_xyz_and_velocity()` and `frame_latlon_and_rates()` should account for the rotation of the body for both frames and sites.

`find_risings()` and `find_settings()` should be able to be called from these sites and return all intersections with the usual horizon marked as True. `find_transits()` should return all intersections with the meridian going west. Even if the body rotates in hours and the target never circles the sky (Earth from the moon). `sunrise_sunset()`, `dark_twilight_day()`, `risings_and_settings()` and `meridian_transits()` should all take these sites and use the body from the ephemeris.

Frame kernels should be able to use other TK frames. Can be defined in relation to `J2000`. Use a SPICE `QUATERNION` from `TKFRAME_<frame>_Q` (normalized like SPICE). Angles can be in any SPICE unit: `RADIANS`, `DEGREES`, `ARCMINUTES`, `ARCSECONDS`, `HOURANGLE` (15 degrees), `MINUTEANGLE` or `SECONDANGLE`. TK variables can use the frame name instead of its ID (`TKFRAME_<name>_SPEC` and so on). PCK frames should use `FRAME_<id>_CLASS_ID` to find the segment.

Should be within 1mm of SPICE results, 1e-9 km/s, 1e-8 deg (or /day), 1e-10 for rotation matrices and 1 second for rise/set/transit.

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
docker build -t skyfield-planetodetic-surfaces .
docker run --rm skyfield-planetodetic-surfaces ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t skyfield-planetodetic-surfaces .
docker run --rm skyfield-planetodetic-surfaces ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm skyfield-planetodetic-surfaces ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
