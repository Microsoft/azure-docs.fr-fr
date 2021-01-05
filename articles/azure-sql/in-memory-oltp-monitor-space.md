---
title: Effectuer le monitorage du stockage en mémoire XTP
description: Estimer et surveiller la capacité et l’utilisation du stockage en mémoire XTP ; résoudre l’erreur de capacité 41823
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 01/25/2019
ms.openlocfilehash: 5717c2479c1d894117bae44826a814c3cfeaa98a
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493387"
---
# <a name="monitor-in-memory-oltp-storage-in-azure-sql-database-and-azure-sql-managed-instance"></a>Surveiller le stockage OLTP en mémoire dans Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Lorsque vous utilisez [OLTP en mémoire](in-memory-oltp-overview.md), les données des tables à mémoire optimisée et les variables de table résident dans un stockage OLTP en mémoire.

## <a name="determine-whether-data-fits-within-the-in-memory-oltp-storage-cap"></a>Déterminer si la taille des données est adaptée à la capacité de stockage en mémoire OLTP

Déterminez les limites maximales de stockage des différents niveaux de service. Chaque niveau de service Premium et critique pour l'entreprise dispose d’une taille de stockage OLTP en mémoire maximale.

- [Limites de ressources basées sur des unités DTU - Base de données unique](database/resource-limits-dtu-single-databases.md)
- [Limites de ressources basées sur des unités DTU - Pools élastiques](database/resource-limits-dtu-elastic-pools.md)
- [Limites de modèle de ressources basées sur des vCore : base de données unique](database/resource-limits-vcore-single-databases.md)
- [Limites de ressources basées sur des vCore : pools élastiques](database/resource-limits-vcore-elastic-pools.md)
- [Limites de ressources basées sur des vCore - instance managée](managed-instance/resource-limits.md)

L’estimation de la mémoire requise pour une table à mémoire optimisée s’effectue de la même façon pour SQL Server que dans Azure SQL Database et Azure SQL Managed Instance. Prenez quelques minutes pour passer en revue la rubrique [Estimer les besoins en mémoire](/sql/relational-databases/in-memory-oltp/estimate-memory-requirements-for-memory-optimized-tables).

La table et les lignes de variable de table, ainsi que les index, sont pris en compte pour le calcul de la taille maximale des données utilisateur. En outre, l’instruction ALTER TABLE a besoin de suffisamment d’espace pour créer une version de la table entière et de ses index.

Une fois cette limite dépassée, les opérations d’insertion et de mise à jour peuvent échouer avec l’erreur 41823 pour les bases de données uniques dans Azure SQL Database et les bases de données dans Azure SQL Managed Instance et avec l’erreur 41840 pour les pools élastiques dans Azure SQL Database. À ce stade, vous devez soit supprimer des données pour libérer de la mémoire, soit mettre à niveau le niveau de service ou la taille de calcul de votre base de données.

## <a name="monitoring-and-alerting"></a>Surveillance et alerte

Vous pouvez surveiller l’utilisation du stockage en mémoire (représentée sous forme de pourcentage de la limite maximale de stockage) de votre taille de calcul dans le [Portail Azure](https://portal.azure.com/) :

1. Sur le panneau Base de données, recherchez la zone Utilisation de ressources, puis cliquez sur Modifier.
2. Sélectionnez la métrique `In-Memory OLTP Storage percentage`.
3. Pour ajouter une alerte, cliquez sur la zone Taux d’utilisation de la ressource pour ouvrir le panneau Métriques, puis cliquez sur Ajouter une alerte.

Vous pouvez également utiliser la requête suivante pour afficher l’utilisation du stockage en mémoire :

```sql
    SELECT xtp_storage_percent FROM sys.dm_db_resource_stats
```

## <a name="correct-out-of-in-memory-oltp-storage-situations---errors-41823-and-41840"></a>Corrigez les situations de stockage OLTP en mémoire insuffisant : les erreurs 41823 et 41840

Lorsque le plafond de stockage OLTP en mémoire est atteint, les opérations de base de données INSERT, UPDATE, ALTER et CREATE échouent avec le message d’erreur 41823 (pour les bases de données uniques) ou 41840 (pour les pools élastiques). Les deux erreurs provoquent l’abandon de la transaction active.

Les messages d’erreur 41823 et 41840 indiquent que les tables optimisées en mémoire et les variables de table dans la base de données ou le pool ont atteint la taille de stockage OLTP en mémoire maximale.

Pour résoudre cette erreur, deux possibilités s’offrent à vous :

- supprimer des données des tables à mémoire optimisée, en déchargeant potentiellement les données vers des tables traditionnelles sur disque ;
- adapter le niveau de service afin de disposer d’un stockage en mémoire suffisant pour les données que vous devez conserver dans des tables à mémoire optimisée.

> [!NOTE]
> Dans de rares cas, les erreurs 41823 et 41840 peuvent être temporaires, ce qui signifie qu’il y a suffisamment de stockage OLTP en mémoire disponible, et que l’opération réussit quand elle est relancée. Par conséquent, nous vous recommandons de surveiller le stockage OLTP en mémoire total disponible et de commencer par recommencer l’opération lorsque vous rencontrez des erreurs 41823 ou 41840. Pour plus d’informations sur la logique de nouvelle tentative, consultez [Détection de conflit et logique de nouvelle tentative avec l’OLTP en mémoire](/sql/relational-databases/In-memory-oltp/transactions-with-memory-optimized-tables#conflict-detection-and-retry-logic).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’instructions de surveillance, consultez [Surveillance à l’aide de vues de gestion dynamique](database/monitoring-with-dmvs.md).