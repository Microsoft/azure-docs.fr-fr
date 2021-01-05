---
title: Partager des vues personnalisées avec des URL paramétrables – Azure Time Series Insights | Microsoft Docs
description: Découvrez comment créer des URL paramétrées pour partager facilement des vues Explorer personnalisées dans Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.topic: conceptual
ms.workload: big-data
ms.date: 10/02/2020
ms.custom: seodec18
ms.openlocfilehash: 9bf857a66643b1e95ea2559601761a7217babad4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91665325"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>Partager une vue personnalisée à l’aide d’une URL paramétrable

Pour partager une vue personnalisée dans l’Explorateur Azure Time Series Insights, vous pouvez créer, par programmation, une URL paramétrable de la vue personnalisée.

Cet Explorateur prend en charge les paramètres de requête d’URL pour spécifier des vues dans l’expérience et ce, directement à partir de l’URL. Par exemple, en utilisant l’URL uniquement, vous pouvez spécifier un environnement cible, un prédicat de recherche et la durée désirée. Lorsqu’un utilisateur sélectionne l’URL personnalisée, l’interface fournit directement un lien vers la ressource dans le portail Azure Time Series Insights. Les stratégies d’accès aux données s’appliquent.

> [!TIP]
>
> * Affichez la [démonstration Azure Time Series Insights](https://insights.timeseries.azure.com/samples) gratuite.
> * Lisez la documentation [Explorateur Azure Time Series Insights](./time-series-insights-explorer.md) connexe.

## <a name="environment-id"></a>ID de l’environnement

Le paramètre `environmentId=<guid>` spécifie l’ID de l’environnement cible. Il s’agit d’un composant du nom de domaine complet pour l’accès aux données. Vous le trouverez dans l’angle supérieur droit de l’écran présentation de l’environnement, sur le portail Azure. Il correspond à tout ce qui précède `env.timeseries.azure.com`.

Exemple de paramètre d’ID de l’environnement : `?environmentId=10000000-0000-0000-0000-100000000108`.

## <a name="time"></a>Temps

Vous pouvez spécifier des valeurs de temps absolues ou relatives avec une URL paramétrable.

### <a name="absolute-time-values"></a>Valeurs de temps absolues

Pour les valeurs de temps absolues, utilisez les paramètres `from=<integer>` et `to=<integer>`.

* `from=<integer>` est une valeur JavaScript, en millisecondes, qui définit l’heure de début de la période de recherche.
* `to=<integer>` est une valeur JavaScript, en millisecondes, qui définit l’heure de fin de la période de recherche.

> [!TIP]
> Pour traduire facilement les dates en quelques millisecondes JavaScript, essayez le [convertisseur d’horodatage Epoch & Unix](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html).

### <a name="relative-time-values"></a>Valeurs de temps relatives

Dans le cas d’une valeur de temps relative, utilisez l’élément `relativeMillis=<value>`, où la *valeur* est exprimée en millisecondes JavaScript à partir de l’horodateur le plus récent reçu de l’API.

Par exemple, l’élément `&relativeMillis=3600000` affiche les dernières 60 minutes de données.

Les valeurs acceptées correspondent au menu **Quick Time** de l’Explorateur Azure Time Series Insights, notamment :

* `1800000` (30 dernières minutes)
* `3600000` (60 dernières minutes)
* `10800000` (3 dernières heures)
* `21600000` (6 dernières heures)
* `43200000` (12 dernières heures)
* `86400000` (24 dernières heures)
* `604800000` (7 derniers jours)
* `2592000000` (30 dernières heures)

### <a name="optional-parameters"></a>Paramètres facultatifs

Le paramètre `timeSeriesDefinitions=<collection of term objects>` spécifie les termes de prédicat qui s’affichent dans une vue Azure Time Series Insights :

| Paramètre | Élément d’URL | Description |
| --- | --- | --- |
| **name** | `\<string>` | nom du *terme* ; |
| **splitBy** | `\<string>` | nom de colonne pour le *fractionnement* ; |
| **measureName** | `\<string>` | nom de colonne de *mesure* ; |
| **predicate** | `\<string>` | clause *where* pour le filtrage côté serveur. |
| **useSum** | `true` | Paramètre facultatif qui spécifie l’utilisation de la somme pour la mesure. |

> [!NOTE]
> Si `Events` est la mesure **useSum** sélectionnée, le nombre est sélectionné par défaut.  
> Si `Events` n’est pas sélectionné, la moyenne est sélectionnée par défaut. |

* La paire clé-valeur `multiChartStack=<true/false>` permet l’empilement dans le graphique.
* La paire clé-valeur `multiChartSameScale=<true/false>` permet la même échelle de l’axe des ordonnées entre les termes au sein d’un paramètre facultatif.  
* `timeBucketUnit=<Unit>&timeBucketSize=<integer>` permet d’ajuster le curseur intervalle pour générer une vue plus granulaire, plus simple, ou plus globale du graphique.  
* Le paramètre `timezoneOffset=<integer>` vous permet de définir le fuseau horaire pour le graphique afin qu’il s’affiche en tant qu’offset de fuseau horaire UTC.

| Paire(s) | Description |
| --- | --- |
| `multiChartStack=false` | `true` est activée par défaut donc passez sur `false` pour empiler. |
| `multiChartStack=false&multiChartSameScale=true` | L’empilement doit être activé pour pouvoir utiliser la même échelle de l’axe des ordonnées entre les termes.  `false` est activé par défaut, donc passer sur `true` pour activer cette fonctionnalité. |
| `timeBucketUnit=<Unit>&timeBucketSize=<integer>` | Unités = `days`, `hours`, `minutes`, `seconds`, `milliseconds`.  Mettez toujours les unités en majuscules. </br> Définissez le nombre d’unités en passant l’entier souhaité pour **timeBucketSize**.  |
| `timezoneOffset=-<integer>` | L’entier est toujours en millisecondes. |

> [!NOTE]
> Les valeurs **timeBucketUnit** peuvent être lissées jusqu’à 7 jours.
> Les valeurs **timezoneOffset** ne sont ni UTC ni heure locale.

### <a name="examples"></a>Exemples

Pour ajouter des définitions de série chronologique dans un environnement Azure Time Series Insights en tant que paramètre URL, ajoutez :

```URL parameter
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

Utilisez ces exemples de définitions de série chronologique pour :

* l’ID de l’environnement,
* les 60 dernières minutes de données,
* les termes (**F1PressureID**, **F2TempStation** et **F3VibrationPL**) qui composent des paramètres facultatifs.

Vous pouvez construire l’URL paramétrable suivante pour une vue :

```URL
https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

[![Azure Time Series Insights Explorer parameterized URL](media/parameterized-url/share-parameterized-url.png)](media/parameterized-url/share-parameterized-url.png#lightbox)

> [!TIP]
> Voir l’Explorateur live [à l’aide de l’URL](https://insights.timeseries.azure.com/classic/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[%7B%22name%22:%22F1PressureId%22,%22splitBy%22:%22Id%22,%22measureName%22:%22Pressure%22,%22predicate%22:%22%27Factory1%27%22%7D,%7B%22name%22:%22F2TempStation%22,%22splitBy%22:%22Station%22,%22measureName%22:%22Temperature%22,%22predicate%22:%22%27Factory2%27%22%7D,%7B%22name%22:%22F3VibrationPL%22,%22splitBy%22:%22ProductionLine%22,%22measureName%22:%22Vibration%22,%22predicate%22:%22%27Factory3%27%22%7D]
) ci-dessus.

L’URL ci-dessus décrit et affiche la vue paramétrée de l’Explorateur Azure Time Series Insights.

* Prédicats paramétrables.

  [![Azure Time Series Insights Explorer parameterized predicates.](media/parameterized-url/share-parameterized-url-predicates.png)](media/parameterized-url/share-parameterized-url-predicates.png#lightbox)

* Vue de graphique complet partagé.

  [![Vue de graphique complet partagé.](media/parameterized-url/share-parameterized-url-full-chart.png)](media/parameterized-url/share-parameterized-url-full-chart.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [interroger des données avec C#](time-series-insights-query-data-csharp.md).

* En savoir plus sur l’[Explorateur Azure Time Series Insights](./time-series-insights-explorer.md).
