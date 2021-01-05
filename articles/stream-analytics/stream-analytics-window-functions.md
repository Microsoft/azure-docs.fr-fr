---
title: Présentation des fonctions de fenêtrage d’Azure Stream Analytics
description: Cet article décrit quatre fonctions de fenêtrage (bascules, récurrentes, glissantes, session) qui sont utilisées dans les travaux Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/16/2020
ms.openlocfilehash: 4765c0c43cfd9760eb5b4392604bfaeedc69c388
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511438"
---
# <a name="introduction-to-stream-analytics-windowing-functions"></a>Présentation des fonctions de fenêtrage de Stream Analytics

Dans les scénarios de streaming en temps réel, il est courant d’effectuer des opérations sur les données contenues dans des fenêtres temporelles. Stream Analytics a une prise en charge native des fonctions de fenêtrage, permettant aux développeurs de créer des travaux de traitement de flux complexes avec un minimum d’effort.

Cinq types de fenêtres temporelles sont disponibles : [**bascule**](/stream-analytics-query/tumbling-window-azure-stream-analytics), [**récurrente**](/stream-analytics-query/hopping-window-azure-stream-analytics), [**glissante**](/stream-analytics-query/sliding-window-azure-stream-analytics), [**session**](/stream-analytics-query/session-window-azure-stream-analytics) et [**d’instantané**](/stream-analytics-query/snapshot-window-azure-stream-analytics).  Vous utilisez les fonctions de fenêtre dans la clause [**GROUP BY**](/stream-analytics-query/group-by-azure-stream-analytics) de la syntaxe de requête dans vos travaux Stream Analytics. Vous pouvez également agréger des événements sur plusieurs fenêtres avec la fonction [**Windows()**](/stream-analytics-query/windows-azure-stream-analytics).

Toutes les opérations de [fenêtrage](/stream-analytics-query/windowing-azure-stream-analytics) renvoient les résultats à la **fin** de la fenêtre. Notez que lorsque vous démarrez une tâche Stream Analytics, vous pouvez spécifier l’*heure de début de la sortie de la tâche* et le système extrait automatiquement les événements précédents dans les flux entrants pour générer la première fenêtre à l’heure spécifiée : par exemple, lorsque vous démarrez avec l’option *Maintenant*, elle commence immédiatement à émettre des données. La sortie de la fenêtre sera un événement unique qui dépendra de la fonction d’agrégation utilisée. L’événement de sortie aura l’horodatage de la fin de la fenêtre et toutes les fonctions de fenêtrage sont définies avec une longueur fixe. 

![Concepts des fonctions de fenêtrage de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-conceptual.png)

## <a name="tumbling-window"></a>Fenêtre bascule
Les fonctions de fenêtre bascule sont utilisées pour segmenter un flux de données en segments temporels distincts et exécuter une fonction sur ces segments, comme dans l’exemple ci-dessous. Les principales caractéristiques des fenêtres bascules sont les suivantes : elles se répètent, elles ne se chevauchent pas et un événement ne peut pas appartenir à plusieurs fenêtres bascules.

![Fenêtre bascule Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-tumbling-intro.png)

## <a name="hopping-window"></a>Fenêtre récurrente
Les fonctions de fenêtre récurrente font des bonds d’une durée fixe dans le temps. Il peut être facile de les considérer comme des fenêtres bascule qui peuvent se chevaucher et être émises plus souvent que la taille de la fenêtre. Les événements peuvent appartenir à plusieurs jeux de résultats de fenêtres récurrentes. Pour créer une fenêtre récurrente identique à une fenêtre bascule, spécifiez une taille de bond égale à la taille de la fenêtre. 

![Fenêtre récurrente Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-hopping-intro.png)

## <a name="sliding-window"></a>Fenêtre glissante

Les fenêtres glissantes, à la différence des fenêtres bascules ou récurrentes, génèrent des événements uniquement pour les points temporels où le contenu de la fenêtre change réellement. En d’autres termes, lorsqu’un événement entre dans la fenêtre ou la quitte. Par conséquent, chaque fenêtre affiche au moins un événement. Comme pour les fenêtres récurrentes, les événements peuvent appartenir à plusieurs fenêtres glissantes.

![Fenêtre glissante Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-sliding-intro.png)

## <a name="session-window"></a>Fenêtre session
Les fonctions de fenêtre session regroupent les événements qui arrivent au même moment, en filtrant les périodes de temps où il n’existe aucune donnée. Il existe trois paramètres principaux : délai d’expiration, durée maximale et clé de partitionnement (facultative).

![Fenêtre session Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-session-intro.png)

Une fenêtre de session commence lorsque le premier événement se produit. Si un autre événement se produit dans le délai d’expiration spécifié à partir du dernier événement réceptionné, alors la fenêtre s’étend pour inclure le nouvel événement. Dans le cas contraire, si aucun événement ne se produit dans le délai imparti, la fenêtre est fermée une fois le délai expiré.

Si des événements continuent à se produire durant le délai d’expiration spécifié, la fenêtre de session continuera à s’étendre jusqu'à ce que la durée maximale soit atteinte. Les intervalles de vérification de la durée maximale sont définis pour être de la même taille que la durée maximale spécifiée. Par exemple, si la durée maximale est égale à 10, alors les vérifications pour voir si la fenêtre dépasse la durée maximale se produisent à t = 0, 10, 20, 30, etc.

Lorsqu’une clé de partition est fournie, les événements sont regroupés par la clé et la fenêtre de session est appliquée indépendamment à chaque groupe. Ce partitionnement est utile lorsque vous avez besoin de fenêtres de session différentes pour différents utilisateurs ou appareils.

## <a name="snapshot-window"></a>Fenêtre d’instantané

Les fenêtres d’instantanés regroupent les événements qui ont le même horodatage. Contrairement à d’autres types de fenêtrage, qui requièrent une fonction de fenêtre spécifique [par exemple, [SessionWindow()](/stream-analytics-query/session-window-azure-stream-analytics)], vous pouvez appliquer une fenêtre d’instantané en ajoutant System.Timestamp() à la clause GROUP BY.

![Fenêtre d’instantané Stream Analytics](media/stream-analytics-window-functions/snapshot.png)

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
