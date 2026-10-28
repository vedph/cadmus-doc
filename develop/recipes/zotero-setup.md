---
title: "Setup Zotero"
parent: "Recipes"
layout: default
nav_order: 2
---

# Setup Zotero

Setting up the editor to use Zotero concerns the frontend only:

▶️ (1) install the Zotero lookup package: `pnpm i @myrmidon/cadmus-refs-zotero-lookup`.

▶️ (2) in `env.js`, add the Zotero endpoint and account:

```js
(function (window) {
  window.__env = window.__env || {};
  // ...
  // Zotero
  window.__env.zoteroApiKey = "TODO:YOUR_ZOTERO_KEY";
  window.__env.zoteroUserId = "TODO:YOUR_ZOTERO_USER_ID";
  window.__env.zoteroLibraryId = "TODO:YOUR_ZOTERO_LIBRARY_ID";
})(this);
```

> 💡 To avoid storing real credentials in code (and thus in a potentially public repository) but still test Zotero during development you might want to leave these credentials empty and override them adding a separate `env.local.js` file with the real credentials only, e.g.:

```js
// development overrides for env.js
// DO NOT COMMIT THIS FILE TO THE REPOSITORY

// Zotero
window.__env.zoteroApiKey = "...";
window.__env.zoteroUserId = "...";
window.__env.zoteroLibraryId = "...";
```

To use this, add this script to your app's `index.html` after `<script src="env.js"></script>`:

```html
<script>
  // check if a local environment file exists and load it
  function loadLocalEnv() {
    var script = document.createElement("script");
    script.onload = function () {
      console.log("Loaded local environment overrides.");
    };
    script.onerror = function () {
      console.warn(
        "Local environment file not found, using default values."
      );
    };
    script.src = "env.local.js";
    document.head.appendChild(script);
  }
  loadLocalEnv();
</script>
```

⚠️ Remember to add `env.local.js` to your `.gitignore` file!

▶️ (3) in `app.config.ts` add the Zotero provider:

```ts
import {
  ZOTERO_API_KEY_TOKEN,
  ZOTERO_LIBRARY_ID_TOKEN,
  ZOTERO_USER_ID_TOKEN,
} from '@myrmidon/cadmus-refs-zotero-lookup';

// ...

// add to providers array:
[
    // Zotero
    {
      provide: ZOTERO_API_KEY_TOKEN,
      useFactory: (env: EnvService) => env.get('zoteroApiKey'),
      deps: [EnvService],
    },
    {
      provide: ZOTERO_USER_ID_TOKEN,
      useFactory: (env: EnvService) => env.get('zoteroUserId'),
      deps: [EnvService],
    },
    {
      provide: ZOTERO_LIBRARY_ID_TOKEN,
      useFactory: (env: EnvService) => env.get('zoteroLibraryId'),
      deps: [EnvService],
    },
]
```

▶️ (4) in `app.ts` configure the lookup service:

```ts
import { ZoteroRefLookupService } from '@myrmidon/cadmus-refs-zotero-lookup';

// ...

constructor() {
	const storage = inject(RamStorageService);
    storage.store(LOOKUP_CONFIGS_KEY, [
      // Zotero
      {
        name: 'Zotero',
        iconUrl: '/img/zotero128.png',
        description: 'Zotero bibliography',
        label: 'ID',
        service: inject(ZoteroRefLookupService),
        itemIdGetter: (item: any) =>
          item ? `${item.library?.id}/${item.key}` : '',
        itemLabelGetter: (item: any) => {
          if (!item) {
            return '';
          }
          const sb: string[] = [];
          if (item.data?.creators && Array.isArray(item.data.creators)) {
            const creators = item.data.creators;
            for (let i = 0; i < creators.length; i++) {
              const c = creators[i];
              if (i > 0) {
                sb.push('; ');
              }
              if (c.lastName) {
                sb.push(c.lastName);
              }
              if (c.firstName) {
                sb.push(' ' + c.firstName.charAt(0) + '.');
              }
            }
          }
          sb.push(': ');
          if (item.title) {
            sb.push(item.title);
          } else if (item.data?.title) {
            sb.push(item.data?.title);
          }
          return sb.join('');
        },
      },
  // ...
  ]);
}
```

▶️ (5) add `zotero128.png` to your `public/img` folder.
