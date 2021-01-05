---
title: Migration de base de données SQL Server vers Azure SQL Database
description: Découvrez la migration de base de données SQL Server vers Azure SQL Database.
keywords: migration de base de données, migration de base de données sql server, outils de migration de base de données, migrer la base de données, migrer la base de données sql
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/11/2019
ms.openlocfilehash: fab52fcea03a2f65c868cfac27f8a8cef115b2be
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917645"
---
# <a name="sql-server-database-migration-to-azure-sql-database"></a>Migration de base de données SQL Server vers Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit les méthodes principales de migration d’une base de données SQL Server 2005 ou version ultérieure vers Azure SQL Database. Pour plus d’informations sur la migration vers Azure SQL Managed Instance, consultez [Migration d’une instance SQL Server vers Azure SQL Managed Instance](../migration-guides/managed-instance/sql-server-to-managed-instance-overview.md). Pour obtenir des conseils sur le choix des options et des outils de migration vers Azure SQL, consultez [Migrer vers Azure SQL](../migration-guides/index.yml).


## <a name="migrate-to-a-single-database-or-a-pooled-database"></a>Migrer vers une base de données unique ou une base de données mise en pool

Deux méthodes principales permettent de migrer une base de données SQL Server 2005 ou version ultérieure vers Azure SQL Database. La première méthode est plus simple, mais elle implique un temps d’arrêt potentiellement important pendant la migration. La seconde méthode est plus complexe, mais elle élimine en grande partie les temps d’arrêt lors de la migration.

Dans les deux cas, vous devez vérifier que la base de données source est compatible avec Azure SQL Database à l’aide de [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595). SQL Database est proche de la [parité des fonctionnalités](features-comparison.md) avec SQL Server, à l’exception des problèmes liés aux opérations au niveau du serveur et sur plusieurs bases de données. Les bases de données et les applications qui reposent sur des [fonctions partiellement ou pas du tout prises en charge](transact-sql-tsql-differences-sql-server.md) ont besoin d’une [nouvelle ingénierie pour corriger ces incompatibilités](migrate-to-database-from-sql-server.md#resolving-database-migration-compatibility-issues) avant de pouvoir migrer la base de données SQL Server.

> [!NOTE]
> Pour migrer une base de données non-SQL Server, notamment Microsoft Access, Sybase, MySQL Oracle et DB2, vers Azure SQL Database, consultez l’ [Assistant Migration SQL Server](https://blogs.msdn.microsoft.com/datamigration/2017/09/29/release-sql-server-migration-assistant-ssma-v7-6/).

## <a name="method-1-migration-with-downtime-during-the-migration"></a>Méthode 1 : Migration avec un temps d’arrêt pendant l’opération

 Utilisez cette méthode pour migrer une base de données unique ou en mise pool si vous pouvez vous permettre un temps d’arrêt ou si vous effectuez un test de migration d’une base de données de production que vous envisagez de migrer. Pour un didacticiel, consultez [Migrer une base de données SQL Server](../../dms/tutorial-sql-server-to-azure-sql.md).

La liste suivante contient le workflow général pour la migration d’une base de données SQL Server unique ou mise en pool à l’aide de cette méthode. Pour la migration vers SQL Managed Instance, consultez [Migration vers SQL Managed Instance](../managed-instance/migrate-to-instance-from-sql-server.md).

  ![Schéma de migration VSSSDT](./media/migrate-to-database-from-sql-server/azure-sql-migration-sql-db.png)

1. [Évaluez](/sql/dma/dma-assesssqlonprem) la compatibilité de la base de données à l’aide de la dernière version de [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595).
2. Préparez les corrections nécessaires en tant que scripts Transact-SQL.
3. Faites une copie cohérente de la base de données source en cours de migration ou interrompez les nouvelles transactions qui se produisent dans la base de données source pendant la migration. Les méthodes qui existent pour accomplir cette dernière option incluent la désactivation de la connectivité des clients ou la création d’un [instantané de base de données](/sql/relational-databases/databases/create-a-database-snapshot-transact-sql). Après la migration, vous pourrez peut-être utiliser la réplication transactionnelle pour mettre à jour les bases de données migrées avec les modifications qui se produisent après le point d’arrêt pour la migration. Consultez [Migration à l’aide de la migration transactionnelle](migrate-to-database-from-sql-server.md#method-2-use-transactional-replication).  
4. Déployez les scripts Transact-SQL pour appliquer les correctifs à la copie de base de données.
5. [Migrez](/sql/dma/dma-migrateonpremsql) la copie de base de données vers une nouvelle base de données dans Azure SQL Database à l’aide de l’Assistant Migration de données.

> [!NOTE]
> Au lieu d’utiliser DMA, vous pouvez également utiliser un fichier BACPAC. Consultez [Importer un fichier BACPAC vers une nouvelle base de données dans Azure SQL Database](database-import.md).

### <a name="optimizing-data-transfer-performance-during-migration"></a>Optimisation des performances de transfert de données pendant la migration

La liste suivante contient des recommandations pour optimiser les performances pendant le processus d’importation.

- Choisissez le niveau de service et la taille de calcul les plus élevés dans la limite de votre budget, afin d’optimiser les performances de transfert. Vous pourrez descendre en puissance une fois la migration terminée pour économiser de l’argent.
- Réduisez la distance entre votre fichier BACPAC et le centre de données de destination.
- Désactivez les statistiques automatiques pendant la migration.
- Tables et index de partition
- Supprimez les vues indexées et recréez-les une fois la migration terminée.
- Déplacez les données historiques rarement interrogées vers une autre base de données et migrez ces données historiques vers un base de données distincte dans Azure SQL Database. Vous pourrez ensuite interroger ces données historiques à l’aide de [requêtes élastiques](elastic-query-overview.md).

### <a name="optimize-performance-after-the-migration-completes"></a>Optimiser les performances une fois la migration terminée

[Mettez à jour les statistiques](/sql/t-sql/statements/update-statistics-transact-sql) avec une analyse complète une fois la migration terminée.

## <a name="method-2-use-transactional-replication"></a>Méthode 2 : Utiliser la réplication transactionnelle

Quand vous ne pouvez pas vous permettre de supprimer votre base de données SQL Server de la production durant la migration, vous pouvez utiliser la réplication transactionnelle SQL Server comme solution de migration. Pour que vous puissiez utiliser cette méthode, la base de données source doit remplir les [conditions requises pour la réplication transactionnelle](./replication-to-sql-database.md) et être compatible avec Azure SQL Database. Pour plus d’informations sur la réplication SQL avec Always On, consultez [Configurer la réplication pour les groupes de disponibilité Always On (SQL Server)](/sql/database-engine/availability-groups/windows/configure-replication-for-always-on-availability-groups-sql-server).

Pour utiliser cette solution, vous devez configurer votre base de données dans Azure SQL Database en tant qu’abonné à l’instance SQL Server que vous souhaitez migrer. Le distributeur de réplication transactionnelle synchronise les données nécessaires de la base de données (l’éditeur), de nouvelles transactions continuant d’avoir lieu.

Avec la réplication transactionnelle, toutes les modifications apportées à vos données ou à votre schéma apparaissent dans votre base de données dans Azure SQL Database. Une fois la synchronisation terminée, lorsque vous êtes prêt à migrer, modifiez la chaîne de connexion de vos applications de façon à ce qu’elle pointe vers votre base de données. Une fois que la réplication transactionnelle a vidé toutes les modifications restant sur votre base de données source et que toutes vos applications pointent vers Azure SQL Database, vous pouvez désinstaller la réplication transactionnelle. Votre base de données dans Azure SQL Database est maintenant votre système de production.

 ![Diagramme SeedCloudTR](./media/migrate-to-database-from-sql-server/SeedCloudTR.png)

> [!TIP]
> Vous pouvez également utiliser la réplication transactionnelle pour migrer un sous-ensemble de votre base de données source. La publication que vous répliquez sur Azure SQL Database peut être limitée à un sous-ensemble des tables dans la base de données en cours de réplication. Pour chaque table répliquée, vous pouvez limiter les données à un sous-ensemble de lignes et/ou un sous-ensemble de colonnes.

## <a name="migration-to-sql-database-using-transaction-replication-workflow"></a>Workflow de migration vers SQL Database à l’aide de la réplication transactionnelle

> [!IMPORTANT]
> Utilisez la dernière version de SQL Server Management Studio pour rester synchronisé avec les mises à jour d’Azure et de SQL Database. Les versions antérieures de SQL Server Management Studio ne peuvent pas configurer Base de données SQL en tant qu’abonné. [Mettre à jour SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

1. Configurer la distribution
   - [Avec SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Avec Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)

2. Créer une publication
   - [Avec SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Avec Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)
3. Créer un abonnement
   - [Avec SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/create-a-push-subscription/)
   - [Avec Transact-SQL](/sql/relational-databases/replication/create-a-push-subscription/)

Quelques conseils et différences pour la migration vers SQL Database

- Utilisez un serveur de distribution local
  - Cela a un impact sur les performances du serveur.
  - Si l’impact sur les performances est inacceptable, vous pouvez utiliser un autre serveur, mais cela contribue à compliquer la gestion et l’administration.
- Lorsque vous sélectionnez un dossier d’instantanés, assurez-vous qu’il est suffisamment grand pour contenir un BCP de chaque table que vous souhaitez répliquer.
- La création d’instantanés verrouille les tables associées jusqu’à la fin de l’opération. Par conséquent, prenez soin de bien planifier votre instantané.
- Seuls les abonnements par push sont pris en charge dans Azure SQL Database. Vous pouvez uniquement ajouter des abonnés à partir de la base de données source.

## <a name="resolving-database-migration-compatibility-issues"></a>Résolution des problèmes de compatibilité de migration de la base de données

Vous pouvez rencontrer une grande variété de problèmes de compatibilité, selon la version de SQL Server dans la base de données source et la complexité de la base de données que vous êtes en train de migrer. Les versions antérieures de SQL Server ont plus de problèmes de compatibilité. Utilisez les ressources suivantes en plus d’une recherche Internet ciblée dans le moteur de recherche de votre choix :

- [Fonctionnalités de base de données SQL Server non prises en charge dans Azure SQL Database](transact-sql-tsql-differences-sql-server.md)
- [Discontinued Database Engine Functionality in SQL Server 2016 (Fonctionnalités du moteur de base de données supprimées dans SQL Server 2016)](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server)
- [Discontinued Database Engine Functionality in SQL Server 2014 (Fonctionnalités du moteur de base de données non disponibles dans SQL Server 2014)](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server?viewFallbackFrom=sql-server-2014)
- [Discontinued Database Engine Functionality in SQL Server 2012 (Fonctionnalités du moteur de base de données non disponibles dans SQL Server 2012)](/previous-versions/sql/sql-server-2012/ms144262(v=sql.110))
- [Discontinued Database Engine Functionality in SQL Server 2008 R2 (Fonctionnalités du moteur de base de données non disponibles dans SQL Server 2008 R2)](/previous-versions/sql/sql-server-2008-r2/ms144262(v=sql.105))
- [Discontinued Database Engine Functionality in SQL Server 2005 (Fonctionnalités du moteur de base de données supprimées dans SQL Server 2005)](/previous-versions/sql/sql-server-2005/ms144262(v=sql.90))

En plus des recherches sur Internet et de l’utilisation de ces ressources, utilisez la [page de questions et réponses Microsoft sur Azure SQL Database](/answers/topics/azure-sql-database.html) ou [StackOverflow](https://stackoverflow.com/).

> [!IMPORTANT]
> Azure SQL Managed Instance vous permet de migrer une instance existante et ses bases de données avec peu voire pas de problèmes de compatibilité. Consultez [Qu’est-ce qu’une instance managée ?](../managed-instance/sql-managed-instance-paas-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Utilisez le script fourni sur le blog Azure SQL EMEA Engineers pour [surveiller l’utilisation de tempdb pendant la migration](/archive/blogs/azuresqlemea/lesson-learned-10-monitoring-tempdb-usage).
- Utilisez le script fourni sur le blog Azure SQL EMEA Engineers pour [surveiller l’espace réservé au journal des transactions de votre base de données pendant la migration](/archive/blogs/azuresqlemea/lesson-learned-7-monitoring-the-transaction-log-space-of-my-database).
- Pour consulter le billet du blog SQL Server Customer Advisory Team sur la migration de SQL Server vers Azure SQL Database à l’aide de fichiers BACPAC (en anglais), rendez-vous [ici](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).
- Pour plus d’informations sur l’utilisation de l’heure UTC après la migration, consultez [Modifying the default time zone for your local time zone](/archive/blogs/azuresqlemea/lesson-learned-4-modifying-the-default-time-zone-for-your-local-time-zone) (Modification du fuseau horaire par défaut pour votre fuseau horaire local).
- Pour plus d’informations sur le changement de langue par défaut d’une base de données après la migration, consultez [Comment changer la langue par défaut d’Azure SQL Database](/archive/blogs/azuresqlemea/lesson-learned-16-how-to-change-the-default-language-of-azure-sql-database).
