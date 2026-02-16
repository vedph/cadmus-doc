---
title: "Migration" 
layout: default
nav_order: 3
---

# Data Migration

Migration functionality in Cadmus refer to both data import and export. The code repository providing migration is <https://github.com/vedph/cadmus-migration>.

Currently, there are three main areas where migration has been implemented:

- **importing** Cadmus data from other resources, even if low-structure. This mostly happens via the [Proteus subsystem](https://myrmex.github.io/overview/proteus/). For instance, we might import a flat set of data from a spreadsheet into a highly-structured Cadmus database. As a real-world example you can look at the [VeLA project](https://github.com/vedph/cadmus-vela#original-spreadsheet). The [tool importing data](https://github.com/vedph/cadmus-vela-tool) is a thin wrapper built on top of Proteus functionality. Another use case was importing Cadmus data from [Word documents](https://github.com/vedph/cadmus-bdm-tool). Any type of resource can be used as input, even plain text. Also, some imports are available within the [editor UI](ui-import) itself.
- **exporting** any subset of Cadmus data via [rendition](rendering/index).
- **exporting** selected Cadmus data into RDF [graphs](graph/index).
- [dumping](dump/index) a subset of Cadmus data into JSON for full or incremental data migration.

>For importing taxonomies, see the section about [thesauri](../models/thesauri.md#editing-thesauri).

Most of the migration functions are available from the Cadmus migration CLI tool.
