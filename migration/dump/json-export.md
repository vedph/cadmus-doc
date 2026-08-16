---
title: "Exporting JSON"
layout: default
parent: "Migration"
nav_order: 3
---

# Exporting JSON

Exporting JSON differs from a [raw JSON export](json-raw.md) as it implies selecting and remapping source data into a different JSON schema. So here we are effectively transforming source data into any other schema, as required by the consumer.

A typical application of this export process is a frontend designed around a set of predefined view models, specifically designed for it. Data is thus transformed into the target schema, similarly to what happens in XML via XSLT.

This export process shares its source data selection and mapping logic with the [graph](../graph/graph.md) exporter, which targets RDF, with an important difference: the graph exporter is designed to constantly map items into an RDF graph, keeping it in synch with user edits. Whenever users save their edits in the editor, data gets selected and mapped to target the graph according to the chosen ontologies. Users can even edit the target RDF graph directly to manually extend it.

The JSON export process instead is not designed to export data in real-time. Rather, its purpose is to provide a massive JSON-based data transform and export, to be later consumed by a third-party client. So, the task of the JSON exporter is only to generate JSON documents, without caring about their usage. The graph exporter instead targets a graph database actively updating it to keep the graph in synch with user edits. Then, at any moment one can also export the nodes and triples of this graph into various standard formats to consume them as required.

While this difference applies to the output of each process, the logic to select the items and parts to export and map them into a different target is shared.

## Mappings

