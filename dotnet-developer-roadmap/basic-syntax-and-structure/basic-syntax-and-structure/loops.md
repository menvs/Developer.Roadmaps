# Loops

Loops are used to **execute a block of code repeatedly** based on a condition or collection.

---

## 📋 Comparison Table

| Loop Type | Description | Use Case | Example |
| --- | --- | --- | --- |
| `for` | Loop with counter | When you know how many times to loop | `for (int i = 0; i < 5; i++)` |
| `while` | Loop while condition is true | When condition is checked **before** | `while (x < 5)` |
| `do-while` | Loop at least once, condition checked after | Ensure the loop runs **at least once** | `do { ... } while (x < 5);` |
| `foreach` | Iterates through a collection | Loop through arrays, lists, etc. | `foreach (var item in list)` |

## 1. `for` Loop

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);
}
```

✅ Best when you **know how many times** the loop should run.

---

## 2. `while` Loop

```csharp
int i = 0;
while (i < 5)
{
    Console.WriteLine(i);
    i++;
}
```

✅ Checks the condition **before** running the loop body.

---

## 3. `do-while` Loop

```csharp
int i = 0;
do
{
    Console.WriteLine(i);
    i++;
} while (i < 5);
```

✅ Runs the body **at least once**, even if the condition is false.

---

## 4. `foreach` Loop

```csharp
string[] names = { "Alice", "Bob", "Charlie" };

foreach (string name in names)
{
    Console.WriteLine(name);
}
```

✅ Best for **iterating over collections**.

💡 Read-only — you **can't modify** the collection inside a `foreach` directly.

---

## Tips & Best Practices

- Use `for` if you need an index or control the loop manually.
- Use `foreach` for readability when looping over items.
- Avoid modifying a collection inside a `foreach`.
- Avoid infinite loops like `while (true)` unless you control exit well.
- Always consider edge cases (e.g., empty collections or boundary conditions).

## Keywords You Can Use in Loops

| Keyword | Description | Example |
| --- | --- | --- |
| `break` | Exit the loop immediately | `if (x == 5) break;` |
| `continue` | Skip the current iteration and move to next | `if (x % 2 == 0) continue;` |
| `return` | Exit the loop and the entire method | `if (x > 100) return;` |

# 🚀 Loop Performance Tips in C#

Understanding how loops affect performance is important for writing high-quality C# code — especially when working with big data, UI rendering, or real-time systems.

---

## ⚠️ Key Performance Considerations

### 1. Avoid Unnecessary Work Inside Loops

Do **not** call methods or perform complex calculations inside the loop if you can move them out.

❌ Bad:

```csharp
for (int i = 0; i < list.Count; i++)
{
    DoSomething(GetDataFromDatabase()); // Expensive!
}
```

✅ Good:

```csharp
var data = GetDataFromDatabase();
for (int i = 0; i < list.Count; i++)
{
    DoSomething(data);
}
```