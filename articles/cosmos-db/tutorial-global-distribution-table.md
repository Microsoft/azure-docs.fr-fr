---
title: Tutoriel de distribution mondiale Azure Cosmos DB pour l’API Table
description: Découvrez le fonctionnement de la diffusion mondiale dans les comptes d’API Table Azure Cosmos DB et la façon de configurer la liste préférée des régions
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 01/30/2020
ms.reviewer: sngun
ms.openlocfilehash: f06fbe482327d01917f648ccdec34baa15c46e80
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93073908"
---
# <a name="set-up-azure-cosmos-db-global-distribution-using-the-table-api"></a>Configurer la distribution mondiale Azure Cosmos DB à l’aide de l’API de table
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Cet article décrit les tâches suivantes : 

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configurer la distribution mondiale à l’aide de [l’API Table](table-introduction.md)

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]


## <a name="connecting-to-a-preferred-region-using-the-table-api"></a>Se connecter à une région de prédilection avec l’API Table

Pour tirer parti de la [distribution globale](distribute-data-globally.md), les applications clientes doivent spécifier l’emplacement actuel où leur application s’exécute. Pour cela, définissez la propriété `CosmosExecutorConfiguration.CurrentRegion`. La propriété `CurrentRegion` doit contenir un seul emplacement. Chaque instance de client peut spécifier sa propre région pour les lectures à faible latence. La région doit être nommée en utilisant son [nom d’affichage](/previous-versions/azure/reference/gg441293(v=azure.100)), comme « USA Ouest ». 

Le SDK de l’API Table d’Azure Cosmos DB sélectionne automatiquement le point de terminaison qui convient le mieux pour communiquer en fonction de la configuration du compte et de la disponibilité régionale actuelle. Il donne la priorité à la région la plus proche pour offrir une meilleure latence aux clients. Une fois que vous avez défini la propriété `CurrentRegion` actuelle, les demandes de lecture et d’écriture sont dirigées comme suit :

* **Demandes de lecture :** Toutes les demandes de lecture sont envoyées à la `CurrentRegion` configurée. En fonction de la proximité, le SDK sélectionne automatiquement une région géorépliquée de secours pour la haute disponibilité.

* **Demandes d’écriture :** Le SDK envoie automatiquement toutes les demandes d’écriture vers la région d’écriture active. Dans un compte avec écritures multirégions, la région actuelle traite également les demandes d’écriture. En fonction de la proximité, le SDK sélectionne automatiquement une région géorépliquée de secours pour la haute disponibilité.

Si vous ne spécifiez pas la propriété `CurrentRegion`, le SDK utilise la région d’écriture actuelle pour toutes les opérations.

Par exemple, si un compte Azure Cosmos se trouve dans les régions « USA Ouest » et « USA Est ». Si « USA Ouest » est la région d’écriture et que l’application se trouve dans « USA Ouest ». Si la propriété CurrentRegion n’est pas configurée, toutes les demandes de lecture et d’écriture sont toujours dirigées vers la région « USA Ouest ». Si la propriété CurrentRegion est configurée, toutes les demandes de lecture sont traitées depuis la région « USA Est ».

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configurer la distribution mondiale à l’aide des API Table d’Azure Cosmos DB