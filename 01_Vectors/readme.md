# Vectors

## What are they really?

### In simple terms

A vector is a mathematical object that represents movement or influence in space.
It has both a magnitude (length) and a direction.

A vector answers the question:

> From here - in which direction - and how far?

Geometrically, a vector is often visualized as a directed line segment:
* The length represents the magnitude
* The arrow represents direction

---

# Key aspects of a vector

### Representation:

#### Geometric (visual)

Vectors are commonly drawn as arrows:
* →, ←, ↑, ↓, ↗, ↖, ↘, ↙

These arrows are not objects, they describe change, not things.

---

#### Numerical (component form)

Vectors are stored as numbers
* 2D: `(x, y)` → e.g `(1, 5)`
* 3D: `(x, y, z)` → e.g `(3, 6, 9)`

Each component represents how much to move along each axis.

A vector can always be decomposed into its components:
* `x`: movement along the horizontal axis
* `y`: movement along the vertical axis
* `z`: depth (in 3D)

---

### Vectors vs Scalars

* A scalar has only magnitude <br>
(speed, time, mass, health)

* A vector has magnitude and direction <br>
(velocity, force, acceleration)

This distinction is fundamental in games.

---

## Common vectors in games

Vectors are everywhere in game development:
* Velocity
  * Speed with direction
* Acceleration
* Force
* Gravity
* Input direction
* Surface normals
* Camera forward / right / up
* Light direction

---

## Position vs displacement

* Position <br>
A location in space (a point)
* Displacement (vector) <br>
A change between two positions

In code, positions are often stored using vector types, but conceptually a vector describes change, not location.

---

## Mathematical operations on vectors

### Addition and subtraction

* Addition combines movements
* Subtraction finds the difference between positions or directions

In games:

* `targetPosition - currentPosition` gives a direction vector

---

### Scalar multiplication (scaling)

Multiplying a vector by a scalar:

* Keeps direction
* Changes magnitude

Used constantly for:
* Speed
* Forces
* Time-based movement

---

## Why vectors cannot be divided

There is no meaningful way to divide one vector by another geometrically.

Division would require:

* A unique direction
* A unique magnitude

Which does not exist for vector ÷ vector.

Because of this, true vector division does not exist in mathematics.

Instead, situations that look like division are handled in other ways.

---

## Vector resolution / decomposition

Vector resolution means breaking a vector into components along chosen directions.

Example:

* A movement vector can be decomposed into:
  * horizontal movement
  * vertical movement

This is how:

* input becomes movement
* velocity affects position
* forces combine

In physics and games, resolution is how multiple influences act together.

---

## Scalar projection

### Projection

The scalar projection measures the magnitude of one vector projected onto another, representing how much of the first vector points in the direction of the second.<br>
It's calculated as the dot product of the vectors divided by the magnitude of the target vector.<br>
We'll learn more about the dot product later.

Scalar projection answers:

> “How much of vector A points in the direction of vector B?”

---

## Normalization

Dividing a vector by its own magnitude produces a unit vector:

* It scales a vector to have a magnitude of 1
* Preserves the original direction

This is the standard and meaningful way to ‘divide’ a vector.

In games, normalization is used to:

* Separate direction from speed
* Prevent diagonal movement from being faster
* Control movement consistently

---

## Element-wise division (programming concept)

In programming, vectors are sometimes divided component-by-component:

```regexp
(x1 / x2, y1 / y2, z1 / z2)
```

This is:

* Not a geometric operation
* Not physically meaningful by default

Element-wise division exists because:

* Computers store vectors as numbers
* Sometimes per-axis scaling is useful

But it should be used deliberately, not casually.

---

## Summary of “division” concepts

* ❌ Vector ÷ Vector → does not exist
* ✅ Vector ÷ Scalar → valid (scaling / normalization)
* ✅ Scalar projection → measures alignment
* ⚠ Element-wise division → programming tool, not math

---

## How vectors relate to graphics and rendering

Everything you see on screen is driven by vectors.

Vertices
* Positions are vectors
* Models are collections of position vectors

Movement
* Velocity vectors move objects
* Acceleration vectors change velocity

Lighting
* Surface normals are vectors
* Light direction is a vector
* Dot product determines brightness

---

## Vectors, matrices, and the camera

Vectors describe what and where. <br>
Matrices describe how vectors are transformed.

---

### Object transformations

An object’s:

* position
* rotation
* scale

are combined into a transformation matrix.

That matrix transforms:

* local vertex vectors <br>
→ world space <br>
→ view space <br>
→ screen space

---

### Camera projection

The camera:

* is defined by vectors (forward, right, up)
* uses matrices to project 3D space into 2D

Perspective projection:

* keeps distant objects smaller
* uses division by depth (after matrix multiplication)

This is why understanding vectors is required before matrices make sense.

---

## How it all fits together

In a game engine:

1. describe positions, directions, and forces
2. Vector math determines movement and interaction
3. Matrices transform vectors between coordinate spaces
4. The camera projects transformed vectors onto the screen
5. The GPU draws the result

Remove vectors, and the entire pipeline collapses.

---

## Final takeaway

> Vectors are not formulas. <br>
They are decisions about direction and magnitude over time.

Understanding vectors means understanding:

* movement
* physics
* cameras
* lighting
* rendering

That is why vectors define modern game development.

---

## Extras

### Surface normals (what is a normal?)
A normal is a vector that points perpendicular (at a 90° angle) to a surface. <br>
In games, a surface normal answers the question:
> "Which way does this surface face?"

--- 

### Simple definition
* A surface normal is a unit vector
* It points directly outward from a surface
* It has direction, but usually no meaningful length beyond 1

Examples:
* A flat floor has a normal pointing straight up
* A wall has a normal pointing sideways
* A sloped surface has a diagonal normal

---

### Why normals matter in games
Normals are essential for:

* Lighting (how bright a surface is)
* Collision response (how objects bounce)
* Physics (sliding, friction, reflection)
* Shading (flat vs smooth surfaces)

Without normals:

* Lighting looks wrong
* Collisions feel incorrect
* Reflections make no sense