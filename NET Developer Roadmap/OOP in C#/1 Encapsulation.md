# 1. Encapsulation

# Definition

**Encapsulation** allows you to **hide internal object details** and **control how data is accessed or modified**. This is achieved through **access modifiers** in C#, which define the **visibility** of class members.

# **Access Modifiers in C#**

| Modifier | Meaning | Accessible From | Typical Use Case |
| --- | --- | --- | --- |
| `private` | Accessible **only within the same class** | Same class | Hide internal data (fields, helper methods) |
| `public` | Accessible **from anywhere** | All classes | Expose APIs, service methods, or reusable components |
| `protected` | Accessible **within the class and its subclasses** | Class and derived classes | Share logic with subclasses without exposing it to others |
| `internal` | Accessible **within the same project/assembly** | Same assembly | Share internally across components in the same system |
| `protected internal` | Accessible **within the assembly or in derived classes** | Subclasses or same assembly | Rare – used when limited but shared access is needed |
| `private protected` | Accessible **only within the declaring class or subclasses in the same assembly** | Subclass + same assembly | Most restricted combination |
| `sealed` | Prevents the class from being inherited | – | Prevent further extension – used for security or fixed logic |

# Code Examples

## a.`private`

```csharp

public class BankAccount
{
    private decimal balance; // Hidden from outside access

    public void Deposit(decimal amount)
    {
        if (amount > 0) balance += amount;
    }

```

## b. `public`

```csharp

public class Car
{
    public string Brand; // Accessible and modifiable from anywhere

    public void Start() => Console.WriteLine("Car is starting");
}
```

## c. `protected`

```csharp

public class Animal
{
    protected void Breathe() => Console.WriteLine("Animal is breathing");
}

public class Dog : Animal
{
    public void Bark()
    {
        Breathe(); // Allowed since Dog inherits from Animal
        Console.WriteLine("Dog is barking");
    }

```

## d. `sealed`

```csharp

public sealed class PaymentProcessor
{
    public void Process() => Console.WriteLine("Processing payment...");
}

// This is NOT allowed:
// public class CustomProcessor : PaymentProcessor {} ❌
```

# When to Use Each Modifier?

| Scenario | Recommended Modifier |
| --- | --- |
| You want to **hide a field or method** from outside access | `private` |
| You need to **expose a method or property** for public use | `public` |
| You want to **share logic only with subclasses** | `protected` |
| You want to **limit access to within the project** | `internal` |
| You want to **restrict further inheritance** | `sealed` |

# Encapsulation Best Practices

- **Use `private` fields** with **`public` properties** (get/set) for controlled access.
- **Keep class internals hidden** unless there's a clear reason to expose them.
- **Use `sealed`** to lock down business-critical or sensitive classes.