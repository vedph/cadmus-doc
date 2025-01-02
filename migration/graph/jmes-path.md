---
title: "Graph - JMES Path" 
layout: default
---

# JMES Path

- [tutorial](https://jmespath.org/tutorial.html)
- [examples](https://jmespath.org/examples.html)

This is just a cheatsheet derived from the excellent JMES tutorial, which also provides interactive examples.

## Basic Expressions

- identifier: `a` (null if not found)
- subexpression: `a.b` (cascaded null if not found)

## Lists

- index expression (list): `[1]` (0-based; negative=from end)
- slicing: `start:stop:step` (all optional; stop is exclusive, step defaults to 1): `[0:4]` = first three elements, equivalent to `[:4]`.

## Projections

Projections are evaluated as two steps:

- left hand side (LHS): creates a JSON array of initial values.
- right hand side (RHS): the expression to project for each element in the JSON array created by the LHS.

If the result of the expression projected onto an individual array element is null, then that value is omitted from the collected set of results.

There are 5 kinds of projections:

- list projections, via a wildcard expression: `people[*].first` = property `first` of each item in `people`.
- object projections, as above for objects: `ops.*.numArgs`.
- flatten Projections: `[]` flattens a list (not recursively, just one level) as generated e.g. from concatenating two list projections, which would result in a list of lists: `reservations[*].instances[].state`.
- slice Projections
- filter Projections: filters the LHS side before evaluating the RHS side: `LHS [?EXPR OP EXPR]`: `machines[?state='running'].name`.

You can stop a projection with a _Pipe Expression_: `people[*].first | [0]` = get 1st element if the list.

## Multiselect

Multiselect allows you to create elements that don’t exist in a JSON document. A multiselect list creates a list, and a multiselect hash creates a JSON object.

- `people[].[name, state.name]` = for each item in the list, create a list including the specified properties. Unlike a projection, the result of the expression is always included, even if the result is a null.
- `people[].{name: name, state: state.name}` = as above but the result will be an object for each item.

## Functions

A number of [functions](https://jmespath.org/specification.html#builtin-functions) is available for expressions. Functions can be combined with filter expressions, e.g.

```txt
myarray[?contains(@, 'foo') == `true`]
```

= finds all elements in `myarray` that contains the string `foo`.
