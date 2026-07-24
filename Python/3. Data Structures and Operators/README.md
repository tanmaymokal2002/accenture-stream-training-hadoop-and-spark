# Python: Data Structures — Quick Revision Notes

**Why data structures?** To group/organize many related values instead of using hundreds of separate variables (e.g. 506 stock tickers → 1 list instead of 506 strings).

---

## 1. Lists

```python
list_of_random_things = [1, 3.4, 'a string', True]   # mixed data types allowed
```

- Defined with `[]`, **0-indexed**, **ordered**, **mutable**.

**Indexing:**

```python
list_of_random_things[0]                          # 1  (first element)
list_of_random_things[len(list_of_random_things)] # ❌ IndexError (out of range)
list_of_random_things[len(list_of_random_things)-1] # True (last element)
list_of_random_things[-1]                          # True  (last)
list_of_random_things[-2]                          # 'a string' (2nd last)
```

⚠️ MCQ trap: `lst[len(lst)]` always throws `IndexError` — valid indices go up to `len(lst)-1`.

**Slicing** → `lst[start:stop]` → **start inclusive, stop exclusive**:

```python
list_of_random_things[1:2]   # [3.4]
list_of_random_things[:2]    # [1, 3.4]        (from beginning)
list_of_random_things[1:]    # [3.4, 'a string', True]   (to end)
```

- Same slicing rules apply to strings too.

**Membership operators — `in` / `not in`:**

```python
'this' in 'this is a string'     # True
'isa' in 'this is a string'      # False (not contiguous substring)
5 not in [1, 2, 3, 4, 6]          # True
5 in [1, 2, 3, 4, 6]               # False
```

---

## 2. Mutability & Order

| Data type  | Mutable?                                | Ordered?                                                   |
| ---------- | --------------------------------------- | ---------------------------------------------------------- |
| String     | ❌ No                                   | ✅ Yes                                                     |
| List       | ✅ Yes                                  | ✅ Yes                                                     |
| Tuple      | ❌ No                                   | ✅ Yes                                                     |
| Set        | ✅ Yes                                  | ❌ No                                                      |
| Dictionary | ✅ Yes (values; keys must be immutable) | insertion-ordered (Python 3.7+) but not _index_-accessible |

```python
my_lst = [1, 2, 3, 4, 5]
my_lst[0] = 'one'          # ✅ works, lists are mutable
print(my_lst)               # ['one', 2, 3, 4, 5]

greeting = "Hello there"
greeting[0] = 'M'           # ❌ TypeError: 'str' object does not support item assignment
```

- To "change" a string, you must reassign it entirely: `greeting = "Mello there"` → creates a **new object** in memory.
- ⚠️ Two key questions for EVERY data structure: **Is it mutable? Is it ordered?**

---

## 3. Useful List Functions & Methods

```python
len(lst)        # number of elements
max(lst)        # largest element (alphabetically last, if strings)
min(lst)        # smallest element
sorted(lst)     # returns NEW sorted list; original lst unchanged
```

⚠️ `max()`/`min()` fail if list has **incomparable mixed types** (e.g. int + str).

```python
"\n".join(["fore", "aft", "starboard", "port"])
# 'fore\naft\nstarboard\nport'   (joins list of strings using separator)

letters = ['a', 'b', 'c', 'd']
letters.append('z')
print(letters)   # ['a', 'b', 'c', 'd', 'z']   (adds to END of list)
```

⚠️ `.join()` is called **on the separator string**, not on the list: `sep.join(list)`.

---

## 4. Tuples

- **Immutable**, **ordered** sequence — good for fixed, related values.

```python
location = (13.4125, 103.866667)
print("Latitude:", location[0])
print("Longitude:", location[1])
```

**Tuple packing/unpacking:**

```python
dimensions = 52, 40, 100          # parentheses optional
length, width, height = dimensions   # unpacking
print("The dimensions are {} x {} x {}".format(length, width, height))

# shortcut - pack and unpack in one line:
length, width, height = 52, 40, 100
```

⚠️ You **cannot** do `tuple[0] = 5` — immutable, unlike lists.

---

## 5. Sets

- **Mutable**, **unordered**, contains only **unique** elements.

```python
numbers = [99, 100, 1, 3, 4, 99, 100]
unique_nums = set(numbers)
print(unique_nums)          # {1, 3, 99, 100, 4}  → duplicates removed
```

```python
fruit = {"apple", "banana", "orange", "grapefruit"}
"watermelon" in fruit        # False  (membership works like lists)
fruit.add("watermelon")      # add element
fruit.pop()                  # removes a RANDOM element (no order!)
```

⚠️ MCQ trap: `.pop()` on a **set** removes a random element (sets are unordered) — different from list `.pop()` which removes the last by default.

**Set operations (math-set style) — faster than equivalent list operations:**

```python
set1.union(set2)          # combine, unique elements
set1.intersection(set2)   # common elements
set1.difference(set2)     # elements in set1 not in set2
```

- Sets are **faster** than lists for union/intersection/difference (no need to loop through each element).

---

## 6. Dictionaries

- **Mutable**, key–value pairs. Keys must be **immutable** types (str, int, tuple); values can be anything.

```python
elements = {"hydrogen": 1, "helium": 2, "carbon": 6}
random_dict = {"abc": 1, 5: "hello"}     # keys can be mixed types too
```

**Lookup:**

```python
elements["helium"]          # 2
elements["dilithium"]       # ❌ KeyError (key not found)
```

**Insert / update:**

```python
elements["lithium"] = 3     # adds new key:value
print(elements)   # {'hydrogen': 1, 'carbon': 6, 'helium': 2, 'lithium': 3}
```

**Safe lookup with `.get()`:**

```python
elements.get("dilithium")     # None (no error!)
"carbon" in elements           # True  (membership check)
```

⚠️ MCQ trap: `dict["missing_key"]` → `KeyError`, but `dict.get("missing_key")` → `None` (no crash). Use `.get()` when a key might not exist.

---

## 7. Identity Operators

| Keyword  | Meaning                                             |
| -------- | --------------------------------------------------- |
| `is`     | True if both sides are the **same object/identity** |
| `is not` | True if different identities                        |

```python
n = elements.get("dilithium")
print(n is None)        # True
print(n is not None)    # False
```

⚠️ Use `is`/`is not` for comparing to `None`; use `==`/`!=` for value equality.

---

## 8. Compound (Nested) Data Structures

Containers inside containers — e.g., dict of dicts, dict of lists.

```python
elements = {"hydrogen": {"number": 1, "weight": 1.00794, "symbol": "H"},
            "helium":   {"number": 2, "weight": 4.002602, "symbol": "He"}}

helium = elements["helium"]                 # get inner dict
hydrogen_weight = elements["hydrogen"]["weight"]   # chained lookup → 1.00794

oxygen = {"number": 8, "weight": 15.999, "symbol": "O"}
elements["oxygen"] = oxygen                 # add new nested entry
```

```python
student_records = {
    'John': {'age': 20, 'major': 'Computer Science', 'grades': [85, 90, 92]},
    'Emma': {'age': 19, 'major': 'Mathematics', 'grades': [95, 88, 91]}
}

student_records['Alex'] = {'age': 21, 'major': 'Physics', 'grades': [80, 85, 88]}  # new student
student_records['John']['grades'].append(88)   # add grade to existing student's list
```

⚠️ To modify a nested value, chain the keys/indices: `dict[key1][key2]` or `dict[key]['list_field'].append(x)`.

---

## 🔑 Quick MCQ Traps — Data Structures

- **List** = ordered + mutable | **Tuple** = ordered + immutable | **Set** = unordered + mutable, unique only | **Dict** = key-value, keys immutable.
- Slicing `[a:b]` → `a` inclusive, `b` exclusive.
- `lst[len(lst)]` → always `IndexError`.
- `dict["key"]` → `KeyError` if missing; `dict.get("key")` → `None` if missing (safe).
- `set.pop()` removes a **random** element (unordered); `list.pop()` removes **last** by default.
- `sorted(lst)` does NOT modify original list — returns a new one.
- Tuple unpacking: `a, b, c = (1, 2, 3)`.
- `is` / `is not` → identity check (mainly used with `None`); `==` → value check.
- Strings are immutable → reassignment creates a **new object**, not an in-place change.
