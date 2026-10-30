# SWE Agent Tasks

Hard software-engineering tasks for evaluating AI coding agents, built on real open-source projects. For each one I wrote the problem statement, the tests, the reference solution and the Docker environment.

The tasks come in two parts, both tested against the same frontier AI coding agents. Part 2 tests those agents from other angles, with bigger changes.

| Part | Folder | Tasks | What it tests |
|---|---|---|---|
| [Part 1: Core](#part-1-core) | [`tasks/`](tasks) | 36 tasks on 29 projects | New features and cross-module fixes, mostly in Python libraries |
| [Part 2: Advanced](#part-2-advanced) | [`advanced/`](advanced) | 30 tasks on 16 projects | Matching real tools, two implementations at once, binary formats, protocols, several tasks per codebase |

- **66 tasks** on **42 open-source projects**
- **Languages:** Python (28), Go (9), C++ (7), C++ and Rust (5), Rust (5), TypeScript (5), Java (4), JavaScript (3)
- **107,324 lines** of new tests and **52,313 lines** of reference solution in total

Each task asks for a real change to a real codebase: a new feature, or a fix that cuts across several modules. It is graded only by tests the solver never sees, so a solution passes only if it actually works. Every task was designed and tested against frontier AI coding agents to be hard for them to solve.

## What's in a task

Each folder under [`tasks/`](tasks) and [`advanced/`](advanced) has:

| File | What it is |
|---|---|
| `README.md` | The problem statement, plus the project, base commit and size of the change |
| `test.patch` | New tests that fail before the change and pass after it, plus `test.sh` (`base` runs the existing suite, `new` runs the new tests) |
| `solution.patch` | The reference solution: code, plus any build files and docs it updates (no tests) |
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

## Part 1: Core

36 tasks on 29 projects in [`tasks/`](tasks), with 3,230 new test functions and 20,387 lines of reference solution. Most of them ask for a new feature in a library, described in prose.

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

## Part 2: Advanced

30 tasks on 16 projects in [`advanced/`](advanced), with 54,554 lines of new tests and 31,926 lines of reference solution. They test the same agents as Part 1 from other angles, and the changes are bigger: the average reference solution is about 1,060 lines, against about 570 in Part 1.

- **Matching a real tool.** In 23 of the 30 tasks, the problem statement names a real tool and the change has to do what that tool does: GNU make 4.2.1, GNU ed 1.19, flex 2.6.4, rsync 3.2.7, OpenSSH 9.2, apt 2.6, dpkg, dose-distcheck, gcc 12, gfortran, glibc 2.36, GDB 13, ffmpeg 5.1, WavPack and FLAC. The tool is installed in the task's environment, and where it disagrees with its own manual, the tool is what counts. An agent has to run the tool to find out what it really does.
- **Two implementations at once.** Each of the five kati tasks changes both ckati (C++) and rkati (Rust), and both have to match GNU make.
- **Binary formats and low-level code.** ELF dependencies, rpaths and split debug symbols (linuxdeploy); MP4 edit lists, exact MPEG and FLAC sample counts, stored checksums, and the original file's chunks kept inside WavPack and FLAC files (music-metadata, tinytag); flex's C interface in generated scanners (RE-flex); and faults on an emulated ARMv6-M core (rp2040js).
- **Protocols and shared state.** Weighted quorums, bounded appends and leadership handoff in etcd's raft; new rsync options in every client, server and daemon role (gokr-rsync); and pruning a workflow database without deleting records that other runs still use (redun).
- **One codebase from several sides.** Seven projects have more than one task: kati (5), aptly (3), linuxdeploy (3), music-metadata (3), raft (3), RE-flex (2) and SCons (2). Each task on a project works on a different part of it. DigitalJS, Skyfield and SCons also appear in Part 1, with different tasks.
- **More languages.** Python is 6 of the 30 tasks here, against 22 of 36 in Part 1. Go, C++ and Rust make up most of the rest.

### Go (8)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [aptly-apt-pattern-queries](advanced/aptly-apt-pattern-queries) | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) | Add apt pattern support to aptly package queries | +2,265 lines, 8 files |
| [aptly-snapshot-diff-changes](advanced/aptly-snapshot-diff-changes) | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) | Show changelog entries in aptly snapshot diff | +1,269 lines, 10 files |
| [aptly-snapshot-installability](advanced/aptly-snapshot-installability) | [aptly-dev/aptly](https://github.com/aptly-dev/aptly) | Add an installability check to aptly snapshot verify | +870 lines, 6 files |
| [raft-bounded-appends](advanced/raft-bounded-appends) | [etcd-io/raft](https://github.com/etcd-io/raft) | Bound the log entries a raft Ready hands out for appending | +243 lines, 5 files |
| [raft-removal-handoff](advanced/raft-removal-handoff) | [etcd-io/raft](https://github.com/etcd-io/raft) | Hand off leadership when a raft leader removes itself | +57 lines, 2 files |
| [raft-weighted-quorums](advanced/raft-weighted-quorums) | [etcd-io/raft](https://github.com/etcd-io/raft) | Add weighted voting to raft quorums | +231 lines, 8 files |
| [rsync-backup-options](advanced/rsync-backup-options) | [gokrazy/rsync](https://github.com/gokrazy/rsync) | Add -R, --files-from, --link-dest and -H to gokr-rsync | +1,600 lines, 32 files |
| [u-root-gnu-ed](advanced/u-root-gnu-ed) | [u-root/u-root](https://github.com/u-root/u-root) | Make u-root's ed behave like GNU ed | +3,488 lines, 13 files |

### Python (6)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [redun-execution-pruning](advanced/redun-execution-pruning) | [insitro/redun](https://github.com/insitro/redun) | Add execution pruning to the redun repository database | +764 lines, 5 files |
| [meson-fortran-module-scanning](advanced/meson-fortran-module-scanning) | [mesonbuild/meson](https://github.com/mesonbuild/meson) | Fix meson's Fortran module scanning to match what gfortran does | +1,196 lines, 5 files |
| [scons-gcc-auxiliary-outputs](advanced/scons-gcc-auxiliary-outputs) | [SCons/scons](https://github.com/SCons/scons) | Declare the files gcc writes besides the object as SCons side effects | +462 lines, 4 files |
| [scons-gcc-include-scanning](advanced/scons-gcc-include-scanning) | [SCons/scons](https://github.com/SCons/scons) | Make the conditional C scanner read the headers GCC reads | +1,739 lines, 2 files |
| [skyfield-galilean-phenomena](advanced/skyfield-galilean-phenomena) | [skyfielders/python-skyfield](https://github.com/skyfielders/python-skyfield) | Add Galilean moon phenomena to Skyfield's almanac | +341 lines, 2 files |
| [tinytag-exact-duration](advanced/tinytag-exact-duration) | [tinytag/tinytag](https://github.com/tinytag/tinytag) | Add an exact duration mode for MPEG and FLAC files | +379 lines, 2 files |

### C++ (5)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [RE-flex-yy-input-buffers](advanced/RE-flex-yy-input-buffers) | [Genivia/RE-flex](https://github.com/Genivia/RE-flex) | Fix flex input buffers in reflex --yy scanners | +391 lines, 9 files |
| [RE-flex-yy-prefixed-scanners](advanced/RE-flex-yy-prefixed-scanners) | [Genivia/RE-flex](https://github.com/Genivia/RE-flex) | Support flex's C interface and prefixes in reflex --yy scanners | +425 lines, 4 files |
| [linuxdeploy-native-dependency-resolution](advanced/linuxdeploy-native-dependency-resolution) | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) | Resolve ELF dependencies natively instead of running ldd | +1,193 lines, 4 files |
| [linuxdeploy-native-rpath-editing](advanced/linuxdeploy-native-rpath-editing) | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) | Read and write rpaths natively instead of calling patchelf | +1,272 lines, 7 files |
| [linuxdeploy-split-debug-symbols](advanced/linuxdeploy-split-debug-symbols) | [linuxdeploy/linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) | Save the debug symbols of stripped files for GDB | +994 lines, 6 files |

### C++ and Rust (5)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [kati-implicit-rule-chains](advanced/kati-implicit-rule-chains) | [google/kati](https://github.com/google/kati) | Chain implicit rules in kati the way GNU make does | +1,561 lines, 8 files |
| [kati-load-plugins](advanced/kati-load-plugins) | [google/kati](https://github.com/google/kati) | Add GNU make's load directive and loaded-object API to ckati and rkati | +3,894 lines, 35 files |
| [kati-oneshell-shellflags](advanced/kati-oneshell-shellflags) | [google/kati](https://github.com/google/kati) | Implement .ONESHELL and .SHELLFLAGS in ckati and rkati | +829 lines, 20 files |
| [kati-recipe-prefix](advanced/kati-recipe-prefix) | [google/kati](https://github.com/google/kati) | Add .RECIPEPREFIX support to ckati and rkati | +1,250 lines, 34 files |
| [kati-secondary-expansion](advanced/kati-secondary-expansion) | [google/kati](https://github.com/google/kati) | Add .SECONDEXPANSION support to ckati and rkati | +1,250 lines, 16 files |

### TypeScript (4)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [music-metadata-decoded-duration](advanced/music-metadata-decoded-duration) | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) | Report the decoded length of MP4 audio | +1,100 lines, 6 files |
| [music-metadata-original-tags](advanced/music-metadata-original-tags) | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) | Read the original file's tags kept in WavPack and FLAC | +696 lines, 10 files |
| [music-metadata-verify-checksums](advanced/music-metadata-verify-checksums) | [Borewit/music-metadata](https://github.com/Borewit/music-metadata) | Verify the checksums stored in audio files | +793 lines, 12 files |
| [rp2040js-hardfault-lockup](advanced/rp2040js-hardfault-lockup) | [wokwi/rp2040js](https://github.com/wokwi/rp2040js) | Add ARMv6-M HardFault and lockup to the RP2040 core | +174 lines, 2 files |

### Java (1)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [mina-sshd-ssh-config-match](advanced/mina-sshd-ssh-config-match) | [apache/mina-sshd](https://github.com/apache/mina-sshd) | Resolve ssh config Match blocks and Include files like OpenSSH | +1,083 lines, 5 files |

### JavaScript (1)

| Task | Project | What it asks for | Reference solution |
|---|---|---|---|
| [digitaljs-signal-forcing](advanced/digitaljs-signal-forcing) | [tilk/digitaljs](https://github.com/tilk/digitaljs) | Add force and release to circuit simulations | +117 lines, 6 files |

## License

Each task patches an open-source project, and its patches fall under that project's license (linked from each task's README).
