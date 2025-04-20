Learn about `dataclasses`, especially around **mutable defaults**, **frozen classes**, **slots**, and **post-init logic**:

---

### 🧠 Key Concepts You've Worked With

#### ✅ 1. **Default values and mutables**
- Mutable defaults like `[]` or `{}` should never be used directly in class definitions.
- Use `field(default_factory=list)` to safely create a new list for every instance.
  ```python
  parents: list[str] = field(default_factory=list)
  ```

#### ✅ 2. **Dataclasses vs. regular classes**
- Regular classes: `self.items = []` inside `__init__` is safe.
- Dataclasses auto-generate `__init__`, so you use `default_factory` to handle mutables.

#### ✅ 3. **Frozen dataclasses (`frozen=True`)**
- Makes the instance immutable — prevents any attribute changes after initialization.
- You **can still modify attributes** inside `__post_init__`, but only using:
  ```python
  object.__setattr__(self, 'attr_name', value)
  ```

#### ✅ 4. **Slots (`slots=True`)**
- Enables `__slots__` for memory/performance optimization and attribute restriction.
- With `slots=True`, you **must declare every attribute** ahead of time.
- If you try to set an undeclared attribute (like `fullname`), Python raises `AttributeError`.

#### ✅ 5. **`__post_init__`**
- This method is automatically called after `__init__` in dataclasses.
- It's where you can do computed fields like `fullname` — useful for logic that depends on multiple fields.

#### ✅ 6. **Combining `frozen` and `slots`**
- You can do it! ✅ But:
  - Declare all attributes (`fullname`, etc.) using `field(init=False)` if you want to set them manually.
  - Use `object.__setattr__` inside `__post_init__` to set values.

---

### ✅ Your Final Working Pattern

```python
@dataclass(frozen=True, slots=True, kw_only=True)
class Person:
    name: str
    surname: str
    age: int
    ID: str = field(init=False, default_factory=generate_id)
    parents: list[str] = field(default_factory=list)
    fullname: str = field(init=False)  # Required for slots

    def __post_init__(self):
        object.__setattr__(self, 'fullname', f'{self.name} {self.surname}')
```

---

### 🏁 TL;DR Cheat Sheet

| Feature         | Use It For                           | Gotchas                             |
|----------------|--------------------------------------|-------------------------------------|
| `default_factory` | Mutable defaults like `[]`, `{}`     | Avoid `=[]` directly                |
| `frozen=True`   | Immutable dataclasses                | Use `object.__setattr__` in `__post_init__` |
| `slots=True`    | Restrict attributes + save memory    | All attributes must be declared     |
| `__post_init__` | Set derived/computed fields          | Needed for logic after init         |

---

kw_only=True — Enforces keyword-only arguments
Forces all fields to be passed as keyword arguments, not positional.
Makes your constructor more readable and less error-prone.

init=True/False — Control if a field is in __init__
init=True (default): Included in the constructor (__init__)
init=False: Not passed in — you’ll usually set it later or inside __post_init__
