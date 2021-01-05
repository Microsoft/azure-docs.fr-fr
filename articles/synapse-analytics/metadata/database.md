---
title: Base de données partagée
description: Azure Synapse Analytics fournit un modèle de métadonnées partagées avec lequel la création d’une base de données dans le pool Apache Spark serverless rend celle-ci accessible depuis ses moteurs de pool SQL serverless et pool SQL.
services: synapse-analytics
author: MikeRys
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: metadata
ms.date: 05/01/2020
ms.author: mrys
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
ms.openlocfilehash: 14ae8b8670db53761d085bc019711e829633601b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451811"
---
# <a name="azure-synapse-analytics-shared-database"></a>Base de données partagée Azure Synapse Analytics

Azure Synapse Analytics permet aux différents moteurs de calcul d’espace de travail de partager des bases de données et des tables entre ses pools Apache Spark serverless et le moteur de son pool SQL serverless.

Une base de données créée avec un travail Spark devient visible avec le même nom pour tous les pools Spark actuels et futurs dans l’espace de travail, y compris le moteur du pool SQL serverless.

La base de données Spark par défaut, nommée `default`, est également visible dans le contexte SQL du pool SQL serverless sous le nom `default`.

Comme les bases de données sont synchronisées avec le pool SQL serverless de façon asynchrone, elles apparaissent après un certain délai.

## <a name="manage-a-spark-created-database"></a>Gérer une base de données créée avec Spark

Utilisez Spark pour gérer les bases de données créées avec Spark. Par exemple, supprimez-les par le biais d’un travail de pool Spark et créez-y des tables à partir de Spark.

Si vous créez des objets dans une base de données créée avec Spark en utilisant le pool SQL serverless ou si vous essayez de supprimer la base de données, l’opération réussit, mais la base de données Spark d’origine ne sera pas modifiée.

## <a name="how-name-conflicts-are-handled"></a>Gestion des conflits de noms

Si le nom d’une base de données Spark est en conflit avec le nom d’une base de données du pool SQL serverless, un suffixe est ajouté à la base de données Spark dans le pool SQL serverless. Le suffixe dans le pool SQL serverless est `_<workspace name>-ondemand-DefaultSparkConnector`.

Par exemple, si une base de données Spark nommée `mydb` est créée dans l’espace de travail Azure Synapse `myws` et qu’une base de données du pool SQL serverless portant ce nom existe déjà, la base de données Spark dans le pool SQL serverless doit être référencée sous le nom `mydb_myws-ondemand-DefaultSparkConnector`.

> [!CAUTION]
> Attention : Vous ne devez pas créer de dépendance envers ce comportement.

## <a name="security-model"></a>Modèle de sécurité

Les bases de données et les tables Spark, ainsi que leurs représentations synchronisées dans le moteur SQL, sont sécurisées au niveau du stockage sous-jacent.

Le principal de sécurité qui crée une base de données est considéré comme le propriétaire de cette base de données, et dispose de tous les droits sur la base de données et ses objets.

Pour donner à un principal de sécurité, tel qu’un utilisateur ou un groupe de sécurité, l’accès à une base de données, fournissez les autorisations de fichier et de dossier POSIX appropriées aux dossiers et fichiers sous-jacents dans le répertoire `warehouse`. 

Par exemple, pour qu’un principal de sécurité puisse lire une table dans une base de données, les autorisations `X` et `R` doivent lui être attribuées pour tous les dossiers à partir du dossier de la base de données dans le répertoire `warehouse`. En outre, les fichiers (tels que les fichiers de données sous-jacents de la table) nécessitent des autorisations `R`. 

Si un principal de sécurité nécessite la capacité à créer ou à supprimer des objets dans une base de données, des autorisations `W` supplémentaires sont requises sur les dossiers et les fichiers du dossier `warehouse`.

## <a name="examples"></a>Exemples

### <a name="create-and-connect-to-spark-database-with-serverless-sql-pool"></a>Créer une base de données Spark et s’y connecter avec le pool SQL serverless

Commencez par créer une base de données Spark nommée `mytestdb` à l’aide d’un cluster Spark que vous avez déjà créé dans votre espace de travail. Pour cela, vous pouvez par exemple utiliser un notebook C# Spark avec l’instruction .NET pour Spark suivante :

```csharp
spark.Sql("CREATE DATABASE mytestdb")
```

Après un bref délai, vous pouvez voir la base de données depuis le pool SQL serverless. Exécutez par exemple l’instruction suivante à partir du pool SQL serverless.

```sql
SELECT * FROM sys.databases;
```

Vérifiez que `mytestdb` est inclus dans les résultats.

## <a name="next-steps"></a>Étapes suivantes

- [Apprenez-en davantage sur les métadonnées partagées Azure Synapse Analytics](overview.md)
- [Apprenez-en davantage sur les tables de métadonnées partagées Azure Synapse Analytics](table.md)
