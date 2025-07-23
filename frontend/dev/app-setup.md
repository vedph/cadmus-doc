---
title: "Creating Frontend App"
parent: "Developing Frontend"
layout: default
nav_order: 7
---

- [Creating Frontend App](#creating-frontend-app)
  - [1. Create Angular App](#1-create-angular-app)
  - [2. Install Packages](#2-install-packages)
  - [3. Setup Environment](#3-setup-environment)
  - [4. Fine-Tune Angular Settings](#4-fine-tune-angular-settings)
  - [5. Add Assets](#5-add-assets)
  - [6. Add Cadmus Infrastructure](#6-add-cadmus-infrastructure)
  - [7. Implement App Component](#7-implement-app-component)
  - [8. Configure Routes](#8-configure-routes)
  - [9. Configure App](#9-configure-app)
  - [10. Add Preview Styles](#10-add-preview-styles)
  - [11. Add Docker Support](#11-add-docker-support)
  - [12. Add Readme](#12-add-readme)

# Creating Frontend App

In what follows, `__PRJ__` is the placeholder for your Cadmus project's short name.

## 1. Create Angular App

▶️ (1) create a **new Angular app**: use SCSS rather than CSS, and don't enable SSR (as per default option):

```sh
ng new cadmus-__PRJ__-app
```

>If you are creating an app for the only purpose of developing component libraries in it, our convention is naming it as `-shell` rather than `-app`.

▶️ (2) enter the newly created directory and **add Angular Material** and **Angular localization package**:

```bash
cd cadmus-__PRJ__-app
ng add @angular/material
ng add @angular/localize
```

>For Angular Material, use a _custom theme_ or pick the theme you prefer, and answer Yes when prompted to _setup global typography_ styles. After installing, be sure that there is no prebuilt Angular Material style set in `angular.json`. The localization package instead is a development package which is required by some localization-ready components such as the authentication libraries (`@myrmidon/auth-jwt-*`). You can also just add the NPM package via `npm -i --save-dev @angular/localize`.

💡 The `$localize` function should be defined when running the schematics `ng add @angular/localize`. If you did not run it, or you have legacy code from older CLI apps, it might happen that you are missing some of the changes done by it. In this case, you might get an undefined error at runtime (or build time) for `$localize`, or an error like this:

```txt
core.mjs:40035 Uncaught Error: It looks like your application or one of its dependencies is using i18n.
Angular 9 introduced a global `$localize()` function that needs to be loaded.
Please run `ng add @angular/localize` from the Angular CLI.
(For non-CLI projects, add `import '@angular/localize/init';` to your `polyfills.ts` file.
For server-side rendering applications add the import to your `main.server.ts` file.)
```

In this case, ensure that these settings are properly configured:

1. you must have installed the NPM package `@angular/localize` (under `devDependencies`).
2. you must add `@angular/localize/init` to your `polyfills` in `angular.json`, e.g. `"polyfills": ["zone.js", "@angular/localize/init"],` under `projects/NAME/architect/build/options` and `projects/NAME/architect/test/options`. In older code you might rather have a `polyfills.ts` file which is imported in `angular.json`; in this case, add the import there under the app imports.
3. ensure `main.ts` has the reference _at the very top_:

    ```ts
    /// <reference types="@angular/localize" />
    ```

>For Angular Material animations you should also add this: `npm i @angular/animations`.

4. in each `tsconfig` file (root `tsconfig.json`, `tsconfig.app.json`, `projects/LIB/tsconfig.lib.json`, `projects/LIB/tsconfig-prod.lib.json`) ensure that under `compilerOptions` you have:

```json
"types": [ "@angular/localize" ],
```

>While `tsconfig.lib.prod.json` typically extends from `tsconfig.lib.json`, it's meant for production builds and can override settings. Having the type declaration in both ensures consistent behavior across development and production builds.

▶️ (3) ensure to apply some [M3 theme](https://material.angular.io/guide/theming) in your app's `styles.scss`:

```scss
@use "@angular/material" as mat;
@import "preview-styles.css";

@include mat.core();

$light-theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      primary: mat.$blue-palette,
    ),
  )
);

$accent-theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      primary: mat.$violet-palette,
    ),
  )
);

$error-theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      primary: mat.$red-palette,
    ),
  )
);

html {
  @include mat.all-component-themes($light-theme);
  & {
    color-scheme: light;
  }
}

html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}

.mat-primary, .mat-accent {
  @include mat.all-component-colors($accent-theme);
}

.mat-error, .mat-warn {
  @include mat.all-component-colors($error-theme);
}

mat-icon.mat-accent, mat-icon.mat-primary {
  color: var(--mdc-filled-text-field-focus-label-text-color) !important;
}
mat-icon.mat-error, mat-icon.mat-warn {
  color: var(--mat-form-field-error-focus-trailing-icon-color) !important;
}
```

>This configuration allows to use Cadmus libraries with classes `mat-primary`, `mat-warn` (or `mat-error`), and `mat-accent` without using legacy compatibility mixins. Since Angular 18 the color directive (which automatically changed the target component's theme) has been removed, and you could use compatibility mixins; but these have side effects. In the approach used here, we just create different themes for color variants. In M2 Cadmus I used classes `mat-primary` for emphasized components and `mat-warn` for warn/error components. In M3 I have a default theme, an error theme corresponding to mat-warn, and an accent theme corresponding to mat-primary (and mat-accent). See also my [SO post about M3 theming](https://stackoverflow.com/questions/79230742/proper-angular-material-v3-theming).

💡 If you are dealing with an existing app using CSS rather than SCSS:

1. rename `styles.css` to `styles.scss`.
2. in `angular.json`, rename `styles.css` in `styles.scss` in the `styles` array.
3. configure Angular CLI to Use SCSS for new components: in `angular.json`, locate the `schematics` section. If it doesn't exist, add it under your project's root. Add or update the `@schematics/angular:component` section to specify scss as the default style extension:

```json
"schematics": {
  "@schematics/angular:component": {
    "style": "scss"
  }
}
```

## 2. Install Packages

▶️ 1. Install the typical Cadmus packages via NPM:

```bash
npm i @auth0/angular-jwt @myrmidon/auth-jwt-admin @myrmidon/auth-jwt-login --force

npm i @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-graph-ui-ex @myrmidon/cadmus-graph-pg-ex @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search --force
npm i @myrmidon/cadmus-preview-pg @myrmidon/cadmus-preview-ui @myrmidon/cadmus-profile-core --force

npm i @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui --force
npm i @myrmidon/cadmus-part-philology-pg @myrmidon/cadmus-part-philology-ui --force

npm i @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-flags-pg @myrmidon/cadmus-flags-ui @myrmidon/cadmus-refs-asserted-ids @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-decorated-ids @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-mat-physical-size @myrmidon/cadmus-refs-lookup @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-state @myrmidon/cadmus-text-block-view @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui @myrmidon/cadmus-ui @myrmidon/cadmus-ui-flag-set @myrmidon/cadmus-ui-pg @myrmidon/ngx-mat-tools @myrmidon/ngx-tools @myrmidon/paged-data-browsers diff-match-patch ts-md5 --force

npm i @myrmidon/cadmus-text-ed @myrmidon/cadmus-text-ed-md @myrmidon/cadmus-text-ed-txt --force

npm i @types/diff-match-patch --save-dev --force
```

The above packages are fairly typical, but you might well omit those you are not interested in, e.g. general parts or philology parts, or some [bricks](https://github.com/vedph/cadmus-bricks-shell-v3). Some of the legacy third party libraries may require `--force`.

💡 If you want some data statistics in your editor, also do:

1. `npm i ngx-echarts echarts @myrmidon/cadmus-statistics`
2. remember to configure echarts adding in `app.config.ts` under `providers`:

```ts
import { NgxEchartsModule } from 'ngx-echarts';

// in providers array:
    importProvidersFrom(
      NgxEchartsModule.forRoot({
        echarts: () => import('echarts'),
      })
    ),
```

▶️ 2. Typically you will also need **Monaco editor** and **Markdown**:

- [NG essentials](https://github.com/cisstech/nge): `npm i @cisstech/nge monaco-editor --force`.
- [ngx-markdown](https://github.com/jfcere/ngx-markdown) if you have components _displaying_ Markdown: `npm i ngx-markdown marked --force`.

⚠️ Note that for such libraries you should also import the providers in `app.config.ts` like:

```ts
import { NgeMonacoModule } from '@cisstech/nge/monaco';
import { NgeMarkdownModule } from '@cisstech/nge/markdown';

// ...

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    importProvidersFrom(NgeMonacoModule.forRoot({})),
    importProvidersFrom(NgeMarkdownModule),
    // ...
  ]
};
```

>Even though you usually all what you have to do is installing the listed packages, be sure to _follow the directions provided by each library_ when installing it.

💡 If you are directly or indirectly using legacy libraries like `leaflet`, add this option under `architect/build/options` to avoid a build warning:

```json
"allowedCommonJsDependencies": [
  "leaflet"
]
```

## 3. Setup Environment

This step is essential to let the frontend find the server, while allowing us to manually edit this URI after a Docker image has been built, when [deploying](../../deploy/app#4-configure-frontend-app) the web app.

▶️ (1) under `public` add an `env.js` file for project-dependent environment variables, with this content (replace the port number, in this sample 60849, with your backend API port number):

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:60849/api/";
  window.__env.version = "0.0.1";
  // enable thesaurus import in thesaurus list for admins
  window.__env.thesImportEnabled = true;
})(this);
```

>💡 You might need additional settings here, like e.g. a Mapbox GL API token, a Geonames account name, etc.

📖 If you are going to use the [external bibliography API](https://github.com/vedph/cadmus_biblioapi), also add its URL here, e.g.:

```js
window.__env.biblioApiUrl = 'http://localhost:60058/api/';
```

In this case typically you will also need to install the bibliography packages:

```bash
npm i @myrmidon/cadmus-biblio-core @myrmidon/cadmus-biblio-api @myrmidon/cadmus-biblio-ui @myrmidon/cadmus-part-biblio-ui @myrmidon/cadmus-part-biblio-pg --force
```

Later, in your app's `part-editor-keys.ts`, remember to _setup the route to the bibliography part editor_ like:

```ts
import { EXT_BIBLIOGRAPHY_PART_TYPEID } from '@myrmidon/cadmus-part-biblio-ui';
const BIBLIO = 'biblio';
// ...

export const PART_EDITOR_KEYS: PartEditorKeys = {
  [EXT_BIBLIOGRAPHY_PART_TYPEID]: {
    part: BIBLIO,
  },
  // ... etc.
};
```

⚠️ Since Angular 18 the `public` folder is the place where you should place items to be copied. So, in this case just place the `env.js` file there. No change is required in `angular.json` because it already has a glob catch-all pattern pointing to the `public` folder. Before this version, you typically added `src/env.js` to the `assets` array in `projects/APPNAME/architect/build/options/assets`.

▶️ (2) in `src/index.html` add an import for `env.js` to your `head` element:

```html
<head>
  ...
  <script src="env.js"></script>
</head>
```

💡 Also, you can change the web app's `title` in `head` to a more human-friendly name, and paste some CSS and HTML for a fancy loading message:

```html
<html>
  <head>
    ...
    <style>
      .loading-wrapper {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        background-color: #fff;
        z-index: 9999;
        font-family: "Roboto", sans-serif;
        color: #3f51b5;
        text-align: center;
      }
      .app-loading {
        font-size: 1.5rem;
        margin-bottom: 20px;
        font-weight: 300;
      }
      .spinner {
        border: 4px solid rgba(63, 81, 181, 0.1);
        border-radius: 50%;
        border-top: 4px solid #3f51b5;
        width: 40px;
        height: 40px;
        animation: spin 1s linear infinite;
      }
      @keyframes spin {
        0% {
          transform: rotate(0deg);
        }
        100% {
          transform: rotate(360deg);
        }
      }
    </style>
  </head>
  <body class="mat-typography">
    <app-root>
      <div class="loading-wrapper">
        <div class="app-loading">Loading Cadmus __PRJ__...</div>
        <div class="spinner"></div>
      </div>
    </app-root>
  </body>
</html>
```

## 4. Fine-Tune Angular Settings

This is suggested to enable source maps in production and avoid nasty warnings after compilation.

▶️ In `angular.json` you will typically have to raise the warning limits for your `budget` size if getting a warning after building.

## 5. Add Assets

This is optional and depends on your visuals.

▶️ (1) copy the required icons and images in `public/img`: usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a couple of banner images for the homepage (`banner-512.jpg`, `banner-1024.jpg`). The logo is used in the `app.component`'s template for the main toolbar, while banner images are used in the default homepage placeholder.

▶️ (2) also, if using [lookup sets](https://github.com/vedph/cadmus-bricks-shell-v3/blob/master/projects/myrmidon/cadmus-refs-lookup/README.md#lookup-set), typically you will also need an icon for each lookup source, e.g. VIAF, GeoNames, etc. You can find some of these icons in the image folder of the [Cadmus shell app](https://github.com/vedph/cadmus-shell-v3/tree/master/src/assets/img).

## 6. Add Cadmus Infrastructure

▶️ (1) add some extension points, optionally adding new entries for your new parts (see [dynamic lookup](https://github.com/vedph/cadmus_doc/blob/master/core/dynamic-lookup)):

- 📁 `src/app/index-lookup-definitions.ts` with this content:

```ts
import { IndexLookupDefinitions } from '@myrmidon/cadmus-core';

export const INDEX_LOOKUP_DEFINITIONS : IndexLookupDefinitions = {}
```

You can add to the definitions object all the pin-based lookup definitions you will need in your project, e.g.:

```ts
// ...
import { METADATA_PART_TYPEID } from '@myrmidon/cadmus-part-general-ui';

export const INDEX_LOOKUP_DEFINITIONS: IndexLookupDefinitions = {
  // item's metadata
  meta_eid: {
    typeId: METADATA_PART_TYPEID,
    name: 'eid',
  },
};
```

- 📁 `src/app/item-browser-keys.ts` with this content:

```ts
/**
 * Mapping between item browser keys and their routes, used to avoid
 * long and complex names in the route by replacing the ID with an alias.
 */
export const ITEM_BROWSER_KEYS = {
  // e.g. ['it.vedph.item-browser.mongo.hierarchy']: 'hierarchy'
};
```

- 📁 `src/app/part-editor-keys.ts`: this is the only file with a real content, the others being just extension points. You must specify here the connection of each part or fragment ID with its hosting library in constant `PART_EDITOR_KEYS`. This object has a property named after each part/fragment type ID, with a value equal to an object with `part` equal to the library ID, and optionally `fragments` (when the part is a layer part). This property is an object with a property for each fragment type for the layer part, named after the fragment type ID, with a value equal to the library ID.

>⚠️ If using external bibliography, remember to [add the route](#3-setup-environment) for its editor.

▶️ (2) copy these folders (each corresponding to an app's page component) into your app's `src/app` folder from the [reference project](https://github.com/vedph/cadmus-shell-v3):

- `home` (adjust the homepage contents according to your project)
- `login-page`
- `manage-users-page`
- `register-user-page`
- `reset-password`

💡 If you want statistics, add the statistics page from `edit-frame-stats-page` and its route in `app.routes.ts`:

```ts
  {
    path: 'stats',
    component: EditFrameStatsPageComponent,
    canActivate: [AuthJwtGuardService],
  },
```

Of course you will then need to add links to this route in your app.

## 7. Implement App Component

The default app component must be updated with a code like that found in the [Cadmus shell](https://github.com/vedph/cadmus-shell-v3/tree/master/src/app):

- [app.component.ts](https://github.com/vedph/cadmus-shell-v3/blob/master/src/app/app.component.ts)
- [app.component.html](https://github.com/vedph/cadmus-shell-v3/blob/master/src/app/app.component.html)
- [app.component.scss](https://github.com/vedph/cadmus-shell-v3/blob/master/src/app/app.component.scss)

## 8. Configure Routes

Setup your routes in [app.routes.ts](https://github.com/vedph/cadmus-shell-v3/blob/master/src/app/app.routes.ts). As you can see from that code, most routes target a single standalone component (via `loadComponent`), while some of them target a module (via `loadChildren`). Targeting modules happens when they work as the entry point for sub-routes, like e.g. for the general or philologic parts module.

Starting from that template, add all the routes you need, and remove those you don't need.

## 9. Configure App

Finally, add to [app.config.ts](https://github.com/vedph/cadmus-shell-v3/blob/master/src/app/app.config.ts) the required services. You can start from this template, and optionally add more providers if required.

## 10. Add Preview Styles

If using preview:

▶️ (1) add the corresponding styles in [preview-styles.css](https://github.com/vedph/cadmus-shell-v3/blob/master/src/preview-styles.css).
▶️ (2) import this file in your app's `styles.scss`:

```scss
@import "preview-styles.css";
```

## 11. Add Docker Support

You can add Docker support to create an image of your frontend app. Use as templates the files in the reference shell app:

- 📁 [Dockerfile](https://github.com/vedph/cadmus-shell-v3/blob/master/Dockerfile) using NGINX to serve the Angular app:

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
RUN rm /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html
COPY dist/cadmus-__PRJ__-app/browser/ .

EXPOSE 80
```

- 📁 [nginx.conf](https://github.com/vedph/cadmus-shell-v3/blob/master/nginx.conf): the NGINX configuration for serving the web app from the Docker container:

```conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile on;

  keepalive_timeout 65;

  server {
    listen 80;
    listen [::]:80;
    server_name localhost;

    gzip on;
    gzip_min_length 1000;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
      root /usr/share/nginx/html;
      index index.html index.htm;
      try_files $uri $uri/ /index.html;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
      root /usr/share/nginx/html;
    }
  }
}
```

- 📁 `docker-compose.yml`. The template here can be used for development; refer to [deployment](../../deploy/app#1-prepare-docker-compose-script) for more settings and details.

```yml
services:
  # MongoDB
  cadmus-__PRJ__-mongo:
    image: mongo
    container_name: cadmus-__PRJ__-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    networks:
      - cadmus-__PRJ__-network

 # PostgreSQL
  cadmus-__PRJ__-pgsql:
    image: postgres
    container_name: cadmus-__PRJ__-pgsql
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ API
  cadmus-__PRJ__-api:
    image: vedph2020/cadmus-__PRJ__-api:0.0.1
    container_name: cadmus-__PRJ__-api
    ports:
      # TODO: change 5080 with your API port in the host
      - 5080:8080
    depends_on:
      - cadmus-__PRJ__-mongo
      - cadmus-__PRJ__-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-__PRJ__-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-__PRJ__-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-__PRJ__-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-__PRJ__-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__DELAY=20
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ App
  cadmus-app:
    image: vedph2020/cadmus-__PRJ__-app:0.0.1
    container_name: cadmus-__PRJ__-app
    ports:
      - 4200:80
    depends_on:
      - cadmus-__PRJ__-api
    networks:
      - cadmus-__PRJ__-network

networks:
  cadmus-__PRJ__-network:
    driver: bridge
```

>⚠️ If using bibliography API, add its service to the compose stack too.

- 📁 `dockerignore`:

```txt
e2e
node_modules
src
```

## 12. Add Readme

Finally you can use a README template like this:

```md
- [models](https://github.com/vedph/cadmus-__PRJ__)
- [API](https://github.com/vedph/cadmus-__PRJ__-api)

## Docker

🐋 Quick Docker image build:

1. `npm run build-lib`
2. update version in `env.js` and `ng build`
3. `docker build . -t vedph2020/cadmus-__PRJ__-app:0.0.1 -t vedph2020/cadmus-__PRJ__-app:latest` (replace with the current version).
```
