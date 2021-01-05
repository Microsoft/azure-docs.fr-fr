---
title: Meilleures pratiques en matière de chargement des données pour les pools SQL dédiés
description: Recommandations et optimisation des performances pour le chargement de données à l’aide de pools SQL dédiés dans Azure Synapse Analytics.
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
ms.openlocfilehash: 39625914f179dfc8d5511b9a3d386cc8332b7efa
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456307"
---
# <a name="best-practices-for-loading-data-using-dedicated-sql-pools-in-azure-synapse-analytics"></a>Meilleures pratiques en matière de chargement de données à l’aide de pools SQL dédiés dans Azure Synapse Analytics

Dans cet article, vous allez découvrir des recommandations et des optimisations des performances pour le chargement de données à l’aide du pool SQL dédié.

## <a name="preparing-data-in-azure-storage"></a>Préparation des données dans Stockage Azure

Pour réduire la latence, colocalisez votre couche de stockage et votre pool SQL dédié.

Lorsque vous exportez des données dans un format de fichier ORC, vous pouvez obtenir des erreurs Java de mémoire insuffisante lorsqu’il existe des colonnes de texte de grande taille. Pour contourner cette limitation, exportez uniquement un sous-ensemble de ces colonnes.

Tous les formats de fichier présentent des caractéristiques de performances différentes. Pour un chargement plus rapide, utilisez des fichiers texte délimités compressés. La différence entre les performances des formats UTF-8 et UTF-16 est minime.

Fractionnez les fichiers compressés volumineux en plusieurs petits fichiers compressés.

## <a name="running-loads-with-enough-compute"></a>Exécution de charges avec suffisamment de ressources de calcul

Pour une vitesse de chargement plus élevée, exécutez un seul travail de chargement à la fois. Si ce n’est pas possible, exécutez simultanément un nombre minimal de charges. Si vous prévoyez un travail de chargement volumineux, envisagez d’effectuer un scale-up de votre pool SQL dédié avant le chargement.

Pour exécuter des charges avec des ressources de calcul appropriées, créez des utilisateurs de chargement désignés pour cette tâche. Classez chaque utilisateur de chargement sur un groupe de charge de travail spécifique. Pour exécuter une charge, connectez-vous en tant qu’utilisateur de chargement, puis exécutez la charge. La charge s’exécute avec le groupe de charge de travail de l’utilisateur.  

### <a name="example-of-creating-a-loading-user"></a>Exemple de création d’un utilisateur de chargement

Cet exemple crée un utilisateur de chargement classé dans un groupe de charge de travail spécifique. La première étape consiste à **se connecter au maître** et à créer une connexion.

```sql
   -- Connect to master
   CREATE LOGIN loader WITH PASSWORD = 'a123STRONGpassword!';
```

Connectez-vous au pool SQL dédié et créez un utilisateur. Le code suivant suppose que vous êtes connecté à la base de données appelée mySampleDataWarehouse. Il montre comment créer un utilisateur appelé chargeur et donne aux utilisateurs les autorisations nécessaires pour créer des tables et charger à l’aide de l’[instruction COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest). Ensuite, il classe l’utilisateur dans le groupe de charge de travail DataLoads avec les ressources maximales. 

```sql
   -- Connect to the dedicated SQL pool
   CREATE USER loader FOR LOGIN loader;
   GRANT ADMINISTER DATABASE BULK OPERATIONS TO loader;
   GRANT INSERT ON <yourtablename> TO loader;
   GRANT SELECT ON <yourtablename> TO loader;
   GRANT CREATE TABLE TO loader;
   GRANT ALTER ON SCHEMA::dbo TO loader;
   
   CREATE WORKLOAD GROUP DataLoads
   WITH ( 
      MIN_PERCENTAGE_RESOURCE = 100
       ,CAP_PERCENTAGE_RESOURCE = 100
       ,REQUEST_MIN_RESOURCE_GRANT_PERCENT = 100
    );

   CREATE WORKLOAD CLASSIFIER [wgcELTLogin]
   WITH (
         WORKLOAD_GROUP = 'DataLoads'
       ,MEMBERNAME = 'loader'
   );
```

Pour exécuter une charge avec des ressources pour le groupe de charge de travail de chargement, connectez-vous en tant que chargeur et exécutez la charge.

## <a name="allowing-multiple-users-to-load-polybase"></a>Autoriser le chargement par plusieurs utilisateurs (PolyBase)

Il est souvent nécessaire que plusieurs utilisateurs chargent des données dans un pool SQL dédié. Le chargement avec [CREATE TABLE AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) [PolyBase] nécessite des autorisations CONTROL de la base de données.  L’autorisation CONTROL permet de contrôler l’accès à tous les schémas.

Vous ne souhaiterez peut-être pas attribuer aux utilisateurs de chargement le contrôle d’accès sur tous les schémas. Pour limiter les autorisations, utilisez l’instruction DENY CONTROL.

Par exemple, les schémas de base de données schema_A pour le service A et schema_B pour le service B laissent les utilisateurs de base de données user_A et user_B être utilisateurs du chargement PolyBase dans les services A et B, respectivement. Ils ont tous deux reçu des autorisations de base de données CONTROL. Les créateurs des schémas A et B verrouillent maintenant leurs schémas à l’aide de DENY :

```sql
   DENY CONTROL ON SCHEMA :: schema_A TO user_B;
   DENY CONTROL ON SCHEMA :: schema_B TO user_A;
```

User_A et user_B ne doivent maintenant plus avoir accès au schéma de l’autre service.

## <a name="loading-to-a-staging-table"></a>Chargement dans une table de mise en lots

Pour atteindre la vitesse de chargement la plus élevée pour le déplacement des données dans une table du pool SQL dédié, chargez les données dans une table de mise en lots.  Définissez la table de mise en lots comme segment de mémoire et utilisez le tourniquet (round-robin) pour l’option de distribution.

Considérez que le chargement est généralement un processus en deux étapes, dans lequel vous effectuez tout d’abord le chargement dans une table de mise en lots puis insérez les données dans une table du pool SQL dédié de production. Si la table de production utilise une distribution par hachage, la durée totale pour le chargement et l’insertion peut être plus rapide si vous définissez la table de mise en lots avec la distribution par hachage.

Le chargement dans la table de mise en lots prend plus de temps, mais la deuxième étape d’insertion des lignes dans la table de production n’entraîne pas de déplacement des données dans les distributions.

## <a name="loading-to-a-columnstore-index"></a>Chargement dans un index columnstore

Les index columnstore ont besoin de beaucoup de mémoire pour compresser les données dans des rowgroups de haute qualité. Pour une meilleure compression et un index plus efficace, l’index columnstore doit compresser au maximum 1 048 576 lignes dans chaque rowgroup.

En cas de sollicitation de la mémoire, l’index columnstore peut ne pas être en mesure d’atteindre les taux de compression maximum. Ce scénario, à son tour, affecte les performances des requêtes. Pour une présentation approfondie, consultez [Columnstore memory optimizations](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md) (Optimisation de mémoire Columstore).

- Pour garantir que l’utilisateur de chargement dispose de suffisamment de mémoire pour atteindre des taux de compression maximum, utilisez des utilisateurs de chargement qui sont membres d’une classe de ressources moyenne ou grande.
- Chargez suffisamment de lignes pour remplir complètement de nouveaux rowgroups. Lors d’un chargement en masse, les 1 048 576 lignes sont compressées directement dans le columnstore en tant que groupe de lignes complet. Les charges de moins de 102 400 lignes envoient les lignes dans le deltastore, où les lignes sont conservées dans un index b-tree.

> [!NOTE]
> Si vous chargez trop peu de lignes, elles risquent d’être toutes routées vers le deltastore et de ne pas être compressées immédiatement au format columnstore.

## <a name="increase-batch-size-when-using-sqlbulkcopy-api-or-bcp"></a>Augmenter la taille de lot lors de l’utilisation de l’API SqLBulkCopy ou de bcp

Le chargement avec l’instruction COPY fournit le débit le plus élevé avec les pools SQL dédiés. Si vous ne pouvez pas utiliser l’instruction COPY pour charger et que vous devez utiliser l’[API SQLBulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) ou [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), songez à augmenter la taille de lot pour bénéficier d’un meilleur débit.

> [!TIP]
> Une taille de lot comprise entre 100 000 et 1 million de lignes est la base de référence recommandée pour déterminer la capacité de taille de lot optimale.

## <a name="handling-loading-failures"></a>Gestion des échecs de chargement

Une charge qui utilise une table externe peut échouer avec l’erreur suivante : *« Requête abandonnée : le seuil de rejet maximal a été atteint durant la lecture d’une source externe »* . Ce message indique que vos données externes contiennent des enregistrements à l’intégrité compromise.

L’intégrité d’un enregistrement de données est considérée comme compromise si l’une des conditions suivantes est remplie :

- Les types de données et le nombre de colonnes ne correspondent pas aux définitions de colonne de la table externe.
- Les données ne sont pas conformes au format de fichier externe spécifié.

Pour corriger les enregistrements compromis, assurez-vous que les définitions de format de votre table externe et de votre fichier externe sont correctes et que vos données externes sont conformes à ces définitions.

Si un sous-ensemble d’enregistrements de données externes est compromis, vous pouvez choisir de rejeter ces enregistrements pour vos requêtes en utilisant les options de rejet dans [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="inserting-data-into-a-production-table"></a>Insertion de données dans une table de production

Une charge unique dans une petite table à l’aide d’une instruction [INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) ou même un rechargement périodique d’une recherche peut suffire avec une instruction comme `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  Cependant, des insertions uniques ne sont pas aussi efficaces qu’un chargement en masse.

Si vous avez au minimum plusieurs milliers d’insertions uniques pendant la journée, regroupez les insertions pour les charger en masse.  Développez votre processus pour ajouter les insertions uniques à un fichier, puis créez un autre processus qui charge régulièrement le fichier.

## <a name="creating-statistics-after-the-load"></a>Création de statistiques après le chargement

Pour améliorer les performances de vos requêtes, il est important de créer les statistiques sur toutes les colonnes de toutes les tables après le premier chargement ou après toute modification substantielle dans les données. Vous pouvez effectuer manuellement la création de statistiques ou vous pouvez activer [AUTO_CREATE_STATISTICS](sql-data-warehouse-tables-statistics.md#automatic-creation-of-statistic).

Pour plus d’informations sur les statistiques, voir [Statistiques](sql-data-warehouse-tables-statistics.md). L’exemple suivant montre comment créer manuellement des statistiques sur cinq colonnes de la table Customer_Speed.

```sql
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## <a name="rotate-storage-keys-polybase"></a>Passer d’une clé de stockage à une autre (PolyBase)

En matière de sécurité, il est recommandé de modifier régulièrement la clé d’accès à votre stockage d’objets blob. Vous avez deux clés de stockage pour votre compte de stockage blob, ce qui vous permet de passer d’une clé à l’autre.

Pour passer d’une clé de compte de stockage Azure à une autre :

Pour chaque compte de stockage dont la clé a changé, exécutez l’instruction [ALTER DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/alter-database-scoped-credential-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Exemple :

Une clé d’origine est créée

```sql
CREATE DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key1'
```

Passer de la clé 1 à la clé 2

```sql
ALTER DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key2'
```

Aucune autre modification des sources de données externes sous-jacentes n’est nécessaire.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’instruction COPY ou PolyBase lors de la conception d’un processus d’extraction, de chargement et transformation (ELT), consultez [Conception d’un processus d’extraction, de chargement et de transformation (ELT) pour Azure Synapse Analytics](design-elt-data-loading.md).
- Pour un didacticiel sur le chargement, consultez [Utiliser l’instruction COPY pour charger des données du Stockage Blob Azure vers Synapse SQL](load-data-from-azure-blob-storage-using-polybase.md).
- Pour surveiller les charges de données, consultez [Surveiller votre charge de travail à l’aide de vues de gestion dynamique](sql-data-warehouse-manage-monitor.md).
