# Add a splitting action that cuts reads at adapter occurrences

| | |
|---|---|
| **Project** | [marcelm/cutadapt](https://github.com/marcelm/cutadapt) (MIT) |
| **Base commit** | [`4927632f7c`](https://github.com/marcelm/cutadapt/tree/4927632f7c546dd290c53501c8417f909252befe) |
| **Language** | Python |
| **Reference solution** | +476 / -30 lines across 5 files |
| **New tests** | 112 test functions, +1,232 lines in 1 test file |

## Problem

Cutadapt turns one input read into at most one output read. Add a `split` value for the action option, and a `--split-suffix` option.

Every occurrence found in a read is removed and the read is cut where it stood, so n occurrences give n+1 pieces, written in read order. Splitting is exhaustive: no piece still contains an occurrence. Occurrences are found as they are for trimming, so the error rate and the minimum overlap keep their meaning. A piece may be empty, which happens at either end of a read and between adjacent occurrences; an empty piece is a piece like any other and the minimum length filter is what removes them.

When a read yields two or more pieces the suffix is appended to each piece's id, the part of the header up to the first blank, leaving the rest of the header alone. A read that yields one piece keeps its name. The suffix is a template understanding `{piece}`, the number of the piece counting from one, and `{pieces}`, how many the read produced. It must contain `{piece}`, may contain nothing else in braces, and defaults to `_{piece}`.

Each piece then runs through the remaining read modifications and every filter on its own and is written on its own, while modifications that already run before adapter removal, such as fixed length cutting and quality trimming, still see the whole read. A piece counts as carrying an adapter when an occurrence was removed directly before or after it, so every piece of a cut read counts as trimmed while a read with no occurrence stays untrimmed. Everything the program already does keeps working, including running on more than one core.

On paired-end data both reads are cut with their own adapters and the pieces pair up by position, so the first pieces of both reads travel together. A pair whose reads do not fall into the same number of pieces is dropped.

Reads or pairs processed stays the number that came in, the per-adapter statistics count each occurrence once and record the length of the occurrence that was removed, and a read counts once towards the reads with an adapter however many occurrences it holds. The JSON report gains `read_counts.split`, the reads or pairs cut into more than one piece, `read_counts.pieces`, the pieces produced altogether, and `read_counts.unequal_pairs`, the pairs dropped for not matching; without this action these are zero, the input count and zero. Adding them raises the report's schema minor version by one. Written pieces plus filtered pieces equal the pieces produced.

Refuse what this action cannot serve, as it already refuses a bad command line: a number of trimming rounds other than one, paired adapters, the info, rest and wildcard files, demultiplexing, reverse complement matching, a linked adapter, and a read end with no adapter. Refuse the suffix option when reads are not being split, and a template lacking `{piece}` or naming anything else.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/marcelm/cutadapt repo && cd repo
git checkout 4927632f7c546dd290c53501c8417f909252befe
cp ../Dockerfile . && git apply ../test.patch
docker build -t cutadapt-read-splitting .
docker run --rm cutadapt-read-splitting ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t cutadapt-read-splitting .
docker run --rm cutadapt-read-splitting ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm cutadapt-read-splitting ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
