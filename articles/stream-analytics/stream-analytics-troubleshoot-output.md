---
title: Résoudre les problèmes liés aux sorties Azure Stream Analytics
description: Cet article décrit des techniques pour résoudre les problèmes liés aux connexions de sortie dans les travaux Azure Stream Analytics.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 10/05/2020
ms.custom: seodec18
ms.openlocfilehash: 6942fd68625fd8eac18ea899330fd99f31f771f7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019835"
---
# <a name="troubleshoot-azure-stream-analytics-outputs"></a>Résoudre les problèmes liés aux sorties Azure Stream Analytics

Cet article décrit les problèmes courants liés aux connexions de sortie Azure Stream Analytics et la façon de résoudre les problèmes de sortie. De nombreuses étapes de résolution des problèmes nécessitent que les journaux de ressources et autres journaux de diagnostic soient activés pour votre travail Stream Analytics. Si les journaux de ressources ne sont pas activés, consultez la section [Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources](stream-analytics-job-diagnostic-logs.md).

## <a name="the-job-doesnt-produce-output"></a>Le travail ne génère pas de sortie

1. Vérifiez la connectivité aux sorties à l’aide du bouton **Tester la connexion** pour chaque sortie.
1. Consultez les [Métriques de surveillance](stream-analytics-monitoring.md) dans l’onglet **Surveiller**. Comme les valeurs sont agrégées, les métriques sont différées de quelques minutes.

   * Si la valeur **Événements d’entrée** est supérieure à zéro, le travail peut lire les données d’entrée. Si la valeur **Événements d’entrée** n’est pas supérieure à zéro, il y a un problème avec l’entrée du travail. Pour plus d’informations, consultez [Résoudre les problèmes liés aux connexions d’entrée](stream-analytics-troubleshoot-input.md). Si votre travail a une entrée de données de référence, appliquez le fractionnement par nom logique lorsque vous examinez la métrique **Événements d’entrée**. S’il n’existe aucun événement d’entrée provenant de vos données de référence, cela signifie probablement que cette source d’entrée n’a pas été configurée correctement pour extraire le jeu de données de référence approprié.
   * Si la valeur **Erreurs de conversion de données** est supérieure à zéro et est en augmentation, consultez [Erreurs de données Azure Stream Analytics](data-errors.md) pour obtenir des informations détaillées sur les erreurs de conversion de données.
   * Si la valeur **Erreurs d’exécution** est supérieure à zéro, votre travail reçoit des données, mais génère des erreurs pendant le traitement de la requête. Pour trouver les erreurs, accédez aux [journaux d’audit](../azure-resource-manager/management/view-activity-logs.md), puis filtrez sur l’état **Échec**.
   * Si la valeur **Événements d’entrée** est supérieure à zéro et que la valeur **Événements de sortie** est égale à zéro, l’une des instructions suivantes a l’état true :
      * Le traitement de la requête a abouti à un nombre nul d’événements de sortie.
      * Les événements ou les champs peuvent être formés de manière inappropriée et n’entraîner aucune sortie après le traitement des requêtes.
      * Le travail n’a pas pu envoyer (push) de données au récepteur de sortie pour des raisons d’authentification ou de connectivité.

   Les messages des journaux d’opérations donnent des détails supplémentaires, notamment sur le déroulé des événements, sauf lorsque la logique de requête filtre l’ensemble des événements. Si le traitement de plusieurs événements génère des erreurs, ces dernières sont regroupées toutes les 10 minutes.

## <a name="the-first-output-is-delayed"></a>La première sortie est retardée

Lorsqu’un travail de Stream Analytics démarre, les événements d’entrée sont lus. Toutefois, dans certaines circonstances la sortie peut être retardée.

Les valeurs de temps importantes dans les éléments de requête temporelle peuvent contribuer au retard de la sortie. Pour produire la sortie correcte sur de grandes fenêtres de temps, le travail de diffusion en continu lit des données à partir du moment le plus récent possible pour remplir la fenêtre de temps. Les données peuvent dater de jusqu’à sept jours. Il n’y a aucune sortie tant que les événements d’entrée en attente ne sont pas lus. Ce problème peut apparaître lorsque le système met à niveau les travaux de diffusion en continu. Lorsqu’une mise à niveau a lieu, le travail redémarre. Ces mises à niveau sont généralement effectuées une fois tous les deux mois.

Faites preuve de discernement lors de la conception d’une requête Stream Analytics. Si vous utilisez une grande fenêtre de temps pour les éléments temporels dans la syntaxe de requête du travail, un retard de la première sortie peut survenir lorsque le travail démarre ou redémarre. Au-delà de plusieurs heures et jusqu’à sept jours sont considérées comme une grande fenêtre de temps.

Une façon d’atténuer ce type de retard de première sortie consiste à utiliser des techniques de parallélisation des requêtes, telles que le partitionnement des données. Vous pouvez aussi ajouter d’autres unités de diffusion en continu pour améliorer le débit jusqu’à ce que le travail soit à jour.  Pour plus d’informations, consultez [Considérations relatives à la création de travaux Stream Analytics](stream-analytics-concepts-checkpoint-replay.md).

Ces facteurs affectent la rapidité d'exécution de la première sortie :

* L’utilisation d’agrégats fenêtrés, tels qu’une clause GROUP BY de fenêtres de bascule, récurrentes et glissantes :

  * Pour les agrégats de fenêtres de bascule ou récurrentes, les résultats sont générés à la fin de la plage de temps de la fenêtre.
  * Pour une fenêtre glissante, les résultats sont générés lorsqu’un événement entre dans la fenêtre glissante ou en sort.
  * Si vous prévoyez d’utiliser une grande taille de fenêtre, par exemple supérieure à une heure, le mieux est de choisir une fenêtre récurrente ou glissante. Ces types de fenêtres vous permettent de voir la sortie plus fréquemment.

* L’utilisation de jointures temporelles, telles que JOIN avec DATEDIFF :
  * génère des correspondances lorsque les deux côtés des événements correspondants surviennent.
  * Les données qui n’ont pas de correspondance telle que LEFT OUTER JOIN sont générées à la fin de la fenêtre DATEDIFF pour chaque événement sur le côté gauche.

* L’utilisation de fonctions analytiques temporelles, telles que ISFIRST, LAST et LAG avec LIMIT DURATION :
  * Pour les fonctions analytiques, la sortie est générée pour chaque événement. Il n’y a pas de retard.

## <a name="the-output-falls-behind"></a>La sortie prend du retard

Pendant le fonctionnement normal d’un travail, la sortie peut avoir des périodes de latence de plus en plus longues. Si la sortie prend un tel retard, vous pouvez identifier les causes racines en examinant les facteurs suivants :

* si le récepteur en aval est limité
* si la source en amont est limitée
* si la logique de traitement de la requête nécessite beaucoup de ressources système

Pour afficher les détails de sortie, sélectionnez le travail de diffusion en continu dans le portail Azure, puis sélectionnez **Diagramme de travail**. Pour chaque entrée, il y a une métrique d’événements de backlog par partition. Si la métrique continue à augmenter, cela indique que les ressources système sont limitées. L’augmentation est potentiellement due à la limitation du récepteur de sortie ou à une forte utilisation de l’UC. Pour plus d’informations, consultez [Débogage piloté par les données à l’aide du diagramme de travail](stream-analytics-job-diagram-with-metrics.md).

## <a name="key-violation-warning-with-azure-sql-database-output"></a>Avertissement de violation de clé avec la sortie Azure SQL Database

Quand vous configurez une base de données Azure SQL comme sortie d’une tâche Stream Analytics, les enregistrements sont insérés en bloc dans la table de destination. En général, Azure Stream Analytics garantit [au moins une remise](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics) au récepteur de sortie. Vous pouvez toujours [obtenir une remise unique]( https://blogs.msdn.microsoft.com/streamanalytics/2017/01/13/how-to-achieve-exactly-once-delivery-for-sql-output/) à une sortie SQL quand une contrainte unique est définie pour une table SQL.

Quand vous configurez des contraintes de clé unique sur la table SQL, Azure Stream Analytics supprime les enregistrements en double. Il fractionne les données en lots qu’il insère les lots de manière récursive jusqu'à trouver un seul enregistrement en double. Le processus de fractionnement et d’insertion ignore les doublons un par un. Pour un travail de diffusion en continu comportant plusieurs lignes en double, le processus est inefficace et prend du temps. Si vous voyez plusieurs messages d’avertissement de violation de clé dans votre journal d’activité pour l’heure précédente, il est probable que votre sortie SQL ralentit l’intégralité de la tâche.

Pour résoudre ce problème, [configurez l’index]( https://docs.microsoft.com/sql/t-sql/statements/create-index-transact-sql) qui provoque la violation de clé en activant l’option IGNORE_DUP_KEY. Cette option permet à SQL d’ignorer les valeurs dupliquées lors des insertions en bloc. Azure SQL Database produit simplement un message d’avertissement au lieu d’une erreur. Par conséquent, Azure Stream Analytics ne produit plus d’erreurs de violation de clé primaire.

Notez les observations suivantes lors de la configuration d’IGNORE_DUP_KEY pour plusieurs types d’index :

* Vous ne pouvez pas définir IGNORE_DUP_KEY sur une clé primaire ni sur une contrainte unique qui utilise ALTER INDEX. Vous devez supprimer l’index et le recréer.  
* Vous pouvez définir IGNORE_DUP_KEY avec ALTER INDEX pour un index unique. Cette instance est différente d’une contrainte PRIMARY KEY/UNIQUE et est créée à l’aide d’une définition CREATE INDEX ou INDEX.  
* L’Option IGNORE_DUP_KEY ne s’applique pas aux index de stockage de colonnes, car vous ne pouvez pas leur imposer l’unicité.

## <a name="sql-output-retry-logic"></a>Logique de nouvelle tentative de sortie SQL

Lorsqu’un travail Stream Analytics avec sortie SQL reçoit le premier lot d’événements, les étapes suivantes se produisent :

1. Le travail tente de se connecter à SQL.
2. Le travail extrait le schéma de la table de destination.
3. Le travail valide les noms de colonne et les types par rapport au schéma de la table de destination.
4. Le travail prépare une table de données en mémoire à partir des enregistrements de sortie dans du lot.
5. Le travail écrit la table de données dans SQL à l’aide de l’[API BulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy.writetoserver).

Au cours de ces étapes, la sortie SQL peut rencontrer les types d’erreurs suivants :

* Les [erreurs temporaires](../azure-sql/database/troubleshoot-common-errors-issues.md#transient-fault-error-messages-40197-40613-and-others) retentées avec une stratégie de nouvelle tentative de backoff exponentiel. L’intervalle minimal avant une nouvelle tentative dépend du code d’erreur individuel, mais les intervalles sont généralement inférieurs à 60 secondes. La limite supérieure peut être de cinq minutes au maximum. 

   [Les échecs de connexion](../azure-sql/database/troubleshoot-common-errors-issues.md#unable-to-log-in-to-the-server-errors-18456-40531) et [les problèmes de pare-feu](../azure-sql/database/troubleshoot-common-errors-issues.md#cannot-connect-to-server-due-to-firewall-issues) sont retentés au moins cinq minutes après la tentative précédente jusqu’à leur réussite.

* Les erreurs de données, telles que les erreurs de forçage de type et les violations de contrainte de schéma, sont gérées à l’aide d’une stratégie d’erreur de sortie. Ces erreurs sont traitées en réessayant des lots de fractionnement binaire jusqu’à ce que l’enregistrement individuel à l’origine de l’erreur soit traité par saut ou nouvelle tentative. La violation de contrainte de clé unique primaire est [toujours gérée](./stream-analytics-troubleshoot-output.md#key-violation-warning-with-azure-sql-database-output).

* Des erreurs non temporaires peuvent se produire en cas de problèmes du service SQL ou de défaillances du code interne. Par exemple, en cas d’erreurs comme (Code 1132) Le pool élastique atteint sa limite de stockage, les nouvelles tentatives ne résolvent pas l’erreur. Dans ces scénarios, le travail Stream Analytics [se dégrade](job-states.md).
* Les délais d’attente `BulkCopy` peuvent expirer pendant l’exécution de `BulkCopy` à l’étape 5. `BulkCopy` peut parfois faire l’expérience d’une expiration des délais d’attente pour les opérations. Le délai minimum configuré par défaut est de cinq minutes et il est doublé lorsqu’il est atteint consécutivement.
Une fois que le délai d’attente est supérieur à 15 minutes, la taille maximale des lots indiquée dans `BulkCopy` est réduite de moitié jusqu’à ce qu’il reste 100 événements par lot.

## <a name="column-names-are-lowercase-in-azure-stream-analytics-10"></a>Le nom des colonnes est en minuscules dans Azure Stream Analytics (1.0)

Avec le niveau de compatibilité d’origine (1.0), Azure Stream Analytics met le nom des colonnes en minuscules. Ce comportement a été résolu dans les niveaux de compatibilité ultérieurs. Pour conserver la casse, passez au niveau de compatibilité 1.1 ou ultérieur. Pour plus d’informations, consultez [Niveau de compatibilité pour les travaux Azure Stream Analytics](./stream-analytics-compatibility-level.md).

## <a name="get-help"></a>Obtenir de l’aide

Pour obtenir de l’aide supplémentaire, essayez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)