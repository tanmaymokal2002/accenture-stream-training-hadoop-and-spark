# Python: Pandas — Quick Revision Notes

**Pandas** = data manipulation/analysis library. Name from "Panel Data". Built on top of NumPy. Adds two key structures: **Series** (1D, labeled) and **DataFrame** (2D, labeled rows+columns, like Excel/spreadsheet).

```python
import pandas as pd    # standard import convention
```

**Why Pandas:** labeled rows/columns, rolling stats on time series, easy NaN handling, loads multiple file formats, joins/merges datasets, integrates with NumPy & Matplotlib.

---

## 1. Pandas Series

**1D, array-like, with custom index labels** — unlike NumPy ndarrays:

- Can assign **custom labels** to each element's index.
- Can hold **mixed data types** (NumPy arrays must be homogeneous).

```python
groceries = pd.Series(data=[30, 6, 'Yes', 'No'], index=['eggs','apples','milk','bread'])
```

```
eggs      30
apples     6
milk     Yes
bread     No
dtype: object
```

**Attributes:**

```python
groceries.shape    # (4,)
groceries.ndim     # 1
groceries.size     # 4
groceries.values    # array of data only
groceries.index     # Index(['eggs','apples','milk','bread'])
```

**Membership check:**

```python
'bananas' in groceries   # False (checks INDEX labels, not values!)
'bread' in groceries     # True
```

⚠️ MCQ trap: `in` checks the **index labels**, not the data values.

### Accessing elements — `.loc` vs `.iloc`

| Attribute | Meaning                                                     |
| --------- | ----------------------------------------------------------- |
| `.loc`    | **loc**ation — access by **label** (index name)             |
| `.iloc`   | **i**nteger **loc**ation — access by **numerical position** |

```python
groceries['eggs']                  # by label
groceries[['milk', 'bread']]       # multiple labels
groceries.loc[['eggs', 'apples']]  # explicit label access
groceries[[0, 1]]                  # by numerical index
groceries[[-1]]                    # negative index → from end
groceries.iloc[[2, 3]]             # explicit numerical access
```

### Modifying & Deleting

```python
groceries['eggs'] = 2         # mutate value (Series are mutable)

groceries.drop('apples')                    # OUT-OF-PLACE (original unchanged)
groceries.drop('apples', inplace=True)      # IN-PLACE (original modified)
```

⚠️ MCQ trap: `.drop()` is **out-of-place by default** — must set `inplace=True` to modify the original Series.

### Arithmetic on Series

```python
fruits = pd.Series([10, 6, 3], index=['apples','oranges','bananas'])

fruits + 2     # element-wise add
fruits * 2     # element-wise multiply
np.sqrt(fruits)  # NumPy math functions work on Series too
```

**Operate on selected elements:**

```python
fruits['bananas'] + 2
fruits.iloc[0] - 2
fruits[['apples','oranges']] * 2
fruits.loc[['apples','oranges']] / 2
```

⚠️ MCQ trap: Arithmetic on **mixed-type** Series works only if the operation is valid for ALL types present. `groceries * 2` → doubles numbers AND duplicates strings (e.g. `"Yes"` → `"YesYes"`) since `*` is valid for both ints and strings; but `/` would error on strings.

---

## 2. Pandas DataFrame

**2D, labeled rows & columns** — like a spreadsheet/Excel sheet.

### Create from a dictionary of Series

```python
items = {'Alice': pd.Series([40,110,500,45], index=['book','glasses','bike','pants']),
         'Bob':   pd.Series([245,25,55], index=['bike','pants','watch'])}
shopping_carts = pd.DataFrame(items)
```

- Row labels = **union** of all Series' index labels.
- Column labels = dictionary **keys**.
- Missing values → filled with **NaN** (Not a Number).

**Without index labels** → Pandas assigns default numerical row indices (0,1,2...), like NumPy.

**Attributes (same style as Series):**

```python
shopping_carts.shape     # (rows, cols)
shopping_carts.ndim      # 2
shopping_carts.size      # total elements
shopping_carts.values     # underlying data as array
shopping_carts.index      # row labels
shopping_carts.columns    # column labels
```

**Select subset while creating:**

```python
pd.DataFrame(items, columns=['Bob'])                       # only Bob's column
pd.DataFrame(items, index=['pants','book'])                # only selected rows
pd.DataFrame(items, index=['glasses','bike'], columns=['Alice'])  # rows + cols
```

### Create from dictionary of lists

```python
data = {'Floats': [4.5, 8.2, 9.6], 'Integers': [1, 2, 3]}
df = pd.DataFrame(data)                                    # default numeric index
df = pd.DataFrame(data, index=['label 1','label 2','label 3'])  # custom index
```

⚠️ All lists in the dict **must be the same length**.

### Create from a list of dictionaries

```python
items2 = [{'bikes': 20, 'pants': 30, 'watches': 35},
          {'watches': 10, 'glasses': 50, 'bikes': 15, 'pants': 5}]
store_items = pd.DataFrame(items2, index=['store 1', 'store 2'])
```

- Missing keys in a given dict entry → **NaN** for that column/row combo.

---

## 3. Accessing & Modifying DataFrame Elements

```python
store_items[['bikes']]                # single column (as DataFrame)
store_items[['bikes', 'pants']]        # multiple columns
store_items.loc[['store 1']]           # single row by label
store_items['bikes']['store 2']        # single element: COLUMN first, then ROW
```

⚠️ Big MCQ trap: Individual element access is `df[column][row]` — **column label first, then row label**. Reversing the order causes an error.

### Add columns

```python
store_items['shirts'] = [15, 2]                        # new column (added at end)
store_items['suits'] = store_items['pants'] + store_items['shirts']  # derived column via arithmetic
```

### Add rows — via `pd.concat()`

```python
new_store = pd.DataFrame([{'bikes':20,'pants':30,'watches':35,'glasses':4}], index=['store 3'])
store_items = pd.concat([store_items, new_store])
```

⚠️ After `pd.concat()`, columns may get reordered **alphabetically**.

### Insert column at specific location

```python
store_items.insert(5, 'shoes', [8, 5, 0])   # insert 'shoes' column at index position 5
```

### Delete rows/columns

```python
store_items.pop('new watches')                    # .pop() → COLUMNS ONLY
store_items = store_items.drop(['watches','shoes'], axis=1)   # .drop() → columns (axis=1)
store_items = store_items.drop(['store 2','store 1'], axis=0) # .drop() → rows (axis=0)
```

⚠️ MCQ trap: `.pop()` only deletes **columns**; `.drop()` deletes **both** rows and columns (specify via `axis`).

### Rename labels

```python
store_items = store_items.rename(columns={'bikes': 'hats'})       # rename column
store_items = store_items.rename(index={'store 3': 'last store'}) # rename row
store_items = store_items.set_index('pants')                       # use existing column AS row index
```

---

## 4. Handling NaN Values

```python
store_items.isnull()             # returns Boolean DataFrame (True where NaN)
store_items.isnull().sum()       # count NaNs per COLUMN (Series)
store_items.isnull().sum().sum() # TOTAL NaN count in entire DataFrame
store_items.count()              # count of NON-NaN values per column
```

⚠️ MCQ trap: `.isnull().sum()` needs to be called **twice** to get a single total — first sum gives per-column counts (a Series), second sum adds those up.

### Eliminating NaN — `.dropna(axis)`

```python
store_items.dropna(axis=0)   # drop ROWS containing any NaN
store_items.dropna(axis=1)   # drop COLUMNS containing any NaN
```

- Out-of-place by default; use `inplace=True` to modify original.

### Replacing NaN

```python
store_items.fillna(0)                  # replace ALL NaN with 0

store_items.ffill(axis=0)              # forward fill DOWN columns (uses previous row's value)
store_items.ffill(axis=1)              # forward fill ACROSS rows (uses previous column's value)

store_items.bfill(axis=0)              # backward fill DOWN columns (uses next row's value)
store_items.bfill(axis=1)              # backward fill ACROSS rows (uses next column's value)

store_items.interpolate(method='linear', axis=0)  # linear interpolation down columns
store_items.interpolate(method='linear', axis=1)  # linear interpolation across rows
```

⚠️ MCQ trap: `ffill`/`bfill`/`interpolate` **cannot fill a NaN that has no valid neighbor** in that direction (e.g., NaN at the very start of a column with `ffill(axis=0)` stays NaN — no "previous" value exists).

- All these methods are **out-of-place by default**; use `inplace=True` to modify original.

---

## 5. Loading Data & Descriptive Statistics

```python
google_stock = pd.read_csv('./GOOG.csv')     # load CSV into DataFrame
google_stock.shape                            # (rows, cols)

google_stock.head()      # first 5 rows (default); head(N) for custom count
google_stock.tail()      # last 5 rows; tail(N) for custom count

google_stock.isnull().any()    # per-column: True if ANY NaN present in that column
```

### Descriptive statistics

```python
google_stock.describe()              # count, mean, std, min, 25%, 50%, 75%, max — per column
google_stock['Adj Close'].describe() # stats for a single column

google_stock.max()                    # max value per column
google_stock['Close'].min()           # min of a single column
google_stock.mean(numeric_only=True)  # average per column (numeric columns only)
google_stock.corr(numeric_only=True)  # correlation matrix between columns
```

⚠️ Correlation value: **1** = strong positive correlation, **0** = no correlation, values can range down to **-1** (strong negative).

---

## 6. `groupby()` Method

Groups rows by one or more column values, then apply an aggregate function.

```python
data.groupby(['Year'])['Salary'].sum()     # total salary PER YEAR
data.groupby(['Year'])['Salary'].mean()    # average salary PER YEAR
data.groupby(['Name'])['Salary'].sum()     # total salary PER EMPLOYEE (across all years)
data.groupby(['Year', 'Department'])['Salary'].sum()   # group by MULTIPLE columns
```

⚠️ Pattern: `df.groupby([columns_to_group_by])[column_to_aggregate].aggregate_function()`.

---

## 🔑 Quick MCQ Traps — Pandas

- Series `in` operator checks **index labels**, not data values.
- `.loc` = label-based access; `.iloc` = integer-position-based access.
- `.drop()` is **out-of-place by default** — needs `inplace=True` to modify original.
- DataFrame single-element access: `df[column][row]` — **column first**, then row (opposite order errors).
- `.pop()` → columns only; `.drop()` → rows or columns (via `axis=0`/`axis=1`).
- Missing dict keys when building a DataFrame → **NaN** filled automatically.
- `.isnull().sum().sum()` → double `.sum()` needed for a single grand total.
- `ffill`/`bfill`/`interpolate` leave NaN unfilled if there's no valid neighbor in that direction.
- `.dropna()`, `.fillna()`, `.ffill()`, `.bfill()`, `.interpolate()` are all **out-of-place by default**.
- `axis=0` → operate down rows/columns; `axis=1` → operate across columns/rows (same convention as NumPy).
- `.describe()` gives count/mean/std/min/25%/50%/75%/max in one call.
- `groupby()` syntax: `df.groupby([cols])[target_col].agg_function()`.
