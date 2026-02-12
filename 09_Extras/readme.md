# Generating Simple Meshes from C++

## Introduction

Before using a 3D modeling tool, every mesh is just mathematics - a collection of vertices, triangles, and normals. This guide shows you how to generate simple shapes entirely in code and pass them into Unity through your DLL.

This is exactly how many game engines generate:
- Primitive shapes (cube, sphere, capsule)
- Procedural terrain
- Dynamic geometry (expanding explosions, growing vines)
- UI elements

---

## What Defines a Mesh?

Any 3D mesh is made of three core components:

**1. Vertices**
Points in 3D space that form the corners of triangles.
```cpp
Vec3 vertex = { 1.0f, 0.0f, 0.0f };
```

**2. Indices (Triangles)**
Integers that reference vertices by index, defining which three vertices form a triangle.
```cpp
// Triangle using vertex 0, 1, and 2
int indices[] = { 0, 1, 2 };
```

**Why indices instead of repeating vertices?**
- Multiple triangles share vertices
- Storing indices is far more memory efficient
- A cube has 8 vertices but 12 triangles

**3. Normals**
A direction vector per vertex indicating which way the surface faces. Required for lighting.
```cpp
Vec3 normal = { 0.0f, 1.0f, 0.0f };  // Facing up
```

**Optionally:**
- **UVs** - texture coordinates per vertex
- **Colors** - per-vertex color

---

## Understanding Triangle Winding Order

**Critical concept:** The order vertices are listed determines which side of the triangle is the front face.

**Counter-clockwise = front face (Unity default):**
```
    1
   / \
  /   \
 0-----2    Vertices go: 0 → 1 → 2 (counter-clockwise when viewed from front)
```

**Clockwise = back face** (will be invisible with back-face culling)

**Always use consistent winding order** or your mesh will look inside-out!

---

## Part 1: Generating a Box

### Box Anatomy

A box has:
- 8 corner vertices
- 6 faces
- 2 triangles per face = 12 triangles
- 36 indices (3 per triangle × 12 triangles)

**However:** For correct lighting, each face needs its own vertices with unique normals.
This means a cube actually uses **24 vertices** (4 per face × 6 faces), not 8.

**Why?** If two faces share a vertex, the normal gets averaged between faces, creating a smooth look. Boxes have sharp edges - we want hard normals.

---

### Box Generation Code

**MeshGenerator.h:**
```cpp
#ifndef MESH_GENERATOR_H
#define MESH_GENERATOR_H

#ifdef _WIN32
    #define EXPORT __declspec(dllexport)
#else
    #define EXPORT __attribute__((visibility("default")))
#endif

struct Vec3 {
    float x, y, z;
};

struct Vec2 {
    float x, y;
};

struct MeshData {
    Vec3* vertices;
    Vec3* normals;
    Vec2* uvs;
    int*  indices;

    int vertexCount;
    int indexCount;
};

extern "C" {
    // Generates a box centered at origin
    // width  = size on X axis
    // height = size on Y axis
    // depth  = size on Z axis
    EXPORT MeshData* GenerateBox(float width, float height, float depth);

    // Generates a circle (flat disc) on the XZ plane
    // radius   = radius of the circle
    // segments = number of triangles (higher = smoother)
    EXPORT MeshData* GenerateCircle(float radius, int segments);

    // Free mesh data after Unity has read it
    EXPORT void FreeMeshData(MeshData* data);
}

#endif
```

---

**MeshGenerator.cpp:**
```cpp
#include "pch.h"
#include "MeshGenerator.h"
#include <cmath>

// Helper: allocate mesh data
MeshData* AllocateMeshData(int vertexCount, int indexCount) {
    MeshData* mesh = new MeshData();

    mesh->vertices    = new Vec3[vertexCount];
    mesh->normals     = new Vec3[vertexCount];
    mesh->uvs         = new Vec2[vertexCount];
    mesh->indices     = new int[indexCount];
    mesh->vertexCount = vertexCount;
    mesh->indexCount  = indexCount;

    return mesh;
}

// ─────────────────────────────────────────────
//  BOX GENERATION
// ─────────────────────────────────────────────

MeshData* GenerateBox(float width, float height, float depth) {
    // 4 vertices per face × 6 faces = 24 vertices
    // 2 triangles per face × 6 faces × 3 indices = 36 indices
    MeshData* mesh = AllocateMeshData(24, 36);

    float w = width  * 0.5f;    // Half extents
    float h = height * 0.5f;
    float d = depth  * 0.5f;

    // Helper: fill a face
    // v0-v3 = corners, n = normal, uvOffset for UV layout
    int vertIndex = 0;
    int idxIndex  = 0;

    // Lambda-style helper (nested function via struct)
    auto AddFace = [&](
        Vec3 v0, Vec3 v1, Vec3 v2, Vec3 v3,
        Vec3 normal)
    {
        // 4 vertices
        mesh->vertices[vertIndex + 0] = v0;
        mesh->vertices[vertIndex + 1] = v1;
        mesh->vertices[vertIndex + 2] = v2;
        mesh->vertices[vertIndex + 3] = v3;

        // Same normal for all 4 vertices (flat shading)
        mesh->normals[vertIndex + 0] = normal;
        mesh->normals[vertIndex + 1] = normal;
        mesh->normals[vertIndex + 2] = normal;
        mesh->normals[vertIndex + 3] = normal;

        // UV coordinates (simple 0-1 mapping per face)
        mesh->uvs[vertIndex + 0] = { 0.0f, 0.0f };
        mesh->uvs[vertIndex + 1] = { 1.0f, 0.0f };
        mesh->uvs[vertIndex + 2] = { 1.0f, 1.0f };
        mesh->uvs[vertIndex + 3] = { 0.0f, 1.0f };

        // Two triangles (counter-clockwise winding)
        // Triangle 1: 0, 1, 2
        // Triangle 2: 0, 2, 3
        mesh->indices[idxIndex + 0] = vertIndex + 0;
        mesh->indices[idxIndex + 1] = vertIndex + 1;
        mesh->indices[idxIndex + 2] = vertIndex + 2;
        mesh->indices[idxIndex + 3] = vertIndex + 0;
        mesh->indices[idxIndex + 4] = vertIndex + 2;
        mesh->indices[idxIndex + 5] = vertIndex + 3;

        vertIndex += 4;
        idxIndex  += 6;
    };

    // ── 6 Faces ──────────────────────────────────────
    //   Each face: 4 corners, 1 normal

    // Front face  (+Z)
    AddFace(
        {-w, -h,  d}, { w, -h,  d}, { w,  h,  d}, {-w,  h,  d},
        { 0,  0,  1}
    );

    // Back face   (-Z)
    AddFace(
        { w, -h, -d}, {-w, -h, -d}, {-w,  h, -d}, { w,  h, -d},
        { 0,  0, -1}
    );

    // Left face   (-X)
    AddFace(
        {-w, -h, -d}, {-w, -h,  d}, {-w,  h,  d}, {-w,  h, -d},
        {-1,  0,  0}
    );

    // Right face  (+X)
    AddFace(
        { w, -h,  d}, { w, -h, -d}, { w,  h, -d}, { w,  h,  d},
        { 1,  0,  0}
    );

    // Top face    (+Y)
    AddFace(
        {-w,  h,  d}, { w,  h,  d}, { w,  h, -d}, {-w,  h, -d},
        { 0,  1,  0}
    );

    // Bottom face (-Y)
    AddFace(
        {-w, -h, -d}, { w, -h, -d}, { w, -h,  d}, {-w, -h,  d},
        { 0, -1,  0}
    );

    return mesh;
}

// ─────────────────────────────────────────────
//  CIRCLE GENERATION
// ─────────────────────────────────────────────

MeshData* GenerateCircle(float radius, int segments) {
    // 1 center vertex + 1 vertex per segment + 1 to close the loop
    // Indices: 3 per triangle × segments triangles
    int vertexCount = segments + 2;
    int indexCount  = segments * 3;

    MeshData* mesh = AllocateMeshData(vertexCount, indexCount);

    // Center vertex
    mesh->vertices[0] = { 0.0f, 0.0f, 0.0f };
    mesh->normals[0]  = { 0.0f, 1.0f, 0.0f };  // Pointing up
    mesh->uvs[0]      = { 0.5f, 0.5f };          // UV center

    // Generate vertices around the circle
    // We use angle steps of (2π / segments)
    const float PI = 3.14159265358979f;
    float angleStep = (2.0f * PI) / segments;

    for (int i = 0; i <= segments; i++) {
        float angle = i * angleStep;

        // Position on circle using sin and cos
        float x = cos(angle) * radius;
        float z = sin(angle) * radius;

        int vertIdx = i + 1;  // Offset by 1 for center vertex

        mesh->vertices[vertIdx] = { x,    0.0f, z    };
        mesh->normals[vertIdx]  = { 0.0f, 1.0f, 0.0f };

        // UV: map circle to 0-1 range
        mesh->uvs[vertIdx] = {
            (cos(angle) * 0.5f) + 0.5f,  // 0.0 to 1.0
            (sin(angle) * 0.5f) + 0.5f
        };
    }

    // Generate triangles (fan from center)
    // Each triangle: center(0), current edge vertex, next edge vertex
    for (int i = 0; i < segments; i++) {
        mesh->indices[i * 3 + 0] = 0;        // Center
        mesh->indices[i * 3 + 1] = i + 1;    // Current
        mesh->indices[i * 3 + 2] = i + 2;    // Next
    }

    return mesh;
}

// ─────────────────────────────────────────────
//  MEMORY CLEANUP
// ─────────────────────────────────────────────

void FreeMeshData(MeshData* data) {
    if (data) {
        delete[] data->vertices;
        delete[] data->normals;
        delete[] data->uvs;
        delete[] data->indices;
        delete data;
    }
}
```

---

## How Circle Generation Works

This is the most mathematically interesting part:

**The unit circle:**
```
     (0, 1)        90°
        |
(-1,0)--+--( 1,0)  0°/360°
        |
     (0,-1)        270°
```

For each step around the circle:
```cpp
float angle = i * (2π / segments);  // Evenly spaced angles
float x = cos(angle) * radius;      // X position
float z = sin(angle) * radius;      // Z position
```

**Visual example with 6 segments:**
```
       v2
      / \
    v3   v1
    |  c  |       c = center vertex
    v4   v6       v1-v6 = edge vertices
      \ /
       v5

Triangles:
c, v1, v2
c, v2, v3
c, v3, v4
c, v4, v5
c, v5, v6
c, v6, v1  ← closes the loop
```

**More segments = smoother circle:**
- 6 segments = hexagon
- 12 segments = dodecagon
- 32 segments = looks like a circle
- 64+ segments = very smooth

---

## Part 2: Unity Integration

### C# Wrapper for Mesh Generation

**MeshGenerator.cs:**
```csharp
using System;
using System.Runtime.InteropServices;
using UnityEngine;

// Must match C++ MeshData struct exactly
[StructLayout(LayoutKind.Sequential)]
public struct MeshDataNative
{
    public IntPtr vertices;     // Vec3*
    public IntPtr normals;      // Vec3*
    public IntPtr uvs;          // Vec2*
    public IntPtr indices;      // int*
    public int vertexCount;
    public int indexCount;
}

// Must match C++ Vec3 struct
[StructLayout(LayoutKind.Sequential)]
public struct Vec3Native
{
    public float x, y, z;
}

// Must match C++ Vec2 struct
[StructLayout(LayoutKind.Sequential)]
public struct Vec2Native
{
    public float x, y;
}

public static class MeshGeneratorDLL
{
    private const string DllName = "VectorMathematics";

    [DllImport(DllName)]
    private static extern IntPtr GenerateBox(float width, float height, float depth);

    [DllImport(DllName)]
    private static extern IntPtr GenerateCircle(float radius, int segments);

    [DllImport(DllName)]
    private static extern void FreeMeshData(IntPtr meshData);

    // ── Generate a Unity Mesh from box parameters ──

    public static Mesh CreateBoxMesh(float width, float height, float depth)
    {
        // Call C++ DLL to generate mesh data
        IntPtr ptr = GenerateBox(width, height, depth);

        // Read the MeshData struct from unmanaged memory
        MeshDataNative data = Marshal.PtrToStructure<MeshDataNative>(ptr);

        // Convert unmanaged arrays to managed C# arrays
        Mesh mesh = ConvertToUnityMesh(data);

        // Free C++ memory
        FreeMeshData(ptr);

        return mesh;
    }

    // ── Generate a Unity Mesh from circle parameters ──

    public static Mesh CreateCircleMesh(float radius, int segments)
    {
        IntPtr ptr = GenerateCircle(radius, segments);
        MeshDataNative data = Marshal.PtrToStructure<MeshDataNative>(ptr);
        Mesh mesh = ConvertToUnityMesh(data);
        FreeMeshData(ptr);
        return mesh;
    }

    // ── Convert unmanaged MeshData to Unity Mesh ──

    private static Mesh ConvertToUnityMesh(MeshDataNative data)
    {
        // Read vertices from unmanaged memory
        Vector3[] vertices = new Vector3[data.vertexCount];
        for (int i = 0; i < data.vertexCount; i++)
        {
            Vec3Native v = Marshal.PtrToStructure<Vec3Native>(
                data.vertices + i * Marshal.SizeOf<Vec3Native>()
            );
            vertices[i] = new Vector3(v.x, v.y, v.z);
        }

        // Read normals
        Vector3[] normals = new Vector3[data.vertexCount];
        for (int i = 0; i < data.vertexCount; i++)
        {
            Vec3Native n = Marshal.PtrToStructure<Vec3Native>(
                data.normals + i * Marshal.SizeOf<Vec3Native>()
            );
            normals[i] = new Vector3(n.x, n.y, n.z);
        }

        // Read UVs
        Vector2[] uvs = new Vector2[data.vertexCount];
        for (int i = 0; i < data.vertexCount; i++)
        {
            Vec2Native uv = Marshal.PtrToStructure<Vec2Native>(
                data.uvs + i * Marshal.SizeOf<Vec2Native>()
            );
            uvs[i] = new Vector2(uv.x, uv.y);
        }

        // Read indices (triangles)
        int[] indices = new int[data.indexCount];
        Marshal.Copy(data.indices, indices, 0, data.indexCount);

        // Build Unity Mesh
        Mesh mesh = new Mesh();
        mesh.vertices  = vertices;
        mesh.normals   = normals;
        mesh.uv        = uvs;
        mesh.triangles = indices;

        // Recalculate bounds so Unity knows the size
        mesh.RecalculateBounds();

        return mesh;
    }
}
```

---

### Using the Mesh Generator in Unity

**ProceduralShapeSpawner.cs:**
```csharp
using UnityEngine;

[RequireComponent(typeof(MeshFilter))]
[RequireComponent(typeof(MeshRenderer))]
public class ProceduralShapeSpawner : MonoBehaviour
{
    public enum ShapeType { Box, Circle }

    [Header("Shape Settings")]
    public ShapeType shape = ShapeType.Box;

    [Header("Box Settings")]
    public float boxWidth  = 2.0f;
    public float boxHeight = 1.0f;
    public float boxDepth  = 2.0f;

    [Header("Circle Settings")]
    public float circleRadius   = 1.0f;
    public int   circleSegments = 32;

    [Header("Material")]
    public Material material;

    void Start()
    {
        GenerateShape();
    }

    public void GenerateShape()
    {
        Mesh mesh = null;

        if (shape == ShapeType.Box)
        {
            mesh = MeshGeneratorDLL.CreateBoxMesh(boxWidth, boxHeight, boxDepth);
            Debug.Log($"Generated box: {boxWidth} x {boxHeight} x {boxDepth}");
        }
        else if (shape == ShapeType.Circle)
        {
            mesh = MeshGeneratorDLL.CreateCircleMesh(circleRadius, circleSegments);
            Debug.Log($"Generated circle: radius={circleRadius}, segments={circleSegments}");
        }

        // Assign mesh to this GameObject
        GetComponent<MeshFilter>().mesh = mesh;

        // Assign material (or use default)
        MeshRenderer renderer = GetComponent<MeshRenderer>();
        if (material != null)
            renderer.material = material;
        else
            renderer.material = new Material(Shader.Find("Standard"));
    }

    // Allow regenerating in editor (for testing)
    #if UNITY_EDITOR
    void OnValidate()
    {
        if (Application.isPlaying)
            GenerateShape();
    }
    #endif
}
```

---

### How Marshal.PtrToStructure Works

This is the key to reading C++ memory from C#:

```csharp
// C++ returns a pointer to MeshData
IntPtr ptr = GenerateBox(1, 1, 1);

// Marshal reads the struct from that memory address
MeshDataNative data = Marshal.PtrToStructure<MeshDataNative>(ptr);
// data now contains the pointers and counts from C++

// For arrays, we read element by element:
Vec3Native v = Marshal.PtrToStructure<Vec3Native>(
    data.vertices + i * Marshal.SizeOf<Vec3Native>()
);
// This calculates the address of element [i] and reads it
```

**Why IntPtr?**
- C++ pointers don't exist in C#
- `IntPtr` is a "raw address" type in C#
- We tell Unity exactly how to interpret that memory via struct layouts

---

## Memory Management Across the DLL Boundary

**This is critical to understand:**

```cpp
// C++ allocates memory
MeshData* GenerateBox(float w, float h, float d) {
    MeshData* mesh = new MeshData();  // C++ heap allocation
    mesh->vertices = new Vec3[24];    // More C++ heap
    // ...
    return mesh;  // Unity receives a pointer to this
}
```

```csharp
// C# READS the data but does NOT own the memory
Mesh mesh = ConvertToUnityMesh(data);  // Data copied to C#

// C# tells C++ to clean up
FreeMeshData(ptr);  // C++ deletes its own memory
```

**Rule:** C++ allocates, C++ frees. Never try to free C++ memory from C#!

---

## Testing in Unity

**Step 1:** Build DLL and copy to `Assets/Plugins/`

**Step 2:** Create GameObject
```
Hierarchy → Right-click → Create Empty
Name: "ProceduralBox"
```

**Step 3:** Add Components
```
Inspector → Add Component → ProceduralShapeSpawner
```

**Step 4:** Configure and Play

**Expected result:**
- **Box:** A white cube centered at origin with correct normals
- **Circle:** A flat disc on the XZ plane

**Test lighting is correct:**
- Add a Directional Light to the scene
- Rotate it 45° - box faces should light differently
- If lighting looks wrong, normals are incorrect

---

## Common Issues and Solutions

### Issue 1: Mesh appears inside-out
**Cause:** Wrong winding order  
**Fix:** Reverse triangle indices order
```cpp
// Change from
mesh->indices[idx + 0] = v + 0;
mesh->indices[idx + 1] = v + 1;
mesh->indices[idx + 2] = v + 2;
// To
mesh->indices[idx + 0] = v + 0;
mesh->indices[idx + 1] = v + 2;
mesh->indices[idx + 2] = v + 1;
```

### Issue 2: Flat shading instead of smooth
**Cause:** Each face has 4 vertices with the same normal  
**Fix:** Share vertices between faces and average normals (or use `mesh.RecalculateNormals()` in Unity)

### Issue 3: Memory leak
**Cause:** Forgetting to call `FreeMeshData`  
**Fix:** Always call after `ConvertToUnityMesh`

### Issue 4: Circle has visible seam
**Cause:** Last vertex doesn't connect back to first  
**Fix:** Generate `segments + 1` edge vertices (first and last are at the same position, closing the loop)

### Issue 5: Shape too dark or black
**Cause:** Normals pointing wrong direction  
**Fix:** Check normal direction or call `mesh.RecalculateNormals()` in Unity for testing

---

## Key Takeaways

### A Mesh is Just Math
- Vertices = points in space (your Vec3!)
- Indices = which points form triangles
- Normals = which way each surface faces (vectors!)

### The Process
```
Define vertices → Define triangles (indices) → Define normals
       ↓
Pass to Unity via DLL
       ↓
Unity assigns to MeshFilter → Renderer draws it
```

### Everything Connects
- **Vertices** use your `Vec3` struct
- **Normals** use your `VectorNormalize` function
- **Circle positions** use `sin` and `cos` (trigonometry)
- **Winding order** uses your right-hand rule knowledge

### Memory Rules
- C++ allocates → C++ frees
- C# copies data → C# has its own copy
- Always free after reading

---

## Extension Ideas

Once you have box and circle working, try:

1. **Sphere** - Extend circle logic to 3D (latitude/longitude vertices)
2. **Cylinder** - Combine circle top/bottom with rectangular sides
3. **Plane** - Grid of quads with configurable resolution
4. **Capsule** - Cylinder with hemisphere caps
5. **Procedural terrain** - Grid with height values per vertex

Each of these uses the exact same principles - just more vertices and triangles!