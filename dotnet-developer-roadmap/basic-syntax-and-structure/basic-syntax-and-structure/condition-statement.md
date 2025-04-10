# Condition Statements

Conditional statements control **flow of execution** based on conditions (boolean expressions).

---

## 📋 Comparison Table

| Statement | Description | Syntax Example | Use Case |
| --- | --- | --- | --- |
| `if` | Executes block if condition is true | `if (x > 10) { ... }` | Run code only when a condition is met |
| `if-else` | One block if true, another if false | `if (x > 10) { ... } else { ... }` | Choose between two paths |
| `else if` | Test multiple conditions | `if (x > 10) { ... } else if (...)` | Chain multiple checks |
| `switch` | Multi-way branching | `switch (value) { case 1: ... }` | Clean alternative to many `else if` |
| Ternary (`?:`) | Short-hand for `if-else` | `var result = x > 10 ? "A" : "B";` | Assign based on condition |

## 1. `if` Statement

```csharp
if (x > 0)
{
    Console.WriteLine("Positive number");
}
```

✅ Executes block **only** when the condition is `true`.

---

## 2. `if-else` Statement

```csharp
if (x % 2 == 0)
{
    Console.WriteLine("Even");
}
else
{
    Console.WriteLine("Odd");
}
```

✅ Runs one of two blocks depending on the condition.

---

## 3. `else if` Ladder

```csharp
if (score >= 90)
{
    grade = "A";
}
else if (score >= 80)
{
    grade = "B";
}
else
{
    grade = "C";
}
```

✅ Use when checking **multiple mutually exclusive conditions**.

---

## 4. `switch` Statement

```csharp
switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    default:
        Console.WriteLine("Other day");
        break;
}
```

✅ Better readability for **multiple values** of the same variable.

📌 C# 8.0+ also supports **switch expressions**:

```csharp
string result = day switch
{
    1 => "Monday",
    2 => "Tuesday",
    _ => "Other"
};
```

---

## 5. Ternary Operator (`?:`)

```csharp
string message = (age >= 18) ? "Adult" : "Minor";
```

✅ Shorthand for simple `if-else` assignments.

---

## 🧠 Tips & Best Practices

- Always prefer `switch` over long `else-if` chains for value comparisons.
- Don't forget `break;` in `switch` cases.
- Use `{}` even for single-line `if` blocks to avoid bugs.
- Use ternary only when it improves readability.
- Avoid deeply nested `if-else` for better maintainability.