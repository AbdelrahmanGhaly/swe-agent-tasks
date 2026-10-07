# Add for, quantified and range expressions to XPath

| | |
|---|---|
| **Project** | [antchfx/xpath](https://github.com/antchfx/xpath) (MIT) |
| **Base commit** | [`f7f8961626`](https://github.com/antchfx/xpath/tree/f7f896162610a97fe285a27bf66d98e706c5f760) |
| **Language** | Go |
| **Reference solution** | +823 / -135 lines across 6 files |
| **New tests** | 12 test functions, +477 lines in 1 test file |

## Problem

Compile will accept xpath 2.0 expressions like for $v in E return R, some $v in E satisfies P, every $v in E satisfies P and A to B. for, some and every clauses may contain multiple variables: for $a in E1, $b in E2 return R. All of these may be used at the top level of an expression whether they are used in a predicate or inside parentheses. They may also be used as function arguments or nested inside other for , some , every or clauses. to has a precedence between + - and the comparison operators. 1 to 2 + 3 will evaluate to 5 values while 1 to 3 = 2 will evaluate to true.

You may use a variable in the clauses after it, not in its own, and anywhere inside of R or P. If you use a variable with the same name as one already inside of another clause it will hide that outer variable. Using a variable outside of where it is declared, a for with no return, a some or every with no satisfies or a clause with no in is a compile time error. The words for, some, every, in, return, satisfies and to may still be used as element names or variable names. for, some and every will not start an expression if there isn't a $ after them. to may also not function as an operator if div would not work in that spot.

R will be evaluated once for every combination of values that all of the variables are bound to. It will start with the first variable and work outward, the first variable being the variable that changes the least often. The results of each evaluation are added to the end of the last value. some will return true if any value combination makes P true. Every will return false if any value combination does not make P true. Therefore some will return false and every will return true when there are no values to bind.

A to B will produce a sequence of all the integer values from A up to and including B, so 2.2 to 4.8 gives 3 and 4. A and B are converted the same way number() does and need to be bigger than -2^53 and smaller than 2^53. to produces no values if the second value is smaller than the first. None of these expressions will change the context node while inside of them or if they are used in the same predicate as another statement.

When a variable holds a node you may use it like a node set with just that 1 node. You can start a new path from it ($s/book) or use predicates on it, so $b[1] and $b[last()] are $b itself. You may filter a for loop or a range by supplying a predicate that evaluates to a single number, which picks the value at that position. It can evaluate to a number or a sequence with only one number in it. So there will be no values in (3 to 5)[.] or in (3 to 5)[2.5].

If your for loop is evaluating to nodes then it will produce node sequences. During Select and Evaluate these node sequences will be returned as a node iterator. The order of the sequence will be preserved and duplicates will be maintained. You can use predicates on these sequences and position() will return the position in this sequence and last() its length. You can start paths from these nodes, like (for $b in //book return $b)/title, or reverse() them.

All other for loops as well as the to operator will return value sequences. During Evaluate they will be returned as a []interface{}. Containing values of types float64, string, and bool. You may filter these, or a variable holding a value, by using the . to represent the current value, also inside a for, some or every in that predicate. (1 to 10)[. mod 3 = 0] will return the sequence 3, 6, 9. (3 to 5)[2] will return 4.

You may use count() sum() string-join() boolean() and not() on both kinds of sequences. Wherever you could put a value or node. If the operation or function you are using requires one single value to work with it will take the first value in the sequence. That includes node sequences and the arguments of substring(), matches(), substring-before() and substring-after(). If there is no first value it will use NaN / "" instead. This is why contains(E,"") will return true even if E has no values. If you compare something to a sequence it will return true if the comparison is true for any combination of values. If either of the values to compare is a number then they will both be compared to each other as numbers. If not then they will be compared to each other as xpath 1.0 style strings compared.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/antchfx/xpath repo && cd repo
git checkout f7f896162610a97fe285a27bf66d98e706c5f760
cp ../Dockerfile . && git apply ../test.patch
docker build -t xpath-for-expressions .
docker run --rm xpath-for-expressions ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t xpath-for-expressions .
docker run --rm xpath-for-expressions ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm xpath-for-expressions ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
