---
title: "Creating API"
parent: "Creating Backend Core"
layout: default
nav_order: 6
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

▶️ (1) create a new ASP.NET Core web API project (no authentication) named `Cadmus<PRJ>Api`: select `None` for `Authentication type`, ensure that `Enable Docker` and `Use HTTPS` is disabled (we'll provide our own Docker files), ensure that `Use controllers`, `Enable OpenAPI support`, and `Do not use top-level statements` are checked.

>Remember to disable HTTPS. In most API configurations HTTPS is managed by a reverse proxy, and this option is not required here in development.

▶️ (2) remove the mock `WeatherForecast.cs` class and its corresponding `WeatherForecastController.cs` class from the `Controllers` folder.

▶️ (3) add NuGet packages (run these from the project folder; remove the packages you do not need):

```ps1
dotnet add package Cadmus.Api.Config
dotnet add package Cadmus.Api.Controllers
dotnet add package Cadmus.Api.Controllers.Export
dotnet add package Cadmus.Api.Controllers.Import
dotnet add package Cadmus.Api.Models
dotnet add package Cadmus.Api.Services
dotnet add package Cadmus.Graph.Ef.PgSql
dotnet add package Cadmus.Graph.Extras
dotnet add package Cadmus.Index.Ef.PgSql
dotnet add package Cadmus.Core
dotnet add package Cadmus.Mongo
dotnet add package Cadmus.Seed
dotnet add package Cadmus.Seed.General.Parts
dotnet add package Cadmus.Seed.Philology.Parts
dotnet add package Fusi.Antiquity
dotnet add package Fusi.Api.Auth.Controllers
dotnet add package Fusi.Microsoft.Extensions.Configuration.InMemoryJson
dotnet add package MessagingApi
dotnet add package MessagingApi.SendGrid
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Logging.Debug
dotnet add package Polly
dotnet add package Scalar.AspNetCore
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Exceptions
dotnet add package Serilog.Extensions.Hosting
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.MongoDB
dotnet add package Serilog.Sinks.Postgresql.Alternative
dotnet add package System.IdentityModel.Tokens.Jwt
```

💡 You can use a Powershell batch like this if you do not want to copy-paste each command:

```ps1
$packages = @(
  "Cadmus.Api.Config", "Cadmus.Api.Controllers", "Cadmus.Api.Controllers.Export",
  "Cadmus.Api.Controllers.Import", "Cadmus.Api.Models", "Cadmus.Api.Services",
  "Cadmus.Graph.Ef.PgSql", "Cadmus.Graph.Extras", "Cadmus.Index.Ef.PgSql",
  "Cadmus.Core", "Cadmus.Mongo", "Cadmus.Seed", "Cadmus.Seed.General.Parts",
  "Cadmus.Seed.Philology.Parts", "Fusi.Antiquity", "Fusi.Api.Auth.Controllers",
  "Fusi.Microsoft.Extensions.Configuration.InMemoryJson", "MessagingApi",
  "MessagingApi.SendGrid", "Microsoft.AspNetCore.Authentication.JwtBearer",
  "Microsoft.AspNetCore.OpenApi", "Microsoft.AspNetCore.Mvc.NewtonsoftJson",
  "Microsoft.Extensions.Configuration", "Microsoft.Extensions.Logging.Debug",
  "Polly", "Scalar.AspNetCore", "Serilog", "Serilog.AspNetCore",
  "Serilog.Exceptions", "Serilog.Extensions.Hosting", "Serilog.Sinks.Console",
  "Serilog.Sinks.File", "Serilog.Sinks.MongoDB", "Serilog.Sinks.Postgresql.Alternative",
  "System.IdentityModel.Tokens.Jwt"
)

foreach ($pkg in $packages) {
  dotnet add package $pkg
}
```

💡 Alternatively, just paste this code in the project file and then use NuGet package manager to update all the packages (replace `__PRJ__` with your project name, removing project's parts if they are not present):

```xml
<ItemGroup>
  <PackageReference Include="Cadmus.Api.Config" Version="10.1.23" />
  <PackageReference Include="Cadmus.Api.Controllers" Version="12.0.8" />
  <PackageReference Include="Cadmus.Api.Controllers.Export" Version="0.0.7" />
  <PackageReference Include="Cadmus.Api.Controllers.Import" Version="11.0.12" />
  <PackageReference Include="Cadmus.Api.Models" Version="10.1.18" />
  <PackageReference Include="Cadmus.Api.Services" Version="12.0.8" />
  <PackageReference Include="Cadmus.Graph.Ef.PgSql" Version="9.0.4" />
  <PackageReference Include="Cadmus.Graph.Extras" Version="8.0.11" />
  <PackageReference Include="Cadmus.Index.Ef.PgSql" Version="9.0.4" />
  <PackageReference Include="Cadmus.Core" Version="8.0.11" />
  <PackageReference Include="Cadmus.Mongo" Version="8.0.11" />
  <PackageReference Include="Cadmus.Seed" Version="8.0.11" />
  <PackageReference Include="Cadmus.Seed.General.Parts" Version="7.0.7" />
  <PackageReference Include="Cadmus.Seed.Philology.Parts" Version="10.0.1" />
  <PackageReference Include="Fusi.Antiquity" Version="5.1.1" />
  <PackageReference Include="Fusi.Api.Auth.Controllers" Version="6.0.5" />
  <PackageReference Include="Fusi.Microsoft.Extensions.Configuration.InMemoryJson" Version="4.0.0" />
  <PackageReference Include="MessagingApi" Version="5.0.0" />
  <PackageReference Include="MessagingApi.SendGrid" Version="5.0.1" />
  <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="9.0.9" />
  <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="9.0.9" />
  <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="9.0.9" />
  <PackageReference Include="Microsoft.Extensions.Configuration" Version="9.0.9" />
  <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="9.0.9" />
  <PackageReference Include="Polly" Version="8.6.4" />
  <PackageReference Include="Scalar.AspNetCore" Version="2.8.11" />
  <PackageReference Include="Serilog" Version="4.3.0" />
  <PackageReference Include="Serilog.AspNetCore" Version="9.0.0" />
  <PackageReference Include="Serilog.Exceptions" Version="8.4.0" />
  <PackageReference Include="Serilog.Extensions.Hosting" Version="9.0.0" />
  <PackageReference Include="Serilog.Sinks.Console" Version="6.0.0" />
  <PackageReference Include="Serilog.Sinks.File" Version="7.0.0" />
  <PackageReference Include="Serilog.Sinks.MongoDB" Version="7.1.0" />
  <PackageReference Include="Serilog.Sinks.Postgresql.Alternative" Version="4.2.0" />
  <PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="8.14.0" />
</ItemGroup>
```

>You can remove the Serilog sinks you are not going to use, like e.g. the PostgreSQL one. Also, typically you will add your project's `Cadmus.PRJ.Services` package(s). Also, if you do not use statistics in your UI you can remove `Cadmus.Api.Controllers.Export`.

## 2. Add Settings

▶️ (1) Add these settings to `appsettings.json` (replace `__PRJ__` with your project's name). Feel free to customize them as required.

>Please notice that all the sensitive data like users and passwords are there only for illustration purposes, and they will be overwritten by environment variables set in the [host server](../deploy).

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
  "AllowedOrigins": [
    "http://localhost:4200",
  ],
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
      "Roles": [
        "admin",
        "editor",
        "operator",
        "visitor"
      ],
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

>⚠️ before API v10, the authentication database was MongoDB. Now it is a PostgreSQL database, as specified by `ConnectionStrings:Auth` and `DatabaseNames:Auth`.

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

        // index and graph
        ServiceConfigurator.ConfigureIndexServices(services, config);
        ServiceConfigurator.ConfigureGraphServices(services, config);

        // previewer
        services.AddSingleton(p => ServiceConfigurator.GetPreviewer(p, config));
    }

    /// <summary>
    /// Configures the services.
    /// </summary>
    /// <param name="services">The services.</param>
    public static void ConfigureServices(IServiceCollection services,
        IConfiguration config, IHostEnvironment hostEnvironment)
    {
        // configuration
        services.AddSingleton(_ => config);
        ServiceConfigurator.ConfigureOptionsServices(services, config);

        // security
        ServiceConfigurator.ConfigureCorsServices(services, config);
        ServiceConfigurator.ConfigureRateLimiterService(services, config, hostEnvironment);
        ServiceConfigurator.ConfigureAuthServices(services, config);

        // proxy
        services.AddHttpClient();
        services.AddResponseCaching();

        // API controllers
        services.AddControllers();
        // camel-case JSON in response
        services.AddMvc()
            // https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-2.2&tabs=visual-studio#jsonnet-support
            .AddJsonOptions(options =>
            {
                options.JsonSerializerOptions.PropertyNamingPolicy =
                    JsonNamingPolicy.CamelCase;
            });

        // framework services
        // IMemoryCache: https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory
        services.AddMemoryCache();

        // user repository service
        services.AddScoped<IUserRepository<NamedUser>,
            UserRepository<NamedUser, IdentityRole>>();

        // messaging
        ServiceConfigurator.ConfigureMessagingServices(services);

        // logging
        ServiceConfigurator.ConfigureLogging(services);

        // app services
        ConfigureAppServices(services, config);
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

If you are not going to use a project-specific services library, add your app services in a new `Services` folder:

- `AppRepositoryProvider.cs`: parts
- `AppPartSeederFactoryProvider.cs`: part seeders

Here are two example implementations, just customize the assemblies to include:

```cs
// AppRepositoryProvider.cs (optional)
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

```cs
// AppPartSeederFactoryProvider.cs (optional)
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

>Inside the `messages` folder you can customize the message templates as you prefer, but usually this is not required.

## 5. Setup Docker

▶️ (1) In the project's root (where the `.sln` file is located), add a `Dockerfile` to build the Docker image (replace `__PRJ__` with your project's name):

```yml
# Stage 1: base
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 443

# Stage 2: build
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj", "Cadmus__PRJ__Api/"]
# copy local packages to avoid using a NuGet custom feed, then restore
# COPY ./local-packages /src/local-packages
RUN dotnet restore "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -s https://api.nuget.org/v3/index.json --verbosity n
# copy the content of the API project
COPY . .
# build it
RUN dotnet build "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/build

# Stage 3: publish
FROM build AS publish
RUN dotnet publish "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/publish

# Stage 4: final
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Cadmus__PRJ__Api.dll"]
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

>⚠️ Note that setting `ASPNETCORE_URLS` for Docker is a requirement because the default HTTP port for ASP.NET core in development mode is 5000.

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

To build a Docker image (replace `PRJ` with your project's name):

```ps1
docker buildx build . --platform linux/amd64,linux/arm64,windows/amd64 -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest --push
```

## 6. Add Readme

▶️ (1) Add a readme like this:

```txt
# Cadmus PRJ API

🐋 Quick Docker image build:

  docker buildx build . --platform linux/amd64,linux/arm64,windows/amd64 -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest --push

(replace with the current version).

This is a Cadmus API layer customized for the PRJ project. Most of its code is derived from [shared Cadmus libraries](https://github.com/vedph/cadmus-api).
```
