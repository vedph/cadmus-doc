---
title: "Graph - Designing"
layout: default
parent: Graph
nav_order: 7
---

# Designing for Graph

The typical steps for designing a graph for a Cadmus projects are:

1. define the ontologies you want to target.
2. define the [mappings](mappings) for each model you want to project into the graph.
3. prepare a list of nodes for at least the most commonly used classes and predicates from your ontologies.

## (1) Ontologies

Defining the ontologies you are going to use (including your own) is completely up to you. The mapping process is neutral, and can project data into nodes and triples from any mix of ontologies. Just decide a conventional prefix for each ontology, and fill the graph database `namespace_lookup` table accordingly.

For instance, here is the default content of such a table:

| id   | uri                                           |
| ---- | --------------------------------------------- |
| owl  | <http://www.w3.org/2002/07/owl#>              |
| rdf  | <http://www.w3.org/1999/02/22-rdf-syntax-ns#> |
| rdfs | <http://www.w3.org/2000/01/rdf-schema#>       |
| xml  | <http://www.w3.org/XML/1998/namespace>        |
| xsd  | <http://www.w3.org/2001/XMLSchema#>           |

>Another benefit of the mapping approach is that you are free to change your target ontologies and rebuild a whole new graph from the same Cadmus data. Of course, unless you are experimenting, it's hard to imagine a scenario where such a radical change could be realistic; but you might want to adjust your projection, change the representation of some specific data, or upgrade to a newer version of your target ontology. In all these scenarios mappings allow to regenerate the graph automatically, after just updating the mapping rules.

## (2) Define Mappings

Defining mappings is the core of the graph projection system. In itself, the process is very simple. For each model:

1. consider all the data you want to export from it;
2. decide how to render your model's properties into nodes and triples;
3. code the mapping in JSON and/or use the [Graph Studio App](graph-studio). Using this app is recommended especially to test how mappings work, so that you can ensure to get the expected results and fine tune them. The app is containarized, so that you can run it from any local computer (or server, if you prefer a distributed environment), whatever its platform.
4. import the mappings into the graph database using the [Cadmus CLI tool](https://github.com/vedph/cadmus_tool).

>💡 If you prefer a command-line based approach, you can also use the CLI tool to run mappings for a specific item and look at the results in the database.

In general, consider that Cadmus data is based on objects, which can be viewed as trees of properties. Each property is either a scalar value, like a string or a number, or another object, which in turn has its own properties. That's why internally data are stored as JSON.

For each properties tree, you will create a corresponding mappings tree: start by matching the "root" property, and then walk down the source data tree creating a corresponding mapping descendant node for each descendant properties you want to project.

💡 For a very simple example of this process, see the [sample mappings](mappings#sample). This is a modular approach, and it's suggested to define a set of reusable mappings whenever you find yourself repeating the same mapping even inside different source objects. This is especially true when your object uses so-called [bricks](https://cadmus-bricks-v3.fusi-soft.com), i.e. data structures which are shared across multiple models and have a corresponding UI widget.

## (3) Nodes List

The main purpose of this step is providing a clearer set of nodes in advance, so that when building triples using any of these nodes the mapper will find the nodes already present in the graph, and won't have to add them. This also has the advantage of providing additional metadata to the nodes, like the `isclass` attribute.

▶️ (1) prepare a JSON document with a root array element, and any number of objects in each, one per node, e.g.:

```json
[
  {
    "uri": "crm:e5_event",
    "isclass": true,
    "tag": null,
    "label": "e5_event"
  },
  {
    "uri": "crm:e7_activity",
    "isclass": true,
    "tag": null,
    "label": "e7_activity"
  },
  {
    "uri": "crm:e8_acquisition",
    "isclass": true,
    "tag": null,
    "label": "e8_acquisition"
  },
  {
    "uri": "crm:p2_has_type",
    "tag": "property",
    "label": "p2_has_type"
  },
  {
    "uri": "crm:p3_has_note",
    "tag": "property",
    "label": "p3_has_note"
  },
  {
    "uri": "crm:p4_has_time-span",
    "tag": "property",
    "label": "p4_has_time-span"
  }
]
```

▶️ (2) use the [Cadmus CLI tool](https://github.com/vedph/cadmus_tool) to [import](https://github.com/vedph/cadmus_tool?tab=readme-ov-file#graph-import-command) the nodes, e.g.:

```sh
./cadmus-tool graph-import c:/users/dfusi/desktop/nodes.json cadmus-itinera -g repository-provider.itinera
```
