# Node Mapping

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

## Graph Node Mapping

TODO

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

You can use your custom filters in templates. Mappings and other parameters are defined in a JSON configuration document, like in this example:

```json
{
  "source": {
    "itemIdCollector": {
      "id": "it.vedph.item-id-collector.mongo"
    },
    "itemJsonReader": {
      "id": "item-json-reader.mongo"
    },
    "partFilter": {},
    "templateFilters": [],

  },
  "namedMappings": [],
  "mappings": []
}
```
