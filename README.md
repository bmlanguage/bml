# Bare Metal Language (BML) - Work in Progress: Contribute to the Compiler!

**[MIT License]**

## Introduction

BML (Bare Metal Language) is a **statically-typed, compiled programming language** meticulously crafted for the demanding world of **bare-metal and embedded systems development.** Our goal is ambitious: to offer developers a productive, maintainable, and memory-safe alternative to assembly, while retaining peak performance and the ability to fine-tune low-level details.

BML does *not* seek to replicate assembly at the user level by providing direct instruction encoding at the user level. Instead, it elegantly abstracts instruction encoding, allowing you to work with high-level constructs, all while harnessing the power of a flexible architecture through our **Composable Architecture Module (CAM) system**. However, through its CAM DSLs, BML provides low-level access to core CPU features using specialized instruction primitives that allows the user to achieve a level of control that is comparable to assembly, when needed.

**Why No Direct User-Level Instruction Encoding (Unless Absolutely Needed)?**

The decision to *not* allow direct user-level instruction encoding by default is deliberate and rooted in the realities of embedded systems development:

*   **Architectural Diversity:** Embedded systems span a vast array of architectures (ARM, RISC-V, x86, custom), each with unique instruction sets, registers, and memory models. Exposing direct instruction encoding would result in code that is *not portable and platform-dependent*.
*   **Optimal Code Generation:** Raw instructions are not always the most optimal. Modern processors often have complex pipelines, caches, and specialized instructions that only a compiler can effectively exploit.
*   **Hardware Abstraction:** By abstracting the hardware complexity using CAMs, developers can benefit from the hardware details, without needing deep hardware-specific knowledge.
*   **Safety and Security:** Direct instruction encoding opens avenues for developers to write unsafe code and bypass security measures. By abstracting those details, we provide guarantees that low level code is correctly handled by the compiler.
*   **Focus on Application Logic:** We want developers to focus on *what* they want to accomplish, not *how* it must be translated into specific hardware opcodes. This separation of concerns significantly increases code quality, maintainability, and portability.
*   **Specialized Optimizations:** CAMs are able to perform architecture specific optimizations that are impossible or very complex for the user to do manually, while they abstract these complexities away from the user.

Instead, BML provides a better and safer alternative: **low-level control is delegated to the CAMs using DSLs and low-level hardware intrinsics, allowing users to customize the code generation**, and also to completely bypass the CAM generated abstractions and access the target hardware directly when needed, by using specialized low level primitives, including `raw` instructions.

## How BML Works: The Power of CAMs

At the heart of BML lies the CAM (Composable Architecture Module) system:

*   **CAMs as Compiler Extensions:** CAMs are compiler extensions, crafted in BML itself, that are responsible for generating optimized, platform-specific machine code, microcode, and instructions.
*   **Separation of Concerns:** Application developers can work with high-level BML constructs, while CAM developers create highly-optimized, architecture-specific implementations, ensuring clean separation between application logic and hardware details.
*   **Flexibility and Performance:** CAMs can be composed, customized, and tailored to specific hardware, ensuring both versatility and performance that is comparable to hand-written assembly.
*   **Low-Level Control:** Although BML does not provide direct user-level instruction encoding by default, low-level control is achieved using specialized DSLs in the CAMs, allowing access to all hardware features, and the `raw` keyword if a specific hardware operation cannot be expressed using a CAM abstraction.

Think of BML as giving you the power to *write code like you would in Python*, but *run it as efficiently as assembly*, with direct control over the low-level aspects, and the possibility to fine-tune even the microcode, if needed.

## Key BML Features

*   **High-Level Syntax, Assembly-Level Control:** Write in an intuitive, readable, high-level syntax while accessing low-level hardware resources via CAMs, microcode specification, low level DSLs, direct access to the compiler intermediate representation (IR), and fine tuning of code generation.
*   **Explicit Memory Safety:**
    *   Static typing ensures fewer runtime errors.
    *   Refinement types allow for compile-time value constraints.
    *   Region-based memory management and capability-based pointers for enhanced memory access control.
    *   Safe arrays, bounds checks, and tracked pointers help eliminate memory-related bugs.
*   **Multi-Target Architecture Support:** Compile code for diverse architectures through flexible CAMs and the use of  `@arc` blocks which enables a clear separation between architecture independent and architecture specific code regions.
*   **Main Entry File:** The project entry point is defined by using `statements` blocks inside `@arc` blocks in the main `.bml` file, which replaces the traditional `main()` function. The `statements` blocks defines the starting point of the application for a specific target architecture.
*   **Zero-Overhead Abstractions:** BML avoids hidden allocations, dynamic dispatch, and garbage collection, aiming for performance comparable to hand-written assembly.
*   **Concurrency and Parallelism:** Built-in primitives for multi-core processors and explicit control over synchronization using threads, mutexes, and semaphores.
*   **Metaprogramming:** Powerful compile-time computation, reflection, and DSLs to extend the language and enable advanced optimizations.
*   **Rule-Based Code Transformations:** The compiler and CAMs use symbolic execution and rule-based DSLs for advanced code analysis and optimizations.
*   **Fine-Grained Hardware Dispatch**: BML provides mechanisms for fine-grained hardware dispatch to accelerators and specialized cores using direct memory-mapped register accesses or specialized instruction sequences provided by CAMs, allowing the developer to explicitly target specific hardware.
*   **ABI, Microcode and Instruction Control**: Using the `callDSL` and `instrDSL`, BML allows for precise control over function calling conventions, stack management, microcode instructions, and ABI specification, enabling the user to have fine-grained control over the execution flow of the code and to optimize even the most specific details of the generated instructions.
*   **Ultra Low-Level Control**: BML provides access to very low level intrinsics, and the possibility to inspect and transform compiler internal structures, giving a high degree of control over the generated code.
*    **Compile-Time Customization:** Using compile time flags, the behavior of the generated code can be fully customized, in a target specific way.
*   **First-Class Code Blocks**: Lambda expressions allows developers to define functions or code blocks that can be used by other functions or modules, improving code reusability.
*   **Real-Time Features**: Built in support for real time systems, by providing explicit memory access control, thread priority and affinity using attributes, and cycle-accurate code generation when needed.
*   **Concise Bitfield Operations**: Using a concise syntax to access and manipulate bitfields, the code becomes more readable, more maintainable and less error prone.
*   **Idiomatic Error Propagation**: The `?` operator simplifies error handling, leading to more concise and readable code.
*   **Performance Optimization**: BML code can be annotated with performance hints, that allow the compiler to optimize code based on user defined hints.

## Why Choose BML?

*   **Increased Productivity:** Develop embedded systems with a high-level language and focus on your application logic instead of low-level details, while still having the tools to fine tune performance.
*   **Maintainable Code:** Reduce complexity, code coupling, and improve readability and long-term maintainability using a higher-level syntax.
*   **Improved Safety:** Reduce the risks of memory corruption, buffer overflows, and other runtime errors using built-in type safety mechanisms.
*   **Hardware Portability:** Easily port your BML code to multiple architectures by using different CAMs and the architecture specific code using `@arc` blocks.
*   **Peak Performance with Fine-Grained Control:** Generate highly optimized machine code with CAMs, and fine-tune low-level aspects, microcode, and instructions when needed, achieving similar performance as hand-written assembly.
*   **Customizable:** The CAM system provides a high degree of configurability using a declarative syntax, enabling optimization and hardware specific access strategies.

## Who Should Use BML?

BML is designed for:

*   Experienced embedded systems developers.
*   Developers looking for a more productive, maintainable, and memory-safe alternative to assembly, while also retaining fine grained control over low level code execution and generated instructions.
*   Developers building real-time systems that require low overhead and precise cycle accurate control.
*   Developers willing to engage and contribute to the CAM ecosystem, and also to create their own custom CAMs based on their hardware needs.

**Important Disclaimer:** BML is *not* an immediate, drop-in replacement for assembly *at the user level* when writing an application. It requires understanding embedded systems concepts and may be unsuitable for novice programmers. The goal is to create a powerful yet safe tool for highly specific and performance-sensitive use cases, that can also completely abstract the target hardware when needed.

## Getting Started

*   Follow the installation instructions in the project’s repository.
*   Read the detailed documentation for in-depth information on the language and its CAM system.
*   Explore the growing community for code examples and CAM definitions.

## Contributing

Contributions to the BML language, compiler, and CAM ecosystem are greatly appreciated.

**BML is still a work in progress, so we encourage contributions to the development of the compiler, language features, and the creation of open-source CAMs for diverse hardware platforms.**

Please refer to the `CONTRIBUTING.md` for guidelines.

## We believe BML will be transformative for embedded development by bringing high-level safety and productivity while still allowing the necessary control needed for resource-constrained bare metal programming. We're excited to see what you build with it!

This is very a very well designed language and we are constantly improving it, feel free to provide any feedback, suggestions, and open source contributions to help us achieve our goal.
```
