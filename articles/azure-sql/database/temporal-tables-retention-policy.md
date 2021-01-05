---
title: Gérer les données historiques dans des tables temporelles
description: Découvrez comment utiliser la stratégie de rétention temporelle pour conserver les données d’historique sous votre contrôle.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: bonova
ms.author: bonova
ms.reviewer: sstein
ms.date: 09/25/2018
ms.openlocfilehash: 1d68163a9fba3ba3bcd4c0c0f3fb5f442296e781
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91619387"
---
# <a name="manage-historical-data-in-temporal-tables-with-retention-policy"></a>Gérer les données historiques dans ses tables temporelles avec la stratégie de rétention
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Par rapport aux tables normales, les tables temporelles peuvent augmenter la taille des bases de données, notamment si vous conservez les données historiques pendant plus longtemps. Par conséquent, une stratégie de rétention des données d’historique est un aspect important de la planification et de la gestion du cycle de vie de chaque table temporelle. Les tables temporelles dans Azure SQL Database et Azure SQL Managed Instance sont fournies avec un mécanisme de conservation des données destiné à vous aider à accomplir cette tâche.

La rétention d’historique temporelle peut être configurée au niveau de la table individuelle, ce qui permet aux utilisateurs de créer des stratégies de vieillissement flexibles. L’application de la rétention temporelle est simple : un seul paramètre doit être défini durant la création de la table ou le changement de schéma.

Une fois que vous avez défini la stratégie de rétention, Azure SQL Database et Azure SQL Managed Instance commencent à vérifier régulièrement s’il existe des lignes de l’historique éligibles pour le nettoyage automatique des données. L’identification des lignes correspondantes et leur suppression de la table d’historique se produisent de façon transparente, dans la tâche en arrière-plan planifiée et exécutée par le système. La condition d’âge des lignes de la table d’historique est vérifiée en fonction de la colonne représentant la fin de la période SYSTEM_TIME. Par exemple, si la période de rétention est définie à six mois, les lignes de table éligibles pour un nettoyage répondent à la condition suivante :

```sql
ValidTo < DATEADD (MONTH, -6, SYSUTCDATETIME())
```

Dans l’exemple précédent, nous avons supposé que la colonne **ValidTo** correspond à la fin de la période SYSTEM_TIME.

## <a name="how-to-configure-retention-policy"></a>Configuration d’une stratégie de rétention

Avant de configurer la stratégie de rétention d’une table temporelle, vérifiez si la rétention historique temporelle est activée *au niveau de la base de données*.

```sql
SELECT is_temporal_history_retention_enabled, name
FROM sys.databases
```

L’indicateur de base de données **is_temporal_history_retention_enabled** a la valeur ON par défaut, mais les utilisateurs peuvent le changer à l’aide de l’instruction ALTER DATABASE. Il est automatiquement défini sur Désactivé après une opération de [limite de restauration dans le temps](recovery-using-backups.md). Si vous souhaitez activer le nettoyage de la rétention d’historique temporelle pour votre base de données, exécutez l’instruction suivante :

```sql
ALTER DATABASE <myDB>
SET TEMPORAL_HISTORY_RETENTION  ON
```

> [!IMPORTANT]
> Vous pouvez configurer la durée de rétention pour les tables temporelles même si **is_temporal_history_retention_enabled** est DÉSACTIVÉ, mais le nettoyage automatique des lignes anciennes n’est pas déclenché dans ce cas.

La stratégie de rétention est configurée durant la création de la table quand vous spécifiez la valeur du paramètre HISTORY_RETENTION_PERIOD :

```sql
CREATE TABLE dbo.WebsiteUserInfo
(  
    [UserID] int NOT NULL PRIMARY KEY CLUSTERED
  , [UserName] nvarchar(100) NOT NULL
  , [PagesVisited] int NOT NULL
  , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
  , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
  , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
 )  
 WITH
 (
     SYSTEM_VERSIONING = ON
     (
        HISTORY_TABLE = dbo.WebsiteUserInfoHistory,
        HISTORY_RETENTION_PERIOD = 6 MONTHS
     )
 );
```

Azure SQL Database et Azure SQL Managed Instance permettent de spécifier la période de rétention à l’aide de différentes unités de temps : DAYS, WEEKS, MONTHS et YEARS. Si HISTORY_RETENTION_PERIOD est omis, la rétention est censée être INFINITE. Vous pouvez également utiliser explicitement le mot clé INFINITE.

Dans certains scénarios, vous pouvez configurer la rétention après la création de la table, ou pour changer une valeur configurée. Dans ce cas, utilisez l’instruction ALTER TABLE :

```sql
ALTER TABLE dbo.WebsiteUserInfo
SET (SYSTEM_VERSIONING = ON (HISTORY_RETENTION_PERIOD = 9 MONTHS));
```

> [!IMPORTANT]
> Le fait de définir SYSTEM_VERSIONING sur Désactivé *ne préserve pas* la valeur de période de rétention. La définition de SYSTEM_VERSIONING sur Activé sans spécifier HISTORY_RETENTION_PERIOD résulte de façon explicite en une période de rétention illimitée (INFINITE).

Pour passer en revue l’état actuel de la stratégie de rétention, utilisez la requête suivante qui permet de joindre l’indicateur d’activation de rétention temporelle au niveau de la base de données aux périodes de rétention de tables individuelles :

```sql
SELECT DB.is_temporal_history_retention_enabled,
SCHEMA_NAME(T1.schema_id) AS TemporalTableSchema,
T1.name as TemporalTableName,  SCHEMA_NAME(T2.schema_id) AS HistoryTableSchema,
T2.name as HistoryTableName,T1.history_retention_period,
T1.history_retention_period_unit_desc
FROM sys.tables T1  
OUTER APPLY (select is_temporal_history_retention_enabled from sys.databases
where name = DB_NAME()) AS DB
LEFT JOIN sys.tables T2
ON T1.history_table_id = T2.object_id WHERE T1.temporal_type = 2
```

## <a name="how-ages-rows-are-deleted"></a>Suppression des lignes des âges

Le processus de nettoyage dépend de la disposition de l’index de la table d’historique. Il est important de noter que *seules les tables d’historique avec un index cluster (arbre B (B-tree) ou columnstore) peuvent avoir une stratégie de rétention limitée et configurée*. Une tâche en arrière-plan est créée afin d’effectuer le nettoyage des anciennes données pour toutes les tables temporelles dont la période de rétention est limitée.
La logique de nettoyage de l’index cluster rowstore (arbre B (B-tree)) supprime les anciennes lignes dans des blocs plus petits (jusqu’à 10 K), ce qui permet de réduire la pression sur le journal de la base de données et le sous-système d’E/S. Bien que la logique de nettoyage utilise l’index d’arbre B (B-tree) nécessaire, l’ordre des suppressions des lignes antérieures à la période de rétention ne peut pas être garanti. Vous ne devez donc *pas accepter de dépendances relatives à l’ordre de nettoyage dans vos applications*.

La tâche de nettoyage pour le cluster columnstore supprime l’ensemble [des groupes de lignes](/sql/relational-databases/indexes/columnstore-indexes-overview) en une fois (ceux-ci contiennent généralement 1 million de lignes chacun), ce qui est très efficace, en particulier lorsque les données d’historique sont générées à un rythme élevé.

![Rétention de columnstore en cluster](./media/temporal-tables-retention-policy/cciretention.png)

En raison de l’excellence de la compression des données et de l’efficacité du nettoyage de la rétention, l’index columnstore cluster est un choix idéal dans les scénarios où votre charge de travail génère rapidement une grande quantité de données d’historique. Ce modèle est généralement utilisé pour les [charges de travail de traitement transactionnel intensives qui utilisent des tables temporelles](/sql/relational-databases/tables/temporal-table-usage-scenarios) à des fins de suivi des modifications et l’audit, d’analyse de tendances ou d’ingestion des données IoT.

## <a name="index-considerations"></a>Considérations relatives aux index

La tâche de nettoyage des tables avec un index cluster rowstore requiert que l’index démarre avec la colonne correspondant à la fin de la période SYSTEM_TIME. Si cet index n’existe pas, vous ne pouvez pas configurer de période de rétention limitée :

*Msg 13765, niveau 16, état 1 <br></br> Échec de la définition d’une période de rétention limitée sur la table temporelle avec version gérée par le système « temporalstagetestdb.dbo.WebsiteUserInfo », car la table d’historique 'temporalstagetestdb.dbo.WebsiteUserInfoHistory » ne contient pas l’index cluster nécessaire. Vous devez créer un index cluster columnstore ou d’arbre B dans lequel la première colonne correspond à la fin de la période SYSTEM_TIME, sur la table d'historique.*

Il est important de noter que la table de l’historique par défaut déjà créée par Azure SQL Database et Azure SQL Managed Instance a un index cluster, ce qui est compatible avec la stratégie de rétention. Si vous essayez de supprimer cet index sur une table avec une période de rétention limitée, l’opération échoue avec l’erreur suivante :

*Msg 13766, niveau 16, état 1 <br></br> Impossible de supprimer l’index cluster « WebsiteUserInfoHistory.IX_WebsiteUserInfoHistory », car il est utilisé pour le nettoyage automatique des données anciennes. Pour supprimer cet index, attribuez la valeur INFINITE à HISTORY_RETENTION_PERIOD sur la table temporelle avec version gérée par le système correspondante.*

Le nettoyage de l’index cluster columnstore fonctionne de façon optimale si des lignes d’historique sont insérées dans l’ordre croissant (par la fin de la colonne de période), ce qui est toujours le cas lorsque la table d’historique est remplie exclusivement par le mécanisme SYSTEM_VERSIONIOING. Si les lignes de la table d’historique ne sont pas ordonnées par la fin de la colonne de période (ce qui peut arriver si vous avez migré des données d’historique existantes), vous devez recréer les index cluster columnstore sur les index d’arbre B rowstore correctement triés, pour optimiser les performances.

Évitez de reconstruire l’index cluster columnstore sur la table d’historique avec la période de rétention limitée, car cela risque de modifier l’ordre des groupes de lignes naturellement imposé par l’opération de système de version. Si vous devez reconstruire l’index cluster columnstore sur la table d’historique, faites-le en le recréant par-dessus l’index d’arbre B conforme, en conservant l’ordre des groupes de lignes nécessaire au nettoyage de données standard. La même approche convient si vous créez la table temporelle avec une table d’historique existante qui a un index de colonne cluster sans ordre de données garanti :

```sql
/*Create B-tree ordered by the end of period column*/
CREATE CLUSTERED INDEX IX_WebsiteUserInfoHistory ON WebsiteUserInfoHistory (ValidTo)
WITH (DROP_EXISTING = ON);
GO
/*Re-create clustered columnstore index*/
CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory ON WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
```

Lorsque la période de rétention limitée est configurée pour la table d’historique avec l’index cluster columnstore, vous ne pouvez pas créer d’index d’arbre B supplémentaire non cluster sur cette table :

```sql
CREATE NONCLUSTERED INDEX IX_WebHistNCI ON WebsiteUserInfoHistory ([UserName])
```

Une tentative d’exécution de l’instruction ci-dessus échoue avec l’erreur suivante :

*Msg 13772, niveau 16, état 1 <br></br> Impossible de créer un index non cluster sur la table d’historique temporelle « WebsiteUserInfoHistory » car celle-ci est définie avec une période de rétention limitée et un index cluster columnstore.*

## <a name="querying-tables-with-retention-policy"></a>Interrogation de tables comportant une stratégie de rétention

Toutes les requêtes effectuées sur une table temporelle filtrent automatiquement les lignes d’historique correspondant à la stratégie de rétention limitée, afin d’éviter des résultats imprévisibles et incohérents, étant donné que les anciennes lignes peuvent être supprimées par la tâche de nettoyage, *à n’importe quel point dans le temps et dans un ordre arbitraire*.

L’illustration suivante montre le plan de requête pour une requête simple :

```sql
SELECT * FROM dbo.WebsiteUserInfo FOR SYSTEM_TIME ALL;
```

Le plan de requête inclut un filtre supplémentaire appliqué à la fin de la colonne de période (ValidTo) dans l’opérateur Analyse d’index cluster sur la table d’historique (mise en surbrillance). Cet exemple suppose que la période de rétention d’un MOIS a été définie sur la table WebsiteUserInfo.

![Filtre de requête de rétention](./media/temporal-tables-retention-policy/queryexecplanwithretention.png)

Cependant, si vous interrogez directement la table d’historique, vous pouvez voir des lignes antérieures à la période de rétention spécifiée, mais sans aucune garantie de bénéficier de résultats de requête reproductibles. L’illustration suivante montre le plan d’exécution de la requête sur la table d’historique sans appliquer de filtres supplémentaires :

![Interrogation de l’historique sans filtre de rétention](./media/temporal-tables-retention-policy/queryexecplanhistorytable.png)

Ne faites pas dépendre votre logique métier sur la lecture de la table d’historique au-delà de la période de rétention, car vous risquez d’obtenir des résultats incohérents ou inattendus. Nous vous recommandons d’utiliser des requêtes temporelles avec la clause FOR SYSTEM_TIME pour l’analyse des données dans les tables temporelles.

## <a name="point-in-time-restore-considerations"></a>Considérations relatives à la limite de restauration dans le temps

Lorsque vous créez une base de données en [restaurant une base de données existante à un point spécifique dans le temps](recovery-using-backups.md), sa rétention temporelle est désactivée au niveau de la base de données. L’indicateur **is_temporal_history_retention_enabled** est défini sur Désactivé. Cette fonctionnalité vous permet d’examiner toutes les lignes d’historique lors de la restauration, sans craindre que les anciennes lignes soient supprimées avant qu’elles aient été interrogées. Vous pouvez l’utiliser pour *inspecter les données d’historique au-delà de la période de rétention configurée*.

Supposons qu’une table temporelle a une période de rétention de un MOIS. Si votre base de données a été créée dans le niveau de service Premium, vous ne pouvez pas créer de copie de base de données avec l’état de la base de données jusqu’à 35 jours dans le passé. Cela vous permet d’analyser avec efficacité les lignes d’historique qui sont anciennes de moins de 65 jours en interrogeant la table d’historique directement.

Si vous souhaitez activer le nettoyage de la rétention temporelle, exécutez l’instruction Transact-SQL suivante après le point de restauration dans le temps :

```sql
ALTER DATABASE <myDB>
SET TEMPORAL_HISTORY_RETENTION  ON
```

## <a name="next-steps"></a>Étapes suivantes

Pour apprendre à utiliser les tables temporelles dans vos applications, consultez [Prise en main des tables temporelles](../temporal-tables.md).

Visitez Channel 9 pour entendre le [témoignage d’un client sur l’implémentation temporelle](https://channel9.msdn.com/Blogs/jsturtevant/Azure-SQL-Temporal-Tables-with-RockStep-Solutions) et regardez une [démonstration temporelle en direct](https://channel9.msdn.com/Shows/Data-Exposed/Temporal-in-SQL-Server-2016).

Pour plus d’informations sur les tables temporelles, consultez la rubrique [Tables temporelles](/sql/relational-databases/tables/temporal-tables).
