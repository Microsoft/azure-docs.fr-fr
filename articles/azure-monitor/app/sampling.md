---
title: Échantillonnage de données de télémétrie dans Azure Application Insights | Microsoft Docs
description: Comment maintenir sous contrôle le volume de télémétrie.
ms.topic: conceptual
ms.date: 01/17/2020
ms.reviewer: vitalyg
ms.custom: fasttrack-edit
ms.openlocfilehash: e9334d222d443679362514481ecd83b90bbda0ac
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96855071"
---
# <a name="sampling-in-application-insights"></a>Échantillonnage dans Application Insights

L’échantillonnage est une fonctionnalité dans [Azure Application Insights](./app-insights-overview.md). Il s’agit de la méthode recommandée pour réduire le trafic, les coûts des données et les coûts de stockage de la télémétrie, tout en conservant une analyse statistiquement correcte des données d’application. L’échantillonnage vous permet aussi d’éviter qu’Application Insights limite votre télémétrie. Le filtre d’échantillonnage sélectionne des éléments liés entre eux, pour vous permettre de naviguer entre les éléments quand vous effectuez un diagnostic.

Lorsque les métriques sont présentées dans le portail, elles sont renormalisées pour prendre en compte l’échantillonnage. Cette opération réduit tout effet sur les statistiques.

## <a name="brief-summary"></a>Bref résumé

* Il existe trois types d’échantillonnages différents : échantillonnage adaptatif, échantillonnage à fréquence fixe et échantillonnage d’ingestion.
* L’échantillonnage adaptatif est activé par défaut dans toutes les dernières versions d’ASP.NET Application Insights et des kits SDK ASP.NET Core. Il est également utilisé par [Azure Functions](../../azure-functions/functions-overview.md).
* L’échantillonnage à fréquence fixe est disponible dans les versions récentes des SDK Application Insights pour ASP.NET, ASP.NET Core, Java (à la fois l’agent et le SDK) et Python.
* L’échantillonnage d’ingestion fonctionne sur le point de terminaison de service Application Insights. Il s’applique seulement quand aucun autre échantillonnage n’est appliqué. Si le SDK échantillonne votre télémétrie, l’échantillonnage d’ingestion est désactivé.
* Pour les applications web, si vous consignez des événements personnalisés et que vous devez garantir qu’un ensemble d’événements sont conservés ou ignorés conjointement, les événements doivent avoir la même valeur pour `OperationId`.
* Si vous écrivez des requêtes Analytics, vous devez [tenir compte de l’échantillonnage](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#aggregations). En particulier, au lieu de compter simplement les enregistrements, vous devez utiliser `summarize sum(itemCount)`.
* Certains types de données de télémétrie, notamment les métriques de performances et les métriques personnalisées, sont toujours conservées, que l’échantillonnage soit ou non activé.

Le tableau suivant récapitule les types d’échantillonnage disponibles pour chaque SDK et chaque type d’application :

| Kit de développement logiciel (SDK) Application Insights | Échantillonnage adaptatif pris en charge | Échantillonnage à fréquence fixe pris en charge | Échantillonnage d’ingestion pris en charge |
|-|-|-|-|
| ASP.NET | [Oui (activé par défaut)](#configuring-adaptive-sampling-for-aspnet-applications) | [Oui](#configuring-fixed-rate-sampling-for-aspnet-applications) | Seulement si aucun autre échantillonnage n’est appliqué |
| ASP.NET Core | [Oui (activé par défaut)](#configuring-adaptive-sampling-for-aspnet-core-applications) | [Oui](#configuring-fixed-rate-sampling-for-aspnet-core-applications) | Seulement si aucun autre échantillonnage n’est appliqué |
| Azure Functions | [Oui (activé par défaut)](#configuring-adaptive-sampling-for-azure-functions) | Non | Seulement si aucun autre échantillonnage n’est appliqué |
| Java | Non | [Oui](#configuring-fixed-rate-sampling-for-java-applications) | Seulement si aucun autre échantillonnage n’est appliqué |
| Node.JS | Non | [Oui](./nodejs.md#sampling) | Seulement si aucun autre échantillonnage n’est appliqué
| Python | Non | [Oui](#configuring-fixed-rate-sampling-for-opencensus-python-applications) | Seulement si aucun autre échantillonnage n’est appliqué |
| Tous les autres | Non | Non | [Oui](#ingestion-sampling) |

> [!NOTE]
> Les informations de la majeure partie de cette page s’appliquent aux versions actuelles des SDK Application Insights. Pour plus d’informations sur les versions antérieures des SDK, [consultez la section ci-dessous](#older-sdk-versions).

## <a name="types-of-sampling"></a>Types d’échantillonnage

Il existe trois méthodes d’échantillonnage différentes :

* L’**échantillonnage adaptatif** ajuste automatiquement le volume de données de télémétrie envoyées à partir du SDK dans votre application ASP.NET/ASP.NET Core, et à partir d’Azure Functions. Il s’agit de l’échantillonnage par défaut quand vous utilisez le SDK ASP.NET ou ASP.NET Core. L’échantillonnage adaptatif est actuellement disponible seulement pour la télémétrie ASP.NET côté serveur et pour Azure Functions.

* **L’échantillonnage à débit fixe** réduit le volume de données de télémétrie envoyées par votre serveur ASP.NET/ASP.NET Core ou Java et par le navigateur de vos utilisateurs. Vous définissez le débit. Le client et le serveur synchronisent leur échantillonnage de façon à ce que vous puissiez naviguer entre les demandes et les affichages de pages associés dans Search.

* L’**échantillonnage d’ingestion** est effectué sur le point de terminaison de service Application Insights. Il ignore certaines données de télémétrie provenant de votre application, à une fréquence d’échantillonnage que vous définissez. Le trafic des données de télémétrie reçu de votre application n’est pas réduit, mais cela vous permet de respecter votre quota mensuel. L’avantage principal de l’échantillonnage d’ingestion est que vous pouvez définir la fréquence d’échantillonnage sans avoir à redéployer votre application. L’échantillonnage d’ingestion fonctionne uniformément pour tous les serveurs et tous les clients, mais il ne s’applique pas quand d’autres types d’échantillonnage sont utilisés.

> [!IMPORTANT]
> Si les méthodes d’échantillonnage à taux adaptatif ou fixe sont activées pour un type de données de télémétrie, l’échantillonnage d’ingestion sera désactivé pour ces données de télémétrie. Toutefois, les types de télémétrie qui sont exclus de l’échantillonnage au niveau du SDK seront toujours soumis à un échantillonnage d’ingestion au taux défini dans le portail.

## <a name="adaptive-sampling"></a>échantillonnage adaptatif

L’échantillonnage adaptatif a une incidence sur le volume de données de télémétrie envoyées à partir de votre application de serveur web au point de terminaison de service Application Insights.

> [!TIP]
> L’échantillonnage adaptatif est activé par défaut quand vous utilisez le SDK ASP.NET ou SDK ASP.NET Core, et il est également activé par défaut pour Azure Functions.

Le volume est automatiquement maintenu en deçà d’un débit de trafic maximal spécifié, et est contrôlé par le paramètre `MaxTelemetryItemsPerSecond`. Si l’application génère une faible quantité de données de télémétrie, comme lors du débogage ou en raison d’une faible utilisation, les éléments ne sont pas abandonnés par le processeur d’échantillonnage tant que le volume est inférieur à `MaxTelemetryItemsPerSecond`. Quand le volume de données de télémétrie augmente, le taux d’échantillonnage est ajusté afin d’obtenir le volume cible. L’ajustement est recalculé à intervalles réguliers en fonction d’une moyenne mobile de la vitesse de transmission sortante.

Pour atteindre le volume cible, certaines des données de télémétrie générées sont ignorées. Toutefois, comme d’autres types d’échantillonnage, l’algorithme conserve les éléments de télémétrie associés. Par exemple, quand vous inspectez les données de télémétrie dans Search, vous pouvez trouver la demande liée à une exception spécifique.

Les données de mesure telles que le taux de demandes et le taux d’exceptions sont ajustées pour compenser le taux d’échantillonnage, afin qu’elles affichent des valeurs approximativement correctes dans Metrics Explorer.

### <a name="configuring-adaptive-sampling-for-aspnet-applications"></a>Configuration de l’échantillonnage adaptatif pour les applications ASP.NET

> [!NOTE]
> Cette section s’applique aux applications ASP.NET, et non pas aux applications ASP.NET Core. [Découvrez plus d’informations sur la configuration de l’échantillonnage adaptatif pour les applications ASP.NET Core plus loin dans ce document.](#configuring-adaptive-sampling-for-aspnet-core-applications)

Dans [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md), vous pouvez ajuster plusieurs paramètres dans le nœud `AdaptiveSamplingTelemetryProcessor`. Les chiffres indiqués correspondent aux valeurs par défaut :

* `<MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>`
  
    Taux cible d’[opérations logiques](./correlation.md#data-model-for-telemetry-correlation) que l’algorithme adaptatif vise à collecter **sur chaque serveur hôte**. Si votre application web s’exécute sur plusieurs hôtes, réduisez cette valeur afin de ne pas dépasser votre débit de trafic cible sur le portail Application Insights.

* `<EvaluationInterval>00:00:15</EvaluationInterval>` 
  
    Intervalle auquel le taux actuel de télémétrie est réévalué. L’évaluation est effectuée sous forme de moyenne mobile. Vous souhaiterez peut-être raccourcir cet intervalle si vos données de télémétrie sont soumises à des pics soudains.

* `<SamplingPercentageDecreaseTimeout>00:02:00</SamplingPercentageDecreaseTimeout>`
  
    Quand la valeur du pourcentage d’échantillonnage change, dans quel délai pouvons-nous réduire le pourcentage d’échantillonnage pour capturer moins de données ?

* `<SamplingPercentageIncreaseTimeout>00:15:00</SamplingPercentageIncreaseTimeout>`
  
    Quand la valeur du pourcentage d’échantillonnage change, dans quel délai pouvons-nous augmenter le pourcentage d’échantillonnage pour capturer plus de données ?

* `<MinSamplingPercentage>0.1</MinSamplingPercentage>`
  
    Quand le pourcentage d’échantillonnage varie, quelle valeur minimale pouvons-nous définir ?

* `<MaxSamplingPercentage>100.0</MaxSamplingPercentage>`
  
    Quand le pourcentage d’échantillonnage varie, quelle valeur maximale pouvons-nous définir ?

* `<MovingAverageRatio>0.25</MovingAverageRatio>` 
  
    Lors du calcul de la moyenne mobile, ceci spécifie le pondération à affecter à la valeur la plus récente. Utilisez une valeur inférieure ou égale à 1. Plus les valeurs sont petites, moins l’algorithme est réactif en cas de modifications brusques.

* `<InitialSamplingPercentage>100</InitialSamplingPercentage>`
  
    Quantité de données de télémétrie à échantillonner quand l’application vient de démarrer. Ne diminuez pas cette valeur pendant le débogage.

* `<ExcludedTypes>Trace;Exception</ExcludedTypes>`
  
    Une liste des types (délimités par des points-virgules) que vous ne voulez pas voir échantillonnés. Les types reconnus sont : `Dependency`, `Event`, `Exception`, `PageView`, `Request`, `Trace`. Toute la télémétrie des types spécifiés est transmise ; les types qui ne sont pas spécifiés sont échantillonnés.

* `<IncludedTypes>Request;Dependency</IncludedTypes>`
  
    Une liste des types (délimités par des points-virgules) que vous voulez voir échantillonnés. Les types reconnus sont : `Dependency`, `Event`, `Exception`, `PageView`, `Request`, `Trace`. Les types spécifiés sont échantillonnés ; toute la télémétrie des autres types est toujours transmise.

**Pour désactiver** l’échantillonnage adaptatif, supprimez le ou les nœuds `AdaptiveSamplingTelemetryProcessor` de `ApplicationInsights.config`.

#### <a name="alternative-configure-adaptive-sampling-in-code"></a>Alternative : Configurer l’échantillonnage adaptatif dans le code

Au lieu de définir le paramètre d’échantillonnage dans le fichier `.config`, vous pouvez définir ces valeurs par programmation.

1. Supprimez tous les nœuds `AdaptiveSamplingTelemetryProcessor` du fichier `.config`.
2. Utilisez l’extrait de code suivant pour configurer l’échantillonnage adaptatif :

    ```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.Channel.Implementation;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    
    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    // Enable AdaptiveSampling so as to keep overall telemetry volume to 5 items per second.
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([En savoir plus sur les processeurs de télémétrie](./api-filtering-sampling.md#filtering).)

Vous pouvez également ajuster le taux d’échantillonnage pour chaque type de données de télémétrie individuellement, ou même exclure complètement certains types de l’échantillonnage :

```csharp
// The following configures adaptive sampling with 5 items per second, and also excludes Dependency telemetry from being subjected to sampling.
builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");
```

### <a name="configuring-adaptive-sampling-for-aspnet-core-applications"></a>Configuration de l’échantillonnage adaptatif pour les applications ASP.NET Core

Il n’existe pas de fichier `ApplicationInsights.config` pour les applications ASP.NET Core : chaque configuration est donc effectuée via du code.
L’échantillonnage adaptatif est activé par défaut pour toutes les applications ASP.NET Core. Vous pouvez désactiver ou personnaliser le comportement d’échantillonnage.

#### <a name="turning-off-adaptive-sampling"></a>Désactivation de l’échantillonnage adaptatif

La fonctionnalité d’échantillonnage par défaut peut être désactivée lors de l’ajout d’un service Application Insights dans la méthode `ConfigureServices`, en utilisant `ApplicationInsightsServiceOptions` dans le fichier `Startup.cs` :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetry(aiOptions);

    //...
}
```

Le code ci-dessus va désactiver l’échantillonnage adaptatif. Pour ajouter un échantillonnage avec davantage d’options de personnalisation, procédez comme suit.

#### <a name="configure-sampling-settings"></a>Configurer les paramètres d’échantillonnage

Pour personnaliser le comportement de l’échantillonnage, utilisez des méthodes d’extension de `TelemetryProcessorChainBuilder`, comme indiqué ci-dessous.

> [!IMPORTANT]
> Si vous utilisez cette méthode pour configurer l’échantillonnage, veillez à définir la propriété `aiOptions.EnableAdaptiveSampling` sur `false` lors de l’appel de `AddApplicationInsightsTelemetry()`.

```csharp
using Microsoft.ApplicationInsights.Extensibility

public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
{
    var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = configuration.TelemetryProcessorChainBuilder;

    // Using adaptive sampling
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // Alternately, the following configures adaptive sampling with 5 items per second, and also excludes DependencyTelemetry from being subject to sampling.
    // builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));
    
    builder.Build();

    // ...
}
```

### <a name="configuring-adaptive-sampling-for-azure-functions"></a>Configuration de l’échantillonnage adaptatif pour Azure Functions

Suivez les instructions de [cette page](../../azure-functions/configure-monitoring.md#configure-sampling) pour configurer l’échantillonnage adaptatif pour les applications qui s’exécutent dans Azure Functions.

## <a name="fixed-rate-sampling"></a>Échantillonnage à fréquence fixe

L’échantillonnage à fréquence fixe réduit le trafic envoyé depuis votre serveur web et depuis les navigateurs web. À la différence de l’échantillonnage adaptatif, il réduit les données de télémétrie à un débit fixe choisi par vos soins. L’échantillonnage à fréquence fixe est disponible pour les applications ASP.NET, ASP.NET Core, Java et Python.

Comme les autres techniques d’échantillonnage, elle conserve également les éléments associés. De plus, elle synchronise l’échantillonnage client et serveur afin que les éléments associés soient conservés. Par exemple, quand vous examinez un affichage de page dans Recherche, vous pouvez trouver ses demandes au serveur associées. 

Dans Metrics Explorer, les taux tels que le nombre de demandes et d’exceptions sont multipliés par un facteur pour compenser le taux d’échantillonnage, afin qu’ils soient approximativement corrects.

### <a name="configuring-fixed-rate-sampling-for-aspnet-applications"></a>Configuration de l’échantillonnage à fréquence fixe pour les applications ASP.NET

1. **Désactivez l’échantillonnage adaptatif** : Dans [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md), supprimez ou placez en commentaire le nœud `AdaptiveSamplingTelemetryProcessor`.

    ```xml
    <TelemetryProcessors>
        <!-- Disabled adaptive sampling:
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
        </Add>
        -->
    ```

2. **Activez le module d’échantillonnage à débit fixe.** Ajoutez cet extrait de code à [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md) :
   
    ```XML
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <!-- Set a percentage close to 100/N where N is an integer. -->
            <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
            <SamplingPercentage>10</SamplingPercentage>
        </Add>
    </TelemetryProcessors>
    ```

      Au lieu de définir le paramètre d’échantillonnage dans le fichier `ApplicationInsights.config`, vous pouvez aussi définir ces valeurs programmatiquement :

    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    builder.UseSampling(10.0); // percentage

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([En savoir plus sur les processeurs de télémétrie](./api-filtering-sampling.md#filtering).)

### <a name="configuring-fixed-rate-sampling-for-aspnet-core-applications"></a>Configuration de l’échantillonnage à fréquence fixe pour les applications ASP.NET Core

1. **Désactivez l’échantillonnage adaptatif** :  Des modifications peuvent être apportées dans la méthode `ConfigureServices` avec `ApplicationInsightsServiceOptions` :

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...

        var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
        aiOptions.EnableAdaptiveSampling = false;
        services.AddApplicationInsightsTelemetry(aiOptions);

        //...
    }
    ```

2. **Activez le module d’échantillonnage à débit fixe.** Des modifications peuvent être apportées dans la méthode `Configure`, comme illustré dans l’extrait de code ci-dessous :

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

        var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
        // For older versions of the Application Insights SDK, use the following line instead:
        // var builder = configuration.TelemetryProcessorChainBuilder;

        // Using fixed rate sampling
        double fixedSamplingPercentage = 10;
        builder.UseSampling(fixedSamplingPercentage);

        builder.Build();

        // ...
    }
    ```

### <a name="configuring-fixed-rate-sampling-for-java-applications"></a>Configuration de l’échantillonnage à fréquence fixe pour les applications Java

Par défaut, aucun échantillonnage n’est activé dans l’agent et le SDK Java. Actuellement, il ne prend en charge que l’échantillonnage à fréquence fixe. L’échantillonnage adaptatif n’est pas pris en charge dans Java.

#### <a name="configuring-java-agent"></a>Configuration de l’agent Java

1. Télécharger [applicationinsights-agent-3.0.0-PREVIEW.5.jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0-PREVIEW.5/applicationinsights-agent-3.0.0-PREVIEW.5.jar)

1. Pour activer l’échantillonnage, ajoutez ceci à votre fichier `applicationinsights.json` :

```json
{
  "sampling": {
    "percentage": 10 //this is just an example that shows you how to enable only only 10% of transaction 
  }
}
```

#### <a name="configuring-java-sdk"></a>Configuration du SDK Java

1. Téléchargez et configurez votre application web avec la dernière version du [SDK Java Application Insights](./java-get-started.md).

2. **Activez le module d’échantillonnage à fréquence fixe** en ajoutant l’extrait de code suivant au fichier `ApplicationInsights.xml` :

    ```XML
    <TelemetryProcessors>
        <BuiltInProcessors>
            <Processor type="FixedRateSamplingTelemetryProcessor">
                <!-- Set a percentage close to 100/N where N is an integer. -->
                <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
                <Add name="SamplingPercentage" value="50" />
            </Processor>
        </BuiltInProcessors>
    </TelemetryProcessors>
    ```

3. Vous pouvez inclure ou exclure de l’échantillonnage des types de données de télémétrie en utilisant les étiquettes suivantes dans le `FixedRateSamplingTelemetryProcessor` de l’étiquette `Processor` :
   
    ```XML
    <ExcludedTypes>
        <ExcludedType>Request</ExcludedType>
    </ExcludedTypes>

    <IncludedTypes>
        <IncludedType>Exception</IncludedType>
    </IncludedTypes>
    ```

Voici les types de données de télémétrie qu’il est possible d’inclure ou d’exclure de l’échantillonnage : `Dependency`, `Event`, `Exception`, `PageView`, `Request` et `Trace`.

> [!NOTE]
> Pour le pourcentage d’échantillonnage, choisissez un pourcentage proche de 100/N, où N est un nombre entier.  L’échantillonnage ne prend actuellement en charge aucune autre valeur.

### <a name="configuring-fixed-rate-sampling-for-opencensus-python-applications"></a>Configuration de l’échantillonnage à fréquence fixe pour les applications OpenCensus Python

Instrumentez votre application avec les derniers [exportateurs OpenCensus Azure Monitor](./opencensus-python.md).

> [!NOTE]
> L’échantillonnage à fréquence fixe n’est pas disponible pour l’exportateur de métriques. Cela signifie que les métriques personnalisées sont les seuls types de données de télémétrie dans lesquels l’échantillonnage ne peut PAS être configuré. L’exportateur de métriques envoie toutes les données de télémétrie qu’il suit.

#### <a name="fixed-rate-sampling-for-tracing"></a>Échantillonnage à fréquence fixe pour le suivi ####
Vous pouvez spécifier un `sampler` dans le cadre de votre configuration `Tracer`. Si aucun échantillonneur explicite n’est fourni, `ProbabilitySampler` est utilisé par défaut. `ProbabilitySampler` utilise par défaut un taux de 1/10 000 par défaut, ce qui signifie qu’une requête sur 10 000 est envoyée à Application Insights. Si vous souhaitez spécifier un taux d’échantillonnage, voir ci-dessous.

Pour spécifier un taux d’échantillonnage, vérifiez que votre `Tracer` spécifie un échantillonneur avec un taux d’échantillonnage compris entre 0,0 et 1,0 (inclusivement). Un taux d’échantillonnage de 1,0 représente 100 %, ce qui signifie que toutes vos requêtes sont envoyées sous forme de télémétrie à Application Insights.

```python
tracer = Tracer(
    exporter=AzureExporter(
        instrumentation_key='00000000-0000-0000-0000-000000000000',
    ),
    sampler=ProbabilitySampler(1.0),
)
```

#### <a name="fixed-rate-sampling-for-logs"></a>Échantillonnage à fréquence fixe pour les journaux ####
Vous pouvez configurer l’échantillonnage à fréquence fixe pour `AzureLogHandler` en modifiant l’argument facultatif `logging_sampling_rate`. Si aucun argument n’est fourni, un taux d’échantillonnage de 1,0 est utilisé. Un taux d’échantillonnage de 1,0 représente 100 %, ce qui signifie que toutes vos requêtes sont envoyées sous forme de télémétrie à Application Insights.

```python
handler = AzureLogHandler(
    instrumentation_key='00000000-0000-0000-0000-000000000000',
    logging_sampling_rate=0.5,
)
```

### <a name="configuring-fixed-rate-sampling-for-web-pages-with-javascript"></a>Configuration de l’échantillonnage à fréquence fixe pour des pages web avec JavaScript

Les pages web basées sur JavaScript peuvent être configurées pour utiliser Application Insights. La télémétrie est envoyée à partir de l’application cliente qui s’exécute dans le navigateur de l’utilisateur, et les pages peuvent être hébergées à partir de n’importe quel serveur.

Quand vous [configurez vos pages web basées sur JavaScript pour Application Insights](javascript.md), modifiez l’extrait de code JavaScript que vous obtenez auprès du portail Application Insights.

> [!TIP]
> Dans les applications ASP.NET avec du JavaScript, l’extrait de code va généralement dans `_Layout.cshtml`.

Insérez une ligne de type `samplingPercentage: 10,` avant la clé d’instrumentation :

```xml
<script>
    var appInsights = // ... 
    ({ 
      // Value must be 100/N where N is an integer.
      // Valid examples: 50, 25, 20, 10, 5, 1, 0.1, ...
      samplingPercentage: 10, 

      instrumentationKey: ...
    }); 

    window.appInsights = appInsights; 
    appInsights.trackPageView(); 
</script>
```

Pour le pourcentage d’échantillonnage, choisissez un pourcentage proche de 100/N, où N est un nombre entier. L’échantillonnage ne prend actuellement en charge aucune autre valeur.

#### <a name="coordinating-server-side-and-client-side-sampling"></a>Coordination de l’échantillonnage entre le côté serveur et le côté client

Le SDK JavaScript côté client participe à l’échantillonnage à fréquence fixe conjointement avec le SDK côté serveur. Les pages instrumentées envoient seulement les données de télémétrie côté client provenant du même utilisateur pour lequel le SDK côté serveur a décidé d’échantillonner. Cette logique est conçue pour préserver l’intégrité des sessions utilisateur entre les applications côté client et côté serveur. En partant de n’importe quel élément de télémétrie dans Application Insights, vous trouverez donc tous les autres éléments de télémétrie pour cet utilisateur ou cette session, et dans Recherche, vous pouvez naviguer entre les vues et les requêtes associées de la page.

Si vos données de télémétrie côté client et côté serveur ne montrent pas d’échantillons coordonnés :

* Vérifiez que vous avez activé l’échantillonnage à taux fixe à la fois sur le serveur et sur le client.
* Vérifiez que vous définissez le même pourcentage d’échantillonnage dans le client et dans le serveur.
* Assurez-vous que vous utilisez bien le kit de développement logiciel version 2.0 ou ultérieure.

## <a name="ingestion-sampling"></a>échantillonnage d’ingestion

L’échantillonnage d’ingestion fonctionne au niveau où les données de télémétrie issues de votre serveur web, des navigateurs et des appareils atteignent le point de terminaison du service Application Insights. Bien qu’elle ne réduise pas le trafic des données de télémétrie envoyées à partir de votre application, elle réduit la quantité traitée et conservée (et facturée) par Application Insights.

Utilisez ce type d’échantillonnage si votre application dépasse souvent son quota mensuel et que vous ne pouvez recourir à aucun type d’échantillonnage basé sur le Kit de développement logiciel (SDK). 

Définissez le taux d’échantillonnage dans la page Utilisation et estimation des coûts :

![Dans le panneau Vue d’ensemble de l’application, cliquez sur Paramètres, Quota, Échantillons, sélectionnez un taux d’échantillonnage, puis cliquez sur Mettre à jour.](./media/sampling/data-sampling.png)

Comme d’autres types d’échantillonnage, l’algorithme conserve les éléments de télémétrie associés. Par exemple, quand vous inspectez les données de télémétrie dans Search, vous pouvez trouver la demande liée à une exception spécifique. Les données de mesure telles que les taux de demandes et le taux d’exceptions sont correctement conservées.

Les points de données ignorés par l’échantillonnage ne sont disponibles dans aucune fonctionnalité Application Insights, par exemple l’ [exportation continue](./export-telemetry.md).

L’échantillonnage d’ingestion ne fonctionne pas pendant que l’échantillonnage adaptatif ou à fréquence fixe est utilisé. L’échantillonnage adaptatif est activé par défaut quand le SDK ASP.NET ou le SDK ASP.NET Core est utilisé, ou quand Application Insights est activé dans [Azure App Service ](azure-web-apps.md) ou avec Status Monitor. Quand des données de télémétrie sont reçues par le point de terminaison de service Application Insights, celui-ci examine la télémétrie et, si le taux d’échantillonnage est signalé comme étant inférieur à 100 % (ce qui indique que la télémétrie est échantillonnée), le taux d’échantillonnage d’ingestion que vous avec défini est ignoré.

> [!WARNING]
> La valeur affichée sur la vignette du portail indique la valeur que vous avec définie pour l’échantillonnage d’ingestion. Il ne représente pas la fréquence d’échantillonnage réelle si un échantillonnage du SDK (adaptatif ou à fréquence fixe) est effectué.

## <a name="when-to-use-sampling"></a>Quand utiliser l’échantillonnage

En général, pour la plupart des applications petites et moyennes, vous n’avez pas besoin de l’échantillonnage. Pour obtenir les informations de diagnostic les plus utiles et les statistiques les plus précises, le mieux est de collecter les données sur toutes vos activités utilisateur. 

Les principaux avantages de l’échantillonnage sont les suivants :

* Le service Application Insights supprime (« limite ») des points de données quand votre application envoie un taux de télémétrie très élevé dans un court laps de temps. L’échantillonnage réduit la probabilité que votre application subisse des limitations.
* Pour respecter le [quota](pricing.md) de points de données pour votre niveau tarifaire. 
* Pour réduire le trafic réseau généré par la collecte de données de télémétrie. 

### <a name="which-type-of-sampling-should-i-use"></a>Quel type d’échantillonnage dois-je utiliser ?

**Utiliser l’échantillonnage d’ingestion dans les situations suivantes :**

* Vous utilisez souvent votre quota mensuel de données de télémétrie.
* Vous obtenez un volume trop élevé de données de télémétrie des navigateurs web de vos utilisateurs.
* Vous utilisez une version du Kit SDK qui ne prend pas en charge l’échantillonnage, par exemple, une version d’ASP.NET antérieure à la version 2.

**Utilisez l’échantillonnage à débit fixe si :**

* Vous voulez un échantillonnage synchronisé entre le client et le serveur afin de pouvoir naviguer entre les événements associés sur le client et le serveur (comme les vues et les requêtes HTTP de la page) quand vous examinez des événements dans [Recherche](./diagnostic-search.md).
* Vous êtes sûr du pourcentage d’échantillonnage approprié pour votre application. Il doit être suffisamment élevé pour obtenir des mesures précises, mais inférieur au pourcentage engendrant le dépassement de votre quota de tarification et des limites de limitation.

**Utilisez l’échantillonnage adaptatif :**

Si les conditions d’utilisation des autres formes d’échantillonnage ne s’appliquent pas, nous vous recommandons l’échantillonnage adaptatif. Ce paramètre est activé par défaut dans le SDK ASP.NET/ASP.NET Core. Il ne réduit le trafic que si un certain taux minimal est atteint : ainsi, les sites peu utilisés ne seront probablement pas échantillonnés du tout.

## <a name="knowing-whether-sampling-is-in-operation"></a>Savoir si l’échantillonnage est en cours

Pour découvrir le taux d’échantillonnage réel, indépendamment de l’endroit où il a été appliqué, utilisez une [requête Analytics](../log-query/log-query-overview.md) telle que celle-ci :

```kusto
union requests,dependencies,pageViews,browserTimings,exceptions,traces
| where timestamp > ago(1d)
| summarize RetainedPercentage = 100/avg(itemCount) by bin(timestamp, 1h), itemType
```

Si vous constatez que `RetainedPercentage` pour n’importe quel type est inférieur à 100, c’est que ce type de télémétrie est échantillonné.

> [!IMPORTANT]
> Application Insights n’échantillonne pas les données de télémétrie des sessions, des métriques (y compris les métriques personnalisées) et des compteurs de performances selon ces techniques d’échantillonnage. Ces types sont toujours exclus de l’échantillonnage, car une réduction de la précision peut être non souhaitable pour ces types de données de télémétrie.

## <a name="how-sampling-works"></a>Fonctionnement de l’échantillonnage

L’algorithme d’échantillonnage détermine les éléments de télémétrie à supprimer ainsi que ceux à conserver. C’est vrai si l’échantillonnage est effectué par le SDK ou dans le service Application Insights. La décision d’échantillonnage est fondée sur plusieurs règles visant à préserver l’intégrité de tous les points de données reliés entre eux, en conservant dans Application Insights une expérience de diagnostic qui demeure exploitable et fiable, même avec un jeu de données réduit. Par exemple, si votre application a une demande ayant échoué incluse dans un exemple, les éléments de télémétrie supplémentaires (comme l’exception et les traces journalisées pour cette demande) sont conservés. L’échantillonnage les conserve ou les supprime tous ensemble. C’est pourquoi, lorsque vous examinez les détails de la requête dans Application Insights, la requête s’affichera toujours avec les éléments de télémétrie qui lui sont associés.

La décision d’échantillonnage repose sur l’identifiant d’opération de la requête, ce qui signifie que tous les éléments de télémétrie appartenant à une opération particulière sont conservés ou supprimés. Pour les éléments de télémétrie qui ne sont pas associés à un ID d’opération (comme des éléments de télémétrie provenant de threads asynchrones sans contexte HTTP), l’échantillonnage capture simplement un pourcentage d’éléments de télémétrie de chaque type.

Lorsque les données de télémétrie vous sont restituées, le service Application Insights ajuste les mesures en fonction du même pourcentage d’échantillonnage que celui utilisé au moment de la collecte, de manière à compenser les points de données manquants. Par conséquent, lorsqu’ils consultent les données de télémétrie dans Application Insights, les utilisateurs constatent des approximations correctes d’un point de vue statistique et très proches des chiffres réels.

La précision de l’approximation dépend en grande partie du pourcentage d’échantillonnage configuré. Elle est également plus précise pour les applications qui gèrent un grand nombre de requêtes globalement similaires générées par un grand nombre d’utilisateurs. En revanche, pour les applications qui ne sont pas soumises à une charge importante, l’échantillonnage n’est pas nécessaire, car ces applications peuvent généralement envoyer toutes leurs données de télémétrie sans dépasser leur quota ni entraîner de perte de données liée à une limitation de la bande passante. 

## <a name="frequently-asked-questions"></a>Forum aux questions

*Quel est le comportement par défaut de l’échantillonnage dans les kits SDK ASP.NET et ASP.NET Core ?*

* Si vous utilisez une des dernières versions des SDK ci-dessus, l’échantillonnage adaptatif est activé par défaut avec cinq éléments de télémétrie par seconde.
  Deux nœuds `AdaptiveSamplingTelemetryProcessor` sont ajoutés par défaut : un inclut le type `Event` dans l’échantillonnage, tandis que l’autre exclut le type `Event` de l’échantillonnage. Cette configuration signifie que le SDK tente de limiter les éléments de télémétrie à cinq éléments de télémétrie de types `Event` et cinq éléments de télémétrie de tous les autres types combinés, ce qui garantit que les `Events` sont échantillonnés séparément des autres types de données de télémétrie. Les événements sont généralement utilisés pour la télémétrie métier et ne seront probablement pas affectés par les volumes de télémétrie de diagnostic.
  
  L’exemple suivant montre le fichier `ApplicationInsights.config` généré par défaut. Dans ASP.NET Core, le même comportement par défaut est activé dans le code. Utilisez les [exemples de la section précédente de cette page](#configuring-adaptive-sampling-for-aspnet-core-applications) pour changer ce comportement par défaut.

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <ExcludedTypes>Event</ExcludedTypes>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <IncludedTypes>Event</IncludedTypes>
        </Add>
    </TelemetryProcessors>
    ```

*Les données de télémétrie peuvent-elles être échantillonnées plusieurs fois ?*

* Non. Les SamplingTelemetryProcessor ignorent les éléments qui ont déjà été échantillonnés. Cela vaut également pour l’échantillonnage d’ingestion, qui n’applique pas l’échantillonnage aux éléments déjà échantillonnés dans le SDK lui-même.

*Pourquoi l’échantillonnage ne permet-il pas simplement de « collecter X % de chaque type de télémétrie » ?*

* Bien que cette approche de l’échantillonnage puisse produire un haut niveau de précision dans les approximations des métriques, elle ne permettrait pas de mettre en corrélation les données de diagnostic par utilisateur, par session et par requête, ce qui est essentiel pour l’établissement de diagnostics. L’échantillonnage est donc plus efficace avec des stratégies comme « collecter tous les éléments de télémétrie pour X % des utilisateurs de l’application » ou « collecter tous les éléments de télémétrie pour X % des requêtes de l’application ». Pour les éléments de télémétrie qui ne sont pas associés aux requêtes (dans le cas, par exemple, d’un traitement asynchrone en arrière-plan), la solution consiste à « collecter X % de tous les éléments de chaque type de télémétrie ». 

*Le pourcentage d’échantillonnage peut-il évoluer au fil du temps ?*

* Oui, l’échantillonnage adaptatif modifie progressivement le pourcentage d’échantillonnage en fonction du volume de données de télémétrie constaté.

*Si j’utilise l’échantillonnage à débit fixe, comment déterminer le pourcentage d’échantillonnage optimal pour mon application ?*

* Pour cela, vous pouvez commencer par l’échantillonnage adaptatif, identifier le taux obtenu (voir la question précédente) et passer à l’échantillonnage à taux fixe en utilisant ce taux. 
  
    Autrement, vous devez procéder par tâtonnements. Analysez votre utilisation actuelle des données de télémétrie dans Application Insights, observez les limitations qui s’appliquent, puis estimez le volume de données de télémétrie collectées. Ces trois entrées, combinées au niveau tarifaire sélectionné, vous indiquent dans quelle mesure vous devrez réduire le volume de données de télémétrie collectées. Toutefois, une augmentation du nombre d’utilisateurs ou toute autre modification au niveau du volume des données de télémétrie peut invalider votre estimation.

*Que se passe-t-il si je configure le pourcentage d’échantillonnage à un niveau trop faible ?*

* Des pourcentages d’échantillonnage excessivement bas provoquent un échantillonnage trop agressif et réduisent la précision des approximations quand Application Insights tente de compenser la visualisation des données pour tenir compte de la réduction du volume de données. Cela peut également affecter votre expérience de diagnostic puisque l’échantillonnage peut inclure certaines requêtes rarement affectées par des échecs ou des ralentissements.

*Que se passe-t-il si je configure le pourcentage d’échantillonnage à un niveau trop élevé ?*

* La configuration d’un pourcentage d’échantillonnage trop élevé (c’est-à-dire pas assez agressif) aboutit à une réduction insuffisante du volume de données de télémétrie collectées. Vous risquez de perdre des données de télémétrie sous l’effet de la limitation de bande passante et de supporter des coûts plus élevés que prévu pour l’utilisation d’Application Insights en raison des frais de dépassement.

*Sur quelles plates-formes puis-je utiliser l’échantillonnage ?*

* L’échantillonnage d’ingestion peut se produire automatiquement pour toute télémétrie au-dessus d’un certain volume, si le Kit de développement logiciel (SDK)n’effectue pas d’échantillonnage. Cette configuration fonctionne par exemple si vous utilisez une version antérieure du SDK ASP.NET ou du SDK Java.
* Si vous utilisez les kits SDK ASP.NET ASP.NET Core (hébergés dans Azure ou sur votre propre serveur), vous obtenez l’échantillonnage adaptatif par défaut, mais vous pouvez passer à l’échantillonnage à fréquence fixe comme décrit ci-dessus. Avec l’échantillonnage à taux fixe, le Kit de développement logiciel (SDK) du navigateur se synchronise automatiquement pour échantillonner les événements connexes. 
* Si vous utilisez l’agent Java actuel, vous pouvez configurer `applicationinsights.json` (pour le SDK Java, configurez `ApplicationInsights.xml`) pour activer l’échantillonnage à fréquence fixe. L’échantillonnage est désactivé par défaut. Avec l’échantillonnage à fréquence fixe, le SDK du navigateur et le serveur se synchronisent automatiquement pour échantillonner les événements associés.

*Je souhaite que certains événements rares soient toujours affichés. Comment faire en sorte qu’ils soient disponibles hors du module d’échantillonnage ?*

* La meilleure façon d’y parvenir consiste à écrire un [TelemetryInitializer](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) personnalisé qui définit le `SamplingPercentage` sur 100 sur l’élément de télémétrie que vous souhaitez conserver, comme indiqué ci-dessous. Les initialiseurs étant assurés d’être exécutés avant les processeurs de télémétrie (échantillonnage compris), toutes les techniques d’échantillonnage ignorent cet élément des considérations d’échantillonnage. Des initialiseurs de données de télémétrie personnalisés sont disponibles dans les kits SDK ASP.NET, ASP.NET Core, JavaScript et Java. Par exemple, vous pouvez configurer un initialiseur de données de télémétrie à l’aide du SDK ASP.NET :

    ```csharp
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if(somecondition)
            {
                ((ISupportSampling)telemetry).SamplingPercentage = 100;
            }
        }
    }
    ```

## <a name="older-sdk-versions"></a>Versions antérieures du SDK

L’échantillonnage adaptatif est disponible pour le SDK Application Insights pour ASP.NET v2.0.0-beta3 et ultérieur, et pour le SDK Microsoft.ApplicationInsights.AspNetCore v2.2.0-beta1 et ultérieur, et il est activé par défaut.

L’échantillonnage à fréquence fixe est une fonctionnalités du SDK dans ASP.NET à partir de la version 2.0.0 et du SDK Java à partir de la version 2.0.1.

Avant la version 2.5.0-beta2 du SDK .NET et la version 2.2.0-beta3 du SDK ASP.NET Core, le choix de l’échantillonnage reposait sur le hachage de l’ID utilisateur pour les applications qui définissent « l’utilisateur » (c’est-à-dire les applications web les plus courantes). Pour les types d’applications qui ne définissaient pas les utilisateurs (tels que les services Web), la décision d’échantillonnage reposait sur l’identifiant d’opération de la requête. Les versions récentes des kits SDK ASP.NET et ASP.NET Core utilisent l’ID d’opération pour décider de l’échantillonnage.

## <a name="next-steps"></a>Étapes suivantes

* [filtrage](./api-filtering-sampling.md) peut fournir un contrôle plus strict de ce que le Kit de développement logiciel (SDK) envoie.
* Lisez l’article Developer Network [Optimiser la télémétrie avec Application Insights](/archive/msdn-magazine/2017/may/devops-optimize-telemetry-with-application-insights).

