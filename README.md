# Type Checking with Rewriting Rules

This page introduces the companion artifact of the paper titled "Type Checking with Rewriting Rules", accepted at SLE 2024.
The artifact consists of the Hylo compiler, which includes a completion procedure to reject generic signatures with undecidable type equality tests.
More details are in the paper.

This repository contains:
- The present document.
- The sources of the Hylo compiler (at commit [5e205b8](https://github.com/hylo-lang/hylo/commit/5e205b87ab25e86b4ec5b93198d22f8bdfe9f9bc)).
- The accepted version of the paper.

Note: The most up to date sources of Hylo's compiler can be downloaded from its [main repository](https://github.com/hylo-lang/hylo).

## Installation

Unzip the sources and follow the installation instructions in the README document.

There are three ways to run Hylo's compiler:
- Build a native executable for your system (macOS, Ubuntu, Windows)
- Use a [devcontainer](https://containers.dev)
- Use the online [Compiler Explorer](https://godbolt.org) (aka Godbolt)

For the purpose of evaluating this artifact, we recommand going with the first or second option so that you may log the rewriting system produced by the type checker.
The online tool can only run programs and/or show x86 assembly.

## Relevant Sources

Code relevant to this artifact is contained in the following source files:

- `Sources/FrontEnd/TypeChecking/Rewriting/*.swift`: implements Knuth-Bendix completion
- `Sources/FrontEnd/TypeChecking/TypeChecker.swift`: implements type checking for Hylo

The entry points to the construction of a rewriting system are the methods named `environment(of:)` in the type checker.

## Construct Rewriting Systems

The compiler can output the rewriting system it has generated for a particular generic signature with the flag `--show-requirements file:line`, where `file` is the path to one of the source files being compiled and `line` is a line number in that file.
For example, given a file `test.hylo` with the following contents:

```
trait P { type X: P }
type A<T: P, U where T.X.X == U> {}
```

The requirement system produced for the generic type `A` can be displayed with the command below.
The option `--typecheck` makes the compiler exit after type checking rather than compiling the input all the way down to an executable binary.

```
hc --typecheck --show-requirements test.hylo:2 test.hylo
```

The output for this specific example will be:

```
0: [P].[P] => [P]
1: [::P.X].[P] => [::P.X]
2: T.[P] => T
3: T.[::P.X].[::P.X] => U
4: U.[P] => U
```

Although all examples from the paper can be reproduced in Hylo, some adaptation might be necessary to conform to Hylo's syntax.
The simplest approach is to define constraints in the where clause of a generic function.
For instance, Example 4.9 from the paper can be reproduced as follows:

```
trait C1 { type S: C1 }
trait C2: C1 {}
fun f<T2: C2 where T2.S == T2>() {}

// hc --typecheck --show-requirements test.hylo:3 test.hylo
//
// 0: [C2].[C2] => [C2]
// 1: [C1].[C1] => [C1]
// 2: [::C1.S].[C1] => [::C1.S]
// 3: T2.[C2] => T2
// 4: T2.[C1] => T2
// 5: T2.[::C1.S] => T2
```
