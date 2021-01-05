---
title: Unités de streaming dans Azure Stream Analytics
description: Cet article décrit le paramètre d’unités de diffusion en continu et d’autres facteurs qui affectent les performances dans Azure Stream Analytics.
author: JSeb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/28/2020
ms.openlocfilehash: 38f649fbff9ea2c1182adb613b9302768708a4c4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019869"
---
# <a name="understand-and-adjust-streaming-units"></a>Comprendre et ajuster les unités de streaming

Les unités de streaming représentent les ressources de calcul allouées pour exécuter un travail Stream Analytics. Plus le nombre d’unités de streaming est élevé, plus il y a de ressources d’UC et de mémoire allouées pour votre travail. Cette capacité vous permet de vous concentrer sur la logique de requête et résume la nécessité de gérer le matériel pour exécuter votre travail Stream Analytics en temps voulu.

Pour obtenir un traitement de streaming à faible latence, les travaux Azure Stream Analytics effectuent tout le traitement en mémoire. Quand la mémoire devient insuffisante, le travail de streaming échoue. Par conséquent, pour un travail de production, il est important de surveiller l’utilisation des ressources d’un travail de streaming et de vérifier qu’il existe suffisamment de ressources allouées afin d’assurer l’exécution des travaux 24 heures sur 24 et 7 jours sur 7.

La métrique de pourcentage d’utilisation des unités de streaming, comprise entre 0 % et 100 %, décrit la consommation de mémoire de votre charge de travail. Pour un travail de streaming avec un encombrement minimal, la métrique se situe généralement entre 10 et 20 %. Si le pourcentage d’utilisation des unités de streaming est élevé (supérieur à 80 %) ou si les événements d’entrée sont mis en backlog (même avec un faible pourcentage d’utilisation des unités de streaming, car il n’affiche pas l’utilisation du processeur), il est probable que votre charge de travail nécessite davantage de ressources de calcul, ce qui vous oblige à augmenter le nombre d’unités de streaming. Il est préférable de conserver une métrique inférieure à 80 % pour prendre en compte les pics d’activité occasionnels. Pour réagir à l’augmentation des charges de travail et augmenter les unités de streaming, définissez une alerte de 80 % sur la métrique d’utilisation de l’unité de stockage. En outre, vous pouvez utiliser des métriques de délai en filigrane et d’événements retardés pour voir si cela a un impact.

## <a name="configure-stream-analytics-streaming-units-sus"></a>Configurer des unités de streaming Stream Analytics
1. Connectez-vous au [portail Azure](https://portal.azure.com/)

2. Dans la liste des ressources, recherchez le travail Stream Analytics que vous souhaitez mettre à l’échelle, puis ouvrez-le. 

3. Dans la page du travail, sous le titre **Configurer**, sélectionnez **Mettre à l’échelle**.  Le nombre par défaut d’unités SU est 3 lors de la création d’un travail.

    ![Configuration d’un travail Stream Analytics sur le portail Azure][img.stream.analytics.preview.portal.settings.scale]
    
4. Définissez les unités SU pour le travail à l’aide du curseur. Notez que vous êtes limité à des paramètres d’unité SU spécifiques. 
5. Vous pouvez modifier le nombre d’unités SU affectées à votre travail, même s’il est en cours d’exécution. Cela n’est pas possible si votre travail utilise une [sortie non partitionnée](./stream-analytics-parallelization.md#query-using-non-partitioned-output) ou dispose d’une[ requête à plusieurs étapes avec différentes valeurs PARTITION BY](./stream-analytics-parallelization.md#multi-step-query-with-different-partition-by-values). Vous pouvez éventuellement choisir parmi un ensemble de valeurs SU lorsque le travail est en cours d’exécution. 

## <a name="monitor-job-performance"></a>Surveillance des performances du travail
À l’aide du portail Azure, vous pouvez suivre le débit d’un travail :

![Surveillance des travaux Azure Stream Analytics][img.stream.analytics.monitor.job]

Calculez le débit prévu pour la charge de travail. Si le débit est plus faible que prévu, paramétrez la partition d’entrée ainsi que la requête, puis ajoutez des unités SU à votre travail.

## <a name="how-many-sus-are-required-for-a-job"></a>Combien d’unités de streaming sont requises pour un travail ?

Le choix du nombre d’unités de streaming requises pour un travail particulier dépend de la configuration de la partition pour les entrées et de la requête définie pour le travail. La page **Mise à l’échelle** vous permet de définir le nombre adapté d’unités de streaming. Il est recommandé d’allouer plus d’unités de streaming que nécessaire. Le moteur de traitement Stream Analytics est configuré pour une latence et un débit supérieurs, en allouant une capacité de mémoire supplémentaire.

En règle générale, la meilleure pratique consiste à démarrer avec 6 unités de streaming pour les requêtes qui n’utilisent pas **PARTITION BY**. Déterminez ensuite la configuration idéale en utilisant une méthode d’essai et d’erreur où vous modifiez le nombre d’unités de streaming, une fois que vous avez transmis le volume représentatif de données et examiné la métrique % d’utilisation de SU. Le nombre maximal d’unités de streaming qui peut être utilisé par un travail Stream Analytics varie selon le nombre d’étapes de la requête définie pour le travail et le nombre de partitions pour chaque étape. Pour obtenir plus d’informations sur les limites à respecter, cliquez [ici](./stream-analytics-parallelization.md#calculate-the-maximum-streaming-units-of-a-job).

Pour plus d’informations sur le choix du nombre adapté d’unités de streaming, consultez cette page : [Mettre à l’échelle des travaux Azure Stream Analytics pour augmenter le débit](stream-analytics-scale-jobs.md)

> [!Note]
> Le choix du nombre d’unités SU requises pour un travail particulier dépend de la configuration de la partition pour les entrées et de la requête définie pour le travail. Vous pouvez sélectionner votre quota en unités SU pour un travail. Par défaut, chaque abonnement Azure dispose d’un quota pouvant atteindre 500 unités de streaming pour tous les travaux Stream Analytics d’une région spécifique. Pour augmenter ce quota d’unités SU pour vos abonnements, contactez le [Support Microsoft](https://support.microsoft.com). Valeurs valides pour les unités SU par travail : 1, 3, 6 et au-dessus par incréments de 6.

## <a name="factors-that-increase-su-utilization"></a>Facteurs qui augmentent l’utilisation du % SU 

Les éléments de requête temporelle (orientée sur le temps) sont l’ensemble des opérateurs fournis par Stream Analytics. Stream Analytics gère l’état de ces opérations en interne au nom de l’utilisateur, grâce à la gestion de la consommation de mémoire, les points de contrôle pour la résilience et la récupération de l’état au cours des mises à niveau du service. Même si Stream Analytics gère totalement les états, il existe un nombre de recommandations de meilleure pratique que les utilisateurs doivent prendre en compte.

Notez qu’un travail avec une logique de requête complexe peut avoir une utilisation élevée d’unités de streaming, même s’il ne reçoit pas continuellement des événements d’entrée. Cela peut se produire après un pic soudain des événements d’entrée et de sortie. Le travail peut continuer à maintenir l’état en mémoire si la requête est complexe.

Le pourcentage d'utilisation des unités de streaming peut soudainement et brièvement passer à 0 avant de revenir aux niveaux prévus. Ce phénomène est dû à des erreurs transitoires ou à des mises à niveau lancées par le système. Augmenter le nombre d’unités de streaming d’un travail ne diminue pas forcément le pourcentage d’utilisation des unités de streaming si votre requête n’est pas [entièrement parallèle](./stream-analytics-parallelization.md).

Lorsque vous comparez l’utilisation sur une période donnée, utilisez les [métriques de taux d’événements](stream-analytics-monitoring.md). Les métriques InputEvents et OutputEvents montrent le nombre d’événements qui ont été lus et traités. Des métriques indiquent également le nombre d’événements d’erreur, tels que les erreurs de désérialisation. Lorsque le nombre d’événements par unité de temps augmente, le pourcentage de SU augmente dans la plupart des cas.

## <a name="stateful-query-logic-in-temporal-elements"></a>Logique de requête avec état dans les éléments temporels
L’une des caractéristiques propres à un travail Azure Stream Analytics consiste à effectuer un traitement avec état, comme des agrégations fenêtrées, jointures temporelles et fonctions d’analyse temporelle. Chacun de ces opérateurs conserve des informations d’état.  La taille maximale de la fenêtre pour ces éléments de requête est de sept jours. 

Le concept de fenêtre temporelle apparaît dans plusieurs éléments de requête Stream Analytics :
1. Agrégats fenêtrés : GROUP BY fenêtres de Bascule, Récurrente et Glissante

2. Jointures temporelles : JOIN à la fonction DATEDIFF

3. Fonctions analytiques temporelles : ISFIRST, LAST et LAG avec LIMIT DURATION

Les facteurs suivants influencent la mémoire utilisée (qui fait partie de la métrique des unités de streaming) par les travaux Stream Analytics :

## <a name="windowed-aggregates"></a>Agrégations fenêtrées
La mémoire consommée (taille de l’état) pour un agrégat fenêtré n’est pas toujours directement proportionnelle à la taille de la fenêtre. Au lieu de cela, la mémoire consommée est proportionnelle à la cardinalité des données ou le nombre de groupes dans chaque fenêtre de temps.


Par exemple, dans la requête suivante, le nombre associé à `clusterid` est la cardinalité de la requête. 

   ```sql
   SELECT count(*)
   FROM input 
   GROUP BY  clusterid, tumblingwindow (minutes, 5)
   ```

Afin d’atténuer les problèmes provoqués par une cardinalité élevée dans la requête précédente, vous pouvez envoyer des événements à Event Hub avec un partitionnement par `clusterid`, et effectuer un scale-out de la requête en autorisant le système à traiter chaque partition d’entrée séparément à l’aide de **PARTITION BY**, comme indiqué dans l’exemple ci-dessous :

   ```sql
   SELECT count(*) 
   FROM input PARTITION BY PartitionId
   GROUP BY PartitionId, clusterid, tumblingwindow (minutes, 5)
   ```

Une fois que la requête est partitionnée, elle est répartie sur plusieurs nœuds. Par conséquent, le nombre de `clusterid` arrivant dans chaque nœud est réduit, ce qui réduit d’autant la cardinalité du groupe par opérateur. 

Les partitions Event Hub doivent être partitionnées par la clé de regroupement pour éviter une étape de réduction. Pour plus d'informations, consultez la [Vue d'ensemble des Concentrateurs d’événements](../event-hubs/event-hubs-about.md). 

## <a name="temporal-joins"></a>Jointures temporelles
La mémoire consommée (taille de l’état) d’une jointure temporelle est proportionnelle au nombre d’événements dans la marge de manœuvre temporelle de la jointure, qui est la vitesse d’entrée des événements multipliée par la taille de la marge de manœuvre. En d’autres termes, la mémoire consommée par les jointures est proportionnelle à l’intervalle de temps DateDiff multipliée par le taux d’événements moyen.

Le nombre d’événements sans correspondance de la jointure affecte la consommation de mémoire pour la requête. La requête suivante vise à identifier l’exposition publicitaire qui génère des clics :

   ```sql
   SELECT clicks.id
   FROM clicks 
   INNER JOIN impressions ON impressions.id = clicks.id AND DATEDIFF(hour, impressions, clicks) between 0 AND 10.
   ```

Dans cet exemple, il est possible qu’un grand nombre de publicités s’affichent et que quelques personnes cliquent dessus ; cette configuration est requise pour conserver l’ensemble des événements dans la fenêtre de temps. La consommation de mémoire est proportionnelle à la taille de la fenêtre et au taux d’événements. 

Pour corriger ce problème, envoyez des événements à Event Hub avec un partitionnement par clés de jointure (ID dans ce cas) et augmentez la taille des instances de la requête en autorisant le système à traiter chaque partition d’entrée séparément à l’aide de **PARTITION BY** comme indiqué :

   ```sql
   SELECT clicks.id
   FROM clicks PARTITION BY PartitionId
   INNER JOIN impressions PARTITION BY PartitionId 
   ON impression.PartitionId = clicks.PartitionId AND impressions.id = clicks.id AND DATEDIFF(hour, impressions, clicks) between 0 AND 10 
   ```

Une fois que la requête est partitionnée, elle est répartie sur plusieurs nœuds. Par conséquent, le nombre d’événements arrivant dans chaque nœud est réduit, ce qui réduit d’autant la taille de l’état dans la fenêtre de jointure. 

## <a name="temporal-analytic-functions"></a>Fonctions analytiques temporelles
La mémoire consommée (taille de l’état) d’une fonction analytique temporelle est proportionnelle au taux d’événements multiplié par la durée. La mémoire consommée par les fonctions analytiques n’est pas proportionnelle à la taille de la fenêtre, mais plutôt le nombre de partitions dans chaque fenêtre de temps.

La solution est similaire à celle de la jointure temporelle. Vous pouvez augmenter la taille des instances de la requête à l’aide de **PARTITION BY**. 

## <a name="out-of-order-buffer"></a>Mémoire tampon d’événements en désordre 
L’utilisateur peut configurer la taille de la mémoire tampon d’événements en désordre dans le volet de configuration Ordre des événements. La mémoire tampon est utilisée pour contenir des entrées pendant la durée d’affichage de la fenêtre et les réorganiser. La taille de la mémoire tampon est proportionnelle à la vitesse d’entrée des événements multipliée par la taille de la fenêtre d’événements en désordre. La taille de fenêtre par défaut est égale à 0. 

Pour corriger le dépassement de capacité de la mémoire tampon, augmenter la taille des instances de la requête à l’aide de **PARTITION BY**. Une fois que la requête est partitionnée, elle est répartie sur plusieurs nœuds. Par conséquent, le nombre d’événements arrivant dans chaque nœud est réduit, ce qui réduit d’autant le nombre d’événements dans chaque mémoire tampon de réorganisation. 

## <a name="input-partition-count"></a>Nombre de partitions d’entrée 
Chaque partition d’entrée d’une entrée de travail a une mémoire tampon. Plus le nombre de partitions d’entrée est élevé, plus le travail consomme de ressources. Pour chaque unité de streaming, Azure Stream Analytics peut traiter environ 1 Mo/s d’entrée. Par conséquent, vous pouvez optimiser en faisant correspondre le nombre d’unités de streaming Stream Analytics avec le nombre de partitions dans votre Concentrateur d’événements. 

En règle générale, un travail configuré avec une unité de streaming est suffisante pour un Concentrateur d’événements avec deux partitions (qui est la valeur minimale du Concentrateur d’événements). Si le Concentrateur d'événements a davantage de partitions, votre travail Stream Analytics consomme davantage de ressources, mais n’utilise pas nécessairement le débit supplémentaire fourni par le Concentrateur d’événements. 

Pour un travail avec 6 unités de streaming, vous devrez peut-être demander 4 ou 8 partitions du Concentrateur d’événements. Toutefois, évitez de trop nombreuses partitions inutiles car cela provoque l’utilisation excessive des ressources. Par exemple, un Concentrateur d’événements avec 16 partitions ou plus dans un travail Stream Analytics qui possède 1 unité de streaming. 

## <a name="reference-data"></a>Données de référence 
Les données de référence dans ASA sont chargées en mémoire pour la recherche rapide. Avec l’implémentation actuelle, chaque opération de jointure avec les données de référence conserve une copie des données de référence en mémoire, même si vous effectuez la jointure plusieurs fois avec les mêmes données de référence. Pour les requêtes avec **PARTITION BY**, comme chaque partition possède une copie des données de référence, les partitions sont entièrement découplées. Avec l’effet multiplicateur, l’utilisation de la mémoire peut être rapidement très élevée si vous effectuez la jointure avec les données de référence plusieurs fois avec plusieurs partitions.  

### <a name="use-of-udf-functions"></a>Utilisation de fonctions UDF
Quand vous ajoutez une fonction UDF, Azure Stream Analytics charge le runtime JavaScript en mémoire. Cela affecte le pourcentage d’utilisation d’unités de streaming.

## <a name="next-steps"></a>Étapes suivantes
* [Créer des requêtes parallélisables dans Azure Stream Analytics](stream-analytics-parallelization.md)
* [Mettre à l’échelle des travaux Azure Stream Analytics pour augmenter le débit](stream-analytics-scale-jobs.md)

<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor-NewPortal.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings-NewPortal.png
