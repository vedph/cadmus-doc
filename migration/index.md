---
title: "Migration" 
layout: default
nav_order: 3
---

# Data Migration

Migration functionality in Cadmus refer to both data import and export. The code repository providing migration is <https://github.com/vedph/cadmus-migration>.

Currently, there are three main areas where migration has been implemented:

- importing Cadmus data from other resources, even if low-structure. This mostly happens via the [Proteus subsystem](https://myrmex.github.io/overview/proteus/). For instance, we might import a flat set of data from a spreadsheet into a highly-structured Cadmus database. As a real-world example you can look at the [VeLA project](https://github.com/vedph/cadmus-vela#original-spreadsheet). The [tool importing data](https://github.com/vedph/cadmus-vela-tool) is a thin wrapper built on top of Proteus functionality. Another use case was importing Cadmus data from [Word documents](https://github.com/vedph/cadmus-bdm-tool). Any type of resource can be used as input, even plain text.
- exporting Cadmus text with annotations into TEI or other XML schemas.
- exporting selected Cadmus data into RDF [graphs](graph/index).

>For importing taxonomies, see the section about [thesauri](../models/thesauri.md#editing-thesauri).
