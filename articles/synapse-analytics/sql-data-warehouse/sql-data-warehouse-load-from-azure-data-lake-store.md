---
title: Tutoriel charger des données à partir d’Azure Data Lake Storage
description: Utilisez l’instruction COPY pour charger des données à partir d’Azure Data Lake Storage pour des pools SQL dédiés.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 4886cf6c24e7e96676b9ca5e74b431685cd5a8bb
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452875"
---
# <a name="load-data-from-azure-data-lake-storage-into-dedicated-sql-pools-in-azure-synapse-analytics"></a>Charger des données à partir d’Azure Data Lake Storage dans des pools SQL dédiés d’Azure Synapse Analytics

Ce guide explique comment utiliser [l’instruction COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) pour charger des données à partir d’Azure Data Lake Storage. Pour obtenir des exemples rapides sur l’utilisation de l’instruction COPY pour toutes les méthodes d’authentification, consultez la documentation suivante : [Charger des données de manière sécurisée à l’aide de pools SQL dédiés](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples).

> [!NOTE]  
> Pour nous faire part de vos commentaires ou signaler des problèmes pour l’instruction COPY, envoyez un e-mail à la liste de distribution suivante : sqldwcopypreview@service.microsoft.com.
>
> [!div class="checklist"]
>
> * Créez la table cible pour charger des données à partir de Azure Data Lake Storage.
> * Créez l’instruction COPY pour charger des données dans l’entrepôt de données.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

Avant de commencer ce didacticiel, téléchargez et installez la dernière version de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) (SSMS).

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Un pool SQL dédié. Consultez [Créer un pool SQL dédié et interroger des données](create-data-warehouse-portal.md).
* Un compte Data Lake Storage. Voir [Prise en main d’Azure Data Lake Storage](../../data-lake-store/data-lake-store-get-started-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Pour ce compte de stockage, vous devez configurer ou spécifier l’une des informations d’identification suivantes pour le chargement : une clé de compte de stockage, une clé de signature d’accès partagé (SAS), un utilisateur d’application Azure Directory ou un utilisateur AAD qui a le rôle Azure approprié pour le compte de stockage.

## <a name="create-the-target-table"></a>Créer la table cible

Connectez-vous à votre pool SQL dédié, puis créez la table cible vers laquelle vous allez effectuer le chargement. Dans cet exemple, nous créons une table de dimension product.

```sql
-- A: Create the target table
-- DimProduct
CREATE TABLE [dbo].[DimProduct]
(
    [ProductKey] [int] NOT NULL,
    [ProductLabel] [nvarchar](255) NULL,
    [ProductName] [nvarchar](500) NULL
)
WITH
(
    DISTRIBUTION = HASH([ProductKey]),
    CLUSTERED COLUMNSTORE INDEX
    --HEAP
);
```


## <a name="create-the-copy-statement"></a>Créer l’instruction COPY

Connectez-vous à votre pool SQL dédié, puis exécutez l’instruction COPY. Pour obtenir la liste complète des exemples, consultez la documentation suivante : [Charger des données de manière sécurisée à l’aide de pools SQL dédiés](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples).

```sql
-- B: Create and execute the COPY statement

COPY INTO [dbo].[DimProduct] 
--The column list allows you map, omit, or reorder input file columns to target table columns. 
--You can also specify the default value when there is a NULL value in the file.
--When the column list is not specified, columns will be mapped based on source and target ordinality
(
    ProductKey default -1 1,
    ProductLabel default 'myStringDefaultWhenNull' 2,
    ProductName default 'myStringDefaultWhenNull' 3
)
--The storage account location where you data is staged
FROM 'https://storageaccount.blob.core.windows.net/container/directory/'
WITH 
(
   --CREDENTIAL: Specifies the authentication method and credential access your storage account
   CREDENTIAL = (IDENTITY = '', SECRET = ''),
   --FILE_TYPE: Specifies the file type in your storage account location
   FILE_TYPE = 'CSV',
   --FIELD_TERMINATOR: Marks the end of each field (column) in a delimited text (CSV) file
   FIELDTERMINATOR = '|',
   --ROWTERMINATOR: Marks the end of a record in the file
   ROWTERMINATOR = '0x0A',
   --FIELDQUOTE: Specifies the delimiter for data of type string in a delimited text (CSV) file
   FIELDQUOTE = '',
   ENCODING = 'UTF8',
   DATEFORMAT = 'ymd',
   --MAXERRORS: Maximum number of reject rows allowed in the load before the COPY operation is canceled
   MAXERRORS = 10,
   --ERRORFILE: Specifies the directory where the rejected rows and the corresponding error reason should be written
   ERRORFILE = '/errorsfolder',
) OPTION (LABEL = 'COPY: ADLS tutorial');
```

## <a name="optimize-columnstore-compression"></a>Optimiser la compression columnstore

Par défaut, les tables sont définies en tant qu’index cluster columnstore. Après un chargement, certaines lignes de données peuvent ne pas être compressées dans le columnstore.  Cela peut être dû à diverses raisons. Pour plus d’informations, consultez [Gérer les index Columnstore](sql-data-warehouse-tables-index.md).

Pour optimiser les performances des requêtes et la compression du columnstore après un chargement, reconstruisez la table afin de forcer l’index columnstore à compresser toutes les lignes.

```sql

ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD;

```

## <a name="optimize-statistics"></a>Optimiser les statistiques

Il est préférable de créer des statistiques sur une colonne immédiatement après un chargement. Les statistiques offrent plusieurs possibilités. Par exemple, si vous créez des statistiques sur une colonne pour chaque colonne, il faudra peut-être beaucoup de temps pour reconstruire toutes les statistiques. S’il est certain que des colonnes ne se trouveront pas dans les prédicats de requête, vous pouvez ignorer la création des statistiques sur ces colonnes.

Si vous décidez de créer des statistiques sur une colonne pour chaque colonne de chaque table, vous pouvez utiliser l’exemple de code de procédure stockée `prc_sqldw_create_stats` dans l’article portant sur les [statistiques](sql-data-warehouse-tables-statistics.md).

L’exemple suivant est un bon point de départ pour la création de statistiques. Il permet de créer des statistiques sur une colonne pour chaque colonne de la table de dimension, et chaque colonne de jointure des tables de faits. Vous pouvez toujours ajouter ultérieurement des statistiques sur une ou plusieurs colonnes dans d’autres colonnes de table de faits.

## <a name="achievement-unlocked"></a>Et voilà !

Vous avez correctement chargé les données dans votre entrepôt de données. Bon travail !

## <a name="next-steps"></a>Étapes suivantes
Le chargement des données est la première étape du développement d’une solution d’entreposage des données à l’aide d’Azure Synapse Analytics. Découvrez nos ressources de développement.

> [!div class="nextstepaction"]
> [Découvrir comment développer des tables pour l’entreposage de données](sql-data-warehouse-tables-overview.md)

Pour obtenir plus d’exemples et de références sur le chargement, consultez la documentation suivante :
- [Documentation de référence sur l’instruction COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest#syntax)
- [Exemples COPY pour chaque méthode d’authentification](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql-examples)
- [Démarrage rapide : Instruction COPY pour une table unique](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql)
