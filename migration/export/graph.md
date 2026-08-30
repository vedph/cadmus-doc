---
title: "Graph" 
layout: default
parent: Export
nav_order: 4
---

# Graph

Graph components allow mapping Cadmus data into RDF-like graphs.

The graph subsystem adds an RDF-based graph on top of the Cadmus database, integrating the edit experience so that:

- graph nodes can be automatically created and kept in synch while editing parts;
- at the same time, users can add or link nodes in a visual UI, thus providing complex relationships across data at a higher abstraction level.

In a sense, just like you might be able to create (via [rendering](../rendering/architecture)) a full-fledged TEI document without even knowing about XML, this subsystem allows you to create an RDF graph without even knowing about semantic web.

In both cases, you just edit data in GUIs. Yet, the graph subsystem is not a later export process typically run once, but rather a real-time mechanism for mapping Cadmus data models into nodes and triples, thus updating an existing graph whenever you save data. Further, the graph subsystem also provides a new editing experience, where you can directly edit the graph by freely adding nodes or connecting them.
