# Graphics and Rendering Fundamentals

## Introduction

Graphics rendering converts 3D mathematical data into 2D images on screen through a series of transformations using vectors and matrices.

---

## The Rendering Pipeline: 3D to 2D

Every 3D object passes through these coordinate spaces:

1. **Model Space** - Local coordinates where the artist created the object
2. **World Space** - Object positioned in the game world
3. **View Space** - World seen from camera's perspective
4. **Clip Space** - 3D projected to 2D
5. **Screen Space** - Final pixel coordinates

**Each transformation is a matrix multiplication.**

---

## Matrices: The Foundation

### What is a Matrix?

A matrix is a rectangular array of numbers that represents a transformation.

**4x4 Matrix (most common in 3D games):**
```
| m00  m01  m02  m03 |
| m10  m11  m12  m13 |
| m20  m21  m22  m23 |
| m30  m31  m32  m33 |
```

### Why Use Matrices?

**Without matrices:**
```cpp
// Apply each transformation separately
x = x + tx;              // Translate
x = x * cos(a) - z * sin(a);  // Rotate
x = x * sx;              // Scale
// Repeat for EVERY vertex, EVERY frame!
```

**With matrices:**
```cpp
Matrix4x4 transform = translation * rotation * scale;
Vector3 result = transform * vertex;  // ONE operation!
```

**Key advantages:**
- Combine multiple transformations into one matrix
- Single multiplication instead of multiple operations
- GPU hardware accelerated
- Consistent for all transformation types

---

## Transformation Matrices

### Translation (Movement)
```
| 1  0  0  tx |
| 0  1  0  ty |
| 0  0  1  tz |
| 0  0  0  1  |
```
Moves object by (tx, ty, tz)

### Scale (Size)
```
| sx  0   0   0 |
| 0   sy  0   0 |
| 0   0   sz  0 |
| 0   0   0   1 |
```
Scales object by (sx, sy, sz)

### Rotation (Around Y-axis example)
```
| cos(θ)    0    sin(θ)   0 |
| 0         1    0        0 |
| -sin(θ)   0    cos(θ)   0 |
| 0         0    0        1 |
```
Rotates by angle θ (tetha)

### Combining Transformations

```cpp
Matrix4x4 model = translation * rotation * scale;
```

**Order matters!** Read right to left:
1. Scale first (around origin)
2. Then rotate (around origin)
3. Then translate (move to final position)

**Why this order?** Scale and rotation happen around the origin (0,0,0). If you translate first, they'll happen around the wrong point!

---

## The 4th Dimension (W Component)

We use 4x4 matrices (not 3x3) to enable translation through matrix multiplication.

**Homogeneous coordinates:**
- 3D position (x, y, z) becomes (x, y, z, 1) - affected by translation
- 3D direction (x, y, z) becomes (x, y, z, 0) - NOT affected by translation

**Rule:**
- `w = 1` for positions (points that should move)
- `w = 0` for directions (vectors that shouldn't move)

This allows us to distinguish between a position in space and a direction of movement.

---

## Coordinate Space Transformations

### Model to World
```cpp
Vector3 worldPosition = modelMatrix * localPosition;
```
Places the object in the game world with position, rotation, and scale.

### World to View
```cpp
Vector3 viewPosition = viewMatrix * worldPosition;
```
Transforms world to camera's perspective (camera becomes origin).

### View to Clip
```cpp
Vector4 clipPosition = projectionMatrix * viewPosition;
```
Projects 3D space onto 2D screen with perspective.

### Complete Chain
```cpp
// Often combined as MVP (Model-View-Projection)
Matrix4x4 mvp = projection * view * model;
Vector4 screenPosition = mvp * localPosition;
```

---

## Lighting Basics

### Surface Normals

A **normal** is a vector perpendicular to a surface, indicating which way it faces.

**Why normals matter:**
- Determine how light interacts with surfaces
- Essential for realistic lighting
- Must be normalized (length = 1)

### Phong Lighting Model

Three lighting components:

**1. Ambient Light** - Base light level everywhere
```
ambient = ambientColor * ambientStrength
```

**2. Diffuse Light** - Main directional lighting (uses dot product!)
```
diffuse = lightColor * max(dot(normal, lightDirection), 0.0)
```
- dot = 1 when surface faces light directly → brightest
- dot = 0 when perpendicular → no direct light
- dot < 0 when facing away → no light (clamped to 0)

**3. Specular Light** - Shiny highlights (uses reflection!)
```
reflectDir = reflect(-lightDirection, normal)
specular = lightColor * pow(max(dot(viewDir, reflectDir), 0.0), shininess)
```

**Complete lighting:**
```
finalColor = (ambient + diffuse + specular) * objectColor
```

### Connection to Your Vector Math

```cpp
Vector3 normal = VectorNormalize(surfaceNormal);
Vector3 lightDir = VectorNormalize(lightPos - fragPos);
float brightness = VectorDot(normal, lightDir);

Vector3 viewDir = VectorNormalize(cameraPos - fragPos);
Vector3 reflectDir = VectorReflect(-lightDir, normal);
float specular = pow(VectorDot(viewDir, reflectDir), 32);
```

---

## Textures

**What:** Images mapped onto 3D surfaces to add detail without geometry.

**UV Coordinates:** 2D coordinates (0 to 1) that map texture pixels to 3D vertices.
- U = horizontal (0 = left, 1 = right)
- V = vertical (0 = bottom, 1 = top)

Each vertex has:
- 3D position (x, y, z)
- 3D normal (nx, ny, nz)
- 2D texture coordinate (u, v)

---

## The GPU Pipeline (Simplified)

1. **Input Assembler** - Reads vertex data, creates triangles
2. **Vertex Shader** - Transforms vertices (you write this!)
   ```glsl
   gl_Position = mvpMatrix * vec4(vertexPosition, 1.0);
   ```
3. **Rasterization** - Converts triangles to pixels
4. **Fragment Shader** - Calculates pixel colors (you write this!)
   ```glsl
   color = texture * lighting;
   ```
5. **Output** - Depth test, blend, write to screen

---

## Optimization Techniques

### Back-Face Culling
Don't render triangles facing away from camera → ~50% fewer triangles!

### Depth Testing (Z-Buffer)
Store distance to camera for each pixel. Only draw if closer than what's already there.

### Frustum Culling
Don't process objects outside camera view.

---

## Key Takeaways

**Matrices are essential because:**
- Combine multiple transformations efficiently
- Hardware accelerated on GPU
- Standard operation for all transformation types

**The transformation chain:**
```
Model → World → View → Clip → Screen
```

**Vector operations drive everything:**
- Matrices transform positions
- Dot product for lighting angles
- Normalization for consistent directions
- Reflection for specular highlights
- Cross product for surface normals

**Your vector library enables:**
- Lighting calculations
- Physics simulation
- Camera movement
- All graphics rendering

---

## Projection Types

**Perspective Projection:**
- Distant objects appear smaller (realistic)
- Used for 3D games
- Creates depth perception

**Orthographic Projection:**
- No perspective distortion
- Objects same size regardless of distance
- Used for UI, 2D games, minimaps

---

## One Frame of Rendering

**CPU (60 times per second):**
```cpp
// Update game
player.position += velocity * deltaTime;

// Create matrices
Matrix4x4 model = Translation(pos) * Rotation(rot) * Scale(scale);
Matrix4x4 view = CameraViewMatrix(camera);
Matrix4x4 projection = PerspectiveMatrix(fov, aspect, near, far);
Matrix4x4 mvp = projection * view * model;

// Send to GPU
SendToShader(mvp);
```

**GPU:**
```glsl
// Transform vertices
gl_Position = mvp * vertexPosition;

// Calculate lighting
color = texture * (ambient + diffuse + specular);
```

---

## Final Thoughts

Graphics rendering is applied mathematics:
- **Matrix transformations** move and rotate objects
- **Vector operations** calculate lighting
- **Dot products** determine angles
- **Normalization** ensures consistency

Understanding the math helps you:
- Debug visual issues
- Optimize performance
- Create custom effects
- Understand game engine internals

**The vector library you're building is the foundation of all 3D graphics.**

---

## Summary Diagram

```
3D Model (vertices)
      ↓
Model Matrix (position, rotation, scale)
      ↓
World Space
      ↓
View Matrix (camera transform)
      ↓
View Space
      ↓
Projection Matrix (3D → 2D)
      ↓
Clip Space
      ↓
Perspective Divide (w component)
      ↓
Screen Pixels
```

Every arrow is a matrix multiplication. Every calculation uses your vector math.