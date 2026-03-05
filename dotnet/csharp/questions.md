# C# Interview Questions

> **Source attribution:** Content adapted from [aam9063/dotnet-interview-questions](https://github.com/aam9063/dotnet-interview-questions) by Albert Alarcón Martínez. Used with attribution. See [third-party notices](../../LICENSES/third-party-notices.md).

## Table of Contents

- [Type System & Variables](#type-system-variables)
- [OOP Concepts](#oop-concepts)
- [Collections & LINQ](#collections-linq)
- [Modern C# Features](#modern-c-features)
- [Async & Threading](#async-threading)
- [Language Fundamentals](#language-fundamentals)
- [Memory & Performance](#memory-performance)
- [Reflection & Type Inspection](#reflection-type-inspection)
- [Delegates & Events](#delegates-events)
- [Tuples & Deconstruction](#tuples-deconstruction)
- [Dynamic & Reflection](#dynamic-reflection)
- [Pattern Matching](#pattern-matching)

---

## Type System & Variables

### Q: `const` vs `readonly` in C#

**A:**

Both `readonly` and `const` are used to define **immutable** values in C#, but they differ significantly in terms of **runtime behavior, assignment timing**, and **use cases**.

---

## 🔹 `const` (Constant)

### Definition:

* A **compile-time constant**.
* Must be assigned a value at the time of **declaration**.
* The value is embedded into the **compiled IL code** wherever it is used.

### Characteristics:

* Implicitly `static`.
* Can only be initialized with **primitive** or **string** values, or other constants.
* Cannot be modified at runtime.
* Changing the value requires recompiling **all dependent assemblies**.

### Example:

```csharp
public class MathConstants
{
    public const double Pi = 3.14159;
}
```

Usage:

```csharp
double circleArea = MathConstants.Pi * radius * radius;
```

---

## 🔹 `readonly`

### Definition:

* A **runtime constant**.
* Can only be assigned during:

  * Declaration, or
  * Within a constructor (either static or instance constructor).

### Characteristics:

* Can be instance-level or static.
* Useful when the value is not known until runtime.
* Changing the value **only requires recompiling the declaring class**, not its consumers.

### Example:

```csharp
public class Configuration
{
    public readonly string AppName;
    public readonly DateTime InitTime;

    public Configuration(string appName)
    {
        AppName = appName;
        InitTime = DateTime.Now;
    }
}
```

Usage:

```csharp
var config = new Configuration("MyApp");
Console.WriteLine(config.AppName); // "MyApp"
Console.WriteLine(config.InitTime); // Runtime value
```

---

## 🔸 Summary Table

| Feature        | `const`                        | `readonly`                           |
| -------------- | ------------------------------ | ------------------------------------ |
| Initialized at | Compile-time                   | Runtime (constructor or declaration) |
| Mutability     | Immutable                      | Immutable after construction         |
| Static?        | Implicitly static              | Can be static or instance            |
| Allowed types  | Primitive, string, other const | Any type                             |
| Use cases      | Mathematical constants         | Config values, runtime settings      |

---

## ✅ When to Use

* Use `const` for values that are **guaranteed not to change** and are **universally known at compile time** (e.g., `Pi`, `DaysInWeek`).
* Use `readonly` when the value is **set at runtime** or when it depends on constructor logic (e.g., `connectionString`, timestamps, GUIDs).

---

---

### Q: `ref` vs `out` in C#

**A:**

Both `ref` and `out` are used to **pass arguments by reference** in C#, allowing the method to modify the caller's variable. However, they have different requirements for initialization and usage.

---

## 🔹 `ref`

### Definition:

* Used to **pass a variable by reference**, allowing both input and output.
* The variable **must be initialized** before passing it.

### Characteristics:

* The method can read and modify the value.
* The caller must initialize the variable before calling the method.

### Example:

```csharp
public void AddTen(ref int number)
{
    number += 10;
}

int myNumber = 5;
AddTen(ref myNumber);
Console.WriteLine(myNumber); // Output: 15
```

---

## 🔹 `out`

### Definition:

* Also passes a variable by reference, but is used **only for output**.
* The variable **does not need to be initialized** before being passed.

### Characteristics:

* The method must assign a value to the `out` parameter before the method ends.
* Used commonly to return multiple values or handle parsing.

### Example:

```csharp
public bool TryParseInt(string input, out int result)
{
    return int.TryParse(input, out result);
}

if (TryParseInt("123", out int value))
{
    Console.WriteLine(value); // Output: 123
}
```

---

## 🔸 Summary Table

| Feature               | `ref`                      | `out`                           |
| --------------------- | -------------------------- | ------------------------------- |
| Initialization needed | Yes                        | No                              |
| Must assign in method | No (can, but not required) | Yes (must be assigned)          |
| Direction             | Input and output           | Output only                     |
| Common use cases      | Modify existing variables  | Return multiple values, parsing |

---

## ✅ When to Use

* Use `ref` when you need to **pass a value into a method** and have it **modified and returned**.
* Use `out` when you need a method to **return a value via parameters**, especially when returning **multiple values** or working with methods like `TryParse`.

---

---

### Q: Boxing and Unboxing in C#

**A:**

**Boxing** and **unboxing** are operations in C# that allow a value type (like `int`, `bool`, `struct`, etc.) to be treated as an **object**. They are essential for interoperability with APIs that work with types in terms of `object`.

---

## 🔹 Boxing

### Definition:

* **Boxing** is the process of **converting a value type to an `object` type**.
* The value is **wrapped inside a reference type**, and stored on the **heap**.

### Example:

```csharp
int number = 42;
object boxed = number; // Boxing
```

* `number` is a value type (stack).
* `boxed` is a reference to a new object on the heap that contains a **copy** of `number`.

---

## 🔹 Unboxing

### Definition:

* **Unboxing** is the process of **extracting the value type** from a boxed object.
* Must be **explicitly cast** to the correct value type.

### Example:

```csharp
object boxed = 42;
int number = (int)boxed; // Unboxing
```

* The boxed object must **actually contain** the value type you’re casting to, otherwise a `InvalidCastException` is thrown.

---

## 🔹 Full Example

```csharp
int original = 100;

// Boxing
object boxed = original;

// Unboxing
int unboxed = (int)boxed;

Console.WriteLine(unboxed); // Output: 100
```

---

## 🔸 Summary Table

| Concept     | Description                                                               | Memory      | Type      |
| ----------- | ------------------------------------------------------------------------- | ----------- | --------- |
| Boxing      | Value type → `object`                                                     | Heap        | Reference |
| Unboxing    | `object` → Value type                                                     | Stack       | Value     |
| Performance | Boxing/unboxing is relatively **slow** due to heap allocation and casting | ❌ Expensive |           |

---

## ✅ When to Use / Avoid

* **Avoid frequent boxing/unboxing** in performance-critical code (e.g., loops, data structures).
* Use **generics** to eliminate boxing for value types (e.g., `List<int>` instead of `List<object>`).
* Required when working with APIs expecting `object`, like `ArrayList`, older collections, or reflection.

---

## 💡 Tip

Use **`.GetType()`** to check what’s actually stored inside a boxed object before unboxing.

```csharp
object boxed = 10;
if (boxed is int)
{
    int value = (int)boxed; // Safe unboxing
}
```

---

---

### Q: What is Heap and Stack in C#

**A:**

In C#, **heap** and **stack** refer to two different areas of memory used during program execution. Understanding their roles is crucial for grasping how memory is allocated and managed for variables and objects.

---

## 🔹 Stack

### Definition:

* A **Last-In-First-Out (LIFO)** memory structure.
* Used for:

  * Storing **value types** (e.g., `int`, `bool`, `struct`)
  * **Method call frames** (parameters, local variables, return addresses)

### Characteristics:

* Very **fast** access.
* Memory is **automatically released** when a method returns.
* Size is **limited**.
* **No garbage collection** required.

### Example:

```csharp
void Add()
{
    int x = 10; // Stored on the stack
}
```

`x` is a local value type stored on the stack, and its memory is released when `Add()` finishes.

---

## 🔹 Heap

### Definition:

* A **dynamic memory area** used for storing **reference types** (e.g., `class`, `string`, `object`, arrays).
* Accessed via **pointers (references)**.

### Characteristics:

* **Slower** than stack due to memory allocation and garbage collection.
* Memory is managed by the **.NET Garbage Collector (GC)**.
* Suitable for data that needs to **persist** beyond a method's execution.

### Example:

```csharp
class Person
{
    public string Name;
}

void CreatePerson()
{
    Person p = new Person(); // `p` is on the stack, object is on the heap
    p.Name = "Alice";
}
```

* `p` is a reference stored on the **stack**.
* The actual `Person` object and its `Name` field are stored on the **heap**.

---

## 🔸 Summary Table

| Feature           | Stack                     | Heap                            |
| ----------------- | ------------------------- | ------------------------------- |
| Memory type       | Static                    | Dynamic                         |
| Stores            | Value types, method calls | Reference types, objects        |
| Lifetime          | Ends with method scope    | Controlled by garbage collector |
| Speed             | Very fast                 | Slower                          |
| Memory management | Automatic (by call stack) | GC-managed                      |
| Thread safety     | Thread-specific           | Shared across threads           |

---

## ✅ Key Insight

* **Value types** are stored on the stack **by default**, but if used in a reference type (e.g., as a field of a class), they live on the heap.
* **Reference types** store only the reference on the stack, and the actual data on the heap.
* Knowing the difference helps with **performance optimization**, especially in large applications or high-frequency operations.

---

---

### Q: How Nullable Reference Types Work in C#

**A:**

**Nullable Reference Types (NRTs)** in C# are a **compiler feature** introduced in **C# 8.0** to help prevent **`NullReferenceException`** at runtime by enabling **null-safety checks** at compile time.

They **do not change the runtime behavior**, but enhance **code quality** through **warnings** and **annotations**.

---

## 🔹 How They Work

When enabled, reference types are **non-nullable by default**. To allow `null`, you must explicitly mark them with `?`.

---

### 🔸 Non-nullable reference (default):

```csharp
string name = "Alice";  // ✅ Safe
name = null;            // ⚠️ Warning: assigning null to a non-nullable reference
```

### 🔸 Nullable reference:

```csharp
string? name = null;    // ✅ Allowed
```

You must then handle the possibility that it could be `null`.

---

## 🔹 Enabling Nullable Reference Types

In your `.csproj` file:

```xml
<Nullable>enable</Nullable>
```

Or at the top of a file:

```csharp
#nullable enable
```

---

## 🔹 Null Safety Warnings

The compiler warns you when:

| Scenario                                     | Example                                     | Result     |
| -------------------------------------------- | ------------------------------------------- | ---------- |
| Possible dereference of a nullable reference | `Console.WriteLine(name.Length);`           | ⚠️ Warning |
| Null assigned to a non-nullable variable     | `string name = null;`                       | ⚠️ Warning |
| Returning null from non-nullable method      | `return null;` (if return type is `string`) | ⚠️ Warning |

---

## 🔹 Safe Access Patterns

### 1. **Null checks**

```csharp
if (name != null)
{
    Console.WriteLine(name.Length); // ✅ Safe
}
```

### 2. **Null-forgiving operator (`!`)**

```csharp
Console.WriteLine(name!.Length); // ⚠️ Suppresses warning (you assert it's not null)
```

### 3. **Null-coalescing operator (`??`)**

```csharp
string? name = null;
string result = name ?? "Default";
```

---

## 🔸 Summary Table

| Feature           | Description                                     |
| ----------------- | ----------------------------------------------- |
| `string`          | Non-nullable reference type (warns on null use) |
| `string?`         | Nullable reference type (must check for null)   |
| `!`               | Null-forgiving operator (bypasses warning)      |
| Compiler behavior | Emits warnings, **no runtime checks**           |
| Goal              | Reduce `NullReferenceException` risks           |

---

## ✅ Benefits

* Encourages **null safety** in code.
* Helps catch bugs **at compile time**, not runtime.
* Improves **code documentation** (explicit nullability).
* Encourages better **API design**.

---

## ⚠️ Things to Remember

* This is a **compile-time** feature — doesn't affect runtime nullability.
* Must be **enabled explicitly** (in projects prior to .NET 6).
* Integrates well with **nullable value types** (`int?`, `bool?`, etc.).

---

## 🧠 Example

```csharp
public class User
{
    public string Name { get; set; } = "";     // Non-nullable, must be initialized
    public string? Email { get; set; }         // Nullable, must check before use
}
```

---

---

### Q: Difference Between `ref` and `out` Parameters in C#

**A:**

Both `ref` and `out` are used in C# to **pass arguments by reference**, allowing a method to **modify the caller's variable**. However, they have **different requirements and use cases**.

---

## 🔹 `ref` Parameter

### Definition:

Allows a variable to be **passed by reference** for **both input and output**.

### Rules:

* The variable **must be initialized** before passing.
* The method **may** modify the variable.

### Example:

```csharp
public void AddFive(ref int number)
{
    number += 5;
}

int x = 10;
AddFive(ref x);
Console.WriteLine(x); // Output: 15
```

---

## 🔹 `out` Parameter

### Definition:

Allows a variable to be **passed by reference** for **output only**.

### Rules:

* The variable **does not need to be initialized** before being passed.
* The method **must assign a value** before returning.

### Example:

```csharp
public void GetFullName(out string name)
{
    name = "Albert Alarcón";
}

string fullName;
GetFullName(out fullName);
Console.WriteLine(fullName); // Output: Albert Alarcón
```

---

## 🔸 Summary Table

| Feature                     | `ref`                    | `out`                     |
| --------------------------- | ------------------------ | ------------------------- |
| Requires initialization?    | ✅ Yes                    | ❌ No                      |
| Must be assigned in method? | ❌ No                     | ✅ Yes                     |
| Direction                   | Input + Output           | Output only               |
| Use case                    | Modify existing variable | Return additional results |

---

## ✅ When to Use

* Use `ref` when:

  * The method **uses and optionally modifies** an existing value.
  * You want **bidirectional data flow**.

* Use `out` when:

  * The method is **meant to return data** (like a second or third return value).
  * You don’t care about the **input value**, only the output.

---

## 🧠 Tip

Use `out` with methods like `TryParse`, and `ref` when you want to **update** the caller’s value **based on existing input**.

---

---

## OOP Concepts

### Q: `sealed` Keyword in C#

**A:**

The `sealed` keyword is used to **prevent inheritance**. It can be applied to **classes** and **overridden methods** to restrict further extension or modification.

---

## 🔹 Sealing a Class

### Definition:

* A `sealed` class **cannot be inherited** by any other class.

### Use Case:

* To prevent unwanted subclassing when you want to **lock the implementation**.
* Improves performance by allowing some compiler optimizations (e.g., inlining).

### Example:

```csharp
public sealed class Logger
{
    public void Log(string message)
    {
        Console.WriteLine($"[LOG] {message}");
    }
}

// This will produce a compile-time error:
// public class FileLogger : Logger { }
```

---

## 🔹 Sealing an Overridden Method

### Definition:

* A `sealed` method is used in a **derived class** to **prevent further overriding** of an already overridden method.

### Use Case:

* Useful in class hierarchies where you want to allow one level of customization, but **no more**.

### Example:

```csharp
public class Base
{
    public virtual void Display()
    {
        Console.WriteLine("Base");
    }
}

public class Intermediate : Base
{
    public sealed override void Display()
    {
        Console.WriteLine("Intermediate");
    }
}

// This will cause a compile-time error:
// public class Derived : Intermediate
// {
//     public override void Display() { }
// }
```

---

## 🔸 Summary Table

| Use                      | Effect                              |
| ------------------------ | ----------------------------------- |
| `sealed class`           | Class cannot be inherited           |
| `sealed override method` | Method cannot be further overridden |

---

## ✅ When to Use

* Use `sealed` on a class when you want to **protect it from being extended**.
* Use `sealed` on a method to **limit polymorphic behavior** to a specific level in the inheritance chain.

---

---

### Q: Access Modifiers for Types in C#

**A:**

Access modifiers control the **visibility** and **accessibility** of classes, interfaces, structs, enums, and their members. In C#, not all modifiers apply to **types**, and the set of modifiers allowed depends on whether the type is **top-level** or **nested**.

---

## 🔹 Top-Level Types (e.g., `class`, `interface`, `struct`, `enum`, `record`)

These types can only have the following access modifiers:

### 1. `public`

* Accessible from **anywhere**.
* Use for types intended to be part of the public API.

```csharp
public class MyClass { }
```

### 2. `internal` (default)

* Accessible **only within the same assembly**.
* This is the **default** access modifier if none is specified.

```csharp
internal class MyInternalClass { }

class DefaultInternalClass { } // Also internal by default
```

---

## 🔹 Nested Types (types declared within another type)

Nested types can use **all five** access modifiers:

### 1. `public`

* Accessible from anywhere, provided the containing type is also accessible.

### 2. `private`

* Accessible **only within the containing type**.

### 3. `protected`

* Accessible within the containing type **and** derived types.

### 4. `internal`

* Accessible within the **same assembly**.

### 5. `protected internal`

* Accessible from:

  * The **same assembly**, or
  * Any derived type (even in another assembly).

### 6. `private protected`

* Accessible from:

  * The **containing class**, or
  * A **derived class** in the **same assembly**.

### Example:

```csharp
public class Outer
{
    private class PrivateNested { }
    protected class ProtectedNested { }
    internal class InternalNested { }
    protected internal class ProtectedInternalNested { }
    private protected class PrivateProtectedNested { }
    public class PublicNested { }
}
```

---

## 🔸 Summary Table

| Modifier             | Top-Level Types | Nested Types | Description                                                        |
| -------------------- | --------------- | ------------ | ------------------------------------------------------------------ |
| `public`             | ✅               | ✅            | Accessible from anywhere                                           |
| `internal`           | ✅               | ✅            | Accessible within the same assembly                                |
| `private`            | ❌               | ✅            | Accessible only within the containing type                         |
| `protected`          | ❌               | ✅            | Accessible within the containing and derived types                 |
| `protected internal` | ❌               | ✅            | Accessible in same assembly or derived types from other assemblies |
| `private protected`  | ❌               | ✅            | Accessible in same assembly and only in derived types              |

---

## ✅ Notes

* **Top-level types** can **only** be `public` or `internal`.
* Use **nesting + access modifiers** when you need more granular control over type exposure within a class or module.

---

---

### Q: `interface` vs `abstract class` in C#

**A:**

Both `interface` and `abstract class` are used to define **contracts** that other classes must implement, but they differ in capabilities, flexibility, and use cases.

---

## 🔹 `interface`

### Definition:

* A pure contract that **declares** members (methods, properties, events, indexers) without providing implementation (until C# 8+).
* A class can implement **multiple interfaces**.

### Characteristics:

* Cannot have fields or constructors.
* All members are **public** by default.
* Can have **default implementations** (C# 8.0+), but usage is limited.
* Best suited for defining **capabilities** or **behaviors**.

### Example:

```csharp
public interface IAnimal
{
    void Speak();
}
```

```csharp
public class Dog : IAnimal
{
    public void Speak()
    {
        Console.WriteLine("Woof!");
    }
}
```

---

## 🔹 `abstract class`

### Definition:

* A class that **cannot be instantiated** and may contain **abstract** (unimplemented) and/or **concrete** (implemented) members.
* A class can **only inherit from one abstract class**.

### Characteristics:

* Can contain fields, constructors, methods (both abstract and non-abstract), and access modifiers.
* Allows partial implementation and shared code across child classes.
* Ideal for **inheritance hierarchies** or **base classes**.

### Example:

```csharp
public abstract class Animal
{
    public abstract void Speak();
    
    public void Eat()
    {
        Console.WriteLine("Eating...");
    }
}

public class Cat : Animal
{
    public override void Speak()
    {
        Console.WriteLine("Meow!");
    }
}
```

---

## 🔸 Summary Table

| Feature          | `interface`                               | `abstract class`                     |
| ---------------- | ----------------------------------------- | ------------------------------------ |
| Inheritance      | Multiple allowed                          | Only one (single inheritance)        |
| Constructors     | ❌ Not allowed                             | ✅ Allowed                            |
| Fields           | ❌ Not allowed                             | ✅ Allowed                            |
| Access modifiers | All members are `public`                  | Supports `public`, `protected`, etc. |
| Implementation   | No (until C# 8 default methods)           | Can include full or partial logic    |
| When to use      | Define capabilities (e.g., `IDisposable`) | Define shared base behavior          |

---

## ✅ When to Use

* Use an **interface** when:

  * You need to support **multiple inheritance** of behaviors.
  * You're defining **capabilities** without implementation.

* Use an **abstract class** when:

  * You want to provide **base functionality** with the option to override.
  * You need fields, constructors, or default logic.

---

---

### Q: Inheritance vs Composition in C#

**A:**

**Inheritance** and **composition** are two fundamental object-oriented design principles used to achieve **code reuse** and **modularity**, but they differ significantly in approach and flexibility.

---

## 🔹 Inheritance

### Definition:

* A mechanism where a class (child) **inherits** members (methods, fields, properties) from another class (parent).

### Characteristics:

* Represents an **"is-a"** relationship.
* Promotes **code reuse** by sharing common functionality via base classes.
* Supports **method overriding** and **polymorphism**.

### Example:

```csharp
public class Animal
{
    public void Eat() => Console.WriteLine("Eating...");
}

public class Dog : Animal
{
    public void Bark() => Console.WriteLine("Barking...");
}
```

Usage:

```csharp
var dog = new Dog();
dog.Eat();  // Inherited from Animal
dog.Bark(); // Defined in Dog
```

---

## 🔹 Composition

### Definition:

* A design technique where a class is **composed** of one or more **objects of other classes**, delegating responsibilities to them.

### Characteristics:

* Represents a **"has-a"** relationship.
* Promotes **flexibility** and **reusability**.
* Preferred over inheritance in **modern design** (favor composition over inheritance).
* Easier to test, extend, and refactor.

### Example:

```csharp
public class Engine
{
    public void Start() => Console.WriteLine("Engine started");
}

public class Car
{
    private readonly Engine _engine = new Engine();

    public void StartCar()
    {
        _engine.Start(); // Delegation via composition
    }
}
```

Usage:

```csharp
var car = new Car();
car.StartCar(); // "Engine started"
```

---

## 🔸 Summary Table

| Feature          | Inheritance                    | Composition                       |
| ---------------- | ------------------------------ | --------------------------------- |
| Relationship     | "Is-a"                         | "Has-a"                           |
| Coupling         | Tightly coupled                | Loosely coupled                   |
| Reusability      | Through base class             | Through contained components      |
| Flexibility      | Less (fixed hierarchy)         | More (components can vary)        |
| Testability      | Harder (tied to hierarchy)     | Easier (components can be mocked) |
| Runtime behavior | Static (fixed at compile-time) | Dynamic (can change at runtime)   |
| Example          | `Dog : Animal`                 | `Car has an Engine`               |

---

## ✅ When to Use

* Use **inheritance** when:

  * There is a clear **hierarchical relationship**.
  * You want to use **polymorphism** or **override virtual methods**.
  * The parent class is **stable and well-designed**.

* Use **composition** when:

  * You want **flexibility**, **encapsulation**, and **testability**.
  * Behavior is better modeled by **delegating to other classes**.
  * You want to **favor composition over inheritance** (recommended by SOLID principles).

---

## 🧠 Design Principle

> **“Favor composition over inheritance”** — Effective Java, Design Patterns (GoF)

Because composition offers more control, better decoupling, and extensibility.

---

---

### Q: How Is Polymorphism Implemented in C#?

**A:**

**Polymorphism** in C# is implemented through **inheritance**, **interfaces**, and **virtual/override** or **abstract** methods. It allows you to use a **single interface** to represent **different types** of behavior.

C# supports two main types of polymorphism:

---

## 🔹 1. **Compile-Time Polymorphism** (a.k.a. *Static Binding*)

### Achieved via:

* **Method Overloading**
* **Operator Overloading**

### Example: Method Overloading

```csharp
public class Printer
{
    public void Print(string text) => Console.WriteLine(text);
    public void Print(int number) => Console.WriteLine(number);
}
```

*Same method name, different parameters.*

---

## 🔹 2. **Run-Time Polymorphism** (a.k.a. *Dynamic Binding*)

### Achieved via:

* **Inheritance + virtual/override**
* **Abstract classes**
* **Interfaces**

### 🔸 Virtual Methods

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal speaks");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof!");
}
```

### Usage:

```csharp
Animal pet = new Dog();
pet.Speak(); // Output: Woof!
```

*The method executed is based on the **actual type** (`Dog`), not the **declared type** (`Animal`).*

---

### 🔸 Abstract Classes

```csharp
public abstract class Shape
{
    public abstract double GetArea();
}

public class Circle : Shape
{
    public override double GetArea() => Math.PI * radius * radius;
    private double radius = 3;
}
```

*Force derived classes to provide specific implementations.*

---

### 🔸 Interfaces

```csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}
```

### Usage:

```csharp
ILogger logger = new ConsoleLogger();
logger.Log("Hello"); // Output: Hello
```

*You can treat different implementations uniformly via the interface.*

---

## 🔸 Summary Table

| Type               | Method                                 | Binding      |
| ------------------ | -------------------------------------- | ------------ |
| Method Overloading | Same method name, different parameters | Compile-time |
| Virtual/Override   | Base and derived class method chaining | Run-time     |
| Abstract Methods   | Must be overridden                     | Run-time     |
| Interfaces         | Method defined via contract            | Run-time     |

---

## ✅ When to Use

* Use **method overloading** for behavior variations based on input.
* Use **virtual/abstract methods** to allow **custom behavior** in derived classes.
* Use **interfaces** to allow **unrelated classes** to share a **common contract**.

---

## 🧠 Tip

Polymorphism is central to the **Open/Closed Principle**: open for extension, closed for modification. It enables flexible and scalable design.

---

---

### Q: How Is Encapsulation Implemented in C#

**A:**

**Encapsulation** in C# is the object-oriented principle of **hiding internal state and implementation details**, exposing only what is necessary through **controlled access** (typically via properties and methods).

It’s implemented using:

* **Access modifiers**
* **Properties (get/set)**
* **Private fields**
* **Interfaces or method contracts**

---

## 🔹 Core Concepts

| Mechanism                   | Role in Encapsulation                           |
| --------------------------- | ----------------------------------------------- |
| `private` members           | Hide internal state and behavior                |
| `public`/`internal` methods | Expose a controlled API surface                 |
| Properties                  | Control access with logic in `get`/`set`        |
| Access modifiers            | Define visibility across classes and assemblies |

---

## 🔹 Example: Basic Encapsulation

```csharp
public class BankAccount
{
    private decimal _balance;

    public decimal Balance
    {
        get => _balance;
        private set
        {
            if (value < 0)
                throw new InvalidOperationException("Balance cannot be negative");
            _balance = value;
        }
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");

        Balance += amount;
    }

    public void Withdraw(decimal amount)
    {
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds");

        Balance -= amount;
    }
}
```

### Key Points:

* `_balance` is **private** and not accessible outside.
* `Balance` has a **private setter** — only internal methods can change it.
* Access to data is controlled through **public methods** like `Deposit` and `Withdraw`.

---

## 🔹 Access Modifiers Recap

| Modifier             | Visibility                          |
| -------------------- | ----------------------------------- |
| `private`            | Inside the same class only          |
| `protected`          | Same class + derived classes        |
| `internal`           | Within the same assembly            |
| `protected internal` | In derived classes OR same assembly |
| `private protected`  | Derived classes AND same assembly   |
| `public`             | Accessible from anywhere            |

---

## 🔹 Benefits of Encapsulation

* Protects **object integrity** by preventing invalid states.
* Hides complex logic and **internal implementation**.
* Makes code easier to **maintain and refactor**.
* Enforces **clear contracts** and **separation of concerns**.

---

## ✅ When to Use

* Always **hide internal fields** using `private` or `protected`.
* Expose data via **properties** — not public fields.
* Use **accessors with validation logic** to guard sensitive data.
* Favor **narrow interfaces** to expose only what's necessary.

---

## 🧠 Tip

Encapsulation enables **loose coupling**: other classes interact with what you expose, not how you do it.

---

---

### Q: What is covariance and contravariance?

**A:**

These allow **flexibility in generic type assignments**.

### Covariance (`out`) — for **return types**

```csharp
IEnumerable<string> strs = new List<string>();
IEnumerable<object> objs = strs; // ✅ OK
```

### Contravariance (`in`) — for **parameter types**

```csharp
Action<object> a1 = o => Console.WriteLine(o);
Action<string> a2 = a1; // ✅ OK
```

---

---

## Collections & LINQ

### Q: Difference Between `HashSet` and `Dictionary` in C#

**A:**

Both `HashSet<T>` and `Dictionary<TKey, TValue>` are **hash-based collections** in C#, but they serve different purposes and structures.

---

## 🔹 `HashSet<T>`

### Definition:

* A collection of **unique values**.
* No duplicate elements allowed.
* Stores only **keys** (no values).
* Ideal for membership tests (i.e., *Does this item exist?*).

### Characteristics:

* Fast lookup, add, and remove (O(1) average time).
* No key-value pair — only single values.
* Backed by a hash table internally.

### Example:

```csharp
HashSet<string> fruits = new HashSet<string>();
fruits.Add("Apple");
fruits.Add("Banana");
fruits.Add("Apple"); // Ignored — duplicates not allowed

Console.WriteLine(fruits.Contains("Apple")); // True
```

---

## 🔹 `Dictionary<TKey, TValue>`

### Definition:

* A collection of **key-value pairs**.
* Each key is **unique**, and each key maps to **a single value**.
* Ideal when you need to **associate data** with a unique identifier.

### Characteristics:

* Fast lookup, add, and remove by key.
* Keys must be unique, but values can be duplicated.
* Also backed by a hash table internally.

### Example:

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["Alice"] = 30;
ages["Bob"] = 25;

// Access value by key
Console.WriteLine(ages["Alice"]); // 30
```

---

## 🔸 Summary Table

| Feature        | `HashSet<T>`              | `Dictionary<TKey, TValue>`            |
| -------------- | ------------------------- | ------------------------------------- |
| Stores         | Only unique values        | Key-value pairs                       |
| Key uniqueness | All values must be unique | Keys must be unique                   |
| Value access   | Not applicable            | Values accessed via keys              |
| Lookup by      | Value                     | Key                                   |
| Use case       | Fast membership test      | Fast key-based lookup and association |

---

## ✅ When to Use

* Use `HashSet<T>` when:

  * You only need to **store unique items**.
  * You need to perform **fast existence checks**.

* Use `Dictionary<TKey, TValue>` when:

  * You need to **map keys to values** (e.g., username → email).
  * You want to **retrieve values based on a unique identifier**.

---

---

### Q: What Is the Purpose of the Method `ToLookup` in C#

**A:**

The `ToLookup` method in C# is a **LINQ extension method** that creates a **one-to-many lookup (multimap)** from a collection. It groups elements by a **key** and stores them in a `Lookup<TKey, TElement>` — a structure similar to a `Dictionary<TKey, List<T>>`, but **read-only** and **lazy-evaluated**.

---

## 🔹 Namespace & Signature

* Defined in `System.Linq`.
* Signature:

```csharp
ILookup<TKey, TElement> ToLookup<TSource, TKey>(
    this IEnumerable<TSource> source,
    Func<TSource, TKey> keySelector
)
```

There are also overloads to:

* Specify an element selector
* Provide a custom key comparer

---

## 🔹 Example: Group People by City

```csharp
public class Person
{
    public string Name { get; set; }
    public string City { get; set; }
}

var people = new List<Person>
{
    new Person { Name = "Alice", City = "Madrid" },
    new Person { Name = "Bob", City = "Madrid" },
    new Person { Name = "Charlie", City = "Berlin" }
};

var lookup = people.ToLookup(p => p.City);

// Access grouped elements
foreach (var group in lookup)
{
    Console.WriteLine(group.Key); // City
    foreach (var person in group)
    {
        Console.WriteLine($" - {person.Name}");
    }
}
```

**Output:**

```
Madrid
 - Alice
 - Bob
Berlin
 - Charlie
```

---

## 🔹 Key Characteristics of `ToLookup`

| Feature            | Description                                                   |
| ------------------ | ------------------------------------------------------------- |
| Return type        | `ILookup<TKey, TElement>`                                     |
| Grouping behavior  | One key maps to multiple elements                             |
| Lookup by key      | Similar to `Dictionary[key]`, but key may have multiple items |
| Read-only          | Cannot add/remove after creation                              |
| Deferred execution | No — it's **immediate** (unlike `GroupBy`)                    |

---

## 🔸 Difference from `GroupBy`

| Feature     | `ToLookup()`              | `GroupBy()`                       |
| ----------- | ------------------------- | --------------------------------- |
| Return type | `ILookup<TKey, TElement>` | `IEnumerable<IGrouping<TKey, T>>` |
| Execution   | **Immediate**             | **Deferred**                      |
| Mutability  | Read-only                 | Enumerable                        |

---

## ✅ When to Use

* When you need to **group items by key** and access them **efficiently by key** (e.g., like a dictionary).
* When you want an **immediate, in-memory lookup structure**.
* For **read-only scenarios** where fast access to grouped data is needed.

---

---

### Q: Does LINQ `Cast<T>` Method Create a New Object?

**A:**

No, the LINQ `Cast<T>()` method **does not create new objects**. It simply **casts each element** of a non-generic `IEnumerable` (like `ArrayList`) to the specified type `T` at **runtime**.

---

## 🔹 Purpose of `Cast<T>()`

* To convert an `IEnumerable` (non-generic) into `IEnumerable<T>`, enabling **LINQ queries**.
* It **does not modify or copy** the original objects — it just **casts the references**.

---

## 🔹 How It Works

```csharp
ArrayList list = new ArrayList { 1, 2, 3 };

IEnumerable<int> numbers = list.Cast<int>();

foreach (var n in numbers)
{
    Console.WriteLine(n); // Outputs: 1, 2, 3
}
```

* Each element in `list` is **cast to `int`**.
* If an element is not of type `int`, it throws an **`InvalidCastException`** at runtime.
* The elements themselves are **not cloned or recreated** — only their references are cast.

---

## 🔹 Comparison with `Select`

```csharp
var result = list.Select(x => (int)x); // Similar in behavior, but allows transformation
```

* `Select` gives you control to transform elements, while `Cast<T>()` only casts.

---

## 🔸 Summary Table

| Feature             | `Cast<T>()`                                           |
| ------------------- | ----------------------------------------------------- |
| Creates new object? | ❌ No                                                  |
| Type safety         | ✅ Checked at runtime (not compile-time)               |
| Purpose             | Convert non-generic `IEnumerable` to `IEnumerable<T>` |
| Performance         | Very fast — just a cast, no allocation                |
| Exceptions          | Throws if an element can't be cast                    |

---

## ✅ When to Use

* When working with legacy or non-generic collections like `ArrayList`.
* When you know all items in the collection are of the same type and want to apply LINQ.
* To enable **LINQ methods** like `Where`, `Select`, `Sum` on non-generic sources.

---

---

### Q: Explain Deferred Execution in LINQ

**A:**

**Deferred execution** in LINQ means that a query is **not executed at the time it is defined**, but **only when the query variable is iterated**, e.g., via `foreach`, `ToList()`, or similar.

---

## 🔹 Key Concept

* The **query definition** does **not retrieve data immediately**.
* Execution is triggered **only when you actually enumerate the results**.
* Applies to most **LINQ methods** returning `IEnumerable<T>` or `IQueryable<T>`.

---

## 🔹 Example

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

var query = numbers.Where(n => n > 2); // No execution here

numbers.Add(6); // Modifies the source before execution

foreach (var num in query) // Execution happens now
{
    Console.WriteLine(num); // Output: 3, 4, 5, 6
}
```

* The query is **not executed** when it's declared.
* It's **re-evaluated** when you iterate it — this is **deferred execution** in action.

---

## 🔹 How to Force Immediate Execution

To get a **snapshot** of the result at the time of query definition, use **immediate execution methods** like:

* `.ToList()`
* `.ToArray()`
* `.Count()`
* `.First()`, `.Any()`, etc.

```csharp
var result = numbers.Where(n => n > 2).ToList(); // Executed immediately
```

---

## 🔹 Methods with Deferred vs Immediate Execution

| Execution Type | Examples                                     |
| -------------- | -------------------------------------------- |
| **Deferred**   | `Where`, `Select`, `OrderBy`, `Take`, etc.   |
| **Immediate**  | `ToList`, `ToArray`, `Count`, `First`, `Sum` |

---

## ✅ Benefits of Deferred Execution

* **Performance**: Avoids unnecessary computation until needed.
* **Fresh data**: Always works with the most up-to-date state of the source.
* **Composability**: Allows you to build queries dynamically before running them.

---

## ⚠️ Caution

* If the underlying data source **changes after query definition**, it **affects the result** when executed.
* Deferred execution can cause **unexpected behavior** if not understood properly.

---

## 🧠 Tip

If you need a **fixed result** that won't change even if the data source changes, use `ToList()` or `ToArray()` **immediately after the query**.

---

---

### Q: How Does the `ImmutableList` Work in C#

**A:**

`ImmutableList<T>` is a collection in the `System.Collections.Immutable` namespace that **cannot be modified** after it is created. Instead of changing the original list, all operations like `Add`, `Remove`, or `Insert` return a **new list with the change applied**.

---

## 🔹 Key Characteristics

* **Immutable**: Once created, its contents **never change**.
* **Thread-safe**: Perfect for **concurrent** or **functional** programming.
* **Structural sharing**: Internally optimized to reuse unchanged data, minimizing memory and performance cost.
* **Efficient**: Although it creates new instances, it does **not copy the entire list** each time thanks to internal trees.

---

## 🔹 How to Use

First, install the required NuGet package:

```bash
dotnet add package System.Collections.Immutable
```

Then import the namespace:

```csharp
using System.Collections.Immutable;
```

---

## 🔹 Example: Basic Usage

```csharp
var list = ImmutableList.Create<string>();

var list1 = list.Add("A");
var list2 = list1.Add("B");

Console.WriteLine(string.Join(", ", list));   // Output: 
Console.WriteLine(string.Join(", ", list1));  // Output: A
Console.WriteLine(string.Join(", ", list2));  // Output: A, B
```

* `list` remains empty.
* `list1` contains "A".
* `list2` contains "A, B".

Each `Add` returns a **new list**, and the original stays untouched.

---

## 🔹 Example: Remove & Update

```csharp
var names = ImmutableList.Create("Alice", "Bob", "Charlie");

var removed = names.Remove("Bob");    // New list without "Bob"
var replaced = names.SetItem(1, "Bobby"); // Replace index 1 (Bob → Bobby)

Console.WriteLine(string.Join(", ", removed));  // Output: Alice, Charlie
Console.WriteLine(string.Join(", ", replaced)); // Output: Alice, Bobby, Charlie
```

---

## 🔸 Summary Table

| Feature              | `ImmutableList<T>`                               |
| -------------------- | ------------------------------------------------ |
| Mutability           | ❌ No (read-only, safe by design)                 |
| Thread safety        | ✅ Yes                                            |
| Modification methods | Return **new instances** with the changes        |
| Performance          | Efficient with **structural sharing**            |
| Use cases            | Functional code, concurrent access, undo history |

---

## ✅ When to Use

* When you want to **ensure immutability** and **avoid bugs** caused by shared mutable state.
* In **multi-threaded** applications to eliminate synchronization complexity.
* In **functional-style programming**, where data is never changed in place.
* When building **undo/redo stacks** — each state is a separate, safe version.

---

---

### Q: What Are the Benefits of Using Frozen Collections in C#

**A:**

**Frozen collections** are a feature introduced in **.NET 8** via the `System.Collections.Frozen` namespace. They are **immutable, read-optimized collections** (e.g., `FrozenSet<T>`, `FrozenDictionary<TKey, TValue>`) designed for **maximum lookup performance** when the data **does not change** after creation.

---

## 🔹 Key Characteristics

* **Immutable**: Cannot be modified after creation.
* **Optimized for reads**: Faster than regular `Dictionary`, `HashSet`, etc., especially in **high-read, low-write** scenarios.
* **Thread-safe**: Safe to use across multiple threads without locks.
* **Built once**: Frozen collections are built once using a `ToFrozen()` method and then used efficiently.

---

## 🔹 How to Use

```csharp
using System.Collections.Frozen;

var words = new[] { "dog", "cat", "bird", "fish" };
var frozenSet = words.ToFrozenSet();

Console.WriteLine(frozenSet.Contains("cat")); // True
```

```csharp
var dict = new Dictionary<string, int>
{
    ["A"] = 1,
    ["B"] = 2
};

var frozenDict = dict.ToFrozenDictionary();
Console.WriteLine(frozenDict["A"]); // Output: 1
```

---

## 🔹 Benefits

### ✅ 1. **Performance**

* Lookup operations (`Contains`, indexers) are **faster** than in regular `HashSet` or `Dictionary`.
* Internally uses layout strategies best suited for the actual data (e.g., dense vs sparse keys).

### ✅ 2. **Thread Safety Without Locks**

* No need to lock or synchronize — safe to use **concurrently** from multiple threads.

### ✅ 3. **Memory Efficiency**

* Optimized internal structure can reduce memory overhead depending on the data pattern.

### ✅ 4. **Immutable Safety**

* Once created, cannot be changed — avoids accidental modifications.
* Encourages **safe, functional-style design**.

---

## 🔸 Summary Table

| Feature                  | `FrozenSet` / `FrozenDictionary` | `HashSet` / `Dictionary`      |
| ------------------------ | -------------------------------- | ----------------------------- |
| Mutability               | ❌ Immutable                      | ✅ Mutable                     |
| Thread-safe              | ✅ Yes                            | ❌ No (unless locked manually) |
| Optimized for reads      | ✅ Yes                            | ⚠️ Moderate                   |
| Build time customization | ✅ Uses adaptive internal layout  | ❌ Standard structure          |
| When to use              | Static, high-read scenarios      | Dynamic, frequent updates     |

---

## ✅ When to Use

* When you have a **fixed dataset** that is read **frequently**, such as:

  * Configuration keys
  * Command or token lists
  * String/ID sets used for matching
* In **multi-threaded environments** where read performance and thread safety are critical.
* When you want **maximum performance** without sacrificing safety.

---

## 🔹 Requirements

* Available in **.NET 8** and later.
* Namespace: `System.Collections.Frozen`.
* Package: Included by default in the SDK — no extra install needed.

---

---

### Q: Thread-Safe Collections in C#

**A:**

Thread-safe collections in C# are designed to allow **multiple threads** to access or modify data **safely without locking manually**. These collections are found in the **`System.Collections.Concurrent`** namespace.

---

## 🔹 Built-in Thread-Safe Collections

| Collection                           | Description                                                                                                                     |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `ConcurrentDictionary<TKey, TValue>` | Thread-safe key-value store. Allows atomic operations like `TryAdd`, `TryUpdate`, etc.                                          |
| `ConcurrentQueue<T>`                 | FIFO (First-In-First-Out) thread-safe queue.                                                                                    |
| `ConcurrentStack<T>`                 | LIFO (Last-In-First-Out) thread-safe stack.                                                                                     |
| `ConcurrentBag<T>`                   | Unordered thread-safe collection optimized for scenarios where **multiple threads produce and consume** items.                  |
| `BlockingCollection<T>`              | Thread-safe wrapper over any `IProducerConsumerCollection<T>` (usually a `ConcurrentQueue<T>`). Supports blocking and bounding. |

---

## 🔹 Examples

### `ConcurrentDictionary`

```csharp
using System.Collections.Concurrent;

var dict = new ConcurrentDictionary<string, int>();
dict.TryAdd("one", 1);
dict["two"] = 2;

if (dict.TryGetValue("one", out int value))
{
    Console.WriteLine(value); // Output: 1
}
```

---

### `ConcurrentQueue`

```csharp
var queue = new ConcurrentQueue<int>();
queue.Enqueue(10);

if (queue.TryDequeue(out int result))
{
    Console.WriteLine(result); // Output: 10
}
```

---

### `BlockingCollection`

```csharp
var bc = new BlockingCollection<int>(boundedCapacity: 100);

Task.Run(() =>
{
    for (int i = 0; i < 5; i++) bc.Add(i);
    bc.CompleteAdding();
});

foreach (var item in bc.GetConsumingEnumerable())
{
    Console.WriteLine(item);
}
```

---

## 🔸 Summary Table

| Collection             | Type      | Characteristics                                  |
| ---------------------- | --------- | ------------------------------------------------ |
| `ConcurrentDictionary` | Key-Value | Fast, lock-free reads & writes                   |
| `ConcurrentQueue`      | FIFO      | Lock-free, safe for multiple producers/consumers |
| `ConcurrentStack`      | LIFO      | Thread-safe stack                                |
| `ConcurrentBag`        | Bag       | Fast unordered insert/remove                     |
| `BlockingCollection`   | Wrapper   | Blocking, bounding, producer-consumer patterns   |

---

## ✅ When to Use

* In **multi-threaded applications** that share collections.
* When building **producers and consumers** (e.g., task queues).
* To **avoid manual locks** and synchronization overhead.

---

## 🔹 Other Approaches to Thread Safety

* Use `ImmutableList`, `FrozenSet`, etc. for **read-only thread safety**.
* Lock manually (`lock { }`) around access to non-thread-safe collections like `List<T>`, if needed.

---

---

### Q: Difference Between `IEnumerable` and `IQueryable` in C#

**A:**

Both `IEnumerable` and `IQueryable` are used for **iterating over collections** in C#, but they differ in how and **where the query logic is executed** — in memory or in the data source (like a database).

---

## 🔹 `IEnumerable`

### Definition:

* Defined in `System.Collections` / `System.Collections.Generic`.
* Represents an **in-memory** collection that can be **enumerated using a `foreach` loop**.

### Characteristics:

* Query logic is **executed in memory** (client-side).
* Suitable for **LINQ to Objects**.
* Supports **deferred execution**.
* Ideal for working with **in-memory collections** (e.g., arrays, lists).

### Example:

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4 };
IEnumerable<int> evens = numbers.Where(n => n % 2 == 0);

foreach (var n in evens)
    Console.WriteLine(n); // 2, 4
```

---

## 🔹 `IQueryable`

### Definition:

* Defined in `System.Linq`.
* Designed for **querying remote data sources** (e.g., databases).
* Used by ORMs like **Entity Framework**.

### Characteristics:

* Translates the LINQ query into **SQL or another provider-specific language**.
* Executes the query **on the server** (e.g., in the database).
* Supports **deferred execution**.
* Great for **large datasets** and optimized performance.

### Example:

```csharp
IQueryable<User> users = dbContext.Users.Where(u => u.IsActive);
```

*The `Where` clause is translated into SQL and executed in the database.*

---

## 🔸 Summary Table

| Feature           | `IEnumerable`                     | `IQueryable`                           |
| ----------------- | --------------------------------- | -------------------------------------- |
| Namespace         | `System.Collections` / `.Generic` | `System.Linq`                          |
| Execution         | In-memory (client-side)           | Remote source (e.g., database)         |
| Query translation | ❌ No (uses delegates)             | ✅ Yes (uses expression trees)          |
| Use case          | LINQ to Objects                   | LINQ to SQL / EF Core / remote APIs    |
| Performance       | Less efficient on large data sets | More efficient for querying large data |

---

## ✅ When to Use

* Use **`IEnumerable`** when:

  * You are working with **in-memory** data (arrays, lists).
  * You want simple iteration or filtering locally.

* Use **`IQueryable`** when:

  * You are querying a **database or remote source**.
  * You want **server-side filtering**, sorting, or pagination for performance.

---

## 🧠 Tip

If you call `.ToList()`, `.ToArray()` or iterate an `IQueryable`, it becomes an `IEnumerable` — and the **query is executed immediately**.

---

---

### Q: What Are Expression Trees in LINQ?

**A:**

**Expression trees** in LINQ are **data structures** that represent **code as a tree of expressions**, rather than executing the code directly. They allow you to **inspect, modify, or translate** code logic at runtime — which is especially powerful in scenarios like **LINQ to SQL**, **Entity Framework**, and **dynamic query generation**.

---

## 🔹 Definition

An **expression tree** is an object of type `System.Linq.Expressions.Expression<TDelegate>` that represents code in a **tree-like form**.

Each node in the tree is an **expression** (e.g., method call, property access, binary operation, etc.).

---

## 🔹 Basic Example

```csharp
Expression<Func<int, int>> square = x => x * x;
```

This does **not** compile into IL like a normal `Func<int, int>`.
Instead, it creates a tree structure like:

```
Lambda
 └── Multiply
     ├── Parameter(x)
     └── Parameter(x)
```

You can inspect it at runtime:

```csharp
Console.WriteLine(square); // Output: x => (x * x)
```

---

## 🔹 Why Expression Trees Matter in LINQ

### ✅ **IQueryable** uses expression trees

When you write:

```csharp
var users = dbContext.Users.Where(u => u.Age > 18);
```

The lambda expression is converted to an **expression tree**, not compiled code.

This allows the LINQ provider (e.g., Entity Framework) to:

* **Parse** the expression
* **Translate it to SQL**
* **Execute it on the database**

---

## 🔹 Expression vs Delegate

| Feature           | `Func<T>` / `Action<T>` | `Expression<Func<T>>`                  |
| ----------------- | ----------------------- | -------------------------------------- |
| Behavior          | Compiled method         | Data structure representing code       |
| Execution         | Immediate               | Must be compiled or interpreted        |
| Can be translated | ❌ No                    | ✅ Yes (e.g., to SQL, JavaScript, etc.) |
| Used in           | LINQ to Objects         | LINQ to Entities, LINQ providers       |

---

## 🔹 Compile an Expression Tree

```csharp
var expr = Expression.Lambda<Func<int, int>>(
    Expression.Multiply(Expression.Parameter(typeof(int), "x"), Expression.Constant(2)),
    Expression.Parameter(typeof(int), "x")
);

Func<int, int> func = expr.Compile();
Console.WriteLine(func(10)); // Output: 20
```

---

## ✅ When to Use

* When building **custom LINQ providers**.
* When working with **Entity Framework**, **Dapper**, or **ORMs** that translate queries.
* When you need **runtime query generation** or **dynamic filtering**.
* For building **rules engines**, **search engines**, or **query builders**.

---

## 🧠 Tip

Expression trees make LINQ more than just a query language — they make it **translatable**, **analyzable**, and **runtime-aware**.

---

---

### Q: What is the difference between `Select()` and `SelectMany()`?

**A:**

### 🔹 `Select()`

Projects each item into a new form — 1:1 transformation.

```csharp
var lengths = words.Select(w => w.Length);
```

---

### 🔹 `SelectMany()`

**Flattens** collections — 1:N projection.

```csharp
var allChars = words.SelectMany(w => w.ToCharArray());
```

✅ Use `SelectMany()` when projecting to **sequences**, not single values.

---

---

## Modern C# Features

### Q: Difference Between `class`, `record`, and `struct` in C#

**A:**

In C#, `class`, `record`, and `struct` are all **types** used to define objects, but they differ in **behavior**, **memory allocation**, **immutability**, and **semantics**.

---

## 🔹 `class`

### Definition:

* A **reference type** stored on the **heap**.
* Supports **inheritance**, **mutable** by default.

### Characteristics:

* Passed by **reference**.
* Can inherit from other classes.
* Good for **complex objects**, services, and entities.

### Example:

```csharp
public class Person
{
    public string Name { get; set; }
}
```

---

## 🔹 `record` (Introduced in C# 9)

### Definition:

* A **reference type** like `class`, but focused on **immutable data** and **value-based equality**.

### Characteristics:

* Stores data with minimal boilerplate.
* Overrides `Equals()`, `GetHashCode()`, and `ToString()` automatically.
* Supports **non-destructive mutation** via `with` expression.
* Ideal for **data models**, **DTOs**, and **functional programming**.

### Example:

```csharp
public record Person(string Name);
```

Usage:

```csharp
var p1 = new Person("Alice");
var p2 = p1 with { Name = "Bob" }; // Creates a copy
```

---

## 🔹 `struct`

### Definition:

* A **value type** stored on the **stack**.
* More lightweight and efficient for small data types.

### Characteristics:

* Passed by **value** (copy).
* Cannot inherit from another struct or class.
* Best for **small, short-lived data** like points, coordinates, etc.
* Can be **mutable** or **immutable**.

### Example:

```csharp
public struct Point
{
    public int X;
    public int Y;
}
```

---

## 🔸 Summary Table

| Feature           | `class`                 | `record`                    | `struct`                        |
| ----------------- | ----------------------- | --------------------------- | ------------------------------- |
| Type              | Reference type          | Reference type              | Value type                      |
| Memory location   | Heap                    | Heap                        | Stack (or inline in heap)       |
| Equality behavior | Reference equality      | Value (by content) equality | Value equality                  |
| Inheritance       | Yes                     | Yes (`record class`)        | No                              |
| Immutability      | Mutable by default      | Immutable by default        | Mutable or immutable            |
| Use case          | Complex logic, services | Data models, DTOs           | Small, lightweight data         |
| `with` expression | ❌ No                    | ✅ Yes                       | ✅ With `record struct` (C# 10+) |

---

## ✅ When to Use

* Use `class` when:

  * You need **inheritance**, polymorphism, or shared references.
  * You model **entities** with behavior and identity.

* Use `record` when:

  * You need **value-based equality** and **immutable data containers**.
  * You work with **DTOs**, **configuration**, or **serialization**.

* Use `struct` when:

  * You need **lightweight** value types.
  * You care about **performance** and **memory efficiency**.
  * The object represents a **single value or group of values** (e.g., coordinates).

---

---

### Q: What Are `ref struct`s Used For in C#

**A:**

A `ref struct` in C# is a **value type** that is **stack-allocated** and **cannot be boxed**, used primarily for **high-performance** and **memory-safe** operations involving **spans of memory** or **unmanaged resources**.

---

## 🔹 Definition

```csharp
ref struct MyRefStruct
{
    public int X;
    public int Y;
}
```

* Introduced in **C# 7.2**.
* Enforced by the compiler to **live only on the stack**.

---

## 🔹 Key Characteristics

| Feature                        | Behavior              |
| ------------------------------ | --------------------- |
| Allocation                     | Only on the **stack** |
| Boxing                         | ❌ Not allowed         |
| Interface implementation       | ❌ Not allowed         |
| Capture in lambdas/async       | ❌ Not allowed         |
| Use in iterator methods        | ❌ Not allowed         |
| Fields must also be stack-only | ✅ Enforced            |

---

## 🔹 Main Use Case: `Span<T>` and `ReadOnlySpan<T>`

These are **stack-only** types that allow **safe, fast, and memory-efficient slicing** of arrays, strings, and unmanaged memory without allocations.

```csharp
public static void PrintSlice(Span<int> span)
{
    foreach (var item in span)
        Console.WriteLine(item);
}
```

You can call this with a portion of an array:

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
PrintSlice(numbers.AsSpan(1, 3)); // Prints 2, 3, 4
```

Because `Span<T>` is a `ref struct`, it **avoids heap allocation** and provides **zero-copy** slicing.

---

## 🔸 Why Use `ref struct`?

### ✅ Performance

* **No GC allocation** → stack-only usage is faster and avoids pressure on the heap.

### ✅ Safety

* The compiler **prevents accidental misuse**, like capturing in async or moving to the heap.

---

## 🔹 Limitations

`ref struct`s **cannot**:

* Be used as fields in **class types** (which live on the heap).
* Be captured in **async methods** or **closures**.
* Implement **interfaces**.
* Be used with **boxing**, reflection, or `object`.

---

## 🔸 Summary Table

| Feature                  | `ref struct`                   |
| ------------------------ | ------------------------------ |
| Allocation               | Stack only                     |
| Boxing                   | ❌ Not allowed                  |
| Async/lambda             | ❌ Cannot capture               |
| Interface implementation | ❌ Not allowed                  |
| Use case                 | High-performance memory access |
| Example types            | `Span<T>`, `ReadOnlySpan<T>`   |

---

## ✅ When to Use

* When working with **`Span<T>`**, **unmanaged memory**, or **performance-critical code**.
* In libraries or APIs that need **low-level control over memory** without sacrificing safety.
* When you want the compiler to **guarantee no heap allocation**.

---

---

### Q: Two Forms of Records in C#

**A:**

In C#, **records** are reference types introduced in **C# 9** that are designed for **immutable, value-based data modeling**. Records come in **two forms**:

---

## 🔹 1. **Positional Record**

### Definition:

* A **concise syntax** that automatically generates:

  * Constructor
  * Properties (with `init` accessors)
  * `Deconstruct()` method
  * Value-based `Equals()` and `GetHashCode()`

### Syntax:

```csharp
public record Person(string Name, int Age);
```

### Usage:

```csharp
var p1 = new Person("Alice", 30);
Console.WriteLine(p1.Name); // Alice

// With-expression (non-destructive mutation)
var p2 = p1 with { Age = 31 };
```

---

## 🔹 2. **Non-Positional Record (Standard Record)**

### Definition:

* Defined with **explicit property declarations** and optional custom logic.
* More flexible than positional records.

### Syntax:

```csharp
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}
```

### Usage:

```csharp
var p1 = new Person { Name = "Alice", Age = 30 };
var p2 = p1 with { Age = 31 };
```

---

## 🔸 Summary Table

| Feature     | **Positional Record**                | **Non-Positional Record**               |
| ----------- | ------------------------------------ | --------------------------------------- |
| Syntax      | Compact, inline property declaration | Explicit property definition            |
| Constructor | Auto-generated                       | You can define custom constructors      |
| Flexibility | Less flexible                        | More control over members/logic         |
| Best for    | Simple, immutable data models        | Complex models needing logic/validation |

---

## ✅ When to Use

* Use a **positional record** when:

  * You need a **simple, immutable data carrier** (like DTOs).
  * You want concise syntax.

* Use a **non-positional record** when:

  * You need to add **custom logic**, **inheritance**, or **data validation**.

---

## Bonus: `record struct` (C# 10)

C# 10 introduced **value-type records**:

```csharp
public readonly record struct Point(int X, int Y);
```

This combines record features with the memory efficiency of a `struct`.

---

---

### Q: What Is the `with` Keyword Used For in C#

**A:**

The `with` keyword in C# is used with **records** and **record structs** to perform a **non-destructive mutation**, meaning it creates a **copy** of an object with one or more **modified properties**, while leaving the original object unchanged.

---

## 🔹 Applicable To

* `record` (reference types) — introduced in **C# 9**
* `record struct` (value types) — introduced in **C# 10**

---

## 🔹 Syntax

```csharp
var newObject = existingObject with { Property1 = newValue };
```

It creates a **shallow copy** of the object and changes only the specified properties.

---

## 🔹 Example with `record`

```csharp
public record Person(string Name, int Age);

var person1 = new Person("Alice", 30);
var person2 = person1 with { Age = 31 };

Console.WriteLine(person1); // Person { Name = Alice, Age = 30 }
Console.WriteLine(person2); // Person { Name = Alice, Age = 31 }
```

* `person2` is a **copy** of `person1` with a new age.
* `person1` remains unchanged.

---

## 🔹 Behind the Scenes

The compiler generates a **`Clone()` method** that is used to perform the copy. The `with` keyword calls that method and sets the modified properties.

---

## 🔸 Summary Table

| Feature                 | Description                                 |
| ----------------------- | ------------------------------------------- |
| Applies to              | `record`, `record struct`                   |
| Operation type          | Non-destructive (creates a copy)            |
| Mutates original object | ❌ No                                        |
| Generates               | Uses auto-generated `Clone()` method        |
| Useful for              | Immutability, safe updates to stateful data |

---

## ✅ When to Use

* When working with **immutable objects** and you need to change just **one or two properties**.
* In scenarios involving **state updates**, **functional programming**, or **data transfer**.

---

## ⚠️ Not Usable With

* Regular `class` or `struct` types.
* Must use `record` or `record struct` for `with` to be available.

---

---

### Q: What Is the Purpose of Primary Constructors in C#

**A:**

**Primary constructors** in C# provide a **concise syntax** to declare **constructor parameters directly in the class (or struct/record) header**. Introduced in **C# 12**, this feature is designed to reduce boilerplate and make **immutable data objects** and **dependency injection** setups cleaner and more readable.

---

## 🔹 Purpose

* **Simplify constructor declaration** by eliminating the need for manually assigning parameters to fields or properties.
* Make code **more readable and concise**, especially for data models and services.
* Useful in **lightweight POCOs**, **records**, and **dependency injection** scenarios.

---

## 🔹 Syntax

```csharp
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}
```

* `name` and `age` are **constructor parameters** and can be used inside the body.
* You can expose them via **expression-bodied properties** or assign them manually to fields if needed.

---

## 🔹 Traditional vs Primary Constructor

### 🔸 Traditional Constructor:

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
```

### 🔸 With Primary Constructor (C# 12):

```csharp
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}
```

---

## 🔹 Usage in Records (C# 9+)

Records already supported primary-like constructors since C# 9:

```csharp
public record Person(string Name, int Age);
```

But now the same clarity is extended to **regular classes and structs** in C# 12.

---

## 🔸 Summary Table

| Feature       | Primary Constructor               |
| ------------- | --------------------------------- |
| Introduced in | C# 12                             |
| Applies to    | `class`, `struct`, `record`       |
| Syntax        | Parameters in type declaration    |
| Reduces       | Boilerplate code for constructors |
| Use case      | Data models, services, DI, POCOs  |

---

## ✅ When to Use

* When building **simple data containers** or **utility classes**.
* When using **dependency injection** in minimal APIs or clean architecture.
* When you want to **eliminate redundant field/property declarations** and assignments.

---

## ⚠️ Limitations (C# 12)

* You can’t use them in combination with traditional constructors in the same type.
* Parameters are only **in scope within the type body** — they are **not automatically assigned** to properties (unlike in records).

---

---

### Q: Do `switch` Expressions Have Any Return Type Limitations?

**A:**

No, **`switch` expressions in C# do not have return type limitations** — they can return **any type**, including:

* Primitive types (`int`, `string`, etc.)
* Complex types (`Person`, `Shape`, etc.)
* Tuples
* Anonymous types
* Even `null`

However, **all branches must return a compatible type**, and the expression must be **exhaustive** (i.e., cover all possible input values either explicitly or with a `_` wildcard).

---

## 🔹 Syntax

```csharp
var result = input switch
{
    1 => "One",
    2 => "Two",
    _ => "Other"
};
```

---

## 🔹 Example: Returning a Primitive Type

```csharp
int GetScoreGrade(int score) => score switch
{
    >= 90 => 5,
    >= 80 => 4,
    >= 70 => 3,
    >= 60 => 2,
    _ => 1
};
```

✅ All arms return `int`.

---

## 🔹 Example: Returning a Complex Object

```csharp
public record Animal(string Name);

Animal GetAnimal(string type) => type switch
{
    "dog" => new Animal("Dog"),
    "cat" => new Animal("Cat"),
    _     => new Animal("Unknown")
};
```

✅ All arms return an `Animal`.

---

## 🔹 Example: Returning `null`

```csharp
string? GetCountryCode(string country) => country switch
{
    "Spain" => "ES",
    "France" => "FR",
    _ => null
};
```

✅ Works fine as long as the return type allows `null`.

---

## 🔸 Rules to Remember

| Rule                         | Explanation                                                       |
| ---------------------------- | ----------------------------------------------------------------- |
| Must return compatible types | All branches must return the same or implicitly convertible type  |
| Must be exhaustive           | Use `_` for default case if not all inputs are explicitly covered |
| Nullable types are allowed   | As long as the return type supports null                          |
| Pattern matching supported   | You can use `when`, type patterns, relational patterns, etc.      |

---

## ✅ When to Use

* To replace verbose `switch`-`case` statements with **expressive, concise logic**.
* When returning **different values based on conditions** (e.g., enums, string states).
* In **expression-bodied members** for clarity.

---

## 🧠 Tip

If you get a compiler error in a `switch` expression, check:

1. **Are all return types compatible?**
2. **Is the switch exhaustive?**
3. **Is there any implicit conversion issue?**

---

---

## Async & Threading

### Q: How to Perform a Lock in Asynchronous Code in C#

**A:**

The traditional `lock` statement in C# (which uses `Monitor.Enter/Exit`) **does not work** with asynchronous code (`async`/`await`), because it blocks the thread and doesn't support asynchronous context switching.

To lock in asynchronous code, you must use **`SemaphoreSlim`** or **an `AsyncLock` pattern**.

---

## 🔹 Why `lock` Doesn’t Work with `async`

```csharp
private readonly object _lock = new object();

public async Task DoWorkAsync()
{
    lock (_lock) // ❌ Not compatible with async/await
    {
        await Task.Delay(1000); // Compilation error
    }
}
```

This produces a compile error because `await` cannot be used inside a synchronous `lock`.

---

## ✅ Recommended: `SemaphoreSlim`

Use `SemaphoreSlim` with `WaitAsync()` and `Release()`.

### Example:

```csharp
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

public async Task DoWorkAsync()
{
    await _semaphore.WaitAsync(); // Acquire lock (async)

    try
    {
        // Critical section
        Console.WriteLine("Doing work...");
        await Task.Delay(1000);
    }
    finally
    {
        _semaphore.Release(); // Always release
    }
}
```

* Ensures only **one thread** can execute the critical section at a time.
* Fully supports `async/await`.

---

## 🔹 Optional: AsyncLock Helper Class

You can encapsulate this logic with a reusable **`AsyncLock`** class:

```csharp
public class AsyncLock
{
    private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

    public async Task<IDisposable> LockAsync()
    {
        await _semaphore.WaitAsync();
        return new Releaser(_semaphore);
    }

    private class Releaser : IDisposable
    {
        private readonly SemaphoreSlim _semaphore;
        public Releaser(SemaphoreSlim semaphore) => _semaphore = semaphore;
        public void Dispose() => _semaphore.Release();
    }
}
```

### Usage:

```csharp
private readonly AsyncLock _lock = new AsyncLock();

public async Task CriticalSectionAsync()
{
    using (await _lock.LockAsync())
    {
        // Safe to await here
        await Task.Delay(500);
    }
}
```

---

## 🔸 Summary Table

| Approach           | Async-compatible | Notes                           |
| ------------------ | ---------------- | ------------------------------- |
| `lock`             | ❌ No             | Only for synchronous code       |
| `SemaphoreSlim`    | ✅ Yes            | Best for most `async` scenarios |
| Custom `AsyncLock` | ✅ Yes            | Cleaner, safer abstraction      |

---

## ✅ Best Practices

* Use `SemaphoreSlim` for **fine-grained control** over access to async methods.
* Always `Release()` in a `finally` block to avoid deadlocks.
* Do **not** use `lock` with `async/await`.

---

---

### Q: All the Ways to Create a New Thread in C#

**A:**

C# offers multiple ways to **create and manage threads**, ranging from **low-level control** (`Thread`) to **high-level abstractions** (`Task`, `async/await`, and `ThreadPool`). Each method has its own use case and level of abstraction.

---

## 🔹 1. `Thread` Class (Low-Level)

* Provides **manual control** over thread lifecycle.
* Best for long-running, dedicated threads.

```csharp
var thread = new Thread(() =>
{
    Console.WriteLine("Running on a new thread");
});
thread.Start();
```

---

## 🔹 2. `ThreadPool` (Shared Thread Reuse)

* Uses a pool of background threads managed by .NET.
* Optimized for **short-lived** and **concurrent** operations.

```csharp
ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine("Running on a thread pool thread");
});
```

---

## 🔹 3. `Task` (Recommended for Most Scenarios)

* Preferred way to handle **asynchronous** and **parallel** code.
* Uses the thread pool internally.

```csharp
Task.Run(() =>
{
    Console.WriteLine("Running inside a Task");
});
```

---

## 🔹 4. `async` / `await` (High-Level Abstraction)

* Works with `Task` and `Task<T>`.
* Lets the **runtime handle thread continuation** and context switches.

```csharp
public async Task DoWorkAsync()
{
    await Task.Delay(1000); // Doesn't block a thread
    Console.WriteLine("Async work completed");
}
```

---

## 🔹 5. `Parallel` Class (For Parallelism)

* Part of `System.Threading.Tasks.Parallel`.
* Used for **CPU-bound, data-parallel operations**.

```csharp
Parallel.For(0, 5, i =>
{
    Console.WriteLine($"Parallel iteration {i}");
});
```

---

## 🔹 6. `BackgroundWorker` (Obsolete/Legacy)

* Old way of doing multithreading in WinForms/WPF.
* Supports events for progress and completion.

```csharp
var worker = new BackgroundWorker();
worker.DoWork += (s, e) => Console.WriteLine("BackgroundWorker thread");
worker.RunWorkerAsync();
```

---

## 🔹 7. `TaskFactory` (Advanced Task Creation)

* Offers more control over `Task` creation (scheduling, options).

```csharp
TaskFactory factory = new TaskFactory();
factory.StartNew(() => Console.WriteLine("TaskFactory thread"));
```

---

## 🔸 Summary Table

| Method              | Type             | Level      | Use Case                                 |
| ------------------- | ---------------- | ---------- | ---------------------------------------- |
| `Thread`            | Manual           | Low        | Full control, dedicated threads          |
| `ThreadPool`        | Shared           | Medium     | Short tasks, avoids thread creation cost |
| `Task.Run`          | ThreadPool-based | High       | General-purpose async work               |
| `async/await`       | Task-based       | High       | Async workflows, modern C# apps          |
| `Parallel.For/Each` | Parallel         | High       | CPU-bound, data parallel loops           |
| `BackgroundWorker`  | Legacy           | Low/Medium | Old GUI apps (WPF/WinForms)              |
| `TaskFactory`       | Advanced Task    | High       | Custom Task scheduling                   |

---

## ✅ When to Use What

* Use `async/await` and `Task` for most modern apps.
* Use `Parallel` for CPU-heavy operations.
* Avoid raw `Thread` unless you need full control.
* Use `ThreadPool` for fast-fire, background jobs.
* Avoid `BackgroundWorker` in new development.

---

---

### Q: How to Execute Multiple `async` Tasks at Once in C#

**A:**

To run multiple asynchronous tasks **concurrently** in C#, you can use **`Task.WhenAll`**, which executes all tasks in **parallel** and waits for all of them to complete. This is essential for optimizing performance when tasks are **independent**.

---

## 🔹 Method 1: `Task.WhenAll()`

### Example:

```csharp
public async Task RunTasksInParallelAsync()
{
    Task task1 = Task.Delay(1000);
    Task task2 = Task.Delay(1500);
    Task task3 = Task.Delay(2000);

    await Task.WhenAll(task1, task2, task3); // Wait for all to finish
    Console.WriteLine("All tasks completed");
}
```

* All tasks **start immediately**.
* `await Task.WhenAll(...)` **waits** for **all** to complete.

---

## 🔹 Method 2: `Task.WhenAll<T>()` with Return Values

```csharp
public async Task RunTasksWithResultsAsync()
{
    Task<int> t1 = GetNumberAsync(1);
    Task<int> t2 = GetNumberAsync(2);

    int[] results = await Task.WhenAll(t1, t2);
    Console.WriteLine($"Sum: {results.Sum()}");
}

private async Task<int> GetNumberAsync(int value)
{
    await Task.Delay(1000);
    return value * 10;
}
```

---

## 🔹 Method 3: `Parallel.ForEachAsync` (.NET 6+)

If you’re using .NET 6 or later, you can process a collection **concurrently**:

```csharp
var urls = new[] { "url1", "url2", "url3" };

await Parallel.ForEachAsync(urls, async (url, token) =>
{
    await FetchUrlAsync(url);
});
```

---

## 🔸 Summary Table

| Method                  | Description                            | .NET Version |
| ----------------------- | -------------------------------------- | ------------ |
| `Task.WhenAll()`        | Run multiple tasks in parallel         | All          |
| `Task.WhenAll<T>()`     | Run and collect results from tasks     | All          |
| `Parallel.ForEachAsync` | Run async loops concurrently over data | .NET 6+      |

---

## ⚠️ Important Notes

* Tasks must be **started before** passing them to `Task.WhenAll`.
* If any task **throws**, `Task.WhenAll` throws an **AggregateException**.
* Avoid `await`ing each task individually if you want **parallel**, not **sequential**, execution.

---

## ✅ Best Practice

```csharp
var tasks = new List<Task>
{
    Task1Async(),
    Task2Async(),
    Task3Async()
};

await Task.WhenAll(tasks); // Run concurrently
```

Use `Task.WhenAll()` to execute **independent async operations** in parallel and boost performance.

---

---

### Q: What is the difference between `Task`, `ValueTask`, and `void` in async methods?

**A:**

| Return Type | Use Case                                              | Awaitable? | Recommended? |
| ----------- | ----------------------------------------------------- | ---------- | ------------ |
| `void`      | For **event handlers** only                           | ❌ No       | ❌ No         |
| `Task`      | For **async operations** with no result               | ✅ Yes      | ✅ Yes        |
| `ValueTask` | For **high-performance** scenarios, avoid allocations | ✅ Yes      | ✅ Sometimes  |

```csharp
public async Task DoWorkAsync() { }

public async ValueTask<int> GetResultAsync() => 42;

public async void OnClick(object sender, EventArgs e) { } // Only for events
```

---

---

### Q: What is a CancellationToken and how do you use it?

**A:**

A `CancellationToken` allows cooperative cancellation of async operations.

```csharp
public async Task DoWorkAsync(CancellationToken token)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(1000);
    }
}
```

Usage:

```csharp
var cts = new CancellationTokenSource();
await DoWorkAsync(cts.Token);
cts.Cancel(); // Request cancellation
```

---

---

### Q: What is the difference between `lock` and `Monitor`?

**A:**

### `lock` is syntactic sugar over `Monitor.Enter/Exit`.

```csharp
lock (locker)
{
    // critical section
}
```

Same as:

```csharp
Monitor.Enter(locker);
try
{
    // critical section
}
finally
{
    Monitor.Exit(locker);
}
```

✅ Use `lock` for clarity unless you need `Monitor.TryEnter` or `Wait/Pulse`.

---

---

### Q: What is the difference between `Thread.Sleep()` and `Task.Delay()`?

**A:**

| Method           | Blocking?       | Use in async code? | Use case    |
| ---------------- | --------------- | ------------------ | ----------- |
| `Thread.Sleep()` | ✅ Blocks thread | ❌ No               | Sync delay  |
| `Task.Delay()`   | ❌ Non-blocking  | ✅ Yes              | Async delay |

```csharp
await Task.Delay(1000); // preferred in async methods
```

---

---

### Q: What is the purpose of the `volatile` keyword?

**A:**

Indicates that a field may be **modified by multiple threads**, and prevents **compiler optimizations** that could reorder access.

```csharp
private volatile bool _isRunning;
```

✅ Ensures **read/write visibility** across threads.
⚠️ Does **not make operations atomic** — use `Interlocked` or locks when needed.

---

---

## Language Fundamentals

### Q: When is a `static` constructor called in C#

**A:**

A `static constructor` is used to **initialize static data or perform actions that only need to be done once**. It is called **automatically** by the **runtime** before any static members are accessed or any instance of the class is created.

---

## 🔹 Characteristics of Static Constructors

* Does **not take parameters**.
* Cannot be called explicitly.
* Executed **once per type**, not per object.
* Runs **before** the first access to any static member **or** the first instance creation (whichever comes first).
* Does **not** have any access modifier — always `private` by default.

---

## 🔹 Example

```csharp
public class DatabaseConnection
{
    public static string ConnectionString;

    static DatabaseConnection()
    {
        Console.WriteLine("Static constructor called");
        ConnectionString = "Server=localhost;Database=AppDb;";
    }

    public static void Connect()
    {
        Console.WriteLine($"Connecting using: {ConnectionString}");
    }
}
```

### Usage:

```csharp
DatabaseConnection.Connect(); 
// Output:
// Static constructor called
// Connecting using: Server=localhost;Database=AppDb;
```

Even if you created an instance like:

```csharp
var db = new DatabaseConnection(); 
```

The static constructor would run **before** the constructor for any instance, but **only once**.

---

## 🔸 Summary Table

| Feature              | Static Constructor               |
| -------------------- | -------------------------------- |
| Parameters allowed   | ❌ No                             |
| Access modifier      | ❌ Not allowed                    |
| Called explicitly    | ❌ No                             |
| Called automatically | ✅ Yes, before first use of type  |
| Execution count      | ✅ Once per type (not per object) |

---

## ✅ When to Use

* Use static constructors to **initialize static fields**.
* Ideal when setup logic is required before any member (static or instance) is used.
* Useful for **dependency injection setup**, **configuration loading**, or **initial logging**.

---

---

### Q: How to Create an Extension Method in C#

**A:**

An **extension method** allows you to **add new methods** to existing types (including classes, structs, interfaces) **without modifying their source code** or using inheritance.

---

## 🔹 Requirements

* Must be declared in a **static class**.
* The method itself must be **static**.
* The first parameter must use the `this` keyword to specify the type being extended.

---

## 🔹 Syntax

```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string input)
    {
        return string.IsNullOrEmpty(input);
    }
}
```

Usage:

```csharp
string name = null;
bool result = name.IsNullOrEmpty(); // Calls the extension method
```

---

## 🔹 Explanation

* The compiler translates the call `name.IsNullOrEmpty()` into `StringExtensions.IsNullOrEmpty(name)`.
* You can use extension methods with:

  * .NET base types (`string`, `int`, etc.)
  * Your own types (e.g., `Customer`, `Order`, etc.)
  * Interfaces (`IEnumerable<T>`, `IDisposable`, etc.)

---

## 🔹 Example: Extending `List<T>`

```csharp
public static class ListExtensions
{
    public static void PrintAll<T>(this List<T> list)
    {
        foreach (var item in list)
        {
            Console.WriteLine(item);
        }
    }
}
```

Usage:

```csharp
var numbers = new List<int> { 1, 2, 3 };
numbers.PrintAll(); // Output: 1 2 3
```

---

## 🔸 Summary Table

| Rule                         | Requirement                          |
| ---------------------------- | ------------------------------------ |
| Declared in                  | `static` class                       |
| Method itself                | Must be `static`                     |
| First parameter              | Has `this` keyword + type to extend  |
| Usage                        | Called like an instance method       |
| Namespace inclusion required | Must `using` the extension namespace |

---

## ✅ When to Use

* To add functionality to **types you can't modify** (e.g., `string`, `DateTime`, third-party classes).
* To improve code **readability and reusability**.
* To avoid **utility/helper methods** with awkward syntax like `Utils.DoSomething(obj)`.

---

---

### Q: Does C# Support Multiple Class Inheritance?

**A:**

No, **C# does not support multiple inheritance of classes**. A class in C# can inherit from **only one base class**. This restriction avoids ambiguity and complexity, such as the “diamond problem” common in multiple inheritance scenarios.

---

## 🔹 What C# Supports Instead

C# supports **multiple interface inheritance**. A class can implement **multiple interfaces**, allowing you to compose behavior from different sources without inheriting from multiple classes.

---

## 🔹 Example: Single Class Inheritance

```csharp
public class Animal
{
    public void Eat() => Console.WriteLine("Eating...");
}

public class Dog : Animal
{
    public void Bark() => Console.WriteLine("Barking...");
}
```

✅ This is allowed — `Dog` inherits from only one base class: `Animal`.

---

## 🔹 Invalid Example: Multiple Class Inheritance

```csharp
public class A { }
public class B { }

// ❌ Compile-time error in C#
public class C : A, B { }
```

This will produce an error:

> *Class 'C' cannot have multiple base classes: 'A' and 'B'*

---

## 🔹 Valid Alternative: Multiple Interface Inheritance

```csharp
public interface IFly
{
    void Fly();
}

public interface ISwim
{
    void Swim();
}

public class Duck : IFly, ISwim
{
    public void Fly() => Console.WriteLine("Flying...");
    public void Swim() => Console.WriteLine("Swimming...");
}
```

✅ `Duck` implements both `IFly` and `ISwim` — no problem.

---

## 🔸 Summary Table

| Feature                            | Supported in C# |
| ---------------------------------- | --------------- |
| Multiple **class** inheritance     | ❌ No            |
| Multiple **interface** inheritance | ✅ Yes           |

---

## ✅ Why This Design?

* Avoids **ambiguity** in base class method resolution.
* Encourages **composition over inheritance**.
* Interfaces provide **flexibility** without introducing complexity.

---

---

### Q: Difference Between `string` and `StringBuilder` in C#

**A:**

Both `string` and `StringBuilder` are used to work with **text** in C#, but they differ significantly in **mutability**, **performance**, and **intended use cases**.

---

## 🔹 `string`

### Definition:

* An **immutable** sequence of characters.
* Once created, it **cannot be changed** — every operation that modifies a string creates a **new instance** in memory.

### Characteristics:

* Stored on the **heap** (reference type).
* Any modification (e.g., `+`, `Replace`, `Substring`) results in a **new string object**.
* Simple to use and efficient for **few concatenations**.

### Example:

```csharp
string name = "John";
name += " Doe"; // Creates a new string in memory
```

---

## 🔹 `StringBuilder`

### Definition:

* A **mutable** string-like object optimized for **frequent or large text modifications**.

### Characteristics:

* Defined in `System.Text` namespace.
* Stores characters in a **buffer**, modifying the same object.
* Better performance for **repeated appending or inserting**.

### Example:

```csharp
using System.Text;

StringBuilder sb = new StringBuilder("John");
sb.Append(" Doe"); // No new object created
string fullName = sb.ToString();
```

---

## 🔸 Performance Comparison

```csharp
// Using string
string s = "";
for (int i = 0; i < 1000; i++)
{
    s += i; // Inefficient: creates many string instances
}

// Using StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i); // Efficient: modifies the same buffer
}
```

---

## 🔸 Summary Table

| Feature          | `string`                      | `StringBuilder`                          |
| ---------------- | ----------------------------- | ---------------------------------------- |
| Mutability       | Immutable                     | Mutable                                  |
| Namespace        | `System`                      | `System.Text`                            |
| Performance      | Poor for many modifications   | Good for repeated modifications          |
| Thread safety    | Yes (because it's immutable)  | No (unless synchronized manually)        |
| Common use cases | Small, simple text operations | Loops, parsers, templating, log building |

---

## ✅ When to Use

* Use `string` when:

  * You have a **small number of operations**.
  * You prefer **readability** and simplicity.
  * Immutability is desired.

* Use `StringBuilder` when:

  * You need to perform **many concatenations or changes**.
  * You're building large text blobs, logs, or dynamic documents.
  * You care about **memory and performance**.

---

---

### Q: How to Create a Date with a Specific Time Zone in C#

**A:**

In C#, `DateTime` itself **does not store time zone information**. To work with specific time zones, you need to use the **`TimeZoneInfo`** class along with `DateTime`.

---

## 🔹 Step-by-Step: Creating a Date in a Specific Time Zone

### 1. Define a `DateTime` (usually in UTC or local time).

### 2. Get the target `TimeZoneInfo`.

### 3. Use `TimeZoneInfo.ConvertTime()` to convert the date.

---

## 🔹 Example: Create Date in “Eastern Standard Time”

```csharp
DateTime utcNow = DateTime.UtcNow;

TimeZoneInfo estZone = TimeZoneInfo.FindSystemTimeZoneById("Eastern Standard Time");

DateTime estTime = TimeZoneInfo.ConvertTimeFromUtc(utcNow, estZone);

Console.WriteLine(estTime); // Shows current time in EST
```

---

## 🔹 Example: Create a Specific Date in a Time Zone

```csharp
// Let's say you want July 29, 2025, 9:00 AM in "Central European Standard Time"
DateTime naiveDate = new DateTime(2025, 7, 29, 9, 0, 0); // Assumed local

TimeZoneInfo cetZone = TimeZoneInfo.FindSystemTimeZoneById("Central European Standard Time");

DateTime cetDate = TimeZoneInfo.ConvertTime(naiveDate, cetZone);

Console.WriteLine(cetDate); // Outputs the time in CET
```

---

## 🔹 List of Common Time Zone IDs (Windows)

| Time Zone             | ID                                 |
| --------------------- | ---------------------------------- |
| UTC                   | `"UTC"`                            |
| Eastern Time (US)     | `"Eastern Standard Time"`          |
| Central European Time | `"Central European Standard Time"` |
| Tokyo                 | `"Tokyo Standard Time"`            |
| Pacific Time (US)     | `"Pacific Standard Time"`          |

Use this to retrieve the correct time zone:

```csharp
var zone = TimeZoneInfo.FindSystemTimeZoneById("TimeZoneId");
```

---

## 🔹 Notes on Cross-Platform

* On **Windows**, use `TimeZoneInfo.FindSystemTimeZoneById("...")` with Windows time zone IDs.
* On **Linux/macOS**, use **IANA time zone IDs** like `"Europe/Madrid"` (requires .NET 6+ or using [NodaTime](https://nodatime.org/)).

---

## ✅ When to Use

* Use `TimeZoneInfo` when converting **between time zones**.
* Use UTC internally and convert to local time zones **only for display**.
* For complex date/time logic, consider using the **NodaTime** library.

---

---

### Q: How to Change the Current Culture in C#

**A:**

In C#, you can change the **current culture** and **UI culture** using the `CultureInfo` class from the `System.Globalization` namespace. This affects how values like **dates**, **numbers**, and **currency** are formatted and parsed.

---

## 🔹 Types of Culture

| Culture Type       | Affects                            |
| ------------------ | ---------------------------------- |
| `CurrentCulture`   | Formatting of dates, numbers, etc. |
| `CurrentUICulture` | Resource lookups for localization  |

---

## 🔹 Example: Change Culture to French (`fr-FR`)

```csharp
using System;
using System.Globalization;
using System.Threading;

class Program
{
    static void Main()
    {
        CultureInfo frenchCulture = new CultureInfo("fr-FR");

        Thread.CurrentThread.CurrentCulture = frenchCulture;
        Thread.CurrentThread.CurrentUICulture = frenchCulture;

        DateTime today = DateTime.Now;
        double number = 1234.56;

        Console.WriteLine(today.ToString());   // e.g., "29/07/2025 17:45:00"
        Console.WriteLine(number.ToString()); // e.g., "1 234,56"
    }
}
```

---

## 🔹 .NET 6+ (Global Change)

In .NET 6 or later, you can set the culture **globally** using:

```csharp
CultureInfo.DefaultThreadCurrentCulture = new CultureInfo("es-ES");
CultureInfo.DefaultThreadCurrentUICulture = new CultureInfo("es-ES");
```

This sets the culture for all threads in the application **by default**.

---

## 🔹 Common Culture Codes

| Language         | Culture Code |
| ---------------- | ------------ |
| English (US)     | `en-US`      |
| Spanish (Spain)  | `es-ES`      |
| French (France)  | `fr-FR`      |
| German (Germany) | `de-DE`      |
| Japanese         | `ja-JP`      |

---

## 🔸 Summary Table

| Culture Property              | Description                             |
| ----------------------------- | --------------------------------------- |
| `CurrentCulture`              | Formatting for numbers, dates, currency |
| `CurrentUICulture`            | Resource localization                   |
| `DefaultThreadCurrentCulture` | Global default (since .NET 4.5+)        |
| `CultureInfo("...")`          | Create a new culture object             |

---

## ✅ When to Use

* When developing **multilingual apps** or dealing with **international data**.
* When formatting output like **currency, dates, decimals** in a locale-specific way.
* To load the correct **localized resources** (`.resx`) in global applications.

---

---

### Q: What Is `yield return` Used For in C#

**A:**

The `yield return` statement is used in C# to **implement custom iterators**. It allows you to return **one element at a time** from a method, without creating a full collection or managing state manually.

It simplifies the creation of **lazy, on-demand sequences** using `IEnumerable` or `IEnumerable<T>`.

---

## 🔹 Purpose

* Enables **deferred execution** and **lazy iteration**.
* Avoids storing all results in memory at once.
* Maintains **iteration state** between calls automatically.

---

## 🔹 Syntax Example

```csharp
public static IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

Usage:

```csharp
foreach (var n in GetNumbers())
{
    Console.WriteLine(n);
}
// Output: 1 2 3
```

---

## 🔹 How It Works

* Each `yield return` **pauses** the method and **remembers its state**.
* On the next iteration, it **resumes** from the last `yield return`.
* `yield break` can be used to **exit** the iterator early.

---

## 🔹 Example: Fibonacci Sequence

```csharp
public static IEnumerable<int> Fibonacci(int count)
{
    int a = 0, b = 1;

    for (int i = 0; i < count; i++)
    {
        yield return a;
        int temp = a;
        a = b;
        b = temp + b;
    }
}
```

---

## 🔸 Summary Table

| Feature              | `yield return`                    |
| -------------------- | --------------------------------- |
| Execution style      | Deferred / lazy                   |
| State management     | Automatic                         |
| Memory usage         | Efficient (one item at a time)    |
| Return type required | `IEnumerable` or `IEnumerable<T>` |
| Stops iteration      | Use `yield break`                 |

---

## ✅ When to Use

* When generating **large or infinite sequences** (e.g., logs, Fibonacci, file lines).
* When **performance or memory** is a concern.
* When you want to **stream** data rather than return full collections.

---

## ⚠️ Limitations

* Cannot use `ref`/`out` parameters.
* Cannot use inside `async` methods.
* Cannot return multiple enumerables from the same method using different logic.

---

## 🧠 Tip

Think of `yield return` as a **lazy version of `return`** that creates **iterators without boilerplate**.

---

---

### Q: What Is the Code Generated by the Compiler for Auto-Properties?

**A:**

When you define **auto-properties** in C#, the compiler **automatically generates a private backing field** behind the scenes to store the value. This keeps your code concise while maintaining encapsulation.

---

## 🔹 Example: Auto-Property

```csharp
public class Person
{
    public string Name { get; set; }
}
```

---

## 🔹 Compiler-Generated Code (Roughly Equivalent To)

```csharp
public class Person
{
    private string _name; // Compiler-generated field (name varies internally)

    public string Name
    {
        get => _name;
        set => _name = value;
    }
}
```

* The backing field is **not accessible directly** from your code.
* The field’s actual name is compiler-generated and usually something like `<Name>k__BackingField`.

---

## 🔹 You Can See It with Reflection

```csharp
var fields = typeof(Person).GetFields(
    System.Reflection.BindingFlags.NonPublic | 
    System.Reflection.BindingFlags.Instance);

foreach (var field in fields)
{
    Console.WriteLine(field.Name); 
    // Output: <Name>k__BackingField
}
```

---

## 🔹 Read-Only Auto-Property (C# 6+)

```csharp
public string Id { get; } = Guid.NewGuid().ToString();
```

Compiler-generated version:

```csharp
private readonly string _id = Guid.NewGuid().ToString();

public string Id => _id;
```

---

## 🔸 Summary Table

| Feature             | Auto-Property                          | Behind the Scenes                        |
| ------------------- | -------------------------------------- | ---------------------------------------- |
| `get; set;`         | Property with both accessors           | Backed by a hidden private field         |
| `get; private set;` | Readable from outside, writable inside | Still backed by a private field          |
| `get; } = value`    | Auto-property initializer              | Backing field initialized in constructor |

---

## ✅ Benefits of Auto-Properties

* Less boilerplate.
* Cleaner and more readable code.
* Still allows full encapsulation and control when needed (via access modifiers or custom accessors).

---

## ⚠️ Limitations

* You can’t directly reference or customize the backing field unless you define it yourself.
* If you need custom logic in `get` or `set`, you must switch to a **full property** definition.

---

---

### Q: How Does the `using` Statement Work in C#

**A:**

The `using` statement in C# is used to **ensure that resources are disposed of properly**, typically those that implement the `IDisposable` interface (e.g., file streams, database connections).

It provides a **clean and safe way** to handle unmanaged resources by **automatically calling `Dispose()`** at the end of the scope.

---

## 🔹 Syntax

### Traditional `using` statement (before C# 8.0):

```csharp
using (var file = new StreamReader("file.txt"))
{
    string content = file.ReadToEnd();
}
// file.Dispose() is called automatically here
```

---

### C# 8.0+ using declaration:

```csharp
using var file = new StreamReader("file.txt");

string content = file.ReadToEnd();
// file is disposed automatically at the end of the scope
```

---

## 🔹 How It Works Internally

For:

```csharp
using (var resource = new SomeDisposable()) { /* ... */ }
```

The compiler translates it roughly to:

```csharp
var resource = new SomeDisposable();
try
{
    // use resource
}
finally
{
    if (resource != null)
        resource.Dispose();
}
```

---

## 🔹 Requirements

The object must implement the `IDisposable` interface:

```csharp
public interface IDisposable
{
    void Dispose();
}
```

---

## 🔹 Common Use Cases

| Type                        | Scenario                      |
| --------------------------- | ----------------------------- |
| `StreamReader`/`FileStream` | File I/O                      |
| `SqlConnection`             | Database connection           |
| `HttpClient`                | Web requests (with care)      |
| `MemoryStream`              | In-memory buffer handling     |
| Custom types                | Releasing unmanaged resources |

---

## 🔸 Summary Table

| Feature       | Description                                    |
| ------------- | ---------------------------------------------- |
| Purpose       | Auto-dispose unmanaged or disposable resources |
| Method called | `Dispose()`                                    |
| Requires      | `IDisposable` implementation                   |
| Scope-bound   | Yes — disposal happens at end of block         |
| C# 8+ syntax  | Supports **using declaration** (no braces)     |

---

## ✅ When to Use

* When working with **files**, **streams**, **database connections**, or any object that implements `IDisposable`.
* When you want to **guarantee cleanup** of resources, even in the presence of exceptions.

---

## 🧠 Tip

If you create your own resource-holding class, **implement `IDisposable`** and always dispose of resources using `using` or `Dispose()` explicitly.

---

---

### Q: What Is a Delegate and How Is It Used in C#

**A:**

A **delegate** in C# is a type that represents a **reference to a method** with a specific **signature**. It allows methods to be **passed as parameters**, stored in variables, or returned from other methods.

Delegates are the foundation for **events**, **callbacks**, and **LINQ expressions**.

---

## 🔹 Definition

```csharp
public delegate int Operation(int x, int y);
```

This defines a delegate that can reference **any method** that takes two `int` parameters and returns an `int`.

---

## 🔹 How to Use

### 1. **Declare a delegate**

```csharp
public delegate void Notify(string message);
```

### 2. **Assign a method to it**

```csharp
void ShowMessage(string msg) => Console.WriteLine(msg);

Notify notifier = ShowMessage;
notifier("Hello from delegate!"); // Output: Hello from delegate!
```

---

## 🔹 With Parameters and Return Values

```csharp
public delegate int MathOperation(int a, int b);

int Add(int x, int y) => x + y;
int Multiply(int x, int y) => x * y;

MathOperation op = Add;
Console.WriteLine(op(2, 3)); // Output: 5

op = Multiply;
Console.WriteLine(op(2, 3)); // Output: 6
```

---

## 🔹 Multicast Delegates

Delegates can **reference multiple methods** using `+=`.

```csharp
Notify notifyAll = ShowMessage;
notifyAll += msg => Console.WriteLine($"Echo: {msg}");

notifyAll("Test"); 
// Output:
// Test
// Echo: Test
```

Note: Only the **last method’s return value is kept** (if it's not `void`).

---

## 🔹 Built-In Generic Delegates

You usually don’t need to define custom delegates because C# includes:

| Delegate       | Description                       | Example               |
| -------------- | --------------------------------- | --------------------- |
| `Action`       | Delegate with **no return value** | `Action<string>`      |
| `Func`         | Delegate with **return value**    | `Func<int, int, int>` |
| `Predicate<T>` | `Func<T, bool>` shorthand         | `Predicate<string>`   |

### Example with `Func`:

```csharp
Func<int, int, int> sum = (a, b) => a + b;
Console.WriteLine(sum(2, 3)); // Output: 5
```

---

## 🔹 Use Cases

* **Event handling**
* **Callbacks**
* **Strategy pattern**
* **LINQ expressions**
* **Asynchronous programming**

---

## 🔸 Summary Table

| Feature                 | Delegate                              |
| ----------------------- | ------------------------------------- |
| Type of                 | Reference type for methods            |
| Supports parameters?    | ✅ Yes                                 |
| Supports return values? | ✅ Yes                                 |
| Combines methods?       | ✅ Multicast support                   |
| Related to              | Events, `Action`, `Func`, `Predicate` |

---

## ✅ When to Use

* When you need to **pass a method as a parameter** (e.g., callbacks).
* When implementing **event-driven** or **plugin-like** behavior.
* When selecting **strategies dynamically** at runtime.

---

## 🧠 Tip

Delegates enable **decoupling logic** from execution — a core principle of **flexible and maintainable code**.

---

---

### Q: How Does Exception Handling Work in C#

**A:**

Exception handling in C# is the mechanism that allows you to **gracefully handle runtime errors** using `try`, `catch`, `finally`, and `throw` keywords. It helps you **separate error-handling logic from normal code flow**, improving robustness and readability.

---

## 🔹 Basic Syntax

```csharp
try
{
    // Code that might throw an exception
    int result = 10 / 0;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    Console.WriteLine("Always executed");
}
```

---

## 🔹 Keywords Explained

### `try`

* Wraps code that might throw an exception.

### `catch`

* Catches specific or general exceptions.
* Multiple `catch` blocks allowed (from most to least specific).

```csharp
catch (FileNotFoundException ex) { }
catch (IOException ex) { }
catch (Exception ex) { } // catch-all (generic)
```

### `finally`

* Always runs, **whether or not an exception occurred**.
* Used to **clean up resources** (e.g., closing a file or connection).

### `throw`

* Re-throws the original exception or throws a new one.

```csharp
throw;              // re-throw the current exception
throw new Exception("Custom error"); // new exception
```

---

## 🔹 Custom Exceptions

You can create your own exceptions by inheriting from `Exception`:

```csharp
public class InvalidAgeException : Exception
{
    public InvalidAgeException(string message) : base(message) { }
}
```

Usage:

```csharp
if (age < 0)
    throw new InvalidAgeException("Age cannot be negative.");
```

---

## 🔹 Common Exception Types

| Exception Type              | Description                             |
| --------------------------- | --------------------------------------- |
| `NullReferenceException`    | Dereferencing a null object             |
| `DivideByZeroException`     | Division by zero                        |
| `IndexOutOfRangeException`  | Accessing an invalid array index        |
| `InvalidOperationException` | Invalid operation in the object's state |
| `FileNotFoundException`     | File not found during IO operations     |

---

## 🔹 Best Practices

✅ **Catch only what you can handle**

```csharp
try
{
    ProcessData();
}
catch (IOException ex)
{
    // Log and handle
}
```

✅ **Use `finally` for cleanup**

```csharp
FileStream fs = null;
try
{
    fs = new FileStream("file.txt", FileMode.Open);
}
finally
{
    fs?.Dispose();
}
```

✅ **Avoid catching `Exception` unless necessary**

✅ **Never swallow exceptions silently**

✅ **Use exception filters (C# 6+)**

```csharp
catch (Exception ex) when (ex.Message.Contains("specific"))
{
    // Handle only if condition matches
}
```

---

## 🔸 Summary Table

| Keyword   | Purpose                                 |
| --------- | --------------------------------------- |
| `try`     | Wraps code that may throw exceptions    |
| `catch`   | Catches and handles specific exceptions |
| `finally` | Always executes, for cleanup            |
| `throw`   | Throws or rethrows an exception         |

---

## ✅ When to Use

* To **recover gracefully** from expected errors (file not found, invalid input).
* To **log and report** critical failures.
* To ensure **resources are released**, even in the case of errors.

---

## 🧠 Tip

Use exceptions for **exceptional conditions**, not for **normal control flow**.

---

---

### Q: All the Ways to Rethrow an Exception in C#

**A:**

In C#, you can **rethrow an exception** using a few different approaches, each with its own behavior and implications, especially regarding the **preservation of the stack trace**.

---

## 🔹 1. `throw;` ✅ (Recommended)

### Description:

* **Rethrows the current exception** without modifying it.
* **Preserves the original stack trace**.

### Example:

```csharp
try
{
    // Some logic that throws
}
catch (Exception ex)
{
    // Log the exception
    Console.WriteLine(ex.Message);
    throw; // Rethrow as-is
}
```

✅ Best practice when you **don’t want to modify or wrap** the exception.

---

## 🔹 2. `throw ex;` ❌ (Not recommended)

### Description:

* Rethrows a **new copy** of the exception.
* **Resets the stack trace**, losing the original throw location.

### Example:

```csharp
catch (Exception ex)
{
    // BAD: Stack trace is reset
    throw ex;
}
```

⚠️ Use only if you **need to modify** the exception and are okay with losing the original trace.

---

## 🔹 3. `throw new Exception("message", ex);` ✅ (Wrap the original)

### Description:

* Creates a **new exception** and **wraps the original** as the `InnerException`.
* Preserves the **original exception info** but changes the outer type and trace.

### Example:

```csharp
catch (Exception ex)
{
    throw new CustomException("Error while processing data.", ex);
}
```

✅ Useful when you want to **add context** to the exception but **still keep the original** via `InnerException`.

---

## 🔹 4. `ExceptionDispatchInfo.Capture(ex).Throw();` ✅ (Advanced)

### Description:

* From `System.Runtime.ExceptionServices`.
* Allows you to **rethrow an exception while preserving its stack trace**, even **after catching and storing** it.

### Example:

```csharp
using System.Runtime.ExceptionServices;

try
{
    throw new Exception("Original");
}
catch (Exception ex)
{
    var edi = ExceptionDispatchInfo.Capture(ex);
    // Some logic
    edi.Throw(); // Stack trace preserved!
}
```

✅ Advanced technique — useful when you **defer rethrowing** to another method or layer.

---

## 🔸 Summary Table

| Syntax                                    | Preserves Stack Trace   | When to Use                                    |
| ----------------------------------------- | ----------------------- | ---------------------------------------------- |
| `throw;`                                  | ✅ Yes                   | Rethrow without modifying the exception        |
| `throw ex;`                               | ❌ No                    | Only if modifying the exception                |
| `throw new Exception("msg", ex);`         | ❌ No (outer trace only) | To wrap with context and keep `InnerException` |
| `ExceptionDispatchInfo.Capture().Throw()` | ✅ Yes                   | To rethrow later with original trace           |

---

## ✅ Best Practice

* Use `throw;` whenever possible to **maintain diagnostic clarity**.
* Avoid `throw ex;` unless you **explicitly need to replace the exception**.
* Use `ExceptionDispatchInfo` for **advanced scenarios**, like background tasks, error logging systems, or async pipelines.

---

---

### Q: Explain Generics in C#

**A:**

**Generics** in C# allow you to define **classes, interfaces, methods, and delegates** with a **placeholder for a data type**, enabling **type-safe**, **reusable**, and **performance-efficient** code without duplicating logic for each data type.

They were introduced in **.NET 2.0** and are fundamental to collections like `List<T>`, `Dictionary<TKey, TValue>`, and LINQ.

---

## 🔹 Why Use Generics?

| Benefit         | Description                                          |
| --------------- | ---------------------------------------------------- |
| **Type safety** | Errors are caught at compile-time instead of runtime |
| **Code reuse**  | Write logic once, use it for any type                |
| **Performance** | Avoids boxing/unboxing (especially with value types) |
| **Readability** | Improves clarity and intent of code                  |

---

## 🔹 Generic Class Example

```csharp
public class Box<T>
{
    public T Value { get; set; }

    public void Print() => Console.WriteLine($"Value: {Value}");
}
```

Usage:

```csharp
var intBox = new Box<int> { Value = 123 };
var strBox = new Box<string> { Value = "Hello" };

intBox.Print(); // Value: 123
strBox.Print(); // Value: Hello
```

---

## 🔹 Generic Method Example

```csharp
public static void Swap<T>(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}
```

Usage:

```csharp
int x = 1, y = 2;
Swap(ref x, ref y);
Console.WriteLine($"{x}, {y}"); // Output: 2, 1
```

---

## 🔹 Generic Interface Example

```csharp
public interface IRepository<T>
{
    void Add(T item);
    IEnumerable<T> GetAll();
}
```

Implementation:

```csharp
public class MemoryRepository<T> : IRepository<T>
{
    private List<T> _items = new();

    public void Add(T item) => _items.Add(item);
    public IEnumerable<T> GetAll() => _items;
}
```

---

## 🔹 Generic Constraints

You can **restrict** the types allowed for `T` using **constraints**:

```csharp
public class Repository<T> where T : class
{
    public void Save(T entity) { /*...*/ }
}
```

### Common Constraints:

| Constraint            | Meaning                                      |
| --------------------- | -------------------------------------------- |
| `where T : class`     | Reference type only                          |
| `where T : struct`    | Value type only                              |
| `where T : new()`     | Must have a public parameterless constructor |
| `where T : BaseClass` | Must inherit from a specific class           |
| `where T : interface` | Must implement the given interface           |

---

## 🔸 Summary Table

| Concept           | Example           | Description                   |
| ----------------- | ----------------- | ----------------------------- |
| Generic class     | `class Box<T>`    | Store values of any type      |
| Generic method    | `void Swap<T>()`  | Reuse logic with any type     |
| Generic interface | `IRepository<T>`  | Common contracts for any type |
| Constraints       | `where T : class` | Limit types that can be used  |

---

## ✅ When to Use Generics

* When you want to build **type-safe**, **reusable**, and **extensible** components.
* When working with collections, services, or algorithms that **don’t depend on a specific type**.
* When you want to **avoid casting** or **boxing/unboxing**.

---

## 🧠 Tip

Most of .NET’s collections (`List<T>`, `Queue<T>`, `Dictionary<TKey, TValue>`, etc.) are built using **generics**, because they offer **safety + performance + flexibility**.

---

---

### Q: What is the difference between `dynamic` and `var`?

**A:**

### 🔹 `var`

* Compile-time typed — the type is inferred by the compiler.
* Cannot change type once inferred.

```csharp
var number = 10;         // int
// number = "text";      // ❌ compile-time error
```

### 🔹 `dynamic`

* Runtime-typed — bypasses compile-time type checking.
* Type is resolved at runtime (like reflection).

```csharp
dynamic value = 10;
value = "text";          // ✅ valid at runtime
```

### ✅ Use `var` when the type is known but verbose.

### ⚠️ Use `dynamic` carefully — it disables IntelliSense and compile-time safety.

---

---

### Q: What is the purpose of attributes in C#?

**A:**

Attributes add **metadata** to types, methods, properties, etc.

```csharp
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

[Serializable]
public class Person { }
```

You can read them via **reflection**:

```csharp
var attrs = typeof(Person).GetCustomAttributes(false);
```

✅ Useful for validation, documentation, code generation, serialization, etc.

---

---

### Q: What are tuples and how are they different from anonymous types?

**A:**

### Tuple

```csharp
var tuple = (Name: "John", Age: 30);
Console.WriteLine(tuple.Name); // John
```

### Anonymous type

```csharp
var anon = new { Name = "John", Age = 30 };
Console.WriteLine(anon.Name); // John
```

| Feature            | Tuple            | Anonymous Type           |
| ------------------ | ---------------- | ------------------------ |
| Return from method | ✅ Yes            | ❌ Not easily             |
| Deconstruction     | ✅ Yes            | ❌ No                     |
| Type-safe          | ✅ Strongly typed | ✅ But compiler-generated |
| Usable outside     | ✅ (public)       | ❌ Scope-limited          |

---

---

### Q: What is the difference between `IDisposable` and a finalizer?

**A:**

### 🔹 `IDisposable`

* Implements the `Dispose()` method to **release resources explicitly**.
* Used with the `using` statement.

```csharp
public class ResourceHolder : IDisposable
{
    public void Dispose()
    {
        // Free resources
    }
}
```

---

### 🔹 Finalizer (`~ClassName()`)

* Invoked by the **GC** when the object is collected.
* Non-deterministic — no guarantee when it's called.

```csharp
~ResourceHolder()
{
    // Cleanup logic
}
```

✅ Use `Dispose()` for **deterministic cleanup**.
⚠️ Finalizers are a **last resort** — avoid unless handling unmanaged memory.

---

---

### Q: What is a `params` parameter?

**A:**

Allows passing a **variable number of arguments** to a method.

```csharp
public int Sum(params int[] numbers)
{
    return numbers.Sum();
}
```

Usage:

```csharp
Sum(1, 2, 3); // 6
```

✅ Must be the **last** parameter in the method signature.

---

---

### Q: What is the `nameof` operator?

**A:**

Returns the **string name** of a variable, type, or member — safely at **compile time**.

```csharp
string prop = nameof(Console.WriteLine); // "WriteLine"
```

✅ Useful for:

* Logging
* Exceptions
* Refactor-safe code

---

---

### Q: What is the difference between `==` and `.Equals()`?

**A:**

| Operator    | Description                                       |
| ----------- | ------------------------------------------------- |
| `==`        | Compares **value or reference** depending on type |
| `.Equals()` | Can be overridden to define **custom equality**   |

```csharp
string a = "hello";
string b = new string("hello".ToCharArray());

Console.WriteLine(a == b);       // true
Console.WriteLine(a.Equals(b));  // true
```

In reference types, `==` may compare **references**, unless overridden (e.g., in `string`).

---

---

### Q: What is a static class?

**A:**

* A class that **cannot be instantiated**.
* Can only contain **static members**.
* Defined using the `static` keyword.

```csharp
public static class MathUtils
{
    public static int Square(int x) => x * x;
}
```

✅ Used for utility/helper methods.

---

---

### Q: What is null-coalescing (`??`) and null-conditional (`?.`)?

**A:**

### 🔹 Null-coalescing `??`

Returns the left-hand operand if not null; otherwise, returns the right.

```csharp
string name = userName ?? "Guest";
```

---

### 🔹 Null-conditional `?.`

Safely accesses a member or method **only if the object is not null**.

```csharp
int? length = name?.Length;
```

✅ Great for **null-safe chaining**.

---

---

### Q: What is tail recursion? Does C# optimize it?

**A:**

**Tail recursion** occurs when the **last operation** in a method is a call to itself.

```csharp
int Factorial(int n, int acc = 1)
{
    if (n == 0) return acc;
    return Factorial(n - 1, acc * n);
}
```

⚠️ C# **does not guarantee tail-call optimization**, even though the CLR supports it. Prefer loops in performance-critical code.

---

---

## Memory & Performance

### Q: How Many Generations Does the Garbage Collector Have in C#

**A:**

The .NET **Garbage Collector (GC)** uses **three generations** to manage memory efficiently:

---

## 🔹 Generations Overview

| Generation | Description                            | Typical Objects                       |
| ---------- | -------------------------------------- | ------------------------------------- |
| **Gen 0**  | Newly allocated, short-lived objects   | Local variables, temp strings         |
| **Gen 1**  | Objects that survived Gen 0 collection | Intermediate-lived objects            |
| **Gen 2**  | Long-lived, persistent objects         | Singletons, static references, caches |

---

## 🔹 Why Use Generations?

The generational model improves performance by:

* Collecting **short-lived objects** more frequently.
* Avoiding full memory scans unless necessary.
* Assuming **most objects die young**.

---

## 🔹 How Collection Works

1. **New objects** are allocated in **Gen 0**.
2. If they **survive** a GC, they are **promoted to Gen 1**.
3. If they survive again, they are **promoted to Gen 2**.
4. **Gen 2 collections** are the **most expensive**, so they are **infrequent**.

---

## 🔹 Example

```csharp
string name = "Albert"; // Allocated in Gen 0

GC.Collect(0); // Collects only Gen 0
GC.Collect(1); // Collects Gen 0 and Gen 1
GC.Collect(2); // Full GC (Gen 0, 1, and 2)
```

You can check the generation of an object:

```csharp
int generation = GC.GetGeneration(name);
Console.WriteLine($"Generation: {generation}");
```

---

## 🔸 Summary Table

| Generation | Purpose                          | Collection Frequency | Typical Lifetime   |
| ---------- | -------------------------------- | -------------------- | ------------------ |
| Gen 0      | Fast, frequent cleanup           | High                 | Very short         |
| Gen 1      | Buffer zone between 0 and 2      | Medium               | Short/intermediate |
| Gen 2      | Full collection, long-lived data | Low                  | Long-term          |

---

## ✅ When to Use This Knowledge

* To write **memory-efficient code**.
* To avoid retaining objects longer than necessary.
* When tuning performance in **high-throughput** or **low-latency** applications.

---

## 🧠 Tip

Avoid calling `GC.Collect()` manually unless absolutely necessary — let the runtime handle it. Focus on **releasing references** to objects so the GC can clean them up.

---

---

### Q: What Is the `Interlocked` Class Used For in C#

**A:**

The `Interlocked` class in C# provides **atomic operations** for variables shared across multiple threads. It ensures **thread-safe read-modify-write operations** without needing explicit `lock` blocks.

It belongs to the `System.Threading` namespace.

---

## 🔹 Purpose

* Prevent **race conditions** when updating shared variables.
* Avoid using heavier synchronization constructs like `lock`.
* Ensure **atomicity** in operations like incrementing, adding, exchanging, or comparing values.

---

## 🔹 Common Methods

| Method                                    | Description                                            |
| ----------------------------------------- | ------------------------------------------------------ |
| `Increment(ref int)`                      | Atomically increments a variable by 1                  |
| `Decrement(ref int)`                      | Atomically decrements a variable by 1                  |
| `Add(ref int, value)`                     | Atomically adds a value                                |
| `Exchange(ref int, value)`                | Sets a variable to a new value and returns the old one |
| `CompareExchange(ref int, new, expected)` | Atomically sets value **if** it matches the expected   |

---

## 🔹 Example: `Increment`

```csharp
int counter = 0;

Parallel.For(0, 1000, _ =>
{
    Interlocked.Increment(ref counter);
});

Console.WriteLine(counter); // Output: 1000
```

Without `Interlocked`, this would result in a **race condition**.

---

## 🔹 Example: `CompareExchange`

```csharp
int x = 10;
int expected = 10;
int newValue = 20;

int original = Interlocked.CompareExchange(ref x, newValue, expected);

Console.WriteLine(x);        // Output: 20
Console.WriteLine(original); // Output: 10
```

* `x` is updated to `20` **only if** its current value equals `expected` (`10`).
* If `x != expected`, no change is made.

---

## 🔸 Summary Table

| Feature          | `Interlocked`                       |
| ---------------- | ----------------------------------- |
| Thread-safe?     | ✅ Yes                               |
| Uses locks?      | ❌ No — lock-free                    |
| Performance      | ⚡ Very fast, low overhead           |
| Common use cases | Counters, flags, reference swapping |
| Namespace        | `System.Threading`                  |

---

## ✅ When to Use

* In **multi-threaded environments** where multiple threads read/write a shared value.
* When you need **simple atomic operations** without full locking.
* In **high-performance scenarios** where locking would introduce overhead.

---

## ⚠️ Limitations

* Only works with **primitive types** (`int`, `long`, `float`, etc.) and **object references**.
* Not suitable for complex critical sections — use `lock` for those.

---

## 🧠 Tip

Use `Interlocked` when you want **lock-free synchronization** for basic variable updates in concurrent code.

---

---

### Q: What is a Span<T>?

**A:**

`Span<T>` is a **stack-only** type that provides a **type-safe and memory-safe view** over arrays, slices of arrays, or unmanaged memory.

```csharp
Span<int> slice = new int[] { 1, 2, 3, 4 }.AsSpan(1, 2);
foreach (var item in slice)
    Console.WriteLine(item); // 2, 3
```

✅ Improves performance by avoiding allocations.

⚠️ Only usable in methods that do **not escape to the heap**.

---

---

## Reflection & Type Inspection

### Q: What is reflection?

**A:**

Reflection allows inspection of metadata and runtime types.

```csharp
Type type = typeof(string);
MethodInfo method = type.GetMethod("Contains");

bool result = (bool)method.Invoke("hello", new object[] { "ell" });
Console.WriteLine(result); // true
```

✅ Powerful for dynamic loading, serialization, and dependency injection.
⚠️ Slower and less type-safe — use wisely.

---

---

### Q: What is the difference between `is`, `as`, and `typeof`?

**A:**

| Keyword  | Purpose                           | Example                    |
| -------- | --------------------------------- | -------------------------- |
| `is`     | Type check                        | `if (obj is MyClass)`      |
| `as`     | Safe cast (returns null if fails) | `var x = obj as MyClass`   |
| `typeof` | Get `System.Type`                 | `Type t = typeof(MyClass)` |

---

---

## Delegates & Events

### Q: What Is the Difference Between a Delegate and an Event in C#?

**A:**

A **delegate** is a type-safe **function pointer** — it holds a reference to one or more methods with a matching signature. An **event** is a **restricted wrapper** around a delegate that enforces an encapsulation boundary: outside the declaring class, consumers can only **subscribe** (`+=`) or **unsubscribe** (`-=`), but they cannot **invoke** or **replace** the delegate directly.

---

## 🔹 Delegate

### Definition:

* A **type** that defines a method signature and can hold references to one or more matching methods.
* Can be **invoked**, **reassigned**, or **combined** freely by any code that has access to it.

### Example:

```csharp
public delegate void Notify(string message);

public class Broadcaster
{
    public Notify OnMessage;
}

var broadcaster = new Broadcaster();

// Any external code can invoke or replace the delegate
broadcaster.OnMessage = msg => Console.WriteLine($"First: {msg}");
broadcaster.OnMessage += msg => Console.WriteLine($"Second: {msg}");

// External code can invoke directly
broadcaster.OnMessage("Hello");

// External code can replace ALL subscribers
broadcaster.OnMessage = null;
```

---

## 🔹 Event

### Definition:

* An **event** wraps a delegate and restricts external access to `+=` and `-=` only.
* Only the **declaring class** can invoke the event or set it to `null`.

### Example:

```csharp
public delegate void Notify(string message);

public class Broadcaster
{
    public event Notify OnMessage;

    public void Send(string message)
    {
        OnMessage?.Invoke(message); // Only the class itself can invoke
    }
}

var broadcaster = new Broadcaster();

// External code can only subscribe and unsubscribe
broadcaster.OnMessage += msg => Console.WriteLine($"Received: {msg}");

// broadcaster.OnMessage("Hello");   // ❌ Compile error — cannot invoke from outside
// broadcaster.OnMessage = null;     // ❌ Compile error — cannot reassign from outside

broadcaster.Send("Hello"); // ✅ Invoked internally
```

---

## 🔹 Custom Event Accessors

You can define custom `add` and `remove` accessors for events, similar to property getters and setters:

```csharp
public class Broadcaster
{
    private Notify _onMessage;

    public event Notify OnMessage
    {
        add    { _onMessage += value; Console.WriteLine("Subscriber added"); }
        remove { _onMessage -= value; Console.WriteLine("Subscriber removed"); }
    }

    public void Send(string message) => _onMessage?.Invoke(message);
}
```

---

## 🔸 Summary Table

| Feature                  | Delegate                          | Event                                  |
| ------------------------ | --------------------------------- | -------------------------------------- |
| Type                     | Type-safe function pointer        | Restricted delegate wrapper            |
| External invocation      | ✅ Allowed                         | ❌ Only the declaring class can invoke  |
| External reassignment    | ✅ Can set to `null` or `=`        | ❌ Only `+=` and `-=` from outside      |
| Multicast support        | ✅ Yes                             | ✅ Yes                                  |
| Encapsulation            | ❌ No restrictions                 | ✅ Enforced                             |
| Common use case          | Callbacks, strategy pattern       | Publisher/subscriber, UI notifications |

---

## ✅ When to Use

* Use a **delegate** when you need a simple **callback** or want to pass a method reference freely.
* Use an **event** when implementing the **publisher/subscriber pattern** and you want to prevent external code from invoking or replacing the handler list.

---

---

### Q: What Are `Func`, `Action`, and `Predicate` Delegates in C#?

**A:**

C# provides **built-in generic delegate types** so you rarely need to declare custom delegates. The three most common are `Func<>`, `Action<>`, and `Predicate<>`.

---

## 🔹 `Func<T, TResult>`

### Definition:

* A delegate that takes **zero or more input parameters** and **returns a value**.
* The **last** type parameter is always the **return type**.

### Example:

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4)); // Output: 7

Func<string, int> getLength = s => s.Length;
Console.WriteLine(getLength("Hello")); // Output: 5

// No input parameters
Func<DateTime> now = () => DateTime.Now;
Console.WriteLine(now());
```

---

## 🔹 `Action<T>`

### Definition:

* A delegate that takes **zero or more input parameters** and returns **`void`**.

### Example:

```csharp
Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
greet("Alice"); // Output: Hello, Alice!

Action<int, int> printSum = (a, b) => Console.WriteLine(a + b);
printSum(3, 4); // Output: 7

// No parameters
Action logTime = () => Console.WriteLine(DateTime.Now);
logTime();
```

---

## 🔹 `Predicate<T>`

### Definition:

* A delegate that takes a **single input parameter** and returns **`bool`**.
* Functionally equivalent to `Func<T, bool>`, but more **semantically clear** for filtering.

### Example:

```csharp
Predicate<int> isEven = n => n % 2 == 0;
Console.WriteLine(isEven(4)); // Output: True
Console.WriteLine(isEven(7)); // Output: False

var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };
List<int> evens = numbers.FindAll(isEven);
// evens: [2, 4, 6]
```

---

## 🔸 Summary Table

| Delegate         | Parameters         | Returns     | Common Use Case                     |
| ---------------- | ------------------ | ----------- | ----------------------------------- |
| `Func<T, ...>`   | 0–16 input params  | `TResult`   | Transformations, computations, LINQ |
| `Action<T, ...>` | 0–16 input params  | `void`      | Side effects, logging, printing     |
| `Predicate<T>`   | Exactly 1 param    | `bool`      | Filtering, validation               |

---

## ✅ When to Use

* Use **`Func`** when you need a delegate that **returns a value** (e.g., `Select()`, `Where()` in LINQ).
* Use **`Action`** when you need a delegate that performs a **side effect** without returning anything.
* Use **`Predicate`** when filtering items in a collection or expressing a **boolean condition** (e.g., `List<T>.FindAll()`).
* Prefer the built-in delegates over custom delegate types unless you need a **named, self-documenting** delegate signature.

---

---

## Tuples & Deconstruction

### Q: What Are Tuples in C# and When Should You Use Them?

**A:**

A **tuple** is a lightweight data structure that groups **multiple values** into a single object without requiring you to define a class or struct. C# supports two kinds: the older `System.Tuple` (reference type) and the modern `System.ValueTuple` (value type, introduced in C# 7).

---

## 🔹 `ValueTuple` (C# 7+, Recommended)

### Definition:

* A **value type** (stored on the stack when local).
* Supports **named elements** and **deconstruction**.
* Created using the `( )` syntax.

### Example:

```csharp
// Named tuple
(string Name, int Age) person = ("Alice", 30);
Console.WriteLine(person.Name); // Alice
Console.WriteLine(person.Age);  // 30

// Method returning a named tuple
(double Lat, double Lon) GetCoordinates()
{
    return (47.6062, -122.3321);
}

var coords = GetCoordinates();
Console.WriteLine($"{coords.Lat}, {coords.Lon}");
```

---

## 🔹 `Tuple` (Legacy)

### Definition:

* A **reference type** (allocated on the heap).
* Elements accessed via `Item1`, `Item2`, etc. — **no named members**.
* Less ergonomic and largely replaced by `ValueTuple`.

### Example:

```csharp
Tuple<string, int> person = Tuple.Create("Alice", 30);
Console.WriteLine(person.Item1); // Alice
Console.WriteLine(person.Item2); // 30
```

---

## 🔹 Named vs Unnamed Tuples

```csharp
// Unnamed — fields are Item1, Item2
var unnamed = (10, "hello");
Console.WriteLine(unnamed.Item1); // 10

// Named — fields have meaningful names
var named = (Id: 10, Message: "hello");
Console.WriteLine(named.Id); // 10
```

> **Note:** Named element names are a **compile-time convenience** — they are not preserved at runtime via reflection.

---

## 🔹 Deconstruction

Tuples can be **deconstructed** into individual variables:

```csharp
(string name, int age) = ("Alice", 30);
Console.WriteLine(name); // Alice

// Discard values you don't need
(_, int ageOnly) = ("Alice", 30);

// Deconstruct from a method
var (lat, lon) = GetCoordinates();
```

You can also add deconstruction to your own types by implementing a `Deconstruct` method:

```csharp
public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public void Deconstruct(out int x, out int y)
    {
        x = X;
        y = Y;
    }
}

var (px, py) = new Point(3, 4);
```

---

## 🔸 Summary Table

| Feature             | `ValueTuple` (C# 7+)          | `Tuple` (Legacy)             |
| ------------------- | ------------------------------ | ---------------------------- |
| Type                | Value type (`struct`)          | Reference type (`class`)     |
| Named elements      | ✅ Yes                          | ❌ No (`Item1`, `Item2`)      |
| Deconstruction      | ✅ Yes                          | ❌ No                         |
| Performance         | ✅ Stack-allocated (when local) | ❌ Heap-allocated              |
| Mutable elements    | ✅ Yes                          | ❌ Read-only properties        |
| Syntax              | `(int, string)`               | `Tuple<int, string>`         |

---

## ✅ When to Use

* Use tuples for **returning multiple values** from a method without creating a dedicated type.
* Use **named tuples** to improve readability when the meaning of elements isn't obvious.
* **Prefer a `record` or `class`** when the data has behavior, needs validation, or is used across many parts of the codebase.
* Avoid tuples in **public APIs** of libraries — named types are more maintainable and self-documenting.

---

---

## Dynamic & Reflection

### Q: What Is the `dynamic` Keyword in C# and How Does It Differ from `var` and `object`?

**A:**

C# provides three ways to work with values whose type may not be explicitly stated: `var`, `object`, and `dynamic`. While they may look similar at a glance, they differ fundamentally in **when** type resolution happens and **what safety guarantees** they provide.

---

## 🔹 `var` — Compile-Time Type Inference

### Definition:

* The compiler **infers the type** from the right-hand side of the assignment.
* The variable is **strongly typed** — the type is known at compile time.
* You get full **IntelliSense** and compile-time checks.

### Example:

```csharp
var number = 42;         // Inferred as int
var name = "Alice";      // Inferred as string

// number = "text";      // ❌ Compile error: cannot convert string to int
Console.WriteLine(name.Length); // ✅ IntelliSense works
```

---

## 🔹 `object` — Base Type with Casting

### Definition:

* The **base type** for all types in C# (`System.Object`).
* Any value can be assigned, but you must **cast** to access type-specific members.
* Checked at **compile time** after casting.

### Example:

```csharp
object value = 42;

// Console.WriteLine(value.Length); // ❌ Compile error: object has no Length
int number = (int)value;            // ✅ Explicit cast required
Console.WriteLine(number);          // 42

value = "Hello";
string text = (string)value;
Console.WriteLine(text.Length);      // 5
```

---

## 🔹 `dynamic` — Runtime Type Resolution (DLR)

### Definition:

* Type checking is **deferred entirely to runtime** using the **Dynamic Language Runtime (DLR)**.
* No IntelliSense, no compile-time type checking.
* If a member doesn't exist, it throws a `RuntimeBinderException` at runtime.

### Example:

```csharp
dynamic value = 42;
Console.WriteLine(value + 10);  // ✅ Works at runtime: 52

value = "Hello";
Console.WriteLine(value.Length); // ✅ Works at runtime: 5

value = 42;
// Console.WriteLine(value.Length); // ❌ RuntimeBinderException at runtime
```

---

## 🔹 Performance Implications

* `var` — **zero overhead**, same as specifying the type directly.
* `object` — may involve **boxing** for value types and requires casting.
* `dynamic` — **significant overhead** due to DLR dispatch, caching, and reflection-like resolution. Avoid in hot paths.

---

## 🔹 Use Cases for `dynamic`

* **COM interop** — working with Office Automation APIs without strongly-typed wrappers.
* **Working with JSON** — accessing properties on deserialized JSON objects (e.g., `ExpandoObject`).
* **Interop with dynamic languages** — IronPython, IronRuby.
* **Avoiding excessive casting** in code that works with many unknown types at runtime.

### Example: Working with ExpandoObject

```csharp
dynamic expando = new System.Dynamic.ExpandoObject();
expando.Name = "Alice";
expando.Age = 30;

Console.WriteLine($"{expando.Name} is {expando.Age}"); // Alice is 30
```

---

## 🔸 Summary Table

| Feature                 | `var`                      | `object`                   | `dynamic`                    |
| ----------------------- | -------------------------- | -------------------------- | ---------------------------- |
| Type resolution         | Compile time               | Compile time               | Runtime (DLR)                |
| Type safety             | ✅ Full                     | ✅ After cast                | ❌ None at compile time       |
| IntelliSense            | ✅ Yes                      | ❌ Only `object` members     | ❌ No                         |
| Boxing for value types  | No                         | ✅ Yes                       | ✅ Yes                        |
| Performance             | ✅ Best                     | ⚠️ Casting overhead         | ❌ DLR overhead               |
| Can change type?        | ❌ No                       | ✅ Yes (via reassignment)    | ✅ Yes                        |
| Must initialize?        | ✅ Yes                      | No                         | No                           |

---

## ✅ When to Use

* Use **`var`** when the type is obvious from the right-hand side and you want cleaner code.
* Use **`object`** when you need a common base type for heterogeneous collections and will cast explicitly.
* Use **`dynamic`** only when compile-time type information is genuinely unavailable (COM interop, JSON, dynamic language interop).
* **Avoid `dynamic`** in performance-critical paths and when compile-time safety is important.

---

---

## Pattern Matching

### Q: What Is Pattern Matching in C# and What Patterns Are Available?

**A:**

**Pattern matching** is a feature in C# that lets you test a value against a **shape** or **condition** and extract information from it. It has evolved significantly across C# versions, from basic type checks to rich, composable patterns.

---

## 🔹 Type Patterns

Test whether a value is of a specific type and optionally bind it to a variable:

```csharp
object obj = "Hello";

if (obj is string s)
{
    Console.WriteLine(s.Length); // 5
}

// In a switch expression
string Describe(object value) => value switch
{
    int i    => $"Integer: {i}",
    string s => $"String of length {s.Length}",
    null     => "null",
    _        => $"Other: {value.GetType().Name}"
};
```

---

## 🔹 Property Patterns

Match on the **properties** of an object:

```csharp
public record Address(string City, string Country);

string GetShippingZone(Address addr) => addr switch
{
    { Country: "US" }              => "Domestic",
    { Country: "CA" }              => "North America",
    { Country: "GB", City: "London" } => "London Express",
    { Country: "GB" }              => "Europe",
    _                              => "International"
};
```

### Nested Property Patterns:

```csharp
public record Order(Address ShippingAddress, decimal Total);

string Classify(Order order) => order switch
{
    { ShippingAddress.Country: "US", Total: > 100 } => "US priority",
    { ShippingAddress.Country: "US" }               => "US standard",
    _                                               => "International"
};
```

---

## 🔹 Positional Patterns

Match on deconstructed values (works with types that have a `Deconstruct` method or positional records):

```csharp
public record Point(int X, int Y);

string Quadrant(Point p) => p switch
{
    (0, 0)       => "Origin",
    (> 0, > 0)   => "Q1",
    (< 0, > 0)   => "Q2",
    (< 0, < 0)   => "Q3",
    (> 0, < 0)   => "Q4",
    _            => "On axis"
};
```

---

## 🔹 Relational Patterns (C# 9)

Use relational operators (`<`, `>`, `<=`, `>=`) directly in patterns:

```csharp
string Classify(int temperature) => temperature switch
{
    < 0    => "Freezing",
    >= 0 and < 15 => "Cold",
    >= 15 and < 25 => "Mild",
    >= 25 and < 35 => "Warm",
    >= 35  => "Hot"
};
```

---

## 🔹 Logical Patterns: `and`, `or`, `not` (C# 9)

Combine patterns with logical combinators:

```csharp
// and
if (obj is int n and > 0 and < 100)
    Console.WriteLine($"Positive two-digit or less: {n}");

// or
if (obj is string or int)
    Console.WriteLine("String or int");

// not
if (obj is not null)
    Console.WriteLine("Not null");

// Combined in a switch
string ClassifyChar(char c) => c switch
{
    >= 'a' and <= 'z' => "lowercase",
    >= 'A' and <= 'Z' => "uppercase",
    >= '0' and <= '9' => "digit",
    _                 => "other"
};
```

---

## 🔹 List Patterns (C# 11)

Match against the **elements** of an array or list:

```csharp
int[] numbers = { 1, 2, 3 };

var result = numbers switch
{
    [1, 2, 3]       => "Exact match: 1, 2, 3",
    [1, ..]         => "Starts with 1",
    [.., 3]         => "Ends with 3",
    [_, 2, _]       => "Middle element is 2",
    { Length: 0 }   => "Empty",
    _               => "Other"
};

// Slice pattern with variable binding
if (numbers is [var first, .. var rest])
{
    Console.WriteLine($"First: {first}, Rest count: {rest.Length}");
}
```

---

## 🔹 `is` Expressions vs `switch` Expressions

```csharp
// is expression — good for single checks
if (shape is Circle { Radius: > 10 } bigCircle)
    Console.WriteLine($"Big circle with radius {bigCircle.Radius}");

// switch expression — good for multiple cases
string Describe(Shape shape) => shape switch
{
    Circle { Radius: 0 }            => "Point",
    Circle { Radius: var r }        => $"Circle (r={r})",
    Rectangle { Width: var w, Height: var h } when w == h => $"Square ({w})",
    Rectangle { Width: var w, Height: var h } => $"Rectangle ({w}x{h})",
    _                               => "Unknown shape"
};
```

---

## 🔸 Summary Table

| Pattern             | C# Version | Description                              | Example                          |
| ------------------- | ----------- | ---------------------------------------- | -------------------------------- |
| Type                | 7.0         | Check and cast type                      | `obj is string s`                |
| Constant            | 7.0         | Match a constant value                   | `x is 42`                        |
| `var`               | 7.0         | Always matches, binds to variable        | `x is var v`                     |
| Property            | 8.0         | Match property values                    | `{ Name: "Alice" }`             |
| Positional          | 8.0         | Match deconstructed values               | `(> 0, > 0)`                    |
| Switch expression   | 8.0         | Expression-based exhaustive matching     | `x switch { ... }`              |
| Relational          | 9.0         | Compare with `<`, `>`, `<=`, `>=`        | `> 0 and < 100`                 |
| Logical             | 9.0         | Combine with `and`, `or`, `not`          | `not null`                       |
| List                | 11.0        | Match array/list elements and slices     | `[1, .., 3]`                     |

---

## ✅ When to Use

* Use **type patterns** for safe casting and polymorphic dispatch without `is`/`as` chains.
* Use **property patterns** to match on object state in a concise, readable way.
* Use **switch expressions** when you have multiple mutually exclusive conditions — they enforce exhaustiveness.
* Use **relational and logical patterns** to replace complex `if`/`else` chains with clear, declarative conditions.
* Use **list patterns** (C# 11) for sequence matching, protocol parsing, or command-line argument processing.

---

---
