---
title: "UI Import Tools" 
layout: default
parent: Migration
nav_order: 4

---

# UI Import Tools

The Cadmus editor UI provides import functions for the most used maintenance tasks:

- import [thesauri](../models/thesauri).
- import [settings](../models/settings).
- import facets.

>⚠️ IMPORTANT: after any of these imports, it is advisable to force a reload of the editor, as some resources (like facets) are loaded only when the app starts, and at any rate the browser might have cached stale data.

## Thesauri

You can import:

- a single thesaurus.
- multiple thesauri at once.

Thesauri can be imported from JSON, XLSX, XLS, or CSV files.

As thesauri in turn include multiple entries, and import granularity must be extended up to single entries, in the import UI you have 3 modes:

- **replace**: if the imported thesaurus already exists, it is fully replaced by the new one.
- **patch**: the existing thesaurus is patched with the imported one: any existing entry has its value overwritten; any non existing entry is just added.
- **synch**: equal to patch, with the addition that any existing entry not found in the imported thesaurus is removed.

To import thesauri, follow the [instructions](../models/thesauri#importing-thesauri).

## Settings

You can import:

- a single settings entry.
- multiple settings entries at once.

Settings are imported from JSON files with a single mode: every setting specified is either added if not existing, or updated if existing. All other settings are left unchanged.

Each setting is represented as a JSON object. The root element of the import JSON file can be:

- an array, including multiple JSON objects.
- a single JSON object.

In both cases, each JSON object has 1 property per settings. So, a JSON document like:

```json
{
  "it.vedph.taxo-store-nodes": {
    "treeId": "animals"
  }
}
```

will import a single settings object with ID `it.vedph.taxo-store-nodes`, whose content is this JSON object:

```json
{
  "_id": "it.vedph.taxo-store-nodes",
  "treeId": "animals"
}
```

## Facets

You can import:

- a single facet definition.
- multiple facet definitions at once.

Facets are imported with 2 modes:

- replace: just add facets which do not exist, and update those which exist. All the other facets are left intact.
- synch: remove all the existing facets and add the imported ones. This synchronizes the database with the import file.

Facets are imported from JSON files whose root element can be:

- an array, including multiple JSON objects, each representing a facet.
- a single JSON object, representing a facet.
