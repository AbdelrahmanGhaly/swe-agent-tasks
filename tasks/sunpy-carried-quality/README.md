# Carry a map's mask and uncertainty through its operations

| | |
|---|---|
| **Project** | [sunpy/sunpy](https://github.com/sunpy/sunpy) (BSD-2-Clause) |
| **Base commit** | [`b9fb3034f5`](https://github.com/sunpy/sunpy/tree/b9fb3034f5d5ada85aff6eac17f0cc1625ad5191) |
| **Language** | Python |
| **Reference solution** | +776 / -29 lines across 4 files |
| **New tests** | 109 test functions, +1,309 lines in 1 test file |

## Problem

Carry a map's mask and uncertainty through `submap`, `resample`, `rotate`, `superpixel`, `shift_reference_coord`, `reproject_to`, and arithmetic with a number, a quantity or an array. An array broadcasting wider than the map still gives a two-dimensional map with a matching mask and uncertainty.

Resampling, rotating and reprojecting interpolate from unmasked pixels only. Partial support is renormalised, so a masked pixel neither contributes a value nor dilutes its neighbours. An output pixel with no unmasked support beyond rounding error is masked. Its data takes the operation's missing value if there is one, and NaN otherwise. When a map with a mask or an uncertainty is rotated, the space outside the original frame counts as masked. When one is reprojected, output pixels no input pixel reached are masked. Reprojection returns its footprint when asked and fills a caller's output array, taking its shape when the target names none.

An uncertainty is interpolated as a variance over the same unmasked support as the data. An interpolated variance below zero is clamped to zero. Resampling, rotating, reprojecting and superpixelling each work out their own mask, whose pixels all get a NaN uncertainty. Submap, a reference shift and arithmetic carry the mask and leave every uncertainty value unchanged.

A superpixel that sums its pixels adds the variances of the unmasked ones. One that averages them divides that sum by the square of the unmasked count. A reduction counts as a sum or mean only if it returns exactly the sum or mean of blocks of the shape asked for: `np.sum`, `np.mean`, their masked-array and NaN-aware versions, and a wrapper around either all qualify. Other reductions, such as `np.min`, `np.max`, `np.std`, a 3 by 3 median, or one that sums at one block shape and not another, keep no uncertainty.

An uncertainty keeps its kind, be it `StdDevUncertainty`, `VarianceUncertainty` or `InverseVariance`. The unit it reports follows the data's, squared for a variance and inverse-squared for an inverse variance. If an operation's result has no unit, neither does its uncertainty. Scaling a map scales the uncertainty by the size of the factor. Adding to, subtracting from or negating a map leaves it alone. Raising a map to a power, or dividing a number by a map, leaves no uncertainty. The mask survives all of them.

A `MapSequence` stacks its maps' uncertainties as it stacks their masks, reporting the stack as an uncertainty of their kind. `MapSequence.uncertainty` is `None` when no map in it has one, and otherwise a map without one contributes a layer of NaN. Only a map with a mask or an uncertainty gets those rotation and reprojection masks, and no other operation adds a mask or an uncertainty a map lacked. A map with neither keeps both as `None` and gets the same data and metadata as before.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/sunpy/sunpy repo && cd repo
git checkout b9fb3034f5d5ada85aff6eac17f0cc1625ad5191
cp ../Dockerfile . && git apply ../test.patch
docker build -t sunpy-carried-quality .
docker run --rm sunpy-carried-quality ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t sunpy-carried-quality .
docker run --rm sunpy-carried-quality ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm sunpy-carried-quality ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
