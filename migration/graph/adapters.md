---
title: "Graph - Adapters"
layout: default
parent: Graph
nav_order: 1
---

# Graph Adapters

To allow for more generic reuse, the [mappings](mappings) used to project data on the semantic graph are designed to be weakly coupled to the type of their source objects.

For this reason, an intermediate layer is added between source objects and mappings, in the form of _adapters_.

A _graph source adapter_ is a component implementing an interface (`IGraphSourceAdapter`), which dictates that the component must have a function that:

- gets as input an item or a part, together with a dictionary of additional metadata;
- outputs an object (or null) adapted to be plugged into the mapping process, plus a filter for node mappings.

Currently, the implementation of the adapter layer is limited to the components required by a JSON-based mapper; this is a very general scenario, where any type of object, serialized as JSON, gets mapped into a set of nodes and/or triples. So, this should cover the majority of usage cases.

These adapters have three main tasks:

- _serialize_ the received object into JSON, so that it can be processed by the JSON mapper.
- build the _filter_ used to match the mapping rules to be applied.
- inject some _metadata_ in the received dictionary, extracting them from the source object. Such metadata are typically referenced by mapping rules to do their work. For instance, the item graph adapter extracts from an item its title, and saves it under a constant key. From there, any mapping rule will be able to refer to such [metadata](mappings.md#metadata-).

Thus, whenever a source object enters the mapping process (e.g. because an item or a part has been saved):

1. the correspondent adapter is used to prepare it, while also providing mappings filter and metadata;
2. once this is done, the mapper can run all the matching mappings, and build the output graph set.
