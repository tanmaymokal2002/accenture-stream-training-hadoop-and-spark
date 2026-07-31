# Python: NumPy — Quick Revision Notes

**NumPy** (Numerical Python) = core scientific computing library. Provides the `ndarray` (n-dimensional array) — faster & more memory-efficient than Python lists, with built-in math/linear algebra support.

```python
import numpy as np    # standard import convention
```

---

## 1. Why NumPy over Lists?

- **Speed** — optimized, memory-efficient; orders of magnitude faster for large arrays.
- **Multidimensional arrays** — represents vectors/matrices (crucial for ML, e.g. neural net weight matrices).
- **Built-in math functions** — vectorized operations avoid manual loops.
- Many libraries (Pandas, etc.) are built **on top of** NumPy.

---

## 2. Creating ndarrays from Python Lists

```python
x = np.array([1, 2, 3, 4, 5])     # 1D array (rank 1)
print(x)                           # [1 2 3 4 5]
```

⚠️ `np.array()` is a **function**, not a class — but it returns an `ndarray` object.

### Rank & Shape

```python
x.ndim     # number of dimensions (rank) → 1 for 1D, 2 for 2D, etc.
x.shape    # tuple of dimension sizes, e.g. (5,) for 1D with 5 elements
x.size     # total number of elements
x.dtype    # data type of elements
```

```python
x = np.array([1, 2, 3, 4, 5])
x.shape    # (5,)
type(x)    # <class 'numpy.ndarray'>
x.dtype    # int64
```

### Rank 2 (2D) array

```python
Y = np.array([[1,2,3],[4,5,6],[7,8,9],[10,11,12]])
Y.ndim     # 2
Y.shape    # (4, 3)  → 4 rows, 3 columns
Y.size     # 12
```

### Data Types & Upcasting

- ndarrays are **homogeneous** — all elements MUST be the same type (unlike Python lists).
- Mixed int + string list → NumPy converts **everything to strings**.
- Mixed int + float list → NumPy **upcasts** all to `float64` (to avoid losing precision).

```python
x = np.array([1, 2, 3])          # dtype: int64
y = np.array([1.0, 2.0, 3.0])    # dtype: float64
z = np.array([1, 2.5, 4])        # dtype: float64  (upcasted)
```

**Manually specify dtype:**

```python
x = np.array([1.5, 2.2, 3.7, 4.0, 5.9], dtype=np.int64)
# x = [1 2 3 4 5]   → decimals truncated
```

⚠️ MCQ trap: Specifying `dtype=np.int64` on floats **truncates** decimals (doesn't round).

---

## 3. Save & Load ndarrays

```python
np.save('my_array', x)          # saves as my_array.npy
y = np.load('my_array.npy')     # must include .npy extension when loading
```

---

## 4. Built-in Functions to Create ndarrays

```python
np.zeros((3,4))          # 3x4 array of 0s, dtype defaults to float64
np.ones((3,2))           # 3x2 array of 1s, dtype defaults to float64
np.full((2,3), 5)        # 2x3 array filled with 5, dtype matches the fill value
np.eye(5)                 # 5x5 Identity matrix (1s on diagonal), dtype float64
np.diag([10,20,30,50])    # diagonal matrix with given values on main diagonal
```

⚠️ MCQ trap: `zeros`, `ones`, `eye` default to **float64**; `full` matches the dtype of the value passed in (e.g. `full((2,3), 5)` → int64).

### `np.arange([start,] stop, [step,])`

```python
np.arange(10)        # 0 to 9   (stop excluded)
np.arange(4, 10)      # 4 to 9   (start inclusive, stop excluded)
np.arange(1, 14, 3)   # [1, 4, 7, 10, 13]  (step = 3)
```

### `np.linspace(start, stop, num=50, endpoint=True)`

```python
np.linspace(0, 25, 10)                  # 10 values, BOTH 0 and 25 included
np.linspace(0, 25, 10, endpoint=False)  # 10 values, 25 EXCLUDED
```

⚠️ Big MCQ trap: `arange(stop)` **excludes** stop; `linspace(start,stop,num)` **includes** stop by default (unless `endpoint=False`). Use `linspace` for non-integer/fractional steps (avoids float precision issues in `arange`).

### `np.reshape()` — function vs method

```python
x = np.arange(20)
x = np.reshape(x, (4,5))       # function form: np.reshape(array, new_shape)
Y = np.arange(20).reshape(4,5)  # method form: array.reshape(new_shape) — no need to pass array again
```

- New shape must be **compatible**: total elements must match (e.g. 20 elements → 4×5 ✅, not 4×4 ❌).

### Random arrays

```python
np.random.random((3,3))              # random floats in [0.0, 1.0)
np.random.randint(4, 15, size=(3,2))  # random ints in half-open interval [4, 15)
np.random.normal(0, 0.1, size=(1000,1000))  # Gaussian dist: mean=0, std=0.1
```

---

## 5. Accessing & Modifying Elements

```python
x = np.array([1, 2, 3, 4, 5])
x[0]     # 1  (first, positive index)
x[-1]    # 5  (last, negative index)
x[3] = 20   # modify element → [1, 2, 3, 20, 5]
```

**2D indexing:** `array[row, column]`

```python
X = np.array([[1,2,3],[4,5,6],[7,8,9]])
X[0,0]     # 1
X[2,2]     # 9
X[0,0] = 20   # modifies top-left element
```

### Delete, Append, Insert

```python
np.delete(x, [0,4])              # delete elements at index 0 and 4 (rank 1)
np.delete(Y, 0, axis=0)          # delete row 0 (rank 2, axis=0 → rows)
np.delete(Y, [0,2], axis=1)      # delete columns 0 & 2 (axis=1 → columns)

np.append(x, 6)                  # append single value to end
np.append(x, [7,8])              # append multiple values
np.append(Y, [[7,8,9]], axis=0)  # append a new row
np.append(Y, [[9],[10]], axis=1) # append a new column

np.insert(x, 2, [3,4])           # insert [3,4] before index 2
np.insert(Y, 1, [4,5,6], axis=0) # insert row before index 1
np.insert(Y, 1, 5, axis=1)       # insert column of 5s before index 1
```

⚠️ `axis=0` → rows, `axis=1` → columns (memory tip: "0 = down rows, 1 = across columns").

### Stacking

```python
np.vstack((x, Y))    # stack vertically (rows) — shapes must be compatible
np.hstack((Y, x.reshape(2,1)))    # stack horizontally (columns)
```

---

## 6. Slicing

```python
ndarray[start:end]   # start inclusive, end EXCLUDED
ndarray[start:]       # from start to last element
ndarray[:end]         # from first element to end (excluded)
```

```python
X = np.arange(20).reshape(4, 5)
X[1:4, 2:5]    # rows 1-3, columns 2-4
X[2, :]        # entire 3rd row → rank 1 array
X[:, 2]        # entire 3rd column → rank 1 array
X[:, 2:3]      # entire 3rd column, but returns rank 2 array (2D shape preserved)
```

⚠️ MCQ trap: `X[:,2]` → **rank 1**; `X[:,2:3]` → **rank 2** (same data, different shape!).

### ⚠️ View vs Copy (CRITICAL MCQ topic!)

```python
Z = X[1:4, 2:5]     # Z is just a VIEW of X — NOT a copy!
Z[2,2] = 555        # this ALSO changes X!
```

- Slicing creates a **view**, not a new array — both variable names point to the same data in memory.
- To make an independent copy, use `.copy()`:

```python
Z = np.copy(X[1:4,2:5])         # function form
W = X[1:4,2:5].copy()           # method form
Z[2,2] = 555                     # does NOT affect X now
```

### Using an array as indices

```python
indices = np.array([1,3])
Y = X[indices, :]     # select rows 1 and 3
Z = X[:, indices]     # select columns 1 and 3
```

### `np.diag(array, k=0)`

```python
np.diag(X)        # main diagonal (k=0)
np.diag(X, k=1)    # diagonal above main
np.diag(X, k=-1)   # diagonal below main
```

### `np.unique()`

```python
np.unique(X)    # returns sorted unique elements (flattened, no duplicates)
```

---

## 7. Boolean Indexing & Set Operations

```python
X = np.arange(25).reshape(5, 5)
X[X > 10]                    # all elements > 10
X[(X > 10) & (X < 17)]       # combine conditions with & (and), | (or)
X[(X > 10) & (X < 17)] = -1  # assign new value to matching elements
```

⚠️ MCQ trap: Use `&` and `|` (NOT `and`/`or`) for element-wise boolean conditions on arrays.

**Set operations:**

```python
x = np.array([1,2,3,4,5])
y = np.array([6,7,2,8,4])

np.intersect1d(x, y)   # [2, 4]         → common elements
np.setdiff1d(x, y)     # [1, 3, 5]      → in x but not y
np.union1d(x, y)       # [1,2,3,4,5,6,7,8]  → all unique elements combined
```

---

## 8. Sorting

```python
np.sort(x)      # FUNCTION → returns sorted COPY, original x unchanged (out-of-place)
x.sort()          # METHOD   → sorts x IN PLACE, original x is modified
```

⚠️ Big MCQ trap: `np.sort(x)` = out-of-place (original untouched); `x.sort()` = in-place (original changed).

**Sorting 2D arrays by axis:**

```python
np.sort(X, axis=0)    # sort each COLUMN independently ("down")
np.sort(X, axis=1)    # sort each ROW independently ("across")
```

---

## 9. Arithmetic & Broadcasting

**Element-wise operations** (both symbol and function forms work identically):

```python
x + y   ==  np.add(x, y)
x - y   ==  np.subtract(x, y)
x * y   ==  np.multiply(x, y)
x / y   ==  np.divide(x, y)
```

- Requires arrays to have the **same shape** or be **broadcastable**.

**Other math functions (element-wise):**

```python
np.exp(x)      # exponential
np.sqrt(x)     # square root
np.power(x, 2) # raise to a power
```

### Broadcasting

- NumPy automatically expands smaller arrays/scalars to match shape during arithmetic — no explicit loop needed.

```python
X = np.array([[1,2],[3,4]])
3 * X          # scalar broadcast: [[3,6],[9,12]]
3 + X          # [[4,5],[6,7]]
```

```python
x = np.array([1,2,3])              # shape (3,)
Y = np.array([[1,2,3],[4,5,6],[7,8,9]])   # shape (3,3)
x + Y     # x is broadcast across each row of Y
```

---

## 10. Statistical Functions

```python
X.mean()          # mean of ALL elements
X.mean(axis=0)    # mean of each COLUMN
X.mean(axis=1)    # mean of each ROW

X.sum() / X.sum(axis=0) / X.sum(axis=1)
X.std()  / X.std(axis=0) / X.std(axis=1)     # standard deviation
np.median(X) / np.median(X, axis=0) / np.median(X, axis=1)
X.max()  / X.max(axis=0) / X.max(axis=1)
X.min()  / X.min(axis=0) / X.min(axis=1)
```

- `axis=0` → operate **down columns**; `axis=1` → operate **across rows**.

---

## 🔑 Quick Reference — Function Glossary

| Category             | Function                                                              | Description                                     |
| -------------------- | --------------------------------------------------------------------- | ----------------------------------------------- |
| **General**          | `.dtype`, `.ndim`, `.shape`, `.size`                                  | data type / rank / dimensions / element count   |
|                      | `np.save()` / `np.load()`                                             | save/load `.npy` files                          |
|                      | `.reshape()`                                                          | change shape, same data                         |
| **Creation**         | `np.array()`                                                          | create ndarray from list                        |
|                      | `np.zeros()` / `np.ones()` / `np.full()`                              | fill with 0s / 1s / constant                    |
|                      | `np.eye()` / `np.diag()`                                              | identity matrix / diagonal matrix               |
|                      | `np.arange()` / `np.linspace()`                                       | evenly spaced values (step-based / count-based) |
|                      | `.copy()`                                                             | independent copy (not a view)                   |
| **Elements/Indices** | `np.insert()` / `np.delete()` / `np.append()`                         | modify array contents                           |
|                      | `np.hstack()` / `np.vstack()`                                         | stack horizontally / vertically                 |
|                      | `np.sort()` / `.sort()`                                               | sort copy (out-of-place) / sort in-place        |
| **Set ops**          | `np.intersect1d()` / `np.setdiff1d()` / `np.union1d()`                | common / difference / all-unique                |
| **Arithmetic/Stats** | `np.add/subtract/multiply/divide()`                                   | element-wise arithmetic                         |
|                      | `np.exp()` / `np.sqrt()` / `np.power()`                               | math functions                                  |
|                      | `.mean()` / `.sum()` / `.std()` / `np.median()` / `.min()` / `.max()` | statistics (support `axis=`)                    |

---

## 🔑 Quick MCQ Traps — NumPy

- `np.array()` is a **function**, not a class.
- Mixed int+str list → all become strings; mixed int+float → all upcast to float64.
- `zeros/ones/eye` default dtype = **float64**; `full()` matches the fill value's dtype.
- `arange(stop)` excludes `stop`; `linspace(start,stop,num)` **includes** stop by default.
- Slicing → creates a **VIEW**, not a copy — changes to the slice affect the original array! Use `.copy()` for independence.
- `X[:,2]` → rank 1 array; `X[:,2:3]` → rank 2 array (same values, different shape).
- Boolean indexing conditions combine with `&` / `|`, NOT `and`/`or`.
- `np.sort(x)` → out-of-place (returns new sorted copy); `x.sort()` → in-place (modifies original).
- `axis=0` → down columns; `axis=1` → across rows.
- Broadcasting lets NumPy do arithmetic between arrays of **different but compatible** shapes (or with scalars) without explicit loops.
