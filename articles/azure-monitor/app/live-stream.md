---
title: Diagnostiquer à l’aide du Flux de métriques temps réel – Azure Application Insights
description: Surveillez votre application web en temps réel avec des métriques personnalisées, et diagnostiquez les problèmes avec un flux temps réel des échecs, des traces et des événements.
ms.topic: conceptual
ms.date: 04/22/2019
ms.reviewer: sdash
ms.openlocfilehash: 1b8b4c43c559831810db9b92da6c2743556cd2ac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90973594"
---
# <a name="live-metrics-stream-monitor--diagnose-with-1-second-latency"></a>Flux de métriques temps réel : Surveiller et diagnostiquer avec une latence de 1 seconde

Supervisez votre application web dynamique en production en utilisant les flux de métriques temps réel (également appelés QuickPulse) à partir d’[Application Insights](./app-insights-overview.md). Sélectionnez et filtrez les métriques et les compteurs de performances à surveiller en temps réel, sans aucune perturbation de votre service. Inspectez les traces de pile à partir d’échantillons de demandes en échec et d’exceptions. En combinaison avec le [profileur](./profiler.md) et le [débogueur d’instantané](./snapshot-debugger.md), les flux de métriques temps réel constituent un outil de diagnostic puissant et non invasif pour votre site web dynamique.

Avec les flux de métriques temps réel, vous pouvez :

* Valider un correctif lors de sa publication, en observant les performances et les nombres d’échecs.
* Observer l’effet des tests de charge et diagnostiquer les problèmes de façon dynamique.
* Vous concentrer sur des sessions de tests particulières ou filtrer les problèmes connus, en sélectionnant et en filtrant les mesures que vous voulez observer.
* Obtenir les traces des exceptions quand elles se produisent.
* Faites des essais avec les filtres pour rechercher les indicateurs de performance clés les plus pertinents.
* Surveiller en temps réel des compteurs de performances Windows.
* Identifiez plus facilement un serveur qui rencontre des problèmes et filtrez tous les indicateurs de performance clés/flux en temps réel sur ce serveur uniquement.

![Onglet Métriques temps réel](./media/live-stream/live-metric.png)

Les métriques temps réel sont actuellement prises en charge pour les applications ASP.NET, ASP.NET Core, Azure Functions, Java et Node.js.

## <a name="get-started"></a>Bien démarrer

1. Suivez les instructions spécifiques au langage pour activer les métriques temps réel.
   * [ASP.NET](./asp-net.md) : les métriques temps réel sont activées par défaut.
   * [ASP.NET Core](./asp-net-core.md) : les métriques temps réel sont activées par défaut.
   * [Console/Worker .NET/.NET Core](./worker-service.md) : les métriques temps réel sont activées par défaut.
   * [Applications .NET : activer l’utilisation du code](#enable-livemetrics-using-code-for-any-net-application).
   * [Node.JS](./nodejs.md#live-metrics)

2. Dans le [portail Azure](https://portal.azure.com), ouvrez la ressource Application Insights pour votre application puis ouvrez Flux temps réel.

3. [Sécurisez le canal de contrôle](#secure-the-control-channel) si vous utilisez des données sensibles comme des noms de clients dans vos filtres.

### <a name="enable-livemetrics-using-code-for-any-net-application"></a>Activer LiveMetrics à l’aide du code pour n’importe quelle application .NET

Même si LiveMetrics est activé par défaut lors de l’intégration à l’aide des instructions recommandées pour les applications .NET, ce qui suit montre comment configurer les métriques temps réel manuellement.

1. Installer le package NuGet [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector)
2. L’exemple de code d’application console suivant illustre la configuration de métriques temps réel.

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
using System;
using System.Threading.Tasks;

namespace LiveMetricsDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create a TelemetryConfiguration instance.
            TelemetryConfiguration config = TelemetryConfiguration.CreateDefault();
            config.InstrumentationKey = "INSTRUMENTATION-KEY-HERE";
            QuickPulseTelemetryProcessor quickPulseProcessor = null;
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder
                .Use((next) =>
                {
                    quickPulseProcessor = new QuickPulseTelemetryProcessor(next);
                    return quickPulseProcessor;
                })
                .Build();

            var quickPulseModule = new QuickPulseTelemetryModule();

            // Secure the control channel.
            // This is optional, but recommended.
            quickPulseModule.AuthenticationApiKey = "YOUR-API-KEY-HERE";
            quickPulseModule.Initialize(config);
            quickPulseModule.RegisterTelemetryProcessor(quickPulseProcessor);

            // Create a TelemetryClient instance. It is important
            // to use the same TelemetryConfiguration here as the one
            // used to setup Live Metrics.
            TelemetryClient client = new TelemetryClient(config);

            // This sample runs indefinitely. Replace with actual application logic.
            while (true)
            {
                // Send dependency and request telemetry.
                // These will be shown in Live Metrics stream.
                // CPU/Memory Performance counter is also shown
                // automatically without any additional steps.
                client.TrackDependency("My dependency", "target", "http://sample",
                    DateTimeOffset.Now, TimeSpan.FromMilliseconds(300), true);
                client.TrackRequest("My Request", DateTimeOffset.Now,
                    TimeSpan.FromMilliseconds(230), "200", true);
                Task.Delay(1000).Wait();
            }
        }
    }
}
```

Bien que l’exemple ci-dessus s’utilise pour une application console, le même code peut être utilisé dans toutes les applications .NET. Si d’autres TelemetryModules sont activés, ce qui collecte automatiquement les données de télémétrie, il est important de s’assurer que la même configuration utilisée pour l’initialisation de ces modules est également utilisée pour le module de métriques temps réel.

## <a name="how-does-live-metrics-stream-differ-from-metrics-explorer-and-analytics"></a>En quoi les flux de métriques temps réel diffèrent-ils de Metrics Explorer et d’Analytique ?

| |Flux temps réel | Metrics Explorer et Analytique |
|---|---|---|
|**Latence**|Données affichées en une seconde|Agrégé sur plusieurs minutes|
|**Pas de conservation des données**|Les données persistent tant qu’elles se trouvent sur le graphique puis elles sont abandonnées.|[Données conservées pendant 90 jours](./data-retention-privacy.md#how-long-is-the-data-kept)|
|**À la demande**|Les données sont diffusées uniquement tant que le volet Métriques temps réel est ouvert |Les données sont envoyées quand le SDK est installé et activé|
|**Gratuit**|Pas de facturation pour les données du flux temps réel|Soumis à [tarification](./pricing.md)
|**Échantillonnage**|Tous les compteurs et métriques sélectionnés sont transmis. Les échecs et les traces de pile sont échantillonnés. |Les événements peuvent être [échantillonnés](./api-filtering-sampling.md)|
|**Canal de contrôle**|Les signaux de contrôle de filtre sont envoyés au SDK. Nous vous recommandons de sécuriser ce canal.|La communication est unidirectionnelle vers le portail|

## <a name="select-and-filter-your-metrics"></a>Sélectionner et filtrer vos métriques

(Disponible avec ASP.NET, ASP.NET Core et Azure Functions (v2).)

Vous pouvez surveiller un KPI personnalisé en temps réel en appliquant des filtres arbitraires sur les données de télémétrie Application Insights à partir du portail. Cliquez sur la commande de filtre qui s’affiche lorsque vous passez la souris sur un graphique. Le graphique suivant indique un KPI de nombre de demandes personnalisé avec des filtres sur les attributs d’URL et de durée. Vérifiez vos filtres grâce à la section Vue d’ensemble du flux qui affiche un flux en temps réel de télémétrie correspondant aux critères que vous avez spécifiés.

![Filtrer le taux de requêtes](./media/live-stream/filter-request.png)

Vous pouvez surveiller une valeur autre qu’un nombre. Les options varient selon le type de flux, qui peut être n’importe quelle télémétrie Application Insights : demandes, dépendances, exceptions, suivis, événements ou métriques. Il peut s’agir d’une [mesure personnalisée](./api-custom-events-metrics.md#properties) :

![Générateur de requêtes sur le taux de demandes avec métrique personnalisée](./media/live-stream/query-builder-request.png)

En plus des données de télémétrie Application Insights, vous pouvez également surveiller n’importe quel compteur de performances Windows en le sélectionnant dans les options de flux et en fournissant le nom du compteur de performances.

Les métriques temps réel sont agrégées à deux endroits : localement sur chaque serveur, puis sur tous les serveurs. Vous pouvez modifier la valeur par défaut en sélectionnant d’autres options dans les listes déroulantes respectives.

## <a name="sample-telemetry-custom-live-diagnostic-events"></a>Exemple de télémétrie : Événements de diagnostic Azure personnalisés et en temps réel
Par défaut, le flux d’événements temps réel présente des exemples de demandes ayant échoué et d’appels de dépendance, d’exceptions, d’événements et de suivis. Cliquez sur l’icône de filtre pour afficher les critères appliqués.

![Bouton Filtrer](./media/live-stream/filter.png)

Comme avec les métriques, vous pouvez spécifier un critère arbitraire sur les types de données de télémétrie Application Insights. Dans cet exemple, nous sélectionnons des échecs de demande spécifiques et des événements.

![Générateur de requêtes](./media/live-stream/query-builder.png)

> [!NOTE]
> pour les critères basés sur un message d’exception, utilisez le message d’exception le plus à l’extérieur. Dans l’exemple précédent, pour éliminer l’exception sans gravité avec le message d’exception interne (après le délimiteur « <-- ») « Client déconnecté ». utilisez un critère de message ne contenant pas « Erreur de lecture du contenu de la demande ».

Consultez les informations détaillées d’un élément dans le flux temps réel en cliquant dessus. Vous pouvez interrompre le flux en cliquant sur **Suspendre**, en faisant simplement défiler ou en cliquant sur un élément. Le flux temps réel reprend lorsque vous refaites défiler vers le haut, ou en cliquant sur le compteur d’éléments collectés alors qu’il était suspendu.

![Capture d’écran montrant la fenêtre d’exemple de télémétrie, avec une exception sélectionnée et les détails de l’exception affichés au bas de la fenêtre.](./media/live-stream/sample-telemetry.png)

## <a name="filter-by-server-instance"></a>Filtrer par instance de serveur

Si vous voulez surveiller une instance de rôle serveur spécifique, vous pouvez appliquer un filtre par serveur. Pour filtrer, sélectionnez le nom du serveur sous *Serveurs*.

![Échecs dynamiques échantillonnés](./media/live-stream/filter-by-server.png)

## <a name="secure-the-control-channel"></a>Sécuriser le canal de contrôle

> [!NOTE]
> Actuellement, vous pouvez uniquement configurer un canal authentifié à l’aide de la supervision basée sur le code, et vous ne pouvez pas authentifier les serveurs à l’aide de l’attachement sans code.

Les critères de filtres personnalisés que vous spécifiez dans la portail des métriques temps réel sont renvoyés au composant de métriques temps réel dans le Kit de développement logiciel (SDK) Application Insights. Les filtres peuvent potentiellement contenir des informations sensibles telles que des ID clients. Vous pouvez sécuriser le canal avec une clé API secrète en plus de la clé d’instrumentation.

### <a name="create-an-api-key"></a>Création d’une clé API

![Clé API > Créer une clé API](./media/live-stream/api-key.png)
![onglet Créer une clé API. Sélectionnez « Authentifier le canal de contrôle du SDK », puis « Générer la clé »](./media/live-stream/create-api-key.png)

### <a name="add-api-key-to-configuration"></a>Ajout d’une clé API à la configuration

### <a name="aspnet"></a>ASP.NET

Dans le fichier applicationinsights.config, ajoutez AuthenticationApiKey à QuickPulseTelemetryModule :

```XML
<Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <AuthenticationApiKey>YOUR-API-KEY-HERE</AuthenticationApiKey>
</Add>
```

### <a name="aspnet-core"></a>ASP.NET Core

Pour les applications [ASP.NET Core](./asp-net-core.md), suivez les instructions ci-dessous.

Modifiez `ConfigureServices` de votre fichier Startup.cs comme suit :

Ajoutez l’espace de noms suivant.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Modifiez ensuite la méthode `ConfigureServices` comme indiqué ci-dessous.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // existing code which include services.AddApplicationInsightsTelemetry() to enable Application Insights.
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
}
```

Pour plus d’informations sur la configuration des applications ASP.NET Core, consultez notre guide sur la [configuration des modules de télémétrie dans ASP.NET Core](./asp-net-core.md#configuring-or-removing-default-telemetrymodules).

### <a name="workerservice"></a>WorkerService

Pour les applications [WorkerService](./worker-service.md), suivez les instructions ci-dessous.

Ajoutez l’espace de noms suivant.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Ensuite, ajoutez la ligne suivante avant l’appel `services.AddApplicationInsightsTelemetryWorkerService`.

```csharp
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```

Pour plus d’informations sur la configuration des applications WorkerService, consultez notre guide sur la [configuration des modules de télémétrie dans WorkerServices](./worker-service.md#configuring-or-removing-default-telemetrymodules).

### <a name="azure-function-apps"></a>Applications de fonction Azure

Pour les applications de fonction Azure (v2), la sécurisation du canal avec une clé API peut se faire avec une variable d’environnement.

Créez une clé API à partir de votre ressource Application Insights et accédez à **Paramètres > Configuration** pour votre application de fonction. Sélectionnez **Nouveau paramètre d’application** et entrez un nom de `APPINSIGHTS_QUICKPULSEAUTHAPIKEY` et une valeur qui correspond à votre clé API.

Toutefois, si vous connaissez et faites confiance à tous les serveurs connectés, vous pouvez essayer les filtres personnalisés sans le canal authentifié. Cette option est disponible pour six mois. Ce remplacement est nécessaire à chaque nouvelle session, ou lorsqu’un nouveau serveur est en ligne.

![Options d’authentification de métriques temps réel](./media/live-stream/live-stream-auth.png)

>[!NOTE]
>Nous vous recommandons vivement de configurer le canal authentifié avant d’entrer des informations potentiellement sensibles comme un ID client dans les critères de filtre.
>

## <a name="supported-features-table"></a>Tableau des fonctionnalités prises en charge

| Langage                         | Métriques de base       | Mesures de performances | Filtrage personnalisé    | Exemple de télémétrie    | UC fractionnée par processus |
|----------------------------------|:--------------------|:--------------------|:--------------------|:--------------------|:---------------------|
| .NET Framework                   | Pris en charge (V2.7.2 et versions ultérieures) | Pris en charge (V2.7.2 et versions ultérieures) | Pris en charge (V2.7.2 et versions ultérieures) | Pris en charge (V2.7.2 et versions ultérieures) | Pris en charge (V2.7.2 et versions ultérieures)  |
| .NET Core (cible=. NET Framework)| Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge (V2.4.1 et versions ultérieures)  |
| .NET Core (cible=. NET Core)     | Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge*          | Pris en charge (V2.4.1 et versions ultérieures) | Pris en charge (V2.4.1 et versions ultérieures) | **Non pris en charge**    |
| Azure Functions v2               | Prise en charge           | Prise en charge           | Prise en charge           | Prise en charge           | **Non pris en charge**    |
| Java                             | Pris en charge (V2.0.0 et versions ultérieures) | Pris en charge (V2.0.0 et versions ultérieures) | **Non pris en charge**   | **Non pris en charge**   | **Non pris en charge**    |
| Node.js                          | Pris en charge (V1.3.0 et versions ultérieures) | Pris en charge (V1.3.0 et versions ultérieures) | **Non pris en charge**   | Pris en charge (V1.3.0 et versions ultérieures) | **Non pris en charge**    |

Les métriques de base incluent les requêtes, les dépendances et le taux d’exceptions. Les métriques du niveau de performance (compteurs de performances) incluent la mémoire et l’UC. L’exemple de télémétrie montre un flux d’informations détaillées pour les demandes et les dépendances ayant échoué, les exceptions, les événements et les traces.

 \* La prise en charge de PerfCounters varie légèrement entre les versions de .NET Core qui ne ciblent pas le .NET Framework :

- Les métriques PerfCounters sont prises en charge lors de l’exécution dans Azure App Service pour Windows. (Kit de développement logiciel [SDK] AspNetCore version 2.4.1 ou ultérieure)
- Les PerfCounters sont pris en charge lorsque l’application s’exécute sur n’importe quel ordinateur Windows (machine virtuelle, service cloud ou ordinateur local, etc.) [Kit de développement logiciel (SDK) AspNetCore version 2.7.1 ou ultérieure], mais pour les applications ciblant .NET Core 2.0 ou une version ultérieure.
- Les PerfCounters sont pris en charge lorsque l’application s’exécute N’IMPORTE OÚ (Linux, Windows, app service pour Linux, conteneurs, etc.) dans les versions les plus récentes (par exemple, AspNetCore SDK version 2.8.0 ou ultérieure), mais uniquement pour les applications ciblant .NET Core 2.0 ou une version ultérieure.

## <a name="troubleshooting"></a>Dépannage

Le Flux de métriques temps réel utilise des adresses IP différentes de celles des autres données de télémétrie Application Insights. Assurez-vous que [ces adresses IP](./ip-addresses.md) sont ouvertes dans votre pare-feu. Vérifiez également que les [ports sortants pour le flux de métriques temps réel](./ip-addresses.md#outgoing-ports) sont ouverts dans le pare-feu de vos serveurs.

## <a name="next-steps"></a>Étapes suivantes

* [Surveillance de l’utilisation avec Application Insights](./usage-overview.md)
* [Utilisation de Diagnostic Search](./diagnostic-search.md)
* [Profiler](./profiler.md)
* [Débogueur de capture instantanée](./snapshot-debugger.md)
