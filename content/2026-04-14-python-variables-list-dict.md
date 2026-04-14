Title: Python Basics — Variables, Lists, and Dictionaries
Date: 2026-01-01
Category: GenAI
Tags: GenAI, LLM, tag1, tag2
Slug: python-variables-list-dict
Status: draft

Python is the go-to language for AI, data science, and backend development — and it all starts with three fundamentals: variables, lists, and dictionaries. Get these right and everything else clicks into place.

## Variables

**Variable** — A named container that stores a value. Python is dynamically typed, so you don't need to declare a type — just assign and go.

```python
name = "Meena"
age = 25
is_active = True
```

**Naming Rules** — Use lowercase with underscores (`snake_case`). Names must start with a letter or underscore, not a number.

**Reassignment** — Variables can be reassigned to a completely different type at any time, since Python doesn't enforce types at runtime.

```python
x = 10
x = "now I'm a string"  # perfectly valid in Python
```

**Type Checking** — Use `type()` to inspect what a variable holds at any point.

```python
print(type(x))  # <class 'str'>
```

## Lists

**List** — An ordered, mutable collection of items. Items can be of any type and duplicates are allowed.

```python
fruits = ["apple", "banana", "cherry"]
mixed = [1, "hello", True, 3.14]
```

**Indexing** — Lists are zero-indexed. Use negative indices to access from the end.

```python
print(fruits[0])   # apple
print(fruits[-1])  # cherry
```

**Common Operations** — Lists come with built-in methods for everyday use.

```python
fruits.append("mango")       # add to end
fruits.insert(1, "grape")    # insert at index
fruits.remove("banana")      # remove by value
fruits.pop()                 # remove last item
print(len(fruits))           # length of list
```

**Slicing** — Extract a portion of a list using `[start:end]` (end is exclusive).

```python
print(fruits[1:3])  # items at index 1 and 2
```

**List Comprehension** — A concise way to build lists from existing iterables.

```python
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]
```

> Lists are great when order matters and you need to iterate or modify a sequence frequently.

## Dictionaries

**Dictionary** — An unordered collection of key-value pairs. Keys must be unique and immutable (strings, numbers, tuples).

```python
person = {
    "name": "Meena",
    "age": 25,
    "city": "Bangalore"
}
```

**Accessing Values** — Use the key inside square brackets, or `.get()` to avoid a `KeyError` on missing keys.

```python
print(person["name"])           # Meena
print(person.get("email", "N/A"))  # N/A (safe fallback)
```

**Adding & Updating** — Assign directly to a key to add or overwrite.

```python
person["email"] = "meena@example.com"  # add new key
person["age"] = 26                     # update existing key
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

**Dictionary Comprehension** — Build dicts concisely, similar to list comprehensions.

```python
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

> Dictionaries are ideal when you need fast lookups by a meaningful key rather than a positional index.

## Quick Comparison

- Use a **variable** when you're storing a single value.
- Use a **list** when you have an ordered collection of items to iterate or modify.
- Use a **dict** when you need to associate keys with values for fast, named access.
