---
title: "Links" 
layout: default
parent: "Linking Data"
nav_order: 3
---

# Links

An essential part of data linking in Cadmus is represented by linking its data to external or internal resources.

Linking to **external** resources essentially means adding a reference to the identifier of that resource in its repository. For instance, the URI of a LOD entity, the URL of a web page, a VIAF numeric ID, etc. Often Cadmus provides lookup facilities for this.

Linking to **internal** resources is done to link items or specific entities in an item's part to some other item or part's entity in the context of the same Cadmus database.

## External Links

For easier data entry, most of external links can leverage lookup providers.

Each of these providers (for VIAF, Zotero, WHG, GeoNames, DBPedia, MUFI, etc.) lets you quickly find the target resource by typing any portion of its human-friendly name.

## Internal Links

Internal links are used to connect items or entities in items parts. In both cases the link is based on **data pins**.

Any part for indexing purposes exposes a set of data pins, which are name/value pairs representing the most relevant data in it.

When linking to a part's entity, we need a way to address it inside the part's model. Given that the part's model can be anything, the Cadmus infrastructure cannot know about it. The part is like a blackbox. That's why data pins are used: we link parts via pins, which are just strings emitted by each part.

### Lookup Definition Links

The definitions-based lookup is the simplest and most traditional Cadmus way of looking up items or part entities. It is implemented by a `LookupPinComponent` UI widget, which allows users to type any part of the target's identifier, and get its item ID, part ID, role ID, part's type ID, and pin's name and value.

This kind of lookup requires a **lookup definition** to be specified at design time. This definition contains:

- the _part's type ID_, to filter the pins in the index, so that lookup only searches among those exposed by a specific part. Additionally, a _part role ID_ might also be required when reusing the same part type for different purposes.
- a _pin name_, to target the specific piece of data we want to pick from the part.

Both these parameters are defined at design time under an arbitrary key for each part. Also, they are just names, and carry no part-specific structure.

For instance, a conventional way for looking up whole items is to add a `MetadataPart` to them and add in it a metadatum named `eid` (=entity ID), providing a human-friendly identifier for that item. This is similar to what you do in TEI when you add an `xml:id` attribute to an element to be able to reference it.

By convention, this `eid` metadatum, which generates a data pin for the `MetadataPart`, is used as the anchor to target the whole item.

## Composite Links

A more advanced and powerful way of linking data in Cadmus is provided by composite links, implemented in the [asserted composite ID brick](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-refs-asserted-ids/README.md).

>Demo: <https://cadmus-bricks-v3.fusi-soft.com/refs/asserted-composite-id>

This is now the preferred way of representing links in items (when the brick is wrapped in a `PinLinksPart`) or inside a part (using the brick directly), because it allows to use the same model and UI for any target: external, internal and [taxonomy](taxonomies.md) links.

### Composite Links: External Targets

An external target ID is just a string, representing the identifier of the target resource. Usually this is globally unique, or it is unique only within the scope defined by its context. For instance, a DBPedia ID is a URI, which makes it a globally unique identifier; while the numeric ID assigned to an entity in a RDBMS is unique only in the scope of its table in that database.

According to their type and purpose, such IDs can be totally opaque for human users, like a number; or provide a hint about their target, like e.g. the URI for Marco Polo in DBPedia: <http://dbpedia.org/resource/Marco_Polo>.

So, in its most complete model an external ID has:

- GID: a value for the ID itself. In many cases this can be looked up using specific services for each provider (VIAF, GeoNames, DBPedia, IconClass, Biblissima+, etc.).
- a human-friendly label conventionally attached to the GID.
- an optional scope (e.g. `viaf` for a VIAF ID, which is just a number).
- an optional tag, which can be used to group or classify IDs in some way.
- an optional set of features, drawn from a thesaurus.
- an optional assertion, possibly used to define the uncertainty level of the assignment of this ID to the context it applies to.

An external ID can also come from a **taxonomy**. In this case:

- the GID is built as `@TX:<TREEID>/<NODEKEY>` where `@TX:` is a fixed prefix (meaning the GID refers to a taxonomy), `<TREEID>` is the tree's ID (an arbitrary ID), and `<NODEKEY>` the taxonomy node's key. The node's key is a string unique only within its tree; that's why we build a global ID by prefixing its tree ID.
- the label comes from the node's label.

### Composite Links: Internal Targets

Linking internal targets always relies on data pins.

For internal lookup, with thousands of parts or fragments providing dozens of pins, you quickly end up with a lot of them. So, to ease their **lookup** in this brick, you can filter them using two modes:

- _by item instance_ (option `pinByTypeMode`=false): lookup pins filtered by a specific item (via its assigned `eid` in its `MetadataPart`), and optionally by any of its parts. This is the default mode, as in most cases users have a top-bottom approach and think first of the item they want to target, and then, possibly, to a specific portion of its data (unless they are just happy to target the item as a whole).
- _by part type_ (option `pinByTypeMode`=true): directly lookup by pin value, in the context of a specific part type and pin namem, as specified by index lookup definitions. The list of part types may come from several sources:
  - explicitly set via the component `lookupDefinitions` property;
  - if this is not set, the lookup definitions will be got via injection when available. This is the traditional mechanism already described for internal links;
  - if the injected definitions are empty, the lookup definitions will be built from the model-types thesaurus;
  - if this is not available either, the by-type lookup will be disabled.

Filtering by item essentially means filtering by an _object instance_: for instance a specific manuscript object.

Filtering by type instead means filtering by _pin class_, as each part or fragment provides its own set of search pins, whose names are meaningful and unique only in their context.

### Composite Links Customization

Whatever the target, the pin name is not globally unique. A pin is unique only in the scope of its part or fragment.

Yet, items and parts always have a globally unique identifier (GUID), like e.g. `30ed7d3d-a70f-4254-a611-8cc1872f10d5`. This is the key to make a link made via pin names globally unique.

Once the pin gets connected with a part and/or an item, the GUID of that item/part ensure that an identifier based on that pin will be globally unique, too.

Of course, while this is granted to be unique, it's not user friendly at all. So, joining a GUID got automatically with a human-friendly name from a data pin gets the best of both worlds: the resulting ID is globally unique, and yet you build it by just looking up human-friendly identifiers, shorter and more readable, like `angel1`.

Internally, a composite link gets stored with its own structure (here INT marks properties present only for internal targets):

- target (`PinTarget`):
  - gid\* (`string`)
  - label\* (`string`)
  - itemId (`string`)
  - partId (`string`)
  - partTypeId (`string`)
  - roleId (`string`)
  - name (`string`) INT
  - value (`string`, starts with `@TX:` for taxonomies) INT
- scope (`string` 📚 `asserted-id-scopes`)
- tag (`string` 📚 `asserted-id-tags`)
- features (`string[]`, 📚 `asserted-id-features`, hierarchical)
- note (`string`)
- assertion (`Assertion`):
  - tag (`string` 📚 `assertion-tags`)
  - rank\* (`short`)
  - references (`DocReference[]`):
    - type (`string` 📚 `doc-reference-types`)
    - tag (`string` 📚 `doc-reference-tags`)
    - citation (`string`)
    - note (`string`)
  - note (`string`)

Optionally, users can customize both gid and label.

As for gid, users have access to a full set of metadata about the target, so that they can build their own global ID.

Once a pin value is picked, the lookup control shows all the relevant data which can be used as components for the ID to build:

- the item GUID.
- the item title.
- the part GUID.
- the part type ID.
- the item's metadata part entries.

The user can then use buttons to append each of these components to the ID being built, and/or variously edit it. When he's ok with the ID, he can then use it as the reference ID being edited.
