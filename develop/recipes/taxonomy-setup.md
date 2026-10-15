---
title: "Setup Taxonomies"
parent: "Recipes"
layout: default
nav_order: 1
---

# Setup Taxonomies

- [taxonomies documentation](../../linking/taxonomies.md)

Setting up taxonomies to use with Cadmus requires updating your backend API and your frontend.

## Backend

### API

In your API project:

▶️ (1) add packages to the required projects in your API's `.csproj` file (update version numbers accordingly):

```xml
<ItemGroup>
  <PackageReference Include="TaxoStore.Api.Controllers" Version="1.1.0" />
  <PackageReference Include="TaxoStore.Core" Version="1.1.0" />
  <PackageReference Include="TaxoStore.PgSql" Version="1.1.0" />
</ItemGroup>
```

▶️ (2) add the TaxoStore connection string to your `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "TaxoStore": "Server=localhost;Database=trees;User Id=postgres;Password=postgres;Include Error Detail=True"
  },
  "TaxoStore": {
    "EnableAutoInitialization": true,
    "InitializationDelaySeconds": 10
  }
}
```

Configuration options in `TaxoStore`:

- `EnableAutoInitialization`: when `true`, automatically creates the database schema and seeds data on startup (default: `true`).
- `InitializationDelaySeconds`: delay before initialization starts, useful in Docker environments where PostgreSQL needs startup time (default: `0`).

▶️ (3) in `Program.cs` register services and controllers:

```cs
using TaxoStore.Api.Controllers;
using TaxoStore.Api.Controllers.Services;

public static class Program
{
    // ...

    private static void ConfigureAppServices(IServiceCollection services,
        IConfiguration config)
    {
        // ...
        // TaxoStore (from wwwroot/taxo/trees.csv and nodes.csv)
        services.AddTaxoStoreServices(options =>
        {
            string? connectionString = config.GetConnectionString("TaxoStore");
            if (string.IsNullOrEmpty(connectionString))
            {
                throw new InvalidOperationException(
                    "Connection string 'TaxoStore' not found.");
            }

            options.ConnectionString = connectionString;

            options.EnableAutoInitialization =
                config.GetValue("TaxoStore:EnableAutoInitialization", true);

            options.InitializationDelaySeconds =
                config.GetValue("TaxoStore:InitializationDelaySeconds", 0);
        });
    }

    public static async Task<int> Main(string[] args)
    {
        // ...
        // add controllers from libraries
        builder.Services.AddControllers()
            // Cadmus.Api.Controllers
            .AddApplicationPart(typeof(ItemController).Assembly)
            // TaxoStore
            .AddApplicationPart(typeof(TaxoTreeController).Assembly)
            .AddControllersAsServices();

            WebApplication app = builder.Build();

            // ...
    }
}
```

### Data Seed

The TaxoStore supports automatic database seeding from CSV files. When the database is created for the first time, it will automatically import data from CSV files if they are present.

By default, `AddTaxoStoreServices` looks for seed files at:

- `wwwroot/taxo/trees.csv`: tree definitions. Each tree is implicitly numbered with an ordinal when importing it, which makes it easier to refer nodes to it. The first imported tree is 1, the second 2, and so forth. So, in the nodes file you will refer to 1 for the first tree, 2 for the second, and so forth.
- `wwwroot/taxo/nodes.csv`: node data.
- `wwwroot/taxo/mappings.csv`: node mappings data. This file is optional.

If these files exist, they will be automatically used for seeding when the database is first created.

**CSV File Formats:**

- 📁 `trees.csv`:

```csv
id,name,note,flat
products,Products,"Product taxonomy",
categories,Categories,"Category hierarchy",
colors,Colors,"A flat list of colors",true
```

- 📁 `nodes.csv`:

```csv
tree_n,parent_key,key,label,filtered_label,flags,note
1,,electronics,Electronics,electronics,,
1,electronics,computers,Computers,computers,,"Desktops, laptops, and servers"
1,electronics,phones,Mobile Phones,mobile phones,,
2,,food,Food & Beverages,food beverages,,
3,,red,Red,red,,
```

- 📁 `mappings.csv` (optional):

```csv
tree_n,node_key,scope,target,label,filtered_label
1,computers,wikidata,Q68,computer,
1,computers,getty,300212998,computers,
1,computers,alias,it,Calcolatóri,calcolatori
3,red,getty,300126225,red (color),
```

Fields in `trees.csv`:

- `id`: unique tree identifier (key), used to reference the tree externally.
- `name`: human-friendly display name for the tree.
- `note`: optional descriptive note about the tree.
- `flat`: optional column; `true`, `1`, `yes` or `y` (case-insensitive) for a flat tree, anything else (or empty) for a hierarchical tree. Nodes of a flat tree must have an empty `parent_key`.

Fields in `nodes.csv`:

- `tree_n`: tree number (1-based index from trees.csv).
- `parent_key`: key of the parent node (empty for root nodes).
- `key`: unique identifier within the tree.
- `label`: display label.
- `filtered_label`: searchable label (auto-generated if empty).
- `flags`: optional flags string.
- `note`: optional note (the column itself can be omitted). Like any other field, it must be wrapped in double quotes when it includes the delimiter, quotes (escaped by doubling them) or line breaks.

Fields in `mappings.csv`:

- `tree_n`: tree number (1-based index from trees.csv) of the mapped node.
- `node_key`: key of the mapped node. The node must be found in `nodes.csv`.
- `scope`: the scope of the target (e.g. `viaf`).
- `target`: the target ID.
- `label`: optional human-friendly label of the target.
- `filtered_label`: optional searchable label of the target (auto-generated from `label` if empty). The column itself can be omitted.

## Frontend

In frontend you can use TaxoStore component directly, or leverage taxonomies from the [asserted composite IDs brick](https://github.com/vedph/cadmus-bricks/blob/master/docs/asserted-composite-id.md), whose demo is available at <https://cadmus-bricks-v3.fusi-soft.com/refs/asserted-composite-id>.

> The demo requires login because it allows writing data in the sample database.

If configured, this component allows you to link not only to internal or external resources, but also to a taxonomy node, while even editing the taxonomy in-place.

▶️ (1) add TaxoStore components to `package.json` (update versions accordingly):

```json
"dependencies": {
    // ...
    "@myrmidon/taxo-store-api": "^1.2.1",
    "@myrmidon/taxo-store-editor": "^1.2.1",
    "@myrmidon/taxo-store-picker": "^1.2.1",
}
```

▶️ (2) in `app.ts` configure TaxoStore lookup:

```ts
private configureTaxoLookup(storage: RamStorageService): void {
  storage.store(LOOKUP_TAXOSTORE_CONFIGS_KEY, [
    // add your taxonomies here, e.g.:
    {
      treeId: 'topics',
      treeName: 'topics',
      canEdit: true,
      canAdd: true,
      canDelete: false,
    },
    {
      treeId: 'subjects',
      treeName: 'subjects',
      canEdit: true,
      canAdd: true,
      canDelete: false,
    },
    // ... etc.
  ] as TaxoStoreLookupConfig[]);
}

// in your app constructor:

constructor() {
  // ...
  // configure TaxoStore
  this.configureTaxoLookup(storage);
}
```
