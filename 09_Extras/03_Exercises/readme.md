# AAA Test Pattern Exercises
## Arrange · Act · Assert

> **Quick Reference:**
> - **Arrange** — Set up your test data, objects, and conditions
> - **Act** — Call the single function or method being tested
> - **Assert** — Verify the result matches what you expected

---

## Exercise 1: Vector Subtraction

### Goal
Write a complete AAA test that verifies `VectorSubtract` correctly subtracts one `Vec3` from another.

### What to Test
- Subtracting `(4, 5, 6)` from `(10, 8, 7)` should give `(6, 3, 1)`
- The result should NOT modify the original vectors

### Starter Code
```cpp
void TestVectorSubtract()
{
    // ─────── ARRANGE ───────
    Vec3 vectorA = { ?, ?, ? };   // Fill in values
    Vec3 vectorB = { ?, ?, ? };   // Fill in values
    Vec3 expected = { ?, ?, ? };  // What result do you expect?

    // ─────── ACT ───────
    Vec3 result = ???;             // Call the correct function

    // ─────── ASSERT ───────
    assert(result.x == expected.x, "X should be 6");
    assert(???, ???);              // Complete Y and Z assertions
}
```

### Hints
1. Look at the function signature: `Vec3 VectorSubtract(Vec3 a, Vec3 b)`
2. Subtraction is: `(a.x - b.x, a.y - b.y, a.z - b.z)`
3. The Arrange section should clearly state the expected answer **before** calling Act
4. Each component (x, y, z) needs its own Assert

---

## Exercise 2: VectorScale and Magnitude

### Goal
Write **two separate** AAA tests: one for `VectorScale` and one that verifies the scaled vector has the correct magnitude.

### What to Test
- Scaling vector `(1, 0, 0)` by `5` should give `(5, 0, 0)`
- The magnitude of `(5, 0, 0)` should be `5`
- Scaling by `0` should produce a zero vector

### Starter Code
```cpp
// Test 1: Scaling
void TestVectorScale()
{
    // ─────── ARRANGE ───────
    Vec3 vector = ???;
    float scalar = ???;
    Vec3 expected = ???;

    // ─────── ACT ───────
    Vec3 result = ???;

    // ─────── ASSERT ───────
    // Write your assertions here
}

// Test 2: Magnitude after scale
void TestScaledMagnitude()
{
    // ─────── ARRANGE ───────
    // Hint: arrange your data so the expected magnitude is easy to calculate

    // ─────── ACT ───────

    // ─────── ASSERT ───────
}
```

### Hints
1. Why **two** tests instead of one? Each test should verify exactly **one behaviour**
2. For the magnitude test, use a simple vector like `(3, 4, 0)` where the result = 5
3. Remember to test the edge case: scaling by 0

---

## Exercise 3: Dot Product and Angle Detection

### Goal
Write **three** AAA tests that verify the dot product behaves correctly for different geometric relationships between vectors.

### What to Test

| Scenario | Vectors | Expected Result |
|---|---|---|
| Perpendicular vectors | X-axis and Y-axis | `0.0` (90° apart) |
| Same direction | `(1,0,0)` and `(1,0,0)` | `1.0` (both normalized) |
| Opposite direction | `(1,0,0)` and `(-1,0,0)` | `-1.0` (facing away) |

### Starter Code
```cpp
// Test 1: Perpendicular
void TestDotProduct_Perpendicular()
{
    // ─────── ARRANGE ───────
    Vec3 vectorA = ???;
    Vec3 vectorB = ???;
    float expected = ???;

    // ─────── ACT ───────
    float result = ???;

    // ─────── ASSERT ───────
    assert(???, ???);
}

// Test 2 and 3: Complete these yourself
void TestDotProduct_SameDirection() { ??? }
void TestDotProduct_OppositeDirection() { ??? }
```

### Hints
1. Dot product = `(a.x * b.x) + (a.y * b.y) + (a.z * b.z)`
2. For the "same direction" test, use normalized vectors (length = 1)
3. Dot product `> 0`: vectors face the same general direction
4. Dot product `< 0`: vectors face opposite directions
5. Dot product `= 0`: vectors are exactly perpendicular
6. **Game connection:** In Pong you can use dot product to check if the ball is moving toward a paddle

---

## Exercise 4: Ball Reflection in Pong

### Goal
Write AAA tests that verify `VectorReflect` works correctly for all the surfaces in your Pong game.

### Pong Surfaces to Test
- **Top wall:** normal points down `(0, -1, 0)`
- **Bottom wall:** normal points up `(0, 1, 0)`
- **Left paddle:** normal points right `(1, 0, 0)`
- **Right paddle:** normal points left `(-1, 0, 0)`

### Starter Code
```cpp
void TestVectorReflect_BottomWall()
{
    // ─────── ARRANGE ───────
    // Ball moving down-right at 45 degrees
    Vec3 ballVelocity = { 1.0f, -1.0f, 0.0f };
    Vec3 wallNormal = ???;   // Bottom wall normal
    Vec3 expected = ???;     // What direction should the ball bounce?

    // ─────── ACT ───────
    Vec3 result = ???;

    // ─────── ASSERT ───────
    // Complete the assertions
}

// Now write tests for the other surfaces:
void TestVectorReflect_TopWall() { ??? }
void TestVectorReflect_LeftPaddle() { ??? }
void TestVectorReflect_RightPaddle() { ??? }
```

### Think About It
Before writing the Assert, work out manually what direction the ball should bounce. **Draw it on paper first!**

### Hints
1. `VectorReflect` formula: `result = v - 2 * dot(v, n) * n`
2. The normal must point **away** from the surface (toward the ball)
3. If the ball hits the top wall going up-right `(+y)`, it should bounce down-right `(-y)`
4. If the ball hits the right paddle going right `(+x)`, it should bounce left `(-x)`
5. The X component should **not** change when hitting top/bottom walls
6. The Y component should **not** change when hitting left/right paddles

---

## Exercise 5: Find the Bug

### Goal
Each test below is wrong in some way. Find the problem and write the corrected version.

**Instructions:**
1. Read each test carefully
2. Identify **what** is wrong and **which section** (Arrange, Act, or Assert) the problem is in
3. Write the corrected version

---

### Test A
```cpp
void TestVectorNormalize()
{
    // ─────── ARRANGE ───────
    Vec3 vector = { 5.0f, 0.0f, 0.0f };

    // ─────── ACT ───────
    Vec3 normalized = VectorNormalize(vector);
    Vec3 scaledBack = VectorScale(normalized, 5.0f);

    // ─────── ASSERT ───────
    assert(VectorMagnitude(normalized), 1.0f, "Should have magnitude 1");
}
```

**What is wrong?**

---

### Test B
```cpp
void TestVectorAdd()
{
    // ─────── ARRANGE ───────
    Vec3 vectorA = { 1.0f, 2.0f, 3.0f };
    Vec3 vectorB = { 4.0f, 5.0f, 6.0f };

    // ─────── ACT ───────
    Vec3 result = VectorAdd(vectorA, vectorB);

    // ─────── ASSERT ───────
    Assert(result.x == 5.0f, "X should be 5");
    Assert(result.y == 8.0f, "Y should be 8");
    Assert(result.z == 9.0f, "Z should be 9");
}
```

**What is wrong?**

---

## Exercise 6: Write Your Own Tests

### Goal
Choose any **three** functions from your vector math library that you have not tested yet. Write a complete AAA test for each one.

### Requirements
- Each test must have clearly labeled Arrange, Act, and Assert sections
- Each test must follow the naming convention: `MethodName_Scenario_ExpectedBehavior`
- You must test at least **one edge case** (zero vector, t=0, t=1, etc.)
- All tests must run and pass in your console test project

### Functions to Consider

| Function | Interesting Test Case | Edge Case to Test |
|---|---|---|
| `VectorLerp` | `t = 0.5` gives exact midpoint | `t = 0` returns start, `t = 1` returns end |
| `VectorDistance` | Known triangle sides (3, 4, 5) | Distance from a point to itself = 0 |
| `VectorCross` | X cross Y gives Z | Parallel vectors give zero vector |
| `VectorNormalize` | Any non-zero vector has length 1 | Zero vector returns zero vector |
| `VectorScale` | Scaling by -1 reverses direction | Scaling by 0 returns zero vector |
| `VectorDot` | Two identical vectors give length² | Zero vector gives 0 |

### Challenge: The Perfect Test

A great test has these qualities:
- Anyone can read it and instantly understand **what** is being tested
- The expected value is calculated **before** calling the function
- It tests **one behaviour** only
- It has a name that reads like a sentence
- It would catch a real bug if your function was broken

---

## Summary

| Exercise | Focus | Key Skill |
|---|---|---|
| 1 | VectorSubtract | Writing a basic AAA test |
| 2 | VectorScale + Magnitude | One test per behaviour |
| 3 | Dot Product | Testing geometric relationships |
| 4 | VectorReflect in Pong | Testing real game logic |
| 5 | Find the Bug | Reading and critiquing tests |
| 6 | Write your own | Designing tests independently |

---

> **Remember:** The goal of the AAA pattern is not just to test that your code works.
> It is to write tests so clear that they serve as **documentation** for how your code **should** work.
> If your test is hard to read, it is harder to trust.