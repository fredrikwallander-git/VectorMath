# Pointers and References in C++

## Understanding Memory First

Before understanding pointers and references, you need to understand how variables are stored.

### Variables in Memory

When you declare a variable, the computer allocates memory for it:
```cpp
int health = 100;
```

This creates:
* A space in memory (let's say at address `0x1000`)
* The value `100` stored at that address
* A name `health` that refers to that address

Every variable has:
1. **A value** - what it contains (100)
2. **An address** - where it lives in memory (0x1000)
3. **A type** - what kind of data it is (int)

---

## The & Operator (Two Different Meanings!)

The `&` symbol has **two completely different uses** in C++:

### 1. Address-of Operator (&)

Gets the memory address of a variable:
```cpp
int health = 100;
int* healthAddress = &health;  // Get the address of health

// If health is at memory address 0x1000:
// healthAddress now contains 0x1000
```

**Meaning:** "Give me the address where this variable lives"

---

### 2. Reference Declaration (&)

Creates an alias (another name) for an existing variable:
```cpp
int health = 100;
int& healthRef = health;  // healthRef is another name for health

healthRef = 50;  // Changes health to 50
// health is now 50 too - they're the same variable!
```

**Meaning:** "This is another name for the same thing"

**Important:** Context determines which meaning:
```cpp
int& ref = x;    // Declaration: this is a reference
int* ptr = &x;   // Expression: get address of x
```

---

## Pointers (*)

### What is a Pointer?

A pointer is a variable that stores a memory address.
```cpp
int health = 100;
int* healthPointer = &health;  // healthPointer stores the address of health
```

**Analogy:**
* Variable = A house with a value inside
* Pointer = A piece of paper with the house's address written on it

---

### Pointer Syntax
```cpp
int* ptr;     // Declares a pointer to an int
float* ptr2;  // Declares a pointer to a float
Vec3* ptr3;   // Declares a pointer to a Vec3
```

**The `*` in declaration means:** "This variable holds an address of this type"

---

### Dereferencing (*)

To access the value at the address a pointer holds, use `*` (dereferencing):
```cpp
int health = 100;
int* healthPointer = &health;

std::cout << healthPointer;   // Prints the address (e.g., 0x1000)
std::cout << *healthPointer;  // Prints the value (100)

*healthPointer = 50;  // Changes health to 50
```

**The `*` when used with an existing pointer means:** "Go to this address and get/set the value"

---

### Visual Example
```cpp
int score = 42;
int* ptr = &score;

// Memory visualization:
// 
// score:  [  42  ] at address 0x1000
//            ↑
//            |
// ptr:    [0x1000] at address 0x2000
```
```cpp
std::cout << score;   // 42 (the value)
std::cout << &score;  // 0x1000 (the address)
std::cout << ptr;     // 0x1000 (pointer holds address)
std::cout << *ptr;    // 42 (dereference to get value)
```

---

### Pointer Operations
```cpp
int x = 10;
int* ptr = &x;

// Read value
int value = *ptr;  // value = 10

// Modify value
*ptr = 20;         // x is now 20

// Check if pointer is valid
if (ptr != nullptr) {
    // Safe to use
}

// Null pointer (points to nothing)
int* emptyPtr = nullptr;
```

---

## References (&)

### What is a Reference?

A reference is an alias - another name for an existing variable.
```cpp
int health = 100;
int& healthRef = health;  // healthRef is health

healthRef = 50;   // health is now 50
health = 75;      // healthRef is now 75
// They are the SAME variable with two names
```

**Analogy:**
* Variable = Person named "Robert"
* Reference = Same person, but you call them "Bob"
* Changing Bob changes Robert (they're the same person!)

---

### Reference Rules

**1. Must be initialized when declared:**
```cpp
int& ref;          // ERROR: reference must be initialized
int& ref = x;      // OK
```

**2. Cannot be reassigned:**
```cpp
int a = 10;
int b = 20;
int& ref = a;      // ref refers to a
ref = b;           // Does NOT make ref refer to b
                   // Instead, it copies b's value into a
```

**3. Cannot be null:**
```cpp
int& ref = nullptr;  // ERROR: references cannot be null
```

---

## Pointers vs References

### Key Differences

| Aspect | Pointer (*) | Reference (&) |
|--------|-------------|---------------|
| **Can be null** | Yes (`nullptr`) | No |
| **Can be reassigned** | Yes | No |
| **Must be initialized** | No | Yes |
| **Syntax to use** | Need `*` to dereference | Use directly |
| **Memory** | Takes memory space | No extra memory |
| **Safety** | Can dangle or be null | Safer (usually) |

---

### When to Use Which?

**Use Pointers when:**
* You need to represent "no value" (nullptr)
* You need to change what it points to
* Working with dynamic memory (`new`/`delete`)
* Interfacing with C libraries or APIs
* Optional parameters

**Use References when:**
* Passing large objects to functions efficiently
* Creating aliases for readability
* Operator overloading
* The reference will always be valid
* You want cleaner syntax

---

## Function Parameters

### Pass by Value (Copy)
```cpp
void modifyValue(int x) {
    x = 100;  // Only modifies the local copy
}

int health = 50;
modifyValue(health);
// health is still 50 (original unchanged)
```

**Problem:** Copies the data (expensive for large objects)

---

### Pass by Pointer
```cpp
void modifyValue(int* x) {
    if (x != nullptr) {
        *x = 100;  // Modifies the original
    }
}

int health = 50;
modifyValue(&health);  // Pass address
// health is now 100
```

**Pros:**
* Can modify original
* Can pass nullptr for "no value"

**Cons:**
* Verbose syntax
* Must check for nullptr
* Can be null (sometimes unintended)

---

### Pass by Reference
```cpp
void modifyValue(int& x) {
    x = 100;  // Modifies the original
}

int health = 50;
modifyValue(health);  // No & needed when calling
// health is now 100
```

**Pros:**
* Clean syntax
* Can modify original
* Cannot be null

**Cons:**
* Not obvious at call site that value might change

---

### Pass by Const Reference (Most Common)
```cpp
void printVector(const Vec3& v) {
    std::cout << v.x << ", " << v.y << ", " << v.z;
    // v.x = 10;  // ERROR: cannot modify const reference
}

Vec3 position = { 1, 2, 3 };
printVector(position);  // Efficient (no copy) and safe (cannot modify)
```

**Best practice for reading large objects:** Pass by const reference
* No copy (efficient)
* Cannot be modified (safe)
* Clear intent

---

## Practical Examples

### Example 1: Swapping Values

**Using Pointers:**
```cpp
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int x = 5, y = 10;
swap(&x, &y);  // x = 10, y = 5
```

**Using References:**
```cpp
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int x = 5, y = 10;
swap(x, y);  // x = 10, y = 5 (cleaner call site!)
```

---

### Example 2: Vector Normalization

**Return by value (typical):**
```cpp
Vec3 VectorNormalize(Vec3 v) {
    float mag = VectorMagnitude(v);
    if (mag < 0.0001f) return { 0, 0, 0 };
    return { v.x / mag, v.y / mag, v.z / mag };
}

Vec3 dir = { 3, 4, 0 };
Vec3 normalized = VectorNormalize(dir);
```

**Modify in place (using pointer):**
```cpp
void VectorNormalize(Vec3* v) {
    if (v == nullptr) return;
    
    float mag = sqrtf(v->x * v->x + v->y * v->y + v->z * v->z);
    if (mag < 0.0001f) {
        v->x = 0; v->y = 0; v->z = 0;
        return;
    }
    v->x /= mag;
    v->y /= mag;
    v->z /= mag;
}

Vec3 dir = { 3, 4, 0 };
VectorNormalize(&dir);  // dir is modified directly
```

**Note:** The `->` operator is shorthand:
```cpp
v->x  // Same as (*v).x
```

---

### Example 3: Passing Large Structs Efficiently

**Bad (copies entire struct):**
```cpp
Vec3 ProcessVector(Vec3 v) {  // Copies all data
    // ... process v
    return v;  // Copies again!
}
```

**Good (passes reference):**
```cpp
Vec3 ProcessVector(const Vec3& v) {  // No copy, cannot modify
    Vec3 result;
    // ... process v, store in result
    return result;
}
```

---

## Pointers and Arrays

Arrays and pointers are closely related in C++:
```cpp
int numbers[5] = { 1, 2, 3, 4, 5 };
int* ptr = numbers;  // Array name decays to pointer

std::cout << ptr[0];   // 1
std::cout << *ptr;     // 1 (same thing)
std::cout << *(ptr+1); // 2 (pointer arithmetic)
```

**Pointer arithmetic:**
```cpp
int* ptr = numbers;
ptr++;        // Points to next int
ptr += 2;     // Skip 2 ints forward
int value = *ptr;  // Get value at current position
```

---

## Common Pointer Pitfalls

### 1. Dangling Pointers
```cpp
int* getDanglingPointer() {
    int x = 42;
    return &x;  // BAD: x is destroyed when function returns
}

int* ptr = getDanglingPointer();
*ptr = 10;  // CRASH: accessing destroyed variable
```

---

### 2. Null Pointer Dereference
```cpp
int* ptr = nullptr;
*ptr = 10;  // CRASH: cannot dereference null pointer

// Always check:
if (ptr != nullptr) {
    *ptr = 10;  // Safe
}
```

---

### 3. Memory Leaks
```cpp
void leak() {
    int* ptr = new int(42);  // Allocate memory
    // Forgot to delete!
}  // Memory is lost forever

// Correct:
void noLeak() {
    int* ptr = new int(42);
    delete ptr;  // Free memory
}
```

---

### 4. Double Delete
```cpp
int* ptr = new int(42);
delete ptr;
delete ptr;  // CRASH: already deleted

// Solution:
delete ptr;
ptr = nullptr;  // Set to null after deleting
```

---

## Pointers and References Across Languages

### C++ to C#/Unity (DLL Boundary)

**Problem:** Pointers don't cross the DLL boundary safely.

**DON'T do this:**
```cpp
// C++
extern "C" __declspec(dllexport) Vec3* CreateVector() {
    return new Vec3{ 1, 2, 3 };  // Dangerous!
}
```
```csharp
// C# - How do you handle this pointer?
IntPtr vectorPtr = CreateVector();  // What now?
```

**DO this instead (return by value):**
```cpp
// C++
extern "C" __declspec(dllexport) Vec3 CreateVector() {
    return { 1, 2, 3 };  // Returns struct by value
}
```
```csharp
// C#
Vec3 vector = CreateVector();  // Clean and safe!
```

---

### If You Must Use Pointers Across DLL Boundary

**Approach 1: IntPtr (Opaque Handle)**
```cpp
// C++
extern "C" __declspec(dllexport) void* CreateVector() {
    return new Vec3{ 1, 2, 3 };
}

extern "C" __declspec(dllexport) void DestroyVector(void* ptr) {
    delete static_cast<Vec3*>(ptr);
}

extern "C" __declspec(dllexport) float GetX(void* ptr) {
    return static_cast<Vec3*>(ptr)->x;
}
```
```csharp
// C#
[DllImport("VectorMath")]
public static extern IntPtr CreateVector();

[DllImport("VectorMath")]
public static extern void DestroyVector(IntPtr ptr);

[DllImport("VectorMath")]
public static extern float GetX(IntPtr ptr);

// Usage
IntPtr vectorPtr = CreateVector();
float x = GetX(vectorPtr);
DestroyVector(vectorPtr);  // Must remember to clean up!
```

**Issues:**
* Manual memory management
* Easy to forget to free
* Verbose

---

**Approach 2: Out Parameters**
```cpp
// C++
extern "C" __declspec(dllexport) void CreateVector(Vec3* outVec) {
    if (outVec != nullptr) {
        outVec->x = 1;
        outVec->y = 2;
        outVec->z = 3;
    }
}
```
```csharp
// C#
[DllImport("VectorMath")]
public static extern void CreateVector(out Vec3 outVec);

// Usage
CreateVector(out Vec3 vector);  // vector is filled in
```

**Better but still awkward.**

---

**Best Practice: Return by Value**
```cpp
// C++ - Simple and clean
extern "C" __declspec(dllexport) Vec3 VectorAdd(Vec3 a, Vec3 b) {
    return { a.x + b.x, a.y + b.y, a.z + b.z };
}
```
```csharp
// C# - Simple and clean
[DllImport("VectorMath")]
public static extern Vec3 VectorAdd(Vec3 a, Vec3 b);

Vec3 result = VectorAdd(a, b);  // Just works!
```

---

## Memory Management Summary

### Stack vs Heap

**Stack (Automatic):**
```cpp
void function() {
    int x = 10;        // Stack
    Vec3 v = {1,2,3};  // Stack
}  // Automatically cleaned up
```
* Fast
* Automatically managed
* Limited size

**Heap (Dynamic):**
```cpp
void function() {
    int* x = new int(10);        // Heap
    Vec3* v = new Vec3{1,2,3};   // Heap
    
    delete x;   // Must manually free
    delete v;
}
```
* Slower
* Manual management
* Large size available
* Can outlive function scope

---

## Best Practices

### For DLL Functions (Unity):

1. ✅ **Return small structs by value**
```cpp
   Vec3 VectorAdd(Vec3 a, Vec3 b);
```

2. ✅ **Pass large structs by const reference (read-only)**
```cpp
   float CalculateComplexity(const LargeStruct& data);
```

3. ✅ **Use out parameters for multiple return values**
```cpp
   void GetMinMax(Vec3* values, int count, Vec3* outMin, Vec3* outMax);
```

4. ❌ **Avoid returning pointers**
```cpp
   Vec3* CreateVector();  // DON'T
```

5. ❌ **Avoid passing ownership across boundary**
```cpp
   void TakeOwnership(Vec3* ptr);  // Who deletes this?
```

---

### General C++ Best Practices:

1. **Prefer references over pointers when possible**
```cpp
   void Process(const Vec3& v);  // Better than: void Process(Vec3* v);
```

2. **Check pointers before dereferencing**
```cpp
   if (ptr != nullptr) { *ptr = value; }
```

3. **Use smart pointers for ownership (modern C++)**
```cpp
   std::unique_ptr<Vec3> v = std::make_unique<Vec3>(1, 2, 3);
   // Automatically deleted
```

4. **Const-correctness**
```cpp
   void Read(const Vec3& v);   // Won't modify
   void Write(Vec3& v);        // Might modify
```

---

## Quick Reference

### Syntax Summary
```cpp
int x = 42;
int* ptr = &x;      // ptr holds address of x
int& ref = x;       // ref is another name for x

// Pointer operations:
int value = *ptr;   // Dereference: get value
*ptr = 100;         // Dereference: set value
ptr = nullptr;      // Point to nothing

// Reference operations:
ref = 100;          // Set value (no special syntax)
int value = ref;    // Get value (no special syntax)

// Cannot do:
ref = nullptr;      // ERROR: references cannot be null
int& ref2;          // ERROR: must initialize
ref = otherVar;     // Does NOT rebind, copies value
```

---

## Conclusion

**Pointers** give you power and flexibility but require careful management.

**References** give you safety and clean syntax but are more restrictive.

**For Unity DLLs:** Keep it simple - pass structs by value, avoid pointers when possible, and never mix C++ heap allocations with C# memory management.

Understanding these concepts is crucial for:
* Writing efficient code
* Creating safe DLLs
* Avoiding memory leaks and crashes
* Interfacing between C++ and other languages