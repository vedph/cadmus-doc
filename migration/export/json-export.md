---
title: "Exporting JSON"
layout: default
parent: "Export"
nav_order: 4
---

# Exporting JSON

{% raw %}
Exporting JSON differs from a [raw JSON export](json-dump) as it implies selecting and remapping source data into a different JSON schema. So here we are effectively _transforming_ source data into any other schema, as required by the consumer.

A typical application of this export process is a frontend designed around a set of predefined view models, specifically designed for it. Data is thus transformed into the target schema, similarly to what happens in XML via XSLT.

This export process shares its source data selection and mapping logic with the [graph](../graph/graph) exporter, which targets RDF, with an important difference:

- the _graph exporter_ is designed to constantly map items into an RDF graph, keeping it in synch with user edits. Whenever users save their edits in the editor, data gets selected and mapped to target the graph according to the chosen ontologies. Users can even edit the target RDF graph directly to manually extend it.
- the _JSON exporter_ instead is not designed to export data in real-time. Rather, its purpose is to provide a massive JSON-based data transform and export, to be later consumed by a third-party client: for instance, a view-model for a web UI based on exported data. So, the task of the JSON exporter is only to generate JSON documents, without caring about their usage. The graph exporter instead targets a graph database actively updating it to keep the graph in synch with user edits. Then, at any moment one can also export the nodes and triples of this graph into various standard formats to consume them as required.

While this difference applies to the output of each process, the logic to select the items and parts to export and map them into a different target is shared for these tasks, covered by [mappings](mappings):

- collecting source items to export.
- apply loaded mappings to project data from each source object's property node to the designed target.

This page is a practical guide for people who have to write these mappings. Rather than just describing each property in isolation, it walks through the exact algorithm the mapper follows, so that you can predict what a mapping will produce before running it, and quickly diagnose it when it does not produce what you expected.

## Configuration

The export configuration is defined in a JSON document like this (see [mappings configuration](mappings.md#configuration) for the details about mappings):

```json
{
  "source": {
    "itemIdCollector": {
      "id": "it.vedph.item-id-collector.mongo",
      "options": {
        "pageNumber": 1,
        "pageSize": 20,
        "title": "...",
        "description": "...",
        "facetId": "...",
        "groupId": "...",
        "flags": null,
        "flagMatching": 0
      }
    },
    "itemJsonReader": {
      "id": "item-json-reader.mongo"
    },
    "partFilter": {
      "isInverted": false,
      "clauses": [
        {
          "typeId": "...",
          "roleId": "..."
        }
      ]
    },
    "templateFilters": [
      /* array of configurable objects, each with a "keys" property */
    ]
  },
  "namedMappings": {
    /* ... */
  },
  "mappings": [
    /* ... */
  ]
}
```

Each entry in `templateFilters` registers a [Fluid filter](https://github.com/sebastienros/fluid) (see [custom filters](#custom-filters) below) so it becomes usable from your mapping `output` templates. It must specify:

- `id`: the filter component's tag (e.g. `fluid-filter.historical-date`).
- `keys`: the Fluid filter keyword(s) it is invoked with in a template (e.g. `historical-date`, used as `{{ value | historical-date }}`).
- `options`: any options the filter requires (e.g. a connection string).

> ⚠️ **If you omit `keys`, the filter is silently never registered.** There is no error either at load time or when the template later references it: Fluid treats an unknown filter name as a no-op passthrough, not an error. So a missing `keys` typically shows up much later, as a value that just did not get transformed the way you expected.

## How Mapping Works: A Mental Model

Each `NodeMapping` (see [abstract node mapping](mappings#abstract-node-mapping)) matches a **source node**, and a `JsonNodeMapping` additionally renders an **output template** for it. Understanding exactly what happens between these two steps is the key to writing mappings that work on the first try. The algorithm, for each mapping, is:

1. **Select.** `source` is a [JMES path](jmes-path) evaluated against the mapping's current context: for a root mapping, that is the whole source object (item + parts); for a child mapping, it is whatever node its parent mapping matched (see [children mappings](#9-nested-children-mappings) below). The special value `.` means "the current context as a whole".
2. **React to what was selected**, one of four cases:
   - **nothing** (the path does not exist, or is JSON `null`): the mapping (and all its children) produce **no output at all**, silently. This is not an error, so a wrong `source` never throws — it just quietly leaves a hole in your output. See [pitfall #1](#pitfall-1-a-wrong-path-fails-silently-not-loudly).
   - **an array**: the mapper automatically **loops** over its items. The mapping (and its children) run once per item, and for that run the _item itself_ becomes the new current context — you never see the raw array. A per-iteration `index` metadatum (0-based) is also available. See [array recipes](#5-map-an-array-of-scalars-into-an-array).
   - **a scalar** (string, number, boolean): it is exposed as a **raw string**, regardless of its original JSON type. This is deliberate (so a string that happens to look like a number is not silently parsed as one), but it means numbers and booleans need care — see [pitfall #3](#pitfall-3-a-number-or-boolean-quoted-as-a-string).
   - **an object**: it is exposed as itself, with its own properties reachable by name.
3. **Render.** For an object or a scalar (not an array, which was already consumed by the loop in step 2), the `output` [Fluid template](https://github.com/sebastienros/fluid) is rendered with:
   - `value`: **exactly** what was selected in step 1/2 — nothing more, nothing less. Fluid never "reaches past" your `source` expression for you: if `source` selects a wrapper object, its inner fields are one level deeper than you might expect. See [pitfall #2](#pitfall-2-an-extra-value-wrapper).
   - `metadata`: the mapper's metadata dictionary (includes `index` while iterating an array, plus anything you or a macro added).
   - `sid`: the [SID](mappings#source-id-sid) resolved for this mapping, if any.
     The rendered text **must be valid JSON** on its own (a full object, or a full array). Because Fluid does not encode its output for JSON by default, always pass values through the built-in `json` filter before embedding them (e.g. `{{ value.name | json }}`), except where you deliberately want a raw, already-valid JSON literal (as when wrapping a whole selected object in `[ ... ]`, see the [array recipes](#5-map-an-array-of-scalars-into-an-array)).
4. **Merge.** The rendered JSON is merged into the shared target object, either at its root (when `targetProperty` is empty — in which case the rendered JSON **must** itself be an object) or under `targetProperty` (which can itself use the [placeholders](#placeholders-in-sid-and-targetproperty) below). Merging follows simple, consistent rules, applied every time a mapping's output lands on the same property (whether because several sibling mappings target it, or because an array made the same mapping run more than once):
   - object + object → **deep merge** (existing properties enriched or overridden by the new ones);
   - array + array → **concatenate** (new items appended);
   - anything else → the new value **replaces** the old one.
5. **Recurse.** If the mapping has `children`, each of them is matched again, but against the node selected in step 1 — not against the root. This is what lets you decompose a complex path into small, single-purpose mappings instead of one giant template (see [nested children](#9-nested-children-mappings)).

Steps 1–5 repeat, independently, for every mapping in your configuration (and every array item that triggers step 2's loop). There is no shared mutable "cursor" across mappings: each one starts fresh from its own `source`.

## Placeholders in `sid` and `targetProperty`

`targetProperty` (and, on any mapping, `sid`) is not rendered by Fluid — it goes through a small placeholder resolver supporting three forms, which can be mixed with literal text:

| Syntax          | Meaning                                                                                                               | Example                                             |
| --------------- | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `{$name}`       | a **metadatum**: looks up `name` in the mapper's `Data`/`metadata` dictionary                                         | `{$slot}` → the value of `mapper.Data["slot"]`      |
| `{@expr}`       | a **data expression**: a JMES path evaluated against the node currently selected by `source` (`.` for the whole node) | `{@type}` → the `type` property of the current node |
| `{!name(args)}` | a **macro**: a registered `INodeMappingMacro`, with `&`-separated arguments                                           | `{!_hdate(...)}`                                    |

For instance, a mapping whose `source` yields items with a `type` field can target a differently-named property per item without writing one mapping per type:

```json
{
  "source": "events[]",
  "output": "{ \"year\": {{ value.year | json }} }",
  "targetProperty": "{@type}"
}
```

> Note the different syntax families: `source`/`output` use JMES path and Fluid respectively, while `sid`/`targetProperty` use this placeholder mini-language. Mixing them up (e.g. writing a JMES path directly in `targetProperty`) is a common source of confusion — `targetProperty` only understands `{$...}`, `{@...}`, and `{!...}`.

## Cookbook

All the examples below use this small source object unless otherwise noted:

```json
{ "name": "Jane", "surname": "Doe" }
```

### 1. Copy a scalar property, renamed

```json
{
  "source": "name",
  "output": "{{ value | json }}",
  "targetProperty": "fullName"
}
```

→ `{ "fullName": "Jane" }`. `value` here is the raw string `"Jane"` (see step 2's scalar case); the `json` filter quotes it as a JSON string, which is what you want for an actual string.

### 2. Copy an object as-is, at the root

```json
{
  "source": ".",
  "output": "{ \"greeting\": \"Hello, {{ value.name }}!\" }"
}
```

→ `{ "greeting": "Hello, Jane!" }`, merged directly at the root because `targetProperty` is empty. This only works because the rendered output is itself an object (rule in step 4); if it were e.g. a bare string, the mapper would throw `InvalidOperationException` rather than guess where to put it.

### 3. Combine sibling mappings into the same target object

Two independent mappings can enrich the same object, thanks to the object+object deep-merge rule (step 4):

```json
[
  {
    "source": "name",
    "output": "{ \"person\": {\"firstName\": {{ value | json }} } }"
  },
  {
    "source": "surname",
    "output": "{ \"person\": {\"lastName\": {{ value | json }} } }"
  }
]
```

→ `{ "person": { "firstName": "Jane", "lastName": "Doe" } }`.

### 4. Build a nested object from several source fields

When several output fields come from the same source node, prefer a single mapping selecting the common ancestor, rather than several siblings:

```json
{
  "source": ".",
  "output": "{ \"address\": { \"street\": \"{{ value.street }}\", \"city\": \"{{ value.city }}\" } }"
}
```

Given source:

```json
{
  "street": "123 Main St",
  "city": "Anytown"
}
```

The output is:

```json
{ "address": {
  "street": "123 Main St",
  "city": "Anytown"
  }
}
```

### 5. Map an array of scalars into an array

Given `{ "tags": ["a", "b"] }`:

```json
{
  "source": "tags",
  "output": "[{{ value | json }}]",
  "targetProperty": "tags"
}
```

Here `source` selects the whole `tags` array, so per step 2 the mapper **loops**: the mapping runs twice, once per item, each time with `value` bound to a single tag string. Each run's output is a one-item array (`["a"]`, then `["b"]`); per step 4's array+array rule these get concatenated into the final `["a", "b"]`. This is why the template wraps a single item in `[ ... ]` rather than trying to render the whole array at once — by the time your template runs, the array is already gone, replaced by one of its items.

### 6. Map an array of objects, reshaping each item

Given `{ "events": [ { "type": "birth", "year": 1265 }, { "type": "death", "year": 1321 } ] }`:

```json
{
  "source": "events",
  "output": "[ { \"type\": {{ value.type | json }} } ]",
  "targetProperty": "events"
}
```

→ `{ "events": [ { "type": "birth" }, { "type": "death" } ] }`. Same looping/concatenation mechanism as above, but each item is an object so you can pick a subset of its properties (here, dropping `year`).

### 7. Pick a single item from an array

Indexing collapses the array back to a single node, so the mapper does **not** loop, and `value` is bound directly to that one item (an object, not an array):

```json
{
  "source": "events[0]",
  "output": "{ \"firstEventType\": {{ value.type | json }} }"
}
```

→ `{ "firstEventType": "birth" }`.

### 8. Numbers and booleans: getting real JSON types out

Given `{ "age": 42 }`, this looks reasonable but is wrong:

```json
{
  "source": "age",
  "output": "{ \"age\": {{ value | json }} }"
}
```

→ `{ "age": "42" }` — a **quoted string**, not a number. This is [pitfall #3](#pitfall-3-a-number-or-boolean-quoted-as-a-string): a directly-selected scalar is always a raw .NET string (step 2), and the `json` filter always JSON-encodes whatever it is given as a string when given a string, so a raw numeric-looking string becomes a quoted string. There are two correct fixes:

- select the parent object instead, and reach the field as a Fluid member access — this way the value passes through normal JSON parsing (which does preserve its type) rather than the scalar-string shortcut:

  ```json
  {
    "source": ".",
    "output": "{ \"age\": {{ value.age | json }} }"
  }
  ```

  → `{ "age": 42 }`, correctly numeric.

- or, if you really must select the scalar directly, drop the `json` filter and interpolate it bare: `{ "age": {{ value }} }` also produces `{ "age": 42 }`, because the raw text `42` is already valid (unquoted) JSON. This only works for numbers/booleans/`null` — never do this for a string value, since the result would not be valid JSON (`{ "name": Jane }`).

### 9. Nested (children) mappings

Instead of one large `output` reaching deep into a structure, split it into a parent mapping that narrows the context, and children that fill in the details (step 5). Given `{ "person": { "name": "Jane", "age": 42 } }`:

```json
{
  "source": "person",
  "children": [
    {
      "source": ".",
      "output": "{ \"fullName\": {{ value.name | json }} }"
    },
    {
      "source": ".",
      "output": "{ \"age\": {{ value.age | json }} }"
    }
  ]
}
```

Each child's `source` (`.`) is evaluated against the parent's selected node (`person`), not the document root — this is the mechanism that lets deeply nested schemas be expressed as a shallow tree of small, single-purpose mappings, mirroring the structure of the source data itself.

## Custom Filters

`JsonTemplateNodeMapper.Filters` is preloaded with Fluid's standard filters, including the `json` filter used throughout this page. Two Cadmus-specific ones ship with the exporter, both registered by tag under a `keys` entry in `source.templateFilters` (see [Configuration](#configuration)):

- `fluid-filter.historical-date` (typically keyed as `historical-date`): parses its input as a `HistoricalDate` (either a string like `"123 AD"`, or the JSON shape produced by serializing one) and returns either its numeric sort value (default), or, with a `"text"` argument, its textual form:

  ```json
  "output": "{ \"when\": {{ value | historical-date: 'text' | json }} }"
  ```

- `fluid-filter.mongo-thesaurus` (typically keyed as `mongo-thesaurus`, requires a `connectionString` option): scans its string input for `$[thesaurusId|entryId]` references and replaces each with the corresponding thesaurus entry's value, looked up in MongoDB (falling back through `@eng`/`@en`, and following alias thesauri). It leaves anything that is not a match untouched, so you typically build the reference around your raw code first:

  ```json
  "output": "[{{ value | prepend: '$[categories_ins-fn@en|' | append: ']' | mongo-thesaurus | json }}]"
  ```

  Given a source category code `function:building` and a thesaurus `categories_ins-fn@en` with an entry `function:building` → `building`, this produces `["building"]`.

You can also register your own filters in code, before calling `Map`:

```csharp
mapper.Filters.AddFilter("shout", (input, _, _) =>
    new ValueTask<FluidValue>(new StringValue(input.ToStringValue().ToUpperInvariant())));
```

**A misspelled or unregistered filter name is not an error.** Fluid treats it as a no-op: `{{ value | nope | json }}` on `"Jane"` silently renders `"Jane"`, exactly as if `| nope` had not been written at all — no exception, no warning, at parse time or at render time. This is [pitfall #4](#pitfall-4-an-unknown-filter-is-a-silent-no-op) below.

## Troubleshooting Checklist

These are, in order, the mistakes most likely to make a mapping produce the wrong thing while looking perfectly reasonable — each one drawn from an actual failure, not a hypothetical.

### Pitfall #1: a wrong path fails silently, not loudly

Per step 2, a `source` that matches nothing (or `null`) makes the mapping produce **no output**, without throwing. A `null`/missing field in your result is the mapper telling you "I found nothing here" — it is not a bug to work around downstream, it is a `source` (or a member access inside `output`) to fix. Before trusting an `output` template, verify what `source` actually selects, independently: paste your source JSON and the JMES path into an [online evaluator](https://jmespath.org/) (or a small unit test asserting on `value`'s shape) and confirm you get what you expect, _before_ writing the template around it.

### Pitfall #2: an extra `value` wrapper

Many Cadmus part models wrap their payload in an extra layer, e.g. a `value` property (an asserted location entry is `{ "tag": ..., "value": { "latitude": ..., "longitude": ... }, "assertion": ... }`, not `{ "latitude": ..., "longitude": ... }` directly).

Since `value` in your template is exactly what `source` selected, this means an extra `.value` hop is often needed: `value.value.latitude`, not `value.latitude`. There is no shortcut here other than checking the actual JSON shape of whatever `source` selects — different parts nest differently, and assuming they are all flat is the single most common mistake when writing a new mapping.

### Pitfall #3: a number or boolean quoted as a string

Covered in [recipe #8](#8-numbers-and-booleans-getting-real-json-types-out): a directly-selected scalar is always a raw string, so piping it through `json` always yields a quoted string, even for a source number or boolean.

### Pitfall #4: an unknown filter is a silent no-op

Covered in [custom filters](#custom-filters): a typo in a filter name, or forgetting to list it under `keys` in `source.templateFilters`, does not raise any error — the filter name is simply skipped, and the value flows through unchanged. If a `historical-date`- or thesaurus-resolved value looks suspiciously like the raw, untransformed source value, this is the first thing to check.

### Pitfall #5: `.` is not a valid Fluid expression

`source` and `targetProperty` both accept `.` to mean "the current node", but `output` is Fluid, which has no such shorthand — `{{ . | json }}` fails to parse (`A value was expected`). Always start a Fluid expression from `value`, `metadata`, or `sid`.

### Pitfall #6: mismatched `targetProperty` types across sibling/iteration outputs

Step 4's merge rules are forgiving between object+object and array+array, but anything else _replaces_ rather than merges. If one mapping (or one array iteration) unexpectedly produces a scalar or a differently-shaped object under a `targetProperty` that another mapping also targets, the second one silently overwrites the first rather than erroring — keep the shape of a given `targetProperty` consistent across every mapping (and every array iteration) that can write to it.

## Complete Worked Example

Here is a realistic example using a single item with its parts as source. To get such a **JSON input** you can use the [JSON dump](json-dump) command in the [CLI tool](../../tools/cadmus-tool). The generated JSON code includes:

- item's metadata.
- an additional `_status` property (this is used in JSON dump).
- an additional `_parts` property which is an array including all the parts in the item.

> Note that each part object includes its metadata and then a `content` property with its data, plus a copy of the part's metadata. This small redundancy is by design since the first versions of Cadmus, as it allows storage scenarios where part metadata are separated from content.

```json
{
  "_id": "80b23b04-625d-4a7c-8cfa-d512034af643",
  "title": "ISic000822",
  "description": "Insediamento con necropoli nei Monti Iblei, documentato tra il VII e l'inizio del V secolo a.C.",
  "facetId": "inscription",
  "groupId": null,
  "sortKey": "isic000822",
  "flags": 0,
  "timeCreated": { "$date": "2026-08-30T16:41:07.026Z" },
  "creatorId": "zeus",
  "timeModified": { "$date": "2026-08-30T16:41:07.027Z" },
  "userId": "zeus",
  "_status": 0,
  "_parts": [
    {
      "_id": "83a128de-56ea-4ec5-a812-3e9c3f68797a",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.asserted-historical-dates",
      "roleId": null,
      "content": {
        "dates": [
          {
            "tag": null,
            "assertion": null,
            "a": {
              "value": -600,
              "isCentury": false,
              "isSpan": false,
              "isApproximate": false,
              "isDubious": false,
              "day": 0,
              "month": 0,
              "hint": null,
              "slide": 0
            },
            "b": {
              "value": -551,
              "isCentury": false,
              "isSpan": false,
              "isApproximate": false,
              "isDubious": false,
              "day": 0,
              "month": 0,
              "hint": null,
              "slide": 0
            }
          }
        ]
      }
    },
    {
      "_id": "8ffcd758-c2b0-4350-b27f-dad6f57e4042",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.geo.asserted-locations",
      "roleId": null,
      "content": {
        "locations": [
          {
            "tag": "origin",
            "value": {
              "eid": null,
              "label": "Syracusae",
              "latitude": 37.08415,
              "longitude": 15.27628,
              "altitude": null,
              "radius": null,
              "geometry": null,
              "note": null
            },
            "assertion": null
          }
        ]
      }
    },
    {
      "_id": "dc9e1d49-aa8a-4f1a-9a88-a90da11b8784",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.categories",
      "roleId": "ins-fn",
      "content": {
        "categories": ["function:building", "function:dedication"]
      }
    }
  ]
}
```

> This is trimmed for readability: the full dump also repeats each part's metadata inside `content`, and includes several more parts (technique, support, text, language category, Pleiades links...). The [Item.json test asset](https://github.com/vedph/cadmus-core/blob/master/Cadmus.Export.Json.Test/Assets/Item.json) has the complete, untrimmed version, and every mapping below is verified against it.

The item refers to an inscription; the parts relevant to this example are:

- `it.vedph.asserted-historical-dates`: date (600-551 BC).
- `it.vedph.geo.asserted-locations`: geographical coordinates (37.08415, 15.27628 for Syracusae).
- `it.vedph.categories`:`ins-fn` listing the inscription's functions (`function:building`, `function:dedication`).

Let us imagine that we want to select and transform a subset of these data into this **JSON output**:

```json
{
  "title": "ISic000822",
  "summary": "Insediamento con necropoli nei Monti Iblei, documentato tra il VII e l'inizio del V secolo a.C.",
  "functions": ["function:building", "function:dedication"],
  "location": {
    "latitude": 37.08415,
    "longitude": 15.27628,
    "altitude": null,
    "certainty": null
  },
  "date": [
    {
      "text": "600 -- 551 BC",
      "scope": "tes"
    }
  ]
}
```

Here is the corresponding export configuration:

```json
{
  "source": {
    "itemIdCollector": {
      "id": "it.vedph.item-id-collector.mongo",
      "options": {
        "facetId": "inscription"
      }
    },
    "itemJsonReader": {
      "id": "item-json-reader.mongo"
    },
    "templateFilters": [
      {
        "id": "fluid-filter.historical-date",
        "keys": ["historical-date"]
      }
    ]
  },
  "mappings": [
    {
      "description": "title => title",
      "source": "title",
      "output": "{{ value | json }}",
      "targetProperty": "title"
    },
    {
      "description": "description => summary",
      "source": "description",
      "output": "{{ value | json }}",
      "targetProperty": "summary"
    },
    {
      "description": "categories:ins-fn codes => functions",
      "source": "_parts[?typeId=='it.vedph.categories' && roleId=='ins-fn'].content.categories",
      "output": "[{{ value | json }}]",
      "targetProperty": "functions"
    },
    {
      "description": "locations[0] => location.{latitude,longitude,altitude,certainty}",
      "source": "_parts[?typeId=='it.vedph.geo.asserted-locations'].content.locations[0]",
      "output": "{ \"latitude\": {{ value.value.latitude | json }}, \"longitude\": {{ value.value.longitude | json }}, \"altitude\": {{ value.value.altitude | json }}, \"certainty\": {{ value.assertion.rank | json }} }",
      "targetProperty": "location"
    },
    {
      "description": "asserted-historical-dates dates[0] => date[0].{text,scope}",
      "source": "_parts[?typeId=='it.vedph.asserted-historical-dates'].content.dates[0]",
      "output": "[{ \"text\": {{ value | historical-date: 'text' | json }}, \"scope\": \"tes\" }]",
      "targetProperty": "date"
    }
  ]
}
```

A few things worth pointing out about this configuration, tying back to the model above:

- the `location` mapping needs the extra `value.value.*` hop (pitfall #2), because `locations[0]` selects `{ "tag", "value": { "latitude", ... }, "assertion" }`, not the coordinates directly; and `value.assertion.rank` rather than a bare `assertion.rank`, since `assertion` is only reachable as a member of `value`, never as a top-level Fluid variable on its own.
- the `functions` mapping just maps the identifiers of each [thesaurus](../../models/thesauri.md) entry. In most cases this is what the consumer requires, because identifiers are the real data and their human-friendly, language-dependent labels vary according to your purpose. Anyway, Cadmus provides a lookup filter should you want to directly get the label from the ID of any thesaurus entry.
- `assertion` is `null` in the source, so `certainty` correctly comes out as `null` too — a missing/null member access does not throw, it just propagates as `null` (do not confuse this with pitfall #1, which is about `source` itself matching nothing: here `source` does match, and it is a specific _member_ of the matched node that is absent).
- the `date` mapping's `historical-date` filter is only usable because it was registered under `keys: ["historical-date"]` in `templateFilters` — dropping that line would silently turn `{{ value | historical-date: 'text' | json }}` into `{{ value | json }}` (pitfall #4), rendering the raw `{ "a": {...}, "b": {...} }` structure instead of `"600 -- 551 BC"`.
- both the `functions` and `date` mappings wrap a single rendered object in `[ ... ]`, even though their `source` does not end in `[0]` for `functions` (it loops instead — see [recipe #5](#5-map-an-array-of-scalars-into-an-array)) and does for `date` (it does not loop — see [recipe #7](#7-pick-a-single-item-from-an-array)); either way, the goal is an array under `targetProperty`, so the template must render one.

## Executing an Export

To execute a JSON export for data, use this command in the [Cadmus CLI tool](../../tools/cadmus-tool):

- [export JSON](../../tools/cadmus-tool.md#export)
  {% endraw %}
