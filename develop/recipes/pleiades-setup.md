---
title: "Setup Pleiades"
parent: "Recipes"
layout: default
nav_order: 3
---

# Setup Pleiades

To setup Pleiades lookup and search, you need to add the Pleiades API service to the backend stack (unless you plan to host it elsewhere) and then simply configure the frontend app similarly to any other API lookup.

## Backend

To host the Pleiades backend in your Cadmus stack, you must add two services:

- a PostgreSQL PostGIS-enabled service.
- the [Pleiades API](https://github.com/vedph/pleiades-api).

▶️ Add services like in this example `docker-compose.yml`:

- **localhost example**:

```yml
  pleiades-pgsql:
    image: postgis/postgis
    container_name: pleiades-pgsql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      # pick any port on the host side, here 5434
      - 127.0.0.1:5434:5432
    # uncomment volume to persist data beyond container lifetime
    #volumes:
    #  - pleiades-vol:/var/lib/postgresql

  # Pleiades API
  pleiades-api:
    image: vedph2020/pleiades-api:3.0.2
    container_name: pleiades-api
    restart: unless-stopped
    ports:
      - 127.0.0.1:29139:8080
    depends_on:
      - pleiades-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=Server=pleiades-pgsql;User ID=postgres;Password=postgres;Port=5434;Database=pleiades;
      # the seconds delay before database seed starts
      - SEEDDELAY=30
    #volumes:
      # 👉 place dump files in host dump folder here
      #- ./dump:/var/pleiades

# ... other services ...

#volumes:
#  pleiades-vol:
```

- **web server example**:

```yml
services:
  # Postgis
  pleiades-pgsql:
    image: postgis/postgis
    container_name: pleiades-pgsql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      # pick any port on the host side, here 5434
      - 127.0.0.1:5434:5432
    volumes:
      - pleiades-vol:/var/lib/postgresql

  # Pleiades API
  pleiades-api:
    image: vedph2020/pleiades-api:3.0.2
    container_name: pleiades-api
    restart: unless-stopped
    ports:
      # pick any port on the host side, here 29139
      - 127.0.0.1:29139:8080
    depends_on:
      - pleiades-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      # note 5432 despite the host-side mapped 5434
      - CONNECTIONSTRINGS__DEFAULT=User ID=postgres;Password=postgres;Host=pleiades-pgsql;Port=5432;Database=pleiades;
      # 👉 add your client origin(s) here, e.g.:
      - ALLOWEDORIGINS__0=https://cadmus-bricks-v3.fusi-soft.com
      # the seconds delay before database seed starts
      - SEEDDELAY=30
      - DATA__SOURCEDIR=/var/pleiades
    volumes:
      # 👉 place dump files in host dump folder here
      - ./dump:/var/pleiades

# ... other services ...

volumes:
  pleiades-vol:
```

> 💡 It is a good practice to add `pleiades-api` as an additional `depends_on` item in your consumer Cadmus app.

As explained in the [Pleiades API](https://github.com/vedph/pleiades-api) repository, you will need to:

1. download the latest Pleiades data from the [official site](http://atlantides.org/downloads/pleiades/json/).
2. create the Pleiades database by importing Pleiades data. You will need the `pleitool` CLI app you can download from [Releases](https://github.com/vedph/pleiades-api/releases) in the Pleiades site.
3. dump the Pleiades database into binary files for fast restore.
4. upload these binary files into the `dump` folder of your server, which should be a subfolder of the folder containing your `docker-compose.yml` file. Once done, when you first run the services the API will create and seed the database at startup.

For steps 2-3 here are the sample CLI commands:

```sh
# pleiades-places.json is the JSON file unzipped from downloaded Pleiades data
./pleitool import-graph c:/users/dfusi/desktop/pleiades-places.json

# pleiades-profiles.json is from Pleiades API repository
./pleitool index c:/users/dfusi/desktop/pleiades-profile.json -c

./pleitool pop-spatial
```

## Frontend

In your frontend Cadmus editor:

▶️ (1) ensure to install the [ngx-maplibre-gl](https://github.com/maplibre/ngx-maplibre-gl) Angular Maplibre wrapper and its dependencies (see <https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-refs-pleiades-lookup/README.md>).

▶️ (2) install packages:

```sh
pnpm i @myrmidon/cadmus-refs-lookup @myrmidon/cadmus-refs-pleiades-lookup
```

▶️ (3) add to `env.js` the base URL for your Pleiades API, e.g.:

```js
// local example:
window.__env.pleiadesUrl = "http://localhost:5000/api/";

// remote example:
window.__env.pleiadesUrl = "https://pleiades-api.fusi-soft.com/api/";
```

▶️ (4) to use Pleiades among the providers of a `RefLookupSetComponent` copy the icon from [public/img/pleiades128.png](https://github.com/vedph/cadmus-bricks-shell-v3/tree/master/public/img) and add a configuration like:

```ts
import {
  PLEIADES_LOOKUP_OPTIONS_DIALOG_CONFIG,
  PleiadesLookupOptionsComponent,
  PleiadesRefLookupService,
  PleiadesSearchResult,
} from '@myrmidon/cadmus-refs-pleiades-lookup';

// in app.ts call configureLookup from your constructor...

private configureLookup(storage: RamStorageService): void {
    storage.store(LOOKUP_CONFIGS_KEY, [
      // ...
      {
        name: 'Pleiades',
        iconUrl: '/img/pleiades128.png',
        description: 'Pleiades gazetteer of ancient places',
        label: 'ID',
        service: inject(PleiadesRefLookupService),
        itemIdGetter: (item: PleiadesSearchResult) => item?.id,
        itemLabelGetter: (item: PleiadesSearchResult) => item?.name,
        // optional: let users set all the filters in the lookup options
        optDialog: PleiadesLookupOptionsComponent,
        optDialogConfig: PLEIADES_LOOKUP_OPTIONS_DIALOG_CONFIG,
      }
    ]);
}
```

> To let users switch from the lookup to the full search, set [hasMore]="true" on the lookup and open PleiadesQueryDialogComponent on its `moreRequest`. See the [Pleiades brick documentation](https://github.com/vedph/cadmus-bricks-shell-v3/tree/master/projects/myrmidon/cadmus-refs-pleiades-lookup) for more.
