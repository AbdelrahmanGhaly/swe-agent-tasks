# Support flex's C interface and prefixes in reflex --yy scanners

| | |
|---|---|
| **Project** | [Genivia/RE-flex](https://github.com/Genivia/RE-flex) (BSD-3-Clause) |
| **Base commit** | [`c06bb16a49`](https://github.com/Genivia/RE-flex/tree/c06bb16a49e79f4f123f4b31b6f53e5bfa884624) |
| **Language** | C++ |
| **Reference solution** | +425 / -27 lines across 4 files |
| **New tests** | +9,392 lines in 676 test files |

## Problem

Currently a c program that uses it's own main.c to drive flex scanners can not be made to work with reflex --yy. If you set a prefix using -P --prefix or %option prefix any actions that reference yytext will fail to compile.

When given the same specs and main.c a reflex --yy program should output the same thing as a flex 2.6.4 program (flex and gcc are installed on the system). This applies to all of the globals and functions that a flex non re-entrant scanner has, yyrestart() and yylex_destroy() included, along with the yywrap() that is called, but not to the buffer functions, yyalloc()/yyrealloc()/yyfree(), yy_flex_debug, yywrap() returning 0 to continue with a new yyin, or setting yylineno any other way than calling yyset_lineno() outside of yylex(). Using a prefix these will be renamed just like flex does. Within your spec file the yy names can still be used, but the %{ %} code only needs to use the variables.

Several scanners can be compiled separately and linked into one program as long as each has a different prefix and no more than one has no prefix. If the specs have YY_EXTERN_C defined as extern "C" the functions get c linkage so a c main.c can call them, otherwise they keep c++ linkage.

The following behaviors remain unchanged. How yylineno is incremented throughout the text, where a match gets the line it starts on. Including if the input stream is changed. Changing yyin in the middle of reading a file. The end of the input stream. Ie when yywrap() is called and what values are returned after the end of the input.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/Genivia/RE-flex repo && cd repo
git checkout c06bb16a49e79f4f123f4b31b6f53e5bfa884624
cp ../Dockerfile . && git apply ../test.patch
docker build -t re-flex-yy-prefixed-scanners .
docker run --rm re-flex-yy-prefixed-scanners ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t re-flex-yy-prefixed-scanners .
docker run --rm re-flex-yy-prefixed-scanners ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm re-flex-yy-prefixed-scanners ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
