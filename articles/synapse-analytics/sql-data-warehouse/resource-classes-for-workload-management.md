---
title: Classes de ressources pour la gestion des charges de travail
description: Conseils d’utilisation des classes de ressources pour gérer la concurrence et les ressources de calcul en lien avec les requêtes dans Azure Synapse Analytics.
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
ms.openlocfilehash: a557d4045b18b5c0ff71b3e47f0c189028702863
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91289529"
---
# <a name="workload-management-with-resource-classes-in-azure-synapse-analytics"></a>Gestion des charges de travail avec des classes de ressources dans Azure Synapse Analytics

Conseils d’utilisation des classes de ressources pour gérer la mémoire et la concurrence pour les requêtes de pool SQL Synapse dans Azure Synapse.  

## <a name="what-are-resource-classes"></a>Que sont les classes de ressources ?

La capacité de performances d’une requête est déterminée par la classe de ressources de cette dernière.  Les classes de ressources sont des limites de ressources prédéterminées dans un pool SQL Synapse qui régissent les ressources de calcul et la concurrence lors de l’exécution des requêtes. Les classes de ressources peuvent vous aider à configurer des ressources pour vos requêtes en définissant des limites applicables au nombre de requêtes qui s’exécutent simultanément et aux ressources de calcul qui leur sont respectivement attribuées.  Il faut faire un compromis entre la mémoire et la concurrence.

- Des classes de ressources plus petites réduisent la mémoire maximale par requête, mais augmentent la simultanéité.
- Des classes de ressources plus grandes augmentent la mémoire maximale par requête, mais réduisent la concurrence.

Il existe deux types de classes de ressources :

- Les classes de ressources statiques, qui sont bien adaptées pour la concurrence accrue sur un jeu de données de taille fixe.
- Les classes de ressources dynamiques, qui sont bien adaptées pour les jeux de données dont la taille augmente et dont les performances doivent s’améliorer à mesure que le niveau de service monte en puissance.

Les classes de ressources utilisent des emplacements de concurrence pour mesurer la consommation des ressources.  Les [emplacements de concurrence](#concurrency-slots) sont expliqués plus loin dans cet article.

- Pour afficher l’utilisation des ressources pour les classes de ressources, consultez [Limites de mémoire et de concurrence](memory-concurrency-limits.md).
- Pour ajuster la classe de ressources, vous pouvez exécuter la requête sous un autre utilisateur ou modifier l’appartenance [de l’utilisateur actuel à une classe de ressources](#change-a-users-resource-class).

### <a name="static-resource-classes"></a>Classes de ressources statiques

Les classes de ressources statiques allouent la même quantité de mémoire, mesurée en [unités d’entrepôt de données](what-is-a-data-warehouse-unit-dwu-cdwu.md), quel que soit le niveau de performance actuel. Étant donné que la mémoire allouée aux requêtes est la même quel que soit le niveau de performance, une [augmentation de l’échelle de l’entrepôt de données](quickstart-scale-compute-portal.md) permet l’exécution d’un plus grand nombre de requêtes au sein d’une classe de ressources.  Les classes de ressources statiques sont idéales si le volume de données est connu et constant.

Les classes de ressources statiques sont implémentées avec ces rôles de base de données prédéfinis :

- staticrc10
- staticrc20
- staticrc30
- staticrc40
- staticrc50
- staticrc60
- staticrc70
- staticrc80

### <a name="dynamic-resource-classes"></a>Classes de ressources dynamiques

Les classes de ressources dynamiques allouent une quantité de mémoire variable en fonction du niveau de service actuel. Alors que les classes de ressources statiques conviennent dans les situations de concurrence accrue et pour les volumes de données statiques, les classes de ressources dynamiques sont mieux adaptées pour une quantité de données croissante ou variable.  Lors de l’augmentation de l’échelle du niveau de service, vos requêtes obtiennent automatiquement davantage de mémoire.  

Les classes de ressources dynamiques sont implémentées avec les rôles de base de données prédéfinis suivants :

- smallrc
- mediumrc
- largerc
- xlargerc

L’allocation de mémoire pour chaque classe de ressources est la suivante.

| Niveau de service  | smallrc           | mediumrc               | largerc                | xlargerc               |
|:--------------:|:-----------------:|:----------------------:|:----------------------:|:----------------------:|
| DW100c         | 25 %               | 25 %                    | 25 %                    | 70 %                    |
| DW200c         | 12,5 %             | 12,5 %                  | 22 %                    | 70 %                    |
| DW300c         | 8 %                | 10 %                    | 22 %                    | 70 %                    |
| DW400c         | 6,25 %             | 10 %                    | 22 %                    | 70 %                    |
| DW500c         | 5 %                | 10 %                    | 22 %                    | 70 %                    |
| DW1000c à<br> DW30000c | 3 %       | 10 %                    | 22 %                    | 70 %                    |

### <a name="default-resource-class"></a>Classe de ressources par défaut

Par défaut, chaque utilisateur appartient à la classe de ressources dynamiques **smallrc**.

La classe de ressources de l’administrateur de service est fixée à smallrc et ne peut pas être changée.  L’administrateur de service est l’utilisateur créé pendant le processus d’approvisionnement.  Dans ce contexte, l’administrateur de services fédérés est la connexion spécifiée pour la « connexion d’administrateur du serveur » au moment de la création d’un pool SQL Synapse avec un nouveau serveur.

> [!NOTE]
> Les utilisateurs ou groupes définis en tant qu’administrateur Active Directory sont également administrateurs de service.
>
>

## <a name="resource-class-operations"></a>Opérations de ressources de classe

Les classes de ressources sont conçues pour améliorer les performances de gestion et de manipulation de données. Les requêtes complexes peuvent également bénéficier d’une exécution sous une classe de ressource de grande taille. Par exemple, les performances des requêtes pour un grand nombre de jointures et de tris peuvent être améliorées lorsque la classe de ressources est suffisamment grande pour permettre l’exécution de la requête dans la mémoire.

### <a name="operations-governed-by-resource-classes"></a>Opérations régies par des classes de ressources

Les opérations suivantes sont régies par des classes de ressources :

- INSERT-SELECT, UPDATE, DELETE
- SELECT (lors de l’interrogation des tables d’utilisateur)
- ALTER INDEX - REBUILD ou REORGANIZE
- ALTER TABLE REBUILD
- CREATE INDEX
- CREATE CLUSTERED COLUMNSTORE INDEX
- CREATE TABLE AS SELECT (CTAS)
- Chargement de données
- Opérations de déplacement de données effectuées par le Service le déplacement des données (DMS)

> [!NOTE]  
> Les instructions SELECT sur des vues de gestion dynamique (DMV) ou d’autres vues système ne sont régies par aucune limite de concurrence. Les utilisateurs peuvent surveiller le système en toutes circonstances, quel que soit le nombre de requêtes en cours d’exécution dessus.
>
>

### <a name="operations-not-governed-by-resource-classes"></a>Opérations non régies par des classes de ressources

Certaines requêtes s’exécutent toujours dans la classe de ressources smallrc, même si l’utilisateur est membre d’une classe de ressources plus grande. Ces requêtes exemptes ne sont pas prises en compte pour la limite de concurrence. Par exemple, si la limite de concurrence est définie sur 16, de nombreux utilisateurs peuvent sélectionner des vues système sans que cela ait d’incidence sur les emplacements de concurrence disponibles.

Les instructions suivantes sont exemptes de classes de ressources et s’exécutent toujours dans smallrc :

- CREATE ou DROP TABLE
- l’instruction ALTER TABLE ... SWITCH, SPLIT ou MERGE PARTITION
- ALTER INDEX DISABLE
- DROP INDEX
- CREATE, UPDATE ou DROP STATISTICS
- TRUNCATE TABLE
- ALTER AUTHORIZATION
- CREATE LOGIN
- CREATE, ALTER ou DROP USER
- CREATE, ALTER ou DROP PROCEDURE
- CREATE ou DROP VIEW
- INSERT VALUES
- SELECT à partir des affichages système et des DMV
- EXPLAIN
- DBCC

<!--
Removed as these two are not confirmed / supported under Azure Synapse Analytics
- CREATE REMOTE TABLE AS SELECT
- CREATE EXTERNAL TABLE AS SELECT
- REDISTRIBUTE
-->

## <a name="concurrency-slots"></a>Emplacements de concurrence

Les emplacements de concurrence sont pratiques pour suivre les ressources disponibles pour l’exécution des requêtes. Il sont comparables à des tickets que vous achetez afin de réserver des sièges pour assister à un concert dont le nombre de places est limité. Le nombre total d’emplacements de concurrence par entrepôt de données est déterminé par le niveau de service. Avant que son exécution puisse démarrer, une requête doit pouvoir réserver suffisamment d’emplacements de concurrence. Une fois terminée, la requête libère ses emplacements de concurrence.  

- Une requête s’exécutant avec 10 emplacements de concurrence peut accéder à 5 fois plus de ressources de calcul qu’une requête s’exécutant avec 2 emplacements de concurrence.
- Si chaque requête nécessite 10 emplacements de concurrence alors que 40 sont disponibles, seules 4 requêtes peuvent s’exécuter simultanément.

Seules ds requêtes régies par des ressources consomment des emplacements de concurrence. Les requêtes système et certaines requêtes banales ne consomment aucun emplacement. Le nombre exact d’emplacements de concurrence consommés est déterminé par la classe de ressources de la requête.

## <a name="view-the-resource-classes"></a>Afficher les classes de ressources

Les classes de ressources sont implémentées sous forme de rôles de base de données prédéfinis. Il existe deux types de classes de ressources : statiques et dynamiques. Pour afficher une liste des classes de ressources, utilisez la requête suivante :

```sql
SELECT name
FROM   sys.database_principals
WHERE  name LIKE '%rc%' AND type_desc = 'DATABASE_ROLE';
```

## <a name="change-a-users-resource-class"></a>Modifier la classe de ressources d’un utilisateur

Les classes de ressources sont implémentées en assignant des utilisateurs à des rôles de base de données. Quand un utilisateur exécute une requête, celle-ci est exécutée avec la classe de ressources de l’utilisateur. Par exemple, si un utilisateur est membre du rôle de base de données staticrc10, ses requêtes s’exécutent avec de petites quantités de mémoire. Si un utilisateur de base de données est membre du rôle de base de données xlargerc ou staticrc80, ses requêtes s’exécutent avec de grandes quantités de mémoire.

Pour augmenter la classe de ressources d’un utilisateur, utilisez [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) afin d’ajouter l’utilisateur à un rôle de base de données d’une grande classe de ressources.  Le code ci-dessous ajoute un utilisateur au rôle de base de données largerc.  Chaque requête obtient 22 % de la mémoire système.

```sql
EXEC sp_addrolemember 'largerc', 'loaduser';
```

Pour réduire la classe de ressources, utilisez [sp_droprolemember](/sql/relational-databases/system-stored-procedures/sp-droprolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Si « loaduser » n’est membre d’aucune autre classe de ressources, il est placé dans la classe de ressources smallrc par défaut avec une allocation de mémoire de 3 %.  

```sql
EXEC sp_droprolemember 'largerc', 'loaduser';
```

## <a name="resource-class-precedence"></a>Précédence de classe de ressources

Les utilisateurs peuvent être membres de plusieurs classes de ressources. Quand un utilisateur appartient à plus d’une classe de ressources :

- les classes de ressources dynamiques ont la précédence sur les classes de ressources statiques. Par exemple, si un utilisateur est membre des classes de ressources mediumrc (dynamique) et staticrc80 (statique), les requêtes s’exécutent avec la classe de ressources mediumrc.
- Les classes de ressources plus grandes ont la précédence sur les classes de ressources plus petites. Par exemple, si un utilisateur est membre des classes de ressources mediumrc et largerc, les requêtes s’exécutent avec la classe de ressources largerc. De même, si un utilisateur est membre des classes de ressources staticrc20 et statirc80, les requêtes s’exécutent avec les allocations de ressources de staticrc80.

## <a name="recommendations"></a>Recommandations

>[!NOTE]
>Tirez parti des fonctionnalités de gestion des charges de travail ([isolation de charge de travail](sql-data-warehouse-workload-isolation.md), [classification](sql-data-warehouse-workload-classification.md) et [importance](sql-data-warehouse-workload-importance.md)) pour obtenir un meilleur contrôle de votre charge de travail et des performances prévisibles.  
>
>

Nous recommandons de créer un utilisateur dédié à l’exécution d’un type spécifique d’opération de requête ou de chargement. Accordez à cet utilisateur une classe de ressources permanente au lieu de modifier fréquemment la classe de ressources. Les classes de ressources statiques offrent un plus grand contrôle global de la charge de travail. Nous vous suggérons donc de les utiliser avant d’envisager l’utilisation de classes de ressources dynamiques.

### <a name="resource-classes-for-load-users"></a>Classes de ressources pour les utilisateurs de chargement

L’instruction `CREATE TABLE` utilise des index cluster columnstore par défaut. La compression de données dans un index columnstore est une opération nécessitant beaucoup de mémoire, et la sollicitation de la mémoire peut réduire la qualité de l’index. La sollicitation de la mémoire peut conduire à un besoin d’une classe de ressources supérieure lors du chargement de données. Pour garantir que les chargements disposent de suffisamment de mémoire, vous pouvez créer un utilisateur désigné pour exécuter les chargements, et assigner cet utilisateur à une classe de ressources supérieure.

La quantité de mémoire nécessaire au traitement des chargements dépend de la nature de la table chargée et de la taille des données. Pour plus d’informations sur la mémoire requise, voir [Optimiser la qualité du rowgroup pour columnstore](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).

Après avoir déterminé la mémoire requise, choisissez d’affecter l’utilisateur de chargement à une classe de ressources statique ou dynamique.

- Utilisez une classe de ressources statique lorsque la mémoire requise pour la table s’inscrit dans une plage spécifique. Les chargements s’exécutent avec une mémoire appropriée. Lorsque vous modifiez l’échelle de l’entrepôt de données, les charges n’ont pas besoin de davantage de mémoire. En utilisant une classe de ressources statique, les allocations de mémoire restent constantes. Cette cohérence conserve la mémoire et permet l’exécution concurrente de davantage de requêtes. Nous recommandons que les nouvelles solutions utilisent prioritairement les classes de ressources statiques, car celles-ci offrent davantage de contrôle.
- Utilisez une classe de ressources dynamique lorsque la mémoire requise pour la table varie considérablement. Des chargements peuvent nécessiter plus de mémoire que ce qu’offre le niveau actuel de DWU ou de cDWU. La mise à l’échelle de l’entrepôt de données ajoute de la mémoire aux opérations de chargement, ce qui permet d’accélérer les chargements.

### <a name="resource-classes-for-queries"></a>Classes de ressources pour les requêtes

Certaines requêtes nécessitent beaucoup de ressources système, et d’autres non.  

- Choisissez une classe de ressources dynamique quand les requêtes sont complexes mais n’impliquent pas de concurrence élevée.  Par exemple, la génération de rapports quotidiens ou hebdomadaires entraîne un besoin occasionnel de ressources. Si les rapports résultent du traitement de grandes quantités de données, la mise à l’échelle de l’entrepôt de données fournit davantage de mémoire à la classe de ressources existante de l’utilisateur.
- Choisissez une classe de ressources statique lorsque les attentes de ressources varient pendant la journée. Par exemple, une classe de ressources statique fonctionne bien lorsque l’entrepôt de données est interrogé par de nombreuses personnes. Lors de la mise à l’échelle de l’entrepôt de données, la quantité de mémoire allouée à l’utilisateur ne change pas. Par conséquent, davantage de requêtes peuvent être exécutées en parallèle sur le système.

L’allocation de mémoire appropriée dépend de nombreux facteurs tels que la quantité de données interrogées, la nature des schémas de table et les divers prédicats joins, select et group. En règle général, si l’allocation de davantage de mémoire permet d’accélérer l’exécution des requêtes, cela limite la concurrence globale. Si la concurrence n’est pas un problème, une allocation excessive de mémoire n’affecte pas le débit.

Pour ajuster les performances, utilisez des classes de ressources différentes. La section suivante fournit une procédure stockée permettant de déterminer la classe de ressources optimale.

## <a name="example-code-for-finding-the-best-resource-class"></a>Exemple de code pour rechercher la classe de ressources optimale

Vous pouvez utiliser la procédure stockée spécifiée suivante pour déterminer la concurrence et l’allocation de mémoire par classe de ressources pour un SLO donné et la classe de ressources optimale pour les opérations ICC sur une table ICC non partitionnée pour une classe de ressources donnée :

Cette procédure stockée vise à :

1. Déterminer la concurrence et l’allocation de mémoire par classe de ressources pour un SLO donné. L’utilisateur doit fournir la valeur NULL pour le schéma et le nom de table, comme indiqué dans cet exemple.  
2. Déterminer la classe de ressources optimale pour les opérations ICC utilisant beaucoup de mémoire (chargement, copie de table, régénération d’index, etc.) sur une table ICC non partitionnée à une classe de ressources donnée. La procédure stockée utilise un schéma de table pour déterminer l’allocation de mémoire requise.

### <a name="dependencies--restrictions"></a>Dépendances et restrictions

- Cette procédure stockée n’est pas conçue pour calculer la mémoire requise pour une table ICC partitionnée.
- Cette procédure stockée ne prend pas en compte la mémoire requise pour la partie SELECT d’une instruction CTAS/INSERT-SELECT et part du principe qu’il s’agit d’une instruction SELECT.
- Cette procédure stockée utilise une table temporaire disponible dans la session où la procédure stockée a été créée.
- Cette procédure stockée dépend des offres du moment (par exemple, configuration matérielle, configuration DMS). Si l’une d’elles change, cette procédure stockée ne fonctionnera plus correctement.  
- Cette procédure stockée dépend des offres de limite de concurrence existantes. Si elles changent, cette procédure stockée ne fonctionnera plus correctement.  
- Cette procédure stockée dépend des offres de classes de ressources existantes. Si celles-ci changent, cette procédure stockée ne fonctionnera plus correctement.  

>[!NOTE]  
>Si vous n’obtenez pas de sortie après exécution de la procédure stockée avec les paramètres fournis, il se peut que vous soyez confronté à l’un des deux cas suivants :
>
>1. Un paramètre DW contient une valeur SLO non valide.
>2. Il n’existe aucune classe de ressources correspondant à l’opération ICC sur la table.
>
>Par exemple, pour DW100c, l’allocation de mémoire la plus élevée disponible est de 1 Go si le schéma de table est suffisamment volumineux pour dépasser l’exigence de 1 Go.

### <a name="usage-example"></a>Exemple d’utilisation

Syntaxe :  
`EXEC dbo.prc_workload_management_by_DWU @DWU VARCHAR(7), @SCHEMA_NAME VARCHAR(128), @TABLE_NAME VARCHAR(128)`
  
1. @DWU: fournissez un paramètre NULL pour extraire la DWU active de la base de données DW ou fournissez une DWU prise en charge au format « DW100c »
2. @SCHEMA_NAME: fournissez le nom de schéma de la table
3. @TABLE_NAME: fournissez le nom de la table qui vous intéresse

Exemples d’exécution de cette procédure stockée :

```sql
EXEC dbo.prc_workload_management_by_DWU 'DW2000c', 'dbo', 'Table1';  
EXEC dbo.prc_workload_management_by_DWU NULL, 'dbo', 'Table1';  
EXEC dbo.prc_workload_management_by_DWU 'DW6000c', NULL, NULL;  
EXEC dbo.prc_workload_management_by_DWU NULL, NULL, NULL;  
```

L’instruction suivante crée la Table1 utilisée dans les exemples précédents.
`CREATE TABLE Table1 (a int, b varchar(50), c decimal (18,10), d char(10), e varbinary(15), f float, g datetime, h date);`

### <a name="stored-procedure-definition"></a>Définition de la procédure stockée

```sql
-------------------------------------------------------------------------------
-- Dropping prc_workload_management_by_DWU procedure if it exists.
-------------------------------------------------------------------------------
IF EXISTS (SELECT * FROM sys.objects WHERE type = 'P' AND name = 'prc_workload_management_by_DWU')
DROP PROCEDURE dbo.prc_workload_management_by_DWU
GO

-------------------------------------------------------------------------------
-- Creating prc_workload_management_by_DWU.
-------------------------------------------------------------------------------
CREATE PROCEDURE dbo.prc_workload_management_by_DWU
(@DWU VARCHAR(8),
 @SCHEMA_NAME VARCHAR(128),
 @TABLE_NAME VARCHAR(128)
)
AS

IF @DWU IS NULL
BEGIN
-- Selecting proper DWU for the current DB if not specified.

SELECT @DWU = 'DW'+ CAST(CASE WHEN Mem> 4 THEN Nodes*500
  ELSE Mem*100
  END AS VARCHAR(10)) +'c'
    FROM (
      SELECT Nodes=count(distinct n.pdw_node_id), Mem=max(i.committed_target_kb/1000/1000/60)
        FROM sys.dm_pdw_nodes n
        CROSS APPLY sys.dm_pdw_nodes_os_sys_info i
        WHERE type = 'COMPUTE')A
END

-- Dropping temp table if exists.
IF OBJECT_ID('tempdb..#ref') IS NOT NULL
BEGIN
  DROP TABLE #ref;
END;

-- Creating ref. temp table (CTAS) to hold mapping info.
CREATE TABLE #ref
WITH (DISTRIBUTION = ROUND_ROBIN)
AS
WITH
-- Creating concurrency slots mapping for various DWUs.
alloc
AS
(
SELECT 'DW100c' AS DWU,4 AS max_queries,4 AS max_slots,1 AS slots_used_smallrc,1 AS slots_used_mediumrc,2 AS slots_used_largerc,4 AS slots_used_xlargerc,1 AS slots_used_staticrc10,2 AS slots_used_staticrc20,4 AS slots_used_staticrc30,4 AS slots_used_staticrc40,4 AS slots_used_staticrc50,4 AS slots_used_staticrc60,4 AS slots_used_staticrc70,4 AS slots_used_staticrc80
  UNION ALL
   SELECT 'DW200c',8,8,1,2,4,8,1,2,4,8,8,8,8,8
  UNION ALL
   SELECT 'DW300c',12,12,1,2,4,8,1,2,4,8,8,8,8,8
  UNION ALL
   SELECT 'DW400c',16,16,1,4,8,16,1,2,4,8,16,16,16,16
  UNION ALL
   SELECT 'DW500c',20,20,1,4,8,16,1,2,4,8,16,16,16,16
  UNION ALL
   SELECT 'DW1000c',32,40,1,4,8,28,1,2,4,8,16,32,32,32
  UNION ALL
   SELECT 'DW1500c',32,60,1,6,13,42,1,2,4,8,16,32,32,32
  UNION ALL
   SELECT 'DW2000c',48,80,2,8,17,56,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW2500c',48,100,3,10,22,70,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW3000c',64,120,3,12,26,84,1,2,4,8,16,32,64,64
  UNION ALL
   SELECT 'DW5000c',64,200,6,20,44,140,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW6000c',128,240,7,24,52,168,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW7500c',128,300,9,30,66,210,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW10000c',128,400,12,40,88,280,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW15000c',128,600,18,60,132,420,1,2,4,8,16,32,64,128
  UNION ALL
   SELECT 'DW30000c',128,1200,36,120,264,840,1,2,4,8,16,32,64,128
)
-- Creating workload mapping to their corresponding slot consumption and default memory grant.
,map  
AS
(
  SELECT CONVERT(varchar(20), 'SloDWGroupSmall') AS wg_name, slots_used_smallrc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupMedium') AS wg_name, slots_used_mediumrc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupLarge') AS wg_name, slots_used_largerc AS slots_used FROM alloc WHERE DWU = @DWU
UNION ALL
  SELECT CONVERT(varchar(20), 'SloDWGroupXLarge') AS wg_name, slots_used_xlargerc AS slots_used FROM alloc WHERE DWU = @DWU
  UNION ALL
  SELECT 'SloDWGroupC00',1
  UNION ALL
    SELECT 'SloDWGroupC01',2
  UNION ALL
    SELECT 'SloDWGroupC02',4
  UNION ALL
    SELECT 'SloDWGroupC03',8
  UNION ALL
    SELECT 'SloDWGroupC04',16
  UNION ALL
    SELECT 'SloDWGroupC05',32
  UNION ALL
    SELECT 'SloDWGroupC06',64
  UNION ALL
    SELECT 'SloDWGroupC07',128
)

-- Creating ref based on current / asked DWU.
, ref
AS
(
  SELECT  a1.*
  ,       m1.wg_name          AS wg_name_smallrc
  ,       m1.slots_used * 250 AS tgt_mem_grant_MB_smallrc
  ,       m2.wg_name          AS wg_name_mediumrc
  ,       m2.slots_used * 250 AS tgt_mem_grant_MB_mediumrc
  ,       m3.wg_name          AS wg_name_largerc
  ,       m3.slots_used * 250 AS tgt_mem_grant_MB_largerc
  ,       m4.wg_name          AS wg_name_xlargerc
  ,       m4.slots_used * 250 AS tgt_mem_grant_MB_xlargerc
  ,       m5.wg_name          AS wg_name_staticrc10
  ,       m5.slots_used * 250 AS tgt_mem_grant_MB_staticrc10
  ,       m6.wg_name          AS wg_name_staticrc20
  ,       m6.slots_used * 250 AS tgt_mem_grant_MB_staticrc20
  ,       m7.wg_name          AS wg_name_staticrc30
  ,       m7.slots_used * 250 AS tgt_mem_grant_MB_staticrc30
  ,       m8.wg_name          AS wg_name_staticrc40
  ,       m8.slots_used * 250 AS tgt_mem_grant_MB_staticrc40
  ,       m9.wg_name          AS wg_name_staticrc50
  ,       m9.slots_used * 250 AS tgt_mem_grant_MB_staticrc50
  ,       m10.wg_name          AS wg_name_staticrc60
  ,       m10.slots_used * 250 AS tgt_mem_grant_MB_staticrc60
  ,       m11.wg_name          AS wg_name_staticrc70
  ,       m11.slots_used * 250 AS tgt_mem_grant_MB_staticrc70
  ,       m12.wg_name          AS wg_name_staticrc80
  ,       m12.slots_used * 250 AS tgt_mem_grant_MB_staticrc80
  FROM alloc a1
  JOIN map   m1  ON a1.slots_used_smallrc     = m1.slots_used and m1.wg_name = 'SloDWGroupSmall'
  JOIN map   m2  ON a1.slots_used_mediumrc    = m2.slots_used and m2.wg_name = 'SloDWGroupMedium'
  JOIN map   m3  ON a1.slots_used_largerc     = m3.slots_used and m3.wg_name = 'SloDWGroupLarge'
  JOIN map   m4  ON a1.slots_used_xlargerc    = m4.slots_used and m4.wg_name = 'SloDWGroupXLarge'
  JOIN map   m5  ON a1.slots_used_staticrc10    = m5.slots_used and m5.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m6  ON a1.slots_used_staticrc20    = m6.slots_used and m6.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m7  ON a1.slots_used_staticrc30    = m7.slots_used and m7.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m8  ON a1.slots_used_staticrc40    = m8.slots_used and m8.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m9  ON a1.slots_used_staticrc50    = m9.slots_used and m9.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m10  ON a1.slots_used_staticrc60    = m10.slots_used and m10.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m11  ON a1.slots_used_staticrc70    = m11.slots_used and m11.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  JOIN map   m12  ON a1.slots_used_staticrc80    = m12.slots_used and m12.wg_name NOT IN ('SloDWGroupSmall','SloDWGroupMedium','SloDWGroupLarge','SloDWGroupXLarge')
  WHERE   a1.DWU = @DWU
)
SELECT  DWU
,       max_queries
,       max_slots
,       slots_used
,       wg_name
,       tgt_mem_grant_MB
,       up1 as rc
,       (ROW_NUMBER() OVER(PARTITION BY DWU ORDER BY DWU)) as rc_id
FROM
(
    SELECT  DWU
    ,       max_queries
    ,       max_slots
    ,       slots_used
    ,       wg_name
    ,       tgt_mem_grant_MB
    ,       REVERSE(SUBSTRING(REVERSE(wg_names),1,CHARINDEX('_',REVERSE(wg_names),1)-1)) as up1
    ,       REVERSE(SUBSTRING(REVERSE(tgt_mem_grant_MBs),1,CHARINDEX('_',REVERSE(tgt_mem_grant_MBs),1)-1)) as up2
    ,       REVERSE(SUBSTRING(REVERSE(slots_used_all),1,CHARINDEX('_',REVERSE(slots_used_all),1)-1)) as up3
    FROM    ref AS r1
    UNPIVOT
    (
        wg_name FOR wg_names IN (wg_name_smallrc,wg_name_mediumrc,wg_name_largerc,wg_name_xlargerc,
        wg_name_staticrc10, wg_name_staticrc20, wg_name_staticrc30, wg_name_staticrc40, wg_name_staticrc50,
        wg_name_staticrc60, wg_name_staticrc70, wg_name_staticrc80)
    ) AS r2
    UNPIVOT
    (
        tgt_mem_grant_MB FOR tgt_mem_grant_MBs IN (tgt_mem_grant_MB_smallrc,tgt_mem_grant_MB_mediumrc,
        tgt_mem_grant_MB_largerc,tgt_mem_grant_MB_xlargerc, tgt_mem_grant_MB_staticrc10, tgt_mem_grant_MB_staticrc20,
        tgt_mem_grant_MB_staticrc30, tgt_mem_grant_MB_staticrc40, tgt_mem_grant_MB_staticrc50,
        tgt_mem_grant_MB_staticrc60, tgt_mem_grant_MB_staticrc70, tgt_mem_grant_MB_staticrc80)
    ) AS r3
    UNPIVOT
    (
        slots_used FOR slots_used_all IN (slots_used_smallrc,slots_used_mediumrc,slots_used_largerc,
        slots_used_xlargerc, slots_used_staticrc10, slots_used_staticrc20, slots_used_staticrc30,
        slots_used_staticrc40, slots_used_staticrc50, slots_used_staticrc60, slots_used_staticrc70,
        slots_used_staticrc80)
    ) AS r4
) a
WHERE   up1 = up2
AND     up1 = up3
;

-- Getting current info about workload groups.
WITH  
dmv  
AS  
(
  SELECT
          rp.name                                           AS rp_name
  ,       rp.max_memory_kb*1.0/1048576                      AS rp_max_mem_GB
  ,       (rp.max_memory_kb*1.0/1024)
          *(request_max_memory_grant_percent/100)           AS max_memory_grant_MB
  ,       (rp.max_memory_kb*1.0/1048576)
          *(request_max_memory_grant_percent/100)           AS max_memory_grant_GB
  ,       wg.name                                           AS wg_name
  ,       wg.importance                                     AS importance
  ,       wg.request_max_memory_grant_percent               AS request_max_memory_grant_percent
  FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
  JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp    ON  wg.pdw_node_id  = rp.pdw_node_id
                                                                  AND wg.pool_id      = rp.pool_id
  WHERE   rp.name = 'SloDWPool'
  GROUP BY
          rp.name
  ,       rp.max_memory_kb
  ,       wg.name
  ,       wg.importance
  ,       wg.request_max_memory_grant_percent
)
-- Creating resource class name mapping.
,names
AS
(
  SELECT 'smallrc' as resource_class, 1 as rc_id
  UNION ALL
    SELECT 'mediumrc', 2
  UNION ALL
    SELECT 'largerc', 3
  UNION ALL
    SELECT 'xlargerc', 4
  UNION ALL
    SELECT 'staticrc10', 5
  UNION ALL
    SELECT 'staticrc20', 6
  UNION ALL
    SELECT 'staticrc30', 7
  UNION ALL
    SELECT 'staticrc40', 8
  UNION ALL
    SELECT 'staticrc50', 9
  UNION ALL
    SELECT 'staticrc60', 10
  UNION ALL
    SELECT 'staticrc70', 11
  UNION ALL
    SELECT 'staticrc80', 12
)
,base AS
(   SELECT  schema_name
    ,       table_name
    ,       SUM(column_count)                   AS column_count
    ,       ISNULL(SUM(short_string_column_count),0)   AS short_string_column_count
    ,       ISNULL(SUM(long_string_column_count),0)    AS long_string_column_count
    FROM    (   SELECT  sm.name                                             AS schema_name
                ,       tb.name                                             AS table_name
                ,       COUNT(co.column_id)                                 AS column_count
                           ,       CASE    WHEN co.system_type_id IN (36,43,106,108,165,167,173,175,231,239)
                                AND  co.max_length <= 32
                                THEN COUNT(co.column_id)
                        END                                                 AS short_string_column_count
                ,       CASE    WHEN co.system_type_id IN (165,167,173,175,231,239)
                                AND  co.max_length > 32 and co.max_length <=8000
                                THEN COUNT(co.column_id)
                        END                                                 AS long_string_column_count
                FROM    sys.schemas AS sm
                JOIN    sys.tables  AS tb   on sm.[schema_id] = tb.[schema_id]
                JOIN    sys.columns AS co   ON tb.[object_id] = co.[object_id]
                           WHERE tb.name = @TABLE_NAME AND sm.name = @SCHEMA_NAME
                GROUP BY sm.name
                ,        tb.name
                ,        co.system_type_id
                ,        co.max_length            ) a
GROUP BY schema_name
,        table_name
)
, size AS
(
SELECT  schema_name
,       table_name
,       75497472                                            AS table_overhead

,       column_count*1048576*8                              AS column_size
,       short_string_column_count*1048576*32                       AS short_string_size,       (long_string_column_count*16777216) AS long_string_size
FROM    base
UNION
SELECT CASE WHEN COUNT(*) = 0 THEN 'EMPTY' END as schema_name
         ,CASE WHEN COUNT(*) = 0 THEN 'EMPTY' END as table_name
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as table_overhead
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as column_size
         ,CASE WHEN COUNT(*) = 0 THEN 0 END as short_string_size

,CASE WHEN COUNT(*) = 0 THEN 0 END as long_string_size
FROM   base
)
, load_multiplier as
(
SELECT  CASE
          WHEN FLOOR(8 * (CAST (CAST(REPLACE(REPLACE(@DWU,'DW',''),'c','') AS INT) AS FLOAT)/6000)) > 0
            AND CHARINDEX(@DWU,'c')=0
          THEN FLOOR(8 * (CAST (CAST(REPLACE(REPLACE(@DWU,'DW',''),'c','') AS INT) AS FLOAT)/6000))
          ELSE 1
        END AS multiplication_factor
)
       SELECT  r1.DWU
       , schema_name
       , table_name
       , rc.resource_class as closest_rc_in_increasing_order
       , max_queries_at_this_rc = CASE
             WHEN (r1.max_slots / r1.slots_used > r1.max_queries)
                  THEN r1.max_queries
             ELSE r1.max_slots / r1.slots_used
                  END
       , r1.max_slots as max_concurrency_slots
       , r1.slots_used as required_slots_for_the_rc
       , r1.tgt_mem_grant_MB  as rc_mem_grant_MB
       , CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) AS est_mem_grant_required_for_cci_operation_MB
       FROM    size
       , load_multiplier
       , #ref r1, names  rc
       WHERE r1.rc_id=rc.rc_id
                     AND CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) < r1.tgt_mem_grant_MB
       ORDER BY ABS(CAST((table_overhead*1.0+column_size+short_string_size+long_string_size)*multiplication_factor/1048576    AS DECIMAL(18,2)) - r1.tgt_mem_grant_MB)
GO
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion de la sécurité et des utilisateurs de base de données, consultez [Sécuriser une base de données dans SQL Synapse](sql-data-warehouse-overview-manage-security.md). Pour plus d’informations sur la façon dont des classes de ressources plus élevées peuvent améliorer la qualité des index cluster columnstore, voir [Optimiser la qualité du rowgroup pour columnstore](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).
