---
title: Filtrage et pré-traitement dans le Kit de développement logiciel (SDK) Application Insights | Microsoft Docs
description: Écrivez des processeurs de télémétrie et des initialiseurs de télémétrie que le Kit de développement logiciel (SDK) doit filtrer ou ajoutez des propriétés aux données avant que la télémétrie ne soit envoyée au portail Application Insights.
ms.topic: conceptual
ms.date: 11/23/2016
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: b3ccc3516d5b31f4c119c2d5a2bd11a63dbdc611
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91758028"
---
# <a name="filter-and-preprocess-telemetry-in-the-application-insights-sdk"></a>Filtrage et pré-traitement de la télémétrie dans le Kit de développement logiciel (SDK) Application Insights

Vous pouvez écrire et configurer des plug-ins pour le kit de développement logiciel (SDK) Application Insights afin de personnaliser l'enrichissement et le traitement des données de télémétrie avant leur envoi au service Application Insights.

* [échantillonnage](sampling.md) réduit le volume des données de télémétrie sans affecter les statistiques. Il maintient ensemble les points de données liés de sorte que vous pouvez naviguer entre eux pour diagnostiquer un problème. Dans le portail, les nombres totaux sont multipliés pour compenser l'échantillonnage.
* Le filtrage avec des processeurs de télémétrie vous permet de filtrer la télémétrie dans le kit de développement logiciel (SDK) avant l’envoi au serveur. Par exemple, vous pouvez réduire le volume de la télémétrie en excluant les demandes émanant de robots. Le filtrage est une approche plus simple que l’échantillonnage pour réduire le trafic. Il vous offre plus de contrôle sur ce qui est transmis, mais affecte vos statistiques. Par exemple, vous pouvez filtrer toutes les requêtes réussies.
* [Les initialiseurs de télémétrie ajoutent ou modifient des propriétés](#add-properties) à n’importe quelle télémétrie envoyée à partir de votre application, notamment les données de télémétrie fournies par les modules standard. Par exemple, vous pouvez ajouter des valeurs calculées ou des numéros de version permettant de filtrer les données dans le portail.
* [L'API SDK](./api-custom-events-metrics.md) est utilisée pour envoyer des événements et des mesures personnalisés.

Avant de commencer :

* Installez le kit de développement logiciel (SDK) correspondant à votre application : [ASP.NET](asp-net.md), [ASP.NET Core](asp-net-core.md), [Non HTTP/Worker for .NET/.NET Core](worker-service.md) ou [JavaScript](javascript.md).

<a name="filtering"></a>

## <a name="filtering"></a>Filtrage

Cette technique vous offre un contrôle direct sur ce qui est inclus ou exclu du flux de télémétrie. Le filtrage peut être utilisé ne pas envoyer certains éléments de télémétrie à Application Insights. Vous pouvez l'utiliser conjointement avec l'échantillonnage, ou séparément.

Pour filtrer la télémétrie, vous écrivez un processeur de télémétrie et l'enregistrez avec le `TelemetryConfiguration`. Toute la télémétrie passe par votre processeur. Vous pouvez choisir de la supprimer du flux ou de la transmettre au processeur suivant dans la chaîne. Cela inclut les données de télémétrie fournies par les modules standard tels que le collecteur de requêtes HTTP et le collecteur de dépendances, ainsi que les données de télémétrie que vous avez suivies vous-même. Vous pouvez, par exemple, filtrer la télémétrie concernant les requêtes émanant de robots ou les appels de dépendance réussis.

> [!WARNING]
> Filtrer la télémétrie envoyée depuis le Kit de développement logiciel (SDK) à l’aide de processeurs peut fausser les statistiques que vous voyez dans le portail et rendre difficile le suivi des éléments associés.
>
> Envisagez plutôt d'utiliser l' [échantillonnage](./sampling.md).
>
>

### <a name="create-a-telemetry-processor-c"></a>Créer un processeur de télémétrie (C#)

1. Pour créer un filtre, implémentez `ITelemetryProcessor`.

    Les processeurs de télémétrie construisent une chaîne de traitement. Lorsque vous instanciez un processeur de télémétrie, une référence vous est donnée vers le processeur suivant dans la chaîne. Lorsqu’un point de données de télémétrie est transmis à la méthode de traitement, il effectue son travail, puis appelle (ou n'appelle pas) le processeur de télémétrie suivant dans la chaîne.

    ```csharp
    using Microsoft.ApplicationInsights.Channel;
    using Microsoft.ApplicationInsights.Extensibility;

    public class SuccessfulDependencyFilter : ITelemetryProcessor
    {
        private ITelemetryProcessor Next { get; set; }

        // next will point to the next TelemetryProcessor in the chain.
        public SuccessfulDependencyFilter(ITelemetryProcessor next)
        {
            this.Next = next;
        }

        public void Process(ITelemetry item)
        {
            // To filter out an item, return without calling the next processor.
            if (!OKtoSend(item)) { return; }

            this.Next.Process(item);
        }

        // Example: replace with your own criteria.
        private bool OKtoSend (ITelemetry item)
        {
            var dependency = item as DependencyTelemetry;
            if (dependency == null) return true;

            return dependency.Success != true;
        }
    }
    ```

2. Ajoutez votre processeur.

**Applications** ASP.NET

Insérez cet extrait de code dans ApplicationInsights.config :

```xml
<TelemetryProcessors>
  <Add Type="WebApplication9.SuccessfulDependencyFilter, WebApplication9">
     <!-- Set public property -->
     <MyParamFromConfigFile>2-beta</MyParamFromConfigFile>
  </Add>
</TelemetryProcessors>
```

Vous pouvez transférer des valeurs de chaîne depuis le fichier .config en fournissant des propriétés publiques nommées dans votre classe.

> [!WARNING]
> Veillez à faire correspondre le nom de type et les noms de propriété dans le fichier .config aux noms de classe et de propriété dans le code. Si le fichier .config fait référence à un type ou à une propriété qui n'existe pas, le kit de développement peut échouer lors de l'envoi d'une télémétrie quelconque.
>

Vous pouvez aussi initialiser le filtre dans le code. Dans une classe d’initialisation appropriée (par exemple, AppStart dans `Global.asax.cs`), insérez votre processeur dans la chaîne :

```csharp
var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
builder.Use((next) => new SuccessfulDependencyFilter(next));

// If you have more processors:
builder.Use((next) => new AnotherProcessor(next));

builder.Build();
```

Les clients de télémétrie créés après ce point utiliseront vos processeurs.

**Applications ASP.NET Core/Worker Service**

> [!NOTE]
> L’ajout du processeur à l’aide de `ApplicationInsights.config` ou `TelemetryConfiguration.Active` n’est pas valide pour les applications ASP.NET Core ou si vous utilisez le kit de développement logiciel (SDK) Microsoft.ApplicationInsights.WorkerService.

Pour les applications écrites avec [ASP.NET Core](asp-net-core.md#adding-telemetry-processors) ou [WorkerService](worker-service.md#adding-telemetry-processors), l’ajout d’un nouveau `AddApplicationInsightsTelemetryProcessor` se fait à l'aide de la méthode d'extension sur `IServiceCollection`, comme indiqué. Cette méthode est appelée dans la méthode `ConfigureServices` de votre classe `Startup.cs`.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...
        services.AddApplicationInsightsTelemetry();
        services.AddApplicationInsightsTelemetryProcessor<SuccessfulDependencyFilter>();

        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<AnotherProcessor>();
    }
```

### <a name="example-filters"></a>Exemples de filtres

#### <a name="synthetic-requests"></a>Demandes synthétiques

Excluez les robots et les tests web. Bien que Metrics Explorer vous donne la possibilité d’exclure les sources synthétiques, cette option réduit le trafic et la taille d'ingestion en les filtrant au niveau du kit de développement (SDK).

```csharp
public void Process(ITelemetry item)
{
  if (!string.IsNullOrEmpty(item.Context.Operation.SyntheticSource)) {return;}

  // Send everything else:
  this.Next.Process(item);
}
```

#### <a name="failed-authentication"></a>Échec d’authentification

Excluez les demandes avec une réponse de type « 401 ».

```csharp
public void Process(ITelemetry item)
{
    var request = item as RequestTelemetry;

    if (request != null &&
    request.ResponseCode.Equals("401", StringComparison.OrdinalIgnoreCase))
    {
        // To filter out an item, return without calling the next processor.
        return;
    }

    // Send everything else
    this.Next.Process(item);
}
```

#### <a name="filter-out-fast-remote-dependency-calls"></a>Excluez les appels de dépendance à distance rapides.

Si vous souhaitez uniquement diagnostiquer les appels lents, excluez les appels rapides.

> [!NOTE]
> Cela faussera les statistiques que vous voyez dans le portail.
>
>

```csharp
public void Process(ITelemetry item)
{
    var request = item as DependencyTelemetry;

    if (request != null && request.Duration.TotalMilliseconds < 100)
    {
        return;
    }
    this.Next.Process(item);
}
```

#### <a name="diagnose-dependency-issues"></a>Diagnostiquer les problèmes de dépendance

[Ce blog](https://azure.microsoft.com/blog/implement-an-application-insights-telemetry-processor/) décrit un projet pour diagnostiquer les problèmes de dépendance en envoyant automatiquement des requêtes ping régulières aux dépendances.

<a name="add-properties"></a>

### <a name="javascript-web-applications"></a>Applications web JavaScript

**Filtrer à l’aide de ITelemetryInitializer**

1. Créez une fonction de rappel de l’initialiseur de télémétrie. La fonction de rappel prend `ITelemetryItem` en tant que paramètre, qui est l’événement en cours de traitement. Le retour de `false` de ce rappel entraîne le filtrage de l’élément de télémétrie.

   ```JS
   var filteringFunction = (envelope) => {
     if (envelope.data.someField === 'tobefilteredout') {
         return false;
     }
  
     return true;
   };
   ```

2. Ajoutez votre rappel de l’initialiseur de télémétrie :

   ```JS
   appInsights.addTelemetryInitializer(filteringFunction);
   ```

## <a name="addmodify-properties-itelemetryinitializer"></a>Ajoutez/modifiez des propriétés : ITelemetryInitializer

Utilisez les initialiseurs de télémétrie pour enrichir la télémétrie avec des informations supplémentaires ou remplacer les propriétés de télémétrie définies par les modules de télémétrie standard.

Par exemple, le package Application Insights pour le Web collecte la télémétrie sur les requêtes HTTP. Il indique par défaut l’échec de toute requête à l’aide d’un code de réponse supérieur ou égal à 400. Toutefois, si 400 vous convient, vous pouvez fournir un initialiseur de télémétrie qui définit la propriété Success.

Si vous fournissez un initialiseur de télémétrie, celui-ci est appelé chaque fois qu'une des méthodes Track*() est appelée. Cela inclut les méthodes `Track()` appelées par les modules de télémétrie standard. Par convention, ces modules ne définissent aucune propriété déjà définie par un initialiseur. Les initialiseurs de télémétrie sont appelés préalablement aux processeurs de télémétrie. Dès lors, tout enrichissement effectué par les initialiseurs est visible des processeurs.

**Définir votre initialiseur**

*C#*

```csharp
using System;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MvcWebRole.Telemetry
{
  /*
   * Custom TelemetryInitializer that overrides the default SDK
   * behavior of treating response codes >= 400 as failed requests
   *
   */
  public class MyTelemetryInitializer : ITelemetryInitializer
  {
    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        // Is this a TrackRequest() ?
        if (requestTelemetry == null) return;
        int code;
        bool parsed = Int32.TryParse(requestTelemetry.ResponseCode, out code);
        if (!parsed) return;
        if (code >= 400 && code < 500)
        {
            // If we set the Success property, the SDK won't change it:
            requestTelemetry.Success = true;

            // Allow us to filter these requests in the portal:
            requestTelemetry.Properties["Overridden400s"] = "true";
        }
        // else leave the SDK to set the Success property
    }
  }
}
```

**Applications ASP.NET : Charger votre initialiseur**

Dans ApplicationInsights.config :

```xml
<ApplicationInsights>
  <TelemetryInitializers>
    <!-- Fully qualified type name, assembly name: -->
    <Add Type="MvcWebRole.Telemetry.MyTelemetryInitializer, MvcWebRole"/>
    ...
  </TelemetryInitializers>
</ApplicationInsights>
```

Vous pouvez également instancier l'initialiseur dans le code, par exemple dans Global.aspx.cs :

```csharp
protected void Application_Start()
{
    // ...
    TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
}
```

[Voir l’intégralité de cet exemple](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole).

**Applications ASP.NET Core/Worker service : Charger votre initialiseur**

> [!NOTE]
> L’ajout de l’initialiseur à l’aide de `ApplicationInsights.config` ou `TelemetryConfiguration.Active` n’est pas valide pour les applications ASP.NET Core ou si vous utilisez le kit de développement logiciel (SDK) Microsoft.ApplicationInsights.WorkerService.

Pour les applications écrites avec [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers) ou [WorkerService](worker-service.md#adding-telemetryinitializers), l’ajout d’un nouvel initialiseur de télémétrie se fait en l’ajoutant au conteneur d’injection de dépendance, comme indiqué. Cette opération est effectuée dans la méthode `Startup.ConfigureServices`.

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```
### <a name="javascript-telemetry-initializers"></a>Initialiseurs de télémétrie JavaScript
*JavaScript*

Insérer un initialiseur de télémétrie immédiatement après le code d’initialisation obtenu à partir du portail :

```JS
<script type="text/javascript">
    // ... initialization code
    ...({
        instrumentationKey: "your instrumentation key"
    });
    window.appInsights = appInsights;


    // Adding telemetry initializer.
    // This is called whenever a new telemetry item
    // is created.

    appInsights.addTelemetryInitializer(function (envelope) {
        var telemetryItem = envelope.data.baseData;

        // To check the telemetry items type - for example PageView:
        if (envelope.name == Microsoft.ApplicationInsights.Telemetry.PageView.envelopeType) {
            // this statement removes url from all page view documents
            telemetryItem.url = "URL CENSORED";
        }

        // To set custom properties:
        telemetryItem.properties = telemetryItem.properties || {};
        telemetryItem.properties["globalProperty"] = "boo";
        
        // To set cloud role name / instance
        envelope.tags["ai.cloud.role"] = "your role name";
        envelope.tags["ai.cloud.roleInstance"] = "your role instance";
    });

    // End of inserted code.

    appInsights.trackPageView();
</script>
```

Pour obtenir un résumé des propriétés non personnalisées disponibles dans le telemetryItem, consultez le [modèle d’exportation de données Application Insights](./export-data-model.md).

Vous pouvez ajouter autant d'initialiseurs que vous le souhaitez. Ils sont appelés dans l’ordre dans lequel ils sont ajoutés.

### <a name="opencensus-python-telemetry-processors"></a>Processeurs de télémétrie OpenCensus Python

Les processeurs de télémétrie dans OpenCensus Python sont tout simplement des fonctions de rappel appelées pour traiter les données de télémétrie avant leur exportation. La fonction de rappel doit accepter un type de données [enveloppe](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py#L86) comme paramètre. Pour filtrer les données de télémétrie à ne pas exporter, vérifiez que la fonction de rappel retourne `False`. Vous pouvez voir le schéma des types de données Azure Monitor dans les enveloppes [sur GitHub](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/protocol.py).

> [!NOTE]
> Vous pouvez modifier le `cloud_RoleName` en modifiant l’attribut `ai.cloud.role` dans le champ `tags`.

```python
def callback_function(envelope):
    envelope.tags['ai.cloud.role'] = 'new_role_name'
```

```python
# Example for log exporter
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)

# Callback function to append '_hello' to each log message telemetry
def callback_function(envelope):
    envelope.data.baseData.message += '_hello'
    return True

handler = AzureLogHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>')
handler.add_telemetry_processor(callback_function)
logger.addHandler(handler)
logger.warning('Hello, World!')
```
```python
# Example for trace exporter
import requests

from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['requests'])

# Callback function to add os_type: linux to span properties
def callback_function(envelope):
    envelope.data.baseData.properties['os_type'] = 'linux'
    return True

exporter = AzureExporter(
    connection_string='InstrumentationKey=<your-instrumentation-key-here>'
)
exporter.add_telemetry_processor(callback_function)
tracer = Tracer(exporter=exporter, sampler=ProbabilitySampler(1.0))
with tracer.span(name='parent'):
response = requests.get(url='https://www.wikipedia.org/wiki/Rabbit')
```

```python
# Example for metrics exporter
import time

from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module

stats = stats_module.stats
view_manager = stats.view_manager
stats_recorder = stats.stats_recorder

CARROTS_MEASURE = measure_module.MeasureInt("carrots",
                                            "number of carrots",
                                            "carrots")
CARROTS_VIEW = view_module.View("carrots_view",
                                "number of carrots",
                                [],
                                CARROTS_MEASURE,
                                aggregation_module.CountAggregation())

# Callback function to only export the metric if value is greater than 0
def callback_function(envelope):
    return envelope.data.baseData.metrics[0].value > 0

def main():
    # Enable metrics
    # Set the interval in seconds in which you want to send metrics
    exporter = metrics_exporter.new_metrics_exporter(connection_string='InstrumentationKey=<your-instrumentation-key-here>')
    exporter.add_telemetry_processor(callback_function)
    view_manager.register_exporter(exporter)

    view_manager.register_view(CARROTS_VIEW)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    mmap.measure_int_put(CARROTS_MEASURE, 1000)
    mmap.record(tmap)
    # Default export interval is every 15.0s
    # Your application should run for at least this amount
    # of time so the exporter will meet this interval
    # Sleep can fulfill this
    time.sleep(60)

    print("Done recording metrics")

if __name__ == "__main__":
    main()
```
Vous pouvez ajouter autant de processeurs que vous le souhaitez. Ils sont appelés dans l’ordre dans lequel ils sont ajoutés. Si un seul processeur émet une exception, cela n’impacte pas les processeurs suivants.

### <a name="example-telemetryinitializers"></a>Exemple TelemetryInitializers

#### <a name="add-a-custom-property"></a>Ajouter une propriété personnalisée

L’exemple d’initialiseur suivant ajoute une propriété personnalisée à chaque télémétrie suivie.

```csharp
public void Initialize(ITelemetry item)
{
  var itemProperties = item as ISupportProperties;
  if(itemProperties != null && !itemProperties.Properties.ContainsKey("customProp"))
    {
        itemProperties.Properties["customProp"] = "customValue";
    }
}
```

#### <a name="add-a-cloud-role-name"></a>Ajouter un nom de rôle cloud

L’exemple d’initialiseur suivant définit le nom du rôle cloud sur chaque télémétrie suivie.

```csharp
public void Initialize(ITelemetry telemetry)
{
    if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
    {
        telemetry.Context.Cloud.RoleName = "MyCloudRoleName";
    }
}
```

#### <a name="add-information-from-httpcontext"></a>Ajouter des informations à partir de HttpContext

L’exemple d’initialiseur suivant lit les données depuis [`HttpContext`](/aspnet/core/fundamentals/http-context?view=aspnetcore-3.1) et les ajoute à une instance `RequestTelemetry`. `IHttpContextAccessor` est automatiquement fourni via l’injection d'une dépendance de constructeur.

```csharp
public class HttpContextRequestTelemetryInitializer : ITelemetryInitializer
{
    private readonly IHttpContextAccessor httpContextAccessor;

    public HttpContextRequestTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        this.httpContextAccessor =
            httpContextAccessor ??
            throw new ArgumentNullException(nameof(httpContextAccessor));
    }

    public void Initialize(ITelemetry telemetry)
    {
        var requestTelemetry = telemetry as RequestTelemetry;
        if (requestTelemetry == null) return;

        var claims = this.httpContextAccessor.HttpContext.User.Claims;
        Claim oidClaim = claims.FirstOrDefault(claim => claim.Type == "oid");
        requestTelemetry.Properties.Add("UserOid", oidClaim?.Value);
    }
}
```

## <a name="itelemetryprocessor-and-itelemetryinitializer"></a>ITelemetryProcessor et ITelemetryInitializer

Quelle est la différence entre les processeurs de télémétrie et les initialiseurs de télémétrie ?

* Il existe des chevauchements dans ce que vous pouvez en faire. Les deux peuvent être utilisés pour ajouter ou modifier des propriétés de télémétrie, bien que nous vous recommandons d’utiliser des initialiseurs à cet effet.
* Les initialiseurs de télémétrie sont appelés avant les processeurs de télémétrie.
* Les initialiseurs de télémétrie peuvent être appelés plusieurs fois. Par convention, ils ne définissent aucune propriété déjà définie.
* Les initialiseurs de télémétrie permettent de remplacer ou supprimer complètement un élément de télémétrie.
* Tous les initialiseurs de télémétrie enregistrés sont assurés d’être appelés pour chaque élément de télémétrie. En ce qui concerne les processeurs de télémétrie, le kit de développement logiciel (SDK) garantit l’appel du premier processeur de télémétrie. L'appel des autres processeurs dépend des processeurs de télémétrie précédents.
* Utilisez les initialiseurs de télémétrie = pour enrichir la télémétrie avec des propriétés supplémentaires ou remplacer une télémétrie existante. Utilisez les initialiseurs de télémétrie = pour filtrer la télémétrie.

## <a name="troubleshoot-applicationinsightsconfig"></a>Résolution des problèmes liés à ApplicationInsights.config

* Vérifiez que le nom de type complet et le nom de l’assembly sont corrects.
* Vérifiez que le fichier applicationinsights.config se trouve dans votre répertoire de sortie et qu’il contient éventuellement des modifications récentes.

## <a name="reference-docs"></a>Documents de référence

* [Présentation de l’API](./api-custom-events-metrics.md)
* [Référence ASP.NET](/previous-versions/azure/dn817570(v=azure.100))

## <a name="sdk-code"></a>Code de Kit de développement logiciel (SDK)

* [Kit de développement logiciel (SDK) principal ASP.NET](https://github.com/Microsoft/ApplicationInsights-aspnetcore)
* [SDK ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Kit de développement logiciel (SDK) JavaScript](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes
* [Recherche d’événements et de journaux d’activité](./diagnostic-search.md)
* [Échantillonnage](./sampling.md)
* [Dépannage](../faq.md)

