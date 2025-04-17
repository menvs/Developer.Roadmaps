# 2. Inheritance

# 📘 Definition:

**Inheritance** allows a class (called a **derived class** or **child class**) to inherit members (fields, properties, methods) from another class (called a **base class** or **parent class**).

It promotes **code reuse**, **hierarchical classification**, and supports **polymorphism**.

# 📚 Real-World Analogy:

Think of a **“Vehicle”** base class. Every vehicle has basic properties like `Speed`, `Start()`, `Stop()`. Then you have derived classes like `Car`, `Truck`, and `Motorbike` that inherit from `Vehicle` but also add their own features.

# ✅ Basic Example:

```csharp
public class Vehicle
{
    public void Start() => Console.WriteLine("Vehicle started");
}

public class Car : Vehicle
{
    public void Honk() => Console.WriteLine("Car honks");
}
```

**Usage:**

```csharp
Car myCar = new Car();
myCar.Start(); // Inherited from Vehicle
myCar.Honk();  // Specific to Car
```

# 🔍 Key Concepts to Know

| Concept | Explanation |
| --- | --- |
| `:` operator | Used to declare inheritance in C#: `class Dog : Animal` |
| Single Inheritance | C# supports only **single inheritance** for classes |
| Constructor Chaining | Derived classes can call base constructors using `: base()` |
| `virtual` / `override` | Used for polymorphism (next topic) |

# ⚠️ Best Practices & Notes for Developers

| ✅ Do’s | ❌ Don’ts |
| --- | --- |
| Use inheritance to model **"is-a"** relationships | Don’t inherit just to reuse code — use composition instead |
| Make base class members **`protected`** or `public` only when needed | Don’t expose internal logic unnecessarily |
| Keep base classes **generic and reusable** | Avoid deep inheritance chains (more than 2-3 levels) |
| Consider using **abstract classes** or **interfaces** for shared behavior | Avoid putting too much logic in base classes |