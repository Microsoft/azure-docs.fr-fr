---
title: Métriques basées sur le journal et pré-agrégées dans Azure Application Insights | Microsoft Docs
description: Pourquoi utiliser des métriques basées sur le journal et pré-agrégées dans Azure Application Insights
ms.topic: conceptual
author: vgorbenko
ms.author: vitalyg
ms.date: 09/18/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 9b93ac774dffb837d93853353e83b8da4ab4d8d4
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027157"
---
# <a name="log-based-and-pre-aggregated-metrics-in-application-insights"></a>Métriques basées sur le journal et pré-agrégées dans Application Insights

Cet article explique la différence entre les métriques Application Insights « traditionnelles » basées sur les journaux d'activité et les métriques pré-agrégées. Les deux types de métriques sont à la disposition des utilisateurs d’Application Insights et chacun présente des atouts uniques pour la surveillance de l’intégrité des applications, des diagnostics et des analyses. Les développeurs qui appliquent l’instrumentation aux applications décident quel type de métrique convient le mieux à un scénario donné, en fonction de la taille de l’application, du volume attendu de télémétrie et des besoins de l’entreprise en matière de précision des métriques et de génération d’alertes.

## <a name="log-based-metrics"></a>Métriques basées sur le journal

Auparavant, le modèle de données de télémétrie de surveillance des applications d'Application Insights était uniquement basé sur un petit nombre de types prédéfinis d'événements, comme les requêtes, les exceptions, les appels de dépendance, les consultations de pages, etc. Les développeurs peuvent utiliser le Kit de développement logiciel (SDK) pour émettre ces événements manuellement (en écrivant du code qui appelle explicitement le SDK), ou bien ils peuvent se reposer sur la collecte automatique des événements assurée par l’instrumentation automatique. Dans les deux cas, le serveur principal d’Application Insights stocke tous les événements collectés sous forme de journaux d’activité. Dans le portail Azure, les panneaux Application Insights servent d’outil d’analyse et de diagnostic et permettent de visualiser les données basées sur les événements à partir des journaux d’activité.

L’utilisation de journaux d’activité pour conserver un ensemble complet d’événements présente un grand avantage en termes d’analyse et de diagnostic. Vous pouvez par exemple obtenir le nombre exact de requêtes envoyées vers une URL donnée avec le nombre d’utilisateurs distincts ayant effectué ces appels. Ou bien vous pouvez obtenir des rapports d’appels de procédures détaillés pour le diagnostic, notamment les exceptions et les appels de dépendance de n’importe quelle session utilisateur. Ce type d’information améliore considérablement la visibilité sur l’intégrité et l’utilisation des applications, diminuant le temps nécessaire pour diagnostiquer les problèmes avec une application.

Dans le même temps, la collecte d'un ensemble complet d'événements peut s'avérer difficile (voire impossible) pour les applications qui génèrent un volume important de données de télémétrie. Dans les situations où le volume d’événements est trop élevé, Application Insights implémente plusieurs techniques de réduction du volume de données de télémétrie, comme [l’échantillonnage](./sampling.md) et le [filtrage](./api-filtering-sampling.md) qui diminuent le nombre d’événements collectés et stockés. Malheureusement, la diminution du nombre d’événements stockés fait également baisser la précision des métriques qui, en arrière-plan, doivent effectuer des agrégations de requêtes des événements stockés dans les journaux d’activité.

> [!NOTE]
> Dans Application Insights, les métriques basées sur l’agrégation des requêtes d’événements et les mesures stockées dans les journaux d’activité sont appelées des métriques basées sur le journal. Ces métriques comportent généralement de nombreuses dimensions de propriétés d’événements, ce qui les rend particulièrement intéressantes pour l’analyse. Toutefois, leur précision pâtit de l’échantillonnage et du filtrage.

## <a name="pre-aggregated-metrics"></a>Métriques pré-agrégées

Outre les métriques basées sur le journal, fin 2018, l'équipe Application Insights a publié une préversion publique des métriques stockées dans un référentiel spécialisé optimisé pour les séries chronologiques. Les nouvelles métriques ne sont plus conservées sous la forme d’événements individuels avec un grand nombre de propriétés. À la place, elles sont stockées sous la forme de séries chronologiques pré-agrégées et seulement avec les principales dimensions. Les métriques sont ainsi de meilleure qualité au moment de la requête : l’extraction des données s’effectue beaucoup plus rapidement et nécessite moins de puissance de calcul. Cela rend possible de nouveaux scénarios comme [la génération d’alertes quasiment en temps réel sur les dimensions des métriques](../platform/alerts-metric-near-real-time.md), des [tableaux de bord](./overview-dashboard.md) plus réactifs et bien plus encore.

> [!IMPORTANT]
> Les métriques basées sur le journal et pré-agrégées coexistent dans Application Insights. Pour différencier les deux, dans l'expérience utilisateur Application Insights, les métriques pré-agrégées sont maintenant appelées « Métriques standard (préversion) », tandis que les métriques traditionnelles des événements ont été renommées « Métriques basées sur le journal ».

Les kits de développement logiciel les plus récents (SDK [Application Insights 2.7](https://www.nuget.org/packages/Microsoft.ApplicationInsights/2.7.2) ou version ultérieure pour .NET) pré-agrègent les métriques pendant la collecte. Cela s'applique aux [métriques standard envoyées par défaut](../platform/metrics-supported.md#microsoftinsightscomponents) afin que la précision ne soit pas affectée par l'échantillonnage ou le filtrage. Cela s'applique également aux métriques personnalisées envoyées à l'aide de [GetMetric](./api-custom-events-metrics.md#getmetric), ce qui permet de réduire l'ingestion de données et les coûts.

Lorsque les SDK n'implémentent pas la pré-agrégation (ce qui est le cas avec les anciennes versions des SDK Application Insights ou pour l'instrumentation du navigateur), le serveur principal d'Application Insights continue à renseigner les nouvelles métriques en agrégeant les événements reçus par le point de terminaison de collecte des événements d'Application Insights. Ainsi, même si vous ne bénéficiez pas de la réduction du volume de données transmises sur le réseau, vous pouvez utiliser les métriques pré-agrégées et profiter de performances et d'une prise en charge améliorées de la génération d'alertes dimensionnelles quasiment en temps réel avec des kits de développement logiciel (SDK) qui ne pré-agrègent pas les métriques pendant la collecte.

Notez par ailleurs que le point de terminaison de collecte pré-agrège les événements avant l’échantillonnage d’ingestion, ce qui signifie que [l’échantillonnage d’ingestion](./sampling.md) n’a pas d’impact sur la précision des métriques pré-agrégées, quelle que soit la version du SDK que vous utilisez pour votre application.  

### <a name="sdk-supported-pre-aggregated-metrics-table"></a>Tableau des métriques pré-agrégées prises en charge par le Kit de développement logiciel (SDK)

| SDK de production actuels | Métriques standard (pré-agrégation du SDK) | Métriques personnalisées (sans pré-agrégation du SDK) | Métriques personnalisées (avec pré-agrégation du SDK)|
|------------------------------|-----------------------------------|----------------------------------------------|---------------------------------------|
| .NET Core et .NET Framework | Prises en charge (v2.13.1 et ultérieures)| Prises en charge via [TrackMetric](api-custom-events-metrics.md#trackmetric)| Prises en charge (v2.7.2 et ultérieure) via [GetMetric](get-metric.md) |
| Java                         | Non pris en charge       | Prises en charge via [TrackMetric](api-custom-events-metrics.md#trackmetric)| Non pris en charge                           |
| Node.js                      | Non pris en charge       | Prises en charge via [TrackMetric](api-custom-events-metrics.md#trackmetric)| Non pris en charge                           |
| Python                       | Non pris en charge       | Prise en charge                                 | Prises en charge via [OpenCensus.stats](opencensus-python.md#metrics) |  


### <a name="codeless-supported-pre-aggregated-metrics-table"></a>Tableau des métriques pré-agrégées prises en charge sans code

| SDK de production actuels | Métriques standard (pré-agrégation du SDK) | Métriques personnalisées (sans pré-agrégation du SDK) | Métriques personnalisées (avec pré-agrégation du SDK)|
|-------------------------|--------------------------|-------------------------------------------|-----------------------------------------|
| ASP.NET                 | Prises en charge<sup>1<sup>    | Non pris en charge                             | Non pris en charge                           |
| ASP.NET Core            | Prises en charge<sup>2<sup>    | Non pris en charge                             | Non pris en charge                           |
| Java                    | Non pris en charge            | Non pris en charge                             | [Pris en charge](java-in-process-agent.md#metrics) |
| Node.js                 | Non pris en charge            | Non pris en charge                             | Non pris en charge                           |

1. L’attachement sans code d’ASP.NET sur App Service émet uniquement des métriques en mode d’analyse « Full ». L’attachement sans code d’ASP.NET sur App Service, VM/VMSS et localement émet des métriques standard sans dimensions. Le Kit de développement logiciel (SDK) est requis pour toutes les dimensions.
2. L’attachement sans code d’ASP.NET Core sur App Service émet des métriques standard sans dimensions. Le Kit de développement logiciel (SDK) est requis pour toutes les dimensions.

## <a name="using-pre-aggregation-with-application-insights-custom-metrics"></a>Utilisation de la pré-agrégation avec des métriques personnalisées Application Insights

Vous pouvez utiliser la pré-agrégation avec des métriques personnalisées. Les deux principaux avantages sont : la possibilité de configurer et d’alerter sur la dimension d’une métrique personnalisée et la réduction du volume de données envoyées à partir du SDK vers le point de terminaison de collecte d’Application Insights.

Il existe plusieurs [méthodes pour envoyer des métriques personnalisées à partir du SDK d’Application Insights](./api-custom-events-metrics.md). Si votre version du SDK propose les méthodes [GetMetric et TrackValue](./api-custom-events-metrics.md#getmetric), ces méthodes sont à privilégier pour l’envoi de mesures personnalisées, car dans ce cas la pré-agrégation est effectuée au sein du SDK, réduisant non seulement le volume de données stockées dans Azure, mais également le volume de données transmises à partir du SDK vers Application Insights. Sinon, utilisez la méthode [trackMetric](./api-custom-events-metrics.md#trackmetric), qui pré-agrège les événements liés aux métriques pendant l’ingestion de données.

## <a name="custom-metrics-dimensions-and-pre-aggregation"></a>Dimensions des métriques personnalisées et pré-agrégation

Toutes les métriques que vous envoyez en utilisant des appels d’API [trackMetric](./api-custom-events-metrics.md#trackmetric) ou [GetMetric et TrackValue](./api-custom-events-metrics.md#getmetric) sont automatiquement stockées dans des journaux d’activité et des magasins de métriques. Toutefois, bien que la version basée sur le journal de votre métrique personnalisée conserve toujours toutes les dimensions, la version pré-agrégée de la métrique est stockée par défaut sans aucune dimension. Vous pouvez activer la collecte des dimensions des métriques personnalisées dans l'onglet [Utilisation et estimation des coûts](./pricing.md) en cochant « Activer la génération d'alertes sur les dimensions des métriques personnalisées » : 

![Utilisation et estimation des coûts](./media/pre-aggregated-metrics-log-metrics/001-cost.png)

## <a name="why-is-collection-of-custom-metrics-dimensions-turned-off-by-default"></a>Pourquoi la collecte des dimensions des métriques personnalisées est-elle désactivée par défaut ?

La collecte des dimensions des métriques personnalisées est désactivée par défaut, car à l’avenir, le stockage des métriques personnalisées avec des dimensions sera facturé indépendamment d’Application Insights, tandis que le stockage des métriques personnalisées non dimensionnelles restera gratuit (jusqu'à un quota donné). Pour en savoir plus sur les modifications à venir de notre modèle de tarification, consultez notre [page officielle de tarification](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="creating-charts-and-exploring-log-based-and-standard-pre-aggregated-metrics"></a>Création de graphiques et exploration des métriques pré-agrégées standard et basées sur le journal

Utilisez [Azure Monitor Metrics Explorer](../platform/metrics-getting-started.md) pour tracer des graphiques à partir des métriques pré-agrégées et basées sur le journal et créez des tableaux de bord avec des graphiques. Après avoir sélectionné la ressource Application Insights souhaitée, utilisez le sélecteur d’espace de noms pour basculer entre les métriques standard (préversion) et basées sur le journal, ou sélectionnez un espace de noms de métrique personnalisée :

![Espace de noms de la métrique](./media/pre-aggregated-metrics-log-metrics/002-metric-namespace.png)

## <a name="pricing-models-for-application-insights-metrics"></a>Modèles de tarification pour les métriques Application Insights

L'ingestion de métriques dans Application Insights, qu'elles soient basées sur le journal ou pré-agrégées, engendre des coûts proportionnels à la taille des données ingérées, comme décrit [ici](./pricing.md#pricing-model). Vos métriques personnalisées (dimensions comprises) sont toujours stockées dans le magasin de journaux Application Insights. En outre, une version pré-agrégée de vos métriques personnalisées (sans les dimensions) est transmise par défaut au magasin de métriques.

La sélection de l'option [Activer la génération d'alertes sur les dimensions des métriques personnalisées](#custom-metrics-dimensions-and-pre-aggregation) pour stocker les dimensions des métriques pré-agrégées dans le magasin de métriques peut engendrer des coûts **supplémentaires** conformément à la [Tarification des métriques personnalisées](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="next-steps"></a>Étapes suivantes

* [Génération d’alertes quasiment en temps réel](../platform/alerts-metric-near-real-time.md)
* [GetMetric et TrackValue](./api-custom-events-metrics.md#getmetric)
