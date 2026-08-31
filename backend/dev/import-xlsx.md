---
title: "Importing Excel Data"
parent: "Creating Backend Core"
layout: default
nav_order: 9
---

# Importing Excel Data

Procedure for [importing Excel data](../../migration/import/import-xlsx). In this document, `__PRJ__` is the name of your project.

- [example: TES import tool](https://github.com/vedph/cadmus-tes/tree/master/tes-tool)

## Setup Solution

In your backend solution, **add 2 projects**:

- a C# library project named `Cadmus.__PRJ__.Import`.
- a C# console app named `__PRJ__-tool`. If you already have one, you can just add a new command to it.

## Tool

This is just infrastructure code, so you can copy it and customize a few parameters. In the tool project, use as template an existing tool like that of TES at <https://github.com/vedph/cadmus-tes/tree/master/tes-tool>: just copy all its files and customize:

- in `Assets`, all `.json` profiles will get their `entryRegionParsers` array items replaced by the tags of the newly created parsers in the library project.
- in `Thesauri.json`, copy all the required thesauri (or just all the thesauri when in doubt) from your backend.
- in `Services/PipelineFactoryProvider.cs`, ensure to import the `RowEntryRegionParser`'s assembly from your library project.

## Library

- add to the project references to:
  - typically, `Cadmus.General.Parts` if you are going to target any of the general parts in the import.
  - `Cadmus.Import.Proteus`.
  - your project own parts or fragments library and any other parts you will need.
- copy `ImportHelper.cs` and possibly customize it (e.g. for `_emptyValues`).
- copy `RowEntryRegionParser.cs` and change its tag to fit your project.
- add as many column parsers as needed, using this template (`__TAG__` is the region tag). Each parser typically adds or reuses a specific part in the current item (representing the row being processed), or accumulates data to be later used by other parsers in the data context.

```cs
using Cadmus.Import.Proteus;
using Cadmus.General.Parts;
using Fusi.Tools.Configuration;
using Microsoft.Extensions.Logging;
using Proteus.Core.Entries;
using Proteus.Core.Regions;
using System;
using System.Collections.Generic;

namespace Cadmus.Tes.Import;

/// <summary>
/// TES column categories entry region parser. This targets TODO.
/// </summary>
/// <seealso cref="EntryRegionParser" />
/// <seealso cref="IEntryRegionParser" />
[Tag("entry-region-parser.tes.col-__TAG__")]
public sealed class Col__TAG__EntryRegionParser :
    EntryRegionParser, IEntryRegionParser
{
    /// <summary>
    /// Gets the tags of the regions that this parser can handle.
    /// </summary>
    public string[] RegionTags => ["col-__TAG__"];

    /// <summary>
    /// Parses the region of entries at <paramref name="regionIndex" />
    /// in the specified <paramref name="entryRegions" />.
    /// </summary>
    /// <param name="entrySet">The entries set.</param>
    /// <param name="entryRegions">The regions.</param>
    /// <param name="entryRegionIndex">Index of the region in the set.</param>
    /// <returns>
    /// The index to the next region to be parsed.
    /// </returns>
    /// <exception cref="ArgumentNullException">set or regions</exception>
    protected override int DoParse(EntrySet entrySet, int entryIndex,
        IReadOnlyList<EntryRegion> entryRegions, int entryRegionIndex)
    {
        ArgumentNullException.ThrowIfNull(entrySet);
        ArgumentNullException.ThrowIfNull(entryRegions);

        CadmusEntrySetContext ctx = (CadmusEntrySetContext)entrySet.Context;
        EntryRegion region = entryRegions[entryRegionIndex];

        if (ctx.CurrentItem == null)
        {
            Logger?.LogError("__TAG__ column without any item at region {Region}",
                region);
            throw new InvalidOperationException(
                "__TAG__ column without any item at region " + region);
        }

        DecodedTextEntry txt = entrySet.GetEntryAt<DecodedTextEntry>(
            entryIndex + 1)!;
        string? value = ImportHelper.FilterValue(txt.Value, false);

        // TODO implement your parsing logic for value here
        // unless it is empty or null, targeting the current
        // item -- you can add or reuse an existing part of any
        // given type with ctx.EnsurePartForCurrentItem<T>
        // (pass it the role ID string if required)

        return entryIndex + 3;
    }   
}
```

## Profiles

Add the Proteus dump and import profiles to your tool assets as content files. This allows having them distributed with the tool so you can use them as the input for the import process.

These files typically are (replace `__PRJ__` with your project name):

- 📁 `Dump-xlsx.json` used to dump each parsed row into Excel files to inspect the decoded entries with their regions:

```json
{
  "context": {
    "id": "it.vedph.entry-set-context.cadmus"
  },
  "entryReader": {
    "id": "entry-reader.xlsx",
    "options": {
      "inputFile": "{{HOMEDRIVE}}{{HOMEPATH}}\\Desktop\\__PRJ__\\__PRJ__.xlsx",
      "hasHeaderRow": true,
      "columnNameFiltering": true,
      "eofCheckColumn": 1
    }
  },
  "entrySetBoundaryDetector": {
    "id": "entry-set-detector.cmd",
    "options": {
      "type": 2,
      "name": "row-end"
    }
  },
  "entryRegionDetectors": [
    {
      "id": "region-detector.explicit",
      "options": {
        "unpairedCommandNames": [
          "sheet",
          "row",
          "col"
        ],
        "tagSuffixArgName": "n",
        "tagSuffixSeparator": "-"
      }
    },
    {
      "id": "region-detector.unmapped",
      "options": {
        "unmappedRegionTag": "x"
      }
    }
  ],
  "entryRegionParsers": [
    {
      "id": "entry-region-parser.__PRJ__.row"
    },
    {
      "id": "entry-region-parser.__PRJ__.col-YOUR_COLUMN_NAME"
    },
    /* ... etc */
  ],
  "entrySetContextPatchers": [
    {
      "id": "it.vedph.entry-set-context-patcher.cadmus"
    }
  ],
  "entrySetExporters": [
    {
      "id": "entry-set-exporter.excel-dump",
      "options": {
        "maxEntriesPerDumpFile": 10000,
        "outputDirectory": "{{HOMEDRIVE}}{{HOMEPATH}}\\Desktop\\__PRJ__\\__PRJ__-dump"
      }
    }
  ]
}
```

- 📁 `Dump-md.json` used to dump each parsed row into Markdown files with the details of the item built from each row with its parts: this is equal to the above profile, and only the exporter module changes as follows:

```json
"entrySetExporters": [
  {
    "id": "it.vedph.entry-set-exporter.cadmus.md-dump",
    "options": {
    "outputDirectory": "{{HOMEDRIVE}}{{HOMEPATH}}\\Desktop\\__PRJ__\\__PRJ__-dump\\",
    "noEntries": true,
    "jsonParts": true
    }
  }
]
```

- 📁 `Import.json` used to effectively import data into the target MongoDB database. As above, only the exporter changes:

```json
"entrySetExporters": [
  {
    "id": "it.vedph.entry-set-exporter.cadmus.mongo"
  }
]
```
