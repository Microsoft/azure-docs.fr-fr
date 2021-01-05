---
title: Modifications à venir des règles d’ingestion et de mise à plat dans Azure Time Series Insights Gen2 | Microsoft Docs
description: Modifications de la règle d’ingestion
ms.service: time-series-insights
services: time-series-insights
author: lyrana
ms.author: lyhughes
manager: dpalled
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/02/2020
ms.custom: lyhughes
ms.openlocfilehash: 56a1d5aab2f665f9c5bd8f6fa322f35e55483c7b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995231"
---
# <a name="upcoming-changes-to-json-flattening-and-escaping-rules-for-new-environments"></a>Modifications à venir des règles JSON de mise à plat et d’échappement pour les nouveaux environnements

> [!IMPORTANT]
> Ces modifications seront appliquées uniquement aux *nouveaux* environnements Microsoft Azure Time Series Insights Gen2. Ces modifications ne s’appliquent pas aux environnements Gen1.

Votre environnement Azure Time Series Insights Gen2 crée de manière dynamique les colonnes de votre stockage, en suivant un ensemble particulier de conventions d’affectation de noms. Lorsqu’un événement est ingéré, Time Series Insights applique un ensemble de règles à la charge utile JSON et aux noms des propriétés. Les modifications apportées à la façon dont les données JSON sont aplaties et stockées entreront en vigueur pour les nouveaux environnements Azure Time Series Insights Gen2 en juillet 2020. Cette modification vous concerne dans les cas suivants :

* Votre charge utile JSON contient des objets imbriqués.
* Votre charge utile JSON contient des tableaux.
* Vous utilisez l’un des quatre caractères spéciaux suivants dans un nom de propriété JSON : `[` `\` `.` `'`
* Une ou plusieurs de vos propriétés ID Time Series (TS) se trouvent dans un objet imbriqué.

Si vous créez un nouvel environnement et qu’un ou plusieurs de ces cas s’appliquent à votre charge utile d’événement, vous verrez vos données aplaties et stockées différemment. Le tableau suivant récapitule les changements :

| Règle actuelle | Nouvelle règle | Exemple JSON | Nom de colonne précédent | Nouveau nom de colonne
|---|---| ---| ---|  ---|
| Le JSON imbriqué est aplati à l’aide d’un trait de soulignement comme délimiteur. |Le JSON imbriqué est aplati à l’aide d’un point comme délimiteur.  | ``{"series" : { "value" : 19.338 }}`` | `series_value_double` |`series.value_double` |
| Les caractères spéciaux ne sont pas placés dans une séquence d’échappement. | Les noms de propriété JSON qui incluent les caractères spéciaux `.` `[`  `\` et `'` sont échappés à l’aide de `['` et `']`. Entre les caractères `['` et `']`, des guillemets simples et des barres obliques inverses supplémentaires sont inclus. Un guillemet simple est écrit sous la forme `\'` et une barre oblique inverse sous la forme `\\`.  | ```"Foo's Law Value": "17.139999389648"``` | `Foo's Law Value_double` | `['Foo\'s Law Value']_double` |
| Les tableaux de primitives sont stockés sous forme de chaîne. | Les tableaux de types primitifs sont stockés sous un type dynamique.  | `"values": [154, 149, 147]` | `values_string`  | `values_dynamic` |
Les tableaux d’objets sont toujours aplatis, ce qui génère plusieurs événements. | Si les objets d’un tableau n’ont pas les propriétés ID TS ou timestamp, le tableau d’objets est stocké sous un type dynamique. | `"values": [{"foo" : 140}, {"bar" : 149}]` | `values_foo_long | values_bar_long` | `values_dynamic` |

## <a name="recommended-changes-for-new-environments"></a>Modifications recommandées pour les nouveaux environnements

### <a name="if-your-ts-id-andor-timestamp-property-is-nested-within-an-object"></a>Si votre propriété ID TS ou timestamp est imbriquée dans un objet

Tous les nouveaux déploiements doivent obéir aux nouvelles règles d’ingestion. Par exemple, si votre TS ID est `telemetry_tagId`, vous devez mettre à jour tous les modèles Azure Resource Manager ou scripts de déploiement automatisés pour configurer `telemetry.tagId` comme le TS ID d’environnement. Cette modification est également nécessaire pour les timestamps de la source d’événement dans le format JSON imbriqué.

### <a name="if-your-payload-contains-nested-json-or-special-characters-and-you-automate-authoring-time-series-model-variable-expressions"></a>Si votre charge utile contient des caractères spéciaux ou un JSON imbriqué et que vous automatisez la création des expressions de variable du[modèle de série chronologique](./concepts-model-overview.md) :

Mettez à jour votre code client en exécutant [TypesBatchPut](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput) pour qu’il respecte les nouvelles règles d’ingestion. Par exemple, une ancienne [expression de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax) `"value": {"tsx": "$event.series_value.Double"}` doit être mise à jour vers l’une des options suivantes :

* `"value": {"tsx": "$event.series.value.Double"}`
* `"value": {"tsx": "$event['series']['value'].Double"}`

## <a name="next-steps"></a>Étapes suivantes

* Consultez [Stockage et entrée Azure Time Series Insights Gen2](./concepts-ingestion-overview.md).

* Découvrez comment interroger vos données à l’aide d’[API de requête de série chronologique](./concepts-query-overview.md).

* En savoir plus sur la [nouvelle syntaxe des expressions de série chronologique](/rest/api/time-series-insights/reference-time-series-expression-syntax).