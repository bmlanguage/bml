
**BML Language Specification - Version 2.1**

**1. Introduction**

Bare Metal Language (BML) is a statically-typed, compiled programming language designed for bare metal and embedded systems development. BML aims to provide a productive, maintainable, and memory-safe alternative to assembly language. BML does not intend to be a direct, instruction-for-instruction assembly replacement at the user level, but it provides mechanisms to achieve a level of control that is comparable to assembly by using low level DSLs. Instead, it abstracts instruction encoding at the user level while providing comparable performance through its Composable Architecture Module (CAM) system, enabling fine-grained hardware control and platform-specific optimizations.

**Key Differentiators of BML:**

*   **High-Level Abstraction with Low-Level Control:** BML offers high-level constructs for improved code organization and readability, while CAMs provide mechanisms to interact directly with hardware peripherals, registers, and memory locations at a level close to assembly but using type-safe and maintainable code.
*   **Explicit Memory Safety Features:** BML integrates multiple layers of memory safety: static typing, refinement types for value constraints, region-based memory management with capabilities, safe arrays with optional runtime bounds checks, and tracked pointers to mitigate dangling pointer issues. Runtime checks are optional and can be selectively disabled for performance-critical sections, placing responsibility for safety on the developer when disabled.
*   **Composable Architecture Modules (CAMs):** CAMs are compiler extensions written in BML itself. They are the cornerstone of BML's architecture versatility and performance. CAMs encapsulate architecture-specific code generation logic, instruction selection, and hardware interaction details, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages. Developers use BML modules and code, while CAM developers create platform-specific implementations, ensuring separation of concerns. CAMs can be composed and customized.
*   **Near-Zero Overhead Design:** BML strives for near-zero runtime overhead, avoiding hidden allocations, dynamic dispatch (unless explicitly requested), and garbage collection. Performance is intended to be comparable to hand-crafted assembly for equivalent functionality when using appropriate CAMs. However, runtime checks, even if optional, do introduce overhead. Developer responsibility for safety is paramount, especially when disabling runtime checks or using raw pointers.
*   **Direct Microcode and ABI Control:** BML introduces `callDSL` and `instrDSL` to provide direct control over function calling conventions, stack management, microcode instructions, ABI specification, enabling fine-grained control over low-level execution details, and providing a method to completely bypass CAMs when needed.
*   **Explicit Control Over Implicit Operations:** BML uses rule-based transformations in `instrDSL` to provide the means to customize implicit operations like function calls, memory accesses, and type conversions.
*   **Ultra-Low Level Code Generation:** By allowing direct access to the compiler IR and using low-level intrinsics, it is possible to generate very fine-tuned code that is comparable to hand-written assembly.
*   **Cycle Accurate Code Generation:** Using specific attributes and low-level intrinsics, developers can have precise control over the timing of code execution.
*   **Compile-Time Customization:** BML allows compile-time flags and pragmas to provide a very granular level of control for code generation and optimization.
*   **Idiomatic Error Propagation:** The `?` operator enables concise error propagation, streamlining error-handling code.
*   **First-Class Code Blocks:** BML now supports first-class code blocks (also known as lambdas), allowing code to be passed as function parameters for enhanced code composability.
*   **Low-Level Optimization**: BML provides primitives to fully control the low-level aspects of the generated code using compiler APIs, code annotation, performance related pragmas, and direct access to microcode and the compilation process.
* **Explicit Memory Access and Control**: BML provides fine-grained control over memory access and cache management, allowing the developer to fine-tune memory access behavior.
*   **ABI Customization**: BML, via the `callDSL`, allows fine-grained customization of calling conventions and the ABI of a given target, giving similar control to assembly.
* **Performance Annotation**: BML allows to add performance hints to code regions to guide the compiler on the code optimization process and also to optimize specific code regions based on those hints.
*   **Concise and Consistent Syntax:** BML strives for conciseness and consistency, avoiding unnecessary verbosity, and using a streamlined syntax with consistent rules.

**Core Principles:**

*   **Zero Overhead:** BML is designed with a zero-overhead philosophy, prioritizing minimal runtime costs and striving for performance comparable to hand-crafted assembly code where applicable and with the correct CAM usage. No hidden allocations, dynamic dispatch (unless explicitly requested), or garbage collection. Compile-time checks and optimizations, hardware capabilities, and CAMs enable this. Runtime checks are optional and can be disabled. Disabling runtime checks is the user's responsibility. The compiler will generate warnings when a `Result` value is ignored, a null pointer is dereferenced, or if a `scoped` pointer escapes its scope if the required flags are enabled.
*   **Low-Level Control:** BML provides mechanisms for direct hardware interaction, precise memory manipulation, and bit-level control via CAMs. Abstracting away direct instruction encoding at the user level, BML, through CAMs, allows for highly optimized code generation close to hand-written assembly. CAMs provide specialized APIs for hardware peripherals, registers, and memory locations. Direct hardware access at the user level is not allowed, unless explicit low-level access primitives are used with the `raw` keyword. Hardware access is done using memory-mapped registers and hardware intrinsics which are controlled directly by the developer.
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
*   **Concise Bitfield Operations:** Concise syntax for bitfield and bit range access using `=`, `:=`, and `with` statements for combined updates, with support for direct setting, implicit access, and bit masks, improving code readability and flexibility. `with` statements optimize bitfield operations.
*   **Array Initialization:** Range-based initialization provides an easier mechanism to initialize arrays.
*   **Optional Parameter Defaults:** Default values for optional function parameters reduce boilerplate code.
*   **More Flexible Struct and Union Initialization:** Initializer lists with `@initializerList` attribute allow flexible and concise struct/union initialization (positional, by name, or mixed).
*   **Simplified Tuple Declaration:** Implicit tuple declaration reduces verbosity.
*   **Expressive Pattern Matching:** More complex pattern matching improves the readability of complex code.
*   **Idiomatic Error Propagation:** `?` operator for concise error propagation: `value = my_function()?;`. If a function returns `Result<OkType, ErrType>`, the `?` operator will propagate an `ErrType` value or return the `OkType` value if the function returns without errors.
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
*  **Compile-Time String Processing:** Intrinsic functions like `stringLength()`, `stringReverse()`, `stringSub()`, `stringEquals()`, `stringConcat()`, and `stringToInt()` are supported at compile time.
*    **Compile-Time String Formatting:** The function `formatString(StringLiteral format, arguments...)` can be used to create new formatted string literals at compile time.
*    **Compile-Time Bitfield Operations:** Functions like `bitExtract`, and `bitInsert` allow manipulating bitfields of an integer at compile time.
*   **Compile-Time Bitmasks:** Constant bit masks can be defined using the `bitmask` keyword and compile-time expressions.
*   **Customizable Literals:** CAMs can register custom literal suffixes to parse and use them in a specific way (e.g., `10_us`).
*   **Compile-Time Enum Literals:** Enums are evaluated at compile time when used in a compile-time context.
*  **First-Class Code Blocks:** Code blocks can be passed as function parameters for improved code composability. Code blocks are defined by a `block` type, which specifies the parameters and return type of a code block that can be created using lambda expressions `(parameters) => expression` or using a `(parameters) => { ... code ... return expression; }` syntax.
*  **Implicit Return Type for Single Expression Functions:** Single-expression functions can omit return type, inferred from the expression.
*   **Type attributes:** Type attributes can be defined using `#` as a prefix.
*   **Implicit scoped pointers:** If the compiler can determine that a pointer variable cannot escape the lexical scope, the `scoped` attribute can be omitted.
*   **Implicit `this` Parameter:** The `this` parameter is automatically added to struct and union method declarations.
*   **Explicit Member Access:** Members of structs or unions can be explicitly accessed using the `@` operator if there is a name collision.
*   **Explicit Control and Developer Responsibility:** BML puts control in the hands of the developer. While providing high-level abstraction, BML gives explicit control over low-level aspects. The programmer is responsible for correct feature usage, optimal performance, and safety. Users are responsible for selecting correct CAMs and error handling. The compiler and CAMs attempt to report errors whenever possible.
*   **Micro-Level Execution Control:** BML allows the developer to fully control the microcode of a given instruction using the `microcode` block inside the `hardwareInstructionDSL`, combined with `instrDSL` transformations, to obtain low-level control on the execution flow of the program.
*   **Explicit Memory Access and Control**: BML provides fine-grained control over memory access and cache management, allowing the developer to fine-tune memory access behavior.
*  **ABI Customization**: BML, via the `callDSL`, allows fine-grained customization of calling conventions and the ABI of a given target, giving similar control to assembly.
* **Performance Annotation**: BML allows to add performance hints to code regions to guide the compiler on the code optimization process and also to optimize specific code regions based on those hints.
*   **Instruction-Level Control:** Using the `instrDSL` block to create customized instructions, and using the `#target` attribute on a given `instr`, and the `#instruction` attribute at function declaration, and by using low level instructions with the `raw` intrinsic.
*   **Fine-Grained Memory Control:** Using memory attributes and new intrinsics for cache management and memory alignment, and explicit memory regions.
*   **Direct Hardware Access:** Direct register bit manipulation using the `bit[]` syntax, MMIO using DSL definitions, instruction-level dispatch using `#target` and `#dispatch` attributes, custom peripherals using DSLs, unchecked intrinsics using the `raw` keyword.
*   **Advanced Concurrency and Real-Time Control:** Using `#affinity`, `#priority` attributes for thread control, using hardware atomic operations and adding `#deadline`, `#period`, `#executionTime`, `#cycleAccurate` to functions, DMA transfers using a specific statement and attributes, interrupt control attributes and hardware timer access through intrinsic functions.
*  **Customizable Literals**:  CAMs can register custom literal suffixes to parse and use them in a specific way, allowing the compiler to use hardware-specific units.

**BML is *not* intended to be:**

*   A direct instruction-for-instruction assembly replacement *at the user level* unless using specific low-level DSLs, a raw instruction, and custom CAMs.
*   A language for novice programmers unfamiliar with embedded concepts.

**Limitations:**

*   **No Direct Instruction Encoding (User-Level):** BML does *not* allow direct instruction encoding or manual opcode/operand/addressing mode specification at the user level unless using an `instrDSL` specification, a `hardwareInstructionDSL` specification, or an `intrinsic raw` instruction definition. CAMs and DSLs generate optimized instruction sequences. Developers relinquish direct control over code generation for a higher-level, maintainable approach. BML is not a direct assembly replacement at the user level, unless the user is an experienced CAM developer. Reliance on CAMs for hardware interaction is mandatory unless raw instructions are used.
*   **Hardware Access Limitations:** BML does not provide direct access to *all* low-level hardware features available in assembly. Developers rely on CAMs for hardware interaction. If a CAM does not expose a feature, it cannot be accessed directly from BML code. BML features are limited by CAM capabilities. Users must use CAMs to interact with specific hardware features. If a feature is not available via the CAM API, users must write or extend a CAM.
*   **Runtime Checks are Optional:** Runtime checks are present and have a performance impact but are optional and can be disabled via compiler flags or pragmas. Disabling runtime checks provides more control but increases the risk of unsafe programs. Tracked pointers and safe arrays perform runtime checks that may impact performance if hardware-assisted bounds checking is disabled or static safety cannot be proven. Users must be aware of safety-performance trade-offs. Disabling runtime checks is the user's responsibility.
*   **Debugging Complexity:** Source-level debugging is supported, but debugging low-level issues involving memory corruption and hardware interactions may be more difficult than assembly-level debugging. Advanced compiler debugging output and CAM debugging features are provided to assist users. Testing CAM implementations in isolation is recommended. Debugging memory-mapped peripherals may be more complex due to the abstraction level and reliance on CAMs for hardware operations.

**2. Core Language Features**

*   **Static Typing:** BML is a statically-typed language, enforcing type rules at compile time to prevent type-related errors, making it easier to reason about the code and reducing the number of runtime errors.

*   **Data Types:**
    *   **Integer Types:** `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`.
        *   Signed integers (`int` prefix) represent positive and negative values, while unsigned integers (`uint` prefix) represent only non-negative values. `int` and `uint` are architecture-dependent and default to `int32` and `uint32` respectively. The numeric suffix specifies the size in bits (e.g., `int32` is a 32-bit signed integer).
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
            *   **Explicit Address Assignment:**
             The `#address(expression)` attribute can place variables, structs, and arrays at specific addresses. The `expression` must be a compile-time constant or a symbol resolved by the linker.
              ```baremetal
                uint32 myVariable #address(0x20001000);
               struct MyStruct #address(0x20002000) { uint32 a; uint32 b; };
                uint32 myBuffer[100] #address(myBufferStartAddress); // symbol resolved at link time
              ```
         *   **Memory Sections:** Data can be placed into named sections using the `#section(stringLiteral)` attribute.
          ```baremetal
          uint32 myData #section("data"); // Place in the 'data' section.
          uint32 myConstants #section("rodata"); // Place in read-only section.
          uint32 myBootCode #section("bootCode");
           ```
          * User-defined memory regions can be defined using the `dataRegion` directive:
           ```baremetal
            dataRegion MyDataRegion uint32 data;
            uint32 myOtherData  @MyDataRegion;
          ```
        *   **Memory Access Control:** The type qualifiers `#noncached`, `#cached`, `#stronglyOrdered`, and `#volatile` provide control over memory access behavior.
           ```baremetal
            uint32 #noncached myPeripheralRegister @MyPeripheralRegion;
             volatile uint32 myData #address(0x1000) #stronglyOrdered;
           ```
       *    **Endianness Control:** The endianness of a data structure can be controlled using the  `#endian(endianness)` attribute:
           ```baremetal
               struct MyStruct #endian(little) { uint32 a; uint32 b; };
           ```
          *   **Byte Order Specification:**
                *   Add an `#byteOrder(order)` attribute to a data structure, where `order` can be `littleEndian` or `bigEndian`. This is in addition to the current `#endian(endianness)` attribute, as `#endian` specified how the structure is accessed, while `#byteOrder` specifies the internal storage.
                    *   The difference is subtle, but if an architecture uses big-endian and the user uses `#byteOrder(littleEndian)`, the layout will be little-endian and when a specific value is read, the memory will be read by converting to a big-endian representation.
                ```baremetal
                     struct MyData #byteOrder(littleEndian) { uint32 value;};
                 ```
                *   The compiler may generate the appropriate byte swapping operations when accessing data with a specific byte order specification.
                *   When accessing a data structure with a specific byte order, a specific CAM is in charge of implementing the required byte swapping operations.
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
        *   Runtime bounds checks are performed on every access. Out-of-bounds access results in a runtime exception (implementation-defined behavior).
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
            *   `DataType^` should be used only when specific low-level memory manipulation is required, when interfacing with hardware or low-level data structures, or when interfacing with legacy code. Safer pointer types should be preferred when possible.
        *   **Caution:** There are no automatic runtime bounds checks or type checks. It is not implicitly convertible to other pointer types. The type signature must match. The user should explicitly convert a `DataType^` pointer to `genericAddress`, `dataAddress` or `codeAddress` if needed, using the `cast` operator.
        *   Pointer arithmetic is allowed, but it's unsafe; the developer is responsible for not overstepping boundaries, otherwise, undefined behavior will occur.
        *   General pointers can be null; if a null pointer is dereferenced, the behavior is undefined. It is highly recommended to use `!`, `@`, or `+` capability qualifiers whenever a pointer can never be null. The compiler will provide warnings about a potential null dereference if the user attempts to access the pointer without checking for null.
        *   **Default Value:** The default value for general pointers is null.
        *   Example: `uint32^ rawPointer;`, `char8^ buffer;`, `uint32+ ptr = &myValue;`
        *   **Capability-Based Pointer Types:** `DataType +`, `DataType -`, `DataType @regionName`, `DataType !`, `DataType scoped`, `DataType tracked`.
            *   `+`: A non-null pointer that can read/write to the memory location unless region restrictions apply, equivalent to `DataType^ nonnull`. The compiler guarantees that a pointer with a `+` type qualifier will never be null; assigning a null value to such a pointer results in a compile-time error.
            *   `-`: A pointer that can be null, and it can read/write to the memory location unless region restrictions apply, the equivalent to `DataType^`. If this pointer is dereferenced without checking if the pointer is null, the compiler may generate a warning.
            *   `@regionName`: A pointer associated with a specific data region, enabling region-based memory management and access controls enforced at compile-time. Accessing memory outside the specified region will result in a compile-time error. The `regionName` must be defined as a `dataRegion` (or `rodataRegion`, or `stackRegion`, or `peripheralRegion`, or `moduleDataRegion`). Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined. `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions, a CAM will have to explicitly read from one region and write to another region, using memory copy functions or by transferring using memory mapped peripherals. CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
            *   `!`: A pointer that cannot be null and can only read from the memory location; write operations are not allowed. If a write operation is performed, it will result in a compile-time error. The compiler guarantees that the pointer will never be null and that no write operation will be performed using this pointer. The memory that it points to can change if the underlying memory region is mapped to a volatile or external device.
           *  `scoped`: A pointer that's valid only within its lexical scope. The compiler prevents the pointer from escaping the scope ensuring that it cannot be dangling. This mechanism is mostly implemented at compile time but may include optional runtime checks based on compiler flags. The `scoped` keyword is optional for local variables when the compiler can statically infer their scope. Scoped pointers cannot be null.
            *   `tracked`: A pointer that is tracked by the compiler during runtime. Every access to the tracked pointer incurs a runtime check to prevent dangling pointers. The implementation is target-specific but can be done by using metadata to store the scope of the tracked pointer and checking for that metadata in every access. If the `boundsCheckRefinement` is enabled, the runtime check may be optimized by using refinement types with more specific constraints. Tracked pointers are initialized with a default null address value. The runtime checks will catch dangling pointers. The metadata associated with `tracked` pointers is stored in a separate memory region and is managed by the runtime environment, often using a hashmap to store the information of a tracked pointer, which can be implemented using software or a hardware accelerator if available. Every access to a tracked pointer will incur an overhead as the runtime must perform the validity checks. When a tracked pointer is used, the runtime performs the following checks: Check if the pointer is valid (not null and allocated) and if the memory access is within the boundaries of the valid region. These checks may be optimized by using specific hardware support, but there is always some overhead when using tracked pointers. When `boundsCheckRefinement` is enabled, the compiler can use refinement types to specify extra constraints that limit the range of a tracked pointer and if the compiler can statically prove the pointer access is valid, the runtime checks can be skipped. The runtime checks for tracked pointers do not include any protection against data races. The user is responsible for protecting the data using the concurrency primitives.
            *   Example: `uint32+ sensorDataPtr = &mySensorData;`, `rawbyte- transmitBuffer;` , `uint32@MyRegion regionPtr;`, `uint32! configPtr = &myConfig;` , `uint8 localPtr = &someArray;` (implicit scope), `uint16 tracked sensorDataPtr = &someValue;`
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

           struct MyOtherStruct { uint32 a; uint32 b }; // positional initialization only
              MyOtherStruct myOtherStruct = { 10, 20 }; //ok positional initialization.
           //   MyOtherStruct myOtherStructError = { b = 20, a = 10}; //error if #initializerList is not present and names are used in the initializer list.
        *   Example: `struct Point { int32 x; int32 y; };`, `union Packet { uint32 rawData; struct { uint8 highByte; uint8 lowByte; } bytes; };`
        *   **Memory Mapped Structs:** `struct #mapped` (members are volatile by default, unless specified `nonvolatile`).
            *   `#mapped` structs define a memory-mapped region, with an implicit volatility status based on the struct's qualifier unless specified otherwise on a member-by-member basis using `volatile` or `nonvolatile`.
            *The members are implicitly treated as `volatile` unless the `nonvolatile` qualifier is used. The compiler will not attempt to optimize accesses to `volatile` variables.
            *   If the structure is `volatile`, members are `volatile` unless declared `nonvolatile`.
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
            *   Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access.
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
                        uint32 myStatus = readout; // readout is read and not modified.
                   }
                  myGpioRegisters.enable = 0; // single bitfield modification using implicit syntax
                 myGpioRegisters.baud = 115200; // single bitfield modification using implicit assignment
                myGpioRegisters.controlRegister = {enable=1, baud = 115200}; //multiple bitfield initialization using struct syntax
            ```
        *   **Memory Mapped Registers**: Specific registers can be defined using the following syntax:
        *   `DataType #register("regionName", "stringLiteral") registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
           * Registers can be defined with access modifiers, using the following syntax:
             * `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
             * `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
             * `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
             * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants improving the readability of the code. If the enum is declared with a simple comma-separated list of values curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
         * Enums can be initialized with specific values using a `struct`-like syntax:
          ```baremetal
           enum ErrorCode { Ok=0, NotFound=1, AccessDenied=2 };
           enum MyEnum { Value1 = 10, Value2 = 20, Value3 = 30 };
          ```
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
        *   The constraint can be an arbitrary expression, a range or a length constraint, and can also include compile-time functions. Refinement types cannot have type parameters.
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
            *   CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
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
        *   `sizeof  dataType`: is also a valid syntax without parentheses.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement, CodeBlock };`.
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
                   statements {
                       function void optimizeData(DataType value) {
                         if (hasDataLayoutAttribute(value, "packed")) {
                            //generate code for packed struct
                         } else {
                         // generate standard code
                         }
                    }
                  }
                }
            }
            ```
    *   **Linear Access Attributes:** `#linearAccess`, `#linearAccessGuaranteed(size)`.
        *   `#linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations or prefetching if combined with `#prefetch` attribute.
        *   `#linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray #linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `#circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer. If combined with the `#prefetch` attribute it can also be used for prefetching.
        *   The compiler must verify that the size of the buffer is a power of 2, otherwise it is a compile-time error.
        *   Example: `uint32 safe [1024] myBuffer #circularBuffer;`
    *   **Read and Write Once Attributes:** `#readOnce`, `#writeOnce`
        *   The `#readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is read more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#readOnce` attribute is violated.
        *   The `#writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is written more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
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
        *   It's the programmer's responsibility to ensure that this is true. If two memory regions marked with `#noOverlapGuaranteed` do overlap the behavior is undefined. The compiler does not provide a mechanism to verify this property at compile  time.
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

    *   **Function Parameter Type Inference:**
        *   BML supports type inference for function parameters when the `#in`, `#out`, or `#inOut` attributes are used.
        *   **`#in`:** The parameter is an input parameter. The compiler will infer its type based on how it's used in the function body. The inferred type will be the type of the expression that is passed as an argument to the function.
        *   **`#out`:** The parameter is an output parameter. Its type must be explicitly specified, as the compiler cannot infer it from the function body alone. The function must assign a value of the specified type to this parameter.
        *   **`#inOut`:** The parameter is both an input and an output parameter. The compiler will infer its type from the argument passed to the function. The function can both read from and assign to this parameter. The argument passed to a function with an `#inOut` parameter must be a variable of a type that is compatible with the inferred type and that can be modified.
        *   **Inference Rules:**
            *   The compiler will analyze the function body to determine the types of `#in` and `#inOut` parameters based on their usage.
            *   If a parameter is used in a way that is incompatible with multiple types, the compiler will generate an error.
            *   If the type of an `#in` or `#inOut` parameter cannot be uniquely inferred, the compiler will generate an error (E0052) and the user must specify the type explicitly.
            *   Type inference does not cross function boundaries. The compiler will not analyze calls to other functions to infer types.
            *   Type inference for function parameters is performed independently for each `@arc` block.

            ```baremetal
            @arc arm {
                statements {
                    function void processData(uint32 data #in, uint32 result #out, uint32 extra #inOut) {
                        result = data * 2;
                        extra~ = extra + result;
                    }

                    function void main() {
                        uint32 x = 10;
                        uint32 y;
                        ~uint32 z = 5;
                        processData(x, y, z); // Type of 'x' and 'z' are inferred as uint32
                    }
                }
            }
            ```
    *   **Transformation Attribute:** `#transform(stringLiteral)`.
        *   This attribute allows transforming the data using target-specific transformations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) #transform("myTransform") { ... };`
    *    **Allocator and Unit Attributes**: `#allocator(stringLiteral)`, `#unit(stringLiteral)`
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
        *  This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
           * Registers can be defined with access modifiers, using the following syntax:
             * `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
             * `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
             * `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Hardware Description Attribute:** `#hardwareDescription(stringLiteral)`.
        *   This attribute allows associating a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `interface intrinsic function void platformInit(uint32 baseAddress) -> void #hardwareDescription("timer.hdl");`
    *   **Enable and Disable Attributes:** `#enable(stringLiteral)`, `#disable(stringLiteral)`.
        *   These attributes provide a way to enable or disable specific functionalities of a given CAM or a function by using a string parameter that maps to a given boolean configuration variable.
    *    **Algorithmic code generation attribute:** `#algorithm(stringLiteral)`.
        *   This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator instead of relying on traditional code generation techniques.
    *   **Symbolic Execution:** `symbolic` and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        * The `symbolic` keyword is used to create a symbolic block where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check at compile time. The `hasSymbolicRegion()` function returns `true` if there is a symbolic region in the current program context, otherwise it returns false, The `symbolicExpression(identifier)` returns the symbolic expression for a given identifier in the current program context.
    *   **Implicit Type Conversions:**
        *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness and from smaller to larger types. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64` but not to `int8`, `int16`, `int32`, `int64`, or `float32` and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed and requires an explicit cast.
        *   The `as` operator is used to perform an explicit type conversion for the following types:
            *   Any integer type to a larger integer type of *the same signedness*. (e.g., `uint8 as uint32`, `int16 as int64`).
            *   Any integer type (signed or unsigned) to a floating point type (e.g., `uint32 as float32`, `int16 as float64`).
            *   Floating-point types to other floating-point types of different precision (e.g., `float32 as float64`).
            *   `DataType^` to `genericAddress`, `dataAddress` or `codeAddress` and vice versa.
            *   Types that are compatible for implicit type conversion (like a formal parameter of a function that is implicitly compatible).
            *   The compiler will perform a compile-time error if the type conversion is invalid.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion and when returning from functions if the return type is compatible for implicit conversion.
        *   No other implicit type conversions are allowed. Conversions must be explicit by using the `cast` operator or by calling conversion functions from the standard library or provided by the CAMs, by using the `into` keyword, by using the `to` method or by using custom suffixes.
    *   **Implicit Type Inference:** Type inference is used in the following cases:
        *   When the return type of a function is not explicitly specified, the compiler will infer the return type by analyzing the `return` statements or generate a compile-time error if no return statement is present and the return type cannot be inferred.
        *   When declaring an array literal the compiler infers the array type from the literal values.
        *   When a variable is assigned an integer literal and the type can be inferred from the context if not an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal the default type is `float32`.
       *   **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
        *   When a local variable is initialized with an expression inside the function scope.
        *   When a register variable is initialized with a given value in the CAM scope.
    *  **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.
        *   **Nested functions:** Functions can be nested within other functions. Inner functions have access to variables in their enclosing function's scope (closure).
         *   Nested functions can only be defined within `@arc` blocks. They are not allowed in the `@arc common` block.
        *   **Shadowing:** Inner scopes can declare variables with the same name as variables in outer scopes. The inner declaration shadows the outer one. The compiler will issue a warning in such cases.
        *   **Global Scope:** Variables declared at the top level of an `@arc` block are considered global for that architecture.
    *   **Instruction Type:**
        *   The `instruction<instructionName, [parameters]>` type is used to instantiate low-level hardware instructions.
        *   The `instructionName` should match a previously defined instruction using the `hardwareInstructionDSL`.
        *   The `parameters` are a list of compile-time expressions that can be used to specify the operands of the instruction or any other parameter specific to the instruction.
        *   The `instruction` type is a base type, similarly to the integer, float and rawbyte types, and can be used as the type of a given variable, a function parameter, or a return type of a function.
        *   Example: `instruction<add, r0, r1, 10> myAddInstruction;`
    *  **Code Block Type:** `block (parameterList) -> returnType;`.
         * A type that defines a function signature for a code block.
         *  **Default value**: The default value of a code block is `null`.
         * Example: `block (uint32, uint32) -> uint32 MathOperationType;`
        * Code blocks can be captured using lambda expressions: `myCodeBlock = (a,b) => a+b;`.
    *    **Bitmask Type:** A constant expression that is used to describe a bitfield using an integer value. Example: `const bitmask ENABLE_BIT = 0x01;`

**3. Modules and the Main File: Structure and Organization (Updated)**

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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope making the exported components of the module visible in the current scope. When using the `#use` directive you can list the specific items that you want to import by using the keyword `only`: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
       *  Module aliases:  Modules can be imported using an alias to avoid namespace conflicts, or when the module name is too long `#use "MyLongModuleName" as MyModule`.
        * Specific versions of the modules can be imported using an `@` syntax. Example: `#use "MyModule@1.2.0";`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
            * Example:
            ```baremetal
                module MyModule {
                    config {
                       config dataSize : uint32 = 16;
                  }
                  statements {
                      function void myFunction(uint32 data) {
                        print("Data size:", config.dataSize);
                      }
                   }
                }
                 module MyOtherModule {
                 statements {
                      #use "MyModule" with config { dataSize = 32 };
                       function void myOtherFunction() {
                          myFunction(10);
                       }
                   }
               }
            ```
            *   In the example above, the function `myFunction` will use the dataSize of `32` which overrides the default `dataSize` value.
         *   **Module Configuration Files:** Modules can load configuration data by specifying a `StringLiteral` that specifies a configuration file, which is loaded and parsed at compile time. The compiler uses the parser that is specified by the type annotation in the configuration section.
          ```baremetal
            module MyModule {
                config {
                  config myConfig : MyConfig =  parseConfigFile("config.txt");
                }
            }
          ```

*   **Main File:**
    *   **The main file is the entry point of the BML application.**
    *   **It is defined in a separate `.bml` file (typically `main.bml`).**
    *   **The main file does *not* use the `module` keyword.**
    *   **It does *not* contain a `main` function.**
    *   **Instead, the `statements` block within each `@arc` block acts as the entry point for that specific architecture.**
     *  **It must have an explicit `@arc common` block to contain architecture-independent code.** Code outside of any `@arc` block is not allowed in the `main.bml` file.
     *  The `@arc common` block is a global scope for all the `@arc` blocks in the `main.bml` file, and code within the `@arc common` block is treated as architecture-independent, meaning that it will be compiled for all target architectures. Direct access to hardware is not allowed in the `@arc common` block. Nested functions are not allowed inside the `@arc common` block, and all function calls are limited to only be able to call other functions inside the `@arc common` block.
    *   **It uses the `#use` directive to import modules.**
    *   The main file can also contain `@arc` blocks for architecture-specific code.
    *   Example (`main.bml`):
        ```baremetal
        #use "MyModule";
         #use "MyGenericCam<uint32>" as GenericUint32Cam;

         @arc common {
            declarations {
                 uint32 commonVar = 5;
            }
            statements {
                 function void commonFunction() {
                  // This function is available to all architectures
                  print("Common function called");
              }
            }
        }


        @arc arm {
            declarations {
                // ... ARM-specific declarations ...
            }
            statements {
                // ... ARM-specific initialization ...
                 commonFunction(); // Can call functions from common section
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
                commonFunction(); // Can call functions from common section
                myFunction(); // Call a function from MyModule
                // ... More RISC-V specific code ...
            }
        }
        ```

**5. Composable Architecture Modules (CAMs) (Updated)**

*   CAMs are compiler extensions that act as "code generation engines." They allow extending the compiler and generating platform-specific code in a type-safe and modular fashion, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM. (CAMs are modules)
        *   `description "CAM Description String";`: (Optional) A description of the CAM which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can load external data from a file by specifying the location in the `config` section and using a parsing function. The `config` parameters can be customized using command line arguments or by using a `with config { ... }` section when the CAM is imported using the `#use` directive. CAM options can be enabled or disabled using the `#enable("optionName")` or `#disable("optionName")` attributes. If the user does not specify a default value for a configuration parameter, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral; config enableDma : boolean #enable("dma") = false; config myOption : CAMOption = {true, 100};}`.
             *  The `config` section can also load a file by specifying the filename in a `StringLiteral` and then processing the file by using a compile-time function that is able to parse the data.
                 * Example: `config { config hardwareDescriptionFile : StringLiteral; config registers : RegisterConfig =  parseRegisterData(hardwareDescriptionFile) ; };` In this example `parseRegisterData` must be a `compileTimeFunction` and `RegisterConfig` a user defined data type.
                *  The `#enable("optionName")` and `#disable("optionName")` attributes can be used to conditionally compile code. Example: `function void myFunction() #enable("myOption") { /*...*/ }` or `config { config enableOpt : boolean #enable("myOption") = true; }`. The `myOption` configuration parameter is defined as `config myOption : CAMOption = {true, 100};`.
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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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
         * The `instrDSL` block can be used to define a domain-specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
         *  The `callDSL` block is used to customize the Application Binary Interface (ABI) of a given function call, by providing a mechanism to specify the calling conventions, the parameter passing mechanism, the stack frame handling and the instructions to setup and teardown a function call.
           *   **`callConvention` Block**: Defines the calling convention for a given architecture, by specifying how the stack is allocated, how the parameters are passed, the register usage, the prologue, the epilogue, and other information about the application binary interface.
             *   **`parameterPassing`:** Specifies how the parameters are passed when calling a function. Parameters can be passed using specific registers or using the stack.
             *  **`returnLocation`:** Specifies how the return value is passed to the caller function.
             *  **`prologue`:** Specifies the code that is executed before a function starts, and usually includes the instructions to save the registers in the stack, and other setup instructions.
             *  **`epilogue`:** Specifies the code that is executed before a function returns, and usually includes the instructions to restore the registers and pop the stack.
             *   **`stackAllocation`:** Defines how the stack memory is allocated and deallocated for a function call.
             *   **`abiName`:** Provides the name of the application binary interface that the compiler must implement when generating the code, which is used to guarantee compatibility with other libraries and code.
            *    **`hardwareInstructionDSL`**: Provides a mechanism to specify the low-level implementation of a hardware instruction.
             * **`encoding`**: Used to specify the bit level layout of the hardware instruction using a string literal.
             * **`sideEffect`**: Used to specify the side effects of an instruction such as flag setting or changes in memory or registers.
             * **`microcode`**: Used to specify the low level microcode implementation of the hardware instruction using a sequence of micro operations specified by string literals.
             * **`latency`**: Used to specify the latency of a given instruction.
        *  Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time.
          * Meta-CAMs can be used to parse instruction set specifications, hardware description files, or other configuration files and then create new CAMs dynamically during compile-time.

     ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean #enable("dma") = false;
                         config functionParameterRegister : StringLiteral #register("r0") = "r0";
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

                    @callDSL {
                     callConvention myCallConvention {
                        parameterPassing {
                           firstParameter  #register(config.functionParameterRegister), // the register will be read from the config section.
                           otherParameters  stack
                       }
                      returnLocation r0,
                       prologue {
                          "push lr"
                       }
                      epilogue {
                          "pop pc"
                       }
                       stackAllocation {
                          allocateStack(frameSize);
                           freeStack(frameSize);
                       }
                     abiName "arm_std_abi"
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
                         memoryStore(dword, sp + 0x10, rd);
                         }
                         microcode {
                         "read r" + rn + ", dataReg"
                            "add r" + rd + ", dataReg"
                           "write r" + rd + ", dataReg"
                            }
                           latency = 2;
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
                       function void platformUartSendByte(uint8 data)  #callConvention("myCallConvention"){
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
        *  The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body to perform low-level hardware operations. The type of the register is inferred by the assigned value.
        *   The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        *   The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
        *  The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
        *   The `hardwareInstructionDSL` block allows the developer to define the encoding of a hardware instruction by specifying the bit layout, side effects, operands, and microcode, using a declarative DSL.
        *   The `callDSL` block allows the developer to specify the application binary interface of a target architecture.
        *   Meta-CAMs are used to generate CAMs dynamically at compile time, using a hardware description or a configuration file.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        *   Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality, and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *    The import system will resolve conflicts with the last imported CAM overriding previous definitions.
         *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
        *   When a CAM is imported using `override function`, the new definition will override the previous one. The previous definition is discarded and not accessible. A compile-time error will be generated if the CAM attempts to use the function definition that is being overridden.
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
    *    A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
    *   The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
     *  If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
        *   CAM options can be enabled or disabled by using the `#enable("myOption")` or `#disable("myOption")` attributes. Example: `function void myFunc() #enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean #enable("myOption") = true; }`

*   **CAM API: Compiler Interface for CAMs**
    *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies. The `dataflowGraph` is composed of nodes and edges representing the data flow through the program. Each node has metadata and information about the operations that are performed including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges and to perform specific data flow analysis tasks.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    *   Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                *  `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer and so on. See enum `TypeKind` for possible types.
                *    `sizeof(DataType)`: Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                *   **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                    *   Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                *   **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                *   **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                 *    **`compilerWarning(StringLiteral message)`:** Generates a custom compiler warning message, allowing for more specific warning reporting.
                *    **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                     *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                    *  The implementation is a function pointer that must have the same signature as the `functionSignature` string.
                *   **`newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification)`:** Creates a new CAM with the given parameters. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton.
                     *  The `interfaceDefinitions` will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                      * The `implementation` will be a set of function pointers that define the implementations of the interface intrinsic functions.
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
                     *  **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                    *   **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                     *   **`target`**: A `TargetData` structure with information about the target including the type of the data being transformed, the memory region, the volatility state, and other metadata about the access.
                        *   **`TargetData`**: The TargetData structure provides information about the target, including:
                            *   `data`: The data type of the target.
                            *   `address`: The memory address of the target if it's a memory access operation.
                            *   `region`: The memory region of the target.
                            *   `isVolatile`: Returns true if the data is volatile.
                            *   `isRegister`: Returns true if the access is a register access.
                             *  `isMemoryMapped`: Returns true if the target is memory-mapped.
                             *   `hasAttribute(StringLiteral attributeName)`: Returns true if the target has a specific attribute.
                *   **`getHardwareDescription(StringLiteral hardwareDescription)`**: Loads a hardware description language file and returns the content as a String literal that can be used by CAMs.
                *   **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context otherwise `false`.
                *   **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                 *  **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                 * **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                  *  **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                *  **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                *   **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal. The pattern language uses the following syntax:
                    *   The pattern language supports the following syntax:
                      * `identifier`: Matches any identifier.
                       *  `stringLiteral`: Matches a specific string literal.
                       *  `integerLiteral`: Matches a specific integer literal.
                       *  `"..."`: Matches a sequence of characters that are present in the code.
                       *  `variable(identifier)`: Matches any variable and captures the identifier.
                        *   `immediate(identifier)`: Matches any immediate value and captures the identifier.
                       *   `instruction`: Matches any instruction.
                      *  `add(x, y)`: Matches an add instruction with two arguments.
                        *   `functionCall(x)`: Matches a function call to the function named `print`.
                         * `*`: Matches anything (wildcard).
                        *  `()`: Creates a group for the match.
                    *    Example:
                        * `"add(x, y)"`: This will match an add instruction with two arguments.
                        *   `"load(address)"`: Matches a load instruction with a parameter named `address`.
                        *    `"functionCall(print)"`: Matches a function call to the function named `print`.
                *   **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern. The captured subexpression will be returned using a string literal, that can be used in other code transformations. The same pattern language as for `match` is used to perform the capture.
                    *    Example: `capture("add(x, y)", code, "myVariable")` if `code` is `add(r0, r1)`, it will return a string `"r0,r1"` that can be used in other code transformations.
                *  **`createInstruction(StringLiteral instructionName, instructionParameters) -> Instruction`**: Creates a new `instruction` using the given parameters.
                  *  **`generateMicrocode(Instruction instruction) -> StringLiteral`**: Generates the low level microcode for a given instruction.
                   *  **`addTemplate(StringLiteral sectionName, StringLiteral templateCode);`**: Add a code template at a given section.
                   *  **`hasHardwareSymbolicSupport() -> boolean`:** returns `true` if the target platform provides hardware support for symbolic checks.
                    *  **`dispatchSymbolicCheck(StringLiteral operation, expression)`:** Dispatches a symbolic check operation to the hardware. The type of operation can be specified by the `operation` string.
                    *   **`getSymbolicCheckResult(address) -> boolean`:** Returns the result of the symbolic check given by the `address`.
                     *   **`hasHardwarePrefetchSupport() -> boolean`:** returns `true` if the target platform provides hardware support for prefetching.
                    *   **`dispatchPrefetch(dataAddress address, size) -> void;`**: dispatches a prefetch instruction to a specific memory location. The `size` parameter indicates how much data should be prefetched from the specified `address`.
                     *    **`getInstructionSchedulingHints(codeBlock) -> List<InstructionSchedulingHint>;`**: Gets a list of hints for instruction scheduling based on data dependencies using symbolic information. The codeBlock is the code region that should be analyzed by the CAM.
                    *   **`addTransformation(StringLiteral removeCode, StringLiteral insertCode, StringLiteral transformationType);`**: Add a code transformation that will remove the code that matches the `removeCode` String literal and will insert `insertCode`, and will use the `transformationType` to perform a specific transformation.
                    * **`getCompilerFlag(StringLiteral flagName)`:** Get the value of a compiler flag.
                      * **`setCompilerFlag(StringLiteral flagName, StringLiteral flagValue)`:** Set the value of a compiler flag for a given region or function.
                    * **`emitCode(StringLiteral code, TargetData targetData)`**: Allows the CAM to directly inject code at a specific target location.
                    * **`skipTransformation();`**: Skips all further transformations for a specific code block.

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
                *  `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *    `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                 *    `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
                 *   `instructionBuilder.atomicRead(address, dataType,  bitmask)`: Creates an atomic read instruction.
                *  `instructionBuilder.atomicWrite(address, value, dataType, bitmask)`: Creates an atomic write instruction.
                  *   `instructionBuilder.atomicTestAndSetBit(address, bitNumber)`: Creates an atomic test and set bit instruction.
                   *   `instructionBuilder.atomicTestAndClearBit(address, bitNumber)`: Creates an atomic test and clear bit instruction.
                *   `instructionBuilder.atomicSetBit(address, bitNumber)`: Creates an atomic set bit instruction.
                 *   `instructionBuilder.atomicClearBit(address, bitNumber)`: Creates an atomic clear bit instruction.
                 *  `instructionBuilder.atomicToggleBit(address, bitNumber)`: Creates an atomic toggle bit instruction.
                *  `instructionBuilder.memoryBarrier(order)`: Creates a memory barrier instruction.
                *  `instructionBuilder.instructionBarrier(sync)`: Creates an instruction barrier instruction.
                 *  `instructionBuilder.rawInstruction(instructionPattern)`: Emits a raw instruction.
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
*  **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages using the `compilerError()` function.
 *  **Custom Warnings and Information Messages:** CAMs can generate customized warning and information messages.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access:** Direct access to hardware registers using memory mapped access and conceptual registers.
*   **ABI Customization**: CAMs can customize the function call ABI using the `callDSL` block.
*   **Microcode Specification:** The microcode of an instruction can be specified using the `microcode` block.
*  **Performance Optimization**: Code regions can be tagged with performance attributes, to guide the compiler optimizations using a target specific implementation.
*  **Code Generation Bypass**: CAMs can bypass code transformations and provide a low-level implementation when necessary using the `emitCode` and `skipTransformation` compiler API functions.

**6. Directives and Pragmas (Updated)**

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
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion that makes the exported components of the module accessible. You can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
     *    Specific module versions can be imported using the `@` syntax. Example: `#use "MyModule@1.2.0";`
    *   `#use "camName" [ genericInstantiation ] [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code.
        *   Example: `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        *   Example: `#raw 0x10 0b00010000 16 0o20 "abc";`
    *   `#template(sectionName) { ... }` : This directive allows for injection of a code template into a specific section. The `sectionName` is defined in a CAM.
     *  `#performance(level, hint, ...)`: A pragma that provides performance information to the compiler.
           * `#performance(critical, memoryAccess, loopUnroll=4, ...)`: example.
            * The `level` is a string literal that can be used to specify the level of performance optimization such as `critical`, `high`, `normal` or `low`, and will be CAM specific, and may have different meanings for different target architectures.
            * The `hint` is a string literal that can be used to provide specific hints for a given code section such as `memoryAccess`, or `loopUnroll`.

*   **Operation Directives:**
        *  `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parentheses can be omitted if only one parameter is passed. Example: `print "Hello world";`.
       * `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier depending on the target platform.
        * `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *   `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parentheses can be omitted if only one argument is passed.
        *   `atomicSetBit(register, bitNumber)`: Atomically sets a specific bit in a register.
        *   `atomicClearBit(register, bitNumber)`: Atomically clears a specific bit in a register.
        *   `atomicToggleBit(register, bitNumber)`: Atomically toggles a specific bit in a register.
        *   `atomicTestAndSetBit(register, bitNumber)`: Atomically tests a bit and sets it if clear. Returns true if the bit was clear before the operation.
        *   `atomicTestAndClearBit(register, bitNumber)`: Atomically tests a bit and clears it if set. Returns true if the bit was set before the operation.
        *   `atomicReadRegister(register, bitmask)`: Atomically reads a value from a register using a specific bitmask.
        *   `atomicWriteRegister(register, value, bitmask)`: Atomically writes a value to a register, applying a given bitmask.
      * `memoryBarrier(order);`: Generates a hardware specific memory barrier. The `order` can be `read`, `write`, `readWrite`, or `all`. If no parameter is specified, it defaults to `all`.
      *   `instructionBarrier(sync);`: Generates a hardware specific instruction barrier. The `sync` can be `instructionFetch`, `dataFetch`, or `dataWrite`.
       * `rawInstr(instructionPattern);`: Embeds raw machine code into the generated output. The `instructionPattern` is CAM specific.
       *  `hardwareSymbolicCheck(expression, range) -> void;` : Performs a hardware-assisted check if the expression satisfies the constraints given by the range. If no hardware support is present, the code will trap.
        *   `hardware_clz(uint32 value) -> uint32;` : Counts the number of leading zeros in the specified value using a hardware-accelerated instruction if available.
        *   `hardware_ctz(uint32 value) -> uint32;` : Counts the number of trailing zeros in the specified value using a hardware-accelerated instruction if available.
        *  `hardware_startTimer();` : Starts a hardware timer.
         * `hardware_endTimer();` : Reads the current value of the hardware timer.
         * `hardware_resetTimer();`: Resets the hardware timer.
          * `hardware_setWatchpoint(address, type);`: Sets a hardware watchpoint that can be used for debugging purposes. The type of the watchpoint can be `read`, `write`, or `readWrite`.
          * `prefetch(address, size, level);`: Dispatches a hardware prefetch instruction for a given `address` with a specific `size` and `level`.
          * `hardware_trap();`: Generates a hardware trap for debugging purposes when a low level error is detected.
        * `cacheFlush(address addr, size size, level level);` Flushes data from the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
        * `cacheInvalidate(address addr, size size, level level);` Invalidates data in the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
        *   `align(address, alignment);`: Aligns a given address to a specific alignment.
    *  **Atomic Operations:**
        *   `atomic { ... }`: Guarantees atomic execution of the code block using the primitives specified in the CAM. If no argument is specified, the compiler will attempt to atomically execute the instructions within the given block, which may involve software locking mechanisms if hardware primitives are not available, which might create an overhead depending on the target architecture and the compiler options. The code block is limited to low level access primitives, and calls to CAM provided `intrinsic` functions that can be mapped to target-specific atomic primitives.
        *   `atomic (variable) { ... }`: Guarantees atomic access and modification of the specified `variable` using the underlying primitives provided by the CAM. The scope is limited to operations involving that specific variable. The compiler will attempt to generate code to atomically read, modify and write to that specific memory location or variable. If the hardware doesn't have a direct atomic read-modify-write instruction the CAM may have to generate code that involves locks and memory barriers to ensure atomicity.
 *  `atomic memoryStore(...)`: Performs an atomic memory store operation.
         * `atomic register identifier = ...;`: Performs an atomic write to a given register using an atomic instruction when it is available.
    *   **Memory Mapped Registers:**
        *  `struct #mapped`: Defines a memory-mapped structure where the members are implicitly `volatile` unless marked `nonvolatile`. Allows for direct access to hardware registers. The `#mapped` specifier specifies that the struct is memory mapped. Members are accessed using the `.` operator, and bitfields can be accessed using the `=` or `:=` operator inside a `with` statement, or directly when assigning to a memory-mapped structure. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using bitmasks: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`, or a mix of the two. Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access. When the `#initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct #mapped(0x40000000) #initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`.
         * Example:
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
                       uint32 status = readout; //no memory read here as the bitfield is not modified
                   }
                 myGpioRegisters.enable = 0; // single bitfield modification using implicit syntax.
                 myGpioRegisters.baud = 115200; // single bitfield modification using implicit assignment.
                  myGpioRegisters.controlRegister = {enable=1, baud=115200}; //bitfield assignment using struct like syntax
            ```
        *  `.` accesses members. If the member is inside a memory-mapped struct, it will access the memory location directly.
        *   **`=` sets/reads bitfields inside a memory-mapped struct.**
        *   **Bit Ranges: `[start..end]` accesses a range of bits in a bitfield inside a memory-mapped struct.**
        *   **Bit Masks: `[binaryMask]` accesses the bits defined by a binary mask in a bitfield inside a memory-mapped struct.**
        *   **Combined Selectors: `[range, bit, mask, ...]` accesses multiple range of bits or individual bits inside a bitfield of a memory-mapped struct.**
        *   Memory Mapped Registers: Specific registers can be defined using the following syntax:
            *   `DataType #register("regionName", "stringLiteral") registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
             * Registers can be defined with access modifiers, using the following syntax:
               *  `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants improving the readability of the code. If the enum is declared with a simple comma-separated list of values curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
         * Enums can be initialized with specific values using a `struct`-like syntax:
          ```baremetal
           enum ErrorCode { Ok=0, NotFound=1, AccessDenied=2 };
           enum MyEnum { Value1 = 10, Value2 = 20, Value3 = 30 };
          ```
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
        *   The constraint can be an arbitrary expression, a range or a length constraint, and can also include compile-time functions. Refinement types cannot have type parameters.
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
            *   CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
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
        *   `sizeof  dataType`: is also a valid syntax without parentheses.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement, CodeBlock };`.
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
                   statements {
                       function void optimizeData(DataType value) {
                         if (hasDataLayoutAttribute(value, "packed")) {
                            //generate code for packed struct
                         } else {
                         // generate standard code
                         }
                    }
                  }
                }
            }
            ```
    *   **Linear Access Attributes:** `#linearAccess`, `#linearAccessGuaranteed(size)`.
        *   `#linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations or prefetching if combined with `#prefetch` attribute.
        *   `#linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray #linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `#circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer. If combined with the `#prefetch` attribute it can also be used for prefetching.
        *   The compiler must verify that the size of the buffer is a power of 2, otherwise it is a compile-time error.
        *   Example: `uint32 safe [1024] myBuffer #circularBuffer;`
    *   **Read and Write Once Attributes:** `#readOnce`, `#writeOnce`
        *   The `#readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is read more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#readOnce` attribute is violated.
        *   The `#writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is written more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
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

    *   **Function Parameter Type Inference:**
        *   BML supports type inference for function parameters when the `#in`, `#out`, or `#inOut` attributes are used.
        *   **`#in`:** The parameter is an input parameter. The compiler will infer its type based on how it's used in the function body. The inferred type will be the type of the expression that is passed as an argument to the function.
        *   **`#out`:** The parameter is an output parameter. Its type must be explicitly specified, as the compiler cannot infer it from the function body alone. The function must assign a value of the specified type to this parameter.
        *   **`#inOut`:** The parameter is both an input and an output parameter. The compiler will infer its type from the argument passed to the function. The function can both read from and assign to this parameter. The argument passed to a function with an `#inOut` parameter must be a variable of a type that is compatible with the inferred type and that can be modified.
        *   **Inference Rules:**
            *   The compiler will analyze the function body to determine the types of `#in` and `#inOut` parameters based on their usage.
            *   If a parameter is used in a way that is incompatible with multiple types, the compiler will generate an error.
            *   If the type of an `#in` or `#inOut` parameter cannot be uniquely inferred, the compiler will generate an error (E0052) and the user must specify the type explicitly.
            *   Type inference does not cross function boundaries. The compiler will not analyze calls to other functions to infer types.
            *   Type inference for function parameters is performed independently for each `@arc` block.

            ```baremetal
            @arc arm {
                statements {
                    function void processData(uint32 data #in, uint32 result #out, uint32 extra #inOut) {
                        result = data * 2;
                        extra~ = extra + result;
                    }

                    function void main() {
                        uint32 x = 10;
                        uint32 y;
                        ~uint32 z = 5;
                        processData(x, y, z); // Type of 'x' and 'z' are inferred as uint32
                    }
                }
            }
            ```
    *   **Transformation Attribute:** `#transform(stringLiteral)`.
        *   This attribute allows transforming the data using target-specific transformations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) #transform("myTransform") { ... };`
    *    **Allocator and Unit Attributes**: `#allocator(stringLiteral)`, `#unit(stringLiteral)`
        *   `#allocator(stringLiteral)` is used to specify the allocator to be used for a specific memory allocation. The `stringLiteral` is CAM specific.        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
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
    *  **Register Attribute:** `#register("regionName", "stringLiteral")`.
        *   This attribute is used to specify a mapping to a memory mapped register. The stringLiteral is CAM specific and is intended to provide additional information for code generation. This attribute can only be used with `register` declarations.
        *  This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
           * Registers can be defined with access modifiers, using the following syntax:
             * `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
             * `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
             * `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Hardware Description Attribute:** `#hardwareDescription(stringLiteral)`.
        *   This attribute allows associating a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `interface intrinsic function void platformInit(uint32 baseAddress) -> void #hardwareDescription("timer.init");`
    *   **Enable and Disable Attributes:** `#enable(stringLiteral)`, `#disable(stringLiteral)`.
        *   These attributes provide a way to enable or disable specific functionalities of a given CAM or a function by using a string parameter that maps to a given boolean configuration variable.
    *    **Algorithmic code generation attribute:** `#algorithm(stringLiteral)`.
        *   This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator instead of relying on traditional code generation techniques.
    *   **Symbolic Execution:** `symbolic` and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        * The `symbolic` keyword is used to create a symbolic block where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check at compile time. The `hasSymbolicRegion()` function returns `true` if there is a symbolic region in the current program context. The `symbolicExpression(identifier)` returns the symbolic expression for a given identifier in the current program context.
    *   **Implicit Type Conversions:**
        *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness and from smaller to larger types. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64` but not to `int8`, `int16`, `int32`, `int64`, or `float32` and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed and requires an explicit cast.
        *   The `as` operator is used to perform an explicit type conversion for the following types:
            *   Any integer type to a larger integer type of *the same signedness*. (e.g., `uint8 as uint32`, `int16 as int64`).
            *   Any integer type (signed or unsigned) to a floating point type (e.g., `uint32 as float32`, `int16 as float64`).
            *   Floating-point types to other floating-point types of different precision (e.g., `float32 as float64`).
            *   `DataType^` to `genericAddress`, `dataAddress` or `codeAddress` and vice versa.
            *   Types that are compatible for implicit type conversion (like a formal parameter of a function that is implicitly compatible).
            *   The compiler will perform a compile-time error if the type conversion is invalid.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion and when returning from functions if the return type is compatible for implicit conversion.
        *   No other implicit type conversions are allowed. Conversions must be explicit by using the `cast` operator or by calling conversion functions from the standard library or provided by the CAMs, by using the `into` keyword, by using the `to` method or by using custom suffixes.
    *   **Implicit Type Inference:** Type inference is used in the following cases:
        *   When the return type of a function is not explicitly specified, the compiler will infer the return type by analyzing the `return` statements or generate a compile-time error if no return statement is present and the return type cannot be inferred.
        *   When declaring an array literal the compiler infers the array type from the literal values.
        *   When a variable is assigned an integer literal and the type can be inferred from the context if not an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal the default type is `float32`.
       *   **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
        *   When a local variable is initialized with an expression inside the function scope.
        *   When a register variable is initialized with a given value in the CAM scope.
    *   **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.
        *   **Nested functions:** Functions can be nested within other functions. Inner functions have access to variables in their enclosing function's scope (closure).
         *   Nested functions can only be defined within `@arc` blocks. They are not allowed in the `@arc common` block.
        *   **Shadowing:** Inner scopes can declare variables with the same name as variables in outer scopes. The inner declaration shadows the outer one. The compiler will issue a warning in such cases.
        *   **Global Scope:** Variables declared at the top level of an `@arc` block are considered global for that architecture.
    *   **Instruction Type:**
        *   The `instruction<instructionName, [parameters]>` type is used to instantiate low-level hardware instructions.
        *   The `instructionName` should match a previously defined instruction using the `hardwareInstructionDSL`.
        *   The `parameters` are a list of compile-time expressions that can be used to specify the operands of the instruction or any other parameter specific to the instruction.
        *   The `instruction` type is a base type, similarly to the integer, float and rawbyte types, and can be used as the type of a given variable, a function parameter, or a return type of a function.
        *   Example: `instruction<add, r0, r1, 10> myAddInstruction;`
    *  **Code Block Type:** `block (parameterList) -> returnType;`.
        * A type that defines a function signature for a code block.
        *  **Default value**: The default value of a code block is `null`.
        * Example: `block (uint32, uint32) -> uint32 MathOperationType;`
       * Code blocks can be captured using lambda expressions: `myCodeBlock = (a,b) => a+b;`.
    *    **Bitmask Type:** A constant expression that is used to describe a bitfield using an integer value. Example: `const bitmask ENABLE_BIT = 0x01;`

**3. Modules and the Main File: Structure and Organization (Updated)**

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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope making the exported components of the module visible in the current scope. When using the `#use` directive you can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
       *  Module aliases:  Modules can be imported using an alias to avoid namespace conflicts, or when the module name is too long `#use "MyLongModuleName" as MyModule`.
        * Specific versions of the modules can be imported using an `@` syntax. Example: `#use "MyModule@1.2.0";`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
            * Example:
            ```baremetal
                module MyModule {
                    config {
                       config dataSize : uint32 = 16;
                  }
                  statements {
                      function void myFunction(uint32 data) {
                        print("Data size:", config.dataSize);
                      }
                   }
                }
                 module MyOtherModule {
                 statements {
                      #use "MyModule" with config { dataSize = 32 };
                       function void myOtherFunction() {
                          myFunction(10);
                       }
                   }
               }
            ```
            *   In the example above, the function `myFunction` will use the dataSize of `32` which overrides the default `dataSize` value.
         *   **Module Configuration Files:** Modules can load configuration data by specifying a `StringLiteral` that specifies a configuration file, which is loaded and parsed at compile time. The compiler uses the parser that is specified by the type annotation in the configuration section.
          ```baremetal
            module MyModule {
                config {
                  config myConfig : MyConfig =  parseConfigFile("config.txt");
                }
            }
          ```

*   **Main File:**
    *   **The main file is the entry point of the BML application.**
    *   **It is defined in a separate `.bml` file (typically `main.bml`).**
    *   **The main file does *not* use the `module` keyword.**
    *   **It does *not* contain a `main` function.**
    *   **Instead, the `statements` block within each `@arc` block acts as the entry point for that specific architecture.**
     *  **It must have an explicit `@arc common` block to contain architecture-independent code.** Code outside of any `@arc` block is not allowed in the `main.bml` file.
     *  The `@arc common` block is a global scope for all the `@arc` blocks in the `main.bml` file, and code within the `@arc common` block is treated as architecture-independent, meaning that it will be compiled for all target architectures. Direct access to hardware is not allowed in the `@arc common` block. Nested functions are not allowed inside the `@arc common` block, and all function calls are limited to only be able to call other functions inside the `@arc common` block.
    *   **It uses the `#use` directive to import modules.**
    *   The main file can also contain `@arc` blocks for architecture-specific code.
    *   Example (`main.bml`):
        ```baremetal
        #use "MyModule";
         #use "MyGenericCam<uint32>" as GenericUint32Cam;

         @arc common {
            declarations {
                 uint32 commonVar = 5;
            }
            statements {
                 function void commonFunction() {
                  // This function is available to all architectures
                  print("Common function called");
              }
            }
        }


        @arc arm {
            declarations {
                // ... ARM-specific declarations ...
            }
            statements {
                // ... ARM-specific initialization ...
                 commonFunction(); // Can call functions from common section
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
                commonFunction(); // Can call functions from common section
                myFunction(); // Call a function from MyModule
                // ... More RISC-V specific code ...
            }
        }
        ```

**5. Composable Architecture Modules (CAMs) (Updated)**

*   CAMs are compiler extensions that act as "code generation engines." They allow extending the compiler and generating platform-specific code in a type-safe and modular fashion, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM. (CAMs are modules)
        *   `description "CAM Description String";`: (Optional) A description of the CAM which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can load external data from a file by specifying the location in the `config` section and using a parsing function. The `config` parameters can be customized using command line arguments or by using a `with config { ... }` section when the CAM is imported using the `#use` directive. CAM options can be enabled or disabled using the `#enable("optionName")` or `#disable("optionName")` attributes. If the user does not specify a default value for a configuration parameter, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral; config enableDma : boolean #enable("dma") = false; config myOption : CAMOption = {true, 100};}`.
             *  The `config` section can also load a file by specifying the filename in a `StringLiteral` and then processing the file by using a compile-time function that is able to parse the data.
                 * Example: `config { config hardwareDescriptionFile : StringLiteral; config registers : RegisterConfig =  parseRegisterData(hardwareDescriptionFile) ; };` In this example `parseRegisterData` must be a `compileTimeFunction` and `RegisterConfig` a user defined data type.
                *  The `#enable("optionName")` and `#disable("optionName")` attributes can be used to conditionally compile code. Example: `function void myFunction() #enable("myOption") { /*...*/ }` or `config { config enableOpt : boolean #enable("myOption") = true; }`. The `myOption` configuration parameter is defined as `config myOption : CAMOption = {true, 100};`.
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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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
         * The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
          *  The `callDSL` block is used to customize the Application Binary Interface (ABI) of a given function call, by providing a mechanism to specify the calling conventions, the parameter passing mechanism, the stack frame handling and the instructions to setup and teardown a function call.
             *   **`callConvention` Block**: Defines the calling convention for a given architecture, by specifying how the stack is allocated, how the parameters are passed, the register usage, the prologue, the epilogue, and other information about the application binary interface.
             *   **`parameterPassing`:** Specifies how the parameters are passed when calling a function. Parameters can be passed using specific registers or using the stack.
             *   **`returnLocation`:** Specifies how the return value is passed to the caller function.
             *   **`prologue`:** Specifies the code that is executed before a function starts, and usually includes the instructions to save the registers in the stack, and other setup instructions.
             *   **`epilogue`:** Specifies the code that is executed before a function returns, and usually includes the instructions to restore the registers and pop the stack.
             *   **`stackAllocation`:** Defines how the stack memory is allocated and deallocated for a function call.
             *   **`abiName`:** Provides the name of the application binary interface that the compiler must implement when generating the code, which is used to guarantee compatibility with other libraries and code.
            *    **`hardwareInstructionDSL`**: Provides a mechanism to specify the low-level implementation of a hardware instruction.
             * **`encoding`**: Used to specify the bit level layout of the hardware instruction using a string literal.
             * **`sideEffect`**: Used to specify the side effects of an instruction such as flag setting or changes in memory or registers.
             * **`microcode`**: Used to specify the low level microcode implementation of the hardware instruction using a sequence of micro operations specified by string literals.
             * **`latency`**: Used to specify the latency of a given instruction.
        *  Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time.
          * Meta-CAMs can be used to parse instruction set specifications, hardware description files, or other configuration files and then create new CAMs dynamically during compile-time.

     ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean #enable("dma") = false;
                         config functionParameterRegister : StringLiteral #register("r0") = "r0";
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

                     @callDSL {
                     callConvention myCallConvention {
                        parameterPassing {
                           firstParameter  #register(config.functionParameterRegister), // the register will be read from the config section.
                           otherParameters  stack
                       }
                      returnLocation r0,
                       prologue {
                          "push lr"
                       }
                      epilogue {
                          "pop pc"
                       }
                       stackAllocation {
                          allocateStack(frameSize);
                           freeStack(frameSize);
                       }
                     abiName "arm_std_abi"
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
                         memoryStore(dword, sp + 0x10, rd);
                         }
                         microcode {
                         "read r" + rn + ", dataReg"
                            "add r" + rd + ", dataReg"
                           "write r" + rd + ", dataReg"
                            }
                           latency = 2;
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
                       function void platformUartSendByte(uint8 data) #callConvention("myCallConvention"){
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
        *  The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body to perform low-level hardware operations. The type of the register is inferred by the assigned value.
        *   The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        *  The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
        *   The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
        *   The `hardwareInstructionDSL` block allows the developer to define the encoding of a hardware instruction by specifying the bit layout, side effects, operands, and microcode, using a declarative DSL.
        *   The `callDSL` block allows the developer to specify the application binary interface of a target architecture.
        *   Meta-CAMs are used to generate CAMs dynamically at compile time, using a hardware description or a configuration file.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        *   Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality, and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *    The import system will resolve conflicts with the last imported CAM overriding previous definitions.
         *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
        *   When a CAM is imported using `override function`, the new definition will override the previous one. The previous definition is discarded and not accessible. A compile-time error will be generated if the CAM attempts to use the function definition that is being overridden.
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
    *    A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
    *   The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
     *  If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
        *   CAM options can be enabled or disabled by using the `#enable("myOption")` or `#disable("myOption")` attributes. Example: `function void myFunc() #enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean #enable("myOption") = true; }`

*   **CAM API: Compiler Interface for CAMs**
    *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies. The `dataflowGraph` is composed of nodes and edges representing the data flow through the program. Each node has metadata and information about the operations that are performed including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges and to perform specific data flow analysis tasks.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    *   Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                *  `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer and so on. See enum `TypeKind` for possible types.
                *    `sizeof(DataType)`: Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                *   **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                    *   Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                *   **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                *   **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                 *    **`compilerWarning(StringLiteral message)`:** Generates a custom compiler warning message, allowing for more specific warning reporting.
                *    **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                     *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                    *  The implementation is a function pointer that must have the same signature as the `functionSignature` string.
                *   **`newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification)`:** Creates a new CAM with the given parameters. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton.
                     *  The `interfaceDefinitions` will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                      * The `implementation` will be a set of function pointers that define the implementations of the interface intrinsic functions.
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
                *  **`addTransformation(Transformation transformation)`:** Adds a transformation to the code which is then performed by the compiler during a code generation phase.
                     * The `Transformation` object contains:
                       * **`removeCode`**: The code that should be removed specified as a string.
                     * **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                       * **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                      *  **`target`**: A `TargetData` structure with information about the target including the type of the data being transformed, the memory region, the volatility state, and other metadata about the access.
                        *  **`TargetData`**: The TargetData structure provides information about the target, including:
                            *   `data`: The data type of the target.
                            *   `address`: The memory address of the target if it's a memory access operation.
                            *   `region`: The memory region of the target.
                            *   `isVolatile`: Returns true if the data is volatile.
                            *  `isRegister`: Returns true if the access is a register access.
                            *  `isMemoryMapped`: Returns true if the target is memory-mapped.
                            *   `hasAttribute(StringLiteral attributeName)`: Returns true if the target has a specific attribute.
                *   **`getHardwareDescription(StringLiteral hardwareDescription)`**: Loads a hardware description language file and returns the content as a String literal that can be used by CAMs.
                *   **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context otherwise `false`.
                *   **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                 *  **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                 * **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                  *  **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                *  **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                *   **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal. The pattern language uses the following syntax:
                    *   The pattern language supports the following syntax:
                      * `identifier`: Matches any identifier.
                       *  `stringLiteral`: Matches a specific string literal.
                       *  `integerLiteral`: Matches a specific integer literal.
                       *  `"..."`: Matches a sequence of characters that are present in the code.
                       *  `variable(identifier)`: Matches any variable and captures the identifier.
                        *   `immediate(identifier)`: Matches any immediate value and captures the identifier.
                       *   `instruction`: Matches any instruction.
                      *  `add(x, y)`: Matches an add instruction with two arguments.
                        *   `functionCall(x)`: Matches a function call to the function named `print`.
                         * `*`: Matches anything (wildcard).
                        *  `()`: Creates a group for the match.
                    *    Example:
                        * `"add(x, y)"`: This will match an add instruction with two arguments.
                        *   `"load(address)"`: Matches a load instruction with a parameter named `address`.
                        *    `"functionCall(print)"`: Matches a function call to the function named `print`.
                *   **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern. The captured subexpression will be returned using a string literal, that can be used in other code transformations. The same pattern language as for `match` is used to perform the capture.
                    *    Example: `capture("add(x, y)", code, "myVariable")` if `code` is `add(r0, r1)`, it will return a string `"r0,r1"` that can be used in other code transformations.
                *  **`createInstruction(StringLiteral instructionName, instructionParameters) -> Instruction`**: Creates a new `instruction` using the given parameters.
                  *  **`generateMicrocode(Instruction instruction) -> StringLiteral`**: Generates the low level microcode for a given instruction.
                   *  **`addTemplate(StringLiteral sectionName, StringLiteral templateCode);`**: Add a code template at a given section.
                   *  **`hasHardwareSymbolicSupport() -> boolean`:** returns `true` if the target platform provides hardware support for symbolic checks.
                    *  **`dispatchSymbolicCheck(StringLiteral operation, expression)`:** Dispatches a symbolic check operation to the hardware. The type of operation can be specified by the `operation` string.
                    *   **`getSymbolicCheckResult(address) -> boolean`:** Returns the result of the symbolic check given by the `address`.
                     *   **`hasHardwarePrefetchSupport() -> boolean`:** returns `true` if the target platform provides hardware support for prefetching.
                    *   **`dispatchPrefetch(dataAddress address, size) -> void;`**: dispatches a prefetch instruction to a specific memory location. The `size` parameter indicates how much data should be prefetched from the specified `address`.
                     *    **`getInstructionSchedulingHints(codeBlock) -> List<InstructionSchedulingHint>;`**: Gets a list of hints for instruction scheduling based on data dependencies using symbolic information. The codeBlock is the code region that should be analyzed by the CAM.
                    *   **`addTransformation(StringLiteral removeCode, StringLiteral insertCode, StringLiteral transformationType);`** Add a code transformation that will remove the code that matches the `removeCode` String literal and will insert `insertCode`, and will use the `transformationType` to perform a specific transformation.
                     * **`getCompilerFlag(StringLiteral flagName)`:** Get the value of a compiler flag.
                      * **`setCompilerFlag(StringLiteral flagName, StringLiteral flagValue)`:** Set the value of a compiler flag for a given region or function.
                      * **`emitCode(StringLiteral code, TargetData targetData)`**: Allows the CAM to directly inject code at a specific target location.
                    * **`skipTransformation()`**: Skips all further transformations for a specific code block.

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
                *  `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *    `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                 *    `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
                 *   `instructionBuilder.atomicRead(address, dataType,  bitmask)`: Creates an atomic read instruction.
                *  `instructionBuilder.atomicWrite(address, value, dataType, bitmask)`: Creates an atomic write instruction.
                  *   `instructionBuilder.atomicTestAndSetBit(address, bitNumber)`: Creates an atomic test and set bit instruction.
                   *   `instructionBuilder.atomicTestAndClearBit(address, bitNumber)`: Creates an atomic test and clear bit instruction.
                *   `instructionBuilder.atomicSetBit(address, bitNumber)`: Creates an atomic set bit instruction.
                 *   `instructionBuilder.atomicClearBit(address, bitNumber)`: Creates an atomic clear bit instruction.
                 *  `instructionBuilder.atomicToggleBit(address, bitNumber)`: Creates an atomic toggle bit instruction.
                *  `instructionBuilder.memoryBarrier(order)`: Creates a memory barrier instruction.
                *  `instructionBuilder.instructionBarrier(sync)`: Creates an instruction barrier instruction.
                 *  `instructionBuilder.rawInstruction(instructionPattern)`: Emits a raw instruction.
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
*  **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages using the `compilerError()` function.
 *  **Custom Warnings and Information Messages:** CAMs can generate customized warning and information messages.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access:** Direct access to hardware registers using memory mapped access and conceptual registers.
*   **ABI Customization**: CAMs can customize the function call ABI using the `callDSL` block.
*  **Microcode Specification:** The microcode of an instruction can be specified using the `microcode` block.
*  **Performance Optimization**: Code regions can be tagged with performance attributes, to guide the compiler optimizations using a target specific implementation.
*   **Code Generation Bypass**: CAMs can bypass code transformations and provide a low-level implementation when necessary using the `emitCode` and `skipTransformation` compiler API functions.

**6. Directives and Pragmas (Updated)**

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
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion that makes the exported components of the module accessible. You can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
     *    Specific module versions can be imported using the `@` syntax. Example: `#use "MyModule@1.2.0";`
    *   `#use "camName" [ genericInstantiation ] [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code.
        *   Example: `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        *   Example: `#raw 0x10 0b00010000 16 0o20 "abc";`
    *   `#template(sectionName) { ... }` : This directive allows for injection of a code template into a specific section. The `sectionName` is defined in a CAM.
    *  `#performance(level, hint, ...)`: A pragma that provides performance information to the compiler.
           * `#performance(critical, memoryAccess, loopUnroll=4, ...)`: example.
             * The `level` is a string literal that can be used to specify the level of performance optimization such as `critical`, `high`, `normal` or `low`, and will be CAM specific, and may have different meanings for different target architectures.
             * The `hint` is a string literal that can be used to provide specific hints for a given code section such as `memoryAccess`, or `loopUnroll`.

*   **Operation Directives:**
        *  `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parentheses can be omitted if only one parameter is passed. Example: `print "Hello world";`.
        * `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier depending on the target platform.
        * `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *   `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parentheses can be omitted if only one argument is passed.
        *   `atomicSetBit(register, bitNumber)`: Atomically sets a specific bit in a register.
        *   `atomicClearBit(register, bitNumber)`: Atomically clears a specific bit in a register.
        *   `atomicToggleBit(register, bitNumber)`: Atomically toggles a specific bit in a register.
        *   `atomicTestAndSetBit(register, bitNumber)`: Atomically tests a bit and sets it if clear. Returns true if the bit was clear before the operation.
        *   `atomicTestAndClearBit(register, bitNumber)`: Atomically tests a bit and clears it if set. Returns true if the bit was set before the operation.
        *   `atomicReadRegister(register, bitmask)`: Atomically reads a value from a register using a specific bitmask.
        *   `atomicWriteRegister(register, value, bitmask)`: Atomically writes a value to a register, applying a given bitmask.
      * `memoryBarrier(order);`: Generates a hardware specific memory barrier. The `order` can be `read`, `write`, `readWrite`, or `all`. If no parameter is specified, it defaults to `all`.
      *   `instructionBarrier(sync);`: Generates a hardware specific instruction barrier. The `sync` can be `instructionFetch`, `dataFetch`, or `dataWrite`.
      *  `rawInstr(instructionPattern);`: Embeds raw machine code into the generated output. The `instructionPattern` is CAM specific.
       *  `hardwareSymbolicCheck(expression, range) -> void;` : Performs a hardware-assisted check if the expression satisfies the constraints given by the range. If no hardware support is present, the code will trap.
        *   `hardware_clz(uint32 value) -> uint32;` : Counts the number of leading zeros in the specified value using a hardware-accelerated instruction if available.
        *   `hardware_ctz(uint32 value) -> uint32;` : Counts the number of trailing zeros in the specified value using a hardware-accelerated instruction if available.
        *  `hardware_startTimer();` : Starts a hardware timer.
         * `hardware_endTimer();` : Reads the current value of the hardware timer.
         * `hardware_resetTimer();`: Resets the hardware timer.
          * `hardware_setWatchpoint(address, type);`: Sets a hardware watchpoint that can be used for debugging purposes. The type of the watchpoint can be `read`, `write`, or `readWrite`.
          * `prefetch(address, size, level);`: Dispatches a hardware prefetch instruction for a given `address` with a specific `size` and `level`.
          * `hardware_trap();`: Generates a hardware trap for debugging purposes when a low level error is detected.
        * `cacheFlush(address addr, size size, level level);` Flushes data from the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
        * `cacheInvalidate(address addr, size size, level level);` Invalidates data in the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
       *  `align(expression, integerLiteral);`: Aligns a given address to a specific alignment.
    *  **Atomic Operations:**
        *   `atomic { ... }`: Guarantees atomic execution of the code block using the primitives specified in the CAM. If no argument is specified, the compiler will attempt to atomically execute the instructions within the given block, which may involve software locking mechanisms if hardware primitives are not available, which might create an overhead depending on the target architecture and the compiler options. The code block is limited to low level access primitives, and calls to CAM provided `intrinsic` functions that can be mapped to target-specific atomic primitives.
        *   `atomic (variable) { ... }`: Guarantees atomic access and modification of the specified `variable` using the underlying primitives provided by the CAM. The scope is limited to operations involving that specific variable. The compiler will attempt to generate code to atomically read, modify and write to that specific memory location or variable. If the hardware doesn't have a direct atomic read-modify-write instruction the CAM may have to generate code that involves locks and memory barriers to ensure atomicity.
        *  `atomic memoryStore(...)`: Performs an atomic memory store operation.
         * `atomic register identifier = ...;`: Performs an atomic write to a given register using an atomic instruction when it is available.
    *   **Memory Mapped Registers:**
        *  `struct #mapped`: Defines a memory-mapped structure where the members are implicitly `volatile` unless marked `nonvolatile`. Allows for direct access to hardware registers. The `#mapped` specifier specifies that the struct is memory mapped. Members are accessed using the `.` operator, and bitfields can be accessed using the `=` or `:=` operator inside a `with` statement, or directly when assigning to a memory-mapped structure. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using bitmasks: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`, or a mix of the two. Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access. When the `#initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct #mapped(0x40000000) #initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`.
         * Example:
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
                       uint32 status = readout; //no memory read here as the bitfield is not modified
                   }
                 myGpioRegisters.enable = 0; // single bitfield modification using implicit syntax.
                 myGpioRegisters.baud = 115200; // single bitfield modification using implicit assignment.
                  myGpioRegisters.controlRegister = {enable=1, baud=115200}; //bitfield assignment using struct like syntax
            ```
        *  `.` accesses members. If the member is inside a memory-mapped struct, it will access the memory location directly.
        *   **`=` sets/reads bitfields inside a memory-mapped struct.**
        *   **Bit Ranges: `[start..end]` accesses a range of bits in a bitfield inside a memory-mapped struct.**
        *   **Bit Masks: `[binaryMask]` accesses the bits defined by a binary mask in a bitfield inside a memory-mapped struct.**
        *   **Combined Selectors: `[range, bit, mask, ...]` accesses multiple range of bits or individual bits inside a bitfield of a memory-mapped struct.**
        *   Memory Mapped Registers: Specific registers can be defined using the following syntax:
            *   `DataType #register("regionName", "stringLiteral") registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
             * Registers can be defined with access modifiers, using the following syntax:
               *  `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants improving the readability of the code. If the enum is declared with a simple comma-separated list of values curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
         * Enums can be initialized with specific values using a `struct`-like syntax:
          ```baremetal
           enum ErrorCode { Ok=0, NotFound=1, AccessDenied=2 };
           enum MyEnum { Value1 = 10, Value2 = 20, Value3 = 30 };
          ```
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
        *   The constraint can be an arbitrary expression, a range or a length constraint, and can also include compile-time functions. Refinement types cannot have type parameters.
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
            *   CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
        *   Example: `type SecureData = uint32 @region("secureRegion");`
    *   **Capability-Based Regions:** `dataRegion regionName [ regionCapabilitySpecifier ] { ... };`
        *  Data regions are used for compile-time region-based memory access controls and memory management, allowing compile-time checking of memory accesses and capabilities, increasing memory safety. If only one member is present curly braces can be removed. `dataRegion MySecureData uint32 secretKey;`
        *  The `regionCapabilitySpecifier` specifies which permissions are available: `read`, `write`, `execute`, `capability_derive`, `peripheral`.
            * If only `read` or `read, write` capabilities are needed, the `capabilities` keyword can be omitted: `dataRegion @read MySecureData uint32 secretKey;`, `dataRegion DeviceData { uint8 safe[100] buffer; };` (implicit read, write capabilities).
        *   `capability_derive` allows creating new pointers with limited capabilities from a pointer that has the `capability_derive` permission, granting a way to control how a pointer is used within a specific scope or region.
        *   `peripheral` allows to access a memory mapped peripheral using the mechanisms defined by the CAM.
        *   A memory region must be declared using this syntax to be used in `@region(regionName)` pointers, otherwise, the compiler will generate a compile-time error.
        *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe [100] buffer; };`
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
        *   `sizeof  dataType`: is also a valid syntax without parentheses.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement, CodeBlock };`.
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
                   statements {
                       function void optimizeData(DataType value) {
                         if (hasDataLayoutAttribute(value, "packed")) {
                            //generate code for packed struct
                         } else {
                         // generate standard code
                         }
                    }
                  }
                }
            }
            ```
    *   **Linear Access Attributes:** `#linearAccess`, `#linearAccessGuaranteed(size)`.
        *   `#linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations or prefetching if combined with `#prefetch` attribute.
        *   `#linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray #linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `#circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer. If combined with the `#prefetch` attribute it can also be used for prefetching.
        *   The compiler must verify that the size of the buffer is a power of 2, otherwise it is a compile-time error.
        *   Example: `uint32 safe [1024] myBuffer #circularBuffer;`
    *   **Read and Write Once Attributes:** `#readOnce`, `#writeOnce`
        *   The `#readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is read more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#readOnce` attribute is violated.
        *   The `#writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is written more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
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

    *   **Function Parameter Type Inference:**
        *   BML supports type inference for function parameters when the `#in`, `#out`, or `#inOut` attributes are used.
        *   **`#in`:** The parameter is an input parameter. The compiler will infer its type based on how it's used in the function body. The inferred type will be the type of the expression that is passed as an argument to the function.
        *   **`#out`:** The parameter is an output parameter. Its type must be explicitly specified, as the compiler cannot infer it from the function body alone. The function must assign a value of the specified type to this parameter.
        *   **`#inOut`:** The parameter is both an input and an output parameter. The compiler will infer its type from the argument passed to the function. The function can both read from and assign to this parameter. The argument passed to a function with an `#inOut` parameter must be a variable of a type that is compatible with the inferred type and that can be modified.
        *   **Inference Rules:**
            *   The compiler will analyze the function body to determine the types of `#in` and `#inOut` parameters based on their usage.
            *   If a parameter is used in a way that is incompatible with multiple types, the compiler will generate an error.
            *   If the type of an `#in` or `#inOut` parameter cannot be uniquely inferred, the compiler will generate an error (E0052) and the user must specify the type explicitly.
            *   Type inference does not cross function boundaries. The compiler will not analyze calls to other functions to infer types.
            *   Type inference for function parameters is performed independently for each `@arc` block.

            ```baremetal
            @arc arm {
                statements {
                    function void processData(uint32 data #in, uint32 result #out, uint32 extra #inOut) {
                        result = data * 2;
                        extra~ = extra + result;
                    }

                    function void main() {
                        uint32 x = 10;
                        uint32 y;
                        ~uint32 z = 5;
                        processData(x, y, z); // Type of 'x' and 'z' are inferred as uint32
                    }
                }
            }
            ```
    *   **Transformation Attribute:** `#transform(stringLiteral)`.
        *   This attribute allows transforming the data using target-specific transformations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) #transform("myTransform") { ... };`
    *    **Allocator and Unit Attributes**: `#allocator(stringLiteral)`, `#unit(stringLiteral)`
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
        *  This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
           * Registers can be defined with access modifiers, using the following syntax:
             * `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
             * `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
             * `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Hardware Description Attribute:** `#hardwareDescription(stringLiteral)`.
        *   This attribute allows associating a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `interface intrinsic function void platformInit(uint32 baseAddress) -> void #hardwareDescription("timer.init");`
    *   **Enable and Disable Attributes:** `#enable(stringLiteral)`, `#disable(stringLiteral)`.
        *   These attributes provide a way to enable or disable specific functionalities of a given CAM or a function by using a string parameter that maps to a given boolean configuration variable.
    *    **Algorithmic code generation attribute:** `#algorithm(stringLiteral)`.
        *   This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator instead of relying on traditional code generation techniques.
    *   **Symbolic Execution:** `symbolic` and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        * The `symbolic` keyword is used to create a symbolic block where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check at compile time. The `hasSymbolicRegion()` function returns `true` if there is a symbolic region in the current program context, otherwise it returns false, The `symbolicExpression(identifier)` returns the symbolic expression for a given identifier in the current program context.
    *    **Implicit Type Conversions:**
         *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness and from smaller to larger integers are implicitly allowed. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64` but not to `int8`, `int16`, `int32`, `int64`, or `float32` and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed and requires an explicit cast.
        *   The `as` operator is used to perform an explicit type conversion for the following types:
            *   Any integer type to a larger integer type of *the same signedness*. (e.g., `uint8 as uint32`, `int16 as int64`).
            *   Any integer type (signed or unsigned) to a floating point type (e.g., `uint32 as float32`, `int16 as float64`).
            *   Floating-point types to other floating-point types of different precision (e.g., `float32 as float64`).
            *   `DataType^` to `genericAddress`, `dataAddress` or `codeAddress` and vice versa.
            *   Types that are compatible for implicit type conversion (like a formal parameter of a function that is implicitly compatible).
            *   The compiler will perform a compile-time error if the type conversion is invalid.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion and when returning from functions if the return type is compatible for implicit conversion.
        *  No other implicit type conversions are allowed. Conversions must be explicit by using the `cast` operator, by calling conversion functions from the standard library or provided by the CAMs, by using the `into` keyword, by using the `to` method or by using custom suffixes.
    *   **Implicit Type Inference:** Type inference is used in the following cases:
        *   When the return type of a function is not explicitly specified, the compiler will infer the return type by analyzing the `return` statements or generate a compile-time error if no return statement is present and the return type cannot be inferred.
        *   When declaring an array literal the compiler infers the array type from the literal values.
        *   When a variable is assigned an integer literal and the type can be inferred from the context if not an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal the default type is `float32`.
       *   **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
        *   When a local variable is initialized with an expression inside the function scope.
        *   When a register variable is initialized with a given value in the CAM scope.
    *   **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.
        *   **Nested functions:** Functions can be nested within other functions. Inner functions have access to variables in their enclosing function's scope (closure).
         *   Nested functions can only be defined within `@arc` blocks. They are not allowed in the `@arc common` block.
        *   **Shadowing:** Inner scopes can declare variables with the same name as variables in outer scopes. The inner declaration shadows the outer one. The compiler will issue a warning in such cases.
        *   **Global Scope:** Variables declared at the top level of an `@arc` block are considered global for that architecture.
    *  **Instruction Type:**
        *   The `instruction<instructionName, [parameters]>` type is used to instantiate low-level hardware instructions.
        *   The `instructionName` should match a previously defined instruction using the `hardwareInstructionDSL`.
        *   The `parameters` are a list of compile-time expressions that can be used to specify the operands of the instruction or any other parameter specific to the instruction.
        *   The `instruction` type is a base type, similarly to the integer, float and rawbyte types, and can be used as the type of a given variable, a function parameter, or a return type of a function.
        * Example: `instruction<add, r0, r1, 10> myAddInstruction;`
    *  **Code Block Type:** `block (parameterList) -> returnType;`.
         * A type that defines a function signature for a code block.
         *  **Default value**: The default value of a code block is `null`.
         * Example: `block (uint32, uint32) -> uint32 MathOperationType;`
         * Code blocks can be captured using lambda expressions: `myCodeBlock = (a,b) => a+b;`.
    *    **Bitmask Type:** A constant expression that is used to describe a bitfield using an integer value. Example: `const bitmask ENABLE_BIT = 0x01;`

**3. Modules and the Main File: Structure and Organization (Updated)**

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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope making the exported components of the module visible in the current scope. When using the `#use` directive you can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
       *  Module aliases:  Modules can be imported using an alias to avoid namespace conflicts, or when the module name is too long `#use "MyLongModuleName" as MyModule`.
        * Specific versions of the modules can be imported using an `@` syntax. Example: `#use "MyModule@1.2.0";`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
            * Example:
            ```baremetal
                module MyModule {
                    config {
                       config dataSize : uint32 = 16;
                  }
                  statements {
                      function void myFunction(uint32 data) {
                        print("Data size:", config.dataSize);
                      }
                   }
                }
                 module MyOtherModule {
                 statements {
                      #use "MyModule" with config { dataSize = 32 };
                       function void myOtherFunction() {
                          myFunction(10);
                       }
                   }
               }
            ```
            *   In the example above, the function `myFunction` will use the dataSize of `32` which overrides the default `dataSize` value.
         *   **Module Configuration Files:** Modules can load configuration data by specifying a `StringLiteral` that specifies a configuration file, which is loaded and parsed at compile time. The compiler uses the parser that is specified by the type annotation in the configuration section.
          ```baremetal
            module MyModule {
                config {
                  config myConfig : MyConfig =  parseConfigFile("config.txt");
                }
            }
          ```

*   **Main File:**
    *   **The main file is the entry point of the BML application.**
    *   **It is defined in a separate `.bml` file (typically `main.bml`).**
    *   **The main file does *not* use the `module` keyword.**
    *   **It does *not* contain a `main` function.**
    *   **Instead, the `statements` block within each `@arc` block acts as the entry point for that specific architecture.**
     *  **It must have an explicit `@arc common` block to contain architecture-independent code.** Code outside of any `@arc` block is not allowed in the `main.bml` file.
     *  The `@arc common` block is a global scope for all the `@arc` blocks in the `main.bml` file, and code within the `@arc common` block is treated as architecture-independent, meaning that it will be compiled for all target architectures. Direct access to hardware is not allowed in the `@arc common` block. Nested functions are not allowed inside the `@arc common` block, and all function calls are limited to only be able to call other functions inside the `@arc common` block.
    *   **It uses the `#use` directive to import modules.**
    *   The main file can also contain `@arc` blocks for architecture-specific code.
    *   Example (`main.bml`):
        ```baremetal
        #use "MyModule";
         #use "MyGenericCam<uint32>" as GenericUint32Cam;

         @arc common {
            declarations {
                 uint32 commonVar = 5;
            }
            statements {
                 function void commonFunction() {
                  // This function is available to all architectures
                  print("Common function called");
              }
            }
        }


        @arc arm {
            declarations {
                // ... ARM-specific declarations ...
            }
            statements {
                // ... ARM-specific initialization ...
                 commonFunction(); // Can call functions from common section
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
                commonFunction(); // Can call functions from common section
                myFunction(); // Call a function from MyModule
                // ... More RISC-V specific code ...
            }
        }
        ```

**4. Composable Architecture Modules (CAMs) (Updated)**

*   CAMs are compiler extensions that act as "code generation engines." They allow extending the compiler and generating platform-specific code in a type-safe and modular fashion, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM. (CAMs are modules)
        *   `description "CAM Description String";`: (Optional) A description of the CAM which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can load external data from a file by specifying the location in the `config` section and using a parsing function. The `config` parameters can be customized using command line arguments or by using a `with config { ... }` section when the CAM is imported using the `#use` directive. CAM options can be enabled or disabled using the `#enable("optionName")` or `#disable("optionName")` attributes. If the user does not specify a default value for a configuration parameter, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral; config enableDma : boolean #enable("dma") = false; config myOption : CAMOption = {true, 100};}`.
             *  The `config` section can also load a file by specifying the filename in a `StringLiteral` and then processing the file by using a compile-time function that is able to parse the data.
                 * Example: `config { config hardwareDescriptionFile : StringLiteral; config registers : RegisterConfig =  parseRegisterData(hardwareDescriptionFile) ; };` In this example `parseRegisterData` must be a `compileTimeFunction` and `RegisterConfig` a user defined data type.
                *  The `#enable("optionName")` and `#disable("optionName")` attributes can be used to conditionally compile code. Example: `function void myFunction() #enable("myOption") { /*...*/ }` or `config { config enableOpt : boolean #enable("myOption") = true; }`. The `myOption` configuration parameter is defined as `config myOption : CAMOption = {true, 100};`.
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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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
         * The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
          * The `callDSL` block is used to customize the Application Binary Interface (ABI) of a given function call, by providing a mechanism to specify the calling conventions, the parameter passing mechanism, the stack frame handling and the instructions to setup and teardown a function call.
             *   **`callConvention` Block**: Defines the calling convention for a given architecture, by specifying how the stack is allocated, how the parameters are passed, the register usage, the prologue, the epilogue, and other information about the application binary interface.
             *   **`parameterPassing`:** Specifies how the parameters are passed when calling a function. Parameters can be passed using specific registers or using the stack.
             *   **`returnLocation`:** Specifies how the return value is passed to the caller function.
             *   **`prologue`:** Specifies the code that is executed before a function starts, and usually includes the instructions to save the registers in the stack, and other setup instructions.
             *   **`epilogue`:** Specifies the code that is executed before a function returns, and usually includes the instructions to restore the registers and pop the stack.
             *   **`stackAllocation`:** Defines how the stack memory is allocated and deallocated for a function call.
             *  **`abiName`:** Provides the name of the application binary interface that the compiler must implement when generating the code, which is used to guarantee compatibility with other libraries and code.
            *    **`hardwareInstructionDSL`**: Provides a mechanism to specify the low-level implementation of a hardware instruction.
             * **`encoding`**: Used to specify the bit level layout of the hardware instruction using a string literal.
             * **`sideEffect`**: Used to specify the side effects of an instruction such as flag setting or changes in memory or registers.
             * **`microcode`**: Used to specify the low level microcode implementation of the hardware instruction using a sequence of micro operations specified by string literals.
             * **`latency`**: Used to specify the latency of a given instruction.
        *  Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time.
          * Meta-CAMs can be used to parse instruction set specifications, hardware description files, or other configuration files and then create new CAMs dynamically during compile-time.

    ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean #enable("dma") = false;
                         config functionParameterRegister : StringLiteral #register("r0") = "r0";
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

                    @callDSL {
                     callConvention myCallConvention {
                        parameterPassing {
                           firstParameter  #register(config.functionParameterRegister), // the register will be read from the config section.
                           otherParameters  stack
                       }
                      returnLocation r0,
                       prologue {
                          "push lr"
                       }
                      epilogue {
                          "pop pc"
                       }
                       stackAllocation {
                          allocateStack(frameSize);
                           freeStack(frameSize);
                       }
                     abiName "arm_std_abi"
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
                         memoryStore(dword, sp + 0x10, rd);
                         }
                         microcode {
                         "read r" + rn + ", dataReg"
                            "add r" + rd + ", dataReg"
                           "write r" + rd + ", dataReg"
                            }
                           latency = 2;
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
                       function void platformUartSendByte(uint8 data) #callConvention("myCallConvention"){
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
        * The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body to perform low-level hardware operations. The type of the register is inferred by the assigned value.
        *  The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        *  The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
         *  The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
        * The `hardwareInstructionDSL` block allows the developer to define the encoding of a hardware instruction by specifying the bit layout, side effects, operands, and microcode, using a declarative DSL.
        *  The `callDSL` block allows the developer to specify the application binary interface of a target architecture.
        *   Meta-CAMs are used to generate CAMs dynamically at compile time, using a hardware description or a configuration file.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        *   Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality, and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *    The import system will resolve conflicts with the last imported CAM overriding previous definitions.
         *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
        *   When a CAM is imported using `override function`, the new definition will override the previous one. The previous definition is discarded and not accessible. A compile-time error will be generated if the CAM attempts to use the function definition that is being overridden.
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
    *    A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
    *   The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
     *  If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
        *   CAM options can be enabled or disabled by using the `#enable("myOption")` or `#disable("myOption")` attributes. Example: `function void myFunc() #enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean #enable("myOption") = true; }`

*   **CAM API: Compiler Interface for CAMs**
    *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies. The `dataflowGraph` is composed of nodes and edges representing the data flow through the program. Each node has metadata and information about the operations that are performed including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges and to perform specific data flow analysis tasks.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    *   Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                *   `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer and so on. See enum `TypeKind` for possible types.
                *    `sizeof(DataType)`: Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                *   **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                    *   Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                *   **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                *   **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                 *    **`compilerWarning(StringLiteral message)`:** Generates a custom compiler warning message, allowing for more specific warning reporting.
                *    **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                     *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                    *  The implementation is a function pointer that must have the same signature as the `functionSignature` string.
                *   **`newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification)`:** Creates a new CAM with the given parameters. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton.
                     *  The `interfaceDefinitions` will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                      * The `implementation` will be a set of function pointers that define the implementations of the interface intrinsic functions.
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
                     *  **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                    *   **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                     *   **`target`**: A `TargetData` structure with information about the target including the type of the data being transformed, the memory region, the volatility state, and other metadata about the access.
                        *   **`TargetData`**: The TargetData structure provides information about the target, including:
                            *   `data`: The data type of the target.
                            *   `address`: The memory address of the target if it's a memory access operation.
                            *   `region`: The memory region of the target.
                            *   `isVolatile`: Returns true if the data is volatile.
                            *   `isRegister`: Returns true if the access is a register access.
                             *  `isMemoryMapped`: Returns true if the target is memory-mapped.
                             *   `hasAttribute(StringLiteral attributeName)`: Returns true if the target has a specific attribute.
                *   **`getHardwareDescription(StringLiteral hardwareDescription)`**: Loads a hardware description language file and returns the content as a String literal that can be used by CAMs.
                *   **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context otherwise `false`.
                *   **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                 *  **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                 * **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                  *  **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                *  **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                *   **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal. The pattern language uses the following syntax:
                    *   The pattern language supports the following syntax:
                      * `identifier`: Matches any identifier.
                       *  `stringLiteral`: Matches a specific string literal.
                       *  `integerLiteral`: Matches a specific integer literal.
                       *  `"..."`: Matches a sequence of characters that are present in the code.
                       *  `variable(identifier)`: Matches any variable and captures the identifier.
                        *   `immediate(identifier)`: Matches any immediate value and captures the identifier.
                       *   `instruction`: Matches any instruction.
                      *  `add(x, y)`: Matches an add instruction with two arguments.
                        *   `functionCall(x)`: Matches a function call to the function named `print`.
                         * `*`: Matches anything (wildcard).
                        *  `()`: Creates a group for the match.
                    *    Example:
                        * `"add(x, y)"`: This will match an add instruction with two arguments.
                        *   `"load(address)"`: Matches a load instruction with a parameter named `address`.
                        *    `"functionCall(print)"`: Matches a function call to the function named `print`.
                *   **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern. The captured subexpression will be returned using a string literal, that can be used in other code transformations. The same pattern language as for `match` is used to perform the capture.
                    *    Example: `capture("add(x, y)", code, "myVariable")` if `code` is `add(r0, r1)`, it will return a string `"r0,r1"` that can be used in other code transformations.
                *  **`createInstruction(StringLiteral instructionName, instructionParameters) -> Instruction`**: Creates a new `instruction` using the given parameters.
                  *  **`generateMicrocode(Instruction instruction) -> StringLiteral`**: Generates the low level microcode for a given instruction.
                   *  **`addTemplate(StringLiteral sectionName, StringLiteral templateCode);`**: Add a code template at a given section.
                   *  **`hasHardwareSymbolicSupport() -> boolean`:** returns `true` if the target platform provides hardware support for symbolic checks.
                    *  **`dispatchSymbolicCheck(StringLiteral operation, expression)`:** Dispatches a symbolic check operation to the hardware. The type of operation can be specified by the `operation` string.
                    *   **`getSymbolicCheckResult(address) -> boolean`:** Returns the result of the symbolic check given by the `address`.
                     *   **`hasHardwarePrefetchSupport() -> boolean`:** returns `true` if the target platform provides hardware support for prefetching.
                    *   **`dispatchPrefetch(dataAddress address, size) -> void;`**: dispatches a prefetch instruction to a specific memory location. The `size` parameter indicates how much data should be prefetched from the specified `address`.
                     *    **`getInstructionSchedulingHints(codeBlock) -> List<InstructionSchedulingHint>;`**: Gets a list of hints for instruction scheduling based on data dependencies using symbolic information. The codeBlock is the code region that should be analyzed by the CAM.
                    *   **`addTransformation(StringLiteral removeCode, StringLiteral insertCode, StringLiteral transformationType);`** Add a code transformation that will remove the code that matches the `removeCode` String literal and will insert `insertCode`, and will use the `transformationType` to perform a specific transformation.
                    * **`getCompilerFlag(StringLiteral flagName)`:** Get the value of a compiler flag.
                      * **`setCompilerFlag(StringLiteral flagName, StringLiteral flagValue)`:** Set the value of a compiler flag for a given region or function.
                      * **`emitCode(StringLiteral code, TargetData targetData)`**: Allows the CAM to directly inject code at a specific target location.
                      * **`skipTransformation();`**: Skips all further transformations for a specific code block.
            * Example:
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
                *  `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *    `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                 *    `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
                 *   `instructionBuilder.atomicRead(address, dataType,  bitmask)`: Creates an atomic read instruction.
                *  `instructionBuilder.atomicWrite(address, value, dataType, bitmask)`: Creates an atomic write instruction.
                  *   `instructionBuilder.atomicTestAndSetBit(address, bitNumber)`: Creates an atomic test and set bit instruction.
                   *   `instructionBuilder.atomicTestAndClearBit(address, bitNumber)`: Creates an atomic test and clear bit instruction.
                *   `instructionBuilder.atomicSetBit(address, bitNumber)`: Creates an atomic set bit instruction.
                 *   `instructionBuilder.atomicClearBit(address, bitNumber)`: Creates an atomic clear bit instruction.
                 *  `instructionBuilder.atomicToggleBit(address, bitNumber)`: Creates an atomic toggle bit instruction.
                *  `instructionBuilder.memoryBarrier(order)`: Creates a memory barrier instruction.
                *  `instructionBuilder.instructionBarrier(sync)`: Creates an instruction barrier instruction.
                 *  `instructionBuilder.rawInstruction(instructionPattern)`: Emits a raw instruction.
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
*  **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages using the `compilerError()` function.
*  **Custom Warnings and Information Messages:** CAMs can generate customized warning and information messages using `compilerWarning()` and `compilerInformation()` functions.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access:** Direct access to hardware registers using memory mapped access and conceptual registers.
*   **ABI Customization**: CAMs can customize the function call ABI using the `callDSL` block.
*   **Microcode Specification:** The microcode of an instruction can be specified using the `microcode` block.
*  **Performance Optimization**: Code regions can be tagged with performance attributes, to guide the compiler optimizations using a target specific implementation.
*  **Code Generation Bypass**: CAMs can bypass code transformations and provide a low-level implementation when necessary using the `emitCode` and `skipTransformation` compiler API functions.

**6. Directives and Pragmas (Updated)**

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
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion that makes the exported components of the module accessible. You can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
     *    Specific module versions can be imported using the `@` syntax. Example: `#use "MyModule@1.2.0";`
    *   `#use "camName" [ genericInstantiation ] [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code.
        *   Example: `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        *   Example: `#raw 0x10 0b00010000 16 0o20 "abc";`
    *   `#template(sectionName) { ... }` : This directive allows for injection of a code template into a specific section. The `sectionName` is defined in a CAM.
    *  `#performance(level, hint, ...)`: A pragma that provides performance information to the compiler.
           * `#performance(critical, memoryAccess, loopUnroll=4, ...)`: example.
             * The `level` is a string literal that can be used to specify the level of performance optimization such as `critical`, `high`, `normal` or `low`, and will be CAM specific, and may have different meanings for different target architectures.
            * The `hint` is a string literal that can be used to provide specific hints for a given code section such as `memoryAccess`, or `loopUnroll`.

*   **Operation Directives:**
        *  `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parentheses can be omitted if only one parameter is passed. Example: `print "Hello world";`.
         *  `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier depending on the target platform.
         *  `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *  `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parentheses can be omitted if only one argument is passed.
        * `atomicSetBit(register, bitNumber)`: Atomically sets a specific bit in a register.
        *   `atomicClearBit(register, bitNumber)`: Atomically clears a specific bit in a register.
        *   `atomicToggleBit(register, bitNumber)`: Atomically toggles a specific bit in a register.
        *   `atomicTestAndSetBit(register, bitNumber)`: Atomically tests a bit and sets it if clear. Returns true if the bit was clear before the operation.
        *   `atomicTestAndClearBit(register, bitNumber)`: Atomically tests a bit and clears it if set. Returns true if the bit was set before the operation.
        *   `atomicReadRegister(register, bitmask)`: Atomically reads a value from a register using a specific bitmask.
        *   `atomicWriteRegister(register, value, bitmask)`: Atomically writes a value to a register, applying a given bitmask.
      * `memoryBarrier(order);`: Generates a hardware specific memory barrier. The `order` can be `read`, `write`, `readWrite`, or `all`. If no parameter is specified, it defaults to `all`.
      *   `instructionBarrier(sync);`: Generates a hardware specific instruction barrier. The `sync` can be `instructionFetch`, `dataFetch`, or `dataWrite`.
       * `rawInstr(instructionPattern);`: Embeds raw machine code into the generated output. The `instructionPattern` is CAM specific.
        *   `hardwareSymbolicCheck(expression, range) -> void;` : Performs a hardware-assisted check if the expression satisfies the constraints given by the range. If no hardware support is present, the code will trap.
        *   `hardware_clz(uint32 value) -> uint32;` : Counts the number of leading zeros in the specified value using a hardware-accelerated instruction if available.
        *   `hardware_ctz(uint32 value) -> uint32;` : Counts the number of trailing zeros in the specified value using a hardware-accelerated instruction if available.
        *  `hardware_startTimer();` : Starts a hardware timer.
         * `hardware_endTimer();` : Reads the current value of the hardware timer.
         * `hardware_resetTimer();`: Resets the hardware timer.
          * `hardware_setWatchpoint(address, type);`: Sets a hardware watchpoint that can be used for debugging purposes. The type of the watchpoint can be `read`, `write`, or `readWrite`.
          * `prefetch(address, size, level);`: Dispatches a hardware prefetch instruction for a given `address` with a specific `size` and `level`.
          * `hardware_trap();`: Generates a hardware trap for debugging purposes when a low level error is detected.
        * `cacheFlush(address addr, size size, level level);` Flushes data from the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
        * `cacheInvalidate(address addr, size size, level level);` Invalidates data in the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
         *  `align(address, integerLiteral);` Aligns a given address to a specific alignment.
    *  **Atomic Operations:**
        *   `atomic { ... }`: Guarantees atomic execution of the code block using the primitives specified in the CAM. If no argument is specified, the compiler will attempt to atomically execute the instructions within the given block, which may involve software locking mechanisms if hardware primitives are not available, which might create an overhead depending on the target architecture and the compiler options. The code block is limited to low level access primitives, and calls to CAM provided `intrinsic` functions that can be mapped to target-specific atomic primitives.
        *   `atomic (variable) { ... }`: Guarantees atomic access and modification of the specified `variable` using the underlying primitives provided by the CAM. The scope is limited to operations involving that specific variable. The compiler will attempt to generate code to atomically read, modify and write to that specific memory location or variable. If the hardware doesn't have a direct atomic read-modify-write instruction the CAM may have to generate code that involves locks and memory barriers to ensure atomicity.
        *  `atomic memoryStore(...)`: Performs an atomic memory store operation.
         * `atomic register identifier = ...;`: Performs an atomic write to a given register using an atomic instruction when it is available.
    *   **Memory Mapped Registers:**
        *  `struct #mapped`: Defines a memory-mapped structure where the members are implicitly `volatile` unless marked `nonvolatile`. Allows for direct access to hardware registers. The `#mapped` specifier specifies that the struct is memory mapped. Members are accessed using the `.` operator, and bitfields can be accessed using the `=` or `:=` operator inside a `with` statement, or directly when assigning to a memory-mapped structure. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using bitmasks: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`, or a mix of the two. Bitfield updates should be performed using a `with` statement to minimize the overhead of reading and writing to memory mapped registers. The `with` statement groups multiple bitfield updates into a single operation. The compiler analyses the `with` block to check if a specific write or read operation to a memory mapped register can be avoided. If a register is read and not modified, the write operation can be skipped. This mechanism is automatically performed by the compiler and requires no additional effort from the user enabling zero-overhead high-performance bitfield access. When the `#initializerList` attribute is added to a `struct`, initialization can be done using a list of initializers: `type GpioRegisters = struct #mapped(0x40000000) #initializerList { uint32 dataRegister; uint32 controlRegister; };  GpioRegisters myGpio = { 10, 20 };`.
         * Example:
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
                       uint32 status = readout; //no memory read here as the bitfield is not modified
                   }
                 myGpioRegisters.enable = 0; // single bitfield modification using implicit syntax.
                 myGpioRegisters.baud = 115200; // single bitfield modification using implicit assignment.
                  myGpioRegisters.controlRegister = {enable=1, baud=115200}; //bitfield assignment using struct like syntax
            ```
        *  `.` accesses members. If the member is inside a memory-mapped struct, it will access the memory location directly.
        *   **`=` sets/reads bitfields inside a memory-mapped struct.**
        *   **Bit Ranges: `[start..end]` accesses a range of bits in a bitfield inside a memory-mapped struct.**
        *   **Bit Masks: `[binaryMask]` accesses the bits defined by a binary mask in a bitfield inside a memory-mapped struct.**
        *   **Combined Selectors: `[range, bit, mask, ...]` accesses multiple range of bits or individual bits inside a bitfield of a memory-mapped struct.**
        *   Memory Mapped Registers: Specific registers can be defined using the following syntax:
            *   `DataType #register("regionName", "stringLiteral") registerName;` This syntax defines a memory-mapped register located in a specific peripheral region. The `stringLiteral` is CAM specific and is intended to provide additional information to CAM implementations. The type must be a base type and cannot be a struct or a union. This attribute can only be used with `register` declarations. The compiler will perform checks to guarantee that the data type of the declared register matches the register's underlying size. If there is a mismatch a compile-time error will be generated. The register name can be accessed and modified as if it was a regular variable. The compiler will emit code that directly reads/writes to that memory location. Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
             * Registers can be defined with access modifiers, using the following syntax:
               *  `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
               *  `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
                * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Enum Types:** `enum` (named constants).
        *   A way to define named constants improving the readability of the code. If the enum is declared with a simple comma-separated list of values curly braces can be removed: `enum ErrorCode Ok, NotFound, AccessDenied;`
         * Enums can be initialized with specific values using a `struct`-like syntax:
          ```baremetal
           enum ErrorCode { Ok=0, NotFound=1, AccessDenied=2 };
           enum MyEnum { Value1 = 10, Value2 = 20, Value3 = 30 };
          ```
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
        *   The constraint can be an arbitrary expression, a range or a length constraint, and can also include compile-time functions. Refinement types cannot have type parameters.
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
            *   CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
        *   Example: `type SecureData = uint32 @region("secureRegion");`
    *   **Capability-Based Regions:** `dataRegion regionName [ regionCapabilitySpecifier ] { ... };`
        *   Data regions are used for compile-time region-based memory access controls and memory management, allowing compile-time checking of memory accesses and capabilities, increasing memory safety. If only one member is present curly braces can be removed. `dataRegion MySecureData uint32 secretKey;`
        *   The `regionCapabilitySpecifier` argument specifies which permissions are available: `read`, `write`, `execute`, `capability_derive`, `peripheral`.
            *   If only `read` or `read, write` capabilities are needed, the `capabilities` keyword can be omitted: `dataRegion @read MySecureData uint32 secretKey;`, `dataRegion DeviceData { uint8 safe[100] buffer; };` (implicit read, write capabilities).
        *   `capability_derive` allows creating new pointers with limited capabilities from a pointer that has the `capability_derive` permission, granting a way to control how a pointer is used within a specific scope or region.
        *   `peripheral` allows to access a memory mapped peripheral using the mechanisms defined by the CAM.
        *   A memory region must be declared using this syntax to be used in `@region(regionName)` pointers, otherwise, the compiler will generate a compile-time error.
        *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe [100] buffer; };`
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
        *  `sizeof dataType`: is also a valid syntax without parentheses.
        *   `isType(DataType, "Integer")`: Returns `true` or `false` if the first argument is of the specified type, which is useful to implement compile-time checks and to conditionally compile CAM code based on the type of the data. Possible types are defined in `enum TypeKind { Integer, Float, Struct, Enum, Pointer, Array, Tuple, Result, Refinement , CodeBlock};`.
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
                   statements {
                       function void optimizeData(DataType value) {
                         if (hasDataLayoutAttribute(value, "packed")) {
                            //generate code for packed struct
                         } else {
                         // generate standard code
                         }
                    }
                  }
                }
            }
            ```
    *   **Linear Access Attributes:** `#linearAccess`, `#linearAccessGuaranteed(size)`.
        *   `#linearAccess`: Marks an array as being accessed sequentially. The compiler may use this information to perform optimizations or prefetching if combined with `#prefetch` attribute.
        *   `#linearAccessGuaranteed(size)`: Guarantees that the array access is linear, and the size is a compile-time constant or an identifier marked with `linearAccessGuaranteed`, which is required to generate code for hardware accelerators that perform linear memory access. The compiler must ensure that all accesses to that array are linear, otherwise, it is a compile-time error.
        *   When `size` is not specified, the compiler must infer it from the array declaration.
        *   Example: `uint32 safe [100] myArray #linearAccess;`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed(100);`, `uint32 safe[MAX_SIZE] myLinearArray #linearAccessGuaranteed;`
    *   **Circular Buffer Attribute:** `#circularBuffer`.
        *   Marks an array as a circular buffer, enabling specific code optimizations and transformations by using modulo operations or other strategies to implement a ring buffer. If combined with the `#prefetch` attribute it can also be used for prefetching.
        *   The compiler must verify that the size of the buffer is a power of 2, otherwise it is a compile-time error.
        *   Example: `uint32 safe [1024] myBuffer #circularBuffer;`
    *   **Read and Write Once Attributes:** `#readOnce`, `#writeOnce`
        *   The `#readOnce` attribute guarantees that a given variable or memory region will be read only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is read more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#readOnce` attribute is violated.
        *   The `#writeOnce` attribute guarantees that a given variable or memory region will be written only once. The compiler may perform optimizations or generate specialized code for that. If the variable or memory region is written more than once, the behavior is undefined. No guarantees about the order of the access are provided. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
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

    *   **Function Parameter Type Inference:**
        *   BML supports type inference for function parameters when the `#in`, `#out`, or `#inOut` attributes are used.
        *   **`#in`:** The parameter is an input parameter. The compiler will infer its type based on how it's used in the function body. The inferred type will be the type of the expression that is passed as an argument to the function.
        *   **`#out`:** The parameter is an output parameter. Its type must be explicitly specified, as the compiler cannot infer it from the function body alone. The function must assign a value of the specified type to this parameter.
        *   **`#inOut`:** The parameter is both an input and an output parameter. The compiler will infer its type from the argument passed to the function. The function can both read from and assign to this parameter. The argument passed to a function with an `#inOut` parameter must be a variable of a type that is compatible with the inferred type and that can be modified.
        *   **Inference Rules:**
            *   The compiler will analyze the function body to determine the types of `#in` and `#inOut` parameters based on their usage.
            *   If a parameter is used in a way that is incompatible with multiple types, the compiler will generate an error.
            *   If the type of an `#in` or `#inOut` parameter cannot be uniquely inferred, the compiler will generate an error (E0052) and the user must specify the type explicitly.
            *   Type inference does not cross function boundaries. The compiler will not analyze calls to other functions to infer types.
            *   Type inference for function parameters is performed independently for each `@arc` block.

            ```baremetal
            @arc arm {
                statements {
                    function void processData(uint32 data #in, uint32 result #out, uint32 extra #inOut) {
                        result = data * 2;
                        extra~ = extra + result;
                    }

                    function void main() {
                        uint32 x = 10;
                        uint32 y;
                        ~uint32 z = 5;
                        processData(x, y, z); // Type of 'x' and 'z' are inferred as uint32
                    }
                }
            }
            ```
    *   **Transformation Attribute:** `#transform(stringLiteral)`.
        *   This attribute allows transforming the data using target-specific transformations. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `function void applyTransform(uint32 data) #transform("myTransform") { ... };`
    *    **Allocator and Unit Attributes**: `#allocator(stringLiteral)`, `#unit(stringLiteral)`
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
        *   This attribute is used to specify a mapping to a memory mapped register. The `stringLiteral` is CAM specific and is intended to provide additional information for code generation. This attribute can only be used with `register` declarations.
        *  This attribute does not change the code behaviour; it is used by CAMs for code generation and analysis.
        *   Example: `uint32 #register("MyAiCore", "control_reg") myMatrixAControl;`
           * Registers can be defined with access modifiers, using the following syntax:
             * `register identifier #register("regionName", "stringLiteral") : dataType #readOnly` This specifies a register that can only be read from.
             * `register identifier #register("regionName", "stringLiteral") : dataType #writeOnce` This specifies that a register can only be written once. If written more than once, the behavior is undefined. The compiler will not check or generate a warning if the `#writeOnce` attribute is violated.
             * `register identifier #register("regionName", "stringLiteral") : dataType #threadLocal` This indicates that the register is thread-local, implying that each thread has its own copy, initialized to the default value of the data type. If no default value is defined, the default value will be zero.
             * Multiple access modifiers can be combined: `register identifier #register("regionName", "stringLiteral") : dataType #readOnly #writeOnce #threadLocal`
    *   **Hardware Description Attribute:** `#hardwareDescription(stringLiteral)`.
        *   This attribute allows associating a specific hardware description to a given interface intrinsic function. This attribute is used when CAMs are automatically generated from hardware descriptions. The `stringLiteral` is CAM specific.
        *   This attribute does not change the code behavior; it is used by CAMs for code generation and analysis.
        *   Example: `interface intrinsic function void platformInit(uint32 baseAddress) -> void #hardwareDescription("timer.init");`
    *   **Enable and Disable Attributes:** `#enable(stringLiteral)`, `#disable(stringLiteral)`.
        *   These attributes provide a way to enable or disable specific functionalities of a given CAM or a function by using a string parameter that maps to a given boolean configuration variable.
    *    **Algorithmic code generation attribute:** `#algorithm(stringLiteral)`.
        *   This attribute specifies that a given function or code region should be compiled using a specific algorithmic code generator instead of relying on traditional code generation techniques.
    *   **Symbolic Execution:** `symbolic` and associated intrinsic function calls `symbolicResult(identifier)`, `assert()`, `hasSymbolicRegion()`, `symbolicExpression(identifier)`.
        * The `symbolic` keyword is used to create a symbolic block where the compiler will track the symbolic values of the expressions and variables. The `symbolicResult()` will provide the compiler with the symbolic expression for further analysis or optimizations. The `assert()` operation will perform a symbolic assertion check at compile time. The `hasSymbolicRegion()` function returns `true` if there is a symbolic region in the current program context, otherwise it returns false, The `symbolicExpression(identifier)` returns the symbolic expression for a given identifier in the current program context.
    *   **Implicit Type Conversions:**
        *   BML does **not** allow implicit type conversions between different data types, except for integer types of the same signedness and from smaller to larger types. Conversions from smaller to larger integers are implicitly allowed. E.g., `uint8` is implicitly convertible to `uint16`, `uint32`, and `uint64` but not to `int8`, `int16`, `int32`, `int64`, or `float32` and not to other types such as `char` or `boolean`.
            *   Implicit type conversion from signed to unsigned integers is not allowed and requires an explicit cast.
        *   The `as` operator is used to perform an explicit type conversion for the following types:
            *   Any integer type to a larger integer type of *the same signedness*. (e.g., `uint8 as uint32`, `int16 as int64`).
            *   Any integer type (signed or unsigned) to a floating point type (e.g., `uint32 as float32`, `int16 as float64`).
            *   Floating-point types to other floating-point types of different precision (e.g., `float32 as float64`).
            *   `DataType^` to `genericAddress`, `dataAddress` or `codeAddress` and vice versa.
            *   Types that are compatible for implicit type conversion (like a formal parameter of a function that is implicitly compatible).
            *   The compiler will perform a compile-time error if the type conversion is invalid.
        *   Implicit conversions are also performed if a function has a formal parameter of the same type or a type compatible for implicit conversion and when returning from functions if the return type is compatible for implicit conversion.
        *   No other implicit type conversions are allowed. Conversions must be explicit by using the `cast` operator or by calling conversion functions from the standard library or provided by the CAMs, by using the `into` keyword, by using the `to` method or by using custom suffixes.
    *   **Implicit Type Inference:** Type inference is used in the following cases:
        *   When the return type of a function is not explicitly specified, the compiler will infer the return type by analyzing the `return` statements or generate a compile-time error if no return statement  is present and the return type cannot be inferred.
        *   When declaring an array literal the compiler infers the array type from the literal values.
        *   When a variable is assigned an integer literal and the type can be inferred from the context if not an architecture-dependent integer (usually 32 bits) is used by default. When a variable is assigned a float literal the default type is `float32`.
       *   **When a tuple variable is declared without an explicit type.** The compiler will infer the type from the values in the tuple literal.
        *   When a local variable is initialized with an expression inside the function scope.
        *   When a register variable is initialized with a given value in the CAM scope.
    *   **Scope:**
        *   BML uses lexical scoping, defining variable visibility and mutability according to the block it is declared.
        *   Mutability is block-scoped; Variables are immutable by default in the block where they are declared. Mutable variables must be declared with the `~` mutability modifier.
        *   **Nested functions:** Functions can be nested within other functions. Inner functions have access to variables in their enclosing function's scope (closure).
         *   Nested functions can only be defined within `@arc` blocks. They are not allowed in the `@arc common` block.
        *   **Shadowing:** Inner scopes can declare variables with the same name as variables in outer scopes. The inner declaration shadows the outer one. The compiler will issue a warning in such cases.
        *   **Global Scope:** Variables declared at the top level of an `@arc` block are considered global for that architecture.
    *   **Instruction Type:**
        *   The `instruction<instructionName, [parameters]>` type is used to instantiate low-level hardware instructions.
        *   The `instructionName` should match a previously defined instruction using the `hardwareInstructionDSL`.
        *   The `parameters` are a list of compile-time expressions that can be used to specify the operands of the instruction or any other parameter specific to the instruction.
        *   The `instruction` type is a base type, similarly to the integer, float and rawbyte types, and can be used as the type of a given variable, a function parameter, or a return type of a function.
        *   Example: `instruction<add, r0, r1, 10> myAddInstruction;`
    *   **Code Block Type:** `block (parameterList) -> returnType;`.
        * A type that defines a function signature for a code block.
        *  **Default value**: The default value of a code block is `null`.
        * Example: `block (uint32, uint32) -> uint32 MathOperationType;`
       * Code blocks can be captured using lambda expressions: `myCodeBlock = (a,b) => a+b;`.
    *    **Bitmask Type:** A constant expression that is used to describe a bitfield using an integer value. Example: `const bitmask ENABLE_BIT = 0x01;`

**3. Modules and the Main File: Structure and Organization (Updated)**

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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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

*   **Module Usage:**
    *   `#use "moduleName";`: Includes exported items from the module in the current scope making the exported components of the module visible in the current scope. When using the `#use` directive you can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
       *  Module aliases:  Modules can be imported using an alias to avoid namespace conflicts, or when the module name is too long `#use "MyLongModuleName" as MyModule`.
        * Specific versions of the modules can be imported using an `@` syntax. Example: `#use "MyModule@1.2.0";`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
            * Example:
            ```baremetal
                module MyModule {
                    config {
                       config dataSize : uint32 = 16;
                  }
                  statements {
                      function void myFunction(uint32 data) {
                        print("Data size:", config.dataSize);
                      }
                   }
                }
                 module MyOtherModule {
                 statements {
                      #use "MyModule" with config { dataSize = 32 };
                       function void myOtherFunction() {
                          myFunction(10);
                       }
                   }
               }
            ```
            *   In the example above, the function `myFunction` will use the dataSize of `32` which overrides the default `dataSize` value.
         *   **Module Configuration Files:** Modules can load configuration data by specifying a `StringLiteral` that specifies a configuration file, which is loaded and parsed at compile time. The compiler uses the parser that is specified by the type annotation in the configuration section.
          ```baremetal
            module MyModule {
                config {
                  config myConfig : MyConfig =  parseConfigFile("config.txt");
                }
            }
          ```

*   **Main File:**
    *   **The main file is the entry point of the BML application.**
    *   **It is defined in a separate `.bml` file (typically `main.bml`).**
    *   **The main file does *not* use the `module` keyword.**
    *   **It does *not* contain a `main` function.**
    *   **Instead, the `statements` block within each `@arc` block acts as the entry point for that specific architecture.**
     *  **It must have an explicit `@arc common` block to contain architecture-independent code.** Code outside of any `@arc` block is not allowed in the `main.bml` file.
     *  The `@arc common` block is a global scope for all the `@arc` blocks in the `main.bml` file, and code within the `@arc common` block is treated as architecture-independent, meaning that it will be compiled for all target architectures. Direct access to hardware is not allowed in the `@arc common` block. Nested functions are not allowed inside the `@arc common` block, and all function calls are limited to only be able to call other functions inside the `@arc common` block.
    *   **It uses the `#use` directive to import modules.**
    *   The main file can also contain `@arc` blocks for architecture-specific code.
    *   Example (`main.bml`):
        ```baremetal
        #use "MyModule";
         #use "MyGenericCam<uint32>" as GenericUint32Cam;

         @arc common {
            declarations {
                 uint32 commonVar = 5;
            }
            statements {
                 function void commonFunction() {
                  // This function is available to all architectures
                  print("Common function called");
              }
            }
        }


        @arc arm {
            declarations {
                // ... ARM-specific declarations ...
            }
            statements {
                // ... ARM-specific initialization ...
                 commonFunction(); // Can call functions from common section
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
                commonFunction(); // Can call functions from common section
                myFunction(); // Call a function from MyModule
                // ... More RISC-V specific code ...
            }
        }
        ```

**4. Composable Architecture Modules (CAMs) (Updated)**

*   CAMs are compiler extensions that act as "code generation engines." They allow extending the compiler and generating platform-specific code in a type-safe and modular fashion, using a rule-based system, explicit hardware intrinsics, and integration with hardware description languages.
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM. (CAMs are modules)
        *   `description "CAM Description String";`: (Optional) A description of the CAM which is used by documentation generation tools.
            *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters used to customize the behavior of the CAM, based on the user's needs. CAMs can load external data from a file by specifying the location in the `config` section and using a parsing function. The `config` parameters can be customized using command line arguments or by using a `with config { ... }` section when the CAM is imported using the `#use` directive. CAM options can be enabled or disabled using the `#enable("optionName")` or `#disable("optionName")` attributes. If the user does not specify a default value for a configuration parameter, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
            *   Example: `config { config baudRate: uint32 = 115200;  config hardwareDescription : StringLiteral; config enableDma : boolean #enable("dma") = false; config myOption : CAMOption = {true, 100};}`.
             *  The `config` section can also load a file by specifying the filename in a `StringLiteral` and then processing the file by using a compile-time function that is able to parse the data.
                 * Example: `config { config hardwareDescriptionFile : StringLiteral; config registers : RegisterConfig =  parseRegisterData(hardwareDescriptionFile) ; };` In this example `parseRegisterData` must be a `compileTimeFunction` and `RegisterConfig` a user defined data type.
                *  The `#enable("optionName")` and `#disable("optionName")` attributes can be used to conditionally compile code. Example: `function void myFunction() #enable("myOption") { /*...*/ }` or `config { config enableOpt : boolean #enable("myOption") = true; }`. The `myOption` configuration parameter is defined as `config myOption : CAMOption = {true, 100};`.
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
          * **Module Visibility:** The keyword `private` can be used to limit visibility of exported data, functions or constants to other modules that are also using this module using the `#use` directive:
                ```baremetal
                module MyModule {
                    export {
                        private functions { myPrivateExportedFunction } //not available to the main module, unless the main module also uses `MyModule`.
                        types { MyExportedType }
                        constants { myExportedConstant }
                    }
                     function private myPrivateExportedFunction() { ... } // this function can only be used by other modules that are using `MyModule`.
                }
               ```
        *   **Architecture-Specific Code (`@arc` blocks):** All code within a module must be enclosed within an `@arc` block targeting a specific architecture. Each module can have multiple `@arc` blocks for different architectures. Code outside an `@arc` block within a module is a compile-time error (E0012).
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
         * The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
          * The `callDSL` block is used to customize the Application Binary Interface (ABI) of a given function call, by providing a mechanism to specify the calling conventions, the parameter passing mechanism, the stack frame handling and the instructions to setup and teardown a function call.
             *   **`callConvention` Block**: Defines the calling convention for a given architecture, by specifying how the stack is allocated, how the parameters are passed, the register usage, the prologue, the epilogue, and other information about the application binary interface.
             *   **`parameterPassing`:** Specifies how the parameters are passed when calling a function. Parameters can be passed using specific registers or using the stack.
             *  **`returnLocation`:** Specifies how the return value is passed to the caller function.
             *  **`prologue`:** Specifies the code that is executed before a function starts, and usually includes the instructions to save the registers in the stack, and other setup instructions.
             *  **`epilogue`:** Specifies the code that is executed before a function returns, and usually includes the instructions to restore the registers and pop the stack.
             *   **`stackAllocation`:** Defines how the stack memory is allocated and deallocated for a function call.
             *   **`abiName`:** Provides the name of the application binary interface that the compiler must implement when generating the code, which is used to guarantee compatibility with other libraries and code.
            *    **`hardwareInstructionDSL`**: Provides a mechanism to specify the low-level implementation of a hardware instruction.
             * **`encoding`**: Used to specify the bit level layout of the hardware instruction using a string literal.
             * **`sideEffect`**: Used to specify the side effects of an instruction such as flag setting or changes in memory or registers.
             * **`microcode`**: Used to specify the low level microcode implementation of the hardware instruction using a sequence of micro operations specified by string literals.
             * **`latency`**: Used to specify the latency of a given instruction.
        *  Meta-CAMs are declared by using the `Meta` keyword before the `module` keyword. Meta-CAMs are executed at compile time.
          * Meta-CAMs can be used to parse instruction set specifications, hardware description files, or other configuration files and then create new CAMs dynamically during compile-time.

     ```baremetal
                module UartCam {
                     config {
                        config baseAddress: uint32 = 0x0;
                         config hardwareDescription : StringLiteral;
                        config enableDma : boolean #enable("dma") = false;
                         config functionParameterRegister : StringLiteral #register("r0") = "r0";
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

                    @callDSL {
                     callConvention myCallConvention {
                        parameterPassing {
                           firstParameter  #register(config.functionParameterRegister), // the register will be read from the config section.
                           otherParameters  stack
                       }
                      returnLocation r0,
                       prologue {
                          "push lr"
                       }
                      epilogue {
                          "pop pc"
                       }
                       stackAllocation {
                          allocateStack(frameSize);
                           freeStack(frameSize);
                       }
                     abiName "arm_std_abi"
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
                         memoryStore(dword, sp + 0x10, rd);
                         }
                         microcode {
                         "read r" + rn + ", dataReg"
                            "add r" + rd + ", dataReg"
                           "write r" + rd + ", dataReg"
                            }
                           latency = 2;
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
                      function void platformUartSendByte(uint8 data)  #callConvention("myCallConvention"){
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
        *  The `register` directive is used to declare variables (registers) in the scope of the intrinsic function, which can be used inside the function body to perform low-level hardware operations. The type of the register is inferred by the assigned value.
        *   The `memoryStore`, `memoryLoad` functions are the main interface to low-level memory accesses.
        *   The `interface intrinsic function` is not callable directly by the user. It is intended to be called by the CAM itself using the `interface intrinsic function` entry point.
        *   The architecture name can be one of: `arm`, `riscv`, `x86`, `intel`, or any valid identifier.
        * The `instrDSL` block can be used to define a domain specific language that is used to define symbolic instructions which may have generic parameters that are evaluated at compile time. The `rule` keyword is used to define a declarative code transformation.
          * The `InstructionBuilder` object can be used inside the `instrDSL` block to create symbolic instructions and to access the value of their type parameters.
         *   The `hardwareInstructionDSL` block allows the developer to define the encoding of a hardware instruction by specifying the bit layout, side effects, operands, and microcode, using a declarative DSL.
            *   The `callDSL` block allows the developer to specify the application binary interface of a target architecture.
        *   Meta-CAMs are used to generate CAMs dynamically at compile time, using a hardware description or a configuration file.
    *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g., `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g., `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g., `"MyCam.bml"`), `documentation_file` (e.g., `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`). The manifest file is used by the compiler to find the CAMs, to perform version compatibility checks, and to provide documentation for the user.
        *   Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Imports:** CAMs can now import other CAMs, inheriting their functionality, and extending or modifying their behaviors.

    *   **Mechanism:**
        *   Similar to `#use`, a CAM can use `#use "otherCam"` to import another CAM. This makes the imported CAM's interface intrinsic functions available within the importing CAM.
        *   Imported CAMs can also redefine interface intrinsic functions, allowing for overriding and specialization.
        *    The import system will resolve conflicts with the last imported CAM overriding previous definitions.
         *   CAM imports can be generic by using type parameters `#use "GenericCam<uint32>";` This enables code reusability and specialization of generic CAMs.
        *   When a CAM is imported using `override function`, the new definition will override the previous one. The previous definition is discarded and not accessible. A compile-time error will be generated if the CAM attempts to use the function definition that is being overridden.
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
    *    A `CAMOption` type can be created to define custom options: `type CAMOption = struct {boolean enabled; uint32 value;};`. The `CAMOption` type can be used as a field inside the `config` section of a given CAM: `config {  config myOption : CAMOption = {true, 100}; }`.
    *   The CAM developer can check the value of the specific options in the CAM code and provide customized behavior for a given CAM, based on the configured options, which can be passed to the compiler by using command-line flags.
     *  If the user does not specify a default, it is assumed that the default value will be used, unless another value is specified by the user using command line flags.
        *   CAM options can be enabled or disabled by using the `#enable("myOption")` or `#disable("myOption")` attributes. Example: `function void myFunc() #enable("myOption") { /*...*/ }`, or by setting a default value on the config section `config { config enableOpt : boolean #enable("myOption") = true; }`

*   **CAM API: Compiler Interface for CAMs**
    *   CAMs, as compiler extensions, interact with the BML compiler through a defined API, enabling them to analyze, transform, and generate code. This section outlines the key components of this API:

        *   **`CompilerContext` Object:**
            *   The central object representing the compilation context provided to each CAM function.
            *   Provides access to:
                *   **`dataflowGraph`:** A representation of the program's dataflow graph, including types, operations, memory access, and data dependencies. The `dataflowGraph` is composed of nodes and edges representing the data flow through the program. Each node has metadata and information about the operations that are performed including input and output parameters. The `edges` field contains the information about data dependencies. The compiler also provides helper methods to iterate through the nodes and edges and to perform specific data flow analysis tasks.
                    *   Example: `CompilerContext.dataflowGraph.nodes` (returns all nodes in the dataflow graph). CAMs can use the dataflow graph to perform complex optimizations based on data dependencies.
                *   **`moduleDeclarations`:** Information about all modules used in the project.
                    *   Example: `CompilerContext.moduleDeclarations.getModule("MyModule").declarations` to access module declarations.
                *   **`typeSystem`:** Allows inspection of data types, including attributes.
                    *   Example: `CompilerContext.typeSystem.getType("uint32").size` (returns the size of `uint32`). Also: `CompilerContext.typeSystem.hasAttribute(DataType, "packed")` checks if a type has the `packed` layout attribute.
                *  `isType(DataType, "Integer")` to identify if a specific type is an integer, `isType(DataType, "Float")` to identify float types, `isType(DataType, "Pointer")` to check if the type is a pointer and so on. See enum `TypeKind` for possible types.
                *    `sizeof(DataType)`: Returns the size (in bytes) of the specified type at compile time.
                *   **`targetArchitecture`**: The target architecture name (`arm`, `riscv`, etc.).
                    *   Example: `CompilerContext.targetArchitecture`
                *   **`config`:** Provides access to the configuration parameters specified in the `#use` directive.
                    *   Example: `CompilerContext.config.baudRate` will return the value associated with the `baudRate` configuration variable.
                *   **`hasDataLayoutAttribute(DataType, "packed")`**: Checks if a given data type has a specific layout attribute.
                *   **`compilerError(StringLiteral message)`:** Generates a custom compiler error message, allowing for more specific error reporting.
                 *    **`compilerWarning(StringLiteral message)`:** Generates a custom compiler warning message, allowing for more specific warning reporting.
                 *   **`compilerInformation(StringLiteral message)`:** Generates a custom compiler information message that provides information about the compilation process and can be used for debugging purposes.
                *    **`registerInterfaceFunction(StringLiteral name, StringLiteral functionSignature, FunctionPtr implementation)`:** Adds a new interface intrinsic function dynamically.
                     *   The `functionSignature` is a string that includes the function parameters and return type. Example: `function(uint32, uint32) -> uint32`
                    *  The implementation is a function pointer that must have the same signature as the `functionSignature` string.
                *   **`newCAM(StringLiteral camName, interfaceDefinitions, implementation, hardwareSpecification)`:** Creates a new CAM with the given parameters. The `hardwareSpecification` is an optional parameter, which specifies the location of an hardware description language file, that is used to generate the CAM skeleton. *  The `interfaceDefinitions` will be defined by a set of strings that define the interface intrinsic functions of the CAM.
                      * The `implementation` will be a set of function pointers that define the implementations of the interface intrinsic functions.
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
                *  **`addTransformation(Transformation transformation)`:** Adds a transformation to the code which is then performed by the compiler during a code generation phase.
                     *   The `Transformation` object contains:
                       *   **`removeCode`**: The code that should be removed specified as a string.
                       *  **`insertCode`**: The code that should be inserted instead of the code that is being removed.
                       *   **`transformationType`**: The type of transformation that should be applied. Some possible types include: `instructionReplacement`, `bitfieldOptimization`, `codeReordering`.
                       *   **`target`**: A `TargetData` structure with information about the target including the type of the data being transformed, the memory region, the volatility state, and other metadata about the access.
                            *   **`TargetData`**: The TargetData structure provides information about the target, including:
                                *   `data`: The data type of the target.
                                *   `address`: The memory address of the target if it's a memory access operation.
                                *   `region`: The memory region of the target.
                                *   `isVolatile`: Returns true if the data is volatile.
                                *   `isRegister`: Returns true if the access is a register access.
                                 *  `isMemoryMapped`: Returns true if the target is memory-mapped.
                                  *   `hasAttribute(StringLiteral attributeName)`: Returns true if the target has a specific attribute.
                *   **`getHardwareDescription(StringLiteral hardwareDescription)`**: Loads a hardware description language file and returns the content as a String literal that can be used by CAMs.
                *   **`hasInstruction(StringLiteral instructionName)`**: Returns `true` if the instruction is present in the current context otherwise `false`.
                *   **`getInstructionCode() -> StringLiteral`**: Returns the current instruction as a string, which can be used to remove or modify it.
                 *  **`instructionTarget() -> TargetData`**: Returns a `TargetData` structure that contains information about the current code target.
                 *  **`symbolicExpression(identifier) -> StringLiteral`**: Returns the symbolic expression of the specified variable or code expression.
                 *  **`hasSymbolicRegion() -> boolean`**: Returns true if there is a symbolic region in the current program context.
                *  **`getAttribute(identifier, StringLiteral attributeName) -> StringLiteral`:** Returns the string literal of the attribute applied to a given code expression or data type.
                *   **`match(StringLiteral pattern, StringLiteral expression) -> boolean`**: checks if a specific code expression matches a specific pattern using a string literal. The pattern language uses the following syntax:
                    * The pattern language supports the following syntax:
                      * `identifier`: Matches any identifier.
                       *  `stringLiteral`: Matches a specific string literal.
                       *  `integerLiteral`: Matches a specific integer literal.
                       *  `"..."`: Matches a sequence of characters that are present in the code.
                       *  `variable(identifier)`: Matches any variable and captures the identifier.
                       * `immediate(identifier)`: Matches any immediate value and captures the identifier.
                       *   `instruction`: Matches any instruction.
                      *  `add(x, y)`: Matches an add instruction with two arguments.
                        *   `functionCall(x)`: Matches a function call to the function named `print`.
                         * `*`: Matches anything (wildcard).
                        *  `()`: Creates a group for the match.
                    *    Example:
                        * `"add(x, y)"`: This will match an add instruction with two arguments.
                        *   `"load(address)"`: Matches a load instruction with a parameter named `address`.
                        *   `"functionCall(print)"`: Matches a function call to the function named `print`.
                *   **`capture(StringLiteral pattern, StringLiteral expression, identifier) -> StringLiteral`**: Capture the subexpression inside a given pattern. The captured subexpression will be returned using a string literal, that can be used in other code transformations. The same pattern language as for `match` is used to perform the capture.
                    *    Example: `capture("add(x, y)", code, "myVariable")` if `code` is `add(r0, r1)`, it will return a string `"r0,r1"` that can be used in other code transformations.
                *   **`createInstruction(StringLiteral instructionName, instructionParameters) -> Instruction`**: Creates a new `instruction` using the given parameters.
                  *  **`generateMicrocode(Instruction instruction) -> StringLiteral`**: Generates the low level microcode for a given instruction.
                   *  **`addTemplate(StringLiteral sectionName, StringLiteral templateCode);`**: Add a code template at a given section.
                   *  **`hasHardwareSymbolicSupport() -> boolean`:** returns `true` if the target platform provides hardware support for symbolic checks.
                    *   **`dispatchSymbolicCheck(StringLiteral operation, expression)`:** Dispatches a symbolic check operation to the hardware. The type of operation can be specified by the `operation` string.
                    *  **`getSymbolicCheckResult(address) -> boolean`:** Returns the result of the symbolic check given by the `address`.
                     *   **`hasHardwarePrefetchSupport() -> boolean`:** returns `true` if the target platform provides hardware support for prefetching.
                    *   **`dispatchPrefetch(dataAddress address, size) -> void;`**: dispatches a prefetch instruction to a specific memory location. The `size` parameter indicates how much data should be prefetched from the specified `address`.
                     *    **`getInstructionSchedulingHints(codeBlock) -> List<InstructionSchedulingHint>;`**: Gets a list of hints for instruction scheduling based on data dependencies using symbolic information. The codeBlock is the code region that should be analyzed by the CAM.
                    *   **`addTransformation(StringLiteral removeCode, StringLiteral insertCode, StringLiteral transformationType);`** Add a code transformation that will remove the code that matches the `removeCode` String literal and will insert `insertCode`, and will use the `transformationType` to perform a specific transformation.
                    *   **`getCompilerFlag(StringLiteral flagName)`:** Get the value of a compiler flag.
                      * **`setCompilerFlag(StringLiteral flagName, StringLiteral flagValue)`:** Set the value of a compiler flag for a given region or function.
                    *  **`emitCode(StringLiteral code, TargetData targetData)`**: Allows the CAM to directly inject code at a specific target location.
                     * **`skipTransformation();`**: Skips all further transformations for a specific code block.
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
                *  `instructionBuilder.storeRegister(dataAddress address, register reg)`: Creates a `store` instruction for a register.
                *   `instructionBuilder.addImm(register dest, register src, uint32 imm)`: Creates an add instruction with an immediate value.
                *    `instructionBuilder.customInstruction(StringLiteral instructionName, [register reg1, register reg2])`: Creates a target specific instruction.
                 *    `instructionBuilder.getParameters()`: Returns an associative array to access the parameters. For example, the parameter names can be used as a key to access the compile time value: `uint32 offset = builder.getParameters().offset;`.
                 *   `instructionBuilder.atomicRead(address, dataType,  bitmask)`: Creates an atomic read instruction.
                *  `instructionBuilder.atomicWrite(address, value, dataType, bitmask)`: Creates an atomic write instruction.
                  *   `instructionBuilder.atomicTestAndSetBit(address, bitNumber)`: Creates an atomic test and set bit instruction.
                   *   `instructionBuilder.atomicTestAndClearBit(address, bitNumber)`: Creates an atomic test and clear bit instruction.
                *   `instructionBuilder.atomicSetBit(address, bitNumber)`: Creates an atomic set bit instruction.
                 *   `instructionBuilder.atomicClearBit(address, bitNumber)`: Creates an atomic clear bit instruction.
                 *  `instructionBuilder.atomicToggleBit(address, bitNumber)`: Creates an atomic toggle bit instruction.
                *  `instructionBuilder.memoryBarrier(order)`: Creates a memory barrier instruction.
                *  `instructionBuilder.instructionBarrier(sync)`: Creates an instruction barrier instruction.
                 *  `instructionBuilder.rawInstruction(instructionPattern)`: Emits a raw instruction.
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
*  **Compiler Caching for Incremental Builds:** The compiler uses a cache to speed up compilation.
*   **CAM-Specific Memory Regions and Attributes:** CAMs can define their own memory regions and attributes.
*   **Custom Error Messages and Hints:** CAMs provide custom error messages using the `compilerError()` function.
 *  **Custom Warnings and Information Messages:** CAMs can generate customized warning and information messages using `compilerWarning()` and `compilerInformation()` functions.
*   **Fine-Grained Hardware Dispatch:** Direct hardware access for accelerators.
*   **Direct Register Access:** Direct access to hardware registers using memory mapped access and conceptual registers.
*  **ABI Customization**: CAMs can customize the function call ABI using the `callDSL` block.
*   **Microcode Specification:** The microcode of an instruction can be specified using the `microcode` block.
*  **Performance Optimization**: Code regions can be tagged with performance attributes, to guide the compiler optimizations using a target specific implementation.
* **Code Generation Bypass**: CAMs can bypass code transformations and provide a low-level implementation when necessary using the `emitCode` and `skipTransformation` compiler API functions.

**6. Directives and Pragmas (Updated)**

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
    *   `#use "moduleName" [with config { ... }] ;`: Module inclusion that makes the exported components of the module accessible. You can list the specific items that you want to import by using the `only` keyword: `#use "MyModule" only { functions {myFunction}, constants {myConstant} };`, or to exclude some items using `exclude` keyword: `#use "MyModule" exclude { functions {myOtherFunction} };`. The `exclude` or `only` keywords can be used only when a list of items is specified.
        *   Generic modules: `#use "genericModuleName<Type>" as alias;`
        *   Example: `#use "GenericVector<uint32>" as intVector;`
        *   `with config { ... }`: Configures parameters for the module.
            *   Example: `#use "MyModule" with config { baudRate = 115200 };`
            *   When importing a module with `config` parameters, the compiler creates a new copy parameterized by those values. If the imported module uses other modules with config parameters, those are also duplicated, respecting the parent's parameters. The parent module's `config` parameters override any conflicting parameters in the imported module.
     *    Specific module versions can be imported using the `@` syntax. Example: `#use "MyModule@1.2.0";`
    *   `#use "camName" [ genericInstantiation ] [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";"`: CAM inclusion that makes the exported interface intrinsic functions of the CAM accessible, with support for `override` and customization.
        *   Example: `#use "MyUartCam";`, `#use "MyGenericCam<uint32>";` `#use "MyUartCam" with config { baudRate = 115200 };` `#use "UartCam" override function platformUartSendByte with config { enableDma = true };`
    *   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code.
        *   Example: `#cpuTarget armCortexM4`
    *   `#raw binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b00010000`), decimal (`16`), or octal literals (`0o20`). This is mostly used to embed boot code or binary firmware into a specific location. String literals can also be used. String literals are encoded using UTF-8.
        *   Example: `#raw 0x10 0b00010000 16 0o20 "abc";`
    *   `#template(sectionName) { ... }` : This directive allows for injection of a code template into a specific section. The `sectionName` is defined in a CAM.
     *  `#performance(level, hint, ...)`: A pragma that provides performance information to the compiler.
           * `#performance(critical, memoryAccess, loopUnroll=4, ...)`: example.
             * The `level` is a string literal that can be used to specify the level of performance optimization such as `critical`, `high`, `normal` or `low`, and will be CAM specific, and may have different meanings for different target architectures.
            * The `hint` is a string literal that can be used to provide specific hints for a given code section such as `memoryAccess`, or `loopUnroll`.

*   **Operation Directives:**
        *  `print(StringLiteral format, ...);` : This function prints a formatted message. The format string uses standard format specifiers such as %d, %x, %s, or using string interpolation using `f""`: `print(f"The value is {x}");` or using string interpolation with formatting specifiers using `{variable:specifier}`: `print("Value = {x:x}");`. This function is intended to provide basic debug or console output, and its implementation depends on the target platform. Parentheses can be omitted if only one parameter is passed. Example: `print "Hello world";`.
        * `memoryStore(BootDataSize , BootAddress , BootOperand);`: Stores a value into memory, at a given address and data size. This function may or may not perform a memory barrier depending on the target platform.
        * `memoryLoad(BootDataSize , BootAddress , BootOperand);`: Loads a value from memory at a given address and data size.
    *  `intrinsic operationName [argumentList] ;`: Provides a way to access hardware functionality through specific operations that are provided by CAMs. Parentheses can be omitted if only one argument is passed.
        *  `atomicSetBit(register, bitNumber)`: Atomically sets a specific bit in a register.
        *  `atomicClearBit(register, bitNumber)`: Atomically clears a specific bit in a register.
        * `atomicToggleBit(register, bitNumber)`: Atomically toggles a specific bit in a register.
        *   `atomicTestAndSetBit(register, bitNumber)`: Atomically tests a bit and sets it if clear. Returns true if the bit was clear before the operation.
        *   `atomicTestAndClearBit(register, bitNumber)`: Atomically tests a bit and clears it if set. Returns true if the bit was set before the operation.
        *   `atomicReadRegister(register, bitmask)`: Atomically reads a value from a register using a specific bitmask.
        *   `atomicWriteRegister(register, value, bitmask)`: Atomically writes a value to a register, applying a given bitmask.
      *   `memoryBarrier(order);`: Generates a hardware specific memory barrier. The `order` can be `read`, `write`, `readWrite`, or `all`. If no parameter is specified, it defaults to `all`.
      *  `instructionBarrier(sync);`: Generates a hardware specific instruction barrier. The `sync` can be `instructionFetch`, `dataFetch`, or `dataWrite`.
       * `rawInstr(instructionPattern);`: Embeds raw machine code into the generated output. The `instructionPattern` is CAM specific.
        *  `hardwareSymbolicCheck(expression, range) -> void;` : Performs a hardware-assisted check if the expression satisfies the constraints given by the range. If no hardware support is present, the code will trap.
        *   `hardware_clz(uint32 value) -> uint32;` : Counts the number of leading zeros in the specified value using a hardware-accelerated instruction if available.
        *   `hardware_ctz(uint32 value) -> uint32;` : Counts the number of trailing zeros in the specified value using a hardware-accelerated instruction if available.
        *  `hardware_startTimer();` : Starts a hardware timer.
         * `hardware_endTimer();` : Reads the current value of the hardware timer.
         * `hardware_resetTimer();`: Resets the hardware timer.
          * `hardware_setWatchpoint(address, type);`: Sets a hardware watchpoint that can be used for debugging purposes. The type of the watchpoint can be `read`, `write`, or `readWrite`.
          * `prefetch(address, size, level);`: Dispatches a hardware prefetch instruction for a given `address` with a specific `size` and `level`.
          * `hardware_trap();`: Generates a hardware trap for debugging purposes when a low level error is detected.
        * `cacheFlush(address addr, size size, level level);` Flushes data from the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
        * `cacheInvalidate(address addr, size size, level level);` Invalidates data in the cache at a specific `address`, for a specific `size` at a given `level` in the cache hierarchy.
       *  `align(address, integerLiteral);`: Aligns a given address to a specific alignment.
    *  **Atomic Operations:**
        *   `atomic { ... }`: Guarantees atomic execution of the code block using the primitives specified in the CAM. If no argument is specified, the compiler will attempt to atomically execute the instructions within the given block, which may involve software locking mechanisms if hardware primitives are not available, which might create an overhead depending on the target architecture and the compiler options. The code block is limited to low level access primitives, and calls to CAM provided `intrinsic` functions that can be mapped to target-specific atomic primitives.
        *   `atomic (variable) { ... }`: Guarantees atomic access and modification of the specified `variable` using the underlying primitives provided by the CAM. The scope is limited to operations involving that specific variable. The compiler will attempt to generate code to atomically read, modify and write to that specific memory location or variable. If the hardware doesn't have a direct atomic read-modify-write instruction the CAM may have to generate code that involves locks and memory barriers to ensure atomicity.
         *  `atomic memoryStore(...)`: Performs an atomic memory store operation.
        *  `atomic register identifier = ...;`: Performs an atomic write to a given register using an atomic instruction when it is available.

**7. Memory Safety in BML (Updated)**

*   `DataType scoped`: Zero-overhead dangling pointer prevention using lexical scope. Pointers are valid only within their lexical scope and the compiler ensures that the pointer cannot escape the scope. This mechanism is mostly implemented at compile time, but may include optional runtime checks based on compiler flags. The `scoped` keyword is optional for local variables when the compiler can statically infer their scope. Scoped pointers cannot be null.
    *   **Static Analysis Process for Implicit Scope:**
        *   The compiler performs static analysis to determine if a pointer's lifetime is strictly confined to its lexical scope specifically its function body or a block inside the function.
        *  The compiler analyses the code flow to verify if the pointer is never passed as a parameter to a function or stored inside a struct that outlives the scope of the pointer, and if the pointer is never returned by a caller function. If all the conditions are satisfied the compiler infers that the pointer is scoped and the `scoped` qualifier can be omitted.
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
    *   `!` pointers cannot be null and can only read from memory. This pointer will prevent writing into a specific memory location guaranteeing read-only access. The memory that it points to can change if the underlying memory region is mapped to a volatile or external device for example.
     *   `scoped` pointers are limited to their scope guaranteeing safety in a more localized and restrictive way.
    *   `tracked` pointers have runtime checks to ensure the pointer is not dangling. The tracked pointers have a small overhead that is associated to the metadata required to perform the runtime checks. The runtime checks can be implemented using different mechanisms, such as metadata, guard pages, or specific hardware support. When a `tracked` pointer is copied, a new `tracked` pointer is created with its own metadata and its own runtime checks. Concurrent access to the same memory location using different tracked pointers may result in data races if no additional synchronization mechanism is used.
    *   Example: `uint32! myReadPtr;`, `rawbyte- myWritePtr;`, `uint32@MySecureRegion regionPtr;`, `uint32+ myNonNullPtr = &myValue;`, `uint8 localPtr = &myStackValue;` (implicit scope), `uint16 tracked myTrackedPtr = &someValue;`
*   **Static Typing:** Enforces type rules at compile time preventing type-related errors, which makes the code safer and easier to reason about. Type checking is performed by the compiler during the type resolution phase of the compilation process.
*   `DataType safe [Size]`: Runtime bounds checking to prevent buffer overflows and out-of-bounds accesses, improving the security of the code. Every access to a `safe` array will be checked at runtime.
    *   Example: `uint16 safe[100] sensorValues; sensorValues[10] = 5;`
    *   `arraySlice<DataType>`: Provides bounds-checked views over a specific section of an existing array which can be used to improve the safety of memory access. The bounds checking will be performed at runtime and a runtime exception will be raised if the access is out of bounds of the original array or the slice.
        *   Example: `uint32 safe [100] myArray; arraySlice<uint32> mySlice = myArray.slice(0,10);`
*   `DataType^`: Unchecked pointers (use with caution). The developer must handle all potential safety issues related to raw memory access. It is recommended to use other more secure pointer types whenever possible. If the developer uses a raw pointer, they will have to check for null pointers and also perform bounds checking manually.
    *   Example: `uint32^ ptr = &memoryLocation;`
*   **Region-Based Memory Management:** Implicit lifetime management through scopes and explicit regions which simplifies memory handling and provides compile-time checks that increase the code safety. Pointers with the `@region` modifier can only access the specified memory region. The compiler may generate a compile-time error if the pointer attempts to access memory outside of the region where it is defined. `dataRegion` declarations do not directly grant access to the data to other memory regions. To transfer data across regions a CAM will have to explicitly read from one region and write to another region, using memory copy functions or by transferring using memory mapped peripherals. CAMs can also define their own memory regions using the `CAMDataRegion` specifier. This data region has the same scope as the CAM, and can be used to perform calculations, store intermediate results, or implement other internal logic within a given CAM. CAM data regions can also be used to store CAM configurations, parameters or other persistent state for that CAM.
*   **Compile-Time Null Dereference Detection:** Static analysis provides warnings for potential null pointer dereferences avoiding runtime crashes due to null pointers. The compiler will issue a warning if a pointer with a `-` modifier or a general pointer using `^` is dereferenced without checking if the pointer is null. The compiler will not perform any null check if the `+` or `!` modifier is used, or if the pointer is a `scoped` pointer as these pointers are guaranteed not to be null.
    *   Example: If a value is known to be potentially null, the compiler will generate a warning if the developer attempts to dereference it without performing a null check.
*   **Refinement Types:** Compile-time data constraints that allow specifying valid ranges or values for variables, further improving type safety by making the code more specific. Refinement types are used by the compiler to perform static analysis and can eliminate the need for runtime bounds checks. The compiler uses techniques such as symbolic execution, range analysis, and constraint solving to statically verify refinement type constraints. The compiler will perform range analysis and dataflow analysis to guarantee that refinement types are respected.
    *   Example: `type validValue = uint16 where value < 100;`
*   **Capability-Based Regions:** Memory access control enforced at compile time (mostly) that allows isolating specific sections of the memory, preventing data corruption by limiting the access capabilities of pointers providing increased safety and security. Memory regions can be defined using different capabilities such as `read`, `write`, `execute`, `capability_derive` and `peripheral` which allows the CAM to perform more fine-grained memory access controls.
    *   Example: `dataRegion @read secureRegion { uint32 myData; };`, `dataRegion DeviceData { uint8 safe [100] buffer; };`
    *   **Stack-Only Allocation:** `@stackOnly` enforces stack allocation, useful for performance and memory predictability since it avoids the overhead of dynamic memory allocation and provides predictable memory usage. When this attribute is used the memory will be allocated in the stack.
        *   Example: `@stackOnly uint8 myStackVar;`
    *   **Hardware-Assisted Bounds Checking:** Optional, when hardware is available and enabled via compiler flag or pragma, leveraging hardware to improve the performance and reduce the overhead of memory access checks. This is only used for safe arrays and tracked pointers and will generate the appropriate memory access configuration for the target platform.
        *   Example: `#pragma enable_hardware_bounds_checking; uint32 safe[10] myArray;`
*   **Runtime Checks Tradeoffs:** While BML aims for minimal overhead, runtime checks are still present for certain features (e.g., `safe` arrays, `tracked` pointers). These checks help prevent memory errors, but they may have a performance impact. These checks can be disabled by using compiler flags or pragmas. Disabling the runtime checks should be done with caution as it can introduce potential safety issues.
*  **Runtime Checks are Optional:** The runtime checks are optional and may be disabled by using compiler flags or pragmas, such as `boundsCheck=off` or specific pragmas such as `#pragma disable_bounds_check;`. This gives the user more control by explicitly disabling the runtime checks, but it also means that the user can create an unsafe program that can crash due to buffer overflows or dangling pointer accesses.
*   **Cutting-Edge Solution (High-Performance Bitfield Access):** To minimize overhead and maximize performance in bitfield manipulation, especially within memory-mapped registers, BML provides a "with" statement which allows grouping bitfield updates, avoiding multiple read-modify-write operations when there are multiple bitfield updates for the same register. This is a cutting-edge feature designed to provide low overhead and high performance. The compiler can optimize the code by using specialized hardware instructions where available, or by using bitwise operations where specialized instructions are not available. Bitfield members can be accessed with the `=` operator and initialized by using an initializer list, without the `with` statement: `myGpioRegisters.controlRegister = { enableBit = 1, dataBit = someValue };`. Bit Ranges can be accessed with the syntax `myGpioRegisters.controlRegister[0..3] = 0b101;` or by using a bit mask: `myGpioRegisters.controlRegister[0b00001111] = 0b101;`.
    *   **"with" statement optimization:** The compiler analyzes the `with` block, and skips read/write operations to memory-mapped registers when they are not modified. This optimization is performed automatically with no additional user effort.
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
                myUartRegisters.controlRegister = {enable=1, baud=115200}; //bitfield assignment using struct like syntax

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

**8. Multi-Target Architecture Versatility (Updated)**

*   `@arc architectureName { ... }`: Provides architecture-specific code implementations inside CAMs, modules, and in the main file. This is the main method to provide different implementations of the same API depending on the target architecture. The compiler will choose the appropriate code block based on the target architecture specified by the user by using the `-target` flag in the command-line interface. **All code in a BML module must be inside an `@arc` block, except for code that is considered architecture-independent.** Code outside `@arc` blocks is only allowed in the main file and within the `statements` block of a module declaration, where it is treated as implicitly architecture-independent (implicit `common` block).
*   CAMs implement hardware abstractions with platform optimizations allowing developers to write code once and reuse it on different platforms without sacrificing performance. The abstraction mechanism allows developers to create reusable code that is portable across different architectures but also perform architecture-specific optimizations using the features of a specific target platform.
*   `#cpuTarget cpuName`: Provides CPU optimization hints allowing the compiler to target specific features and improve the performance of the generated code. The compiler will use the information of the target CPU to generate specialized code.
    *   The compiler selects CAMs and generates code based on the `-target` flag which can be used to specify the target architecture and CPU.
    *   Example: `#cpuTarget armCortexM4`
*   **Multiple `@arc` Blocks:** A single BML file (including the main file) can contain multiple `@arc` blocks, one for each supported architecture. The compiler selects the appropriate block based on the `-target` flag during compilation.
*    **Explicit `@arc common` Block:** Code outside of any `@arc` block is not allowed and it's a compile-time error. Architecture-independent code should be placed inside an `@arc common` block.
    *   **Restrictions:** The compiler enforces restrictions on code in the `@arc common` block to ensure portability. These restrictions include disallowing direct hardware access, limiting the use of certain intrinsics, and potentially restricting data types to a common subset.
    *   **No Nested Functions:** Nested function definitions are *not* allowed in the `@arc common` block. This restriction is in place to avoid complexities in code generation and to maintain a clear separation between architecture-independent and architecture-specific code.
     *   **CAM-Provided Defaults:** CAMs are expected to provide default implementations for common functions (like `print`) that can be used within the `@arc common` block. These defaults are architecture-specific but are automatically selected by the compiler based on the target.

**9. Concurrency and Parallelism (Updated)**

*   `threadHandle identifier = startThread(functionName) [ threadAttributes ];`: Creates concurrent tasks allowing the developer to implement multi-threaded applications. The `startThread()` function creates a new thread. The way that the stack is handled depends on the target architecture and the compiler flags and is implementation-defined. There is no mechanism to kill a thread or to join a thread directly. If required you can use CAMs to implement such functionality. The `#priority` and `#affinity` attributes can be added to this statement.
    *   Example: `threadHandle myThread = startThread(myThreadFunction) #priority("realtime") #affinity("cpu1");`
*   `parallel { ... }`: Structured concurrency allowing the developer to create parallel tasks that execute concurrently. The compiler may map these tasks to different threads depending on the underlying platform and target architecture but the developer has no control over how the tasks are scheduled. Each task inside a parallel block should have its own separate scope. Variables declared within a task are not accessible to other tasks within the same parallel block. Tasks can access variables declared in the outer scope (e.g. the function scope).
    *   Example:
        ```baremetal
        uint32 sharedVar = 0;
        parallel {
            {
                uint32 localVar = 10; // localVar is local to this task
                 sharedVar = localVar; // access to sharedVar if needed.
            }
            {
                uint32 anotherLocal = 20; // anotherLocal is local to this task
                 sharedVar += anotherLocal; // access to sharedVar if needed.
            }
        }
                // localVar and anotherLocal are not accessible here
                 print("Shared data", sharedVar);
        ```
*   `await identifier = expression;` / `yield(expression);`: Asynchronous operations using awaitable types, which can be used to implement non-blocking operations and implement cooperative multitasking. The `await` statement will wait for the result of an asynchronous operation without blocking the current thread. The `yield` statement can be used to return the result of the asynchronous operation.
    *   Example: `await myAwaitable = myAsyncFunction(); yield(myResult);`
*   `lock(identifier) { ... }` / `unlock(identifier);`: Critical section access to protect shared memory from data races, guaranteeing atomic access to shared resources by different threads. A mutex is a synchronization primitive used to implement critical sections.
    *   Example: `lock(myMutex) { /* Critical section */ }; unlock(myMutex);`
*   `wait(identifier);` / `signal(identifier);`: Synchronization primitives used to manage access to shared resources and coordinate concurrent tasks.
    *   The `wait` operation may or may not release a lock (depending on implementation details). The `signal` operation does not release the lock. CAMs can offer more control over these primitives using customized implementations.
    *   Example: `wait(mySemaphore); /* Access resource */; signal(mySemaphore);`
*  Default memory consistency is sequential consistency, using `memoryBarrier();` for explicit ordering of memory accesses. There is no implicit memory barrier at the end of `parallel` blocks as the developer must have full control of the memory access and ordering semantics.
    *   **Memory Barriers:**
        *   BML uses sequential consistency as its default memory model which means that by default memory operations appear to execute in the order specified by the program.
        *   When interacting with shared memory volatile variables or peripheral devices the compiler may not preserve the order of the memory accesses due to optimizations or memory hierarchies.
        *   Memory barriers are used to enforce a specific order on memory access operations. When a memory barrier is executed, the CPU guarantees that all previous memory operations are completed before any subsequent operation starts.
         * The `memoryBarrier(order);` operation will generate a target-specific instruction that will act as a memory barrier. It is usually not required unless you have a specific memory ordering that the compiler cannot enforce. The order can be `read`, `write`, `readWrite`, or `all`, and defaults to `all` if the argument is missing.
        *  Memory barriers should be used when you have multiple threads that access the same memory regions or when you are interacting with memory-mapped devices that have dependencies on the order of the memory operations.
        *   Memory barriers can be particularly important when dealing with memory mapped peripherals, because the behavior is not always predictable if the access is done in a different order than it was intended.
*   `transaction { ... }`: Ensures atomic execution of the code block. The implementation of transaction blocks depends on the target. For a single-core processor the compiler will typically generate code that disables interrupts and system preemption during the execution of the transaction. For a multi-core system, the code will generate a lock or a memory barrier operation which depends on the availability of hardware support. There are limitations on the operations that can be performed within a transaction to ensure atomicity. Calling functions may break the atomicity guarantees and the compiler will try to enforce these rules at compile time, if possible. The behavior of calling functions inside transactions is implementation-specific. The user is responsible for understanding the hardware and software implications of using transaction blocks.

**10. Functions (Updated)**

*   **Function Definition:**
    ```
    function [ "#callConvention(stringLiteral)" ] [  "#instruction(stringLiteral)" ]  returnType functionName(parameterList) {
        [declarationsBlock]
        // Function body (statements)
    }
        // or
     function [ "#callConvention(stringLiteral)" ] [ "#instruction(stringLiteral)" ]   returnType functionName(parameterList) "=" expression;
    ```
* The `#callConvention` and `#instruction` attributes have been added to function declarations.
*   **Implicit `statements` Block:** The body of a function is implicitly treated as a `statements` block. You do not need to write `statements { ... }` inside a function.
*  **Simplified function definition:** If a function consists of a single expression the `return` type and curly braces can be omitted.
  ```baremetal
    function add(uint32 a, uint32 b) = a + b; // return type inferred from the expression.

   function void init() = print("initializing"); // single expression void function
```
*  If a function has a single expression with an implicit return type the `return` statement can be omitted.
*   **`declarations` Block (Optional):** You can optionally use a `declarations` block at the beginning of a function body to declare local variables. This block does not create a new scope.
*   **Lexical Scoping:** Variables declared within a function are local to that function. Nested functions have access to variables in their enclosing function's scope. Nested functions can only be defined within `@arc` blocks and are not allowed in the `@arc common` block.
*   **Return Type:** The `returnType` specifies the type of value returned by the function. Use `void` if the function does not return a value. If the function consists of a single expression, the return type can be omitted, and the compiler will infer it from the expression. If there is no explicit return statement and no implicit return type can be inferred, a compile-time error will be generated.
*   **Parameter List:** The `parameterList` is a comma-separated list of parameter declarations.
*   **Function Parameters:**
    *   Parameters can have default values: `function myFunc(uint32 param1 = 10, boolean flag = false) { ... }`. Default parameters must follow non-default parameters when defining a function.
    *  Parameters are immutable by default within the function body.
    *   **Type Inference with Attributes:** You can use the `#in`, `#out`, and `#inOut` attributes for function parameters to enable type inference.
        *   Example: `function myFunc(uint32 x #in, uint32 y #out) { y = x * 2; }`
    *   **Implicit "this" Parameter:** The `this` parameter is automatically added to struct or union method declarations:
        *   The `this` parameter is available inside the function, allowing access to the members of the struct or union.
        * The `this` parameter is implicit for struct and union methods and can be omitted when there is no name collision.
        *    When a name collision occurs with a local variable or a function parameter, the explicit `this` keyword or the `@` operator should be used to access the member of a structure or union.
        *   The `this` parameter is implicitly provided and it is not needed to explicitly specify it as a parameter of the function:
            ```baremetal
            type MyType = struct {
                  uint32 value;
              };
            function void myMethod() {  // "this" is implicit.
              print(value); // Implicitly this.value.
              @value = 10; //Explicit member access using the @ operator
              this.value = 10; // Explicit member access using the this keyword.
           }
           ```
 *   **Code Blocks**: Code blocks that match a specific `block` type can be passed as parameters to a given function. A code block can be created by using a lambda expression with one of the following forms: `(parameters) => expression` or using a `(parameters) => { ... code ... return expression }` syntax.

**11. Error Handling and Diagnostics (Updated)**

*   **11.1 Compiler Error Codes**
    *   The BML compiler shall produce clear and informative error messages with the following format: `filename:line:column: error [Error Code]: Error message`
    *   The following error codes are defined, and a conforming compiler must use them where applicable:
        *   **E0001:** Syntax error.
        *   **E0002:** Type mismatch.
        *   **E0003:** Undeclared identifier. (This error is used when a member is accessed without using `this` in a shadowed scope).
        *   **E0004:** Invalid use of `const`.
        *   **E0005:** Invalid use of `volatile`.
        *   **E0006:** Invalid type conversion.
        *   **E0007:** Out of bounds array access.
        *   **E0008:** Null pointer dereference.
        *   **E0009:** Division by zero.
        *   **E0010:** Invalid module import.
        *   **E0011:** CAM configuration error.
        *   **E0012:** Code outside `@arc` block in module.
        *   **E0013:** Declaration outside `declarations` block.
        *   **E0014:** Statement outside `statements` block.
        *   **E0015:** Invalid use of `with` statement.
        *   **E0016:** Invalid preprocessor directive.
        *   **E0017:** Invalid attribute usage.
        *   **E0018:** Function call does not match function signature.
        *   **E0019:** Invalid use of `generic` type.
        *   **E0020:** Missing `return` statement.
        *   **E0021:** Unreachable code.
        *   **E0022:** Invalid memory access. (used when memory addresses are not valid or are misaligned or when performing invalid memory accesses with atomic instructions).
        *   **E0023:** Refinement type constraint violation.
        *   **E0024:** Invalid symbolic expression.
        *   **E0025:** Thread creation error
        *   **E0026:** Asynchronous operation error.
        *   **E0027:** Memory Region Access Violation
        *   **E0028:** Invalid Hardware Access (Used when using hardware intrinsics or accessing registers incorrectly)
        *   **E0029:** Unsupported Operation
        *   **E0030:** Invalid Region Capability
        *   **E0031:** Region Conflict
        *   **E0032:** Use of Uninitialized Variable
        *   **E0033:** Violation of `readOnce` Attribute
        *   **E0034:** Violation of `writeOnce` Attribute
        *   **E0035:** Concurrency Error
        *   **E0036:** CAM Error (Used by CAMs to report internal errors)
        *   **E0037:** Internal Compiler Error (Should not occur in normal operation)
        *  **E0038:** Invalid Tuple Access
        *   **E0039:** Invalid Bitfield Access
        *   **E0040:** Invalid `with` Statement Optimization
        *   **E0041:** Invalid Operation in `symbolic` Block
        *   **E0042:** Invalid Instruction Definition
        *   **E0043:** Invalid `instrDSL` Usage
        *  **E0044:** Invalid `hardwareInstructionDSL` Usage
        *  **E0045:** Meta-CAM Error
        *   **E0046:** Scope Violation
         *   **E0047:** Unsupported Target Architecture
        *   **E0048:** Use of nested functions in `@arc common` code.
        *   **E0049:** Function declaration outside of `@arc` block in module.
        *   **E0050:** `declarations` block used outside of a function body or an `@arc` block.
       *  **E0051:** Missing `statements` block in function definition.
        *   **E0052:** Invalid function parameter type inference.
        *   **E0053:** Conflicting type inference.
        *  **E0054:** Invalid type for `@inOut` parameter.
         *   **E0055:** Invalid use of `~` (mutability modifier).
        *  **E0056:** Parameter name conflicts with local variable or structure member.
         *   **E0057:** Duplicate declaration in the same scope.
        *   **E0058:** Invalid attempt to modify an immutable variable.
        *   **E0059:** Invalid CAM override.
        *   **E0060:** Invalid CAM configuration.
        *   **E0061:** Circular module dependency.
        *   **E0062:** Invalid `main.bml` structure.
       *  **E0063:** Missing `@arc` block for a target architecture.
       *   **E0064:** Invalid `boot` code block
        *   **E0065:** Invalid combination of sub-target architecture and aliases
         *   **E0066:** Invalid type qualifier.
          *  **E0067:** Byte literal out of range (0-255).
          *  **E0068:** Invalid use of custom suffix
        *  **E0069:** Hardware Error
        *  **E0070**: Invalid attribute for memory mapped struct, or register definition
        *  **E0071**: Invalid access modifier to register declaration, if readOnly, writeOnce or threadLocal are not used on a `register` declaration.
        *  **E0072**: Attempt to use a register declared in a CAM outside the scope of the CAM.
        * **E0073**: Invalid atomic block declaration.
        * **E0074**: Invalid use of code block declaration.
         *  **E0075**: Invalid use of type conversion using the `@` operator.
         * **E0076**: Invalid type for hardware symbolic check.
         * **E0077**: Invalid use of raw instruction.
    *   **Note:** This list is not exhaustive but provides a starting point for consistent error reporting. The compiler may define additional error codes as needed. CAMs can also define their own custom error codes.
*  **11.2 Warnings**
    *   The BML compiler may also issue warnings for conditions that are not strictly errors but may indicate potential problems in the code.
    *   Warnings should use a similar format as error messages but use a `warning` prefix.
    *  The compiler should provide options to enable or disable specific warnings or classes of warnings.
     *   **Example:** `MyFile.bml:25:10: warning [W0001]: Unused variable 'x'`
        *   **W0001:** Unused variable.
        *   **W0002:** Variable shadowing.
        *   **W0003:** Implicit type conversion may result in loss of data.
        *   **W0004:** Suspicious use of uninitialized variable.
        *   **W0005:** Possible unintended integer overflow.
        *   **W0006:** Code may not be reachable.
        *   **W0007:** Potentially unsafe memory operation.
        *   **W0008:** Inefficient use of memory (e.g., poor alignment).
        *   **W0009:** Potential concurrency issue (e.g., data race).
        *   **W0010:** CAM-specific warning.
        *  **W0011:** Implicit conversion from signed to unsigned integer
        *   **W0012**: Ignoring a `Result` value.
         *  **W0013**: Potential null dereference.
         *  **W0014**: Scoped pointer may escape its scope.
         *   **W0015**: Implicit return type.
         *   **W0016**: Refinement type check may be computationally expensive.
         *  **W0017**: Unused module
          * **W0018:** Variable declared in a scoped region not being used.
          * **W0019**: Variable used outside of atomic block.
        *   **W0020:** Invalid use of registers.
         *  **W0021:** Implicit conversion may lead to data loss due to atomic operations.
          * **W0022**: Ignoring a prefetch operation that might create a race condition.

**12. Operator Precedence and Associativity (Updated)**

BML follows the following operator precedence rules (from highest to lowest precedence):

| Precedence | Operator                                   | Description                                                                                                                                  | Associativity |
| :--------- | :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| 1          | `()`                                       | Parentheses (grouping)                                                                                                                   | Left-to-right |
|            | `[]`                                       | Array subscripting                                                                                                                         | Left-to-right |
|            | `.`                                        | Structure/union member access                                                                                                                | Left-to-right |
|            | `@`                                        | Explicit member access / Type conversion operator                                                                                                                      | Right-to-left |
|            | `~` (within a `with` block or after `=`)    | Bitfield access/modification                                                                                                              | Right-to-left |
|            | `#`                                        | Attribute access                                                                                                                         | Right-to-left |
| 2          | `!`, `~`, `+` (unary), `-` (unary), `^` | Logical NOT, Bitwise NOT, Unary Plus, Unary Minus, Pointer dereference, Address-of, `countLeadingZeros`, `countTrailingZeros`, `countSetBits` | Right-to-left |
|            | `sizeof`                               | Sizeof operator                                                                                                                            | Right-to-left |
| 3          | `*`, `/`, `%`, `+^`, `-^`                                      | Multiplication, Division, Modulo, Rotate left, Rotate right                                                                                                                    | Left-to-right |
| 4          | `+`, `-`                                   | Addition, Subtraction                                                                                                                    | Left-to-right |
| 5          | `<<`, `>>`                                 | Bitwise left shift, Bitwise right shift                                                                                                      | Left-to-right |
| 6          | `<`, `<=`, `>`, `>=`                      | Relational operators                                                                                                                         | Left-to-right |
| 7          | `==`, `!=`                                 | Equality operators                                                                                                                           | Left-to-right |
| 8          | `&`|                                        | Bitwise AND                                                                                                                              | Left-to-right |
| 9          | `^`                                        | Bitwise XOR                                                                                                                              | Left-to-right |
| 10         | `|`                                        | Bitwise OR                                                                                                                               | Left-to-right |
| 11         | `&&`                                       | Logical AND                                                                                                                              | Left-to-right |
| 12         | `||`                                       | Logical OR                                                                                                                               | Left-to-right |
| 13         | `? :`                                      | Conditional operator                                                                                                                     | Right-to-left |
|            | `?`                                       | Error propagation operator                                                                                                               | Right-to-left |
| 14         | `=`, `+=`, `-=`, `*=`, `/=`, `%=`,         | Assignment operators                                                                                                                         | Right-to-left |
|            | `&=`, `|=`, `^=`, `<<=`, `>>=`              |                                                                                                                                          |               |
|            | `+^=`, `-^=`                             |                                                                                                                                          |               |
| 15         | `,`                                      | Comma operator                                                                                                                   | Left-to-right|

*   **Notes:**
    *   Operators with the same precedence are evaluated according to their associativity.
    *   Parentheses `()` can be used to override the default precedence and associativity.

**13. EBNF Grammar (Updated)**
```ebnf
sourceFile =
  { preprocessorDirectiveStatement | regionDeclarationBlock | moduleUseDirective | moduleDefinition | targetArchitectureBlock | statement | declaration } , eof ;

moduleDefinition =
    [ genericModuleDeclarationHeader ] , "module" , moduleName , "{" ,
    moduleDeclarationBlock ,
    moduleStatementBlock ,
    exportBlock ,
     [ metaBlock ] ,
     { targetArchitectureBlock } ,
    "}" ;

metaBlock =
    "Meta" ;

genericModuleDeclarationHeader =
     "generic" , "<" , identifier { "," , identifier } , ">"  ;

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
      [ exportedPrivateFunctions ] ,
    "}" ;
exportedPrivateFunctions =
    "private" , "functions" , "{" ,  exportedFunctionList  , "}" ;

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
     "@arc" , (architectureName | "common" ) , [ architectureAlias ] , "{" ,
    [declarationBlock] ,
    statementBlock,
    { functionDefinition | declaration | statement }
    [ bootFunctionDefinition ],
    [ instrDSLBlock ] ,
      [ hardwareInstructionDSLBlock ] ,
      [ callDSLBlock ] ,
     [ camFunctionDefinition ] ,
    "}"
   |  "@arc" , ( architectureName | "common" ) , [ architectureAlias ] , "." , identifier , "{" ,
    [declarationBlock] ,
    statementBlock,
    { functionDefinition | declaration | statement }
    [ bootFunctionDefinition ],
     [ instrDSLBlock ] ,
      [ hardwareInstructionDSLBlock ] ,
       [ callDSLBlock ],
     [ camFunctionDefinition ] ,
    "}" ;

architectureAlias =
     "alias" , "=" , architectureName
  ;

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
     |  "register" , identifier, typeAttributeList, ":" , dataType , typeAttributeList, ";"
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
    "interface" ,  [ genericFunctionDeclarationHeader ] , identifier , "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , [ "ensures",  expression ] , ";" ;

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
      "function" , [ "#callConvention" , "(" , stringLiteral , ")" ] , [ "#instruction", "(" , stringLiteral , ")" ] , [ "inline" | "pure" | "noinline" | bootFunctionAttribute  | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute  | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier ,  "(" , [ parameterList ] , ")" , [ "->" , dataType | awaitableType ] , [ "ensures",  expression ] , block
   | "function",  [ "#callConvention" , "(" , stringLiteral , ")" ]  , [ "#instruction", "(" , stringLiteral , ")" ] , [ "inline" | "pure" | "noinline" | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute  | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier, dataType, identifier, functionBody;
  | "function", [ "#callConvention" , "(" , stringLiteral , ")" ]  , [ "#instruction", "(" , stringLiteral , ")" ] ,  [ "inline" | "pure" | "noinline" | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier, [ "->" , dataType | awaitableType ] , "=" ,  expression ";"
    | "function", [ "#callConvention" , "(" , stringLiteral , ")" ]  ,  [ "#instruction", "(" , stringLiteral , ")" ] ,  [ "inline" | "pure" | "noinline" | genericFunctionDeclarationHeader | stackOnlyFunctionAttribute | compileTimeFunctionAttribute | "inlineAlways" | "inlineIfSmall" ] , [ "export" ] , identifier, [ "->" , dataType | awaitableType ] ,  block;
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
     parameter { ","? , parameter } ;

parameter =
     dataType? identifier [ "=" constantExpression ] , [ typeQualifierList ]
   ;

typeDefinition =
    "type" , [ "export" ] , identifier , "=" , dataTypeExpression , ";" ;

dataTypeExpression =
    dataType ;

structDefinition =
    "struct" , [ typeAttributeList ] , identifier , [ "@mapped" , [ "(" , addressExpression , ")" ] ] ,  [ "#initializerList"] , "{" ,
    { structMemberDeclaration }
    "}" , ";"
   | "struct" , [ typeAttributeList ] , identifier ,  dataType, identifier, ";"
;

structMemberDeclaration =
    attributedStructMemberDeclaration
  | structMemberDeclarationCore ;

attributedStructMemberDeclaration =
    typeAttributeList , structMemberDeclarationCore ;

structMemberDeclarationCore =
    [ "nonvolatile" ] , [ "volatile" ] , dataType , identifier , [ typeAttribute ] ,";" ;

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
    { statement |bootCodeStatement}
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
    "intel" | "arm" | "riscv" | identifier ;

cpuName =
    "intelPentium4" | "intelCorei7" | "amdAthlon" | "amdRyzen" | "armCortexM0" | "armCortexM3" | "armCortexM4" | "armCortexM7" | "armCortexM33" | "armCortexM55" | "armCortexA7" | "armCortexA9" | "armCortexA15" | "armCortexA53" | "armCortexA57" | "armCortexA72" | "armCortexA76" | "armCortexA78" | "riscv32i" | "riscv32e" | "riscv64i" | "riscv64g" | identifier ;

moduleName =
    identifier | stringLiteral ;

dataType =
    [ regionSpecifier ] , [  typeQualifierList ], baseDataType,  [ arrayDimension ], [ pointerSymbol ], [ genericTypeInstantiation ] , [dependentTypeInstantiation] , [ scopedOrTrackedQualifier ] ;
scopedOrTrackedQualifier =
    ( dataType , ( "scoped" | "tracked")) | (  "scoped" | "tracked" ) ;

dependentTypeInstantiation =
    "<" , constantExpression , ">";

typeQualifierList =
    typeQualifier { "," , typeQualifier } ;

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
    | bitmaskType
   | codeBlockType
  ;
bitmaskType =
   "bitmask";

instructionType = "instruction" , "<" , identifier, { "," , expression } , ">";
codeBlockType =
    "block", "(" , [ parameterList ], ")" , "->" , [ dataType | awaitableType ] ;
regionBasedType =
    identifier;

resultType =
    "Result" , "<" , dataType, "," , dataType, ">" ;

capabilityPointerType =
    dataType , capabilityQualifier;

capabilityQualifier =
    "+"
  | "-"
  | "@" , identifier
  | "!"
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
    "read" | "write" | "execute" | "capability_derive" | "peripheral" { "," , ( "read" | "write" | "execute" | "capability_derive" | "peripheral") } ;

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
     "dataRegion" , [ regionCapabilitySpecifier ] , [ regionAttributeList ] ,  "{" , { dataRegionDeclaration } , "}"
   | "dataRegion" , [ regionCapabilitySpecifier ] ,  [ regionAttributeList ], identifier, dataType, identifier, ";"
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
    "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" | "CodeBlock" ;

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
     [ boundsCheckAttribute ] [ layoutAttribute ] [ optimizationAttribute ] [ otherAttribute ]  ;

boundsCheckAttribute =
     "#boundsCheck"
  | "#noBoundsCheck"
  | "#boundsCheck" , ":" , boundsCheckOption
  | "boundsCheckRefinement" , "(" , boundsCheckOption , ")"
  | "isType" , "(" , typeKindIdentifier , ")"
 | "sizeof"  , [ typeKindIdentifier ]
  |  "#potentialNull"
  | "#guaranteedNonNull"
  | "#symbolicCheck", "(" , expression, ",", rangeExpression, ")"
  ;

layoutAttribute =
     "#packed"
  | "#aligned" , "(" , integerLiteral , ")"
  | "#row_major"
  | "#array_of_structures"
   | "linearAccess"
  | "circularBuffer"
  | "cacheLineAligned"
  | "vector" , "(" , integerLiteral , ")"
  | "vectorAligned"
   | "#endian" , "(" ,  endianness  , ")"
   |  "#byteOrder" , "(" ,  endianness  , ")"
   ;
endianness =
    "bigEndian" | "littleEndian" ;

optimizationAttribute =
  "#register" , "(" , stringLiteral , "," , stringLiteral , ")"
  | "schedule" , "(" , stringLiteral , ")"
  | "linearAccessGuaranteed" , "(" , [integerLiteral ] ,")"
  | "noOverlapGuaranteed"
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
   |  "#prefetch" , [ "(" , constantExpression, ")" ]
  | "#instruction", "(" , stringLiteral , ")"
   | "#callConvention", "(" , stringLiteral , ")"
   |  "#cpuTarget" , "(" , cpuName , ")"
  ;

otherAttribute =
    identifier
 | "#attribute" , "(" , attributeName , [ "," , attributeArgumentList ] , ")"
  | "bitfield" , "(" , identifier , "." , identifier , ")"
   | "readOnce"
  | "writeOnce"
  | "stackOnly"
   |  "#address" , "(" , expression , ")"
  |  "#section" , "(" , stringLiteral , ")"
  | "#noncached"
  | "#cached"
  | "#stronglyOrdered"
  | "#volatile"
   | "#dmaSource", [ "(" , dmaConfigParameters, ")" ]
  | "#dmaDestination", [ "(" , dmaConfigParameters, ")" ]
  | "bitOffset", "(" , integerLiteral , ")"
     | "readOnly"
  | "threadLocal"
   | "writeOnce";
dmaConfigParameters =
  dmaConfigParameter { "," , dmaConfigParameter } ;
dmaConfigParameter =
    identifier, "=" , expression ;

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
    "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" | "CodeBlock" ;

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
    "register" , registerName , "=" , bootOperand , ";"
  | "memoryStore" ,  bootDataSize ,  bootAddress , "," , bootOperand , ";"
   | "memoryLoad" , bootDataSize ,  bootAddress , "," , bootOperand , ";"
    | "bootRegion" , regionName , identifier , ";"
  |  identifier , ":" , ";"
    | "jump" , addressExpression , ";"
   | "jumpIf" , conditionCode , "," , identifier , ";"
 | "goto" , ( identifier | "tailcall" ) , [ identifier , "(" , [ argumentList ] , ")" ] , ";"
    | comment
    | emptyStatement
    | rawBinaryDirectiveStatement
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
    "#use" ,  moduleName , [ genericModuleInstantiation ] ,  [ "only" , "{" , moduleItemList , "}"  | "exclude" , "{" , moduleItemList, "}" ], [ "with" , "config" , "{" , { configParameterAssignment } , "}" ] , [ "as", identifier] , [ "@" , identifier] , ";" ;
moduleItemList =
  ( "functions" , "{" ,  exportedFunctionList  , "}" )
  | ( "types" , "{" ,  exportTypeList  , "}" )
  | ( "constants" , "{" ,  exportConstantList  , "}" )
    | ( "conceptualRegisters" , "{" ,  exportConceptualRegisterList , "}" )
  | ( "private" , "functions" , "{" ,  exportedFunctionList , "}" )

useCamDirectiveStatement =
      "#use" ,  moduleName , [ genericModuleInstantiation ] ,  [ "merge" , moduleName  { "," , moduleName } ] ,  [ "override" ,  (  "function" , identifier , "with" , "config" , "{" , { configParameterAssignment } , "}" ) | "function" , identifier ] ,  ";" ;

genericModuleInstantiation =
    "<" , concreteTypeList , ">" ;

cpuTargetDirectiveStatement =
    "cpuTarget" , cpuName , lineTerminator ;

rawBinaryDirectiveStatement =
   "#raw", binaryEncodingList , ";" ;

binaryEncodingList =
  byteLiteral { byteLiteral } ;

byteLiteral =
   [ prefix ], digit { digit } ;

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
    | withStatement
  | compileTimeIfStatement
   | transactionStatement
   | hardwareForLoopStatement
   |  "memoryStore" ,  bootDataSize ,   bootAddress , "," , bootOperand , ";"
  | "memoryLoad" ,  bootDataSize ,  bootAddress , "," , bootOperand , ";"
    | "compilerError" , "(" , stringLiteral, [ "," , identifier] , ")" , ";"
   | "compilerWarning" , "(" , stringLiteral, [ "," , identifier ] , ")" , ";"
    | bootCodeStatement
    | symbolicBlock
     | templateStatement
      | hardwareTrapStatement
     | hardwarePrefetchStatement
       | hardwareMemoryBarrierStatement
     | hardwareInstructionBarrierStatement
      | hardwareWatchpointStatement
        | rawInstructionStatement
        | atomicRegisterOperationStatement
       | atomicBlockStatement
        | lambdaExpressionStatement
   |  intrinsicStatement

  ;

hardwareTrapStatement =
   "hardware_trap" , ";" ;

templateStatement =
    "#template" , "(" , stringLiteral , ")" , "{" , { sourceCharacter } , "}" ;

symbolicBlock =
      "symbolic" , "{" , { statement }, "}" ;

bootCodeStatement =
    bootRegisterDirectiveStatement
  | bootMemoryDirectiveStatement
  | bootControlFlowStatement
   | rawBinaryDirectiveStatement
  | "bootRegion" , regionName , identifier , ";"
 ;

functionBody =
    block;

block =
    "{" , [declarationsBlock] , { statement | declaration } , "}" ;

declarationsBlock =
    "declarations" , "{" , { declaration } , "}" ;

hardwareForLoopStatement =
    "for" , "hardware" , "(" , variableDeclaration | expression, ";" , expression , ";" , expression , ")" , statement ;

transactionStatement =
    "transaction" ,block ;

stackOnlyVariableDeclarationStatement =
    "stackOnly" , variableDeclarationCore ;

destructuringDeclaration =
    [ typeAttributeList ], [ mutabilityModifier ] , dataType? , destructuringVariableList , [ "from" ] , identifier , ";"
   | [ typeAttributeList],  [ mutabilityModifier ] , identifier, "=" , tupleLiteral , ";" ;

destructuringVariableList =
    identifier { "," , identifier } ;

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
    "case" , constantExpression , ":" , statement , "break" , ";"
     | "case" , identifier , [ ":" , (dataType | rangeExpression) [ "where" , expression ] ] , ":" , statement, "break", ";"
  | "case" , "null" , ":" , statement, "break", ";" ;
defaultCaseStatement =
    "default" , ":" , statement , "break" , ";" ;

iterationStatement =
    whileLoopStatement
  | doWhileLoopStatement
  | forLoopStatement
    | rangeBasedForLoopStatement
    | simplifiedForLoopStatement
  ;
simplifiedForLoopStatement =
   "for" , "(" , rangeIdentifier, ":" ,  constantExpression , ")" , statement ;

rangeBasedForLoopStatement =
   "for" , "(" , rangeIdentifier , ":" , rangeExpression , ")" , statement ;

rangeExpression =
    expression , ".." , expression ;
rangeIdentifier = identifier;

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
   "return" , [ expression | tupleLiteral ] ,  ";" ;

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
    "+" | "-" | "!" | "~" | "^" | "&" | "countLeadingZeros" | "countTrailingZeros" | "countSetBits" ;

sizeofExpression =
    "sizeof" ,  ( dataType | expression )  ;

postfixExpression =
    primaryExpression { postfixOperator } ;

postfixOperator =
    arrayAccessSuffix
  | memberAccessSuffix
  | functionCallSuffix
  ;

functionCallSuffix =
    "(" , [ argumentList ] , ")"
  ;

arrayAccessSuffix =
    "[" , expression , "]" ;

memberAccessSuffix =
    "." , identifier ;

primaryExpression =
    identifier
  | constantLiteral
  | stringLiteral
  | "(" , expression , ")"
  | functionCall
  | arrayAccess
  | memberAccess
  | conceptualRegisterAccess
  | resultConstructor
   | bitfieldAccess
  | adtConstructor
  | tupleLiteral
   | instructionInstantiation
   | codeBlockExpression
   | typeConversion;
typeConversion=
  identifier , "(" , expression , ")"
    | postfixExpression , "as" , dataType
    | postfixExpression , "into" , dataType
    | postfixExpression , "from" , dataType
    | postfixExpression , typeConversionOperator
  | postfixExpression , "." , "to", "<", concreteTypeList, ">", "(" , ")" ;

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

memberAccess =
    primaryExpression , "." , identifier ;

conceptualRegisterAccess =
    "%" , identifier ;

bitfieldAccess =
    primaryExpression , "=" ,  "{" , { identifier , bitfieldAssignmentOperator, expression, [","] } , "}"
   |  primaryExpression , "=" , expression
   | primaryExpression , "[" , bitSelector , "]" ,  ( ":=" | "=" ) ,  expression ;
bitSelector =
    expression
    | rangeExpression
  |   bitSelector { "," , bitSelector }
;

bitfieldValue =
   "{" , { bitfieldAssignment }, "}"
  | expression ;

bitfieldAssignment =
    identifier , bitfieldAssignmentOperator,  expression , [","] ;
bitfieldAssignmentOperator = "=" | ":=";

conversionExpression =
     postfixExpression , "as" , dataType
    | postfixExpression , "into" , dataType
    | postfixExpression , "from" , dataType
      | postfixExpression , typeConversionOperator
    | postfixExpression , "to" , "<", concreteTypeList, ">", "(" , ")" ;
castOperator =
    "(" , dataType , ")" ;

constantExpression =
    constantLiteral
  | constantIdentifier
  | constantUnaryExpression
  | constantBinaryExpression
  | constantParenthesizedExpression
  | compileTimeFunctionCall
  | compileTimeBitfieldOperation;
compileTimeBitfieldOperation =
   "bitExtract", "(" , constantExpression , ",", constantExpression, "," , constantExpression , ")"
  | "bitInsert" , "(" , constantExpression , ",", constantExpression , "," , constantExpression , "," , constantExpression, ")"
    ;

compileTimeFunctionCall =
    identifier , "(" , [ argumentList ] , ")" ;

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
    | "+"
  | "-"
   | "@" , identifier
  | "!"
   | "noncached"
  | "cached"
    | "stronglyOrdered"
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
    "threadHandle" , identifier , "=" , "startThread" , "(" , identifier , ")" , [ typeAttributeList ] , ";" ;

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
    "states" , ":" , "{" , { dslStateDefinition} , "}"
  | dslCustomSection ;

dslCustomSection =
    identifier , "{" , { dslStatement } , "}" ;

dslStateDefinition =
    identifier , "{" , { dslStatement }, "}" ;

dslStatement =
    dslIdentifier , "(" , [ dslArgumentList ], ")" , [ "->" , dslTypeIdentifier ], ";"
  | emptyStatement;

dslArgumentList =
    dslExpression { "," , dslExpression };
dslExpression =
    dslConstantExpression | identifier;

dslConstantExpression =
    constantLiteral | constantIdentifier | constantUnaryExpression | constantBinaryExpression | constantParenthesizedExpression ;

dslIdentifier = identifier;
dslTypeIdentifier = identifier;

adtDefinition =
    "adt" , identifier , "{" , { adtMember } , "}" , ";" ;

adtMember =
    "enum" , identifier , "(" , [dataType] , ")" ,";"
  ;
instrDSLBlock =
  "@instrDSL" , "{" , { instrDefinition } , "}" ;

instrDefinition =
  "instr" , [ "export" ] ,  dataType , identifier, [ genericInstructionParameters ] , "(" , [ parameterList ] , ")" , [ "#target", "(" , stringLiteral , ")" ] , "{" , [instructionEncoding], [sideEffectBlock], [microcodeBlock] ,  [instructionLatency] , "}";

genericInstructionParameters=
   "<" , concreteTypeList , ">"
  ;
instructionEncoding =
  "encoding" , "=" , stringLiteral , ";" ;

sideEffectBlock =
   "sideEffect" , "{" , { sideEffectStatement } , "}" ;

sideEffectStatement =
    "setFlag" , "(" , identifier , ")" , "=" , expression , ";"
  |  "setRegister" , "(" , identifier , ")" , "=" , expression , ";"
    | "memoryStore" , "(" , dataType , "," , expression, "," , expression, ")" , ";"
        | "memoryLoad" , "(" , dataType , "," , expression, "," , expression, ")" , ";"
  ;

microcodeBlock =
   "microcode" , "{" , { microcodeStatement } , "}" ;

microcodeStatement =
  stringLiteral , ";"

instructionLatency =
 "latency" , "=" , integerLiteral , ";" ;

addressingModeEnum =
    "enum" , "AddressingMode", "{" , { addressingModeList } , "}" ;

addressingModeList =
  addressingMode { "," , addressingMode } ;

addressingMode =
    "RegisterDirect" | "Immediate" | "MemoryIndirect" | identifier ;

instructionTemplateDefinition =
    "template" , identifier,  "<" , "type" , identifier , ">" , "(" , [ parameterList ] , ")" , "{" , { microcodeStatement }, "}";

hardwareInstructionDSLBlock =
  "@hardwareInstructionDSL" , "{" , { hardwareInstructionDefinition } , "}" ;

hardwareInstructionDefinition =
 "instruction" , identifier, "(" , [ parameterList ] , ")" , "{" ,
  instructionBody , "}" ;

instructionBody =
  "encoding" , ":" , stringLiteral , ";" ,
   [ "sideEffect" , "{" , { sideEffectStatement } , "}" ] ,
    { operandEncoding } ,
    [ "microcode", "{" , { microcodeStatement } , "}" ] ,
      [ "latency" , "=" , integerLiteral , ";" ]
   ;
operandEncoding =
    identifier , ":" , typeName , "+" , stringLiteral , ";"
  ;
callDSLBlock =
   "@callDSL" , "{" , { callConventionDefinition} , "}" ;

callConventionDefinition =
 "callConvention", identifier, "{" , callConventionBody,  "}" ;

callConventionBody =
    parameterPassingBlock
    , [ returnLocationSpecifier ],
    [ prologueBlock ],
    [ epilogueBlock ],
    [ stackAllocationBlock ],
    [ abiNameSpecifier ] ;

parameterPassingBlock =
 "parameterPassing", "{" , { parameterSpecifier} , "}" ;
parameterSpecifier =
    "firstParameter", registerParameter, ","
    | "otherParameters" , "stack" , ","
     | "otherParameters" , "register", identifier, ","
  ;
registerParameter = typeAttributeList;
returnLocationSpecifier =
  "returnLocation" , identifier , "," ;
prologueBlock =
    "prologue" , "{" , { microcodeStatement }, "}" ;
epilogueBlock =
    "epilogue" , "{" , { microcodeStatement }, "}" ;
stackAllocationBlock =
    "stackAllocation" , "{" , stackAllocationOperations,  "}" ;

stackAllocationOperations=
 "allocateStack" , "(" , identifier ,")" , ";"
 | "freeStack" , "(" , identifier ,")" , ";"

abiNameSpecifier =
  "abiName", stringLiteral ,  "," ;

 atomicBlockStatement =
      "atomic" , [ "(" , expression , ")" ] , "{" , { statement }, "}" ;

atomicRegisterOperationStatement =
     "atomicSetBit" , "(" , expression, "," , expression, ")" , ";"
    |"atomicClearBit" , "(" , expression, "," , expression , ")" , ";"
    |"atomicToggleBit" , "(" , expression, "," , expression , ")" , ";"
    |"atomicTestAndSetBit" , "(" , expression, "," , expression , ")" , ";"
     |"atomicTestAndClearBit" , "(" , expression, "," , expression , ")" , ";"
    |"atomicReadRegister" , "(" , expression, "," , expression, ")" , ";"
     |"atomicWriteRegister" , "(" , expression, "," , expression, "," , expression, ")" , ";"
        ;
hardwareWatchpointStatement =
        "hardware_setWatchpoint" , "(" , expression, "," , typeIdentifier ,  ")" , ";" ;

rawInstructionStatement =
        "rawInstr", "(" , stringLiteral , ")" , ";" ;

hardwarePrefetchStatement =
    "prefetch", "(" , expression , "," , expression, "," , expression, ")" , ";" ;

hardwareMemoryBarrierStatement =
       "memoryBarrier" ,  [ "(" , identifier , ")" ] ,  ";" ;

hardwareInstructionBarrierStatement =
    "instructionBarrier" , "(" , identifier , ")" , ";" ;

typeIdentifier = identifier;
lambdaExpressionStatement =
     [ typeAttributeList] , [ mutabilityModifier ] , dataType , identifier , "=" , codeBlockExpression , ";"
    |  [ typeAttributeList] , [ mutabilityModifier ] , identifier , "=" , codeBlockExpression , ";" ;

codeBlockExpression =
    "(" , [ parameterList ], ")" , "=>" , ( expression  | "{" , { statement },  [ "return" , expression ] , "}" ) ;

prefix =
    "0x" | "0b" | "0o";

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
```
