---
title: Types de problèmes de performances des requêtes dans Azure SQL Database
description: Dans cet article, découvrez les types de problèmes de performances des requêtes dans Azure SQL Database, et apprenez également à identifier et résoudre les requêtes rencontrant ces problèmes.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: troubleshooting
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: wiassaf, sstein
ms.date: 03/10/2020
ms.openlocfilehash: 6ea17f04538e3444b1baddaa8862add2cfbbaa9c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493421"
---
# <a name="detectable-types-of-query-performance-bottlenecks-in-azure-sql-database"></a>Types détectables de goulots d’étranglement des performances des requêtes dans Azure SQL Database
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Quand vous tentez de résoudre un goulot d’étranglement des performances, commencez par déterminer si ce goulot d’étranglement se produit pendant que la requête est en cours d’exécution ou en attente. Des résolutions différentes s’appliquent en fonction du résultat de cette détermination. Utilisez le diagramme suivant pour comprendre les facteurs qui peuvent entraîner un problème lié à l’exécution ou un problème lié à l’attente. Les problèmes et résolutions relatifs à chaque type de problème sont décrits dans cet article.

Vous pouvez utiliser Azure SQL Database [Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#detectable-database-performance-patterns) ou des [vues de gestion dynamique (DMV)](database/monitoring-with-dmvs.md) SQL Server pour détecter ces types de goulots d’étranglement des performances.

![États de la charge de travail](./media/identify-query-performance-issues/workload-states.png)

**Problèmes liés à l’exécution** : les problèmes liés à l’exécution sont généralement associés à des problèmes de compilation, ce qui entraîne des problèmes liés à un plan de requête non optimal ou à l’exécution associés à des ressources insuffisantes ou surutilisées.
**Problèmes liés à l’attente** : les problèmes liés à l’attente sont généralement associés aux éléments suivants :

- Verrous (blocage)
- E/S
- Contention liée à l’utilisation de TempDB
- Attentes d’allocation de mémoire

## <a name="compilation-problems-resulting-in-a-suboptimal-query-plan"></a>Problèmes de compilation entraînant un plan de requête non optimal

Un plan non optimal généré par l’optimiseur de requête SQL peut être à l’origine du ralentissement des performances des requêtes. L’optimiseur de requête SQL peut produire un plan non optimal à cause d’un index manquant, de statistiques périmées, d’une estimation incorrecte du nombre de lignes à traiter ou bien d’une estimation inexacte de la mémoire nécessaire. Si vous savez que la requête a été exécutée plus rapidement dans le passé ou sur une autre instance, comparez les plans d’exécution réels pour voir s’ils diffèrent.

- Identifiez les index manquants à l’aide de l’une des méthodes suivantes :

  - Utilisation d’[Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#missing-index).
  - [Database Advisor](database/database-advisor-implement-performance-recommendations.md) pour les bases de données uniques et mises en pool.
  - Vues de gestion dynamique (DMV). Cet exemple montre l’impact d’un index manquant, comment détecter des[index manquants](database/performance-guidance.md#identifying-and-adding-missing-indexes) à l’aide de vues DMV et l’impact de l’implémentation de la recommandation d’index manquant.
- Essayez d’appliquer des [indicateurs de requête](/sql/t-sql/queries/hints-transact-sql-query), de [mettre à jour les statistiques](/sql/t-sql/statements/update-statistics-transact-sql) ou de [regénérer les index](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes) pour obtenir le meilleur plan. Activez la [correction automatique des plans](../azure-sql/database/automatic-tuning-overview.md) dans Azure SQL Database pour atténuer automatiquement ces problèmes.

  Cet [exemple](database/performance-guidance.md#query-tuning-and-hinting) montre l’impact d’un plan de requête non optimal en raison d’une requête paramétrée, comment détecter cette condition et comment utiliser un indicateur de requête pour résoudre le problème.

- Essayez de changer le niveau de compatibilité de la base de données et d’implémenter le traitement de requêtes intelligent. L’optimiseur de requête SQL peut générer un autre plan de requête en fonction du niveau de compatibilité de votre base de données. Des niveaux de compatibilité plus élevés fournissent plus de [fonctionnalités du traitement de requêtes intelligentes](/sql/relational-databases/performance/intelligent-query-processing).

  - Pour plus d’informations sur le traitement des requêtes, consultez le [Guide d’architecture de traitement des requêtes](/sql/relational-databases/query-processing-architecture-guide).
  - Pour changer les niveaux de compatibilité de la base de données et en savoir plus sur les différences entre les niveaux de compatibilité, consultez [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level).
  - Pour en savoir plus sur l’estimation de la cardinalité, consultez [Estimation de la cardinalité](/sql/relational-databases/performance/cardinality-estimation-sql-server).

## <a name="resolving-queries-with-suboptimal-query-execution-plans"></a>Résolution de requêtes avec des plans d’exécution de requête non optimaux

Les sections suivantes expliquent comment résoudre les requêtes avec un plan d’exécution de requête non optimal.

### <a name="queries-that-have-parameter-sensitive-plan-psp-problems"></a><a name="ParamSniffing"></a> Requêtes confrontées à des problèmes de plan sensible aux paramètres

Un problème de plan sensible aux paramètres se produit quand l’optimiseur de requête génère un plan d’exécution de requête qui est optimal uniquement pour une valeur de paramètre (ou un ensemble de valeurs) spécifique. Le plan mis en cache n’est donc pas optimal pour les valeurs de paramètre qui sont utilisées dans des exécutions consécutives. Les plans non optimaux peuvent alors provoquer des problèmes de performances des requêtes et entraîner une dégradation du débit global de la charge de travail.

Pour plus d’informations sur la détection des paramètres et le traitement des requêtes, consultez le [Guide de l’architecture de traitement des requêtes](/sql/relational-databases/query-processing-architecture-guide#ParamSniffing).

Plusieurs solutions de contournement peuvent atténuer les problèmes de plan sensible aux paramètres. Chaque solution de contournement présente des avantages et des inconvénients :

- Utilisez l’indicateur de requête [RECOMPILE](/sql/t-sql/queries/hints-transact-sql-query) à chaque exécution de la requête. Cette solution compense la durée de compilation et augmente l’UC pour un plan de meilleure qualité. L’option `RECOMPILE` n’est souvent pas possible pour les charges de travail exigeant un débit élevé.
- Utilisez l’indicateur de requête [OPTION (OPTIMIZE FOR...) ](/sql/t-sql/queries/hints-transact-sql-query) pour remplacer la valeur de paramètre réelle par une valeur de paramètre standard qui produit un plan suffisant pour la plupart des possibilités de valeurs de paramètre. Cette option nécessite une bonne compréhension des valeurs de paramètre optimal et des caractéristiques du plan associé.
- Utilisez l’indicateur de requête [OPTION (OPTIMIZE FOR UNKNOWN)](/sql/t-sql/queries/hints-transact-sql-query) pour remplacer la valeur de paramètre réelle par la moyenne du vecteur de densité. Pour effectuer cette opération, vous pouvez également capturer les valeurs de paramètres entrantes dans des variables locales, puis utiliser ces variables locales dans des prédicats à la place des paramètres. Pour cette correction, la densité moyenne doit être *suffisante*.
- Désactivez entièrement la détection de paramètres en spécifiant l’indicateur de requête [DISABLE_PARAMETER_SNIFFING](/sql/t-sql/queries/hints-transact-sql-query).
- Utilisez l’indicateur de requête [KEEPFIXEDPLAN](/sql/t-sql/queries/hints-transact-sql-query) pour empêcher les recompilations dans le cache. Cette solution de contournement suppose que le plan courant suffisant est celui qui se trouve déjà dans le cache. Vous pouvez également désactiver les mises à jour de statistiques automatiques afin de réduire le risque d’éviction du plan suffisant et de compilation d’un plan insuffisant.
- Forcez le plan en spécifiant explicitement l’indicateur de requête [USE PLAN](/sql/t-sql/queries/hints-transact-sql-query) dans le texte de la requête. Ou définissez un plan spécifique en utilisant le Magasin des requêtes ou en activant le [réglage automatique](../azure-sql/database/automatic-tuning-overview.md).
- Remplacez la procédure unique par un ensemble imbriqué de procédures, qui peuvent être utilisées en fonction d’une logique conditionnelle et des valeurs de paramètre associées.
- Créez des alternatives d’exécution de chaîne dynamique sur une définition de procédure statique.

Pour plus d’informations sur la résolution des problèmes de plan sensible aux paramètres, consultez ces billets de blog :

- [I smell a parameter](/archive/blogs/queryoptteam/i-smell-a-parameter)
- [Conor vs. dynamic SQL vs. procedures vs. plan quality for parameterized queries](/archive/blogs/conor_cunningham_msft/conor-vs-dynamic-sql-vs-procedures-vs-plan-quality-for-parameterized-queries)
- [SQL query optimization techniques in SQL Server: Parameter sniffing](https://www.sqlshack.com/query-optimization-techniques-in-sql-server-parameter-sniffing/)

### <a name="compile-activity-caused-by-improper-parameterization"></a>Compilations provoquées par un paramétrage incorrect

Lorsqu’une requête contient des littéraux, le moteur de base de données paramètre automatiquement l’instruction, ou un utilisateur paramètre explicitement l’instruction pour réduire le nombre de compilations. Un nombre élevé de compilations pour une requête utilisant le même modèle mais différentes valeurs littérales peut entraîner une augmentation de l’utilisation du processeur. De même, si vous ne paramétrez que partiellement une requête qui garde des littéraux, le moteur de base de données ne paramètre pas davantage la requête.

Voici un exemple de requête partiellement paramétrée :

```sql
SELECT *
FROM t1 JOIN t2 ON t1.c1 = t2.c1
WHERE t1.c1 = @p1 AND t2.c2 = '961C3970-0E54-4E8E-82B6-5545BE897F8F'
```

Dans cet exemple, `t1.c1` prend la valeur `@p1`, mais `t2.c2` garde un GUID comme littéral. Dans ce cas, si vous changez la valeur de `c2`, la requête est traitée comme une requête différente, ce qui engendre une nouvelle compilation. Pour réduire le nombre de compilations dans cet exemple, vous pouvez également paramétrer le GUID.

La requête suivante affiche le nombre de requêtes par hachage de requête pour déterminer si une requête est correctement paramétrée :

```sql
SELECT TOP 10
  q.query_hash
  , count (distinct p.query_id ) AS number_of_distinct_query_ids
  , min(qt.query_sql_text) AS sampled_query_text
FROM sys.query_store_query_text AS qt
  JOIN sys.query_store_query AS q
     ON qt.query_text_id = q.query_text_id
  JOIN sys.query_store_plan AS p
     ON q.query_id = p.query_id
  JOIN sys.query_store_runtime_stats AS rs
     ON rs.plan_id = p.plan_id
  JOIN sys.query_store_runtime_stats_interval AS rsi
     ON rsi.runtime_stats_interval_id = rs.runtime_stats_interval_id
WHERE
  rsi.start_time >= DATEADD(hour, -2, GETUTCDATE())
  AND query_parameterization_type_desc IN ('User', 'None')
GROUP BY q.query_hash
ORDER BY count (distinct p.query_id) DESC
```

### <a name="factors-that-affect-query-plan-changes"></a>Facteurs impactant les changements de plan de requête

Une recompilation du plan d’exécution de requête peut aboutir à un plan de requête généré qui diffère du plan initial mis en cache. Un plan initial existant peut être automatiquement recompilé pour diverses raisons :

- Des changements dans le schéma sont référencés par la requête
- Des changements de données dans les tables sont référencés par la requête
- Des options de contexte de requête ont été changées

Un plan compilé peut être supprimé du cache pour plusieurs raisons. Par exemple :

- Redémarrages d’instance
- Changements de configuration dont l’étendue est la base de données
- Sollicitation de la mémoire
- Requêtes explicites d’effacement du cache

Si vous utilisez un indicateur RECOMPILE, le plan n’est pas mis en cache.

Une recompilation (ou une nouvelle compilation après éviction du cache) peut toujours aboutir à la génération d’un plan d’exécution de requête identique au plan initial. Quand le plan change par rapport au plan précédent ou au plan initial, cela s’explique généralement par l’une des raisons suivantes :

- **Modification de la conception physique** : par exemple, de nouveaux index créés couvrent plus efficacement les exigences d’une requête. Les nouveaux index peuvent être utilisés sur une nouvelle compilation si l’optimiseur de requête juge que l’utilisation des nouveaux index aboutit à un plan plus optimal que l’utilisation de la structure de données qui avait été initialement sélectionnée pour la première version de l’exécution de la requête. Toute modification physique des objets référencés peut entraîner un nouveau choix de plan au moment de la compilation.

- **Différences au niveau des ressources serveur** : quand un plan d’un système diffère du plan d’un autre système, la disponibilité des ressources, comme le nombre de processeurs disponibles, peut déterminer le choix du plan généré. Par exemple, si un système a davantage de processeurs, un plan parallèle peut être choisi.

- **Statistiques différentes** : les statistiques associées aux objets référencés ont peut-être changé ou sont peut-être matériellement différentes par rapport aux statistiques du système initiales. Si les statistiques changent et qu’une recompilation a lieu, l’optimiseur de requête utilise les statistiques à partir du changement. Les fréquences et distributions de données indiquées dans les statistiques révisées peuvent différer de celles de la compilation initiale. Ces changements sont utilisés pour estimer la cardinalité. (Les *estimations de cardinalité* correspondent au nombre de lignes qui sont supposées être transmises dans l’arborescence de requêtes logique.) Les changements apportés aux estimations de cardinalité peuvent vous amener à choisir des opérateurs physiques et ordres d’opérations associés différents. Même des modifications mineures apportées aux statistiques peuvent entraîner une modification du plan d’exécution de la requête.

- **Changement du niveau de compatibilité de la base de données ou de la version de l’estimateur de cardinalité** : un changement du niveau de compatibilité de la base de données peut donner lieu à de nouvelles stratégies et caractéristiques qui peuvent entraîner un plan d’exécution de requête différent. Au-delà du niveau de compatibilité de la base de données, l’activation ou la désactivation d’un indicateur de trace 4199 ou bien un changement d’état de l’indicateur QUERY_OPTIMIZER_HOTFIXES de configuration à l’échelle de la base de données peuvent également impacter les choix de plan d’exécution de requête au moment de la compilation. Les indicateurs de trace 9481 (forcer le CE hérité) et 2312 (forcer le CE par défaut) impactent également le choix du plan.

## <a name="resource-limits-issues"></a>Problèmes de limites des ressources

Les performances de requêtes lentes non liées aux plans de requête non optimaux et aux index manquants sont généralement associées à des ressources insuffisantes ou surutilisées. Si le plan de requête est optimal, la requête (et la base de données) peuvent atteindre les limites de ressources pour la base de données, le pool élastique ou l’instance managée. Le débit d’écriture du journal excessif pour le niveau de service en est un exemple.

- Détection des problèmes de ressources à l’aide du portail Azure : Pour déterminer si les limites de ressources sont à l’origine du problème, consultez [Supervision des ressources SQL Database](database/monitor-tune-overview.md#azure-sql-database-and-azure-sql-managed-instance-resource-monitoring). Pour les bases de données uniques et les pools élastiques, consultez [Recommandations relatives aux performances de Database Advisor](database/database-advisor-implement-performance-recommendations.md) et [Query Performance Insights](database/query-performance-insight-use.md).
- Détection des limites de ressources à l’aide d’[Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#reaching-resource-limits)
- Détection des problèmes de ressources à l’aide de [vues de gestion dynamique (DMV)](database/monitoring-with-dmvs.md) :

  - La DMV [sys.dm_db_resource_stats](database/monitoring-with-dmvs.md#monitor-resource-use) retourne la consommation du processeur, des E/S et de la mémoire pour la base de données. Il y a une ligne pour chaque intervalle de 15 secondes, même s’il n’existe aucune activité dans la base de données. Les données historiques sont conservées pendant une heure.
  - La DMV [sys.resource_stats](database/monitoring-with-dmvs.md#monitor-resource-use) retourne les données d’utilisation du processeur et les données sur le stockage pour Azure SQL Database. Les données sont collectées et agrégées par intervalles de cinq minutes.
  - [De nombreuses requêtes individuelles qui consomment une quantité cumulée élevée de processeur](database/monitoring-with-dmvs.md#many-individual-queries-that-cumulatively-consume-high-cpu)

Si vous identifiez que le problème est une ressource insuffisante, vous pouvez mettre à niveau des ressources pour augmenter la capacité de votre base de données afin d’absorber les besoins en processeur. Pour plus d’informations, consultez [Mettre à l’échelle des ressources de base de données unique dans Azure SQL Database](database/single-database-scale.md) et [Mettre à l’échelle un pool élastique dans Azure SQL Database](database/elastic-pool-scale.md). Pour plus d’informations sur la mise à l’échelle d’une instance managée, consultez [Limites de ressources des niveaux de service](managed-instance/resource-limits.md#service-tier-characteristics)

## <a name="performance-problems-caused-by-increased-workload-volume"></a>Problèmes de performances dus à une hausse du volume de la charge de travail

Une hausse du trafic d’application et du volume de la charge de travail peut entraîner une plus forte sollicitation du processeur. Prenez toutefois le temps de bien diagnostiquer ce problème. Si vous constatez un problème d’utilisation intensive du processeur, posez-vous ces questions pour déterminer si la hausse est due à une variation du volume de la charge de travail :

- Les requêtes de l’application sont-elles à l’origine du problème d’utilisation intensive du processeur ?
- Pour les [principales requêtes consommatrices du processeur que vous pouvez identifier](database/monitoring-with-dmvs.md#the-cpu-issue-occurred-in-the-past) :

  - Y avait-il plusieurs plans d’exécution associés à la même requête ? Le cas échéant, pour quelle raison ?
  - Entre les requêtes ayant le même plan d’exécution, les temps d’exécution étaient-ils similaires ? Y a-t-il eu davantage d’exécutions ? Si c’est le cas, la hausse de la charge de travail est probablement à l’origine des problèmes de performances.

En résumé, si le plan d’exécution de requête s’est exécuté de la même manière, mais que l’utilisation du processeur a augmenté en même temps que le nombre d’exécutions, le problème de performances est probablement lié à une hausse de la charge de travail.

Il est souvent difficile d’identifier qu’une variation du volume de la charge de travail est à l’origine d’un problème d’utilisation du processeur. Tenez compte de ces facteurs :

- **Changement dans l’utilisation des ressources** : par exemple, imaginez un scénario où l’utilisation du processeur a enregistré une hausse de 80 pour cent durant une période prolongée. Cela ne suffit pas pour conclure à une variation du volume de la charge de travail. Des régressions dans le plan d’exécution de requête et des changements dans la distribution des données peuvent également entraîner une plus grande utilisation des ressources, même si l’application exécute la même charge de travail.

- **Apparence d’une nouvelle requête** : une application peut exécuter un nouvel ensemble de requêtes à des moments différents.

- **Augmentation ou diminution du nombre de requêtes** : ce scénario constitue la mesure la plus évidente de la charge de travail. Le nombre de requêtes ne correspond pas toujours à une augmentation de l’utilisation des ressources. Toutefois, cette mesure est toujours un signal important, qui suppose que d’autres facteurs restent inchangés.

Utilisez Intelligent Insights pour détecter les [augmentations de charge de travail](database/intelligent-insights-troubleshoot-performance.md#workload-increase) et les [régressions de plan](database/intelligent-insights-troubleshoot-performance.md#plan-regression).

## <a name="waiting-related-problems"></a>Problèmes liés à l’attente

Une fois que vous avez éliminé les problèmes liés à un plan non optimal et ceux *liés à l’attente* qui sont associés à des problèmes d’exécution, le problème de performances est généralement dû au fait que les requêtes attendent probablement des ressources. Les problèmes liés à l’attente peuvent être provoqués par :

- Un **blocage** :

  une requête peut maintenir le verrou sur certains objets de la base de données pendant que d’autres requêtes essaient d’accéder aux mêmes objets. Vous pouvez identifier les requêtes bloquantes à l’aide de [vues de gestion dynamiques (DMV)](database/monitoring-with-dmvs.md#monitoring-blocked-queries) ou d’[Intelligent Insights](database/intelligent-insights-troubleshoot-performance.md#locking).
- **Problèmes d’E/S**

  les requêtes peuvent attendre que les pages soient écrites dans les fichiers de données ou les fichiers journaux. Dans ce cas, examinez les statistiques d’attente `INSTANCE_LOG_RATE_GOVERNOR`, `WRITE_LOG` ou `PAGEIOLATCH_*` dans la DMV. Consultez l’utilisation des vues de gestion dynamique (DMV) pour [identifier les problèmes de performances d’E/S](database/monitoring-with-dmvs.md#identify-io-performance-issues).
- **Problèmes liés à TempDB**

  si la charge de travail utilise des tables temporaires ou s’il y a des dépassements de TempDB dans les plans, les requêtes ont peut-être un problème avec le débit de TempDB. Consultez l’utilisation des vues de gestion dynamique (DMV) pour [identifier les problèmes liés à TempDB](database/monitoring-with-dmvs.md#identify-tempdb-performance-issues).
- **Problèmes liés à la mémoire**

  si la charge de travail n’a pas assez de mémoire, l’espérance de vie des pages peut diminuer, ou les requêtes risquent de ne pas obtenir toute la mémoire dont elles ont besoin. Dans certains cas, l’intelligence intégrée dans l’optimiseur de requête résoudra les problèmes de mémoire. Consultez l’utilisation des vues de gestion dynamique (DMV) pour [identifier les problèmes liés à l’allocation de mémoire](database/monitoring-with-dmvs.md#identify-memory-grant-wait-performance-issues).

### <a name="methods-to-show-top-wait-categories"></a>Méthodes permettant d’afficher les principales catégories d’attente

Ces méthodes sont couramment utilisées pour afficher les principales catégories de types d’attentes :

- Utilisez Intelligent Insights pour identifier les requêtes subissant une dégradation des performances en raison de l’[augmentation des attentes](database/intelligent-insights-troubleshoot-performance.md#increased-wait-statistic).
- Utilisez le [Magasin des requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) pour avoir des statistiques sur les attentes par requête dans le temps. Dans le magasin des requêtes, les types d’attentes sont combinés en catégories d’attentes. Vous pouvez consulter une table de mappage des catégories d’attentes avec les types d’attentes dans [sys.query_store_wait_stats](/sql/relational-databases/system-catalog-views/sys-query-store-wait-stats-transact-sql#wait-categories-mapping-table).
- Utilisez [sys.dm_db_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database) pour retourner des informations sur toutes les attentes rencontrées par les threads exécutés pendant une opération de requête. Vous pouvez utiliser cette vue agrégée pour diagnostiquer les problèmes de performances avec Azure SQL Database, ainsi qu’avec des requêtes et des lots spécifiques. Les requêtes peuvent subir des attentes sur les ressources, des attentes sur la file d’attente ou des attentes externes.
- Utilisez [sys.dm_os_waiting_tasks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql) pour retourner des informations sur la file d’attente des tâches qui sont en attente de certaines ressources.

Dans les scénarios d’utilisation intensive du processeur, le Magasin des requêtes et les statistiques d’attente ne reflètent pas forcément l’utilisation du processeur dans les cas suivants :

- Des requêtes grandes consommatrices de ressources processeur sont toujours en cours d’exécution.
- Les requêtes consommant beaucoup de ressources processeur étaient en cours d’exécution au moment d’un basculement.

Les vues de gestion dynamiques qui font le suivi des données du Magasin des requêtes et des statistiques d’attente affichent des résultats uniquement pour les requêtes terminées avec succès et les requêtes ayant expiré. Elles n’affichent pas de données ayant trait aux instructions en cours d’exécution tant que ces instructions ne sont pas terminées. Utilisez la vue de gestion dynamique [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) pour suivre les requêtes en cours d’exécution et la durée du worker associé.

> [!TIP]
> Outils supplémentaires :
>
> - [Attentes et verrous TigerToolbox](https://github.com/Microsoft/tigertoolbox/tree/master/Waits-and-Latches)
> - [TigerToolbox usp_whatsup](https://github.com/Microsoft/tigertoolbox/tree/master/usp_WhatsUp)

## <a name="next-steps"></a>Étapes suivantes

[Vue d’ensemble de la supervision et du réglage de SQL Database](database/monitor-tune-overview.md)