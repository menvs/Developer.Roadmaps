# 4. Abstraction

# 📘 Definition

**Abstraction** means **exposing only the relevant features** of an object while hiding the internal implementation details.

Think of it as creating a **simplified interface** to interact with complex systems.

---

# 🎯 Real-world Analogy:

When you **drive a car**, you interact with the **steering wheel, pedals, and dashboard** — you don’t need to know how the **engine**, **transmission**, or **electronics** work internally. That’s **abstraction**.

---

# 🧪 How It's Done in C#

C# supports **abstraction** through:

| Mechanism | Description |
| --- | --- |
| **Abstract classes** | Define a base with common structure but leave implementation to subclasses |
| **Interfaces** | Define only contracts — implementations are provided by implementing classes |
| **Access modifiers** | Control visibility to hide implementation details (`private`, `internal`, etc.) |

## 🧩 **Abstract Class vs Interface in C#**

Both **abstract classes** and **interfaces** are used to define **contracts** or **shared behavior**, but they have key differences in terms of **purpose, usage, and flexibility**.

### 📊 Comparison Table

| Feature | **Abstract Class** | **Interface** |
| --- | --- | --- |
| **Purpose** | Provides a **partial implementation** of behavior | Provides a **contract** that must be implemented entirely |
| **Members** | Can contain **fields, constructors, methods (with or without body)** | Can contain only **methods, properties, events, indexers** (no fields) |
| **Multiple Inheritance** | ❌ Not supported (only one base class allowed) | ✅ Supports multiple interface inheritance |
| **Access Modifiers** | Members can be `public`, `protected`, etc. | All members are `public` by default |
| **Use When** | Classes are **closely related** and share base logic | Unrelated classes need to **follow the same contract** |
| **Constructor Support** | ✅ Yes | ❌ No |
| **Default Implementation** | ✅ Yes (C# 8.0+ allows default methods) | ✅ Yes (C# 8.0+ allows default methods with body) |
| **Inheritance Keyword** | `: BaseClass` | `: IInterfaceName` |
| **Example Use Case** | Animal → Dog, Cat (common logic & structure) | IPrintable, IDisposable, IComparable |

### 🎯 When to Use What?

| If you need... | Use |
| --- | --- |
| A base class with shared logic | ✅ **Abstract Class** |
| A contract for unrelated types | ✅ **Interface** |
| To inherit from multiple sources | ✅ **Interface** |
| To provide base functionality with enforced methods | ✅ **Abstract Class** |
| Lightweight contracts without implementation | ✅ **Interface** |

### ✅ Example: Using an Interface for Abstraction

```csharp

public interface IDatabase
{
    void Connect();
    void Disconnect();
}

public class SqlDatabase : IDatabase
{
    public void Connect() => Console.WriteLine("Connected to SQL DB");
    public void Disconnect() => Console.WriteLine("Disconnected");
}
```

```csharp

public class DataService
{
    private readonly IDatabase _database;

    public DataService(IDatabase database)
    {
        _database = database;
    }

    public void UseDatabase()
    {
        _database.Connect();
        // Do work...
        _database.Disconnect();
    }
}
```

➡️ `DataService` doesn’t care *how* `SqlDatabase` connects — just that it **can**. That’s abstraction in action.

---

# ⚠️ Best Practices

| ✅ Do’s | ❌ Don’ts |
| --- | --- |
| Use interfaces/abstract classes to define contracts | Don’t expose internal methods publicly |
| Keep APIs clean and minimal | Don’t expose unnecessary properties or methods |
| Separate **what the class does** from **how it does it** | Don’t tightly couple services to specific implementations |
| Combine abstraction with **dependency injection** for clean architecture | Avoid writing large classes with mixed responsibilities |

---

# 🧠 Summary

- Abstraction focuses on **what** an object does, not **how**
- It simplifies code usage, improves flexibility, and enhances security
- Achieved through **interfaces**, **abstract classes**, and **access control**