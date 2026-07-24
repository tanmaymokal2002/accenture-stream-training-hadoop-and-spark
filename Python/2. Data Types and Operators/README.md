# Python: Data Types & Operators — Quick Revision Notes

---

## 1. Print Statement

- `print()` → built-in function to display output.

---

## 2. Arithmetic Operators

| Symbol | Meaning             | Note                                   |
| ------ | ------------------- | -------------------------------------- |
| `+`    | Addition            |                                        |
| `-`    | Subtraction         |                                        |
| `*`    | Multiplication      |                                        |
| `/`    | Division            | always returns float                   |
| `%`    | Modulus (remainder) |                                        |
| `**`   | Exponentiation      | `^` does NOT work like other languages |
| `//`   | Floor division      | rounds down to nearest int             |

- Order of operations = **PEMDAS** (Parentheses, Exponents, Mult/Div, Add/Sub).
- ⚠️ MCQ trap: `^` is bitwise XOR in Python, **not power**.

---

## 3. Variables & Assignment

```python
mv_population = 74728          # name = value
x, y, z = 3, 4, 5              # multiple assignment
```

**Naming rules:**

- Letters, numbers, underscores only; must start with letter/underscore.
- No spaces, no reserved keywords.
- Use **snake_case** (Pythonic): `my_height = 58` ✅ not `MyHeight` ❌

**Reserved keywords (sample):** `False, None, True, and, or, not, if, elif, else, for, while, def, class, return, import, in, is, lambda, try, except, finally, with, yield`

**Assignment operators:**
| Symbol | Example | Same as |
|--------|---------|---------|
| `=` | `x = 2` | `x = 2` |
| `+=` | `x += 2` | `x = x + 2` |
| `-=` | `x -= 2` | `x = x - 2` |

---

## 4. Integers & Floats

```python
x = int(4.7)     # 4  (truncates, not rounds)
y = float(4)      # 4.0
type(x)           # check data type
```

⚠️ **Float precision issue (common MCQ):**

```python
print(.1 + .1 + .1 == .3)   # False!
```

→ floats are approximations, not exact.

**PEP8 style:** keep lines ≤ 79–99 chars; write clean spacing: `print(4 + 5)` not `print(    4 + 5)`.

---

## 5. Booleans & Comparison/Logical Operators

`bool` → `True`(1) / `False`(0)

**Comparison operators (return bool):**
| Expr | Result |
|------|--------|
| `5 < 3` | False |
| `5 > 3` | True |
| `3 <= 3` | True |
| `3 >= 5` | False |
| `3 == 5` | False |
| `3 != 5` | True |

**Logical operators:**
| Expr | Result | Meaning |
|------|--------|---------|
| `5<3 and 5==5` | False | all must be True |
| `5<3 or 5==5` | True | at least one True |
| `not 5<3` | True | flips value |

---

## 6. Strings

```python
s1 = 'this is a string'
s2 = "this is also a string"
s3 = 'Simon\'s skateboard'      # escape quote with \
```

- String starting with `'` ends at next unescaped `'` (same for `"`).
- 0-indexed: `first_word[0]` → first char.

**Common operations:**

```python
first_word + second_word     # concatenation → 'HelloThere'
first_word + ' ' + second_word
first_word * 5                # repetition
len(first_word)                # length (returns int)
```

⚠️ MCQ trap: `len("ababa") / len("ab")` → `2.5` → result is **float** because `/` always gives float.

---

## 7. Type Conversion

```python
type(4)        # int
type(3.7)      # float
type('this')   # str
type(True)     # bool
```

⚠️ Common trick questions:

```python
"0" + "5"   # '05'  (string concatenation)
0 + 5       # 5     (int addition)
"0" + 5     # ❌ TypeError (can't mix str + int)
0 + "5"     # ❌ TypeError
```

---

## 8. String Methods

Methods = functions called with **dot notation**: `sample_string.lower()`

```python
my_string.islower()     # True/False, no extra arg
my_string.count('a')    # counts occurrences
my_string.find('a')     # returns index of first match
```

### `.format()`

```python
print("Mohammed has {} balloons".format(27))
# Mohammed has 27 balloons

print("Does your {} {}?".format("dog", "bite"))
# Does your dog bite?
```

Rule: number of `{}` = number of args passed.

### F-strings (Python 3.6+)

```python
name = "John"
print(f"Hello, {name}")          # Hello, John

a, b = 5, 3
print(f"Sum is {a+b}")           # Sum is 8
```

→ Faster/cleaner than `.format()`; can embed expressions directly.

### `.split(sep, maxsplit)`

```python
"The cow jumped over the moon.".split()
# ['The', 'cow', 'jumped', 'over', 'the', 'moon.']

"The cow jumped over the moon.".split(' ', 3)
# ['The', 'cow', 'jumped', 'over the moon.']   # maxsplit=3 → 4 elements

"The cow jumped over the moon.".split('.')
# ['The cow jumped over the moon', '']
```

- Default separator = whitespace.
- `maxsplit` → gives `maxsplit + 1` elements; rest stays as last item.

---

## 9. Debugging Basics

**Common errors:**
| Error | Cause |
|-------|-------|
| `ZeroDivisionError` | dividing by 0 |
| `SyntaxError: unexpected EOF while parsing` | missing closing bracket/quote |
| `TypeError: len() takes exactly one argument (0 given)` | missing required function argument |

**Debug tips:**

1. Read error messages carefully (they tell type + cause).
2. Search error message on Google/StackOverflow.
3. Use temporary `print()` statements to trace variable values.

---

## 🔑 Quick MCQ Traps to Remember

- `/` → always float, `//` → floor (int-like), `**` → power (not `^`).
- `int(4.7)` truncates to `4`, doesn't round.
- `0.1 + 0.1 + 0.1 == 0.3` → **False** (float precision).
- String + int → `TypeError`.
- `len()` always returns `int`; but division of two `len()` → `float`.
- Python is **0-indexed**.
