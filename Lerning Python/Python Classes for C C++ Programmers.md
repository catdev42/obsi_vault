# Python Classes for C/C++ Programmers

Oct 7, 2026 · @Cat

## Mental model

A Python object is a dictionary of attributes plus a pointer to its class. There is no memory layout, no header file, no compile-time type check; attributes are looked up by name at runtime and can be added or removed at any time.

| C / C++ | Python |
| --- | --- |
| `struct` with fixed fields | Object with a dict of attributes (`obj.__dict__`) |
| Stack or heap allocation, you choose | Always heap, always reference-counted and garbage collected |
| Variables hold values | Variables hold references; assignment never copies |
| `this` is implicit | `self` is an explicit first parameter |
| Static typing, overload resolution at compile time | Dynamic typing, one method per name, resolved at runtime |
| `public` / `private` / `protected` | Everything public; underscores are a convention |
| Destructors run deterministically | `__del__` exists but timing is not guaranteed; use `with` for cleanup |
| Classes are compile-time entities | Classes are objects too, created at runtime, can be passed around |

The consequence that matters most: because everything is a reference, two names can point at the same object, and mutating through one is visible through the other. There is no copy constructor. To copy, call `copy.copy(obj)` for a shallow copy (a new object whose attributes still point at the same inner objects) or `copy.deepcopy(obj)` for a deep copy (inner lists, dicts and objects duplicated recursively too).

## Defining a class

`__init__` is the constructor body; `self` is the object being built. Every method takes `self` explicitly as its first parameter, and attributes exist only once something assigns them.

```python
class Plant:
    kingdom = "Plantae"            # class attribute: one copy, shared by all instances

    def __init__(self, name, water_days=7):
        self.name = name            # instance attributes: one per object
        self.water_days = water_days
        self.last_watered = None    # placeholder for "not yet"; unset attributes raise AttributeError

    def water(self):
        self.last_watered = "today"

p = Plant("fern")                   # no `new`; calling the class allocates and runs __init__
p.water()                           # sugar for Plant.water(p)
p.name                              # 'fern'
p.kingdom                           # 'Plantae', found on the class
p.height = 30                       # legal: attributes can be added from outside
```

**Class vs instance attributes.** Lookup goes instance dict first, then class, then base classes. Assigning `p.kingdom = "x"` creates an instance attribute that shadows the class one; it does not change the class. The trap is a mutable class attribute:

```python
class Bad:
    tags = []                       # shared by every instance

a, b = Bad(), Bad()
a.tags.append("x")
b.tags                              # ['x']  -- oops
```

Put per-object mutable state in `__init__`, always.

**No overloading.** One `__init__` per class. Use default arguments, `*args`/`**kwargs`, or a `@classmethod` factory (next section) for alternative constructors.

**Type annotations** are optional and not enforced at runtime; they are documentation plus input for static checkers like mypy:

```python
def __init__(self, name: str, water_days: int = 7) -> None:
```

## Methods and properties

Three kinds of method, chosen by what the first parameter is.

```python
class Plant:
    count = 0

    def __init__(self, name):
        self.name = name
        Plant.count += 1

    def describe(self):                 # instance method: gets the object
        return f"{self.name}"

    @classmethod
    def from_string(cls, text):         # class method: gets the class, not an instance
        name, _ = text.split(",")       # the standard way to write alternative constructors
        return cls(name)                # cls() so subclasses get their own type back

    @staticmethod
    def valid_name(text):               # static method: gets nothing; just a function namespaced in the class
        return text.isalpha()

p = Plant.from_string("fern,7")
Plant.valid_name("fern")                # True
```

C++ mapping: `@staticmethod` is a `static` member function. `@classmethod` has no C++ equivalent; it is a static function that knows which class it was called on, so it works correctly with inheritance.

**Properties** replace getter/setter pairs. The attribute syntax stays the same for callers, so you can start with a plain attribute and add logic later without breaking anyone.

```python
class Plant:
    def __init__(self, height_cm):
        self._height = height_cm        # storage, by convention not touched directly

    @property
    def height(self):                   # p.height  -> calls this
        return self._height

    @height.setter
    def height(self, value):            # p.height = 5  -> calls this
        if value < 0:
            raise ValueError("height must be non-negative")
        self._height = value

    @property
    def height_m(self):                 # read-only computed property: no setter defined
        return self._height / 100

p = Plant(30)
p.height = 40                           # goes through the setter
p.height_m                              # 0.4
p.height_m = 1                          # AttributeError: can't set attribute
```

## Visibility

There is no access control. Two naming conventions do the job that `private` and `protected` do in C++.

| Spelling | Meaning | What Python does |
| --- | --- | --- |
| `name` | Public API | Nothing |
| `_name` | Internal; roughly `protected` | Nothing, except `from m import *` skips it |
| `__name` | Internal, and must not collide with subclasses | Renamed to `_ClassName__name` inside the class body (name mangling) |
| `__name__` | Reserved for Python's own hooks (`__init__`, `__len__`) | Not mangled; never invent your own |

```python
class Plant:
    def __init__(self):
        self._cache = {}        # "please don't touch"; p._cache still works
        self.__id = 7           # stored as self._Plant__id

p = Plant()
p.__id                          # AttributeError
p._Plant__id                    # 7
```

Use `_name` by default. Use `__name` only when writing a base class and you want to guarantee a subclass cannot accidentally overwrite your attribute with one of the same name; mangling is about collisions, not secrecy.

## Inheritance

List base classes in parentheses. A subclass inherits everything; it overrides a method by defining one with the same name, and reaches the parent's version through `super()`.

```python
class Plant:
    def __init__(self, name):
        self.name = name

    def describe(self):
        return f"{self.name}"

class Succulent(Plant):
    def __init__(self, name, sun_hours):
        super().__init__(name)          # run the parent's __init__ first; NOT automatic
        self.sun_hours = sun_hours

    def describe(self):                 # override
        return super().describe() + f", {self.sun_hours}h sun"

s = Succulent("aloe", 6)
s.describe()                            # 'aloe, 6h sun'
isinstance(s, Plant)                    # True
issubclass(Succulent, Plant)            # True
type(s) is Plant                        # False: type() is exact, isinstance() follows the hierarchy
```

**Differences from C++ that bite:**

- The parent `__init__` is not called for you. Forget `super().__init__(...)` and the parent's attributes never exist.
- Every method is virtual. Calling `self.describe()` from a base-class method dispatches to the subclass version. There is no `final` keyword that is enforced (`@typing.final` is advice to type checkers only).
- There is no interface keyword. Any class with the right methods works wherever those methods are called; this is duck typing. `isinstance` checks are usually unnecessary and often discouraged.
- `super()` with no arguments is the normal form. It resolves to the next class in the method resolution order, which matters with multiple inheritance (later section).

**Constructor parameters are not inherited automatically.** If the subclass defines no `__init__`, it inherits the parent's whole constructor, parameters included. If it defines its own, the parent's is hidden and the subclass must accept the parent's parameters and forward them in `super().__init__(...)`. To avoid repeating a long parameter list, pass them through:

```python
class Succulent(Plant):
    def __init__(self, sun_hours, **kwargs):
        super().__init__(**kwargs)          # name, water_days, ... go straight to Plant
        self.sun_hours = sun_hours

Succulent(sun_hours=6, name="aloe")
```

Dataclasses are the exception: a subclass dataclass inherits the parent's fields and its generated `__init__` takes them first.

**Polymorphism** needs no special syntax:

```python
for plant in [Plant("fern"), Succulent("aloe", 6)]:
    print(plant.describe())             # each object's own version runs
```

**Prefer composition** when a class merely uses another rather than being a kind of it. `Garden` holds a list of `Plant`s; it does not inherit from `list`.

## Dunder methods

Operator overloading and language hooks are methods with double-underscore names. Python calls them; you never call them directly. Define the ones your type needs.

| You write | Python calls | Notes |
| --- | --- | --- |
| `Plant("fern")` | `__init__(self, ...)` | Constructor body (`__new__` allocates; rarely overridden) |
| `repr(p)`, REPL echo | `__repr__(self)` | For developers; aim for valid Python like `Plant('fern')`. Define this one always. |
| `str(p)`, `print(p)`, f-strings | `__str__(self)` | For users; falls back to `__repr__` if missing |
| `a == b` | `__eq__(self, other)` | Default compares identity. Defining it sets `__hash__` to None, so also define `__hash__` if instances go in sets or dict keys |
| `a < b`, `<=`, `>`, `>=` | `__lt__`, `__le__`, `__gt__`, `__ge__` | Define `__eq__` and one ordering, then `@functools.total_ordering` fills the rest |
| `hash(p)` | `__hash__(self)` | Return `hash((self.a, self.b))` over the fields `__eq__` uses |
| `len(p)` | `__len__(self)` | Also makes `bool(p)` false when 0 |
| `p[i]`, `p[i] = v`, `del p[i]` | `__getitem__`, `__setitem__`, `__delitem__` | `__getitem__` alone makes the object iterable and sliceable |
| `for x in p` | `__iter__(self)` | Return an iterator; often `return iter(self._items)` |
| `x in p` | `__contains__(self, x)` | Falls back to iterating if missing |
| `a + b`, `a * b`, `-a` | `__add__`, `__mul__`, `__neg__` | `__radd__` handles `3 + a`; `__iadd__` handles `a += b` (defaults to `__add__`) |
| `p()` | `__call__(self, ...)` | Makes instances callable, like a functor |
| `with p:` | `__enter__`, `__exit__` | Deterministic cleanup; the Python answer to RAII |
| `bool(p)`, `if p:` | `__bool__(self)` | Falls back to `__len__`, then True |

```python
from functools import total_ordering

@total_ordering
class Plant:
    def __init__(self, name, height):
        self.name, self.height = name, height

    def __repr__(self):
        return f"Plant({self.name!r}, {self.height})"

    def __eq__(self, other):
        if not isinstance(other, Plant):
            return NotImplemented           # not False: lets Python try other.__eq__
        return (self.name, self.height) == (other.name, other.height)

    def __hash__(self):
        return hash((self.name, self.height))

    def __lt__(self, other):
        return self.height < other.height

sorted([Plant("a", 30), Plant("b", 10)])   # works via __lt__
{Plant("a", 30)}                             # works via __hash__
```

Returning `NotImplemented` (not raising) from a binary dunder is how you say "I don't know how to compare with that type"; Python then tries the reflected operation on the other operand.

## Dataclasses, abstract classes, multiple inheritance

**Dataclasses** generate `__init__`, `__repr__` and `__eq__` from annotated fields. Use them for any class that is mostly data; they replace the boilerplate from the previous sections.

```python
from dataclasses import dataclass, field

@dataclass
class Plant:
    name: str
    water_days: int = 7
    tags: list = field(default_factory=list)   # never `= []`; see Gotchas

    def describe(self):                         # ordinary methods still allowed
        return f"{self.name} every {self.water_days}d"

p = Plant("fern")
p                                               # Plant(name='fern', water_days=7, tags=[])
Plant("fern") == Plant("fern")                  # True, field by field
```

Useful options: `@dataclass(frozen=True)` makes instances immutable and hashable; `order=True` generates the comparison operators; `slots=True` (3.10+) stores fields in fixed slots instead of a dict, like a C struct, saving memory and blocking typo attributes.

**Abstract base classes** are the closest thing to a pure virtual interface. A class with an unimplemented `@abstractmethod` cannot be instantiated.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Square(Shape):
    def __init__(self, s): self.s = s
    def area(self): return self.s ** 2

Shape()                                         # TypeError: can't instantiate abstract class
Square(2).area()                                # 4
```

Many Python codebases skip ABCs and rely on duck typing plus `typing.Protocol` (structural typing checked by mypy, nothing at runtime). Use an ABC when you want the runtime error.

**Multiple inheritance** is allowed and resolved by the method resolution order (MRO), a linearisation of the class graph (C3). `super()` follows the MRO, not "the parent", which is why it is written without arguments.

```python
class A:
    def hi(self): print("A")
class B(A):
    def hi(self): print("B"); super().hi()
class C(A):
    def hi(self): print("C"); super().hi()
class D(B, C):
    pass

D.__mro__          # (D, B, C, A, object)
D().hi()           # B, C, A  -- B's super() is C, not A; each class runs once
```

The common legitimate use is **mixins**: small classes with no state that add one behaviour (`class JSONMixin: def to_json(self): ...`), listed before the main base: `class Plant(JSONMixin, Base)`. Deep multiple-inheritance hierarchies are as painful here as in C++; composition is usually better.

## Gotchas for C/C++ programmers

1. **Assignment never copies.** `b = a` makes two names for one object. `b.x = 1` changes what `a` sees. Copy explicitly with `copy.copy(a)` (shallow) or `copy.deepcopy(a)`.
2. **Mutable default arguments are evaluated once**, at definition time, and shared across calls:

   ```python
   def add(item, bag=[]):      # same list every call
       bag.append(item); return bag
   add(1); add(2)              # [1, 2]
   ```

   Write `bag=None` and `if bag is None: bag = []` inside. Same trap for class attributes.
3. **Forgetting `self`.** `def water():` inside a class gives `TypeError: takes 0 positional arguments but 1 was given`. Inside methods, `x = 1` creates a local; `self.x = 1` sets the attribute.
4. **Forgetting `()` on a call.** `p.water` is the bound method object, which is truthy and never runs. `p.water()` runs it.
5. **`super().__init__()` is not implicit.** Skip it and the parent's attributes don't exist.
6. **`==` vs `is`.** `==` calls `__eq__`; `is` compares identity (pointer equality). Use `is` only for `None`, `True`, `False` and sentinels.
7. **No private, no const, no overloading, no enforced final.** Convention and tests replace the compiler. Type hints plus mypy get back some of what you lost.
8. **Attribute typos create attributes.** `self.watr_days = 5` silently adds a new field. `@dataclass(slots=True)` or `__slots__ = ("name", "water_days")` turns that into an `AttributeError`.
9. **No destructor discipline.** `__del__` runs when the refcount hits zero, which is usually but not guaranteed. Files, locks and connections use `with` (context managers), not destructors.
10. **Integer division and ints.** `7 / 2` is `3.5`; `7 // 2` is `3`; ints are arbitrary precision, no overflow.
11. **Everything is virtual, and the class is data.** You can replace a method on a class at runtime (`Plant.describe = other_fn`), add attributes to instances, and inspect `obj.__dict__` or `type(obj)`. Powerful for testing and metaprogramming; avoid in normal code.
12. **Indentation is syntax.** A method body ends when the indentation does. There is no `};` to tell you where a class stops.
