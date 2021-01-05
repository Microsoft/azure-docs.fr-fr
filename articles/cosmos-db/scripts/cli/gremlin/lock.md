---
title: Créer un verrou de ressource pour une base de données et un graphe Gremlin pour Azure Cosmos DB
description: Créer un verrou de ressource pour une base de données et un graphe Gremlin pour Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 492246b5dfb19664ea54ce8b5462c7d77f8d951b
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94562710"
---
# <a name="create-a-resource-lock-for-azure-cosmos-gremlin-api-database-and-graph-using-azure-cli"></a>Créer un verrou de ressource pour une base de données et un graphe d’API Gremlin Azure Cosmos DB à l’aide d’Azure CLI
[!INCLUDE[appliesto-gremlin-api](../../../includes/appliesto-gremlin-api.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.9.1 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

> [!IMPORTANT]
> Les verrous de ressources ne fonctionnent pas pour les modifications apportées par les utilisateurs qui se connectent à l’aide d’un SDK Gremlin ou du portail Azure, sauf si le compte Cosmos DB est d’abord verrouillé avec la propriété `disableKeyBasedMetadataWriteAccess` activée. Pour en savoir plus sur l’activation de cette propriété, consultez [Prévention des modifications à partir des SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Exemple de script

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/gremlin/lock.sh "Create a resource lock for an Azure Cosmos DB Gremlin API database and graph.")]

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az lock create](/cli/azure/lock#az-lock-create) | Crée un verrou. |
| [az lock list](/cli/azure/lock#az-lock-list) | Liste les informations sur les verrous. |
| [az lock show](/cli/azure/lock#az-lock-show) | Affiche les propriétés d’un verrou. |
| [az lock delete](/cli/azure/lock#az-lock-delete) | Supprime un verrou. |

## <a name="next-steps"></a>Étapes suivantes

- [Verrouiller les ressources pour empêcher les modifications inattendues](../../../../azure-resource-manager/management/lock-resources.md)

- [Documentation sur l’interface CLI Azure Cosmos DB](/cli/azure/cosmosdb).

- [Dépôt GitHub de l’interface CLI Azure Cosmos DB](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
