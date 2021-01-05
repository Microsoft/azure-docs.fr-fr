---
title: Automatiser la réplication des modifications de schéma dans SQL Data Sync
description: Découvrez comment automatiser la réplication des modifications de schéma dans Azure SQL Data Sync.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: data sync
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 11/14/2018
ms.openlocfilehash: f9997ea737b96185b31a7f51996a396fb5fc46c4
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790183"
---
# <a name="automate-the-replication-of-schema-changes-in-azure-sql-data-sync"></a>Automatiser la réplication des modifications de schéma dans Azure SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

SQL Data Sync permet aux utilisateurs de synchroniser des données entre des bases de données Azure SQL Database et des instances SQL Server, de façon unidirectionnelle ou bidirectionnelle. Une des limitations actuelles de SQL Data Sync est un manque de prise en charge pour la réplication des modifications de schéma. À chaque fois que vous modifiez le schéma de table, vous devez appliquer les modifications manuellement sur tous les points de terminaison, notamment le concentrateur et tous les membres, puis mettre à jour le schéma de synchronisation.

Cet article présente une solution pour répliquer automatiquement les modifications de schéma sur tous les points de terminaison SQL Data Sync.

1. Cette solution utilise un déclencheur DDL pour effectuer le suivi des modifications de schéma.
1. Le déclencheur insère les commandes de modification de schéma dans une table de suivi.
1. Cette table de suivi est synchronisée avec tous les points de terminaison à l’aide du service Data Sync.
1. Les déclencheurs DML après insertion sont utilisés pour appliquer les modifications de schéma aux autres points de terminaison.

Cet article utilise ALTER TABLE comme exemple d’une modification de schéma, mais cette solution fonctionne également pour d’autres types de modifications de schéma.

> [!IMPORTANT]
> Nous vous recommandons de lire cet article avec soin, notamment les sections sur la [Résolution des problèmes](#troubleshoot) et [Autres considérations](#other), avant de commencer à implémenter la réplication de modification de schéma automatisée dans votre environnement de synchronisation. Nous vous recommandons également de lire [Synchroniser des données sur plusieurs bases de données cloud et locales avec SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md). La solution décrite dans cet article peut ne pas fonctionner avec certaines opérations de base de données. Des connaissances supplémentaires sur le domaine de SQL Server et Transact-SQL peuvent être nécessaires pour résoudre ces problèmes.

![Automatisation de la réplication des modifications de schéma](./media/sql-data-sync-update-sync-schema/automate-schema-changes.png)

## <a name="set-up-automated-schema-change-replication"></a>Configurer la réplication de modification de schéma automatisée

### <a name="create-a-table-to-track-schema-changes"></a>Créer une table pour effectuer le suivi des modifications de schéma

Créez une table pour effectuer le suivi des modifications de schéma dans toutes les bases de données du groupe de synchronisation :

```sql
CREATE TABLE SchemaChanges (
ID bigint IDENTITY(1,1) PRIMARY KEY,
SqlStmt nvarchar(max),
[Description] nvarchar(max)
)
```

Cette table possède une colonne d’identité pour suivre l’ordre des modifications de schéma. Vous pouvez ajouter davantage de champs pour enregistrer davantage d’informations, si nécessaire.

### <a name="create-a-table-to-track-the-history-of-schema-changes"></a>Créer une table pour effectuer le suivi de l’historique des modifications de schéma

Sur tous les points de terminaison, créez une table pour effectuer le suivi de l’ID de la commande de modification de schéma appliquée en dernier.

```sql
CREATE TABLE SchemaChangeHistory (
LastAppliedId bigint PRIMARY KEY
)
GO

INSERT INTO SchemaChangeHistory VALUES (0)
```

### <a name="create-an-alter-table-ddl-trigger-in-the-database-where-schema-changes-are-made"></a>Créer un déclencheur DDL ALTER TABLE dans la base de données où sont apportées les modifications de schéma

Créez un déclencheur DDL pour les opérations ALTER TABLE. Il vous suffit de créer ce déclencheur dans la base de données où sont apportées les modifications de schéma. Pour éviter les conflits, autorisez uniquement les modifications de schéma dans une base de données dans un groupe de synchronisation.

```sql
CREATE TRIGGER AlterTableDDLTrigger
ON DATABASE
FOR ALTER_TABLE
AS

-- You can add your own logic to filter ALTER TABLE commands instead of replicating all of them.

IF NOT (EVENTDATA().value('(/EVENT_INSTANCE/SchemaName)[1]', 'nvarchar(512)') like 'DataSync')

INSERT INTO SchemaChanges (SqlStmt, Description)
    VALUES (EVENTDATA().value('(/EVENT_INSTANCE/TSQLCommand/CommandText)[1]', 'nvarchar(max)'), 'From DDL trigger')
```

Le déclencheur insère un enregistrement dans la table de suivi des modifications de schéma pour chaque commande ALTER TABLE. Cet exemple ajoute un filtre pour éviter la réplication des modifications de schéma apportées sous le schéma **DataSync** , car elles sont sans doute effectuées par le service Data Sync. Ajoutez d’autres filtres si vous souhaitez seulement répliquer certains types de modifications de schéma.

Vous pouvez également ajouter davantage de déclencheurs pour répliquer d’autres types de modifications de schéma. Par exemple, créez des déclencheurs CREATE_PROCEDURE, ALTER_PROCEDURE et DROP_PROCEDURE pour répliquer les modifications apportées à des procédures stockées.

### <a name="create-a-trigger-on-other-endpoints-to-apply-schema-changes-during-insertion"></a>Créer un déclencheur sur d’autres points de terminaison pour appliquer les modifications de schéma lors de l’insertion

Ce déclencheur exécute la commande de modification de schéma lorsqu’il est synchronisé avec d’autres points de terminaison. Vous devez créer ce déclencheur sur tous les points de terminaison, à l’exception de celui où les modifications de schéma sont effectuées (autrement dit, dans la base de données où le déclencheur DDL `AlterTableDDLTrigger` est créé à l’étape précédente).

```sql
CREATE TRIGGER SchemaChangesTrigger
ON SchemaChanges
AFTER INSERT
AS
DECLARE \@lastAppliedId bigint
DECLARE \@id bigint
DECLARE \@sqlStmt nvarchar(max)
SELECT TOP 1 \@lastAppliedId=LastAppliedId FROM SchemaChangeHistory
SELECT TOP 1 \@id = id, \@SqlStmt = SqlStmt FROM SchemaChanges WHERE id \> \@lastAppliedId ORDER BY id
IF (\@id = \@lastAppliedId + 1)
BEGIN
    EXEC sp_executesql \@SqlStmt
        UPDATE SchemaChangeHistory SET LastAppliedId = \@id
    WHILE (1 = 1)
    BEGIN
        SET \@id = \@id + 1
        IF exists (SELECT id FROM SchemaChanges WHERE ID = \@id)
            BEGIN
                SELECT \@sqlStmt = SqlStmt FROM SchemaChanges WHERE ID = \@id
                EXEC sp_executesql \@SqlStmt
                UPDATE SchemaChangeHistory SET LastAppliedId = \@id
            END
        ELSE
            BREAK;
    END
END
```

Ce déclencheur s’exécute après l’insertion et vérifie si la commande actuelle doit s’exécuter ensuite. La logique du code garantit qu’aucune instruction de modification de schéma n’est ignorée et que toutes les modifications sont appliquées, même si l’insertion est hors service.

### <a name="sync-the-schema-change-tracking-table-to-all-endpoints"></a>Synchroniser la table de suivi des modifications de schéma avec tous les points de terminaison

Vous pouvez synchroniser la table de suivi des modifications de schéma avec tous les points de terminaison à l’aide du groupe de synchronisation existant ou d’un nouveau groupe de synchronisation. Assurez-vous que les modifications dans la table de suivi peuvent être synchronisées avec tous les points de terminaison, en particulier si vous utilisez la synchronisation dans un seul sens.

Ne synchronisez pas la table d’historique des modifications de schéma, étant donné que cette table conserve un état différent sur différents points de terminaison.

### <a name="apply-the-schema-changes-in-a-sync-group"></a>Appliquer les modifications de schéma dans un groupe de synchronisation

Seules les modifications de schéma apportées dans la base de données où le déclencheur DDL est créé sont répliquées. Les modifications de schéma apportées dans d’autres bases de données ne sont pas répliquées.

Une fois les modifications de schéma répliquées vers tous les points de terminaison, vous devez également prendre des mesures supplémentaires pour mettre à jour le schéma de synchronisation afin de démarrer ou d’arrêter la synchronisation des nouvelles colonnes.

#### <a name="add-new-columns"></a>Ajouter de nouvelles colonnes

1.  Modifiez le schéma.

1.  Évitez toute modification de données dans laquelle les nouvelles colonnes sont impliquées, jusqu’à ce que vous ayez terminé l’étape qui crée le déclencheur.

1.  Attendez que les modifications de schéma soient appliquées à tous les points de terminaison.

1.  Actualisez le schéma de base de données et ajoutez la nouvelle colonne au schéma de synchronisation.

1.  Les données dans la nouvelle colonne sont synchronisées au cours de la prochaine opération de synchronisation.

#### <a name="remove-columns"></a>Supprimer des colonnes

1.  Supprimez les colonnes du schéma de synchronisation. Data Sync arrête la synchronisation des données dans ces colonnes.

1.  Modifiez le schéma.

1.  Actualisez le schéma de base de données.

#### <a name="update-data-types"></a>Mettre à jour les types de données

1.  Modifiez le schéma.

1.  Attendez que les modifications de schéma soient appliquées à tous les points de terminaison.

1.  Actualisez le schéma de base de données.

1.  Si les types de données anciens et nouveaux ne sont pas entièrement compatibles (par exemple, si vous passez de `int` à `bigint`), la synchronisation peut échouer avant la fin des étapes qui permettent de créer les déclencheurs. La synchronisation réussit après une nouvelle tentative.

#### <a name="rename-columns-or-tables"></a>Renommer des colonnes ou des tables

Data Sync cesse de fonctionner si des colonnes ou des tables sont renommées. Créez une nouvelle table ou colonne, renseignez les données, puis supprimez l’ancienne table ou colonne au lieu de la renommer.

#### <a name="other-types-of-schema-changes"></a>Autres types de modifications de schéma

Pour les autres types de modifications de schéma (par exemple, la création de procédures stockées ou la suppression d’un index), la mise à jour du schéma de synchronisation n’est pas requise.

## <a name="troubleshoot-automated-schema-change-replication"></a><a name="troubleshoot"></a> Résoudre les problèmes de réplication de modification de schéma automatisée

La logique de réplication décrite dans cet article s’arrête dans certaines situations, par exemple, si vous avez effectué une modification de schéma dans une base de données locale qui n’est pas prise en charge dans Azure SQL Database. Dans ce cas, la synchronisation de la table de suivi des modifications de schéma échoue. Vous devez corriger ce problème manuellement :

1.  Désactivez le déclencheur DDL et évitez toute autre modification de schéma jusqu’à ce que le problème soit résolu.

1.  Dans la base de données du point de terminaison où le problème se produit, désactivez le déclencheur AFTER INSERT sur le point de terminaison où la modification de schéma n’est pas possible. Cette action permet la synchronisation de la commande de modification de schéma.

1.  Déclenchez la synchronisation pour synchroniser la table de suivi des modifications de schéma.

1.  Dans la base de données du point de terminaison où le problème se produit, interrogez la table d’historique des modifications de schéma pour obtenir l’ID de la dernière commande de modification de schéma appliquée.

1.  Interrogez la table de suivi des modifications de schéma pour répertorier toutes les commandes avec un ID supérieur à la valeur d’ID que vous avez récupérée à l’étape précédente.

    a.  Ignorez les commandes qui ne peuvent pas être exécutées dans la base de données du point de terminaison. Vous devez traiter l’incohérence de schéma. Annulez les modifications de schéma d’origine si l’incohérence a un impact sur votre application.

    b.  Appliquez manuellement les commandes qui doivent être appliquées.

1.  Mettez à jour la table d’historique des modifications de schéma et définissez le dernier ID appliqué sur la valeur correcte.

1.  Vérifiez si le schéma est à jour.

1.  Réactivez le déclencheur AFTER INSERT désactivé dans la deuxième étape.

1.  Réactivez le déclencheur DDL désactivé dans la première étape.

Si vous souhaitez nettoyer les enregistrements dans la table de suivi des modifications de schéma, utilisez DELETE au lieu de TRUNCATE. Ne réamorcez jamais la colonne d’identité dans la table de suivi des modifications de schéma à l’aide de DBCC CHECKIDENT. Vous pouvez créer de nouvelles tables de suivi des modifications de schéma et mettre à jour le nom de la table dans le déclencheur DDL si un réamorçage est requis.

## <a name="other-considerations"></a><a name="other"></a> Autres points à considérer

-   Les utilisateurs de base de données qui configurent le concentrateur et les bases de données membres doivent disposer des autorisations suffisantes pour exécuter les commandes de modification de schéma.

-   Vous pouvez ajouter davantage de filtres dans le déclencheur DDL pour répliquer uniquement la modification de schéma dans les opérations ou les tables sélectionnées.

-   Vous pouvez uniquement apporter des modifications de schéma dans la base de données où le déclencheur DDL est créé.

-   Si vous apportez une modification dans une base de données SQL Server, assurez-vous que la modification de schéma est prise en charge dans Azure SQL Database.

-   Si des modifications de schéma sont apportées dans des bases de données autres que la base de données où le déclencheur DDL est créé, les modifications ne sont pas répliquées. Pour éviter ce problème, vous pouvez créer des déclencheurs DDL afin de bloquer les modifications sur les autres points de terminaison.

-   Si vous avez besoin modifier le schéma de la table de suivi des modifications de schéma, désactivez le déclencheur DDL avant d’apporter la modification, puis appliquez manuellement la modification à tous les points de terminaison. La mise à jour du schéma dans un déclencheur AFTER INSERT sur la même table ne fonctionne pas.

-   Ne réamorcez pas la colonne d’identité à l’aide de DBCC CHECKIDENT.

-   N’utilisez pas TRUNCATE pour nettoyer les données dans la table de suivi des modifications de schéma.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur SQL Data Sync, consultez :

-   Vue d’ensemble - [Synchroniser des données entre plusieurs bases de données cloud et locales avec Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
-   Configurer Data Sync
    - Sur le portail - [Tutoriel : Configurer SQL Data Sync pour synchroniser les données entre Azure SQL Database et SQL Server](sql-data-sync-sql-server-configure.md)
    - Avec PowerShell
        -  [Utiliser PowerShell pour synchroniser des données entre plusieurs bases de données dans Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Utiliser PowerShell pour synchroniser des données entre une base de données d'Azure SQL Database et une base de données d'une instance SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Agent de synchronisation des données - [Agent de synchronisation des données pour Azure SQL Data Sync](sql-data-sync-agent-overview.md)
-   Bonnes pratiques - [Bonnes pratiques pour Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Supervision – [Superviser SQL Data Sync avec les journaux d’activité Azure Monitor](./monitor-tune-overview.md)
-   Résolution des problèmes - [Résoudre les problèmes liés à Azure SQL Data Sync]()
-   Mettre à jour le schéma de synchronisation
    -   Avec PowerShell - [Utiliser PowerShell pour mettre à jour le schéma de synchronisation dans un groupe de synchronisation existant](scripts/update-sync-schema-in-sync-group.md)