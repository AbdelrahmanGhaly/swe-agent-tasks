# SWE Agent Tasks

Hard software-engineering tasks for evaluating AI coding agents, built on real open-source projects. For each one I wrote the problem statement, the tests, the reference solution and the Docker environment.

- **36 tasks** on **29 open-source projects**
- **Languages:** Python (22), Rust (5), Java (3), C++ (2), JavaScript (2), Go (1), TypeScript (1)
- **3,230 new test functions** and **20,387 lines** of reference solution in total

Each task asks for a real change to a real codebase: a new feature, or a fix that cuts across several modules. It is graded only by tests the solver never sees, so a solution passes only if it actually works. Every task was designed and tested against frontier AI coding agents to be hard for them to solve.

## What's in a task

Each folder under [`tasks/`](tasks) has:

| File | What it is |
|---|---|
| `README.md` | The problem statement, plus the project, base commit and size of the change |
| `test.patch` | New tests that fail before the change and pass after it, plus `test.sh` (`base` runs the existing suite, `new` runs the new tests) |
| `solution.patch` | The reference solution (source files only) |
| `Dockerfile` | The environment the tests run in |
| `BASE_COMMIT.txt` | The upstream commit both patches apply to |

## Running a task

Each task's README has the exact commands. The pattern is:

```bash
git clone <project> repo && cd repo && git checkout <base commit>
cp ../Dockerfile . && git apply ../test.patch
docker build -t task . && docker run --rm task ./test.sh --output_path /tmp/r.xml new   # new tests fail
git apply ../solution.patch && docker build -t task .
docker run --rm task ./test.sh --output_path /tmp/r.xml new    # new tests pass
docker run --rm task ./test.sh --output_path /tmp/r.xml base   # existing tests still pass
```

## Tasks

### Python (22)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [mwparserfromhell-table-cell-merging](tasks/mwparserfromhell-table-cell-merging) | [earwig/mwparserfromhell](https://github.com/earwig/mwparserfromhell) | Add a table grid with cell merging and splitting | +434 lines, 5 files |
| [basana-margin-liquidation](tasks/basana-margin-liquidation) | [gbeced/basana](https://github.com/gbeced/basana) | Add margin calls and liquidations to the backtesting exchange | +432 lines, 8 files |
| [bumble-a2dp-media-pipeline](tasks/bumble-a2dp-media-pipeline) | [google/bumble](https://github.com/google/bumble) | Fix media packetization across the AVDTP and A2DP audio path | +403 lines, 3 files |
| [libcst-control-flow-provider](tasks/libcst-control-flow-provider) | [Instagram/LibCST](https://github.com/Instagram/LibCST) | Add a control flow provider for statement reachability | +407 lines, 5 files |
| [cutadapt-read-splitting](tasks/cutadapt-read-splitting) | [marcelm/cutadapt](https://github.com/marcelm/cutadapt) | Add a splitting action that cuts reads at adapter occurrences | +476 lines, 5 files |
| [pyang-resolved-modules](tasks/pyang-resolved-modules) | [mbj4668/pyang](https://github.com/mbj4668/pyang) | Add resolved module output to the YANG and YIN formats | +560 lines, 7 files |
| [wfdb-edf-discontinuous](tasks/wfdb-edf-discontinuous) | [MIT-LCP/wfdb-python](https://github.com/MIT-LCP/wfdb-python) | Read discontinuous EDF+ files into variable-layout records | +517 lines, 3 files |
| [nibabel-cifti-parcellation](tasks/nibabel-cifti-parcellation) | [nipy/nibabel](https://github.com/nipy/nibabel) | Add greyordinate operations for CIFTI-2 images | +546 lines, 3 files |
| [optiland-constraint-resolution](tasks/optiland-constraint-resolution) | [optiland/optiland](https://github.com/optiland/optiland) | Resolve pickups and solves in dependency order | +1,239 lines, 9 files |
| [pgmpy-network-templates](tasks/pgmpy-network-templates) | [pgmpy/pgmpy](https://github.com/pgmpy/pgmpy) | Add reusable templates for composing discrete Bayesian networks | +547 lines, 5 files |
| [pony-collection-iteration](tasks/pony-collection-iteration) | [ponyorm/pony](https://github.com/ponyorm/pony) | Iterate over array and JSON values in Pony queries | +425 lines, 7 files |
| [pony-python-numeric-semantics](tasks/pony-python-numeric-semantics) | [ponyorm/pony](https://github.com/ponyorm/pony) | Give query arithmetic Python's numeric semantics | +536 lines, 7 files |
| [rdflib-query-provenance](tasks/rdflib-query-provenance) | [RDFLib/rdflib](https://github.com/RDFLib/rdflib) | Report what a SPARQL update changed and why | +1,004 lines, 7 files |
| [scons-cachedir-directory-targets](tasks/scons-cachedir-directory-targets) | [SCons/scons](https://github.com/SCons/scons) | Make directory targets rebuild and cache like file targets | +410 lines, 3 files |
| [pythran-range-analysis](tasks/pythran-range-analysis) | [serge-sans-paille/pythran](https://github.com/serge-sans-paille/pythran) | Sharpen integer range analysis in the pythran optimizer | +733 lines, 4 files |
| [skyfield-planetodetic-surfaces](tasks/skyfield-planetodetic-surfaces) | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) | Support planetodetic sites and local skies on other bodies | +313 lines, 5 files |
| [skyfield-spk-orbit-types](tasks/skyfield-spk-orbit-types) | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) | Load SPK orbit segments and multi-segment PCK files | +244 lines, 3 files |
| [sunpy-carried-quality](tasks/sunpy-carried-quality) | [sunpy/sunpy](https://github.com/sunpy/sunpy) | Carry a map's mask and uncertainty through its operations | +776 lines, 4 files |
| [textx-assigned-actions](tasks/textx-assigned-actions) | [textX/textX](https://github.com/textX/textX) | Add Xtext-style actions to textX grammar rules | +702 lines, 7 files |
| [vivisect-c-struct-layout](tasks/vivisect-c-struct-layout) | [vivisect/vivisect](https://github.com/vivisect/vivisect) | Lay out vstruct C structures the way GCC does | +887 lines, 5 files |
| [vivisect-elf-symbol-versioning](tasks/vivisect-elf-symbol-versioning) | [vivisect/vivisect](https://github.com/vivisect/vivisect) | Resolve ELF GNU symbol versions in the loader and linker | +354 lines, 5 files |
| [vivisect-symbolik-byte-memory](tasks/vivisect-symbolik-byte-memory) | [vivisect/vivisect](https://github.com/vivisect/vivisect) | Make symbolik memory and register writes byte-accurate | +403 lines, 8 files |
### Rust (5)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [egg-eclass-enumeration](tasks/egg-eclass-enumeration) | [egraphs-good/egg](https://github.com/egraphs-good/egg) | Add cost-ordered enumeration of the expressions in an e-class | +406 lines, 3 files |
| [syncpack-partitioned-version-election](tasks/syncpack-partitioned-version-election) | [JamieMason/syncpack](https://github.com/JamieMason/syncpack) | Add per-partition version election to version groups | +734 lines, 25 files |
| [fundsp-oversampling-units](tasks/fundsp-oversampling-units) | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) | Fix block processing in oversample and add dynamic oversampling | +494 lines, 6 files |
| [fundsp-sequencer-automation](tasks/fundsp-sequencer-automation) | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) | Add sample-accurate setting automation to Sequencer events | +501 lines, 6 files |
| [fundsp-time-stretch](tasks/fundsp-time-stretch) | [SamiPerttu/fundsp](https://github.com/SamiPerttu/fundsp) | Add time stretching to Wave and a variable-speed player | +470 lines, 7 files |
### Java (3)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [brouter-via-way-restrictions](tasks/brouter-via-way-restrictions) | [abrensch/brouter](https://github.com/abrensch/brouter) | Honor via-way turn restrictions in map creation and routing | +580 lines, 17 files |
| [timefold-solver-value-range-algebra](tasks/timefold-solver-value-range-algebra) | [TimefoldAI/timefold-solver](https://github.com/TimefoldAI/timefold-solver) | Add union, intersection and difference value ranges | +1,320 lines, 19 files |
| [archunit-static-initialization-cycles](tasks/archunit-static-initialization-cycles) | [TNG/ArchUnit](https://github.com/TNG/ArchUnit) | Add static initialization cycle analysis to ArchUnit | +414 lines, 14 files |
### C++ (2)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [highs-model-roundtrip](tasks/highs-model-roundtrip) | [ERGO-Code/HiGHS](https://github.com/ERGO-Code/HiGHS) | Preserve model structure through the HiGHS file handlers | +415 lines, 8 files |
| [ispc-designated-initializers](tasks/ispc-designated-initializers) | [ispc/ispc](https://github.com/ispc/ispc) | Add C99 designated initializers to ispc initializer lists | +538 lines, 7 files |
### JavaScript (2)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [digitaljs-conditional-breakpoints](tasks/digitaljs-conditional-breakpoints) | [tilk/digitaljs](https://github.com/tilk/digitaljs) | Add conditional breakpoints to circuit simulations | +309 lines, 6 files |
| [digitaljs-simulation-state](tasks/digitaljs-simulation-state) | [tilk/digitaljs](https://github.com/tilk/digitaljs) | Add saving, restoring and live engine switching for circuit simulations | +450 lines, 9 files |
### Go (1)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [xpath-for-expressions](tasks/xpath-for-expressions) | [antchfx/xpath](https://github.com/antchfx/xpath) | Add for, quantified and range expressions to XPath | +823 lines, 6 files |
### TypeScript (1)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [messageformat-parser-recovery](tasks/messageformat-parser-recovery) | [messageformat/messageformat](https://github.com/messageformat/messageformat) | Report every MessageFormat 1 parse error instead of the first | +585 lines, 5 files |

## License

Each task patches an open-source project, and its patches fall under that project's license (linked from each task's README).
