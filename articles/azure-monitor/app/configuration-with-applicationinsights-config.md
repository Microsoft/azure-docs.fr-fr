---
title: Référence sur ApplicationInsights.config - Azure | Microsoft Docs
description: Activez ou désactivez les modules de collecte de données et ajoutez des compteurs de performances et d’autres paramètres.
ms.topic: conceptual
ms.date: 05/22/2019
ms.custom: devx-track-csharp
ms.reviewer: olegan
ms.openlocfilehash: 7c0759e78b1adc1704acb602daa12cf9cabbe153
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934801"
---
# <a name="configuring-the-application-insights-sdk-with-applicationinsightsconfig-or-xml"></a>Configuration du kit de développement logiciel (SDK) Application Insights à l’aide du fichier ApplicationInsights.config ou .xml
Le kit de développement logiciel (SDK) .NET Application Insights se compose d’un certain nombre de packages NuGet. Le [package principal](https://www.nuget.org/packages/Microsoft.ApplicationInsights) fournit l'API pour l'envoi des données télémétriques à Application Insights. Des [packages supplémentaires](https://www.nuget.org/packages?q=Microsoft.ApplicationInsights) fournissent les *modules* et les *initialiseurs* de télémétrie pour le suivi télémétrique automatique de votre application et de son contexte. La modification du fichier config permet d’activer ou de désactiver les modules et initialiseurs de télémétrie, et de définir les paramètres pour certains d’entre eux.

Le fichier de configuration est nommé `ApplicationInsights.config` ou `ApplicationInsights.xml`, selon le type de votre application. Il est automatiquement ajouté à votre projet lorsque vous [installez la plupart des versions du kit de développement logiciel (SDK)][start]. Par défaut, quand vous utilisez l’expérience automatisée des projets de modèle Visual Studio qui prennent en charge **Ajouter > Application Insights Telemetry**, le fichier ApplicationInsights.config est créé dans le dossier racine du projet et, quand il est compilé, il est copié dans le dossier Bin. Il est également ajouté à une application web par [Status Monitor sur un serveur IIS][redfield]. Le fichier de configuration est ignoré si l'[extension pour le site web Azure](azure-web-apps.md) ou l'[extension pour la machine virtuelle Azure et le groupe de machines virtuelles identiques](azure-vm-vmss-apps.md) est utilisée.

Il n’existe aucun fichier équivalent permettant de contrôler le [kit de développement logiciel (SDK) dans une page web][client].

Ce document décrit les sections du fichier de configuration, la façon dont ils contrôlent les composants du Kit de développement logiciel (SDK) et les packages NuGet qui chargent ces composants.

> [!NOTE]
> Les instructions ApplicationInsights.config et ApplicationInsights.xml ne s’appliquent pas au kit SDK .NET Core. Pour configurer les applications .NET Core, suivez [ce guide](./asp-net-core.md).

## <a name="telemetry-modules-aspnet"></a>Modules de télémétrie (ASP.NET)
Chaque module de télémétrie collecte un type de données précis et utilise l’API de base pour envoyer les données. Les modules sont installés par différents packages NuGet, qui ajoutent également les lignes requises dans le fichier .config.

Il existe un nœud dans le fichier de configuration pour chaque module. Pour désactiver un module, supprimez le nœud ou commentez-le.

### <a name="dependency-tracking"></a>Suivi des dépendances
[Dependency tracking](./asp-net-dependencies.md) collecte la télémétrie des appels de votre application aux bases de données et aux services et bases de données externes. Pour permettre à ce module de fonctionner dans un serveur IIS, vous devez [installer Status Monitor][redfield].

Vous pouvez également écrire votre propre code de suivi des dépendances à l'aide de l’ [API TrackDependency](./api-custom-events-metrics.md#trackdependency).

* `Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) .

Les dépendances peuvent être collectées automatiquement, sans modification de votre code, à l’aide de la jonction (sans code) basée sur l'agent. Pour l’utiliser dans des applications web Azure, activez l’[extension Application Insights](azure-web-apps.md). Pour l’utiliser dans une machine virtuelle Azure ou un groupe de machines virtuelles identiques Azure, activez l’[extension Surveillance des applications pour machine virtuelle et groupe de machines virtuelles identiques](azure-vm-vmss-apps.md).

### <a name="performance-collector"></a>Collecteur de performances
[Collecte les compteurs de performances système](./performance-counters.md), notamment l’UC, la mémoire et la charge réseau, à partir des installations IIS. Vous pouvez spécifier les compteurs à collecter, y compris les compteurs de performances que vous avez configurés vous-même.

* `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule`
* [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector) .

### <a name="application-insights-diagnostics-telemetry"></a>Télémétrie des diagnostics Application Insights
Le `DiagnosticsTelemetryModule` répertorie les erreurs dans le code d'instrumentation Application Insights lui-même. Par exemple, si le code ne peut pas accéder aux compteurs de performances ou si un `ITelemetryInitializer` renvoie une exception. La télémétrie de trace suivie par ce module s’affiche dans [Recherche de diagnostic][diagnostic].

```
* `Microsoft.ApplicationInsights.Extensibility.Implementation.Tracing.DiagnosticsTelemetryModule`
* [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) NuGet package. If you only install this package, the ApplicationInsights.config file is not automatically created.
```

### <a name="developer-mode"></a>Mode développeur
`DeveloperModeWithDebuggerAttachedTelemetryModule` force Application Insights `TelemetryChannel` à envoyer des données immédiatement, un élément de télémétrie à la fois, lorsqu’un débogueur est associé au processus d'application. Cela réduit le délai entre le moment où votre application effectue le suivi de télémétrie et celui où les données apparaissent sur le portail Application Insights. Cela cause une surcharge importante au niveau de l'UC et de la bande passante réseau.

* `Microsoft.ApplicationInsights.WindowsServer.DeveloperModeWithDebuggerAttachedTelemetryModule`
* [Application Insights Windows Server](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/)

### <a name="web-request-tracking"></a>Suivi des requêtes web
Indique le [temps de réponse et le code résultant](../../azure-monitor/app/asp-net.md) des requêtes HTTP.

* `Microsoft.ApplicationInsights.Web.RequestTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web)

### <a name="exception-tracking"></a>Suivi des exceptions
`ExceptionTrackingTelemetryModule` comptabilise le nombre d’exceptions non traitées dans votre application web. Consultez [Échecs et exceptions][exceptions].

* `Microsoft.ApplicationInsights.Web.ExceptionTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web)
* `Microsoft.ApplicationInsights.WindowsServer.UnobservedExceptionTelemetryModule` - suit les exceptions de tâche non prises en charge
* `Microsoft.ApplicationInsights.WindowsServer.UnhandledExceptionTelemetryModule` - suit les exceptions non gérées pour les rôles de travail, les services Windows et les applications de console.
* [Application Insights Windows Server](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/) .

### <a name="eventsource-tracking"></a>Suivi EventSource
`EventSourceTelemetryModule` vous permet de configurer les événements EventSource à envoyer à Application Insights en tant que traces. Pour plus d’informations sur le suivi des événements EventSource, consultez [Utilisation d’événements EventSource](./asp-net-trace-logs.md#use-eventsource-events).

* `Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule`
* [Microsoft.ApplicationInsights.EventSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener) 

### <a name="etw-event-tracking"></a>Suivi des événements ETW
`EtwCollectorTelemetryModule` vous permet de configurer les événements provenant des fournisseurs ETW à envoyer à Application Insights en tant que traces. Pour plus d’informations sur le suivi des événements ETW, consultez [Utilisation d’événements ETW](../../azure-monitor/app/asp-net-trace-logs.md#use-etw-events).

* `Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule`
* [Microsoft.ApplicationInsights.EtwCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector) 

### <a name="microsoftapplicationinsights"></a>Microsoft.ApplicationInsights
Le package Microsoft.ApplicationInsights fournit l’[API de base](/dotnet/api/microsoft.applicationinsights?view=azure-dotnet) du Kit de développement logiciel (SDK). Les autres modules de télémétrie l’utilisent. Vous pouvez également [l’utiliser pour définir votre propre télémétrie](./api-custom-events-metrics.md).

* Aucune entrée dans ApplicationInsights.config.
* [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) . Si vous installez simplement ce package NuGet, aucun fichier .config n'est créé.

## <a name="telemetry-channel"></a>Canal de télémétrie
Le [canal de télémétrie](telemetry-channels.md) gère la mise en mémoire tampon et la transmission de la télémétrie au service Application Insights.

* `Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel` est le canal par défaut des applications web. Il met les données en mémoire tampon, et emploie des mécanismes de nouvelle tentative et de stockage de disque local pour fournir des données de télémétrie plus fiables.
* `Microsoft.ApplicationInsights.InMemoryChannel` est un canal de télémétrie léger, qui est utilisé si aucun autre canal n’est configuré. 

## <a name="telemetry-initializers-aspnet"></a>Initialiseurs de télémétrie (ASP.NET)
Les initialiseurs de télémétrie définissent les propriétés de contexte envoyées avec chaque élément de télémétrie.

Vous pouvez [écrire vos propres initialiseurs](./api-filtering-sampling.md#add-properties) pour définir les propriétés de contexte.

Les initialiseurs standard sont tous définis par les packages NuGet web ou WindowsServer :

* `AccountIdTelemetryInitializer` définit la propriété AccountId.
* `AuthenticatedUserIdTelemetryInitializer` définit la propriété AuthenticatedUserId déterminée par le Kit de développement logiciel (SDK) JavaScript.
* `AzureRoleEnvironmentTelemetryInitializer` met à jour les propriétés `RoleName` et `RoleInstance` du contexte `Device` pour tous les éléments de télémétrie avec des informations extraites de l'environnement d’exécution Azure.
* `BuildInfoConfigComponentVersionTelemetryInitializer` met à jour la propriété `Version` du contexte `Component` pour tous les éléments de télémétrie avec la valeur extraite du fichier `BuildInfo.config` produit par MSBuild.
* `ClientIpHeaderTelemetryInitializer` met à jour la propriété `Ip` du contexte `Location` de tous les éléments de télémétrie à partir de l’en-tête HTTP `X-Forwarded-For` de la demande.
* `DeviceTelemetryInitializer` met à jour les propriétés suivantes du contexte `Device` pour tous les éléments de télémétrie.
  * `Type` est défini sur « PC »
  * `Id` est défini sur le nom de domaine de l'ordinateur sur lequel l'application web est exécutée.
  * `OemName` est défini sur la valeur extraite du champ `Win32_ComputerSystem.Manufacturer` à l'aide de WMI.
  * `Model` est défini sur la valeur extraite du champ `Win32_ComputerSystem.Model` à l'aide de WMI.
  * `NetworkType` est défini sur la valeur extraite de l’`NetworkInterface`.
  * `Language` est défini sur le nom de `CurrentCulture`.
* `DomainNameRoleInstanceTelemetryInitializer` met à jour la propriété `RoleInstance` du contexte `Device` pour tous les éléments de télémétrie avec le nom de domaine de l'ordinateur sur lequel l'application web est exécutée.
* `OperationNameTelemetryInitializer` met à jour la propriété `Name` de `RequestTelemetry` et la propriété `Name` du contexte `Operation` de tous les éléments de télémétrie à partir de la méthode HTTP, ainsi que les noms du contrôleur ASP.NET MVC et de l’action appelée pour traiter la demande.
* `OperationIdTelemetryInitializer` ou `OperationCorrelationTelemetryInitializer` met à jour la propriété de contexte `Operation.Id` de tous les éléments de télémétrie suivis lors du traitement d'une demande avec le `RequestTelemetry.Id` généré automatiquement.
* `SessionTelemetryInitializer` met à jour la propriété `Id` du contexte `Session` pour tous les éléments de télémétrie avec la valeur extraite du cookie `ai_session` généré par le code d’instrumentation JavaScript Application Insights en cours d'exécution dans le navigateur de l'utilisateur.
* `SyntheticTelemetryInitializer` ou `SyntheticUserAgentTelemetryInitializer` met à jour les propriétés de contexte `User`, `Session` et `Operation` de tous les éléments de télémétrie suivis lors du traitement d'une requête émanant d'une source synthétique, comme un test de disponibilité ou un robot de moteur de recherche. Par défaut, [Metrics Explorer](../platform/metrics-charts.md) n'affiche pas la télémétrie synthétique.

    Ensemble de `<Filters>` qui identifie les propriétés des requêtes.
* `UserTelemetryInitializer` met à jour les propriétés `Id` et `AcquisitionDate` du contexte `User` pour tous les éléments de télémétrie avec les valeurs extraites du cookie `ai_user` généré par le code d’instrumentation JavaScript Application Insights en cours d'exécution dans le navigateur de l'utilisateur.
* `WebTestTelemetryInitializer` définit l’ID utilisateur, l’ID de session et les propriétés de la source de synthèse pour des requêtes HTTP provenant des [tests de disponibilité](./monitor-web-app-availability.md).
  Ensemble de `<Filters>` qui identifie les propriétés des requêtes.

Pour les applications .NET en cours d’exécution dans Service Fabric, vous pouvez inclure le package NuGet `Microsoft.ApplicationInsights.ServiceFabric`. Ce package comprend un `FabricTelemetryInitializer`, qui ajoute des propriétés Service Fabric pour les éléments de télémétrie. Pour plus d’informations, consultez la [page GitHub](https://github.com/Microsoft/ApplicationInsights-ServiceFabric/blob/master/README.md) sur les propriétés ajoutées par ce package NuGet.

## <a name="telemetry-processors-aspnet"></a>Processeurs de télémétrie (ASP.NET)
Les processeurs de télémétrie peuvent filtrer et modifier chaque élément de télémétrie avant son envoi au portail à partir du Kit de développement logiciel (SDK).

Vous pouvez [écrire vos propres processeurs de télémétrie](./api-filtering-sampling.md#filtering).

#### <a name="adaptive-sampling-telemetry-processor-from-200-beta3"></a>Processeur de télémétrie d’échantillonnage adaptatif (à partir de 2.0.0-beta3)
Cette option est activée par défaut. Si votre application envoie beaucoup de télémétrie, ce processeur en supprime une partie.

```xml

    <TelemetryProcessors>
      <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
        <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
      </Add>
    </TelemetryProcessors>

```

Le paramètre fournit la cible que l'algorithme essaie d'atteindre. Chaque instance du Kit de développement logiciel (SDK) fonctionne de manière indépendante. Ainsi, si votre serveur est un cluster de plusieurs ordinateurs, le volume réel des données de télémétrie sera multiplié en conséquence.

[En savoir plus sur l'échantillonnage](./sampling.md).

#### <a name="fixed-rate-sampling-telemetry-processor-from-200-beta1"></a>Processeur de télémétrie d’échantillonnage à taux fixe (à partir de 2.0.0-beta1)
Il existe également un [processeur de télémétrie d’échantillonnage](./api-filtering-sampling.md) standard (à partir de 2.0.1) :

```XML

    <TelemetryProcessors>
     <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">

     <!-- Set a percentage close to 100/N where N is an integer. -->
     <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
     <SamplingPercentage>10</SamplingPercentage>
     </Add>
   </TelemetryProcessors>

```

## <a name="instrumentationkey"></a>InstrumentationKey
Ce paramètre détermine la ressource Application Insights dans laquelle vos données s’affichent. En général, vous créez une ressource séparée, avec une clé distincte, pour chacune de vos applications.

Si vous souhaitez définir la clé de manière dynamique, par exemple si vous souhaitez transmettre des résultats de votre application vers différentes ressources, vous pouvez omettre la clé du fichier de configuration, et la définir plutôt dans le code.

Pour définir la clé pour toutes les instances de TelemetryClient, y compris les modules standard de télémétrie. Définissez-la dans une méthode d’initialisation, telle que global.aspx.cs dans un service ASP.NET :

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights;

    protected void Application_Start()
    {
        TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
        configuration.InstrumentationKey = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
        var telemetryClient = new TelemetryClient(configuration);
   
```

Si vous souhaitez simplement envoyer un ensemble spécifique d’événements à une autre ressource, vous pouvez définir la clé pour un TelemetryClient spécifique :

```csharp

    var tc = new TelemetryClient();
    tc.Context.InstrumentationKey = "----- my key ----";
    tc.TrackEvent("myEvent");
    // ...

```

Pour obtenir une nouvelle clé, [créez une ressource dans le portail Application Insights][new].



## <a name="applicationid-provider"></a>Fournisseur ApplicationId

_Disponible à partir de la version 2.6.0_

L’objectif de ce fournisseur est de rechercher un ID d’application à partir d’une clé d’instrumentation. L’ID d’application est inclus dans RequestTelemetry et DependencyTelemetry, et utilisé pour déterminer Correlation dans le portail.

Cette option est disponible en définissant `TelemetryConfiguration.ApplicationIdProvider` dans le code ou dans la configuration.

### <a name="interface-iapplicationidprovider"></a>Interface : IApplicationIdProvider

```csharp
public interface IApplicationIdProvider
{
    bool TryGetApplicationId(string instrumentationKey, out string applicationId);
}
```


Nous fournissons deux implémentations dans le kit de développement logiciel (SDK) [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) : `ApplicationInsightsApplicationIdProvider` et `DictionaryApplicationIdProvider`.

### <a name="applicationinsightsapplicationidprovider"></a>ApplicationInsightsApplicationIdProvider

Il s’agit d’un wrapper placé autour de notre API de profil. Il limitera les requêtes et les résultats du cache.

Ce fournisseur est ajouté à votre fichier de configuration lorsque vous installez [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) ou [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/).

Cette classe possède une propriété facultative `ProfileQueryEndpoint`.
Par défaut, elle est définie sur `https://dc.services.visualstudio.com/api/profiles/{0}/appId`.
Si vous avez besoin de configurer un proxy pour cette configuration, nous vous recommandons d’utiliser le proxy sur l’adresse de base et d’inclure « /api/profiles/{0}/appId ». Notez que « {0} » est remplacé, lors de l’exécution par requête, par la clé d’instrumentation.

#### <a name="example-configuration-via-applicationinsightsconfig"></a>Exemple de configuration via ApplicationInsights.config :
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
        <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>Exemple de configuration à l’aide du code :
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new ApplicationInsightsApplicationIdProvider();
```

### <a name="dictionaryapplicationidprovider"></a>DictionaryApplicationIdProvider

Il s’agit d’un fournisseur statique qui s’appuie sur votre paire configurée clé d’instrumentation/ID d’application.

Cette classe possède une propriété `Defined` qui est un Dictionary<string,string> des paires clé d’instrumentation/ID d’application.

Cette classe possède une propriété facultative `Next` qui peut servir à configurer un autre fournisseur à utiliser lorsqu’une clé d’instrumentation demandée n’existe pas dans votre configuration.

#### <a name="example-configuration-via-applicationinsightsconfig"></a>Exemple de configuration via ApplicationInsights.config :
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.DictionaryApplicationIdProvider, Microsoft.ApplicationInsights">
        <Defined>
            <Type key="InstrumentationKey_1" value="ApplicationId_1"/>
            <Type key="InstrumentationKey_2" value="ApplicationId_2"/>
        </Defined>
        <Next Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights" />
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>Exemple de configuration à l’aide du code :
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new DictionaryApplicationIdProvider{
 Defined = new Dictionary<string, string>
    {
        {"InstrumentationKey_1", "ApplicationId_1"},
        {"InstrumentationKey_2", "ApplicationId_2"}
    }
};
```




## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur l’API][api].

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[exceptions]: ./asp-net-exceptions.md
[netlogs]: ./asp-net-trace-logs.md
[new]: ./create-new-resource.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md

