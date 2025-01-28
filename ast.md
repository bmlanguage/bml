

# BML Abstract Syntax Tree (AST) - Version 1.0 - Monomorphic Design for Zero-Overhead Performance


**The BML AST is a meticulously designed, monomorphic data structure that forms the core of the BML compiler. Its architecture is optimized for zero-overhead access and extreme performance in code analysis, transformation, and generation. Extensibility is achieved through highly specialized Composable Architecture Modules (CAMs), ensuring maximum flexibility without compromising performance.**

## Introduction

The BML Abstract Syntax Tree (AST) provides a complete and highly detailed, yet strictly monomorphic, representation of source code. Unlike traditional AST designs, BML's AST prioritizes performance by eliminating runtime type checks and dynamic memory indirections. This monomorphic design guarantees predictable memory access patterns, while still providing full access to source code semantics. The AST is a direct target for code transformations performed by Composable Architecture Modules (CAMs), and is essential for achieving low-level, high-performance code generation.

## Key Performance and Design Features

*   **Strictly Monomorphic Core:** The core AST structure is composed of explicitly typed nodes and type information, eliminating all forms of runtime type dispatch, generic containers, and implicit memory indirection. This guarantees predictable performance and simplifies data access.

*   **Zero-Overhead Access:** The explicit type structures for nodes and type information allow for direct memory access with zero indirection. This predictable memory access pattern is extremely cache-friendly, maximizing performance.

*   **Compile-Time Specialization with CAMs:** Although monomorphic, the AST is designed to be manipulated via highly specialized Composable Architecture Modules (CAMs), which inject custom analysis passes and code transformations. CAMs are fully aware of each node's explicit type and rely on compile-time reflection and computation, ensuring zero runtime overhead.

*    **Explicit Type Structures:** The design eschews generic containers (`void*`, unions) in favor of explicit node and type structures. This reduces data access overhead, as well as the memory footprint, since no dynamic sizing is needed.

*   **Fine-Grained Type Properties:** Each node type and type information structure includes specific properties using explicit data structures and bitmasks. These allow for compile-time decision-making and specialized optimizations, ensuring minimal overhead in the generated code.

*   **Data Layout Attributes:** The AST is optimized for data-driven code generation based on data layout attributes (e.g., `packed`, `aligned`, `row_major`). These attributes enable precise control over memory layout, which leads to maximum performance in critical data processing loops.

*   **Direct Memory Access:** With explicit struct types, data is directly accessed, removing the need for indirect memory accesses via pointers to dynamically typed data.

*   **`exactInstr` for Ultimate Control:** `exactInstr { ... }` allows direct embedding of machine instructions when low-level control is required, which is a powerful feature for ultimate performance but should be used sparingly for safety and portability reasons.

### Usage: Low-Overhead Performance in Practice

**1. Specific AST Node Structures:**

The AST is built using specific, monomorphic node structures. Below is an example of an `IntegerLiteralNode_t` and a `FunctionDeclarationNode_t`:

```c
typedef struct IntegerLiteralNode_t {
    uint32_t nodeType;
    SourceLocation location;
    IntegerTypeInfo_t typeInfo;
    int64_t value;
    uint32_t properties;
} IntegerLiteralNode_t;

typedef struct FunctionDeclarationNode_t {
    uint32_t nodeType;
    SourceLocation location;
    FunctionTypeInfo_t typeInfo;
    char* identifier;
    ASTNodeUnion_t** parameters;
    uint32_t parameterCount;
    ASTNodeUnion_t* returnType;
    ASTNodeUnion_t* body;
      uint8_t inlineAttribute;
        uint8_t pureAttribute;
        uint8_t stackOnly;
        uint8_t compileTimeAttribute;
        GenericFunctionDeclarationHeaderNode* genericHeader;
    uint32_t properties;
} FunctionDeclarationNode_t;
```

   *  Note that the `ASTNodeUnion_t` types are used to maintain relationships between nodes.

**2. Specific Type Information Structures:**

Type information structures are also specifically designed:

```c
typedef struct IntegerTypeInfo_t {
    uint32_t typeCategory;
    uint32_t properties;
    int bitwidth;
    bool isSigned;
} IntegerTypeInfo_t;


typedef struct FunctionTypeInfo_t {
    uint32_t typeCategory;
    uint32_t properties;
     ASTNodeUnion_t* returnType;
    ASTNodeUnion_t** parameters;
  } FunctionTypeInfo_t;
```

**3. Monomorphic Node Creation:**

Each node type has its specific creation function. For example:

```c
IntegerLiteralNode_t* createIntegerLiteralNode(SourceLocation location, IntegerTypeInfo_t* typeInfo, int64_t value) {
    IntegerLiteralNode_t* integerNode = (IntegerLiteralNode_t*)malloc(sizeof(IntegerLiteralNode_t));
    integerNode->nodeType = AST_NODE_INTEGER_LITERAL;
    integerNode->location = location;
    integerNode->typeInfo = *typeInfo;
    integerNode->value = value;
    return integerNode;
}
```

**4.  Type-Safe Access Via Tagged Unions:**

Type-safe access of related or child AST nodes can be done using  `ASTNodeUnion_t`

  ```c
  typedef struct ASTNodeUnion_t{
          uint32_t nodeType;
         union {
              IntegerLiteralNode_t integerLiteral;
              FloatLiteralNode_t floatLiteral;
               FunctionCallNode_t functionCall;
                  FunctionDeclarationNode_t functionDeclaration;
              // and so on
              } data;
   } ASTNodeUnion_t;
  ```
   *   This allows the compiler to access the monomorphic data types based on node type, without any need for unsafe casts, implicit dynamic dispatch, or runtime overhead.

**5. Monomorphic CAMs:**

CAMs now explicitly handle each node type via pattern matching, ensuring zero-overhead specialized transformations.

```c
void optimizeNode(ASTNodeUnion_t* node) {
    switch(node->nodeType) {
        case AST_NODE_INTEGER_LITERAL:
        {
            IntegerLiteralNode_t integerLiteral = node->data.integerLiteral;
              // optimize int literal
         }
        break;
        case AST_NODE_FUNCTION_DECLARATION:
        {
                FunctionDeclarationNode_t functionNode = node->data.functionDeclaration;
                  //optimize function
         }
        break;
        // etc
    }
}
```

**6. Compile-Time Flexibility:**

CAMs use `sizeof()`, `isType()` and `compileTimeFunction` to generate specialized code at compile time based on explicit types and their properties, with zero runtime overhead.

```c
  // Compile Time Lambda example.
 auto stackAllocator = compileTimeFunction [](TypeInformation* typeInfo) {
      if (typeInfo->properties & AST_PROPERTY_IS_SAFEARRAY) {
          return stack_allocation;
       } else {
           return heap_allocation;
        }
    };
```

**7.  `exactInstr` for Low-Level Control:**

   * `exactInstr { ... }` can embed machine instructions for absolute control, at the cost of portability and safety. It should be used as a last resort only.

```
