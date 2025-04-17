# 3. Polymorphism

# 📘 Definition

**Polymorphism** allows objects of **different types** to be treated as objects of a **common base type**, and for **the same method to behave differently** based on the object that invokes it.

It helps you write **extensible, flexible, and maintainable code**.

---

# 🧠 Types of Polymorphism in C#

| Type | Description | Keyword |
| --- | --- | --- |
| **Compile-time (Static)** | Method Overloading – same method name, different parameters | `Method()` / `Method(int)` |
| **Run-time (Dynamic)** | Method Overriding – derived class overrides base class logic | `virtual`, `override` |

### 1. **Compile-time Polymorphism (Method Overloading)**

```csharp
public class Printer
{
    public void Print(string text) => Console.WriteLine(text);
    public void Print(int number) => Console.WriteLine(number);
}
```

**Usage:**

```csharp

Printer p = new Printer();
p.Print("Hello");   // Calls Print(string)
p.Print(123);       // Calls Print(int)
```

➡️ The method name is the same, but the **parameters are different**.

### 2. **Run-time Polymorphism (Method Overriding)**

```csharp

public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal sound");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof!");
}

public class Cat : Animal
{
    public override void Speak() => Console.WriteLine("Meow!");
}

```

**Usage:**

```csharp

Animal a1 = new Dog();
Animal a2 = new Cat();

a1.Speak();  // Output: Woof!
a2.Speak();  // Output: Meow!
```

➡️ Same method name `Speak()` is called, but the behavior **depends on the actual object type** at runtime.

---

# 🎯 Why Polymorphism Is Powerful

- Allows writing **general-purpose code** that works with many types
- Supports **open/closed principle** (open for extension, closed for modification)
- Essential for frameworks, plugins, and enterprise architecture

---

# 🎨 **Polymorphism via Inheritance**

```csharp

            Animal (Base Class)
              + virtual void Speak()
                    |
   --------------------------------------
   |                                    |
Dog (Override)                   Cat (Override)
+ override Speak()           + override Speak()

           ↓                              ↓
   Animal a1 = new Dog();         Animal a2 = new Cat();
        a1.Speak();                   a2.Speak();
      ➜ "Woof!"                     ➜ "Meow!"

```

# 🎯 **Polymorphism via Interface**

```csharp

         IShape (Interface)
         + double GetArea()

     /-----------------------\
Circle                  Rectangle
+ GetArea()             + GetArea()

         ↓                         ↓
List<IShape> shapes = new List<IShape>();
shapes.Add(new Circle());
shapes.Add(new Rectangle());

foreach (var shape in shapes)
    shape.GetArea(); // Polymorphic call

```

➡️ All shapes implement the same method, but each class provides its **unique implementation**.

---

# 🛒 **Real-World Analogy: Payment System**

Imagine an e-commerce system:

## Interface: `IPaymentMethod`

```csharp

public interface IPaymentMethod
{
    void Pay(decimal amount);
}

```

## Implementations:

```csharp

public class CreditCard : IPaymentMethod
{
    public void Pay(decimal amount) => Console.WriteLine($"Paid {amount} with Credit Card.");
}

public class PayPal : IPaymentMethod
{
    public void Pay(decimal amount) => Console.WriteLine($"Paid {amount} via PayPal.");
}

```

## Usage:

```csharp

List<IPaymentMethod> methods = new List<IPaymentMethod>
{
    new CreditCard(),
    new PayPal()
};

foreach (var method in methods)
{
    method.Pay(100); // Same method, different behavior
}

```

---

# 🧠 Best Practices

| ✅ Do’s | ❌ Don’ts |
| --- | --- |
| Use `virtual` and `override` intentionally | Don’t override methods unnecessarily |
| Use base types or interfaces to enable polymorphism | Don’t rely only on concrete types |
| Use `abstract` or `interface` when method must be implemented | Don’t forget to make base method `virtual` or `abstract` |
| Follow **Liskov Substitution Principle** | Don’t break expected behavior when overriding methods |

---

# 📦 Summary

- **Polymorphism** = One interface, many behaviors
- Enables you to treat **different types as the same base type**
- Makes your code **scalable**, **extensible**, and **clean**
- Works through **inheritance (virtual/override)** and **interfaces (shared contracts)**