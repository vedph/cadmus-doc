---
title: "Migration Tool" 
layout: default
parent: Migration
nav_order: 3

---

- [Migration Tool](#migration-tool)
  - [Dump Command](#dump-command)
  - [Dump Thesauri Command](#dump-thesauri-command)
  - [Render Items Command](#render-items-command)
  - [Export RDF Command](#export-rdf-command)

# Migration Tool

The migration tool provides a multi-platform CLI tool to use data migration functions for Cadmus databases.

## Dump Command

🎯 [Dump](./dump/index.md) any subset of Cadmus data objects into JSON file(s).

Syntax:

```sh
./cadmus-mig dump <DatabaseName> [-i] [--no-part-date] [--no-deleted] [--no-parts] [-o OutputDirectory] [--max-file-items N] [--indented] [-w PartTypeKey] [-b PartTypeKey] [-u UserId] [-n MinModified] [-m MaxModified] [-t Title] [--description Description] [-f FacetId] [-g GroupId] [-l Flags] [--flag-matching AllSet|AnySet|AllClear|AnyClear] [-p PageNumber] [-s PageSize]
```

- `DatabaseName`: the Cadmus database name. The MongoDB connection string template to use is specified in the CLI `appsettings.json` configuration file. In it, `{0}` is the placeholder for the database name.
- `-i`: incremental dump, including only items/parts changed in the timeframe.
- `--no-part-date`: do not consider parts' date when filtering items by time-based parameters.
- `--no-deleted`: do not include deleted items.
- `--no-parts`: do not include parts in exported items.
- `-o OutputDirectory`: the output directory. If not specified, a desktop folder with the current date will be used. If the folder does not exist, it will be created.
- `--max-file-items N`: The maximum number of items to export per file. If not specified (0), all items will be exported in a single file.
- `--indented`: indent JSON output.
- `-w PartTypeKey`: the keys of the part types to include in the export with form `typeId[:roleId]`. Can be used multiple times, one per part type. If not specified, all part types are included.
- `-b PartTypeKey`: the keys of the part types to exclude in the export with form `typeId[:roleId]`. Can be used multiple times, one per part type. If not specified, no part types are excluded.
- `-u UserId`: the user ID to filter items by. If not specified, items by all users are included.
- `-n MinModified`: the minimum modified date and time filter.
- `-m MaxModified`: the maximum modified date and time filter.
- `-t Title`: the item's title filter (any portion of it).
- `--description Description`: the item's description filter (any portion of it).
- `-f FacetId`: the item's facet ID.
- `-g GroupId`: the item's group ID.
- `-l Flags`: the item's flags. This is a numeric value representing a bitset where each bit is a flag.
- `--flag-matching AllSet|AnySet|AllClear|AnyClear`: the item's flags matching mode.
- `-p PageNumber`: the page number (1-N), when you want just a page of results.
- `-s PageSize`: the page size. By default this is 0, i.e. there is no paging.

Example:

```sh
./cadmus-mig dump cadmus-ndp --indented
```

## Dump Thesauri Command

🎯 Dump all the thesauri into a JSON file.

Syntax:

```sh
./cadmus-mig dump-thesauri <DatabaseName> [-o OutputPath] [--indented]
```

- `DatabaseName`: the Cadmus database name. The MongoDB connection string template to use is specified in the CLI `appsettings.json` configuration file. In it, `{0}` is the placeholder for the database name.
- `-o OutputPath`: the output JSON file path.
- `--indented`: indent JSON output.

## Render Items Command

🎯 [Render](./render/architecture.md) Cadmus items.

Syntax:

```sh
./cadmus-mig render <DatabaseName> <ConfigPath> [-p PluginTag] [-r PluginTag] [-c ComposerKey] [-m MaxItemsCount]
```

- `DatabaseName`: the Cadmus database name. The MongoDB connection string template to use is specified in the CLI `appsettings.json` configuration file. In it, `{0}` is the placeholder for the database name.
- `ConfigPath`: the path to the rendering pipeline configuration file.
- `-p PluginTag`: the tag of the factory provider plugin for preview (`ICadmusRenderingFactoryProvider`).
- `-r PluginTag`: the tag of the Cadmus repository provider plugin (`IRepositoryProvider`).
- `-c ComposerKey`: the item composer key to use (default is `default`).
- `-m MaxItemsCount`: the maximum number of items to render (0=all).

## Export RDF Command

🎯 Export Cadmus [semantic graph](../migration/graph/graph.md) into various standard RDF formats.

Syntax:

```sh
./cadmus-mig export-rdf <DatabaseName> <OutputPath> [-f Format] [--p true|false] [-c true|false] [--base-uri URI] [--batch-size Size] [--r true|false] [--export-referenced-nodes-only true|false] [--node-tag-filter Tags] [--triple-tag-filter Tags] [--encoding Encoding]
```

- `DatabaseName`: the Cadmus database name. The MongoDB connection string template to use is specified in the CLI `appsettings.json` configuration file. In it, `{0}` is the placeholder for the database name.
- `OutputPath`: the output file path.
- `-f Format`: the format to export (`turtle`, `rdfxml`, `ntriples`, `jsonld`). Default is `turtle`.
- `-p` or `--include-prefixes`: include prefix declarations in the output. Default is `true`.
- `-c` or `--include-comments`: include comments in the output. Default is `true`.
- `--base-uri URI`: the base URI to use for relative URIs. If null or empty, no base URI is used.
- `--batch-size Size`: maximum number of triples to process in a single batch. Default is 10000.
- `-r` or `--pretty-print`: pretty-print the output (add indentation and line breaks). Default is `true`.
- `--export-referenced-nodes-only`: export only nodes that are referenced in triples. Default is false (exports all nodes).
- `--node-tag-filter`: optional filter for node tags. If specified, only nodes with matching tags are exported. Comma-separated.
- `--triple-tag-filter Tags`: optional filter for triple tags. If specified, only triples with matching tags are exported. Comma-separated.
- --`encoding Encoding`: the character encoding to use for output files. Default is `UTF-8`.

Examples:

- **Turtle**: a compact, human-readable syntax for RDF. Great for editing and reading manually (<https://www.w3.org/TR/rdf12-turtle/>).

```sh
./cadmus-mig export-rdf cadmus-rdf-test c:/users/dfusi/desktop/triples.ttl
```

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix x: <http://test.org/> .
@prefix xml: <http://www.w3.org/XML/1998/namespace> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

# RDF data exported from Cadmus Graph database
# Export date: 2025-09-30T08:24:08Z

x:events/pid/birth rdf:type crm:e67_birth .
x:events/pid/birth crm:p2_has_type x:event-types/person.birth .
x:events/pid/birth crm:p98_brought_into_life x:persons/petrarch .
x:events/pid/birth crm:p3_has_note "Petrarch was born on July 20, 1304 at Arezzo from ser Petracco and Eletta Canigiani."@en .
x:places/arezzo rdf:type crm:e53_place .
x:events/pid/birth crm:p7_took_place_at x:places/arezzo .
x:events/pid/birth crm:p4_has_time-span x:timespans/ts#4 .
x:timespans/ts#4 crm:p82_at_some_time_within "1304"@en .
x:timespans/ts#4 crm:p87_is_identified_by "20 Jul 1304 AD"@en .
x:events/pid/birth crm:p96_by_mother x:persons/eletta_canigiani .
x:events/pid/birth crm:p97_from_father x:persons/ser_petracco .

# End of RDF data
```

- **RDF/XML**: the original RDF syntax using XML. Verbose and harder to read, but widely supported (<https://www.w3.org/TR/rdf-syntax-grammar/>).

```sh
./cadmus-mig export-rdf cadmus-rdf-test c:/users/dfusi/desktop/triples.xml -f rdfxml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#" xmlns:crm="http://www.cidoc-crm.org/cidoc-crm/" xmlns:owl="http://www.w3.org/2002/07/owl#" xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#" xmlns:x="http://test.org/" xmlns:xsd="http://www.w3.org/2001/XMLSchema#">
  <!-- RDF data exported from Cadmus Graph database -->
  <!-- Export date: 2025-09-30T08:22:40Z -->
  <rdf:Description rdf:about="http://test.org/events/pid/birth">
    <rdf:type rdf:resource="http://www.cidoc-crm.org/cidoc-crm/e67_birth" />
    <crm:p2_has_type rdf:resource="http://test.org/event-types/person.birth" />
    <crm:p98_brought_into_life rdf:resource="http://test.org/persons/petrarch" />
    <crm:p3_has_note xml:lang="en">Petrarch was born on July 20, 1304 at Arezzo from ser Petracco and Eletta Canigiani.</crm:p3_has_note>
    <crm:p7_took_place_at rdf:resource="http://test.org/places/arezzo" />
    <crm:p4_has_time-span rdf:resource="http://test.org/timespans/ts#4" />
    <crm:p96_by_mother rdf:resource="http://test.org/persons/eletta_canigiani" />
    <crm:p97_from_father rdf:resource="http://test.org/persons/ser_petracco" />
  </rdf:Description>
  <rdf:Description rdf:about="http://test.org/places/arezzo">
    <rdf:type rdf:resource="http://www.cidoc-crm.org/cidoc-crm/e53_place" />
  </rdf:Description>
  <rdf:Description rdf:about="http://test.org/timespans/ts#4">
    <crm:p82_at_some_time_within xml:lang="en">1304</crm:p82_at_some_time_within>
    <crm:p87_is_identified_by xml:lang="en">20 Jul 1304 AD</crm:p87_is_identified_by>
  </rdf:Description>
</rdf:RDF>
```

- **NTriples**: a simple, line-based format where each RDF triple is written on a separate line (<https://w3c.github.io/rdf-n-triples/spec/>).

```sh
./cadmus-mig export-rdf cadmus-rdf-test c:/users/dfusi/desktop/triples.nt -f ntriples
```

```nt
# RDF data exported from Cadmus Graph database
# Export date: 2025-09-30T08:25:36Z
<http://test.org/events/pid/birth> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.cidoc-crm.org/cidoc-crm/e67_birth> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p2_has_type> <http://test.org/event-types/person.birth> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p98_brought_into_life> <http://test.org/persons/petrarch> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p3_has_note> "Petrarch was born on July 20, 1304 at Arezzo from ser Petracco and Eletta Canigiani."@en .
<http://test.org/places/arezzo> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.cidoc-crm.org/cidoc-crm/e53_place> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p7_took_place_at> <http://test.org/places/arezzo> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p4_has_time-span> <http://test.org/timespans/ts#4> .
<http://test.org/timespans/ts#4> <http://www.cidoc-crm.org/cidoc-crm/p82_at_some_time_within> "1304"@en .
<http://test.org/timespans/ts#4> <http://www.cidoc-crm.org/cidoc-crm/p87_is_identified_by> "20 Jul 1304 AD"@en .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p96_by_mother> <http://test.org/persons/eletta_canigiani> .
<http://test.org/events/pid/birth> <http://www.cidoc-crm.org/cidoc-crm/p97_from_father> <http://test.org/persons/ser_petracco> .
# End of RDF data
```

- **JSON-LD**: RDF in JSON format, designed for easy integration with web applications (<https://json-ld.org/>).

```sh
./cadmus-mig export-rdf cadmus-rdf-test c:/users/dfusi/desktop/triples.json -f jsonld
```

```json
{
  "@comment": "RDF data exported from Cadmus Graph database at 2025-09-30T08:26:16Z",
  "@context": {
    "crm": "http://www.cidoc-crm.org/cidoc-crm/",
    "owl": "http://www.w3.org/2002/07/owl#",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
    "x": "http://test.org/",
    "xml": "http://www.w3.org/XML/1998/namespace",
    "xsd": "http://www.w3.org/2001/XMLSchema#"
  },
  "@graph": [
    {
      "@id": "x:events/pid/birth",
      "rdf:type": {"@id": "crm:e67_birth"},
      "crm:p2_has_type": {"@id": "x:event-types/person.birth"},
      "crm:p98_brought_into_life": {"@id": "x:persons/petrarch"},
      "crm:p3_has_note": {"@value": "Petrarch was born on July 20, 1304 at Arezzo from ser Petracco and Eletta Canigiani.", "@language": "en"},
      "crm:p7_took_place_at": {"@id": "x:places/arezzo"},
      "crm:p4_has_time-span": {"@id": "x:timespans/ts#4"},
      "crm:p96_by_mother": {"@id": "x:persons/eletta_canigiani"},
      "crm:p97_from_father": {"@id": "x:persons/ser_petracco"}
    },
    {
      "@id": "x:places/arezzo",
      "rdf:type": {"@id": "crm:e53_place"}
    },
    {
      "@id": "x:timespans/ts#4",
      "crm:p82_at_some_time_within": {"@value": "1304", "@language": "en"},
      "crm:p87_is_identified_by": {"@value": "20 Jul 1304 AD", "@language": "en"}
    }
  ]
}
```
