# AAA Test Pattern: Arrange, Act, Assert

## Introduction

The **AAA (Arrange, Act, Assert)** pattern is a standard way of structuring tests so
they are easy to read, understand, and maintain. Every test you write should follow
this structure.

**Why AAA?**
- **Clarity:** Anyone can read the test and understand what it does
- **Consistency:** All tests look and feel the same
- **Debuggability:** When a test fails, you know exactly where to look
- **Documentation:** Tests become living examples of how your code should behave

---

## The Three Phases

### 1. Arrange
Set up everything the test needs before anything happens.
- Create input values
- Initialize objects
- Define what you expect the result to be

### 2. Act
Execute **one** thing — the function or behaviour being tested.
- This should almost always be a **single line of code**
- If you find yourself writing multiple lines here, you are probably testing too much at once

### 3. Assert
Verify the result matches your expectation.
- Compare actual output to expected output
- One assertion per component where necessary (x, y, z each need checking)
- If this fails, the **Act** produced the wrong result

---

## Visual Structure

```cpp
void TestSomething()
{
    // ─────── ARRANGE ───────
    // Set up inputs and expected values

    // ─────── ACT ───────
    // Call the function being tested

    // ─────── ASSERT ───────
    // Check the result is correct
}
```

---

## A Simple Example

Here is a complete AAA test for a made-up `Add` function:

```cpp
void TestAdd_TwoPositiveNumbers_ReturnsCorrectSum()
{
    // ─────── ARRANGE ───────
    float a = 3.0f;
    float b = 4.0f;
    float expected = 7.0f;

    // ─────── ACT ───────
    float result = Add(a, b);

    // ─────── ASSERT ───────
    AssertEqual(result, expected, "3 + 4 should equal 7");
}
```

Notice:
- The **expected value is defined in Arrange**, before the function is ever called
- The **Act is one line**
- The **Assert compares result to expected**

---

## Test Naming

Good test names follow this pattern:

```
MethodName_Scenario_ExpectedBehavior
```

Examples:
```
Add_TwoPositiveNumbers_ReturnsCorrectSum
Add_NegativeAndPositive_ReturnsCorrectSum
Divide_ByZero_ReturnsZero
```

A good name should read like a sentence. If someone reads only the name, they should
understand what is being tested and what the correct outcome is — without reading the
code at all.

---

## The TestRunner

Your test project might use a `TestRunner` class with, for example, these assert methods:

```cpp
// Check a boolean condition is true
runner.Assert(condition, "test name", "failure message");

// Compare two floats (handles floating point precision)
runner.AssertEqual(actual, expected, "test name");

// Compare two Vec3 values
runner.AssertVec3Equal(actual, expected, "test name");
```

Always use `AssertEqual` when comparing floats — never use `==` directly. Floating
point arithmetic can produce tiny rounding differences that would cause a correct
result to incorrectly fail.

---

## One Test, One Behaviour

Each test should verify **one thing only**. If you want to test two different
behaviours, write two separate tests.

```cpp
// Good — tests one thing
void TestVectorScale_ScaleByTwo_DoublesAllComponents() { ... }

// Good — tests one thing
void TestVectorMagnitude_UnitVector_ReturnsOne() { ... }

// Bad — tests two things at once
void TestScaleAndMagnitude() { ... }
```

This matters because when a test fails, you should immediately know **what broke**
without needing to investigate further.

---

## Common Mistakes

### Multiple lines in Act
```cpp
// Bad — two actions!
Vec3 normalized = VectorNormalize(vector);
Vec3 scaled = VectorScale(normalized, 2.0f);  // Second action
```
Split into two tests, or combine into a single named operation.

---

### Calculating the expected value inside Assert
```cpp
// Bad — expected is computed on the fly
Assert(result.x == vectorA.x + vectorB.x, "...");

// Good — expected is a hardcoded known answer set in Arrange
Vec3 expected = { 5.0f, 7.0f, 9.0f };
Assert(result.x == expected.x, "...");
```
The expected value should always be a **known answer you worked out beforehand**,
not something calculated from the same inputs during the test.

---

### No separation between phases
```cpp
// Bad — everything blurred together, hard to read
void TestVectorAdd() {
    Vec3 result = VectorAdd({1,2,3}, {4,5,6});
    Assert(result.x == 5, "...");
}
```
If you used section comments on your first test, then the rest should follow the same pattern.
Separation adds readability and clumping everything together will make it hard to understand and quickly assess.

---

## In Unity with C#

Unity uses the **NUnit** test framework. The structure is identical — only the
syntax changes:

```csharp
using NUnit.Framework;

public class MyTests
{
    [Test]
    public void MethodName_Scenario_ExpectedBehavior()
    {
        // ─────── ARRANGE ───────
        // set up values

        // ─────── ACT ───────
        // call the function

        // ─────── ASSERT ───────
        Assert.AreEqual(expected, result, tolerance, "message");
    }
}
```

The `[Test]` attribute tells Unity's Test Runner that this method is a test. Tests
appear in **Window → General → Test Runner** and can be run with a single click.

Use `Assert.AreEqual(expected, actual, tolerance)` for floats — the tolerance
argument handles floating point precision the same way `AssertEqual` does in C++.

---

## Summary

| Phase | Purpose | Typical Length |
|---|---|---|
| **Arrange** | Set up inputs and expected values | 2–5 lines |
| **Act** | Call the function being tested | 1 line |
| **Assert** | Verify the result | 1–4 lines |

**The golden rule:** If you cover up the test name and read only the code, the
Arrange, Act, and Assert comments should make the test's purpose completely clear on
their own.