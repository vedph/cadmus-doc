---
title: "Adding Backend Parts"
parent: "Creating Backend Core"
layout: default
nav_order: 1
---

- [Adding Backend Parts](#adding-backend-parts)
  - [Part - Single Entity](#part---single-entity)
  - [Part - Multiple Entities](#part---multiple-entities)
  - [Part Test Templates](#part-test-templates)
    - [Part Test Helper](#part-test-helper)
    - [Part Test - Single Entity](#part-test---single-entity)
    - [Part Test - Multiple Entities](#part-test---multiple-entities)
  - [Layer Parts](#layer-parts)
    - [Layer Fragment Test Template](#layer-fragment-test-template)

# Adding Backend Parts

Guidelines for **implementing a part**:

- _derive_ from `PartBase`, even if this is not strictly a requirement, but rather a commodity. The part class must anyway implement the `IPart` interface.
- _decorate_ the class with a `TagAttribute` providing the part's type ID.
- _do not add any logic_ to the part. The part is just a POCO object modeling the data it represents, and should have no logic. The only piece of logic required is the method returning the part's data pins, which is just a form of reflecting on the part's data themselves, e.g. for indexing or semantic projection.
- if creating a part representing a _base text_ for text layers, implement the `IHasText` interface by providing a `GetText()` method which, whatever the part's model, produces a single string representing its whole text. The same interface should be implemented whenever your part has some rather long piece of free, unstructured text you might want to be included in processes like full-text indexing. Also, when the part represents the base text in a stack of textual layers, set the role ID to `base-text` (defined in `PartBase.BASE_TEXT_ROLE_ID`).
- consider that the part will be subject to automatic serialization and deserialization. As the part is just a POCO object, this should not pose any issue.

>The parts project requires at least the `Cadmus.Core` package.

The typical **procedure** when adding a new part is:

1. create the part class.
2. create the part seeder class.
3. create the part seeder test.
4. create the part test (using both the part under test and its part seeder).

Here I provide a number of templates, according to whether your part represents a single entity or a collection of entities. For instance, a part listing the names assigned to an item has a collection of names, each being a model on its own. Instead, a part representing a free text note is a single entity part.

💡 If you need a **generic filter** for your pins, you can use a filter with the `DataPinBuilder` utility class. If using this filter many times, you may want to make it a singleton, e.g. like this:

```cs
internal static class DataPinHelper
{
    private static StandardDataPinTextFilter? _filter;

    /// <summary>
    /// Gets the default filter used for pins.
    /// This improves performance, as we can share this filter
    /// among several parts.
    /// </summary>
    static public IDataPinTextFilter DefaultFilter
    {
        get { return _filter ??= new StandardDataPinTextFilter(); }
    }
}
```

>The filter used here is a builtin utility component which preserves only letters, apostrophes and whitespaces, also removing any diacritics from the letters and lowercasing them. Whitespaces are flattened into spaces and normalized. Digits are dropped (by default) or preserved according to the options specified (pass `true` as the options argument to this filter to preserve them).

💡 The suggested order of operations for creating a part is:

1. create the part.
2. create its seeder (as it is typically used in part tests).
3. create the seeder tests.
4. create the part tests.

## Part - Single Entity

In the following template replace `__NAME__` with your part's name, minus the `Part` suffix:

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Configuration;

namespace Cadmus.__PRJ__.Parts;

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__Part : PartBase
{
    // TODO: add your properties here...

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins.</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem? item = null)
    {
        // TODO: build pins, eventually using DataPinBuilder like this
        // (optionally using DataPinHelper.DefaultFilter as an argument):
        // DataPinBuilder builder = new(new StandardDataPinTextFilter());
        //// latitude
        // builder.AddValue("lat", Latitude);
        //// tot-count
        //builder.Set("tot", Entries?.Count ?? 0, false);
        //return builder.Build(this);

        // ...or just use a simpler logic, like:
        // sample:
        // return Tag != null
        //    ? new[]
        //    {
        //        CreateDataPin("tag", Tag)
        //    }
        //    : Enumerable.Empty<DataPin>();

        throw new NotImplementedException();
    }

    /// <summary>
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(
        [
            // TODO: add pins definitions...
            // sample:
            // new DataPinDefinition(DataPinValueType.Integer,
            //    "tot-count",
            //    "The total count of entries.")
        ]);
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new();

        sb.Append("[__NAME__]");

        // TODO: append summary data...

        return sb.ToString();
    }
}
```

## Part - Multiple Entities

You can use this template when your part is just a container of an object representing an entry in a list of entries. Of course you are free to add more properties besides the list; this template just makes it easier to deal with typical objects-container parts.

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Configuration;

namespace Cadmus.__PRJ__.Parts;

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__Part : PartBase
{
    /// <summary>
    /// Gets or sets the entries.
    /// TODO: rename Entries in something more specific.
    /// </summary>
    public List<__ENTRY__> Entries { get; set; } = [];

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins: <c>tot-count</c> and a collection of pins with
    /// these keys: ....</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem? item = null)
    {
        // TODO: remove the filter if not using it, or make it a singleton
        // if using it in several components in the same library
        // e.g. using DataPinHelper.DefaultFilter as an argument
        DataPinBuilder builder = new(new StandardDataPinTextFilter());

        builder.Set("tot", Entries?.Count ?? 0, false);

        if (Entries?.Count > 0)
        {
            foreach (var entry in Entries)
            {
                // TODO: add values or increase counts like:
                // id unique values if not null:
                // builder.AddValue("id", entry.Id);
                // type-X-count counts if not null, unfiltered:
                // builder.Increase(entry.Type, false, "type-");
            }
        }

        return builder.Build(this);
    }

    /// <summary>
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(
        [
            // TODO: add pins definitions...
            new DataPinDefinition(DataPinValueType.Integer,
               "tot-count",
               "The total count of entries.")
        ]);
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new();

        sb.Append("[__NAME__]");

        if (Entries?.Count > 0)
        {
            sb.Append(' ');
            int n = 0;
            foreach (var entry in Entries)
            {
                if (++n > 3) break;
                if (n > 1) sb.Append("; ");
                sb.Append(entry);
            }
            if (Entries.Count > 3)
                sb.Append("...(").Append(Entries.Count).Append(')');
        }

        return sb.ToString();
    }
}
```

## Part Test Templates

Testing requires a bit of [infrastructure](#part-test-helper), usually encapsulated in a `TestHelper` class.

### Part Test Helper

This helper class provides the methods used to ease testing:

```cs
using Cadmus.Core;
using Cadmus.Core.Layers;
using System;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.RegularExpressions;
using Xunit;

namespace Cadmus.__PRJ__.Parts.Test;

internal static class TestHelper
{
    private static readonly JsonSerializerOptions _options =
        new()
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

    public static string SerializePart(IPart part)
    {
        ArgumentNullException.ThrowIfNull(part);
        return JsonSerializer.Serialize(part, part.GetType(), _options);
    }

    public static T? DeserializePart<T>(string json)
        where T : class, IPart, new()
    {
        ArgumentNullException.ThrowIfNull(json);
        return JsonSerializer.Deserialize<T>(json, _options);
    }

    public static string SerializeFragment(ITextLayerFragment fr)
    {
        ArgumentNullException.ThrowIfNull(fr);
        return JsonSerializer.Serialize(fr, fr.GetType(), _options);
    }

    public static T? DeserializeFragment<T>(string json)
        where T : class, ITextLayerFragment, new()
    {
        ArgumentNullException.ThrowIfNull(json);
        return JsonSerializer.Deserialize<T>(json, _options);
    }

    public static void AssertPinIds(IPart part, DataPin pin)
    {
        Assert.Equal(part.ItemId, pin.ItemId);
        Assert.Equal(part.Id, pin.PartId);
        Assert.Equal(part.RoleId, pin.RoleId);
    }

    static public bool IsDataPinNameValid(string name) =>
        Regex.IsMatch(name, @"^[a-zA-Z0-9\-_\.]+$");

    static public void AssertValidDataPinNames(IList<DataPin> pins)
    {
        foreach (DataPin pin in pins)
        {
            Assert.True(IsDataPinNameValid(pin.Name!), pin.ToString());
        }
    }
}
```

### Part Test - Single Entity

This template refers to the [single-entity part template](#part---single-entity):

```cs
using System;
using Xunit;
using Cadmus.Core;
using System.Collections.Generic;
using System.Linq;

namespace Cadmus.__PRJ__.Parts.Test;

public sealed class __NAME__PartTest
{
    private static __NAME__Part GetPart()
    {
        __NAME__PartSeeder seeder = new();
        IItem item = new Item
        {
            FacetId = "default",
            CreatorId = "zeus",
            UserId = "zeus",
            Description = "Test item",
            Title = "Test Item",
            SortKey = ""
        };
        return (__NAME__Part)seeder.GetPart(item, null, null)!;
    }

    private static __NAME__Part GetEmptyPart()
    {
        return new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
        };
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart();

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 = TestHelper.DeserializePart<__NAME__Part>(json)!;

        Assert.Equal(part.Id, part2.Id);
        Assert.Equal(part.TypeId, part2.TypeId);
        Assert.Equal(part.ItemId, part2.ItemId);
        Assert.Equal(part.RoleId, part2.RoleId);
        Assert.Equal(part.CreatorId, part2.CreatorId);
        Assert.Equal(part.UserId, part2.UserId);
        // TODO: check parts data here...
    }

    // TODO: check pins here, e.g. for the NotePart we get a single pin
    // when the tag is set, with name=tag and value=tag value:
    // [Fact]
    // public void GetDataPins_NoTag_Empty()
    // {
    //     __NAME__Part part = GetEmptyPart();
    //     part.Tag = null;

    //     Assert.Empty(part.GetDataPins());
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     __NAME__Part part = GetEmptyPart();
    //     // TODO: set only the properties required for pins
    //     // in a predictable way so we can test them

    //     List<DataPin> pins = part.GetDataPins(null).ToList();
    //     Assert.Single(pins);

    //     DataPin? pin = pins.Find(p => p.Name == "id" && p.Value == "steph");
    //     Assert.NotNull(pin);
    //     TestHelper.AssertPinIds(part, pin!);
    // }
}
```

### Part Test - Multiple Entities

This template refers to [multiple-entities container parts](#part---multiple-entities):

```cs
using Cadmus.Core;
using System;
using System.Collections.Generic;
using System.Linq;
using Xunit;

namespace Cadmus.__PRJ__.Parts.Test;

public sealed class __NAME__PartTest
{
    private static __NAME__Part GetPart()
    {
        __NAME__PartSeeder seeder = new();
        IItem item = new Item
        {
            FacetId = "default",
            CreatorId = "zeus",
            UserId = "zeus",
            Description = "Test item",
            Title = "Test Item",
            SortKey = ""
        };
        return (__NAME__Part)seeder.GetPart(item, null, null)!;
    }

    private static __NAME__Part GetEmptyPart()
    {
        return new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
        };
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart();

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 =
            TestHelper.DeserializePart<__NAME__Part>(json)!;

        Assert.Equal(part.Id, part2.Id);
        Assert.Equal(part.TypeId, part2.TypeId);
        Assert.Equal(part.ItemId, part2.ItemId);
        Assert.Equal(part.RoleId, part2.RoleId);
        Assert.Equal(part.CreatorId, part2.CreatorId);
        Assert.Equal(part.UserId, part2.UserId);

        Assert.Equal(part.Entries.Count, part2.Entries.Count);
    }

    [Fact]
    public void GetDataPins_NoEntries_Ok()
    {
        __NAME__Part part = GetPart();
        part.Entries.Clear();

        List<DataPin> pins = [.. part.GetDataPins(null)];

        Assert.Single(pins);
        DataPin pin = pins[0];
        Assert.Equal("tot-count", pin.Name);
        TestHelper.AssertPinIds(part, pin);
        Assert.Equal("0", pin.Value);
    }

    [Fact]
    public void GetDataPins_Entries_Ok()
    {
        __NAME__Part part = GetEmptyPart();

        for (int n = 1; n <= 3; n++)
        {
            // TODO add entry to part setting its pin-related
            // properties in a predictable way, so we can test them
        }

        List<DataPin> pins = [.. part.GetDataPins(null)];

        Assert.Equal(5, pins.Count);

        DataPin? pin = pins.Find(p => p.Name == "tot-count");
        Assert.NotNull(pin);
        TestHelper.AssertPinIds(part, pin!);
        Assert.Equal("3", pin!.Value);

        // TODO: assert counts and values e.g.:
        // pin = pins.Find(p => p.Name == "pos-bottom-count");
        // Assert.NotNull(pin);
        // TestHelper.AssertPinIds(part, pin!);
        // Assert.Equal("2", pin.Value);
    }
}
```

## Layer Parts

For layer parts, the same guidelines already listed for the other parts are applicable, with the following additions:

- create a `...LayerFragment` class representing the fragment for the layer part. This is the true data model for the metatextual data represented by the layer. The class must implement `ITextLayerFragment`. Do not add any other property to the class; _by design, the only property of a layer part is its collection of fragments_.

- give the fragment a type ID (via the usual `TagAttribute`), which _must_ begin with the prefix `fr.` (=`PartBase.FR_PREFIX`; note the trailing dot).

- if adding pins in the fragment, just provide the pin's name and value; the other properties will be supplied by the container part. By convention, you should prefix your pin name with the `fr.` prefix (defined in `PartBase.FR_PREFIX`).

Anyway, adding a new layer part would be rarely required, as there is just a generic (parameterized) layer part provided for this: one part, many fragments. You rather have to provide fragments and their tests.

### Layer Fragment Test Template

**Fragment test template** sample:

```cs
public sealed class __NAME__LayerFragmentTest
{
    private static __NAME__LayerFragment GetFragment()
    {
        return new __NAME__LayerFragment
        {
            Location = "1.23",
            // TODO: add properties here...
        };
    }

    [Fact]
    public void Fragment_Has_Tag()
    {
        TagAttribute attr = typeof(__NAME__LayerFragment).GetTypeInfo()
            .GetCustomAttribute<TagAttribute>();
        string typeId = attr != null ? attr.Tag : GetType().FullName;
        Assert.NotNull(typeId);
        Assert.StartsWith(PartBase.FR_PREFIX, typeId);
    }

    [Fact]
    public void Fragment_Is_Serializable()
    {
        __NAME__LayerFragment fr = GetFragment();

        string json = TestHelper.SerializeFragment(fr);
        __NAME__LayerFragment fr2 =
            TestHelper.DeserializeFragment<__NAME__LayerFragment>(json);

        Assert.Equal(fr.Location, fr2.Location);
        // TODO: check properties here...
    }

    // TODO: check pins here, e.g. for the CommentLayerFragment
    // we get a single pin when the tag is set, with name=fr.tag
    // and value=tag value:
    // [Fact]
    // public void GetDataPins_NoTag_0()
    // {
    //     CommentLayerFragment fr = GetFragment();
    //     fr.Tag = null;

    //     Assert.Empty(fr.GetDataPins(null));
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     CommentLayerFragment fr = GetFragment();

    //     List<DataPin> pins = [.. part.GetDataPins(null)];

    //     Assert.Single(pins);
    //     DataPin pin = pins[0];
    //     Assert.Equal("fr.tag", pin.Name);
    //     Assert.Equal("some-tag", pin.Value);
    // }
}
```
