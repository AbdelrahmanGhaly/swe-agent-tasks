# Report every MessageFormat 1 parse error instead of the first

| | |
|---|---|
| **Project** | [messageformat/messageformat](https://github.com/messageformat/messageformat) (MIT) |
| **Base commit** | [`0ffba11d49`](https://github.com/messageformat/messageformat/tree/0ffba11d49b1aa4579497ccec7fb9ec3c082e709) |
| **Language** | TypeScript |
| **Reference solution** | +585 / -44 lines across 5 files |
| **New tests** | 241 test functions, +2,241 lines in 3 test files |

## Problem

Add an `onError` parse option. Without it every existing failure still throws exactly the message it throws today. With it the parser stops throwing its own failures: as long as the callback returns normally, each is passed to it in source order and a best-effort token list is returned. Source order means by position, not discovery order: a problem inside a placeholder follows one starting earlier.

Each reported `ParseError` carries an `offset`, `line`, `col`, `length`, and a `code` from `missing-end`, `invalid-select-key`, `invalid-plural-key`, `unexpected-offset`, `offset-after-cases`, `invalid-syntax`, `duplicate-case` for a repeated key, and `missing-other` for a select lacking `other`. A report starts at the token the parser was reading when it found the problem, so a malformed `{b, }` and a strict `{b, x}` are both reported at their comma rather than at the brace; `missing-other` takes the select's closing brace alone, `duplicate-case` the repeated case, not the first. No reported problem discards text: an invalid or duplicated key keeps its case and content, and a rejected offset is ignored while other cases stay. Neither of the last two is ever raised without `onError`.

Only `invalid-syntax` produces an `error` token. For it the parser skips the whole outermost placeholder containing the failure, from its opening brace through its closing brace, emits one `error` token whose `value` and `ctx.text` are exactly that skipped source, then keeps parsing what follows. Not every `{` opens something: unless followed by an argument name or preceded by a case key, as with the `{,` in `{a, number, {, }`, it is ordinary text and does not move the span's end.

A brace inside an apostrophe-quoted span neither opens nor closes one. The rule holds across the whole skipped region, in argument text as in body text. An apostrophe begins such a span only directly before `{`, `}` or `#`, and only when a later apostrophe closes it. A doubled apostrophe stands for a literal one: outside a span it opens nothing, inside one it does not close it. A failure outside any placeholder gets an `error` token covering just that text.

Input ends early when that outermost placeholder is never closed later. That case skips the recovery above and reports `missing-end`: whatever went wrong inside, no `invalid-syntax` joins it and no `error` token stands in. What was parsed is kept: an unterminated argument, formatter or select comes back as the partial node. A formatter keeps the type it read, so `{n, nope` comes back as a `function` token with key `nope`. A select that never closes has no closing brace to carry `missing-other`. Positions after a recovery stay in original-source coordinates.

The `error` token joins the exported token union. Compiling through `MessageFormat` with `onError` produces no output for a skipped span while the rest still formats, and when `returnType` is `values` the span adds nothing. Its callback receives the same problems the parser reports, though its own checks still apply: a select with no `other` still fails to compile. `mf1ToMessageData` converts around the token too.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/messageformat/messageformat repo && cd repo
git checkout 0ffba11d49b1aa4579497ccec7fb9ec3c082e709
cp ../Dockerfile . && git apply ../test.patch
docker build -t messageformat-parser-recovery .
docker run --rm messageformat-parser-recovery ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t messageformat-parser-recovery .
docker run --rm messageformat-parser-recovery ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm messageformat-parser-recovery ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
