Title: Python Variables, Lists, and Dictionaries
Date: 2026-04-14
Category: GenAI
Tags: GenAI, LLM, Python
Slug: python-variables-list-dict
Status: draft

Python's three core building blocks — variables, lists, and dictionaries — are the foundation of everything you'll write. Get these right and everything else clicks into place.

## Variables

**Variable** — A named container that stores a value. Python is dynamically typed, so no type declaration needed — just assign and go.

```python
name = "Meena"
age = 25
is_active = True
```

**snake_case** — The Python convention for naming variables. Must start with a letter or underscore, not a number.

**Reassignment** — Variables can be reassigned to a different type at any time.

```python
x = 10
x = "now I'm a string"  # perfectly valid
```

**Type Checking** — Use `type()` to inspect what a variable holds.

```python
print(type(x))  # <class 'str'>
```

## Lists

**List** — An ordered, mutable collection. Items can be any type; duplicates are allowed.

```python
fruits = ["apple", "banana", "cherry"]
mixed = [1, "hello", True, 3.14]
```

**Indexing** — Zero-indexed. Negative indices count from the end.

```python
print(fruits[0])   # apple
print(fruits[-1])  # cherry
```

**Common Operations** — Built-in methods for everyday use.

```python
fruits.append("mango")       # add to end
fruits.insert(1, "grape")    # insert at index
fruits.remove("banana")      # remove by value
fruits.pop()                 # remove last item
print(len(fruits))           # length
```

**Slicing** — Extract a portion using `[start:end]` (end is exclusive).

```python
print(fruits[1:3])  # items at index 1 and 2
```

**List Comprehension** — Build lists concisely from iterables.

```python
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]
```

> Lists are great when order matters and you need to iterate or modify a sequence frequently.

## Dictionaries

**Dictionary** — An unordered collection of key-value pairs. Keys must be unique and immutable.

```python
person = {
    "name": "Meena",
    "age": 25,
    "city": "Bangalore"
}
```

**Accessing Values** — Use the key directly, or `.get()` to avoid a `KeyError` on missing keys.

```python
print(person["name"])               # Meena
print(person.get("email", "N/A"))   # N/A (safe fallback)
```

**Adding & Updating** — Assign directly to a key to add or overwrite.

```python
person["email"] = "meena@example.com"  # add
person["age"] = 26                     # update
```

**Removing Keys** — Use `del` or `.pop()`.

```python
del person["city"]
person.pop("age")
```

**Iterating** — Loop over keys, values, or both.

```python
for key, value in person.items():
    print(f"{key}: {value}")
```

**Dictionary Comprehension** — Build dicts concisely.

```python
squares = {x: x**2 for x in range(5)}
```

> Dictionaries are ideal when you need fast lookups by a meaningful key rather than a positional index.

## Quick Comparison

- Use a **variable** when storing a single value.
- Use a **list** when you have an ordered collection to iterate or modify.
- Use a **dict** when you need named key-value access.
