
### **Bare Metal Language Specification (Version 1.5) - Complete and Concise Reference (Detailed with Examples)**

**1. Introduction**

The Bare Metal Language (BML) is a high-level language for bare metal and embedded systems development, intended as a replacement for assembly language. BML provides direct hardware access, memory safety features, and concurrent programming constructs with minimal runtime overhead. Version 1.5 is a production-ready and feature-complete language. BML promotes code portability through Composable Architecture Modules (CAMs) that enable architecture-specific code and optimizations.

**Core Principles:**

*   Zero Overhead: Minimize runtime costs for peak performance.
*   Low-Level Control: Provide direct hardware access and precise memory manipulation.
*   Memory Safety Mitigation: Incorporate language features to reduce common memory errors.
*   Multi-Target Versatility: Enable compilation for different architectures.
*   Modularity through CAMs: Build systems from reusable, optimized modules.
*   Readability and Maintainability: Promote clear and concise code.
*   Tooling and Debugging: Support source-level debugging.
*   Concurrency and Parallelism: Support multi-core processors.

**Version 1.5 Enhancements:**

Version 1.5 includes:

*   Compile-Time Reflection (`sizeof()`, `isType()`) for CAMs.
*   Compile-Time Functions (static lambdas) using `compileTimeFunction`.
*   Data-Driven Code Generation using Data Layout Attributes.
*  Memory Mapped structs using `struct @mapped` (implicit volatility, `nonvolatile`).
*   Bitfield access with `:=` and the `with` statement.
*   Flexible whitespace handling.

**2. Core Language Features**

*   **Static Typing:** BML is statically typed for compile-time error detection.
*   **Data Types:**
    *   **Integer Types:** `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`.
        * Example: `uint32 counter = 10;`, `int16 sensorValue = -20;`
    *   **Floating-Point Types:** `float32`, `float64`.
        * Example: `float32 temperature = 25.5f;`, `float64 pi = 3.14159265d;`
    *   **Address Types:** `address`, `codeAddress`, `dataAddress`, `genericAddress`, `bootAddress`.
         *  Example: `dataAddress bufferPtr;`, `codeAddress handlerAddress;`
    *   **Boolean:** `boolean` (`true`, `false`).
        *  Example: `boolean isReady = true;`, `boolean hasError = false;`
    *   **Character Types:** `char8`, `char16`, `char32`.
         *  Example: `char8 myChar = 'A';`, `char16 unicodeChar = L'Ω';`, `char32 emoji = U'😀';`
    *  **Raw Byte Type:** `raw byte` (direct memory access). Use with caution.
        *  Example: `raw byte registerValue = 0xAF;`, `raw byte^ buffer;`
    *   **Safe Array Type:** `safeArray<DataType, Size>` (runtime bounds checks).
        *  Example: `safeArray<uint16, 100> sensorReadings;`, `safeArray<float32, 25> coefficients;`
            *   **Array Slice Type:** `arraySlice<DataType>` (dynamic views).
                * Example: `arraySlice<uint32> mySlice = mySafeArray.slice(0, 10);`
                * **Range-Based For Loops:** `for (identifier in rangeExpression) {}` example: `for (uint32 index in 0..10) { /* ... */ }`
    *   **General Pointer:** `DataType^` (unchecked pointer, use with caution).
         *  Example: `uint32^ rawPointer;`, `char8^ buffer;`
        *   **Capability-Based Pointer Types:** `DataType ^readonly`, `DataType ^writeonly`, `DataType ^region(regionName)`, `DataType ^nonnull`.
             * Example: `uint32^ readonly sensorDataPtr;`, `raw byte^ writeonly transmitBuffer;` , `uint32^ region(MyRegion) regionPtr;`, `uint32^ nonnull configPtr;`
     *    **Scope-Bound Pointer:** `DataType ^scoped` (lexical lifetime management).
         *  Example: `uint8^ scoped localPtr = &someArray;`
    *   **Runtime-Tracked Pointer:** `DataType ^tracked` (runtime checks).
         *  Example: `uint16^ tracked sensorDataPtr;`
    *    **Tuple Types:** `tuple<DataType1, DataType2, ...>`.
           *  Example: `tuple<uint32, boolean> sensorResult; tuple(value, status) = sensorResult;`
     *    **Function Pointer Type:** `function(...) -> ...`.
          * Example: `function(uint32, uint32) -> uint32 MathFunctionType;`
    *   **Struct and Union Types:** `struct` and `union` for composite data structures.
          *  Example: `struct Point { int32 x; int32 y; };`, `union Packet { uint32 rawData; struct { uint8 highByte; uint8 lowByte; } bytes; };`
        *    **Memory Mapped Structs:** `struct @mapped @address(address)` (volatile members by default, unless specified `nonvolatile`). Example: `type GpioRegisters = struct @mapped @address(0x40000000) { uint32 dataRegister; nonvolatile uint32 configRegister; };`
    *   **Enum Types:** `enum` (named constants).
          *   Example: `enum ErrorCode { Ok, NotFound, AccessDenied };`
    *   **Thread Handle Type:** `threadHandle`.
        * Example: `threadHandle myThread;`
    *   **Mutex Type:** `mutex`.
          *  Example: `mutex myMutex;`
    *   **Semaphore Type:** `semaphore`.
       * Example: `semaphore mySemaphore(10);`
    *    **Awaitable Type:** `awaitable<DataType>`.
         * Example: `awaitable<uint32> asyncResult;`
    *  **Generic Modules and Functions:** `generic <DataType>`.
          *  Example: `generic <DataType> module MyModule { ... };`, `generic <DataType> function DataType myFunc (DataType value) -> DataType { ... };`
    *   **Result Type:** `Result<OkType, ErrType>` for explicit error handling.
           * Example: `Result<uint32, ErrorCode> myResult;`
       *   **Refinement Types:** `type identifier = baseType where constraint` (type constraints).
        * Example: `type ValidSensorReading = uint16 where value in range(0..1000);`
         *   **Capability-Based Regions:** `dataRegion regionName capabilities(read, write) { ... };`
        * Example: `dataRegion secureData capabilities(read) { uint32 secretKey; };`
        *   **Compile-Time Stack-Only Allocation:** `@stackOnly` for stack allocation.
        * Example: `@stackOnly uint32 myStackValue;` , `stackOnly safeArray<uint8,10> localBuffer;`
      *  **Hardware-Assisted Bounds Checking:** Optional using a compiler flag or pragma if the hardware supports it.
        *  Example: `#pragma enable_hardware_bounds_checking; safeArray<uint32, 100> myArray;`
        *   **Data-Layout Attributes:** `@layout(packed)`, `@layout(aligned(4))`, `@layout(rowMajor)`,  `@layout(array_of_structures)`.
         *  Example: `struct MyStruct @layout(packed) { uint8 a; uint32 b; };`, `safeArray<uint32, 100> myArray @layout(rowMajor);`
       *    **Compile-Time Reflection:** `sizeof()` and `isType()` intrinsics.
            * Example: `sizeof(uint32)`, `isType(DataType, "Integer")`
        *  **Compile-Time Functions:** Code execution during compilation via `compileTimeFunction`.
             * Example: `compileTimeFunction void myFunc(){ /* ... */}`
        *  **Data-Driven Code Generation:** CAMs use `hasDataLayoutAttribute()`.
            *  Example: `hasDataLayoutAttribute(DataType, "packed")`

*   **Scope:** Lexical scoping; mutability is block-scoped.

**3. Modules and CAMs: Structure and Organization**

*   **Modules:** Self-contained code units.
    *   `module moduleName { ... }`: Declares a module, creating a namespace.
        *   `description "String";`: (Optional) Provides a module description.
            *  Example: `description "A simple example module";`
        *  `declarations { ... }`: Module-private and exported declarations using `export` inline.
            * Example: `const export uint32 moduleConstant = 123;`
        *   `statements { ... }`: Module initialization code (optional).
          *  Example: `statements { uartSendString("Module initialized"); }`
        *   `export { ... }`: Defines exported items by declaring them `export` inline with declaration: `functions`, `types`, `constants`, `conceptualRegisters`, `interface intrinsic functions`.
*   **Module Usage:**
    *   `#use module "moduleName";`: Includes exported items from the module in the current scope.
        *  `with config { ... }`: Configures parameters for the module.
             *  Example: `#use module "MyModule" with config { baudRate = 115200; };`
*   **Composable Architecture Modules (CAMs):** Compiler extensions for architecture-specific code and optimization.

**4. Composable Architecture Modules (CAMs)**

*   CAMs are compiler extensions that act as "optimization engines".
*   **Structure:**
    *   `module camName { ... }`: Declares a CAM.
       *   `description "CAM Description String";`: (Optional) Description.
         *   Example: `description "Provides a UART Driver";`
        *   `config { ... }`: Configurable parameters.
            *  Example: `config { config baudRate: uint32 = 115200; }`
        *   `export { ... interface intrinsic functions { ... } }`: Declares architecture-agnostic interface functions via  `interface intrinsic export function`.
             *  Example: `interface intrinsic export function void platformUartInit(uint32 baudRate) -> void;`
        *   `@arc architectureName { ... }`: Architecture-specific implementations using `intrinsic function`.
            *   Example: `@arc arm { intrinsic function void platformUartInit(uint32 baudRate) { /* ARM implementation */ } }`
        *   `.manifest` files: Contains metadata: `name`, `version` (e.g., `"1.2.3"`), `description`, `license` (e.g., `"MIT"`), `bml_version_compatibility` (e.g. `">=1.5"`), `architectures` (e.g., `"arm, riscv"`), `keywords` (e.g. `"uart, serial"`), `exports_functions` (comma separated), `interface_intrinsic_functions` (comma separated), `source_file` (e.g. `"MyCam.bml"`), `documentation_file` (e.g. `"README.md"`), `changelog_summary` (e.g., `"v1.0: initial release"`).
            * Example: `name = "UartCam"; version = "1.0.0"; architectures="arm"; exports_functions="init,send"; interface_intrinsic_functions="platformInit,platformSend"; source_file="UartCam.bml";`
*   **CAM Benefits:** Architecture-specific optimization, hardware abstraction, code reusability, compiler extensibility, type safety, generics.
*   **CAM Configuration:** `#use module ... with config { ... };` configures CAM parameters.
*  **CAMs in OS Development:** CAMs are used to implement OS components and concurrency features.

**5. Directives and Pragmas**

*   **Preprocessor Directives (Start with `#`):**
    *   `#define identifier value`: Macro definition for compile time variables.
        * Example: `#define MAX_BUFFER_SIZE 1024`
    *    `#if condition`, `#elif condition`, `#else`, `#endif`: Conditional compilation.
        *   Condition expressions support macro expansion and boolean logic (`==`, `!=`, `&&`, `||`, `!`).
          * Example: `#if DEBUG_MODE == true; ... #endif`.
    *   `#use module "moduleName" [with config { ... }] ;`: Module inclusion.
        *   Generic modules: `#use module "genericModuleName<Type>" as alias;`
            *   Example: `#use module "GenericVector<uint32>" as intVector;`
    *   `#cpuTarget cpuName`: CPU optimization hint.
         * Example:  `#cpuTarget armCortexM4`
    *   `#rawBinary binaryEncodingList ;`: Embeds raw byte sequences using hex (`0x10`), binary (`0b10`), decimal (`10`), or octal literals (`0o10`).
         * Example: `#rawBinary 0x10 0b00010000 16 0o20;`

*   **Operation Directives:** `#operation operationName [argumentList] ;` provides a way to access hardware functionality through specific operations.
     *   **Boot Code Specific:**
        *   `archCodeBlock identifier @arc architectureName { ... }`: Architecture-specific raw code with `#goto`, `#operation`, `rawBinary`, registers and memory access.
         *  Example:
        ```baremetal
            archCodeBlock bootEntry @arc arm {
                conceptualRegisters {
                  register %stackPointer : address;
               }
             statements {
                #operation register %stackPointer = 0x20001000;
               #goto bootLoaderLabel;
                bootLoaderLabel:
            // ...
            }
         }
        ```
      *    `#interruptVector vectorNumber, functionName ;`: Defines interrupt vector table entries.
          * Example: `#interruptVector 0, resetHandler;`
         *   `#enableInterrupts ;`: Enables global interrupts.
       *   `#disableInterrupts ;`: Disables global interrupts.
        *   `#clearInterruptFlag identifier ;`: Clears an interrupt flag.
    *  **Memory Mapped Registers:**
         *   `struct @mapped @address(address)`: Defines a memory-mapped structure (members are volatile unless marked `nonvolatile`).
              * Example: `type GpioRegisters = struct @mapped @address(0x4000000) { uint32 dataRegister; };`
          *  `.` accesses members; `:=` sets/reads bitfields.
          *   `with Register { ... }` combines bitfield updates into a single register access.
              *   Example: `with myRegister { bitfield1 := 10; bitfield2 := otherField; };`

**6. Memory Safety in BML**

*  `DataType ^scoped`: Zero-overhead dangling pointer prevention.
    *   Pointers are valid only within their lexical scope.
    *  Example: `uint32^ scoped myPtr = &localValue;`
*  Capability-Based Pointers: `DataType ^readonly`, `DataType ^writeonly`, `DataType ^region(regionName)`, `DataType ^nonnull` for compile-time access control.
     *   Example: `uint32^ readonly myReadPtr;`, `raw byte^ writeonly myWritePtr;`
*   Static Typing: Enforces type rules.
*   `safeArray<DataType, Size>`: Runtime bounds checking.
     *   Example: `safeArray<uint16, 100> sensorValues; sensorValues[10] = 5;`
      *   `arraySlice<DataType>`: Provides bounds-checked views.
          *  Example: `arraySlice<uint32> mySlice = mySafeArray.slice(0,10);`
*   `DataType^`: Unchecked pointers (use with caution).
    *    Example: `uint32^ ptr = &memoryLocation;`
*   `DataType ^tracked`: Runtime checks.
    *   Example:  `uint32^ tracked myTrackedPtr = &someValue;`
*   Region-Based Memory Management: Implicit lifetime management through scopes.
*   Compile-Time Null Dereference Detection: Static analysis provides warnings for potential issues.
    *    Example: When a value is known to be potentially null, the compiler will generate a warning if the developer attempts to dereference it, without performing a null check.
*   Refinement Types: Compile-time data constraints.
    *   Example: `type validValue = uint16 where value < 100;`
*    Capability-Based Regions: Memory access control, enforced at compile time (mostly).
       *   Example: `dataRegion secureRegion capabilities(read) { uint32 myData; };`
   *  Stack-Only Allocation: `@stackOnly` enforces stack allocation, useful for performance and memory predictability.
       *   Example: `@stackOnly uint8 myStackVar;`
    *  Hardware-Assisted Bounds Checking: Optional, when hardware is available, and enabled via compiler flag or pragma.
          * Example `#pragma enable_hardware_bounds_checking; safeArray<uint32, 10> myArray;`

**7. Multi-Target Architecture Versatility**

*   `@arc architectureName { ... }`:  Provides architecture-specific code.
   *  Example: `@arc arm { /* ... */ }`, `@arc riscv { /* ... */ }`, `@arc x86 { /* ... */ }`
*  CAMs: Implement hardware abstractions with platform optimizations.
*   `#cpuTarget cpuName`: Provides CPU optimization hints.
  * The compiler selects CAMs and generates code based on the `-target` flag.

**8. Concurrency and Parallelism**

*   `thread`: Creates concurrent tasks.
     *   Example: `thread myThread myThreadFunction();`
*  `parallel`: Structured concurrency.
  *  Example: `parallel { /* ... */ }`
*    `await` / `yield`: Asynchronous operations.
    *   Example: `await myAwaitable; yield myAwaitable;`
*   `mutex` / `lock` / `unlock`: Critical section access.
     *   Example: `lock(myMutex); unlock(myMutex);`
*  `semaphore` / `wait` / `signal`: Synchronization.
     *   Example: `wait(mySemaphore); signal(mySemaphore);`
*   Default memory consistency is sequential consistency, using `#operation memoryBarrier` for explicit ordering. There is no implicit memory barrier at the end of `parallel` blocks.

**9. Tooling and Debugging**

*   **`bmlc` Compiler:** Command-line tool for compiling BML code:
    *  `-target <architecture>`: Target architecture.
    *  `-g`: Enables debug information for GDB/LLDB.
        *  `-S`: Outputs assembly code.
     *    `-O<level>`: Optimization level.
    *   `-I<include_path>`: Add include path.
       *  `-cam_path <cam_path>`: CAM module path.
    *   `-enableAi`: Enable abstract interpretation (experimental - may increase compilation time and may be unstable).
*   **Source-Level Debugging:** With GDB or LLDB (command-line).
*   **`BML Advanced Compiler Guide`:** (Separate document) - Compiler internals and advanced debugging.

**10. Standard Library and CAM Ecosystem**

*   **Minimal Core Library:** Zero-overhead. Examples:
    *  `uint32 stringLength(StringLiteral str) -> uint32;` (string length).
    *  `toUint32(int16 value) -> uint32;` (integer conversion).
*   **CAM Ecosystem:** Provides extended and reusable components.

**11. Conformance and Language Evolution**

*   **Conforming Compiler:** Adheres to BML Language Specification.
*   **Conforming Program:** Valid syntax, type-correct, uses supported features, and compatible CAMs.
*   **Language Evolution:** Backward compatibility and community participation are encouraged.

**12. Appendix: EBNF Grammar**

```ebnf
SourceFile =
    MultiTargetSourceFile
  | ModuleDefinitionFile ;

MultiTargetSourceFile =
    { PreprocessorDirectiveStatement }
 , { RegionDeclarationBlock }
  , { ModuleUseDirective }
   , { TargetArchitectureBlock }
  , EOF ;

ModuleDefinitionFile =
     ModuleDefinition , EOF ;

ModuleDefinition =
    [ GenericModuleDeclarationHeader ] , "module" , ModuleName , "{" ,
   ModuleDeclarationBlock ,
    ModuleStatementBlock ,
    ExportBlock ,
   "}" ;

GenericModuleDeclarationHeader =
     "generic" , "<" , TypeParameterList , ">" ;

ModuleDeclarationBlock =
     "declarations" , "{" , { Declaration } , "}" ;

ModuleStatementBlock =
     "statements" , "{" , { Statement } , "}" ;

ExportBlock =
    "export" , "{" ,
    [ ExportedFunctions ] ,
     [ ExportedTypes ] ,
    [ ExportedConstants ] ,
    [ ExportedConceptualRegisters ] ,
    [ ExportedInterfaceIntrinsicFunctions ] ,
    "}" ;

ExportedFunctions =
  "functions" , "{" , [ ExportFunctionList ] , "}" ;
ExportedTypes =
    "types" , "{" , [ ExportTypeList ] , "}" ;
ExportedConstants =
  "constants" , "{" , [ ExportConstantList ] , "}" ;
ExportedConceptualRegisters =
  "conceptualRegisters" , "{" , [ ExportConceptualRegisterList ] , "}" ;
ExportedInterfaceIntrinsicFunctions =
   "interface intrinsic functions" , "{" , [ ExportInterfaceIntrinsicFunctionList ] , "}" ;

ExportFunctionList =
     Identifier { "," , Identifier } ;
ExportTypeList =
    Identifier { "," , Identifier } ;
ExportConstantList =
  Identifier { "," , Identifier } ;
ExportConceptualRegisterList =
    Identifier { "," , Identifier } ;
ExportInterfaceIntrinsicFunctionList =
   FunctionSignature { "," , FunctionSignature } ;

TargetArchitectureBlock =
    "@arc" , ArchitectureName , "{" ,
    DeclarationBlock ,
    StatementBlock ,
    [ BootFunctionDefinition ],
    "}" ;

DeclarationBlock =
   "declarations" , "{" , { Declaration } , "}" ;

StatementBlock =
   "statements" , "{" , { Statement } , "}" ;

Declaration =
    VariableDeclaration
   | ConstantDeclaration
    | FunctionDeclaration
  | TypeDefinition
    | StructDefinition
    | UnionDefinition
   | ConfigDefinition
  | ArchitectureConditionalDeclaration
    | archCodeBlock
   | InterruptServiceRoutineDeclaration
   | EnumDefinition
  | TypeAliasDeclaration
    | AttributedVariableDeclaration
   | AttributedFunctionDeclaration
    | AttributedStructDefinition
  | AttributedEnumDefinition
  | ThreadDeclaration
    | MutexDeclaration
    | SemaphoreDeclaration
  | AwaitableDeclaration
    | DestructuringDeclaration
    | InterfaceIntrinsicFunctionDeclaration
    | RefinementTypeDefinition ;

RefinementTypeDefinition =
     "type" , Identifier , "=" , BaseDataType , "where" , ConstraintExpression , ";" ;

ConstraintExpression =
   ValueRangeConstraint
  | LengthConstraint
 ;

ValueRangeConstraint =
    "value" , "in" , "range" , "(" , ConstantExpression , ".." , ConstantExpression , ")" ;

LengthConstraint =
     "length" , "<" , ConstantExpression ;

InterfaceIntrinsicFunctionDeclaration =
    "interface" , "intrinsic" , "function" , FunctionSignature , ";" ;

FunctionSignature =
   [ GenericFunctionDeclarationHeader ] , Identifier , "(" , [ ParameterList ] , ")" , [ "->" , DataType ] ;

VariableDeclarationCore =
    DataType Identifier [ "=" Expression ] ";"
  | [ MutabilityModifier ],  Identifier , ":=" , Expression  , ";" ;

VariableDeclaration =
     [ TypeAttributeList ], [ MutabilityModifier ], VariableDeclarationCore ;

MutabilityModifier =
    "~" ;

ConstantDeclaration =
   "const" , [ "export" ] , DataType , Identifier , "=" , ConstantExpression , ";" ;

FunctionDeclaration =
    "function" , [ "inline" | "pure" | "noinline" | BootFunctionAttribute | GenericFunctionDeclarationHeader | StackOnlyFunctionAttribute | CompileTimeFunctionAttribute ] , [ "export" ] , Identifier , "(" , [ ParameterList ] , ")" , [ "->" , DataType ] , Block ;

BootFunctionDefinition =
   BootFunctionDeclaration , Block ;

BootFunctionDeclaration =
   "@boot" , "function" , [ "inline" | "pure" | "noinline" | GenericFunctionDeclarationHeader | StackOnlyFunctionAttribute | CompileTimeFunctionAttribute ] , Identifier , "(" , [ ParameterList ] , ")" , [ "->" , DataType ] , Block ;

BootFunctionAttribute =
    "@boot" ;

CompileTimeFunctionAttribute =
   "compileTimeFunction" ;

StackOnlyFunctionAttribute =
    "@stackOnly" ;

GenericFunctionDeclarationHeader =
    "generic" , "<" , TypeParameterList , ">" ;

TypeParameterList =
    Identifier { "," , Identifier } ;

ParameterList =
   Parameter { "," , Parameter } ;

Parameter =
    DataType , Identifier [ "=" ConstantExpression ] ;

TypeDefinition =
    "type" , [ "export" ] , Identifier , "=" , DataTypeExpression , ";" ;

DataTypeExpression =
   DataType ;

StructDefinition =
    "struct" , [ TypeAttributeList ] , Identifier , [ "@mapped" , [ "@address" , "(" , AddressExpression , ")" ] ] , "{" ,
     { StructMemberDeclaration }
    "}" , ";" ;

StructMemberDeclaration =
    AttributedStructMemberDeclaration
   | StructMemberDeclarationCore ;

AttributedStructMemberDeclaration =
    TypeAttributeList , StructMemberDeclarationCore ;

StructMemberDeclarationCore =
    [ "nonvolatile" ] , [ "volatile" ] , DataType , Identifier , [ DataLayoutAttribute ] , ";" ;

UnionDefinition =
   "union" , Identifier , "{" ,
   { UnionMemberDeclaration }
    "}" , ";" ;

UnionMemberDeclaration =
    DataType , Identifier , ";" ;

ConfigDefinition =
    "config" , Identifier , [ "extends" , Identifier ] , "{" ,
    { ConfigMemberDeclaration }
    "}" , ";" ;

ConfigMemberDeclaration =
   [ "override" ] , "config" , DataType , Identifier , "=" , ConstantExpression , ";" ;

ArchitectureConditionalDeclaration =
    "when" , ( "@arc" | "cpu" ) , "is" , CpuName , Declaration
  { "elseWhen" , ( "@arc" | "cpu" ) , "is" , CpuName , Declaration }
    [ "else" , Declaration ] ;

archCodeBlock =
    "archCodeBlock" , Identifier , [ "extends" , Identifier ] , "@arc" , ArchitectureName , "{" ,
  { BootStatement }
   "}" ;

InterruptServiceRoutineDeclaration =
    "interrupt" , [ InterruptAttributes ] , "function" , Identifier , "(" , ")" , Block ;

InterruptAttributes =
    "(" , InterruptAttribute { "," , InterruptAttribute } , ")" ;

InterruptAttribute =
   "fast"
  | ( "priority" , "=" , IntegerLiteral )
   | "naked" ;

ArchitectureName =
   "intel" | "arm" | "riscV" | Identifier ;

CpuName =
    "intelPentium4" | "intelCorei7" | "amdAthlon" | "amdRyzen" | "armCortexM0" | "armCortexM3" | "armCortexM4" | "armCortexM7" | "armCortexM33" | "armCortexM55" | "armCortexA7" | "armCortexA9" | "armCortexA15" | "armCortexA53" | "armCortexA57" | "armCortexA72" | "armCortexA76" | "armCortexA78" | "riscv32i" | "riscv32e" | "riscv64i" | "riscv64g" | Identifier ;

ModuleName =
  Identifier | StringLiteral ;

DataType =
    [ RegionSpecifier ] , { TypeQualifier } , BaseDataType , [ ":" , IntegerLiteral ] , [ ArrayDimension ], [ PointerSymbol ], [ GenericTypeInstantiation ] ;

BaseDataType =
    IntegerTypeSpecifier
   | FloatTypeSpecifier
    | "integer"
    | "longInt"
    | "address"
  | "codeAddress"
   | "dataAddress"
  | "genericAddress"
    | "bootAddress"
   | "boolean"
   | CharacterTypeSpecifier
    | "struct" , Identifier
  | "union" , Identifier
    | "config" , Identifier
   | Identifier
   | FunctionPointerType
    | "raw" , "byte"
   | SafeArrayType
  | DataType , "^scoped"
    | DataType , "^tracked"
    | "threadHandle"
    | "mutex"
    | "semaphore"
   | AwaitableType
  | ArraySliceType
   | TupleType
  | CapabilityPointerType
   | ResultType
  | RefinementType ;

RefinementType =
     Identifier;

ResultType =
    "Result" , "<" , DataType, "," , DataType, ">" ;

CapabilityPointerType =
    DataType , CapabilityQualifier;

CapabilityQualifier =
    "^readonly"
  | "^writeonly"
  | "^region" , "(" , Identifier, ")"
    | "^nonnull"
    | "^scoped"
    | "^tracked" ;

TupleType =
    "tuple" , "<" , DataType { "," , DataType } , ">" ;

ArraySliceType =
    "arraySlice" , "<" , DataType, ">" ;

AwaitableType =
    "awaitable" , "<" , DataType, ">" ;

SafeArrayType =
     "safeArray" , "<" , DataType, "," , ConstantExpression, ">" ;

PointerSymbol =
    "^" ;

FunctionPointerType =
     [ RegionSpecifier ] , "function" , "(" , [ ParameterList ] , ")" , [ "->" , DataType ] ;

RegionSpecifier =
    "dataRegion" , [ RegionCapabilitySpecifier ]
  | "rodataRegion" , [ RegionCapabilitySpecifier ]
  | "stackRegion" , [ RegionCapabilitySpecifier ]
    | "peripheralRegion" , [ RegionCapabilitySpecifier ]
    | "moduleDataRegion" , [ RegionCapabilitySpecifier ]
   | "bootCodeRegion" , [ RegionCapabilitySpecifier ]
    | "interruptVectorRegion" , [ RegionCapabilitySpecifier ] ;

RegionCapabilitySpecifier =
  "capabilities" , "(" , CapabilityList , ")" ;

CapabilityList =
   Capability { "," , Capability } ;

Capability =
    "read" | "write" | "execute" | "capability_derive" ;

RegionDeclarationBlock =
    BootRegionBlock
 | AttributedDataRegionBlock
  | AttributedRodataRegionBlock
    | AttributedStackRegionBlock
   | AttributedPeripheralRegionBlock
   | AttributedModuleDataRegionBlock
    ;

BootRegionBlock =
    "bootRegions" , "{" ,
  [ BootCodeRegionBlock ] ,
  [ BootDataRegionBlock ] ,
    [ InterruptVectorRegionBlock ] ,
   "}" ;

BootCodeRegionBlock =
   "bootCodeRegion" , "{" , { BootCodeRegionDeclaration } , "}" ;

BootDataRegionBlock =
  "bootDataRegion" , "{" , { BootDataRegionDeclaration } , "}" ;

InterruptVectorRegionBlock =
    "interruptVectorRegion" , "{" , { InterruptVectorRegionDeclaration } , "}" ;

AttributedDataRegionBlock =
  TypeAttributeList , DataRegionBlockCore;

AttributedRodataRegionBlock =
    TypeAttributeList , RodataRegionBlockCore;

AttributedStackRegionBlock =
  TypeAttributeList , StackRegionBlockCore;

AttributedPeripheralRegionBlock =
    TypeAttributeList , PeripheralRegionBlockCore;

AttributedModuleDataRegionBlock =
   TypeAttributeList , ModuleDataRegionBlockCore;

DataRegionBlockCore =
    "dataRegion" , [ RegionCapabilitySpecifier ], "{" , { DataRegionDeclaration } , "}" ;

RodataRegionBlockCore =
     "rodataRegion" , [ RegionCapabilitySpecifier ], "{" , { RodataRegionDeclaration } , "}" ;

StackRegionBlockCore =
   "stackRegion" , [ RegionCapabilitySpecifier ], "{" , { StackRegionDeclaration } , "}" ;

PeripheralRegionBlockCore =
  "peripheralRegion" , [ RegionCapabilitySpecifier ], "{" , { PeripheralRegionDeclaration } , "}" ;

ModuleDataRegionBlockCore =
   "moduleDataRegion" , [ RegionCapabilitySpecifier ], "{" , { ModuleDataRegionDeclaration } , "}" ;

DataRegionDeclaration =
    VariableDeclaration
  | ConstantDeclaration
    | StructDefinition
   | UnionDefinition
  | ConfigDefinition
    | TypeDefinition
    | Comment
   | EnumDefinition
    | InterfaceIntrinsicFunctionDeclaration
    | DestructuringDeclaration
    | StackOnlyVariableDeclaration
     | RefinementTypeDefinition
   ;


RodataRegionDeclaration =
    ConstantDeclaration
   | StructDefinition
    | UnionDefinition
    | ConfigDefinition
    | TypeDefinition
    | Comment
  | EnumDefinition
    | InterfaceIntrinsicFunctionDeclaration
  | DestructuringDeclaration
    | StackOnlyVariableDeclaration
   | RefinementTypeDefinition
;

StackRegionDeclaration =
    VariableDeclaration
  | Comment
   | EmptyStatement
    | InterfaceIntrinsicFunctionDeclaration
   | DestructuringDeclaration
    | StackOnlyVariableDeclaration
   | RefinementTypeDefinition
  ;

PeripheralRegionDeclaration =
   VariableDeclaration
   | ConstantDeclaration
   | Comment
   | EmptyStatement
    | InterfaceIntrinsicFunctionDeclaration
   | StructDefinition
  | DestructuringDeclaration
   | StackOnlyVariableDeclaration
  | RefinementTypeDefinition;

ModuleDataRegionDeclaration =
    VariableDeclaration
  | ConstantDeclaration
    | StructDefinition
    | UnionDefinition
    | ConfigDefinition
   | TypeDefinition
   | Comment
  | EnumDefinition
    | InterfaceIntrinsicFunctionDeclaration
    | DestructuringDeclaration
  | StackOnlyVariableDeclaration
    | RefinementTypeDefinition;

BootCodeRegionDeclaration =
  Comment
  | EmptyStatement
    | InterfaceIntrinsicFunctionDeclaration
   | RawBinaryDirectiveStatement
  ;

BootDataRegionDeclaration =
    VariableDeclaration
    | ConstantDeclaration
  | Comment
  | EmptyStatement
    | InterfaceIntrinsicFunctionDeclaration
   | DestructuringDeclaration
    | StackOnlyVariableDeclaration
    | RefinementTypeDefinition;

InterruptVectorRegionDeclaration =
    InterruptVectorDirective
   | Comment
  | EmptyStatement
    | InterfaceIntrinsicFunctionDeclaration
  ;

EnumDefinition =
   "enum" , Identifier , "{" ,
   [ EnumMemberList ]
    "}" , ";"
    |  "enum" , "TypeKind" , "{" ,
   [ TypeKindMemberList ]
    "}" , ";"
   | "enum" , "LayoutAttributeKind" , "{" ,
     [ LayoutAttributeKindMemberList ]
   "}" , ";" ;

EnumMemberList =
    EnumMember { "," , EnumMember } ;

EnumMember =   
  Identifier [ "=" ConstantExpression ] ;

TypeKindMemberList =
  TypeKindMember { "," , TypeKindMember } ;

TypeKindMember =
    "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" ;

LayoutAttributeKindMemberList =
    LayoutAttributeKindMember { "," , LayoutAttributeKindMember } ;

LayoutAttributeKindMember =
  "Packed" | "Aligned" | "RowMajor" | "ColumnMajor" | "StructureOfArrays" | "ArrayOfStructures" ;

TypeAliasDeclaration =
  "type" , Identifier , "=" , DataType , ";" ;

AttributedVariableDeclaration =
   TypeAttributeList , [ MutabilityModifier ], VariableDeclarationCore ;

AttributedFunctionDeclaration =
    TypeAttributeList , FunctionDeclaration ;

AttributedStructDefinition =
     TypeAttributeList , StructDefinition ;

AttributedEnumDefinition =
   TypeAttributeList , EnumDefinition ;

TypeAttributeList =
    "[" , TypeAttribute { "," , TypeAttribute } , "]" ;

TypeAttribute =
   Identifier
  | AttributeName , "(" , AttributeArgumentList , ")"
    | "boundsCheck"
   | "noBoundsCheck"
    | "boundsCheckStatic"
   | "optimize" , "(" , OptimizationAttributeList , ")"
    | "potentialNull"
  | "guaranteedNonNull"
   | "boundsCheck" , ":" , BoundsCheckOption
   | "stackOnly"
  | "layout" , "(" , LayoutAttributeList , ")"
 | "boundsCheckRefinement" , "(" , BoundsCheckOption , ")"
  | "hasDataLayoutAttribute" , "(" , LayoutAttributeKindIdentifier , ")"
    | "isType" , "(" , TypeKindIdentifier , ")"
    | "sizeof" , "(" , TypeKindIdentifier , ")"
    | "bitfield" , "(" , Identifier , "." , Identifier , ")" ;

LayoutAttributeList =
  LayoutAttribute { "," , LayoutAttribute } ;

LayoutAttribute =
    "packed"
   | "aligned" , "(" , IntegerLiteral , ")"
  | "row_major"
   | "column_major"
    | "structure_of_arrays"
   | "array_of_structures" ;

LayoutAttributeKindIdentifier =
   "Packed" | "Aligned" | "RowMajor" | "ColumnMajor" | "StructureOfArrays" | "ArrayOfStructures" ;

TypeKindIdentifier =
   "Integer" | "Float" | "Struct" | "Enum" | "Pointer" | "Array" | "Tuple" | "Result" | "Refinement" ;

BoundsCheckOption =
   "runtime" | "compileTime" | "auto" | "off" | Identifier ;

OptimizationAttributeList =
   OptimizationAttribute { "," , OptimizationAttribute } ;

OptimizationAttribute =
   "instructionSchedule" , ":" , InstructionScheduleOption ;

InstructionScheduleOption =
    "aggressive" | "default" | "none" | Identifier ;

AttributeName = Identifier;
AttributeArgumentList = ConstantExpression { "," , ConstantExpression };

RawBinaryDirectiveStatement =
     "#" , "rawBinary" , BinaryEncodingList , ";" ;

BinaryEncodingList =
   [ BinaryEncodingLiteral { BinaryEncodingLiteral } ] ;

BinaryEncodingLiteral =
    HexadecimalLiteral | BinaryLiteral | DecimalLiteral | OctalLiteral ;

BootStatement =
    BootOperationDirectiveStatement
  | BootRegisterDirectiveStatement
    | BootMemoryDirectiveStatement
    | BootControlFlowStatement
  | Comment
    | EmptyStatement
    | RawBinaryDirectiveStatement
   ;

BootOperationDirectiveStatement = OperationDirectiveStatement ;

BootRegisterDirectiveStatement =
    "register" , RegisterName , "=" , BootOperand , ";" ;

BootMemoryDirectiveStatement =
    ( "store" | "load" ) , BootDataSize , BootAddress , "," , BootOperand , ";" ;

BootDataSize =
   "byte" | "word" | "dword" | "qword" ;

BootAddress =
    ConstantExpression
   | MemoryAddressing ;

BootOperand =
    Register
    | ImmediateValue
    | BootMemoryOperand ;

BootMemoryOperand =
    "[" , BootAddress , "]" ;

BootControlFlowStatement =
    LabelDefinition
  | JumpStatementDirective
    | ConditionalJumpStatementDirective
    | GotoDirectiveStatement
    ;

LabelDefinition =
  Identifier , ":" , ";" ;

JumpStatementDirective =
    "jump" , AddressExpression , ";" ;

ConditionalJumpStatementDirective =
   "jumpIf" , ConditionCode , "," , Identifier , ";" ;

GotoDirectiveStatement =
     "#" , "goto" , Identifier , ";" ;

ConditionCode =
    "zero" | "notZero" | "carry" | "noCarry" | "overflow" | "noOverflow" | "greaterThan" | "greaterEqual" | "lessThan" | "lessEqual" | "equal" | "notEqual";

OperationDirectiveStatement =
    "#" , "operation" , OperationName , [ OperationArgumentList ] , ";" ;

OperationGroupStatement =
  "operationGroup" , Identifier , [ TypeAttributeList ] , "{" , { Statement } , "}" ;

InterruptVectorDirective =
    "#" , "interruptVector" , IntegerLiteral , "," , Identifier , ";" ;

EnableInterruptsDirectiveStatement =
  "#" , "enableInterrupts" , ";" ;

DisableInterruptsDirectiveStatement =
    "#" , "disableInterrupts" , ";" ;

ClearInterruptFlagDirectiveStatement =
   "#" , "clearInterruptFlag" , Identifier , ";" ;

PreprocessorDirectiveStatement =
     DefineDirectiveStatement
   | IfDirectiveStatement
    | UseModuleDirectiveStatement
   | CpuTargetDirectiveStatement
  ;

DefineDirectiveStatement =
   "#" , "define" , Identifier , PreprocessorExpression , LineTerminator ;

IfDirectiveStatement =
   IfBlock { ElseIfBlock } [ ElseBlock ] , EndIfBlock ;

IfBlock =
     "#" , "if" , PreprocessorExpression , LineTerminator, { PreprocessorDirectiveStatement | RegionDeclarationBlock | ModuleUseDirective | TargetArchitectureBlock | ModuleDefinition | Declaration | Statement | BootStatement | InterruptServiceRoutineDeclaration | Comment | EmptyStatement } ;

ElseIfBlock =
    "#" , "elif" , PreprocessorExpression, LineTerminator, { PreprocessorDirectiveStatement | RegionDeclarationBlock | ModuleUseDirective | TargetArchitectureBlock | ModuleDefinition | Declaration | Statement | BootStatement | InterruptServiceRoutineDeclaration | Comment | EmptyStatement } ;

ElseBlock =
   "#" , "else" , LineTerminator, { PreprocessorDirectiveStatement | RegionDeclarationBlock | ModuleUseDirective | TargetArchitectureBlock | ModuleDefinition | Declaration | Statement | BootStatement | InterruptServiceRoutineDeclaration | Comment | EmptyStatement } ;

EndIfBlock =
    "#" , "endif" , LineTerminator ;

UseModuleDirectiveStatement =
    "#" , "use" , "module" , ModuleName , [ GenericModuleInstantiation ] , [ "with" , "config" , "{" , { ConfigParameterAssignment } , "}" ] , ";" ;

GenericModuleInstantiation =
    "<" , ConcreteTypeList , ">" ;

CpuTargetDirectiveStatement =
     "#" , "cpuTarget" , CpuName , LineTerminator ;

PreprocessorExpression =
    ConstantExpression ;

Statement =
   ExpressionStatement
  | CompoundStatement
   | SelectionStatement
   | IterationStatement
  | JumpStatement
    | DirectiveStatement
   | OperationGroupStatement
    | EmptyStatement
    | ParallelBlockStatement
  | ThreadStatement
   | LockStatement
    | UnlockStatement
   | WaitStatement
    | SignalStatement
  | YieldStatement
    | AwaitStatement
   | ExactInstructionSequenceStatement
    | DestructuringDeclaration
   | StackOnlyVariableDeclarationStatement
   | CompileTimeFunctionCallStatement
   | WithStatement ;

StackOnlyVariableDeclarationStatement =
  "stackOnly" , VariableDeclarationCore ;

DestructuringDeclaration =
   [ TypeAttributeList ], [ MutabilityModifier ] , DataType , DestructuringVariableList , "from" , Identifier , ";" ;

DestructuringVariableList =
   Identifier { "," , Identifier } ;

ExactInstructionSequenceStatement = // REMOVED `exactInstr`
  ;

InstructionRepresentation =
    ;

OperandList =
  ;

Operand =
    ;

CompileTimeFunctionCallStatement = // ADDED CompileTimeFunctionCallStatement to support compiler extensions via CAMs
  Identifier , "(" , [ ArgumentList ] , ")" , ";" ; // ADDED CompileTimeFunctionCallStatement to support compiler extensions via CAMs

WithStatement =
     "with" , Identifier , "{" , { WithStatementMember } , "}" , ";" ;

WithStatementMember =
   Identifier , ":=" ,  Expression , ";" ;


ExpressionStatement =
    [ Expression ] , ";" ;

CompoundStatement =
    Block ;

Block =
    "{" , { Statement } , "}" ;

SelectionStatement =
    IfStatement
   | SwitchStatement ;

IfStatement =
    "if" , "(" , Expression , ")" , Statement , [ "else" , Statement ] ;

SwitchStatement =
    "switch" , "(" , Expression , ")" , "{" , { CaseStatement } , [ DefaultCaseStatement ] , "}" ;

CaseStatement =
   "case" , ConstantExpression , ":" , Statement , "break" , ";" ;

DefaultCaseStatement =
    "default" , ":" , Statement , "break" , ";" ;

IterationStatement =
  WhileLoopStatement
    | DoWhileLoopStatement
    | ForLoopStatement
  | RangeBasedForLoopStatement
   ;

RangeBasedForLoopStatement =
    "for" , "(" , Identifier , "in" , RangeExpression , ")" , Statement ;

RangeExpression =
   Expression , ".." , Expression ;

WhileLoopStatement =
    "while" , "(" , Expression , ")" , Statement ;

DoWhileLoopStatement =
    "do" , Statement , "while" , "(" , Expression , ")" , ";" ;

ForLoopStatement =
   "for" , "(" , ForInitializer , ForCondition , ForUpdater , ")" , Statement ;

ForInitializer =
     [ VariableDeclaration | Expression ] ;

ForCondition =
    [ Expression ] , ";" ;

ForUpdater =
   [ Expression ] ;

JumpStatement =
   GotoStatement
  | ContinueStatement
    | BreakStatement
   | ReturnStatement ;

GotoStatement =
  "goto" , Identifier , ";" ;

ContinueStatement =
    "continue" , ";" ;

BreakStatement =
    "break" , ";" ;

ReturnStatement =
   "return" , [ Expression | TupleLiteral ] , ";" ;

TupleLiteral =
    "tuple" , "(" , [ ArgumentList ] , ")" ;

Expression =
    AssignmentExpression ;

AssignmentExpression =
  ConditionalExpression [ AssignmentOperator AssignmentExpression ] ;

AssignmentOperator =
    "=" | "+=" | "-=" | "*=" | "/=" | "%=" | "&=" | "|=" | "^=" | "<<=" | ">>="
   | ":=" ;

ConditionalExpression =
    LogicalOrExpression [ "?" , Expression , ":" , ConditionalExpression ] ;

LogicalOrExpression =
  LogicalAndExpression { "||" LogicalAndExpression } ;

LogicalAndExpression =
   BitwiseOrExpression { "&&" BitwiseOrExpression } ;

BitwiseOrExpression =
    BitwiseXorExpression { "|" BitwiseXorExpression } ;

BitwiseXorExpression =
   BitwiseAndExpression { "^" BitwiseXorExpression } ;

BitwiseAndExpression =
   EqualityExpression { "&" EqualityExpression } ;

EqualityExpression =
  RelationalExpression { ( "==" | "!=" ) RelationalExpression } ;

RelationalExpression =
   ShiftExpression { ( "<" | ">" | "<=" | ">=" ) ShiftExpression } ;

ShiftExpression =
   AdditiveExpression { ( "<<" | ">>" ) AdditiveExpression } ;

AdditiveExpression =
     MultiplicativeExpression { ( "+" | "-" ) MultiplicativeExpression } ;

MultiplicativeExpression =
     CastExpression { ( "*" | "/" | "%" ) CastExpression } ;

CastExpression =
    UnaryExpression ;

UnaryExpression =
    PostfixExpression
   | SizeofExpression
   | UnaryOperator , CastExpression ;

UnaryOperator =
  "+" | "-" | "!" | "~" | "*" | "&" ;

SizeofExpression =
   "sizeof" , "(" , ( DataType | Expression ) , ")" ;

PostfixExpression =
     PrimaryExpressionWithLikelyUnlikely { PostfixOperator } ;

PostfixOperator =
  ArrayAccessSuffix
   | StructMemberAccessSuffix
  | UnionMemberAccessSuffix
    | MmrBlockMemberAccessSuffix
 | FunctionCallSuffix
    ;

FunctionCallSuffix =
    "(" , [ ArgumentList ] , ")" ;

ArrayAccessSuffix =
    "[" , Expression , "]" ;

StructMemberAccessSuffix =
    "." , Identifier ;

UnionMemberAccessSuffix =
   "." , Identifier ;

MmrBlockMemberAccessSuffix =
  "." , Identifier ;

PrimaryExpression =
   Identifier
  | ConstantLiteral
   | StringLiteral
  | "(" , Expression , ")"
    | FunctionCall
  | ArrayAccess
   | StructMemberAccess
  | UnionMemberAccess
   | MmrBlockAccess
    | ConceptualRegisterAccess
    | ResultConstructor
    | BitfieldAccess;

PrimaryExpressionCore =
     Identifier
   | ConstantLiteral
   | StringLiteral
  | "(" , Expression , ")" ;

PrimaryExpressionWithPath =
   PrimaryExpression
   | ArrayAccess
   | StructMemberAccess
  | UnionMemberAccess
    | MmrBlockAccess
    | ConceptualRegisterAccess;

PrimaryExpressionWithLikelyUnlikely =
    PrimaryExpressionWithPath
   | LikelyExpression
  | UnlikelyExpression;

LikelyExpression =
  "likely" , "(" , Expression , ")" ;

UnlikelyExpression =
    "unlikely" , "(" , Expression , ")" ;

FunctionCall =
    Identifier , [ GenericFunctionInstantiation ] , "(" , [ ArgumentList ] , ")" ;

GenericFunctionInstantiation =
   "<" , ConcreteTypeList , ">" ;

ArgumentList =
    Expression { "," , Expression } ;

ArrayAccess =
    PrimaryExpressionWithLikelyUnlikely , "[" , Expression , "]" ;

StructMemberAccess =
     PrimaryExpressionWithLikelyUnlikely , "." , Identifier ;

UnionMemberAccess =
     PrimaryExpressionWithLikelyUnlikely , "." , Identifier ;

MmrBlockAccess =
  PrimaryExpressionWithLikelyUnlikely , "." , Identifier ;

ConceptualRegisterAccess =
    ConceptualRegisterIdentifier ;

BitfieldAccess =
  PrimaryExpressionWithLikelyUnlikely , "." , Identifier;

CastOperator =
  "(" , DataType , ")" ;

ConstantExpression =
     ConstantLiteral
   | ConstantIdentifier
    | ConstantUnaryExpression
  | ConstantBinaryExpression
    | ConstantParenthesizedExpression ;

ConstantParenthesizedExpression =
    "(" , ConstantExpression , ")" ;

ConstantUnaryExpression =
   ( "+" | "-" | "!" | "~" ) , ConstantExpression ;

ConstantBinaryExpression =
    ConstantExpression , BinaryOperator , ConstantExpression ;

BinaryOperator =
    "+" | "-" | "*" | "/" | "%" | "==" | "!=" | "<" | ">" | "<=" | ">=" | "&&" | "||" | "&" | "|" | "^" | "<<" | ">>" ;

ConstantLiteral =
   IntegerLiteral
  | FloatLiteral
    | CharacterLiteral
   | BooleanLiteral ;

IntegerLiteral =
  DecimalLiteral
  | HexadecimalLiteral
   | BinaryLiteral
   | BCDLiteral
   | OctalLiteral ;

OctalLiteral =
    "0o" , OctalDigit { OctalDigit } ;

OctalDigit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" ;

DecimalLiteral =
    Digit { Digit } ;

HexadecimalLiteral =
     "0x" , HexadecimalDigit { HexadecimalDigit } ;

BinaryLiteral =
    "0b" , BinaryDigit { BinaryDigit } ;

BCDLiteral =
    "%" , Digit { Digit } ;

FloatLiteral =
    DecimalFloatLiteral
    | ExponentFloatLiteral ;

DecimalFloatLiteral =
    Digit { Digit } , "." , Digit { Digit } , [ FloatSuffix ] ;

ExponentFloatLiteral =
   ( Digit { Digit } | DecimalFloatLiteral ) , ( "e" | "E" ) , [ "+" | "-" ] , Digit { Digit } , [ FloatSuffix ] ;

FloatSuffix =
  "f" | "F" | "d" | "D" ;

CharacterLiteral =
   "'" , ( SourceCharacter | EscapeSequence ) , "'"
    | "L" , "'" , ( SourceCharacter | EscapeSequence ) , "'"
  | "U" , "'" , ( SourceCharacter | EscapeSequence ) , "'" ;

StringLiteral =
    '"' , { SourceCharacter | EscapeSequence } , '"'
   | "L" , '"' , { SourceCharacter | EscapeSequence } , '"'
    | "U" , '"' , { SourceCharacter | EscapeSequence } , '"' ;

BooleanLiteral =
    "true" | "false" ;

IntegerTypeSpecifier =
    SignedIntegerTypeSpecifier | UnsignedIntegerTypeSpecifier ;

SignedIntegerTypeSpecifier =
  "int8" | "int16" | "int32" | "int64" | "int" | "longInt";

UnsignedIntegerTypeSpecifier =
  "uint8" | "uint16" | "uint32" | "uint64";

FloatTypeSpecifier =
    "float32" | "float64" | "float";

CharacterTypeSpecifier =
    "char8" | "char16" | "char32" | "char";

TypeQualifier =
    "volatile"
  | "const"
    | "nonnull"
   | "restrict"
    | "device"
  | "readOnly"
   | "threadLocal"
    | "bootConstant"
   | "^readonly"
  | "^writeonly"
    | "^region" , "(" , Identifier, ")"
    | "^nonnull"
    | "stackOnly"
  | "^scoped"
  | "^tracked" ;

ConceptualRegisterIdentifierCore =
   "%" , Identifier ;

RegisterName =
    ConceptualRegisterIdentifier ;

ImmediateValue =
    ConstantExpression ;

Comment =
  SingleLineComment | MultiLineComment ;

SingleLineComment =
    "//" , { SourceCharacter } , LineTerminator ;

MultiLineComment =
     "/*" , { SourceCharacter } , "*/" ;

Whitespace =
    { " " | "\t" | LineTerminator } ;

LineTerminator = ? Unicode line terminator character (LF, CR, CRLF, etc.) ? ;

EmptyStatement =
    ";" ;

Identifier =
    Letter { Letter | Digit | "_" } ;

(* Lexical Terminals (Unicode categories) *)
Letter = ? Unicode letter character ? ;
Digit = ? Unicode digit character ? ;
HexadecimalDigit = Digit | "a" | "b" | "c" | "d" | "e" | "f" | "A" | "B" | "C" | "D" | "E" | "F" ;
BinaryDigit = "0" | "1" ;
SourceCharacter = ? any Unicode character ? ;
EscapeSequence =  "\\" , ( "n" | "r" | "t" | "\\" | "'" | '"' | "\0" ) ;
```
