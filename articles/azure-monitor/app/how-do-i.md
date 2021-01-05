---
title: Comment... dans Azure Application Insights | Microsoft Docs
description: FAQ dans Application Insights
ms.topic: conceptual
ms.date: 04/04/2017
ms.openlocfilehash: 134089f4df8f80147182835ca8746322c1de7e50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87319250"
---
# <a name="how-do-i--in-application-insights"></a>Comment ... dans Application Insights ?
## <a name="get-an-email-when-"></a>Recevoir un message électronique quand...
### <a name="email-if-my-site-goes-down"></a>Envoyer un message électronique si mon site tombe en panne
Définissez un [test web de disponibilité](./monitor-web-app-availability.md).

### <a name="email-if-my-site-is-overloaded"></a>Envoyer un message électronique si mon site est surchargé
Définissez une [alerte](../platform/alerts-log.md) sur le **Temps de réponse du serveur**. Un seuil compris entre 1 et 2 secondes doit fonctionner.

![Capture d’écran montrant comment définir une alerte sur le temps de réponse du serveur.](./media/how-do-i/030-server.png)

Votre application peut également indiquer des signes de surcharge en retournant des codes d’erreur. Définissez une alerte sur les **Demandes ayant échoué**.

Si vous voulez définir une alerte sur les **Exceptions du serveur**, vous devrez peut-être effectuer une [installation supplémentaire](./asp-net-exceptions.md) pour afficher les données.

### <a name="email-on-exceptions"></a>Envoyer un message électronique en cas d’exceptions
1. [Configurer la surveillance des exceptions](./asp-net-exceptions.md)
2. [Définir une alerte](../platform/alerts-log.md) sur la métrique Nombre d’exceptions

### <a name="email-on-an-event-in-my-app"></a>Envoyer un message électronique en réponse à un événement dans mon application
Supposons que vous vouliez recevoir un e-mail quand un événement spécifique se produit. Application Insights ne fournit pas directement cette fonctionnalité, mais peut [envoyer une alerte quand une métrique dépasse un seuil](../platform/alerts-log.md).

Les alertes peuvent être définies sur des [métriques personnalisées](./api-custom-events-metrics.md#trackmetric), et non sur des événements personnalisés. Écrivez du code pour augmenter une métrique quand l’événement se produit :

```csharp
telemetry.TrackMetric("Alarm", 10);
```

ou :

```csharp
var measurements = new Dictionary<string,double>();
measurements ["Alarm"] = 10;
telemetry.TrackEvent("status", null, measurements);
```

Les alertes ayant deux états, vous devez envoyer une valeur faible quand vous considérez que l’alerte est terminée :

```csharp
telemetry.TrackMetric("Alarm", 0.5);
```

Créez un graphique dans [Metrics Explorer](../platform/metrics-charts.md) pour afficher votre alarme :

![Capture d’écran montrant comment créer un graphe dans Metrics Explorer pour afficher l’alarme.](./media/how-do-i/010-alarm.png)

Maintenant, définissez une alerte qui se déclenche quand la métrique dépasse une valeur moyenne pendant une courte période :

![Capture d’écran montrant comment définir une alerte qui se déclenche quand la métrique dépasse la valeur moyenne sur une courte période.](./media/how-do-i/020-threshold.png)

Définissez la durée moyenne sur la valeur minimale.

Vous recevrez des messages électroniques quand la métrique est supérieure et inférieure au seuil.

Éléments à prendre en considération :

* Une alerte possède deux états (« alerte » et « intègre »). L’état est évalué uniquement quand une métrique est reçue.
* Un message électronique est envoyé uniquement quand l’état change. C’est pourquoi vous devez envoyer à la fois des métriques de valeur haute et faible.
* Pour évaluer l’alerte, la moyenne est calculée à partir des valeurs reçues pendant la période précédente. Comme cela se produit chaque fois qu’une métrique est reçue, des messages électroniques peuvent être envoyés plus fréquemment que la période que vous avez définie.
* Étant donné que les messages électroniques sont envoyés à la fois pour les états « alerte » et « intègre », vous devrez peut-être remplacer votre événement à déclenchement unique par une condition à deux états. Par exemple, au lieu d’un événement « tâche terminée », définissez une condition « tâche en cours », pour laquelle vous obtenez des messages électroniques au début et à la fin d’une tâche.

### <a name="set-up-alerts-automatically"></a>Configurer automatiquement des alertes
[Utiliser PowerShell pour créer des alertes](../platform/alerts-log.md)

## <a name="use-powershell-to-manage-application-insights"></a>Utiliser PowerShell pour gérer Application Insights
* [Créer des ressources](./create-new-resource.md#creating-a-resource-automatically)
* [Créer des alertes](../platform/alerts-log.md)

## <a name="separate-telemetry-from-different-versions"></a>Télémétrie distincte des différentes versions

* Plusieurs rôles dans une application : utilisez une seule ressource Application Insights et filtrez sur [cloud_Rolename](./app-map.md).
* Séparation des versions de développement, de test et de mise en production : utilisez différentes ressources Application Insights. Sélectionnez les clés d’instrumentation à partir de web.config. [En savoir plus](./separate-resources.md)
* Génération de rapports sur les versions : ajoutez une propriété à l’aide d’un initialiseur de télémétrie. [En savoir plus](./separate-resources.md)

## <a name="monitor-backend-servers-and-desktop-apps"></a>Surveiller les serveurs principaux et les applications de bureau
[Utilisez le module du Kit de développement logiciel (SDK) Windows Server](./windows-desktop.md).

## <a name="visualize-data"></a>Visualiser les données
#### <a name="dashboard-with-metrics-from-multiple-apps"></a>Tableau de bord avec des métriques de plusieurs applications
* Dans [Metrics Explorer](../platform/metrics-charts.md), personnalisez votre graphique et enregistrez-le comme favori. Épinglez-le au tableau de bord Azure.

#### <a name="dashboard-with-data-from-other-sources-and-application-insights"></a>Tableau de bord avec des données provenant d’autres sources et d’Application Insights
* [Exportez la télémétrie dans Power BI](./export-power-bi.md).

ou

* Utilisez SharePoint comme tableau de bord, pour afficher les données des composants WebPart de SharePoint. [Utilisez l’exportation continue et Stream Analytics pour exporter vers SQL](./code-sample-export-sql-stream-analytics.md).  Utilisez PowerView pour examiner la base de données et créer un composant WebPart de SharePoint pour PowerView.

<a name="search-specific-users"></a>

### <a name="filter-out-anonymous-or-authenticated-users"></a>Filtrer les utilisateurs authentifiés ou anonymes
Si vos utilisateurs se connectent, vous pouvez définir l’[ID d’utilisateur authentifié](./api-custom-events-metrics.md#authenticated-users). (Cela n’est pas automatique.)

Ensuite, vous pouvez :

* Rechercher des ID d’utilisateur spécifiques

![Capture d’écran montrant les options de recherche d’identifiants utilisateurs précis.](./media/how-do-i/110-search.png)

* Filtrer des métriques sur les utilisateurs anonymes ou authentifiés

![Capture d’écran montrant le filtrage des métriques sur des utilisateurs anonymes ou bien authentifiés.](./media/how-do-i/115-metrics.png)

## <a name="modify-property-names-or-values"></a>Modification de noms ou de valeurs de propriété
Créez un [filtre](./api-filtering-sampling.md#filtering). Cela vous permet de modifier ou de filtrer la télémétrie avant son envoi depuis votre application vers Application Insights.

## <a name="list-specific-users-and-their-usage"></a>Répertorier les utilisateurs spécifiques et leur utilisation
Si vous voulez simplement [rechercher des utilisateurs spécifiques](#search-specific-users), vous pouvez définir [l’ID utilisateur authentifié](./api-custom-events-metrics.md#authenticated-users).

Si vous voulez une liste des utilisateurs avec des données telles que les pages qu’ils ont consultées ou leur fréquence de connexion, deux options s’offrent à vous :

* [Définissez l’ID utilisateur authentifié](./api-custom-events-metrics.md#authenticated-users), [exportez vers une base de données](./code-sample-export-sql-stream-analytics.md) et utilisez des outils pour analyser vos données utilisateur.
* Si vous ne disposez que d’un petit nombre d’utilisateurs, envoyez des événements ou des métriques personnalisés, en utilisant les données d’intérêt comme le nom de l’événement ou la valeur de la métrique et en définissant l’ID d’utilisateur en tant que propriété. Pour analyser les vues de page, remplacez l’appel standard de trackPageView JavaScript. Pour analyser la télémétrie côté serveur, utilisez un initialiseur de télémétrie pour ajouter l’ID d’utilisateur à toute la télémétrie de serveur. Vous pouvez ensuite filtrer et segmenter les métriques et les recherches sur l’ID d’utilisateur.

## <a name="reduce-traffic-from-my-app-to-application-insights"></a>Réduire le trafic de mon application vers Application Insights
* Dans [ApplicationInsights.config](./configuration-with-applicationinsights-config.md), désactivez tous les modules dont vous n’avez pas besoin, comme le collecteur de compteurs de performances.
* Utilisez [Échantillonnage et filtrage](./api-filtering-sampling.md) dans le Kit de développement logiciel (SDK).
* Dans vos pages web, limitez le nombre d'appels Ajax signalés pour chaque affichage de page. Dans l(extrait de script après `instrumentationKey:...`, insérez : `,maxAjaxCallsPerView:3` (ou un nombre approprié).
* Si vous utilisez [TrackMetric](./api-custom-events-metrics.md#trackmetric), calculez l’agrégat des lots de valeurs de métriques avant d’envoyer le résultat. Il existe une surcharge de TrackMetric() qui se charge de cette tâche.

En savoir plus sur la [tarification et les quotas](./pricing.md).

## <a name="disable-telemetry"></a>Désactiver la télémétrie
Pour **arrêter et démarrer dynamiquement** la collecte et la transmission de la télémétrie à partir du serveur :

### <a name="aspnet-classic-applications"></a>Applications ASP.NET classiques

```csharp
using  Microsoft.ApplicationInsights.Extensibility;

TelemetryConfiguration.Active.DisableTelemetry = true;
```

### <a name="other-applications"></a>Autres applications
Il n’est pas recommandé d’utiliser singleton `TelemetryConfiguration.Active` sur la console ou les applications ASP.NET Core.
Si vous avez créé une instance `TelemetryConfiguration` vous-même - définissez `DisableTelemetry` à `true`.

Pour les applications ASP.NET Core, vous pouvez accéder à l’instance `TelemetryConfiguration` à l’aide de [l’injection de dépendances ASP.NET Core](/aspnet/core/fundamentals/dependency-injection/). Vous trouverez plus de détails dans l’article[ApplicationInsights pour les applications ASP.NET Core](./asp-net-core.md).

## <a name="disable-selected-standard-collectors"></a>Désactivez les collecteurs standards sélectionnés
Vous pouvez désactiver les collecteurs standard (par exemple, les compteurs de performances, les requêtes HTTP ou les dépendances)

* **Applications ASP.NET** : supprimez ou commentez les lignes correspondantes dans [ApplicationInsights.config](./configuration-with-applicationinsights-config.md)
* **Applications ASP.NET Core** : suivez les options de configuration de modules de télémétrie dans [ApplicationInsights ASP.NET Core](./asp-net-core.md#configuring-or-removing-default-telemetrymodules)

## <a name="view-system-performance-counters"></a>Afficher les compteurs de performances système
Parmi les métriques que vous pouvez afficher dans Metrics Explorer, il existe un ensemble de compteurs de performances système. Un panneau prédéfini intitulé **Serveurs** affiche plusieurs d'entre eux.

![Ouvrez votre ressource Application Insights et cliquez sur Serveurs.](./media/how-do-i/121-servers.png)

### <a name="if-you-see-no-performance-counter-data"></a>Si aucune donnée de compteur de performances ne s’affiche
* **Serveur IIS** sur votre propre ordinateur ou sur une machine virtuelle. [Installer Status Monitor](./monitor-performance-live-website-now.md).
* **Site Web Azure** : nous ne prenons pas encore en charge les compteurs de performances. Il existe plusieurs métriques que vous pouvez obtenir de manière standard dans le panneau de configuration des sites web Azure.
* **Serveur Unix** - [installer collectd](./java-collectd.md)

### <a name="to-display-more-performance-counters"></a>Pour afficher davantage de compteurs de performances
* Tout d’abord, [ajoutez un nouveau graphique](../platform/metrics-charts.md) et vérifiez si le compteur se trouve dans le jeu de base que nous offrons.
* Dans le cas contraire, [ajoutez le compteur au jeu collecté par le module de compteur de performances](./performance-counters.md).

