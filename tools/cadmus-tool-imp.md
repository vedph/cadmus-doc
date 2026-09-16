---
title: "Import" 
layout: default
parent: "Cadmus Tool"
nav_order: 4
---

# Import Commands

## Graph Import Command

🎯 Import preset nodes, triples, node mappings, or thesauri class nodes into graph (the JSON document references must be [dereferenced](cadmus-tool-exp.md#graph-dereference-mappings) first!).

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

## Thesaurus Import Command

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

### File Format

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

## TaxoStore Import Command

🎯 Import a taxonomies store into its database, creating it if not existing.

```sh
taxo-tool import-store <TREES_CSV> <NODES_CSV> -d <DATABASE_NAME> [-c <CONNECTION_TEMPLATE>]
```

- `TREES_CSV`, `NODES_CSV`: paths to the seed CSV files.
- `-d`/`--database`: name of the database to create and seed.
- `-c`/`--connection`: optional PostgreSQL connection string template, with `{0}` as a placeholder for the database name (e.g. `Server=localhost;Database={0};User Id=postgres;Password=postgres`). When omitted, it is read from the `TaxoStore` connection string in the tool's own configuration (`appsettings.json`/`appsettings.local.json`/environment variables, resolved next to the tool's executable).

Example `appsettings.json` next to `taxo-tool.exe`:

```json
{
  "ConnectionStrings": {
    "TaxoStore": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True"
  }
}
```

Example usage, creating and seeding database `taxo`:

```sh
taxo-tool import-store wwwroot/taxo/trees.csv wwwroot/taxo/nodes.csv -d taxo
```

**CSV File Formats:**

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
