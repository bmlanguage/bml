**Bare Metal Language (BML) Specification (Version 2.0)**

**1. Introduction**

Bare Metal Language (BML) is a statically-typed programming language meticulously designed for bare metal and embedded systems development. BML is a *compiled language* offering a productive, maintainable, and memory-safe alternative to assembly language. BML is *not* a direct assembly replacement for instruction-level programming. Instead, BML abstracts instruction encoding at the user level while delivering *comparable performance* to assembly through its unique Composable Architecture Module (CAM) system. This system allows for fine-grained hardware control and platform-specific optimizations where and when needed.

**Key Differentiators of BML:**

*   **High-Level Abstraction with Low-Level Control:** BML offers high-level constructs for improved code organization and readability, while CAMs provide mechanisms to interact directly with hardware peripherals, registers, and memory locations at a level close to assembly but using type-safe and maintainable code.
*   **Explicit Memory Safety Features:** BML integrates multiple layers of memory safety: static typing, refinement types for value constraints, region-based memory management with capabilities, safe arrays with optional runtime bounds checks, and tracked pointers to mitigate dangling pointer issues. Runtime checks are optional and can be selectively disabled for performance-critical sections, placing responsibility for safety on the developer when disabled.
*   **Composable Architecture Modules (CAMs):** CAMs are compiler extensions written in BML itself. They are the cornerstone of BML's architecture versatility and performance. CAMs encapsulate architecture-specific code generation logic, instruction selection, and hardware interaction details, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages. Developers use BML modules and code, while CAM developers create platform-specific implementations, ensuring separation of concerns.
*   **Near-Zero Overhead Design:** BML strives for near-zero runtime overhead, avoiding hidden allocations, dynamic dispatch (unless explicitly requested), and garbage collection. Performance is intended to be comparable to hand-crafted assembly for equivalent functionality when using appropriate CAMs. However, runtime checks, even if optional, do introduce overhead. Developer responsibility for safety is paramount, especially when disabling runtime checks or using raw pointers.

**Core Principles:**

*   **Zero Overhead:** BML is designed with a zero-overhead philosophy, prioritizing minimal runtime costs and striving for performance comparable to hand-crafted assembly code where applicable and with the correct CAM usage. No hidden allocations, dynamic dispatch (unless explicitly requested), or garbage collection. Compile-time checks and optimizations, hardware capabilities, and CAMs enable this. Runtime checks are optional and can be disabled. Disabling runtime checks is the user's responsibility. The compiler will generate warnings when a `Result` value is ignored, a null pointer is dereferenced, or if a `scoped` pointer escapes its scope if the required flags are enabled.
*   **Low-Level Control:** BML provides mechanisms for direct hardware interaction, precise memory manipulation, and bit-level control via CAMs. Abstracting away direct instruction encoding at the user level, BML, through CAMs, allows for highly optimized code generation close to hand-written assembly. CAMs provide specialized APIs for hardware peripherals, registers, and memory locations. Direct hardware access at the user level is not allowed. Hardware access is done using memory-mapped registers and hardware intrinsics which are controlled directly by the developer.
*   **Memory Safety Mitigation:** BML incorporates features to mitigate memory errors at compile time. Static analysis, type constraints, region-based memory management, and hardware-assisted bounds checking provide layered memory safety. Runtime checks are optional, giving user control over the safety-performance tradeoff. Tracked pointers and safe arrays perform runtime checks that may impact performance if hardware-assisted bounds checking is disabled or static safety cannot be proven. Users must be aware of safety-performance trade-offs.
*   **Multi-Target Versatility:** BML supports compilation for diverse architectures through flexible CAM configurations. CAMs abstract architecture-specific details, promoting code reuse. Users must select the correct target architecture and CAMs. Code transformations and code optimizations are done using a rule-based system that is fully controlled by the developer.
*   **Modularity through CAMs:** BML enables system construction from reusable, optimized, and type-safe modules using CAMs, facilitating code reuse, platform-specific optimizations, and low-level hardware interaction. Code generation is done by CAMs by parsing a hardware description or by using code transformations defined in a rule-based domain-specific language.
*   **Readability and Maintainability:** BML prioritizes clear, concise code to enhance software engineering practices, improving code understanding and long-term maintenance.
*   **Tooling and Debugging:** BML enables source-level debugging with GDB/LLDB and provides advanced compiler debugging output and CAM-level debug output for efficient troubleshooting and code understanding.
*   **Concurrency and Parallelism:** BML includes built-in language-level support for multi-core processors, offering primitives for concurrent tasks and explicit control over synchronization and memory consistency. High-level concurrency primitives such as threads, mutexes, and semaphores are provided. The default memory consistency model is sequential consistency, and explicit memory barriers must be used when specific memory ordering is required.
*   **Metaprogramming:** BML provides compile-time computation and reflection capabilities that allow extending the language and implementing advanced optimizations using traits, compile-time functions, and DSLs, enabling flexible CAMs and compile-time code optimizations.
*   **Rule-Based Code Transformations:** The compiler uses symbolic execution and a rule-based DSL defined in CAMs to analyze code and optimize bit-level operations, generating optimized instruction sequences depending on the target architecture.
*   **Fine-Grained Hardware Dispatch:** BML provides mechanisms to dispatch operations to hardware accelerators using direct memory-mapped register accesses or specialized instruction sequences provided by CAMs. The developer must explicitly select the hardware accelerator by using specific functions or intrinsics.
*   **Direct Register Access for Hardware Accelerators:** BML provides mechanisms to directly access registers of hardware accelerators in hardware components (DSPs, neural processing units) using memory-mapped access and conceptual registers implemented by CAMs.
*   **Automatic Generation of CAMs from Hardware Specifications:** BML parses hardware specifications, described in a hardware description language (HDL) to automatically generate the CAM skeleton. Meta-CAMs enable flexible CAM generation during compile time based on high-level descriptions or hardware information.
*   **Configurable DSLs:** BML DSLs can be customized by using parameters defined in their `config` sections based on user needs or application context, tailoring DSLs to specific user requirements or target architectures.
*   **Symbolic Model Checking for Hardware Interaction:** BML provides a mechanism to formally verify hardware peripheral interactions, especially memory-mapped registers, using refinement types combined with symbolic execution and static analysis techniques, reducing the risk of hardware malfunctions.
*   **Customizable String Interpolation:** Customizable string interpolation using `{variable:formatSpecifier}` syntax provides control over output.
*   **Concise Bitfield Operations:** Concise syntax for bitfield and bit range access using `=`, `|=`, `&=`, and `with` statements for combined updates, with support for direct setting, implicit access, and bit masks, improving code readability and flexibility. `with` statements optimize bitfield operations.
*   **Array Initialization:** Range-based initialization provides an easier mechanism to initialize arrays.
*   **Optional Parameter Defaults:** Default values for optional function parameters reduce boilerplate code.
*   **More Flexible Struct and Union Initialization:** Initializer lists with `@initializerList` attribute allow flexible and concise struct/union initialization (positional, by name, or mixed).
*   **Simplified Tuple Declaration:** Implicit tuple declaration reduces verbosity.
*   **Expressive Pattern Matching:** More complex pattern matching improves the readability of complex code.
*   **Idiomatic Error Propagation:** `?` operator for concise error propagation: `value = my_function()?;`.
*   **CAM Composition:** `merge` keyword allows composing CAMs for building complex reusable components.
*   **CAM Customization:** CAM customization via `config` sections and attributes, including `@enable()` and `@disable()`, provides flexible CAM adaptation to user needs.
*   **Static Analysis Optimization:** The compiler analyzes the whole program's context for code optimizations using data flow analysis and code transformation rules defined in the CAM.
*   **Rule-Based Code Refactoring:** An optional tool is provided to perform code refactoring based on static analysis of the code or by using rule-based patterns.
*   **Formal Hardware Verification:** Static analysis tools provide more information about the formal verification process, and it is done by combining refinement types and symbolic execution.
*   **HDL Generated CAM stubs:** Generation of CAM code stubs by parsing a hardware description language file. The user has to fill the implementation of the CAM functions manually.
*   **Optional Parentheses:** Parentheses are optional for single-argument functions or intrinsics, allowing less verbose code.
*   **Low-Level Control through CAMs and DSLs:** Low-level code generation is possible using CAMs and their DSLs. Users must use CAMs to interact with hardware.
*   **Symbolic Execution:** Compiler performs symbolic execution using the `symbolic` keyword, enabling advanced code transformations and formal verification.
*   **Algorithmic Code Generation:** Compiler uses algorithmic code generators when `@algorithm()` attribute is used for optimized code in specific scenarios.
*   **CAM-based Rule Systems:** CAMs can define code transformation rules using `instrDSL` and `match`, `capture`, and `apply` statements.
*   **Configurable DSL Adaptation:** DSLs can be adapted based on user parameters using a `config` block in the DSL definition.
*   **Implicit Type Conversions with `as`:** Type conversions between related data types (integers of the same sign or to floats) using `as` keyword enhance readability while enforcing type safety. Signed to unsigned integer conversions are not implicit.
*   **Inline Conditional Assignments:** Compact syntax using `if(condition) value1 else value2` for conditional assignments.
*   **Short Lambda functions:** Short lambda functions using `parameter => expression` simplify function declarations.
*   **Implicit Register Declarations:** CAMs can omit register type when defining conceptual registers.
*   **Concise Region Definition:** Single-field data regions can be defined as `dataRegion MyData dataType myVariable;`.
*   **Compile-Time String Processing:** Intrinsic functions like `stringLength()`, `stringReverse()`, `stringSub()`, `stringEquals()`, `stringConcat()`, and `stringToInt()` are supported at compile time.
*   **First Class Code Blocks:** Code blocks can be passed as function parameters for improved code composability.
*   **Implicit Return Type for Single Expression Functions:** Single-expression functions can omit return type, inferred from the expression.
*   **Type attributes:** Type attributes can be defined using `#` as a prefix.
*   **Explicit Control and Developer Responsibility:** BML puts control in the hands of the developer. While providing high-level abstraction, BML gives explicit control over low-level aspects. The programmer is responsible for correct feature usage, optimal performance, and safety. Users are responsible for selecting correct CAMs and error handling. The compiler and CAMs attempt to report errors whenever possible.

**BML is *not* intended to be:**

*   A direct instruction-for-instruction assembly replacement at the user level.
*   A language for novice programmers unfamiliar with embedded concepts.

**Limitations:**

*   **No Direct Instruction Encoding (User-Level):** BML does *not* allow direct instruction encoding or manual opcode/operand/addressing mode specification at the user level. CAMs and DSLs generate optimized instruction sequences. Developers relinquish direct control over code generation for a higher-level, maintainable approach. BML is not a direct assembly replacement at the user level. Reliance on CAMs for hardware interaction is mandatory.
*   **Hardware Access Limitations:** BML does not provide direct access to *all* low-level hardware features available in assembly. Developers rely on CAMs for hardware interaction. If a CAM does not expose a feature, it cannot be accessed directly from BML code. BML features are limited by CAM capabilities. Users must use CAMs to interact with specific hardware features. If a feature is not available via the CAM API, users must write or extend a CAM.
*   **Runtime Checks are Optional:** Runtime checks are present and have a performance impact but are optional and can be disabled via compiler flags or pragmas. Disabling runtime checks provides more control but increases the risk of unsafe programs. Tracked pointers and safe arrays perform runtime checks that may impact performance if hardware-assisted bounds checking is disabled or static safety cannot be proven. Users must be aware of safety-performance trade-offs.
*   **Debugging Complexity:** Source-level debugging is supported, but debugging low-level issues involving memory corruption and hardware interactions may be more difficult than assembly-level debugging. Advanced compiler debugging output and CAM debugging features are provided to assist users. Testing CAM implementations in isolation is recommended. Debugging memory-mapped peripherals may be more complex due to the abstraction level and reliance on CAMs for hardware operations.

**2. Core Language Features**

*   **Static Typing:** BML is a statically-typed language, enforcing type rules at compile time to prevent type-related errors, making it easier to reason about the code and reducing the number of runtime errors.

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
        *   `address` represents a generic memory address without any specific region association.
        *   `codeAddress` is used for memory addresses that point to executable code (e.g., function pointers or interrupt handlers).
        *   `dataAddress` is used for memory addresses that point to data (e.g., variables or memory buffers).
        *   `genericAddress` is a general-purpose address that does not imply a specific memory region and can be used as a generic memory address.
        *   `bootAddress` specifically represents the boot address, the initial address where execution begins after reset.
        *   **Default Value**: The default value for all address types is `null` (represented internally as `0`). Dereferencing a null address without a null check will result in *undefined behavior* if runtime null-pointer checks are disabled (and a *potential runtime error* if enabled). The compiler will issue a warning in many cases if it detects a potential null dereference, *unless* the pointer is marked with `!` or `+` (guaranteed non-null).
        *   Example: `dataAddress bufferPtr;`, `codeAddress handlerAddress;`, `bootAddress bootPtr = 0x1000;`
    *   **Boolean:** `boolean` (`true`, `false`).
        *   Represents logical truth values.
        *   **Default Value**: The default value for boolean types is `false`.
        *   Example: `boolean isReady = true;`, `boolean hasError = false;`
    *   **Character Types:** `char8`, `char16`, `char32`, `char`.
        *   `char8` is an 8-bit character type often used for ASCII characters.
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
        *   Example: `uint16 safe [100] sensorReadings;`, `float32 safe [MAX_COEFFICIENTS] coefficients #linearAccessGuaranteed(25);`
        *   **Concise Array Literals:** Arrays can be initialized by using an initializer list between square brackets: `uint32 safe[5] myArray = [1, 2, 3, 4, 5];` or using range initialization: `uint32 safe[10] myOtherArray = [0..9];`. The compiler will infer the type of the array elements from the array literal or the range expression.
        *   **Array Slice Type:** `arraySlice<DataType>` (dynamic views).
            *   Provides a mutable view over an existing `safeArray` or another `arraySlice`. The view does not copy the data; it represents a reference to a portion of the original array which means that modifications on the slice will be reflected in the underlying array.
            *   Bounds checks are performed at runtime, using the slice's bounds (not the bounds of the original array). If the slice is out of the bounds of the original safe array or another slice, a runtime exception will occur.
            *   Multiple slices of the same array are allowed.
            *   Slices can overlap.
            *   Slices are fully compatible with capability-based pointers.
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
        *   **Use Cases:**
            *   `DataType^` should be used only when specific memory manipulation is required and there are no safer alternatives.
            *   When interfacing with hardware that requires explicit memory addresses or when working with existing low-level data structures.
            *   When it is required to circumvent the compiler safety mechanisms and handle low-level memory accesses, such as when accessing hardware directly.
            *   It is recommended to use other pointer qualifiers when possible.
            *   When interfacing with legacy code that is already using raw pointers.
        *   **Caution:** There are no automatic runtime bounds checks or type checks. It is not implicitly convertible to other pointer types. The type signature must match. The user should explicitly convert a `DataType^` pointer to `genericAddress`, `dataAddress` or `codeAddress` if needed, using the `cast` operator.
        *   Pointer arithmetic is allowed, but it's unsafe; the developer is responsible for not overstepping boundaries, otherwise, undefined behavior will occur.
        *   General pointers can be null; if a null pointer is dereferenced, the behavior is undefined. It is highly recommended to use `!`, `@`, or `+` capability qualifiers whenever a pointer can never be null. The compiler will provide warnings about a potential null dereference if the user attempts to access the pointer without checking for null.
        *   **Default Value:** The default value for general pointers is null.
        *   Example: `uint32^ rawPointer;`, `char8^ buffer;`, `uint32+ ptr = &myValue;`
        *   **Capability-Based Pointer Types:** `DataType +`, `DataType -`, `DataType @regionName`, `DataType !`, `DataType scoped`, `DataType tracked`.
            *   `+`: A pointer that cannot be null and it can read/write to the memory location unless region restrictions apply, the equivalent to `DataType^ nonnull`. The compiler guarantees that a pointer with a `+` type qualifier will never be null.
            *   `-`: A pointer that can be null, and it can read/write to the memory location unless region restrictions apply, the equivalent to `DataType^`. If this pointer is dereferenced without checking if the pointer is null, the compiler may generate a warning.
            *   `@regionName`: A pointer associated with a specific data region, enabling region-based memory management and access controls enforced at compile-time. Accessing memory outside the specified region will result in a compile-time error. The `regionName` must be defined as a `dataRegion` (or `rodataRegion`, or `stackRegion`, or `peripheralRegion`, or `moduleDataRegion`).
            *   `!`: A pointer that cannot be null and can only read from the memory location; write operations are not allowed. If a write operation is performed, it will result in a compile-time error. The compiler guarantees that the pointer will never be null and that no write operation will be performed using this pointer.
            *   `scoped`: A pointer that's valid only within its lexical scope. The compiler prevents the pointer from escaping the scope ensuring that it cannot be dangling. This mechanism is mostly implemented at compile time. The compiler will generate errors if the pointer escapes the lexical scope.
            *   **Default value**: Scoped pointers cannot be null.
            *   `tracked`: A pointer that is tracked by the compiler during runtime. Every access to the tracked pointer incurs a runtime check to prevent dangling pointers. The implementation is target-specific but can be done by using metadata to store the scope of the tracked pointer and checking for that metadata in every access. If the `boundsCheckRefinement` is enabled, the runtime check may be optimized by using refinement types with more specific constraints.
            *   **Default Value**: Tracked pointers are initialized with a default null address value. The runtime checks will catch dangling pointers.
            *   Example: `uint32+ sensorDataPtr = &mySensorData;`, `rawbyte- transmitBuffer;` , `uint32@MyRegion regionPtr;`, `uint32! configPtr = &myConfig;` , `uint8 localPtr = &someArray;` (implicit scope), `uint16 tracked sensorDataPtr = &someValue;`
    *   **Scope-Bound Pointer:** `DataType scoped` (lexical lifetime management).
        *   A pointer that's valid only within its lexical scope. The compiler prevents the pointer from escaping the scope, ensuring that it cannot be dangling. This mechanism is mostly implemented at compile time. The compiler will generate errors if the pointer escapes the lexical scope.
        *   The runtime behavior depends on the compiler flags. For increased runtime safety the compiler can generate code to trap or invoke a runtime error handler if a scoped pointer is accessed after its scope has finished (e.g. by using guard pages or similar techniques).
        *   `DataType scoped` cannot be null.
        *   The `scoped` keyword is optional for local variables and can be omitted if the compiler can infer this using static analysis.
        *   **Static Analysis Process for Implicit Scope:**
            *   The compiler performs static analysis to determine if a pointer's lifetime is strictly confined to its lexical scope specifically its function body or a block inside the function.
            *   The compiler analyses the code flow to verify if the pointer is never passed as a parameter to a function or stored inside a struct that outlives the scope of the pointer, and if the pointer is never returned to a caller function. If all the conditions are satisfied the compiler infers that the pointer is scoped and the `scoped` qualifier can be omitted.
            *   If the compiler detects that the pointer is escaping its lexical scope a compile time error will be generated.
        *   **Scope Rules:**
            *   If a `scoped` pointer is stored inside a struct, the struct will inherit the scope of the pointer and can only be declared in the same scope or a scope that does not outlive the scope of the scoped pointer. If the struct is used outside the scope of the scoped pointer, a compile-time error will be generated. This rule also applies to unions and tuples.
            *   A `scoped` pointer cannot be passed as a parameter to a function or be returned by a function unless the `scoped` qualifier is also applied to the return value.
        *   Example:
            ```baremetal
            function void myFunction() {
                uint32 localValue = 10;
                uint32 localPtr = &localValue; // valid pointer (scoped is implicitly added).
                // ...
            } // localPtr is no longer valid
            ```
    *   **Runtime-Tracked Pointer:** `DataType tracked` (runtime checks).
        *   A pointer that is tracked by the compiler during runtime. Every access to the tracked pointer incurs a runtime check to prevent dangling pointers. The implementation is target-specific but can be done by using metadata to store the scope of the tracked pointer and checking for that metadata in every access. If the `boundsCheckRefinement` is enabled, the runtime check may be optimized by using refinement types with more specific constraints.
        *   **Runtime Check Implementation Details:**
            *   **Metadata Approach:** A common implementation involves storing metadata alongside the tracked pointer. This metadata can include information about the allocation's starting address, the size of the allocated region, and a validity flag. Each time the tracked pointer is accessed:
                1. The validity flag is checked. If the flag indicates the pointer is invalid (e.g., it has been deallocated), a runtime error is triggered.
                2. The access address is compared to the allocation's start address and size. If the access is out of bounds of the allocation region, a runtime error is triggered.
                3. The metadata can be updated when the allocation or deallocation occurs.
            *   **Guard Pages:** Some architectures may utilize guard pages (protected memory regions that surround allocated memory) for more efficient checks. Accessing memory outside the allocation will trigger a hardware exception. The operating system or runtime library can trap the exception, detect a dangling pointer access, and trigger a runtime error.
            *   **Compiler-Specific Techniques:** The compiler can also use other techniques such as generating code that performs dynamic range checks on access, but the performance may be slower compared with metadata or guard pages techniques.
        *   The metadata associated with `tracked` pointers is stored in a separate memory region and is managed by the runtime environment.
        *   A common strategy is to store the information of a tracked pointer in a hashmap, associated with the tracked address. The hashmap can be implemented using software or a hardware accelerator if available, to improve its performance.
        *   Metadata includes information about the pointer's allocation scope and validity flags.
        *   Every access to a tracked pointer will incur an overhead as the runtime must perform the validity checks.
        *   When a tracked pointer is used, the runtime performs the following checks:
            *   Check if the pointer is valid (not null and allocated).
            *   Check if the memory access is within the boundaries of the valid region.
        *   These checks may be optimized by using specific hardware support, but there is always some overhead when using tracked pointers.
        *   When `boundsCheckRefinement` is enabled, the compiler can use refinement types to specify extra constraints that limit the range of a tracked pointer and if the compiler can statically prove the pointer access is valid, the runtime checks can be skipped.
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
        *   A fixed-size immutable collection of values which can be accessed by name or by index.
        *   Tuple values are mutable by adding the `~` modifier at declaration time.
        *   **Default value**: The default value for a tuple is the tuple of the default values of its members with no padding between the elements of the tuple. The layout of a tuple is similar to that of a struct with no padding.
        *   Destructuring is supported using the `=` operator or the `from` keyword, which allows you to easily access the tuple's values after declaring a variable of the same tuple type.
        *   Tuples can be initialized using a tuple literal, with implicit mapping by position, where the values in the tuple literal are assigned to the members based on their position in the tuple type.
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
        *   Function pointers are unsafe similarly to raw pointers, meaning that the compiler does not track if the function pointer is a valid code address.
        *   Function pointers are not implicitly convertible to other function pointer types. The type signature must match.
        *   **Default value**: The default value of a function pointer is `null`.
        *   Example: `function(uint32, uint32) -> uint32 MathFunctionType;`
    *   **Struct and Union Types:** `struct` and `union` for composite data structures.
        *   `struct` creates a composite data type where members are laid out in a sequence in memory. The members can be of different types.
        *   If a struct contains only one member curly braces can be removed: `struct MyStruct uint32 value;`
        *   `union` creates a type where different members share the same memory location. Only one member can be active at a given time so care must be taken to not corrupt data when accessing members of a union.
        *   **Default value:** The default value for a struct is the struct where all members are set to their default values. The default value of a union is the union where the first member is active and is set to its default value. The members are initialized in the order they are declared with no padding between them. The members are initialized using the default value of their type.
        *   If the `initializerList` is enabled with the `#initializerList` attribute, structs, and unions can be initialized using implicit initialization syntax.
        *   Structs and unions can be initialized by using implicit mapping, using the order of the members, or by specifying the name of each member in the initializer list.
        *   If a member is not present in the initializer list, it will be initialized with its default value.
        *   Example:
            ```baremetal
            type Point = struct #initializerList { int32 x; int32 y; };
             Point myPoint = { 10, 20 };  // x=10, y=20 (positional)

             type MyStruct = struct #initializerList { uint32 a; boolean b; uint8 c = 10;};
              MyStruct myValue = { 1, true }; // a=1, b=true, c=10, using positional initialization.

            type Config = struct #initializerList { uint32 baud; boolean enable;  uint8 dataBits = 8;};
             Config myConfig = { enable=true, baud=115200 }; // baud=115200, enable=true, dataBits=8, using names for implicit mapping.

              type Config = struct #initializerList { uint32 baud; boolean enable = true;  uint8 dataBits;};
             Config myOtherConfig = { 115200, dataBits = 7 }; // baud=115200, enable=true, dataBits=7, mix of positional and name mapping.

           ```
        *   Example: `struct Point { int32 x; int32 y; };`, `union Packet { uint32 rawData; struct { uint8 highByte; uint8 lowByte; } bytes; };`
        *   **Memory Mapped Structs:** `struct #mapped` (members are volatile by default, unless specified `nonvolatile`).
            *   `#mapped` structs define a memory-mapped region, with an implicit volatility status based on the struct's qualifier unless specified otherwise on a member-by-member basis using `volatile` or `nonvolatile`.
            *   The members are implicitly treated as `volatile` unless the `nonvolatile` qualifier is used. The compiler will not attempt to optimize accesses to `volatile` variables*   If the structure is `volatile`, members are `volatile` unless declared `nonvolatile`.
            *   If the structure is `nonvolatile`, members are `nonvolatile` unless declared `volatile`.
            *   If there is no attribute on the struct, members are volatile unless declared `nonvolatile`.
            *   When using memory-mapped structures, the endianness of the members must be carefully considered; if the architecture is little-endian, a `uint32` member will be accessed byte-by-byte in little-endian order.
            *   Data hazards (multiple reads/writes) are not automatically handled by the compiler. The developer must use memory barriers or other means to manage data hazards in shared memory-mapped regions.
            *   Example: `type GpioRegisters = struct #mapped(0x40000000) { uint32 dataRegister; nonvolatile uint32 configRegister; };`
            *   When the `initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct #mapped(0x40000000) #initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`
            *   Memory-mapped structure access should be performed via an instance of the struct that is mapped to a given base address, for example:
                ```baremetal
                type GpioRegisters = struct #mapped { uint32 dataRegister;  uint32 controlRegister; };
                 function void initGpio(dataAddress base) {
                    GpioRegisters myGpio = base; //maps the memory region
                     myGpio.controlRegister = 1;
                }
                ```
            *   **Bitfield members can be accessed with the `=` operator and initialized by using an initializer list without the `with` statement: `myGpioRegisters.controlRegister = { enableBit = 1, dataBit = someValue };`.**
            *   **Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using bit masks: `myGpioRegisters.controlRegister[0b00001111] = 0b101`.**
            *   Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation.
            *   **"with" statement optimization:** The compiler analyzes the `with` block to check if a specific write or read operation to a memory-mapped register can be avoided. If a register is read and not modified, the write operation can be skipped.
            *   This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access.
            *   Example:
                ```baremetal
                type GpioRegisters = struct #mapped { uint32 controlRegister; };
                GpioRegisters myGpioRegisters = 0x40000000;
                myGpioRegisters.controlRegister = {
                    enableBit = 1,
                    dataBit = someValue
                };
                 myGpioRegisters.controlRegister[0..3] = 0b101;
                 myGpioRegisters.controlRegister[0b00001111] = 0b101;
                  with (myGpioRegisters) {
                         enableBit := 1;
                          dataBit := someValue;
                     }
            ```
        *   **Memory Mapped Registers**: Specific registers can be defined using the following syntax:
        *   `DataType #register("regionName", "stringLiteral")  registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants improving the readability of the code. If the enum is declared with a simple comma-separated list of values curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
        *   **Default Value:** The default value for an enum is the first member of the enum.
        *   Example: `enum ErrorCode Ok, NotFound, AccessDenied;`, `enum ErrorCode { Ok, NotFound, AccessDenied };`
    *   **Thread Handle Type:** `threadHandle`.
        *   A handle to a concurrent task that allows the developer to interact with the task.
        *   **Default value:** The default value for a `threadHandle` is `null`.
        *   Example: `threadHandle myThread;`
    *   **Mutex Type:** `mutex`.
        *   A synchronization primitive used to implement critical sections protecting shared memory from data races, guaranteeing atomic access to shared resources by different threads.
        *   **Default value**: The default value of a mutex is an unlocked mutex.
        *   Example: `mutex myMutex;`
    *   **Semaphore Type:** `sync`.
        *   A synchronization primitive that provides signaling with a counting capability useful for limiting the number of concurrent accesses to a specific resource.
        *   **Default value**: The default value of a `sync` is a semaphore with its counter set to `0`.
        *   Example: `sync mySemaphore(10);`
    *   **Awaitable Type:** `DataType await`.
        *   A type used to represent an asynchronous computation that produces a value of type `DataType`. Can be used in conjunction with the `await` and `yield` keywords.
        *   **Default value**: The default value of an awaitable is implementation specific.
        *   Example: `uint32 await asyncResult;`
    *   **Generic Modules and Functions:** `generic <DataType>`.
        *   Allows code to be parameterized with types which allows creating reusable and type-safe components.
        *   Example: `generic <DataType> module MyModule { ... };`, `generic <DataType> function DataType myFunc (DataType value) -> DataType { ... };`
    *   **Result Type:** `Result<OkType, ErrType>` for explicit error handling.
        *   A type that represents either a successful result (`OkType`) or an error (`ErrType`), improving the clarity of the error-handling code and making the compiler enforce error checking.
        *   Pattern matching or a method to easily get the data value from the `Result` type is implementation-specific.
        *   The user must propagate the error up the call stack and implement a proper error-handling policy. The default value is a result where the Ok value is the default value of `OkType` and the Err value is the default value of `ErrType`.
        *   **Error Handling and Propagation:**
            *   The `Result` type is intended to enforce proper error handling at compile time. The user must explicitly handle the results by checking if there is a valid `Ok` value or an `Err` value. The user can propagate errors using different techniques, such as using a chain of functions that explicitly propagate the errors using the `Result` type.
            *   The `Result` type is designed to improve the code clarity and to provide static checks to avoid runtime errors. The user must use CAMs or user-defined types to implement specific error reporting and handling strategies.
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
            *   **Error Handling Policy:** BML does not enforce a specific error handling policy and it is the user's responsibility to handle errors correctly. If an error is not properly handled or ignored by the user the behavior is implementation specific, but it is usually a program termination.
            *   **Example (Error handling):**
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
                   } else {
                        print("Error:", result.getErr()); //handle the error here
                   }
                }
                ```
        *   Example: `Result<uint32, ErrorCode> myResult;`
    *   **Refinement Types:** `type identifier = baseType where constraint` (type constraints).
        *   Allows to define custom data types with compile-time constraints, adding extra static type checks to the language.
        *   The constraint can be an arbitrary expression, a range or a length constraint, and can also include compile-time functions.
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
            *   **More Detailed Bounds Checks:**
                ```baremetal
                    type SmallBuffer = uint8^ where length < 100;

                    function void processBuffer(SmallBuffer buffer) {
                      // code that uses buffer assuming that the length is less than 100
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
            *   **Type Enforcement:**
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
            *   **Refinement types are fully compatible with other language features** such as type attributes, scoped pointers, and so on, as they are just types with additional constraints:

                ```baremetal
                type validPointer = uint32 tracked where value > 0x1000;

                function void processData(validPointer ptr) {
                      // if the compiler can prove that the access to *ptr will be within the valid range, no runtime checks will be needed.
                      // if the access might be out of the range, a runtime check must be performed.
                        print(*ptr);
                  }
                ```
        *   **Type Level Computation:**
            ```baremetal
              compileTimeFunction function boolean isPowerOfTwo(uint32 x) {
                  if(x == 0) return false;
                 return (x & (x - 1)) == 0;
                }
             type PowerOfTwo = uint32 where isPowerOfTwo(value) == true;
            function void processValue(PowerOfTwo number) {
              // the value is guaranteed to be a power of two.
            }
            ```
        *   Example: `type ValidSensorReading = uint16 where value in range(0..1000);`, `type SmallBuffer = uint8^ where length < 100;`, `type EvenNumber = uint32 where value % 2 == 0;`
    *   **Region-Based Types:** `type identifier = baseType @region(regionName)`
        *   Allows defining types that are associated with a specific data region enabling region-based memory management and access controls enforced at compile time.
        *   **Default value**: The default value of a region-based type is the default value of the underlying base type.
        *   **Memory Access and Region Interaction:**
            *   Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined.
            *   `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions, a CAM will have to explicitly read from one region and write to another region using memory copy functions or by transferring using memory mapped peripherals.
            *   CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters, or other persistent states for that CAM.
        *   Example: `type SecureData = uint32 @region("secureRegion");`
    *   **Capability-Based Regions:** `dataRegion regionName capabilities(read, write) { ... };`
        *   Data regions are used for compile-time region-based memory access controls and memory management, allowing compile-time checking of memory accesses and capabilities, increasing memory safety. If only one member is present curly braces can be removed. `dataRegion MySecureData uint32 secretKey;`
        *   The `capabilities` argument specifies which permissions are available: `read`, `write`, `execute`, `capability_derive`, `peripheral`.
            *   If only `read` or `read, write` capabilities are needed, the `capabilities` keyword can be omitted: `dataRegion @read MySecureData uint32 secretKey;`, `dataRegion DeviceData { uint8 safe[100] buffer; };` (implicit read, write capabilities).
        *   `capability_derive` allows creating new pointers with limited capabilities from a pointer that has the `capability_derive` permission, granting a way to control how a pointer is used within a specific scope or region.
        *   `peripheral` allows to access a memory mapped peripheral using the mechanisms defined by the CAM.
        *   A memory region must be declared using this syntax to be used in `@region(regionName)` pointers, otherwise, the compiler will generate a compile-time error.
        *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe[100] buffer; };`
    *   **Compile-Time Stack-Only Allocation:** `@stackOnly` for stack allocation.
        *   Forces the variable, struct, or array to be allocated in the stack, which avoids the overhead of dynamic memory allocation and provides predictable memory allocation, which is useful for real-time systems.
        *   **Default value**: Stack-only types have the default value of the underlying base type.
        *   Example: `@stackOnly uint32 myStackValue;` , `stackOnly uint8 safe[10] localBuffer;`
    *   **Hardware-Assisted Bounds Checking:** Optional using a compiler flag or pragma if the hardware supports it.
        *   Requires specific hardware support, like memory management units, for actual hardware-based checking of memory accesses, improving security and performance.
        *   If hardware support is not available, a runtime software-based implementation may be used by the compiler.
        *   Example: `#pragma enable_hardware_bounds_checking; uint32 safe[10] myArray;`
    *   **Data-Layout Attributes:** `@packed`, `@aligned(4)`, `@row_major`, `@array_of_structures`.
        *   `packed`: Removes padding from structs, resulting in a smaller memory footprint, at the cost of potential alignment problems which can affect the performance of certain memory accesses, or cause errors on specific architectures. The compiler does not perform any alignment checks.
        *   `aligned(N)`: Aligns the data at a memory address that's a multiple of N (where N is a power of 2), useful to improve memory access performance especially when dealing with SIMD instructions.
        *   `row_major`: Arranges multi-dimensional arrays in row-major order (C/C++ standard) which is important when interfacing with code or data from other languages and libraries. This attribute is useful when performing memory accesses in row-major order. For example when accessing a matrix sequentially, accessing the elements in this order will result in better cache usage. For a 2-dimensional array: `data[row][col]`, when traversing the array, the row will be incremented before the column. If this attribute is not used, the compiler may choose other layout strategies depending on the architecture.
            *   Example: `uint32 safe[10] safe[10] myMatrix @row_major;`
        *   `array_of_structures`: forces the array to have the following structure: `[ { member_1, member_2, ..., member_n}, {member_1, member_2, ..., member_n}, ..., ]`. This forces the data to be laid out in the most cache-friendly way for algorithms where all members of a given struct element are accessed sequentially. For example, when you have an array of structs, if you access all members of a given struct sequentially, this is the most cache-friendly layout. If this attribute is not used, the compiler may choose other layout strategies depending on the architecture.
            *   Example:
                ```baremetal
                type MyStruct = struct {
                    uint32 a;
                    float32 b;
                 };
                MyStruct safe[10] myData @array_of_structures;
              ```
        *   Example: `struct MyStruct #packed { uint8 a; uint32 b; };`, `uint32 safe[100] myArray @row_major;`
    *   **Compile-Time Reflection:** `sizeof` and `isType` intrinsics.
        *   `sizeof DataType`: Returns the size (in bytes) of the specified type at compile time, which allows CAMs to generate optimized code based on the data structure sizes.
        *   `sizeof (DataType)` is also a valid syntax when parentheses are used.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement };`.
        *   Example: `sizeof uint32`, `isType(DataType, "Integer")`, `sizeof(uint32)`,
    *   **Compile-Time Functions:** Code execution during compilation via `compileTimeFunction`.
        *   `compileTimeFunction` functions have specific limitations: they cannot perform dynamic memory allocation and cannot call non `compileTimeFunction` functions, because these restrictions are needed to execute code during compilation.
        *   They can be used for static analysis of the data structures or code, or generate code based on compile time parameters.
        *   Compile-time variables that are declared in a `compileTimeFunction` must be initialized with a compile-time constant value. It is an error to use run-time variables in `compileTimeFunction` functions.
        *   **Debugging Compile-Time Functions:**
            *   Debugging compile-time functions directly is not supported by traditional debuggers (GDB/LLDB) because they execute during the compiler's runtime not the target's runtime.
            *   The compiler may offer options to inspect compile-time variables and function execution during compilation by using debug flags or a compiler API. The user can use the `print` function which can be used to print debugging information during compilation.
            *   The compiler can also provide information about how the code was transformed by a compile-time function using debug flags or an output file format that shows which transformations occurred in each compilation step.
            *   The user can also inspect the generated intermediate representation of the code using compiler flags.
            *   `compileTimeFunction` can call other `compileTimeFunction` functions, as long as no cyclic dependency is generated. The dependency will be checked by the compiler during the compile-time phase.
        *   Example:
            ```baremetal
            compileTimeFunction void myFunc() {
               //Code to be executed during compilation
             print("Hello from compile time function");
            }
            ```
    *   **Data-Driven Code Generation:** CAMs use `hasDataLayoutAttribute()`.
        *   `hasDataLayoutAttribute(DataType, "packed")` returns `true` if the data type has the "packed" layout attribute. This is used by CAMs to make decisions about code generation based on the data structure layout. Possible layout attributes are defined in `enum LayoutAttributeKind { Packed, Aligned, RowMajor, ArrayOfStructures };`.
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
    *   **Linear Access Attributes:** `#linearAccess`, `#linearAccessGuaranteed(size)`.
        *   `#linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations.
        *   `#linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray #linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `#circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer.
        *   The compiler must verify that the size of the buffer is a power of 2, otherwise it is a compile-time error.
        *   Example: `uint32 safe [1024] myBuffer #circularBuffer;`
    *   **Read and Write Once Attributes:** `#readOnce`, `#writeOnce`
        *   The `#readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that.
        *   If the variable or memory region is read more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#readOnce` attribute is violated.
        *   The `#writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that.
        *   If the variable or memory region is written more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
        *   Example: `uint32 counter #readOnce;`, `uint8 safe [10] buffer #writeOnce;`
    *   **Cache Line Aligned Attribute:** `#cacheLineAligned`
        *   Guarantees that the data will be aligned to the size of the processor cache line which enables better cache utilization and access performance, preventing issues related to false sharing.
        *   The size of the cache line depends on the target architecture.
        *   Example: `struct MyStruct #cacheLineAligned { ... };`
    *   **Register and Schedule Attributes:** `#register("stringLiteral", "stringLiteral")`, `#schedule("stringLiteral")`
        *   These attributes provide hints to the CAM about how to assign registers and schedule the execution of a given function. The `stringLiteral` is CAM specific and should be used by the CAM developer to provide additional information to the code generator. For example:
            *   `#register("r0")`: The `r0` string can be used by the ARM CAM to associate this variable to the `r0` register.
            *   `#register("offset0x20")`: The `offset0x20` string can be used by a CAM to access an offset of 0x20 from the address where the variable is located.
            *   `#schedule("low")`: The `low` string can be used to mark that a function should be scheduled with low priority when possible.
        *   Example: `function processData() #register("r1") #schedule("low") { ... };`, `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **No Overlap Guaranteed Attribute:** `#noOverlapGuaranteed`
        *   Guarantees that a given memory region does not overlap with any other memory region, allowing for potential optimizations if the compiler can enforce this at compile time.
        *   It's the programmer's responsibility to ensure that this is true. If two memory regions marked with `#noOverlapGuaranteed` do overlap the behavior is undefined. The compiler does not provide a mechanism to verify this property at compile time.
        *   Example: `uint8 safe[100] array1 #noOverlapGuaranteed; uint8 safe[100] array2 #noOverlapGuaranteed;`
    *   **Vectorization Attributes:** `#vector(N)`, `#vectorAligned`.
        *   The `#vector(N)` attribute enables vectorization of a specific data access, and the compiler is in charge of generating vectorized instructions. `N` is the vector size in bytes.
        *   If `N` does not match the expected vector size of the target architecture, the behavior is undefined.
        *   The `#vectorAligned` attribute guarantees that the data is aligned with the vector size which is useful for SIMD operations.
        *   The vector size depends on the target architecture.
        *   Example: `uint32 safe[100] myVector #vector(16);`, `uint8 safe[1024] bigBuffer #vectorAligned;`
    *   **Function Inlining Attributes:** `#inlineAlways`, `#noinline`, `#inlineIfSmall`.
        *   `#inlineAlways`: Forces the inlining of the function. The compiler will emit an error if the function cannot be inlined.
        *   `#noinline`: Prevents inlining of the function even if optimizations suggest it's beneficial.
        *   `#inlineIfSmall`: Suggests to the compiler to inline the function only if it's considered small.
        *   Example: `function mySmallFunction() #inlineIfSmall { ... };`, `function myBigFunction() #noinline { ... };`, `function myFunction() #inlineAlways { ... };`
    *   **Dataflow Attributes:** `#in`, `#out`, `#inOut`.
        *   These attributes are used in CAMs to track how data is passed through the system, enabling optimizations and compiler checks based on the data flow.
        *   These attributes do not change the code behavior; they are used by CAMs for code generation and analysis.
        *   **Dataflow Example:**
            *   CAMs use the `#dataFlow` attributes to analyze how data is passed through the system and use this information to perform optimizations. For example, an optimization can be to perform computations in a different order to minimize cache misses or to perform memory access in a different order to improve performance.
                *   Example:
                    ```baremetal
                    module DataFlowCam {
                        export {
                            interface intrinsic function void processData(uint32 input #in, uint32 output #out) -> void;
                        }
                        @arc arm {
                           statements {
                               function void processData(uint32 input #in, uint32 output #out) {
                                    // code to analyze the dataflow graph
                                     print("DataFlow: Analyzing the flow of data");
                                   memoryBarrier();  // memory barrier can be moved by the optimizer if it's guaranteed that the read/write operation will always be performed in the right order.
                                  // ... generate the optimized code
                                }
                           }
                      }
                    }
                     module MyModule {
                      #use "DataFlowCam";
                        statements {
                            function void main() {
                                uint32 data = 10;
                                uint32 result;
                                processData(data, result);
                               print("Processed data:", result);

                             }
                        }
                    }
                    ```
        *   Example: `function void processData(uint32 input #in, uint32 output #out) { ... };`
    *   **Transformation Attribute:** `#transform(stringLiteral)`.
        *   This attribute allows transforming the data using target-specific transformations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) #transform("myTransform") { ... };`
    *   **Allocator and Unit Attributes**: `#allocator(stringLiteral)`, `#unit(stringLiteral)`
        *   `#allocator(stringLiteral)` is used to specify the allocator to be used for a specific memory allocation. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   `#unit(stringLiteral)` is used to mark a specific part of a CAM with a specific unit or component, which is used to group related functionalities and perform analysis on a unit basis. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `type MyType = struct #allocator("myAllocator") { ... };`, `module MyCam #unit("myUnit") { ... };`
    *   **Lifetime Attribute:** `#lifetime(stringLiteral)`
        *   The `#lifetime(stringLiteral)` attribute provides a way to assign a lifetime to a pointer which can be tracked by the compiler to prevent dangling pointers. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32^ data #lifetime("myLifetime");`
    *   **Capability Attribute:** `#capability(capabilityList)`
        *   Used to specify the capabilities of a given data type. The `capabilityList` is a list of capabilities (read, write, execute, capability_derive, peripheral) that the data type has.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example `struct Data #capability(read, write, execute) { ... };`
    *   **Limit Attribute:** `#limit(stringLiteral)`
        *   Provides a way to limit a specific resource useful for CAM implementations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `module MyCam { ... config { config maxThreads: uint32 #limit("threadLimit"); } }`
    *   **Register Attribute:** `#register("regionName", "stringLiteral")`.
        *   This attribute is used to specify a mapping to a memory mapped register. The stringLiteral is CAM specific and is intended to provide additional information for code generation. This attribute can only be used with `register` declarations.
        *   This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **Hardware Description Attribute:** `#hardwareDescription(stringLiteral)`.
        *   This attribute allows associating a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `interface intrinsic function void platformInit(uint32 baseAddress) -> void #hardwareDescription("timer.init");`
    *   **Enable and Disable Attributes:** `#enable(stringLiteral)`, `#disable(stringLiteral)`.
        *   These attributes provide a way to enable or disable specific functionalities of a given CAM or a function by using a string parameter that maps to a given boolean configuration variable.
    *   **Algorithmic code generation attribute:** `#algorithm(stringLiteral)`.
        *   This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator instead of relying on traditional code generation techniques.
    *   **Symbolic Execution:** `symbolic` and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        *   The `symbolic` keyword is used to create a symbolic block where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check at compile time.
    *   **Implicit Type Conversions:**
        *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64` but not to `int8`, `int16`, `int32`, `int64`, or `float32` and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed and requires an explicit cast.
        *   The `as` operator is used to perform an explicit type conversion for the following types:
            *   Any integer type to a larger integer type of *the same signedness*. (e.g., `uint8 as uint32`, `int16 as int64`.)
            *   Any integer type (signed or unsigned) to a floating point type (e.g., `uint32 as float32`, `int16 as float64`).
            *   Floating-point types to other floating-point types of different precision (e.g., `float32 as float64`).
            *   `DataType^` to `genericAddress`, `dataAddress` or `codeAddress` and vice versa.
            *   Types that are compatible for implicit type conversion (like a formal parameter of a function that is implicitly compatible).
            *   The compiler will perform a compile-time error if the type conversion is invalid.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion and when returning from functions if the return type is compatible for implicit conversion.
        *   No other implicit type conversions are allowed. Conversions must be explicit by using the `cast` operator or by calling conversion functions from the standard library or provided by the CAMs.
    *   **Implicit Type Inference:** Type inference is used in the following cases:
        *   When the return type of a function is not explicitly specified the compiler will infer the return type by analyzing the `return` statements or by using `void` as default if there is no return statement.
        *   When declaring an array literal the compiler infers the array type from the literal values.
        *   When a variable is assigned an integer literal and the type can be inferred from the context if not an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal the default type is `float32`.
        *   **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
        *   When a local variable is initialized with an expression inside the function scope.
        *   When a register variable is initialized with a given value in the CAM scope.
    *   **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.
    *   **Instruction Type:**
        *   The `instruction<instructionName, [parameters]>` type is used to instantiate low-level hardware instructions.
        *   The `instructionName` should match a previously defined instruction using the `hardwareInstructionDSL`.
        *   The `parameters` are a list of compile-time expressions that can be used to specify the operands of the instruction or any other parameter specific to the instruction.
        *   The `instruction` type is a base type, similarly to the integer, float and rawbyte types, and can be used as the type of a given variable, a function parameter, or a return type of a function.

**3. Modules and the Main File: Structure and Organization**

*   **Modules:** Self-contained code units that encapsulate data and functionality, promoting code reusability and organization.
    *   `module moduleName { ... }`: Declares a module, creating a namespace.
        *   `description "String";`: (Optional) Provides a module description that is used for documentation purposes.
            *   Example: `description "A simple example module";`
        *   `declarations { ... }`: Module-private declarations. Module declarations are private by default.
        *   `statements { ... }`: Module initialization code (optional). This code is executed when the module is first loaded and before the execution of the entry point. Module initialization code can also include `use module` statements.
            *   Example: `statements { print("Module initialized"); #use "MyConfigModule"; }`
        *   `export { ... }`: Defines exported items by declaring them inline (implicitly or explicitly) using the `export` keyword or by listing them inside the block: `functions`, `types`, `constants`, `conceptualRegisters`. This is used to make specific module components visible outside of the module.
            *   Example:
                ```baremetal
                module MyModule {
                    export {
                        functions { myExportedFunction }
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                    statements {
                      const MAX_VALUE = 100; // implicitly exported
                      function  myExportedFunction() { ... }; //implicitly exported
                    }
                    
                    type export MyExportedType = struct { uint32 a; };
                    const export uint32 myExportedConstant = 10; //explicitly exported
                }
                ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module *must* be enclosed within an `@arc` block, targeting a specific architecture. Each module can have multiple `@arc` blocks to support different architectures. **Code outside of an `@arc` block is not permitted within a module and will result in a compile-time error.**
            *   Example:
                ```baremetal
                module MyModule {
                    @arc arm {
                        declarations {
                           // ARM-specific declarations
                        }

                        statements {
                            // ARM-specific initialization code
                           function void myFunction() {
                             // ARM-specific code
                           }
                        }
                    }
                    @arc riscv {
                        declarations {
                            // RISC-V specific declarations
                         }
                         statements {
                            // RISC-V specific initialization code
                            function void myRiscVFunction() {
                              // RISC-V specific code
                            }
                         }
                    }
                }
                ```
        *   **Architecture-Independent Code (Implicit `common` Block):** Code outside of any `@arc` block is considered architecture-independent and will be treated as if it were inside an implicit `common` block. This code will be compiled for all target architectures. The compiler will enforce restrictions on what can be done within this implicit `common` block to ensure portability (e.g., disallowing direct hardware access or architecture-specific intrinsics). CAMs are expected to provide default implementations for common functions (like `print`) for each architecture, and these defaults will be available within the implicit `common` block.
            *   Example:
                ```baremetal
                module MyModule {
                    // Implicit common block (architecture-independent)
                    declarations {
                        // Declarations common to all architectures
                        uint32 commonCounter;
                    }

                    statements {
                        // Code common to all architectures
                        function void commonFunction() {
                            print("This is a common function"); // 'print' provided by CAM
                        }
                    }
                    
                    @arc arm {
                        // ... ARM-specific code ...
                    }

                    @arc riscv {
                        // ... RISC-V specific code ...
                    }
                }
                ```

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope making the exported components of the module visible in the current scope.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200; };`
            *   The `with config` clause is used to customize the CAMs behavior.

*   **Main File:**
    *   **The main file is the entry point of the BML application.**
    *   **It is defined in a separate `.bml` file (typically `main.bml`).**
    *   **The main file does *not* use the `module` keyword.**
    *   **It does *not* contain a `main` function.**
    *   **Instead, each `@arc` block in the main file acts as the entry point for that specific architecture.** The code within the `@arc` block's `statements` block is executed sequentially when the program starts on that architecture.
    *   **It uses the `#use` directive to import modules.**
    *   **Code within the main file can be placed outside of any `@arc` block, making it implicitly architecture-independent (treated as if it were in a `common` block).**
    *   **The main file can also contain `@arc` blocks for architecture-specific code.**
    *   Example (`main.bml`):
        ```baremetal
        #use "MyModule";

        // Implicit 'common' section (optional architecture-independent code)
        // ...

        @arc arm {
            declarations {
                // ... ARM-specific declarations ...
            }
            statements {
                // ... ARM-specific initialization ...
                myFunction(); // Call a function from MyModule
                // ... More ARM-specific code ...
            }
        }

        @arc riscv {
            declarations {
                // ... RISC-V specific declarations ...
            }
            statements {
                // ... RISC-V specific initialization ...
                myFunction(); // Call a function from MyModule
                // ... More RISC-V specific code ...
            }
        }
        ```

**5. Composable Architecture Modules (CAMs)**

*   CAMs are compiler extensions that act as "code generation engines." They allow extending the compiler and generating platform-specific code in a type-safe and modular fashion, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM. (CAMs are modules)
        *   `description "CAM Description String";`: (Optional) A description of the CAM which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can now load external data from a file by specifying the location in the `config` section and using a parsing function.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral;}`
        *   `export { ... }`: Declares architecture-agnostic interface functions via `interface intrinsic function`, which is the main entry point for CAM-provided operations, making them callable from other BML code. It can also export conceptual registers that are used by the CAM code to generate target-specific instructions improving the abstraction of the hardware. The `instrDSL` block is used to declare a domain-specific language for defining low-level symbolic instructions. Symbolic instructions can have generic parameters, which can be used to specify the instruction behavior at compile time. Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time. The `hardwareInstructionDSL` block is used to declare low level hardware instructions by specifying the encoding, side-effects and microcode using a DSL.
            *   Example:
                ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean #enable("dma") = false;
                   }
                    export {
                       interface intrinsic function void platformUartInit(uint32 baudRate) -> void #hardwareDescription("uart.hdl");
                       interface intrinsic function void platformUartSendByte(uint8 data) -> void;
                       interface intrinsic function void platformUartSendString(StringLiteral str) -> void;
                       conceptualRegisters {
                          %uartBaseAddress,
                          %baudRateValue,
                          %byteToSend
                       }
                   }
                    @hardwareInstructionDSL {
                     instruction add(rd, rn, imm) {
                        encoding: "0bxxxxxxx100010xxxyyyyzzzz";
                         rd: "r" + rd;
                         rn: "r" + rn;
                          imm: imm;
                        sideEffect {
                          setFlag(carry) = overflow();
                           setFlag(negative) = rd < 0;
                         }
                         microcode {
                         "read r" + rn + ", dataReg"
                            "add r" + rd + ", dataReg"
                           "write r" + rd + ", dataReg"
                            }
                        }
                        instruction store(address, register) {
                          encoding: "0bxxxxxxyyyyy";
                          address: address;
                           register: register;
                          sideEffect {
                            memoryStore(address, register);
                          }
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
                          rule add_imm match("add(x, immediate(y))", instruction) -> replace(instruction, add_reg(x,y));
                        }
                      statements {
                        function void parseHardwareDescription () {
                         print(f"Parsing {config.hardwareDescription}");
                         // load the data using the config.hardwareDescription and generate new interface intrinsic functions
                        }
                        function void platformUartInit(uint32 baudRate) {
                         register %uartBaseAddress = config.baseAddress + 0x0; //Example UART base address
                           register %baudRateValue = baudRate;
                            memoryStore(dword, %uartBaseAddress + 0x04, %baudRateValue); // Example code to set baud rate
                            print("ARM: Uart Initialized");

                        }
                       function void platformUartSendByte(uint8 data) {
                         register %uartBaseAddress = config.baseAddress + 0x0; //Example UART base address
                         register %byteToSend = data;
                         if (#enable("dma")) {
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
                    }
                   @arc riscv {
                    statements {
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
                }
            Meta module MyMetaCam {
                config {
                config hardwareDescription : StringLiteral;
                    config camName : StringLiteral = "MyGeneratedCam";
             }
             statements {
               @arc any { // the @arc block must be included. The code in this block is executed at compile time.
                  function void generateCam (CompilerContext context) {
                      //read the config values
                       print("Meta Cam generating: " + context.config.camName);
                       //  parse the hardwareDescription file and extract the register information
                       // generate a new CAM
                       StringLiteral interfaceDefinitions = "interface intrinsic function void platformInit(uint32 baseAddress) -> void;";
                      StringLiteral implementation = " function void platformInit(uint32 baseAddress) {  register %myReg = baseAddress; } ";
                      StringLiteral hdl = context.getHardwareDescription("myperipheral.hdl");
                      context.newCAM(context.config.camName, interfaceDefinitions, implementation, hdl);
                   }
                 }
             }
         }

         module MyOtherCam {
           statements {
                #use "UartCam" with config { enableDma = false };
                #use "UartCam" override function platformUartSendByte with config { enableDma = true };
           }
            }
              ```
        *   The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body to perform low-level hardware operations. The type of the register is inferred by the assigned value.
        *   The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        *   The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
        *   The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
        *   The `hardwareInstructionDSL` block allows the developer to define the encoding of a hardware instruction by specifying the bit layout, side effects, operands, and microcode, using a declarative DSL.
        *   Meta-CAMs are used to generate CAMs dynamically at compile time, using a hardware description or a configuration file.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        *   Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality, and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *   The import system will resolve conflicts with the last imported CAM overriding previous definitions. Explicit override mechanisms may be added in the future.
        *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
    *   **Syntax:** `#use "camName" [genericInstantiation];`
        *   Example: `#use "MyBaseUartCam";`, `#use "GenericMathCam<float32>";`
        *   The `genericInstantiation` is similar to the module instantiation syntax: `<typeList>`.
        *   **Conflict Resolution:** When CAMs redefine interface intrinsic functions, the last CAM imported that defines that function will override previous definitions. There is no explicit mechanism to select a specific version of the interface intrinsic functions. The order of imports is important to resolve these conflicts.
        *   **Config Inheritance:** Imported CAMs can access the `config` parameters of the parent CAM. The `config` attributes are merged, and if there are conflicts, the parent CAM configuration overrides the imported CAM configurations. The scope of config parameters is the module where the CAM is used. Config variables are resolved at compile-time, using static analysis.
*   **CAM Composition:**
    **   **Mechanism:** CAMs can be composed by using the `merge` keyword in the `#use` directive: `#use "MyCam1" merge "MyCam2";`.
    *   The `merge` keyword will attempt to combine the interface intrinsic functions of two or more CAMs into a single CAM. If there are conflicts, an error will be generated.
    *   The `config` parameters will be merged, and if there are conflicts, the CAM listed first will have priority.
    *   If a CAM is imported without the merge keyword, it can still be used without merging it with the current CAM.
*   **CAM Customization**
    *   CAMs can be customized by using the `config` section and specific attributes.
    *   A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
    *   The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
    *   If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
        *   CAM options can be enabled or disabled by using the `#enable("myOption")` or `#disable("myOption")` attributes. Example: `function void myFunc() #enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean #enable("myOption") = true; }`

*   **CAM API: Compiler Interface for CAMs**
    *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                    *   The `dataflowGraph` is composed of nodes and edges representing the data flow through the program. Each node can be accessed by using the `nodes` field of the dataflow graph. Each node has metadata and information about the operations that are performed including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges and to perform specific data flow analysis tasks.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    *   Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                *   `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer and so on. See enum `TypeKind` for possible types.
                *   `sizeof(DataType)`: Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                *   **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                    *   Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                *   **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                *   **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                *   **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                    *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                    *   The implementation is a function pointer that must have the same signature as the `functionSignature` string.
                *   **`newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification)`:** Creates a new CAM with the given parameters. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton.
                    *   The `interfaceDefinitions` will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                    *   The `implementation` will be a set of function pointers that define the implementations of the interface intrinsic functions.
                    *   The `hardwareSpecification` parameter is an optional parameter that can specify a file containing information about the hardware that will be used to generate the CAM.
                        ```baremetal
                          @arc any { // the @arc block must be included. The code in this block is executed at compile time.
                             function void generateCam (CompilerContext context) {
                                // read the config values
                                 print("Meta Cam generating: " + context.config.camName);
                                 //  parse the hardwareDescription file and extract the register information
                                 // generate a new CAM
                                 StringLiteral interfaceDefinitions = "interface intrinsic function void platformInit(uint32 baseAddress) -> void;";
                                StringLiteral implementation = " function void platformInit(uint32 baseAddress) {  register %myReg = baseAddress; } ";
                                StringLiteral hdl = context.getHardwareDescription("myperipheral.hdl");
                                context.newCAM(context.config.camName, interfaceDefinitions, implementation, hdl);
                             }
                           }
                      ```
                *   **`addTransformation(Transformation transformation)`:** Adds a transformation to the code which is then performed by the compiler during a code generation phase.
                    *   The `Transformation` object contains:
                    *   **`removeCode`**: The code that should be removed specified as a string.
                    *   **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                    *   **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                    *   **`target`**: A data structure with information about the target including the type of the data being transformed, the memory region, and other metadata about the access allowing the CAM to be more specific when performing code transformations.
                *   **`getHardwareDescription(StringLiteral hardwareDescription)`**: Loads a hardware description language file and returns the content as a String literal that can be used by CAMs.
                *   **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context otherwise `false`.
                *   **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                *   **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                *   **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                *   **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                *   **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                *   **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal.
                *   **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern.
                *   **`createInstruction(StringLiteral instructionName, instructionParameters) -> Instruction`**: Creates a new `instruction` using the given parameters.
                *   **`generateMicrocode(Instruction instruction) -> StringLiteral`**: Generates the low level microcode for a given instruction.
                *   **`addTemplate(StringLiteral sectionName, StringLiteral templateCode);`**: Add a code template at a given section.
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
        *   **`InstructionBuilder` Object:**
            *   Provides an interface for creating symbolic instruction sequences in the `instrDSL` and provides the following methods:
            *   Methods include:
                *   `instructionBuilder.loadRegister(dataAddress address, register reg)`: Creates a `load` instruction for a register.
                *   `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *   `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                *   `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
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

*   **Dataflow Analysis and Optimization:** CAMs can analyze data flow and optimize code using the compiler API.
*   **Type-Specific Optimizations and Specializations:** CAMs can generate code based on specific types.
*   **Hardware-Specific Functionality via DSLs:** CAMs can use DSLs for hardware interaction.
*   **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages using the `compilerError()` function.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access:** Direct access to hardware registers using memory mapped access and conceptual registers.

**6. Directives and Pragmas**

*   **Preprocessor Directives (Start with `#`):**
    *   `#define identifier value`: Macro definition for compile-time variables that are replaced during the preprocessing phase which can improve readability and performance.
        *   Example: `#define MAX_BUFFER_SIZE 1024`
    *   `#if condition`, `#elif condition`, `#else`, `#endif`: Conditional compilation based on compile-time conditions allowing different code paths to be compiled for different configurations.
        *   Condition expressions support macro expansion, boolean logic (`==`, `!=`, `&&`, `||`, `!`) and also integer comparison operators (`>`, `>=`, `<`, `<=`).
        *   The preprocessor expressions only consider integer, boolean, or literal values. There is no type checking of preprocessor expressions. The preprocessor directives are evaluated before all other parts of the code and the order in which they are defined in the file is important. The preprocessor works by text substitution of the values defined with `#define` and by evaluating the `#if`, `#elif` and `#else` blocks.
        *   Example:
            ```baremetal
             #define DEBUG_MODE true
              #if DEBUG_MODE == true
                  print("Debug Mode Enabled");
              #endif
            ```
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion that makes the exported components of the module accessible.
    *   `#use "camName" [ genericInstantiation ] [ "merge", moduleName  { "," , moduleName } ] , [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
            *   Example: `#use "GenericVector<uint32>" as intVector;`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code.
        *   Example: `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        *   Example: `#raw 0x10 0b00010000 16 0o20 "abc";`
    *   `#template(sectionName) { ... }` : This directive allows for injection of a code template into a specific section. The `sectionName` is defined in a CAM.

*   **Operation Directives:**
    *   `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parentheses can be omitted if only one parameter is passed. Example: `print "Hello world";`.
    *   `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier depending on the target platform.
    *   `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *   `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parentheses can be omitted if only one argument is passed.
    *   `hardware_clz(uint32 value) -> uint32;` : Counts the number of leading zeros in the specified value using a hardware-accelerated instruction if available.
    *   `hardware_ctz(uint32 value) -> uint32;` : Counts the number of trailing zeros in the specified value using a hardware-accelerated instruction if available.
    *   `hardware_barrier();` : Generates a hardware specific memory barrier.
    *   **Boot Code Specific:**
        *   `archCodeBlock [ @boot ] identifier @arc architectureName { ... }`: Architecture-specific raw code block for low-level initialization, bootloaders, and other specialized code. The `@boot` attribute specifies that this is the entry point for the boot code. Allows the developer to write low-level code using `goto`, `memoryStore`, `memoryLoad`, `raw`, `register`, and memory access, that can be target-specific. The `identifier` is the name of the code block, that can be referenced using `goto` inside the architecture block. The `@arc architectureName` specifies the target architecture. **Code inside `archCodeBlock` does not follow the normal rules of BML code and is treated specially by the compiler.** This block is only intended to be used to write low-level code that interacts directly with memory and registers. The `@boot` attribute specifies that this is the entry point for the boot code.
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
        *   `#clearInterruptFlag identifier ;`: Clears an interrupt flag, acknowledging a pending interrupt, and allowing the CPU to handle the next interrupt. The interrupt flag identifier is target specific.
        *   `bootRegion regionName identifier;`: Declares that a boot region should be populated using an identifier defined in the boot code region. If two boot regions are defined for the same address, a compile-time error will be generated. Example: `bootRegion bootData myBootData;`
    *   **Memory Mapped Registers:**
        *   `struct #mapped`: Defines a memory-mapped structure where the members are implicitly `volatile` unless marked `nonvolatile`. Allows for direct access to hardware registers. The `#mapped` specifier specifies that the struct is memory mapped. Members are accessed using the `.` operator, and bitfields can be accessed using the `=` operator inside a `with` statement, or directly when assigning to a memory-mapped structure. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using bitmasks: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`, or a mix of the two. Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access. When the `#initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct #mapped(0x40000000) #initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`.
            *   Example:
                ```baremetal
                type GpioRegisters = struct #mapped { uint32 controlRegister; };
                GpioRegisters myGpioRegisters = 0x40000000;
                myGpioRegisters.controlRegister = {
                    enableBit = 1,
                    dataBit = someValue
                };
                 myGpioRegisters.controlRegister[0..3] = 0b101;
                 myGpioRegisters.controlRegister[0b00001111] = 0b101;
                  with (myGpioRegisters) {
                         enableBit := 1;
                          dataBit := someValue;
                     }
            ```
        *   `.` accesses members. If the member is inside a memory-mapped struct, it will access the memory location directly.
        *   **`=` sets/reads bitfields inside a memory-mapped struct.**
        *   **Bit Ranges: `[start..end]` accesses a range of bits in a bitfield inside a memory-mapped struct.**
        *   **Bit Masks: `[binaryMask]` accesses the bits defined by a binary mask in a bitfield inside a memory-mapped struct.**
        *   **Combined Selectors: `[range, bit, mask, ...]` accesses multiple range of bits or individual bits inside a bitfield of a memory-mapped struct.**
        *   Memory Mapped Registers: Specific registers can be defined using the following syntax:
            *   `DataType #register("regionName", "stringLiteral") registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The stringLiteral is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
    *   **Symbolic Execution:**
        *   `symbolic { ... }` block:
            *   The `symbolic` block allows the developer to define a region of code where the compiler will perform symbolic execution. Inside the symbolic block, the compiler will generate symbolic expressions for the operations and variables, which allows to verify program properties and perform advanced code optimizations by using the `CompilerContext` API.
                *   `symbolicResult(identifier)`: Returns the symbolic expression of a variable inside a symbolic block.
                *   `assert(expression)`: Performs a symbolic assertion generating a compiler error if the assertion is false.
                *   `hasSymbolicRegion():` Returns true if there is a symbolic region in the current program context.
                *   `symbolicExpression(identifier)`: returns the symbolic expression for a given identifier in the current program context.

**7. Memory Safety in BML**

*   `DataType scoped`: Zero-overhead dangling pointer prevention using lexical scope. Pointers are valid only within their lexical scope and the compiler ensures that the pointer cannot escape the scope. This is implemented mostly at compile time, but depending on compiler flags runtime checks can be added. The `scoped` keyword is optional for local variables and can be omitted if the compiler can infer this using static analysis.
    *   **Static Analysis Process for Implicit Scope:**
        *   The compiler performs static analysis to determine if a pointer's lifetime is strictly confined to its lexical scope specifically its function body or a block inside the function.
        *   The compiler analyses the code flow to verify if the pointer is never passed as a parameter to a function or stored inside a struct that outlives the scope of the pointer, and if the pointer is never returned to a caller function. If all the conditions are satisfied the compiler infers that the pointer is scoped and the `scoped` qualifier can be omitted.
        *   If the compiler detects that the pointer is escaping its lexical scope, a compile-time error will be generated.
    *   **Scope Rules:**
        *   If a `scoped` pointer is stored inside a struct, the struct will inherit the scope of the pointer and can only be declared in the same scope or a scope that does not outlive the scope of the scoped pointer. If the struct is used outside the scope of the scoped pointer, a compile-time error will be generated. This rule also applies to unions and tuples.
        *   A `scoped` pointer cannot be passed as a parameter to a function or be returned by a function unless the `scoped` qualifier is also applied to the return value.
        *   Example:
            ```baremetal
            function void myFunction() {
                uint32 localValue = 10;
                uint32 localPtr = &localValue; // valid pointer, scoped is implicitly inferred.
                // ...
            } // localPtr is no longer valid
            ```
*   **Capability-Based Pointers:** `DataType +`, `DataType -`, `DataType @regionName`, `DataType !`, `DataType scoped`, `DataType tracked` for compile-time access control and memory management.
    *   `+` pointers can only be used to read and write from memory and cannot be null.
    *   `-` pointers can be null and can be used to read/write from memory.
    *   `@regionName` pointers are associated with a specific data region, enabling access control and isolation. The compiler will issue an error if the pointer accesses memory outside the specified region.
    *   `!` pointers cannot be null and can only read from memory. This pointer will prevent writing into a specific memory location guaranteeing read-only access.
    *   `scoped` pointers are limited to their scope guaranteeing safety in a more localized and restrictive way.
    *   `tracked` pointers have runtime checks to ensure the pointer is not dangling. The runtime checks can be implemented using different mechanisms, such as metadata, guard pages, or specific hardware support.
    *   `DataType! ` is a constant read-only pointer. It will never allow writing to the address. The memory that it points to can change if the underlying memory region is mapped to a volatile or external device for example.
    *   Multiple pointers with different capabilities can point to the same memory location, as long as they have compatible capabilities (e.g. a read-only pointer and a read-write pointer can point to the same location). If a conflict is detected by the compiler a compile time error will be issued.
    *   Example: `uint32! myReadPtr;`, `rawbyte- myWritePtr;`, `uint32@MySecureRegion myRegionPtr;`, `uint32+ myNonNullPtr = &myValue;`, `uint8 localPtr = &myStackValue;` (implicit scope), `uint16 tracked myTrackedPtr = &someValue;`
*   **Static Typing:** Enforces type rules at compile time preventing type-related errors, which makes the code safer and easier to reason about. Type checking is performed by the compiler during the type resolution phase of the compilation process.
*   `DataType safe [Size]`: Runtime bounds checking to prevent buffer overflows and out-of-bounds accesses, improving the security of the code. Every access to a `safe` array will be checked at runtime.
    *   Example: `uint16 safe[100] sensorValues; sensorValues[10] = 5;`
    *   `arraySlice<DataType>`: Provides bounds-checked views over a specific section of an existing array which can be used to improve the safety of memory access. The bounds checking will be performed at runtime and a runtime exception will be raised if the access is out of bounds of the original array or the slice.
        *   Example: `uint32 safe [100] myArray; arraySlice<uint32> mySlice = myArray.slice(0,10);`
*   `DataType^`: Unchecked pointers (use with caution). The developer must handle all potential safety issues related to raw memory access. It is recommended to use other more secure pointer types whenever possible. If the developer uses a raw pointer, they will have to check for null pointers and also perform bounds checking manually.
    *   Example: `uint32^ ptr = &memoryLocation;`
*   `DataType tracked`: Runtime checks to prevent dangling pointers which is especially useful when handling complex memory accesses or concurrency. The tracked pointers have a small overhead that is associated to the metadata required to perform the runtime checks. The runtime checks can be implemented using different mechanisms, such as metadata, guard pages, compiler-specific techniques, or hardware support if available which can reduce the overhead.
    *   Example: `uint32 tracked myTrackedPtr = &someValue;`
*   **Region-Based Memory Management:** Implicit lifetime management through scopes and explicit regions which simplifies memory handling and provides compile-time checks that increase the code safety. Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined. `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions a CAM will have to explicitly read from one region and write to another region, using memory copy functions or by transferring using memory mapped peripherals. CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
*   **Compile-Time Null Dereference Detection:** Static analysis provides warnings for potential null pointer dereferences avoiding runtime crashes due to null pointers. The compiler will issue a warning if a pointer with a `-` modifier or a general pointer using `^` is dereferenced without checking if the pointer is null. The compiler will not perform any null check if the `+` or `!` modifier is used, or if the pointer is a `scoped` pointer as these pointers are guaranteed not to be null.
    *   Example: If a value is known to be potentially null, the compiler will generate a warning if the developer attempts to dereference it without performing a null check.
*   **Refinement Types:** Compile-time data constraints that allow specifying valid ranges or values for variables, further improving type safety by making the code more specific. Refinement types are used by the compiler to perform static analysis and can eliminate the need for runtime bounds checks.
    *   Example: `type validValue = uint16 where value < 100;`
*   **Capability-Based Regions:** Memory access control enforced at compile time (mostly) that allows isolating specific sections of the memory, preventing data corruption by limiting the access capabilities of pointers providing increased safety and security. Memory regions can be defined using different capabilities such as `read`, `write`, `execute`, `capability_derive` and `peripheral` which allows the CAM to perform more fine-grained memory access controls.
    *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe [100] buffer; };`
    *   **Stack-Only Allocation:** `@stackOnly` enforces stack allocation, useful for performance and memory predictability since it avoids the overhead of dynamic memory allocation and provides predictable memory usage. When this attribute is used the memory will be allocated in the stack.
        *   Example: `@stackOnly uint8 myStackVar;`
    *   **Hardware-Assisted Bounds Checking:** Optional, when hardware is available and enabled via compiler flag or pragma, leveraging hardware to improve the performance and reduce the overhead of memory access checks. This is only used for safe arrays and tracked pointers and will generate the appropriate memory access configuration for the target platform.
        *   Example: `#pragma enable_hardware_bounds_checking; uint32 safe[10] myArray;`
*   **Runtime Checks Tradeoffs:** While BML aims for minimal overhead, runtime checks are still present for certain features (e.g., `safe` arrays, `tracked` pointers). These checks help prevent memory errors, but they may have a performance impact. These checks can be disabled by using compiler flags or pragmas. Disabling the runtime checks should be done with caution as it can introduce potential safety issues.
*   **Runtime Checks are Optional:** The runtime checks are optional and may be disabled by using compiler flags or pragmas, such as `boundsCheck=off` or specific pragmas such as `#pragma disable_bounds_check;`. This gives the user more control by explicitly disabling the runtime checks, but it also means that the user can create an unsafe program that can crash due to buffer overflows or dangling pointer accesses.
*   **Cutting-Edge Solution (High-Performance Bitfield Access):** To minimize overhead and maximize performance in bitfield manipulation, especially within memory-mapped registers, BML provides a "with" statement which allows grouping bitfield updates, avoiding multiple read-modify-write operations when there are multiple bitfield updates for the same register. This is a cutting-edge feature designed to provide low overhead and high performance. The compiler can optimize the code by using specialized hardware instructions where available, or by using bitwise operations where specialized instructions are not available. Bitfield members can be accessed with the `=` operator and initialized by using an initializer list, without the `with` statement: `myGpioRegisters.controlRegister = { enableBit = 1, dataBit = someValue };`. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101` or by using a bit mask: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`.
    *   **"with" statement optimization:** The compiler analyzes the `with` block to check if a specific write or read operation to a memory-mapped register can be avoided. If a register is read and not modified, the write operation can be skipped.
        *   This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access.
    *   **Example:**
        ```baremetal
          type UartControlRegister = struct #mapped {
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
                myUartRegisters.baud = 115200; //single bitfield modification using implicit assignment.
                myUartRegisters.controlRegister = {enable=1, baud = 115200}; //bitfield assignment using struct like syntax

               myUartRegisters.controlRegister[0..3] = 0b101;  //accessing a range of bits
               myUartRegisters.controlRegister[0b00001111] = 0b101;  //accessing using a binary bitmask.

           }
         ```
    *   **Optimizations:** The compiler will:
        *   Read the register `myUartRegisters` only once at the beginning of the `with` block.
        *   Modify the bitfields in a local copy of the register.
        *   Perform a single memory write at the end of the `with` block if any of the bitfields was modified.
        *   If the compiler can detect that the values assigned to a bitfield do not change, the write operation can be optimized away.
        *   The compiler can use specific insert/extract instructions when available or generate bitwise operations with no runtime overhead.
*   **Implicit Read optimization:** When a bitfield is read inside the `with` block, but no bitfield is modified, the compiler may skip reading the register. For example:
    ```baremetal
        with (UART2.ccr) {
           uint32 myStatus = readout; //only read, not modified. The read operation may be optimized away if no bitfields are modified
         }
    ```

**8. Multi-Target Architecture Versatility**

*   `@arc architectureName { ... }`: Provides architecture-specific code implementations inside CAMs, modules, and the main file. This is the main method to provide different implementations of the same API depending on the target architecture. The compiler will choose the appropriate code block based on the target architecture specified by the user by using the `-target` flag in the command-line interface. **All code in a BML file must be inside an `@arc` block, except for code that can be considered architecture-independent**.
*   **Architecture-independent code can be placed outside of `@arc` blocks** (implicit `common` block) in both modules and in the main file. The compiler treats this code as if it were inside a `common` block and will compile it for all target architectures, applying necessary restrictions to ensure portability.
*   CAMs implement hardware abstractions with platform optimizations allowing developers to write code once and reuse it on different platforms without sacrificing performance. The abstraction mechanism allows developers to create reusable code that is portable across different architectures but also perform architecture-specific optimizations using the features of a specific target platform.
*   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code. The compiler will use the information of the target CPU to generate specialized code.
    *   The compiler selects CAMs and generates code based on the `-target` flag which can be used to specify the target architecture and CPU.
    *   Example: `#cpuTarget armCortexM4`
*   **Multiple `@arc` Blocks:** A single BML file (including the main file) can contain multiple `@arc` blocks, one for each supported architecture. The compiler selects the appropriate block based on the `-target` flag.

**9. Concurrency and Parallelism**

*   `threadHandle identifier = startThread(functionName);`: Creates concurrent tasks allowing the developer to implement multi-threaded applications. The `startThread()` function creates a new thread. The way that the stack is handled depends on the target architecture and the compiler flags and is implementation-defined. There is no mechanism to kill a thread or to join a thread directly. If required you can use CAMs to implement such functionality.
    *   Example: `threadHandle myThread = startThread(myThreadFunction);`
*   `parallel { ... }`: Structured concurrency allowing the developer to create parallel tasks that execute concurrently. The compiler may map these tasks to different threads depending on the underlying platform and target architecture but the developer has no control over how the tasks are scheduled.
    *   Example:
        ```baremetal
        parallel {
            task1();
            task2();
        }
        ```
*   `await identifier = expression;` / `yield(expression);`: Asynchronous operations using awaitable types, which can be used to implement non-blocking operations and implement cooperative multitasking. The `await` statement will wait for the result of an asynchronous operation without blocking the current thread. The `yield` statement can be used to return the result of the asynchronous operation.
    *   Example: `await myAwaitable = myAsyncFunction(); yield(myResult);`
*   `lock(identifier) { ... }` / `unlock(identifier);`: Critical section access to protect shared memory from data races, guaranteeing atomic access to shared resources by different threads. A mutex is a synchronization primitive used to implement critical sections.
    *   Example: `lock(myMutex) { /* Critical section */ }; unlock(myMutex);`
*   `wait(identifier);` / `signal(identifier);`: Synchronization primitives used to manage access to shared resources and coordinate concurrent tasks.
    *   The `wait` operation may or may not release a lock (depending on implementation details). The `signal` operation does not release the lock. CAMs can offer more control over these primitives using customized implementations.
    *   Example: `wait(mySemaphore); /* Access resource */; signal(mySemaphore);`
*   Default memory consistency is sequential consistency, using `memoryBarrier();` for explicit ordering of memory accesses. There is no implicit memory barrier at the end of `parallel` blocks as the developer must have full control of the memory access and ordering semantics.
    *   **Memory Barriers:**
        *   BML uses sequential consistency as its default memory model which means that by default memory operations appear to execute in the order specified by the program.
        *   When interacting with shared memory volatile variables or peripheral devices the compiler may not preserve the order of the memory accesses due to optimizations or memory hierarchies.
        *   Memory barriers are used to enforce a specific order on memory access operations. When a memory barrier is executed, the CPU guarantees that all previous memory operations are completed before any subsequent operation starts.
        *   The `memoryBarrier();` operation will generate a target-specific instruction that will act as a memory barrier. It is usually not required unless you have a specific memory ordering that the compiler cannot enforce.
        *   Memory barriers should be used when you have multiple threads that access the same memory regions or when you are interacting with memory-mapped devices that have dependencies on the order of the memory operations.
*   `transaction { ... }`: Ensures atomic execution of the code block. The implementation of transaction blocks depends on the target. For a single-core processor the compiler will typically generate code that disables interrupts and system preemption during the execution of the transaction. For a multi-core system, the code will generate a lock or a memory barrier operation which depends on the availability of hardware support. There are limitations on the operations that can be performed within a transaction to ensure atomicity. Calling functions may break the atomicity guarantees and the compiler will try to enforce these rules at compile time, if possible. The behavior of calling functions inside transactions is implementation-specific. The user is responsible for understanding the hardware and software implications of using transaction blocks.

**10. Tooling and Debugging**

*   **Compiler Error Handling**
    *   The compiler reports compile-time errors in the following format (or similar depending on the compiler implementation):
        *   `filename:line:column: error: Error message`
        *   For example: `MyFile.bml:10:20: error: Type mismatch in assignment`.
        *   The compiler may also output a specific code associated with each error to facilitate automated analysis of the error messages.
        *   The CAMs can generate error messages with a specific format, by using the function `compilerError()`. The compiler may provide a CAM API or some other mechanism to guarantee a specific formatting for the error messages.
    *   **Specific Error Conditions:**
        *   **Code Outside `@arc` Block in Modules:** If code that should be inside an `@arc` block is found at the top level of a module (outside of any `@arc` block), the compiler will generate an error.
        *   **Declarations Outside `declarations` Block:** If declarations are found outside of a `declarations` block within an `@arc` block, the compiler will generate an error.
        *   **Statements Outside `statements` Block:** If executable statements or function definitions with bodies are found outside of a `statements` block within an `@arc` block (or outside of a function definition), the compiler will generate an error.
*   **`bmlc` Compiler:** Command-line tool for compiling BML code:
    *   `-target <architecture>`: Specifies the target architecture e.g., `arm`, `riscv` or `x86`.
    *   `-g`: Enables debug information for GDB/LLDB to allow for source-level debugging using standard tools.
    *   `-S`: Outputs assembly code for the specified target which allows for analysis of the generated code.
    *   `-O<level>`: Optimization level (e.g., `-O0`, `-O1`, `-O2`, `-O3`). `-O0` disables optimization, and `-O3` enables all optimizations.
    *   `-I<include_path>`: Adds a directory to the include path to allow the compiler to find modules, libraries, or header files.
    *   `-cam_path <cam_path>`: Adds a directory to the CAM module path to allow the compiler to locate and use CAMs.
    *   `-enableCaching`: Enables caching of CAM outputs to speed up compilation times. By default, this is disabled.
    *   `-enable-hardware-bounds-checking`: Enables hardware-assisted bounds checking using the MMU if available which may reduce or eliminate the overhead of performing runtime bounds checks.
    *   `-debug-ir`: Enables outputting the intermediate representation of the code for debugging purposes.
    *   `-debug-cam`: Enables outputting debug information for the CAMs, allowing developers to understand the CAM's code and transformations.
    *   `-debug-transformations`: Enables outputting information about the code transformations performed by the compiler, which allows the developer to understand how the code is being modified during the compilation process.
    *   `-debug-dsl`: To see the transformations performed by the hardware instruction DSL.
    *   `-debug-meta`: To inspect the state of the meta-CAM execution during compile time.
    *   `-debug-templates`: To inspect the templates that are being injected by the code.
*   **Source-Level Debugging:** With GDB or LLDB (command-line) which allows the user to set breakpoints, inspect variables, and step through the code. It also allows debugging CAMs, and stepping through the CAM code.
*   **Runtime Error Handling:**
    *   Runtime exceptions are generated when an access to a `safeArray` is performed out of bounds, or if the tracked pointer points to an invalid memory region.
    *   The default behavior is target and compiler-specific. It might be implemented by generating a trap instruction or by invoking a runtime exception handler.
    *   The user can override the default error handling mechanism by using CAMs which are responsible to map the error handling mechanism to a specific platform. For example, a CAM can map the error handling to an interrupt handler that can handle specific error codes related to the runtime exceptions generated by BML code.
    *   It is also possible to perform custom error handling by using the `Result` type which forces the user to explicitly check for errors.
*   **`BML Advanced Compiler Guide`:** (Separate document) - Compiler internals and advanced debugging techniques which is intended for advanced users that require deeper insight into the compilation process. The advanced compiler guide also includes more details on how to debug the CAM code and use the output provided by the compiler to debug low-level code transformations.
*   **Debugging Complexity:** While source-level debugging is supported, debugging low-level issues that involve memory corruption and interactions with hardware components may be more difficult in BML compared to the direct observability provided by assembly-level debugging. BML introduces an abstraction layer via the CAMs and the developer must be familiar with how the CAMs interact with the hardware and the BML runtime to fully debug low-level issues. Debugging memory-mapped peripherals may be more complex in BML compared to assembly language because there is no direct instruction level control over the hardware. The user must rely on the CAMs to generate the correct instructions. Debugging runtime exceptions caused by out-of-bounds accesses, or other memory-related errors, may require more effort by the user to analyze the stack traces and to identify the source of the issue. However, the advanced debugging tools and information provided by the compiler help to reduce the overall debugging complexity.

**11. Standard Library and CAM Ecosystem**

*   **Minimal Core Library:** Zero-overhead.
    *   String literals are encoded using UTF-8. Character literals are encoded using the encoding of the character type (e.g. UTF-8, UTF-16, or UTF-32).
    *   Examples:
        *   `uint32 stringLength(StringLiteral str) -> uint32;` (string length calculation). This function computes the length of a given string literal. It can be used for creating loops that iterate over characters in a string literal, or for allocating buffers based on the size of a given string literal. It does not check if the string literal is null terminated and does not support generic string variables. The compiler must evaluate the string length during the compilation process.
        *   `StringLiteral stringReverse(StringLiteral str) -> StringLiteral;` (string reversal). This function creates a new string that is the reverse of a string literal. The compiler must evaluate this at compile time.
        *   `StringLiteral stringSub(StringLiteral str, uint32 start, uint32 length) -> StringLiteral;` (string substring extraction). This function returns a substring of a string literal, and the operation must be performed at compile time.
        *   `boolean stringEquals(StringLiteral str1, StringLiteral str2) -> boolean;` (string equality check). This function checks if two string literals are the same and must be evaluated during compilation.
        *   `StringLiteral stringConcat(StringLiteral str1, StringLiteral str2) -> StringLiteral;` (string concatenation). This function concatenates two string literals.
        *   `uint32 stringToInt(StringLiteral str) -> uint32;` (string to int conversion). Parses a string literal as an unsigned decimal integer and returns the result, which will be evaluated during the compile-time phase. If the string is not a valid unsigned decimal number, it will return a compile-time error. This is a compile-time function that does not support floating-point numbers or signed values.
        *   `StringLiteral stringFormat(StringLiteral format, ...)` (string formatting). Creates a new string literal by formatting a string template using the specified named parameters. The format is a string literal, that may contain placeholders, and the `...` represents named parameters with their values. String interpolation is performed using the syntax `{identifier:specifier}` which uses standard format specifiers such as `x`, `X`, `b`, `d`, `0Nd` , `s` and `c`.
        *   `toUint32(int16 value) -> uint32;` (integer conversion from a 16-bit signed integer to a 32-bit unsigned integer). This function converts a 16-bit signed integer to a 32-bit unsigned integer. It can be used to perform type conversion between integer types. No error checking or overflow checking is performed.
        *   `memoryBarrier();` (explicit memory barrier). This function generates a hardware-specific memory barrier that guarantees that memory operations are performed in the specified order. This function should be used to guarantee memory consistency when interacting with shared memory, or peripherals. It is an empty operation in systems without caches or multiple cores.
        *   **`print(StringLiteral format, ...);`** (prints a string or a variable). This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. The parenthesis can be omitted if only one parameter is passed: `print "Hello";`.
        *   `void exit(int32 code);` (exits the program with a given code). This function exits the program with a given exit code. The exact implementation is platform specific.
        *   `arraySlice<DataType> slice(uint32 safe [Size] array, uint32 start, uint32 length);` (Creates a slice of an existing safe array). This function returns a mutable view of a specified section of an existing array. The start and the length parameters must be inside the array limits.
        *   `void compilerError(StringLiteral message);` (Generates a custom compiler error, used by CAMs to generate more specific error messages). This function generates a compile-time error. This function should only be used in CAMs.
        *   `uint32 countLeadingZeros(uint32 value) -> uint32;` (Counts the leading zero bits). This function counts the number of leading zero bits (starting from the MSB). The result is between 0 and 32.
        *   `uint32 countTrailingZeros(uint32 value) -> uint32;` (Counts the trailing zero bits). This function counts the number of trailing zero bits (starting from the LSB). The result is between 0 and 32.
        *   `uint32 countSetBits(uint32 value) -> uint32;` (Counts the number of bits that are set to 1). This function counts the number of bits that are set to 1 in a 32-bit integer.
        *   `Result<OkType, ErrType> myFunction() -> Result<OkType, ErrType>;`: If there is an error, the function may return `Err`, otherwise it will return `Ok` followed by the data or it may return `Ok()`, if no data is provided.
        *   `void registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation);` (CAM API to register new interface intrinsic function dynamically). This function allows the CAM to register a new interface intrinsic function dynamically that will be visible to other modules. The `functionSignature` is a string that includes the function parameters and return type. The implementation is a function pointer that must have the same signature as the `functionSignature` string.
        *   `void addTransformation(Transformation transformation);` (CAM API to add a new transformation to the code). This function allows the CAM to add code transformations by passing a transformation object. The `Transformation` object contains the code to remove the code to insert the transformation type, and a data structure with information about the target.
        *   `void newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification);` (CAM APIto create new CAMs during compile time). This function allows the CAM to dynamically generate a new CAM with a given name, a set of interface intrinsic functions, a set of implementation functions, and a hardware specification to load from a given file path. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton.
        *   `boolean hasSymbolicRegion()`: Returns true if there is a symbolic region in the current program context.
        *   `StringLiteral symbolicExpression(identifier)`: Returns the symbolic expression for a given identifier in the current program context.
        *   `StringLiteral getAttribute(identifier, StringLiteral attributeName)`: Returns the string literal of a given attribute.
        *   `boolean hasInstruction(StringLiteral instructionName)`: Returns true if an instruction is present in the current code context.
        *   `StringLiteral getInstructionCode()`: Returns the current instruction as a string, which can be used to perform code transformations.
        *   `TargetData instructionTarget()`: Returns information about the current code target.
        *   `StringLiteral capture(StringLiteral pattern, StringLiteral expression, identifier)`: Captures the subexpression based on a given pattern that can be used to perform code transformations.
        *   `boolean match(StringLiteral pattern, StringLiteral expression)`: Checks if the expression matches the pattern specified as a string.
        *   `StringLiteral getHardwareDescription(StringLiteral hardwareDescription)`; This function loads a hardware description language file and returns the content as a string literal, that can be used by CAMs.
        *   `instruction createInstruction(StringLiteral instructionName, instructionParameters) `: Creates an instruction from the hardware DSL specification.
        *   `StringLiteral generateMicrocode(Instruction instruction) `: Generates low level microcode for a specific instruction.
        *   `void addTemplate(StringLiteral sectionName, StringLiteral templateCode);`: Add a code template at a given section.
*   **CAM Ecosystem:** Provides extended and reusable components, hardware drivers, operating system components, algorithms, and data structures. The main mechanism to extend the language is through CAMs. BML relies on CAMs to generate optimized code for a given target architecture. BML does not allow bypassing the CAMs and accessing the hardware directly. If a CAM does not expose a specific hardware feature or if it does not provide access to some specific hardware instruction that feature or instruction cannot be accessed directly from BML code. The developer must request the CAM developer to expose new features. BML relies on CAMs for generating the required code to interact with peripherals. If a CAM does not expose a specific hardware feature it cannot be accessed directly from BML code. The features exposed by BML are limited to the capabilities of the CAMs used in the compilation process. CAMs can be extended to expose new hardware features and to provide different optimizations. If a CAM does not provide some specific feature, the developer can create a new CAM or extend an existing one to expose that functionality. This approach allows one to have a flexible and adaptable system.

**12. Conformance and Language Evolution**

*   **Conforming Compiler:** Adheres to BML Language Specification and generates code that follows the described behavior.
    *   The compiler must generate a warning when a null pointer is dereferenced without checking for null, and also generate a warning when a scoped pointer escapes its scope, if the appropriate flags are enabled.
    *   The compiler must perform all the compile-time checks specified by the language, such as refinement type restrictions, region-based memory access, and other static safety checks.
    *   The compiler must properly manage the cache of CAM outputs and intermediate representations, when the `-enableCaching` flag is specified.
    *   The compiler must report compile-time errors using a clear and concise format. The format for error reporting can vary depending on the compiler implementation. A typical error message should have the file name, line number, and a clear description of the error, and also provide context where the error occurred. The CAMs can generate error messages with a specific format, by using the function `compilerError()`. The compiler may provide a CAM API or some other mechanism to guarantee a specific formatting for the error messages.
    *   **The compiler must generate an error if code is found outside of an `@arc` block in modules.** In the main file, code outside `@arc` blocks is considered architecture-independent (implicit `common` block).
    *   **The compiler must generate an error if declarations are found outside of a `declarations` block within an `@arc` block.**
    *   **The compiler must generate an error if executable statements or function definitions with bodies are found outside of a `statements` block within an `@arc` block (or outside of a function definition in the implicit common block).**
*   **Conforming Program:** Valid syntax, type-correct, uses supported features and compatible CAMs, and follows all the language specifications.
*   **Language Evolution:** Backward compatibility is an important goal, and community participation is welcome.

**13. Appendix: EBNF Grammar**

```ebnf
sourceFile =
  {preprocessorDirectiveStatement | regionDeclarationBlock | moduleDefinition | targetArchitectureBlock | statement | declaration } , eof ;

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
      [ hardwareInstructionDSLBlock ] ,
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
    dataType , [ regionSpecifier ] , identifier, typeAttributeList, ";"
    | "register" ,  conceptualRegisterIdentifier,  "=" , bootOperand , ";"
    ;

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
    | "struct" , [ typeAttributeList ] , identifier ,  "@mapped" , [ "(" , addressExpression , ")" ] ,  "#initializerList"  , "{" ,
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
  | regionBasedType
  | instructionType
   ;
 instructionType = "instruction" , "<" , identifier, { "," , expression } , ">";

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
  | "interruptVectorRegion" , [ regionCapabilitySpecifier ]
    | "CAMDataRegion" , [regionCapabilitySpecifier]
  ;

regionCapabilitySpecifier =
    "capabilities" , "(" , capabilityList , ")" ;

capabilityList =
    capability { "," , capability } ;

capability =
    "read" | "write" | "execute" | "capability_derive" | "peripheral" ;

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
    typeAttributeList , camDataRegionBlockCore;

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
  | unionDefinition
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
  | "#attribute" , "(" , attributeName , [ "," , attributeArgumentList ] , ")"
  | "#boundsCheck"
  | "#noBoundsCheck"
  | "#boundsCheckStatic"
  | "#optimize" , "(" , optimizationAttributeList , ")"
  | "#potentialNull"
  | "#guaranteedNonNull"
  | "#boundsCheck" , ":" , boundsCheckOption
  | "#stackOnly"
  | "#packed"
  | "#aligned" , "(" , integerLiteral , ")"
  | "#row_major"
  | "#array_of_structures"
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
   | "#register" , "(" , stringLiteral , "," , stringLiteral , ")"
  | "schedule" , "(" , stringLiteral , ")"
  | "linearAccessGuaranteed" , "(" , [integerLiteral ] ,")"
  | "noOverlapGuaranteed"
  | "vector" , "(" , integerLiteral , ")"
  | "vectorAligned"
  | "inlineAlways"
  | "noinline"
  | "inlineIfSmall"
   | "#in"
  | "#out"
  | "#inOut"
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
    |  "@algorithm", "(" , stringLiteral, ")"
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
  | rawBinaryDirectiveStatement
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
rawBinaryDirectiveStatement =
   "#raw", binaryEncodingList , ";" ;

binaryEncodingList =
   constantLiteral {  constantLiteral } ;

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
     | "compilerError" , "(" , stringLiteral , ")" ,
| bootCodeStatement
     | symbolicBlock
     | templateStatement
  ;
templateStatement =
    "#template" , "(" , stringLiteral , ")" , "{" , { sourceCharacter } , "}"
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
   | "if" , "(" , expression , ")" , block , [ "else" , block ]
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
  |  "=>" , [ expression | tupleLiteral ] , [";"]; //implicit return for single expression and code block.

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
    logicalOrExpression [ "?" , expression , ":" , conditionalExpression ]
   | ifElseExpression
     ;
ifElseExpression =
    "if" , "(" , expression , ")" , expression , [ "else" , expression ] ;

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
    unaryExpression
   | conversionExpression ;

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
  | tupleLiteral
   | instructionInstantiation ;

instructionInstantiation =
     "instruction" , "<" , identifier, { "," , expression } , ">"
  ;

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
    | primaryExpression , "[" , bitfieldSelector , "]" ,  "=" , expression
    |  identifier , ":=" , expression
  ;
bitfieldSelector =
    bitfieldRangeSelector { "," , bitfieldRangeSelector }
  ;

bitfieldRangeSelector =
     expression , ".." , expression
    | expression
    ;

bitfieldAssignment =
    identifier , "=" ,  expression , "," ;

conversionExpression =
     postfixExpression , "as" , dataType
    | postfixExpression , "into" , dataType
    | postfixExpression , "from" , dataType ;
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
   | [ "f" ] , '"' , { sourceCharacter | escapeSequence | stringInterpolation } , '"'
  | "U" , '"' , { sourceCharacter | escapeSequence } , '"' ;

stringInterpolation =
     "{" ,  expression , [ ":" , identifier ]  , "}" ;

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
hardwareInstructionDSLBlock =
  "@hardwareInstructionDSL" , "{" , { hardwareInstructionDefinition } , "}" ;

hardwareInstructionDefinition =
 "instruction" , identifier, "(" , [ parameterList ] , ")" , "{" ,
  hardwareInstructionBody , "}" ;

hardwareInstructionBody =
  "encoding" , ":" , stringLiteral , ";" ,
   [ "sideEffect" , "{" , { sideEffectStatement } , "}" ] ,
    { operandEncoding } ,
    [ "microcode", "{" , { sourceCharacter } , "}" ] ;

sideEffectStatement =
    "setFlag" , "(" , identifier , ")" , "=" , expression , ";"
  |  "setRegister" , "(" , identifier , ")" , "=" , expression , ";"
   ;
operandEncoding =
    identifier , ":" , typeName , "+" , stringLiteral , ";"
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
     "match", "(" , stringLiteral,  expression, ")" , [matchCondition];
  matchCondition =
     "where" , expression ;
  transformationOperation =
    "replace" , "(" , expression, expression, ")" ;
```
