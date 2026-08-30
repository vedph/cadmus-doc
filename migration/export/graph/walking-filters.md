---
title: "Graph - Walking Graph Filters" 
layout: default
parent: "Graph - Walking Graph"
nav_order: 1
---

# Walker Filters

While walking, the nodes in the graph work also has handles to control paging, filtering, and sorting of triples and nodes.

There are two scenarios in walking, according to the current origin: from node, or from a properties group.

## Walking from Node

When walking from a node, the node can project property groups, either outbound (the node being the subject) or inbound (the node being the object):

1. **outbound**: required filter is _subject ID_ = origin node ID.
2. **inbound**: required filter is _object ID_ = origin node ID.

The filter model (`TripleFilter`) includes these properties:

- `PageNumber`
- `PageSize`
- `SubjectId`: equal to the origin node for outbound links.
- `PredicateIds`: a whitelist of predicate IDs. At least 1 of these must be matched.
- `NotPredicateIds`: a blacklist of predicate IDs. None of these must be matched.
- `HasLiteralObject`: true to match only triples having a literal object; false for the inverse; null to match both.
- `ObjectId`: the ID of the triple's object to match.
- `Sid`: the SID to match.
- `IsSidPrefix`: true to treat the SID filter as a prefix.
- `Tag`: the tag to match (empty to match null tags, null to ignore tags in matching).
- `LiteralPattern`: the pattern of the triple's literal object value to match.
- `LiteralType`: the type of the triple's literal object to match.
- `LiteralLanguage`: the language of the triple's literal object to match.
- `MinLiteralNumber`: the minimum numeric value for a numeric literal object.
- `MaxLiteralNumber`: the maximum numeric value for a numeric literal object.

## Walking from Property Group

When walking from a property group, the group can project nodes of three types:

1. **outbound** non-literals: required filters are _predicate ID_ = group's property ID; _subject ID_ = the ID of the subject node linked to this property group. The result is a page of nodes.
2. **outbound** literals: required filters are _predicate ID_ = group's property ID; _subject ID_ = the ID of the subject node linked to this property group. The result is a page of triples.
3. **inbound**: required filters are _predicate ID_ = group's property ID; _object ID_ = the ID of the object node linked to this property group (literal=false is implied by the fact that the nodes being fetched are subjects). The result is a page of nodes.

The filter model is different:

(1) linked node (`LinkedNodeFilter`):

- `PageNumber`
- `PageSize`
- `OtherNodeId`: the other node identifier, which is the subject node ID when `IsObject` is true, otherwise the object node ID.
- `PredicateId`: the predicate ID to match.
- `IsObject`: true if the node to match is the object of the triple; false if it's the subject.
- `Uid`: any portion of the node ID to match.
- `IsClass`: true to match nodes representing classes; false to match nodes not representing classes; null to match both.
- `Tag`: the tag to match (empty to match null tags, null to ignore tags in matching).
- `Label`: any portion of the label to match.
- `SourceType`: the source type to match.
- `Sid`: the SID to match.
- `IsSidPrefix`: true to treat the SID filter as a prefix.
- `ClassIds`: the classes IDs to match. When specified, the node must derive either directly or indirectly from any of the classes.

(2) linked literal (`LinkedLiteralFilter`):

- `PageNumber`
- `PageSize`
- `SubjectId`: the subject ID in the triple including the literal to match.
- `PredicateId`: the property identifier in the triple including the  literal to match.
- `LiteralPattern`: the pattern of the triple's literal object value to match.
- `LiteralType`: the type of the triple's literal object to match.
- `LiteralLanguage`: the language of the triple's literal object to match.
- `MinLiteralNumber`: the minimum numeric value for a numeric literal object.
- `MaxLiteralNumber`: the maximum numeric value for a numeric literal object.
