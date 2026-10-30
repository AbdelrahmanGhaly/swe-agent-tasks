# Declare the files gcc writes besides the object as SCons side effects

| | |
|---|---|
| **Project** | [SCons/scons](https://github.com/SCons/scons) (MIT) |
| **Base commit** | [`5087fc0c80`](https://github.com/SCons/scons/tree/5087fc0c80aa6cad3538891299cd0c81e326a564) |
| **Language** | Python |
| **Reference solution** | +462 / -0 lines across 4 files |
| **New tests** | +998 lines in 1 test file |

## Problem

gcc will generate some additional files depending on what flags you give it. These include a dependency file, coverage information, stack-usage file, split-DWARF object file, files generated with -save-temps, optimization record and assembler listing. If you use the -Map flag when linking you will get a map file. SCons knows nothing about these files so scons -c will leave them around and if your building in parallel you may end up with 2 commands trying to write the same file.

You should list all these files as side effects of the object file when using the gcc, g++ or gas toolchain to create a static or shared object from C, C++, .S or .sx files. You should also list the map file as a side effect of what ever you use gnulink to link.

Make sure the path is what gcc 12 and the assembler and linker on your system will generate given the command that SCons will execute. These paths will be relative to the top level directory that you run SCons from even if the target is defined in a SConscript in a sub directory. You need to figure out what these paths are based on the command that will be executed when the builder is invoked. Do not run gcc or anything else to determine these.

Call graphs, -aux-info, dump files, dependency files created by the assembler and linker and any files created when linking with LTO should be ignored. Also ignore any commands that fail.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/SCons/scons repo && cd repo
git checkout 5087fc0c80aa6cad3538891299cd0c81e326a564
cp ../Dockerfile . && git apply ../test.patch
docker build -t scons-gcc-auxiliary-outputs .
docker run --rm scons-gcc-auxiliary-outputs ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t scons-gcc-auxiliary-outputs .
docker run --rm scons-gcc-auxiliary-outputs ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm scons-gcc-auxiliary-outputs ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
