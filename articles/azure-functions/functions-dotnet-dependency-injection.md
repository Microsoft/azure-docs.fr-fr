---
title: Utiliser l’injection de dépendances dans .NET Azure Functions
description: Découvrez comment utiliser l’injection de dépendances pour l’inscription et l’utilisation de services dans les fonctions .NET
author: ggailey777
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/15/2020
ms.author: glenga
ms.reviewer: jehollan
ms.openlocfilehash: ee2e7dc577e000878884655c0ed5f4bcb1aabab5
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92167693"
---
# <a name="use-dependency-injection-in-net-azure-functions"></a>Utiliser l’injection de dépendances dans .NET Azure Functions

Azure Functions prend en charge le modèle de conception logicielle d’injection de dépendances, qui est une technique pour obtenir une [inversion de contrôle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) entre les classes et leurs dépendances.

- L’injection de dépendance dans Azure Functions repose sur les fonctionnalités d’injection de dépendance .Net Core. Il est recommandé d’avoir une certaine connaissance de [l’injection de dépendance .NET Core](/aspnet/core/fundamentals/dependency-injection). Il existe des différences dans le remplacement des dépendances et la lecture des valeurs de configuration avec Azure Functions sur le plan de consommation.

- L’injection de dépendances est prise en charge depuis Azure Functions 2.x.

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir utiliser l’injection de dépendances, vous devez installer les packages NuGet suivants :

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/)

- Package [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/) version 1.0.28 ou ultérieure

## <a name="register-services"></a>Inscrire des services

Pour inscrire des services, créez une méthode pour configurer et ajouter des composants à une instance `IFunctionsHostBuilder`.  L’hôte Azure Functions crée une instance de `IFunctionsHostBuilder` et la passe directement dans votre méthode.

Pour inscrire la méthode, ajoutez l’attribut d’assembly `FunctionsStartup` qui spécifie le nom de type utilisé lors du démarrage.

```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();

            builder.Services.AddSingleton<IMyService>((s) => {
                return new MyService();
            });

            builder.Services.AddSingleton<ILoggerProvider, MyLoggerProvider>();
        }
    }
}
```

Cet exemple utilise le package [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) requis pour inscrire un `HttpClient` au démarrage.

### <a name="caveats"></a>Mises en garde

Des étapes d’inscription s’exécutent avant et après le traitement de la classe de démarrage par le runtime. Vous devez donc garder à l’esprit les considérations suivantes :

- *La classe de démarrage est destinée uniquement à la configuration et à l’inscription.* Évitez d’utiliser des services inscrits au cours du processus de démarrage. Par exemple, n’essayez pas de consigner un message dans un enregistreur d’événements inscrit lors du démarrage. Cette étape du processus d’inscription est trop précoce pour que vos services soient disponibles. Une fois la méthode `Configure` exécutée, le runtime Functions continue d’inscrire des dépendances supplémentaires, ce qui peut affecter le fonctionnement de vos services.

- *Le conteneur d’injection de dépendances contient uniquement des types inscrits explicitement* . Les seuls services disponibles en tant que types injectables sont ceux qui sont configurés dans la méthode `Configure`. Par conséquent, des types spécifiques d’Azure Functions, tels que `BindingContext` et `ExecutionContext`, ne sont pas disponibles pendant la configuration ou en tant que types injectables.

## <a name="use-injected-dependencies"></a>Utiliser les dépendances injectées

L’injection de constructeur est utilisée pour rendre vos dépendances disponibles dans une fonction. Pour pouvoir utiliser l’injection de constructeur, vous ne devez pas vous servir de classes statiques pour les services injectés ni pour les classes de votre fonction.

L’exemple suivant montre comment les dépendances `IMyService` et `HttpClient` sont injectées dans une fonction déclenchée via HTTP.

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Net.Http;
using System.Threading.Tasks;

namespace MyNamespace
{
    public class MyHttpTrigger
    {
        private readonly HttpClient _client;
        private readonly IMyService _service;

        public MyHttpTrigger(HttpClient httpClient, IMyService service)
        {
            this._client = httpClient;
            this._service = service;
        }

        [FunctionName("MyHttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            var response = await _client.GetAsync("https://microsoft.com");
            var message = _service.GetMessage();

            return new OkObjectResult("Response from function with injected dependencies.");
        }
    }
}
```

Cet exemple utilise le package [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) requis pour inscrire un `HttpClient` au démarrage.

## <a name="service-lifetimes"></a>Durées de service

Les durées de service des applications Azure Functions sont identiques à celles du service d’[injection de dépendance ASP.NET](/aspnet/core/fundamentals/dependency-injection#service-lifetimes). Pour une application Azure Functions, les différentes durées de vie de service se comportent comme suit :

- **Temporaire**  : Des services temporaires sont créés à chaque demande du service.
- **Inclus dans l’étendue**  : La durée de vie du service étendu correspond à celle d’exécution de la fonction. Les services délimités sont créés une fois par exécution. Les demandes ultérieures pour ce service pendant l’exécution réutilisent l’instance de service existante.
- **Singleton**  : La durée de vie de service singleton correspond à celle de l’hôte, et est réutilisée entre les exécutions de la fonction sur cette instance. Les services à durée de vie singleton sont recommandés pour des connexions et des clients, par exemple, pour des instances `DocumentClient` ou `HttpClient`.

Affichez ou téléchargez un exemple [des différentes durées de vie de service](https://github.com/Azure/azure-functions-dotnet-extensions/tree/main/src/samples/DependencyInjection/Scopes) sur GitHub.

## <a name="logging-services"></a>Services de journalisation

Si vous avez besoin de votre propre fournisseur de journalisation, inscrivez un type personnalisé en tant qu’instance de [`ILoggerProvider`](/dotnet/api/microsoft.extensions.logging.iloggerfactory), disponible dans le package NuGet [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/).

Application Insights est ajouté automatiquement par Azure Functions.

> [!WARNING]
> - N’ajoutez pas `AddApplicationInsightsTelemetry()` à la collection de services, car il enregistre des services en conflit avec les services fournis par l’environnement.
> - N’inscrivez pas votre propre `TelemetryConfiguration` ou `TelemetryClient` si vous utilisez la fonctionnalité intégrée Application Insights. Si vous devez configurer votre propre instance `TelemetryClient`, créez-en une via l’instance `TelemetryConfiguration` injectée, comme indiqué dans [Journaliser des données de télémétrie personnalisées dans les fonctions C#](functions-dotnet-class-library.md?tabs=v2%2Ccmd#log-custom-telemetry-in-c-functions).

### <a name="iloggert-and-iloggerfactory"></a>ILogger<T> et ILoggerFactory

L’hôte injecte des services `ILogger<T>` et `ILoggerFactory` dans les constructeurs.  Toutefois, ces nouveaux filtres de journalisation sont par défaut retirés des journaux de fonction.  Vous devez modifier le fichier `host.json` pour accepter les filtres et les catégories supplémentaires.

L’exemple suivant montre comment ajouter un `ILogger<HttpTrigger>` avec les journaux qui sont exposés à l’hôte.

```csharp
namespace MyNamespace
{
    public class HttpTrigger
    {
        private readonly ILogger<HttpTrigger> _log;

        public HttpTrigger(ILogger<HttpTrigger> log)
        {
            _log = log;
        }

        [FunctionName("HttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req)
        {
            _log.LogInformation("C# HTTP trigger function processed a request.");

            // ...
    }
}
```

L’exemple suivant de fichier `host.json` ajoute le filtre de journal.

```json
{
    "version": "2.0",
    "logging": {
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            }
        },
        "logLevel": {
            "MyNamespace.HttpTrigger": "Information"
        }
    }
}
```

## <a name="function-app-provided-services"></a>Services fournis par Function App

L’hôte de la fonction inscrit de nombreux services. Les services suivants peuvent être injectés comme dépendances en toute sécurité dans votre application :

|Type de service|Durée de vie|Description|
|--|--|--|
|`Microsoft.Extensions.Configuration.IConfiguration`|Singleton|Configuration du runtime|
|`Microsoft.Azure.WebJobs.Host.Executors.IHostIdProvider`|Singleton|Chargé de fournir l’ID de l’instance d’hôte|

Si vous souhaitez voir d’autres services pris en charge par l’injection de dépendances, [créez un ticket et proposez-les sur GitHub](https://github.com/azure/azure-functions-host).

### <a name="overriding-host-services"></a>Remplacement des services de l’hôte

Le remplacement des services fournis par l’hôte n’est pas pris en charge actuellement.  Si vous souhaitez remplacer des services, [créez un ticket et proposez-les sur GitHub](https://github.com/azure/azure-functions-host).

## <a name="working-with-options-and-settings"></a>Utilisation d’options et de paramètres

Des valeurs définies dans les [paramètres de l’application](./functions-how-to-use-azure-function-app-settings.md#settings) sont disponibles dans une instance `IConfiguration`, ce qui vous permet de lire les valeurs de paramètres d’application dans la classe de démarrage.

Vous pouvez extraire des valeurs de l’instance `IConfiguration` dans un type personnalisé. La copie des valeurs de paramètres d’application dans un type personnalisé facilite le test de vos services en rendant ces valeurs injectables. Les paramètres lus dans l’instance de configuration doivent être des paires clé-valeur simples.

Considérons la classe suivante qui comprend une propriété nommée cohérente avec un paramètre d’application :

```csharp
public class MyOptions
{
    public string MyCustomSetting { get; set; }
}
```

Et un fichier `local.settings.json` qui pourrait structurer le paramètre personnalisé comme suit :
```json
{
  "IsEncrypted": false,
  "Values": {
    "MyOptions:MyCustomSetting": "Foobar"
  }
}
```

À partir de la méthode `Startup.Configure`, vous pouvez extraire des valeurs de l’instance `IConfiguration` dans votre type personnalisé à l’aide du code suivant :

```csharp
builder.Services.AddOptions<MyOptions>()
    .Configure<IConfiguration>((settings, configuration) =>
    {
        configuration.GetSection("MyOptions").Bind(settings);
    });
```

L’appel de `Bind` copie des valeurs dont les noms de propriété correspondent de la configuration vers l’instance personnalisée. L’instance options est désormais disponible dans le conteneur IoC pour injection dans une fonction.

L’objet d’options est injecté dans la fonction en tant qu’instance de l’ interface `IOptions` générique. Utilisez la propriété `Value` pour accéder aux valeurs figurant dans votre configuration.

```csharp
using System;
using Microsoft.Extensions.Options;

public class HttpTrigger
{
    private readonly MyOptions _settings;

    public HttpTrigger(IOptions<MyOptions> options)
    {
        _settings = options.Value;
    }
}
```

Pour plus d’informations sur l’utilisation des options, voir [Modèle Options dans ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

## <a name="customizing-configuration-sources"></a>Personnalisation des sources de configuration

> [!NOTE]
> La personnalisation de la source de configuration est disponible à partir des versions d’hôte 2.0.14192.0 et 3.0.14191.0 d’Azure Functions.

Pour spécifier des sources de configuration supplémentaires, remplacez la méthode `ConfigureAppConfiguration` dans la classe `StartUp` de votre application de fonction.

L’exemple suivant ajoute des valeurs de configuration à partir d’un fichier de paramètres d’application de base et d’un autre facultatif et spécifique à l’environnement.

```csharp
using System.IO;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            FunctionsHostBuilderContext context = builder.GetContext();

            builder.ConfigurationBuilder
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, "appsettings.json"), optional: true, reloadOnChange: false)
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, $"appsettings.{context.EnvironmentName}.json"), optional: true, reloadOnChange: false)
                .AddEnvironmentVariables();
        }
    }
}
```

Ajoutez des fournisseurs de configuration à la propriété `ConfigurationBuilder` de `IFunctionsConfigurationBuilder`. Pour plus d’informations sur l’utilisation de fournisseurs de configuration, consultez [Configuration dans ASP.NET Core](/aspnet/core/fundamentals/configuration/#configuration-providers).

`FunctionsHostBuilderContext` est obtenu à partir de `IFunctionsConfigurationBuilder.GetContext()`. Utilisez ce contexte pour récupérer le nom de l’environnement actuel et résoudre l’emplacement des fichiers de configuration dans le dossier de votre application de fonction.

Par défaut, les fichiers de configuration tels que *appsettings.json* ne sont pas automatiquement copiés dans le dossier de sortie de l’application de fonction. Mettez à jour votre fichier *.csproj* pour qu’il corresponde à l’exemple suivant afin de vous assurer que les fichiers sont bien copiés.

```xml
<None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>      
</None>
<None Update="appsettings.Development.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <CopyToPublishDirectory>Never</CopyToPublishDirectory>
</None>
```

> [!IMPORTANT]
> Pour les applications de fonction qui s’exécutent dans les plans Consommation ou Premium, les modifications apportées aux valeurs de configuration utilisées dans les déclencheurs peuvent entraîner des erreurs de mise à l’échelle. Toutes les modifications apportées à ces propriétés par la classe `FunctionsStartup` entraînent une erreur de démarrage de l’application de fonction.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

- [Surveiller votre application de fonction](functions-monitoring.md)
- [Meilleures pratiques pour Functions](functions-best-practices.md)
