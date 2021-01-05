---
title: 'SQL Server vers SQL Managed Instance : Analyse des performances'
description: Apprenez à créer et à comparer une ligne de base des performances lors de la migration de vos bases de données SQL Server vers Azure SQL Managed Instance.
ms.service: sql-database
ms.subservice: migration
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: mokabiru
ms.date: 11/06/2020
ms.openlocfilehash: 2b6f9bef445cc07a3aa00377b41010a536855935
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358858"
---
# <a name="migration-performance-sql-server-to-sql-managed-instance-performance-analysis"></a>Performances en matière de migration : Analyse des performances de SQL Server vers SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

Créez une ligne de base des performances pour comparer les performances de votre charge de travail s’exécutant sur SQL Managed Instance avec votre charge de travail d’origine s’exécutant sur SQL Server. 

## <a name="create-a-baseline"></a>Créer une ligne de base

Dans l’idéal, les performances sont similaires ou meilleures après la migration. Il est donc important de mesurer et d’enregistrer les valeurs de performances de la ligne de base sur la source, puis de les comparer à l’environnement cible. Une ligne de base des performances est un ensemble de paramètres qui définissent votre charge de travail moyenne sur votre source. 

Sélectionnez un ensemble de requêtes qui sont importantes pour la charge de travail de votre entreprise et représentatives de celle-ci. Mesurez et documentez la durée minimale/moyenne/maximale et l’utilisation de l’UC pour ces requêtes, ainsi que les métriques de performances sur le serveur source, telles que l’utilisation moyenne/maximale de l’UC, la latence d’E/S de disque moyenne/maximum, le débit, l’IOPS, l’espérance de vie de page moyenne/maximale et la taille moyenne/maximale de tempdb. 

Les ressources suivantes peuvent vous aider à définir une ligne de base des performances : 

   - [Analysez l’utilisation de l’UC](https://techcommunity.microsoft.com/t5/azure-sql-database/monitor-cpu-usage-on-sql-server-and-azure-sql/ba-p/680777#M131).
   - [Analysez l’utilisation de la mémoire](/sql/relational-databases/performance-monitor/monitor-memory-usage) et déterminez la quantité de mémoire utilisée par différents composants, tels que le pool de mémoires tampons, le cache du plan, le pool de stockage en colonne, l’ [OLTP en mémoire](/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage), etc. Par ailleurs, vous devez rechercher les valeurs moyennes et maximales du compteur de performance de la mémoire Espérance de vie d’une page. 
   - Analysez l’utilisation des E/S disque sur l’instance SQL source à l’aide de la vue  [sys.dm_io_virtual_file_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql) ou des  [compteurs de performance](/sql/relational-databases/performance-monitor/monitor-disk-usage). 
   - Analysez les performances des requêtes et des charges de travail en examinant les vues de gestion dynamique (ou Magasin des requêtes si vous effectuez une migration à partir de SQL Server 2016 et versions ultérieures). Identifiez la durée moyenne et l’utilisation de l’UC des requêtes les plus importantes dans votre charge de travail. 

Tout problème de performances sur le serveur SQL source doit être résolu avant la migration. Migrer des problèmes connus sur un nouveau système peut provoquer des résultats inattendus et invalider toute comparaison de performance. 


## <a name="compare-performance"></a>Comparer les performances 

Une fois que vous avez défini une ligne de base, comparez les performances de charges de travail similaires sur l’instance SQL Managed Instance cible. Pour des raisons de précision, il est important que l’environnement SQL Managed Instance soit comparable à l’environnement SQL Server autant que possible. 

Il existe des différences dans l’infrastructure SQL Managed Instance qui rendent improbable la correspondance exacte des performances. Certaines requêtes peuvent s’exécuter plus rapidement que prévu, tandis que d’autres peuvent être plus lentes. L’objectif de cette comparaison est de vérifier que les performances de charge de travail dans l’instance gérée correspondent à celles observées sur SQL Server (en moyenne) et d’identifier les requêtes critiques dont les performances sont éloignées de vos performances d’origine. 

La comparaison des performances est susceptible d’entraîner les résultats suivants : 

- Les performances de charge de travail sur l’instance gérée s’alignent sur celles enregistrées sur votre serveur SQL source ou leur sont supérieures. En l’occurrence, vous confirmez de façon que la migration a réussi. 

- La majorité des paramètres de performance et les requêtes dans la charge de travail fonctionnent comme prévu, avec quelques exceptions entraînant une dégradation des performances. Dans ce cas, identifiez les écarts et leur degré d’importance. S’il existe quelques requêtes importantes accusant une détérioration des performances, cherchez à savoir si les plans SQL sous-jacents ont été modifiés ou si les requêtes ont atteint les limites des ressources. Vous pouvez atténuer ce problème en appliquant quelques conseils aux requêtes critiques (par exemple, modifier le niveau de compatibilité, l’estimateur de cardinalité héritée) soit directement, soit à l’aide des repères de plan. Vérifiez que les statistiques et les index sont à jour et équivalents dans les deux environnements. 

- La plupart des requêtes sont plus lentes sur l’instance gérée par rapport à votre instance SQL source. Dans ce cas, essayez d’identifier les causes racines de cette différence, comme l’ [atteinte de certaines limites de ressources](../../managed-instance/resource-limits.md#service-tier-characteristics) : limites des E/S, de mémoire ou du débit du journal d’instance par exemple. Si aucune limite de ressources n’est à l’origine de cette différence, essayez de changer le niveau de compatibilité de la base de données ou de changer des paramètres de base de données, comme l’estimation de la cardinalité existante, puis réexécutez le test. Examinez les recommandations fournies par les vues de l’instance gérée ou de Magasin des requêtes pour identifier les requêtes dont les performances ont diminué. 

SQL Managed Instance dispose d’une fonction intégrée de correction automatique du plan qui est activée par défaut. Cette fonctionnalité garantit que les requêtes qui fonctionnaient correctement par le passé ne se dégraderont pas à l’avenir. Si cette fonctionnalité n’est pas activée, exécutez la charge de travail avec les anciens paramètres afin que SQL Managed Instance puisse connaître la ligne de base des performances. Ensuite, activez la fonctionnalité et réexécutez la charge de travail avec les nouveaux paramètres. 

Modifiez les paramètres de votre test ou effectuez une mise à niveau vers des niveaux de service supérieurs afin d’atteindre la configuration optimale pour les performances de charge de travail qui répondent à vos besoins. 

## <a name="monitor-performance"></a>Superviser les performances 

SQL Managed Instance fournit des outils avancés de surveillance et de résolution des problèmes, et vous devez les utiliser pour analyser les performances de votre instance. Voici quelques-unes des principales métriques pour analyser : 

- L’utilisation de l’UC sur l’instance pour déterminer si le nombre de vCores que vous avez approvisionnés convient pour votre charge de travail. 
- L’espérance de vie d’une page sur votre instance gérée pour déterminer si vous avez besoin de [mémoire supplémentaire](https://techcommunity.microsoft.com/t5/azure-sql-database/do-you-need-more-memory-on-azure-sql-managed-instance/ba-p/563444).
-  Des statistiques telles que INSTANCE_LOG_GOVERNOR  ou  PAGEIOLATCH qui identifient les problèmes d’E/S de stockage, plus particulièrement au niveau Usage général, où vous devrez peut-être préallouer des fichiers pour obtenir de meilleures performances d’E/S. 


## <a name="considerations"></a>Considérations  

Lorsque vous comparez les performances, tenez compte des points suivants : 

- Les paramètres correspondent entre la source et la cible. Vérifiez que les différents paramètres de l’instance, de la base de données et de tempdb sont équivalents entre les deux environnements. Les différences de configuration, de niveaux de compatibilité, de paramètres de chiffrement, d’indicateurs de trace, etc., peuvent toutes fausser les performances. 

- Le stockage est configuré conformément aux [meilleures pratiques](https://techcommunity.microsoft.com/t5/datacat/storage-performance-best-practices-and-considerations-for-azure/ba-p/305525). Par exemple, pour Usage général, vous devrez peut-être préallouer la taille des fichiers pour améliorer les performances. 

- Il existe des [différences d’environnement clés](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/) pouvant entraîner des écarts de performances entre l’instance gérée et SQL Server. Identifiez les risques pertinents pour votre environnement qui pourraient avoir une incidence sur les performances. 

- Le magasin des requêtes et le réglage automatique doivent être activés sur votre instance SQL Managed Instance, car ils vous aident à mesurer les performances de la charge de travail et à atténuer automatiquement les éventuels problèmes de performances. 



## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’optimisation de votre nouvel environnement Azure SQL Managed Instance, consultez les ressources suivantes : 

- [How to identify why workload performance on Azure SQL Managed Instance is different than SQL Server? (Comment identifier les raisons pour lesquelles les performances de la charge de travail sur Azure SQL Managed Instance diffèrent de celles sur SQL Server ?)](https://medium.com/azure-sqldb-managed-instance/what-to-do-when-azure-sql-managed-instance-is-slower-than-sql-server-dd39942aaadd)
- [Causes principales des différences de performances entre SQL Managed Instance et SQL Server](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/)
- [Meilleures pratiques et considérations relatives aux performances de stockage pour Azure SQL Managed Instance (Usage général)](https://techcommunity.microsoft.com/t5/datacat/storage-performance-best-practices-and-considerations-for-azure/ba-p/305525)
- [Analyse des performances en temps réel pour Azure SQL Managed Instance (this is archived, is this the intended target?)](/archive/blogs/sqlcat/real-time-performance-monitoring-for-azure-sql-database-managed-instance)