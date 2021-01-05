---
title: Bonnes pratiques pour les pools SQL serverless
description: Recommandations et meilleures pratiques pour l’utilisation d’un pool SQL serverless.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 05/01/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: b8b93471b6d7f2555cfd71e524718ed0ea1ee191
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96457903"
---
# <a name="best-practices-for-serverless-sql-pool-in-azure-synapse-analytics"></a>Meilleures pratiques pour un pool SQL serverless dans Azure Synapse Analytics

Cet article présente un ensemble de meilleures pratiques pour l’utilisation d’un pool SQL serverless. Un pool SQL serverless est une ressource dans Azure Synapse Analytics.

## <a name="general-considerations"></a>Considérations d’ordre général

Un pool SQL serverless vous permet d’interroger des fichiers dans vos comptes de stockage Azure. Il ne dispose pas de capacités de stockage ou d’ingestion locales. Cela qui signifie que tous les fichiers que la requête cible sont externes au pool SQL serverless. Tout ce qui est lié à la lecture de fichiers à partir du stockage peut avoir un impact sur les performances des requêtes.

## <a name="colocate-your-azure-storage-account-and-serverless-sql-pool"></a>Colocaliser votre compte de stockage Azure et le pool SQL serverless

Pour réduire la latence, colocalisez votre compte de stockage Azure et le point de terminaison de votre pool SQL serverless. Les comptes de stockage et les points de terminaison approvisionnés lors de la création de l’espace de travail se trouvent dans la même région.

Pour optimiser les performances, si vous accédez à d’autres comptes de stockage avec le pool SQL serverless, assurez-vous qu’ils se trouvent dans la même région. Autrement, vous obtiendrez une latence accrue pour le transfert réseau des données entre la région distante et la région du point de terminaison.

## <a name="azure-storage-throttling"></a>Limitation de Stockage Azure

Plusieurs applications et services peuvent accéder à votre compte de stockage. Lorsque les IOPS ou le débit combinés générés par des applications, des services et la charge de travail du pool SQL serverless dépassent les limites du compte de stockage, une limitation du stockage se produit. Celle-ci a un impact significatif sur les performances des requêtes.

Quand une limitation est détectée, le pool SQL serverless dispose d’une fonction de gestion intégrée pour résoudre le problème. Le pool SQL serverless ralentit la fréquence des demandes adressées au stockage jusqu’à ce que la limitation soit résolue.

> [!TIP]
> Pour optimiser l’exécution des requêtes, évitez de solliciter le compte de stockage avec d’autres charges de travail lors de l’exécution de la requête.

## <a name="prepare-files-for-querying"></a>Préparer les fichiers pour l’interrogation

Si possible, vous pouvez préparer les fichiers pour améliorer les performances :

- Convertissez les formats CSV et JSON en Parquet. Parquet est un format en colonnes. Comme il est compressé, ses fichiers sont de plus petite taille que des fichiers CSV ou JSON contenant les mêmes données. Le pool SQL serverless a besoin de moins de temps et de demandes de stockage pour le lire.
- Si une requête cible un seul fichier volumineux, il est avantageux de fractionner celui-ci en fichiers plus petits.
- Essayez de conserver une taille de fichier CSV inférieure à 10 Go.
- Il est préférable d’avoir des fichiers de taille identique pour un chemin d’accès OPENROWSET unique ou un emplacement de table externe.
- Partitionnez vos données en stockant les partitions dans des dossiers ou sous des noms de fichiers différents. Consultez [Utiliser les fonctions filename et filepath pour cibler des partitions spécifiques](#use-filename-and-filepath-functions-to-target-specific-partitions).

## <a name="push-wildcards-to-lower-levels-in-the-path"></a>Envoyer (push) des caractères génériques à des niveaux inférieurs dans le chemin d’accès

Vous pouvez utiliser des caractères génériques dans votre chemin pour [interroger plusieurs fichiers et dossiers](query-data-storage.md#query-multiple-files-or-folders). Le pool SQL serverless répertorie les fichiers figurant dans votre compte de stockage, à partir du premier caractère générique (*), en utilisant l’API de stockage. Il élimine les fichiers qui ne correspondent pas au chemin d’accès spécifié. La réduction de la liste de fichiers initiale peut améliorer les performances si de nombreux fichiers correspondent au chemin spécifié jusqu’au premier caractère générique.

## <a name="use-appropriate-data-types"></a>Utiliser les types de données appropriés

Les types de données que vous utilisez dans votre requête affectent les performances. Vous pouvez obtenir de meilleures performances en suivant les instructions suivantes : 

- Utilisez la plus petite taille de données pouvant prendre en charge la plus grande valeur possible.
  - Si la longueur maximale est de 30 caractères, utilisez un type de données d’une longueur de 30 caractères.
  - Si toutes les valeurs de colonne de caractères ont une taille fixe, utilisez le type **char** ou **nchar**. Sinon, utilisez le type **varchar** ou **nvarchar**.
  - Si la valeur maximale de la colonne d’entiers est 500, utilisez le type **smallint**, car il s’agit du plus petit type de données pouvant prendre en charge cette valeur. Vous trouverez les plages de types d’entiers dans [cet article](/sql/t-sql/data-types/int-bigint-smallint-and-tinyint-transact-sql?view=azure-sqldw-latest&preserve-view=true).
- Autant que possible, utilisez les types **varchar** et **char** plutôt que les types **nvarchar** et **nchar**.
- Utilisez des types de données de type Integer si possible. Les opérations de tri (SORT), de jointure (JOIN) et de regroupement (GROUP BY) sont effectuées plus rapidement sur des entiers que sur des données caractères.
- Si vous utilisez une inférence de schéma, [vérifiez les types de données déduits](#check-inferred-data-types).

## <a name="check-inferred-data-types"></a>Vérifier les types de données déduits

Une [inférence de schéma](query-parquet-files.md#automatic-schema-inference) vous permet d’écrire rapidement des requêtes et d’explorer des données sans connaître le schéma de fichier. Le coût de ce confort est que les types de données inférés peuvent être plus volumineux que les types de données réels. Cela se produit quand il n’y a pas assez d’informations dans les fichiers sources pour garantir que le type de données approprié est utilisé. Par exemple, des fichiers Parquet ne contiennent pas de métadonnées sur la longueur maximale des colonnes de caractères. Le pool SQL serverless en déduit qu’il s’agit d’une valeur varchar(8000).

Pour vérifier les types de données résultant de votre requête, vous pouvez utiliser la commande [sp_describe_first_results_set](/sql/relational-databases/system-stored-procedures/sp-describe-first-result-set-transact-sql?view=sql-server-ver15&preserve-view=true).

L’exemple suivant montre comment vous pouvez optimiser les types de données déduits. Cette procédure permet d’afficher les types de données déduits : 
```sql  
EXEC sp_describe_first_result_set N'
    SELECT
        vendor_id, pickup_datetime, passenger_count
    FROM 
        OPENROWSET(
            BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/*/*/*'',
            FORMAT=''PARQUET''
        ) AS nyc';
```

Voici le jeu de résultats :

|is_hidden|column_ordinal|name|system_type_name|max_length|
|----------------|---------------------|----------|--------------------|-------------------||
|0|1|vendor_id|varchar(8000)|8000|
|0|2|pickup_datetime|datetime2(7)|8|
|0|3|passenger_count|int|4|

Une fois que vous connaissez les types de données déduits pour la requête, vous pouvez spécifier les types de données appropriés :

```sql  
SELECT
    vendor_id, pickup_datetime, passenger_count
FROM 
    OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/*/*/*',
        FORMAT='PARQUET'
    ) 
    WITH (
        vendor_id varchar(4), -- we used length of 4 instead of the inferred 8000
        pickup_datetime datetime2,
        passenger_count int
    ) AS nyc;
```

## <a name="use-filename-and-filepath-functions-to-target-specific-partitions"></a>Utiliser les fonctions filename et filepath pour cibler des partitions spécifiques

Les données sont souvent organisées en partitions. Vous pouvez demander au pool SQL serverless d’interroger des dossiers et fichiers particuliers. Cela contribue à réduire le nombre de fichiers et la quantité de données que la requête doit lire et traiter. En prime, vous obtiendrez de meilleures performances.

Pour plus d’informations, consultez les fonctions [filename](query-data-storage.md#filename-function) et [filepath](query-data-storage.md#filepath-function), ainsi que les exemples d’[interrogation de fichiers spécifiques](query-specific-files.md).

> [!TIP]
> Castez toujours les résultats des fonctions filepath et filename vers les types de données appropriés. Si vous utilisez des types de données caractères, veillez à utiliser la longueur appropriée.

> [!NOTE]
> Les fonctions utilisées pour l’élimination de partition, filepath et filename, ne sont actuellement pas prises en charge pour des tables externes autres que celles créées automatiquement pour chaque table créée dans Apache Spark pour Azure Synapse Analytics.

Si vos données stockées ne sont pas partitionnées, envisagez de les partitionner. De cette façon, vous pourrez utiliser ces fonctions pour optimiser les requêtes ciblant ces fichiers. Lorsque vous [interrogez des tables partitionnées Apache Spark pour Azure Synapse](develop-storage-files-spark-tables.md) à partir du pool SQL serverless, automatiquement, la requête ne cible que les fichiers nécessaires.

## <a name="use-parser_version-20-to-query-csv-files"></a>Utiliser PARSER_VERSION 2.0 pour interroger des fichiers CSV

Vous pouvez utiliser l’analyseur à performances optimisées lors de l’interrogation de fichiers CSV. Pour plus d’informations, consultez [PARSER_VERSION](develop-openrowset.md).

## <a name="manually-create-statistics-for-csv-files"></a>Créer manuellement des statistiques pour les fichiers CSV

Le pool SQL serverless s’appuie sur des statistiques pour générer des plans optimaux d’exécution des requêtes. Les statistiques sont automatiquement créées pour les colonnes de fichiers Parquet si nécessaire. À ce stade, les statistiques ne sont pas automatiquement créées pour les colonnes de fichiers CSV, et vous devez créer des statistiques manuellement pour les colonnes que vous utilisez dans les requêtes, en particulier celles utilisées dans DISTINCT, JOIN, WHERE, ORDER BY et GROUP BY. Pour plus d’informations, consultez [statistiques du pool SQL serverless] (develop-tables-statistics.md#statistics-in-serverless-sql-pool).

## <a name="use-cetas-to-enhance-query-performance-and-joins"></a>Utiliser CETAS pour améliorer les performances des requêtes et les jointures

[CETAS](develop-tables-cetas.md) est l’une des fonctionnalités les plus importantes disponibles dans le pool SQL serverless. CETAS est une opération parallèle qui crée des métadonnées de table externe et exporte le résultat de la requête SELECT vers un ensemble de fichiers dans votre compte de stockage.

Vous pouvez utiliser CETAS pour stocker dans un nouveau jeu de fichiers des parties de requêtes souvent utilisées, telles des tables de référence jointes. Vous pouvez ensuite établir une jointure avec cette table externe au lieu de répéter des jointures communes dans plusieurs requêtes.

Comme CETAS génère des fichiers Parquet, les statistiques sont automatiquement créées lorsque la première requête cible cette table externe, ce qui améliore les performances pour les requêtes suivantes ciblant un tableau généré avec CETAS.

## <a name="azure-ad-pass-through-performance"></a>Performances d’authentification directes Azure AD

Le pool SQL serverless vous permet d’accéder à des fichiers dans le stockage en utilisant une authentification pass-through Azure Active Directory (Azure AD) ou des informations d’identification SAP. Il se peut que vous constatiez que l’authentification directe Azure AD est moins performante que la technologie SAP.

Si vous avez besoin de meilleures performances, essayez d’utiliser des informations d’identification SAP pour accéder au stockage en attendant que les performances d’authentification directe Azure AD soient améliorées.

## <a name="next-steps"></a>Étapes suivantes

Pour trouver des solutions aux problèmes courants, consultez l’article sur la [résolution des problèmes](../sql-data-warehouse/sql-data-warehouse-troubleshoot.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Si vous utilisez un pool SQL dédié plutôt qu’un pool SQL serverless, consultez [Meilleures pratiques pour les pools SQL dédiés](best-practices-sql-pool.md) afin d’obtenir des instructions spécifiques.
