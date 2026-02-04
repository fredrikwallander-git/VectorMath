# The Right-hand Rule

## What is it?

The right-hand rule is a convention used in 3D mathematics to determine the direction of the cross product.

When you take the cross product of two vectors:

````ini
C = A × B
````

there are two possible perpendicular directions.
The right-hand rule tells us which one is correct.

Without this rule, orientation in 3D space would be ambiguous.

---

### The Physical Rule

Use your right hand:
1. Point your index finger in the direction of the first vector (A)
2. Point your middle finger in the direction of the second vector (B)
3. Your thumb now points in the direction of the cross product (A × B)
* Index → first vector
* Middle → second vector
* Thumb → result

This only works with your right hand.

--- 

## Why it exists

The right-hand rule is a **convention**, mathematics could work with a left-hand rule,
but everyone agrees on the right-hand rule for consistency across all fields.

In 3D space:
* Two vectors define a plane
* That plane has two possible perpendicular directions

The right-hand rule ensures:
* A consistent orientation
* A shared understanding across math, physics, and graphics
* Predictable rotation and facing behavior

---

## Right-Hand Rule in Game Development

### Coordinate systems

Game engines use different coordinate system conventions:

* Some use right-handed systems (OpenGL, older engines)
* Some use left-handed systems (Unity, Unreal, DirectX)

However, the right-hand rule still applies to the cross product operation itself.

The difference is in how axes are oriented, not in how cross product works.

---

### Example: Finding the “Right” Direction

If you have:

* A forward direction
* An up direction

You can compute the right direction:
````ini
right = forward × up
````

Using the right-hand rule:

* The result points to the object’s right side

This is fundamental for:
* Camera movement
* Player strafing
* Object orientation

---

### Rotation Direction

The right-hand rule also defines positive rotation.

Imagine:

* Curling your right-hand fingers in the direction of rotation
* Your thumb points along the positive rotation axis

This tells you:

* Which way is clockwise vs counter-clockwise
* How angular velocity vectors work

---

### Normals and the Right-Hand Rule

Surface normals are also affected by the right-hand rule.

For a triangle:

* Vertices are ordered (winding order)
* The cross product of two edges gives the surface normal

Reversing the vertex winding order (clockwise ↔ counter-clockwise) flips the normal direction.

This affects:

* Lighting
* Back-face culling
* Collision response

---

### Common Pitfall

Swapping the order of a cross product changes the result:

```ini
A × B ≠ B × A
```

In fact:

```ini
B × A = −(A × B)
```

That means:

* The direction is reversed
* Orientation flips
* Lighting and physics can break

This is one of the most common bugs in 3D math.

---

## 2D Note

In 2D:
* The cross product is reduced to a scalar
* The result conceptually points "into or out of the screen" (along the Z-axis)

The right-hand rule still applies, but it's usually hidden.

Key applications in 2D:
* Left/Right detection
* Determining turn direction (clockwise vs counter-clockwise)
* Calculating signed area

---

## Final Takeaway

> Vectors describe directions.<br>
The cross product describes orientation.<br>
The right-hand rule makes orientation consistent.

Without it, 3D graphics and physics would not work.