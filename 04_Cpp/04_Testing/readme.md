# Testing Your DLL

## Creating a Test Project

A quick and easy way to test your implementation is to create a small console test project where you can verify that your functions work as expected:

Example:
```cpp
#include <iostream>
#include "../VectorMath.h"  // Note: Path may vary based on your structure

int main() {
    Vec3 a = { 1.0f, 2.0f, 3.0f };
    Vec3 b = { 4.0f, 5.0f, 6.0f };

    Vec3 result = VectorAdd(a, b);
    std::cout << "Result: (" << result.x << ", "
              << result.y << ", " << result.z << ")\n";

    float mag = VectorMagnitude(a);
    std::cout << "Magnitude of a: " << mag << "\n";

    std::cout << "\nPress Enter to exit..." << std::endl;
    std::cin.get();  // Keeps window open

    return 0;
}
```

---

## How to Add It to Your Solution

With your DLL solution open in Visual Studio:

1. Right-click the **solution** (not the project) in Solution Explorer
2. Go to `Add` → `New Project`
3. Select `Console App` (C++)
4. Name it the same as your DLL + "Tests" (e.g., `VectorMathematicsTests`)
5. Click `Create`

You should now see a solution with 2 projects.

---

## Linking Them Together

Now that you have the test project, we need to configure it to use your DLL.

**Important:** Make sure you've built your DLL project at least once before continuing. This creates the `.lib` and `.dll` files needed for linking.

### Step 1: Configure Project Properties

1. Right-click your **test project** → `Properties`
2. **Important:** Set `Configuration` dropdown at the top to **"All Configurations"**

### Step 2: Set Up Include Directories

Navigate to: **C/C++ → General → Additional Include Directories**

**Add one of these (depending on your project structure):**

**If solution and project are in the same directory:**
```
$(SolutionDir)
```

**If solution and project are in separate directories:**
```
$(SolutionDir)VectorMathematics
```

Click `Apply`

---

### Step 3: Set Up Library Directories

Navigate to: **Linker → General → Additional Library Directories**

**Add:**
```
$(SolutionDir)x64\$(Configuration)
```

Click `Apply`

**Note**: Again, depending on if the project and solution is in separate folders.

---

### Step 4: Link the Library File

Navigate to: **Linker → Input → Additional Dependencies**

**Important:** Don't replace the existing text, **append** to it!

**Add this to the end (with a semicolon):**
```
;VectorMathematics.lib
```

The full line should look like:
```
$(CoreLibraryDependencies);%(AdditionalDependencies);VectorMathematics.lib
```

Click `OK` to close the properties window.

---

### Step 5: Set Project Dependencies

This ensures your DLL builds before the test project:

1. Right-click the **solution** (not project) → `Project Dependencies`
    - **Alternative:** Right-click test project → `Build Dependencies` → `Project Dependencies`
2. Under `Projects:` dropdown, select your **test project**
3. In the `Depends on:` list, **check the box** next to your DLL project
4. Click `OK`

---

### Step 6: Set Test Project As Startup

This ensures your test project is the one that runs:

* Right-click the **Test Project** → `Set as Startup Project`

---

## Verify the Setup

### Test the Include Path

In your test project's `.cpp` file, try adding:
```cpp
#include "VectorMath.h"  // Should work if configured correctly
```

**If you get an error:** The include path might need adjustment based on your folder structure.

---

### Build and Run

1. Right-click your **test project** → `Set as Startup Project` (if you skipped step 6)
2. Build the solution: `Build` → `Build Solution` (or press `Ctrl+Shift+B`)
3. Run the test: Press `Ctrl+F5` (runs without debugger - recommended for tests)

**Expected output:**
```
Result: (5, 7, 9)
Magnitude of a: 3.74166

Press Enter to exit...
```

---

## Common Issues and Solutions

### Issue 1: "Cannot open include file 'VectorMath.h'"

**Solution:** Check your Additional Include Directories path:
- If solution and project are in same directory: Use `$(SolutionDir)`
- If separate directories: Use `$(SolutionDir)VectorMathematics`
- As a temporary fix: Use relative path `#include "../VectorMath.h"`

---

### Issue 2: "Cannot open file 'VectorMathematics.lib'"

**Solution:**
1. Make sure you've built the DLL project at least once
2. Check that the `.lib` file exists in `x64\Debug\` or `x64\Release\`
3. Verify Additional Library Directories is set to `$(SolutionDir)x64\$(Configuration)`

---

### Issue 3: "Cannot open file 'VectorMathematicsTests.exe'" when building

**Solution:** The .exe is locked (probably still running)
1. Close any open console windows from previous runs
2. Try `Build` → `Clean Solution` then rebuild
3. If still stuck, restart Visual Studio

---

## Writing Better Tests

Instead of just printing values, write actual tests with assertions: (don't forget the include for assertions: `#include <cassert>`)
```cpp
#include <iostream>
#include <cassert>
#include <cmath>
#include "VectorMath.h"

// Helper function for float comparison
bool FloatEquals(float a, float b, float epsilon = 0.0001f) {
    return fabs(a - b) < epsilon;
}

void TestVectorAdd() {
    std::cout << "Testing VectorAdd..." << std::endl;
    
    Vec3 a = { 1.0f, 2.0f, 3.0f };
    Vec3 b = { 4.0f, 5.0f, 6.0f };
    Vec3 result = VectorAdd(a, b);
    
    // Use assertions to verify correctness
    assert(result.x == 5.0f && "VectorAdd X component failed");
    assert(result.y == 7.0f && "VectorAdd Y component failed");
    assert(result.z == 9.0f && "VectorAdd Z component failed");
    
    std::cout << "  PASSED!" << std::endl;
}

void TestVectorMagnitude() {
    std::cout << "Testing VectorMagnitude..." << std::endl;
    
    // Test with 3-4-5 triangle (should equal 5)
    Vec3 v = { 3.0f, 4.0f, 0.0f };
    float mag = VectorMagnitude(v);
    
    assert(FloatEquals(mag, 5.0f) && "VectorMagnitude failed");
    
    std::cout << "  Magnitude: " << mag << std::endl;
    std::cout << "  PASSED!" << std::endl;
}

void TestVectorNormalize() {
    std::cout << "Testing VectorNormalize..." << std::endl;
    
    Vec3 v = { 3.0f, 4.0f, 0.0f };
    Vec3 normalized = VectorNormalize(v);
    float mag = VectorMagnitude(normalized);
    
    // Normalized vector should have magnitude of 1
    assert(FloatEquals(mag, 1.0f) && "Normalized vector magnitude should be 1");
    
    std::cout << "  Normalized magnitude: " << mag << std::endl;
    std::cout << "  PASSED!" << std::endl;
}

void TestZeroVectorNormalize() {
    std::cout << "Testing zero vector normalization..." << std::endl;
    
    Vec3 zero = { 0.0f, 0.0f, 0.0f };
    Vec3 result = VectorNormalize(zero);
    
    // Should return zero vector, not crash
    assert(result.x == 0.0f && result.y == 0.0f && result.z == 0.0f);
    
    std::cout << "  PASSED!" << std::endl;
}

int main() {
    std::cout << "=== Vector Math Tests ===" << std::endl << std::endl;
    
    TestVectorAdd();
    TestVectorMagnitude();
    TestVectorNormalize();
    TestZeroVectorNormalize();
    
    std::cout << std::endl << "All tests passed!" << std::endl;
    
    std::cout << "\nPress Enter to exit..." << std::endl;
    std::cin.get();
    
    return 0;
}
```

**Note**: Assert will crash the program if the condition is false.

---

### Build your own assertion methods

A way to mitigate this is to build custom assertions:

```cpp
// Custom assert function
void Assert(bool condition, const char* message) {
    if (!condition) {
        std::cerr << "[FAIL] " << message << std::endl;
        std::cin.get();  // Pause so you can see the error
        exit(1);  // Exit with error code
        // OR continue without pause and exit to run all tests with output information
    } else {
        std::cout << "[PASS] " << message << std::endl;
    }
}
```

---

## Tips

✅ **Always build in order:** DLL first, then test project (dependencies handle this automatically)

✅ **Use Ctrl+F5** instead of F5 to run tests (keeps console window open automatically)

✅ **Test edge cases:** Zero vectors, unit vectors, very large/small values

✅ **Add tests as you implement:** Don't wait until everything is done

✅ **Use assertions:** They stop execution at the first failure, making debugging easier

---

## Next Steps

Once your basic tests work:
1. Add tests for all your vector functions
2. Test edge cases (zero vectors, negative values, etc.)
3. Compare results with Unity's built-in Vector3 to verify correctness
4. Add performance tests if needed

Your test project is now ready to help you verify your DLL works correctly before integrating it with Unity!