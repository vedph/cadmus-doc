---
title: "Export"
parent: Migration 
layout: default
nav_order: 2
---

# Export

Currently, export functions target these scenarios:

- [rendering data](export/rendering/rendition.md). This renders data within the editor or into specialized outputs like TEI.
- [dumping raw JSON data](export/json-raw.md). This dumps the items with their parts into JSON. Apart from nesting parts inside the JSON of each item, there is no change in JSON schemas. This is used when you just need a massive or incremental data import into a system sharing the same schema
- [exporting data into custom JSON objects](export/json-export.md). This exports items with their parts into JSON documents having a different schema, mapping source to target as required. This is used to adapt data to any target, e.g. a frontend requiring specific view models.
- [exporting an RDF graph](export/graph/graph.md). This projects data into an RDF graph using any ontologies. The graph is kept in synch with editor data whenever they get updated.

Most of the migration functions are available from the [Cadmus CLI tool](../tools/cadmus-tool.md).
