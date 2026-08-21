# DALi Coding Guidelines
To ensure the quality and consistency of contributions to DALi, all developers are required to adhere to the DALi Coding Style and Conventions. Additionally, it is crucial to guarantee that all code is ABI (Application Binary Interface) compatible, maintaining seamless integration and stability across different components and platforms.

## Table of Contents
- [DALi Coding Guidelines](#dali-coding-guidelines)
  - [Table of Contents](#table-of-contents)
  - [Coding Style](#coding-style)
  - [Coding Convention](#coding-convention)
  - [ABI Compatibility](#abi-compatibility)
    - [Class Layout \& Memory Alignment](#class-layout--memory-alignment)
    - [Virtual Function Tables (vtable)](#virtual-function-tables-vtable)
    - [Name Mangling](#name-mangling)
    - [Classes Vs. Structs](#classes-vs-structs)
    - [Data Type Sizes and Endianness](#data-type-sizes-and-endianness)
    - [Static and Global Variables](#static-and-global-variables)
    - [Enumerations](#enumerations)
    - [Explicitly Exporting Symbols](#explicitly-exporting-symbols)
    - [Deprecation of APIs](#deprecation-of-apis)
    - [Categories of APIs in DALi Libraries](#categories-of-apis-in-dali-libraries)
      - [Public API](#public-api)
      - [Devel API](#devel-api)
      - [Integration API](#integration-api)
    - [Use of the Standard Template Library (STL)](#use-of-the-standard-template-library-stl)


## Coding Style
A consistent coding style is essential for enhancing readability, maintainability, and collaboration in software development. It ensures that code is easy to understand and modify, reduces errors, and facilitates smoother teamwork. By adhering to a uniform style, developers can improve onboarding for new team members, leverage automation tools effectively, and project professionalism, ultimately leading to more robust and reliable software.

DALi's coding guidelines can be found [here](coding-style.md).

## Coding Convention
Coding conventions are essential for fostering a cohesive and efficient development process. They standardize code structure, naming, and formatting, making it easier for developers to read, understand, and maintain the codebase.

DALi has a coding convention which can be found [here](coding-convention.md).

## ABI Compatibility
The ABI (Application Binary Interface) defines how software components interact at the binary level, ensuring compatibility between compiled code, libraries, and the runtime environment. It specifies details like function call mechanisms, data structure layouts, and object file formats, enabling different parts of a program to work together seamlessly. ABI stability is crucial because it allows developers to update or replace components without breaking the entire system, facilitating modular design and long-term maintainability. By adhering to ABI standards, software can achieve cross-platform compatibility, efficient updates, and smoother integration with third-party libraries, ultimately enhancing reliability and reducing development overhead.

The DALi library incorporates various practices to maintain ABI compatibility.

### Class Layout & Memory Alignment
In C++, adding or removing data members can disrupt ABI compatibility, especially since header files are directly exposed to application developers. To mitigate this, DALi employs the [Pimpl idiom](https://www.geeksforgeeks.org/cpp/pimpl-idiom-in-c-with-examples/) which encapsulates implementation details and ensures that changes to the internal structure do not affect the public interface.

### Virtual Function Tables (vtable)
Modifying the order of virtual functions or adding/removing them can disrupt the vtable layout, leading to ABI compatibility issues.

To address this, our classes in the Public API generally avoid including virtual methods. Instead, we utilize the handle-body idiom with BaseHandle and BaseObject. BaseHandle is exposed to application developers, while BaseObject remains internal and can contain virtual methods. Please refrain from adding any virtual methods to Public classes.

Exceptions to this rule include CustomActorImpl and ControlImpl, where virtual methods are allowed. However, these classes must not have virtual methods added or removed, and the order of existing methods must remain unchanged. If new virtual methods are required, the Extension interface in these classes should be utilized to maintain ABI compatibility.

```c++
class CustomActorImpl
{
public:
  class Extension; ///< Forward declare future extension interface
  ...
  virtual Extension* GetExtension()
  {
    return nullptr;
  }
  ...
};
```

Additionally, but very importantly, the inheritance of public classes CANNOT be changed.

### Name Mangling
Compiler-specific name mangling can lead to linker errors when combining object files compiled with different compilers, as the mangled names may not match. To prevent this, use ``extern "C"`` to disable name mangling for C++ functions that are exposed to C code. This ensures consistent naming conventions across compilers, facilitating seamless integration between C and C++ components.

### Classes Vs. Structs
Avoid using visible structs, even for simple data, as they can become problematic if it has to be extended in the future. Instead, opt for a class with well-defined getters and setters, and that use the [Pimpl idiom](https://www.geeksforgeeks.org/cpp/pimpl-idiom-in-c-with-examples/). This approach provides greater flexibility and control over data access and modification, allowing for easier maintenance and scalability.

### Data Type Sizes and Endianness
Differences in data type sizes (e.g., ``int``, ``long``) or endianness across platforms can lead to ABI compatibility issues, as these variations can affect how data is interpreted and stored. To address this, use fixed-width integer types (such as ``int32_t``, ``uint64_t``, etc.) from the ``<cstdint>`` header, ensuring consistent sizes regardless of the platform.

### Static and Global Variables
Changes in the initialization order or definition of static or global variables can disrupt ABI compatibility, as these variables are tied to specific memory locations and initialization sequences. To address this, DALi enforces strict guidelines: global variables are prohibited, and static variables should be confined to ``.cpp`` files or restricted to internal library use.

### Enumerations
Once enumerations are added to the public API, their values and order must remain unchanged to maintain ABI compatibility. To accommodate future additions, new enumerations can be appended to the end of the list. If there's a likelihood that additional enumerations will be needed in the future and should be grouped closely, reserve space by adding padding:

```c++
// Public API Enumeration (e.g., in a header file)
enum class Color : uint8_t
{
  RED      = 0,
  GREEN    = 1,
  BLUE     = 2,
  // Reserved space for future additions (not explicitly named)
  YELLOW   = 10
};
```

In this example, the Color enumeration includes RED, GREEN, and BLUE, with YELLOW added at the end with a value of 10. This leaves room for additional values between BLUE and YELLOW without affecting existing code.

```c++
// Future Addition (e.g., in a new version)
enum class Color : uint8_t
{
  RED      = 0,
  GREEN    = 1,
  BLUE     = 2,
  CYAN     = 3,  // 3 (New value, similar to BLUE)
  TEAL     = 4,  // 4 (New value, similar to BLUE)
  YELLOW   = 10  // 10 (Existing value, unchanged)
};
```

New enumerations like CYAN and TEAL are added between BLUE and YELLOW, using values 3 and 4. The existing value of YELLOW remains unchanged (10), ensuring ABI compatibility.

### Explicitly Exporting Symbols
By default, functions and methods in the DALi libraries are not exported. Developers must explicitly mark symbols as exportable to make them accessible to external units. The appropriate macro to use depends on the specific DALi library:

- ``DALI_CORE_API``: Use this macro when exporting symbols from the DALi Core library.
- ``DALI_ADAPTOR_API``: Use this macro when exporting symbols from the DALi Adaptor library.
- ``DALI_TOOLKIT_API``: Use this macro when exporting symbols from the DALi Toolkit library.

```c++
// Exporting a function from the DALi Core library
DALI_CORE_API void CoreFunction();

// Exporting a function from the DALi Adaptor library
DALI_ADAPTOR_API void AdaptorFunction();

// Exporting a class from the DALi Toolkit library
class DALI_TOOLKIT_API ToolkitClass {
public:
  void ToolkitMethod();
};
```

It is recommended that other libraries follow this pattern as well. See ``dali-common.h`` in the dali-core repository for more information.

### Deprecation of APIs
When an API needs to be deprecated, it should be clearly marked in both the documentation and the code to ensure developers are aware of its status and can transition to alternative solutions. Mark the deprecated API in the doxygen documentation with a clear indication and use the ``DALI_DEPRECATED_API`` macro to alert developers to the deprecation and encourage them to update their code.

```c++
/**
 * DEPRECATED_1_1.39 Position based retrieval is no longer supported after extending the key type to both Index and String.
 * ...
 */
  const std::string& GetKey(SizeType position) const DALI_DEPRECATED_API;
```

The deprecated API remains available for use for two minor version increases after deprecation. This provides developers with sufficient time to update their code. After two minor version increases, the deprecated API is removed from the codebase.

Example Timeline:
 - Version 1.0: API is introduced.
 - Version 1.2: API is deprecated (marked in documentation and flagged with DALI_DEPRECATED_API).
 - Version 1.4: Deprecated API is still available but will be removed in the next minor version.
 - Version 1.6: Deprecated API is removed.

### Categories of APIs in DALi Libraries
APIs exported by the DALi libraries are categorized into three distinct groups, each with specific usage and stability guarantees:

#### Public API
These are designed for application developers to build their applications and are guaranteed to remain consistent across DALi versions unless explicitly deprecated. Application developers can rely on these APIs for long-term development, ensuring backward compatibility. This API can be found in the ``public-api`` folder.

When adding an API to the public API, it's essential to document the version in which the API was introduced using a doxygen tag:

```c++
/**
 * @brief A brief description of my new function.
 * @SINCE_1_9.28
 */
void MyNewFunction();
```

The @SINCE_1_9.28 tag indicates that the API MyNewFunction was introduced in version 1.9.28 of the library.

#### Devel API
These are experimental APIs used by DALi developers to create new effects and features, and are subject to change at any point until they are moved to the Public API. They are intended for internal development and testing, not for application developers. This API can be found in the ``devel-api`` folder.

C-style functions can be used to extend the functionality of a class by taking a handle to the class as the first parameter. This approach allows for additional features without modifying the class itself. Here's an example:

```c++
DALI_CORE_API Vector2 CalculateScreenPosition(Actor actor);
```

#### Integration API
These APIs are used for communication between DALi libraries, and can be modified by DALi developers as needed, as all libraries are compiled and installed together. The are internal to the DALi ecosystem, and not exposed to application developers. This API can be found in the ``integration-api`` folder.

### Use of the Standard Template Library (STL)
The use of STL containers in public API headers is discouraged due to potential ABI issues that may arise from changes in compiler versions, as the memory layout and size of STL containers like ``std::vector`` and ``std::map`` can vary. While STL containers can be safely used internally or within ``.cpp`` files, exposing them in public APIs should be avoided.

Similarly, ``std::string`` can suffer from ABI issues due to changes in memory layout or allocator behavior, but its use in the public API is not prohibited due to the performance and memory costs of wrapping it. However, ``std::string`` should only be used in the public API if absolutely necessary.
