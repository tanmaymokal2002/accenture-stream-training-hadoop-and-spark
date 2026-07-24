# Python: Advanced Topics — Quick Revision Notes

Topics: Iterators, Generators, Generator Expressions.

---

## 1. Iterables vs Iterators

| Term         | Meaning                                                                                    |
| ------------ | ------------------------------------------------------------------------------------------ |
| **Iterable** | An object that can return its elements one at a time (e.g. a `list`, `string`, `dict`).    |
| **Iterator** | An object that represents a **stream of data** — produces values one at a time, on demand. |

⚠️ MCQ trap: **All iterators are iterables, but not all iterables are iterators.**

- A `list` is iterable, but is **NOT** an iterator (its elements already all exist in memory at once — not a "stream").
- Built-in functions like `enumerate()` return an **iterator**.

---

## 2. Generators

**Generator** = a simple way to create an **iterator**, using a function with the `yield` keyword instead of `return`.

```python
def my_range(x):
    i = 0
    while i < x:
        yield i
        i += 1
```

- `yield` → returns a value **one at a time**, and **pauses** the function, remembering exactly where it left off.
- Next time the generator is asked for a value, execution **resumes right after the `yield`** (not from the top).
- This is what makes it a generator instead of a normal function — `yield` vs `return` is the key distinguishing feature.

**Using a generator:**

```python
for x in my_range(5):
    print(x)
# 0
# 1
# 2
# 3
# 4
```

- Since it returns an iterator, you can also do `list(my_range(5))` to materialize all values into a list.

---

## 3. Why Use Generators Over Lists?

- **Lazy evaluation** — generators compute values **on demand**, one at a time, instead of building the whole collection upfront.
- **Memory efficient** — useful when a fully realized list would be **too large to fit in memory**.
- Useful when each element is **expensive to compute** and you want to delay that cost until it's actually needed.

⚠️ MCQ trap: Generators can only be **iterated over once** — once exhausted, you can't loop through them again (unlike a list, which can be re-iterated any number of times).

---

## 4. Generator Expressions (Generator Comprehensions)

Same syntax as a **list comprehension**, but using `()` parentheses instead of `[]` square brackets.

```python
sq_list = [x**2 for x in range(10)]      # LIST comprehension → builds full list in memory
sq_iterator = (x**2 for x in range(10))  # GENERATOR expression → lazy iterator, computes on demand
```

⚠️ MCQ trap: `()` around a comprehension does **NOT** create a tuple here — it creates a **generator object**. To get a tuple, you'd need `tuple(x**2 for x in range(10))`.

---

## 🔑 Quick MCQ Traps — Advanced Topics

- Iterator ⊂ Iterable: every iterator is iterable, but a plain `list` is iterable and **not** an iterator.
- `yield` (not `return`) is what makes a function a **generator function**.
- Generator execution **pauses** at `yield` and **resumes** from there on the next call — doesn't restart from the top.
- Generators are **lazy** (compute on demand) → memory-efficient for huge/expensive sequences.
- Generators can be **iterated only once** — no re-looping after exhaustion.
- `(expr for x in iterable)` → generator expression, NOT a tuple (despite the parentheses).
