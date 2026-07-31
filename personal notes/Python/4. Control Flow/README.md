# Python: Control Flow — Quick Revision Notes

Topics: Conditional statements, Boolean expressions, For/While loops, Break/Continue, Zip/Enumerate, List comprehensions.

---

## 1. If Statement

```python
if phone_balance < 5:
    phone_balance += 10
    bank_balance -= 10
```

- `if` + condition (boolean expression) + `:` → indented block runs only if condition is `True`.
- ⚠️ MCQ trap: use `==` (comparison) not `=` (assignment) inside conditions. `if x = 5:` → SyntaxError.

---

## 2. If / Elif / Else

```python
if season == 'spring':
    print('plant the garden!')
elif season == 'summer':
    print('water the garden!')
elif season == 'fall':
    print('harvest the garden!')
else:
    print('unrecognized season')
```

- `if` → required, checked first.
- `elif` → "else if", checked only if previous conditions were `False`; can have multiple.
- `else` → no condition, runs if ALL above are `False`; must be last.
- Once one condition is `True`, Python skips the rest (only ONE block runs).

**Indentation:**

- Python uses indentation (not braces `{}`) to define code blocks.
- Convention: **4 spaces** per indent level (per PEP8).
- ⚠️ Never mix tabs and spaces → Python 3 disallows this (causes error).

---

## 3. Complex Boolean Expressions

```python
if 18.5 <= weight / height**2 < 25:
    print("BMI is considered 'normal'")

if is_raining and is_sunny:
    print("Is there a rainbow?")

if (not unsubscribed) and (location == "USA" or location == "CAN"):
    print("send email")
```

- Use parentheses `()` to make combined `and`/`or`/`not` logic clear.

**Good vs Bad practices (common MCQ area):**

❌ Don't use `True`/`False` literally as conditions:

```python
if True:            # always runs — useless condition
```

❌ Don't write conditions that are always True:

```python
if is_cold or not is_cold:   # always True (tautology)
```

❌ Watch out for **non-boolean expressions with `or`**:

```python
if weather == "snow" or "rain":   # BUG! "rain" is always truthy (non-empty string)
```

✅ Correct way:

```python
if weather == "snow" or weather == "rain":
```

❌ Don't compare booleans to `True`/`False` explicitly:

```python
if is_cold == True:      # unnecessary
```

✅ Better:

```python
if is_cold:
if not is_cold:            # check for False
```

---

## 4. Truth Value Testing (Truthy / Falsy)

Non-boolean objects are evaluated for **truthiness** in an `if`.

**Falsy values** (everything else is Truthy):

- `None`, `False`
- Zero of any numeric type: `0`, `0.0`, `0j`
- Empty sequences/collections: `""`, `()`, `[]`, `{}`, `set()`, `range(0)`

```python
errors = 3
if errors:                    # non-zero → Truthy
    print("You have {} errors to fix!".format(errors))
else:
    print("No errors to fix!")
```

⚠️ MCQ trap: `if []:` , `if "":` , `if 0:` → all evaluate to **False**.

---

## 5. For Loops

Used for **definite iteration** — iterating over an **iterable** (string, list, tuple, dict, set, file).

```python
cities = ['new york city', 'mountain view', 'chicago', 'los angeles']
for city in cities:
    print(city)
print("Done!")
```

- `for <item> in <iterable>:` — loop variable takes each element in order.
- Loop body must be indented; code after loop (unindented) runs after loop finishes.

### `range(start=0, stop, step=1)`

```python
range(4)        # 0, 1, 2, 3        (only stop given)
range(2, 6)     # 2, 3, 4, 5        (start, stop)
range(1, 10, 2) # 1, 3, 5, 7, 9     (start, stop, step)
```

- `stop` is **required**; result excludes `stop` itself (like slicing).
- `start` defaults to `0`, `step` defaults to `1`.

```python
for i in range(3):
    print("Hello!")     # prints "Hello!" 3 times
```

### Creating/Modifying Lists with For Loops

```python
# Creating a new list
cities = ['new york city', 'mountain view', 'chicago', 'los angeles']
capitalized_cities = []
for city in cities:
    capitalized_cities.append(city.title())

# Modifying a list IN PLACE — needs range() + index
for index in range(len(cities)):
    cities[index] = cities[index].title()
```

⚠️ To modify a list while looping, loop over `range(len(lst))` to get indices, not the elements directly.

---

## 6. Counter Pattern (For Loop + Dictionary)

**Method 1 — using `if`/`else`:**

```python
word_counter = {}
for word in book_title:
    if word not in word_counter:
        word_counter[word] = 1
    else:
        word_counter[word] += 1
```

**Method 2 — using `.get()` (cleaner):**

```python
word_counter = {}
for word in book_title:
    word_counter[word] = word_counter.get(word, 0) + 1
```

- `.get(key, default)` → returns `default` if key missing (here `0`), avoids `KeyError`.
- Both methods give identical results — Method 2 is more Pythonic/concise.

---

## 7. Iterating Through Dictionaries

```python
cast = {"Jerry Seinfeld": "Jerry Seinfeld", "Jason Alexander": "George Costanza"}

for key in cast:                  # default → only KEYS
    print(key)

for key, value in cast.items():   # BOTH keys and values
    print("Actor: {}    Role: {}".format(key, value))
```

⚠️ MCQ trap: plain `for x in dict:` gives **keys only**. Need `.items()` for key-value pairs (returns tuples).

---

## 8. While Loops

Used for **indefinite iteration** — repeats until a condition becomes `False`.

```python
card_deck = [4, 11, 8, 5, 13, 2, 8, 10]
hand = []

while sum(hand) <= 17:
    hand.append(card_deck.pop())
```

- `sum(lst)` → adds up list elements.
- `lst.pop()` → removes & returns the **last** element of a list.
- ⚠️ The loop body MUST change a variable in the condition, or it becomes an **infinite loop**.

### For vs While — when to use which

| Use **for** when...                      | Use **while** when...                                      |
| ---------------------------------------- | ---------------------------------------------------------- |
| iterating over a known/finite collection | looping until a condition is met (unknown # of iterations) |
| `for name in names:`                     | `while count <= 100:`                                      |
| `for i in range(5):`                     | `while user_input == 'y':`                                 |

---

## 9. Break & Continue

```python
break      # terminates/exits the loop entirely
continue   # skips rest of current iteration, moves to next
```

- Both work in `for` and `while` loops.
- ⚠️ MCQ trap: `break` exits the **whole loop**; `continue` only skips the **current iteration** (loop keeps going).

---

## 10. Zip & Enumerate

### `zip()` — combine multiple iterables into tuples

```python
list(zip(['a', 'b', 'c'], [1, 2, 3]))
# [('a', 1), ('b', 2), ('c', 3)]

letters = ['a', 'b', 'c']
nums = [1, 2, 3]
for letter, num in zip(letters, nums):
    print("{}: {}".format(letter, num))
```

**Unzip** using `*`:

```python
some_list = [('a', 1), ('b', 2), ('c', 3)]
letters, nums = zip(*some_list)
```

⚠️ `zip()` returns an **iterator** — wrap with `list()` to view/print it directly.

### `enumerate()` — get index + value together

```python
letters = ['a', 'b', 'c', 'd', 'e']
for i, letter in enumerate(letters):
    print(i, letter)
# 0 a
# 1 b
# 2 c ...
```

---

## 11. List Comprehensions

Compact way to build a list using a `for` loop in one line.

```python
# Traditional for loop
capitalized_cities = []
for city in cities:
    capitalized_cities.append(city.title())

# List comprehension equivalent
capitalized_cities = [city.title() for city in cities]
```

**Syntax:** `[expression for item in iterable]`

### With a condition (filter) — `if` AFTER the iterable:

```python
squares = [x**2 for x in range(9) if x % 2 == 0]
# [0, 4, 16, 36, 64]   → only even x squared
```

- Here `if` acts as a **filter** — skips elements that don't match.

### With if-else (conditional expression) — `if...else` BEFORE the `for`:

```python
squares = [x**2 if x % 2 == 0 else x + 3 for x in range(9)]
```

- Here `if-else` evaluates for **every** element (no filtering, always produces a value).

⚠️ **Big MCQ trap:**

```python
squares = [x**2 for x in range(9) if x % 2 == 0 else x + 3]   # ❌ SyntaxError!
```

- `if` after iterable (filter form) **cannot** have `else`.
- To use `else`, the `if-else` must come **right after the expression**, **before** `for`.

| Form                                          | Position               | Behavior                                          |
| --------------------------------------------- | ---------------------- | ------------------------------------------------- |
| `[expr for x in iterable if cond]`            | filter                 | skips non-matching elements                       |
| `[expr if cond else expr2 for x in iterable]` | conditional expression | keeps ALL elements, transforms based on condition |

---

## 🔑 Quick MCQ Traps — Control Flow

- `if x = 5:` → SyntaxError (use `==`).
- `if weather == "snow" or "rain":` → bug, non-empty string always Truthy; must repeat `weather ==`.
- Falsy values: `None, False, 0, 0.0, "", (), [], {}, set(), range(0)`.
- `for x in dict:` → keys only; use `dict.items()` for key-value pairs.
- `range(stop)` excludes `stop` itself, just like slicing.
- `while` loop with unchanging condition → **infinite loop**.
- `break` = exit loop completely; `continue` = skip to next iteration.
- `zip()`/`enumerate()` return **iterators** — use `list()` to materialize.
- List comprehension filter `if` (after `for`) ≠ conditional `if-else` (before `for`) — mixing them wrong causes `SyntaxError`.
