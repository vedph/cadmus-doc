---
title: "Dumping"
layout: default
parent: "Migration"
nav_order: 2
---

# Dumping

Currently Cadmus provides two ways of dumping its items data into JSON documents:

- [dump raw JSON data](json-raw.md): dump the items with their parts into JSON. Apart from nesting parts inside the JSON of each item, there is no change in JSON schemas. This is used when you just need a massive or incremental data import into a system sharing the same schema.
- [export JSON](json-export.md): export items with their parts into JSON documents having a different schema, mapping source to target as required. This is used to adapt data to any target, e.g. a frontend requiring specific view models.
