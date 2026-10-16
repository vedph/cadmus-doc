---
title: "Setup CLI"
layout: default
parent: "Deployment"
nav_order: 5
---

# Setup CLI

If you need to setup the [CLI tool](../tools/index.md) in the server:

▶️ (1) open the Cadmus CLI tool backend repository at <https://github.com/vedph/cadmus-core> and find the URL of the latest release version under `Releases`.

▶️ (2) download it and unzip in some folder, e.g.:

```sh
# download
wget https://github.com/vedph/cadmus-core/releases/download/v.16.0.10/v.16.0.10-linux-x64.tar.gz

# unzip
tar -xzf v.16.0.10-linux-x64.tar.gz
rm v.16.0.10-linux-x64.tar.gz

# move to unversioned folder
mv v.16.0.10-linux-x64/* cadmus-tool/
rm v.16.0.10-linux-x64

# ensure the tool is executable
chmod +x cadmus-tool/cadmus-tool
```

You can now enter the folder and type `./cadmus-tool` to get an overview of available commands.
