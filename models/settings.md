---
title: "Models - Settings"
layout: default
parent: Models
nav_order: 6
---

# Editor Settings

Backend data can also include editor-specific settings to customize the behavior of specific components.

Editor settings are defined in the backend **JSON profile** under the root's `settings` property, which is a single object where each property is an object setting: the property name is the setting key, and the property value is an object representing settings with any type of structure.

By convention, each setting refers to an editor, and its **ID** is the editor's type ID, optionally followed by its role ID prefixed by an underscore. For instance, categories editor's settings are under `it.vedph.categories`, and any role-specific settings for it are under `it.vedph.categories_role`. Example:

```json
"settings": {
  "it.vedph.chronotopes": {
    "placeLookupServiceId": "biblissima",
    "lookupProviderOptions": {
      "biblissima": {
        "q26719": {
          "label": "place",
          "options": {
            "type": "Q26719"
          }
        }
      }
    }
  },
  "it.vedph.codicology.shelfmarks": {
    "cityFromLibPattern": "\\((.+)\\)$"
  },
  "it.vedph.pin-links": {
    "lookupProviderOptions": {
      "biblissima": {
        "q168": {
          "label": "people",
          "options": {
            "type": "Q168"
          }
        },
        "q282950": {
          "label": "works",
          "options": { "type": "Q282950" }
        }
      }
    }
  }
}
```

Here we define 3 settings:

- `it.vedph.chronotopes`: settings for a Biblissima+ lookup service to be limited to places for the [chronotopes part](../models/shared.md#general) (which lists places and/or dates, so that limiting to places avoids fetching irrelevant entries).
- `it.vedph.codicology.shelfmarks`: settings for the [codicological shelfmarks part](../models/shared.md#codicology), instructing the part to deduct city names from libraries using the specified regular expression pattern (e.g. cities are extracted from the trailing text in brackets).
- `it.vedph.pin-links`: settings for the [links part](../models/shared.md#general), to configure a Bilissima+ lookup service to fetch either people or works, according to what the user will pick in the corresponding UI.

> In MongoDB, each setting is stored as a document in the settings collection, with an ID equal to this identifier.

This allows adding specific settings for configurable editors in the UI. Before this feature was introduced, this was possible via thesauri used for this purpose; but this forced settings to be structured as flat string entries, which is not flexible except for simple cases. Now, each editor type (also distinguishing its roles, if required) can have its own settings, and these can be structured as needed in a freely modeled object.

⚙️ To use this feature in your frontend UI part or fragment editor:

1. inject the `AppRepository` service.
2. request the setting object for the editor of the part/fragment type ID and role via `getSettingFor<T>(typeId, roleId?, reload?)`. This will return an object with any model, representing all the settings for that specific editor. The repository will cache these settings for future requests, thus avoiding further trips to the server.
