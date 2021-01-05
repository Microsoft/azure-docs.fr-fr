---
title: Supervision du portail de gestion des charges de travail
description: Aide pour la supervision du portail de gestion des charges de travail dans Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 13b0dc3af524b16430408f8a920c7477c412414d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91362727"
---
# <a name="azure-synapse-analytics--workload-management-portal-monitoring"></a>Azure Synapse Analytics – Supervision du portail de gestion des charges de travail

Cet article explique comment surveiller l’utilisation des ressources et l’activité des requêtes des [groupes de charge de travail](sql-data-warehouse-workload-isolation.md#workload-groups).
Pour plus d’informations sur la configuration d’Azure Metrics Explorer, consultez l’article [Prise en main d’Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).  Pour plus d’informations sur la supervision de la consommation des ressources système, consultez la section [Utilisation des ressources](sql-data-warehouse-concept-resource-utilization-query-activity.md#resource-utilization) dans la documentation Supervision d’Azure Synapse Analytics.
Il existe deux catégories différentes de mesures de groupe de charge de travail fournies pour la surveillance de la gestion des charges de travail : l’allocation des ressources et l’activité des requêtes.  Ces mesures peuvent être fractionnées et filtrées par groupe de charge de travail.  Les mesures peuvent être fractionnées et filtrées selon qu’elles sont définies par le système (groupes de charge de travail de la classe ressources) ou définis par l’utilisateur (créées par l’utilisateur avec la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)).

## <a name="workload-management-metric-definitions"></a>Définitions de mesures de gestion de la charge de travail

|Nom de métrique                    |Description  |Type d’agrégation |
|-------------------------------|-------------|-----------------|
|Pourcentage de ressources limitées réelles | Le *pourcentage de ressources limitées réelles* est une limite inconditionnelle sur le pourcentage de ressources accessibles par le groupe de charge de travail, en tenant compte du *pourcentage minimal de ressources réelles* alloué pour d’autres groupes de charge de travail. La métrique *Pourcentage de ressources limitées réelles* est configurée à l’aide du paramètre `CAP_PERCENTAGE_RESOURCE` dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  La valeur effective est décrite ici.<br><br>Par exemple, si un groupe de charge de travail `DataLoads` est créé avec `CAP_PERCENTAGE_RESOURCE` = 100 et qu’un autre groupe de charge de travail est créé avec un pourcentage minimal de ressources réelles de 25 %, le *pourcentage de ressources limitées réelles* pour le groupe de charge de travail `DataLoads` est de 75 %.<br><br>Le *pourcentage de ressources limitées réelles* détermine la limite supérieure de concurrence (et, par conséquent, le débit potentiel) qu’un groupe de charge de travail peut atteindre.  Si un débit supplémentaire est nécessaire au-delà de ce qui est actuellement signalé par la métrique *Pourcentage de ressources limitées réelles*, augmentez `CAP_PERCENTAGE_RESOURCE`, diminuez `MIN_PERCENTAGE_RESOURCE` d’autres groupes de charge de travail ou montez en puissance l’instance pour ajouter d’autres ressources.  La diminution de `REQUEST_MIN_RESOURCE_GRANT_PERCENT` peut augmenter la concurrence, mais peut ne pas augmenter le débit global.| Min, Moy, Max |
|Pourcentage minimal de ressources réelles |Le *pourcentage minimal de ressources réelles* est le pourcentage minimal de ressources réservées et isolées pour le groupe de charge de travail, en tenant compte du niveau de service minimum.  La métrique Pourcentage minimal de ressources réelles est configurée à l’aide du paramètre `MIN_PERCENTAGE_RESOURCE` dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  La valeur effective est décrite [ici](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest#effective-values).<br><br>Utilisez le type d’agrégation Somme lorsque cette métrique est non filtrée et non fractionnée pour surveiller l’isolation totale de la charge de travail configurée sur le système.<br><br>Le *pourcentage minimal de ressources réelles* détermine la limite inférieure de la concurrence garantie (et, par conséquent, le débit garanti) qu’un groupe de charge de travail peut atteindre.  Si des ressources garanties supplémentaires sont nécessaires au-delà de ce qui est actuellement signalé par la métrique *pourcentage minimal de ressources réelles*, augmentez le paramètre `MIN_PERCENTAGE_RESOURCE` configuré pour le groupe de charge de travail.  La diminution de `REQUEST_MIN_RESOURCE_GRANT_PERCENT` peut augmenter la concurrence, mais peut ne pas augmenter le débit global. |Min, Moy, Max|
|Requêtes actives du groupe de charge de travail  |Cette métrique signale les requêtes actives dans le groupe de charge de travail.  L’utilisation de cette métrique non filtrée et non fractionnée affiche toutes les requêtes actives en cours d’exécution sur le système.|SUM         |
|Allocation du groupe de charge de travail par pourcentage maximal de ressources |Cette mesure affiche l’allocation en pourcentage des ressources par rapport au *pourcentage de ressources limitées réelles* par groupe de charge de travail.  Cette métrique fournit l’utilisation effective du groupe de charge de travail.<br><br>Prenons l’exemple d’un groupe de charge de travail `DataLoads` avec un *pourcentage de ressources limitées réelles* de 75 % et `REQUEST_MIN_RESOURCE_GRANT_PERCENT` configuré à 25 %.  La valeur *Allocation de groupe de charge de travail par pourcentage maximal de ressources* filtrée sur `DataLoads` serait 33 % (25 %/75 %) si une requête unique s’exécutait dans ce groupe de charge de travail.<br><br>Utilisez cette métrique pour identifier l’utilisation d’un groupe de charge de travail.  Une valeur proche de 100 % indique que toutes les ressources disponibles pour le groupe de charge de travail sont utilisées.  En outre, la métrique *Requêtes en file d’attente du groupe de charge de travail* pour le même groupe de charge de travail, indiquant une valeur supérieure à zéro, indiquerait que le groupe de charge de travail utiliserait des ressources supplémentaires si elles étaient allouées.  À l’inverse, si cette métrique est régulièrement faible et que celle de *Requêtes actives du groupe de charge de travail* est faible, le groupe de charge de travail n’est pas utilisé.  Cette situation est particulièrement problématique si le *pourcentage de ressources limitées réelles* est supérieur à zéro, car cela indique une [isolation de la charge de travail sous-exploitée](#underutilized-workload-isolation).|Min, Moy, Max |
|Allocation du groupe de charge de travail par pourcentage système | Cette métrique affiche le pourcentage d’allocation des ressources par rapport à l’ensemble du système.<br><br>Prenons l’exemple d’un groupe de charge de travail `DataLoads` avec `REQUEST_MIN_RESOURCE_GRANT_PERCENT` configuré à 25 %.  La valeur *Allocation de groupe de charge de travail par pourcentage système* filtrée sur `DataLoads` serait 25 % (25 %/100 %) si une requête unique s’exécutait dans ce groupe de charge de travail.|Min, Moy, Max |
|Délais d’expiration des requêtes du groupe de charge de travail |Requêtes pour le groupe de charge de travail dont le délai a expiré.  Les délais d’expiration des requêtes signalés par cette métrique démarrent uniquement une fois que l’exécution de la requête a commencé (cela n’inclut pas le temps d’attente dû au verrouillage ou à l’attente des ressources).<br><br>Le délai d’expiration des requêtes est configuré à l’aide du paramètre `QUERY_EXECUTION_TIMEOUT_SEC` dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  L’augmentation de la valeur peut réduire le nombre de délais d’expiration de requête.<br><br>Envisagez d’augmenter le paramètre `REQUEST_MIN_RESOURCE_GRANT_PERCENT` pour le groupe de charge de travail afin de réduire le nombre de délais d’expiration et d’allouer davantage de ressources par requête.  Notez que l’augmentation de `REQUEST_MIN_RESOURCE_GRANT_PERCENT` réduit la quantité de concurrence pour le groupe de charge de travail. |SUM |
|Requêtes en file d’attente du groupe de charge de travail | Requêtes pour le groupe de charge de travail actuellement en file d’attente pour en commencer l’exécution.  Les requêtes peuvent être en file d’attente parce qu’elles sont en attente de ressources ou de verrous.<br><br>Les requêtes peuvent être en attente pour de nombreuses raisons.  Si le système est surchargé et que la demande de concurrence est supérieure à ce qui est disponible, les requêtes sont mises en file d’attente.<br><br>Envisagez d’ajouter davantage de ressources au groupe de charge de travail en augmentant le paramètre `CAP_PERCENTAGE_RESOURCE` dans l’instruction [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Si la valeur de `CAP_PERCENTAGE_RESOURCE` est supérieure à la métrique *Pourcentage de ressources limitées réelles*, l’isolation de la charge de travail configurée pour d’autres groupes de charge de travail a un impact sur les ressources allouées à ce groupe de charge de travail.  Envisagez de réduire le paramètre `MIN_PERCENTAGE_RESOURCE` d’autres groupes de charge de travail ou de monter en puissance l’instance pour ajouter des ressources supplémentaires. |SUM |

## <a name="monitoring-scenarios-and-actions"></a>Scénarios et actions de surveillance

Vous trouverez ci-dessous une série de configurations de graphiques pour mettre en évidence l’utilisation des métriques de gestion de la charge de travail pour la détection d’un problème, ainsi que les actions associées pour le résoudre.

### <a name="underutilized-workload-isolation"></a>Isolation de la charge de travail sous-exploitée

Considérons le groupe de charge de travail et la configuration de classifieur suivants selon lesquels un groupe de charge de travail nommé `wgPriority` est créé et *TheCEO* `membername` est mappé à celui-ci à l’aide du classifieur de charge de travail `wcCEOPriority`.  Le groupe de charge de travail `wgPriority` a une isolation de la charge de travail de 25 % configurée (`MIN_PERCENTAGE_RESOURCE` = 25).  Chaque requête soumise par *TheCEO* reçoit 5 % des ressources système (`REQUEST_MIN_RESOURCE_GRANT_PERCENT` = 5).

```sql
CREATE WORKLOAD GROUP wgPriority
WITH ( MIN_PERCENTAGE_RESOURCE = 25
      ,CAP_PERCENTAGE_RESOURCE = 50
      ,REQUEST_MIN_RESOURCE_GRANT_PERCENT = 5);

CREATE WORKLOAD CLASSIFIER wcCEOPriority
WITH ( WORKLOAD_GROUP = 'wgPriority'
      ,MEMBERNAME = 'TheCEO');
```

Le graphique ci-dessous est configuré comme suit :<br>
Métrique 1 : *Pourcentage minimal de ressources réelles* (agrégation Moy, `blue line`)<br>
Métrique 2 : *Allocation du groupe de charge de travail par pourcentage système* (agrégation Moy, `purple line`)<br>
Filtre : [Workload Group] = `wgPriority`<br>
![Capture d’écran montrant un graphique avec deux métriques et un filtre.](./media/sql-data-warehouse-workload-management-portal-monitor/underutilized-wg.png)
Le graphique montre qu’avec une isolation de la charge de travail à 25 %, seul 10 % sont utilisés en moyenne.  Dans ce cas, la valeur du paramètre `MIN_PERCENTAGE_RESOURCE` peut être diminuée de 10 ou 15 points et ainsi permettre à d’autres charges de travail sur le système d’utiliser les ressources.

### <a name="workload-group-bottleneck"></a>Goulot d’étranglement du groupe de charge de travail

Considérons le groupe de charge de travail et la configuration de classifieur suivants selon lesquels un groupe de charge de travail nommé `wgDataAnalyst` est créé et *DataAnalyst* `membername` est mappé à celui-ci à l’aide du classifieur de charge de travail `wcDataAnalyst`.  Le groupe de charge de travail `wgDataAnalyst` a une isolation de la charge de travail de 6 % configurée (`MIN_PERCENTAGE_RESOURCE` = 6) et des ressources limitées à 9 % (`CAP_PERCENTAGE_RESOURCE` = 9).  Chaque requête soumise par *DataAnalyst* reçoit 3 % des ressources système (`REQUEST_MIN_RESOURCE_GRANT_PERCENT` = 3).

```sql
CREATE WORKLOAD GROUP wgDataAnalyst  
WITH ( MIN_PERCENTAGE_RESOURCE = 6
      ,CAP_PERCENTAGE_RESOURCE = 9
      ,REQUEST_MIN_RESOURCE_GRANT_PERCENT = 3);

CREATE WORKLOAD CLASSIFIER wcDataAnalyst
WITH ( WORKLOAD_GROUP = 'wgDataAnalyst'
      ,MEMBERNAME = 'DataAnalyst');
```

Le graphique ci-dessous est configuré comme suit :<br>
Métrique 1 : *Pourcentage de ressources limitées réelles* (agrégation Moy, `blue line`)<br>
Métrique 2 : *Allocation du groupe de charge de travail par pourcentage maximal de ressources* (agrégation Moy, `purple line`)<br>
Métrique 3 : *Requêtes en file d’attente du groupe de charge de travail* (agrégation Somme, `turquoise line`)<br>
Filtre : [Workload Group] = `wgDataAnalyst`<br>
![Capture d’écran montrant un graphique avec les trois métriques et un filtre.](./media/sql-data-warehouse-workload-management-portal-monitor/bottle-necked-wg.png)
Le graphique montre qu’avec une limite de 9 % sur les ressources, le groupe de charge de travail est utilisé à plus de 90 % (d’après la *métrique Allocation du groupe de charge de travail par pourcentage maximal de ressources*).  Il existe une mise en file d’attente stable des requêtes, comme indiqué dans la *métrique Requêtes en file d’attente du groupe de charge de travail*.  Dans ce cas, l’augmentation de `CAP_PERCENTAGE_RESOURCE` à une valeur supérieure à 9 % permettra à plus de requêtes de s’exécuter simultanément.  L’augmentation de `CAP_PERCENTAGE_RESOURCE` suppose que suffisamment de ressources sont disponibles et non isolées par d’autres groupes de charge de travail.  Vérifiez l’augmentation de la limite en vérifiant la *métrique Pourcentage de ressources limitées réelles*.  Si vous souhaitez obtenir un débit plus élevé, envisagez également d’augmenter `REQUEST_MIN_RESOURCE_GRANT_PERCENT` à une valeur supérieure à 3.  L’augmentation de `REQUEST_MIN_RESOURCE_GRANT_PERCENT` peut permettre l’exécution plus rapide des requêtes.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Configurer l’isolation de la charge de travail à l’aide de T-SQL](quickstart-configure-workload-isolation-tsql.md)<br>
- [CREATE WORKLOAD GROUP (Transact-SQL)](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)<br>
- [CREATE WORKLOAD CLASSIFIER (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)<br>
- [Surveillance de l’utilisation des ressources](sql-data-warehouse-concept-resource-utilization-query-activity.md)
