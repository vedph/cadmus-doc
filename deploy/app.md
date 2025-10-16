---
title: "Deployment - App Setup"
layout: default
parent: "Deployment"
nav_order: 2
---

# App Setup

- [App Setup](#app-setup)
  - [1. Prepare Docker Compose Script](#1-prepare-docker-compose-script)
  - [2. Configure Data Persistence](#2-configure-data-persistence)
    - [Named Volumes and Bind Mounts](#named-volumes-and-bind-mounts)
  - [3. Customize API Settings](#3-customize-api-settings)
    - [Settings and Environment Variables](#settings-and-environment-variables)
    - [Overriding Settings](#overriding-settings)
    - [AllowedOrigins](#allowedorigins)
    - [JWT Key](#jwt-key)
    - [StockUsers](#stockusers)
    - [Rate Limiting](#rate-limiting)
    - [Server](#server)
    - [Auditing and Privacy](#auditing-and-privacy)
    - [Database](#database)
      - [Securing Databases](#securing-databases)
      - [External Databases](#external-databases)
    - [Other Services](#other-services)
  - [4. Configure Frontend App](#4-configure-frontend-app)

Essentially, to host Cadmus app on a server you should just customize the default _Docker compose script_, which is designed for hosting the system on a local machine. This usually resolves to changing some URIs, and replacing mock security data.

⚠️ In what follows, `__PRJ__` is the placeholder for your Cadmus project name.

## 1. Prepare Docker Compose Script

▶️ 1. create a folder for your Cadmus project in the host, e.g. `mkdir dockers/cadmus-__PRJ__-app`.

▶️ 2. place there a `docker-compose.yml` file starting with this template:

```yml
services:
  # MongoDB
  cadmus-__PRJ__-mongo:
    image: mongo
    container_name: cadmus-__PRJ__-mongo
    restart: unless-stopped
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
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-__PRJ__-network

  # Biblio API
  # TODO: remove this service if not required
  cadmus-biblio-api:
    image: vedph2020/cadmus-biblio-api:7.0.1
    container_name: cadmus-biblio-api
    restart: unless-stopped
    ports:
      - 60058:8080
    depends_on:
      - cadmus__PRJ__mongo
      - cadmus__PRJ__pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus__PRJ__mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus__PRJ__pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - CONNECTIONSTRINGS__BIBLIO=Server=cadmus__PRJ__pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SEED__DELAY=50
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus__PRJ__mongo:27017/{0}-log
      # TODO: change the default user password (same as API below)
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ API
  cadmus-__PRJ__-api:
    image: vedph2020/cadmus-__PRJ__-api:0.0.1
    container_name: cadmus-__PRJ__-api
    restart: unless-stopped
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
      # TODO: change the default user password
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__DELAY=20
      # TODO: if email is enabled, set the URLs and email as desired
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ App
  cadmus-__PRJ__-app:
    image: vedph2020/cadmus-__PRJ__-app:0.0.1
    container_name: cadmus-__PRJ__-app
    restart: unless-stopped
    ports:
      - 4200:80
    depends_on:
      - cadmus-__PRJ__-api
    volumes:
      # TODO: bind host env.js to container to configure base API URL
      - /opt/dockers/cadmus-__PRJ__/env.js:/usr/share/nginx/html/env.js
    networks:
      - cadmus-__PRJ__-network

networks:
  cadmus-__PRJ__-network:
    driver: bridge
```

💡 This template includes 4 services:

- `cadmus-PRJ-mongo`: MongoDB service.
- `cadmus-PRJ-pgsql`: PostgreSQL service used for indexes and semantic graphs (and optionally for logging).
- `cadmus-PRJ-api`: API backend service. This is exposed at some port in `localhost`. This is ASP.NET 8 served by [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel) at port 8080.
- `cadmus-PRJ-app`: web app service, exposed at port 4200 in `localhost`. This is Angular served by [NGINX](https://www.nginx.com/).

Additionally, if your project uses external bibliography you will have another service for the bibliographic API. Of course, you are free to add as many other services as your app requires, but in most cases these are all what you need.

💡 Should you need an admin UI frontend for MongoDB, you could add other services e.g. from the [mongo-express Docker image](https://hub.docker.com/_/mongo-express):

```yml
  mongo-express:
    image: mongo-express
    container_name: mongo-express
    restart: unless-stopped
    ports:
      - 8082:8081
    environment:
      - ME_CONFIG_MONGODB_SERVER=cadmus-__PRJ__-mongo
      - ME_CONFIG_MONGODB_PORT=27017
      - ME_CONFIG_BASICAUTH=false
    depends_on:
      - cadmus-__PRJ__-mongo
    networks:
      - cadmus-__PRJ__-network
```

## 2. Configure Data Persistence

▶️ Once you are sure that everything works as intended, you should configure the script to provide data persistence. Otherwise, data will be destroyed when you destroy the corresponding database service containers.

Change the above template to include a volume for each database service, defining the volumes in an additional section:

```yml
services:
  # MongoDB
  cadmus-mongo:
    # ...
    volumes:
      - mongo-vol:/data/db
  # PostgreSQL
  cadmus-pgsql:
    # ...
    volumes:
      - pgsql-vol:/var/lib/postgresql/data

# ...
volumes:
  mongo-vol:
  pgsql-vol:
```

💡 The `volumes:` section in a `docker-compose.yml` file is used to define named volumes. _Named volumes_ are a type of Docker volume that can persist data between container lifecycles, and can be more easily referenced and managed. When you specify a volume in this section, Docker Compose will ensure that this volume is created if it does not already exist.

>In this example, `mongo-vol` and `pgsql-vol` are the named volumes on the host machine, while `/data/db` and `/var/lib/postgresql/data` are the directories inside the respective containers where volumes are mounted.

This tells Docker to mount the named volumes at the specified paths within the containers. Any data that the MongoDB or PostgreSQL services write to these paths will be stored in the named volumes, and will persist even if the containers are stopped or deleted (unless you destroy also the volumes like in `docker compose down -v`).

### Named Volumes and Bind Mounts

>💡 This is a note for newcomers. Experienced Docker compose users can safely skip it.

For databases, named volumes are the default choice rather than bind mounts. A _named volume_ is created and managed by Docker, while a _bind mount_ is a file or directory on the host machine that is mounted into a container.

Named volumes are created using the `docker volume create` command, or by specifying them in the `volumes:` section of a `docker-compose.yml` file. They are stored in the Docker host's filesystem, typically under `/var/lib/docker/volumes`, and can be more easily referenced and managed using the `docker volume` command.

On the other hand, bind mounts are created by specifying the path to a file or directory on the host machine when starting a container, using the `-v` or `--mount` flag, or by using another syntax in the `docker-compose.yml` file, like:

```yml
services:
  cadmus-itinera-app:
    # ...
    volumes:
      - /opt/cadmus/web/env.js:/usr/share/nginx/html/env.js
```

In this case we have a single JavaScript file (`env.js`) which contains environment-dependent values defined. A patched copy of this file has been placed in the host's `/opt/cadmus/web` directory, and is directly bound to the corresponding file in the NGINX container, under `/usr/share/nginx/html/`.

Bind mounts rely on the host machine's filesystem having a specific directory structure available, and give the container access to files and directories on the host machine. So, one key difference between named volumes and bind mounts is that named volumes are completely managed by Docker, while bind mounts depend on the directory structure and operating system of the host machine. This means that named volumes are generally easier to back up, migrate, and manage than bind mounts.

## 3. Customize API Settings

In this section you will find information about all the relevant API settings. At a minimum, you must configure API-related services as follows:

▶️ 1. setup CORS for your own URLs ([AllowedOrigins](#allowedorigins)).

▶️ 2. change the JWT token key ([JWT Key](#jwt-key)).

▶️ 3. change the default user password ([StockUsers](#stockusers)).

▶️ 4. change the base API URL for the [frontend app](#4-frontend-app-settings).

### Settings and Environment Variables

💡 All the API settings are defined in `appsettings.json`. This is packed in the Docker image, but you can override any of its values by just providing an _environment variable_ in the Docker compose script. So, you don't need to change the Docker image, but just setup some environment variables in your host.

As `appsettings.json` is a JSON document, overriding any of its properties means that you must use an expression to point to it. In the ASP.NET convention, used here, you just have to append the name of each property (case insensitive; conventionally, use uppercase for environment variables) starting from the root object. Each name is separated by a double underscore (or by `:` in Windows hosts: see [this post](https://github.com/aspnet/Configuration/issues/469)).

>👉 The double underscore is supported by all platforms, and is automatically converted into a colon. Paths are case insensitive.

For instance, say your configuration document has a `ConnectionStrings` object like this:

```json
{
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Auth": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "Index": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True"
  }
}
```

In this context, the path to the `Default` connection property will be `CONNECTIONSTRINGS__DEFAULT` (notice the 2 underscores between its words). This is the name of the environment variable to set for overriding the value of this setting.

💡 When dealing with _arrays_ in settings, use the index of each item in it to refer to that item. So, in an array like this:

```json
"AllowedOrigins": [
  "http://localhost:4200",
  "https://www.fusisoft.it/"
],
```

the first item of the array will be targeted by `ALLOWEDORIGINS__0`; the second item by `ALLOWEDORIGINS__1`; and so forth.

### Overriding Settings

To override settings you typically use environment variables in the host. Every sensitive setting is systematically overridden; other settings can be overridden at will, to customize the API's behavior and fit it into its hosting environment.

The Cadmus API uses the standard ASP.NET Core [default configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#default-configuration) order, namely:

1. settings from `appsettings.json`;
2. settings from the environment-specific versions of `appsettings.json` (named like `appsettings.{Environment}.json`);
3. secret manager (in the `Development` environment); this relies on a user secrets file, a JSON file stored on the local developer's machine, outside of the source directory (and thus of source control);
4. environment variables;
5. command-line arguments.

The last override wins. Thus, the command line arguments have the highest precedence, followed by environment variables, etc.

As for [command line](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#command-line-configuration-provider), these are the main points:

- the argument is a key=value pair. The value (which can be empty) either follows the `=` sign, or a space when the key is prefixed with `--` or `/`.
- do not mix the two alternative syntaxes (with `=` or prefix).

Samples from the quoted documentation:

```bash
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```

### AllowedOrigins

Cadmus API implement [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) for cross-origin access. You must configure the allowed origins for CORS under the `AllowedOrigins` section of the API settings. This is an array of strings, listing all the allowed CORS origins, like:

```json
"AllowedOrigins": [
  "http://localhost:4200",
],
```

▶️ 1. Replace the `localhost` origin (`ALLOWEDORIGINS__0`) with the URL of your host, like:

```yml
  environment:
    - ALLOWEDORIGINS__0=https://myproject.somewhere.edu
```

### JWT Key

▶️ 1. Change `Jwt:SecureKey`. The `jwt` object in the configuration contains the configuration for the authentication JWT token. Among its properties, the relevant one is `SecureKey`, which contains a seed for generating the token. This must be a string whose length _must_ be a multiple of 8. Use some long and complex string here; it should at least be 48 characters long; the longer the better. For instance:

```yml
  environment:
    - JWT__SECUREKEY=Rh+m(dkh_Rn6DhOD-wKcd;>P=]Q*T}J/MPbnfenDKOL[1y4I_1Oy1JAU./V98Zex
```

>⚠️ Be sure to make this key long enough (48+ characters), or the ASP.NET service will throw an error at startup.

Other, less relevant settings in the `JWT` section are:

- `Issuer` (`JWT__ISSUER`): the issuer URI.
- `Audience` (`JWT__AUDIENCE`): the audience URI.

### StockUsers

▶️ 1. Change the password of the default stock user (`StockUsers:0:Password`), and optionally add as many other stock users as desired. The `StockUsers` array contains all the stock users, i.e. those users which are seeded together with the database. Typically here we have at least 1 admin user, to ensure that there always is an account for configuring the system.

In Cadmus API settings, each stock user object has its username (which must be unique), password, email address, roles, and first and last name, like in this example:

```json
"StockUsers": [
  {
    "UserName": "zeus",
    "Password": "P4ss-W0rd!",
    "Email": "zeus@gmail.com",
    "Roles": [
      "admin",
      "editor"
    ],
    "FirstName": "Daniele",
    "LastName": "Fusi"
  }
],
```

You must at least replace the default stock user _password_. You can change all the other settings at will, but be sure to at least change the password, because the one provided in the image is the same mock password used for developing, and thus published in the code repository. So, as for any other security related settings, you must change this sensitive data. For instance:

```yml
  environment:
    - STOCKUSERS__0__PASSWORD=2R$M&o*4ej4@
```

>💡 If you want to preload a set of users, create them directly in `appsettings.json`, like in the above example. For security reasons, just ensure that their usernames and passwords are fake. Then, override their usernames and passwords with the real ones via environment variables in your host. This way, you can create fully structured user accounts in JSON, and just provide the sensitive data from environment variables.

In descending order of assigned authorizations, **roles** are:

1. `admin`
2. `editor`
3. `operator`
4. `visitor`

>⚠️ Please ensure that you are complying with all the requirements setup in Cadmus for a password, or you will get errors at startup. The password must include at least 8 characters, uppercase and lowercase letters, digits, and symbols like dash, stop, parentheses, etc.

### Rate Limiting

Cadmus API provides a rate limiting mechanism, which by default is _not_ enabled. If you want to enable it, ensure to carefully calculate the parameters so that you don't cause unwanted service denials. The default settings are:

```json
"RateLimit": {
  "IsDisabled": true,
  "PermitLimit": 100,
  "QueueLimit": 0,
  "TimeWindow": "00:01:00"
},
```

### Server

This object contains the configuration for HTTPS. You might need this if you are configuring HTTPS at the level of the Docker containers. If instead you are using a reverse proxy to handle HTTPS, and dealing with HTTP-only containerized services (which is the easiest and most common configuration), this is not necessary.

```json
"Server": {
  "UseHSTS": false,
  "UseHttpsRedirection": false
}
```

You can turn these settings `true` to enforce HTTPS in Kestrel.

### Auditing and Privacy

Cadmus has a granular auditing policy for data being edited. Most edits are logged in the auditing log (hosted in a MongoDB database), and the full editing history of each datum is stored in the data themselves. Please take the appropriate measures to periodically check this log for suspected activities, and protect it to comply with privacy requirements.

Anyway, no personal data is directly found in the log, as it just stores user names, which usually mean nothing outside a team. For instance, my user name for testing is "zeus"; and the mapping between zeus and my real name, if any, is found only in another database, related to user accounts. Of course, it's up to you to decide whether you want to map user names to real names, or just use first names, fake names, or whatever you prefer.

If you enable logging, all sensitive operations on data are logged with user names and their IP address. In most cases the log is cyclic, so that it won't grow indefinitely (usually it's limited to 10 MB).

>Data history never gets pruned, as it's part of the data themselves and can be used to recover from errors or other accidents, and track the evolution of data being entered.

There are usually 4 types of log targets:

- file: log to a local text file named `cadmus-log.txt`.
- console: log to console. This can be useful when you are running the API from a Docker container, so you can look at the log with `docker logs <CONTAINER_NAME>`.
- MongoDB: log to a MongoDB database. This is specified by `ConnectionStrings:MongoLog`.
- PostgreSQL: log to a PostgreSQL database. This is specified by `ConnectionStrings:PostgresLog`.

The corresponding boolean switches to turn this logging on or off are:

- Auditing:
  - File
  - Mongo
  - Postgres
  - Console

For instance:

```json
"Auditing": {
  "File": true,
  "Mongo": true,
  "Postgres": false,
  "Console": true
}
```

### Database

The database-related settings essentially refer to connection strings. In `appsettings.json`, they are (here `PRJ` is a placeholder for your project's short name):

```json
{
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Auth": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "Index": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "MongoLog": "mongodb://cadmus-PRJ-mongo:27017/cadmus-PRJ-log",
    "PostgresLog": "Server=cadmus-PRJ-pgsql;Database=cadmus-PRJ-log;User Id=postgres;Password=postgres;Include Error Detail=True"
  },
  "DatabaseNames": {
    "Auth": "cadmus-PRJ-auth",
    "Data": "cadmus-PRJ"
  },
  "Seed": {
    "ProfileSource": "%wwwroot%/seed-profile.json",
    "ItemCount": 100,
    "SeedDelay": 0
  }
}
```

>💥 Please remember that the containerized API layer requiring the database service connects to it within a Docker network. This means that the server name is just the name of the container, and that the port is the default port for its database service. Even when you remap the database service port, e.g. to avoid a clash with the same service (PostgreSQL at 5432) on the host, nonetheless the containerized API will continue to connect to the default port 5432.

- `ConnectionStrings` This object contains the connection string template to the MongoDB database (`Default`), and the authentication and index databases (`Auth`, `Index`). You must ensure that the strings use the database service as named in the Docker stack (or, if you prefer, any other external database service, in this case removing it from the bottom of the Docker stack). Typically, they are as follows:

```yml
environment:
  - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
  - CONNECTIONSTRINGS__AUTH=Server=cadmus-PRJ-pgsql;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
  - CONNECTIONSTRINGS__INDEX=Server=cadmus-PRJ-pgsql;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
```

>💡 Note that here the hostnames (like `cadmus-PRJ-mongo` or `cadmus-PRJ-pgsql`) are just the names of the database containerized services in the Docker compose script. In development we just use `localhost`.

- `DatabaseNames` (`DATABASENAMES`): the names of the authentication and data databases. These are not found in the connection string templates, where there is a placeholder `{0}` representing them, as the program needs to know both the full connection string and the database name alone.
  - `Auth` (`DATABASENAMES__AUTH`): user accounts database. Usually `cadmus-PRJ-auth`.
  - `Data` (`DATABASENAMES__DATA`): Cadmus data database. Usually `cadmus-PRJ`.
- `Seed`:
  - `ProfileSource` (`SEED__PROFILESOURCE`): the source of the profile file. Typically you should not change this.
  - `ItemCount` (`SEED__ITEMCOUNT`): the count of the mock items you might want to seed into your database when creating it. On startup, the API service creates and seeds the database if not found.
  - `SeedDelay` (`SEED__DELAY`): an optional delay in seconds. When greater than 0, the API service waits the specified amount of time before starting to seed the database index. This is usually required because the underlying database service takes some time to startup in your host. It is usually advisable to set this to some seconds; for slower machines in local environments you might also increase it up to 25 seconds or more.

#### Securing Databases

⚠️ By default, when you publish a port in Docker, it binds the port to all network interfaces on the host machine. This means the port _is accessible from outside_ the host, which can be a security risk. To ensure that the port is only accessible from the host machine (localhost), you can bind the port to the `127.0.0.1` interface. To do this, just prefix your port with `127.0.0.1:` in compose, like in this example configuration:

```yml
services:
  # MongoDB
  cadmus__PRJ__mongo:
    image: mongo
    container_name: cadmus__PRJ__mongo
    restart: unless-stopped
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/var/log/mongodb
    command: mongod --config /etc/mongod.conf
    ports:
      # 👉 CHANGE HERE
      - 127.0.0.1:27017:27017
    volumes:
      - mongo-vol:/data/db
      - ./mongod.conf:/etc/mongod.conf
      - ./mongo-logs:/var/log/mongodb
    networks:
      - nginx-proxy

  # PostgreSQL
  cadmus__PRJ__pgsql:
    image: postgres
    container_name: cadmus__PRJ__pgsql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      # 👉 CHANGE HERE
      - 127.0.0.1:5432:5432
    volumes:
      - pgsql-vol:/var/lib/postgresql/data
    networks:
      - nginx-proxy

volumes:
  mongo-vol:
  pgsql-vol:
  mongo-logs:

networks:
  nginx-proxy:
    external: true
```

#### External Databases

Should you prefer to use a database service outside the Docker compose stack, you must slightly modify the script.

The original script uses container names as data server names in the connection strings. In this scenario, you will rather replace these with your data service URI.

⚠️ As the containers in the compose script all refer to a Docker virtual network, you must use `extra_hosts` to address a service outside this network, as it is the case for your external data services. Thus, you should end up adding an `extra_hosts` section to each API service using a database, having an arbitrary name for your database service, and its mapping to an IP address. Once you have this, just replace the server name in the connection string with the arbitrary name chosen for your database service.

To sum up, the API section should look like this:

```yml
services:
  cadmus-PRJ-api:
    image: ...
    environment:
      - CONNECTIONSTRINGS__DEFAULT=Server=somedb;...
    networks:
      - cadmus-PRJ-network
    # added for mapping to IP outside the Docker virtual network
    extra_hosts:
      - "somedb:130.140.150.200"
```

In this API service the connection string points to the `somedb` server (`Server=somedb`), which is mapped to the IP 130.140.150.200 in its `extra_hosts` section.

### Other Services

- `Messaging`: this object is used to build automatic messages in the API service, mostly related to users management. The messaging object is like:

```json
"Messaging": {
  "AppName": "Cadmus",
  "ApiRootUrl": "https://cadmus.azurewebsites.net/api/",
  "AppRootUrl": "https://fusisoft.it/apps/cadmus/",
  "SupportEmail": "webmaster@fusisoft.net"
},
```

- AppName (`MESSAGING__APPNAME`): application name.
- ApiRootUrl: (`MESSAGING__APIROOTURL`): root URL to the API services.
- AppRootUrl: (`MESSAGING__APPROOTURL`): root URL to the web application.
- SupportEmail: (`MESSAGING__SUPPORTEMAIL`): email address for support.

The settings here are used to fill some placeholders in the body of the messages being built. The provided values are mock values. Feel free to change these settings as you prefer, or just leave them as they are if you are not going to use email.

- `Mailer`: this object is used to configure an SMTP service to be consumed by the API service, when managing users. For instance, a user might want to recover his/her password, which here requires an email message to reset the forgotten password and setup a new one. Anyway, this is not a requirement, especially if you just manage a few users; so you can just disable this service altogether. The configuration object is like this for a standard SMTP server:

```json
"Mailer": {
  "IsEnabled": false,
  "SenderEmail": "webmaster@fusisoft.net",
  "SenderName": "Cadmus",
  "TestRecipient": "you@somewhere.com",
  "Host": "YourSmtpHost",
  "Port": 587,
  "UseSsl": true,
  "UserName": "SmtpUserName",
  "Password": "SmtpPassword"
}
```

All the messaging services share these options:

- `IsEnabled` (`MAILER__ISENABLED`): a boolean used to enable or disable messaging.
- `SenderEmail` (`MAILER__SENDEREMAIL`): the sender email address.
- `SenderName` (`MAILER__SENDERNAME`): the sender human-friendly name.
- `TestRecipient` (`MAILER__TESTRECIPIENT`): the recipient email address to be used when sending a test email message. If not specified, no mail message will be sent.

Other options are specific for each service. Other mailing services can be used (e.g. SendGrid, MailJet, etc.); this requires just swapping the default service with another in the API startup code.

>💡 The messaging section is separated from the mailer section because building a text message is a process potentially shared among different messaging systems (email, SMS, etc.). Messages are built according to a set of HTML templates in the API layer (in folder `wwwroot/messages` of its source code).

## 4. Configure Frontend App

When hosting on a server, the web application needs to know the address of the API service, which is no longer found at `localhost`. To avoid having to recompile the Angular app when changing this address, the app has an uncompiled `env.json` file including the base URIs accessed by it. So, you can just change the URIs inside this file and you are ready to go.

>💡 Also remember that your web app too will be relocated on a server, so it will no longer be running at `localhost`. This means that you should ensure to include its new location under the [allowed CORS origins](#allowedorigins) of the API layer.

This is the default content of `env.js` (local port and version numbers of course vary):

```js
(function (window) {
  window.__env = window.__env || {};
  window.__env.apiUrl = "http://localhost:29557/api/";
  window.__env.version = "0.0.10";
  // enable thesaurus import in thesaurus list for admins
  window.__env.thesImportEnabled = true;
})(this);
```

As you can see, the only URI to change in this file is `apiUrl`. Optioanlly, you can also change the version label, which usually gets displayed at the bottom of the page in the app.

▶️ 1. create a copy of `env.js` in your host file system, and change it as required.

▶️ 2. add to the Docker compose script a _bind mount_ with the modified copy of `env.js`, e.g.:

```yml
services:
  cadmus-PRJ-app:
    image: yourdockerrepo/cadmus-__PRJ__-app:0.0.1
    container_name: cadmus-__PRJ__-app
    # ...
    # 👉 bind mount
    volumes:
      - /opt/dockers/cadmus-__PRJ__/env.js:/usr/share/nginx/html/env.js
```

In this example:

- your _host_ machine has a patched `env.js` file in its local file system, in directory `/opt/dockers/cadmus-__PRJ__/web`
- this file gets bound to the container's `env.js` file in its NGINX `/usr/share/nginx/html` directory.

>Alternatively, you might create an ad-hoc Docker image for production, but this would be more cumbersome. You might even "hack" the Docker image on the host by directly changing the file's content in the container, but manually changing a Docker image is not a good practice.
