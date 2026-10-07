# Add greyordinate operations for CIFTI-2 images

| | |
|---|---|
| **Project** | [nipy/nibabel](https://github.com/nipy/nibabel) |
| **Base commit** | [`014fc4a952`](https://github.com/nipy/nibabel/tree/014fc4a952aa4cec4daa94384a8d63e81f71457a) |
| **Language** | Python |
| **Reference solution** | +546 / -0 lines across 3 files |
| **New tests** | 173 test functions, +1,587 lines in 1 test file |

## Problem

nibabel can manipulate the axes describing a CIFTI-2 matrix, but nothing applies them to whole images. Add `parcellate`, `merge_parcels`, `concat` and `split` to `nibabel.cifti2`, along with a `ParcelsAxis.from_greyordinate_labels` constructor. Each returns an image whose data and header agree, so a result written out and read back gives the same axes and values; a dimension an operation does not touch is unchanged.

`parcellate(img, parcels, axis=None, method='mean')` reduces the greyordinates a parcel covers to one value. Parcels arrive as a `ParcelsAxis` or a dense label image, and `method` is one of mean, sum, max or min, computed and held in double precision. `axis` may name a brain model dimension; otherwise every one is, and an image offering none is an error. A greyordinate belongs to a parcel when it is the same kind of element with the same vertex or voxel index, structure names compared and stored in canonical form. Each parcel aggregates the distinct greyordinates it selects. Members the image does not cover are ignored, but a parcel selecting nothing at all is an error.

`ParcelsAxis.from_greyordinate_labels(brain_model, labels, label_table)` turns one label key per greyordinate and a `LabelAxis`-style table into parcels. Keys are taken in ascending order, each becoming a parcel named after the table, so every key carried must appear there. A key named `???` marks unlabeled greyordinates and forms no parcel, and neither does a key no greyordinate carries. The parcels it builds describe the same surfaces and volume as the brain model they came from. A label image given to `parcellate` goes through this constructor and must hold exactly one map.

`merge_parcels(img, groups, axis=None, method='mean')` works a level up: `groups` is a sequence of (name, members) pairs naming each new parcel and the parcels it comes from, aggregating those parcels' values rather than the greyordinates underneath, with the same reductions and the same `axis` choice over parcel dimensions. Groups come out in the order given, a parcel named twice in one group counts once, and one parcel may belong to several groups. Naming a parcel the image does not hold, or giving a group nothing to build from, is an error. A merged parcel covers everything its members cover, once each in the order given, and keeps their vertex counts and volume.

`concat(images, axis=0)` joins images along a dimension by adding the axes there together; every other dimension has to agree. `split(img, axis=0, sizes=None)` reverses that, cutting an image into one piece per element, or into consecutive non-empty `sizes` summing to the dimension's length; joining the pieces back returns the original.

Where two things being put together describe the same volume or surface, they have to agree about it: a differing affine, volume shape or vertex count is an error, while geometry a parcel leaves unset is completed from the brain model it is applied to. They need not cover the same structures, so images of different regions join fine, but a parcel may only name a structure its image covers.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/nipy/nibabel repo && cd repo
git checkout 014fc4a952aa4cec4daa94384a8d63e81f71457a
cp ../Dockerfile . && git apply ../test.patch
docker build -t nibabel-cifti-parcellation .
docker run --rm nibabel-cifti-parcellation ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t nibabel-cifti-parcellation .
docker run --rm nibabel-cifti-parcellation ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm nibabel-cifti-parcellation ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
