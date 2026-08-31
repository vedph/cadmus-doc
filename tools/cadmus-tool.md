---
title: "Cadmus Tool" 
layout: default
nav_order: 8
---

# Cadmus Tool

The main Cadmus configuration and utility command-line tool is `cadmus-tool`.

- [code repository](https://github.com/vedph/cadmus_tool)
- [database commands](cadmus-tool-db)
- [user accounts commands](cadmus-tool-acc)
- [export commands](cadmus-tool-exp)
- [import commands](cadmus-tool-imp)

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

### Plugin Setup Example

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
