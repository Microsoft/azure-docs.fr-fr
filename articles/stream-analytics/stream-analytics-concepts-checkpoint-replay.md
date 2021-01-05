---
title: Concepts de point de contrôle et de récupération par réexécution dans Azure Stream Analytics
description: Cet article décrit les concepts de point de contrôle et de relecture pour la récupération de travail dans Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/06/2018
ms.custom: seodec18
ms.openlocfilehash: 10d9053e082a995085fa255cc0d9f63a2b4e2b17
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84020606"
---
# <a name="checkpoint-and-replay-concepts-in-azure-stream-analytics-jobs"></a>Concepts de point de contrôle et de relecture dans les travaux Azure Stream Analytics
Cet article décrit les concepts de point de contrôle et de relecture internes dans Azure Stream Analytics ainsi que leur impact sur la récupération de travail. Chaque fois qu’un travail Stream Analytics s’exécute, les informations d’état sont gérées en interne. Ces informations d’état sont régulièrement enregistrées dans un point de contrôle. Dans certains scénarios, les informations de point de contrôle sont utilisées pour la récupération du travail en cas d’échec ou de mise à niveau. Dans d’autres circonstances, le point de contrôle ne peut pas être utilisé pour la récupération et une relecture est nécessaire.

## <a name="stateful-query-logicin-temporal-elements"></a>Logique de requête avec état dans les éléments temporels
L’une des caractéristiques propres à un travail Azure Stream Analytics consiste à effectuer un traitement avec état, comme des agrégations fenêtrées, jointures temporelles et fonctions d’analyse temporelle. Chacun de ces opérateurs conserve des informations d’état pendant l’exécution du travail. La taille maximale de la fenêtre pour ces éléments de requête est de sept jours. 

Le concept de fenêtre temporelle apparaît dans plusieurs éléments de requête Stream Analytics :
1. Agrégats fenêtrés (GROUP BY de fenêtres de Bascule, Récurrente et Glissante)

2. Jointures temporelles (JOIN avec DATEDIFF)

3. Fonctions analytiques temporelles (ISFIRST, LAST et LAG avec LIMIT DURATION)


## <a name="job-recovery-from-node-failure-including-os-upgrade"></a>Récupération de travail après un échec de nœud, y compris la mise à niveau du système d’exploitation
Chaque fois qu’un travail Stream Analytics s’exécute, il est monté en puissance en interne pour pouvoir gérer plusieurs nœuds worker. L’état de chaque nœud worker est consigné dans un point de contrôle à quelques minutes d’intervalle, ce qui permet au système d’effectuer une récupération en cas d’échec.

Dans certains cas, un nœud worker donné peut échouer ou son système d’exploitation peut être mis à niveau. Pour effectuer automatiquement la récupération, Stream Analytics acquiert un nouveau nœud sain et l’état du nœud worker précédent est restauré à partir du dernier point de contrôle disponible. Pour reprendre le travail, une petite relecture est nécessaire afin de restaurer l’état à partir du moment de la capture du point de contrôle. En règle générale, l’écart de restauration est seulement de quelques minutes. Quand le nombre d’unités de streaming sélectionnées est suffisant pour le travail, la relecture doit être effectuée rapidement. 

Dans une requête entièrement parallèle, le temps nécessaire pour rattraper le retard après un échec de nœud worker est proportionnel à l’opération suivante :

[taux d’événements d’entrée] x [longueur d’écart] / [nombre de partitions de traitement]

Si vous observez un retard de traitement important en raison d’un échec de nœud et d’une mise à niveau de système d’exploitation, définissez la requête comme entièrement parallèle et changez l’échelle du travail pour allouer plus d’unités de streaming. Pour plus d’informations, consultez [Mettre à l’échelle des travaux Azure Stream Analytics pour augmenter le débit](stream-analytics-scale-jobs.md).

La version actuelle de Stream Analytics n’affiche pas de rapport quand ce type de processus de récupération a lieu.

## <a name="job-recovery-from-a-service-upgrade"></a>Récupération de travail après une mise à niveau de service 
Microsoft met occasionnellement à niveau les fichiers binaires qui exécutent les travaux Stream Analytics dans le service Azure. À cette occasion, les travaux en cours d’exécution sont mis à niveau vers une version plus récente et le travail redémarre automatiquement. 

Actuellement, le format de point de contrôle de récupération n’est pas conservé d’une mise à niveau à l’autre. Par conséquent, l’état de la requête de streaming doit être restauré entièrement à l’aide de la technique de relecture. Pour autoriser les travaux Stream Analytics à relire exactement la même entrée qu’avant, il faut définir la stratégie de rétention des données sources sur au moins la taille des fenêtres de votre requête. Sinon, vous obtenez des résultats incorrects ou partiels pendant la mise à niveau de service, car les données sources peuvent ne pas être conservées suffisamment longtemps pour contenir la taille de fenêtre entière.

En règle générale, la relecture nécessaire est proportionnelle à la taille de la fenêtre multipliée par le taux d’événements moyen. Par exemple, pour un travail avec un taux d’entrée de 1 000 événements par seconde, une taille de fenêtre supérieure à une heure est considérée comme une grande relecture. Il peut être nécessaire de retraiter jusqu’à une heure de données pour initialiser l’état afin de pouvoir produire des résultats corrects et complets, ce qui peuvent entraîner un retard de sortie (aucune sortie) pour une période prolongée. Les requêtes sans fenêtre ou autre opérateur temporel, tel que `JOIN` ou `LAG`, n’aurait aucune relecture.

## <a name="estimate-replay-catch-up-time"></a>Estimer le temps de rattrapage de relecture
Pour estimer la longueur du délai suite à une mise à niveau de service, vous pouvez suivre cette technique :

1. Chargez le hub d’événements d’entrée avec suffisamment de données pour couvrir la plus grande taille de fenêtre dans votre requête, au taux d’événements attendu. L’horodatage des événements doit être proche du temps horloge de la durée de cette période, puisqu’il s’agit d’un flux d’entrée en direct. Par exemple, si vous avez une fenêtre de 3 jours dans votre requête, envoyez des événements au hub d’événements pendant trois jours, puis continuez à envoyer des événements. 

2. Démarrez le travail en définissant l’heure de début sur **Maintenant**. 

3. Mesurez le temps entre l’heure de début et la génération de la première sortie. La durée correspond approximativement au retard théorique du travail pendant une mise à niveau de service.

4. Si le délai est trop long, essayez de partitionner votre travail et d’augmenter le nombre d’unités de streaming pour répartir la charge sur davantage de nœuds. Sinon, réduisez la taille des fenêtres dans votre requête et effectuez d’autres agrégations ou un autre traitement avec état sur la sortie produite par le travail Stream Analytics dans le récepteur en aval (par exemple, avec Azure SQL Database).

Pour assurer la stabilité générale du service pendant la mise à niveau des travaux critiques, exécutez les travaux en double dans des régions Azure appairées. Pour plus d’informations, consultez [Garantir la fiabilité des travaux Stream Analytics pendant les mises à jour de service](stream-analytics-job-reliability.md).

## <a name="job-recovery-from-a-user-initiated-stop-and-start"></a>Récupération de travail après un arrêt/démarrage lancé par l’utilisateur
Pour modifier la syntaxe de requête sur un travail de streaming ou pour ajuster les entrées et les sorties, le travail doit être arrêté pour effectuer les modifications et mettre à niveau la conception du travail. Dans ce type de scénarios, quand un utilisateur arrête le travail de streaming et le redémarre, le scénario de récupération est similaire à une mise à niveau de service. 

Les données de point de contrôle ne peuvent pas être utilisées pour un redémarrage de travail lancé par l’utilisateur. Pour estimer le délai de sortie pendant ce type de redémarrage, utilisez la même procédure que celle décrite dans la section précédente et appliquez la même solution si le délai est trop long.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la fiabilité et la scalabilité, consultez ces articles :
- [Tutoriel : Configurer des alertes pour des tâches Azure Stream Analytics](stream-analytics-set-up-alerts.md)
- [Mettre à l’échelle des travaux Azure Stream Analytics pour augmenter le débit](stream-analytics-scale-jobs.md)
- [Garantir la fiabilité des travaux Stream Analytics pendant les mises à jour de service](stream-analytics-job-reliability.md)
