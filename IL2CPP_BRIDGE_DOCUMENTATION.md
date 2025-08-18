# IL2CPP Bridge Library - Comprehensive Documentation

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Generic Types](#generic-types)
4. [Value Types](#value-types)
5. [Boxing and Unboxing](#boxing-and-unboxing)
6. [Method Overloads](#method-overloads)
7. [Usage Examples](#usage-examples)
8. [API Reference](#api-reference)

## Overview

The `frida-il2cpp-bridge` library is a powerful Frida module designed to interact with IL2CPP (Intermediate Language To C++) applications at runtime. It provides comprehensive functionality for dumping, tracing, and manipulating IL2CPP applications without requiring the `global-metadata.dat` file.

### Key Features
- Runtime IL2CPP application analysis
- Class, method, and field introspection
- Method interception and replacement
- Generic type handling
- Value type manipulation
- Boxing/unboxing operations
- Method overload resolution
- Cross-platform support (Android, Linux, Windows, iOS, macOS)

### Supported Unity Versions
- Unity 5.3.0 through 6000.1.x
- Comprehensive backward compatibility

## Architecture

The library is structured into several core components:

### Core Modules
- **`lib/index.ts`**: Main entry point and module loader
- **`lib/exports.ts`**: Native IL2CPP API bindings
- **`lib/application.ts`**: Application context management
- **`lib/memory.ts`**: Memory management utilities
- **`lib/tracer.ts`**: Method tracing functionality

### Structural Components (`lib/structs/`)
- **`class.ts`**: IL2CPP class representation
- **`method.ts`**: Method handling and invocation
- **`type.ts`**: Type system implementation
- **`object.ts`**: Object manipulation
- **`value-type.ts`**: Value type handling
- **`field.ts`**: Field access and modification

### Utility Components (`lib/utils/`)
- **`lazy.ts`**: Lazy initialization patterns
- **`native-struct.ts`**: Native structure handling
- **`console.ts`**: Enhanced console functionality

## Generic Types

The IL2CPP Bridge provides comprehensive support for generic types, including both generic classes and generic methods.

### Generic Class Handling

#### Basic Generic Class Operations
```typescript
// Access a generic class definition
const listClass = Il2Cpp.corlib.class("System.Collections.Generic.List`1");

// Check if a class is generic
console.log(listClass.isGeneric); // true

// Get generic parameters
console.log(listClass.generics.length); // 1 for List<T>

// Inflate a generic class with concrete types
const stringListClass = listClass.inflate(Il2Cpp.corlib.class("System.String"));
console.log(stringListClass.fullName); // System.Collections.Generic.List<System.String>
```

#### Generic Class Properties
```typescript
// Generic class identification
class.isGeneric: boolean          // Whether the class is a generic definition
class.isInflated: boolean         // Whether the class has concrete type parameters
class.generics: Il2Cpp.Class[]   // Array of generic parameter classes
class.genericClass: Il2Cpp.Class | null // Original generic definition for inflated classes
```

#### Working with Inflated Generic Classes
```typescript
// Create instances of generic types
const stringList = stringListClass.new();

// Access methods on generic instances
const addMethod = stringList.method("Add", 1);
addMethod.invoke(Il2Cpp.string("Hello"));

// Generic inheritance handling
class PartiallyInflated<T> : AbstractGeneric<T, String> { }
const partialClass = Il2Cpp.domain.assembly("GameAssembly")
    .image.class("PartiallyInflatedClass`1");
```

### Generic Method Handling

#### Generic Method Operations
```typescript
// Access a generic method
const method = someClass.method("StaticGenericMethod");

// Check if method is generic
console.log(method.isGeneric); // true
console.log(method.isInflated); // false (not yet inflated)

// Get generic parameters
console.log(method.generics.length); // Number of generic parameters

// Inflate with concrete types
const inflatedMethod = method.inflate(
    Il2Cpp.corlib.class("System.String"),
    Il2Cpp.corlib.class("System.Int32")
);

// Invoke the inflated method
inflatedMethod.invoke(Il2Cpp.string("test"), 42);
```

#### Generic Method Properties
```typescript
method.isGeneric: boolean         // Whether method has generic parameters
method.isInflated: boolean        // Whether method has concrete type arguments
method.generics: Il2Cpp.Class[]  // Generic parameter types
```

#### Advanced Generic Method Usage
```typescript
// Method overload resolution with generics
const overloadedGeneric = someClass.method("ProcessData")
    .overload("System.Collections.Generic.List`1<System.String>");

// Generic method inflation with complex types
const complexMethod = someClass.method("Transform")
    .inflate(
        Il2Cpp.corlib.class("System.Collections.Generic.Dictionary`2")
            .inflate(
                Il2Cpp.corlib.class("System.String"),
                Il2Cpp.corlib.class("System.Object")
            )
    );
```

### Generic Type Enumeration Values

The type system uses specific enumeration values for generic types:

```typescript
// Generic type identification
Il2Cpp.Type.Enum.VAR           // Generic parameter in class definition
Il2Cpp.Type.Enum.MVAR          // Generic parameter in method definition
Il2Cpp.Type.Enum.GENERIC_INSTANCE // Inflated generic type

// Example usage
if (type.enumValue === Il2Cpp.Type.Enum.VAR) {
    console.log("This is a generic class parameter");
}
```

## Value Types

Value types in IL2CPP include primitives, structs, and enums. The library provides comprehensive support for value type manipulation.

### Value Type Identification

```typescript
// Check if a class represents a value type
class.isValueType: boolean       // True for structs, primitives, enums
class.isStruct: boolean         // True for struct types
class.isEnum: boolean           // True for enum types
class.isPrimitive: boolean      // True for built-in primitives
```

### Primitive Types

The library supports all standard .NET primitive types:

```typescript
// Primitive type enumeration
Il2Cpp.Type.Enum.VOID     // System.Void
Il2Cpp.Type.Enum.BOOLEAN  // System.Boolean
Il2Cpp.Type.Enum.CHAR     // System.Char
Il2Cpp.Type.Enum.BYTE     // System.SByte
Il2Cpp.Type.Enum.UBYTE    // System.Byte
Il2Cpp.Type.Enum.SHORT    // System.Int16
Il2Cpp.Type.Enum.USHORT   // System.UInt16
Il2Cpp.Type.Enum.INT      // System.Int32
Il2Cpp.Type.Enum.UINT     // System.UInt32
Il2Cpp.Type.Enum.LONG     // System.Int64
Il2Cpp.Type.Enum.ULONG    // System.UInt64
Il2Cpp.Type.Enum.NINT     // System.IntPtr
Il2Cpp.Type.Enum.NUINT    // System.UIntPtr
Il2Cpp.Type.Enum.FLOAT    // System.Single
Il2Cpp.Type.Enum.DOUBLE   // System.Double
```

### Working with Value Types

#### Value Type Creation and Manipulation
```typescript
// Create value type instances
const structClass = Il2Cpp.domain.assembly("GameAssembly").image.class("MyStruct");
const structInstance = structClass.new();

// Access value type fields
const field = structInstance.field<number>("myField");
field.value = 42;

// Value type methods
const method = structInstance.method("Calculate");
const result = method.invoke();
```

#### Value Type Size Information
```typescript
// Get value type size
class.valueTypeSize: number      // Size in bytes of the value type
class.actualInstanceSize: number // Actual instance size including headers
```

### Enum Handling

```typescript
// Working with enums
const enumClass = Il2Cpp.corlib.class("System.DayOfWeek");
console.log(enumClass.isEnum); // true

// Get enum base type
const baseType = enumClass.baseType; // System.Int32

// Create enum values
const enumInstance = enumClass.new();
```

### Struct Operations

```typescript
// Struct-specific operations
const structClass = Il2Cpp.domain.assembly("GameAssembly").image.class("MyStruct");

// Check struct properties
console.log(structClass.isStruct); // true
console.log(structClass.isValueType); // true

// Struct field access
const structInstance = structClass.new();
const field = structInstance.field<string>("name");
field.value = "Test";
```

## Boxing and Unboxing

Boxing and unboxing are fundamental operations for converting between value types and reference types.

### Boxing Operations

#### Automatic Boxing with `Il2Cpp.boxed()`
```typescript
// Box primitive values
const boxedBool = Il2Cpp.boxed(true);
const boxedInt = Il2Cpp.boxed(42);
const boxedPtr = Il2Cpp.boxed(ptr(0x1000));

// Box with explicit type specification
const boxedByte = Il2Cpp.boxed(255, "uint8");   // System.Byte
const boxedShort = Il2Cpp.boxed(1000, "int16"); // System.Int16
const boxedLong = Il2Cpp.boxed(123456789, "int64"); // System.Int64
const boxedChar = Il2Cpp.boxed(65, "char");     // System.Char

// Box pointers with type specification
const boxedIntPtr = Il2Cpp.boxed(ptr(0x1000), "intptr");  // System.IntPtr
const boxedUIntPtr = Il2Cpp.boxed(ptr(0x1000), "uintptr"); // System.UIntPtr
```

#### Supported Boxing Types
```typescript
// Numeric types
"int8"    -> System.SByte
"uint8"   -> System.Byte
"int16"   -> System.Int16
"uint16"  -> System.UInt16
"int32"   -> System.Int32
"uint32"  -> System.UInt32
"int64"   -> System.Int64
"uint64"  -> System.UInt64
"char"    -> System.Char

// Pointer types
"intptr"  -> System.IntPtr
"uintptr" -> System.UIntPtr

// Automatic type inference
boolean   -> System.Boolean
number    -> System.Int32 (default)
Int64     -> System.Int64
UInt64    -> System.UInt64
NativePointer -> System.IntPtr (default)
```

#### Manual Boxing with ValueType.box()
```typescript
// Box value types manually
const valueType = new Il2Cpp.ValueType(handle, type);
const boxedValue = valueType.box();
```

### Unboxing Operations

#### Basic Unboxing
```typescript
// Unbox objects to value types
const boxedInt = Il2Cpp.boxed(42);
const unboxedValue = boxedInt.unbox();

// Access the unboxed value
console.log(unboxedValue.field("m_value").value); // 42
```

#### Unboxing Validation
```typescript
// Unboxing only works on value types
const stringObj = Il2Cpp.string("Hello");
try {
    stringObj.unbox(); // Throws error
} catch (e) {
    console.log(e); // "couldn't unbox instances of System.String as they are not value types"
}

// Check before unboxing
if (obj.class.isValueType) {
    const valueType = obj.unbox();
    // Safe to unbox
}
```

### Boxing/Unboxing in Method Calls

```typescript
// Method parameters requiring boxing
const method = someClass.method("ProcessValue");

// Automatic boxing in parameter passing
method.invoke(42); // Automatically boxed if needed

// Manual boxing for explicit control
const boxedParam = Il2Cpp.boxed(42, "int32");
method.invoke(boxedParam);

// Field assignment with boxing/unboxing
const field = obj.field<number>("numericField");
field.value = 42; // May involve boxing/unboxing internally
```

### Performance Considerations

```typescript
// Efficient value type operations
const valueType = obj.unbox();
const ToString = valueType.method<Il2Cpp.String>("ToString", 0);

// Avoid boxing when possible for value types
if (ToString.class.isValueType) {
    // Direct call without boxing
    const result = ToString.invoke();
} else {
    // Requires boxing
    const result = valueType.box().toString();
}
```

## Method Overloads

The IL2CPP Bridge provides sophisticated method overload resolution that handles complex inheritance hierarchies and generic types.

### Basic Overload Resolution

#### Simple Overload Selection
```typescript
// Get specific overload by parameter types
const method = someClass.method("ProcessData")
    .overload("System.String");

const genericOverload = someClass.method("ProcessData")
    .overload("System.Collections.Generic.List`1<System.String>");

// Multiple parameter overload
const multiParamOverload = someClass.method("Calculate")
    .overload("System.Int32", "System.Double");
```

#### Overload with Class Objects
```typescript
// Use class objects for overload resolution
const stringClass = Il2Cpp.corlib.class("System.String");
const intClass = Il2Cpp.corlib.class("System.Int32");

const method = someClass.method("Process")
    .overload(stringClass, intClass);
```

### Advanced Overload Resolution

#### Inheritance-Based Resolution
The overload resolver uses a scoring system to find the best match:

```typescript
// Scoring system:
// - Exact type match: 2 points
// - Assignable type match: 1 point
// - Minimum score: parameter_count * 1
// - Maximum score: parameter_count * 2

class Root {}
class Child1 extends Root {}
class Child11 extends Child1 {}

// Method overloads:
// void Foo(Root obj)    - Score: 1 for Child11 parameter
// void Foo(Child1 obj)  - Score: 2 for Child11 parameter
// Foo(Child1) is selected as the better match
```

#### Complex Overload Scenarios
```typescript
// Test class with multiple overloads
const overloadTest = Il2Cpp.domain.assembly("GameAssembly")
    .image.class("OverloadTest");

// Generic overload resolution
const genericMethod = overloadTest.method("A")
    .overload("Child4`1<Root>");

// Multi-generic overload
const multiGenericMethod = overloadTest.method("A")
    .overload("Child41`2<Child1,Child2>");

// Static vs instance method disambiguation
const staticMethod = overloadTest.method("D")
    .overload("Child1");  // Static method

const instanceMethod = overloadTest.new().method("D")
    .overload("Root");    // Instance method
```

### Overload Enumeration

#### Listing All Overloads
```typescript
// Get all overloads of a method
const method = someClass.method("ProcessData");
for (const overload of method.overloads()) {
    console.log(`${overload.name}(${overload.parameters.join(', ')})`);
}

// Filter overloads by criteria
const staticOverloads = [...method.overloads()].filter(m => m.isStatic);
const genericOverloads = [...method.overloads()].filter(m => m.isGeneric);
```

### Safe Overload Resolution

#### Try-Based Overload Access
```typescript
// Safe overload resolution
const method = someClass.method("ProcessData")
    .tryOverload("System.String", "System.Int32");

if (method) {
    const result = method.invoke("test", 42);
} else {
    console.log("Overload not found");
}

// Fallback overload resolution
const primaryOverload = someClass.method("Process")
    .tryOverload("System.String");
    
const fallbackOverload = someClass.method("Process")
    .tryOverload("System.Object");
    
const selectedMethod = primaryOverload ?? fallbackOverload;
```

### Generic Method Overloads

#### Generic Overload Resolution
```typescript
// Generic method overload handling
const genericClass = Il2Cpp.domain.assembly("GameAssembly")
    .image.class("MethodInflateTest").nested("Parent`1");

// Inflate class first
const inflatedClass = genericClass.inflate(Il2Cpp.corlib.class("System.Object"));

// Then resolve generic method overloads
const method = inflatedClass.method("C")
    .overload("System.String");  // C<T>(String a)

// Or inflate method directly
const inflatedMethod = inflatedClass.method("C")
    .inflate(Il2Cpp.corlib.class("System.Int32"))
    .overload("System.Int32");   // C<T>(T a) where T = Int32
```

### Nested Class Overload Resolution

```typescript
// Overload resolution in nested classes
const nestedClass = Il2Cpp.domain.assembly("GameAssembly")
    .image.class("OverloadTest").nested("Nested");

// Inherited overloads from parent class
const parentOverload = nestedClass.method("A")
    .overload("Root");  // Inherited from OverloadTest

// New overloads in nested class
const nestedOverload = nestedClass.method("A")
    .overload("Child2"); // Defined in Nested class

// Overridden methods
const overriddenMethod = nestedClass.method("C")
    .overload();  // new int C() in Nested class
```

## Usage Examples

### Complete Generic Type Example
```typescript
Il2Cpp.perform(() => {
    // Access a generic collection
    const listClass = Il2Cpp.corlib.class("System.Collections.Generic.List`1");
    
    // Create a string list
    const stringListClass = listClass.inflate(Il2Cpp.corlib.class("System.String"));
    const stringList = stringListClass.new();
    
    // Add items
    const addMethod = stringList.method("Add", 1);
    addMethod.invoke(Il2Cpp.string("Hello"));
    addMethod.invoke(Il2Cpp.string("World"));
    
    // Get count
    const countProperty = stringList.field<number>("_size");
    console.log(`List count: ${countProperty.value}`);
    
    // Iterate through items
    const getMethod = stringList.method<Il2Cpp.String>("get_Item", 1);
    for (let i = 0; i < countProperty.value; i++) {
        const item = getMethod.invoke(i);
        console.log(`Item ${i}: ${item.content}`);
    }
});
```

### Value Type and Boxing Example
```typescript
Il2Cpp.perform(() => {
    // Work with a custom struct
    const structClass = Il2Cpp.domain.assembly("GameAssembly").image.class("MyStruct");
    const structInstance = structClass.new();
    
    // Set struct fields
    structInstance.field<number>("x").value = 10;
    structInstance.field<number>("y").value = 20;
    
    // Box the struct for method calls requiring objects
    const boxedStruct = structInstance.box();
    
    // Call method that requires boxed value
    const processMethod = someClass.method("ProcessStruct", 1);
    processMethod.invoke(boxedStruct);
    
    // Unbox back to value type
    const unboxedStruct = boxedStruct.unbox();
    console.log(`X: ${unboxedStruct.field<number>("x").value}`);
    
    // Work with primitives
    const boxedInt = Il2Cpp.boxed(42, "int32");
    console.log(`Boxed type: ${boxedInt.class.fullName}`);
    console.log(`Boxed value: ${boxedInt.field<number>("m_value").value}`);
});
```

### Method Overload Resolution Example
```typescript
Il2Cpp.perform(() => {
    const testClass = Il2Cpp.domain.assembly("GameAssembly").image.class("OverloadTest");
    const instance = testClass.new();
    
    // Test different overload scenarios
    const rootClass = testClass.nested("Root");
    const child1Class = testClass.nested("Child1");
    const child11Class = testClass.nested("Child11");
    
    // Create test instances
    const rootInstance = rootClass.new();
    const child1Instance = child1Class.new();
    const child11Instance = child11Class.new();
    
    // Test overload resolution with inheritance
    const methodA = instance.method("A");
    
    console.log(`A(Root): ${methodA.overload("Root").invoke(rootInstance)}`);         // Returns 0
    console.log(`A(Child1): ${methodA.overload("Child1").invoke(child1Instance)}`);   // Returns 1
    console.log(`A(Child11): ${methodA.overload("Child1").invoke(child11Instance)}`); // Returns 1 (best match)
    
    // Test generic overloads
    const child4Class = testClass.nested("Child4`1").inflate(rootClass);
    const child4Instance = child4Class.new();
    
    console.log(`A(Child4<Root>): ${methodA.overload("Child4`1<Root>").invoke(child4Instance)}`); // Returns 4
});
```

## API Reference

### Core Classes

#### Il2Cpp.Class
```typescript
class Class extends NativeStruct {
    // Generic properties
    isGeneric: boolean
    isInflated: boolean
    generics: Il2Cpp.Class[]
    genericClass: Il2Cpp.Class | null
    
    // Value type properties
    isValueType: boolean
    isStruct: boolean
    isEnum: boolean
    valueTypeSize: number
    
    // Methods
    inflate(...classes: Il2Cpp.Class[]): Il2Cpp.Class
    method<T>(name: string, parameterCount?: number): Il2Cpp.Method<T>
    field<T>(name: string): Il2Cpp.Field<T>
    new(): Il2Cpp.Object
    alloc(): Il2Cpp.Object
}
```

#### Il2Cpp.Method
```typescript
class Method<T = Il2Cpp.Method.ReturnType> extends NativeStruct {
    // Generic properties
    isGeneric: boolean
    isInflated: boolean
    generics: Il2Cpp.Class[]
    
    // Overload methods
    overload(...typeNamesOrClasses: (string | Il2Cpp.Class)[]): Il2Cpp.Method<T>
    tryOverload<U>(...typeNamesOrClasses: (string | Il2Cpp.Class)[]): Il2Cpp.Method<U> | undefined
    overloads(): Generator<Il2Cpp.Method>
    
    // Generic methods
    inflate<R>(...classes: Il2Cpp.Class[]): Il2Cpp.Method<R>
    
    // Invocation
    invoke(...parameters: Il2Cpp.Parameter.Type[]): T
    invokeRaw(instance: NativePointerValue, ...parameters: Il2Cpp.Parameter.Type[]): T
}
```

#### Il2Cpp.Object
```typescript
class Object extends NativeStruct {
    // Boxing/Unboxing
    unbox(): Il2Cpp.ValueType
    
    // Method and field access
    method<T>(name: string, parameterCount?: number): Il2Cpp.BoundMethod<T>
    field<T>(name: string): Il2Cpp.BoundField<T>
    
    // Utilities
    toString(): string
    class: Il2Cpp.Class
}
```

#### Il2Cpp.ValueType
```typescript
class ValueType extends NativeStruct {
    // Boxing
    box(): Il2Cpp.Object
    
    // Method and field access
    method<T>(name: string, parameterCount?: number): Il2Cpp.BoundMethod<T>
    field<T>(name: string): Il2Cpp.BoundField<T>
    
    // Type information
    type: Il2Cpp.Type
    
    // Utilities
    toString(): string
}
```

#### Il2Cpp.Type
```typescript
class Type extends NativeStruct {
    // Type classification
    isPrimitive: boolean
    isByReference: boolean
    enumValue: number
    
    // Associated class
    class: Il2Cpp.Class
    
    // Utilities
    name: string
    object: Il2Cpp.Object
    fridaAlias: NativeCallbackArgumentType
    
    // Comparison
    is(other: Il2Cpp.Type): boolean
}
```

### Utility Functions

#### Boxing Function
```typescript
function boxed<T extends boolean | number | Int64 | UInt64 | NativePointer>(
    value: T,
    type?: "int8" | "uint8" | "int16" | "uint16" | "int32" | "uint32" | 
           "int64" | "uint64" | "char" | "intptr" | "uintptr"
): Il2Cpp.Object
```

#### Performance Function
```typescript
function perform(block: () => void): void
```

### Constants and Enums

#### Type Enumeration
```typescript
Il2Cpp.Type.Enum = {
    VOID: number,
    BOOLEAN: number,
    CHAR: number,
    BYTE: number,
    UBYTE: number,
    SHORT: number,
    USHORT: number,
    INT: number,
    UINT: number,
    LONG: number,
    ULONG: number,
    NINT: number,
    NUINT: number,
    FLOAT: number,
    DOUBLE: number,
    POINTER: number,
    VALUE_TYPE: number,
    OBJECT: number,
    STRING: number,
    CLASS: number,
    ARRAY: number,
    NARRAY: number,
    GENERIC_INSTANCE: number,
    VAR: number,      // Generic class parameter
    MVAR: number      // Generic method parameter
}
```

This documentation provides comprehensive coverage of the IL2CPP Bridge library's capabilities, with particular focus on generic types, value types, boxing/unboxing, and method overload resolution as requested.