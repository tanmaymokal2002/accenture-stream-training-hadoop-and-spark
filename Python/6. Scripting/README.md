# Python: Scripting — Quick Revision Notes

**Script** = Python code saved in a file with `.py` extension, run via command line (`python filename.py` or `python3 filename.py`).

Topics: Environment setup, Running scripts, User input, Exceptions, File I/O, Importing modules, Interpreter.

---

## 1. Python Installation Basics (quick facts)

- Course uses **Python 3** (Python 2 no longer updated).
- Check version: `python --version` in terminal.
- **Anaconda** → recommended distribution; bundles Python + data science libraries + environment management (`conda --version` to check).
- Run a script: `python first_script.py` (or `python3 ...` if both versions installed).

---

## 2. Raw Input (`input()`)

```python
name = input("Enter your name: ")
print("Hello there, {}!".format(name.title()))
```

- `input()` always returns a **string**, regardless of what the user types.
- To use as another type, wrap with a conversion function:

```python
num = int(input("Enter an integer"))
print("hello" * num)
```

### `eval()` — evaluate input as Python code

```python
result = eval(input("Enter an expression: "))
print(result)
```

- If user enters `2 * 3` → evaluates and outputs `6`.
- ⚠️ MCQ trap: `input()` → always **string**; `eval(input())` → executes the string **as Python code** (risky in real apps, but useful for quick math-expression input).

---

## 3. Errors vs Exceptions

| Type             | When it occurs                                                                                      |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| **Syntax Error** | Python can't even parse/interpret the code (typos, bad syntax)                                      |
| **Exception**    | Code is syntactically correct, but something unexpected happens at runtime (e.g., dividing by zero) |

---

## 4. Try / Except / Else / Finally

```python
try:
    # code that might raise an exception
except ValueError:
    # runs only if a ValueError occurs
else:
    # runs only if NO exception occurred in try block
finally:
    # ALWAYS runs, no matter what (even if program is exiting)
```

| Clause    | Mandatory? | Runs when                            |
| --------- | ---------- | ------------------------------------ |
| `try`     | ✅ Yes     | always first                         |
| `except`  | optional   | if an exception occurs in `try`      |
| `else`    | optional   | if **no** exception occurred         |
| `finally` | optional   | **always**, regardless of exceptions |

**Specifying exceptions:**

```python
try:
    # code
except ValueError:
    # handles only ValueError

try:
    # code
except (ValueError, KeyboardInterrupt):    # handle multiple types with tuple
    # code

try:
    # code
except ValueError:
    # code
except KeyboardInterrupt:                   # multiple except blocks
    # code
```

**Accessing the error message with `as`:**

```python
try:
    # code
except ZeroDivisionError as e:
    print("ZeroDivisionError occurred: {}".format(e))

try:
    # code
except Exception as e:        # catches ANY exception (base class)
    print("Exception occurred: {}".format(e))
```

⚠️ MCQ trap: `Exception` is the **base class** for all built-in exceptions — using `except Exception` catches everything (use specific exceptions when possible for clarity).

---

## 5. Reading & Writing Files

### Reading

```python
f = open('my_path/my_file.txt', 'r')   # 'r' = read mode (default)
file_data = f.read()
print(file_data)
f.close()
```

### Writing

```python
f = open('my_path/my_file.txt', 'w')   # 'w' = write mode
f.write("Hello there!")
f.close()
```

⚠️ MCQ trap — file modes:
| Mode | Behavior |
|------|----------|
| `'r'` | read only (default); file must exist |
| `'w'` | write; **overwrites/deletes** existing content, creates file if missing |
| `'a'` | append; adds to existing content without deleting |

### `with` statement — auto-closes file (best practice)

```python
with open('my_path/my_file.txt', 'r') as f:
    file_data = f.read()
# file is automatically closed here, no need for f.close()
```

- File object `f` only accessible **within** the indented `with` block.
- ⚠️ Forgetting `.close()` (without `with`) can lead to **"too many open files"** errors if done repeatedly (e.g., in a loop).

---

## 6. Importing Local Scripts

```python
import useful_functions                      # import whole module
useful_functions.add_five([1, 2, 3, 4])       # access via dot notation

import useful_functions as uf                 # import with alias
uf.add_five([1, 2, 3, 4])
```

- Import statements conventionally go at the **top** of a script, one per line.
- Importing creates a **module object**; access its contents with dot notation.

### `if __name__ == "__main__":` block

```python
def main():
    print("Testing...")

if __name__ == '__main__':
    main()
```

- Every module has a built-in `__name__` variable.
- When a script is **run directly**, `__name__` = `"__main__"`.
- When a script is **imported** into another script, `__name__` = the module's own name (not `"__main__"`).
- ⚠️ MCQ trap: This pattern prevents code from **auto-running** when a file is imported elsewhere — only runs when the file itself is executed directly.

---

## 7. Techniques for Importing Modules

```python
from module_name import object_name                     # import a single function/class
from module_name import first_object, second_object      # import multiple objects
import module_name as new_name                            # rename whole module
from module_name import object_name as new_name           # rename specific object
from module_name import *                                 # ❌ import everything (AVOID — pollutes namespace)
```

⚠️ MCQ trap: `from module import *` is **discouraged** — can cause naming conflicts; prefer `import module_name` + dot notation if you need many objects.

**Packages & submodules:**

```python
import package_name.submodule_name
```

- A **package** = a module containing sub-modules, accessed via dot notation.

---

## 8. Third-Party Libraries & pip

```bash
pip install package_name              # install a single package
pip install -r requirements.txt       # install all dependencies listed in file
```

**`requirements.txt` example:**

```
beautifulsoup4==4.5.1
bs4==0.0.1
requests==2.11.1
```

- Lists package name + version (version optional but recommended, for reproducibility).

**Common third-party libraries (good to recognize names for MCQ):**
| Library | Purpose |
|---------|---------|
| `requests` | making web/API requests |
| `Flask` / `Django` | web app frameworks |
| `Beautiful Soup` | HTML parsing / web scraping |
| `pytest` | testing |
| `NumPy` | numerical computing, arrays |
| `pandas` | dataframes, data analysis |
| `Matplotlib` | plotting/visualization |
| `Pillow` | image processing |
| `Pygame` | game development |

---

## 9. Interpreter (Interactive Mode)

- Start with `python` command in terminal → opens REPL (Read-Eval-Print Loop).

```python
>>> type(5.23)
<class 'float'>
```

- Automatically prints the value of the **last line** — no need for `print()` on a single final expression.
- Continuation lines (e.g., inside a function def) show `...` prompt; manual indentation required.
- Exit with `exit()` or `Ctrl-D` (Mac/Linux) / `Ctrl-Z` + Enter (Windows).
- **IPython** = enhanced alternative interpreter — tab completion, `?` for object info, `!` to run shell commands, syntax highlighting.

---

## 🔑 Quick MCQ Traps — Scripting

- `input()` always returns a **string** — must explicitly convert (`int()`, `float()`) for other types.
- `eval(input())` executes user input as actual Python code.
- Syntax Error = code won't even parse; Exception = runtime problem in valid code.
- `try` is the only **mandatory** clause; `finally` **always** runs regardless of exceptions.
- File modes: `'r'` read, `'w'` write (overwrites!), `'a'` append.
- `with open(...) as f:` → auto-closes file, no manual `.close()` needed.
- `if __name__ == "__main__":` → code only runs when file executed directly, not when imported.
- `from module import *` → bad practice, avoid due to naming collisions.
- `pip install -r requirements.txt` → installs all dependencies from file at once.
