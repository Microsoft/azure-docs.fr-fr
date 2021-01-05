---
title: Utiliser des tables temporaires dans Synapse SQL
description: Conseils de base pour l'utilisation de tables temporaires dans SQL Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: 06faa1da71331c299245a93af96166880e7732de
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451775"
---
# <a name="temporary-tables-in-synapse-sql"></a>Tables temporaires dans SQL Synapse

Cet article contient des conseils de base pour l’utilisation des tables temporaires et met en évidence les principes des tables temporaires au niveau de la session dans SQL Synapse. 

Tant les ressources du pool SQL dédié que celles du pool SQL serverless peuvent utiliser des tables temporaires. Les limitations liées au pool SQL serverless sont présentées à la fin de cet article. 

## <a name="temporary-tables"></a>tables temporaires ;

Les tables temporaires sont utiles lors du traitement des données, notamment lors d’une transformation lorsque les résultats intermédiaires sont temporaires. Dans SQL Synapse, les tables temporaires existent au niveau de la session.  Elles ne sont visibles que dans la session dans laquelle elles ont été créées. Ainsi, elles sont automatiquement supprimées à la fermeture de cette session. 

## <a name="temporary-tables-in-dedicated-sql-pool"></a>Tables temporaires dans le pool SQL dédié

Dans la ressource de pool SQL dédié, les tables temporaires offrent un gain de performances, car leurs résultats sont écrits en local et non dans un stockage distant.

### <a name="create-a-temporary-table"></a>Créer une table temporaire

Les tables temporaires sont créées en faisant simplement précéder le nom de votre table de `#`.  Par exemple :

```sql
CREATE TABLE #stats_ddl
(
    [schema_name]        NVARCHAR(128) NOT NULL
,    [table_name]            NVARCHAR(128) NOT NULL
,    [stats_name]            NVARCHAR(128) NOT NULL
,    [stats_is_filtered]     BIT           NOT NULL
,    [seq_nmbr]              BIGINT        NOT NULL
,    [two_part_name]         NVARCHAR(260) NOT NULL
,    [three_part_name]       NVARCHAR(400) NOT NULL
)
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
```

Vous pouvez également utiliser `CTAS` pour créer des tables temporaires à l’aide de la même approche :

```sql
CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
,    HEAP
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
;
```

> [!NOTE]
> `CTAS` est une commande puissante et présente l’avantage d’être efficace dans son utilisation de l’espace de journal des transactions. 
> 
> 

### <a name="drop-temporary-tables"></a>Déposer des tables temporaires

Lorsqu’une nouvelle session est créée, aucune table temporaire ne doit exister.  Cela étant, si vous appelez la même procédure stockée qui crée une table temporaire avec le même nom, pour vous assurer de la réussite de vos instructions `CREATE TABLE`, utilisez une simple vérification d’existence préalable avec `DROP` : 

```sql
IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END
```

Pour la cohérence de codage, il convient d’utiliser ce modèle pour les tables et les tables temporaires.  Il est également judicieux d’utiliser `DROP TABLE` pour supprimer les tables temporaires lorsque vous avez fini de les utiliser.  

Dans le développement de procédure stockée, il est courant de voir les commandes de suppression regroupées à la fin d’une procédure pour s’assurer que ces objets sont nettoyés.

```sql
DROP TABLE #stats_ddl
```

### <a name="modularize-code"></a>Modulariser le code

Les tables temporaires peuvent être utilisées partout dans une session utilisateur. Cette fonctionnalité vous permet ensuite de modulariser votre code d’application.  À titre d'exemple, la procédure stockée suivante génère le langage DDL pour mettre à jour toutes les statistiques dans la base de données par nom de statistique.

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_update_stats]
(   @update_type    tinyint -- 1 default 2 fullscan 3 sample 4 resample
    ,@sample_pct     tinyint
)
AS

IF @update_type NOT IN (1,2,3,4)
BEGIN;
    THROW 151000,'Invalid value for @update_type parameter. Valid range 1 (default), 2 (fullscan), 3 (sample) or 4 (resample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
    DROP TABLE #stats_ddl
END

CREATE TABLE #stats_ddl
WITH
(
    DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
        sm.[name]                                                                AS [schema_name]
,        tb.[name]                                                                AS [table_name]
,        st.[name]                                                                AS [stats_name]
,        st.[has_filter]                                                            AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,                                 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,        QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM    sys.objects            AS ob
JOIN    sys.stats            AS st    ON    ob.[object_id]        = st.[object_id]
JOIN    sys.stats_columns    AS sc    ON    st.[stats_id]        = sc.[stats_id]
                                    AND st.[object_id]        = sc.[object_id]
JOIN    sys.columns            AS co    ON    sc.[column_id]        = co.[column_id]
                                    AND    sc.[object_id]        = co.[object_id]
JOIN    sys.tables            AS tb    ON    co.[object_id]        = tb.[object_id]
JOIN    sys.schemas            AS sm    ON    tb.[schema_id]        = sm.[schema_id]
WHERE    1=1
AND        st.[user_created]   = 1
GROUP BY
        sm.[name]
,        tb.[name]
,        st.[name]
,        st.[filter_definition]
,        st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
GO
```

À ce stade, la seule action qui s’est produite est la création d’une procédure stockée qui génère la table temporaire, #stats_ddl.  La procédure stockée supprime #stats_ddl, le cas échéant. Cette suppression permet d'éviter son échec en cas d'exécutions multiples au cours d'une session.  

En l'absence de `DROP TABLE` à la fin de la procédure stockée, lorsque la procédure stockée se termine, la table créée demeure afin de pouvoir être lue en dehors de la procédure stockée.  

Contrairement aux autres bases de données SQL Server, SQL Synapse vous permet d’utiliser la table temporaire en dehors de la procédure qui l’a créée.  Les tables temporaires créées via le pool SQL dédié peuvent être utilisées **partout** dans la session. Dès lors, vous disposerez de plus de code modulaire et gérable, comme illustré dans l’exemple ci-dessous :

```sql
EXEC [dbo].[prc_sqldw_update_stats] @update_type = 1, @sample_pct = NULL;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''

WHILE @i <= @t
BEGIN
    SET @s=(SELECT update_stats_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

### <a name="temporary-table-limitations"></a>Limitations relatives aux tables temporaires

Le pool SQL dédié présente plusieurs limitations d’implémentation pour les tables temporaires :

- Seules les tables temporaires de la session sont prises en charge.  Les tables temporaires globales ne sont pas prises en charge.
- Il n’est pas possible de créer des vues sur des tables temporaires.
- Les tables temporaires peuvent être créées uniquement avec une distribution par hachage ou par tourniquet (round robin).  La distribution de tables temporaires répliquées n’est pas prise en charge. 

## <a name="temporary-tables-in-serverless-sql-pool"></a>Tables temporaires dans un pool SQL serverless

Dans un pool SQL serverless, les tables temporaires sont prises en charge, mais leur utilisation est limitée. Elles ne peuvent pas être utilisées dans le cadre de requêtes ciblant des fichiers. 

Par exemple, vous ne pouvez pas joindre une table temporaire avec des données de fichiers dans le stockage. Le nombre de tables temporaires est limité à 100 et leur taille totale à 100 Mo.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le développement de tables, consultez [Conception de tables à l’aide des ressources SQL Analytics](develop-tables-overview.md).

