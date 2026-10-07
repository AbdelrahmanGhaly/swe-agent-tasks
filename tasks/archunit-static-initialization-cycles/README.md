# Add static initialization cycle analysis to ArchUnit

| | |
|---|---|
| **Project** | [TNG/ArchUnit](https://github.com/TNG/ArchUnit) (Apache-2.0) |
| **Base commit** | [`a0a8aff514`](https://github.com/TNG/ArchUnit/tree/a0a8aff514d948615e7de2ed067e9f651a466a44) |
| **Language** | Java |
| **Reference solution** | +414 / -10 lines across 14 files |
| **New tests** | 60 test functions, +1,253 lines in 8 test files |

## Problem

Classes whose static initializers trigger each other can observe one another half initialized, or deadlock when two threads start from different ends. ArchUnit should be able to find these cycles.

Add `JavaClass.getStaticInitializationDependenciesFromSelf()`, returning the dependencies from the class to every other class that initializing it starts to initialize. That is a single step, not a closure: the result names each class reached, but nothing those classes go on to initialize. For a class whose initializer reads a field of `B`, where the initializer of `B` reads a field of `C`, the result holds `B` and not `C`.

Initializing a class first initializes its superclass, and then every superinterface the class implements itself, direct or inherited through other interfaces, that declares a default method. An interface does not initialize its superinterfaces, and an interface implemented by the superclass belongs to the superclass, so it is not part of what the class below it initializes.

Then its static initializer runs, together with all code it calls: called methods and constructors. Calling an instance method may run that method or any method that a subclass declares with the same name and parameter types; one that is private or static never counts, and neither does one declared outside the package of a package-private called method. A call written as `super.method()` runs the method it names and never an override of it. Code inside lambda bodies does not run, and method references, `Type::new` among them, do not call anything.

Within the code that runs, reading or writing a static field or calling a static method initializes the class declaring that member, and creating an instance initializes the created class. A constructor calling a constructor of its own class or of its direct superclass does not create an instance, while the body of the constructor it calls still runs. The class itself never counts. A dependency caused by code is described like the access that causes it, one for a supertype like inheritance.

Add `GeneralCodingRules.beFreeOfStaticInitializationCycles()`, a condition described as "be free of static initialization cycles" that reports each cycle among the checked classes with ArchUnit's usual cycle report, naming each class as `Class <name>`. Initialization running through classes that the rule does not check still connects the checked classes it reaches, shown by the dependency that reaches them. A class never forms a cycle with itself. Add the same rule for slices as `beFreeOfStaticInitializationCycles()` on slices, honouring ignored dependencies like the existing slice cycle rule. It is backed by `Slice.getStaticInitializationDependenciesFromSelf()`, the dependencies of the slice's classes on classes of other slices, where classes assigned to no slice pass initialization through.

Add `GeneralCodingRules.triggerStaticInitializationOfClassesThat(predicate)`, described as "trigger static initialization of classes that" plus the predicate description. Using ArchUnit's usual message form for classes, it reports each matching class reachable through initializations once, as "triggers static initialization of <name> by [a->b->name]" along a shortest chain of full class names, or "does not trigger static initialization of any matching class".

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (source files only)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/TNG/ArchUnit repo && cd repo
git checkout a0a8aff514d948615e7de2ed067e9f651a466a44
cp ../Dockerfile . && git apply ../test.patch
docker build -t archunit-static-initialization-cycles .
docker run --rm archunit-static-initialization-cycles ./test.sh --output_path /tmp/results.xml new   # fails: not implemented yet
git apply ../solution.patch && docker build -t archunit-static-initialization-cycles .
docker run --rm archunit-static-initialization-cycles ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm archunit-static-initialization-cycles ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
