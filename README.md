# Mini-Dafny: Program Verification with Z3

This repository provides a technical overview of a CMSC433 coursework project. The original implementation and assignment materials are not publicly distributed in accordance with course policies.

**CMSC433 — Programming Language Technologies and Paradigms**  
**University of Maryland, College Park | Fall 2025 | Project 4**

## Overview

Mini-Dafny is a small **program verification tool** written in Python using the [Z3 SMT solver](https://github.com/Z3Prover/z3). Inspired by the ideas behind the Dafny verification language, it checks whether simple programs satisfy assertions and specified preconditions/postconditions **without executing them on individual inputs**.

The verifier works by computing **weakest preconditions** backwards through a program, converting the resulting verification condition into a logical formula, and asking Z3 whether a counterexample exists. If Z3 proves that the negation of the verification condition is unsatisfiable, the program is verified under the stated assumptions.

## Key Features

- **Weakest-precondition calculation:** Generate logical proof obligations by reasoning backward from a desired postcondition.
- **Assignments and substitution:** Apply Z3's `substitute` operation to account for how assignments change the program state.
- **Assertions and assumptions:** Model properties that must hold (`ASSERT`) and conditions assumed about program inputs (`ASSUME`).
- **Conditional branches:** Verify both paths of an `IF` statement, including optional `else` branches.
- **Loop invariants:** Generate conditions to check that a `WHILE` loop's invariant holds initially, is preserved by its body, and is sufficient for reasoning about the state after the loop.
- **Counterexample generation:** Use Z3 to report a model demonstrating when a verification condition does not hold.
- **Example programs and tests:** Explore correct and incorrect examples involving assignments, swaps, conditionals, maximum-of-three values, and iterative loops.

## Technologies and Concepts

- **Language:** Python 3
- **Solver:** Z3 (`z3-solver` Python package)
- **Concepts:** Hoare logic, weakest preconditions, SMT solving, symbolic reasoning, first-order logic, loop invariants, universal quantification, and formal program verification
- **Testing:** Python `unittest`

## How It Works

Programs are represented as Python expressions using a small set of statement classes rather than a separate text parser.

| Construct | Purpose |
| --- | --- |
| `BLOCK(...)` | Execute a sequence of statements in order. |
| `ASSIGN(variable, expression)` | Assign a symbolic expression to a variable. |
| `ASSUME(condition)` | Introduce an assumed property of the state. |
| `ASSERT(condition)` | Require a property to be true. |
| `IF(condition, then_branch, else_branch)` | Model conditional control flow. |
| `WHILE(condition, invariant, ...)` | Reason about a loop using an explicit invariant. |
| `SKIP()` | Represent a no-op statement. |
| `AND(...)` / `OR(...)` | Combine logical predicates. |

The custom infix operator `|ᐘᆃ|` is also available as shorthand for assignment (`:=`).

The main entry point is:

```python
verify(stmt, pre=True, post=True, verbose=True)
```

Internally, the verifier calculates a verification condition of the form:

```text
precondition => weakest_precondition(program, postcondition)
```

It then asks Z3 whether the **negation** of that condition can be satisfied:

- **`unsat`** — no counterexample was found; the verification condition is proved valid.
- **`sat`** — a counterexample exists, so the claimed property does not hold universally.
- **`unknown`** — the solver could not determine a result.

## Example

The following example verifies that incrementing a nonnegative integer produces a value of at least 1:

```python
from mini_dafny import *

x = INT_VARS('x')

program = BLOCK(
    ASSUME(x >= 0),
    ASSIGN(x, x + 1),
    ASSERT(x >= 1),
)

verify(program, verbose=True)
```

Z3 checks the corresponding logical condition for **all integer values of `x`**, subject to the assumption `x >= 0`. For this example, the expected solver response is `proved!`.

Other included examples investigate loop correctness using invariants, including copying a value through repeated addition and computing a square through repeated addition.

## Project Structure

```text
.
├── mini_dafny.py      # Verification constructs, weakest preconditions, Z3 integration
├── test1.py           # Tests for assignments, skip, swaps, and conditionals
├── test2.py           # Tests for loops and more complex examples
├── asgn_*.py          # Assignment examples
├── if_*.py            # Conditional examples
├── skip_*.py          # No-op examples
├── swap_*.py          # Variable-swapping examples
├── loop_to_*.py       # Loop verification examples
├── slow_copy_*.py     # Iterative copying examples
├── slow_square_*.py   # Iterative squaring examples
├── max3_*.py          # Maximum-of-three examples
└── subst.py           # Z3 substitution demonstration
```

## Installation and Usage

**Prerequisite:** Python 3

Install Z3's Python bindings:

```bash
python3 -m pip install z3-solver
```

Run a sample verification:

```bash
python3 asgn_0.py
```

Run the included test suites:

```bash
python3 test1.py
python3 test2.py
```

The test cases compare `verify(...)` results with Z3's `sat` and `unsat` values. As supplied, the test files include some assignment-provided examples and test scaffolding.

## What I Implemented and Learned

The principal assignment was to complete the **`wp(self, post)` methods** that compute weakest preconditions for program constructs such as blocks, assignments, assertions, assumptions, conditionals, and loops. The implementation uses symbolic substitution, logical implications, and quantified loop-invariant obligations to reason about program correctness.

This project provided hands-on experience translating programming-language semantics into formulas that an automated theorem prover can check, including how counterexamples help diagnose invalid assertions and insufficient loop invariants.

## Academic Context

Developed for **CMSC433, Project 4**, at the University of Maryland, College Park. The Mini-Dafny framework, assignment specification, and example tests were provided as course materials; the project work focused on implementing the weakest-precondition logic. Check the course's code-sharing policy before publishing coursework or instructor-provided test files publicly.
