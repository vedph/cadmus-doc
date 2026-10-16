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
  <PackageReference Include="TaxoStore.Api.Controllers" Version="2.0.0" />
  <PackageReference Include="TaxoStore.Core" Version="2.0.0" />
  <PackageReference Include="TaxoStore.PgSql" Version="2.0.0" />
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

To configure the asserted composite ID brick you have additional options. Typically you will add an `asserted-id-tags@en` thesaurus to provide a list of roles for the entities you link to.

You can create many versions of this thesaurus giving them other identifiers, e.g. `asserted-id-tags-pers@en`, `asserted-id-tags-org@en`, etc. if you want to conditionally select them according to the role of the part editor using the brick. Your part must support this by passing its settings down to the brick. For a code example, see the [codicology content parts](https://github.com/vedph/cadmus-codicology-shell/blob/master/projects/myrmidon/cadmus-part-codicology-contents/src/lib/cod-contents-part/cod-contents-part.component.ts).

By default, the tag of an asserted composite ID is picked from a closed list when there is an `asserted-id-tags` thesaurus, or else freely typed. The optional `tagSettings` brick input (of type `AssertedCompositeIdTagSettings`) refines this behavior, and also allows hiding some of the globally configured lookup sources in a specific instance of the component:

- `hiddenLookupProviderIds`: the IDs of the lookup providers to hide. The ID of a provider is the ID of its lookup service (`RefLookupConfig.service.id`, e.g. `viaf`, `whg`, `geonames`), not its human-friendly name.
- `hiddenTaxoTreeIds`: the IDs of the taxonomy trees to hide (`TaxoStoreLookupConfig.treeId`). When all the trees are hidden, the taxonomy source is no longer available. An existing ID whose target comes from a hidden tree can still be edited with its own tree.
- `tagThesaurusMap`: maps the ID of a lookup source (a lookup provider ID or a taxonomy tree ID) to the ID of the thesaurus providing the tags for that source. When the currently selected source has an entry in this map, the tags are loaded from that thesaurus overriding the brick's `idTagEntries` input property. In any other case (no map, source not found in the map, internal source, thesaurus empty or not found) the default behavior applies.

Example:

```ts
public tagSettings: AssertedCompositeIdTagSettings = {
  hiddenLookupProviderIds: ['whg'],
  hiddenTaxoTreeIds: ['food'],
  tagThesaurusMap: {
    // lookup provider ID => thesaurus ID
    viaf: 'asserted-id-tags-viaf',
    // taxonomy tree ID => thesaurus ID
    animals: 'asserted-id-tags-animals',
  },
};
```

The template of your parent component should pass the tag settings down to the child, e.g.:

```html
<cadmus-refs-asserted-composite-id
  [idTagEntries]="idTagEntries"
  [tagSettings]="tagSettings"
  [(id)]="id"
/>
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
animals,Animals,Sample hierarchy of animals.,
food,Food,Sample hierarchy of foods.,
philosophers,Greek Philosophers,Sample flat list of ancient Greek philosophers.,true
```

- 📁 `nodes.csv`:

```csv
tree_n,parent_key,key,label,filtered_label,note
1,,animal,animals,animals,"Kingdom Animalia, i.e. multicellular, eukaryotic organisms."
1,animal,animal.mammal,animal: mammals,animal: mammals,
1,animal.mammal,animal.mammal.dog,animal: mammals: dogs,animal: mammals: dogs,"Domestic dog (Canis familiaris), the ""best friend"" of humans."
1,animal.mammal,animal.mammal.cat,animal: mammals: cats,animal: mammals: cats,Domestic cat
```

- 📁 `mappings.csv` (optional):

```csv
tree_n,node_key,scope,target,label,filtered_label
1,animal,mock-wiki,MW-000001,animal,
1,animal,mock-thesaurus,300000001,animals (fauna),
1,animal.mammal,mock-wiki,MW-000010,mammal,
```

- 📁 `features.csv` (optional):

```csv
tree_n,node_key,name,value,filtered_value
2,fruit.pome.apple,color,red,
2,fruit.pome.apple,color,green,
2,fruit.pome.apple,color,yellow,
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
- `note`: optional note (the column itself can be omitted). Like any other field, it must be wrapped in double quotes when it includes the delimiter, quotes (escaped by doubling them) or line breaks.

Fields in `mappings.csv`:

- `tree_n`: tree number (1-based index from trees.csv) of the mapped node.
- `node_key`: key of the mapped node. The node must be found in `nodes.csv`.
- `scope`: the scope of the target (e.g. `viaf`).
- `target`: the target ID.
- `label`: optional human-friendly label of the target.
- `filtered_label`: optional searchable label of the target (auto-generated from `label` if empty). The column itself can be omitted.

Fields in `features.csv`:

- `tree_n`: tree number (1-based index from trees.csv) of the mapped node.
- `node_key`: key of the node owning the feature. The node must be found in `nodes.csv`.
- `name`: the feature name.
- `value`: the feature value.
- `filtered_value`: optional searchable value (auto-generated from `value` if empty). The column itself can be omitted.

## Frontend

In frontend you can use TaxoStore component directly, or leverage taxonomies from the [asserted composite IDs brick](https://github.com/vedph/cadmus-bricks/blob/master/docs/asserted-composite-id.md), whose demo is available at <https://cadmus-bricks-v3.fusi-soft.com/refs/asserted-composite-id>.

> The demo requires login because it allows writing data in the sample database.

If configured, this component allows you to link not only to internal or external resources, but also to a taxonomy node, while even editing the taxonomy in-place.

▶️ (1) add TaxoStore components to `package.json` (update versions accordingly):

```json
"dependencies": {
    // ...
    "@myrmidon/taxo-store-api": "^1.3.0",
    "@myrmidon/taxo-store-editor": "^1.3.0",
    "@myrmidon/taxo-store-picker": "^1.3.0",
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

### Node Editor

The taxonomies store node editor component allows users to edit or add nodes in the taxonomy (as far as editing capabilities are allowed in the app configuration). In most cases, the node's key can be automatically generated from its label. The default logic for generating it is:

- trim the label and remove diacritics from its letters;
- convert it to lowercase;
- replace each sequence of whitespaces with a single `_`, each `:` with `.`, each `'` with `_`;
- keep only letters `a`-`z`, digits, `_`, `-` and `.`, dropping any other character (including letters with no `a`-`z` base, like Greek letters or `ß`).

For instance, `Roberto d'Angiò` becomes `roberto_d_angio`.

To use your own key generator, provide a `TaxoStoreEditorConfig` object with the `TAXO_STORE_EDITOR_CONFIG` injection token, e.g. in `app.config.ts`:

```ts
import { TAXO_STORE_EDITOR_CONFIG, TaxoStoreEditorConfig } from '@myrmidon/taxo-store-editor';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    {
      provide: TAXO_STORE_EDITOR_CONFIG,
      useValue: {
        // the label passed to the generator is trimmed
        keyGenerator: (label: string) => label.toUpperCase().replace(/\s+/g, '-'),
      } satisfies TaxoStoreEditorConfig,
    },
  ],
};
```

> The configuration is optional: when it is not provided, or its `keyGenerator` is not set, the default logic is used. The generated key is just a suggestion, which users can still edit before saving.
