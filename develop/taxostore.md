# Adding Taxonomy Store

- 📖 [documentation](../linking/taxonomies.md)

- [Adding Taxonomy Store](#adding-taxonomy-store)
  - [Backend](#backend)
    - [CSV Format](#csv-format)
  - [Frontend](#frontend)

## Backend

In your API project:

1. **add package references**:
   - `TaxoStore.Core`
   - `TaxoStore.PgSql`
   - `TaxoStore.Api.Controllers`
   - `Cadmus.TaxoStore.Parts`
   - `Cadmus.Seed.TaxoStore.Parts`

2. **add service configuration** to `Program.cs`:

   ```cs
   #region Taxo Store
   private static void ConfigureTaxoStoreServices(IServiceCollection services,
       IConfiguration config)
   {
       services.AddTaxoStoreServices(options =>
       {
           // get connection string
           string? connectionString = config.GetConnectionString("TaxoStore");
           if (string.IsNullOrEmpty(connectionString))
           {
               throw new InvalidOperationException(
                   "Connection string 'TaxoStore' not found.");
           }
           options.ConnectionString = connectionString;

           // read auto-initialization flag
           options.EnableAutoInitialization =
               config.GetValue("TaxoStore:EnableAutoInitialization", true);

           // read initialization delay (useful for Docker/PostgreSQL startup)
           options.InitializationDelaySeconds =
               config.GetValue("TaxoStore:InitializationDelaySeconds", 0);
       });
   }
   #endregion
   ```

3. **call the services configuration method** in `Program.ConfigureAppServices`:

   ```cs
   // taxostore
   ConfigureTaxoStoreServices(services, config);
   ```

4. **add controllers** in `Program.Main`:

   ```cs
   builder.Services.AddControllers()
     .AddApplicationPart(typeof(ItemController).Assembly)
     // taxostore
     .AddApplicationPart(typeof(TaxoTreeController).Assembly)
     .AddControllersAsServices();
   ```

5. **add settings** to `appsettings.json`:

   ```json
   {
     "ConnectionStrings": {
       "TaxoStore": "Server=localhost;Database=taxo;User Id=postgres;Password=postgres;Include Error Detail=True"
     },
     "TaxoStore": {
       "EnableAutoInitialization": true,
       "InitializationDelaySeconds": 0
     }
   }
   ```

   > ⚠️ When deploying you will typically need to override `InitializationDelaySeconds` (usually in your `docker-compose.yml` script) in order to allow the database service to start before seeding the store.

6. **add seed data**: if you want to seed data in the taxostore, add a `taxo` folder in your `wwwroot` folder with 2 files:

- `wwwroot/taxo/trees.csv`: tree definitions. Each tree is implicitly numbered with an ordinal when importing it, which makes it easier to refer nodes to it. The first imported tree is 1, the second 2, and so forth. So, in the nodes file you will refer to 1 for the first tree, 2 for the second, and so forth.
- `wwwroot/taxo/nodes.csv`: node data.

If these files exist, they will be automatically used for seeding when the database is first created.

### CSV Format

- 📁 `trees.csv`:

```csv
id,name,note
products,Products,"Product taxonomy"
categories,Categories,"Category hierarchy"
```

- 📁 `nodes.csv`:

```csv
tree_n,parent_key,key,label,filtered_label,flags
1,,electronics,Electronics,electronics,
1,electronics,computers,Computers,computers,
1,electronics,phones,Mobile Phones,mobile phones,
2,,food,Food & Beverages,food beverages,
```

Fields in `trees.csv`:

- `id`: unique tree identifier (key), used to reference the tree externally.
- `name`: human-friendly display name for the tree.
- `note`: optional descriptive note about the tree.

Fields in `nodes.csv`:

- `tree_n`: tree number (1-based index from trees.csv).
- `parent_key`: key of the parent node (empty for root nodes).
- `key`: unique identifier within the tree.
- `label`: display label.
- `filtered_label`: searchable label (auto-generated if empty).
- `flags`: optional flags string.

## Frontend

1. **add packages**: `pnpm i @myrmidon/taxo-store-api @myrmidon/taxo-store-editor @myrmidon/taxo-store-picker @myrmidon/cadmus-part-taxo-ui @myrmidon/cadmus-part-taxo-pg`.

2. **add the part editor** in `part-editor-keys.ts`:

   ```ts
   import { TAXO_STORE_NODES_PART_TYPEID } from "@myrmidon/cadmus-part-taxo-ui";

   const TAXONOMY = "taxonomy";

   export const PART_EDITOR_KEYS: PartEditorKeys = {
     // ...
     // taxonomy
     [TAXO_STORE_NODES_PART_TYPEID]: {
       part: TAXONOMY,
     },
   };
   ```

3. **add the editor route** in `app.routes.ts`:

   ```ts
   export const routes: Routes = [
     // ...
     // taxostore
     {
       path: "items/:iid/taxonomy",
       loadChildren: () =>
         import("@myrmidon/cadmus-part-taxo-pg").then(
           (module) => module.CADMUS_PART_TAXO_PG_ROUTES,
         ),
       canActivate: [jwtGuard],
     },
   ];
   ```

4. **configure lookup service**: if you use taxostore in asserted composite link IDs, configure taxostore lookup in `app.ts`:

```ts
import {
  LOOKUP_TAXOSTORE_CONFIGS_KEY,
  TaxoStoreLookupConfig,
} from '@myrmidon/cadmus-refs-asserted-ids';

export class App {
    // ...
    constructor() {
        // ...
        // configure taxonomy lookup for asserted composite IDs
        this.configureTaxoLookup(storage);
    }

    private configureTaxoLookup(storage: RamStorageService): void {
        storage.store(LOOKUP_TAXOSTORE_CONFIGS_KEY, [
            // TODO: configure trees as desired
            {
                treeId: 'animals',
                treeName: 'animals',
                canEdit: false,
                canAdd: false,
                canDelete: false,
            },
            {
                treeId: 'food',
                treeName: 'food',
                canEdit: false,
                canAdd: false,
                canDelete: false,
            },
        ] as TaxoStoreLookupConfig[]);
    }
```
