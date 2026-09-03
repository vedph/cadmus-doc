---
title: "Exporting JSON"
layout: default
parent: "Export"
nav_order: 4
---

# Exporting JSON

Exporting JSON differs from a [raw JSON export](json-dump) as it implies selecting and remapping source data into a different JSON schema. So here we are effectively _transforming_ source data into any other schema, as required by the consumer.

A typical application of this export process is a frontend designed around a set of predefined view models, specifically designed for it. Data is thus transformed into the target schema, similarly to what happens in XML via XSLT.

This export process shares its source data selection and mapping logic with the [graph](../graph/graph) exporter, which targets RDF, with an important difference:

- the graph exporter is designed to constantly map items into an RDF graph, keeping it in synch with user edits. Whenever users save their edits in the editor, data gets selected and mapped to target the graph according to the chosen ontologies. Users can even edit the target RDF graph directly to manually extend it.
- the JSON export process instead is not designed to export data in real-time. Rather, its purpose is to provide a massive JSON-based data transform and export, to be later consumed by a third-party client. So, the task of the JSON exporter is only to generate JSON documents, without caring about their usage. The graph exporter instead targets a graph database actively updating it to keep the graph in synch with user edits. Then, at any moment one can also export the nodes and triples of this graph into various standard formats to consume them as required.

While this difference applies to the output of each process, the logic to select the items and parts to export and map them into a different target is shared for these tasks, covered by [mappings](mappings):

- collecting source items to export.
- apply loaded mappings to project data from each source object's property node to the designed target.

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
      /* array of configurable objects */
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

## JSON Node Mapping

The JSON node mapping is derived from the [abstract node mapping](mappings#abstract-node-mapping) and targets a JSON object with any schema. Its core task is to transform a source JSON object into a target JSON object having a different schema.

This mapping **adds** these properties to the abstract node mapping:

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

> As Fluid does not JSON-encode its output by default, we use the mapper's built-in `json` filter (e.g. `{{ value | json }}`) to safely embed arbitrary values into it.

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

## Example

Here is a realistic example using a single item with its parts as source. To get such a JSON you can use the [JSON dump](json-dump) command in the [CLI tool](../../tools/cadmus-tool). The generated JSON code includes:

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
      "_id": "fcf6d7ac-ab97-462b-b9d8-d87ff53cec3b",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.pin-links",
      "roleId": null,
      "thesaurusScope": null,
      "content": {
        "links": [
          {
            "target": {
              "gid": "Syracusae",
              "label": "Syracusae",
              "itemId": null,
              "partId": null,
              "partTypeId": null,
              "roleId": null,
              "name": null,
              "value": null
            },
            "scope": "toponym-ancient",
            "tag": "origin",
            "features": null,
            "note": null,
            "assertion": null
          },
          {
            "target": {
              "gid": "Siracusa",
              "label": "Siracusa",
              "itemId": null,
              "partId": null,
              "partTypeId": null,
              "roleId": null,
              "name": null,
              "value": null
            },
            "scope": "toponym-modern",
            "tag": "origin",
            "features": null,
            "note": null,
            "assertion": null
          },
          {
            "target": {
              "gid": "places/462503",
              "label": "Syracusae",
              "itemId": null,
              "partId": null,
              "partTypeId": null,
              "roleId": null,
              "name": null,
              "value": null
            },
            "scope": "pleiades",
            "tag": "origin",
            "features": null,
            "note": null,
            "assertion": null
          }
        ],
        "id": "fcf6d7ac-ab97-462b-b9d8-d87ff53cec3b",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.pin-links",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0270304Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0530055Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.053Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "6338cbda-0cda-460e-bca1-addf7a70a8b6",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.epigraphy.technique",
      "roleId": null,
      "thesaurusScope": null,
      "content": {
        "grooveType": null,
        "techniques": ["execution:chiselled"],
        "tools": [],
        "note": null,
        "id": "6338cbda-0cda-460e-bca1-addf7a70a8b6",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.epigraphy.technique",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0271851Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0784502Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.078Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "83a128de-56ea-4ec5-a812-3e9c3f68797a",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.asserted-historical-dates",
      "roleId": null,
      "thesaurusScope": null,
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
        ],
        "id": "83a128de-56ea-4ec5-a812-3e9c3f68797a",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.asserted-historical-dates",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0269935Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.046236Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.026Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.046Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "e8f247c0-9374-4e95-a10a-a8badd1035c6",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.metadata",
      "roleId": null,
      "thesaurusScope": null,
      "content": {
        "metadata": [
          {
            "type": "string",
            "name": "eid",
            "value": "ISic000822"
          },
          {
            "type": "string",
            "name": "preservation-place",
            "value": "Tempio di Apollo"
          }
        ],
        "id": "e8f247c0-9374-4e95-a10a-a8badd1035c6",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.metadata",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0267433Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0356082Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.026Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.035Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "8ffcd758-c2b0-4350-b27f-dad6f57e4042",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.geo.asserted-locations",
      "roleId": null,
      "thesaurusScope": null,
      "content": {
        "locations": [
          {
            "tag": "origin",
            "value": {
              "eid": null,
              "label": "Syracusae",
              "latitude": 37.084150000000001,
              "longitude": 15.27628,
              "altitude": null,
              "radius": null,
              "geometry": null,
              "note": null
            },
            "assertion": null
          }
        ],
        "id": "8ffcd758-c2b0-4350-b27f-dad6f57e4042",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.geo.asserted-locations",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0270825Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.059378Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.059Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "ef14eb25-53fc-442f-bd28-83c21b6d9949",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.epigraphy.support",
      "roleId": null,
      "thesaurusScope": null,
      "content": {
        "material": "material:stone:limestone",
        "objectType": "object:arch_element:crepidoma",
        "features": null,
        "size": null,
        "textAreas": null,
        "counts": [],
        "note": null,
        "id": "ef14eb25-53fc-442f-bd28-83c21b6d9949",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.epigraphy.support",
        "roleId": null,
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0271248Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0665338Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.066Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "ca82d23f-e32a-4597-9a75-29a287003b16",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.token-text",
      "roleId": "base-text",
      "thesaurusScope": null,
      "content": {
        "citation": null,
        "lines": [
          {
            "y": 1,
            "text": "Κλεομ [c.3-5]ε\u0304ς ∶ ἐποίε\u0304σε το\u0304\u0313πέλο\u0304νι ∶ ͱο Κνιδ\u0323ι\u0323ε\u0323[ί]δ\u0323α ∶ τ\u0323ἐν\u0323τ\u0323ε\u0323[λ]ε\u0342 στύλεια ∶ κα[λὰ] ϝ\u0323έργ[α]"
          }
        ],
        "id": "ca82d23f-e32a-4597-9a75-29a287003b16",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.token-text",
        "roleId": "base-text",
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0273507Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0874133Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.087Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "1dc341ba-15a1-4a95-a2b0-1203a8b97238",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.categories",
      "roleId": "ins-lng",
      "thesaurusScope": null,
      "content": {
        "categories": ["grc"],
        "id": "1dc341ba-15a1-4a95-a2b0-1203a8b97238",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.categories",
        "roleId": "ins-lng",
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0273077Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0833095Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.083Z" },
      "userId": "zeus",
      "_status": 0
    },
    {
      "_id": "dc9e1d49-aa8a-4f1a-9a88-a90da11b8784",
      "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
      "typeId": "it.vedph.categories",
      "roleId": "ins-fn",
      "thesaurusScope": null,
      "content": {
        "categories": ["function:building", "function:dedication"],
        "id": "dc9e1d49-aa8a-4f1a-9a88-a90da11b8784",
        "itemId": "80b23b04-625d-4a7c-8cfa-d512034af643",
        "typeId": "it.vedph.categories",
        "roleId": "ins-fn",
        "thesaurusScope": null,
        "timeCreated": "2026-08-30T16:41:07.0271677Z",
        "creatorId": "zeus",
        "timeModified": "2026-08-30T16:41:07.0727355Z",
        "userId": "zeus"
      },
      "timeCreated": { "$date": "2026-08-30T16:41:07.027Z" },
      "creatorId": "zeus",
      "timeModified": { "$date": "2026-08-30T16:41:07.072Z" },
      "userId": "zeus",
      "_status": 0
    }
  ]
}
```

The item refers to an inscription and its parts are:

- `it.vedph.pin-links`: links targeting Pleiades places.
- `it.vedph.epigraphy.technique`: epigraphic technique ("chiselled").
- `it.vedph.asserted-historical-dates`: date(s) (600-551 BC).
- `it.vedph.metadata`: identifier (`ISic000822`) and preservation place (`Tempio di Apollo`).
- `it.vedph.geo.asserted-locations`: geographical coordinates (37.08415, 15.27628 for Syracusae).
- `it.vedph.epigraphy.support`: material support description ("limestone", "crepidoma").
- `it.vedph.token-text`: text in Leiden notation, line by line.
- `it.vedph.categories`:`ins-lng` listing the language(s) used in the inscription (`grc`=Ancient Greek in BCP47).
- `it.vedph.categories`:`ins-fn` listing the inscription's functions ("building", "dedication").

Let us imagine that we want to select and transform a subset of these data into this JSON output:

```json
{
  "projectIds": ["tes"],
  "title": "ISic000822",
  "summary": "Insediamento con necropoli nei Monti Iblei, documentato tra il VII e l'inizio del V secolo a.C.",
  "siteTypes": ["settlement", "necropolis"],
  "features": ["status-marker", "representations-figurative", "inscriptions"],
  "location": {
    "latitude": 36.9507789,
    "longitude": 14.6504504,
    "altitude": 650,
    "certainty": null
  },
  "chronology": [
    {
      "from": -700,
      "to": -475,
      "approximate": true,
      "dubious": false,
      "scope": "tes"
    }
  ]
}
```

Here is the corresponding export configuration:

{% raw %}
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
        "id": "fluid-filter.historical-date"
      },
      {
        "id": "fluid-filter.mongo-thesaurus"
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
      "description": "select categories:ins-fn entry values",
      "source": "_parts[?typeId=='it.vedph.categories' && roleId=='ins-fn'].content.categories",
      "output": "[{{ value | json }}]",
      "targetProperty": "functions"
    }
  ]
}
```
{% endraw %}

## Executing an Export

To execute a JSON export for data, use this commands in the [Cadmus CLI tool](../../tools/cadmus-tool):

- [export JSON](../../tools/cadmus-tool.md#export)
