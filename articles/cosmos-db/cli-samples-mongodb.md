---
title: Exemples Azure CLI pour l’API Azure Cosmos DB pour MongoDB
description: Exemples Azure CLI pour l’API Azure Cosmos DB pour MongoDB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: b17d3b0072d893751586f87d9a4ceb7ac8607416
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342094"
---
# <a name="azure-cli-samples-for-azure-cosmos-db-api-for-mongodb"></a>Exemples Azure CLI pour l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Le tableau suivant comprend des liens vers des exemples de scripts Azure CLI pour Azure Cosmos DB. Utilisez les liens sur la droite pour accéder aux exemples spécifiques aux API. Les exemples communs sont les mêmes pour toutes les API. Des pages d’informations de référence pour toutes les commandes Azure Cosmos DB CLI sont disponibles dans les [Informations de référence sur Azure CLI](/cli/azure/cosmosdb). Vous trouverez également les exemples de scripts CLI Azure Cosmos DB dans le [dépôt GitHub CLI Azure Cosmos DB](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).

Ces exemples nécessitent Azure CLI version 2.12.1 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="common-samples"></a>Exemples communs

Ces exemples s’appliquent à toutes les API Azure Cosmos DB.

|Tâche | Description |
|---|---|
| [Ajouter des régions de basculement](scripts/cli/common/regions.md?toc=%2fcli%2fazure%2ftoc.json) | Ajoutez une région, changez la priorité de basculement, déclenchez un basculement manuel.|
| [Clés de compte et chaînes de connexion](scripts/cli/common/keys.md?toc=%2fcli%2fazure%2ftoc.json) | Listez les clés de compte, les clés en lecture seule, regénérez les clés et listez les chaînes de connexion.|
| [Sécuriser avec le pare-feu IP](scripts/cli/common/ipfirewall.md?toc=%2fcli%2fazure%2ftoc.json)| Créez un compte Cosmos avec le pare-feu IP configuré.|
| [Sécuriser un nouveau compte avec des points de terminaison de service](scripts/cli/common/service-endpoints.md?toc=%2fcli%2fazure%2ftoc.json)| Créez un compte Cosmos et sécurisez-le avec des points de terminaison de service.|
| [Sécuriser un compte existant avec des points de terminaison de service](scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2fazure%2ftoc.json)| Mettez à jour un compte Cosmos pour le sécuriser avec des points de terminaison de service quand le sous-réseau est finalement configuré.|
|||

## <a name="mongodb-api-samples"></a>Exemples d’API MongoDB

|Tâche | Description |
|---|---|
| [Créer un compte, une base de données et une collection Azure Cosmos](scripts/cli/mongodb/create.md?toc=%2fcli%2fazure%2ftoc.json)| Créez un compte, une base de données et une collection Azure Cosmos DB pour l’API MongoDB. |
| [Créer un compte et une base de données avec mise à l’échelle automatique et deux collections Azure Cosmos avec débit partagé](scripts/cli/mongodb/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crée un compte et une base de données avec mise à l’échelle automatique et deux collections Azure Cosmos DB avec débit partagé pour l’API MongoDB. |
| [Opérations de débit](scripts/cli/mongodb/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Opérations sur une base de données et une collection, comme la lecture du débit, la mise à jour du débit et la migration entre le débit standard et le débit avec mise à l’échelle automatique.|
| [Verrouiller des ressources contre la suppression](scripts/cli/mongodb/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Empêchez la suppression de ressources à l’aide de verrous de ressources.|
|||
