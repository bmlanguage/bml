# BML Abstract Syntax Tree (AST) - Version 1.0 - Monomorphic Design for Zero-Overhead Performance

**The BML AST is a meticulously designed, monomorphic data structure that forms the core of the BML compiler. Its architecture is optimized for zero-overhead access and extreme performance in code analysis, transformation, and generation. Extensibility is achieved through highly specialized Composable Architecture Modules (CAMs), ensuring maximum flexibility without compromising performance.**

## Introduction

The BML Abstract Syntax Tree (AST) provides a complete and highly detailed, yet strictly monomorphic, representation of source code. Unlike traditional AST designs, BML's AST prioritizes performance by eliminating runtime type checks and dynamic memory indirections. This monomorphic design guarantees predictable memory access patterns, while still providing full access to source code semantics. The AST is a direct target for code transformations performed by Composable Architecture Modules (CAMs), and is essential for achieving low-level, high-performance code generation.

## Key Performance and Design Features

*   **Strictly Monomorphic Core:** The core AST structure is composed of explicitly typed nodes and type information, eliminating all forms of runtime type dispatch, generic containers, and implicit memory indirection. This guarantees predictable performance and simplifies data access.

*   **Zero-Overhead Access:** The explicit type structures for nodes and type information allow for direct memory access with zero indirection. This predictable memory access pattern is extremely cache-friendly, maximizing performance.

*   **Compile-Time Specialization with CAMs:** Although monomorphic, the AST is designed to be manipulated via highly specialized Composable Architecture Modules (CAMs), which inject custom analysis passes and code transformations. CAMs are fully aware of each node's explicit type and rely on compile-time reflection and computation, ensuring zero runtime overhead.

*   **Explicit Type Structures:** The design eschews generic containers (`void*`, unions) in favor of explicit node and type structures. This reduces data access overhead, as well as the memory footprint, since no dynamic sizing is needed.

*   **Fine-Grained Type Properties:** Each node type and type information structure includes specific properties using explicit data structures and bitmasks. These allow for compile-time decision-making and specialized optimizations, ensuring minimal overhead in the generated code.

*   **Data Layout Attributes:** The AST is optimized for data-driven code generation based on data layout attributes (e.g., `packed`, `aligned`, `row_major`). These attributes enable precise control over memory layout, which leads to maximum performance in critical data processing loops.

*   **Direct Memory Access:** With explicit struct types, data is directly accessed, removing the need for indirect memory accesses via pointers to dynamically typed data.

*   **`exactInstr` for Ultimate Control:** `exactInstr(StringLiteral code)` allows direct embedding of machine instructions when low-level control is required, which is a powerful feature for ultimate performance but should be used sparingly for safety and portability reasons.

### Usage: Low-Overhead Performance in Practice

**1. Specific AST Node Structures:**

The AST is built using specific, monomorphic node structures. Below is an example of `IntegerLiteralNode` and `FunctionDeclarationNode` LBM types:

```lbm
type IntegerTypeInfo = struct {
    typeCategory: uint32,
    properties: uint32,
    bitwidth: int,
    isSigned: boolean
};

type FunctionTypeInfo = struct {
    typeCategory: uint32,
    properties: uint32,
    returnType: address, // points to ASTNodeUnion
    parameters: address  // points to array of ASTNodeUnion
};

type SourceLocation = struct {
    filename: char*,
    line: uint32,
    column: uint32
};

type IntegerLiteralNode = struct {
    nodeType: uint32 #const,
    location: SourceLocation,
    typeInfo: IntegerTypeInfo,
    value: int64,
    properties: uint32
};

type GenericFunctionDeclarationHeaderNode = struct {
     typeParameterList: address // points to a type identifier array
};

type FunctionDeclarationNode = struct {
    nodeType: uint32 #const,
    location: SourceLocation,
    typeInfo: FunctionTypeInfo,
    identifier: char*,
    parameters: address, // Points to array of ASTNodeUnion
    parameterCount: uint32,
    returnType: address,  // Points to ASTNodeUnion
    body: address,      // Points to ASTNodeUnion
    inlineAttribute: uint8,
    pureAttribute: uint8,
    stackOnly: uint8,
    compileTimeAttribute: uint8,
    genericHeader: address,  // Points to GenericFunctionDeclarationHeaderNode
    properties: uint32
};
```

   * Note that the `address` type are used to maintain relationships between nodes in the union.
   * `#const` attribute specifies a constant value

**2. Specific Type Information Structures:**

Type information structures are also specifically designed. These LBM types correspond to the C structures `IntegerTypeInfo_t` and `FunctionTypeInfo_t` previously shown:

```lbm
type IntegerTypeInfo = struct {
    typeCategory: uint32,
    properties: uint32,
    bitwidth: int,
    isSigned: boolean
};

type FunctionTypeInfo = struct {
    typeCategory: uint32,
    properties: uint32,
    returnType: address,  // Points to ASTNodeUnion
    parameters: address // Points to array of ASTNodeUnion
};
```

**3. Monomorphic Node Creation:**

Each node type has its specific creation function. For example, in LBM :

```lbm
function IntegerLiteralNode* createIntegerLiteralNode(SourceLocation location, IntegerTypeInfo typeInfo, int64 value) {
    IntegerLiteralNode* integerNode = memoryAllocate(sizeof IntegerLiteralNode);
    integerNode.nodeType = AST_NODE_INTEGER_LITERAL;
    integerNode.location = location;
    integerNode.typeInfo = typeInfo;
    integerNode.value = value;
    return integerNode;
}

// Note: In real BML, memory allocation should be done using an allocator that is provided by the CAM.

function ASTNodeUnion* createFunctionDeclarationNode(SourceLocation location, FunctionTypeInfo typeInfo, char* identifier, ASTNodeUnion*[] parameters,  ASTNodeUnion* returnType, ASTNodeUnion* body) {
      FunctionDeclarationNode* functionNode = memoryAllocate(sizeof FunctionDeclarationNode);
    functionNode.nodeType = AST_NODE_FUNCTION_DECLARATION;
    functionNode.location = location;
    functionNode.typeInfo = typeInfo;
    functionNode.identifier = identifier;
    functionNode.parameters = parameters;
    functionNode.parameterCount = arrayLength(parameters);
    functionNode.returnType = returnType;
    functionNode.body = body;
     return functionNode;
 }
```

**4. Type-Safe Access Via Tagged Unions:**

Type-safe access of related or child AST nodes is done using  `ASTNodeUnion`.

```lbm
 type ASTNodeUnion = struct {
  nodeType: uint32,
  data: union {
        integerLiteral: IntegerLiteralNode,
        floatLiteral: FloatLiteralNode,
        functionCall: FunctionCallNode,
        functionDeclaration: FunctionDeclarationNode,
         // etc
       }
};
```

   * This allows the compiler to access the monomorphic data types based on `nodeType`, without any need for unsafe casts, implicit dynamic dispatch, or runtime overhead.

**5. Monomorphic CAMs:**

CAMs now explicitly handle each node type via pattern matching using a `switch` statement, ensuring zero-overhead specialized transformations.

```lbm
function void optimizeNode(ASTNodeUnion* node) {
    switch (node.nodeType) {
        case AST_NODE_INTEGER_LITERAL:
        {
           IntegerLiteralNode integerLiteral = node.data.integerLiteral;
           // Optimize int literal code using memory access to integerLiteral and other fields
          }
        break;
        case AST_NODE_FUNCTION_DECLARATION:
        {
          FunctionDeclarationNode functionNode = node.data.functionDeclaration;
            //optimize function
         }
         break;
         //etc
     }
}
```

**6. Compile-Time Flexibility:**

CAMs use `sizeof()`, `isType()` and `compileTimeFunction` to generate specialized code at compile time based on explicit types and their properties, with zero runtime overhead.

```lbm
  // Compile Time Lambda example.
 compileTimeFunction auto stackAllocator =  (TypeInformation* typeInfo) => {
   if( typeInfo.properties &  AST_PROPERTY_IS_SAFEARRAY ) {
      return "stack";
   } else {
      return "heap";
     }
};
```

**7. `exactInstr` for Low-Level Control:**

   *   `exactInstr(StringLiteral code)` can embed machine instructions for absolute control, at the cost of portability and safety. It should be used as a last resort only.

```lbm
 // intrinsic function can be implemented using a DSL
 interface intrinsic function void  exactInstr(StringLiteral code);
 exactInstr("mov r0, #10"); //example to use it
```

**8. Constants**

```lbm
// --- Constants for Node Types and Properties ---
const uint32 AST_NODE_INTEGER_LITERAL = 1;
const uint32 AST_NODE_FUNCTION_DECLARATION = 2;
const uint32 AST_PROPERTY_IS_SAFEARRAY = 4;
const StringLiteral stack_allocation = "stack";
const StringLiteral heap_allocation = "heap";
```
