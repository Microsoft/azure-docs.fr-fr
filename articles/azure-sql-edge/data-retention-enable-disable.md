---
title: Activer et désactiver des stratégies de rétention de données – Azure SQL Edge
description: En savoir plus sur l’activation et la désactivation des stratégies de rétention de données dans Azure SQL Edge
keywords: SQL Edge, rétention de données
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: 5b8dd911952a63ba8775f27a6128ff61e849e823
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861423"
---
# <a name="enable-and-disable-data-retention-policies"></a>Activer et désactiver des stratégies de rétention de données

Cette rubrique explique comment activer et désactiver des stratégies de rétention de données pour une base de données et une table. 

## <a name="enable-data-retention-for-a-database"></a>Activer la rétention de données pour une base de données

L’exemple suivant indique comment activer le suivi des modifications à l’aide de la commande [Alter Database](/sql/t-sql/statements/alter-database-transact-sql-set-options).

```sql
ALTER DATABASE [<DatabaseName>] SET DATA_RETENTION  ON;
```

## <a name="check-if-data-retention-is-enabled-for-a-database"></a>Vérifier si la rétention de données est activée pour une base de données

La commande suivante peut être utilisée pour vérifier si la rétention de données est activée pour une base de données
```sql
SELECT is_data_retention_enabled, name
FROM sys.databases;
```

## <a name="enable-data-retention-for-a-table"></a>Activer la rétention de données pour une table

La rétention de données doit être activée pour chaque table dont vous souhaitez que les données soient purgées automatiquement. Lorsque la rétention de données est activée sur la base de données et la table, une tâche système en arrière-plan analyse périodiquement la table pour identifier et supprimer toutes les lignes obsolètes (anciennes). La rétention de données peut être activée sur une table lors de la création de celle-ci à l’aide de la comment [Create Table](/sql/t-sql/statements/create-table-transact-sql) ou de la commande [Alter Table](/sql/t-sql/statements/alter-table-transact-sql).

L’exemple suivant montre comment activer la rétention de données pour une table à l’aide de la commande [Create Table](/sql/t-sql/statements/create-table-transact-sql). 

```sql
CREATE TABLE [dbo].[data_retention_table] 
(
[dbdatetime2] datetime2(7), 
[product_code] int, 
[value] char(10),  
CONSTRAINT [pk_current_data_retention_table] PRIMARY KEY CLUSTERED ([product_code])
) WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )
```

La partie `WITH (DATA_DELETION = ON ( FILTER_COLUMN = [dbdatetime2], RETENTION_PERIOD = 1 day ) )` de la commande Create Table définit la rétention de données sur la table. La commande utilise les paramètres obligatoires suivants 

- DATA_DELETION : indique si la rétention de données est activée ou désactivée.
- FILTER_COLUMN : nom sur la colonne de la table, qui sera utilisé pour déterminer si les lignes sont obsolètes ou non. La colonne de filtre ne peut être qu’une colonne avec les types de données suivants 
    - Date
    - SmallDateTime
    - DateTime
    - DateTime2
    - DateTimeOffset
- RETENTION_PERIOD : valeur entière suivie d’un descripteur d’unité. Les unités autorisées sont DAY, DAYS, WEEK, WEEKS, MONTH, MONTHS, YEAR et YEARS.

L’exemple suivant montre comment activer la rétention de données pour une table en utilisant la commande [Alter Table](/sql/t-sql/statements/alter-table-transact-sql).  

```sql
Alter Table [dbo].[data_retention_table]
SET (DATA_DELETION = On (FILTER_COLUMN = [timestamp], RETENTION_PERIOD = 1 day))
```

## <a name="check-if-data-retention-is-enabled-for-a-table"></a>Vérifier si la rétention de données est activée pour une table

La commande suivante peut être utilisée pour vérifier les tables pour lesquelles la rétention de données est activée

```sql
select name, data_retention_period, data_retention_period_unit from sys.tables
```

Une valeur de data_retention_period = -1 and data_retention_period_unit définie sur INFINIE indique que la rétention de données n’est pas définie sur la table.

La requête suivante permet d’identifier la colonne utilisée en tant que filter_column pour la rétention de données. 

```sql
Select name from sys.columns
where is_data_deletion_filter_column =1 
and object_id = object_id(N'dbo.data_retention_table', N'U')
```

## <a name="correlating-db-and-table-data-retention-settings"></a>Corrélation des paramètres de conservation des données de bases de données et de tables

Les paramètres de rétention de données définis sur la base de données et la table sont utilisé conjointement pour déterminer si le nettoyage automatique des lignes anciennes s’exécutera ou non sur les tables. 

|Option de base de données | Option de table | Comportement |
|----------------|--------------|----------|
| OFF | OFF | La stratégie de rétention de données est désactivée, ainsi que le nettoyage automatique et manuel des enregistrements anciens.|
| OFF | ACTIVÉ  | La stratégie de rétention de données est activée pour la table. Le nettoyage automatique des enregistrements obsolètes est désactivé mais vous pouvez utiliser une méthode manuelle pour nettoyer les enregistrements obsolètes. |
| ACTIVÉ | OFF | La stratégie de rétention de données est activée au niveau de la base de données. Toutefois, étant donné que l’option est désactivée au niveau de la table, il n’existe aucun nettoyage de lignes anciennes basé sur la rétention.|
| ACTIVÉ | ACTIVÉ | La stratégie de rétention de données est activée pour la base de données et les tables. Le nettoyage automatique des enregistrements obsolètes est activé. |

## <a name="disable-data-retention-on-a-table"></a>Désactiver la rétention de données sur une table 

La rétention de données peut être désactivée sur une table à l’aide de la commande [Alter Table](/sql/t-sql/statements/alter-table-transact-sql). La commande suivante peut être utilisée pour désactiver la rétention de données sur une table.

```sql
Alter Table [dbo].[data_retention_table]
Set (DATA_DELETION = OFF)
```

## <a name="disable-data-retention-on-a-database"></a>Désactiver la rétention de données sur une base de données

La rétention de données peut être désactivée sur une table à l’aide de la commande [Alter Database](/sql/t-sql/statements/alter-database-transact-sql-set-options). La commande suivante permet de désactiver la rétention de données sur une base de données.

```sql
ALTER DATABASE <DatabaseName> SET DATA_RETENTION  OFF;
```

## <a name="next-steps"></a>Étapes suivantes
- [Rétention de données et purge automatique des données](data-retention-overview.md)
- [Gérer les données historiques avec une stratégie de rétention](data-retention-cleanup.md)
