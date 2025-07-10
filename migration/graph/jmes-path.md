---
title: "Graph - JMES Path" 
layout: default
parent: "Graph - Mappings"
nav_order: 1
---

# JMES Path

- [tutorial](https://jmespath.org/tutorial.html)
- [examples](https://jmespath.org/examples.html)

This is just a cheatsheet derived from the excellent JMES tutorial, which also provides interactive examples.

## Basic Expressions

- identifier: `a` (null if not found)
- subexpression: `a.b` (cascaded null if not found)

**Examples:**

```json
// Data: {"name": "John", "age": 30}
name        // → "John"
age         // → 30
address     // → null
```

## Lists

- index expression (list): `[1]` (0-based; negative=from end)
- slicing: `start:stop:step` (all optional; stop is exclusive, step defaults to 1): `[0:4]` = first three elements, equivalent to `[:4]`.

**Examples:**

```json
// Data: ["a", "b", "c", "d", "e"]
[0]         // → "a"
[-1]        // → "e"
[1:3]       // → ["b", "c"]
[:2]        // → ["a", "b"]
[::2]       // → ["a", "c", "e"]
```

## Projections

Projections extract values from arrays or objects and are the core power of JMES Path. They transform collections by applying expressions to each element.

Projections are evaluated as two steps:

- left hand side (LHS): creates a JSON array of initial values.
- right hand side (RHS): the expression to project for each element in the JSON array created by the LHS.

If the result of the expression projected onto an individual array element is null, then that value is omitted from the collected set of results.

There are 5 kinds of projections:

### List Projections

- **Syntax:** `array[*].property`
- **Purpose:** Extract property from each array element
- **Example:**

```json
// Data: [{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]
[*].name    // → ["John", "Jane"]
[*].age     // → [30, 25]
```

### Object Projections

- **Syntax:** `object.*.property`
- **Purpose:** Extract values from object properties
- **Example:**

```json
// Data: {"user1": {"name": "John"}, "user2": {"name": "Jane"}}
*.name      // → ["John", "Jane"]
```

### Flatten Projections

- **Syntax:** `[]`
- **Purpose:** Flattens nested arrays by one level
- **Example:**

```json
// Data: [["a", "b"], ["c", "d"]]
[]          // → ["a", "b", "c", "d"]
```

### Slice Projections

- **Syntax:** `array[start:stop:step]`
- **Purpose:** Create array slice then project
- **Example:**

```json
// Data: [{"id": 1}, {"id": 2}, {"id": 3}, {"id": 4}]
[1:3].id    // → [2, 3]
```

### Filter Projections

- **Syntax:** `array[?expression]`
- **Purpose:** Filter elements before projection
- **Example:**

```json
// Data: [{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]
[?age > `27`].name              // → ["John"]
[?name == 'Jane'].age           // → [25]
```

### Pipe Expressions

Stop a projection with a pipe to apply further operations:

```json
// Data: [{"name": "John"}, {"name": "Jane"}]
[*].name | [0]                  // → "John"
[*].name | length(@)            // → 2
```

## Multiselect

Multiselect allows you to create elements that don't exist in a JSON document. A multiselect list creates a list, and a multiselect hash creates a JSON object.

### Multiselect List

- **Syntax:** `[expr1, expr2, ...]`
- **Example:**

```json
// Data: [{"name": "John", "age": 30, "city": "NYC"}]
[*].[name, age]                 // → [["John", 30]]
```

### Multiselect Hash

- **Syntax:** `{key1: expr1, key2: expr2, ...}`
- **Example:**

```json
// Data: [{"name": "John", "age": 30, "city": "NYC"}]
[*].{person: name, years: age}  // → [{"person": "John", "years": 30}]
```

## Functions

A number of [functions](https://jmespath.org/specification.html#builtin-functions) is available for expressions. Functions can be combined with filter expressions.

**Common Functions:**

- `length(array)` - array/object/string length
- `keys(object)` - object keys
- `values(object)` - object values
- `contains(array, value)` - check if array contains value
- `starts_with(string, prefix)` - string starts with prefix
- `ends_with(string, suffix)` - string ends with suffix
- `join(separator, array)` - join array elements
- `reverse(array)` - reverse array
- `sort(array)` - sort array
- `sort_by(array, expression)` - sort by expression
- `max(array)` - maximum value
- `min(array)` - minimum value
- `sum(array)` - sum of numeric values
- `avg(array)` - average of numeric values
- `type(value)` - value type
- `to_string(value)` - convert to string
- `to_number(value)` - convert to number

**Examples:**

```json
// Data: [{"name": "John", "scores": [85, 90, 78]}, {"name": "Jane", "scores": [92, 88, 95]}]
[*].scores | [0]                    // → [85, 92]
[*].{name: name, avg: avg(scores)}  // → [{"name": "John", "avg": 84.33}, {"name": "Jane", "avg": 91.67}]
[?contains(name, 'J')].name         // → ["John", "Jane"]
[?length(scores) > `2`].name        // → ["John", "Jane"]
```

**Filter with Functions:**

```txt
myarray[?contains(@, 'foo') == `true`]
```

= finds all elements in `myarray` that contains the string `foo`.
