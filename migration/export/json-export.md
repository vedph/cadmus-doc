---
title: "Exporting JSON"
layout: default
parent: "Export"
nav_order: 3
---

# Exporting JSON

Exporting JSON differs from a [raw JSON export](json-raw.md) as it implies selecting and remapping source data into a different JSON schema. So here we are effectively transforming source data into any other schema, as required by the consumer.

A typical application of this export process is a frontend designed around a set of predefined view models, specifically designed for it. Data is thus transformed into the target schema, similarly to what happens in XML via XSLT.

This export process shares its source data selection and mapping logic with the [graph](../graph/graph.md) exporter, which targets RDF, with an important difference: the graph exporter is designed to constantly map items into an RDF graph, keeping it in synch with user edits. Whenever users save their edits in the editor, data gets selected and mapped to target the graph according to the chosen ontologies. Users can even edit the target RDF graph directly to manually extend it.

The JSON export process instead is not designed to export data in real-time. Rather, its purpose is to provide a massive JSON-based data transform and export, to be later consumed by a third-party client. So, the task of the JSON exporter is only to generate JSON documents, without caring about their usage. The graph exporter instead targets a graph database actively updating it to keep the graph in synch with user edits. Then, at any moment one can also export the nodes and triples of this graph into various standard formats to consume them as required.

While this difference applies to the output of each process, the logic to select the items and parts to export and map them into a different target is shared for these tasks:

- collecting source items to export.
- apply loaded mappings to project data from each source object's property node to the designed target.

For the purpose of exporting, a source JSON object can be considered as a **tree structure** where each node is a property, which in turn can branch into other nodes.

So, this is a hierarchical structure, similar to the document object model of an XML document. XML documents can be transformed into other XML documents via XSLT. In this case, the logic of the transformation is modular. Rather than having a monolithic approach, the logic is typically split into smaller bits, corresponding to templates:

- in most cases, each template gets applied when it matches a specific data path (expressed via XPath).
- the template specifies the output structure, filling it with data collected from its source.
- templates can be nested, either materially writing them inside another template, or (better) by recalling them at a given point in processing (e.g. via a generic `xsl:apply-templates` element)

Such templates are typically designed to be applied recursively, which is especially useful when the same logic must be applied to branches found in different positions.

The logic used to transform JSON objects in Cadmus is similar, and relies on **mappings** to ensure a modular approach:

- each mapping matches a specific data path, expressed via a [JMESPath](https://jmespath.org) (a query language for JSON).
- the mapping specifies the output structure, filling it with data collected from its source.
- mappings can be nested, i.e. each mapping can include children mappings, which in turn are matched against the source branch.

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
        "flagMatching": 0,
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
      /* array of configurable objects */
    ],
  },
  "namedMappings": {
    /* ... */
  },
  "mappings": [
    /* ... */
  ]
}
```

## JSON Node Mapping

The JSON node mapping targets a JSON object with any schema. Its core task is to transform a source JSON object into a target JSON object having a different schema.

This mapping adds these properties to the abstract node mapping:

- `output`: The [Fluid template](https://github.com/sebastienros/fluid) used to render the JSON fragment output by this mapping when it matches its `NodeMapping.Source`. The template is evaluated with:
  - `value`: the JSON value currently selected by this mapping (an object exposes its properties; an array its items; a scalar is exposed as a plain string);
  - `metadata`: the mapper's metadata dictionary;
  - `sid`: the SID resolved for this mapping, if any.
- `targetProperty`: the name of the property of the root JSON object under which the output of this mapping is to be merged. This can contain the same placeholders used by SID (metadata, data expressions, macros). When null or empty, the output (which must then be a JSON object) is merged directly into the root JSON object. Multiple mappings can target the same property: when they both output objects they get deep-merged (existing properties get enriched or overridden), when they both output arrays the new items get appended, otherwise the newer output replaces the older.

Examples (to keep examples simple, we use abstract JSON objects, not true Cadmus source objects):

- **object**: consider this JSON object:

```json
{
  "name": "Jane",
  "surname": "Doe"
}
```

Assume that we have 2 mappings with these properties:

- mapping 1:
  - `source`: `name`
  - `output`: `{ "person": {"firstName": {{ value | json }} } }`
- mapping 2:
  - `source`: `surname`
  - `output`: `{ "person": {"lastName": {{ value | json }} } }`

>As Fluid does not JSON-encode its output by default, we use the mapper's built-in `json` filter (e.g. `{{ value | json }}`) to safely embed arbitrary values into it.

This will output this JSON object:

```json
{
  "person": {
    "firstName": "Jane",
    "lastName": "Doe"
  }
}
```

- **array**: consider this JSON object:

```json
{
  "events": [
    {
      "type": "birth",
      "year": 1265
    },
    {
      "type": "death",
      "year": 1321
    }
  ]
}
```

Mapping:

- `source`: `events`
- `output`: `[ { \"type\": {{ value.type | json }} } ]`
- `targetProperty`: `events`

Result:

```json
{
  "events": [
    {
      "type": "birth"
    },
    {
      "type": "death"
    }
  ]
}
```

## Executing an Export

To execute a JSON export for data, use this commands in the [Cadmus CLI tool](../../tools/cadmus-tool.md):

- [export JSON](../../tools//cadmus-tool.md#export)
