---
title: "Migration Tool" 
layout: default
parent: Migration
nav_order: 2

---

- [Migration Tool](#migration-tool)
  - [Dump Command](#dump-command)
  - [Render Items Command](#render-items-command)

# Migration Tool

The migration tool provides a multi-platform CLI tool to use data migration functions for Cadmus databases.

## Dump Command

🎯 [Dump](./dump/index.md) any subset of Cadmus data objects into JSON file(s).

Syntax:

```sh
./cadmus-mig <DatabaseName> [-i] [--no-part-date] [--no-deleted] [--no-parts] [-o OutputDirectory] [--max-file-items N] [--indented] [-w PartTypeKey] [-b PartTypeKey] [-u UserId] [-n MinModified] [-m MaxModified] [-t Title] [--description Description] [-f FacetId] [-g GroupId] [-l Flags] [--flag-matching AllSet|AnySet|AllClear|AnyClear] [-p PageNumber] [-s PageSize]
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

## Render Items Command

🎯 [Render](./render/architecture.md) Cadmus items.

Syntax:

```sh
./cadmus-mig <DatabaseName> <ConfigPath> [-p PluginTag] [-r PluginTag] [-c ComposerKey] [-m MaxItemsCount]
```

- `DatabaseName`: the Cadmus database name. The MongoDB connection string template to use is specified in the CLI `appsettings.json` configuration file. In it, `{0}` is the placeholder for the database name.
- <ConfigPath>: the path to the rendering pipeline configuration file.
- `-p PluginTag`: the tag of the factory provider plugin for preview (`ICadmusRenderingFactoryProvider`).
- `-r PluginTag`: the tag of the Cadmus repository provider plugin (`IRepositoryProvider`).
- `-c ComposerKey`: the item composer key to use (default is `default`).
- `-m MaxItemsCount`: the maximum number of items to render (0=all).
