---
title: Application Insights pour les applications Service Worker (applications non HTTP)
description: Surveillance des applications .NET Core/.NET Framework non HTTP avec Azure Monitor Application Insights.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/11/2020
ms.openlocfilehash: 3d02b6e70d0832b92ae88db237b4c554b92e7f3b
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91875068"
---
# <a name="application-insights-for-worker-service-applications-non-http-applications"></a>Application Insights pour les applications Service Worker (applications non HTTP)

Le nouveau [kit de développement logiciel (SDK) Application Insights pour services Worker](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) est le mieux adapté aux charges de travail non HTTP, telles que la messagerie, les tâches en arrière-plan, les applications console, etc. À la différence d’une application web ASP.NET/ASP.NET Core traditionnelle, ces types d’applications ne peuvent pas gérer les requêtes HTTP entrantes. Par conséquent, l’utilisation de packages Application Insights pour les applications [ASP.NET](asp-net.md) ou [ASP.NET Core](asp-net-core.md) n’est pas prise en charge.

Le nouveau kit de développement logiciel (SDK) n’effectue pas de collecte de données de télémétrie par lui-même. Au lieu de cela, il intègre d’autres collecteurs automatiques Application Insights réputés, comme [DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/), [PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/), [ApplicationInsightsLoggingProvider](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights), etc. Ce kit de développement logiciel (SDK) expose des méthodes d’extension sur `IServiceCollection` pour activer et configurer la collecte de données de télémétrie.

## <a name="supported-scenarios"></a>Scénarios pris en charge

Le [SDK Application Insights pour Service Worker](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) est particulièrement bien adapté aux applications non HTTP, peu importe où et comment elles s’exécutent. Si votre application est en cours d’exécution et dispose d’une connectivité réseau vers Azure, les données de télémétrie peuvent être collectées. La surveillance Application Insights est prise en charge partout où .NET Core est pris en charge. Ce package peut être utilisé dans le nouveau [Service Worker .NET Core 3.0](https://devblogs.microsoft.com/aspnet/dotnet-core-workers-in-azure-container-instances), les [tâches en arrière-plan dans Asp.Net Core 2.1/2.2](/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.2&preserve-view=true), les applications console (.NET Core/ .NET Framework), etc.

## <a name="prerequisites"></a>Prérequis

Clé d’instrumentation Application Insights valide. Cette clé est requise pour envoyer les données de télémétrie à Application Insights. Si vous avez besoin créer une ressource Application Insights pour obtenir une instrumentation clé, consultez [Créer une ressource Application Insights](./create-new-resource.md).

## <a name="using-application-insights-sdk-for-worker-services"></a>Utilisation du kit de développement logiciel (SDK) Application Insights pour services Worker

1. Installez le package [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) dans l’application.
   L’extrait de code suivant montre les modifications qui doivent être ajoutées au fichier `.csproj` de votre projet.

```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.WorkerService" Version="2.13.1" />
    </ItemGroup>
```

1. Appelez la méthode d’extension `AddApplicationInsightsTelemetryWorkerService(string instrumentationKey)` sur `IServiceCollection`, en fournissant la clé d’instrumentation. Cette méthode doit être appelée au début de l’application. L’emplacement exact dépend du type d’application.

1. Récupérez une instance `ILogger` ou `TelemetryClient` à partir du conteneur Dependency Injection (DI) en appelant `serviceProvider.GetRequiredService<TelemetryClient>();` ou en utilisant Constructor Injection. Cette étape déclenchera la configuration de `TelemetryConfiguration` et des modules de collecte automatique.

Les instructions spécifiques pour chaque type d’application sont décrites dans les sections suivantes.

## <a name="net-core-30-worker-service-application"></a>Application de service Worker .NET Core 3.0

Un exemple complet est disponible [ici](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights)

1. Télécharger et installer [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0)
2. Créer un projet Service Worker à l’aide du nouveau modèle de projet ou de la ligne de commande Visual Studio `dotnet new worker`
3. Installez le package [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) dans l’application.

4. Ajoutez `services.AddApplicationInsightsTelemetryWorkerService();` à la méthode `CreateHostBuilder()` dans la classe `Program.cs`, comme dans cet exemple :

```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
                services.AddHostedService<Worker>();
                services.AddApplicationInsightsTelemetryWorkerService();
            });
```

5. Modifiez votre fichier `Worker.cs` comme dans l’exemple ci-dessous.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;
        private static HttpClient _httpClient = new HttpClient();

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
                {
                    _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                    _logger.LogInformation("Calling bing.com");
                    var res = await _httpClient.GetAsync("https://bing.com");
                    _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                    _telemetryClient.TrackEvent("Bing call event completed");
                }

                await Task.Delay(1000, stoppingToken);
            }
        }
    }
```

6. Configurez la clé d’instrumentation.

    Bien que vous puissiez fournir la clé d’instrumentation en tant qu’argument à `AddApplicationInsightsTelemetryWorkerService`, nous vous recommandons de la spécifier dans la configuration. L’exemple de code suivant montre comment spécifier une clé d’instrumentation dans `appsettings.json`. Assurez-vous que `appsettings.json` est copié dans le dossier racine de l’application lors de la publication.

```json
    {
        "ApplicationInsights":
            {
            "InstrumentationKey": "putinstrumentationkeyhere"
            },
        "Logging":
        {
            "LogLevel":
            {
                "Default": "Warning"
            }
        }
    }
```

Vous pouvez également spécifier la clé d’instrumentation dans une des variables d’environnement suivantes.
`APPINSIGHTS_INSTRUMENTATIONKEY` ou `ApplicationInsights:InstrumentationKey`

Par exemple : `SET ApplicationInsights:InstrumentationKey=putinstrumentationkeyhere`
OU `SET APPINSIGHTS_INSTRUMENTATIONKEY=putinstrumentationkeyhere`

En règle générale, `APPINSIGHTS_INSTRUMENTATIONKEY` spécifie la clé d’instrumentation pour les applications déployées vers Web App en tant que tâches web.

> [!NOTE]
> Une clé d’instrumentation spécifiée dans le code prévaut sur la variable d’environnement `APPINSIGHTS_INSTRUMENTATIONKEY`, qui prévaut sur les autres options.

## <a name="aspnet-core-background-tasks-with-hosted-services"></a>Tâches ASP.NET Core en arrière-plan avec services hébergés

[Ce](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio&view=aspnetcore-2.2&preserve-view=true) document décrit la façon de créer des tâches en arrière-plan dans une application ASP.NET Core 2.1/2.2.

Un exemple complet est disponible [ici](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/BackgroundTasksWithHostedService)

1. Installez le package [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) dans l’application.
2. Ajoutez `services.AddApplicationInsightsTelemetryWorkerService();` à la méthode `ConfigureServices()`, comme dans cet exemple :

```csharp
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .ConfigureAppConfiguration((hostContext, config) =>
            {
                config.AddJsonFile("appsettings.json", optional: true);
            })
            .ConfigureServices((hostContext, services) =>
            {
                services.AddLogging();
                services.AddHostedService<TimedHostedService>();

                // instrumentation key is read automatically from appsettings.json
                services.AddApplicationInsightsTelemetryWorkerService();
            })
            .UseConsoleLifetime()
            .Build();

        using (host)
        {
            // Start the host
            await host.StartAsync();

            // Wait for the host to shutdown
            await host.WaitForShutdownAsync();
        }
    }
```

Voici le code pour `TimedHostedService` où réside la logique de tâche en arrière-plan.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class TimedHostedService : IHostedService, IDisposable
    {
        private readonly ILogger _logger;
        private Timer _timer;
        private TelemetryClient _telemetryClient;
        private static HttpClient httpClient = new HttpClient();

        public TimedHostedService(ILogger<TimedHostedService> logger, TelemetryClient tc)
        {
            _logger = logger;
            this._telemetryClient = tc;
        }

        public Task StartAsync(CancellationToken cancellationToken)
        {
            _logger.LogInformation("Timed Background Service is starting.");

            _timer = new Timer(DoWork, null, TimeSpan.Zero,
                TimeSpan.FromSeconds(1));

            return Task.CompletedTask;
        }

        private void DoWork(object state)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

            using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
            {
                _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                _logger.LogInformation("Calling bing.com");
                var res = httpClient.GetAsync("https://bing.com").GetAwaiter().GetResult();
                _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                _telemetryClient.TrackEvent("Bing call event completed");
            }
        }
    }
```

3. Configurez la clé d’instrumentation.
   Utilisez le même `appsettings.json` que l’exemple de service Worker .NET Core 3.0 ci-dessus.

## <a name="net-corenet-framework-console-application"></a>Application console .NET Core /. NET Framework

Comme indiqué au début de cet article, le nouveau package peut être utilisé pour activer Application Insights Telemetry à partir d’une application de console normale. Ce package cible [`NetStandard2.0`](/dotnet/standard/net-standard) et peut donc être utilisé pour les applications console .NET Core 2.0 ou version ultérieure, et dans .NET Framework 4.7.2 ou version ultérieure.

Un exemple complet est disponible [ici](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/ConsoleAppWithApplicationInsights)

1. Installez le package [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) dans l’application.

2. Modifiez Program.cs comme dans l’exemple ci-après.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using System;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace WorkerSDKOnConsole
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                // Create the DI container.
                IServiceCollection services = new ServiceCollection();

                // Being a regular console app, there is no appsettings.json or configuration providers enabled by default.
                // Hence instrumentation key and any changes to default logging level must be specified here.
                services.AddLogging(loggingBuilder => loggingBuilder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>("Category", LogLevel.Information));
                services.AddApplicationInsightsTelemetryWorkerService("instrumentationkeyhere");

                // Build ServiceProvider.
                IServiceProvider serviceProvider = services.BuildServiceProvider();

                // Obtain logger instance from DI.
                ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

                // Obtain TelemetryClient instance from DI, for additional manual tracking or to flush.
                var telemetryClient = serviceProvider.GetRequiredService<TelemetryClient>();

                var httpClient = new HttpClient();

                while (true) // This app runs indefinitely. replace with actual application termination logic.
                {
                    logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                    // Replace with a name which makes sense for this operation.
                    using (telemetryClient.StartOperation<RequestTelemetry>("operation"))
                    {
                        logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                        logger.LogInformation("Calling bing.com");                    
                        var res = await httpClient.GetAsync("https://bing.com");
                        logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                        telemetryClient.TrackEvent("Bing call event completed");
                    }

                    await Task.Delay(1000);
                }

                // Explicitly call Flush() followed by sleep is required in Console Apps.
                // This is to ensure that even if application terminates, telemetry is sent to the back-end.
                telemetryClient.Flush();
                Task.Delay(5000).Wait();
            }
        }
    }
```

Cette application console utilise également la même valeur par défaut `TelemetryConfiguration`, et elle peut être personnalisée de la même façon que les exemples de la section précédente.

## <a name="run-your-application"></a>Exécuter votre application

Exécutez votre application. Les exemples de Worker ci-dessus appellent tous « bing.com » via une requête HTTP chaque seconde, et ils émettent également peu de journaux en utilisant `ILogger`. Ces lignes sont encapsulées à l’intérieur de l’appel `StartOperation` de `TelemetryClient`, qui est utilisé pour créer une opération (dans cet exemple `RequestTelemetry`, elle est nommée « operation »). Application Insights recueillera ces journaux ILogger (avertissement ou niveau d’alerte supérieur par défaut) et les dépendances, et ils seront corrélés à `RequestTelemetry` avec une relation parent-enfant. La corrélation fonctionne également entre les limites réseau et inter-processus. Par exemple, si l’appel a été effectué vers un autre composant surveillé, il est également corrélé à ce parent.

Cette opération personnalisée de `RequestTelemetry` peut être considérée comme l’équivalent d’une requête web entrante dans une application web classique. S’il n’est pas nécessaire d’utiliser une opération, celle-ci est parfaitement bien adaptée au [modèle de données de corrélation Application Insights](./correlation.md). `RequestTelemetry` agit en tant qu’opération parente, et les données de télémétrie générées à l’intérieur de l’itération du worker sont considérées comme appartenant logiquement à la même opération. Cette approche garantit également que toutes les données de télémétrie produites (automatiques et manuelles) posséderont le même `operation_id`. Comme l’échantillonnage est basé sur `operation_id`, l’algorithme d’échantillonnage conserve ou supprime toutes les données de télémétrie d’une même itération.

La liste suivante répertorie les données de télémétrie complètes collectées automatiquement par Application Insights.

### <a name="live-metrics"></a>Métriques temps réel

Les [métriques temps réel](./live-stream.md) peuvent être utilisées pour vérifier rapidement si la supervision Application Insights est correctement configurée. Quelques minutes peuvent être nécessaires avant l'apparition des données de télémétrie sur le portail et dans les analyses, mais les métriques temps réel affichent en temps quasi réel l'utilisation UC du processus en cours. Elles peuvent également afficher d’autres données de télémétrie comme les requêtes, les dépendances, les traces, etc.

### <a name="ilogger-logs"></a>Journaux ILogger

Les journaux émis via `ILogger` de gravité `Warning` ou supérieure sont automatiquement capturés. Suivez les [docs ILogger](ilogger.md#control-logging-level) pour personnaliser les niveaux de journalisation capturés par Application Insights.

### <a name="dependencies"></a>Les dépendances

La collecte des dépendances est activée par défaut. [Cet article](asp-net-dependencies.md#automatically-tracked-dependencies) décrit les dépendances qui sont automatiquement collectées, et présentent les étapes permettant d'effectuer un suivi manuel.

### <a name="eventcounter"></a>EventCounter

`EventCounterCollectionModule` est activé par défaut, et il collectera un ensemble de compteurs par défaut à partir des applications .NET Core 3.0. Le didacticiel [​​EventCounter](eventcounters.md) répertorie l'ensemble par défaut de compteurs collectés. Il contient également des instructions sur la personnalisation de la liste.

### <a name="manually-tracking-additional-telemetry"></a>Suivi manuel des données de télémétrie supplémentaires

Le kit de développement logiciel (SDK) collecte automatiquement les données de télémétrie comme expliqué ci-dessus ; dans la plupart des cas, l’utilisateur doit envoyer des données de télémétrie supplémentaires à Application Insights service. La méthode recommandée pour le suivi des données de télémétrie supplémentaires consiste à obtenir une instance de `TelemetryClient` auprès de l'injection de dépendance, puis à appeler l'une des méthodes de `TrackXXX()` [l'API](api-custom-events-metrics.md) prises en charge sur cette instance. Le [suivi personnalisé des opérations](custom-operations-tracking.md) est un autre cas d’utilisation classique de cette approche. Celle-ci est illustrée dans les exemples Worker ci-dessus.

## <a name="configure-the-application-insights-sdk"></a>Configurer le SDK Application Insights

La valeur par défaut `TelemetryConfiguration` utilisée par le kit de développement logiciel (SDK) du service Worker est similaire à la configuration automatique utilisée dans une application ASP.NET ou ASP.NET Core, moins les TelemetryInitializers utilisés pour enrichir les données de télémétrie de `HttpContext`.

Vous pouvez personnaliser le SDK Application Insights pour Service Worker afin de modifier la configuration par défaut. Les utilisateurs du SDK Application Insights ASP.NET Core peuvent maîtriser la modification de configuration à l’aide de l’[injection de dépendances](/aspnet/core/fundamentals/dependency-injection) intégrée à ASP.NET Core. Le kit de développement logiciel (SDK) WorkerService est également basé sur des principes similaires. Effectuez pratiquement toutes les modifications de configuration dans la section `ConfigureServices()` en appelant les méthodes appropriées sur `IServiceCollection`, comme indiqué ci-dessous.

> [!NOTE]
> Quand vous utilisez ce kit de développement logiciel (SDK), il n’est pas possible de modifier la configuration à l’aide de `TelemetryConfiguration.Active`, et les modifications réalisées par ce biais ne sont pas répercutées.

### <a name="using-applicationinsightsserviceoptions"></a>Utilisation d’ApplicationInsightsServiceOptions

Vous pouvez modifier quelques paramètres courants en passant de `ApplicationInsightsServiceOptions` à `AddApplicationInsightsTelemetryWorkerService`, comme dans cet exemple :

```csharp
using Microsoft.ApplicationInsights.WorkerService;

public void ConfigureServices(IServiceCollection services)
{
    var aiOptions = new ApplicationInsightsServiceOptions();
    // Disables adaptive sampling.
    aiOptions.EnableAdaptiveSampling = false;

    // Disables QuickPulse (Live Metrics stream).
    aiOptions.EnableQuickPulseMetricStream = false;
    services.AddApplicationInsightsTelemetryWorkerService(aiOptions);
}
```

Notez que `ApplicationInsightsServiceOptions` dans ce kit de développement logiciel (SDK) se trouve dans l’espace de noms `Microsoft.ApplicationInsights.WorkerService` au lieu de `Microsoft.ApplicationInsights.AspNetCore.Extensions` dans le SDK ASP.NET Core.

Paramètres couramment utilisés dans `ApplicationInsightsServiceOptions`

|Paramètre | Description | Default
|---------------|-------|-------
|EnableQuickPulseMetricStream | Active/désactive la fonctionnalité LiveMetrics | true
|EnableAdaptiveSampling | Active/désactive l’échantillonnage adaptatif | true
|EnableHeartbeat | Active/désactive la fonctionnalité des pulsations, qui envoie régulièrement (toutes les 15 minutes, par défaut) une métrique personnalisée nommée « HeartBeatState » avec des informations sur le runtime, telles que la version de .NET, des informations relatives à l'environnement Azure, le cas échéant, etc. | true
|AddAutoCollectedMetricExtractor | Active/désactive l’extracteur AutoCollectedMetrics, qui est un TelemetryProcessor qui envoie des métriques pré-agrégées sur les demandes/dépendances avant l’échantillonnage. | true
|EnableDiagnosticsTelemetryModule | Active/désactive `DiagnosticsTelemetryModule`. La désactivation de cet élément entraîne l’abandon des paramètres suivants : `EnableHeartbeat`, `EnableAzureInstanceMetadataTelemetryModule`, `EnableAppServicesHeartbeatTelemetryModule`. | true

Consultez les [paramètres configurables dans `ApplicationInsightsServiceOptions`](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/NETCORE/src/Shared/Extensions/ApplicationInsightsServiceOptions.cs) pour obtenir la liste la plus à jour.

### <a name="sampling"></a>échantillonnage

Le SDK Application Insights pour Service Worker prend en charge l’échantillonnage à taux fixe et adaptatif. L’échantillonnage adaptatif est activé par défaut. L’échantillonnage peut être désactivé à l’aide de l’option `EnableAdaptiveSampling` qui se trouve dans [ApplicationInsightsServiceOptions](#using-applicationinsightsserviceoptions).

Pour configurer des paramètres d’échantillonnage supplémentaires, vous pouvez utiliser l’exemple suivant.

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WorkerService;

public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new ApplicationInsightsServiceOptions();
    
    // Disable adaptive sampling.
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetryWorkerService(aiOptions);

    // Add Adaptive Sampling with custom settings.
    // the following adds adaptive sampling with 15 items per sec.
    services.Configure<TelemetryConfiguration>((telemetryConfig) =>
        {
            var builder = telemetryConfig.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
            builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond: 15);
            builder.Build();
        });
    //...
}
```

Vous trouverez des informations supplémentaires dans le document [Échantillonnage](#sampling).

### <a name="adding-telemetryinitializers"></a>Ajout de TelemetryInitializers

Utilisez des [initialiseurs de télémétrie](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) lorsque vous souhaitez définir des propriétés qui sont envoyées avec toutes les données de télémétrie.

Ajoutez un nouveau `TelemetryInitializer` au conteneur `DependencyInjection` et le kit de développement logiciel (SDK) les ajoutera automatiquement au `TelemetryConfiguration`.

```csharp
    using Microsoft.ApplicationInsights.Extensibility;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="removing-telemetryinitializers"></a>Suppression de TelemetryInitializers

Les initialiseurs de télémétrie sont présents par défaut. Pour supprimer tout ou partie des initialiseurs de télémétrie, utilisez l’exemple de code suivant *après* avoir appelé `AddApplicationInsightsTelemetryWorkerService()`.

```csharp
   public void ConfigureServices(IServiceCollection services)
   {
        services.AddApplicationInsightsTelemetryWorkerService();
        // Remove a specific built-in telemetry initializer
        var tiToRemove = services.FirstOrDefault<ServiceDescriptor>
                            (t => t.ImplementationType == typeof(AspNetCoreEnvironmentTelemetryInitializer));
        if (tiToRemove != null)
        {
            services.Remove(tiToRemove);
        }

        // Remove all initializers
        // This requires importing namespace by using Microsoft.Extensions.DependencyInjection.Extensions;
        services.RemoveAll(typeof(ITelemetryInitializer));
   }
```

### <a name="adding-telemetry-processors"></a>Ajout de processeurs de télémétrie

Vous pouvez ajouter des processeurs de télémétrie personnalisés à `TelemetryConfiguration` à l’aide de la méthode d’extension `AddApplicationInsightsTelemetryProcessor` sur `IServiceCollection`. Vous utilisez des processeurs de télémétrie dans des [scénarios de filtrage avancés](./api-filtering-sampling.md#itelemetryprocessor-and-itelemetryinitializer) pour permettre un contrôle plus direct sur les informations contenues ou non dans le données de télémétrie que vous envoyez au service Application Insights. Utilisez l’exemple suivant.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
        services.AddApplicationInsightsTelemetryProcessor<MyFirstCustomTelemetryProcessor>();
        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<MySecondCustomTelemetryProcessor>();
    }
```

### <a name="configuring-or-removing-default-telemetrymodules"></a>Configuration ou suppression de l’élément TelemetryModules par défaut

Application Insights utilise les modules de télémétrie pour collecter automatiquement des informations de télémétrie sur des charges de travail spécifiques sans qu’un suivi manuel ne soit nécessaire.

Les modules de collecte automatique suivants sont activés par défaut. Ces modules sont chargés de collecter automatiquement les données de télémétrie. Vous pouvez les désactiver ou les configurer pour modifier leur comportement par défaut.

* `DependencyTrackingTelemetryModule`
* `PerformanceCollectorModule`
* `QuickPulseTelemetryModule`
* `AppServicesHeartbeatTelemetryModule` -(Il existe actuellement un problème impliquant ce module de télémétrie. Pour une solution de contournement temporaire, voir le [problème GitHub 1689](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1689
).)
* `AzureInstanceMetadataTelemetryModule`

Pour configurer tout `TelemetryModule` par défaut, utilisez la méthode d’extension `ConfigureTelemetryModule<T>` sur `IServiceCollection`, comme illustré dans l’exemple suivant.

```csharp
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();

            // The following configures QuickPulseTelemetryModule.
            // Similarly, any other default modules can be configured.
            services.ConfigureTelemetryModule<QuickPulseTelemetryModule>((module, o) =>
            {
                module.AuthenticationApiKey = "keyhere";
            });

            // The following removes PerformanceCollectorModule to disable perf-counter collection.
            // Similarly, any other default modules can be removed.
            var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>
                                        (t => t.ImplementationType == typeof(PerformanceCollectorModule));
            if (performanceCounterService != null)
            {
                services.Remove(performanceCounterService);
            }
    }
```

### <a name="configuring-telemetry-channel"></a>Configuration d’un canal de télémétrie

Le canal par défaut est `ServerTelemetryChannel`. Vous pouvez le remplacer, comme le montre l’exemple suivant.

```csharp
using Microsoft.ApplicationInsights.Channel;

    public void ConfigureServices(IServiceCollection services)
    {
        // Use the following to replace the default channel with InMemoryChannel.
        // This can also be applied to ServerTelemetryChannel.
        services.AddSingleton(typeof(ITelemetryChannel), new InMemoryChannel() {MaxTelemetryBufferCapacity = 19898 });

        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="disable-telemetry-dynamically"></a>Désactiver la télémétrie dynamiquement

Si vous souhaitez désactiver la télémétrie de manière conditionnelle et dynamique, vous pouvez résoudre l’instance `TelemetryConfiguration` avec le conteneur d’injection de dépendance ASP.net n’importe où dans votre code et définir l’indicateur `DisableTelemetry` sur celle-ci.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
    {
        configuration.DisableTelemetry = true;
        ...
    }
```

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-can-i-track-telemetry-thats-not-automatically-collected"></a>Comment puis-je suivre les données de télémétrie qui ne sont pas automatiquement collectées ?

Obtenez une instance de `TelemetryClient` à l’aide de l’injection de constructeur et appelez la méthode `TrackXXX()` requise dessus. Nous vous déconseillons de créer de nouvelles instances `TelemetryClient`. Une instance singleton de `TelemetryClient` est déjà inscrite dans le conteneur `DependencyInjection`, qui partage `TelemetryConfiguration` avec le reste de la télémétrie. La création d’une instance `TelemetryClient` est recommandée uniquement si celle-ci a besoin d’une configuration distincte du reste de la télémétrie.

### <a name="can-i-use-visual-studio-ide-to-onboard-application-insights-to-a-worker-service-project"></a>Puis-je utiliser l’IDE de Visual Studio pour intégrer Application Insights à un projet de service Worker ?

L’intégration de l’IDE Visual Studio est actuellement prise en charge uniquement pour les applications ASP.NET/ASP.NET Core. Ce document sera mis à jour lorsque Visual Studio prendra en charge l’intégration d’applications de service Worker.

### <a name="can-i-enable-application-insights-monitoring-by-using-tools-like-status-monitor"></a>Puis-je activer la surveillance Application Insights à l’aide d’outils tels que Status Monitor ?

Non. [Status Monitor](./monitor-performance-live-website-now.md) et [Status Monitor version 2](./status-monitor-v2-overview.md) prennent actuellement en charge ASP.NET 4.x uniquement.

### <a name="if-i-run-my-application-in-linux-are-all-features-supported"></a>Si j’exécute mon application sous Linux, toutes les fonctionnalités sont-elles prises en charge ?

Oui. La prise en charge de fonctionnalités pour ce SDK est la même sur toutes les plateformes, avec les exceptions suivantes :

* Les compteurs de performance ne sont pris en charge que sous Windows, à l’exception de Process CPU/Memory (UC/Mémoire de processus) affiché dans Métriques temps réel.
* Même si `ServerTelemetryChannel` est activé par défaut, si l’application est exécutée sous Linux ou MacOS, le canal ne crée pas automatiquement un dossier de stockage local pour conserver temporairement les données de télémétrie s’il existe des problèmes de réseau. En raison de cette limitation, les données de télémétrie sont perdues en cas de problèmes temporaires de réseau ou de serveur. Pour contourner ce problème, configurez un dossier local pour le canal :

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    public void ConfigureServices(IServiceCollection services)
    {
        // The following will configure the channel to use the given folder to temporarily
        // store telemetry items during network or Application Insights server issues.
        // User should ensure that the given folder already exists
        // and that the application has read/write permissions.
        services.AddSingleton(typeof(ITelemetryChannel),
                                new ServerTelemetryChannel () {StorageFolder = "/tmp/myfolder"});
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

## <a name="sample-applications"></a>Exemples d'applications

[Application console .NET Core](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/ConsoleAppWithApplicationInsights) Choisissez cet exemple si vous utilisez une application console écrite en .NET Core (version 2.0 ou ultérieure) ou .NET Framework (version 4.7.2 ou ultérieure)

[Tâches en arrière-plan ASP .NET Core avec HostedServices](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/BackgroundTasksWithHostedService) Choisissez cet exemple si vous utilisez Asp.Net Core 2.1/2.2 et créez des tâches en arrière-plan conformément aux instructions officielles stipulées [ici](/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.2&preserve-view=true)

[Service Worker .NET Core 3.0](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights) Utilisez cet exemple si vous disposez d’une application Service Worker .NET Core 3.0, conformément aux instructions officielles ([ici](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio&view=aspnetcore-3.0&preserve-view=true#worker-service-template))

## <a name="open-source-sdk"></a>Kit de développement logiciel (SDK) open source

* [Lisez et contribuez au code](https://github.com/microsoft/ApplicationInsights-dotnet).

Pour obtenir les mises à jour et correctifs de bogues les plus récents, [consultez les notes de publication](./release-notes.md).

## <a name="next-steps"></a>Étapes suivantes

* [Utilisez l’API](./api-custom-events-metrics.md) pour envoyer vos propres événements et mesures pour obtenir une vue détaillée des performances et de l’utilisation de votre application.
* [Suivre les dépendances supplémentaires non suivies automatiquement](./auto-collect-dependencies.md).
* [Enrichir ou filtrer les données de télémétrie collectées automatiquement](./api-filtering-sampling.md).
* [Injection de dépendances dans ASP.NET Core](/aspnet/core/fundamentals/dependency-injection).
