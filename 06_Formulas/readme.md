# Vector Mathematics Formula Reference

## Implementation guide for C++ Vector Library

This document contains the essential formulas needed to build a complete vector mathematics library in C++ for game development.

In the very end of this file you'll find a useful implementation checklist so that you can get a know-how of what to start with.

---

## Basic vector structure

### 2D Vector:

```c++
struct Vec2 {
    float x;
    float y;
}
```

### 3D Vector:

```c++
struct Vec3 {
    float x;
    float y;
    float z;
}
```

---

# Fundamental Operations

## Vector Addition

Formula:

```
A + B = (Ax + Bx, Ay + By, Az + Bz)
```

Usage example: Combining movements, forces or displacements.

---

## Vector Subtraction

Formula:

```
A - B = (Ax - Bx, Ay - By, Az - Bz)
```

Usage example: Finding direction from B to A

```
direction = targetPosition - currentPosition
```

---

## Scalar Multiplication

Formula:

```
k * A = (k * Ax, k * Ay, k * Az)
```

Usage example: Scaling speed, forces, or any vector quantity

---

## Scalar Division

Formula:

```
A / k = (Ax / k, Ay / k, Az / k)
```

Note: Only valid when `k ≠ 0`

---

## Magnitude and Distance

Vector Magnitude (Length) formula:

```
|A| = √(Ax² + Ay² + Az²)
```

2D Version:

```
|A| = √(Ax² + Ay²)
```

Usage example: Finding speed, distance, or vector strength

---

## Distance Between Two Points

Formula:

```
distance = |B - A| = √((Bx - Ax)² + (By - Ay)² + (Bz - Az)²)
```

Usage: Finding how far apart two positions are

---

## Squared Magnitude (Optimization)

Formula:

```
|A|² = Ax² + Ay² + Az²
```

Usage: When comparing distances (avoids expensive square root)

```
Instead of:     if (|A| < |B|)
Use:            if (|A|² < |B|²)
```

---

# Normalization

## Normalize a Vector

Formula:

```
normalized(A) = A / |A| = (Ax/|A|, Ay/|A|, Az/|A|)
```

**Result**: A unit vector (magnitude = 1) pointing in the same direction.<br>
**Important**: Check that `|A| ≠ 0` before normalizing.<br>
Usage:

* Separating direction from magnitude
* Movement directions
* Surface normals

---

# Dot Product

Formula:

```
A · B = Ax * Bx + Ay * By + Az * Bz
```

2D Version:

```
A · B = Ax * Bx + Ay * By
```

Alternative Formula (using angle):

```
A · B = |A| * |B| * cos(θ)
```

---

## Dot Product (Normalized Vectors)

Formula:

```
normalize(A) · normalize(B) = cos(θ)
```

**Result Range**: [-1, 1]

* `1` = vectors point in same direction (θ = 0°)
* `0` = vectors are perpendicular (θ = 90°)
* `-1` = vectors point in opposite directions (θ = 180°)

Usage example: Field of view checks, alignment detection

## Angle Between Vectors

Formula:

```
θ = arccos((A · B) / (|A| * |B|))
```

For normalized vectors:

```
θ = arccos(A · B)
```

**Result**: Angle in radians (convert to degrees: **θ * 180 / π**)

---

## Projection of A onto B (Scalar Projection)

Formula:

```
scalar_projection = (A · B) / |B|
```

For normalized B:

```
scalar_projection = A · normalize(B)
```

Result: How much of A points along B (can be negative)

---

## Projection of A onto B (Vector Projection)

Formula:

```
vector_projection = ((A · B) / |B|²) * B
```

For normalized B:

```
vector_projection = (A · B) * B
```

Result: The component of A that lies along B

---

# Cross Product (3D Only)

Formula:
```
A × B = (
Ay * Bz - Az * By,
Az * Bx - Ax * Bz,
Ax * By - Ay * Bx
)
```

Properties:

* Result is perpendicular to both A and B
* Direction determined by right-hand rule
* A × B = -(B × A) (order matters!)
* |A × B| = |A| * |B| * sin(θ)

Usage:

* Finding perpendicular directions
* Camera orientation (right, up, forward vectors)
* Surface normals from triangles

---

## Cross Product Magnitude

Formula:
```
|A × B| = |A| * |B| * sin(θ)
```

Geometric meaning: Area of parallelogram formed by A and B

---

## 2D Cross Product (Scalar Result)

Formula:
```
A × B = Ax * By - Ay * Bx
```

Result:

* Positive = B is counter-clockwise from A (left turn)
* Negative = B is clockwise from A (right turn)
* Zero = vectors are parallel

Usage: Left/right detection, turn direction

---

# Reflection

### Reflect Vector Across Surface

Formula:
```
reflected = V - 2 * (V · N) * N
```
Requirements:

* `V` = incoming vector (usually velocity)
* `N` = surface normal (must be normalized)

Usage:

* Ball bouncing
* Light reflection
* Bullet ricochets

---

# Interpolation
## Linear Interpolation (Lerp)

Formula:
```
lerp(A, B, t) = A + t * (B - A)
```
Alternative form:
```
lerp(A, B, t) = (1 - t) * A + t * B
```
Parameters:

* `t = 0` returns A
* `t = 1` returns B
* `t = 0.5` returns midpoint

Usage: Smooth movement, animation blending

---

## Spherical Linear Interpolation (Slerp)

Formula:
```
slerp(A, B, t) = (sin((1-t) * θ) / sin(θ)) * A + (sin(t * θ) / sin(θ)) * B
```

where `θ = arccos(A · B)` (A and B must be normalized).

Usage: Smooth rotation interpolation, camera movement.

Note: For small angles, use lerp instead (more efficient)

# Clamping and Limiting

## Clamp Magnitude

Formula:
```
if (|V| > maxLength) {
    V = normalize(V) * maxLength
}
```
Usage: Limiting speed, capping forces

---

## Clamp Vector to Range

Formula (per component):
```
clamp(V, min, max) = (
    clamp(Vx, min, max),
    clamp(Vy, min, max),
    clamp(Vz, min, max)
)
```

where `clamp(value, min, max)` returns value constrained to [min, max]

---

# Useful Utility Functions

## Check if Vectors are Approximately Equal

Formula:
```
bool approximately_equal(A, B, epsilon) {
return |A - B| < epsilon
}
```

Usage: Handling floating-point precision errors
Typical epsilon: `0.0001f` or `1e-6f`

---

## Check if Vector is Zero

Formula:
```
bool is_zero(V, epsilon) {
return |V|² < epsilon²
}
```

Note: Using squared magnitude avoids square root

---

## Perpendicular Vector (2D)

Formula:
```
perpendicular(A) = (-Ay, Ax)  // 90° counter-clockwise
// or
perpendicular(A) = (Ay, -Ax)  // 90° clockwise
```

Usage: Finding tangent directions, 2D normals

---

# Distance and Direction Helpers

## Direction From A to B (Normalized)

Formula:

```
direction = normalize(B - A)
```

Safe version (checking for zero):

```
Vector3 direction_to(A, B) {
    Vector3 diff = B - A;
    if (|diff| < epsilon) return Vector3(0, 0, 0);
    return normalize(diff);
}
```

---

## Move Towards Target

Formula:

```
new_position = current + normalize(target - current) * speed * deltaTime
```

With max distance:

```
Vector3 direction = target - current;
float distance = |direction|;

if (distance <= maxDistance) {
    return target;
}

return current + normalize(direction) * maxDistance;
```

# Rotation (2D)

## Rotate Vector by Angle

Formula:

```
rotated = (
    Vx * cos(θ) - Vy * sin(θ),
    Vx * sin(θ) + Vy * cos(θ)
)
```

Usage: 2D rotation, aiming, turrets

## Special Vectors

### Common Unit Vectors (3D)

```
Vector3 right   = (1, 0, 0);
Vector3 up      = (0, 1, 0);
Vector3 forward = (0, 0, 1);
Vector3 zero    = (0, 0, 0);
Vector3 one     = (1, 1, 1);
```

Note: Forward direction depends on the coordinate system

* Unity uses (0, 0, 1) for Z-forward
* Some engines use (0, 0, -1) or (1, 0, 0)

---

# Performance Notes

### Expensive Operations (avoid in inner loops)

1. Square root - Used in magnitude calculation
    * Use squared magnitude when possible

2. Division - Slower than multiplication
    * Normalize once and reuse if possible

3. Trigonometric functions - arccos, arcsin, etc.
   * Cache results when possible

### Optimization Tips

```
// Avoid:
if (|A - B| < distance) { ... }

// Use instead:
if (|A - B|² < distance²) { ... }
```

```
// Avoid normalizing repeatedly:
for (each frame) {
    Vector3 dir = normalize(target - pos);  // Don't recalculate if target doesn't move
}

// Cache instead:
Vector3 dir = normalize(target - pos);

for (each frame) {
    use dir;  // Update only when target moves
}
```

---

# Implementation Checklist

When building your C++ vector library, implement these in order:

### Essential (Core Math)

* Vector2 and Vector3 structures
* Addition, subtraction
* Scalar multiplication, division
* Magnitude, squared magnitude
* Normalize
* Dot product
* Cross product (3D)
* Distance

### Important (Common Operations)

* Lerp
* Reflection
* Angle between vectors
* Projection (scalar and vector)
* Clamp magnitude
* Perpendicular (2D)

### Useful (Utilities)

* Approximately equal
* Is zero check
* Direction to
* Move towards
* Rotate (2D)
* Slerp

### Advanced (Optional)

* Component-wise operations (min, max, abs)
* Smooth damp
* Bezier curves
* Catmull-Rom splines