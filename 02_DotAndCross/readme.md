# Dot & Cross Products

## Dot Product

### What it is

The dot product is an operation between two vectors that produces a scalar (a single number).

It measures how aligned two vectors are.

---

### Intuition

The dot product answers:

> “How much does vector A point in the same direction as vector B?”

* Large positive value → vectors point in similar directions
* Zero → vectors are perpendicular
* Negative value → vectors point in opposite directions

### Why dot product is important in games

Dot product is used constantly for:

* Field of view checks
  * Is the enemy in front of the player?
* Lighting
  * How much light hits a surface
* Reflection
  * How much velocity goes into a surface
* AI
  * Facing direction vs target direction

---

### Key idea

When vectors are normalized, the dot product becomes especially meaningful:

* `1` → same direction
* `0` → perpendicular
* `-1` → opposite direction

Un-normalized vectors still give the same directional information, but bear in mind that they are scaled by their magnitudes.<br>
This can be useful when you need both direction and distance information in one operation.

---

### Example: Field of view check

To check if an enemy is in front of the player:

1. Calculate direction to enemy: `enemyPosition - playerPosition`
2. Normalize both this direction and the player's forward vector
3. Take the dot product

```csharp
var dotProduct = normalize(enemyPos - playerPos) · normalize(playerForward)

if (dotProduct > 0.7)  // roughly 45° cone in front
    enemyInView = true;
```

This is how stealth games determine if guards can see the player.

---

## Cross Product

### What it is

The cross product takes two vectors and produces a new vector that is:

* Perpendicular to both input vectors
* Its magnitude equals the area of the parallelogram formed by the two vectors
* Oriented according to the right-hand rule (in 3D)
  * More information in the right-hand rule folder

---

### Intuition

The cross product answers:

> “Which direction is ‘sideways’ relative to these two directions?”

---

### Why cross product is useful in games

Cross product is used for:

* Finding right/left directions
* Camera orientation
* Object orientation in 3D
* Surface tangent generation
* Determining rotation direction

In 2D games, the cross product reduces to a scalar value that indicates:
* Positive → left turn
* Negative → right turn

---

## Important note

* Cross product exists primarily in 3D
* It produces a vector, not a scalar
* It defines orientation, not similarity

### Order matters

Unlike the dot product, the cross product is not commutative:

* `A × B` and `B × A` point in **opposite directions**
* They have the same magnitude but flipped orientation

This is why cross product is useful for determining left vs right:
* `forward × right = up`
* `right × forward = down`

Always maintain consistent ordering when using cross product for orientation.

---

## Reflection (mirroring a vector)
### What reflection means

Reflection takes a vector (usually velocity or direction) and mirrors it across a surface.

It answers:

> “If this vector hits this surface, which direction should it leave?”

---

### Reflection in games

Reflection is used for:

* Ball bouncing (Pong, Breakout)
* Lasers
* Raycasts
* Bullet ricochets
* Physics responses

---

### How reflection works conceptually

Reflection uses:

* The incoming vector
* The surface normal
* The dot product

The normal tells us how the surface is oriented. <br>
And how much of the vector goes toward/into the surface along the normal direction.

The reflected vector removes the part that goes into the surface and mirrors it outward.

---

### Why reflection is powerful

Reflection turns:

* abstract math
* into visible, intuitive behavior

When you understand reflection, you also understand:

* normals
* dot product
* collision response

All at once.

---

## How these concepts work together in games

In a typical collision:

1. Velocity is a vector
2. Surface normal describes the surface orientation
3. Dot product measures how strongly the object hits the surface
4. Reflection produces a new velocity
5. Rendering visualizes the result

This chain is at the heart of:

* physics
* movement
* interaction
* realism

---

### Final takeaway

Normals describe surfaces.<br>
Dot product describes alignment.<br>
Cross product describes orientation.<br>
Reflection describes response.

Together, these operations turn math into motion - and motion into games.