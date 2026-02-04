# Header Files (.h) vs Implementation Files (.cpp)

## What are they?

In C++, code is typically split into two types of files:

* **Header files (.h)** - Contain declarations (what exists)
* **Implementation files (.cpp)** - Contain definitions (how it works)

This separation allows you to organize code and share interfaces without exposing implementation details.

---

### Header Files (.h)

**Purpose:** Declare what exists (the interface)

**Contains:**
* Function declarations (prototypes)
* Struct/class definitions
* Constants and macros
* Type definitions

**Example - VectorMath.h:**
```cpp
#ifndef VECTOR_MATH_H
#define VECTOR_MATH_H

// Struct definition
struct Vec3 {
    float x;
    float y;
    float z;
};

// Function declarations (just the signature)
Vec3 VectorAdd(Vec3 a, Vec3 b);
Vec3 VectorSubtract(Vec3 a, Vec3 b);
float VectorMagnitude(Vec3 v);
Vec3 VectorNormalize(Vec3 v);

#endif // VECTOR_MATH_H
```

**Key points:**
* Tells other files what functions and types are available
* Can be included in multiple .cpp files
* Uses **include guards** (`#ifndef`, `#define`, `#endif`) to prevent duplicate declarations
  * Read more about duplicate declarations using the `#pragma once` (google it)

---

### Implementation Files (.cpp)

**Purpose:** Define how things work (the implementation)

**Contains:**
* Function definitions (the actual code)
* Implementation details
* Private helper functions

**Example - VectorMath.cpp:**
```cpp
#include "VectorMath.h"
#include <cmath>

// Function definitions (the actual implementation)
Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}

Vec3 VectorSubtract(Vec3 a, Vec3 b) {
    return { a.x - b.x, a.y - b.y, a.z - b.z };
}

float VectorMagnitude(Vec3 v) {
    return sqrt(v.x * v.x + v.y * v.y + v.z * v.z);
}

Vec3 VectorNormalize(Vec3 v) {
    float mag = VectorMagnitude(v);
    if (mag < 0.0001f) return { 0, 0, 0 };
    return { v.x / mag, v.y / mag, v.z / mag };
}
```

**Key points:**
* Contains the actual logic and algorithms
* Includes the corresponding header file
* Compiled separately and linked together

---

### Why Split Them?

**1. Organization:**
* Header = "table of contents" (what's available)
* Implementation = "the book" (how it works)

**2. Compilation efficiency:**
* When you change implementation (.cpp), only that file needs recompiling
* If you change a header (.h), all files that include it must recompile

**3. Information hiding:**
* Users only need the header to use your library
* Implementation details stay hidden in the .cpp file

**4. Sharing code:**
* Give someone your .h file and compiled .dll/.lib
* They can use your functions without seeing your source code

---

### Include Guards

Header files use **include guards** to prevent multiple inclusions:
```cpp
#ifndef VECTOR_MATH_H  // If not defined
#define VECTOR_MATH_H  // Define it

// Your declarations here

#endif // End of guard
```

**Why?** If multiple files include the same header, without guards you'd get "redefinition" errors.

**Modern alternative:**
```cpp
#pragma once

// Your declarations here
```

`#pragma once` is simpler but not officially part of the C++ standard (though widely supported).

---

### For Your DLL Project

**Typical structure:**
```
VectorMathLibrary/
├── VectorMath.h         // Declarations (share this with Unity)
├── VectorMath.cpp       // Implementation (compile into DLL)
└── VectorMathDLL.cpp    // DLL entry point (if needed)
```

**In Unity (C#):**
You'll recreate the function signatures from your .h file using `DllImport`, effectively treating the .h file as documentation for what's available in the compiled DLL.

---

### Quick Reference

| Aspect | .h (Header) | .cpp (Implementation) |
|--------|-------------|----------------------|
| **Contains** | Declarations | Definitions |
| **Purpose** | Interface | Logic |
| **Included by** | Other files | Nothing (compiled directly) |
| **Changes trigger** | Widespread recompilation | Only that file recompiles |
| **Shareable** | Yes (with users) | No (stays private) |
| **Compiled** | No (just included) | Yes (into object files) |

---

### Best Practice

**Rule of thumb:**
* If someone needs to **know it exists** → put it in the .h file
* If someone needs to **know how it works** → put it in the .cpp file

This keeps your code modular, maintainable, and efficient to compile.

---

# Do You Actually Need a .h File for a DLL?

### Short answer: No, but it's helpful.

When creating a DLL for Unity, you don't technically need a header file at all. Here's why:

**Unity doesn't use your .h file directly.**

Unity is written in C#, not C++. When you import DLL functions into Unity, you manually declare them using `DllImport`:
```csharp
[DllImport("VectorMathematics")]
public static extern Vec3 VectorAdd(Vec3 a, Vec3 b);
```

This C# declaration completely replaces what a .h file would normally do in C++.

**You can write everything in a single .cpp file:**
```cpp
// VectorMath.cpp - No header file needed!

struct Vec3 {
    float x, y, z;
};

extern "C" __declspec(dllexport) Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}

extern "C" __declspec(dllexport) Vec3 VectorSubtract(Vec3 a, Vec3 b) {
    return { a.x - b.x, a.y - b.y, a.z - b.z };
}

// ... all your functions here
```

Compile this single file into a DLL, and Unity can use it directly.

---

## So Why Use a .h File At All?

Even though Unity doesn't need it, header files are still valuable:

**1. Documentation**
* Acts as a quick reference for what functions are available
* Shows function signatures in one place
* Easier to see the API at a glance

**2. Organization**
* Keeps declarations separate from implementation
* Makes code easier to navigate and maintain
* Follows standard C++ conventions

**3. C++ Code Reuse**
* If you want to use your DLL in other C++ projects (not just Unity)
* Other C++ programs can include your .h file
* Enables proper linking in native C++ applications

**4. Team Collaboration**
* Other developers can quickly understand your API
* Header serves as a contract/specification
* Implementation can change without breaking the interface

**5. IDE Support**
* Better autocomplete and IntelliSense
* Easier debugging and navigation
* Code analysis tools work better

---

## Practical Recommendation

**For a simple Unity-only DLL:**
* Single .cpp file is fine
* Keep it simple if the project is small

**For a professional/larger project:**
* Use .h and .cpp separation
* Better organization and maintainability
* Easier to expand later

**For a library used by multiple projects:**
* Definitely use .h files
* Enables proper C++ integration
* Standard practice for shared libraries

---

### The Bottom Line

The .h file is for **C++ convenience**, not Unity requirement. Unity only cares about:
1. The compiled .dll file
2. The C# declarations you write with `DllImport`

Choose based on project size and whether you'll use the code outside Unity.

---

## Side Note: .hpp Files

### What are .hpp files?

`.hpp` stands for "Header Plus Plus" - it's an alternative extension for C++ header files.

### .h vs .hpp

| Extension | Usage |
|-----------|-------|
| `.h` | Traditional, used for C and C++ headers |
| `.hpp` | Explicitly indicates C++ code (not C-compatible) |

**Technically, they're identical** - both are header files. The difference is purely conventional.

---

### Why .hpp Exists

**Historical reason:**

In the early days, C and C++ shared the `.h` extension. This caused confusion:
* Is this header C-compatible?
* Does it contain C++ features (classes, templates, namespaces)?

`.hpp` was introduced to clearly signal: **"This is C++ only"**

---

### When to Use .hpp

**Use .hpp when:**
* Your header contains C++ specific features:
    * Classes
    * Templates
    * Namespaces
    * Operator overloading
    * C++ standard library includes

**Example:**
```cpp
// VectorMath.hpp - Contains C++ classes
#pragma once
#include <iostream>

namespace Math {
    class Vector3 {
    public:
        float x, y, z;
        
        Vector3 operator+(const Vector3& other) const;
        float magnitude() const;
    };
}
```

**Use .h when:**
* Writing C-compatible code
* Using `extern "C"` for DLL exports
* Maintaining compatibility with C projects

**Example:**
```cpp
// VectorMath.h - C-compatible
#ifndef VECTOR_MATH_H
#define VECTOR_MATH_H

#ifdef __cplusplus
extern "C" {
#endif

struct Vec3 {
    float x, y, z;
};

Vec3 VectorAdd(Vec3 a, Vec3 b);

#ifdef __cplusplus
}
#endif

#endif
```

---

### For Your Unity DLL Project

**Recommendation: Use .h**

Why?
* Your DLL functions use `extern "C"` (C linkage)
* Unity expects C-style function calls
* .h clearly indicates C compatibility
* Standard practice for DLL exports

**However:** Both work fine. It's a convention, not a technical requirement.

---

### Industry Practice

**Common conventions:**
* **Game engines:** Often use `.h` for everything (Unreal, Unity native plugins)
* **Modern C++ libraries:** Increasingly use `.hpp` for pure C++ code
* **Mixed projects:** `.h` for C-compatible, `.hpp` for C++-only

**Most important:** Be consistent within your project.

---

### Quick Decision Guide
```
Is your code C-compatible (extern "C", plain structs)?
├─ Yes → Use .h
└─ No (using classes, templates, etc.)
   └─ Use .hpp to signal "C++ only"
```

**For your vector math DLL:** `.h` is the right choice since you're exporting C-compatible functions to Unity.

---

### The Reality

In practice, many projects use `.h` for everything and rely on `extern "C"` blocks to control linkage. The `.hpp` distinction is helpful but not mandatory.

**Choose what makes sense for your team and stick with it.**