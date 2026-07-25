---
title: "Importing Excel Data"
parent: "Creating Backend Core"
layout: default
nav_order: 9
---

# Importing Excel Data

Procedure for [importing Excel data](../../migration/import/import-xlsx.md). In this document, `__PRJ__` is the name of your project.

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

        // TODO

        return entryIndex + 3;
    }   
}
```
