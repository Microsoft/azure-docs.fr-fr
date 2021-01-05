---
title: Remplacer les points de terminaison par défaut des SDK Azure Application Insights
description: Modifiez les points de terminaison par défaut des SDK Azure Monitor Application Insights pour certaines régions, comme Azure Government.
ms.topic: conceptual
ms.date: 07/26/2019
ms.custom: references_regions, devx-track-js
ms.openlocfilehash: d6cea9044cd4898480fcc30532a05e6c8a407012
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91333288"
---
# <a name="application-insights-overriding-default-endpoints"></a>Remplacer les points de terminaison par défaut d’Application Insights

Pour pouvoir envoyer des données Application Insights vers certaines régions, vous devez remplacer les adresses des points de terminaison par défaut. Chaque SDK nécessite des modifications légèrement différentes, qui sont toutes décrites dans cet article. Ces modifications incluent d’adapter l’exemple de code et de remplacer les valeurs des espaces réservés `QuickPulse_Endpoint_Address`, `TelemetryChannel_Endpoint_Address` et `Profile_Query_Endpoint_address` par les adresses réelles des points de terminaison applicables à votre région. Vous trouverez à la fin de cet article des liens vers les adresses des points de terminaison pour les régions où cette configuration est requise.

> [!NOTE]
> Les [chaînes de connexion](./sdk-connection-string.md?tabs=net) sont la nouvelle méthode préférée pour définir des points de terminaison personnalisés dans Application Insights.

---

## <a name="sdk-code-changes"></a>Modification du code des SDK

# <a name="net"></a>[.NET](#tab/net)

> [!NOTE]
> Le fichier applicationinsights.config est automatiquement remplacé à chaque mise à niveau d’un SDK. Après avoir effectué une mise à niveau d’un SDK, n’oubliez pas d’entrer à nouveau les valeurs de points de terminaison propres à votre région.

```xml
<ApplicationInsights>
  ...
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <QuickPulseServiceEndpoint>Custom_QuickPulse_Endpoint_Address</QuickPulseServiceEndpoint>
    </Add>
  </TelemetryModules>
    ...
  <TelemetryChannel>
     <EndpointAddress>TelemetryChannel_Endpoint_Address</EndpointAddress>
  </TelemetryChannel>
  ...
  <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
    <ProfileQueryEndpoint>Profile_Query_Endpoint_address</ProfileQueryEndpoint>
  </ApplicationIdProvider>
  ...
</ApplicationInsights>
```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

Modifiez le fichier appsettings.json dans votre projet comme suit pour adapter le point de terminaison principal :

```json
"ApplicationInsights": {
    "InstrumentationKey": "instrumentationkey",
    "TelemetryChannel": {
      "EndpointAddress": "TelemetryChannel_Endpoint_Address"
    }
  }
```

Les valeurs des points de terminaison Live Metrics et Profile Query peuvent être définies uniquement au moyen du code. Pour remplacer les valeurs par défaut de tous les points de terminaison en modifiant le code, apportez les modifications suivantes dans la méthode `ConfigureServices` du fichier `Startup.cs` :

```csharp
using Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse; //Place at top of Startup.cs file

   services.ConfigureTelemetryModule<QuickPulseTelemetryModule>((module, o) => module.QuickPulseServiceEndpoint="QuickPulse_Endpoint_Address");

   services.AddSingleton<IApplicationIdProvider, ApplicationInsightsApplicationIdProvider>(_ => new ApplicationInsightsApplicationIdProvider() { ProfileQueryEndpoint = "Profile_Query_Endpoint_address" });

   services.AddSingleton<ITelemetryChannel>(_ => new ServerTelemetryChannel() { EndpointAddress = "TelemetryChannel_Endpoint_Address" });

    //Place in the ConfigureServices method. Place this before services.AddApplicationInsightsTelemetry("instrumentation key"); if it's present
```

# <a name="azure-functions"></a>[Azure Functions](#tab/functions)

Par Azure Functions, il est désormais recommandé d’utiliser des [chaînes de connexion](./sdk-connection-string.md?tabs=net) définies dans les paramètres d’application de la fonction. Pour accéder aux paramètres de l’application pour votre fonction à partir du volet des fonctions, sélectionnez **Paramètres** > **Configuration** > **Paramètre d’application**. 

Nom : Valeur `APPLICATIONINSIGHTS_CONNECTION_STRING` : `Connection String Value`

# <a name="java"></a>[Java](#tab/java)

Modifiez le fichier applicationinsights.xml pour changer l’adresse du point de terminaison par défaut.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
  <InstrumentationKey>ffffeeee-dddd-cccc-bbbb-aaaa99998888</InstrumentationKey>
  <TelemetryModules>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
  </TelemetryModules>
  <TelemetryInitializers>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
    <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
  </TelemetryInitializers>
  <!--Add the following Channel value to modify the Endpoint address-->
  <Channel type="com.microsoft.applicationinsights.channel.concrete.inprocess.InProcessTelemetryChannel">
  <EndpointAddress>TelemetryChannel_Endpoint_Address</EndpointAddress>
  </Channel>
</ApplicationInsights>
```

### <a name="spring-boot"></a>Spring Boot

Modifiez le fichier `application.properties` en ajoutant ce qui suit :

```yaml
azure.application-insights.channel.in-process.endpoint-address= TelemetryChannel_Endpoint_Address
```

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

```javascript
var appInsights = require("applicationinsights");
appInsights.setup('INSTRUMENTATION_KEY');
appInsights.defaultClient.config.endpointUrl = "TelemetryChannel_Endpoint_Address"; // ingestion
appInsights.defaultClient.config.profileQueryEndpoint = "Profile_Query_Endpoint_address"; // appid/profile lookup
appInsights.defaultClient.config.quickPulseHost = "QuickPulse_Endpoint_Address"; //live metrics
appInsights.Configuration.start();
```

Les points de terminaison peuvent également être configurés à l’aide de variables d’environnement :

```
Instrumentation Key: "APPINSIGHTS_INSTRUMENTATIONKEY"
Profile Endpoint: "Profile_Query_Endpoint_address"
Live Metrics Endpoint: "QuickPulse_Endpoint_Address"
```

# <a name="javascript"></a>[JavaScript](#tab/js)

```javascript
<script type="text/javascript">
    var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
    {
      instrumentationKey:"INSTRUMENTATION_KEY",
      endpointUrl: "TelemetryChannel_Endpoint_Address"
    }
    );window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```

# <a name="python"></a>[Python](#tab/python)

Pour obtenir des conseils sur la modification du point de terminaison d’ingestion pour le SDK opencensus-python, consultez le [référentiel opencensus-python.](https://github.com/census-instrumentation/opencensus-python/blob/af284a92b80bcbaf5db53e7e0813f96691b4c696/contrib/opencensus-ext-azure/opencensus/ext/azure/common/__init__.py)

---

## <a name="regions-that-require-endpoint-modification"></a>Régions nécessitant une modification des points de terminaison

Actuellement, seules les régions [Azure Government](../../azure-government/compare-azure-government-global-azure.md#application-insights) et [Azure Chine](/azure/china/resources-developer-guide) nécessitent des modifications de leurs points de terminaison.

|Région |  Nom du point de terminaison | Valeur |
|-----------------|:------------|:-------------|
| Azure Chine | Canal de télémétrie | `https://dc.applicationinsights.azure.cn/v2/track` |
| Azure Chine | QuickPulse (Live Metrics) |`https://live.applicationinsights.azure.cn/QuickPulseService.svc` |
| Azure Chine | Profile Query |`https://dc.applicationinsights.azure.cn/api/profiles/{0}/appId`  |
| Azure Government | Canal de télémétrie |`https://dc.applicationinsights.us/v2/track` |
| Azure Government | QuickPulse (Live Metrics) |`https://quickpulse.applicationinsights.us/QuickPulseService.svc` |
| Azure Government | Profile Query |`https://dc.applicationinsights.us/api/profiles/{0}/appId` |

Si vous utilisez actuellement l’[Application Insights API REST](https://dev.applicationinsights.io/
), normalement accessible via « api.applicationinsights.io », vous aurez besoin d’utiliser un point de terminaison local à votre région :

|Région |  Nom du point de terminaison | Valeur |
|-----------------|:------------|:-------------|
| Azure Chine | API REST | `api.applicationinsights.azure.cn` |
| Azure Government | API REST | `api.applicationinsights.us`|

> [!NOTE]
> La supervision par extension/agent sans code pour Azure App Services **n’est pas prise en charge actuellement** dans ces régions. Dès que cette fonctionnalité sera disponible, le présent article sera mis à jour.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur les modifications personnalisées relatives à la région Azure Government, consultez les instructions détaillées sur la [supervision et gestion Azure](../../azure-government/compare-azure-government-global-azure.md#application-insights).
- Pour en savoir plus sur les modifications propres à la région Azure China, consultez le [playbook Azure China](/azure/china/).
