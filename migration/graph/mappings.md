---
title: "Graph - Mappings" 
layout: default
parent: Graph
nav_order: 2
---

# Graph Mappings

- [Graph Mappings](#graph-mappings)
  - [Sources](#sources)
  - [Identifiers](#identifiers)
    - [Source ID (SID)](#source-id-sid)
    - [Entry ID (EID)](#entry-id-eid)
    - [Entity ID (UID)](#entity-id-uid)
      - [UID Builder](#uid-builder)
  - [Mapping Rule](#mapping-rule)
  - [Templates](#templates)
    - [Expressions](#expressions)
    - [Node Keys](#node-keys)
    - [Metadata](#metadata)
    - [Macros](#macros)
    - [Filters](#filters)
  - [Sample](#sample)
    - [Sample Data](#sample-data)
    - [Sample Mappings](#sample-mappings)
    - [Sample Results](#sample-results)

The mapping between Cadmus source data (items and parts) and nodes is defined by node mappings.

## Sources

At the input side of mappings, there are two types of sources:

- **item**: an item. You can have mappings for the item; its group; its facet.
- **part**: a part.

Note that for _item titles_ a couple of conventions dictate that:

- if the title ends with `[#...]`, then the text between `[#` and `]` is assumed as the UID. The only processing is prepending the prefix defined in the mapping, if any.
- if the title ends with `[@...]`, then the text between `[@` and `]` is prefixed to the generated UID. If the mapping already defines a prefix, it gets prepended to this one.

## Identifiers

Before illustrating the mapping process, we must discuss the different identifiers used in it. In this document, these are referred to with `SID` (source ID), `UID` (entity's URI-based ID), and `EID` (entry's ID).

From the point of view of the mapping flow, it all starts with a SID, which specifies the source for the mapping. Mapping rules generate entities identified by UIDs. To this end, they may use the EIDs found in the source data.

### Source ID (SID)

The entity source ID (SID for short) is calculated so that _the same sources always point to the same entities_. The SID is essential for connecting Cadmus data to the entities and keeping them in synch, as it provides the path by which data get added and updated.

The algorithm building the SID is idempotent, so you can run it any time being confident that the same input will always produce the same output. This is ensured by the fact that GUIDs are unique by definitions.

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

### Entry ID (EID)

The "entry" ID is just a convention followed in models of Cadmus multi-entity parts.

A Cadmus part corresponding to a _single entity_ is a single "entry". In this case, its ID is simply provided by the part's item.

For instance, a person-information part inside a person item just adds data to the unique entity represented by the item (=the person). So, the target entity is simply the one derived from the item. There is no need for an EID here, because the item will already get its own identifier.

Conversely, a manuscript's decorations part is a collection of decorations, each corresponding to an entry, optionally having its EID (exposed via an `eid` property). All the entries with EIDs get mapped into entities.

Thus, here we call EIDs the identifiers provided by users for entries in a Cadmus collection-part. When present, such EIDs are used to build node identifiers URIs (UIDs).

>This is not the unique purpose of EIDs. In general, this convention provides a mechanism to set a human-friendly identifier for some entity contained in a data model. For instance, should the decorations of a manuscript include images, we could use their EIDs to name each image after them.

### Entity ID (UID)

The entity ID is a _shortened URI_ where (like in Turtle) a conventional prefix replaces the namespace, calculated as defined by the entity mapping.

To get relatively human-friendly UIDs, the UID is essentially derived from a _template_ defined in the mapping rule generating a node.

Yet, as we have to ensure that each UID is unique, whenever the template provides a result which happens to be already present and the mapping explicitly requests a unique UID, the UID gets a numeric suffix preceded by `#`. This suffix is granted to be unique in the context of our data.

>💡 By convention, any UID built by mapping must end with `##` to indicate that a unique UID is required. For instance, `itn:timespans/ts##` means that the first time such a UID is generated it will be stored as `itn:timespans/ts`; the next time, it will rather be suffixed with a number, e.g. `itn:timespans/ts#3`.

So, this mechanism ensures that the UID is unique, even though it is specified by users as a human-friendly identifier.

#### UID Builder

⚙️ TECHNICAL NOTE

The UID is built by a component implementing interface `IUidBuilder`. A RAM-based implementation of this UID (`RamUidBuilder`) is provided for testing.

In real world, the implementation relies on a RDBMS database. The table used for it is named `uid_lookup`, and has these fields:

- `id` PK AI: an autonumber primary key handled by the database engine.
- `sid`: the SID linked to the UID.
- `unsuffixed`: the UID as generated by the mapping rule, before any suffixation.
- `has_suffix`: true if the UID must be suffixed. If true, the numeric value to append to the suffix is represented by `id`. This trick allows us to efficiently leverage the autonumber capabilities of a RDBMS to append a unique numeric suffix to whatever UID.

So, the RDBMS based implementation of the builder looks in this table for an UID whose unsuffixed form is equal to that being handled. If none is found, the UID is used as such, without any suffix, and stored there with `has_suffix`=false.

If any is found but the caller requested a unique UID, the UID gets its `has_suffix` field set to true. This means that effectively the UID of the entity will be equal to the unsuffixed form + `#` + the value of `id`.

## Mapping Rule

A mapping rule is modeled as an object having a number of properties, defining:

- its _metadata_ (like source type, SID, description, etc.).
- its _input_ (used to match sources).
- its _output_ (which nodes and triples to emit).

In turn, each mapping rule can include any number of _children rules_. The model as it used in a JSON-based serialization is as follows:

- `sourceType`\*: the type of the source object. This is meaningful for _root_ mappings only. The source type is a number: `0`=user, `1`=item, `2`=part, `3`=thesaurus, `4`=implicit (assigned to nodes automatically added because used in a triple without yet being present in the graph). Thus, mappings defined in a mappings document effectively use only `1` and `2`.
- `sid`\*: the source ID (SID) of this mapping. This is usually specified at the root mapping level, but you can also override the root sid in your children mappings (almost always this means adding suffix(es) to the root SID). Thus, SIDs are inherited unless overridden in descendants. If a SID includes the `index` metadatum, and the mapper is processing an array, it will be recalculated for each array's item.
- `facetFilter`: an optional item's facet filter. When specified, the mapping will target only those items whose facet ID is _equal_ to this value.
- `groupFilter`: an optional item's group filter. This is a regular expression; when specified, the mapping will target only those items whose group ID _matches_ this expression.
- `flagsFilter`: an optional item's flags filter. This is a numeric value, where each bit represents a flag. When specified, the mapping will target only those items whose flags include at least _all_ the bits set in this value, i.e. all the flags specified in the filter must be present.
- `partTypeFilter`: an optional part's type ID filter. When specified, the mapping will target only those items whose part type ID is _equal_ to this value.
- `partRoleFilter`: an optional part's role ID filter. When specified, the mapping will target only those items whose part role ID is _equal_ to this value.
- `description`: an optional, human-readable short description for the mapping rule. This is useful for documentation purposes.

- `source`\*: the source expression representing the data selected by this mapping. In the current implementation this is a [JMES path](jmes-path). For instance, `events[?type=='person.birth']` matches only the entries in the `events` array property of a part's model whose type is equal to `person.birth`. When your source expression selects an object, you can refer to it as a whole with `.`, or to any of its properties by their name. When it selects an array, the mapper will implicitly loop through all its items, and be run for each of them. So, you can still define your mappings in terms of a single object, which here is the array's item. Additionally, the `index` metadatum will be used to represent the index number of the item in the array (0-N).
- `scalarPattern`:  the optional regular expression pattern which should match against a scalar value defined by the mapping's source expression for the mapping to be applied. When this is defined and does not match, the mapping will not be applied. This can be used to overcome the limitations of the source expression in languages like JMESPath, where e.g. `.[?lost==true]` is always evaluated as a match, even when the value of the scalar property `lost` is `false`. So, in this example setting `scalarPattern` to `true` and source to `lost` will apply the mapping only if this property's value is `true`.

- `output`:
  - `nodes`: an object (dictionary) where each property is the key of a node emitted by the mapping rule, whose string value is the node's identifier [template](#templates). Optionally, this template can be followed by space plus the node's label, and/or its tag between square brackets, the tag being preceded by `|`. For instance, `x:events/{$.} [label|tag]` defines the node's UID, its human-friendly label, and an optional tag.
  - `triples`: an array of strings, each representing a triple [template](#templates). Each triple is in any of these forms:
    - `S P O`: subject, predicate, object (all URIs);
    - `S P "O"`: subject, predicate, literal object in double quotes;
    - `S P "O"@lang`: subject, predicate, literal object in double quotes followed by an [ISO639](https://en.wikipedia.org/wiki/ISO_639) language identifier (e.g. `"sample"@en`);
    - `S P "O"^^type`: subject, predicate, literal object in double quotes followed by a type specifier (e.g. `"123"^^xs:int`).
  - `metadata`: optional metadata to be consumed in [templates](#templates). Metadata come from several sources: the source object, the mapping process itself, and these definitions in the mapping.

- `children`: children mappings. Each child mapping has the same properties of a root mapping, except for those which would make no sense in children, as noted above.

>Note: the _source type_ is a number where `0`=user, `1`=item, `2`=part, `3`=thesaurus, `4`=implicit (assigned to nodes automatically added because used in a triple without yet being present in the graph). This is not a closed enumerated value, so that you can optionally add new values by just defining new constants.

## Templates

Templates are extensively used in mappings to build node identifiers and triple values.

A template has any number of placeholders, delimited by `{}`, where the opening brace is followed by a single character representing the placeholder type:

1. `{@...}` = _expression_: this represents the expression used to select some source data for the mapping.
2. `{?...}` = _node key_: the key for a previously emitted node, optionally suffixed.
3. `{$...}` = _metadata_: any metadata set during the mapping process.
4. `{!...}` = _macro_: the output of a custom function, receiving the current data context from the source, and returning a string or null.

These placeholders can be freely nested. The mapping rules will take care of resolving them starting from the deepest ones.

>⚙️ The placeholder resolution is driven by a simple tree shaped representation of the template (`TemplateTree`).

### Expressions

- syntax: `{@...}`

Expressions select data from the source. The syntax of an expression depends on the mapper's implementation.

Currently the only implementation is JSON-based, so expressions are [JMES paths](jmes-path). This is a very powerful selection and transformation language, which should cover most of the mapping requirements.

For instance, say you are mapping an event object having an `eid` property equal to some string: you can select the value of this string with the placeholder `{@eid}`.

### Node Keys

- syntax: `{?...}`

During the mapping process, nodes emitted in the context of each mapping (including all its descendant mappings) are stored in a dictionary with the keys specified in the mapping itself for each node.

For instance, say your event object emits a node for each of its events. The mapping output for each node specifies an arbitrary key, used to refer to this node from other templates in the root mapping's context.

As a sample, consider this mapping fragment:

```json
{
  "id": "events.type=birth",
  "sourceType": "part",
  "partTypeFilter": "it.vedph.historical-events",
  "source": "events[?type=='person.birth']",
  "children": [
    {
      "source": "eid",
      "output": {
        "nodes": {
          "event": "x:events/{$.}"
        },
      }
    }
  ]
}
```

Here we map each birth event (as specified by `source`). For each of them, a child mapping matches the event's `eid` property, and outputs a node under the key `event`, whose template is `x:events/{$.}` (where `{$.}` is a [macro](#macros) representing the value of the current leaf node in the source tree). So, in this case the generated node will have an UID equal to `x:events/` plus the node's URI.

As a node is a complex object, in a template placeholder you can pick different properties from it. These are specified by adding a **suffix** preceded by `:` to the node's key. Available suffixes are:

- `:uri` = the node's generated URI. This is the default property; so when there is no suffix specified, the URI is picked.
- `:label` = the node's label.
- `:sid` = the node's SID.
- `:src_type` = the node's source type.

### Metadata

- syntax: `{$...}`

The mapping process can set some metadata, which get stored under arbitrary keys, and are available to any template in the context of its root mapping.

Metadata can be emitted by the mapping process itself, or be defined in a mapping's output under the `metadata` property. This is an object where each property is a metadatum with its string value.

Currently the mapping process emits these metadata:

- `item-id`: the item ID (GUID).
- `item-eid` (*): the EID of the item, as conventionally defined by the first matching metadatum with name = `eid` from the item's `MetadataPart`, if present. As this is the typical lookup mechanism, your consumer code can provide this additional metadatum by opting in via a metadata supplier.
- `part-id`: the part ID (GUID).
- `group-id`: the item's group ID.
- `facet-id`: the item's facet ID.
- `flags`: the item's flags.
- `.`: the value of the current leaf node in the source JSON data. For instance, if the mapping is selecting a string property from `events/event[0].eid`, this is the value of `eid`.
- `index`: the index of the element being processed from a source array. When the source expression used by the mapping points to an array, every item of the array gets processed separately from that mapping onwards. At each iteration, the `index` metadatum is set to the current index.

⚙️ Additionally, your backend code might use a metadata supplier with extra metadata sources to provide more metadata. A typical source is `ItemEidMetadataSource`, which adds these metadata:

- `item-eid`: the value of metadatum `eid` in the `MetadataPart` (if any) of the current item.
- `metadata-pid`: the part ID (GUID) of the metadata part (if any) of the current item.

> (*) As an example, see the Cadmus CLI tool code which by default opts into this metadatum with a code like this:

```cs
GraphUpdater updater = new(graphRepository)
  {
      // we want item-eid as an additional metadatum, derived from
      // eid in the role-less MetadataPart of the item, when present
      MetadataSupplier = new MetadataSupplier()
          .SetCadmusRepository(repository)
          // from Cadmus.Graph.Extras
          .AddItemEid()
  };
```

### Macros

- syntax: `{!...}`

Macros are a modular way for customizing the mapping process when more complex logic is required. A macro is just an object implementing an interface (`INodeMappingMacro`), requiring:

- the macro `Id` (an arbitrary string). This is used to call the macro from the template.
- the `Run` method, which runs the macro receiving the current data context, the placeholder position in the template and the template itself, and any arguments following the macro's ID; and returning a string or null.

The macro syntax in the placeholder is very simple: it consists of the macro ID, optionally followed by any number of arguments, separated by ` & `, included in brackets. For instance:

```txt
!{some_macro(arg1 & arg2)}
```

Some macros are **built-in**, and conventionally their ID start with an underscore. Currently there is only one:

- `_hdate(json,property)`: this macro handles a Cadmus historical date and returns either its sort value, or its human-friendly, machine-parsable text value. Its arguments are:
  1. the JSON code representing a Cadmus historical date;
  2. the property of the date to return: `value` (default) or `text`.

### Filters

Whenever a template represents a URI, i.e. in all the cases except for triple's object literals, once the template has been filled the result gets filtered as follows:

- whitespaces are replaced with underscores;
- only letters, digits 0-9, and characters `:-_#/&%=.?` are preserved;
- letters are all lowercased;
- diacritics are removed.

Should you want to disable this filtering, start the template with `!`, which being a preprocessing directive will be discarded from the template itself.

## Sample

As a sample, consider this historical events part. This contains any number of events, optionally with their place and/or time, and directly-related entities.

In this sample we have two events:

- the birth of Petrarch in 1304 at Arezzo from ser Petracco and Eletta Cangiani;
- the death of Petrarch at Arquà in 1374.

### Sample Data

Our data here come from a Cadmus part. Its serialized form (stripping out some unnecessary clutter) essentially is an array of two event objects, with their properties. The first event has type `person.birth`, the second has type `person.death` (these types come from a [thesaurus](../../models/thesauri)).

```json
{
  "events": [
    {
      "eid": "birth",
      "type": "person.birth",
      "chronotope": {
        "place": {
          "value": "Arezzo"
        },
        "date": {
          "a": {
            "value": 1304
          }
        }
      },
      "description": "Petrarch was born in 1304 at Arezzo from ser Petracco and Eletta Cangiani.",
      "relatedEntities": [
        {
          "relation": "mother",
          "id": "eletta_cangiani"
        },
        {
          "relation": "father",
          "id": "ser_petracco"
        }
      ]
    },
    {
      "eid": "death",
      "type": "person.death",
      "chronotope": {
        "place": {
          "value": "Arquà"
        },
        "date": {
          "a": {
            "value": 1374
          }
        }
      },
      "description": "Petrarch died in 1374 at Arquà."
    }
  ]
}
```

As you can see, each event in the `events` array is identified by an arbitrarily assigned [EID](#entry-id-eid), unique only in the context of this part's scope. Event types are drawn from a thesaurus.

Each event usually is connected to a place (`Arezzo`) and a date (`1304`). Also, it has a human-friendly description, and a list of related entities, each having a relation type (from another thesaurus), and the ID of the related entity.

So, here:

- the _first event_ represents an event of type birth, which took place at Arezzo in 1304, with a couple of related entities for the parents (mother and father). The person who was born (Petrarca) is implicit, as this events part is inside a person item, which represents that person.

- the _second event_ represents an event of type death, which took place at Arquà in 1374. Again, the person took out of existence by this event is implicit.

### Sample Mappings

We can arrange some basic mappings to project each event from this part into a node, with linked nodes for classification, attributes, and relations.

These mappings are encoded in a simple JSON document, which can be imported into the index database. In Cadmus, mappings are found in the index database (in table `node_mapping`); but it's easier to design them in a simple JSON document, to be later imported into it.

The JSON document is an array of mapping objects. Each mapping object has some properties, and optionally any number of children mappings, nested under their `children` property.

Here is a quick recap of mappings for the first event, reading the file from top to bottom:

👉 (1) the first mapping matches any event of type `person.birth` (see its `source` property: this is the JMES path). Its output is just a metadatum, which will be consumed by its children mappings. This has key `eid-sid` and is built from a template, collecting the part ID and the event's EID. So, all the children of this mapping start with their source located at the birth event.

```json
{
  "name": "birth event",
  "sourceType": 2,
  "facetFilter": "person",
  "partTypeFilter": "it.vedph.historical-events",
  "description": "Map birth event",
  "source": "events[?type=='person.birth']",
  "output": {
    "metadata": {
      "eid-sid": "{$part-id}/{@eid}"
    }
  }
}
```

👉 (2) the first child of this mapping matches the event's [EID](#entry-id-eid) property, and uses it to emit a node for that event. As a sample, its template has a prefix (`x:events/`, where `x:` stands for some URI namespace), and the EID from the event. These build the node's UID. This node is keyed under `event`: this key will be used by other mappings to refer to the UID of this node. As you know, the [UID](#entity-id-uid) cannot be known in advance, as it might receive a numeric suffix to disambiguate it. So, having a key for each emitted node allows us to refer to it in other mappings. Of course, the key is meaningful only in the scope of the process of this mapping. It will have no existence outside of the mapping process. Think of it as a sort of variable name, to be used in mapping templates.

```json
{
  "name": "birth event - eid",
  "source": "eid",
  "sid": "{$eid-sid}",
  "output": {
    "nodes": {
      "event": "x:events/{$.}"
    },
    "triples": [
      "{?event} a crm:E67_Birth",
      "{?event} crm:P98_brought_into_life {$item-uri}"
    ]
  }
}
```

👉 (3) the same mapping, once emitted the event node, uses it to build a couple of triples. One tells that this event is a birth event; and another that it brought into life the entity corresponding to the item containing this part. This item corresponds to the person who was born. Note that triples use node and metadata placeholders:

- `{?event}` represents the event's node UID, as generated by this mapping. Here we are using the node's key to refer to it, as the UID cannot be known in advance.
- `{$item-uri}` represents the UID of the item containing this part, i.e. the UID of the person who was born. This is a metadatum injected by the context, before the mapping process starts. Every object mapped - be it an item, a part, or a thesaurus - injects into the context some metadata, besides providing JSON code representing it.

👉 (4) the second child mapping matches the event's `note` property. It then emits a node representing a free textual annotation, and a corresponding triple using it. The triple links the event (referred to via its key) to the note's literal text (`{$.}` wraps a simple dot, which is the path to the current node; here, `note` being a string property, the current node is just the string's value, i.e. the note's text). Also notice that the UID here is just `x:notes/n`, which is generic as this node carries no intrinsic data we could use for a more meaningful user-friendly ID. The infrastructure will ensure that each additional note emitted gets a numeric suffix, thus producing sequences like `x:notes/n`, `x:notes/n#1`, `x:notes/n#23`, etc. (the actual numbers vary, the only requirement being that each is unique, so often they won't be progressive).

```json
{
   "name": "birth event - note",
   "source": "note",
   "sid": "{$eid-sid}/note",
   "output": {
     "nodes": {
       "note": "x:notes/n"
     },
     "triples": [ "{?event} crm:P3_has_note \"{$.}\"" ]
   }
}
```

👉 (5) the third child walks down the event's `chronotope` property, including the place and/or date. As such, it has no output, but it just provides an ad hoc SID and a number of children mappings. As you can see, here the hierarchy of mappings reflects the hierarchy of the object. That's a very intuitive way of designing such processes.

```json
{
  "name": "birth event - chronotope",
  "source": "chronotope",
  "sid": "{$eid-sid}/chronotope",
}
```

👉 (6) down to the `chronotope`'s mapping children, we have a couple of them, for `place` and `date`. The `place` mapping emits a place node, and a couple of triples telling that this is a place, and that the event took place there. The `date` mapping looks more interesting, as it requires a macro. We want to emit two nodes for each date: one with an approximate numeric value, calculated from the historical date model, and useful for processing data (for filtering, sorting, etc.); another with the human-friendly (yet parsable) representation of the date. So, the logic required for this could not be represented by the simple mapping model, which is purely declarative, and is bound to be simple for performance reasons. Rather, we use a macro, i.e. an external function, previously registered with the mapper (via the Cadmus data profile). Macro are pluggable components, so they represent an easy and powerful extension point. In this case, the macro `hdate` is used to calculate the values from the JSON code representing the historical date's model. The output is stored in a couple of metadata, and then used in the triples.

```json
"children": [
  {
    "source": "place",
    "output": {
      "nodes": {
        "place": "x:places/{@value}"
      },
      "triples": [
        "{?place} a crm:E53_Place",
        "{?event} crm:P7_took_place_at {?place}"
      ]
    }
  },
  {
    "name": "birth event - chronotope - date",
    "source": "date",
    "output": {
      "metadata": {
        "date_value": "{!_hdate({@.} & value)}",
        "date_text": "{!_hdate({@.} & text)}"
      },
      "nodes": {
        "timespan": "x:timespans/ts"
      },
      "triples": [
        "{?event} crm:P4_has_time_span {?timespan}",
        "{?timespan} crm:P82_at_some_time_within \"{$date_value}\"^^xs:float",
        "{?timespan} crm:P87_is_identified_by \"{$date_text}\"@en"
      ]
    }
  }
]
```

👉 (7) at this point, we're done with the `chronotope` property. The next mapping is child of the `event` object, and matches all the related entities whose `relation` property has value `mother`, i.e. the mother of this person. It outputs a node for her, using the received `id` with some prefix, just to show how we can still manipulate the received ID if needed, so users can enter a shortened version if useful. The corresponding triple links the event and the mother.

```json
{
  "name": "birth event - related - mother",
  "sid": "{$eid-sid}/related",
  "source": "relatedEntities[?relation=='mother']",
  "output": {
    "nodes": {
      "mother": "x:persons/{@id}"
    },
    "triples": [ "{?event} crm:P96_by_mother {?mother}" ]
  }
}
```

(8) finally, a sibling mapping does the same for the father.

Here is the full code for those mappings:

```json
[
  {
    "name": "birth event",
    "sourceType": 2,
    "facetFilter": "person",
    "partTypeFilter": "it.vedph.historical-events",
    "description": "Map birth event",
    "source": "events[?type=='person.birth']",
    "output": {
      "metadata": {
        "eid-sid": "{$part-id}/{@eid}"
      }
    },
    "children": [
      {
        "name": "birth event - eid",
        "source": "eid",
        "sid": "{$eid-sid}",
        "output": {
          "nodes": {
            "event": "x:events/{$.}"
          },
          "triples": [
            "{?event} a crm:E67_Birth",
            "{?event} crm:P98_brought_into_life {$item-uri}"
          ]
        }
      },
      {
        "name": "birth event - note",
        "source": "note",
        "sid": "{$eid-sid}/note",
        "output": {
          "nodes": {
            "note": "x:notes/n"
          },
          "triples": [ "{?event} crm:P3_has_note \"{$.}\"" ]
        }
      },
      {
        "name": "birth event - chronotope",
        "source": "chronotope",
        "sid": "{$eid-sid}/chronotope",
        "children": [
          {
            "source": "place",
            "output": {
              "nodes": {
                "place": "x:places/{@value}"
              },
              "triples": [
                "{?place} a crm:E53_Place",
                "{?event} crm:P7_took_place_at {?place}"
              ]
            }
          },
          {
            "name": "birth event - chronotope - date",
            "source": "date",
            "output": {
              "metadata": {
                "date_value": "{!_hdate({@.} & value)}",
                "date_text": "{!_hdate({@.} & text)}"
              },
              "nodes": {
                "timespan": "x:timespans/ts"
              },
              "triples": [
                "{?event} crm:P4_has_time_span {?timespan}",
                "{?timespan} crm:P82_at_some_time_within \"{$date_value}\"^^xs:float",
                "{?timespan} crm:P87_is_identified_by \"{$date_text}\"@en"
              ]
            }
          }
        ]
      },
      {
        "name": "birth event - related - mother",
        "sid": "{$eid-sid}/related",
        "source": "relatedEntities[?relation=='mother']",
        "output": {
          "nodes": {
            "mother": "x:guys/{@id}"
          },
          "triples": [ "{?event} crm:P96_by_mother {?mother}" ]
        }
      },
      {
        "name": "birth event - related - father",
        "sid": "{$eid-sid}/related",
        "source": "relatedEntities[?relation=='father']",
        "output": {
          "nodes": {
            "father": "x:guys/{@id}"
          },
          "triples": [ "{?event} crm:P97_by_father {?father}" ]
        }
      }
    ]
  },
  {
    "name": "death event",
    "sourceType": 2,
    "facetFilter": "person",
    "partTypeFilter": "it.vedph.historical-events",
    "description": "Map death event",
    "source": "events[?type=='person.death']",
    "output": {
      "metadata": {
        "eid-sid": "{$part-id}/{@eid}"
      }
    },
    "children": [
      {
        "name": "death event - eid",
        "source": "eid",
        "sid": "{$eid-sid}",
        "output": {
          "nodes": {
            "event": "x:events/{$.}"
          },
          "triples": [
            "{?event} a crm:E69_Death",
            "{?event} crm:P93_took_out_of_existence {$item-uri}"
          ]
        }
      },
      {
        "name": "death event - note",
        "source": "note",
        "sid": "{$eid-sid}/note",
        "output": {
          "nodes": {
            "note": "x:notes/n"
          },
          "triples": [ "{?event} crm:P3_has_note \"{$.}\"" ]
        }
      },
      {
        "name": "death event - chronotope",
        "source": "chronotope",
        "sid": "{$eid-sid}/chronotope",
        "children": [
          {
            "source": "place",
            "output": {
              "nodes": {
                "place": "x:places/{@value}"
              },
              "triples": [
                "{?place} a crm:E53_Place",
                "{?event} crm:P7_took_place_at {?place}"
              ]
            }
          },
          {
            "name": "death event - chronotope - date",
            "source": "date",
            "output": {
              "metadata": {
                "date_value": "{!_hdate({@.} & value)}",
                "date_text": "{!_hdate({@.} & text)}"
              },
              "nodes": {
                "timespan": "x:timespans/ts"
              },
              "triples": [
                "{?event} crm:P4_has_time_span {?timespan}",
                "{?timespan} crm:P82_at_some_time_within \"{$date_value}\"^^xs:float",
                "{?timespan} crm:P87_is_identified_by \"{$date_text}\"@en"
              ]
            }
          }
        ]
      }
    ]
  }
]
```

### Sample Results

The results of these mapping rules are a set of nodes with their triples. First we have the nodes for each entity:

- the birth event;
- the birth place;
- the birth time;
- the mother;
- the father;
- the death event;
- the death place;
- the death time.

label                  | URI                    | SID
-----------------------|------------------------|------------------------------------------------------
x:events/birth         | x:events/birth         | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth
x:places/arezzo        | x:places/arezzo        | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:timespans/ts         | x:timespans/ts         | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:guys/eletta_cangiani | x:guys/eletta_cangiani | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/related
x:guys/ser_petracco    | x:guys/ser_petracco    | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/related
x:events/death         | x:events/death         | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death
x:places/arqua         | x:places/arqua         | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope
x:timespans/ts#1       | x:timespans/ts#1       | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope

Then, these are their triples:

(a) for **birth**:

- the event is classified as a birth event;
- the event brought into life the entity corresponding to the item (=Petrarch);
- Arezzo is a place;
- the event has a timespan node;
- this timespan is located at about 1304,
- and is identified by text "1304 AD";
- the birth had Eletta Cangiani as the mother;
- the birth had ser Petracco as the father.

(b) for **death**:

- the event is classified as a death event;
- the event took out of existence the entity corresponding to the item (=Petrarch);
- Arquà is a place;
- the event has a timespan node;
- this timespan is located at about 1374.

S                | P                             | O                         | SID
-----------------|-------------------------------|---------------------------|------------------------------------------------------
x:events/birth   | a                             | crm:e67_birth             | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth
x:events/birth   | crm:p98_brought_into_life     | x:guys/francesco_petrarca | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth
x:places/arezzo  | a                             | crm:e53_place             | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:events/birth   | crm:p7_took_place_at          | x:places/arezzo           | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:events/birth   | crm:p4_has_time_span          | x:timespans/ts            | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:timespans/ts   | crm:p82_at_some_time_within   | "1304"^^xs:float          | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:timespans/ts   | crm:p87_is_identified_by      | "1304 AD"@en              | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/chronotope
x:events/birth   | crm:p96_by_mother             | x:guys/eletta_cangiani    | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/related
x:events/birth   | crm:p97_by_father             | x:guys/ser_petracco       | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/birth/related
x:events/death   | a                             | crm:e69_death             | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death
x:events/death   | crm:p93_took_out_of_existence | x:guys/francesco_petrarca | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death
x:places/arqua   | a                             | crm:e53_place             | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope
x:events/death   | crm:p7_took_place_at          | x:places/arqua            | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope
x:events/death   | crm:p4_has_time_span          | x:timespans/ts#1          | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope
x:timespans/ts#1 | crm:p82_at_some_time_within   | "1374"^^xs:float          | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope
x:timespans/ts#1 | crm:p87_is_identified_by      | "1374 AD"@en              | bdd152f1-2ae2-4189-8a4a-e3d68c6a9d7e/death/chronotope

So, these are the outcome of the mapping process. The user is not aware of all this: his only task is filling a form in a UI. This form lists events. Then, whenever he saves his work, the mapping process for the edited part steps in, and generates this graph of nodes. The graph will then be merged to the graph stored in the database.
