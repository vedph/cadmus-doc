---
title: "Graph - Walker" 
layout: default
parent: "Graph - Walking Graph"
nav_order: 2
---

# Graph Walker

⚙️ TECHNICAL NOTE

The `GraphWalkerComponent` gets a single node ID (`nodeId`) representing the starting node, and can emit `nodePick` events when the user picks any of the displayed nodes. This component is just a wrapper of the `GraphWalker` class, which mantains a collection of nodes and a collection of edges. In turn, this class uses the `GraphService` to communicate with the [graph API backend](#api).

## Properties

The parameters of the `GraphWalker` class are:

- `pageSize`: the paging size.
- `maxLiteralLen`: the max length of the literal value to display.

The class exposes a number of observables:

- `nodes$`: nodes.
- `edges$`: edges.
- `loading$`: true when loading data.
- `error$`: last error if any.
- `selectedNode$`: the currently selected node (a `GraphNode`, whose data property is any of the `WalkerData` types).
- `pOutFilter$`: the outbound linked nodes filter for the currently selected P node.
- `pInFilter$`: the inbound linked nodes filter for the currently selected P node.
- `pLitFilter$`: the literal linked nodes filter for the selected P node.
- `nOutFilter$`: the outbound triples filter for the currently selected N node.
- `nInFilter$`: the inbound triples filter for the currently selected N node.
- `childTotals$`: the total items fetched for each filter of the currently selected node. This is an object with a property for each total.

Nodes in the walker may represent nodes, literals, or property groups. While each node in the backend has a numeric ID, the nodes displayed in the walker get a calculated ID used inside it; the same happens for edges. These IDs are defined by the following conventions:

- **nodes**: `N<node_id>`, e.g. `N12` where `12` is the node's numeric ID.
- **literals**: `L<triple_id>`, e.g. `L34` where `34` is the ID of the triple the literal is the object of.
- **property groups**: `P<predicate_id>N<node_id>`, e.g. `P12N34` where `12` is the ID of the node acting as a predicate, and `N` is the source node ID. The source node is either the subject or the object of the triple, according to the walk direction.
- **edges**: `E<source>_<target>`.

All the nodes visualized in the graph include node data in their `data` property. These data are defined in types having the following hierarchy:

- `WalkerData`: the base class for all the nodes data. This contains its origin ID and basic properties (color and visibility).
  - `WalkerWidgetData`: a node or a property group (selectable and expandable).
    - `WalkerNodeData`: a node (node data and its outbound and inbound filters).
    - `WalkerPropData`: a property group (property group data and its outbound, inbound, and literal filters).
    - `WalkerLitData`: a literal node (literal value and metadata).

## Methods

- `getSelectedNode`.
- `selectNode`: select a node as the current node.
- `reset`: reset the graph setting its origin to the specified node ID.
- `expandNode`: expand the specified node by loading its property groups.
- `expandSelectedNode`: expand the currently selected node by loading its property groups.
- `expandProperty`: expand the currently selected properties group node, by loading its outbound nodes, inbound nodes, and literal nodes.
- `expandSelectedProperty`: expand the currently selected properties group node, by loading its outbound nodes, inbound nodes, and literal nodes. If there is no selection, or the selected node is not a properties group node, nothing is done.
- `toggleNode`: toggle the specified node by expanding or collapsing it.

## API

Cadmus provides a graph-specific API to handle the graph, with a number of endpoints.

- [API code repository](https://github.com/vedph/cadmus_api): [GraphController](https://github.com/vedph/cadmus_api/blob/master/Cadmus.Api.Controllers/GraphController.cs)

### API - Nodes

- `GET api/graph/nodes`: get a page of graph nodes. Filters:
  - `pageNumber`
  - `pageSize`
  - `uid`: any portion of the node's UID to match.
  - `isClass`: match node's class status.
  - `tag`: null (match any), empty (match null tags), non-empty (match the specified tag).
  - `label`: any portion of the node's label to match.
  - `sourceType`: type of source (0-N).
  - `sid`: match node's SID.
  - `isSidPrefix`: match only the initial portion of the node's SID.
  - `classIds`: match only nodes inside any of the listed classes.
  - `linkedNodeId`: match only nodes directly linked to the specified node ID.
  - `linkedNodeRole`: the role of the node identified by linkedNodeId: `S`ubject or `O`bject.
- `GET api/graph/nodes/ID`: get the node with the specified ID.
- `GET api/graph/nodes-set`: get the nodes with the specified IDs.
- `GET api/graph/nodes-by-uri`: get nodes by URI.
- `POST api/graph/nodes`: add or update the specified node.
- `DELETE api/graph/nodes/ID`: delete the node with the specified ID.

### API - Triples

- `GET api/graph/triples`: get a page of graph triples. Filters:
  - `pageNumber`
  - `pageSize`
  - `subjectId`: the ID of the subject node.
  - `predicateIds`: the ID(s) the predicate node should belong to.
  - `notPredicateIds`: the ID(s) the predicate node should not belong to.
  - `hasLiteralObject`: true to match only triples with a literal object, false to match only triples with a non literal object, null to match any.
  - `objectId`: the ID of the object node.
  - `sid`: match triple's SID.
  - `isSidPrefix`: match only the initial portion of the triple's SID.
  - `tag`: null (match any), empty (match null tags), non-empty (match the specified tag).
  - `sort`: the sort order.
- `GET api/graph/triples/ID`: get the triple with the specified ID.
- `POST api/graph/triples`: add or update the specified node.
- `DELETE api/graph/triples/ID`: delete the triple with the specified ID.

### API - Walker

- `GET api/graph/walk/triples`: get walker triples. Filters:
  - `pageNumber`
  - `pageSize`
  - `subjectId`: the ID of the subject node.
  - `predicateIds`: the ID(s) the predicate node should belong to.
  - `notPredicateIds`: the ID(s) the predicate node should not belong to.
  - `hasLiteralObject`: true to match only triples with a literal object, false to match only triples with a non literal object, null to match any.
  - `objectId`: the ID of the object node.
  - `sid`: match triple's SID.
  - `isSidPrefix`: match only the initial portion of the triple's SID.
  - `tag`: null (match any), empty (match null tags), non-empty (match the specified tag).
  - `sort`: the sort order.
- `GET api/graph/walk/nodes`: get walker linked nodes. Filter:
  - `pageNumber`
  - `pageSize`
  - `uid`: match node's UID.
  - `isClass`: match node's class status.
  - `tag`: null (match any), empty (match null tags), non-empty (match the specified tag).
  - `label`: any portion of the node's label to match.
  - `sourceType`: type of source (0-N).
  - `sid`: match node's SID.
  - `isSidPrefix`: match only the initial portion of the node's SID.
  - `classIds`: match only nodes inside any of the listed classes.
  - `otherNodeId`: match the other node identifier, which is the subject node ID when `isObject` is true, otherwise the object node ID.
  - `predicateId`: the property identifier in the triple including the node to match, either as a subject or as an object (according to `isObject`).
  - `isObject`: whether the node to match is the object (true) or the subject (false) of the triple having predicate `predicateId`.
- `GET api/graph/walk/nodes/literal`: get walker linked literals. Filter:
  - `pageNumber`
  - `pageSize`
  - `literalPattern`: the regular expression to match the literal.
  - `literalType`: the type of the object literal. This corresponds to literal suffixes after `^^` in Turtle: e.g. `"12.3"^^xs:double`.
  - `literalLanguage`: literal's language to match.
  - `minLiteralNumber`: max numeric value for a numeric object literal.
  - `maxLiteralNumber`: min numeric value for a numeric object literal.
  - `subjectId`: the subject node ID in the triple including the literal to match.
  - `predicateId`: the predicate node ID in the triple including the literal to match.
