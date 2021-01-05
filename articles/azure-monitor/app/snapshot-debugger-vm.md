---
title: Activer le Débogueur de capture instantanée pour les applications .NET dans Azure Service Fabric, le service cloud et les machines virtuelles | Microsoft Docs
description: Activer le Débogueur de capture instantanée pour les applications .NET dans Azure Service Fabric, le service cloud et les machines virtuelles
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/07/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 4bccc2922cf20262149ef54fbe2a1a821d9551ab
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673499"
---
# <a name="enable-snapshot-debugger-for-net-apps-in-azure-service-fabric-cloud-service-and-virtual-machines"></a>Activer le Débogueur de capture instantanée pour les applications .NET dans Azure Service Fabric, le service cloud et les machines virtuelles

Si votre application ASP.NET ou ASP.NET core est exécuté dans Azure App Service, il est vivement recommandé d’[activer le Débogueur de capture instantanée par le biais de la page du portail Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json). Toutefois, si votre application nécessite une configuration personnalisée de débogueur de capture instantanée ou une version préliminaire de .NET core, cette instruction doit être suivie ***en plus*** des instructions d’[activation via la page du portail Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json).

Si votre application s’exécute dans Azure Service Fabric, le service cloud, des machines virtuelles ou des machines locales, vous devez utiliser les instructions suivantes. 

## <a name="configure-snapshot-collection-for-aspnet-applications"></a>Configurer la collecte de captures instantanées pour les applications ASP.NET

1. Si vous ne l’avez pas encore fait, [Activez Application Insights dans votre application web](./asp-net.md).

2. Incluez le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) dans votre application.

3. Si nécessaire, personnalisez la configuration du Débogueur de capture instantanée ajoutée à [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). La configuration du Débogueur de capture instantanée par défaut est pratiquement vide et tous les paramètres sont facultatifs. Voici un exemple illustrant une configuration équivalente à la configuration par défaut :

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
        <!-- The default is true, but you can disable Snapshot Debugging by setting it to false -->
        <IsEnabled>true</IsEnabled>
        <!-- Snapshot Debugging is usually disabled in developer mode, but you can enable it by setting this to true. -->
        <!-- DeveloperMode is a property on the active TelemetryChannel. -->
        <IsEnabledInDeveloperMode>false</IsEnabledInDeveloperMode>
        <!-- How many times we need to see an exception before we ask for snapshots. -->
        <ThresholdForSnapshotting>1</ThresholdForSnapshotting>
        <!-- The maximum number of examples we create for a single problem. -->
        <MaximumSnapshotsRequired>3</MaximumSnapshotsRequired>
        <!-- The maximum number of problems that we can be tracking at any time. -->
        <MaximumCollectionPlanSize>50</MaximumCollectionPlanSize>
        <!-- How often we reconnect to the stamp. The default value is 15 minutes.-->
        <ReconnectInterval>00:15:00</ReconnectInterval>
        <!-- How often to reset problem counters. -->
        <ProblemCounterResetInterval>1.00:00:00</ProblemCounterResetInterval>
        <!-- The maximum number of snapshots allowed in ten minutes.The default value is 1. -->
        <SnapshotsPerTenMinutesLimit>3</SnapshotsPerTenMinutesLimit>
        <!-- The maximum number of snapshots allowed per day. -->
        <SnapshotsPerDayLimit>30</SnapshotsPerDayLimit>
        <!-- Whether or not to collect snapshot in low IO priority thread. The default value is true. -->
        <SnapshotInLowPriorityThread>true</SnapshotInLowPriorityThread>
        <!-- Agree to send anonymous data to Microsoft to make this product better. -->
        <ProvideAnonymousTelemetry>true</ProvideAnonymousTelemetry>
        <!-- The limit on the number of failed requests to request snapshots before the telemetry processor is disabled. -->
        <FailedRequestLimit>3</FailedRequestLimit>
        </Add>
    </TelemetryProcessors>
    ```

4. Des captures instantanées sont collectées uniquement sur des exceptions signalées à Application Insights. Dans certains cas (par exemple, des versions plus anciennes de la plateforme .NET), il se peut que vous deviez [configurer la collecte des exceptions](./asp-net-exceptions.md#exceptions) afin de voir celles-ci avec des captures instantanées dans le portail.


## <a name="configure-snapshot-collection-for-applications-using-aspnet-core-20-or-above"></a>Configurer la collecte de captures instantanées pour les applications utilisant ASP.NET Core 2.0 ou une version ultérieure

1. Si vous ne l’avez pas encore fait, [Activez Application Insights dans votre application web ASP.NET Core](./asp-net-core.md).

    > [!NOTE]
    > Être sûr que votre application fait référence à la version 2.1.1 ou plus récente, du package Microsoft.ApplicationInsights.AspNetCore.

2. Incluez le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) dans votre application.

3. Modifiez la classe `Startup` de votre application pour ajouter et configurer le processeur de télémétrie du collecteur de captures instantanées.
    1. Si le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) version 1.3.5 ou ultérieure est utilisée, ajoutez les instructions suivantes à `Startup.cs`.

       ```csharp
            using Microsoft.ApplicationInsights.SnapshotCollector;
       ```

       Ajoutez ce qui suit à la fin de la méthode ConfigureServices dans la classe `Startup` dans `Startup.cs`.

       ```csharp
            services.AddSnapshotCollector((configuration) => Configuration.Bind(nameof(SnapshotCollectorConfiguration), configuration));
       ```
    2. Si le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) version 1.3.4 ou antérieure est utilisée, ajoutez les instructions suivantes à `Startup.cs`.

       ```csharp
       using Microsoft.ApplicationInsights.SnapshotCollector;
       using Microsoft.Extensions.Options;
       using Microsoft.ApplicationInsights.AspNetCore;
       using Microsoft.ApplicationInsights.Extensibility;
       ```

       Ajoutez les paramètres suivants de la classe `SnapshotCollectorTelemetryProcessorFactory` à la classe `Startup`.

       ```csharp
       class Startup
       {
           private class SnapshotCollectorTelemetryProcessorFactory : ITelemetryProcessorFactory
           {
               private readonly IServiceProvider _serviceProvider;

               public SnapshotCollectorTelemetryProcessorFactory(IServiceProvider serviceProvider) =>
                   _serviceProvider = serviceProvider;

               public ITelemetryProcessor Create(ITelemetryProcessor next)
               {
                   var snapshotConfigurationOptions = _serviceProvider.GetService<IOptions<SnapshotCollectorConfiguration>>();
                   return new SnapshotCollectorTelemetryProcessor(next, configuration: snapshotConfigurationOptions.Value);
               }
           }
           ...
        ```
        Ajoutez les services `SnapshotCollectorConfiguration` et `SnapshotCollectorTelemetryProcessorFactory` au pipeline de start-up :

        ```csharp
           // This method gets called by the runtime. Use this method to add services to the container.
           public void ConfigureServices(IServiceCollection services)
           {
               // Configure SnapshotCollector from application settings
               services.Configure<SnapshotCollectorConfiguration>(Configuration.GetSection(nameof(SnapshotCollectorConfiguration)));

               // Add SnapshotCollector telemetry processor.
               services.AddSingleton<ITelemetryProcessorFactory>(sp => new SnapshotCollectorTelemetryProcessorFactory(sp));

               // TODO: Add other services your application needs here.
           }
       }
       ```

4. Si nécessaire, personnalisez la configuration du Débogueur de capture instantanée en ajoutant une section SnapshotCollectorConfiguration au fichier appsettings.json. Tous les paramètres de la configuration du Débogueur de capture instantanée sont facultatifs. Voici un exemple illustrant une configuration équivalente à la configuration par défaut :

   ```json
   {
     "SnapshotCollectorConfiguration": {
       "IsEnabledInDeveloperMode": false,
       "ThresholdForSnapshotting": 1,
       "MaximumSnapshotsRequired": 3,
       "MaximumCollectionPlanSize": 50,
       "ReconnectInterval": "00:15:00",
       "ProblemCounterResetInterval":"1.00:00:00",
       "SnapshotsPerTenMinutesLimit": 1,
       "SnapshotsPerDayLimit": 30,
       "SnapshotInLowPriorityThread": true,
       "ProvideAnonymousTelemetry": true,
       "FailedRequestLimit": 3
     }
   }
   ```

## <a name="configure-snapshot-collection-for-other-net-applications"></a>Configurer la collecte de captures instantanées pour d’autres applications .NET

1. Si votre application n’est pas instrumentée avec Application Insights, commencez par [activer Application Insights et définir la clé d’instrumentation](./windows-desktop.md).

2. Incluez le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) dans votre application.

3. Des captures instantanées sont collectées uniquement sur des exceptions signalées à Application Insights. Il se peut que vous deviez modifier votre code pour les signaler. Le code de gestion des exceptions dépend de la structure de votre application, mais en voici un exemple :
    ```csharp
   TelemetryClient _telemetryClient = new TelemetryClient();

   void ExampleRequest()
   {
        try
        {
            // TODO: Handle the request.
        }
        catch (Exception ex)
        {
            // Report the exception to Application Insights.
            _telemetryClient.TrackException(ex);

            // TODO: Rethrow the exception if desired.
        }
   }
    ```

## <a name="next-steps"></a>Étapes suivantes

- Générez du trafic vers votre application pouvant déclencher une exception. Attendez ensuite 10 à 15 minutes le temps que des captures instantanées soient envoyées à l’instance Application Insights.
- Consultez [Captures instantanées](snapshot-debugger.md?toc=/azure/azure-monitor/toc.json#view-snapshots-in-the-portal) dans le portail Azure.
- Pour obtenir de l’aide sur la résolution des problèmes liés au débogueur de capture instantanée, consultez [Résolution des problèmes liés au Débogueur de capture instantanée](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

