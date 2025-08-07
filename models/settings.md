---
title: "Models - Settings" 
layout: default
parent: Models
nav_order: 5
---

# Editor Settings

Backend data can also include editor-specific settings to customize the behavior of specific components.

Editor settings are defined in the backend **JSON profile** under the root's `settings` property, which is a single object where each property is an object setting: the property name is the setting key, and the property value is an object representing settings with any type of structure.

By convention, each setting refers to an editor, and its **ID** is the editor's type ID, optionally followed by its role ID prefixed by an underscore.

For instance, categories editor's settings are under `it.vedph.categories`, and any role-specific settings for it are under `it.vedph.categories_role`.

>In MongoDB, each setting is stored as a document in the settings collection, with an ID equal to this identifier.

This allows adding specific settings for configurable editors in the UI. Before this feature was introduced, this was possible via thesauri used for this purpose; but this forced settings to be structured as flat string entries, which is not flexible except for simple cases. Now, each editor type (also distinguishing its roles, if required) can have its own settings, and these can be structured as needed in a freely modeled object.

⚙️ To use this feature in your frontend UI part or fragment editor:

1. inject the `AppRepository` service.
2. request the setting object for the editor of the part/fragment type ID and role via `getSettingFor(typeId, roleId?)`. This will return an object with any model, representing all the settings for that specific editor. The repository will cache these settings for future requests, thus avoiding further trips to the server.
