---
title: "Creating API"
parent: "Developing Backend"
layout: default
nav_order: 7
---

- [Creating API](#creating-api)
  - [1. Create Project](#1-create-project)
  - [2. Add Settings](#2-add-settings)
  - [3. Add Program](#3-add-program)
  - [4. Add Assets](#4-add-assets)
  - [5. Setup Docker](#5-setup-docker)
  - [6. Add Readme](#6-add-readme)

# Creating API

## 1. Create Project

The [reference API backend project](https://github.com/vedph/cadmus-api) is the model for this section.

▶️ (1) **create a new ASP.NET Core web API project** (no authentication) named `Cadmus<PRJ>Api`: select `None` for `Authentication type`, ensure that `Enable container support` and `Use HTTPS` is disabled (we'll provide our own Docker files), ensure that `Use controllers`, `Enable OpenAPI support`, and `Do not use top-level statements` are checked.

> Remember to disable HTTPS. In most API configurations HTTPS is managed by a reverse proxy, and this option is not required here in development.

▶️ (2) **remove mock** `WeatherForecast.cs` class and its corresponding `WeatherForecastController.cs` class from the `Controllers` folder.

▶️ (3) **add NuGet packages**, e.g. (paste these references in the project file and modify them as needed, using the NuGet package manager to update all the packages):

```xml
<ItemGroup>
  <PackageReference Include="Cadmus.Api.Config" />
  <PackageReference Include="Cadmus.Api.Controllers" />
  <PackageReference Include="Cadmus.Api.Controllers.Export" />
  <PackageReference Include="Cadmus.Api.Controllers.Import" />
  <PackageReference Include="Cadmus.Api.Models" />
  <PackageReference Include="Cadmus.Api.Services" />
  <PackageReference Include="Cadmus.Graph" />
  <PackageReference Include="Cadmus.Graph.Ef.PgSql" />
  <PackageReference Include="Cadmus.Graph.Extras" />
  <PackageReference Include="Cadmus.Seed" />
  <!-- add/remove parts packages as needed -->
  <PackageReference Include="Cadmus.Seed.Codicology.Parts" />
  <PackageReference Include="Cadmus.Seed.Epigraphy.Parts" />
  <PackageReference Include="Cadmus.Seed.General.Parts" />
  <PackageReference Include="Cadmus.Seed.Philology.Parts" />
  <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" />
  <PackageReference Include="Microsoft.AspNetCore.OpenApi" />
  <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" />
  <PackageReference Include="Newtonsoft.Json" />
  <PackageReference Include="Polly" />
  <PackageReference Include="Scalar.AspNetCore" />
  <PackageReference Include="Serilog" />
  <PackageReference Include="Serilog.AspNetCore" />
  <PackageReference Include="Serilog.Exceptions" />
  <PackageReference Include="Serilog.Extensions.Hosting" />
  <PackageReference Include="Serilog.Sinks.Console" />
  <PackageReference Include="Serilog.Sinks.File" />
  <PackageReference Include="Serilog.Sinks.MongoDB" />
  <PackageReference Include="Serilog.Sinks.Postgresql.Alternative" />
</ItemGroup>
```

> You can remove the Serilog sinks you are not going to use, like e.g. the PostgreSQL one.

## 2. Add Settings

▶️ (1) **Add settings** to `appsettings.json` (replace `__PRJ__` with your project's name). Feel free to customize them as required.

> ⚠️ Please notice that all the sensitive data like users and passwords are there only for illustration purposes, and they will be overwritten by environment variables set in the [host server](../deploy).

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Auth": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "Index": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "MongoLog": "mongodb://localhost:27017/cadmus-__PRJ__-log",
    "PostgresLog": "Server=localhost;Database=cadmus-__PRJ__-log;User Id=postgres;Password=postgres;Include Error Detail=True"
  },
  "DatabaseNames": {
    "Auth": "cadmus-__PRJ__-auth",
    "Data": "cadmus-__PRJ__"
  },
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.File",
      "Serilog.Sinks.MongoDB",
      "Serilog.Sinks.Postgresql.Alternative"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Information",
        "System": "Warning"
      }
    }
  },
  "Auditing": {
    "File": true,
    "Mongo": true,
    "Postgres": false,
    "Console": true
  },
  "AllowedOrigins": ["http://localhost:4200"],
  "RateLimit": {
    "IsDisabled": true,
    "PermitLimit": 100,
    "QueueLimit": 0,
    "TimeWindow": "00:01:00"
  },
  "Seed": {
    "ProfileSource": "%wwwroot%/seed-profile.json",
    "ItemCount": 100,
    "Delay": 0
  },
  "Jwt": {
    "Issuer": "https://cadmus.azurewebsites.net",
    "Audience": "https://www.fusisoft.it",
    "SecureKey": "7W^3*y5@a!3%5Wu4xzd@au5Eh9mdFG6%WmzQpjDEB8#F5nXT"
  },
  "StockUsers": [
    {
      "UserName": "zeus",
      "Password": "P4ss-W0rd!",
      "Email": "dfusi@hotmail.com",
      "Roles": ["admin", "editor", "operator", "visitor"],
      "FirstName": "Daniele",
      "LastName": "Fusi"
    }
  ],
  "Messaging": {
    "AppName": "Cadmus __PRJ__",
    "ApiRootUrl": "https://cadmus.azurewebsites.net/api/",
    "AppRootUrl": "https://fusisoft.it/apps/cadmus/",
    "SupportEmail": "webmaster@fusisoft.net"
  },
  "Editing": {
    "BaseToLayerToleranceSeconds": 60
  },
  "Indexing": {
    "IsEnabled": true,
    "IsGraphEnabled": false
  },
  "Preview": {
    "IsEnabled": true
  },
  "Mailer": {
    "IsEnabled": false,
    "SenderEmail": "webmaster@fusisoft.net",
    "SenderName": "Cadmus __PRJ__",
    "Host": "",
    "Port": 0,
    "UseSsl": true,
    "UserName": "place in environment",
    "Password": "place in environment"
  }
}
```

> ⚠️ before API v10, the authentication database was MongoDB. Now it is a PostgreSQL database, as specified by `ConnectionStrings:Auth` and `DatabaseNames:Auth`.

## 3. Add Program

▶️ (1) Use this template to replace the code in `Program.cs` (replace `__PRJ__` with your project's name):

```cs
using Cadmus.Api.Services;
using Cadmus.Api.Services.Seeding;
using Cadmus.Core;
using CadmusApi.Services;
using Fusi.Api.Auth.Models;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Serilog;
using System.Diagnostics;
using Cadmus.Core.Config;
using Cadmus.Seed;
using Fusi.Api.Auth.Services;
using System.Text.Json;
using Serilog.Events;
using Microsoft.AspNetCore.HttpOverrides;
using Scalar.AspNetCore;
using Cadmus.Api.Controllers;
using Cadmus.Api.Controllers.Import;
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Configuration;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Cadmus.Api.Config.Services;
using Cadmus.Api.Config;

namespace Cadmus__PRJ__Api;

/// <summary>
/// Program.
/// </summary>
public static class Program
{
    // startup log file name, Serilog is configured later via appsettings.json
    private const string STARTUP_LOG_NAME = "startup.log";

    private static void ConfigureAppServices(IServiceCollection services,
        IConfiguration config)
    {
        // Cadmus repository
        string dataCS = string.Format(
        config.GetConnectionString("Default")!,
            config.GetValue<string>("DatabaseNames:Data"));
        services.AddSingleton<IRepositoryProvider>(
            _ => new AppRepositoryProvider { ConnectionString = dataCS });

        // part seeder factory provider
        services.AddSingleton<IPartSeederFactoryProvider,
            AppPartSeederFactoryProvider>();

        // item browser factory provider
        services.AddSingleton<IItemBrowserFactoryProvider>(_ =>
        new StandardItemBrowserFactoryProvider(
                config.GetConnectionString("Default")!));

        // metadata builder factory provider
        services.AddSingleton<IItemMetadataBuilderFactoryProvider>(_ =>
            new StandardItemMetadataBuilderFactoryProvider(
                config.GetConnectionString("Default")!));

        // index and graph
        ServiceConfigurator.ConfigureIndexServices(services, config);
        ServiceConfigurator.ConfigureGraphServices(services, config);

        // previewer
        services.AddSingleton(p => ServiceConfigurator.GetPreviewer(p, config));
    }

    /// <summary>
    /// Entry point.
    /// </summary>
    /// <param name="args">The arguments.</param>
    public static async Task<int> Main(string[] args)
    {
        // early startup logging to ensure we catch any exceptions
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Debug()
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            .Enrich.FromLogContext()
            .WriteTo.Console()
#if DEBUG
            .WriteTo.File(STARTUP_LOG_NAME, rollingInterval: RollingInterval.Day)
#endif
            .CreateLogger();

        try
        {
            Log.Information("Starting Cadmus API host");
            ServiceConfigurator.DumpEnvironmentVars();

            WebApplicationBuilder builder = WebApplication.CreateBuilder(args);
            ServiceConfigurator.ConfigureLogger(builder);

            IConfiguration config = new ConfigurationService(builder.Environment)
                .Configuration;

            ServiceConfigurator.ConfigureServices(builder.Services, config,
                builder.Environment);
            ConfigureAppServices(builder.Services, config);

            builder.Services.AddOpenApi();

            // controllers from Cadmus.Api.Controllers
            builder.Services.AddControllers()
                .AddApplicationPart(typeof(ItemController).Assembly)
                .AddApplicationPart(typeof(ThesaurusImportController).Assembly)
                .AddControllersAsServices();

            WebApplication app = builder.Build();

            // forward headers for use with an eventual reverse proxy
            app.UseForwardedHeaders(new ForwardedHeadersOptions
            {
                ForwardedHeaders = ForwardedHeaders.XForwardedFor
                    | ForwardedHeaders.XForwardedProto
            });

            // development or production
            if (builder.Environment.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                // https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0&tabs=visual-studio
                app.UseExceptionHandler("/Error");
                if (config.GetValue<bool>("Server:UseHSTS"))
                {
                    Console.WriteLine("HSTS: yes");
                    app.UseHsts();
                }
                else
                {
                    Console.WriteLine("HSTS: no");
                }
            }

            // HTTPS redirection
            if (config.GetValue<bool>("Server:UseHttpsRedirection"))
            {
                Console.WriteLine("HttpsRedirection: yes");
                app.UseHttpsRedirection();
            }
            else
            {
                Console.WriteLine("HttpsRedirection: no");
            }

            // CORS
            app.UseCors("CorsPolicy");
            // rate limiter
            if (!config.GetValue<bool>("RateLimit:IsDisabled"))
                app.UseRateLimiter();
            // authentication
            app.UseAuthentication();
            app.UseAuthorization();
            // proxy
            app.UseResponseCaching();

            // seed auth database (via Services/HostAuthSeedExtensions)
            await app.SeedAuthAsync();

            // seed Cadmus database (via Services/HostSeedExtension)
            await app.SeedAsync();

            // map controllers and Scalar API
            app.MapControllers();
            app.MapOpenApi();
            app.MapScalarApiReference(options =>
            {
                options.WithTitle("Cadmus __PRJ__ API")
                       .AddPreferredSecuritySchemes("Bearer");
            });

            Log.Information("Running API");
            await app.RunAsync();

            return 0;
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Cadmus API host terminated unexpectedly");
            Debug.WriteLine(ex.ToString());
            Console.WriteLine(ex.ToString());
            return 1;
        }
        finally
        {
            await Log.CloseAndFlushAsync();
        }
    }
}
```

If you are not going to use a project-specific services library, **add your app services** in a new `Services` folder:

- `AppRepositoryProvider.cs`: parts.
- `AppPartSeederFactoryProvider.cs`: part seeders.

> 💡 It is suggested to add a `Cadmus.__PRJ__.Services` project to contain these project-specific files if you are going to reuse them. This typically happens when you plan to use a CLI tool (either the generic tool -- to build a plugin for it -- or a project-specific tool).

Here are two example implementations; just customize the assemblies to include:

- 📁 `AppRepositoryProvider.cs` (optional):

```cs
using System;
using System.Reflection;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Epigraphy.Parts;
using Cadmus.General.Parts;
using Cadmus.Geo.Parts;
using Cadmus.Mongo;
using Cadmus.Philology.Parts;

namespace CadmusDemoApi.Services;

/// <summary>
/// Application's repository provider. Usually, this is implemented in your
/// project's Services library. Here we have no specific project, so we
/// just provide an API app service here.
/// </summary>
public sealed class AppRepositoryProvider : IRepositoryProvider
{
    private readonly IPartTypeProvider _partTypeProvider;

    /// <summary>
    /// Gets or sets the connection string.
    /// </summary>
    public string ConnectionString { get; set; } = "";

    /// <summary>
    /// Initializes a new instance of the <see cref="AppRepositoryProvider"/> class.
    /// </summary>
    /// <exception cref="ArgumentNullException">configuration</exception>
    public AppRepositoryProvider()
    {
        TagAttributeToTypeMap _map = new();
        _map.Add(
        [
            // TODO: add/remove assemblies as required for your app
            // Cadmus.General.Parts
            typeof(NotePart).GetTypeInfo().Assembly,
            // Cadmus.Philology.Parts
            typeof(ApparatusLayerFragment).GetTypeInfo().Assembly,
            // Cadmus.Epigraphy.Parts
            typeof(EpiScriptsPart).GetTypeInfo().Assembly,
            // Cadmus.Geo.Parts
            typeof(AssertedLocationsPart).GetTypeInfo().Assembly
        ]);

        _partTypeProvider = new StandardPartTypeProvider(_map);
    }

    /// <summary>
    /// Gets the part type provider.
    /// </summary>
    /// <returns>part type provider</returns>
    public IPartTypeProvider GetPartTypeProvider()
    {
        return _partTypeProvider;
    }

    /// <summary>
    /// Creates a Cadmus repository.
    /// </summary>
    /// <returns>repository</returns>
    /// <exception cref="ArgumentNullException">null database</exception>
    public ICadmusRepository CreateRepository()
    {
        // create the repository (no need to use container here)
        MongoCadmusRepository repository =
            new(_partTypeProvider, new StandardItemSortKeyBuilder());

        repository.Configure(new MongoCadmusRepositoryOptions
        {
            ConnectionString = ConnectionString ??
                throw new InvalidOperationException(
                "No connection string set for IRepositoryProvider implementation")
        });

        return repository;
    }
}
```

- 📁 `AppPartSeederFactoryProvider.cs` (optional):

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.Epigraphy.Parts;
using Cadmus.Seed.General.Parts;
using Cadmus.Seed.Geo.Parts;
using Cadmus.Seed.Philology.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Hosting;
using System;
using System.Reflection;

namespace CadmusDemoApi.Services;

/// <summary>
/// Application's part seeders factory provider. Usually, this is implemented
/// in your project's Services library. Here we have no specific project, so we
/// just provide an API app service here.
/// </summary>
public sealed class AppPartSeederFactoryProvider : IPartSeederFactoryProvider
{
    private static IHost GetHost(string config)
    {
        // build the tags to types map for parts/fragments
        Assembly[] seedAssemblies =
        [
            // TODO: add/remove assemblies as required for your app
            // Cadmus.General.Seed.Parts
            typeof(NotePartSeeder).Assembly,
            // Cadmus.Seed.Philology.Parts
            typeof(ApparatusLayerFragmentSeeder).Assembly,
            // Cadmus.Seed.Geo.Parts
            typeof(AssertedLocationsPartSeeder).GetTypeInfo().Assembly,
            // Cadmus.Seed.Epigraphy.Parts
            typeof(EpiScriptsPartSeeder).GetTypeInfo().Assembly
        ];
        TagAttributeToTypeMap map = new();
        map.Add(seedAssemblies);

        return new HostBuilder()
            .ConfigureServices((hostContext, services) =>
            {
                PartSeederFactory.ConfigureServices(services,
                    new StandardPartTypeProvider(map),
                    seedAssemblies);
            })
            // extension method from Fusi library
            .AddInMemoryJson(config)
            .Build();
    }

    /// <summary>
    /// Gets the part/fragment seeders factory.
    /// </summary>
    /// <param name="profile">The profile.</param>
    /// <returns>Factory.</returns>
    /// <exception cref="ArgumentNullException">profile</exception>
    public PartSeederFactory GetFactory(string profile)
    {
        ArgumentNullException.ThrowIfNull(profile);

        return new PartSeederFactory(GetHost(profile));
    }
}
```

## 4. Add Assets

▶️ (1) Copy the whole `wwwroot` from [CadmusApi](https://github.com/vedph/cadmus_api), and customize its contents (the Cadmus profile, and if needed the messages template text).

Inside that folder, edit:

- the `seed-profile.json` file, which contains the full Cadmus configuration for data and editors used in the project:
  - remove all the parts, fragments, and seeders you do not use and all the parts, fragment, and seeders you require;
  - do the same for thesaurus entries.
- the `preview-profile.json` file, which contains the configuration for parts preview. If you have no preview, just use an empty JSON object `{}` as its content.

This is the core customization for the whole project. Usually, the profile file is created after the documentation is completed, and before creating the code.

> Inside the `messages` folder you can customize the message templates as you prefer, but usually this is not required.

## 5. Setup Docker

▶️ (1) In the project's root (where the `.slnx` file is located), add a `Dockerfile` to build the Docker image (replace `__PRJ__` with your project's name):

```yml
# Stage 1: base (uses target platform architecture for ASP.NET runtime)
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 443

# Stage 2: build/publish (SDK runs natively on host platform for speed)
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG TARGETARCH
ARG TARGETOS

WORKDIR /src

# Copy project files and source
COPY . .

# Use bash to map amd64 -> x64 and build for the target RID cleanly
RUN /bin/bash -c '\
    RID_ARCH="${TARGETARCH/amd64/x64}" && \
    dotnet publish "Cadmus.__PRJ__.Api/Cadmus.__PRJ__.Api.csproj" \
    -c Release \
    -r "${TARGETOS:-linux}-${RID_ARCH}" \
    --no-self-contained \
    -o /app/publish'

# Stage 3: final image
FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "Cadmus.__PRJ__.Api.dll"]
```

▶️ (2) add a `docker-compose.yml` file to allow you using the API in a composer stack (replace `PRJ` with your project name; of course, you can change your image name as required to fit your organization).

⚠️ **ATTENTION**: under cadmus-api ports replace `5052` with the port value used by your API project (you can find it under the project's properties, Debug, Launch Profiles, HTTP).

```yml
services:
  # MongoDB
  cadmus-PRJ-mongo:
    image: mongo
    container_name: cadmus-PRJ-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    networks:
      - cadmus-PRJ-network

  # PostgreSQL
  cadmus-PRJ-pgsql:
    image: postgres
    container_name: cadmus-PRJ-pgsql
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-PRJ-network

  # Biblio API
  # TODO: remove if you are not using it
  cadmus-biblio-api:
    image: vedph2020/cadmus-biblio-api:7.0.0
    container_name: cadmus-biblio-api
    ports:
      - 60058:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - CONNECTIONSTRINGS__BIBLIO=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SEED__BIBLIODELAY=50
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
    networks:
      - cadmus-PRJ-network

  # Cadmus PRJ API
  cadmus-PRJ-api:
    image: vedph2020/cadmus-PRJ-api:0.0.1
    container_name: cadmus-PRJ-api
    ports:
      # TODO: set your port replacing 5052
      - 5052:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__DELAY=20
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-PRJ-network

networks:
  cadmus-PRJ-network:
    driver: bridge
```

> ⚠️ Note that setting `ASPNETCORE_URLS` for Docker is a requirement because the default HTTP port for ASP.NET core in development mode is 5000.

▶️ (3) add a `.dockerignore` file with this content:

```txt
**/.classpath
**/.dockerignore
**/.env
**/.git
**/.gitignore
**/.project
**/.settings
**/.toolstarget
**/.vs
**/.vscode
**/*.*proj.user
**/*.dbmdl
**/*.jfm
**/azds.yaml
**/bin
**/charts
**/docker-compose*
**/Dockerfile*
**/node_modules
**/npm-debug.log
**/obj
**/secrets.dev.yaml
**/values.dev.yaml
LICENSE
README.md
```

🐋 To **build a Docker image**:

(1) Before creating Docker images, ensure that you have published all the required NuGet packages and that you have a buildx builder instance running that supports multi-arch:

```sh
docker buildx create --use --name multi-arch-builder || docker buildx use multi-arch-builder
docker buildx inspect --bootstrap
```

> To run natively on Linux VMs, macOS (both Intel and Apple Silicon), and Windows (via WSL2 or Docker Desktop)—`linux/amd64` and `linux/arm64` are the only two targets we need. Note that `docker buildx` automatically injects variables like `TARGETARCH` and `TARGETOS` into the scope of your build. In `Dockerfile` we pass these directly to the .NET CLI commands.

(2) Build for multiple platforms and push directly to Docker Hub:

```sh
docker buildx build --platform linux/amd64,linux/arm64 -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest --push .
```

> When there is no available ARM image, add `platform: linux/amd64` as a sibling of the `image` line in docker compose script to force Docker to pull and run the x64 image under emulation, e.g.:

- add `platform: linux/arm64` to MongoDB and PostgreSQL. Note that some images (especially for PostgreSQL) are not available for ARM, so you may need to add `platform: linux/amd64` to them too or downgrade to a version which supports ARM (e.g. `image: postgres:16`).
- add `platform: linux/amd64` to API and app.

## 6. Add Readme

▶️ (1) Add a readme like this:

```txt
# Cadmus PRJ API

🐋 Quick Docker image build:

(1) ensure buildx is running:

```sh
docker buildx create --use --name multi-arch-builder || docker buildx use multi-arch-builder
docker buildx inspect --bootstrap
``

(2) build for multiple platforms and push directly to Docker Hub:

```sh
docker buildx build --platform linux/amd64,linux/arm64 -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest --push .
``

(replace with the current version).

This is a Cadmus API layer customized for the PRJ project. Most of its code is derived from [shared Cadmus libraries](https://github.com/vedph/cadmus-api).
```
