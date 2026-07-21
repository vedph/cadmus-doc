---
title: "Cadmus Tool" 
layout: default
nav_order: 8
---

# Cadmus Tool

- [Cadmus Tool](#cadmus-tool)
  - [Commands](#commands)
    - [Database](#database)
      - [Create Database Command](#create-database-command)
      - [Index Database Command](#index-database-command)
      - [Seed Database Command](#seed-database-command)
      - [Seed Users Command](#seed-users-command)
      - [Graph One Command](#graph-one-command)
      - [Graph Many Command](#graph-many-command)
      - [Update Graph Classes Command](#update-graph-classes-command)
      - [Build SQL Command](#build-sql-command)
      - [Run Mongo Command](#run-mongo-command)
    - [Accounts](#accounts)
      - [Add User Command](#add-user-command)
      - [Add User Roles Command](#add-user-roles-command)
      - [Delete User Command](#delete-user-command)
      - [Delete User Roles Command](#delete-user-roles-command)
      - [List Users Command](#list-users-command)
      - [Update User Command](#update-user-command)
    - [Export](#export)
      - [Get Object Command](#get-object-command)
      - [Graph Dereference Mappings](#graph-dereference-mappings)
    - [Import](#import)
      - [Graph Import Command](#graph-import-command)
      - [Thesaurus Import Command](#thesaurus-import-command)
        - [File Format](#file-format)
  - [Plugin Architecture](#plugin-architecture)
    - [Setup](#setup)

The main Cadmus configuration and utility command-line tool is `cadmus-tool`.

## Commands

### Database

#### Create Database Command

🎯 Create an index or graph database with its own schema.

```sh
./cadmus-tool create-db index|graph DATABASE_NAME
```

👉 Samples:

```sh
./cadmus-tool create-db index cadmus-itinera
./cadmus-tool create-db graph cadmus-itinera-graph
```

#### Index Database Command

🎯 Index the specified Cadmus database. If the index database does not exist, it will be created; if it exists, it will be cleared if requested.

This requires a plugin with providers for the repository factory and the parts seeders factory. Each project has its own plugin, which must be placed in a subfolder of the tool's `plugins` folder.

```sh
./cadmus-tool index DATABASE_NAME JSON_PROFILE_PATH [-g REPOSITORY_PLUGIN_TAG] [-c]
```

- `-g`: the target repository provider plugin tag (e.g. `repository-provider.itinera`).
- `-c`=clear the target database when it exists.

👉 Sample:

```sh
./cadmus-tool index cadmus-itinera ./plugins/Cadmus.Itinera.Services/seed-profile.json -g repository-provider.itinera
```

#### Seed Database Command

🎯 Create a new Cadmus MongoDB database (if the specified database does not already exists), and seed it with a specified number of random items.

```sh
./cadmus-tool seed DATABASE_NAME JSON_PROFILE_PATH [-g REPOSITORY_PLUGIN_PATH] [-s SEEDERS_FACTORY_PLUGIN_TAG] [-c COUNT] [-d] [-h]
```

- `-c N`: the number of items to seed. Default is 100.
- `-d`: dry run, i.e. create the items and parts, but do not create the database nor store anything into it. This is used to test for seeder issues before actually running the command.
- `-h`: add history items and parts together with the seeded items and parts. Default is `false`. In a real-world database you should set this to `true`.

👉 Sample:

```sh
./cadmus-tool seed cadmus-itinera ./plugins/Cadmus.Itinera.Services/seed-profile.json -g repository-provider.itinera -s seeder-factory-provider.itinera -c 10 -h
```

#### Seed Users Command

🎯 Seed user accounts into a Cadmus auth database from a JSON file.

```sh
./cadmus-tool seed-users JSON_FILE_PATH DATABASE_NAME [-d]
```

- `-d`: dry run. This is used to test for seeder issues before actually running the command.

👉 Sample:

```sh
./cadmus-tool seed-users c:/users/dfusi/desktop/users.json cadmus-gve-auth
```

The seed file is like this:

```json
[
  {
    "UserName": "alpha",
    "Password": "ThePasswordHere",
    "Email": "alpha@somewhere.com",
    "Roles": ["admin", "editor", "operator", "visitor"],
    "FirstName": "Andrew",
    "LastName": "Alpha"
  },
  {
    "UserName": "beta",
    "Password": "ThePasswordHere",
    "Email": "beta@somewhereelse.com",
    "Roles": ["editor", "operator", "visitor"],
    "FirstName": "Betty",
    "LastName": "Beta"
  }
]
```

#### Graph One Command

🎯 Map a single item/part into the graph database.

```sh
./cadmus-tool graph-one DATABASE_NAME ID [-g REPOSITORY_PLUGIN_PATH] [-p] [-d]
```

- `-g`: the target repository provider plugin tag (e.g. `repository-provider.itinera`).
- `-p`: the ID argument refers to a part rather than to an item.
- `-d`: the ID argument refers to an item/part which was deleted.
- `-x`: explain the update without actually performing it.

👉 Sample:

```sh
./cadmus-tool graph-one cadmus-itinera 4a0ce97e-84d1-417d-9fb0-a91d9dfc4da7 -g repository-provider.itinera -px
```

#### Graph Many Command

🎯 Map all the items into the target graph. This is used to rebuild the graph database from Cadmus data.

```sh
./cadmus-tool graph-many DATABASE_NAME [-g REPOSITORY_PLUGIN_PATH]
```

- `-g`: the target repository provider plugin tag (e.g. `repository-provider.itinera`).

👉 Sample:

```sh
./cadmus-tool graph-many cadmus-itinera repository-provider.itinera
```

#### Update Graph Classes Command

🎯 Update the index of nodes classes in the index database. This is a potentially long task, depending on the number of nodes and the depth of class hierarchies.

```sh
./cadmus-tool graph-cls DATABASE_NAME PROFILE_PATH
```

👉 Sample:

```sh
./cadmus-tool graph-cls cadmus-itinera ./plugins/Cadmus.Itinera.Services/seed-profile.json
```

#### Build SQL Command

🎯 Build SQL code for querying the Cadmus index database, just once or interactively.

```sh
./cadmus-tool build-sql [-q QUERY] [-l]
```

- `-q`: the query (for non-interactive mode).
- `-l`: use legacy syntax for the query. Default is `false`.

👉 Samples:

```sh
./cadmus-tool build-sql
./cadmus-tool build-sql -q "[dsc*=even]"
```

#### Run Mongo Command

🎯 Run Mongo a script targeting the specified database.

⚠️ This is an obsolete feature, preserved for scenarios where access to the MongoDB shell is not available. Note that not all the MongoDB commands are supported.

```sh
./cadmus-tool run-mongo DATABASE_NAME [-s SCRIPT] [-f SCRIPT_FILE_PATH]
```

- `-s`: the script to run.
- `-f`: the file with the script to run.

👉 Sample:

```sh
./cadmus-tool run-mongo cadmus-vela -s "db.getCollection(flags).updateOne({ _id: 4 },{ $set:{isAdmin: true }})"
```

Supported commands:

1. Collection Operations:
   - data manipulation: `find`, `findOne`, `insertOne`, `insertMany`, `updateOne`, `updateMany`, `replaceOne`, `deleteOne`, `deleteMany`.
   - collection administration: `drop`, `createIndex`, `dropIndex`, `countDocuments`.
2. Database Operations:
   - administration: `createCollection`, `dropDatabase`, `getCollectionNames`, `listCollections`.
   - information: `stats`, `runCommand`.
3. Connection Operations:
   - `use [database]` - switches to the specified database.
4. Collection Access Methods:
   - `db.collection` - directly access a collection with dot notation.
   - `db.getCollection("name")` - access a collection by name.

### Accounts

#### Add User Command

🎯 Add a user account.

```sh
./cadmus-tool add-user NAME PASSWORD EMAIL FIRST_NAME LAST_NAME
```

#### Add User Roles Command

🎯 Add role(s) to a user account.

```sh
./cadmus-tool add-user-roles NAME ROLE1 ROLEn
```

#### Delete User Command

🎯 Delete a user account.

```sh
./cadmus-tool delete-user USER_NAME [-y]
```

#### Delete User Roles Command

🎯 Delete role(s) from a user account.

```sh
./cadmus-tool delete-user-roles
```

#### List Users Command

🎯 List user accounts.

```sh
./cadmus-tool list-users
```

#### Update User Command

🎯 Update a user account.

```sh
./cadmus-tool update-user
```

### Export

#### Get Object Command

🎯 Get the JSON code representing an item or a part's content, optionally converted in XML.

```sh
./cadmus-tool get-obj DATABASE_NAME ID OUTPUT_DIR [-g REPOSITORY_PLUGIN_TAG] [-p] [-x]
```

- `p`: the ID refers to a part rather than to an item.
- `x`: also write an XML version of the result.

👉 Sample:

```sh
./cadmus-tool get-obj cadmus 8e5d5b5d-4b27-4d00-9038-f611a8e199b9 c:/users/dfusi/desktop -g repository-provider.itinera -p -x
```

#### Graph Dereference Mappings

🎯 Dereference mappings in a JSON mappings file by outputting a fully dereferenced list of mappings into another file. This can then be imported via [graph-import](#graph-import-command).

```sh
./cadmus-tool graph-deref INPUT_PATH OUTPUT_PATH
```

💡 A JSON mappings document has an object at its root with two main sections:

- `namedMappings`, an optional object where each property is a named mapping template. This is used to avoid repeating the same mapping node in each mapping tree where it is used.
- `documentMappings`: an array of mapping objects. Each is the root of a tree of mappings. Among the mapping's `children` property, you can reference a named mapping by just adding its name. For instance, here the first 5 children are just references, expected to be found in `namedMappings`:

```json
{
  "name": "person_birth_event",
  "sourceType": 2,
  "facetFilter": "person",
  "partTypeFilter": "it.vedph.historical-events",
  "description": "Map person birth event",
  "source": "events[?type=='person.birth']",
  "sid": "{$part-id}/{@eid}",
  "output": {
    "metadata": {
      "sid": "{$part-id}/{@eid}",
      "person": "x:persons/{$metadata-pid}/{$item-eid}"
    },
    "nodes": {
      "event": "x:events/{$sid} [x:events/{@eid}]"
    },
    "triples": [
      "{?event} a crm:E67_birth",
      "{?event} crm:P2_has_type x:event-types/person.birth",
      "{?event} crm:P98_brought_into_life {$person}"
    ]
  },
  "children": [
    {
      "name": "event_description"
    },
    {
      "name": "event_note"
    },
    {
      "name": "event_chronotopes"
    },
    {
      "name": "event_assertion"
    },
    {
      "name": "event_tag"
    },
    {
      "name": "person_birth_event/related/by_mother",
      "source": "relatedEntities[?relation=='mother']",
      "output": {
        "nodes": {
          "mother": "{@id.target.gid}"
        },
        "triples": ["{?event} crm:P96_by_mother {?mother}"]
      }
    },
    {
      "name": "person_birth_event/related/from_father",
      "source": "relatedEntities[?relation=='father']",
      "output": {
        "nodes": {
          "father": "{@id.target.gid}"
        },
        "triples": ["{?event} crm:P97_from_father {?father}"]
      }
    }
  ]
}
```

👉 Sample:

```sh
./cadmus-tool graph-deref c:/users/dfusi/desktop/mappings.json c:/users/dfusi/desktop/mappings-d.json
```

### Import

#### Graph Import Command

🎯 Import preset nodes, triples, node mappings, or thesauri class nodes into graph (the JSON document references must be [dereferenced](#graph-dereference-mappings) first!).

```sh
./cadmus-tool graph-import SOURCE_PATH DATABASE_NAME [-g REPOSITORY_PLUGIN_PATH] [-m IMPORT_MODE] [-d] [-r] [-p THESAURUS_ID_PREFIX]
```

- `-m`: import mode: `n`odes (default), `t`riples, `m`appings, t`h`esauri. Mappings are imported by their _name_, so if you import a mapping with a name equal to one already present in the database, the old one will be updated.
- `-r`: when importing thesauri, make the thesaurus' ID the root class node.
- `-p <ThesaurusIdPrefix>`: when importing thesauri, set the prefix to be added to each class node.
- `-d`: dry mode - don't write to database.

👉 Sample:

```sh
./cadmus-tool graph-import c:/users/dfusi/desktop/nodes.json cadmus-itinera -g repository-provider.itinera
```

>Note: if you are importing mappings, ensure that the JSON document has a root array property including mappings. When working with a compact mappings document using references, dereference all the referenced mappings via the [apposite command](#graph-dereference-mappings) before importing.

All data files are JSON documents, having as their root element an **array** of objects. For instance:

- **node** (omit all the properties you don't need):

```json
[
  {
    "uri": "x:alpha",
    "isClass": true,
    "tag": null,
    "label": "Alpha"
  }
]
```

- **triple** with non-literal object:

```json
[
  {
    "subjectUri": "x:beta",
    "predicateUri": "rdfs:subClassOf",
    "objectUri": "x:alpha",
    "tag": null
  }
]
```

- **triple** with literal object:

```json
[
  {
    "subjectUri": "x:alpha",
    "predicateUri": "rdf:label",
    "objectLiteral": "Alpha",
    "objectLiteralIx": "alpha",
    "literalType": "xs:string",
    "literalLanguage": "en",
    "literalNumber": null,
    "tag": null
  }
]
```

- **thesaurus**:

```json
[
  {
    "id": "languages@en",
    "entries": [
      {
        "id": "eng",
        "value": "English"
      },
      {
        "id": "fre",
        "value": "French"
      }
    ]
  }
]
```

#### Thesaurus Import Command

🎯 Import one or more thesauri from one or more file(s) into a Cadmus database. Files can be JSON, CSV, XLS, XLSX and are selected according to their extension. Any unknown extension is treated as a JSON source.

```sh
./cadmus-tool thes-import INPUT_FILE_MASK DATABASE_NAME [-m <R|P|S>] [-d]
```

- `-m`: the import mode, specifying how to deal when importing onto existing thesauri:
  - `R` = replace (default): if the imported thesaurus already exists, it is fully replaced by the new one.
  - `P` = patch: the existing thesaurus is patched with the imported one: any existing entry has its value overwritten; any non existing entry is just added.
  - `S` = synch: the existing thesaurus is synched with the imported one: this is equal to patch, with the addition that any existing entry not found in the imported thesaurus is removed.
- `-d`: dry run (don't write to database).
- `-s`: for Excel sources, the ordinal number of the sheet to read data from (1-N; default=1).
- `-r`: for Excel sources, the ordinal number of the first row to read data from (1-N; default=1).
- `-c`: for Excel sources, the ordinal number of the first column to read data from (1-N; default=1).

👉 Sample:

```sh
./cadmus-tool thes-import c:/users/dfusi/desktop/thesauri/*.json cadmus-itinera -d
```

##### File Format

- **JSON**: a single thesaurus as an _object_, or a list of thesauri as an _array of objects_. Each object is encoded like in this sample:

```json
{
  "id": "colors@en",
  "entries": [
    {
      "id": "r",
      "value": "red"
    },
    {
      "id": "g",
      "value": "green"
    },
    {
      "id": "b",
      "value": "blue"
    },
  ]
}
```

An alias thesaurus is encoded like:

```json
{
  "id": "colours@en",
  "targetId": "colors"
}
```

- **CSV**: a comma-delimited UTF8 text file, like in this sample:

```csv
thesaurusId,id,value,targetId
colors@en,r,red,
colors@en,g,green,
colors@en,b,blue,
shapes@en,trg,triangle,
shapes@en,rct,rectangle,
```

You can omit the thesaurus ID if equal to the previous row, e.g.:

```csv
thesaurusId,id,value,targetId
colors@en,r,red,
,g,green,
,b,blue,
shapes@en,trg,triangle,
,rct,rectangle,
```

You must include the header row as the first row of the file. This allows changing the column order at will, as they will be identified by their name.

- **Excel**: XLSX or XLS files. It is assumed that your columns are in this order:

1. thesaurus
2. id
3. value
4. target

You can add a header row or not, and use whatever name you want, as columns get identified by their order. You can anyway specify the sheet number, the first row number, and the first column number.

## Plugin Architecture

Since version 2, this tool requires plugin providers under its `plugins` folder. The plugin architecture makes the tool independent from Cadmus projects, each having its own models. Otherwise, every Cadmus project should be included as a dependency in the CLI tool, defeating the purpose of a generic and universal tool.

Plugins are used to get Cadmus factory providers. A Cadmus factory provider plugin acts as a hub entry point for all the components to be packed in the CLI tool for a specific project.

>You can build your plugin with all its dependencies by publishing the library you wish to use as the import target. For instance, if you are going to use library `Cadmus.Itinera.Services` as a plugin, publish it and then copy published files into the corresponding plugin folder.

The tool is like an empty shell, where project-dependent components are demanded to plugins under its `plugins` folder. The commands requiring plugins are those used to build a full Cadmus index from its Mongo database, or to seed a Mongo Cadmus database with mock data. To this end, the CLI tool requires two factory objects: one for the repository, acting as the hub for all its parts and fragments; and another for the part and fragment seeders.

These providers glue together the composable Cadmus parts, and as such are surface components laid on top of each Cadmus solution, just like services in the web APIs. Usually they are located in the `Cadmus.PRJ.Services` (where `PRJ` is your project name) library of your project.Plugins are an easy solution for the CLI tool because runtime binding via reflection there is a viable option, which instead is not the case for the API backend (which gets packed into a different Docker image for each solution).

To add a plugin:

1. create a subfolder of this folder, named after the DLL plugin filename (usually `Cadmus.PRJ.Services`, where `PRJ` is your project name). For instance, the plugin `Cadmus.Tgr.Services.dll` should be placed in a subfolder of this folder named `Cadmus.Tgr.Services`.
2. copy the plugin files including all its dependencies in this folder.
3. it is also useful to copy the project configuration file (`seed-profile.json`) in this folder, so you can have it at hand when required.

### Setup

In this sample I setup the tool with a plugin in an Ubuntu server.

(1) download the tool (change the version to the latest one):

```sh
wget https://github.com/vedph/cadmus_tool/releases/download/v.10.0.3/App-v.10.0.3-linux-x64.tar.gz
```

(2) unzip it and remove the archive:

```sh
tar -xf App-v.10.0.3-linux-x64.tar.gz
rm App-v.10.0.3-linux-x64.tar.gz
```

(3) rename the folder and grant permissions to the tool:

```sh
mv App-v.10.0.3-linux-x64 cadmus-tool
cd cadmus-tool
chmod +x cadmus-tool
```

(4) get the plugin and unzip it:

```sh
cd plugins
wget http://www.fusisoft.it/xfer/cadmus/cli/plugins/Cadmus.Itinera.Services.zip
unzip Cadmus.Itinera.Services.zip
rm Cadmus.Itinera.Services.zip
```

To run the tool, enter its folder and run:

```sh
./cadmus-tool
```
