# Add Xtext-style actions to textX grammar rules

| | |
|---|---|
| **Project** | [textX/textX](https://github.com/textX/textX) (MIT) |
| **Base commit** | [`174f8219a0`](https://github.com/textX/textX/tree/174f8219a0e0e8a6868d2527f274270564bf0244) |
| **Language** | Python |
| **Reference solution** | +702 / -89 lines across 7 files |
| **New tests** | 52 test functions, +818 lines in 1 test file |

## Problem

textX grammars should support Xtext-style actions, which let a rule decide the class of the object it builds. An action is written in braces inside a rule body. `{Class}` makes a new object of Class the object being built. `{Class.attr=current}` and `{Class.attr+=current}` do the same, but first hand the new object the object built so far through attr, assigned to it or appended to it as a list.

Assignments reached after an action fill whichever object is current at that point, and a rule returns the object that is current when it finishes. Before any action, the object built so far is the rule's own object when the rule has assignments of its own, and the object returned by the rule it calls first when it has none. For example, `Sum: Term ({Add.left=current} '+' right=Term)*;` parses 1 + 2 + 3 into an Add whose left is the Add for 1 + 2. The same holds around a repetition: an assignment that comes before an action inside a repeated group fills the object that action created on an earlier pass.

An action class is looked up among the grammar's rules first. If no rule has that name, it is added to the metamodel and inherits from every rule using it in an action, so references to those rules find its objects too. Each class gets exactly the attributes its objects can receive: the action attribute holds the handed-over object by containment, each assignment that can fill an object of the class becomes its attribute, and a rule keeps only the assignments that can fill its own object.

An attribute an action appends to is always a list; otherwise multiplicity is decided per object, so an attribute is a list only when a single object can receive it more than once. A `{Class}` action that can follow an assignment to the rule's own object is a TextXSemanticError when the metamodel is built.

The handed-over object gets the action object as its parent, and the action object gets the parent the rule's result would otherwise have had. An action object starts where the handed-over object starts, a `{Class}` object starts at the first input the rule matches, and the rule's own object and every action object end where the last input consumed while they were current ends. Whitespace skipped before an action is not part of the span of the object that action replaces. Action classes otherwise behave like rule classes, so user classes (constructed after the objects they contain), object processors and reference resolution work with them. Grammars without actions behave as before.

In textX grammar models, such as those from grammar_model_from_str, an action is the `action` attribute of an Expression: an `Action` object whose `class_name` is the class and, for the two assigned forms, whose `attribute` is the attribute name and whose `operator` is `=` or `+=`.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/textX/textX repo && cd repo
git checkout 174f8219a0e0e8a6868d2527f274270564bf0244
cp ../Dockerfile . && git apply ../test.patch
docker build -t textx-assigned-actions .
docker run --rm textx-assigned-actions ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t textx-assigned-actions .
docker run --rm textx-assigned-actions ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm textx-assigned-actions ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
