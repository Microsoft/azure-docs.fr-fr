---
title: Azure Application Insights pour les applications console | Microsoft Docs
description: Surveiller la disponibilité, les performances et l’utilisation.des applications Web.
ms.topic: conceptual
ms.date: 05/21/2020
ms.custom: devx-track-csharp
ms.reviewer: lmolkova
ms.openlocfilehash: aa39a1eca04621fc4db75f755402d3679403e814
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96920597"
---
# <a name="application-insights-for-net-console-applications"></a>Application Insights pour les applications console .NET

[Application Insights](./app-insights-overview.md) vous permet de surveiller la disponibilité, les performances et l’utilisation de votre application web.

Vous avez besoin d’un abonnement à [Microsoft Azure](https://azure.com). Connectez-vous avec un compte Microsoft, que vous pouvez avoir pour Windows, Xbox Live ou d’autres services cloud de Microsoft. Si votre équipe dispose d’un abonnement d’organisation, demandez à son propriétaire d’y ajouter votre compte Microsoft.

> [!NOTE]
> Il est *vivement recommandé* d’utiliser le package de [Microsoft. ApplicationInsights. WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) et les instructions associées de [ici](./worker-service.md) pour toutes les applications console. Ce package cible [`NetStandard2.0`](/dotnet/standard/net-standard) et peut donc être utilisé dans .NET Core 2.1 ou version ultérieure, et dans .NET Framework 4.7.2 ou version ultérieure.

## <a name="getting-started"></a>Prise en main

> [!IMPORTANT]
> Les nouvelles régions Azure **exigent** l’utilisation de chaînes de connexion au lieu de clés d’instrumentation. Une [chaîne de connexion](./sdk-connection-string.md?tabs=net) identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Elle vous permet également de modifier les points de terminaison que votre ressource utilisera comme destination pour votre télémétrie. Vous devrez copier la chaîne de connexion et l’ajouter au code de votre application ou à une variable d’environnement.

* Dans le [portail Azure](https://portal.azure.com), [créez une ressource Application Insights](./create-new-resource.md). Choisissez **Général** comme type d’application.
* Copiez la clé d'instrumentation. Recherchez la clé dans la liste déroulante **Essentials** de la ressource créée.
* Installez le package [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) le plus récent.
* Définissez la clé d’instrumentation dans votre code avant d’effectuer le suivi de la télémétrie (ou définissez la variable d’environnement APPINSIGHTS_INSTRUMENTATIONKEY). Après cela, vous devriez pouvoir suivre manuellement les données de télémétrie et les visualiser dans le portail Azure.

```csharp
// you may use different options to create configuration as shown later in this article
TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
configuration.InstrumentationKey = " *your key* ";
var telemetryClient = new TelemetryClient(configuration);
telemetryClient.TrackTrace("Hello World!");
```

> [!NOTE]
> Les données de télémétrie ne sont pas envoyées instantanément. Les éléments de télémétrie sont traités par lot et envoyés par le SDK Application Insights. Dans les applications de console, qui se ferment juste après avoir appelé des méthodes `Track()`, la télémétrie ne peut pas être envoyée, sauf si `Flush()` et `Sleep`/`Delay` sont exécutés avant l’arrêt de l’application, comme le montre l’[exemple complet](#full-example) plus loin dans cet article. `Sleep` n’est pas requis si vous utilisez `InMemoryChannel`. Il existe un problème actif concernant le besoin de `Sleep` qui est suivi ici : [ApplicationInsights-dotnet/issues/407](https://github.com/microsoft/ApplicationInsights-dotnet/issues/407)


* Installez la version la plus récente du package [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) : il effectue automatiquement le suivi des appels SQL, HTTP ou autres appels de dépendance externe.

Vous pouvez initialiser et configurer Application Insights à partir du code ou à l’aide du fichier `ApplicationInsights.config`. Vérifiez que l’initialisation se produit dès que possible. 

> [!NOTE]
> Les instructions faisant référence à **ApplicationInsights.config** ne s’appliquent qu’aux applications qui ciblent .NET Framework, et non aux applications .NET Core.

### <a name="using-config-file"></a>Utilisation du fichier de configuration

Par défaut, le SDK Application Insights recherche le fichier `ApplicationInsights.config` dans le répertoire de travail lors de la création de `TelemetryConfiguration`.

```csharp
TelemetryConfiguration config = TelemetryConfiguration.Active; // Reads ApplicationInsights.config file if present
```

Vous pouvez également spécifier le chemin du fichier de configuration.

```csharp
using System.IO;
TelemetryConfiguration configuration = TelemetryConfiguration.CreateFromConfiguration(File.ReadAllText("C:\\ApplicationInsights.config"));
var telemetryClient = new TelemetryClient(configuration);
```

Pour plus d’informations, consultez la [référence du fichier de configuration](configuration-with-applicationinsights-config.md).

Vous pouvez obtenir un exemple complet du fichier de configuration en installant la version la plus récente du package [Microsoft.ApplicationInsights.WindowsServer](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer). Voici la configuration **minimale** pour la collecte de dépendances qui est équivalente à l’exemple de code.

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
  <InstrumentationKey>Your Key</InstrumentationKey>
  <TelemetryInitializers>
    <Add Type="Microsoft.ApplicationInsights.DependencyCollector.HttpDependenciesParsingTelemetryInitializer, Microsoft.AI.DependencyCollector"/>
  </TelemetryInitializers>
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule, Microsoft.AI.DependencyCollector">
      <ExcludeComponentCorrelationHttpHeadersOnDomains>
        <Add>core.windows.net</Add>
        <Add>core.chinacloudapi.cn</Add>
        <Add>core.cloudapi.de</Add>
        <Add>core.usgovcloudapi.net</Add>
        <Add>localhost</Add>
        <Add>127.0.0.1</Add>
      </ExcludeComponentCorrelationHttpHeadersOnDomains>
      <IncludeDiagnosticSourceActivities>
        <Add>Microsoft.Azure.ServiceBus</Add>
        <Add>Microsoft.Azure.EventHubs</Add>
      </IncludeDiagnosticSourceActivities>
    </Add>
  </TelemetryModules>
  <TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel, Microsoft.AI.ServerTelemetryChannel"/>
</ApplicationInsights>

```

### <a name="configuring-telemetry-collection-from-code"></a>Configuration de la collecte de données de télémétrie à partir du code
> [!NOTE]
> La lecture du fichier config n’est pas pris en charge sur .NET Core. Vous pouvez envisager d’utiliser [Application Insights SDK pour ASP.NET Core](./asp-net-core.md)

* Lors du démarrage de l’application, créez et configurez une instance de `DependencyTrackingTelemetryModule` : il doit s’agir d’un singleton, et elle doit être conservée pendant toute la durée de vie de l’application.

```csharp
var module = new DependencyTrackingTelemetryModule();

// prevent Correlation Id to be sent to certain endpoints. You may add other domains as needed.
module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.windows.net");
//...

// enable known dependency tracking, note that in future versions, we will extend this list. 
// please check default settings in https://github.com/Microsoft/ApplicationInsights-dotnet-server/blob/develop/Src/DependencyCollector/DependencyCollector/ApplicationInsights.config.install.xdt

module.IncludeDiagnosticSourceActivities.Add("Microsoft.Azure.ServiceBus");
module.IncludeDiagnosticSourceActivities.Add("Microsoft.Azure.EventHubs");
//....

// initialize the module
module.Initialize(configuration);
```

* Ajouter des initialiseurs de télémétrie communs

```csharp
// ensures proper DependencyTelemetry.Type is set for Azure RESTful API calls
configuration.TelemetryInitializers.Add(new HttpDependenciesParsingTelemetryInitializer());
```

Si vous avez créé une configuration avec un constructeur `TelemetryConfiguration()` simple, vous devez également activer la prise en charge des corrélations. **Ce n’est pas nécessaire** si vous lisez la configuration à partir du fichier, utilisé `TelemetryConfiguration.CreateDefault()` ou `TelemetryConfiguration.Active`.

```csharp
configuration.TelemetryInitializers.Add(new OperationCorrelationTelemetryInitializer());
```

* Vous pouvez également vouloir installer et initialiser le module collecteur Compteur de performances comme décrit [ici](https://apmtips.com/posts/2017-02-13-enable-application-insights-live-metrics-from-code/)


#### <a name="full-example"></a>Exemple complet

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DependencyCollector;
using Microsoft.ApplicationInsights.Extensibility;
using System.Net.Http;
using System.Threading.Tasks;

namespace ConsoleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();

            configuration.InstrumentationKey = "removed";
            configuration.TelemetryInitializers.Add(new HttpDependenciesParsingTelemetryInitializer());

            var telemetryClient = new TelemetryClient(configuration);
            using (InitializeDependencyTracking(configuration))
            {
                // run app...

                telemetryClient.TrackTrace("Hello World!");

                using (var httpClient = new HttpClient())
                {
                    // Http dependency is automatically tracked!
                    httpClient.GetAsync("https://microsoft.com").Wait();
                }

            }

            // before exit, flush the remaining data
            telemetryClient.Flush();

            // flush is not blocking when not using InMemoryChannel so wait a bit. There is an active issue regarding the need for `Sleep`/`Delay`
            // which is tracked here: https://github.com/microsoft/ApplicationInsights-dotnet/issues/407
            Task.Delay(5000).Wait();

        }

        static DependencyTrackingTelemetryModule InitializeDependencyTracking(TelemetryConfiguration configuration)
        {
            var module = new DependencyTrackingTelemetryModule();

            // prevent Correlation Id to be sent to certain endpoints. You may add other domains as needed.
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.windows.net");
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.chinacloudapi.cn");
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.cloudapi.de");
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.usgovcloudapi.net");
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("localhost");
            module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("127.0.0.1");

            // enable known dependency tracking, note that in future versions, we will extend this list. 
            // please check default settings in https://github.com/microsoft/ApplicationInsights-dotnet-server/blob/develop/WEB/Src/DependencyCollector/DependencyCollector/ApplicationInsights.config.install.xdt

            module.IncludeDiagnosticSourceActivities.Add("Microsoft.Azure.ServiceBus");
            module.IncludeDiagnosticSourceActivities.Add("Microsoft.Azure.EventHubs");

            // initialize the module
            module.Initialize(configuration);

            return module;
        }
    }
}

```

## <a name="next-steps"></a>Étapes suivantes
* [Surveillez les dépendances](./asp-net-dependencies.md) pour voir si des ressources REST, SQL ou d’autres ressources externes vous ralentissent.
* [Utilisez l’API](./api-custom-events-metrics.md) pour envoyer vos propres événements et mesures pour obtenir une vue plus détaillée des performances et de l’utilisation de votre application.

