---
title: Fonctionnalités prises en charge par Azure Synapse Link (préversion) pour Azure Cosmos DB
description: Présentation de la liste des actions prises en charge par Azure Synapse Link pour Azure Cosmos DB
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: synapse-link
ms.date: 09/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 3d12e60e2aeee667b89d56589870d53366772014
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459043"
---
# <a name="azure-synapse-link-for-azure-cosmos-db-supported-features"></a>Fonctionnalités prises en charge par Azure Synapse Link pour Azure Cosmos DB

Cet article décrit les fonctionnalités actuellement prises en charge par Azure Synapse Link pour Azure Cosmos DB.

## <a name="azure-synapse-support"></a>Prise en charge d’Azure Synapse

Il existe deux types de conteneurs dans Azure Cosmos DB :
* Le conteneur HTAP : il s’agit d’un conteneur prenant en charge Synapse Link. Ce conteneur a à la fois un magasin transactionnel et un magasin analytique. 
* Conteneur OLTP : conteneur ne prenant pas en charge Synapse Link. Ce conteneur dispose uniquement d’un magasin transactionnel et n’a aucun magasin analytique.

> [!IMPORTANT]
> Azure Synapse Link pour Azure Cosmos DB est actuellement pris en charge dans les espaces de travail Synapse où le réseau virtuel managé n’est pas activé. 

Vous pouvez vous connecter à un conteneur Azure Cosmos DB sans activer Synapse Link. Dans ce scénario, vous pouvez uniquement lire et écrire des données dans le magasin transactionnel. La liste ci-dessous répertorie les fonctionnalités qui sont actuellement prises en charge dans Synapse Link pour Azure Cosmos DB. 

| Category              | Description |[Pool Apache Spark](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) | [Pool SQL serverless](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- | ----------------------------------------------------------- |
| **Prise en charge à l’exécution** |Runtime Azure Synapse pris en charge pour accéder à Azure Cosmos DB| ✓ | PRÉVERSION |
| **Prise en charge de l’API Azure Cosmos DB** : | Type d’API Azure Cosmos DB pris en charge | SQL / MongoDB | SQL / MongoDB |
| **Object**  |Possibilité de créer des objets (tels qu’une table), qui pointent directement vers le conteneur Azure Cosmos DB| DataFrame, Affichage, Table | Affichage |
| **Lire**    | Type de conteneur Azure Cosmos DB qui peut être lu | OLTP / HTAP | HTAP  |
| **Écrire**   | Le runtime Azure Synapse peut-il être utilisé pour écrire des données dans un conteneur Azure Cosmos DB | Oui | Non |

* Si vous écrivez des données dans un conteneur Azure Cosmos DB à partir de Spark, ce processus s’effectuera via le magasin transactionnel d’Azure Cosmos DB. Ce processus aura un impact sur les performances transactionnelles d’Azure Cosmos DB, puisqu’il consommera des unités de requête.
* L’intégration d’un pool SQL dédié par le biais de tables externes n’est pas prise en charge actuellement.
 
## <a name="supported-code-generated-actions-for-spark"></a>Actions générées par le code prises en charge pour Spark

| Mouvement              | Description |OLTP |HTAP  |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Charger dans un DataFrame** |Charger et lire des données dans un DataFrame Spark |✓| ✓ |
| **Créer une table Spark** |Créer une table pointant vers un conteneur Azure Cosmos DB|✓| ✓ |
| **Écrire un DataFrame dans un conteneur** |Écrire des données dans un conteneur|✓| ✓ |
| **Charger un DataFrame de streaming à partir d’un conteneur** |Streamer des données à l’aide du flux de modification Azure Cosmos DB|✓| ✓ |
| **Écrire un DataFrame de streaming dans le conteneur** |Streamer des données à l’aide du flux de modification Azure Cosmos DB|✓| ✓ |


## <a name="supported-code-generated-actions-for-serverless-sql-pool"></a>Actions générées par le code prises en charge pour un pool SQL serverless

| Mouvement              | Description |OLTP |HTAP |
| -------------------- | ----------------------------------------------------------- |----------------------------------------------------------- |----------------------------------------------------------- |
| **Explorer des données** |Explorer des données d’un conteneur avec une syntaxe T-SQL familière et une inférence de schéma automatique|X| ✓ |
| **Créer des vues et générer des rapports BI** |Créer une vue SQL pour avoir un accès direct à un conteneur pour BI par le biais d’un pool SQL serverless |X| ✓ |
| **Joindre des sources de données disparates et des données Cosmos DB** | Stocker les résultats de la requête de lecture des données à partir de conteneurs Cosmos DB avec les données dans le stockage Blob Azure ou Azure Data Lake Storage à l’aide de CETAS |X| ✓ |

## <a name="next-steps"></a>Étapes suivantes

* [Découvrir comment se connecter à Synapse Link pour Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md)
* [Découvrir comment interroger le magasin analytique Cosmos DB avec Spark](how-to-query-analytical-store-spark.md)
