# Resolve ELF GNU symbol versions in the loader and linker

| | |
|---|---|
| **Project** | [vivisect/vivisect](https://github.com/vivisect/vivisect) (Apache-2.0) |
| **Base commit** | [`66c93f0c1c`](https://github.com/vivisect/vivisect/tree/66c93f0c1caa949434b9b1ba8bb64c8bad0eb3d9) |
| **Language** | Python |
| **Reference solution** | +354 / -12 lines across 5 files |
| **New tests** | 50 test functions, +1,351 lines in 1 test file |

## Problem

vivisect does not respect gnu symbol versions. `foo@@VERS_2.0` and `foo@VERS_1.0` are loaded as `foo`. Imports are bound to the last one. Have `Elf`, the loader and `vivisect.analysis.generic.linker` support these.

`Elf.getVersionDefs()` returns the `.gnu.version_d` symbols in order they appear in the file. They contain `ndx`, `name` (first aux name), `parents` (rest of the aux names) and `isbase` (VER_FLG_BASE). `getVersionNeeds()` returns `.gnu.version_r` symbols. They have a `file` and `auxs` (`other`, version index and `name`). `getVersionSymbols()` returns the list of `.gnu.version` values, one for each dynamic symbol. These entries and aux's are chained together by their byte offset. A zero value ends the chain. They can be 32 or 64 bits depending on the file.

`getSymbolVersion(i)` returns `(version, isdefault)` for the i'th dynamic symbol. `getVersionedName(i)` returns a string, the name with the version appended. If the version is 0 or 1, or the symbol is a version label (a def in a reserved section index like SHN_ABS named after its own version), then `(None, True)` and just the name. Other defs in a reserved section keep their version. If `i` is out of bounds then `(None, True)` and an empty string. Undefined symbols get their version from the needs: `(v, False)` and `sym@v`. Defined symbols use the defs. `sym@@v` or if the 0x8000 bit is set then `(v, False)` and `sym@v`. If there are no versions then empty lists and no version on the name.

The loader marks the values in `.gnu.version` as a 2 byte number. Each entry and aux is marked as `elf.ElfVerdef`, `elf.ElfVerdaux`, `elf.ElfVerneed` or `elf.ElfVernaux`. Hidden defs are exported as `sym@version`. Default defs have no version. Imports keep their plain name like `*.printf` even when they need a version. Relocs to a def with a version point to it.

The linker only binds an import that needs a version within the library it was needed from: to the def with that version, or by name if that library has no versions. If the library has versions then an import without a version binds to the def with a version index of 1 or 2. Even if its hidden. If there is no such def it binds to the default. If only hidden versions exist it stays unbound. If the library has no versions it binds by name.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/vivisect/vivisect repo && cd repo
git checkout 66c93f0c1caa949434b9b1ba8bb64c8bad0eb3d9
cp ../Dockerfile . && git apply ../test.patch
docker build -t vivisect-elf-symbol-versioning .
docker run --rm vivisect-elf-symbol-versioning ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t vivisect-elf-symbol-versioning .
docker run --rm vivisect-elf-symbol-versioning ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm vivisect-elf-symbol-versioning ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
