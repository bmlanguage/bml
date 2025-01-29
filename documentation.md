
**Bare Metal Language (BML) Specification (Version 1.9 - Complete)**

**1. Introduction**

The Bare Metal Language (BML) is a high-level, statically-typed language meticulously designed for bare metal and embedded systems development. It aims to provide a productive, maintainable, and safe alternative to assembly language, while retaining the fine-grained control and performance characteristics necessary in such environments. BML provides direct hardware access with minimal runtime overhead, explicit memory safety features, and concurrent programming constructs, while also focusing on portability across different architectures. Composable Architecture Modules (CAMs) are a central element of BML, enabling architecture-specific code and optimizations. Version 1.9 introduces significant enhancements, including constrained instruction DSLs within CAMs, dynamically extensible CAMs, Meta-CAMs for automated CAM generation, improved CAM composition and customization, hardware-assisted bounds checking, static analysis with refinement types, advanced debugging with compiler output and CAM level debug, updated memory model, concise struct/union/tuple initialization, implicit bitfield access, optional parentheses for single-argument functions, default function parameter values, and concise result handling with the `?` operator. BML is intended to be a high-level, statically-typed replacement for assembly language, providing similar levels of performance and fine-grained hardware control *where applicable*, while drastically improving code maintainability, type safety, and developer productivity. BML achieves this through its Composable Architecture Module (CAM) system, allowing platform-specific optimizations. While BML abstracts away direct instruction-level control, its focus is on providing similar levels of performance using high-level constructs.

**Core Principles:**

*   **Zero Overhead:** BML is designed with a *zero-overhead* philosophy. It prioritizes minimal runtime costs, striving for performance comparable to hand-crafted assembly code where applicable. This means no hidden allocations, dynamic dispatch (unless explicitly requested), or garbage collection, ensuring predictable and efficient performance. BML achieves this by using compile-time checks and optimizations, hardware capabilities, and providing low-level access mechanisms.
*   **Low-Level Control:** BML provides mechanisms for direct hardware interaction, precise memory manipulation, and bit-level control. While abstracting away direct instruction encoding at user-level, BML, via Composable Architecture Modules (CAMs), allows for highly optimized code generation that is as close as possible to hand-written assembly. The developer has a high degree of control over the target device, using high-level constructs with low-level hardware support.
*   **Memory Safety Mitigation:** BML incorporates a comprehensive set of features aimed at mitigating common memory errors at compile time. Static analysis, type constraints, region-based memory management, and hardware-assisted bounds checking provide a layered approach to memory safety, with optional runtime checks available when needed, giving the user more control. BML does not enforce runtime checks, and allows the user to bypass them when needed, to achieve the maximum performance. The tradeoff between safety and performance is up to the user.
*   **Multi-Target Versatility:** BML supports compilation for diverse architectures through flexible CAM configurations. CAMs abstract away architecture-specific details, promoting code reuse across different platforms.
*   **Modularity through CAMs:** BML enables the construction of systems from reusable, optimized, and type-safe modules using the Composable Architecture Module system. CAMs provide a powerful mechanism for code reuse and platform specific optimizations.
*   **Readability and Maintainability:** BML prioritizes clear, concise code to enhance software engineering practices. This improves the understanding of the code, and simplifies long-term maintenance.
*   **Tooling and Debugging:** BML enables source-level debugging with industry-standard tools (GDB/LLDB), and also provides more advanced debugging output from the compiler, promoting efficient troubleshooting and code understanding. BML also provides advanced tooling to debug CAMs and their code transformations.
*   **Concurrency and Parallelism:** BML includes built-in language-level support for multi-core processors, offering primitives for concurrent tasks and explicit control over synchronization and memory consistency. BML offers high level concurrency primitives such as threads, mutexes and semaphores that can be used to implement concurrent programs.
*   **Metaprogramming:** BML provides compile-time computation and reflection capabilities that allow to extend the language and implement advanced optimizations by using traits, compile-time functions, and DSLs. This enables developers to create very flexible CAMs and perform code optimizations at compile time.
*   **AI-Powered Bit-Level Optimizations:** The compiler uses symbolic execution to analyze the code and optimize bit-level operations, generating optimized sequences of instructions that may depend on the target architecture.
*   **Fine-Grained Hardware Dispatch:** It provides a mechanism to directly dispatch operations to specific hardware accelerators by using direct memory mapped register accesses, or by using specialized instruction sequences provided by specific CAMs.
*   **Direct Register Access for AI Ops:** It provides a mechanism to directly access registers related to AI operations, located in hardware components (e.g., DSPs, AI accelerators) using special memory mapped access mechanisms, and through conceptual registers.
*   **Automatic Generation of CAMs from Hardware Specifications:** BML can now leverage AI to automatically generate CAMs based on hardware specifications, instead of manually writing the CAM code. Meta-CAMs provide a more flexible mechanism to generate new CAMs during compile time, allowing the developers to create new CAMs based on high-level descriptions, or by using specific hardware information.
*   **Adaptive DSLs:** BML DSLs can be customized by using AI techniques based on the user's needs or the application context. This allows to customize the DSL based on the specific user needs, or on a given target architecture.
*   **Symbolic Model Checking for Hardware Interaction:** BML will use AI to formally verify that interactions with hardware peripherals (especially memory-mapped registers) adhere to the correct protocol, reducing the risk of hardware malfunctions.
*   **String Interpolation:** Support for f-strings for more readable output: `print(f"The value is {x}");` allows for more readable output.
*   **Concise Bitfield Operations:** More concise syntax to access bitfields and bit ranges using `=` and `|=`, and the `with` statement for combined bitfield updates, and implicit bitfield access, which provides more flexibility and improves code readability.
*   **Array Initialization:** Support for range based initialization provides an easier mechanism to initialize an array.
*   **Optional Parameter Defaults:** Allows default values for optional function parameters, reducing boilerplate code in function calls.
*   **More Flexible struct and union Initialization:** Allows initialization using an initializer list, when the `initializerList` attribute is specified, providing a more flexible and concise way to initialize data structures, such as structs and unions.
*   **Simplified Tuple Declaration:** Allows implicit tuple declaration, which makes tuple initialization and declarations less verbose.
*   **Expressive Pattern Matching:** Allows more complex pattern matching, which will improve the readability of complex code.
*   **Idiomatic Error Propagation:** Support for the `?` operator for error propagation: `value = my_function()?;` provides a concise mechanism for propagating errors.
*   **CAM Composition:** CAMs can now be composed by using the `merge` keyword, which will allow developers to build more complex and reusable components.
*   **CAM Customization:** CAMs can now be customized using the config section and specific attributes. This provides a very flexible way to generate CAMs that can adapt to specific user needs.
*  **More Context-Aware AI Optimizations:** The AI optimization engine will now consider the whole program's context when performing code optimizations.
*  **Automated Code Refactoring:** The compiler will provide an AI-powered refactoring tool to improve code maintainability.
*  **Automated Hardware Verification:** The automated verification tools will provide more information about the formal verification process, giving the user more visibility of the verification process.
*  **AI Generated CAM stubs:** Allows the generation of CAM code by only defining the interface intrinsic function, making it faster to create new CAMs.
*   **Optional Parentheses:** Parentheses are now optional when a function or intrinsic has only one argument, which allows for less verbose code, and more similar to other programming languages.
*   **Low Level Control through CAMs and DSLs:** The language was not intended to be a low-level assembly replacement, but a high-level alternative, that offers better safety, maintainability, productivity, and portability. Low-level code generation is possible using CAMs and their DSLs.
*  **Symbolic Execution**: The compiler can now perform symbolic execution by using the `symbolic` keyword and associated operations, allowing advanced code transformations and formal verification techniques to be used during compilation.
*  **Algorithmic Code Generation**: The compiler will now use specific algorithmic code generators to generate optimized code, when the `@algorithm()` attribute is used, improving code performance in specific scenarios.
* **CAM based Rule Systems**: CAMs can now define code transformation rules by using the `instrDSL` and associated `match`, `capture` and `apply` statements.
*  **Algorithmic DSL Adaptation**: DSLs can now be adapted based on user-specified parameters, by adding a `config` block in the DSL definition and specifying parameters with default values.
*   **Explicit Control and Developer Responsibility:** BML puts a great level of control in the hands of the developer. While providing a high-level abstraction, BML also gives the developer explicit control over low-level aspects, such as memory access and code generation. The programmer is responsible for using the language features correctly to achieve the optimal performance and safety that is needed for their specific application.

**Limitations:**

*   **No Direct Instruction Encoding (User-Level):** BML does *not* allow the developer to directly specify instruction encodings, or manually encode an instruction using opcodes, operands, and addressing modes *at user level*. Instead, BML relies on CAMs and their DSLs to generate optimized instruction sequences. This means the developer gives up some direct control over the code generation process, to have a more high-level and maintainable approach.
*   **Hardware Access Limitations:** BML does not provide direct access to *all* low-level hardware features available in assembly. The developer relies on CAMs for generating the required code to interact with peripherals. If a CAM does not expose a specific hardware feature, it cannot be accessed directly from BML code. The features exposed by BML are limited to the capabilities of the CAMs used in the compilation process.
*   **Runtime Checks are Optional:** While BML tries to minimize the runtime checks, they are still present, and have some performance impact. The runtime checks are optional, and may be disabled by using compiler flags or pragmas. This gives the user more control, by explicitly disabling the runtime checks, but it also means that the user can create an unsafe program that can crash. Tracked pointers and safe arrays will perform runtime checks, which may have a performance impact, if hardware-assisted bounds checking is not enabled, or if the compiler cannot statically prove that the accesses are safe.
*   **Debugging Complexity:** While source-level debugging is supported, debugging low-level issues that involve memory corruption and interactions with hardware components may be more difficult in BML compared to the direct observability provided by assembly-level debugging. However, advanced debugging features are provided by the compiler to assist the user in analyzing the compiler's output, and also the code generated by the CAMs.

**2. Core Language Features**

*   **Static Typing:** BML is a statically-typed language, enforcing type rules at compile time to prevent type-related errors, making it easier to reason about the code, and reducing the number of runtime errors.

*   **Data Types:**
    *   **Integer Types:** `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`.
        *   Signed integers (`int` prefix) represent both positive and negative values. `int` is architecture-dependent and defaults to `int32`.
        *   Unsigned integers (`uint` prefix) represent only non-negative values. `uint` is architecture-dependent and defaults to `uint32`.
        *   The number after the prefix indicates the size in bits (e.g., `int32` is a 32-bit signed integer).
        *   **Default value**: The default value for all integer types is `0`.
        *   Example: `uint32 counter = 10;`, `int16 sensorValue = -20;`, `uint32 myCounter = 10;` (explicit type declaration).
    *   **Floating-Point Types:** `float32`, `float64`, `float`.
        *   `float32` represents single-precision floating-point numbers.
        *   `float64` represents double-precision floating-point numbers.
        *   `float` defaults to `float32`.
        *   **Default value:** The default value for all float types is `0.0f` for `float32` and `0.0d` for `float64`.
        *   Example: `float32 temperature = 25.5f;`, `float64 pi = 3.14159265d;`, `float32 myTemperature = 25.5f;` (explicit type declaration).
    *   **Address Types:** `address`, `codeAddress`, `dataAddress`, `genericAddress`, `bootAddress`.
        *   `address` represents a generic memory address, without any specific region association.
        *   `codeAddress` is used for memory addresses that point to executable code (e.g., function pointers or interrupt handlers).
        *   `dataAddress` is used for memory addresses that point to data (e.g., variables or memory buffers).
        *   `genericAddress` is a general-purpose address that does not imply a specific memory region, and can be used as a generic memory address.
        *   `bootAddress` specifically represents the boot address, the initial address where execution begins after reset.
        *   **Default Value**: The default value for all address types is `null`. The compiler will issue a warning if the null address is dereferenced without a null check, unless it is annotated with a `!` or `+` modifier, which explicitly states that the pointer can not be null.
        *   Example: `dataAddress bufferPtr;`, `codeAddress handlerAddress;`, `bootAddress bootPtr = 0x1000;`
    *   **Boolean:** `boolean` (`true`, `false`).
        *   Represents logical truth values.
        *   **Default Value**: The default value for boolean types is `false`.
        *   Example: `boolean isReady = true;`, `boolean hasError = false;`
    *   **Character Types:** `char8`, `char16`, `char32`, `char`.
        *   `char8` is an 8-bit character type, often used for ASCII characters.
        *   `char16` is a 16-bit character type, often used for UTF-16 encoded characters.
        *   `char32` is a 32-bit character type, often used for UTF-32 encoded characters.
        *   `char` defaults to `char8`.
         *   **Default value**: The default value for all character types is `'\0'`.
           *   **String Literals:** String literals are encoded using UTF-8. Character literals are encoded using the corresponding character type encoding (UTF-8, UTF-16 or UTF-32).
        *   Example: `char8 myChar = 'A';`, `char16 unicodeChar = L'Ω';`, `char32 emoji = U'😀';`, `char aChar = 'B';` (explicit type declaration).
    *   **Raw Byte Type:** `rawbyte` (direct memory access).
        *   Allows direct access to memory as a sequence of bytes.
        *   **Caution:** Type conversions to/from `rawbyte` are explicit using `cast` or type conversion functions (e.g., `toUint32`), avoiding implicit conversion. Accessing a `rawbyte` as any other type is an error. When casting to other types, you must ensure that alignment and size constraints are satisfied. The compiler does not perform any automatic alignment check.
        *   The size of a raw byte is always 8 bits.
        *   **Default value**: The default value is `0`.
        *   Example: `rawbyte registerValue = 0xAF;`, `rawbyte^ buffer;`, `uint32 convertedValue = toUint32(registerValue);`, `uint32 otherConvertedValue = cast(uint32) registerValue;`
    *   **Safe Array Type:** `DataType safe [Size]` (runtime bounds checks).
        *   A fixed-size array where elements of type `DataType` are allocated contiguously in memory.
        *   Bounds are checked at runtime for every access. If an access is out-of-bounds, a runtime exception will occur (implementation-defined behavior, possibly an abort or an error handler invocation).
        *   `Size` must be a compile-time constant or an identifier with a `linearAccessGuaranteed` attribute.
         *   **Default value**: The default value for all elements in the array is the default value of the `DataType`.
        *   Example: `uint16 safe [100] sensorReadings;`, `float32 safe [MAX_COEFFICIENTS] coefficients @linearAccessGuaranteed(25);`
         *    **Concise Array Literals:** Arrays can be initialized by using an initializer list between square brackets: `uint32 safe[5] myArray = [1, 2, 3, 4, 5];` or using range initialization: `uint32 safe[10] myOtherArray = [0..9];`. The compiler will infer the type of the array elements from the array literal or the range expression.
        *   **Array Slice Type:** `arraySlice<DataType>` (dynamic views).
            *   Provides a mutable view over an existing `safeArray` or another `arraySlice`. The view does not copy the data, it represents a reference to a portion of the original array, which means that modifications on the slice will be reflected in the underlying array.
            *   Bounds checks are performed at runtime, using the slice's bounds (not the bounds of the original array). If the slice is out of the bounds of the original safe array or another slice, a runtime exception will occur.
            *  Multiple slices of the same array are allowed.
            *  Slices can overlap.
            *  Slices are fully compatible with capability-based pointers.
             *   **Default Value**: The default value of a slice is undefined. A warning is generated by the compiler if a slice is accessed without being initialized or having a default value.
            *   Example:
                ```baremetal
                uint32 safe[100] mySafeArray;
                arraySlice<uint32> mySlice = mySafeArray.slice(0, 10);
                arraySlice<uint32> anotherSlice = mySlice.slice(2,5);
                mySlice[0] = 1;  // mySafeArray[0] will be 1
                anotherSlice[0] = 2; // mySafeArray[2] will be 2
                ```
            *   **Range-Based For Loops:** `for (identifier : rangeExpression) { ... }`.
                *   Iterates over a range of values.
                *   The `identifier` is a read-only variable within the loop, automatically incremented by one in each iteration.
                *   Example: `for (uint32 index : 0..10) print(index);`
    *   **General Pointer:** `DataType^` (unchecked pointer, use with caution).
        *   Provides direct access to memory using a raw address.
             *  **Use Cases:**
                *   `DataType^` should be used only when specific memory manipulation is required and there are no safer alternatives.
                * When interfacing with hardware that requires explicit memory addresses or when working with existing low-level data structures.
                *  When it is required to circumvent the compiler safety mechanisms, and handle low level memory accesses, such as when accessing hardware directly.
                *   It is recommended to use other pointer qualifiers when possible.
                * When interfacing with legacy code that is already using raw pointers.
        *   **Caution:** There are no automatic runtime bounds checks or type checks. It is not implicitly convertible to other pointer types. The type signature must match. The user should explicitly convert a `DataType^` pointer to `genericAddress`, `dataAddress` or `codeAddress` if needed, using the `cast` operator.
        *   Pointer arithmetic is allowed, but it's unsafe; the developer is responsible for not overstepping boundaries, otherwise undefined behavior will occur.
        *   General pointers can be null; if a null pointer is dereferenced, the behavior is undefined. It is highly recommended to use `!`, `@`, or `+` capability qualifiers whenever a pointer can never be null. The compiler will provide warnings about a potential null dereference if the user attempts to access the pointer without checking for null.
         *   **Default Value:** The default value for general pointers is null.
        *   Example: `uint32^ rawPointer;`, `char8^ buffer;`, `uint32+ ptr = &myValue;`
        *   **Capability-Based Pointer Types:** `DataType +`, `DataType -`, `DataType @regionName`, `DataType !`, `DataType scoped`, `DataType tracked`.
            *   `+`: A pointer that can not be null and it can read/write to the memory location, unless region restrictions apply, the equivalent to `DataType^ nonnull`. The compiler guarantees that a pointer with a `+` type qualifier will never be null.
            *   `-`: A pointer that can be null, and it can read/write to the memory location, unless region restrictions apply, the equivalent to `DataType^`. If this pointer is dereferenced without checking if the pointer is null, the compiler may generate a warning.
            *   `@regionName`: A pointer associated with a specific data region, enabling region-based memory management and access controls enforced at compile-time. Accessing memory outside the specified region will result in a compile-time error. The `regionName` must be defined as a `dataRegion` (or `rodataRegion`, or `stackRegion`, or `peripheralRegion`, or `moduleDataRegion`).
            *   `!`: A pointer that can not be null, and it can only read from the memory location; write operations are not allowed. If a write operation is performed, it will result in a compile-time error. The compiler guarantees that the pointer will never be null, and that no write operation will be performed using this pointer.
            *   `scoped`: A pointer that's valid only within its lexical scope. The compiler prevents the pointer from escaping the scope, ensuring that it can not be dangling. This mechanism is mostly implemented at compile time. The compiler will generate errors if the pointer escapes the lexical scope.
             *   **Default value**: Scoped pointers can not be null.
            *   `tracked`: A pointer that is tracked by the compiler during runtime. Every access to the tracked pointer incurs a runtime check to prevent dangling pointers. The implementation is target-specific but can be done by using metadata to store the scope of the tracked pointer, and checking for that metadata in every access. If the `boundsCheckRefinement` is enabled, the runtime check may be optimized by using refinement types with more specific constraints.
             *    **Default Value**: Tracked pointers are initialized with a default null address value. The runtime checks will catch dangling pointers.
            *   Example: `uint32+ sensorDataPtr = &mySensorData;`, `rawbyte- transmitBuffer;` , `uint32@MyRegion regionPtr;`, `uint32! configPtr = &myConfig;` , `uint8 localPtr = &someArray;` (implicit scope), `uint16 tracked sensorDataPtr = &someValue;`
    *   **Scope-Bound Pointer:** `DataType scoped` (lexical lifetime management).
         *   A pointer that's valid only within its lexical scope. The compiler prevents the pointer from escaping the scope, ensuring that it can not be dangling. This mechanism is mostly implemented at compile time. The compiler will generate errors if the pointer escapes the lexical scope.
        *   The runtime behaviour depends on the compiler flags. For increased runtime safety, the compiler can generate code to trap or invoke a runtime error handler if a scoped pointer is accessed after its scope has finished (e.g. by using guard pages or similar techniques).
        *   `DataType scoped` cannot be null.
         *   The `scoped` keyword is optional for local variables, and can be omitted, if the compiler can infer this, using static analysis.
            *   **Static Analysis Process for Implicit Scope:**
            *   The compiler performs static analysis to determine if a pointer's lifetime is strictly confined to its lexical scope, specifically its function body or a block inside the function.
            *   The compiler analyses the code flow to verify if the pointer is never passed as a parameter to a function or stored inside a struct that outlives the scope of the pointer, and if the pointer is never returned to a caller function. If all the conditions are satisfied, the compiler infers that the pointer is scoped and the `scoped` qualifier can be omitted.
            *   If the compiler detects that the pointer is escaping its lexical scope, a compile time error will be generated.
        * **Scope Rules:**
          *  If a `scoped` pointer is stored inside a struct, the struct will inherit the scope of the pointer, and can only be declared in the same scope or a scope that does not outlive the scope of the scoped pointer. If the struct is used outside the scope of the scoped pointer, a compile-time error will be generated. This rule also applies to unions and tuples.
           *  A `scoped` pointer can not be passed as a parameter to a function or be returned by a function, unless the `scoped` qualifier is also applied to the return value.
        *   Example:
           ```baremetal
              function void myFunction() {
                  uint32 localValue = 10;
                  uint32 localPtr = &localValue; // valid pointer (scoped is implicitly added).
                 // ...
              } // localPtr is no longer valid
           ```
    *   **Runtime-Tracked Pointer:** `DataType tracked` (runtime checks).
        *   A pointer that is tracked by the compiler during runtime. Every access to the tracked pointer incurs a runtime check to prevent dangling pointers. The implementation is target-specific but can be done by using metadata to store the scope of the tracked pointer, and checking for that metadata in every access. If the `boundsCheckRefinement` is enabled, the runtime check may be optimized by using refinement types with more specific constraints.
         *   **Runtime Check Implementation Details:**
            *   **Metadata Approach:** A common implementation involves storing metadata alongside the tracked pointer. This metadata can include information about the allocation's starting address, the size of the allocated region, and a validity flag. Each time the tracked pointer is accessed:
                1.  The validity flag is checked. If the flag indicates the pointer is invalid (e.g., it has been deallocated), a runtime error is triggered.
                2.  The access address is compared to the allocation's start address and size. If the access is out of bounds of the allocation region, a runtime error is triggered.
                3.  The metadata can be updated when the allocation or deallocation occurs.
            *   **Guard Pages:** Some architectures may utilize guard pages (protected memory regions that surround allocated memory) for more efficient checks. Accessing memory outside the allocation will trigger a hardware exception. The operating system or runtime library can trap the exception, detect a dangling pointer access, and trigger a runtime error.
            *   **Compiler-Specific Techniques:** The compiler can also use other techniques, such as generating code that performs dynamic range checks on access, but the performance may be slower compared with metadata or guard pages techniques.
           * The metadata associated with `tracked` pointers is stored in a separate memory region and is managed by the runtime environment.
           * A common strategy is to store the information of a tracked pointer in a hashmap, associated with the tracked address. The hashmap can be implemented using software or a hardware accelerator if available, to improve its performance.
            * Metadata includes information about the pointer's allocation scope and validity flags.
            * Every access to a tracked pointer will incur an overhead, as the runtime must perform the validity checks.
            * When a tracked pointer is used, the runtime performs the following checks:
                 * Check if the pointer is valid (not null and allocated).
                * Check if the memory access is within the boundaries of the valid region.
             * These checks may be optimized by using specific hardware support, but there is always some overhead when using tracked pointers.
            * When `boundsCheckRefinement` is enabled, the compiler can use refinement types to specify extra constraints that limit the range of a tracked pointer, and, if the compiler can statically prove the pointer access is valid, the runtime checks can be skipped.
                *   This means that the overhead for a tracked pointer can be minimal if the access can be statically determined by using refinement type constraints.
                *   Example:
                    ```baremetal
                    type validPointer = uint32 tracked where value > 0x1000;

                    function void processData(validPointer ptr) {
                      // if the compiler can prove that the access to *ptr will be within the valid range, no runtime checks will be needed.
                      // if the access might be out of the range, a runtime check must be performed.
                        print(*ptr);
                  }
                ```
        *   Example: `uint16 tracked sensorDataPtr;`
    *   **Tuple Types:** `tuple<DataType1, DataType2, ...>`.
        *   A fixed-size, immutable collection of values, which can be accessed by name or by index.
        *   Tuple values are mutable by adding the `~` modifier at declaration time.
         *   **Default value**: The default value for a tuple is the tuple of the default values of its members, with no padding between the elements of the tuple. The layout of a tuple is similar to that of a struct, with no padding.
        *   Destructuring is supported using the `=` operator or the `from` keyword, which allows you to easily access the tuple's values after declaring a variable of the same tuple type.
         *  Tuples can be initialized using a tuple literal, with implicit mapping by position, where the values in the tuple literal are assigned to the members, based on their position in the tuple type.
        *   Example:
            ```baremetal
             tuple<uint32, boolean> sensorResult = (10, true);
             (value, status) = sensorResult; // implicit type inference
            ~tuple<uint32, boolean> (anotherValue, anotherStatus) = (10, true); // explicit mutable tuple declaration
             tuple<uint32, boolean> (anotherValue, anotherStatus) from sensorResult; //explicit type
            myImplicitTuple = (10, true); //implicit tuple declaration
            myOtherTuple = (value1, value2); //Implicit type is tuple <dataType of value1, dataType of value2>

            ```
    *   **Function Pointer Type:** `function(...) -> ...`.
        *   A pointer to a function with a specified signature.
        *   Function pointers are unsafe, similarly to raw pointers, meaning that the compiler does not track if the function pointer is a valid code address.
        *   Function pointers are not implicitly convertible to other function pointer types. The type signature must match.
        *   **Default value**: The default value of a function pointer is `null`.
        * Example: `function(uint32, uint32) -> uint32 MathFunctionType;`
    *   **Struct and Union Types:** `struct` and `union` for composite data structures.
        *   `struct` creates a composite data type where members are laid out in a sequence in memory. The members can be of different types.
           * If a struct contains only one member, curly braces can be removed: `struct MyStruct uint32 value;`
        *   `union` creates a type where different members share the same memory location. Only one member can be active at a given time, so care must be taken to not corrupt data when accessing members of a union.
          * **Default value:** The default value for a struct is the struct where all members are set to their default values. The default value of a union is the union where the first member is active and is set to its default value. The members are initialized in the order they are declared, with no padding between them. The members are initialized using the default value of their type.
           * If the `initializerList` is enabled with the  `@initializerList` attribute, structs and unions can be initialized using implicit initialization syntax.
         *  Structs and unions can be initialized by using implicit mapping, using the order of the members, or by specifying the name of each member in the initializer list.
          *  If a member is not present in the initializer list, it will be initialized with its default value.
        *   Example:
        ```baremetal
                type Point = struct @initializerList { int32 x; int32 y; };
                 Point myPoint = { 10, 20 };  // x=10, y=20 (positional)

                 type MyStruct = struct @initializerList { uint32 a; boolean b; uint8 c = 10;};
                  MyStruct myValue = { 1, true }; // a=1, b=true, c=10, using positional initialization.

                type Config = struct @initializerList { uint32 baud; boolean enable;  uint8 dataBits = 8;};
                 Config myConfig = { enable=true, baud=115200 }; // baud=115200, enable=true, dataBits=8, using names for implicit mapping.

                  type Config = struct @initializerList { uint32 baud; boolean enable = true;  uint8 dataBits;};
                 Config myOtherConfig = { 115200, dataBits = 7 }; // baud=115200, enable=true, dataBits=7, mix of positional and name mapping.

               ```
        *   Example: `struct Point { int32 x; int32 y; };`, `union Packet { uint32 rawData; struct { uint8 highByte; uint8 lowByte; } bytes; };`
        *   **Memory Mapped Structs:** `struct @mapped` (members are volatile by default, unless specified `nonvolatile`).
             *   `@mapped` structs defines a memory-mapped region, with an implicit volatility status based on the struct's qualifier, unless specified otherwise on a member-by-member basis, using `volatile` or `nonvolatile`.
            *   The members are implicitly treated as `volatile`, unless the `nonvolatile` qualifier is used. The compiler will not attempt to optimize accesses to `volatile` variables.
             *   If the structure is `volatile`, members are `volatile`, unless declared `nonvolatile`.
            *   If the structure is `nonvolatile`, members are `nonvolatile`, unless declared `volatile`.
            *   If there is no attribute on the struct, members are volatile, unless declared `nonvolatile`.
            *   When using memory-mapped structures, the endianness of the members must be carefully considered; if the architecture is little-endian, a `uint32` member will be accessed byte-by-byte in little-endian order.
            *   Data hazards (multiple reads/writes) are not automatically handled by the compiler. The developer must use memory barriers or other means to manage data hazards in shared memory-mapped regions.
            *   Example: `type GpioRegisters = struct @mapped(0x40000000) { uint32 dataRegister; nonvolatile uint32 configRegister; };`
             * When the `initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct @mapped(0x40000000) @initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`
             *  Memory-mapped structure access should be performed via an instance of the struct that is mapped to a given base address, for example:
                ```baremetal
                type GpioRegisters = struct @mapped { uint32 dataRegister;  uint32 controlRegister; };
                 function void initGpio(dataAddress base) {
                    GpioRegisters myGpio = base; //maps the memory region
                     myGpio.controlRegister = 1;
                }
                ```
             * **Bitfield members can be accessed with the `=` operator and initialized by using an initializer list, without the `with` statement: `myGpioRegisters.controlRegister = { enableBit = 1, dataBit = someValue };`.**
            * **Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;`.**
              *  Bitfield updates should be performed using a `with` statement, to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation.
          *    **"with" statement optimization:** The compiler analyses the `with` block, to check if a specific write or read operation to a memory
                 *   This mechanism is automatically performed by the compiler and requires no additional effort from the user, enabling zero-overhead high-performance bitfield access.
             *   Example:
                ```baremetal
                type GpioRegisters = struct @mapped { uint32 controlRegister; };
                GpioRegisters myGpioRegisters = 0x40000000;
                myGpioRegisters.controlRegister = {
                    enableBit = 1,
                    dataBit = someValue
                };
                 myGpioRegisters.controlRegister[0..3] = 0b101;
                  with (myGpioRegisters) {
                         enableBit := 1;
                          dataBit := someValue;
                     }
            ```
         *  **Memory Mapped Registers**: Specific registers can be defined using the following syntax:
        *   `DataType @register("regionName", "stringLiteral")  registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type, and can not be a struct or a union. This attribute can only be used with `register` declarations.
        *   The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch, a compile-time error will be generated.
        *   The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location.
            *  Example: `uint32 @register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants, improving the readability of the code. If the enum is declared with a simple comma separated list of values, curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
         *   **Default Value:** The default value for an enum is the first member of the enum.
        *   Example: `enum ErrorCode Ok, NotFound, AccessDenied;`, `enum ErrorCode { Ok, NotFound, AccessDenied };`
    *   **Thread Handle Type:** `threadHandle`.
        *   A handle to a concurrent task that allows the developer to interact with the task.
         *   **Default value:** The default value for a `threadHandle` is `null`.
        *   Example: `threadHandle myThread;`
    *   **Mutex Type:** `mutex`.
        *   A synchronization primitive used to implement critical sections, protecting shared memory from data races, guaranteeing atomic access to shared resources by different threads.
        *    **Default value**: The default value of a mutex is an unlocked mutex.
        *   Example: `mutex myMutex;`
    *   **Semaphore Type:** `sync`.
        *   A synchronization primitive that provides signaling with a counting capability, useful for limiting the number of concurrent accesses to a specific resource.
         *   **Default value**: The default value of a `sync` is a semaphore with its counter set to `0`.
        *   Example: `sync mySemaphore(10);`
    *   **Awaitable Type:** `DataType await`.
        *   A type used to represent an asynchronous computation that produces a value of type `DataType`. Can be used in conjunction with the `await` and `yield` keywords.
         *  **Default value**: The default value of an awaitable is implementation specific.
        *   Example: `uint32 await asyncResult;`
    *   **Generic Modules and Functions:** `generic <DataType>`.
        *   Allows code to be parameterized with types, which allows creating reusable and type-safe components.
        *   Example: `generic <DataType> module MyModule { ... };`, `generic <DataType> function DataType myFunc (DataType value) -> DataType { ... };`
    *   **Result Type:** `Result<OkType, ErrType>` for explicit error handling.
        *   A type that represents either a successful result (`OkType`) or an error (`ErrType`), improving the clarity of the error handling code and making the compiler enforce error checking.
        *   Pattern matching or a method to easily get the data value from the `Result` type is implementation-specific.
        *   The user must propagate the error up the call stack and implement a proper error handling policy. The default value is a result where the Ok value is the default value of `OkType`, and the Err value is the default value of `ErrType`.
          *   **Error Handling and Propagation:**
            * The `Result` type is intended to enforce proper error handling at compile time. The user must explicitly handle the results, by checking if there is a valid `Ok` value or an `Err` value. The user can propagate errors using different techniques, such as using a chain of functions that explicitly propagate the errors using the `Result` type.
            * The `Result` type is designed to improve the code clarity and to provide static checks to avoid runtime errors. The user must use CAMs or user-defined types to implement specific error reporting and handling strategies.
            *   **Accessing `Ok` and `Err` Values:** Accessing data from a result type is implementation-specific. The compiler may use an `isOk()` or `isErr()` method for checking the state of the result and a `getOk()` or `getErr()` function to extract the values from a given `Result` variable. For example:
              ```baremetal
               function Result<uint32, ErrorCode> performOperation() {
                 // ...
                  if (/* condition */) {
                    return Ok(10);
                  } else {
                     return Err(ErrorCode.InvalidInput);
                  }
               }
                function void main() {
                    Result<uint32, ErrorCode> myResult = performOperation();
                    if (myResult.isOk()) {
                       uint32 value = myResult.getOk();
                      print("Operation success, data:", value);
                    } else {
                      ErrorCode code = myResult.getErr();
                      print("Operation error, code:", code);
                    }
                }
               ```
           *  **Error Handling Policy:** BML does not enforce a specific error handling policy, and it is the user's responsibility to handle errors correctly. If an error is not properly handled or ignored by the user, the behaviour is implementation specific.
           * **Example (Error handling):**
              ```baremetal
                 enum ErrorCode { Ok, InvalidInput, ResourceUnavailable };
                 function Result<uint32, ErrorCode> readSensor() {
                    if (/*Sensor is not ready*/) {
                      return Err(ResourceUnavailable);
                   } else {
                     return Ok(100); //read from sensor
                   }
                }
                function Result<uint32, ErrorCode> processSensor() {
                  Result<uint32, ErrorCode> result = readSensor();
                   if (result.isErr()) {
                      return Err(result.getErr()); // propagate the error
                   }
                   uint32 value = result.getOk();
                  //do something with value, and return a result
                 }
                  function void main () {
                     Result<uint32, ErrorCode> result = processSensor();
                   if(result.isOk()) {
                    print("Data:", result.getOk());
                     print("Error:", result.getErr()); //handle the error here
                 }
              }
              ```
        *   Example: `Result<uint32, ErrorCode> myResult;`
    *   **Refinement Types:** `type identifier = baseType where constraint` (type constraints).
        *   Allows to define custom data types with compile-time constraints, adding extra static type checks to the language.
        *   The constraint can be an arbitrary expression, a range, or a length constraint.
          *   **Default Value**: The default value of a refinement type is the default value of the underlying base type.
        *   **Examples:**
            *   **Bounds Checks:**

                ```baremetal
                type ValidSensorReading = uint16 where value in range(0..1000);
                function void processSensorData(ValidSensorReading reading) {
                    // Reading is guaranteed to be within 0 and 1000
                    print("Sensor reading:", reading);
                }
                 function void main() {
                     ValidSensorReading myValue = 500; // ok
                      processSensorData(myValue); //ok
                   // ValidSensorReading invalidValue = 1001; // compile-time error, not a valid value for ValidSensorReading
                  }
                ```

            *   **Specific Value Ranges:**

                ```baremetal
                type ClockDivider = uint8 where value in range(1..10);
                function void setClockDivider(ClockDivider divider) {
                   // ... configure the clock divider, it's guaranteed to be in range 1..10
                }
                 function void main() {
                   ClockDivider myDivider = 5; // valid
                  setClockDivider(myDivider);
                    //ClockDivider invalidDivider = 0; // compile-time error, not in valid range
                }
                ```
            * **More Detailed Bounds Checks:**
              ```baremetal
                  type SmallBuffer = uint8^ where length < 100;

                  function void processBuffer(SmallBuffer buffer) {
                    // code that uses buffer, assuming that the length is less than 100
                   // ...
                      print(sizeof buffer); // will always be less than 100
                  }
                  function void main() {
                    uint8 safe[50] mySafeBuffer;
                   SmallBuffer myBuffer = &mySafeBuffer[0]; //ok
                   processBuffer(myBuffer);
                   uint8 safe[120] otherSafeBuffer; //this will generate a compilation error because otherSafeBuffer has a size bigger than 100
                    // SmallBuffer anotherBuffer = &otherSafeBuffer[0]; //compile-time error: length is not correct
                  }
                 ```
            *  **Type Enforcement:**
                ```baremetal
                type EvenNumber = uint32 where value % 2 == 0;

                function void processEvenNumber(EvenNumber number) {
                     // This function is guaranteed to only get even numbers
                   print("Number is even:", number);
                }
                function void main () {
                    EvenNumber myNumber = 4; // valid

processEvenNumber(myNumber);
                     //EvenNumber invalidNumber = 5; // compile-time error, invalid value
                }
               ```
           * **Refinement types are fully compatible with other language features** such as type attributes, scoped pointers, and so on, as they are just types with additional constraints:

                ```baremetal
                type validPointer = uint32 tracked where value > 0x1000;

                function void processData(validPointer ptr) {
                      // if the compiler can prove that the access to *ptr will be within the valid range, no runtime checks will be needed.
                      // if the access might be out of the range, a runtime check must be performed.
                        print(*ptr);
                  }
                ```
        *   Example: `type ValidSensorReading = uint16 where value in range(0..1000);`, `type SmallBuffer = uint8^ where length < 100;`, `type EvenNumber = uint32 where value % 2 == 0;`
    *   **Region-Based Types:** `type identifier = baseType @region(regionName)`
        *   Allows to define types that are associated with a specific data region, enabling region-based memory management and access controls enforced at compile time.
         *   **Default value**: The default value of a region-based type is the default value of the underlying base type.
       *  **Memory Access and Region Interaction:**
         *  Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined.
          *    `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions, a CAM will have to explicitly read from one region and write to another region, using memory copy functions, or by transferring using memory mapped peripherals.
          * CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
        *   Example: `type SecureData = uint32 @region("secureRegion");`
    *    **Capability-Based Regions:** `dataRegion regionName capabilities(read, write) { ... };`
        *   Data regions are used for compile-time region-based memory access controls and memory management, allowing compile-time checking of memory accesses and capabilities, increasing memory safety. If only one member is present, curly braces can be removed. `dataRegion MySecureData uint32 secretKey;`
        *   The `capabilities` argument specifies which permissions are available: `read`, `write`, `execute`, `capability_derive`.
          *   If only `read` or `read, write` capabilities are needed, the `capabilities` keyword can be omitted: `dataRegion @read MySecureData uint32 secretKey;`, `dataRegion DeviceData { uint8 safe[100] buffer; };` (implicit read, write capabilities).
        *   `capability_derive` allows to create new pointers with limited capabilities from a pointer that has the `capability_derive` permission, granting a way to control how a pointer is used within a specific scope or region.
        *   A memory region must be declared using this syntax to be used in `@region(regionName)` pointers, otherwise, the compiler will generate a compile-time error.
        *   Example: `dataRegion @read secureData { uint32 myData; };`, `dataRegion DeviceData { uint8 safe[100] buffer; };`
    *   **Compile-Time Stack-Only Allocation:** `@stackOnly` for stack allocation.
        *   Forces the variable, struct, or array to be allocated in the stack, which avoids the overhead of dynamic memory allocation and provides a predictable memory allocation, which is useful for real-time systems.
         *   **Default value**: Stack only types have the default value of the underlying base type.
        *   Example: `@stackOnly uint32 myStackValue;` , `stackOnly uint8 safe[10] localBuffer;`
    *   **Hardware-Assisted Bounds Checking:** Optional using a compiler flag or pragma if the hardware supports it.
        *   Requires specific hardware support, like memory management units, for actual hardware-based checking of memory accesses, improving security and performance.
        *   If hardware support is not available, a runtime software-based implementation may be used by the compiler.
        *   Example: `#pragma enable_hardware_bounds_checking; uint32 safe[10] myArray;`
    *   **Data-Layout Attributes:** `@packed`, `@aligned(4)`, `@row_major`, `@array_of_structures`.
        *   `packed`: Removes padding from structs, resulting in a smaller memory footprint, at the cost of potential alignment problems, which can affect the performance of certain memory accesses, or cause errors on specific architectures. The compiler does not perform any alignment checks.
        *   `aligned(N)`: Aligns the data at a memory address that's a multiple of N (where N is a power of 2), useful to improve memory access performance, especially when dealing with SIMD instructions.
        *   `row_major`: Arranges multi-dimensional arrays in row-major order (C/C++ standard), which is important when interfacing with code or data from other languages and libraries. This attribute is useful when performing memory accesses in row-major order. For example when accessing a matrix sequentially, accessing the elements in this order will result in better cache usage. For a 2-dimensional array: `data[row][col]`, when traversing the array, the row will be incremented before the column. If this attribute is not used, the compiler may choose other layout strategies, depending on the architecture.
          *  Example: `uint32 safe[10] safe[10] myMatrix @row_major;`
        *   `array_of_structures`: forces the array to have the following structure: `[ { member_1, member_2, ..., member_n}, {member_1, member_2, ..., member_n}, ..., ]`. This forces the data to be laid out in the most cache-friendly way for algorithms where all members of a given struct element are accessed sequentially. For example, when you have an array of structs, if you access all members of a given struct sequentially, this is the most cache-friendly layout. If this attribute is not used, the compiler may choose other layout strategies depending on the architecture.
           *  Example:
             ```baremetal
             type MyStruct = struct {
                  uint32 a;
                  float32 b;
               };
              MyStruct safe[10] myData @array_of_structures;
            ```
        *   Example: `struct MyStruct @packed { uint8 a; uint32 b; };`, `uint32 safe[100] myArray @row_major;`
    *   **Compile-Time Reflection:** `sizeof` and `isType` intrinsics.
        *   `sizeof DataType`: Returns the size (in bytes) of the specified type at compile time, which allows CAMs to generate optimized code based on the data structure sizes.
         * `sizeof (DataType)` is also a valid syntax when parentheses are used.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement };`.
        *   Example: `sizeof uint32`, `isType(DataType, "Integer")`, `sizeof(uint32)`,
    *   **Compile-Time Functions:** Code execution during compilation via `compileTimeFunction`.
        *   `compileTimeFunction` functions have specific limitations: they cannot perform dynamic memory allocation, and cannot call non `compileTimeFunction` functions, because these restrictions are needed to execute code during compilation,
        *   They can be used for static analysis of the data structures or code, or generate code based on compile time parameters.
        *    Compile-time variables that are declared in a `compileTimeFunction` must be initialized with a compile-time constant value. It is an error to use run-time variables in `compileTimeFunction` functions.
           *   **Debugging Compile-Time Functions:**
                *   Debugging compile-time functions directly is not supported by traditional debuggers (GDB/LLDB) because they execute during the compiler's runtime, not the target's runtime.
                *   The compiler may offer options to inspect compile-time variables and function execution during compilation by using debug flags or a compiler API. The user can use the `print` function, which can be used to print debugging information during compilation.
                *   The compiler can also provide information about how the code was transformed by a compile-time function using debug flags or an output file format that shows which transformations occurred in each compilation step.
                *   The user can also inspect the generated intermediate representation of the code using compiler flags.
                *  `compileTimeFunction` can call other `compileTimeFunction` functions, as long as no cyclic dependency is generated. The dependency will be checked by the compiler during the compile-time phase.
        *   Example:
            ```baremetal
            compileTimeFunction void myFunc() {
               //Code to be executed during compilation
             print("Hello from compile time function");
            }
            ```
    *  **Data-Driven Code Generation:** CAMs use `hasDataLayoutAttribute()`.
        *  `hasDataLayoutAttribute(DataType, "packed")` returns `true` if the data type has the "packed" layout attribute. This is used by CAMs to make decisions about code generation based on the data structure layout. Possible layout attributes are defined in `enum LayoutAttributeKind { Packed, Aligned, RowMajor, ArrayOfStructures };`.
        *   Example:
            ```baremetal
            module MyCam {
                @arc arm {
                   function void optimizeData(DataType value) {
                     if (hasDataLayoutAttribute(value, "packed")) {
                        //generate code for packed struct
                     } else {
                     // generate standard code
                     }
                }
            }
            }
            ```
    *   **Linear Access Attributes:** `@linearAccess`, `@linearAccessGuaranteed(size)`.
        *   `@linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations.
        *   `@linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray @linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray @linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray @linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `@circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer.
        *  The compiler must verify that the size of the buffer is a power of 2, otherwise, it is a compile-time error.
        * Example: `uint32 safe [1024] myBuffer @circularBuffer;`
    *   **Read and Write Once Attributes:** `@readOnce`, `@writeOnce`
        *   The `@readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that.
         *  If the variable or memory region is read more than once, the behaviour is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `readOnce` attribute is violated.
        *   The `@writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that.
          *   If the variable or memory region is written more than once, the behaviour is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `writeOnce` attribute is violated.
        *   Example: `uint32 counter @readOnce;`, `uint8 safe [10] buffer @writeOnce;`
    *  **Cache Line Aligned Attribute:** `@cacheLineAligned`
         *  Guarantees that the data will be aligned to the size of the processor cache line, which enables better cache utilization and access performance, preventing issues related to false sharing.
         *  The size of the cache line depends on the target architecture.
         * Example: `struct MyStruct @cacheLineAligned { ... };`
    *   **Register and Schedule Attributes:** `@register("stringLiteral", "stringLiteral")`, `@schedule("stringLiteral")`
        *   These attributes provide hints to the CAM about how to assign registers and schedule the execution of a given function. The `stringLiteral` is CAM specific and should be used by the CAM developer to provide additional information to the code generator. For example:
            *  `register("r0")`: The `r0` string can be used by the ARM CAM to associate this variable to the `r0` register.
            * `register("offset0x20")`: The `offset0x20` string can be used by a CAM to access an offset of 0x20 from the address where the variable is located.
            *`schedule("low")`: The `low` string can be used to mark that a function should be scheduled with low priority, when possible.
         *   Example: `function processData() @register("r1") @schedule("low") { ... };`,  `uint32 @register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **No Overlap Guaranteed Attribute:** `@noOverlapGuaranteed`
        *  Guarantees that a given memory region does not overlap with any other memory region, allowing for potential optimizations, if the compiler can enforce this at compile time.
         * It's the programmer's responsibility to ensure that this is true. If two memory regions marked with `@noOverlapGuaranteed` do overlap, the behaviour is undefined. The compiler does not provide a mechanism to verify this property at compile time.
         * Example: `uint8 safe[100] array1 @noOverlapGuaranteed; uint8 safe[100] array2 @noOverlapGuaranteed;`
    *   **Vectorization Attributes:** `@vector(N)`, `@vectorAligned`.
        *   The `@vector(N)` attribute enables vectorization of a specific data access, and the compiler is in charge of generating vectorized instructions. `N` is the vector size in bytes.
        *    If `N` does not match the expected vector size of the target architecture, the behaviour is undefined.
        *   The `@vectorAligned` attribute guarantees that the data is aligned with the vector size, which is useful for SIMD operations.
         *  The vector size depends on the target architecture.
        *   Example: `uint32 safe[100] myVector @vector(16);`, `uint8 safe[1024] bigBuffer @vectorAligned;`
    *   **Function Inlining Attributes:** `@inlineAlways`, `@noinline`, `@inlineIfSmall`.
        *   `@inlineAlways`: Forces the inlining of the function. The compiler will emit an error if the function can not be inlined.
        *   `@noinline`: Prevents inlining of the function, even if optimizations suggest it's beneficial.
        *   `@inlineIfSmall`: Suggests to the compiler to inline the function only if it's considered small.
        *   Example: `function mySmallFunction() @inlineIfSmall { ... };`, `function myBigFunction() @noinline { ... };`, `function myFunction() @inlineAlways { ... };`
    *   **Dataflow Attributes:** `@in`, `@out`, `@inOut`.
        *   These attributes are used in CAMs to track how data is passed through the system, enabling optimizations and compiler checks based on the data flow.
         *    These attributes do not change the code behaviour; they are used by CAMs for code generation and analysis.
            *  **Dataflow Example:**
               *   CAMs use the `@dataFlow` attributes to analyze how data is passed through the system and use this information to perform optimizations. For example, an optimization can be to perform computations in a different order to minimize cache misses, or to perform memory access in a different order to improve performance.
                   *  Example:
                     ```baremetal
                    module DataFlowCam {
                        export {
                            interface intrinsic function void processData(uint32 input @in, uint32 output @out) -> void;
                        }
                        @arc arm {
                           function void processData(uint32 input @in, uint32 output @out) {
                                // code to analyze the dataflow graph
                                 print("DataFlow: Analyzing the flow of data");
                               memoryBarrier();  // memory barrier can be moved by the optimizer if it's guaranteed that the read/write operation will always be performed in the right order.
                              // ... generate the optimized code
                            }
                      }
                    }
                     module MyModule {
                      #use "DataFlowCam";
                        function void main() {
                            uint32 data = 10;
                            uint32 result;
                            processData(data, result);
                           print("Processed data:", result);

                         }
                    }
                     ```

        *   Example: `function void processData(uint32 input @in, uint32 output @out) { ... };`
    *   **Transformation Attribute:** `@transform(stringLiteral)`.
        *   This attribute allows to transform the data using target-specific transformations. The `stringLiteral` is CAM specific.
         * This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) @transform("myTransform") { ... };`
    *   **Allocator and Unit Attributes**: `@allocator(stringLiteral)`, `@unit(stringLiteral)`
        *   `@allocator(stringLiteral)` is used to specify the allocator to be used for a specific memory allocation. The `stringLiteral` is CAM specific.
          *    This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   `@unit(stringLiteral)` is used to mark a specific part of a CAM with a specific unit or component, which is used to group related functionalities and perform analysis on a unit basis. The `stringLiteral` is CAM specific.
           *    This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `type MyType = struct @allocator("myAllocator") { ... };`, `module MyCam @unit("myUnit") { ... };`
    *  **Lifetime Attribute:** `@lifetime(stringLiteral)`
        *  The `@lifetime(stringLiteral)` attribute provides a way to assign a lifetime to a pointer, which can be tracked by the compiler to prevent dangling pointers. The `stringLiteral` is CAM specific.
        *    This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        * Example: `uint32^ data @lifetime("myLifetime");`
    * **Capability Attribute:** `@capability(capabilityList)`
        *   Used to specify the capabilities of a given data type. The `capabilityList` is a list of capabilities (read, write, execute, capability_derive) that the data type has.
          *    This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example `struct Data @capability(read, write, execute) { ... };`
    *   **Limit Attribute:** `@limit(stringLiteral)`
        *   Provides a way to limit a specific resource, useful for CAM implementations. The `stringLiteral` is CAM specific.
         *    This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `module MyCam { ... config { config maxThreads: uint32 @limit("threadLimit"); } }`
      *  **Register Attribute:** `@register("regionName", "stringLiteral")`.
           * This attribute is used to specify a mapping to a memory mapped register. The stringLiteral is CAM specific and is intended to provide additional information for code generation. This attribute can only be used with `register` declarations.
          * This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
          *  Example: `uint32 @register("MyAiCore", "control_reg") myMatrixAControl;`
     * **Hardware Description Attribute:** `@hardwareDescription(stringLiteral)`.
         * This attribute allows to associate a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
          * This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
          * Example:  `interface intrinsic function void platformInit(uint32 baseAddress) -> void @hardwareDescription("timer.init");`
      *  **Enable and Disable Attributes:** `@enable(stringLiteral)`, `@disable(stringLiteral)`.
         * These attributes provide a way to enable or disable specific functionalities of a given CAM or a function, by using a string parameter, that maps to a given boolean configuration variable.
     * **Algorithmic code generation attribute:** `@algorithm(stringLiteral)`.
         *  This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator, instead of relying on traditional code generation techniques.
     *  **Symbolic Execution:** `symbolic`, and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        *   The `symbolic` keyword is used to create a symbolic block, where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression, for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check, at compile time.
    *   **Implicit Type Conversions:**
        *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64`, but not to `int8`, `int16`, `int32`, `int64` or `float32`, and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed, and requires an explicit cast.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion, and when returning from functions, if the return type is compatible for implicit conversion.
        *   No other implicit type conversions are allowed. Conversions must be explicit, by using the `cast` operator, or by calling conversion functions from the standard library, or provided by the CAMs.
    *  **Implicit Type Inference:** Type inference is used in the following cases:
        * When the return type of a function is not explicitly specified, the compiler will infer the return type by analysing the `return` statements, or by using `void` as default if there is no return statement.
         * When declaring an array literal, the compiler infers the array type from the literal values.
         * When a variable is assigned an integer literal, and the type can be inferred from the context, if not, an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal, the default type is `float32`.
         * **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
         *  When a local variable is initialized with an expression inside the function scope.
    *   **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.

**3. Modules and CAMs: Structure and Organization**

*   **Modules:** Self-contained code units that encapsulate data and functionality, promoting code reusability and organization.
    *   `module moduleName { ... }`: Declares a module, creating a namespace.
        *   `description "String";`: (Optional) Provides a module description that is used for documentation purposes.
            *Example: `description "A simple example module";`
        *   `declarations { ... }`: Module-private and exported declarations. Module declarations are private by default, and must be declared with `export` keyword (or listed in the export block), to be accessible outside the module.
            *   Example: `const export uint32 moduleConstant = 123;`, `uint32 modulePrivateVariable = 456; function export void myModuleFunction() { ... };`
         *  If a function or a constant is listed in the `export { }` block, it is implicitly considered as exported, removing the `export` keyword from the declaration.
        *   `statements { ... }`: Module initialization code (optional). This code is executed when the module is first loaded, and before the execution of the entry point. Module initialization code can also include `use module` statements.
            *   Example: `statements { print("Module initialized"); #use "MyConfigModule"; }`
        *  `export { ... }`: Defines exported items by declaring them inline (implicitly or explicitly), using the `export` keyword, or by listing them inside the block: `functions`, `types`, `constants`, `conceptualRegisters`. This is used to make specific module components visible outside of the module.
            *   Example:
                ```baremetal
                module MyModule {
                    export {
                        functions { myExportedFunction }
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     const MAX_VALUE = 100; // implicitly exported
                    function  myExportedFunction() { ... }; //implicitly exported
                    type export MyExportedType = struct { uint32 a; };
                    const export uint32 myExportedConstant = 10; //explicitly exported
                }
                ```

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope, making the exported components of the module visible in the current scope.
         *  Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200; };`
            *   The `with config` clause is used to customize the CAMs behavior.

*   **Composable Architecture Modules (CAMs):** Compiler extensions that act as "optimization engines," enabling architecture-specific code generation and customization. CAMs are a core feature of BML, and the main mechanism to implement target-specific code and optimizations.

**4. Composable Architecture Modules (CAMs)**

*   CAMs are compiler extensions that act as "optimization engines". They allow for extending the compiler, and generating platform-specific code in a type-safe and modular fashion.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM.
        *   `description "CAM Description String";`: (Optional) A description of the CAM, which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can now load external data from a file, by specifying the location in the `config` section, and using a parsing function.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral;}`
        *   `export { ...  }`: Declares architecture-agnostic interface functions via `interface intrinsic function`, which is the main entry point for CAM-provided operations, making them callable from other BML code. It can also export conceptual registers that are used by the CAM code to generate target-specific instructions, improving the abstraction of the hardware. The `instrDSL` block is used to declare a domain specific language for defining low level symbolic instructions. Symbolic instructions can have generic parameters, which can be used to specify the instruction behavior at compile time. Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time.
            *   Example:
                ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean @enable("dma") = false;
                   }
                    export {
                       interface intrinsic function void platformUartInit(uint32 baudRate) -> void;
                       interface intrinsic function void platformUartSendByte(uint8 data) -> void;
                       interface intrinsic function void platformUartSendString(StringLiteral str) -> void;
                       conceptualRegisters {
                          %uartBaseAddress,
                          %baudRateValue,
                          %byteToSend
                       }
                   }
                   @arc arm {
                     @instrDSL {
                         instr void load_reg<uint32 offset>(dataAddress addr, register reg) {
                           // Symbolic instruction for register load, with a generic parameter.
                          // if offset is 0, generate a normal load
                           // if offset is greater than zero, load from an offset
                            }
                            instr void store_reg(dataAddress addr, register reg) {
                             // Symbolic instruction for register store.
                            }
                           instr void custom_dma(register dest, register src) {
                            // Symbolic instruction using DMA
                           }
                          instr void add_reg<enum Operation>(register reg1, register reg2) {
                             // Symbolic instructions using an enum as a type parameter.
                            }
                        enum Operation {
                          ADD,
                          SUB,
                          MUL
                          }
                          rule add_imm match(add(x, immediate(y)), instruction) -> replace(instruction, add_reg(x,y));
                        }
                      function void parseHardwareDescription () {
                       print(f"Parsing {config.hardwareDescription}");
                       // load the data using the config.hardwareDescription and generate new interface intrinsic functions
                      }
                        function void platformUartInit(uint32 baudRate) {
                          parseHardwareDescription();
                          register %uartBaseAddress = config.baseAddress + 0x0; //Example UART base address
                            register %baudRateValue = baudRate;
                            memoryStore(dword, %uartBaseAddress + 0x04, %baudRateValue); // Example code to set baud rate
                            print("ARM: Uart Initialized");

                        }
                       function void platformUartSendByte(uint8 data) {
                         register %uartBaseAddress = config.baseAddress + 0x0; //Example UART base address
                         register %byteToSend = data;
                         if (@enable("dma")) {
                          custom_dma(%uartBaseAddress, %byteToSend); //using the custom_dma instruction using dma registers
                           } else {
                            memoryStore(byte, %uartBaseAddress, %byteToSend);
                           }
                         print("ARM: Sent byte using register");
                       }
                      function void platformUartSendString(StringLiteral str) {
                             for (uint32 i : 0..stringLength(str)) {
                                platformUartSendByte(str[i]);
                             }
                       }
                    function void my_function(dataAddress address, uint32 value, register myReg) {
                     load_reg<0>(address, myReg);
                        store_reg(address, myReg);
                          add_reg<ADD>(myReg, myReg);
                        add_reg<SUB>(myReg, myReg);
                       }
                     }
                   @arc riscv {
                       function void platformUartInit(uint32 baudRate) {
                        register %uartBaseAddress = config.baseAddress + 0x0; // Example UART base address
                        register %baudRateValue = baudRate;
                         memoryStore(dword, %uartBaseAddress + 0x04, %baudRateValue); // Example code to set baud rate
                           print("RISCV: Uart Initialized");
                       }
                      function void platformUartSendByte(uint8 data) {
                           register %uartBaseAddress = config.baseAddress + 0x0; //Example UART base address
                           register %byteToSend = data;
                           memoryStore(byte, %uartBaseAddress, %byteToSend);
                          print ("RISCV: Sent byte using register");
                       }
                       function void platformUartSendString(StringLiteral str) {
                             for (uint32 i : 0..stringLength(str)) {
                                platformUartSendByte(str[i]);
                             }
                       }
                   }
                }
            Meta module MyMetaCam {
                config {
                config hardwareDescription : StringLiteral;
                    config camName : StringLiteral = "MyGeneratedCam";
             }
         @arc any {
           function void generateCam (CompilerContext context) {
               //read the config parameters, and generate the new CAM.
                }
            }
         }

         module MyOtherCam {
                #use "UartCam" with config { enableDma = false };
                #use "UartCam" override function platformUartSendByte with config { enableDma = true };
            }
              ```
        *   The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body, to perform low-level hardware operations.
        *   The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself, using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        *  The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions, which may have generic parameters, that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
        *  The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        * Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*  **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *   The import system will resolve conflicts, with the last imported CAM overriding previous definitions. Explicit override mechanisms may be added in the future.
        *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
    *   **Syntax:** `#use "camName" [genericInstantiation];`
       *   Example:  `#use "MyBaseUartCam";`, `#use "GenericMathCam<float32>";`
        *   The `genericInstantiation` is similar to the module instantiation syntax: `<typeList>`.
        *   **Conflict Resolution:** When CAMs redefine interface intrinsic functions, the last CAM imported that defines that function will override previous definitions. There is no explicit mechanism to select a specific version of the interface intrinsic functions. The order of imports is important to resolve these conflicts.
        *   **Config Inheritance:** Imported CAMs can access the `config` parameters of the parent CAM. The `config` attributes are merged and if there are conflicts, the parent CAM configuration overrides the imported CAM configurations. The scope of config parameters is the module where the CAM is used. Config variables are resolved at compile-time, using static analysis.
*   **CAM Composition:**
    **   **Mechanism:** CAMs can be composed by using the `merge` keyword in the `#use` directive: `#use "MyCam1" merge "MyCam2";`.
    *   The `merge` keyword will attempt to combine the interface intrinsic functions of two or more CAMs into a single CAM. If there are conflicts, an error will be generated.
    *   The `config` parameters will be merged, and if there are conflicts, the CAM listed first will have priority.
    *   If a CAM is imported without the merge keyword, it can still be used, without merging it with the current CAM.
*   **CAM Customization**
  * CAMs can be customized by using the `config` section and specific attributes.
  *  A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
   *  The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
   * If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
     * CAM options can be enabled or disabled by using the  `@enable("myOption")` or `@disable("myOption")` attributes. Example: `function void myFunc() @enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean @enable("myOption") = true; }`

*  **CAM API: Compiler Interface for CAMs**
     *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context, provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                      *  The `dataflowGraph` is composed of nodes and edges, representing the data flow through the program. Each node can be accessed by using the `nodes` field of the dataflow graph. Each node has metadata and information about the operations that are performed, including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges, and to perform specific data flow analysis tasks.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    * Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                 *    `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer, and so on. See enum `TypeKind` for possible types.
                *    `sizeof(DataType)`:  Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                 *  **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                     * Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                 *  **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                 * **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                 *  **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                      *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                     * The implementation is a function pointer, that must have the same signature as the `functionSignature` string.
                 *  **`newCAM(StringLiteral camName, interfaceDefinitions, implementation)`:** Creates a new CAM with the given parameters.
                     * The interfaceDefinitions will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                      * The implementation will be a set of function pointers that define the implementations of the interface intrinsic functions.
                      ```baremetal
                          @arc any { // the @arc block must be included. The code in this block is executed at compile time.
                             function void generateCam (CompilerContext context) {
                                // read the config values
                                 print("Meta Cam generating: " + context.config.camName);
                                 //  parse the hardwareDescription file and extract the register information
                                 // generate a new CAM
                                 StringLiteral interfaceDefinitions = "interface intrinsic function void platformInit(uint32 baseAddress) -> void;";
                                StringLiteral implementation = " function void platformInit(uint32 baseAddress) {  register %myReg = baseAddress; } ";
                                context.newCAM(context.config.camName, interfaceDefinitions, implementation);
                             }
                           }
                      ```
                *   **`addTransformation(Transformation transformation)`:** Adds a transformation to the code, which is then performed by the compiler during a code generation phase.
                    * The `Transformation` object contains:
                 *  **`removeCode`**: The code that should be removed, specified as a string.
                 * **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                 * **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                 *  **`target`**: A data structure with information about the target, including the type of the data being transformed, the memory region, and other metadata about the access, allowing the CAM to be more specific when performing code transformations.
                * **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context, otherwise `false`.
                 * **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                 *  **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                  * **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                  * **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                  *   **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                  * **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal.
                  * **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern.
            *   Example:
                ```baremetal
                 @arc arm {
                    function void myCamFunction(CompilerContext context) {
                        if (context.targetArchitecture == "arm") {
                            // Target specific code.
                            // ... use the compiler API
                             context.compilerError("My custom error message, when the architecture is ARM");
                        }
                  }
                }
                ```
        * **`InstructionBuilder` Object:**
            *   Provides an interface for creating symbolic instruction sequences in the `instrDSL`, and provides the following methods:
            *   Methods include:
                *   `instructionBuilder.loadRegister(dataAddress address, register reg)`: Creates a `load` instruction for a register.
                *   `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *   `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                *    `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
            *   Example of using the `instructionBuilder` in the `instrDSL` block:
                ```baremetal
                 @arc arm {
                      @instrDSL {
                        instr void myInstruction(dataAddress address, register reg, uint32 imm) {
                            InstructionBuilder builder = new InstructionBuilder();
                            builder.loadRegister(address, reg);
                            builder.addImm(reg, reg, imm);
                            // generate the instruction sequence using the builder object
                            // the compiler is responsible for mapping these symbolic instructions to real hardware instructions
                           }
                       }
                    function void my_function(dataAddress address, uint32 value, register myReg) {
                         myInstruction(address, myReg, 10);
                   }
                 }
                ```

*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *   The import system will resolve conflicts, with the last imported CAM overriding previous definitions. Explicit override mechanisms may be added in the future.
        *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
    *   **Syntax:** `#use "camName" [genericInstantiation];`
       *   Example:  `#use "MyBaseUartCam";`, `#use "GenericMathCam<float32>";`
        *   The `genericInstantiation` is similar to the module instantiation syntax: `<typeList>`.
        *   **Conflict Resolution:** When CAMs redefine interface intrinsic functions, the last CAM imported that defines that function will override previous definitions. There is no explicit mechanism to select a specific version of the interface intrinsic functions. The order of imports is important to resolve these conflicts.
        *   **Config Inheritance:** Imported CAMs can access the `config` parameters of the parent CAM. The `config` attributes are merged and if there are conflicts, the parent CAM configuration overrides the imported CAM configurations. The scope of config parameters is the module where the CAM is used. Config variables are resolved at compile-time, using static analysis.
*   **CAM Composition:**
    **   **Mechanism:** CAMs can be composed by using the `merge` keyword in the `#use` directive: `#use "MyCam1" merge "MyCam2";`.
    *   The `merge` keyword will attempt to combine the interface intrinsic functions of two or more CAMs into a single CAM. If there are conflicts, an error will be generated.
    *   The `config` parameters will be merged, and if there are conflicts, the CAM listed first will have priority.
    *   If a CAM is imported without the merge keyword, it can still be used, without merging it with the current CAM.
*   **CAM Customization**
  * CAMs can be customized by using the `config` section and specific attributes.
  *  A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
   *  The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
   * If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
     * CAM options can be enabled or disabled by using the  `@enable("myOption")` or `@disable("myOption")` attributes. Example: `function void myFunc() @enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean @enable("myOption") = true; }`

*   **Dataflow Analysis and Optimization:** CAMs can analyze data flow and optimize code.
*   **Type-Specific Optimizations and Specializations:** CAMs can generate code based on specific types.
*   **Hardware-Specific Functionality via DSLs:** CAMs can use DSLs for hardware interaction.
*   **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access for AI Ops:** Direct access to AI registers.

**5. Directives and Pragmas**

*   **Preprocessor Directives (Start with `#`):**
    *   `#define identifier value`: Macro definition for compile-time variables that are replaced during the preprocessing phase, which can improve readability and performance.
        *   Example: `#define MAX_BUFFER_SIZE 1024`
    *   `#if condition`, `#elif condition`, `#else`, `#endif`: Conditional compilation based on compile-time conditions, allowing different code paths to be compiled for different configurations.
        *   Condition expressions support macro expansion, boolean logic (`==`, `!=`, `&&`, `||`, `!`) and also integer comparison operators (`>`, `>=`, `<`, `<=`).
         *   The preprocessor expressions only consider integer, boolean, or literal values. There is no type checking of preprocessor expressions. The preprocessor directives are evaluated before all other parts of the code, and the order in which they are defined in the file is important. The preprocessor works by text substitution of the values defined with `#define`, and by evaluating the `#if`, `#elif` and `#else` blocks.
        * Example:
            ```baremetal
             #define DEBUG_MODE true
              #if DEBUG_MODE == true
                  print("Debug Mode Enabled");
              #endif
            ```
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion, that makes the exported components of the module accessible.
    *   `#use "camName" [ genericInstantiation ]  [ "merge" , moduleName  { "," , moduleName } ] , [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion, that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
            *   Example: `#use "GenericVector<uint32>" as intVector;`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints, allowing the compiler to target specific features and improve the performance of the generated code.
        * Example:  `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        * Example: `#raw 0x10 0b00010000 16 0o20 "abc";`

*   **Operation Directives:**
    *   `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers, such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parenthesis can be omitted if only one parameter is passed. Example: `print "Hello world";`.
    *   `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier, depending on the target platform.
    *   `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *   `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parenthesis can be omitted if only one argument is passed.
    *  **Boot Code Specific:**
        *   `archCodeBlock [ @boot ] identifier @arc architectureName { ... }`: Architecture-specific raw code block for low-level initialization, bootloaders, and other specialized code. The `@boot` attribute specifies that this is the entry point for the boot code. Allows the developer to write low-level code using `goto`, `memoryStore`, `memoryLoad`, `raw`, `register`,  and memory access, that can be target-specific. The `identifier` is the name of the code block, that can be referenced using `goto` inside the architecture block. The `@arc architectureName` specifies the target architecture.
             *   Example:
                ```baremetal
                    archCodeBlock @boot bootEntry @arc arm {
                        register %stackPointer = 0x20001000;
                        goto bootLoaderLabel;
                        bootLoaderLabel:
                        // ...
                    }
                 ```
        *   `#interruptVector vectorNumber, functionName ;`: Defines interrupt vector table entries. Defines a mapping between a specific interrupt vector number and a corresponding interrupt service routine (ISR). The vector number is architecture specific. Example: `#interruptVector 0, resetHandler;`
        *   `#enableInterrupts ;`: Enables global interrupts, allowing the CPU to respond to external events.
        *   `#disableInterrupts ;`: Disables global interrupts, preventing the CPU from responding to external events.
        *   `#clearInterruptFlag identifier ;`: Clears an interrupt flag, acknowledging a pending interrupt and allowing the CPU to handle the next interrupt. The interrupt flag identifier is target specific.
        *    `bootRegion regionName identifier;`: Declares that a boot region should be populated using an identifier defined in the boot code region. If two boot regions are defined for the same address, a compile time error will be generated. Example: `bootRegion bootData myBootData;`
    *   **Memory Mapped Registers:**
        *   `struct @mapped`: Defines a memory-mapped structure, where the members are implicitly `volatile` unless marked `nonvolatile`. Allows for direct access to hardware registers. The `@mapped` specifier specifies that the struct is memory mapped. Members are accessed using the `.` operator, and bitfields can be accessed using `=` operator inside a `with` statement. Bitfield members can be accessed with the `=` operator and initialized by using an initializer list, without the `with` statement: `myGpioRegisters.controlRegister = { enableBit = 1, dataBit = someValue };`. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;`. Bitfield updates should be performed using a `with` statement, to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block, to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user, enabling zero-overhead high-performance bitfield access. When the `initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct @mapped(0x40000000) @initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`.
             *   Example:
                ```baremetal
                type GpioRegisters = struct @mapped { uint32 controlRegister; };
                GpioRegisters myGpioRegisters = 0x40000000;
                myGpioRegisters.controlRegister = {
                    enableBit = 1,
                    dataBit = someValue
                };
                myGpioRegisters.controlRegister[0..3] = 0b101;
                with (myGpioRegisters) {
                         enableBit := 1;
                          dataBit := someValue;
                     }
            ```
        *   `.` accesses members. If the member is inside a memory-mapped struct, it will access the memory location directly.
        *   **`=` sets/reads bitfields inside a memory-mapped struct.**
       *  Bit Ranges: **`[start..end]` accesses a range of bits in a bitfield inside a memory-mapped struct.**
        *   Memory Mapped Registers:  Specific registers can be defined using the following syntax:
         *   `DataType @register("regionName", "stringLiteral")  registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type, and can not be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch, a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 @register("MyAiCore", "control_reg") myMatrixAControl;`
   *   **Symbolic Execution:**
       *  `symbolic { ... }` block:
         *   The `symbolic` block allows the developer to define a region of code where the compiler will perform symbolic execution. Inside the symbolic block, the compiler will generate symbolic expressions for the operations and variables, which allows to verify program properties and perform advanced code optimizations by using the `CompilerContext` API.
            *   `symbolicResult(identifier)`: Returns the symbolic expression of a variable inside a symbolic block.
            *  `assert(expression)`: Performs a symbolic assertion, generating a compiler error if the assertion is false.
            * `hasSymbolicRegion():` Returns true if there is a symbolic region in the current program context.
              * `symbolicExpression(identifier)`: returns the symbolic expression for a given identifier in the program context.
*    **Algorithmic Code Generation:**
        * `@algorithm("stringLiteral")`: Attribute that allows a user to select the specific algorithmic code generator that will be used by the compiler for a given code region.

**6. Memory Safety in BML**

*   `DataType scoped`: Zero-overhead dangling pointer prevention using lexical scope. Pointers are valid only within their lexical scope, and the compiler ensures that the pointer can not escape the scope. This is implemented mostly at compile time, but depending on compiler flags, run time checks can be added. The `scoped` keyword is optional for local variables, and can be omitted, if the compiler can infer this, using static analysis.
     *   **Static Analysis Process for Implicit Scope:**
            *   The compiler performs static analysis to determine if a pointer's lifetime is strictly confined to its lexical scope, specifically its function body or a block inside the function.
            *   The compiler analyses the code flow to verify if the pointer is never passed as a parameter to a function or stored inside a struct that outlives the scope of the pointer, and if the pointer is never returned to a caller function. If all the conditions are satisfied, the compiler infers that the pointer is scoped and the `scoped` qualifier can be omitted.
            *   If the compiler detects that the pointer is escaping its lexical scope, a compile time error will be generated.
        * **Scope Rules:**
          *  If a `scoped` pointer is stored inside a struct, the struct will inherit the scope of the pointer, and can only be declared in the same scope or a scope that does not outlive the scope of the scoped pointer. If the struct is used outside the scope of the scoped pointer, a compile-time error will be generated. This rule also applies to unions and tuples.
           *  A `scoped` pointer can not be passed as a parameter to a function or be returned by a function, unless the `scoped` qualifier is also applied to the return value.
        *   Example:
        ```baremetal
        function void myFunction() {
            uint32 localValue = 10;
            uint32 localPtr = &localValue; // valid pointer, scoped is implicitly inferred.
            // ...
        } // localPtr is no longer valid
        ```
*   **Capability-Based Pointers:** `DataType +`, `DataType -`, `DataType @regionName`, `DataType !`, `DataType scoped`, `DataType tracked` for compile-time access control and memory management.
        *   `+` pointers can only be used to read and write from memory, and can not be null.
        *   `-` pointers can be null, and can be used to read/write from memory.
        *   `@regionName` pointers are associated with a specific data region, enabling access control and isolation. The compiler will issue an error if the pointer accesses memory outside the specified region.
        *   `!` pointers can not be null, and can only read from memory. This pointer will prevent writing into a specific memory location, guaranteeing read only access.
        *  `scoped` pointers are limited to their scope, guaranteeing safety in a more localized and restrictive way.
        *   `tracked` pointers have runtime checks to ensure the pointer is not dangling. The runtime checks can be implemented using different mechanisms, such as metadata, guard pages, or specific hardware support.
          * `DataType! ` is a constant read-only pointer. It will never allow writing to the address. The memory that it points to can change if the underlying memory region is mapped to a volatile or external device, for example.
          * Multiple pointers with different capabilities can point to the same memory location, as long as they have compatible capabilities (e.g. a read-only pointer, and a read-write pointer can point to the same location). If a conflict is detected by the compiler, a compile time error will be issued.
        *   Example: `uint32! myReadPtr;`, `rawbyte- myWritePtr;`, `uint32@MySecureRegion myRegionPtr;`, `uint32+ myNonNullPtr = &myValue;`, `uint8 localPtr = &myStackValue;` (implicit scope), `uint16 tracked myTrackedPtr = &someValue;`
*   **Static Typing:** Enforces type rules at compile time, preventing type-related errors, which makes the code safer and easier to reason about. Type checking is performed by the compiler, during the type resolution phase of the compilation process.
*   `DataType safe [Size]`: Runtime bounds checking to prevent buffer overflows and out-of-bounds accesses, improving the security of the code. Every access to a `safe` array will be checked at run time.
    *   Example: `uint16 safe[100] sensorValues; sensorValues[10] = 5;`
    *   `arraySlice<DataType>`: Provides bounds-checked views over a specific section of an existing array, which can be used to improve the safety of memory access. The bounds checking will be performed at runtime, and a runtime exception will be raised if the access is out of bounds of the original array, or the slice.
        *   Example: `uint32 safe [100] myArray; arraySlice<uint32> mySlice = myArray.slice(0,10);`
*   `DataType^`: Unchecked pointers (use with caution). The developer must handle all potential safety issues related to raw memory access. It is recommended to use other more secure pointer types whenever possible. If the developer uses a raw pointer, they will have to check for null pointers, and also perform bounds checking manually.
    *   Example: `uint32^ ptr = &memoryLocation;`
*   `DataType tracked`: Runtime checks to prevent dangling pointers, which is especially useful when handling complex memory accesses or concurrency. The tracked pointers have a small overhead that is associated to the metadata required to perform the runtime checks. The runtime checks can be implemented using metadata, guard pages, compiler specific techniques, or hardware support if available, which can reduce the overhead.
    *   Example:  `uint32 tracked myTrackedPtr = &someValue;`
*   **Region-Based Memory Management:** Implicit lifetime management through scopes and explicit regions, which simplifies memory handling and provides compile-time checks that increases the code safety. Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined. `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions, a CAM will have to explicitly read from one region and write to another region, using memory copy functions, or by transferring using memory mapped peripherals. CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
*   **Compile-Time Null Dereference Detection:** Static analysis provides warnings for potential null pointer dereferences, avoiding runtime crashes due to null pointers. The compiler will issue a warning if a pointer with a `-` modifier, or a general pointer using `^` is dereferenced without checking if the pointer is null. The compiler will not perform any null check if the `+` or `!` modifier is used, or if the pointer is a `scoped` pointer, as these pointers are guaranteed to not be null.
    *   Example: If a value is known to be potentially null, the compiler will generate a warning if the developer attempts to dereference it without performing a null check.
*   **Refinement Types:** Compile-time data constraints that allow to specify valid ranges or values for variables, further improving type safety by making the code more specific. Refinement types are used by the compiler to perform static analysis and can eliminate the need for runtime bounds checks.
    *   Example: `type validValue = uint16 where value < 100;`
*    **Capability-Based Regions:** Memory access control, enforced at compile time (mostly) that allows to isolate specific sections of the memory, preventing data corruption by limiting the access capabilities of pointers, providing increased safety and security. Memory regions can be defined using different capabilities, such as `read`, `write`, `execute` and `capability_derive`, which allows the CAM to perform more fine grained memory access controls.
        *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe [100] buffer; };`
    *   **Stack-Only Allocation:** `@stackOnly` enforces stack allocation, useful for performance and memory predictability, since it avoids the overhead of dynamic memory allocation and provides a predictable memory usage. When this attribute is used, the memory will be allocated in the stack.
        *   Example: `@stackOnly uint8 myStackVar;`
    *   **Hardware-Assisted Bounds Checking:** Optional, when hardware is available, and enabled via compiler flag or pragma, leveraging hardware to improve the performance and reduce the overhead of memory access checks. This is only used for safe arrays and tracked pointers, and will generate the appropriate memory access configuration for the target platform.
        *   Example: `#pragma enable_hardware_bounds_checking; uint32 safe[10] myArray;`
*   **Runtime Checks Tradeoffs:** While BML aims for minimal overhead, runtime checks are still present for certain features (e.g., `safe` arrays, `tracked` pointers). These checks help prevent memory errors, but they may have a performance impact. These checks can be disabled by using compiler flags or pragmas. Disabling the runtime checks should be done with caution, as it can introduce potential safety issues.
*   **Runtime Checks are Optional:** The runtime checks are optional, and may be disabled by using compiler flags or pragmas, such as `boundsCheck=off` or specific pragmas such as `#pragma disable_bounds_check;` This gives the user more control by explicitly disabling the runtime checks, but it also means that the user can create an unsafe program that can crash due to buffer overflows or dangling pointer accesses.
*   **Cutting-Edge Solution (High-Performance Bitfield Access):** To minimize overhead and maximize performance in bitfield manipulation, especially within memory-mapped registers, BML provides a "with" statement which allows for grouping bitfield updates, avoiding multiple read-modify-write operations, when there are multiple bitfield updates for the same register. This is a cutting-edge feature, designed to provide low overhead and high performance. The compiler can optimize the code by using specialized hardware instructions, where available, or by using bitwise operations where specialized instructions are not available.
     *   **"with" statement optimization:** The compiler analyses the `with` block, to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped.
        *   This mechanism is automatically performed by the compiler and requires no additional effort from the user, enabling zero-overhead high-performance bitfield access.
    *   **Example:**
        ```baremetal
          type UartControlRegister = struct @mapped {
               uint32 baud : 16;
               uint32 parity: 2;
                uint32 readout: 1;
               uint32 enable : 1;
          };
          function void initUart(dataAddress base) {
                UartControlRegister myUartRegisters = base;
                 with (myUartRegisters) {
                      baud := 115200;
                      parity := 2;
                     enable := 1;
                     uint32 myStatus = readout;
                   }
                  myUartRegisters.enable = 0; //single bitfield modification using read-modify-write.
           }
         ```
            *  **Optimizations:** The compiler will:
                 *  Read the register `myUartRegisters` only once, at the beginning of the `with` block.
                 *  Modify the bitfields in a local copy of the register.
                  * Perform a single memory write at the end of the `with` block, if any of the bitfields was modified.
                   * If the compiler can detect that the values assigned to a bitfield do not change, the write operation can be optimized away.
                   *  The compiler can use specific insert/extract instructions, when available, or generate bitwise operations with no runtime overhead.
*   **Implicit Read optimization:** When a bitfield is read inside the `with` block, but no bitfield is modified, the compiler may skip reading the register. For example:
    ```baremetal
        with (UART2.ccr) {
           uint32 myStatus = readout; //only read, not modified. The read operation may be optimized away if no bitfields are modified
         }
    ```

**7. Multi-Target Architecture Versatility**

*   `@arc architectureName { ... }`: Provides architecture-specific code implementations inside CAMs. This is the main method to provide different implementations of the same API, depending on the target architecture. The compiler will choose the appropriate code block, based on the target architecture specified by the user, by using the `-target` flag in the command line interface.
    *   Example: `@arc arm { /* ... */ }`, `@arc riscv { /* ... */ }`, `@arc x86 { /* ... */ }`
*   CAMs: Implement hardware abstractions with platform optimizations, allowing developers to write code once and reuse it in different platforms, without sacrificing performance. The abstraction mechanism allows developers to create reusable code that is portable across different architectures, but also perform architecture specific optimizations, using the features of a specific target platform.
*   `#cpuTarget cpuName`: Provides CPU optimization hints, allowing the compiler to target specific features and improve the performance of the generated code. The compiler will use the information of the target CPU to generate specialized code.
    *   The compiler selects CAMs and generates code based on the `-target` flag, which can be used to specify the target architecture and CPU.
       *  Example:  `#cpuTarget armCortexM4`

**8. Concurrency and Parallelism**

*   `threadHandle identifier = startThread(functionName);`: Creates concurrent tasks, allowing the developer to implement multi-threaded applications. The `startThread()` function creates a new thread. The way that the stack is handled depends on the target architecture and the compiler flags, and is implementation-defined. There is no mechanism to kill a thread or to join a thread directly. If required, you can use CAMs to implement such functionality.
     *   Example: `threadHandle myThread = startThread(myThreadFunction);`
*   `parallel { ... }`: Structured concurrency, allowing the developer to create parallel tasks that execute concurrently. The compiler may map these tasks to different threads depending on the underlying platform and target architecture, but the developer has no control over how the tasks are scheduled.
    *   Example:
        ```baremetal
        parallel {
            task1();
            task2();
        }
        ```
*   `await identifier = expression;` / `yield(expression);`: Asynchronous operations using awaitable types, which can be used to implement non-blocking operations and implement cooperative multitasking. The `await` statement will wait for the result of an asynchronous operation, without blocking the current thread. The `yield` statement can be used to return the result of the asynchronous operation.
    *   Example: `await myAwaitable = myAsyncFunction(); yield(myResult);`
*   `lock(identifier) { ... }` / `unlock(identifier);`: Critical section access to protect shared memory from data races, guaranteeing atomic access to shared resources by different threads. A mutex is a synchronization primitive used to implement critical sections.
    *   Example: `lock(myMutex) { /* Critical section */ }; unlock(myMutex);`
*   `wait(identifier);` / `signal(identifier);`: Synchronization primitives used to manage access to shared resources and coordinate concurrent tasks.
     * The `wait` operation may or may not release a lock (depending on implementation details).  The `signal` operation does not release the lock. CAMs can offer more control over these primitives, using customized implementations.
    *   Example: `wait(mySemaphore); /* Access resource */; signal(mySemaphore);`
*   Default memory consistency is sequential consistency, using `memoryBarrier();` for explicit ordering of memory accesses. There is no implicit memory barrier at the end of `parallel` blocks, as the developer must have full control of the memory access and ordering semantics.
     *  **Memory Barriers:**
        *   BML uses sequential consistency as its default memory model, which means that, by default, memory operations appear to execute in the order specified by the program.
        *   When interacting with shared memory, volatile variables or peripheral devices, the compiler may not preserve the order of the memory accesses, due to optimizations, or memory hierarchies.
       *   Memory barriers are used to enforce a specific order on memory access operations. When a memory barrier is executed, the CPU guarantees that all previous memory operations are completed before any subsequent operation starts.
       *   The `memoryBarrier();` operation will generate a target specific instruction that will act as a memory barrier. It is usually not required, unless you have a specific memory ordering that the compiler cannot enforce.
       *   Memory barriers should be used when you have multiple threads that access the same memory regions, or when you are interacting with memory-mapped devices that have dependencies on the order of the memory operations.
*    `transaction { ... }`: Ensures atomic execution of the code block. The implementation of transaction blocks depends on the target. For a single-core processor, the compiler will typically generate code that disables interrupts and system preemption during the execution of the transaction. For a multi-core system, the code will generate a lock or a memory barrier operation, which depends on the availability of hardware support. There are limitations on the operations that can be performed within a transaction to ensure atomicity. Calling functions may break the atomicity guarantees and the compiler will try to enforce these rules at compile time, if possible. The behaviour of calling functions inside transactions is implementation specific.

**9. Tooling and Debugging**

*  **Compiler Error Handling**
     *  The compiler reports compile-time errors in the following format (or similar, depending on the compiler implementation):
     *   `filename:line:column: error: Error message`
     * For example: `MyFile.bml:10:20: error: Type mismatch in assignment`.
     *   The compiler may also output a specific code associated with each error, to facilitate automated analysis of the error messages.
     * The CAMs can generate error messages with a specific format, by using the function `compilerError()`. The compiler may provide a CAM API or some other mechanism to guarantee a specific formatting for the error messages.
*   **`bmlc` Compiler:** Command-line tool for compiling BML code:
    *   `-target <architecture>`: Specifies the target architecture, e.g., `arm`, `riscv`, or `x86`.
    *   `-g`: Enables debug information for GDB/LLDB, to allow for source-level debugging using standard tools.
    *   `-S`: Outputs assembly code for the specified target, which allows for analysis of the generated code.
        *   `-O<level>`: Optimization level (e.g., `-O0`, `-O1`, `-O2`, `-O3`). `-O0` disables optimization, and `-O3` enables all optimizations.
    *   `-I<include_path>`: Adds a directory to the include path to allow the compiler to find modules, libraries, or header files.
    *   `-cam_path <cam_path>`: Adds a directory to the CAM module path, to allow the compiler to locate and use CAMs.
     *  `-enableCaching`: Enables caching of CAM outputs, to speed up compilation times. By default this is disabled.
    *   `-enableAi`: Enables abstract interpretation for enhanced compile-time error detection (experimental and may increase compilation time and may not be stable).
     *   `-enable-hardware-bounds-checking`: Enables hardware assisted bounds checking, using the MMU if available, which may reduce or eliminate the overhead of performing runtime bounds checks.
        * `-debug-ir`: Enables outputting the intermediate representation of the code for debugging purposes.
        *  `-debug-cam`: Enables outputting debug information for the CAMs, allowing developers to understand the CAM's code and transformations.
        * `-debug-transformations`: Enables outputting information about the code transformations performed by the compiler, which allows the developer to understand how the code is being modified during the compilation process.
*   **Source-Level Debugging:** With GDB or LLDB (command-line), which allows the user to set breakpoints, inspect variables, and step through the code. It also allows debugging CAMs, and stepping through the CAM code.
*   **Runtime Error Handling:**
        *   Runtime exceptions are generated when an access to an `safeArray` is performed out of bounds, or if the tracked pointer points to an invalid memory region.
       *   The default behavior is target and compiler specific. It might be implemented by generating a trap instruction, or by invoking a runtime exception handler.
       * The user can override the default error handling mechanism by using CAMs, which are responsible to map the error handling mechanism to a specific platform. For example, a CAM can map the error handling to an interrupt handler, that can handle specific error codes related to the runtime exceptions generated by BML code.
        * It is also possible to perform custom error handling by using the `Result` type, which forces the user to explicitly check for errors.
*   **`BML Advanced Compiler Guide`:** (Separate document) - Compiler internals and advanced debugging techniques, which is intended for advanced users that require deeper insight into the compilation process. The advanced compiler guide also includes more details on how to debug the CAM code, and use the output provided by the compiler to debug low-level code transformations.
*  **Debugging Complexity:** While source-level debugging is supported, debugging low-level issues that involve memory corruption and interactions with hardware components may be more difficult in BML compared to the direct observability provided by assembly-level debugging. BML introduces an abstraction layer via the CAMs, and the developer must be familiar with how the CAMs interact with the hardware and the BML runtime to fully debug low-level issues. Debugging memory-mapped peripherals may be more complex in BML compared to assembly language, because there is no direct instruction level control over the hardware. The user must rely on the CAMs to generate the correct instructions. Debugging runtime exceptions caused by out of bounds accesses, or other memory related errors may require more effort by the user, to analyse the stack traces, and to identify the source of the issue. However, the advanced debugging tools, and information provided by the compiler, helps to reduce the overall debugging complexity.

**10. Standard Library and CAM Ecosystem**

*   **Minimal Core Library:** Zero-overhead.
    * String literals are encoded using UTF-8. Character literals are encoded using the encoding of the character type (e.g. UTF-8, UTF-16, or UTF-32).
    *   Examples:
        *   `uint32 stringLength(StringLiteral str) -> uint32;` (string length calculation). This function computes the length of a given string literal. It can be used for creating loops that iterate over characters in a string literal, or for allocating buffers based on the size of a given string literal. It does not check if the string literal is null terminated, and does not support generic string variables.
        *   `toUint32(int16 value) -> uint32;` (integer conversion from a 16 bit signed integer to a 32 bit unsigned integer). This function converts a 16 bit signed integer to a 32 bit unsigned integer. It can be used to perform type conversion between integer types. No error checking or overflow checking is performed.
        *   `memoryBarrier();` (explicit memory barrier). This function generates a hardware-specific memory barrier that guarantees that memory operations are performed in the specified order. This function should be used to guarantee memory consistency when interacting with shared memory, or peripherals. It is an empty operation in systems without caches or multiple cores.
        *   **`print(StringLiteral format, ...);`  (prints a string or a variable). This function prints a formatted message. The format string uses standard format specifiers, such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. The parenthesis can be omitted if only one parameter is passed: `print "Hello";`.
        *   `void exit(int32 code);` (exits the program with a given code). This function exits the program with a given exit code. The exact implementation is platform specific.
        *  `arraySlice<DataType> slice(uint32 safe [Size] array, uint32 start, uint32 length);` (Creates a slice of an existing safe array). This function returns a mutable view of a specified section of an existing array. The start and the length parameters must be inside the array limits.
        *  `void compilerError(StringLiteral message);` (Generates a custom compiler error, used by CAMs to generate more specific error messages). This function generates a compile-time error. This function should only be used in CAMs.
        *  `uint32 countLeadingZeros(uint32 value) -> uint32;` (Counts the leading zero bits). This function counts the number of leading zero bits (starting from the MSB). The result is between 0 and 32.
        *  `uint32 countTrailingZeros(uint32 value) -> uint32;` (Counts the trailing zero bits). This function counts the number of trailing zero bits (starting from the LSB). The result is between 0 and 32.
        * `uint32 countSetBits(uint32 value) -> uint32;` (Counts the number of bits that are set to 1). This function counts the number of bits that are set to 1 in a 32 bit integer.
        *   `Result<OkType, ErrType> myFunction() -> Result<OkType, ErrType>;`:  If there is an error, the function may return `Err`, otherwise, it will return `Ok` followed by the data, or it may return `Ok()`, if no data is provided.
        *   `void registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation);` (CAM API to register new interface intrinsic function dynamically). This function allows the CAM to register a new interface intrinsic function dynamically, that will be visible to other modules. The `functionSignature` is a string that includes the function parameters and return type. The implementation is a function pointer, that must have the same signature as the `functionSignature` string.
        *  `void addTransformation(Transformation transformation);` (CAM API to add a new transformation to the code). This function allows the CAM to add code transformations by passing a transformation object. The `Transformation` object contains the code to remove, the code to insert, the transformation type, and a data structure with information about the target.
       *   `void newCAM(StringLiteral camName, interfaceDefinitions, implementation);` (CAM API to create new CAMs during compile time). This function allows the CAM to dynamically generate a new CAM with a given name, a set of interface intrinsic functions and a set of implementation functions.
        *  `boolean hasSymbolicRegion()`: Returns true if there is a symbolic region in the current program context.
         * `StringLiteral symbolicExpression(identifier)`: Returns the symbolic expression for a given identifier in the current program context.
         * `StringLiteral getAttribute(identifier, StringLiteral attributeName)`: Returns the string literal of a given attribute.
        *  `boolean hasInstruction(StringLiteral instructionName)`: Returns true if an instruction is present in the current code context.
          * `StringLiteral getInstructionCode()`: Returns the current instruction as a string, which can be used to perform code transformations.
          *`TargetData instructionTarget()`: Returns information about the current code target.
          *   `StringLiteral capture(StringLiteral pattern, StringLiteral expression, identifier)`: Captures the sub-expression based on a given pattern, that can be used to perform code transformations.
           *`boolean match(StringLiteral pattern, StringLiteral expression)`: checks if the expression matches the pattern specified as a string.
*   **CAM Ecosystem:** Provides extended and reusable components, hardware drivers, operating system components, algorithms and data structures. The main mechanism to extend the language is through CAMs. BML relies on CAMs to generate optimized code for a given target architecture. BML does not allow to bypass the CAMs and access the hardware directly. If a CAM does not expose a specific hardware feature, or if it does not provide access to some specific hardware instruction, that feature or instruction cannot be accessed directly from BML code. The developer must request the CAM developer to expose new features. BML relies on CAMs for generating the required code to interact with peripherals. If a CAM does not expose a specific hardware feature, it cannot be accessed directly from BML code. The features exposed by BML are limited to the capabilities of the CAMs used in the compilation process. CAMs can be extended to expose new hardware features and to provide different optimizations. If a CAM does not provide some specific feature, the developer can create a new CAM or extend an existing one, to expose that functionality. This approach allows to have a flexible and adaptable system.

**11. Conformance and Language Evolution**

*   **Conforming Compiler:** Adheres to BML Language Specification, and generates code that follows the described behaviour.
    *   The compiler must generate a warning when a null pointer is dereferenced without checking for null, and also generate a warning when a scoped pointer escapes its scope, if the appropriate flags are enabled.
    *   The compiler must perform all the compile-time checks specified by the language, such as refinement type restrictions, region based memory access, and other static safety checks.
    *   The compiler must properly manage the cache of CAM outputs and intermediate representations, when the `-enableCaching` flag is specified.
     *   The compiler must report compile time errors using a clear and concise format. The format for error reporting can vary depending on the compiler implementation. A typical error message should have the file name, line number, and a clear description of the error, and also provide context where the error occurred. The CAMs can generate error messages with a specific format, by using the function `compilerError()`. The compiler may provide a CAM API or some other mechanism to guarantee a specific formatting for the error messages.
*   **Conforming Program:** Valid syntax, type-correct, uses supported features, and compatible CAMs, and follows all the language specifications.
*   **Language Evolution:** Backward compatibility is an important goal, and community participation is welcome.

**12. Appendix: EBNF Grammar**

```ebnf
sourceFile =
    { multiTargetSourceFile } , eof;

multiTargetSourceFile =
    { preprocessorDirectiveStatement }
  , { regionDeclarationBlock }
  , { moduleUseDirective }
  , { targetArchitectureBlock | moduleDefinition }
  , eof ;

moduleDefinition =
    [ genericModuleDeclarationHeader ] , "module" , moduleName , "{" ,
    moduleDeclarationBlock ,
    moduleStatementBlock ,
    exportBlock ,
     [ metaBlock ] ,
    "}" ;

metaBlock =
    "Meta" ;

genericModuleDeclarationHeader =
    "generic" , "<" , typeParameterList , ">" ;

moduleDeclarationBlock =
    "declarations" , "{" , { declaration } , "}" ;

moduleStatementBlock =
    "statements" , "{" , { statement | moduleUseDirective} , "}" ;

exportBlock =
    "export" , "{" ,
    [ exportedFunctions ] ,
    [ exportedTypes ] ,
    [ exportedConstants ] ,
    [ exportedConceptualRegisters ] ,
    "}" ;

exportedFunctions =
    "functions" , "{" ,  exportedFunctionList  , "}" ;

exportedTypes =
    "types" , "{" ,  exportTypeList  , "}" ;

exportedConstants =
    "constants" , "{" ,  exportConstantList  , "}" ;

exportedConceptualRegisters =
    "conceptualRegisters" , "{" ,  exportConceptualRegisterList , "}" ;

exportedFunctionList =
    identifier { "," , identifier } ;

exportTypeList =
    identifier { "," , identifier } ;

exportConstantList =
    identifier { "," , identifier } ;

exportConceptualRegisterList =
    identifier { "," , identifier } ;

targetArchitectureBlock =
    "@arc" , architectureName , "{" ,
    declarationBlock ,
    statementBlock ,
    [ bootFunctionDefinition ],
    [ instrDSLBlock ] ,
     [ camFunctionDefinition ] ,
    "}" ;

declarationBlock =
    "declarations" , "{" , { declaration } , "}" ;

statementBlock =
    "statements" , "{" , { statement } , "}" ;

camFunctionDefinition =
    "function", identifier, "(" , "CompilerContext", identifier , ")" , block;
declaration =
    variableDeclaration
  | constantDeclaration
  | functionDeclaration
  | typeDefinition
  | structDefinition
  | unionDefinition
  | configDefinition
  | architectureConditionalDeclaration
  | archCodeBlock
  | interruptServiceRoutineDeclaration
  | enumDefinition
  | attributedVariableDeclaration
  | attributedFunctionDeclaration
  | attributedStructDefinition
  | attributedEnumDefinition
  | threadDeclaration
  | mutexDeclaration
  | semaphoreDeclaration
  | awaitableDeclaration
  | destructuringDeclaration
  | interfaceIntrinsicFunctionDeclaration
  | refinementTypeDefinition
  | registerPoolDeclaration
  | traitDefinition
  | dslDefinition
  |  regionBasedTypeDefinition
  | compileTimeFunctionDeclaration
  | adtDefinition
  | camDataRegionDefinition
  | registerDeclaration
  ;

registerDeclaration =
    dataType , [ regionSpecifier ] , identifier, typeAttributeList, ";" ;

camDataRegionDefinition =
    "CAMDataRegion",  [regionCapabilitySpecifier], identifier , "{" , {camDataRegionDeclaration }, "}" ;

camDataRegionDeclaration =
    variableDeclaration
    | constantDeclaration
    | structDefinition
    | unionDefinition
    | configDefinition
    | typeDefinition
    | comment
    | enumDefinition
    | interfaceIntrinsicFunctionDeclaration
    | destructuringDeclaration
    | stackOnlyVariableDeclaration
    | refinementTypeDefinition
    ;

compileTimeFunctionDeclaration =
    "compileTimeFunction" , [ "export" ] , identifier , "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , block;

regionBasedTypeDefinition =
    "type" , identifier , "=" , baseDataType , "@region" , "(" , stringLiteral , ")" , ";" ;

registerPoolDeclaration =
    "registerPool",  identifier , typeAttributeList, ";" ;

refinementTypeDefinition =
    "type" , identifier , "=" , baseDataType , "where" , constraintExpression , ";" ;

constraintExpression =
    valueRangeConstraint
  | lengthConstraint
  | arbitraryPredicateConstraint
  ;

arbitraryPredicateConstraint =
    expression ;

valueRangeConstraint =
    "value" , "in" , "range" , "(" , constantExpression , ".." , constantExpression , ")" ;

lengthConstraint =
    "length" , "<" , constantExpression ;

interfaceIntrinsicFunctionDeclaration =
    "interface" , "intrinsic" , "function" , functionSignature , ";" ;

functionSignature =
    [ genericFunctionDeclarationHeader ] , identifier , "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , [ "ensures",  expression ] ;

variableDeclarationCore =
    dataType? identifier [ "=" expression ] ";"
    ;

variableDeclaration =
    [ typeAttributeList], [ mutabilityModifier ], variableDeclarationCore ;

mutabilityModifier =
  "~";

constantDeclaration =
    "const" , [ "export" ] , dataType , identifier , "=" , constantExpression , ";" ;

functionDeclaration =
     "function" , [ "inline" | "pure" | "noinline" | bootFunctionAttribute | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute  | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier ,  "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , [ "ensures",  expression ] , block
   | "function", [ "inline" | "pure" | "noinline" | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute  | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier, dataType, identifier, block;

bootFunctionDefinition =
    bootFunctionDeclaration , block ;

bootFunctionDeclaration =
    "@boot" , "function" , [ "inline" | "pure" | "noinline" | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute ] , identifier , "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , [ "ensures",  expression ] , block ;

bootFunctionAttribute =
    "@boot" ;

compileTimeFunctionAttribute =
    "compileTimeFunction" ;

stackOnlyFunctionAttribute =
    "@stackOnly" ;

genericFunctionDeclarationHeader =
    "generic" , "<" , typeParameterList , ">" ;

typeParameterList =
    identifier { "," , identifier } ;

parameterList =
    parameter { "," , parameter } ;

parameter =
    dataType , identifier [ "=" constantExpression ] ;

typeDefinition =
    "type" , [ "export" ] , identifier , "=" , dataTypeExpression , ";" ;

dataTypeExpression =
    dataType ;

structDefinition =
    "struct" , [ typeAttributeList ] , identifier , [ "@mapped" , [ "(" , addressExpression , ")" ] ] , "{" ,
    { structMemberDeclaration }
    "}" , ";"
   | "struct" , [ typeAttributeList ] , identifier ,  dataType, identifier, ";"
    | "struct" , [ typeAttributeList ] , identifier ,  "@mapped" , [ "(" , addressExpression , ")" ] ,  "@initializerList"  , "{" ,
        { structMemberDeclaration }
        "}" , ";"
;

structMemberDeclaration =
    attributedStructMemberDeclaration
  | structMemberDeclarationCore ;

attributedStructMemberDeclaration =
    typeAttributeList , structMemberDeclarationCore ;

structMemberDeclarationCore =
    [ "nonvolatile" ] , [ "volatile" ] , dataType , identifier , [ typeAttribute ] , ";" ;

unionDefinition =
    "union" , identifier , "{" ,
    { unionMemberDeclaration }
    "}" , ";" ;

unionMemberDeclaration =
    dataType , identifier , ";" ;

configDefinition =
    "config" , identifier , [ "extends" , identifier ] , "{" ,
    { configMemberDeclaration }
    "}" , ";" ;

configMemberDeclaration =
    [ "override" ] , "config" , dataType , identifier , "=" , constantExpression , ";" ;

architectureConditionalDeclaration =
    "when" , ( "@arc" | "cpu" ) , "is" , cpuName , declaration
  { "elseWhen" , ( "@arc" | "cpu" ) , "is" , cpuName , declaration }
  [ "else" , declaration ] ;

archCodeBlock =
   [ "@boot" ] , "archCodeBlock" , identifier , [ "extends" , identifier ] , "@arc" , architectureName , "{" ,
    { statement | bootCodeStatement}
    "}"
 | "@arc" , architectureName , "{" ,
    { statement | bootCodeStatement}
   "}" ;

interruptServiceRoutineDeclaration =
    "interrupt" , [ interruptAttributes ] , "function" , identifier , "(" , ")" , block ;

interruptAttributes =
    "(" , interruptAttribute { "," , interruptAttribute } , ")" ;

interruptAttribute =
    "fast"
  | ( "priority" , "=" , integerLiteral )
  | "naked" ;

architectureName =
    "intel" | "arm" | "riscV" | identifier ;

cpuName =
    "intelPentium4" | "intelCorei7" | "amdAthlon" | "amdRyzen" | "armCortexM0" | "armCortexM3" | "armCortexM4" | "armCortexM7" | "armCortexM33" | "armCortexM55" | "armCortexA7" | "armCortexA9" | "armCortexA15" | "armCortexA53" | "armCortexA57" | "armCortexA72" | "armCortexA76" | "armCortexA78" | "riscv32i" | "riscv32e" | "riscv64i" | "riscv64g" | identifier ;

moduleName =
    identifier | stringLiteral ;

dataType =
    [ regionSpecifier ] , { typeQualifier } , baseDataType , [ ":" , integerLiteral ] , [ arrayDimension ], [ pointerSymbol ], [ genericTypeInstantiation ] , [dependentTypeInstantiation] ;

dependentTypeInstantiation =
    "<" , constantExpression , ">";

baseDataType =
    integerTypeSpecifier
  | floatTypeSpecifier
  | "address"
  | "codeAddress"
  | "dataAddress"
  | "genericAddress"
  | "bootAddress"
  | "boolean"
  | characterTypeSpecifier
  | "struct" , identifier
  | "union" , identifier
  | "config" , identifier
  | identifier
  | functionPointerType
  | "rawbyte"
  | safeArrayType
  | dataType , "scoped"
  | dataType , "tracked"
  | "threadHandle"
  | "mutex"
  | "sync"
  | awaitableType
  | arraySliceType
  | tupleType
  | capabilityPointerType
  | resultType
  | refinementType
  | regionBasedType ;

regionBasedType =
    identifier;

resultType =
    "Result" , "<" , dataType, "," , dataType, ">" ;

capabilityPointerType =
    dataType , capabilityQualifier;

capabilityQualifier =
    "" , "+"
  | "" , "-"
  | "" , "@" , identifier
  | "" , "!"
  | "scoped"
  | "tracked"
  ;

tupleType =
    "tuple" , "<" , dataType { "," , dataType } , ">" ;

arraySliceType =
    "arraySlice" , "<" , dataType, ">" ;

awaitableType =
    dataType , "await" ;

safeArrayType =
     baseDataType, [ "safe" | "!" ], "[" , (constantExpression | identifier) , "]" ;

pointerSymbol =
    "^" ;

functionPointerType =
    [ regionSpecifier ] , "function" , "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType] ;

regionSpecifier =
   "dataRegion" , [ regionCapabilitySpecifier ]
  | "rodataRegion" , [ regionCapabilitySpecifier ]
  | "stackRegion" , [ regionCapabilitySpecifier ]
  | "peripheralRegion" , [ regionCapabilitySpecifier ]
    | "moduleDataRegion" , [ regionCapabilitySpecifier ]
  | "bootCodeRegion" , [ regionCapabilitySpecifier ]
  | "interruptVectorRegion" , [ regionCapabilitySpecifier ] ;

regionCapabilitySpecifier =
    "capabilities" , "(" , capabilityList , ")" ;

capabilityList =
    capability { "," , capability } ;

capability =
    "read" | "write" | "execute" | "capability_derive" ;

regionDeclarationBlock =
    bootRegionBlock
  | attributedDataRegionBlock
  | attributedRodataRegionBlock
  | attributedStackRegionBlock
  | attributedPeripheralRegionBlock
  | attributedModuleDataRegionBlock
    | attributedCamDataRegionBlock
  ;

bootRegionBlock =
    "bootRegions" , "{" ,
    [ bootCodeRegionBlock ] ,
    [ bootDataRegionBlock ] ,
    [ interruptVectorRegionBlock ] ,
    "}" ;

bootCodeRegionBlock =
    "bootCodeRegion" , "{" , { bootCodeRegionDeclaration } , "}" ;

bootDataRegionBlock =
    "bootDataRegion" , "{" , { bootDataRegionDeclaration } , "}" ;

interruptVectorRegionBlock =
    "interruptVectorRegion" , "{" , { interruptVectorRegionDeclaration } , "}" ;

attributedDataRegionBlock =
    typeAttributeList , dataRegionBlockCore;

attributedRodataRegionBlock =
    typeAttributeList , rodataRegionBlockCore;

attributedStackRegionBlock =
    typeAttributeList , stackRegionBlockCore;

attributedPeripheralRegionBlock =
    typeAttributeList , peripheralRegionBlockCore;

attributedModuleDataRegionBlock =
    typeAttributeList , moduleDataRegionBlockCore;

attributedCamDataRegionBlock =
  typeAttributeList, camDataRegionBlockCore;

dataRegionBlockCore =
    "dataRegion" ,  [regionCapabilitySpecifier] , "{" , { dataRegionDeclaration } , "}"
 | "dataRegion" , [ regionCapabilitySpecifier ], identifier, dataType, identifier, ";"
 ;

rodataRegionBlockCore =
    "rodataRegion" , [ regionCapabilitySpecifier ], "{" , { rodataRegionDeclaration } , "}" ;

stackRegionBlockCore =
    "stackRegion" , [ regionCapabilitySpecifier ], "{" , { stackRegionDeclaration } , "}" ;

peripheralRegionBlockCore =
    "peripheralRegion" , [ regionCapabilitySpecifier ], "{" , { peripheralRegionDeclaration } , "}" ;

moduleDataRegionBlockCore =
    "moduleDataRegion" , [ regionCapabilitySpecifier ], "{" , { moduleDataRegionDeclaration } , "}" ;

camDataRegionBlockCore =
    "CAMDataRegion",  [regionCapabilitySpecifier], identifier, "{" , {camDataRegionDeclaration }, "}" ;

dataRegionDeclaration =
    variableDeclaration
  | constantDeclaration
  | structDefinition
  | union
  | configDefinition
  | typeDefinition
  | comment
  | enumDefinition
  | interfaceIntrinsicFunctionDeclaration
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition
  | dataRegionDeclarationCore
  ;

dataRegionDeclarationCore =
    identifier , "=" , [ "new" , dataType | expression ] , ";" ;

rodataRegionDeclaration =
    constantDeclaration
  | structDefinition
  | unionDefinition
  | configDefinition
  | typeDefinition
  | comment
  | enumDefinition
  | interfaceIntrinsicFunctionDeclaration
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition
  ;

stackRegionDeclaration =
    variableDeclaration
  | comment
  | emptyStatement
  | interfaceIntrinsicFunctionDeclaration
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition
  ;

peripheralRegionDeclaration =
    variableDeclaration
  | constantDeclaration
  | comment
  | emptyStatement
  | interfaceIntrinsicFunctionDeclaration
  | structDefinition
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition;

moduleDataRegionDeclaration =
    variableDeclaration
  | constantDeclaration
  | structDefinition
  | unionDefinition
  | configDefinition
  | typeDefinition
  | comment
  | enumDefinition
  | interfaceIntrinsicFunctionDeclaration
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition;

bootCodeRegionDeclaration =
    comment
  | emptyStatement
  | interfaceIntrinsicFunctionDeclaration
  | rawBinaryDirectiveStatement
  ;

bootDataRegionDeclaration =
    variableDeclaration
  | constantDeclaration
  | comment
  | emptyStatement
  | interfaceIntrinsicFunctionDeclaration
  | destructuringDeclaration
  | stackOnlyVariableDeclaration
  | refinementTypeDefinition;

interruptVectorRegionDeclaration =
    interruptVectorDirective
  | comment
  | emptyStatement
  | interfaceIntrinsicFunctionDeclaration
  ;

enumDefinition =
   "enum" , identifier , "{" ,
    [ enumMemberList ]
    "}" , ";"
  |  "enum" , identifier , enumMemberList , ";"
  |  "enum" , "typeKind" , "{" ,
    [ typeKindMemberList ]
    "}" , ";"
  | "enum" , "layoutAttributeKind" , "{" ,
    [ layoutAttributeKindMemberList ]
    "}" , ";" ;

enumMemberList =
    enumMember { "," , enumMember } ;

enumMember =
    identifier [ "=" constantExpression ] ;

typeKindMemberList =
    typeKindMember { "," , typeKindMember } ;

typeKindMember =
    "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" ;

layoutAttributeKindMemberList =
    layoutAttributeKindMember { "," , layoutAttributeKindMember } ;

layoutAttributeKindMember =
    "Packed" | "Aligned" | "RowMajor" | "ArrayOfStructures" ;


attributedVariableDeclaration =
    typeAttributeList , [ mutabilityModifier ], variableDeclarationCore ;

attributedFunctionDeclaration =
    typeAttributeList , functionDeclaration ;

attributedStructDefinition =
    typeAttributeList , structDefinition ;

attributedEnumDefinition =
    typeAttributeList , enumDefinition ;

typeAttributeList =
     [ typeAttribute { "," , typeAttribute } ] ;

typeAttribute =
     identifier
  | "@attribute" , "(" , attributeName , [ "," , attributeArgumentList ] , ")"
  | "boundsCheck"
  | "noBoundsCheck"
  | "boundsCheckStatic"
  | "optimize" , "(" , optimizationAttributeList , ")"
  | "potentialNull"
  | "guaranteedNonNull"
  | "boundsCheck" , ":" , boundsCheckOption
  | "stackOnly"
   | "@packed"
  | "@aligned" , "(" , integerLiteral , ")"
  | "@row_major"
  | "@array_of_structures"
  | "boundsCheckRefinement" , "(" , boundsCheckOption , ")"
  | "hasDataLayoutAttribute" , "(" , layoutAttributeKindIdentifier , ")"
  | "isType" , "(" , typeKindIdentifier , ")"
   | "sizeof" , [ "(" , typeKindIdentifier , ")" ]
  | "bitfield" , "(" , identifier , "." , identifier , ")"
    | "linearAccess"
  | "circularBuffer"
  | "readOnce"
  | "writeOnce"
  | "cacheLineAligned"
   | "@register" , "(" , stringLiteral , "," , stringLiteral , ")"
  | "schedule" , "(" , stringLiteral , ")"
  | "linearAccessGuaranteed" , "(" , [integerLiteral ] ,")"
  | "noOverlapGuaranteed"
  | "vector" , "(" , integerLiteral , ")"
  | "vectorAligned"
  | "inlineAlways"
  | "noinline"
  | "inlineIfSmall"
   | "@in"
  | "@out"
  | "@inOut"
    | "transform" , "(" , stringLiteral , ")"
  |  "allocator" , "(" , identifier , ")"
  | "unit" , "(" , stringLiteral , ")"
  | "registerPool" , "(" , stringLiteral , ")"
    | "scope" ,"(" , "this" ,")"
    | "lifetime" , "(" , stringLiteral ,")"
    | "capability" , "(" , capabilityList ,")"
    | "limit" , "(" , stringLiteral ,")"
    | "hardwareDescription", "(" , stringLiteral ,")"
     | "enable" , "(" , stringLiteral ,")"
      | "disable" , "(" , stringLiteral ,")"
    | "@algorithm", "(" , stringLiteral, ")"
    ;

layoutAttributeList =
    layoutAttribute { "," , layoutAttribute } ;

layoutAttribute =
    "packed"
  | "aligned" , "(" , integerLiteral , ")"
  | "row_major"
  | "array_of_structures" ;

layoutAttributeKindIdentifier =
    "Packed" | "Aligned" | "RowMajor" | "ArrayOfStructures" ;

typeKindIdentifier =
    "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" ;

boundsCheckOption =
    "runtime" | "compileTime" | "auto" | "off" | identifier ;

optimizationAttributeList =
    optimizationAttribute { "," , optimizationAttribute } ;

optimizationAttribute =
    "instructionSchedule" , ":" , instructionScheduleOption ;

instructionScheduleOption =
    "aggressive" | "default" | "none" | identifier ;

dataFlowAttribute =
    "in" | "out" | "inOut" ;

attributeName = identifier;
attributeArgumentList = constantExpression { "," , constantExpression };


bootStatement =
    bootRegisterDirectiveStatement
  | bootMemoryDirectiveStatement
  | bootControlFlowStatement
  | comment
  | emptyStatement
  | rawBinaryDirectiveStatement
  | bootRegionDirectiveStatement
  ;

bootRegionDirectiveStatement =
    "bootRegion" , regionName , identifier , ";" ;

regionName =
    "bootCode" | "bootData" | "interruptVector" ;

bootRegisterDirectiveStatement =
    "register" , registerName , "=" , bootOperand , ";" ;

bootMemoryDirectiveStatement =
    "memoryStore" , "(" , bootDataSize , "," , bootAddress , ","
  , bootOperand , ")" , ";"
  | "memoryLoad" , "(" , bootDataSize , "," , bootAddress , "," , bootOperand , ")" , ";"
   | "config" , identifier , "=" , constantExpression , ";" ;

bootDataSize =
    "byte" | "word" | "dword" | "qword" ;

bootAddress =
    constantExpression ;

bootOperand =
    register
  | immediateValue
  | bootMemoryOperand ;

bootMemoryOperand =
    "[" , bootAddress , "]" ;

bootControlFlowStatement =
    labelDefinition
  | jumpStatementDirective
  | conditionalJumpStatementDirective
  | gotoDirectiveStatement
  ;

labelDefinition =
    identifier , ":" , ";" ;

jumpStatementDirective =
    "jump" , addressExpression , ";" ;

conditionalJumpStatementDirective =
    "jumpIf" , conditionCode , "," , identifier , ";" ;

gotoDirectiveStatement =
    "goto" , ( identifier | "tailcall" ) , [ identifier , "(" , [ argumentList ] , ")" ] , ";" ;

conditionCode =
    "zero" | "notZero" | "carry" | "noCarry" | "overflow" | "noOverflow" | "greaterThan" | "greaterEqual" | "lessThan" | "lessEqual" | "equal" | "notEqual";


preprocessorDirectiveStatement =
    defineDirectiveStatement
  | ifDirectiveStatement
  | useModuleDirectiveStatement
  | useCamDirectiveStatement
  | cpuTargetDirectiveStatement
  ;

defineDirectiveStatement =
    "#define" , identifier , constantExpression , lineTerminator ;

ifDirectiveStatement =
    ifBlock { elseIfBlock } [ elseBlock ] , endIfBlock ;

ifBlock =
    "#if" , constantExpression , lineTerminator, { preprocessorDirectiveStatement | regionDeclarationBlock | moduleUseDirective | targetArchitectureBlock | moduleDefinition | declaration | statement | bootStatement | interruptServiceRoutineDeclaration | comment | emptyStatement } ;

elseIfBlock =
    "#elif" , constantExpression, lineTerminator, { preprocessorDirectiveStatement | regionDeclarationBlock | moduleUseDirective | targetArchitectureBlock | moduleDefinition | declaration | statement | bootStatement | interruptServiceRoutineDeclaration | comment | emptyStatement } ;

elseBlock =
    "#else" , lineTerminator, { preprocessorDirectiveStatement | regionDeclarationBlock | moduleUseDirective | targetArchitectureBlock | moduleDefinition | declaration | statement | bootStatement | interruptServiceRoutineDeclaration | comment | emptyStatement } ;

endIfBlock =
    "#endif" , lineTerminator ;

useModuleDirectiveStatement =
    "#use" ,  moduleName , [ genericModuleInstantiation ] , [ "with" , "config" , "{" , { configParameterAssignment } , "}" ] , ";" ;

useCamDirectiveStatement =
      "#use" ,  moduleName , [ genericModuleInstantiation ] ,  [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";" ;

genericModuleInstantiation =
    "<" , concreteTypeList , ">" ;

cpuTargetDirectiveStatement =
    "cpuTarget" , cpuName , lineTerminator ;

statement =
    expressionStatement
  | compoundStatement
  | selectionStatement
  | iterationStatement
  | jumpStatement
  | directiveStatement
  | emptyStatement
  | parallelBlockStatement
  | threadStatement
  | lockStatement
  | unlockStatement
  | waitStatement
  | signalStatement
  | yieldStatement
  | awaitStatement
  | destructuringDeclaration
  | stackOnlyVariableDeclarationStatement
  | compileTimeFunctionCallStatement
  | withStatement
    | compileTimeIfStatement
    | transactionStatement
     | hardwareForLoopStatement
     | "memoryStore" , "(" , bootDataSize , "," , bootAddress , "," , bootOperand , ")" , ";"
     | "memoryLoad" , "(" , bootDataSize , "," , bootAddress , "," , bootOperand , ")" , ";"
     | "compilerError" , "(" , stringLiteral , ")" , ";"
      | bootCodeStatement
     | symbolicBlock
  ;
symbolicBlock =
      "symbolic" , "{" , { statement }, "}" ;

bootCodeStatement =
    bootRegisterDirectiveStatement
  | bootMemoryDirectiveStatement
  | bootControlFlowStatement
  | rawBinaryDirectiveStatement
  | bootRegionDirectiveStatement
;

block =
    "{" , { statement | declaration } , "}" ;

hardwareForLoopStatement =
    "for" , "hardware" , "(" , variableDeclaration | expression, ";" , expression , ";" , expression , ")" , statement ;

transactionStatement =
    "transaction" ,block ;

stackOnlyVariableDeclarationStatement =
    "stackOnly" , variableDeclarationCore ;

destructuringDeclaration =
    [ typeAttributeList ], [ mutabilityModifier ] , dataType , destructuringVariableList , [ "from" ] , identifier , ";"
  | [ typeAttributeList ],  [ mutabilityModifier ] , "tuple" , "(" , destructuringVariableList , ")" , "=" , identifier , ";" ;

destructuringVariableList =
    identifier { "," , identifier } ;

compileTimeFunctionCallStatement =
    identifier , "(" , [ argumentList ] , ")" , ";" ;

withStatement =
     "with" , "(" , expression , ")" , "{" , { withStatementMember } , "}" , ";" ;

withStatementMember =
   identifier , ":=" , expression , ";" ;

expressionStatement =
    [ expression ] , ";" ;

compoundStatement =
    block ;

selectionStatement =
    ifStatement
  | switchStatement
    | matchStatement;

matchStatement =
    "match" , "(" , expression , ")" , "{" , { caseStatement } , "}" ;

ifStatement =
     "if" , "(" , expression , ")" , statement , [ "else" , statement ]
  |  "if" , "(" , expression , ")" , block , [ "else" , block ]
  ;

switchStatement =
    "switch" , "(" , expression , ")" , "{" , { caseStatement } , [ defaultCaseStatement ] , "}" ;

caseStatement =
    "case" , constantExpression , ":" , statement , "break" , ";" ;

defaultCaseStatement =
    "default" , ":" , statement , "break" , ";" ;

iterationStatement =
    whileLoopStatement
  | doWhileLoopStatement
  | forLoopStatement
  | rangeBasedForLoopStatement
  ;

rangeBasedForLoopStatement =
    "for" , "(" , identifier , ":" , rangeExpression , ")" , statement ;

rangeExpression =
    expression , ".." , expression ;

whileLoopStatement =
    "while" , "(" , expression , ")" , statement ;

doWhileLoopStatement =
    "do" , statement , "while" , "(" , expression , ")" , ";" ;

forLoopStatement =
    "for" , "(" , forInitializer , forCondition , forUpdater , ")" , statement ;

forInitializer =
    [ variableDeclaration | expression ] ;

forCondition =
    [ expression ] , ";" ;

forUpdater =
    [ expression ] ;

jumpStatement =
    gotoStatement
  | continueStatement
  | breakStatement
  | returnStatement ;

gotoStatement =
    "goto" , identifier , ";" ;

continueStatement =
    "continue" , ";" ;

breakStatement =
    "break" , ";" ;

returnStatement =
   "return" , [ expression | tupleLiteral ] ,  [ expression | tupleLiteral ] ,  ";"
  | [ expression | tupleLiteral ] , ";" ; //implicit return

tupleLiteral =
    "(" , [ argumentList ] , ")" ;

expression =
    assignmentExpression ;

assignmentExpression =
    conditionalExpression [ assignmentOperator assignmentExpression ] ;

assignmentOperator =
    "=" | "+=" | "-=" | "*=" | "/=" | "%=" | "&=" | "|=" | "^=" | "<<=" | ">>="
    | "+^=" | "-^=";

conditionalExpression =
    logicalOrExpression [ "?" , expression , ":" , conditionalExpression ] ;

logicalOrExpression =
    logicalAndExpression { "||" logicalAndExpression } ;

logicalAndExpression =
    bitwiseOrExpression { "&&" bitwiseOrExpression } ;

bitwiseOrExpression =
    bitwiseXorExpression { "|" bitwiseXorExpression } ;

bitwiseXorExpression =
    bitwiseAndExpression { "^" bitwiseAndExpression } ;

bitwiseAndExpression =
    equalityExpression { "&" equalityExpression } ;

equalityExpression =
    relationalExpression { ( "==" | "!=" ) relationalExpression } ;

relationalExpression =
    shiftExpression { ( "<" | ">" | "<=" | ">=" ) shiftExpression } ;

shiftExpression =
    additiveExpression { ( "<<" | ">>" ) additiveExpression } ;

additiveExpression =
    multiplicativeExpression { ( "+" | "-" ) multiplicativeExpression } ;

multiplicativeExpression =
    castExpression { ( "*" | "/" | "%"  | "+^" | "-^") castExpression } ;

castExpression =
    unaryExpression ;

unaryExpression =
    postfixExpression
  | sizeofExpression
  | unaryOperator , castExpression ;

unaryOperator =
    "+" | "-" | "!" | "~" | "^" | "&"  | "countLeadingZeros" | "countTrailingZeros" | "countSetBits" ;

sizeofExpression =
    "sizeof" , [ "(" , ( dataType | expression ) , ")" ] ;

postfixExpression =
    primaryExpression { postfixOperator } ;

postfixOperator =
    arrayAccessSuffix
  | structMemberAccessSuffix
  | unionMemberAccessSuffix
  | mmrBlockMemberAccessSuffix
  | functionCallSuffix
  ;

functionCallSuffix =
    "(" , [ argumentList ] , ")"
    | [ argumentList ]
  ;

arrayAccessSuffix =
    "[" , expression , "]" ;

structMemberAccessSuffix =
    "." , identifier ;

unionMemberAccessSuffix =
    "." , identifier ;

mmrBlockMemberAccessSuffix =
    "." , identifier ;

primaryExpression =
    identifier
  | constantLiteral
  | stringLiteral
  | "(" , expression , ")"
  | functionCall
  | arrayAccess
  | structMemberAccess
  | unionMemberAccess
  | mmrBlockAccess
  | conceptualRegisterAccess
  | resultConstructor
  | bitfieldAccess
  | adtConstructor
  | tupleLiteral;

functionCall =
    identifier , [ genericFunctionInstantiation ] , "(" , [ argumentList ] , ")" ;

genericFunctionInstantiation =
    "<" , concreteTypeList , ">" ;

argumentList =
    expression { "," , expression } ;

arrayAccess =
    primaryExpression , "[" , expression , "]" ;

structMemberAccess =
    primaryExpression , "." , identifier ;

unionMemberAccess =
    primaryExpression , "." , identifier ;

mmrBlockAccess =
    primaryExpression , "." , identifier ;

conceptualRegisterAccess =
    conceptualRegisterIdentifier ;

bitfieldAccess =
    primaryExpression , "=" , "{" , { bitfieldAssignment }, "}"
     | primaryExpression, "[" , expression , ".." , expression , "]" ,  "=" , expression
  | identifier , ":=" , expression ;

bitfieldAssignment =
    identifier , "=" ,  expression , "," ;

castOperator =
    "(" , dataType , ")" ;

constantExpression =
    constantLiteral
  | constantIdentifier
  | constantUnaryExpression
  | constantBinaryExpression
  | constantParenthesizedExpression ;

constantParenthesizedExpression =
    "(" , constantExpression , ")" ;

constantUnaryExpression =
    ( "+" | "-" | "!" | "~" ) , constantExpression ;

constantBinaryExpression =
    constantExpression , binaryOperator , constantExpression ;

binaryOperator =
    "+" | "-" | "*" | "/" | "%" | "==" | "!=" | "<" | ">" | "<=" | ">=" | "&&" | "||" | "&" | "|" | "^" | "<<" | ">>" ;

constantLiteral =
   integerLiteral
  | floatLiteral
  | characterLiteral
  | booleanLiteral ;

integerLiteral =
    [ prefix ], digit { digit } ;

prefix =
    "0x" | "0b" | "0o";

digit =
   "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" |
    "a" | "b" | "c" | "d" | "e" | "f" | "A" | "B" | "C" | "D" | "E" | "F" ;

floatLiteral =
    decimalFloatLiteral
  | exponentFloatLiteral ;

decimalFloatLiteral =
    digit { digit } , "." , digit { digit } , [ floatSuffix ] ;

exponentFloatLiteral =
    ( digit { digit } | decimalFloatLiteral ) , ( "e" | "E" ) , [ "+" | "-" ] , digit { digit } , [ floatSuffix ] ;

floatSuffix =
    "f" | "F" | "d" | "D" ;

characterLiteral =
    "'" , ( sourceCharacter | escapeSequence ) , "'"
  | "L" , "'" , ( sourceCharacter | escapeSequence ) , "'"
  | "U" , "'" , ( sourceCharacter | escapeSequence ) , "'" ;

stringLiteral =
    '"' , { sourceCharacter | escapeSequence } , '"'
  | "L" , '"' , { sourceCharacter | escapeSequence } , '"'
  | "f" , '"' , { sourceCharacter | escapeSequence } , '"'
  | "U" , '"' , { sourceCharacter | escapeSequence } , '"' ;

booleanLiteral =
    "true" | "false" ;

integerTypeSpecifier =
    signedIntegerTypeSpecifier | unsignedIntegerTypeSpecifier ;

signedIntegerTypeSpecifier =
    "int8" | "int16" | "int32" | "int64" | "int";

unsignedIntegerTypeSpecifier =
    "uint8" | "uint16" | "uint32" | "uint64" | "uint";

floatTypeSpecifier =
    "float32" | "float64" | "float";

characterTypeSpecifier =
    "char8" | "char16" | "char32" | "char";

typeQualifier =
    "volatile"
  | "const"
  | "nonnull"
  | "readOnly"
  | "threadLocal"
  | "" , "+"
  | "" , "-"
  | "" , "@" , identifier
  | "" , "!"
  | "stackOnly"
  | "scoped"
  | "tracked"
  ;

conceptualRegisterIdentifierCore =
    "%" , identifier ;

registerName =
    conceptualRegisterIdentifier ;

immediateValue =
    constantExpression ;

comment =
    singleLineComment | multiLineComment ;

singleLineComment =
    "//" , { sourceCharacter } , lineTerminator ;

multiLineComment =
    "/*" , { sourceCharacter } , "*/" ;

whiteSpace =
    { " " | "\t" | lineTerminator } ;

lineTerminator =  ? unicode line terminator character (lf, cr, crlf, etc.) ? ;

emptyStatement =
    ";" ;

identifier =
    letter { letter | digit | "_" } ;

(* lexical terminals (unicode categories) *)
letter =  ? unicode letter character ? ;
digit =  ? unicode digit character ? ;
sourceCharacter =  ? any unicode character ? ;
escapeSequence =  "\\" , ( "n" | "r" | "t" | "\\" | "'" | '"' | "0" ) ;

configParameterAssignment =
    identifier , "=" , constantExpression , ";" ;

concreteTypeList =
    dataType { "," , dataType} ;

parallelBlockStatement =
    "parallel" , "{" , { statement } , "}" ;

threadStatement =
    "threadHandle" , identifier , "=" , "startThread" , "(" , identifier , ")" , ";" ;

lockStatement =
    "lock" , "(" , identifier , ")" , block ;

unlockStatement =
    "unlock" , "(" , identifier , ")" , ";" ;

waitStatement =
    "wait" , "(" , identifier , ")" , ";" ;

signalStatement =
    "signal" , "(" , identifier , ")" , ";" ;

yieldStatement =
    "yield" , "(" , constantExpression , ")" , ";" ;

awaitStatement =
    "await" , identifier , "=" ,  expression , ";"
  ;

compileTimeIfStatement =
    "compileTimeIf" , "(" , constantExpression , ")" , "{" , { statement }, "}" ,  [ "compileTimeElse", "{" , { statement } , "}" ] ;

conceptualRegisterIdentifier = "%" , identifier ;

traitDefinition =
    "trait" , identifier , "<" , "type" ,  identifier , ">" , "{" , { traitMember }, "}" ;

traitMember =
    compileTimeFunctionDeclaration
  | implBlock;

implBlock =
    "impl" , identifier , "<" , identifier, ">" , "{" , { functionDeclaration }, "}" ;

dslDefinition =
    "dsl" , identifier , [ dslConfigParameters ] , "{" , dslBlock , "}" ;

dslConfigParameters =
   "(" , "config" , parameter { "," , parameter } , ")"
  ;

dslBlock =
    { dslSection } ;

dslSection =
    "states" , ":" , "{" , { dslStateDefinition } , "}"
  | dslCustomSection ;

dslCustomSection =
    identifier , "{" , { dslStatement } , "}" ;

dslStateDefinition =
    identifier , "{" , { dslStatement }, "}" ;

dslStatement =
    identifier , "(" , [ dslArgumentList ], ")" , [ "->" , identifier ], ";"
  | emptyStatement;

dslArgumentList =
    constantExpression { "," , constantExpression };

adtDefinition =
    "adt" , identifier , "{" , { adtMember } , "}" , ";" ;

adtMember =
    "enum" , identifier , "(" , [dataType] , ")" , ";"
  ;
instrDSLBlock =
  "@instrDSL" , "{" , { instrDefinition } , "}" ;

instrDefinition =
  "instr" , [ "export" ] ,  dataType , identifier, [ genericInstructionParameters ] , "(" , [ parameterList ] , ")" , "{" , { statement } , "}";

genericInstructionParameters=
   "<" , concreteTypeList , ">"
  ;
transformation =
  "remove" , codeBlock, "insert", codeBlock , "type" , transformationType;

codeBlock =
    "{" , { statement } , "}";

transformationType =
  "instructionReplacement" | "bitfieldOptimization" | "codeReordering";
  rule =
   "rule" , identifier ,  matchOperation , "->" , transformationOperation ,  ";";
   matchOperation =
     "match", "(" , identifier, expression, ")" ;
  transformationOperation =
    "replace" , "(" , expression, expression, ")" ;
}
```
