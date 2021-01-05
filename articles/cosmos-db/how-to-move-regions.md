---
title: Déplacer un compte Azure Cosmos DB vers une autre région
description: Découvrez comment déplacer un compte Azure Cosmos DB vers une autre région.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 09/12/2020
ms.author: mjbrown
ms.openlocfilehash: 29b5c5d3cf55cd11fe505c0d9ab9b894dc2ad267
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342026"
---
# <a name="move-an-azure-cosmos-db-account-to-another-region"></a>Déplacer un compte Azure Cosmos DB vers une autre région
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment :

- Déplacer une région où les données sont répliquées dans Azure Cosmos DB.
- Migrer les métadonnées et les données d’un compte (Azure Resource Manager) d’une région vers une autre.

## <a name="move-data-from-one-region-to-another"></a>Déplacer des données d’une région vers une autre

Azure Cosmos DB prend en charge la réplication des données en mode natif, ce qui facilite le déplacement de données d’une région vers une autre. Vous pouvez procéder à l’aide du portail Azure, d’Azure PowerShell ou d’Azure CLI. Cela implique les étapes suivantes :

1. Ajouter une nouvelle région au compte

    Pour ajouter une nouvelle région à un compte Azure Cosmos DB, consultez [Ajouter/supprimer des régions dans un compte Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

1. Effectuer un basculement manuel vers la nouvelle région

    Lorsque la région à supprimer est la région d’écriture du compte, vous devez démarrer un basculement vers la nouvelle région ajoutée à l’étape précédente. Il s’agit d’une opération sans temps d’arrêt. Si vous déplacez une région de lecture dans un compte à plusieurs régions, vous pouvez ignorer cette étape. 
    
    Pour démarrer un basculement, consultez [Effectuer un basculement manuel sur un compte Azure Cosmos](how-to-manage-database-account.md#manual-failover).

1. Supprimer la région d’origine

    Pour supprimer une région d’un compte Azure Cosmos DB, consultez [Ajouter/supprimer des régions dans un compte Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account).

## <a name="migrate-azure-cosmos-db-account-metadata"></a>Migrer les métadonnées d’un compte Azure Cosmos DB

Azure Cosmos DB ne prend pas en charge en mode natif la migration des métadonnées d’un compte d’une région vers une autre. Pour migrer les métadonnées du compte et les données client d’une région vers une autre, vous devez créer un nouveau compte dans la région souhaitée, puis copier les données manuellement. 

Une migration quasiment sans temps d’arrêt pour l’API SQL requiert l’utilisation du [flux de modification](change-feed.md) ou d’un outil qui l’utilise. Si vous migrez l’API MongoDB, l’API Cassandra ou une autre API, ou si vous voulez en savoir plus sur les options de migration de données entre comptes, consultez [Options de migration de vos données locales ou cloud vers Azure Cosmos DB](cosmosdb-migrationchoices.md). 

Les étapes suivantes montrent comment migrer un compte Azure Cosmos DB pour l’API SQL et ses données d’une région vers une autre :

1. Créer un nouveau compte Azure Cosmos DB dans la région souhaitée

    Pour créer un nouveau compte via le portail Azure, PowerShell ou Azure CLI, consultez [Créer un compte Azure Cosmos DB](how-to-manage-database-account.md#create-an-account).

1. Créer une nouvelle base de données et un nouveau conteneur

    Pour créer une nouvelle base de données et un nouveau conteneur, consultez [Créer un conteneur Azure Cosmos](how-to-create-container.md).

1. Migrer les données à l’aide de l’outil Azure Cosmos DB Live Data Migrator

    Pour migrer les données quasiment sans temps d’arrêt, consultez [Outil Azure Cosmos DB Live Data Migrator](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator).

1. Mettre à jour la chaîne de connexion de l’application

    Lorsque l’outil Live Data Migrator est toujours en cours d’exécution, mettez à jour les informations de connexion dans le nouveau déploiement de votre application. Vous pouvez récupérer les points de terminaison et les clés de votre application à partir du portail Azure.

    :::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-portal.png" alt-text="Contrôle d’accès dans le portail Azure, illustrant la sécurité de la base de données NoSQL.":::

1. Rediriger les demandes vers la nouvelle application

    Une fois que la nouvelle application est connectée à Azure Cosmos DB, vous pouvez rediriger les demandes des clients vers votre nouveau déploiement.

1. Supprimer les ressources dont vous n’avez plus besoin

    Les demandes étant désormais entièrement redirigées vers la nouvelle instance, vous pouvez supprimer l’ancien compte Azure Cosmos DB et l’outil Live Data Migrator.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations et d’exemples sur la gestion du compte Azure Cosmos ainsi que des bases de données et des conteneurs, consultez les articles suivants :

* [Gérer un compte Azure Cosmos](how-to-manage-database-account.md)
* [Flux de modification dans Azure Cosmos DB](change-feed.md)
