---
title: Gérer les données historiques avec la stratégie de conservation - Azure SQL Edge
description: Découvrez comment gérer les données historiques avec la stratégie de conservation dans Azure SQL Edge
keywords: SQL Edge, conservation des données
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: f547f0b3219889b54364c4805c2dd5b183a9861a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392365"
---
# <a name="manage-historical-data-with-retention-policy"></a>Gérer les données historiques avec la stratégie de conservation

La conservation des données peut être activée pour une base de données et chacune de ses tables sous-jacentes, ce qui permet aux utilisateurs de créer des stratégies de gestion de l’ancienneté des données contenues dans leurs bases de données et les tables sous-jacentes. L’application de la conservation des données est simple : il suffit de définir un seul paramètre durant la création de la table ou dans le cadre d’une modification de la table avec ALTER TABLE. 

Une fois la stratégie de conservation des données définie pour une base de données et la table sous-jacente, une tâche de minuteur de temps en arrière-plan s’exécute pour supprimer tous les enregistrements obsolètes de la table à laquelle la stratégie de conservation est appliquée. L’identification des lignes en question et leur suppression de la table sont effectuées de façon transparente par la tâche en arrière-plan planifiée et exécutée par le système. La condition d’ancienneté pour les lignes de la table est vérifiée par rapport à la colonne utilisée en tant que `filter_column` dans la définition de la table. Si, par exemple, la période de conservation est définie à une semaine, les lignes de la table prises en compte dans le nettoyage remplissent l’une des conditions suivantes : 

- Si la colonne de filtre utilise le type de données DATETIMEOFFSET, la condition est `filter_column < DATEADD(WEEK, -1, SYSUTCDATETIME())`
- Sinon, la condition est `filter_column < DATEADD(WEEK, -1, SYSDATETIME())`

## <a name="data-retention-cleanup-phases"></a>Phases de nettoyage de la conservation des données

L’opération de nettoyage de la conservation des données comprend deux phases. 
- Phase de découverte : l’opération de nettoyage identifie toutes les tables dans les bases de données utilisateur afin de créer une liste de nettoyage. La découverte se produit une fois par jour.
- Phase de nettoyage : le nettoyage est lancé sur toutes les tables identifiées dans la phase de découverte qui ont une période de conservation des données limitée. Si l’opération de nettoyage ne peut pas être effectuée sur une table, cette table est ignorée pendant la tâche en cours et fera l’objet d’une nouvelle tentative dans l’itération suivante. Le nettoyage obéit aux principes suivants
    - Si une ligne obsolète est verrouillée par une autre transaction, cette ligne est ignorée. 
    - Le nettoyage s’exécute avec un paramètre de délai d’attente de verrou de 5 secondes par défaut. Si les verrous ne peuvent pas être acquis sur les tables dans ce délai, la table est ignorée pendant la tâche en cours, et elle fera l’objet d’une nouvelle tentative dans l’itération suivante.
    - En cas d’erreur lors du nettoyage d’une table, cette table est ignorée jusqu’à une nouvelle tentative dans la prochaine itération.

## <a name="manual-cleanup"></a>Nettoyage manuel

Selon les paramètres de conservation des données appliqués à une table et la nature de la charge de travail sur la base de données, il peut arriver que le thread de nettoyage automatique ne supprime pas complètement toutes les lignes obsolètes lors de son exécution. Pour résoudre ce problème et permettre aux utilisateurs de supprimer manuellement les lignes obsolètes restantes, la procédure stockée `sys.sp_cleanup_data_retention` a été introduite dans Azure SQL Edge. 

Cette procédure stockée a trois paramètres. 
    - Schema Name : nom du schéma propriétaire de la table. Il s’agit d’un paramètre obligatoire. 
    - Table Name : nom de la table qui fait l’objet du nettoyage manuel. Il s’agit d’un paramètre obligatoire. 
    - rowcount (Output) : variable de sortie. Retourne le nombre de lignes qui ont été nettoyées par la procédure stockée de nettoyage manuel. Il s'agit d'un paramètre facultatif. 

L’exemple suivant illustre l’exécution de la procédure stockée de nettoyage manuel pour la table `dbo.data_retention_table`.

```sql
declare @rowcnt bigint 
EXEC sys.sp_cleanup_data_retention 'dbo', 'data_retention_table', @rowcnt output 
select @rowcnt 
```

## <a name="how-obsolete-rows-are-deleted"></a>Comment les lignes obsolètes sont supprimées

Le processus de nettoyage dépend de la disposition de l’index de la table. Une tâche en arrière-plan est créée pour effectuer le nettoyage des données obsolètes dans toutes les tables où la période de conservation est limitée. La logique de nettoyage de l’index rowstore (arbre B ou segment de mémoire) supprime les lignes anciennes par petits blocs (jusqu’à 10 000), ce qui réduit la charge sur le journal de la base de données et le sous-système d’E/S. Bien que la logique de nettoyage utilise l’index d’arbre B (B-tree) nécessaire, l’ordre des suppressions des lignes antérieures à la période de rétention ne peut pas être garanti. Vous ne devez donc pas accepter de dépendances relatives à l’ordre de nettoyage dans vos applications.

La tâche de nettoyage pour le cluster columnstore supprime l’ensemble des groupes de lignes en une fois (ceux-ci contiennent généralement 1 million de lignes chacun). Ce procédé est très efficace, surtout dans les scénarios où les données sont générées et obsolètes très rapidement.

![Nettoyage de la conservation des données](./media/data-retention-cleanup/data-retention-cleanup.png)

Grâce à une excellente compression des données et à un nettoyage efficace de la conservation des données, l’index columnstore cluster est un choix idéal dans les scénarios où la charge de travail produit rapidement de grandes quantités de données.

> [!Note]
> Dans le cas des index d’arbre B et des segments de mémoire, la conservation des données exécute une requête de suppression sur les tables sous-jacentes, qui peut être en conflit avec les déclencheurs de suppression sur les tables. Il est recommandé de supprimer les déclencheurs de suppression des tables ou de ne pas activer la conservation des données sur les tables qui comportent un déclencheur DML de suppression.

## <a name="monitoring-data-retention-cleanup"></a>Supervision du nettoyage de la conservation des données

Les opérations de nettoyage de la stratégie de conservation des données peuvent être supervisées à l’aide d’événements étendus (XEvents) dans Azure SQL Edge. Pour plus d’informations sur les événements étendus, consultez [Vue d’ensemble de XEvents](/sql/relational-databases/extended-events/extended-events). 

Les six événements étendus suivants facilitent le suivi de l’état des opérations de nettoyage. 

| Nom | Description |
|------| ------------|
| data_retention_task_started  | Se produit au lancement d’une tâche en arrière-plan de nettoyage des tables auxquelles s’applique une stratégie de conservation. |
| data_retention_task_completed  | Se produit à la fin d’une tâche en arrière-plan de nettoyage des tables auxquelles s’applique une stratégie de conservation. |
| data_retention_task_exception  | Se produit quand une tâche en arrière-plan de nettoyage des tables avec une stratégie de conservation est exécutée en dehors du processus de nettoyage de conservation spécifique à la table. |
| data_retention_cleanup_started  | Se produit au démarrage du processus de nettoyage d’une table à laquelle s’applique une stratégie de conservation des données. |
| data_retention_cleanup_exception  | Se produit en cas d’échec du processus de nettoyage d’une table à laquelle s’applique une stratégie de conservation. |
| data_retention_cleanup_completed  | Se produit au terme du processus de nettoyage d’une table à laquelle s’applique une stratégie de conservation des données. |  

De plus, un nouveau type de mémoire tampon en anneau nommé `RING_BUFFER_DATA_RETENTION_CLEANUP` a été ajouté à la vue de gestion dynamique sys.dm_os_ring_buffers. Cette vue peut être utilisée pour superviser les opérations de nettoyage de la stratégie de conservation des données. 


## <a name="next-steps"></a>Étapes suivantes
- [Stratégie de conservation des données](data-retention-overview.md)
- [Activer et désactiver les stratégies de conservation des données](data-retention-enable-disable.md)