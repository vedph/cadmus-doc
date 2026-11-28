---
title: "Setup Iconclass"
parent: "Recipes"
layout: default
nav_order: 4
---

# Setup Iconclass

To setup [Iconclass](https://iconclass.org) lookup and search, you need to add the Iconclass API service to the backend stack (unless you plan to host it elsewhere) and then simply configure the frontend app similarly to any other API lookup.

The Iconclass project integrated in Cadmus (but also usable as a standalone resource) contains the logic to import the downloadable text documents with Iconclass data into a PostgreSQL database and an API to support lookup and extensive search.

The API provides these essential functions:

- parsing an Iconclass ID (notation).
- building an Iconclass ID in an assisted, multi-step process: the user starts selecting the root class, and then adds more and more lower nodes (keywords or keys) until satisfied.
- quickly lookup an Iconclass entity by its label.

## Backend

See <https://github.com/vedph/iconclass-api/blob/master/DEVELOPER.md> for details.

▶️ (1) add the `Iconclass.Api.Controllers` NuGet package.

▶️ (2) add to your API `Program.cs` in `ConfigureAppServices` or wherever you configure application services (configure options as preferred):

```cs
// Iconclass
IConfigurationSection section = config.GetSection("Iconclass");
services.AddIconclassServices(options =>
{
    options.ConnectionString = config.GetConnectionString("Iconclass");
    options.DataDirectory = section["DataDirectory"];
    options.Languages = section.GetSection("Languages").Get<string[]>();
    options.InitializationDelaySeconds =
        section.GetValue("InitializationDelaySeconds", 0);
    options.WikidataUserAgent = section["WikidataUserAgent"];
});
```

▶️ (3) in `appsettings.json` add Iconclass settings, e.g.:

```json
"ConnectionStrings": {
    /* ... */
    "Iconclass": "Server=localhost;Database=iconclass;User Id=postgres;Password=postgres;Include Error Detail=True"
},
"Iconclass": {
    "DataDirectory": "/data/iconclass",
    "Languages": [ "en", "it", "de", "es", "fr" ],
    "InitializationDelaySeconds": 30,
    "EnableWikidata": true,
    "WikidataUserAgent": "MyApp/1.0 (https://example.org; admin@example.org)"
},
```

> Every setting can be overridden with environment variables (e.g. in a Docker compose script), such as `ConnectionStrings__Iconclass`, `Iconclass__DataDirectory` and `Iconclass__Languages__0`.

▶️ (4) to seed the database, you typically download Iconclass data files into the specified seed directory (see below) or manually do it via the [CLI tool](https://github.com/vedph/iconclass-api/releases) with a command like:

```sh
icocl import /data/iconclass
```

▶️ (5) in your `docker-compose.yml` script for the API service, remember to override the connection string and properly set the Iconclass initialization delay if needed. Also add a volume for seeding data at API startup if you do not do it manually:

```yml
 __PRJ__-api:
    environment:
      ConnectionStrings__Iconclass: "Host=iconclass-db;Username=postgres;Password=postgres;Database=iconclass"
      Iconclass__InitializationDelaySeconds: "30"
    volumes:
     # download iconclass data from its GitHub in ./iconclass-data:
     # git clone https://github.com/iconclass/data
      - ./iconclass-data:/data/iconclass:ro
```

## Frontend

▶️ (1) add NPM packages:

```sh
pnpm i @myrmidon/iconclass-api @myrmidon/iconclass-builder @myrmidon/iconclass-parser @myrmidon/iconclass-ui
```

▶️ (2) add to `app.ts` the lookup provider configuration:

```ts
import {
  ICONCLASS_BUILDER_DIALOG_CONFIG,
  IconclassBuilderDialogComponent,
  IconclassNotation,
  IconclassRefLookupService,
} from '@myrmidon/cadmus-refs-iconclass-lookup';

export class App {
  // ...
  private configureLookup(storage: RamStorageService): void {
    storage.store(LOOKUP_CONFIGS_KEY, [
      // ICONCLASS
      {
        name: 'Iconclass',
        iconUrl: '/img/iconclass128.png',
        description: 'Iconclass classification system',
        label: 'ID',
        service: inject(IconclassRefLookupService),
        itemIdGetter: (item: IconclassNotation) => item?.n,
        itemLabelGetter: (item: IconclassNotation) => item?.txt?.en || item?.n,
        linkTemplate: 'https://iconclass.org/{n}',
        // the More button opens the Iconclass builder
        moreDialog: IconclassBuilderDialogComponent,
        moreDialogConfig: ICONCLASS_BUILDER_DIALOG_CONFIG,
      },
    ]);
  }
}
```

▶️ (3) add to `public/img` the Iconclass icon [iconclass128.png](https://github.com/vedph/cadmus-bricks-shell-v3/tree/master/public/img/iconclass128.png).
