---
title: "Database" 
layout: default
parent: "Cadmus Tool"
nav_order: 1
---

# Database Commands

## Create Database Command

🎯 Create an index or graph database with its own schema.

```sh
./cadmus-tool create-db index|graph DATABASE_NAME
```

👉 Samples:

```sh
./cadmus-tool create-db index cadmus-itinera
./cadmus-tool create-db graph cadmus-itinera-graph
```

## Index Database Command

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

## Seed Database Command

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

## Seed Users Command

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

## Graph One Command

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

## Graph Many Command

🎯 Map all the items into the target graph. This is used to rebuild the graph database from Cadmus data.

```sh
./cadmus-tool graph-many DATABASE_NAME [-g REPOSITORY_PLUGIN_PATH]
```

- `-g`: the target repository provider plugin tag (e.g. `repository-provider.itinera`).

👉 Sample:

```sh
./cadmus-tool graph-many cadmus-itinera repository-provider.itinera
```

## Update Graph Classes Command

🎯 Update the index of nodes classes in the index database. This is a potentially long task, depending on the number of nodes and the depth of class hierarchies.

```sh
./cadmus-tool graph-cls DATABASE_NAME PROFILE_PATH
```

👉 Sample:

```sh
./cadmus-tool graph-cls cadmus-itinera ./plugins/Cadmus.Itinera.Services/seed-profile.json
```

## Build SQL Command

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

## Run Mongo Command

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

