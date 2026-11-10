---
title: "Mutation Testing for AI-Generated Code and Tests"
date: 2026-11-10T12:00:00+01:00
draft: false
tags: ["testing", "mutation-testing", "ai"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

AI coding assistants can generate a working implementation in a few seconds. They can also generate a test suite that passes in the same amount of time. That speed is useful, but a green test run does not tell us whether the tests would catch a real defect.

Mutation testing gives us a practical way to check that question. A mutation tool makes small changes to the production code, runs the test suite, and reports which changes the tests detected. The changed programs are called mutants. A test suite that fails when a mutant is present has killed that mutant. A surviving mutant shows a behavior that the tests did not distinguish.

This matters when an AI assistant writes both the code and the tests. The assistant may produce plausible tests that exercise the happy path while missing the decisions that matter. Mutation testing checks the relationship between the implementation and the assertions, rather than whether the test process finished successfully.

## A passing test is only one signal

The [mutant-testing repository](https://github.com/tomasalmeida/mutant-testing) is a small Java example with two independent Maven modules:

* `validators` calculates and validates Spanish DNI and NIE control letters.
* `text-analysis` checks whether parentheses, square brackets, and curly brackets are balanced.

The tests are small enough to read in one sitting. The validator tests check a valid DNI (`3A`) and a valid NIE (`X0000001R`). The bracket tests check one balanced expression and one mismatched expression (`{[}]`). Every test passes. That is useful evidence, but it does not describe the whole behavior of either class.

For example, the validator tests do not check an invalid document, lowercase input, the other NIE prefixes, or enough control-letter cases to exercise the modulo-23 table thoroughly. The bracket tests do not check an empty expression, ordinary text without brackets, an unmatched closing bracket, or each bracket type independently.

Line coverage does not expose those omissions. The PIT reports for this project show 100% line coverage in both modules:

```text
validators:    17/17 lines, 96/115 mutants killed  (83%)
text-analysis: 19/19 lines, 119/140 mutants killed (85%)
```

The exact percentages are results from the reports currently checked out in the example repository, not a universal quality threshold. They make the important point: executing every production line is compatible with leaving meaningful changes undetected.

## What mutation testing changes

PIT can apply changes such as replacing a comparison, changing a return value, altering an arithmetic operator, or removing a method call. It then runs the tests against each modified version. A useful mutation is one that represents a plausible mistake in the production code.

In `BalancedBracketsValidator`, the implementation pushes opening brackets onto a stack, pops when it sees a closing bracket, and checks whether the pair matches:

```java
if (isOpeningBracket(bracket)) {
    stack.push(bracket);
} else if (isClosingBracket(bracket)) {
    final Character openingBracket = stack.pop();
    if (areUnmatchablePair(openingBracket, bracket)) {
        return false;
    }
}
```

The existing mismatched-expression test kills mutations in the pairing logic. Other survivors show that the suite has not fully specified the behavior around the loop and the helper methods. Most notably, the implementation calls `stack.pop()` for a closing bracket even when the stack is empty. An input such as `")"` is therefore worth deciding explicitly: should it return `false`, or is an exception part of the contract? Mutation testing cannot choose the requirement, but it makes the missing decision visible.

The result is usually summarized as a mutation score:

```text
mutation score = killed mutants / (all mutants - excluded mutants)
```

The score is a useful signal, but the surviving mutants tell us more than the number. They show where the tests allow a meaningful change in behavior to pass unnoticed.

Mutation testing does not prove that the implementation is correct. A test can kill a mutant for the wrong reason, and a mutation operator cannot represent every possible production bug. The technique measures the strength of the test suite against a defined set of small changes. That is a much stronger check than line coverage alone.

## What the repository reports reveal

The validator report contains survivors in several interesting places:

* Replacing the return from `isValid` with `true` survives because the tests only assert valid documents. An invalid document test would make the contract more explicit.
* Changing the case conversion in `getControlDigit` survives for one mutation, so the case behavior is not fully distinguished.
* Changing the `23` in the control-letter calculation survives. The current examples do not cover enough different remainders to protect that algorithm.
* Several mutations around NIE prefix conversion survive. Testing `Y` and `Z`, in addition to `X`, would check the mapping rather than only one prefix.

For example, these tests would add a negative assertion and exercise all three NIE prefixes:

```java
@Test
void rejectsAnIncorrectControlLetter() {
    assertFalse(validator.isValid("X0000001S"));
}

@Test
void validatesEachNiePrefix() {
    assertTrue(validator.isValid("X0000001R"));
    assertTrue(validator.isValid("Y0000001S"));
    assertTrue(validator.isValid("Z0000001Y"));
}
```

The calculation also deserves examples with different remainders, rather than only one document number:

```java
@Test
void calculatesDifferentControlLetters() {
    assertEquals('T', validator.getControlDigit("00000000"));
    assertEquals('R', validator.getControlDigit("00000001"));
    assertEquals('Z', validator.getControlDigit("12345678"));
}
```

The bracket report tells a similar story. The existing nested and mismatched examples kill many important mutations, but survivors remain around the loop boundary, return value, and bracket classification helpers. Adding examples for empty input, text around brackets, unmatched closings, and all three bracket pairs would describe the public behavior better than adding assertions solely to increase the score.

Some of those cases can be expressed directly with the current implementation:

```java
@Test
void acceptsEmptyInputAndTextWithoutBrackets() {
    assertTrue(validator.isBalanced(""));
    assertTrue(validator.isBalanced("ordinary text"));
}

@Test
void acceptsAllSupportedBracketTypes() {
    assertTrue(validator.isBalanced("()[]{}"));
    assertTrue(validator.isBalanced("text ({[value]})"));
}
```

An unmatched closing bracket needs an explicit contract before adding its test. The current implementation calls `stack.pop()` while the stack is empty, so `isBalanced(")")` throws `EmptyStackException` instead of returning `false`. If returning `false` is the intended behavior, the implementation should guard the pop and the test should make that requirement visible:

```java
@Test
void rejectsAnUnmatchedClosingBracket() {
    assertFalse(validator.isBalanced(")"));
}
```

This is the useful feedback loop: the report does not say “write more tests” in the abstract. It points to a changed behavior and lets the developer decide whether that behavior matters.

## Why AI-generated tests often survive mutants

AI-generated tests commonly have several characteristics that mutation testing exposes.

### They follow the implementation too closely

When the same assistant writes the method and its tests, it can repeat the implementation's assumptions in both places. The test may reconstruct the expected value with the same formula as the production code. If both use the same mistake, the test passes without checking the intended behavior.

An independent expected value is safer:

```java
assertTrue(validator.isValid("X0000001R"));
```

The assertion describes a known business example rather than calling the same calculation to construct its expected result.

### They prefer the easiest path through a branch

A generated test often uses ordinary values because they make a clean example. Boundary values, empty collections, duplicate keys, time zones, retries, and malformed input require more interpretation. Those are the cases that often kill conditional, boundary, and return-value mutants.

### They assert that something happened

Mocks can make a test look detailed while checking little. A test may verify that a dependency was called and still miss an incorrect result, an incorrect argument, or an extra call. Mutation testing can reveal this when changes to the returned value or branch condition survive.

### They optimize for coverage-shaped output

An assistant can see that an uncovered line needs a test and create one that executes the line. Execution alone does not establish a useful assertion. A test that only checks that no exception was thrown may leave several mutations alive.

Mutation testing gives the developer a concrete failure to investigate. The question changes from whether a test looks complete to why a specific change survived.

## Reproducing the example

The repository targets Java 11 and keeps the build self-contained with Docker. From its root, the README runs both unit tests and PIT through the Maven `test` phase:

```bash
docker run --rm \
  -v "$PWD":/workspace \
  -v "$HOME/.m2":/root/.m2 \
  -w /workspace \
  maven:3.9-eclipse-temurin-11 \
  mvn test
```

PIT is configured in the root POM with all mutators enabled, and its `mutationCoverage` goal is bound to `test`. Reports are written below each module's `target/pit-reports/` directory. A single module can be run with `mvn -pl validators test` or `mvn -pl text-analysis test` inside the same container.

Running mutation testing after the ordinary suite is green is important: PIT changes the program deliberately. The first run may be slow, so start with the code changed in the current task and use incremental features where available.

## How to use mutation testing with AI-generated code

I use the following workflow when an assistant generates an implementation and its tests.

### 1. Define the behavior first

Write down the inputs, outputs, errors, side effects, and boundaries that matter. This can be a short list or a few examples. The list gives the review a reference that is independent from the generated code.

For the bracket validator, that list should answer what happens for empty input, ordinary text, correctly nested pairs, mismatched pairs, an unmatched closing bracket, and an unmatched opening bracket. For the document validator, it should include valid and invalid DNI values, all NIE prefixes, case handling, and malformed input.

### 2. Ask the assistant for the initial implementation and tests

The assistant can save time on setup, fixtures, and repetitive cases. I would still read the code before running mutation testing. Look for assumptions about rounding, null handling, time, ordering, concurrency, and external calls.

### 3. Run ordinary checks

Compile the code and execute the normal test suite. Static analysis and ordinary tests catch syntax errors, type errors, and known regressions. Run mutation testing after the tests are green because it changes the program deliberately.

### 4. Run the mutation tool

Tools such as [PIT](https://pitest.org/) for Java, [Stryker](https://stryker-mutator.io/) for JavaScript and TypeScript, and [mutmut](https://mutmut.readthedocs.io/) for Python can generate mutants and run the test suite against them.

Start with the code changed in the current task. Exclude generated code or framework glue that does not contain application decisions, and inspect the report rather than treating its score as the outcome.

### 5. Read every meaningful surviving mutant

For each survivor, decide which case applies:

1. The behavior matters and the test suite needs a new test.
2. The behavior matters, but the implementation or test needs to be changed.
3. The mutant is equivalent to the original code and should be excluded.
4. The code is too complicated to test clearly and should be simplified.

An equivalent mutant produces the same observable behavior even though the source code changed. These mutants are a limitation of mutation testing, not evidence that the suite is weak. Exclude them with a documented reason rather than adding an assertion that only exists to satisfy the report.

### 6. Give the survivors back to the assistant carefully

A mutation report is useful input for another AI pass. Ask the assistant to explain the behavior represented by a surviving mutant and propose a test for that behavior. Include the original code, the mutant diff, and the requirement. Asking only for a higher mutation score can produce assertions coupled to the tool instead of tests that describe the domain.

Review the proposed test and run the mutation tool again. The test should fail against the mutant for a reason that matches the requirement.

## Mutation score is not a target by itself

A high score can still hide weak tests. Some mutants are easy to kill without checking meaningful behavior, and some important defects are outside the mutation operators enabled by the tool. A low score can also be acceptable for code that contains defensive branches, integration boundaries, or equivalent mutants that are expensive to eliminate.

I prefer using mutation testing in three ways:

* As a review tool for newly generated or heavily changed code.
* As a way to find weak tests in business logic that is expected to change.
* As a regression check when a test suite is being rewritten by an AI assistant.

The goal is not to make every line survive a fixed number of mutations. The goal is to understand which changes the tests would detect and whether those changes represent behavior the system promises to preserve.

## Conclusion

The `mutant-testing` example makes the distinction concrete: both modules reach 100% line coverage, yet PIT still finds surviving mutants. The tests execute the code, but they do not fully specify its boundaries, negative cases, or algorithmic variation.

That distinction matters most when one AI assistant created both sides of the test. A passing suite can share the implementation's assumptions. Surviving mutants expose those assumptions, point to missing examples, and create specific work for the next review.

Use the mutation report as evidence, not as a score to optimize blindly. Keep the tests that describe real behavior, investigate the survivors, document equivalent mutants, and let the final decision come from the requirements the software must meet.

## References

1. [Mutant Testing example repository](https://github.com/tomasalmeida/mutant-testing)
2. [PIT Mutation Testing](https://pitest.org/)
3. [Stryker Mutation Testing](https://stryker-mutator.io/)
4. [mutmut documentation](https://mutmut.readthedocs.io/)
