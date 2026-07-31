# Python: Functions — Quick Revision Notes

**Function** = a reusable block of code that encapsulates a task (input → processing → output). Write once, use many times.

Topics: Defining Functions, Variable Scope, Documentation, Lambda Expressions.

---

## 1. Defining Functions

```python
def cylinder_volume(height, radius):
    pi = 3.14159
    return height * pi * radius ** 2

cylinder_volume(10, 3)     # function call
```

**Function Header (1st line):**

- Starts with `def` keyword.
- Followed by function name (same naming rules as variables — snake_case, no spaces/keywords).
- Parentheses `()` hold arguments/parameters, separated by commas (empty if none needed).
- Ends with colon `:`.

**Function Body:**

- Indented code block below the header.
- Can define new variables (local to the function) and use argument variables.
- `return` statement sends a value back to the caller.
- ⚠️ MCQ trap: **No `return` statement → function returns `None` by default.**

---

## 2. Naming Conventions for Functions

- Only letters, numbers, underscores; must start with letter/underscore.
- Can't use reserved keywords.
- Use descriptive, snake_case names.

---

## 3. Print vs Return (classic MCQ topic)

```python
def add_print(a, b):
    print(a + b)        # only displays; returns None

def add_return(a, b):
    return a + b        # sends the value back, usable later
```

```python
x = add_print(2, 3)     # prints "5", but x = None
y = add_return(2, 3)    # nothing printed, but y = 5
```

⚠️ **Trap:** `print()` shows output on screen but does NOT give the caller a usable value. `return` does. A function using only `print()` will yield `None` if you try to store/use its result.

---

## 4. Default Arguments

```python
def cylinder_volume(height, radius=5):
    pi = 3.14159
    return height * pi * radius ** 2

cylinder_volume(10)       # radius defaults to 5
cylinder_volume(10, 7)    # radius overwritten to 7
```

- Default value used **only if** that argument is omitted in the call.

**Passing arguments — by position vs by name (keyword):**

```python
cylinder_volume(10, 7)                    # positional
cylinder_volume(height=10, radius=7)      # keyword/named — order doesn't matter
```

⚠️ MCQ trap: Positional args are matched **in order**; keyword args can be in **any order** since they're explicitly named.

---

## 5. Variable Scope

**Scope** = where in the program a variable can be accessed.

### Local Scope

```python
def some_function():
    word = "hello"

print(word)    # ❌ NameError: word is not defined (local to function only)
```

- Variables created inside a function exist **only inside** that function.
- Different functions can reuse the same variable name safely (they don't clash):

```python
def some_function():
    word = "hello"

def another_function():
    word = "goodbye"     # separate scope, no conflict
```

### Global Scope

```python
word = "hello"            # defined outside any function → global

def some_function():
    print(word)            # ✅ can READ global variable

some_function()            # prints "hello"
```

⚠️ **Big MCQ trap:** A function can **read** a global variable, but **cannot modify** it directly inside the function (without `global` keyword — not covered here but good to know exists). To change a value, pass it in as an **argument** instead.

**Good practice:** Define variables in the **smallest scope** needed. Avoid relying on variables from an outer/larger scope inside functions — keeps code predictable.

---

## 6. Documentation (Docstrings)

```python
def population_density(population, land_area):
    """Calculate the population density of an area."""
    return population / land_area
```

- Docstrings = comments explaining what a function does, written in **triple quotes** `""" """`.
- Placed as the **first line** in the function body, right after the header.
- Single-line docstrings are acceptable for simple functions.

**Longer/detailed docstring example:**

```python
def population_density(population, land_area):
    """Calculate the population density of an area.

    INPUT:
    population: int. The population of that area
    land_area: int or float. Unit-agnostic (km² or mi²).

    OUTPUT:
    population_density: population / land_area.
    """
    return population / land_area
```

- All parts (INPUT/OUTPUT descriptions) are **optional**, but recommended for good practice/readability.

---

## 7. Lambda Expressions

**Lambda** = anonymous (unnamed), one-line function — useful for short, throwaway functions (often passed as arguments to other functions).

```python
# Regular function
def multiply(x, y):
    return x * y

# Equivalent lambda expression
multiply = lambda x, y: x * y

multiply(4, 7)     # 28
```

**Components:**

- `lambda` keyword → signals an anonymous function.
- Arguments (comma-separated) → before the colon `:` (names are arbitrary, like normal functions).
- Single expression after `:` → automatically evaluated and returned (no explicit `return` keyword needed).

⚠️ MCQ trap: Lambda functions can only contain **one expression** — no multi-line logic, no statements like `if/for` blocks (only conditional _expressions_ are allowed, e.g. `lambda x: x if x>0 else -x`). Best for short/simple operations, often used with `map()`, `filter()`, `sorted()`.

---

## 🔑 Quick MCQ Traps — Functions

- No `return` statement → function returns `None`.
- `print()` inside a function ≠ `return` — printing doesn't give the caller a usable value.
- Default argument used only when that argument is **omitted** in the call.
- Positional args → matched by **order**; keyword args → matched by **name**, order-independent.
- Local variables (inside function) are **not accessible** outside it → `NameError`.
- Global variables **can be read** inside a function but **not reassigned** without special handling — pass as argument to modify.
- Docstrings use **triple quotes** `"""..."""`, placed as first line in function body.
- Lambda = `lambda args: expression` → single expression only, no `return` keyword, no name (unless assigned to a variable).
