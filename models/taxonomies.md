---
title: "Models - Taxonomies" 
layout: default
parent: Models
nav_order: 5
---

# Taxonomies

In most cases, taxonomies in Cadmus are relatively static and small data, defined via [thesauri](thesauri.md). Some projects anyway have more specialized requirements for specific taxonomies which could not be optimally satisfied by thesauri alone.

For instance, consider a huge taxonomy about iconographic subjects, including a deep hierarchy of thousands of entries. In this scenario, we would have these requirements:

- manage large data with good performance: usually these taxonomies are large, while thesauri are designed for relatively small set of flat or hierarchical data.
- edit with high granularity: thesauri are edited as a set, while in the example scenario users might want to edit single entries from a taxonomy. For instance, it might happen that while describing an iconography, users do not find a proper term in the taxonomy, and would want to add a new one. In this case, we need per-entry editing, as such taxonomy is large and shared, so that it might even be edited concurrently.

In these cases, Cadmus can adopt a [taxonomies store](https://github.com/vedph/taxo-store). This is an independent subsystem, which could be used even outside Cadmus, backed by a PostgreSQL database including trees and nodes.

In this store, each taxonomy is a tree with 1 or more root nodes; so it could be a flat list as well as a hierarchical list (which happens in most cases). Users can browse and find nodes in a tree, and edit it. To provide an easy experience, each tree (=taxonomy) has a string identifier and a human-friendly name, and each node has a string key and a human-friendly label. Internally, nodes use numeric IDs for performance reasons; but the software layer interacting with the database provides access to nodes via keys, which are designed to be unique within a single tree.

Thus, to provide a global node identifier, the convention is using the tree ID (a string) + `/` + the node's key (a string). While working in the scope of a single tree, of course a node key is enough.

The taxonomies subsystem provides:

- full backend logic to manage taxonomies.
- API endpoints ready to be integrated in your API.
- [frontend components](https://github.com/vedph/taxo-store-shell) to integrate in your frontend. Cadmus provides a part editor based on these components.
- a Cadmus part, [taxonomies store tree nodes part](https://github.com/vedph/taxo-store/blob/master/docs/taxo-store-nodes.md), provides a ready to use part which allows user to pick nodes from any specific tree, while also editing it.

## Data Source

Taxonomies can have any data source. A CSV-based importer is provided, so that you can fully instantiate a taxonomies store by just providing a set of CSV files in your Cadmus API: the API will take care of creating the database if not found, and populating it from the CSV files.
