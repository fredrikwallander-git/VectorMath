# Unity Integration Notes

#### When creating a DLL (dynamic-link library) for Unity:

1. Calling convention: use `extern "C"` and `__declspec(dllexport)` in your DLL
2. Data Layout: Ensure struct memory packing matches the layout in Unity (C#)
   * Make use of the attribute `[StructLayout(LayoutKind.Sequential)]`
3. Float Precision: Unity uses `float`, not `double` for vectors; if you intend to use other types (like double), ensure proper conversion handling at the DLL boundary
4. Return Values: Return structs by value (Unity marshals them correctly)

## Struct Definition

#### Your C++ and C# structs must match exactly:

```cpp
// C++ side (in your DLL)
struct Vec3 {
    float x;
    float y;
    float z;
};
```

```csharp
// C# side (in Unity)
[StructLayout(LayoutKind.Sequential)]
public struct Vec3 
{
    public float x;
    public float y;
    public float z;
}
```

* Attribute `StructLayout`: Lets you control the physical layout of the data fields of a class or structure in memory
* Enum `LayoutKind`:
  * `Squential`: The members of the object are laid out sequentially, in the order in which they appear when exported to unmanaged memory
  * `Explicit`: The precise position of each member of an object in unmanaged memory is explicitly controlled
  * `Auto`: The runtime automatically chooses an appropriate layout for the members of an object in unmanaged memory
    * Objects defined with this enumeration member cannot be exposed outside managed code
    * Attempting to do so generates an exception

---

## Example export:
```cpp
extern "C" __declspec(dllexport) Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}
```

* `__declspec(dllimport / dllexport)` is a Microsoft-specific keyword used in C++ to define storage-class information.
  * It is used commonly to import or export DLL functions and acts as a modifier before declarations.

* `extern "C"` is a linkage-specification for the compiler's translation unit and specifies the linkage conventions of other languages.

---

## Example import:
```csharp
public static class VectorMath 
{
    const string DllName = "VectorMathematics";
    
    [DllImport(DllName)]
    public static extern Vec3 VectorAdd(Vec3 a, Vec3 b);
}
```

---

* `DllImport`: Indicates that the attributed method is exposed by an unmanaged dynamic-link library (DLL) as a static entry point
* `extern`: A modifier to declare a method that's implemented externally, most commonly used with the `DllImport` attribute when you use Interop services
   * In this case, you must also declare the method as `static`
   * Declared method also needs to match the signature of the external method; return value (if any), name and parameters (if any)

---

### Platform Considerations

* Windows: Use `__declspec(dllexport)`
* Linux/macOS: Use `__attribute__((visibility("default")))` or compiler flags
* Cross-platform: Consider using preprocessor macros

Example cross-platform macro:
```cpp
#ifdef _WIN32
    #define EXPORT __declspec(dllexport)
#else
    #define EXPORT __attribute__((visibility("default")))
#endif

extern "C" EXPORT Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}
```

See section "Understanding #define (Preprocessor Macros)" further down for more information.

---

## DLL Placement in Unity (Cross-Platform)

#### Place your compiled libraries in the appropriate plugin folders:

**Windows:**
* `Assets/Plugins/` for general Windows builds
* `Assets/Plugins/x86/` for 32-bit Windows
* `Assets/Plugins/x86_64/` for 64-bit Windows
* File extension: `.dll`

**macOS:**
* `Assets/Plugins/` for universal builds
* `Assets/Plugins/macOS/` for macOS-specific builds
* File extension: `.bundle` or `.dylib`
* Note: macOS uses `.bundle` (loadable bundle) or `.dylib` (dynamic library) instead of `.dll`

**Linux:**
* `Assets/Plugins/` for general Linux builds
* `Assets/Plugins/x86_64/` for 64-bit Linux
* File extension: `.so` (shared object)

Unity will automatically select and load the correct library based on the target platform.

**Important:** When building for macOS, you may need to set appropriate code signing and entitlements, especially for builds distributed outside the Mac App Store.

---

### Understanding #define (Preprocessor Macros)

The `#define` directive is a preprocessor command that creates a macro - essentially a find-and-replace operation that happens before your code is compiled.

#### What it does:

`#define` tells the preprocessor: "Every time you see this name, replace it with this code."

#### Basic Syntax:
```cpp
#define NAME replacement_text
```

#### Simple Example:
```cpp
#define PI 3.14159

// When you write:
float radius = 5.0f;
float area = PI * radius * radius;

// The preprocessor replaces it with:
float area = 3.14159 * radius * radius;
```

#### Function-like Macros:

You can also create macros that take parameters:
```cpp
#define SQUARE(x) ((x) * (x))

// When you write:
int result = SQUARE(5);

// It becomes:
int result = ((5) * (5));
```

**Note:** The extra parentheses `((x) * (x))` are important to avoid operator precedence issues.

#### Cross-Platform Example:
```cpp
// Define a platform-specific export macro
#ifdef _WIN32
    #define EXPORT __declspec(dllexport)
#else
    #define EXPORT __attribute__((visibility("default")))
#endif

// Now you can write:
extern "C" EXPORT Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}

// On Windows, the preprocessor converts it to:
extern "C" __declspec(dllexport) Vec3 VectorAdd(Vec3 a, Vec3 b) { ... }

// On Linux/macOS, it becomes:
extern "C" __attribute__((visibility("default"))) Vec3 VectorAdd(Vec3 a, Vec3 b) { ... }
```

#### Key Points:

* **Preprocessor runs first:** Before the compiler sees your code, the preprocessor performs text replacement
* **No type checking:** Macros are pure text replacement, so they don't have type safety
* **Debugging can be tricky:** The debugger sees the replaced code, not your macro names
* **Convention:** Macro names are typically written in UPPERCASE to distinguish them from functions

#### Common Preprocessor Conditionals:
```cpp
#ifdef SYMBOL      // If SYMBOL is defined
#ifndef SYMBOL     // If SYMBOL is NOT defined
#if condition      // If condition is true
#else              // Otherwise
#endif             // End conditional block
```

#### Practical Use in DLLs:
```cpp
// In a header file shared between your DLL and Unity:

#ifdef BUILD_DLL
    #define API_EXPORT __declspec(dllexport)
#else
    #define API_EXPORT __declspec(dllimport)
#endif

// When compiling the DLL, define BUILD_DLL
// When using the DLL in Unity, don't define it
// This way the same header works for both building and using the DLL
```

This approach lets you write platform-independent code where `EXPORT` automatically becomes the correct syntax for whatever platform you're compiling for, making your code cleaner and more maintainable.

* More information on .h (header files) & .cpp (implementation files) in 04_CPP