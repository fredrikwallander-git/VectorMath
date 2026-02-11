# Physics Simulation Principles

## Introduction

Game physics creates realistic movement through discrete calculations at 60+ FPS. Unlike real physics, game physics is:
- **Discrete** - calculated in time steps
- **Approximate** - "good enough" beats perfect
- **Optimized** - must run fast

---

## Newton's Laws of Motion

### First Law: Inertia
Objects in motion stay in motion unless a force acts on them.

```cpp
// Without forces, velocity stays constant
position += velocity * deltaTime;
```

**Key point:** Need friction to stop, forces to change direction.

---

### Second Law: F = ma
Force equals mass times acceleration.

```cpp
Vector3 acceleration = force / mass;
velocity += acceleration * deltaTime;
position += velocity * deltaTime;
```

**Key insight:** Heavier objects accelerate slower for the same force.

---

### Third Law: Action-Reaction
Every action has an equal and opposite reaction.

```cpp
// Player pushes box
box.ApplyForce(pushForce);
player.ApplyForce(-pushForce);  // Player pushed back
```

**Applications:** Recoil, explosions, rocket propulsion.

---

## Physical Quantities

### Position
**What:** Location in space  
**Type:** Vector
```cpp
Vector3 position = { 10.0f, 0.0f, 5.0f };
```

### Velocity
**What:** Rate of change of position (speed + direction)  
**Type:** Vector  
**Units:** m/s or units/s

```cpp
Vector3 velocity = { 5.0f, 0.0f, 0.0f };  // 5 units/sec in X
position += velocity * deltaTime;
```

### Acceleration
**What:** Rate of change of velocity  
**Type:** Vector  
**Unit:** m/s²  
**Formula:** `a = F / m`

```cpp
float light = 2.0f;
float heavy = 50.0f;
accel1 = force / light; // lighter object = faster acceleration
accel2 = force / heavy; // heavier object = slower acceleration
```

### Mass
**What:** Resistance to acceleration  
**Type:** Scalar
**Unit:** kg  
**Formula:** `m = F / a`

```cpp
mass = force / accel;
```

### Force
**What:** Push/pull causing acceleration  
**Type:** Vector  
**Formula:** `F = m × a`

```cpp
force = mass * accel;
```

---

## Integration: Simulating Motion

### Semi-Implicit Euler (Recommended)

```cpp
void UpdatePhysics(float deltaTime) {
    // Update velocity FIRST
    velocity += (totalForce / mass) * deltaTime;
    
    // Then position with NEW velocity
    position += velocity * deltaTime;
    
    totalForce = { 0, 0, 0 };  // Clear for next frame
}
```

**Why this order:** More stable, better energy conservation.

---

### Fixed Timestep

```cpp
const float FIXED_TIMESTEP = 1.0f / 60.0f; // 1 / X frames = ~0.0166666...
float accumulator = 0.0f;

void GameLoop(float deltaTime) {
    accumulator += deltaTime;
    
    while (accumulator >= FIXED_TIMESTEP) {
        UpdatePhysics(FIXED_TIMESTEP);
        accumulator -= FIXED_TIMESTEP;
    }
    
    Render();
}
```

**Why:** Consistent physics regardless of frame rate.

---

## Common Forces

### Gravity
```cpp
const Vector3 GRAVITY = { 0.0f, -9.81f, 0.0f };
body.acceleration += GRAVITY;  // Direct addition (a = F/m = g)
```

### Friction (Simple)
```cpp
velocity *= 0.98f;  // Lose 2% speed per frame
```

### Air Resistance (Drag)
```cpp
Vector3 drag = -velocity * dragCoefficient;
body.AddForce(drag);
```

### Spring Force (Hooke's Law)
```cpp
Vector3 displacement = anchorPoint - body.position;
float extension = VectorMagnitude(displacement) - restLength;
Vector3 direction = VectorNormalize(displacement);
Vector3 springForce = direction * extension * stiffness;
body.AddForce(springForce);
```

* Robert Hooke stated that the force required to extend or compress a spring by som distance is directly proportional to that distance, expressed as `F = kx` and it defines linear-elastic behaviour.
* Doubling the force also doubles the displacement.

---

## Collision Detection

### Bounding Sphere (Fastest)
```cpp
bool SpheresCollide(Sphere a, Sphere b) {
    Vector3 diff = b.center - a.center;
    float distanceSqr = VectorMagnitudeSqr(diff);
    float radiusSum = a.radius + b.radius;
    return distanceSqr < (radiusSum * radiusSum);
}
```

* A simple spherical volume, defined by a center point and radius which is efficient and fast for collision detection because of it's rotation-invariance.

### AABB (Axis-Aligned Bounding Box)
```cpp
bool AABBsCollide(AABB a, AABB b) {
    return (a.min.x <= b.max.x && a.max.x >= b.min.x) &&
           (a.min.y <= b.max.y && a.max.y >= b.min.y) &&
           (a.min.z <= b.max.z && a.max.z >= b.min.z);
}
```

* AABB's are simple, non-rotated 2D rectangles or 3D boxes that offer low cost and fast collision detection, defined by minimum and maximum vectors.

---

## Collision Response

### Bouncing (Elastic)
```cpp
void HandleCollision(RigidBody& body, Vector3 normal, float restitution) {
    if (VectorDot(body.velocity, normal) < 0) {  // Moving toward surface
        body.velocity = VectorReflect(body.velocity, normal) * restitution;
    }
}
```

**Restitution:** 0 = no bounce, 1 = perfect bounce

### Sliding (No Bounce)
```cpp
void SlideAlongSurface(RigidBody& body, Vector3 surfaceNormal) {
    Vector3 normalVel = surfaceNormal * VectorDot(body.velocity, surfaceNormal);
    Vector3 tangentVel = body.velocity - normalVel;
    body.velocity = tangentVel;  // Keep only sliding component
}
```

### Impulse-Based (Two Objects)
```cpp
void ResolveCollision(RigidBody& a, RigidBody& b, Vector3 normal) {
    Vector3 relVel = b.velocity - a.velocity;
    float velAlongNormal = VectorDot(relVel, normal);
    
    if (velAlongNormal > 0) return;  // Separating
    
    float restitution = min(a.restitution, b.restitution);
    float j = -(1 + restitution) * velAlongNormal;
    j /= (1 / a.mass) + (1 / b.mass);
    
    Vector3 impulse = normal * j;
    a.velocity -= impulse / a.mass;
    b.velocity += impulse / b.mass;
}
```

---

## Simple Physics Engine

### RigidBody Class

```cpp
class RigidBody {
public:
    Vector3 position, velocity, acceleration;
    float mass, inverseMass;  // inverseMass = 1/mass (0 = static)
    float restitution, friction;
    Vector3 forceAccumulator;
    
    RigidBody(float m, float rest = 0.5f, float fric = 0.3f) {
        mass = m;
        inverseMass = (m > 0) ? (1.0f / m) : 0.0f;
        restitution = rest;
        friction = fric;
        position = velocity = acceleration = forceAccumulator = {0,0,0};
    }
    
    void AddForce(Vector3 force) {
        forceAccumulator += force;
    }
    
    void Update(float deltaTime) {
        if (inverseMass == 0) return;  // Static
        
        acceleration = forceAccumulator * inverseMass;
        velocity += acceleration * deltaTime;
        position += velocity * deltaTime;
        forceAccumulator = { 0, 0, 0 };
    }
};
```

---

## Common Patterns

### Character Controller
```cpp
void Move(Vector3 inputDir, float speed, float deltaTime) {
    position += inputDir * speed * deltaTime;
    
    if (!grounded) {
        velocity.y += -9.81f * deltaTime;
        position.y += velocity.y * deltaTime;
    }
    
    if (OnGround()) {
        grounded = true;
        velocity.y = 0;
    }
}

void Jump(float jumpForce) {
    if (grounded) {
        velocity.y = jumpForce;
        grounded = false;
    }
}
```

### Projectile Motion
```cpp
void Update(float deltaTime) {
    velocity.y += -9.81f * deltaTime;  // Gravity
    position += velocity * deltaTime;
}
```

---

## Optimization

### Spatial Partitioning
Divide space into grid cells. Only check collisions within same cell.

### Sleeping Bodies
Put stationary objects to sleep to skip physics updates.

```cpp
if (speed < 0.01f && stillFor > 1.0f) {
    sleeping = true;
}
```

---

## Connection to Your Vector Library

**Every physics operation uses your functions:**

```cpp
// Movement
position += VectorScale(velocity, deltaTime);

// Collision detection  
float distance = VectorDistance(pos1, pos2);
Vector3 normal = VectorNormalize(pos2 - pos1);

// Collision response
velocity = VectorReflect(velocity, normal);

// Force calculations
float alignment = VectorDot(force, direction);
```

**Your Pong game uses:**
- `VectorAdd` - update position
- `VectorReflect` - ball bouncing
- `VectorNormalize` - surface normals
- `VectorDot` - check ball direction

---

## Key Takeaways

### Physics = Vector Math
Position, velocity, acceleration, forces are all vectors. Understanding vectors = understanding physics.

### Core Update Loop
```cpp
acceleration = force / mass;
velocity += acceleration × Δt;
position += velocity × Δt;
```
This creates all motion in games.

### Simulation is Approximate
- Game physics: discrete, "good enough"
- Real physics: continuous, perfect
- Consistency > accuracy

### Forces Create Behavior
- Gravity → falling
- Friction → stopping
- Springs → bouncing
- Drag → terminal velocity

---

## Practice Projects (if you want more to do)

1. **Bouncing balls** - gravity + wall collisions
2. **Pool/billiards** - object-to-object collisions
3. **Projectile motion** - parabolic trajectories
4. **Simple platformer** - character with jumping

---

## Final Thoughts

Physics simulation turns simple math into game mechanics:
- `F = ma` creates jumping, falling, pushing
- Vector reflection creates bouncing
- Dot products detect collisions
- Your vector library makes it all possible

**Every movement in every game uses these principles.**

Understanding physics + vectors = understanding game development.