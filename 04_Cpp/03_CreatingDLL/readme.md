# Creating a C++ DLL for Unity

## Overview

This guide walks through creating a Dynamic Link Library (DLL) in C++ that Unity can use through P/Invoke (Platform Invocation Services).

---

## Project Setup

### Visual Studio (Windows)

**Step 1: Create a new project**
1. Open Visual Studio
2. Create New Project
3. Select "Dynamic-Link Library (DLL)"
4. Name it: `VectorMathematics`
5. Click Create

**Step 2: Project configuration**
* Configuration: Release (for final builds) or Debug (for development)
* Platform: x64 (64-bit) or x86 (32-bit)
    * Modern Unity typically uses x64
    * Match your Unity editor architecture

---

## Code Structure

### Option 1: Header + Implementation (Recommended)

**VectorMath.h:**
```cpp
#ifndef VECTOR_MATH_H
#define VECTOR_MATH_H

// Platform-specific export macro
#ifdef _WIN32
    #define EXPORT __declspec(dllexport)
#else
    #define EXPORT __attribute__((visibility("default")))
#endif

// Vector structures
struct Vec2 {
    float x;
    float y;
};

struct Vec3 {
    float x;
    float y;
    float z;
};

// Function declarations
extern "C" {
    // Basic operations
    EXPORT Vec3 VectorAdd(Vec3 a, Vec3 b);
    EXPORT Vec3 VectorSubtract(Vec3 a, Vec3 b);
    EXPORT Vec3 VectorScale(Vec3 v, float scale);
    
    // Magnitude and distance
    EXPORT float VectorMagnitude(Vec3 v);
    EXPORT float VectorMagnitudeSqr(Vec3 v);
    EXPORT float VectorDistance(Vec3 a, Vec3 b);
    
    // Normalization
    EXPORT Vec3 VectorNormalize(Vec3 v);
    
    // Dot and Cross products
    EXPORT float VectorDot(Vec3 a, Vec3 b);
    EXPORT Vec3 VectorCross(Vec3 a, Vec3 b);
    
    // Reflection
    EXPORT Vec3 VectorReflect(Vec3 v, Vec3 normal);
    
    // Interpolation
    EXPORT Vec3 VectorLerp(Vec3 a, Vec3 b, float t);
}

#endif // VECTOR_MATH_H
```

**VectorMath.cpp:**
```cpp
#include "VectorMath.h"
#include <cmath>

// Basic operations
Vec3 VectorAdd(Vec3 a, Vec3 b) {
    // Todo
}

Vec3 VectorSubtract(Vec3 a, Vec3 b) {
    // Todo
}

Vec3 VectorScale(Vec3 v, float scale) {
    // Todo
}

// Magnitude and distance
float VectorMagnitude(Vec3 v) {
    // Todo
}

float VectorMagnitudeSqr(Vec3 v) {
    // Todo
}

float VectorDistance(Vec3 a, Vec3 b) {
    // Todo
}

// Normalization
Vec3 VectorNormalize(Vec3 v) {
    // Todo
}

// Dot product
float VectorDot(Vec3 a, Vec3 b) {
    // Todo
}

// Cross product
Vec3 VectorCross(Vec3 a, Vec3 b) {
    // Todo
}

// Reflection
Vec3 VectorReflect(Vec3 v, Vec3 normal) {
    // Todo
}

// Linear interpolation
Vec3 VectorLerp(Vec3 a, Vec3 b, float t) {
    // Todo
}
```

---

### Option 2: Single File (Simpler for small projects)

**VectorMath.cpp:**
```cpp
#ifdef _WIN32
    #define EXPORT __declspec(dllexport)
#else
    #define EXPORT __attribute__((visibility("default")))
#endif

#include <cmath>

struct Vec3 {
    float x;
    float y;
    float z;
};

extern "C" {
    EXPORT Vec3 VectorAdd(Vec3 a, Vec3 b) {
        // Todo
    }
    
    EXPORT float VectorMagnitude(Vec3 v) {
        // Todo
    }
    
    // ... rest of your functions
}
```

---

## Important Considerations

### 1. Calling Convention

Always use `extern "C"` to prevent C++ name mangling:

**Without extern "C":**
```cpp
// C++ compiler mangles the name to something like:
// ?VectorAdd@@YA?AUVec3@@U1@0@Z
```

**With extern "C":**
```cpp
// Function name stays as: VectorAdd
// Unity can find it by name
```

---

### 2. Data Types

**Safe types to pass between C++ and C#:**
* ✅ `float`, `double`
* ✅ `int`, `bool`
* ✅ Simple structs (no pointers, no methods)
* ✅ Arrays (as pointers with length parameter)

**Avoid:**
* ❌ C++ classes (use structs instead)
* ❌ `std::string` (use `char*` instead)
* ❌ `std::vector` (pass arrays manually)
* ❌ Pointers to complex objects

---

### 3. Memory Management

**Rule:** Whoever allocates memory should free it.

**Safe approach - Return by value:**
```cpp
// Good: Returns struct by value (Unity handles it)
EXPORT Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z }; // this one you get for free ;)
}
```

**Dangerous - Returning pointers:**
```cpp
// BAD: Who frees this memory?
EXPORT Vec3* CreateVector(float x, float y, float z) {
    Vec3* v = new Vec3{x, y, z};
    return v; // Memory leak potential!
}
```

**If you must use pointers:**
```cpp
// Allocate in C++
EXPORT Vec3* CreateVector(float x, float y, float z) {
    return new Vec3{x, y, z};
}

// Free in C++
EXPORT void FreeVector(Vec3* v) {
    delete v;
}
```

Then in Unity:
```csharp
IntPtr vectorPtr = CreateVector(1, 2, 3);
// ... use vector
FreeVector(vectorPtr); // Must remember to free!
```

**Best practice:** Avoid pointers entirely when possible. Use structs by value.

---

### 4. Error Handling

C++ exceptions don't cross the DLL boundary safely.

**Don't do this:**
```cpp
EXPORT Vec3 VectorNormalize(Vec3 v) {
    float mag = VectorMagnitude(v);
    if (mag == 0) {
        throw std::runtime_error("Cannot normalize zero vector");
    }
    // Unity will crash!
}
```

**Do this instead:**
```cpp
// Return a safe default value
EXPORT Vec3 VectorNormalize(Vec3 v) {
    float mag = VectorMagnitude(v);
    if (mag < 0.0001f) {
        return { 0.0f, 0.0f, 0.0f }; // Safe fallback
    }
    return { v.x / mag, v.y / mag, v.z / mag };
}
```

**Or use error codes:**
```cpp
EXPORT bool VectorNormalize(Vec3 v, Vec3* result) {
    float mag = VectorMagnitude(v);
    if (mag < 0.0001f) {
        return false; // Indicate error
    }
    result->x = v.x / mag;
    result->y = v.y / mag;
    result->z = v.z / mag;
    return true; // Success
}
```

---

## Building the DLL

### Visual Studio

**Debug build:**
1. Set configuration to Debug
2. Build → Build Solution (or Ctrl+Shift+B)
3. Find DLL in: `x64/Debug/VectorMathematics.dll`

**Release build:**
1. Set configuration to Release
2. Build → Build Solution
3. Find DLL in: `x64/Release/VectorMathematics.dll`

**Release vs Debug:**
* **Debug:** Larger, slower, includes debugging symbols
* **Release:** Smaller, faster, optimized
* Use Debug for development, Release for final builds

---

### Build Configuration Tips

**Project Properties (Right-click project → Properties):**

**C/C++ → General:**
* Warning Level: Level3 or Level4
* Treat Warnings as Errors: Yes (good practice)

**C/C++ → Optimization:**
* Debug: Disabled (/Od)
* Release: Maximum Optimization (/O2)

**C/C++ → Code Generation:**
* Runtime Library:
    * Multi-threaded DLL (/MD) for Release
    * Multi-threaded Debug DLL (/MDd) for Debug

**Linker → General:**
* Output File: `$(OutDir)VectorMathematics.dll`

---

## Testing Your DLL

### Create a simple test in C++

**test.cpp:**
```cpp
#include <iostream>
#include "VectorMath.h"

int main() {
    Vec3 a = { 1.0f, 2.0f, 3.0f };
    Vec3 b = { 4.0f, 5.0f, 6.0f };
    
    Vec3 result = VectorAdd(a, b);
    std::cout << "Result: (" << result.x << ", " 
              << result.y << ", " << result.z << ")\n";
    
    float mag = VectorMagnitude(a);
    std::cout << "Magnitude of a: " << mag << "\n";
    
    return 0;
}
```

---

## Common Issues and Solutions

### Issue 1: "Cannot find entry point"

**Problem:** Unity can't find your function

**Solution:**
* Ensure you're using `extern "C"`
* Check function name matches exactly in C#
* Use a DLL viewer tool (like Dependency Walker) to verify exports

---

### Issue 2: Crashes when calling functions

**Problem:** Unity crashes on DLL function call

**Causes:**
* Struct layout mismatch between C++ and C#
* Wrong calling convention
* Returning pointers incorrectly
* Throwing C++ exceptions

**Solution:**
* Use `[StructLayout(LayoutKind.Sequential)]` in C#
* Always use `extern "C"`
* Return values by value, not pointers
* Never throw exceptions across DLL boundary

---

### Issue 3: Different results on different platforms

**Problem:** Math gives different results on Windows vs macOS

**Causes:**
* Float precision differences
* Compiler optimizations

**Solution:**
* Use consistent types (`float`, not `double` mixed with `float`)
* Add epsilon comparisons for float equality
* Test on all target platforms

---

### Issue 4: DLL not loading in Unity

**Problem:** Unity doesn't recognize the DLL

**Solution:**
* Check DLL is in correct `Assets/Plugins/` folder
* Verify platform architecture matches (x64 vs x86)
* On macOS, ensure proper file extension (.bundle or .dylib)
* Check Unity's import settings for the DLL

---

## Platform-Specific Notes

### Windows
* Extension: `.dll`
* Export: `__declspec(dllexport)`
* Most straightforward platform

### macOS
* Extension: `.bundle` or `.dylib`
* Export: `__attribute__((visibility("default")))`
* May need code signing for distribution
* Use Xcode or command-line tools

### Linux
* Extension: `.so`
* Export: `__attribute__((visibility("default")))`
* Use `g++` with `-shared -fPIC` flags

---

## Best Practices Summary

1. ✅ **Always use `extern "C"`** for exports
2. ✅ **Keep it simple** - basic types and structs
3. ✅ **Return by value** when possible
4. ✅ **Check for edge cases** (division by zero, null vectors)
5. ✅ **No exceptions** across DLL boundary
6. ✅ **Match struct layouts** exactly between C++ and C#
7. ✅ **Use epsilon comparisons** for float equality
8. ✅ **Test on all target platforms**
9. ✅ **Use Release builds** for final distribution
10. ✅ **Document your API** clearly

---

## Next Steps

After creating your DLL:
1. Copy it to Unity's `Assets/Plugins/` folder
2. Create C# wrapper (covered in Unity Integration section)
3. Test all functions in Unity
4. Create unit tests
5. Profile performance if needed
6. Build for all target platforms

Your DLL is now ready to provide high-performance vector math to Unity!