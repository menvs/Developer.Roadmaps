# Operators

## 🧮 Arithmetic Operators

| Operator | Description | Example | Result |
| --- | --- | --- | --- |
| `+` | Addition | `5 + 2` | `7` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `5 * 2` | `10` |
| `/` | Division | `5 / 2` | `2` (integer) |
| `%` | Modulo (remainder) | `5 % 2` | `1` |

📌 **Note**: Division of integers truncates decimals. Use `double` for precision.

---

## 📝 Assignment Operators

| Operator | Description | Example | Equivalent To |
| --- | --- | --- | --- |
| `=` | Assign value | `a = 5` |  |
| `+=` | Add and assign | `a += 2` | `a = a + 2` |
| `-=` | Subtract and assign | `a -= 2` | `a = a - 2` |
| `*=` | Multiply and assign | `a *= 2` | `a = a * 2` |
| `/=` | Divide and assign | `a /= 2` | `a = a / 2` |
| `%=` | Modulo and assign | `a %= 2` | `a = a % 2` |

---

## 🔍 Comparison Operators

| Operator | Description | Example | Result |
| --- | --- | --- | --- |
| `==` | Equal to | `a == b` | `true/false` |
| `!=` | Not equal to | `a != b` | `true/false` |
| `>` | Greater than | `a > b` | `true/false` |
| `<` | Less than | `a < b` | `true/false` |
| `>=` | Greater than or equal | `a >= b` | `true/false` |
| `<=` | Less than or equal | `a <= b` | `true/false` |

---

## ⚙️ Logical Operators

| Operator | Description | Example | Result |
| --- | --- | --- | --- |
| `&&` | Logical AND | `a && b` | `true/false` |
| ` |  | ` | Logical OR |
| `!` | Logical NOT | `!a` | `true/false` |

💡 Use these for conditionals like `if (a && b)`.

---

## 🧠 Bitwise Operators

| Operator | Description | Example | Meaning |
| --- | --- | --- | --- |
| `&` | Bitwise AND | `a & b` | 1 if both bits are 1 |
| ` | ` | Bitwise OR | `a |
| `^` | Bitwise XOR | `a ^ b` | 1 if bits are different |
| `~` | Bitwise NOT (1's complement) | `~a` | Inverts all bits |
| `<<` | Left shift | `a << 2` | Shift bits left (×2 each shift) |
| `>>` | Right shift | `a >> 2` | Shift bits right (÷2 each shift) |

---

## 🔁 Increment / Decrement

| Operator | Description | Example | Result |
| --- | --- | --- | --- |
| `++` | Increment by 1 | `a++` | `a = a + 1` |
| `--` | Decrement by 1 | `a--` | `a = a - 1` |

## 🔄 Pre-increment (`++i`) vs Post-increment (`i++`)

| Feature | `++i` (Pre-increment) | `i++` (Post-increment) |
| --- | --- | --- |
| **When it increments** | Increments **before** value is used | Increments **after** value is used |
| **Returned value** | New (incremented) value | Original (old) value |
| **Effect on variable** | Variable is increased immediately | Variable is increased after use |
| **Usage example** | `int a = ++i;` → `a = 6`, `i = 6` | `int b = i++;` → `b = 6`, `i = 7` |
| **Common in** | Value needed **after** increment | Value needed **before** increment |

---

## ❓ Ternary Operator

| Operator | Description | Example |
| --- | --- | --- |
| `?:` | Short if-else | `a > b ? "A" : "B"` |

---

## 💬 Null-Coalescing Operators

| Operator | Description | Example |
| --- | --- | --- |
| `??` | Return right-hand value if null | `name = input ?? "N/A"` |
| `??=` | Assign only if null | `name ??= "Unknown"` |

---

## 🧷 Null-Conditional Operators

| Operator | Description | Example |
| --- | --- | --- |
| `?.` | Safe navigation for members | `user?.Name` |
| `?[]` | Safe indexing | `array?[0]` |

🛡 Helps avoid `NullReferenceException`.

---

## 🧪 Type Operators

| Operator | Description | Example |
| --- | --- | --- |
| `is` | Type check | `if (obj is string)` |
| `as` | Safe cast | `var s = obj as string` |
| `typeof` | Get `Type` object | `typeof(int)` |
| `(T)` | Direct cast | `int x = (int)obj` |

---

## ➡️ Lambda Operator

| Operator | Description | Example |
| --- | --- | --- |
| `=>` | Lambda expression | `x => x * x` |

⚡ Often used in LINQ and event handlers.

---