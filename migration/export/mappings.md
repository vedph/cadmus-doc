---
title: "Mappings"
parent: "Export"
layout: default
nav_order: 2
---

# Mappings

The mapping between Cadmus source data (items and parts) and various export targets (JSON, RDF, etc.) is defined by node mappings. This is the core of the projection mechanism, which extracts a subset of source data from an object treated as a tree of properties. A mappings-based approach provides an export architecture which is:

- flexible, because it adapts to any source and target types;
- modular, because it splits a monolithic complex logic into many smaller pieces, which fit their source models;
- reusable, because scholars just provide mapping declarations, without having to write code, nor delving into the details of an imperative, step by step procedure.

Each mapping rule projects a small bit of data. These rules are very simple and small, but they can be organized in a _tree_ structure, which nicely fits the structure of Cadmus source data (objects, i.e. trees of properties). So, each mapping rule is the root of a tree structure, where each _node_ is a mapping rule.

## Node Mapping

A node mapping (`NodeMapping`) is a generic abstraction used to represent a node in the properties tree underlying an object. The mapping has:

- generic metadata;
- reference to parent and/or children mappings;
- source definition and filters to determine whether it is applicable to the currently processed source.

Each mapping selects a specific path in the source JSON object, and in turn can include children mappings which do the same. Mappings are recursively applied, so that you can keep the logic of each mapping very simple (as it targets a single path) while still being able to build a complex output structure by means of nested mappings.

This abstract mapping representation is then variously implemented by concrete mappings according to their output. Currently two such mappings exist:

- graph node mapping, targeting an RDF graph to be actively kept in synch with the data source.
- JSON node mapping, targeting any JSON schema and designed for one-pass data export.

## Abstract Node Mapping

The properties of each abstract mappings are:

- `id`: the node mapping ID. This is a positive integer number assigned when the mapping gets stored into a database.
- `parentId`: the parent node mapping ID, or 0 if no parent.
- `children`: children mappings. Each child mapping has the same properties of a root mapping, except for those which would make no sense in children, as noted above.
- `ordinal`: an optional ordinal value used to define the order of application of sibling mappings. Default is 0.
- `name`: a human-friendly name for the mapping.
- `description`: an optional, human-readable short description for the mapping rule. This is useful for documentation purposes.

- `sourceType`\*: the type of the source object. This is meaningful for _root_ mappings only. The source type is a number: `0`=user, `1`=item, `2`=part, `3`=thesaurus, `4`=implicit (assigned to nodes automatically added because used in a triple without yet being present in the graph). Thus, mappings defined in a mappings document effectively use only `1` and `2`.
- `facetFilter`: an optional item's facet filter. When specified, the mapping will target only those items whose facet ID is _equal_ to this value.
- `groupFilter`: an optional item's group filter. This is a regular expression; when specified, the mapping will target only those items whose group ID _matches_ this expression.
- `flagsFilter`: an optional item's flags filter. This is a numeric value, where each bit represents a flag. When specified, the mapping will target only those items whose flags include at least _all_ the bits set in this value, i.e. all the flags specified in the filter must be present.
- title filter
- `partTypeFilter`: an optional part's type ID filter. When specified, the mapping will target only those items whose part type ID is _equal_ to this value.
- `partRoleFilter`: an optional part's role ID filter. When specified, the mapping will target only those items whose part role ID is _equal_ to this value.
- `source`\*: the source expression representing the data selected by this mapping. In the current implementation this is a [JMES path](jmes-path). For instance, `events[?type=='person.birth']` matches only the entries in the `events` array property of a part's model whose type is equal to `person.birth`. When your source expression selects an object, you can refer to it as a whole with `.`, or to any of its properties by their name. When it selects an array, the mapper will implicitly loop through all its items, and be run for each of them. So, you can still define your mappings in terms of a single object, which here is the array's item. Additionally, the `index` metadatum will be used to represent the index number of the item in the array (0-N).
- `scalarPattern`: the optional regular expression pattern which should match against a scalar value defined by the mapping's source expression for the mapping to be applied. When this is defined and does not match, the mapping will not be applied. This can be used to overcome the limitations of the source expression in languages like JMESPath, where e.g. `.[?lost==true]` is always evaluated as a match, even when the value of the scalar property `lost` is `false`. So, in this example setting `scalarPattern` to `true` and source to `lost` will apply the mapping only if this property's value is `true`.
- `SID`\*: the source ID (SID) of this mapping. This is usually specified at the root mapping level, but you can also override the root SID in your children mappings (almost always this means adding suffix(es) to the root SID). Thus, SIDs are inherited unless overridden in descendants. If a SID includes the `index` metadatum, and the mapper is processing an array, it will be recalculated for each array's item.

### Source ID (SID)

The source ID (SID for short) is calculated so that _the same source always point to the same entity_. The SID is essential for connecting Cadmus data to the entities when keeping them in synch or just tracing their origin, as it provides the path by which data get added and updated.

The algorithm building the SID is idempotent; so you can run it any time being confident that the same input will always produce the same output. This is ensured by the fact that GUIDs are unique by definitions.

A SID is built with these components:

(a) for **items**:

1. the 36-characters _GUID_ of the source (item).
2. if the node comes from a group or a facet, the suffix `|group` or `|facet`. On passage, note that the group ID can be composite (using slash, e.g. `alpha/beta`); in this case, a mapping producing nodes for groups emits several nodes, one for each component. The top component is the first in the group ID, followed by its children (in the above sample, `beta` is child of `alpha`). Each of these nodes has an additional suffix for the component ordinal, preceded by `|`.

Examples:

- `76066733-6f81-48dd-a653-284d5be54cfb`: an entity derived from an item.
- `76066733-6f81-48dd-a653-284d5be54cfb|group`: an entity derived from an item's group.
- `76066733-6f81-48dd-a653-284d5be54cfb|group|2`: an entity derived from the 2nd component of an item's composite group.

(b) for **parts**:

1. the _GUID_ of the source (part).
2. if the part has a role ID, the _role ID_ preceded by `#`.

Examples:

- `76066733-6f81-48dd-a653-284d5be54cfb`: an entity derived from a part.
- `76066733-6f81-48dd-a653-284d5be54cfb#some-role`: an entity derived from a part with a role.

## Configuration

Mappings and possibly other parameters related to data export are defined in a JSON configuration document, like in this example:

```json
{
  "namedMappings": {
    "sample": {
      /* mapping */
    }
  },
  "mappings": [
    {
      "id": 0,
      "parentId": 0,
      "ordinal": 0,
      "name": "...",
      "sourceType": 0,
      "facetFilter": "...",
      "groupFilter": "...",
      "flagsFilter": null,
      "titleFilter": "...",
      "partTypeFilter": "...",
      "partRoleFilter": "...",
      "description": "...",
      "source": "...",
      "sid": "...",
      "scalarPattern": null,
      "children": [
        {
          "name": "sample"
        }
      ]
    },
  ]
}
```
