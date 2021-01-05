---
title: Analyse de l’usage avec Azure Application Insights | Microsoft Docs
description: Comprenez vos utilisateurs et ce qu’ils font avec votre application.
ms.topic: conceptual
ms.date: 03/25/2019
ms.openlocfilehash: 4f4954451bfa195b07c580ffa451b8cb333eb32c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532101"
---
# <a name="usage-analysis-with-application-insights"></a>Analyse de l'utilisation avec Application Insights

Quelles sont les fonctionnalités de votre application web ou mobile les plus populaires ? Vos utilisateurs atteignent-ils leurs objectifs avec votre application ? Disparaissent-ils à des stades spécifiques, et reviennent-ils plus tard ?  [Azure Application Insights](./app-insights-overview.md) vous permet d’obtenir des insights utiles sur l’utilisation de votre application. Chaque fois que vous mettez à jour votre application, vous pouvez évaluer son bon fonctionnement pour les utilisateurs. Grâce à ces informations, vous pouvez prendre des décisions basées sur des données sur les cycles de développement suivants.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Cijb]

## <a name="send-telemetry-from-your-app"></a>Envoyer des données de télémétrie à partir de votre application

La meilleure expérience est obtenue en installant Application Insights à la fois dans votre code serveur d’applications et dans vos pages web. Les composants client et serveur de votre application envoient la télémétrie au portail Azure pour analyse.

1. **Code serveur :** installez le module approprié pour votre [ASP.NET](./asp-net.md), [Azure](./app-insights-overview.md), [Java](./java-get-started.md), [Node.js](./nodejs.md) ou une [autre](./platforms.md) application.

    * *Vous ne voulez pas installer de code serveur ? Vous pouvez simplement [créer une ressource Azure Application Insights](./create-new-resource.md).*

2. **Code de page web :** ajoutez le script suivant à votre page web avant la balise ``</head>`` de fermeture. Remplacez la clé d’instrumentation par la valeur appropriée pour votre ressource Application Insights :
    
    ```html
    <script type="text/javascript">
    var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(e){function n(e){t[e]=function(){var n=arguments;t.queue.push(function(){t[e].apply(t,n)})}}var t={config:e};t.initialize=!0;var i=document,a=window;setTimeout(function(){var n=i.createElement("script");n.src=e.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",i.getElementsByTagName("script")[0].parentNode.appendChild(n)});try{t.cookie=i.cookie}catch(e){}t.queue=[],t.version=2;for(var r=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];r.length;)n("track"+r.pop());n("startTrackPage"),n("stopTrackPage");var s="Track"+r[0];if(n("start"+s),n("stop"+s),n("setAuthenticatedUserContext"),n("clearAuthenticatedUserContext"),n("flush"),!(!0===e.disableExceptionTracking||e.extensionConfig&&e.extensionConfig.ApplicationInsightsAnalytics&&!0===e.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){n("_"+(r="onerror"));var o=a[r];a[r]=function(e,n,i,a,s){var c=o&&o(e,n,i,a,s);return!0!==c&&t["_"+r]({message:e,url:n,lineNumber:i,columnNumber:a,error:s}),c},e.autoExceptionInstrumented=!0}return t}(
    {
      instrumentationKey:"INSTRUMENTATION_KEY"
    }
    );window[aiName]=aisdk,aisdk.queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
    </script>
    ```

    Pour en savoir plus les configurations plus complexes pour la supervision des sites web, consultez l’[article de référence du SDK JavaScript](./javascript.md).

3. **Code de l’application mobile :** utilisez le SDK App Center pour collecter les événements à partir de votre application, puis envoyer des copies de ces événements à Application Insights pour analyse en [suivant ce guide](../learn/mobile-center-quickstart.md).

4. **Obtenir la télémétrie :** exécutez votre projet en mode débogage pendant quelques minutes, puis examinez les résultats dans le panneau Vue d’ensemble dans Application Insights.

    Publiez votre application pour surveiller les performances de votre application et découvrir ce que vos utilisateurs font avec votre application.

## <a name="include-user-and-session-id-in-your-telemetry"></a>Inclure l’ID d’utilisateur et l’ID de session dans votre télémétrie
Pour effectuer le suivi des utilisateurs au fil du temps, Application Insights nécessite un moyen de les identifier. L’outil Événements est le seul outil d’utilisation qui ne nécessite pas d’ID d’utilisateur ni d’ID de session.

Démarrer l’envoi des ID d’utilisateur et de session à l’aide de [ce processus](./usage-send-user-context.md).

## <a name="explore-usage-demographics-and-statistics"></a>Explorer des données démographiques et des statistiques de l’utilisation
Découvrez quand des personnes utilisent votre application, les pages qui les intéressent le plus, où vos utilisateurs se trouvent, les navigateurs et les systèmes d’exploitation qu’ils utilisent. 

Les rapports Utilisateurs et sessions filtrent vos données par pages ou événements personnalisés, et les segmentent par propriétés telles que l’emplacement, l’environnement et la page. Vous pouvez également ajouter vos propres filtres.

![Capture d’écran montrant la page Vue d’ensemble des utilisateurs pour une société fictive.](./media/usage-overview/users.png)  

Aperçu des modèles intéressants appropriés dans le jeu de données.  

* Le rapport **Utilisateurs** compte le nombre d’utilisateurs uniques qui accèdent à vos pages dans les délais que vous avez choisis. Pour les applications web, les utilisateurs sont comptés avec les cookies. Si un utilisateur accède à votre site avec différents navigateurs ou ordinateurs clients, ou efface ses cookies, il est compté plusieurs fois.
* Le rapport **Sessions** compte le nombre de sessions utilisateur qui accèdent à votre site. Une session est une période d’activité d’un utilisateur, qui se termine par une période d’inactivité de plus d’une demi-heure.

[En savoir plus sur les outils Utilisateurs, Sessions et Événements](usage-segmentation.md)  

## <a name="retention---how-many-users-come-back"></a>Rétention - Combien d’utilisateurs reviennent ?

La rétention vous permet de comprendre la fréquence à laquelle vos utilisateurs reviennent utiliser leur application, en fonction des cohortes d’utilisateurs qui ont effectué une action pendant une période donnée. 

- Comprendre quelles fonctionnalités spécifiques font revenir les utilisateurs plus que d’autres 
- Formuler des hypothèses en fonction des données utilisateur réel 
- Déterminer si la rétention est un problème dans votre produit 

![Capture d’écran montrant la page Vue d’ensemble de la rétention avec des informations sur la fréquence à laquelle les utilisateurs reviennent pour utiliser leur application.](./media/usage-overview/retention.png) 

Les commandes de rétention en haut vous permettent de définir des événements spécifiques et un intervalle de temps pour calculer la rétention. Le graphique au centre fournit une représentation visuelle du pourcentage de rétention globale sur l’intervalle de temps spécifié. Le graphique en bas représente la rétention sur une période de temps donnée. Ce niveau de détail vous permet de comprendre de manière plus approfondie ce que font vos utilisateurs et ce qui les peut amener à revenir.  

[En savoir plus sur l’outil de rétention](usage-retention.md)

## <a name="custom-business-events"></a>Événements personnalisés

Pour bien comprendre ce que font les utilisateurs avec votre application, il est utile d’insérer des lignes de code pour enregistrer des événements personnalisés. Ces événements permettent de suivre toute activité, des actions détaillées des utilisateurs comme un clic sur un bouton, aux événements plus significatifs comme un de faire un achat ou de gagner à un jeu. 

Bien que les pages consultées puissent représenter des événements utiles, cela n’est en général pas le cas. Un utilisateur peut ouvrir une page produit sans acheter le produit. 

Grâce aux événements spécifiques, vous pouvez représenter la progression de vos utilisateurs sur votre site. Vous pouvez connaître leurs préférences sur les différentes options et où ils abandonnent ou rencontrent des difficultés. Grâce à ces informations, vous pouvez prendre des décisions avisées sur les priorités de vos travaux de développement en souffrance.

Les événements peuvent être enregistrés à partir du côté client de l’application :

```JavaScript

    appInsights.trackEvent("ExpandDetailTab", {DetailTab: tabName});
```

Ou du côté serveur :

```csharp
    var tc = new Microsoft.ApplicationInsights.TelemetryClient();
    tc.TrackEvent("CreatedAccount", new Dictionary<string,string> {"AccountType":account.Type}, null);
    ...
    tc.TrackEvent("AddedItemToCart", new Dictionary<string,string> {"Item":item.Name}, null);
    ...
    tc.TrackEvent("CompletedPurchase");
```

Vous pouvez joindre des valeurs de propriété à ces événements, afin de pouvoir filtrer ou fractionner les événements lorsque vous les étudiez dans le portail. Un ensemble standard de propriétés est également associé à chaque événement, comme des ID d’utilisateur anonymes, vous permettant ainsi de suivre la séquence d’activités d’un utilisateur.

En savoir plus sur les [événements personnalisés](./api-custom-events-metrics.md#trackevent) et les [propriétés](./api-custom-events-metrics.md#properties).

### <a name="slice-and-dice-events"></a>Segmenter et traiter les événements

Dans les outils Utilisateurs, Sessions et Événements, vous pouvez segmenter et traiter des événements personnalisés par utilisateur, nom d’événement et propriétés.
![Capture d’écran montrant la page Vue d’ensemble des utilisateurs pour une société fictive.](./media/usage-overview/users.png)  
  
## <a name="design-the-telemetry-with-the-app"></a>Concevoir la télémétrie avec l’application

Lorsque vous concevez chaque fonctionnalité de votre application, prenez en compte comment vous allez mesurer son succès auprès de vos utilisateurs. Choisissez les événements à enregistrer et coder les appels de suivi de ces événements dans votre application dès le début.

## <a name="a--b-testing"></a>Test A | B
Si vous ne savez pas quelle variante de la fonctionnalité sera la plus efficace, publiez les deux versions et mettez-les à disposition de différents utilisateurs. Mesurer le taux de réussite de chaque version et créez-en une version unifiée.

Pour cette technique, vous joignez des valeurs de propriétés distinctes à toute la télémétrie envoyée par chaque version de votre application. Pour cela, vous définissez des propriétés dans le contexte TelemetryContext actif. Ces propriétés par défaut sont ajoutées à chaque message de télémétrie que l'application envoie, non seulement vos messages personnalisés, mais aussi la télémétrie standard.

Dans le portail Application Insights, filtrez et segmentez vos données sur les valeurs de propriétés, afin de comparer les différentes versions.

Pour ce faire, [configurez un initialiseur de télémétrie](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) :

**Applications ASP.NET**

```csharp
    // Telemetry initializer class
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry item)
            {
                var itemProperties = item as ISupportProperties;
                if (itemProperties != null && !itemProperties.Properties.ContainsKey("AppVersion"))
                {
                    itemProperties.Properties["AppVersion"] = "v2.1";
                }
            }
    }
```

Dans l’initialiseur de l’application web, par exemple Global.asax.cs :

```csharp

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers
         .Add(new MyTelemetryInitializer());
    }
```

**Applications ASP.NET Core**

> [!NOTE]
> Ajouter l’initialiseur en utilisant `ApplicationInsights.config` ou `TelemetryConfiguration.Active` n’est pas valide pour les applications ASP.NET Core. 

Pour les applications [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers), l’ajout d’un nouveau `TelemetryInitializer` se fait en l’ajoutant au conteneur d’injection de dépendance, comme indiqué ci-dessous. Ceci est fait dans la méthode `ConfigureServices` de votre classe `Startup.cs`.

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```

Tous les nouveaux TelemetryClients ajoutent automatiquement la valeur de propriété que vous spécifiez. Les événements de télémétrie individuels peuvent remplacer les valeurs par défaut.

## <a name="next-steps"></a>Étapes suivantes
   - [Utilisateurs, sessions, événements](usage-segmentation.md)
   - [Entonnoirs](usage-funnels.md)
   - [Rétention](usage-retention.md)
   - [Flux d’utilisateurs](usage-flows.md)
   - [Classeurs](../platform/workbooks-overview.md)
   - [Ajouter du contexte utilisateur](usage-send-user-context.md)

