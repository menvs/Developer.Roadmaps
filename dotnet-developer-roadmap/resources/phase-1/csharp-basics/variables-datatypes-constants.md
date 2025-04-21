# 1. Variables, Data Types, Constant

## 1. Variables

A **variable** is a named memory location that holds a value, which can be changed during program execution. In C#, variables must be declared with a specific **data type**.

### **Declaring a Variable**

```csharp

int age = 25;  // Declaring and initializing a variable
string name = "John";  // String variable
double price = 99.99;  // Floating-point number
```

### **Rules for Variable Names**

- Must start with a letter or underscore (`_`).
- Cannot be a C# keyword (e.g., `int`, `class`).
- Case-sensitive (`Name` and `name` are different variables).
- Can contain letters, numbers, and underscores (e.g., `user_age`, `price99`).

## 2. Data types:

## a. Value Type:

These store actual values in Stack and include:

| **Data Type** | **Size** | **Example** | **Description** |
| --- | --- | --- | --- |
| `bool` | 1 byte | `bool isActive = true;` | Stores `true` or `false` |
| `byte` | 1 byte | `byte smallNum = 255;` | 8-bit unsigned integer (`0 to 255`) |
| `sbyte` | 1 byte | `sbyte smallNum = -128;` | 8-bit signed integer (`-128 to 127`) |
| `short` | 2 bytes | `short x = 30000;` | 16-bit signed integer (`-32,768 to 32,767`) |
| `ushort` | 2 bytes | `ushort y = 60000;` | 16-bit unsigned integer (`0 to 65,535`) |
| `int` | 4 bytes | `int age = 30;` | 32-bit signed integer (`-2.1B to 2.1B`) |
| `uint` | 4 bytes | `uint num = 4000000000;` | 32-bit unsigned integer (`0 to 4.2B`) |
| `long` | 8 bytes | `long bigNum = 9223372036854775807;` | 64-bit signed integer |
| `ulong` | 8 bytes | `ulong bigNum = 18446744073709551615;` | 64-bit unsigned integer |
| `float` | 4 bytes | `float pi = 3.14f;` | 32-bit floating-point number |
| `double` | 8 bytes | `double price = 99.99;` | 64-bit floating-point number |
| `decimal` | 16 bytes | `decimal balance = 9999.99m;` | 128-bit high-precision number (financial) |
| `char` | 2 bytes | `char letter = 'A';` | 16-bit Unicode character |
| `enum` | Depends on underlying type | `enum Status {Active, Inactive};` | Enumeration type (default `int`) |
| `struct` | Depends on fields | `struct Point { int x, y; }` | Custom value type stored in stack |

## b. Reference Type:

Therese store **references** (memory addresses) instead of actual values.

| **Data Type** | **Size** | **Example** | **Description** |
| --- | --- | --- | --- |
| `string` | Variable | `string message = "Hello";` | Immutable sequence of characters |
| `class` | Variable | `class Person { string Name; }` | Reference type, stored in heap |
| `object` | Variable | `object obj = 42;` | Base type for all reference types |
| `interface` | Variable | `interface IShape { void Draw(); }` | Defines behavior for classes and structs |
| `array` | Variable | `int[] numbers = {1, 2, 3};` | Arrays are reference types |
| `delegate` | Variable | `delegate void PrintMessage();` | Stores method references |

## c. Nullable Type:

Allow value types to hold null values, using a structure (Nullable<T>) that adds an additional flag HasValue and an underlying value.

```csharp
int? num = null; // Nullable int
double? salary = 5000.50;
```

### Breakdown Comparison: Nullable Type vs. Value Type vs. Reference Type

| **Aspect** | **Value Type** | **Nullable Type** | **Reference Type** |
| --- | --- | --- | --- |
| **Definition** | Holds a direct value of its type | A value type that can hold a `null` value as well as a value | Holds a reference to an object in memory |
| **Examples** | `int`, `double`, `struct`, `char` | `int?`, `double?`, `DateTime?` | `class`, `interface`, `delegate`, `string`, `array` |
| **Can be null?** | No | Yes, it can be assigned `null` | Yes, it can hold `null` as a valid value |
| **Stored in** | Stack (for local variables) or Heap (for fields in objects) | Stored on the stack or heap, based on usage (struct) | Stored on the heap (reference to object) |
| **Size** | Depends on the type (e.g., `int` is 4 bytes) | Same size as the value type, with a bit more for `HasValue` | Typically 4-8 bytes for the reference, but the object’s size varies |
| **Memory Management** | Directly managed by the stack or the object/struct allocation | Managed by `Nullable<T>`, which adds a `HasValue` flag to store nullability | Managed by garbage collection in the heap |
| **Default Value** | The default value for the type (e.g., `0` for `int`) | `null` (if not assigned a value) | `null` (if not initialized) |

## d. Implicitly Typed Variables (`var`)

The `var` keyword lets the complier determine the data type.  `var`  must be initialized when declared.

```csharp
var message = "Hello, C#";  // Compiler infers it as string
var age = 25;  // Compiler infers it as int
```

### **Comparision: `var` vs `dynamic` in C#**

| **Aspect** | **var** | **dynamic** |
| --- | --- | --- |
| **Definition** | Statically typed at **compile-time** (type is inferred by the compiler). | Dynamically typed at **runtime** (type is resolved during execution). |
| **Type Checking** | Type is checked at **compile-time**. | Type is checked at **runtime**. |
| **Performance** | Faster, as the type is determined at compile-time. | Slower, due to runtime type resolution. |
| **Flexibility** | Less flexible (type cannot change after initialization). | More flexible (type can change during execution). |
| **IntelliSense Support** | Fully supported (compiler knows the type). | Limited IntelliSense (compiler doesn’t know the type). |
| **Error Detection** | Errors are detected **at compile-time**. | Errors are detected **at runtime**. |
| **Usage with Reflection & COM Objects** | Not suitable (must specify type explicitly). | Ideal for working with **reflection, COM objects, and dynamic JSON/XML parsing**. |
| **When to Use?** | Use when the type is **known** but you want to avoid redundancy. | Use when working with **unknown types** or **dynamic data structures**. |

## 3. Constant

A constant is a variable whose value cannot be changed after it is initialized

**Declaring Constants**

```csharp
const double Pi = 3.14159;
const int MaxValue = 100;
```

### **readonly Fields** (Alternative to Constant)

`readonly` variables can be assigned only in constuctor

```csharp
class Example
{
    public readonly int MyNumber;

    public Example(int value)
    {
        MyNumber = value; // Can be assigned only in the constructor
    }
}
```

**Comparison:** `readonly` vs `const` in C#

| **Aspect** | **const** | **readonly** |
| --- | --- | --- |
| **Definition** | A compile-time constant whose value is fixed at compilation. | A runtime constant whose value is set at runtime but cannot change afterward. |
| **Value Assignment** | Must be assigned at declaration. | Can be assigned at declaration or inside a constructor. |
| **Modification** | Cannot be modified after compilation. | Can be modified inside a constructor but not anywhere else. |
| **Scope** | Implicitly `static`, meaning it's shared across all instances. | Can be either **instance-level** or **static**. |
| **Storage Location** | Stored in **assembly metadata** (inline replacement at compile-time). | Stored in **memory** (heap for reference types, stack for value types). |
| **Performance** | Faster, as it's replaced by literal values at compile time. | Slightly slower, as it requires memory access at runtime. |
| **Usage in Methods** | Cannot be used inside instance methods (only in static context). | Can be used inside instance or static methods. |
| **Use with Reference Types** | Only works with immutable types like `string`. | Can be used with any reference type. |
| **Supports Calculations?** | No, must be assigned a **constant literal** (e.g., `5`, `"Hello"`). | Yes, can be assigned **computed values** (e.g., `DateTime.Now`, configuration values). |
| **When to Use?** | Use for fixed, unchanging values (e.g., mathematical constants). | Use when the value depends on runtime logic (e.g., configuration settings). |