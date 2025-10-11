---
title: "Adding Backend Part Seeders"
parent: "Creating Backend Core"
layout: default
nav_order: 2
---

- [Adding Backend Part Seeders](#adding-backend-part-seeders)
  - [Part Seeder](#part-seeder)
  - [Test Helper](#test-helper)
  - [Part Seeder Test](#part-seeder-test)

# Adding Backend Part Seeders

Part seeders are used to generate mock data for the editor. There can be a part seeder class for each [part class](parts.md).

>The part seeder project requires at the 📦 `Cadmus.Seed` package. Typically you add a seeder for each part or fragment.

## Part Seeder

1. add a `<NAME>PartSeeder.cs` for the seeder class using the template below (replace `__NAME__` with the part name, using the proper case, and `__PRJ__` with the project name).
2. if the seeder does not require configuration options (as it happens in most cases), remove the `__NAME__PartSeederOptions` class, the corresponding `_options` member, and the `IConfigurable<T>` interface of this template.

```cs
using Bogus;
using Cadmus.Core;
using Fusi.Tools.Configuration;
using System;

namespace Cadmus.Seed.__PRJ__.Parts;

/// <summary>
/// Seeder for <see cref="__NAME__Part"/>.
/// Tag: <c>seed.it.vedph.__PRJ__.__NAME__</c>.
/// </summary>
/// <seealso cref="PartSeederBase" />
[Tag("seed.it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__PartSeeder : PartSeederBase
    // TODO: uncomment for configurable seeders, remove otherwise
    // , IConfigurable<__NAME__PartSeederOptions>
{
    // TODO: uncomment for configurable seeders, remove otherwise
    // private __NAME__PartSeederOptions _options;

    // TODO: uncomment for configurable seeders, remove otherwise
    // /// <summary>
    // /// Configures the object with the specified options.
    // /// </summary>
    // /// <param name="options">The options.</param>
    // public void Configure(__NAME__PartSeederOptions options)
    // {
    //     _options = options;
    // }

    /// <summary>
    /// Creates and seeds a new part.
    /// </summary>
    /// <param name="item">The item this part should belong to.</param>
    /// <param name="roleId">The optional part role ID.</param>
    /// <param name="factory">The part seeder factory. This is used
    /// for layer parts, which need to seed a set of fragments.</param>
    /// <returns>A new part or null.</returns>
    /// <exception cref="ArgumentNullException">item or factory</exception>
    public override IPart? GetPart(IItem item, string? roleId,
        PartSeederFactory? factory)
    {
        ArgumentNullException.ThrowIfNull(item);
        // for layer parts only, add this check:
        // if (factory == null)
        //    throw new ArgumentNullException(nameof(factory));

        // TODO: uncomment for configurable seeders, remove otherwise
        // (add more options validation check; if invalid, ret null)
        // if (_options == null)
        // {
        //     return null;
        // }

        __NAME__Part part = new Faker<__NAME__Part>()
            // TODO: add rules for most of the part properties
            // (suggestion: use private methods to mock objects nested
            // in the part), e.g.:
            // .RuleFor(p => p.X, f => TODO: set value or call method)
           .Generate();

        // alternatively, create and fill the part manually:
        // __NAME__Part part = new();

        SetPartMetadata(part, roleId, item);

        return part;
    }
}

/// <summary>
/// Options for <see cref="__NAME__PartSeeder"/>.
/// </summary>
public sealed class __NAME__PartSeederOptions
{
    // TODO: add options here...
}
```

## Test Helper

The test template requires some infrastructure files:

- a minimalist JSON configuration file for the seeders to be tested: `SeedConfig.json` (embedded resource) under your test project's `Assets` folder.
- a `TestHelper` to use this configuration.
- package `Fusi.Microsoft.Extensions.Configuration.InMemoryJson` to read the configuration from the embedded `SeedConfig.json`.

Sample configuration:

```json
{
  "facets": [
    {
      "typeId": "it.vedph.pura.word-forms",
      "name": "forms",
      "description": "Word forms.",
      "colorKey": "31AB54",
      "groupKey": "lexicon",
      "sortKey": "forms"
    }
  ],
  "seed": {
    "options": {
      "seed": 1,
      "baseTextPartTypeId": "it.vedph.token-text",
      "users": [ "zeus" ],
      "partRoles": [],
      "fragmentRoles": []
    },
    "partSeeders": [
      {
        "id": "seed.it.vedph.pura.word-forms"
      }
    ],
    "fragmentSeeders": []
  }
}
```

In this file, add all the parts to a single facet, and inside it add all the parts (under `facets`) and their seeders (under `seed.partSeeders`).

Template for `TestHelper`:

```cs
using System;
using System.IO;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.__PRJ__.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using System.Reflection;
using System.Text;
using Microsoft.Extensions.Hosting;

namespace Cadmus.Seed.__PRJ__.Parts.Test;

static internal class TestHelper
{
    static public Stream GetResourceStream(string name)
    {
        ArgumentNullException.ThrowIfNull(name);

        return Assembly.GetExecutingAssembly().GetManifestResourceStream(
            $"Cadmus.Seed.__PRJ__.Parts.Test.Assets.{name}")!;
    }

    static public string LoadResourceText(string name)
    {
        ArgumentNullException.ThrowIfNull(name);

        using StreamReader reader = new(GetResourceStream(name),
            Encoding.UTF8);
        return reader.ReadToEnd();
    }

    private static IHost GetHost(string config)
    {
        // map
        TagAttributeToTypeMap map = new();
        map.Add(
        [
            // TODO: your parts assemblies here
            // Cadmus.Core
            typeof(StandardItemSortKeyBuilder).Assembly,
            // Cadmus.__PRJ__.Parts
            typeof(YOURPART).Assembly
        ]);

        return new HostBuilder().ConfigureServices((hostContext, services) =>
            {
                PartSeederFactory.ConfigureServices(services,
                    new StandardPartTypeProvider(map),
                        // TODO: your seeder assembly here
                        // Cadmus.Seed.__PRJ__.Parts
                        typeof(YOURSEEDER).Assembly);
            })
            // extension method from Fusi library
            .AddInMemoryJson(config)
            .Build();
    }

    static public PartSeederFactory GetFactory()
    {
        return new PartSeederFactory(GetHost(LoadResourceText("SeedConfig.json")));
    }

    static public void AssertPartMetadata(IPart part)
    {
        Assert.NotNull(part.Id);
        Assert.NotNull(part.ItemId);
        Assert.NotNull(part.UserId);
        Assert.NotNull(part.CreatorId);
    }
}
```

## Part Seeder Test

1. for each part seeder, add a test using the following template (replace `__NAME__` with the part name, `__PRJ__` with the project name).
2. implement at least `Seed_Ok` or create more test methods if you are going to test conditioned seeders.

This test template requires some [infrastructure](#test-helper).

```cs
using Cadmus.Core;
using Fusi.Tools.Configuration;
using System;
using System.Reflection;
using Xunit;

namespace Cadmus.Seed.__PRJ__.Parts.Test;

public sealed class __NAME__PartSeederTest
{
    private static readonly PartSeederFactory _factory =
        TestHelper.GetFactory();
    private static readonly SeedOptions _seedOptions =
        _factory.GetSeedOptions();
    private static readonly IItem _item =
        _factory.GetItemSeeder().GetItem(1, "facet");

    [Fact]
    public void TypeHasTagAttribute()
    {
        Type t = typeof(__NAME__PartSeeder);
        TagAttribute? attr = t.GetTypeInfo().GetCustomAttribute<TagAttribute>();
        Assert.NotNull(attr);
        Assert.Equal("seed.it.vedph.__PRJ__.__NAME__", attr!.Tag);
    }

    [Fact]
    public void Seed_Ok()
    {
        __NAME__PartSeeder seeder = new();
        seeder.SetSeedOptions(_seedOptions);

        IPart? part = seeder.GetPart(_item, null, _factory);

        Assert.NotNull(part);

        __NAME__Part? p = part as __NAME__Part;
        Assert.NotNull(p);

        TestHelper.AssertPartMetadata(p!);

        // TODO: assert properties like:
        // Assert.NotEmpty(p!.Entries);
    }
}
```
