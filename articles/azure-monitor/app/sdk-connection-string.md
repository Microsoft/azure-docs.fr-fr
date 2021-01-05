---
title: Chaînes de connexion dans Azure Application Insights | Microsoft Docs
description: Guide pratique pour utiliser des chaînes de connexion.
ms.topic: conceptual
author: timothymothra
ms.author: tilee
ms.date: 01/17/2020
ms.custom: devx-track-js, devx-track-csharp
ms.reviewer: mbullwin
ms.openlocfilehash: f4227c28329233c7f414c6c45e4a3c1420bf47be
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91335158"
---
# <a name="connection-strings"></a>Chaînes de connexion

## <a name="overview"></a>Vue d’ensemble

Les chaînes de connexion fournissent aux utilisateurs Application Insights un seul paramètre de configuration, ce qui évite d’avoir à utiliser plusieurs paramètres de proxy. Elles s’avèrent très utiles aux serveurs web intranet et aux environnements cloud souverains ou hybrides qui cherchent à envoyer des données au service de supervision.

Les paires clé-valeur permettent aux utilisateurs de définir facilement une combinaison préfixe-suffixe pour chaque service/produit Application Insights (AI).

> [!IMPORTANT]
> Nous déconseillons de définir à la fois une chaîne de connexion et une clé d’instrumentation. Au cas où un utilisateur définirait les deux, la valeur définie en dernier prévaut. 


## <a name="scenario-overview"></a>Présentation du scénario 

Scénarios client dans lesquels nous visualisons l’impact le plus important :

- Exceptions de pare-feu ou redirections de proxy 

    Quand la supervision d’un serveur web intranet est nécessaire, notre solution antérieure demandait aux clients d’ajouter des points de terminaison de service individuels à leur configuration. Vous pourrez trouver plus d’informations [ici](../faq.md#can-i-monitor-an-intranet-web-server). 
    Les chaînes de connexion offrent une meilleure alternative en réduisant cet effort à un seul paramètre. La simple modification d’un préfixe et d’un suffixe permet de renseigner tous les points de terminaison et de les rediriger automatiquement vers les services appropriés. 

- Environnements cloud souverains ou hybrides

    Les utilisateurs peuvent envoyer des données à une [région Azure Government](../../azure-government/compare-azure-government-global-azure.md#application-insights) définie.
    Les chaînes de connexion vous permettent de définir des paramètres de point de terminaison pour vos serveurs intranet ou des paramètres cloud hybrides. 

## <a name="getting-started"></a>Prise en main

### <a name="finding-my-connection-string"></a>Où trouver ma chaîne de connexion ?

Votre chaîne de connexion s’affiche dans le panneau Vue d’ensemble de votre ressource Application Insights.

![Chaîne de connexion dans le panneau Vue d’ensemble](media/overview-dashboard/overview-connection-string.png)

### <a name="schema"></a>schéma

#### <a name="max-length"></a>Longueur maximale

La longueur maximale prise en charge pour la connexion s’élève à 4 096 caractères.

#### <a name="key-value-pairs"></a>Paires clé-valeur

La chaîne de connexion se compose d’une liste de paramètres représentés sous forme de paires clé-valeur séparées par un point-virgule : `key1=value1;key2=value2;key3=value3`

#### <a name="syntax"></a>Syntaxe

- `InstrumentationKey` (exemple : 00000000-0000-0000-0000-000000000000) La chaîne de connexion est un champ **obligatoire**.
- `Authorization` (exemple : ikey) (Ce paramètre est facultatif car, pour le moment, nous ne prenons en charge que l’autorisation ikey.)
- `EndpointSuffix` (exemple : applicationinsights.azure.cn) La définition du suffixe du point de terminaison indique au SDK à quel cloud Azure se connecter. Le SDK assemble le reste du point de terminaison pour les services individuels.
- Points de terminaison explicites.
  Tout service peut être substitué explicitement dans la chaîne de connexion.
   - `IngestionEndpoint` (exemple : `https://dc.applicationinsights.azure.com`)
   - `LiveEndpoint` (exemple : `https://live.applicationinsights.azure.com`)
   - `ProfilerEndpoint` (exemple : `https://profiler.applicationinsights.azure.com`)
   - `SnapshotEndpoint` (exemple : `https://snapshot.applicationinsights.azure.com`)

#### <a name="endpoint-schema"></a>Schéma de point de terminaison

`<prefix>.<suffix>`
- Préfixe : Définit un service. 
- Suffixe : Définit le nom de domaine commun.

##### <a name="valid-suffixes"></a>Suffixes valides

Voici la liste des suffixes valides. 
- applicationinsights.azure.cn
- applicationinsights.us


Voir aussi : https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification


##### <a name="valid-prefixes"></a>Préfixes valides

- [Ingestion de données de télémétrie](./app-insights-overview.md) : `dc`
- [Métriques temps réel](./live-stream.md) : `live`
- [Profiler](./profiler-overview.md): `profiler`
- [Capture instantanée](./snapshot-debugger.md) : `snapshot`



## <a name="connection-string-examples"></a>Exemples de chaînes de connexion courantes


### <a name="minimal-valid-connection-string"></a>Chaîne de connexion valide minimale

`InstrumentationKey=00000000-0000-0000-0000-000000000000;`

Dans cet exemple, seule la clé d’instrumentation a été définie.

- Le schéma d’autorisation est défini par défaut sur « ikey » 
- Clé d’instrumentation : 00000000-0000-0000-0000-000000000000
- Les URI de services régionaux se basent sur les [valeurs par défaut du SDK](https://github.com/microsoft/ApplicationInsights-dotnet/blob/e50d569cebf485e72e98f4a08a0bc0e30cdf42bc/BASE/src/Microsoft.ApplicationInsights/Extensibility/Implementation/Endpoints/Constants.cs#L6) et se connectent au service Azure global public :
   - Ingestion : `https://dc.services.visualstudio.com/`
   - Métriques temps réel : `https://rt.services.visualstudio.com/`
   - Profiler : `https://agent.azureserviceprofiler.net/`
   - Débogueur : `https://agent.azureserviceprofiler.net/`



### <a name="connection-string-with-endpoint-suffix"></a>Chaîne de connexion avec suffixe de point de terminaison

`InstrumentationKey=00000000-0000-0000-0000-000000000000;EndpointSuffix=ai.contoso.com;`

Dans cet exemple, cette chaîne de connexion spécifie le suffixe du point de terminaison et le SDK construit les points de terminaison de service.

- Le schéma d’autorisation est défini par défaut sur « ikey » 
- Clé d’instrumentation : 00000000-0000-0000-0000-000000000000
- Les URI de services régionaux se basent sur le suffixe de point de terminaison fourni : 
   - Ingestion : `https://dc.ai.contoso.com`
   - Métriques temps réel : `https://live.ai.contoso.com`
   - Profiler : `https://profiler.ai.contoso.com`
   - Débogueur : `https://snapshot.ai.contoso.com`  



### <a name="connection-string-with-explicit-endpoint-overrides"></a>Chaîne de connexion avec des substitutions de points de terminaison explicites 

`InstrumentationKey=00000000-0000-0000-0000-000000000000;IngestionEndpoint=https://custom.com:111/;LiveEndpoint=https://custom.com:222/;ProfilerEndpoint=https://custom.com:333/;SnapshotEndpoint=https://custom.com:444/;`

Dans cet exemple, cette chaîne de connexion spécifie des substitutions explicites pour chaque service. Le SDK utilise strictement les points de terminaison fournis sans apporter aucune modification.

- Le schéma d’autorisation est défini par défaut sur « ikey » 
- Clé d’instrumentation : 00000000-0000-0000-0000-000000000000
- Les URI de services régionaux se basent sur les valeurs de remplacement explicites : 
   - Ingestion : `https://custom.com:111/`
   - Métriques temps réel : `https://custom.com:222/`
   - Profiler : `https://custom.com:333/`
   - Débogueur : `https://custom.com:444/`  


## <a name="how-to-set-a-connection-string"></a>Comment définir une chaîne de connexion

Les chaînes de connexion sont prises en charge dans les versions de SDK suivantes :
- .NET et .NET Core v2.12.0
- Java v2.5.1 et Java 3.0
- JavaScript v2.3.0
- NodeJS v1.5.0
- Python v1.0.0

Une chaîne de connexion peut être définie dans du code, une variable d’environnement ou un fichier de configuration.



### <a name="environment-variable"></a>Variable d’environnement

- Chaîne de connexion : `APPLICATIONINSIGHTS_CONNECTION_STRING`

# <a name="netnetcore"></a>[.NET/.NetCore](#tab/net)

TelemetryConfiguration.ConnectionString : https://github.com/microsoft/ApplicationInsights-dotnet/blob/add45ceed35a817dc7202ec07d3df1672d1f610d/BASE/src/Microsoft.ApplicationInsights/Extensibility/TelemetryConfiguration.cs#L271-L274

.NET défini explicitement :
```csharp
var configuration = new TelemetryConfiguration
{
    ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
};
```

Fichier de configuration .NET :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000</ConnectionString>
</ApplicationInsights>
```

NetCore défini explicitement :
```csharp
public void ConfigureServices(IServiceCollection services)
{
    var options = new ApplicationInsightsServiceOptions { ConnectionString = "InstrumentationKey=00000000-0000-0000-0000-000000000000;" };
    services.AddApplicationInsightsTelemetry(options: options);
}
```

NetCore config.json : 

```json
{
  "ApplicationInsights": {
    "ConnectionString" : "InstrumentationKey=00000000-0000-0000-0000-000000000000;"
    }
  }
```


# <a name="java"></a>[Java](#tab/java)


Java (v2.5.x) défini explicitement :
```java
TelemetryConfiguration.getActive().setConnectionString("InstrumentationKey=00000000-0000-0000-0000-000000000000");
```

ApplicationInsights.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
    <ConnectionString>InstrumentationKey=00000000-0000-0000-0000-000000000000;</ConnectionString>
</ApplicationInsights>
```

# <a name="javascript"></a>[JavaScript](#tab/js)

Important : JavaScript ne prend pas en charge l’utilisation de variables d’environnement.

Utilisation de l’extrait de code :

```javascript
<script type="text/javascript">
var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("addTelemetryInitializer"),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),t.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4},!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
{
  connectionString:"InstrumentationKey=00000000-0000-0000-0000-000000000000;"
}
);window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```


Installation manuelle :
```javascript
import { ApplicationInsights } from '@microsoft/applicationinsights-web'

const appInsights = new ApplicationInsights({ config: {
  connectionString: 'InstrumentationKey=00000000-0000-0000-0000-000000000000;'
  /* ...Other Configuration Options... */
} });
appInsights.loadAppInsights();
appInsights.trackPageView();
```

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

```javascript
const appInsights = require("applicationinsights");
appInsights.setup("InstrumentationKey=00000000-0000-0000-0000-000000000000;");
appInsights.start();
```

# <a name="python"></a>[Python](#tab/python)

Nous recommandons aux utilisateurs de définir la variable d’environnement.

Pour définir explicitement la chaîne de connexion :

```python
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace.samplers import ProbabilitySampler
from opencensus.trace.tracer import Tracer

tracer = Tracer(exporter=AzureExporter(connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'), sampler=ProbabilitySampler(1.0))
```


## <a name="next-steps"></a>Étapes suivantes

Prise en main lors de l’exécution avec :

* [Applications hébergées par IIS sur une machine virtuelle Azure et un groupe de machines virtuelles identiques Azure](./azure-vm-vmss-apps.md)
* [Serveur IIS](./monitor-performance-live-website-now.md)
* [Azure Web Apps](./azure-web-apps.md)

Prise en main pendant le développement avec :

* [ASP.NET](./asp-net.md)
* [ASP.NET Core](./asp-net-core.md)
* [Java](./java-get-started.md)
* [Node.JS](./nodejs.md)
* [Python](./opencensus-python.md)

