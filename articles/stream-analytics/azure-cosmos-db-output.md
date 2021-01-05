---
title: Sortie Azure Cosmos DB depuis Azure Stream Analytics
description: Cet article explique comment sortir des données de Azure Stream Analytics vers Azure Cosmos DB.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: a086d9fe150766c6b31210f29bf802a75e0ee4ec
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491730"
---
# <a name="azure-cosmos-db-output-from-azure-stream-analytics"></a>Sortie Azure Cosmos DB depuis Azure Stream Analytics

[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) est un service de base de données distribuée à l’échelle mondiale, qui offre une mise à l’échelle élastique et sans limite dans le monde entier, des requêtes enrichies et une indexation automatique sur les modèles de données indépendants des schémas. Pour en savoir plus sur les options de conteneur Azure Cosmos DB pour Stream Analytics, consultez l’article [Sortie Azure Stream Analytics dans Azure Cosmos DB](stream-analytics-documentdb-output.md).

La sortie Azure Cosmos DB de Stream Analytics n’est pas disponible dans les régions Azure - Chine 21Vianet et Azure - Allemagne (T-Systems International).

> [!Note]
> Azure Stream Analytics prend uniquement en charge la connexion à Azure Cosmos DB à l’aide de l’API SQL.
> Les autres API Azure Cosmos DB ne sont pas encore prises en charge. Si vous pointez Azure Stream Analytics vers les comptes Azure Cosmos DB créés avec d'autres API, les données risquent de ne pas être correctement stockées.

Le tableau suivant décrit les propriétés de création d’une sortie Azure Cosmos DB.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie | Alias référençant cette sortie dans votre requête Stream Analytics. |
| Récepteur | Azure Cosmos DB. |
| Option d’importation | Choisissez **Sélectionner Cosmos DB dans votre abonnement** ou **Fournir manuellement les paramètres Cosmos DB**.
| ID de compte | Nom ou URI de point de terminaison du compte Azure Cosmos DB. |
| Clé de compte | Clé d’accès partagé du compte Azure Cosmos DB. |
| Base de données | Nom de la base de données Azure Cosmos DB. |
| Nom du conteneur | Nom du conteneur à utiliser, qui doit se trouver dans Cosmos DB. Exemple :  <br /><ul><li> _MyContainer_ : Un conteneur nommé « MyContainer » doit exister.</li>|
| ID du document |facultatif. Nom du champ dans les événements de sortie utilisé pour spécifier la clé primaire sur laquelle sont basées les opérations d’insertion ou de mise à jour.

## <a name="partitioning"></a>Partitionnement

La clé de partition est basée sur la clause PARTITION BY dans la requête. Le nombre d’enregistreurs de sortie suit le partitionnement d’entrée de [requêtes entièrement mises en parallèle ](stream-analytics-scale-jobs.md). Stream Analytics convertit la clé de partition de sortie Cosmos DB en chaîne. Par exemple, si vous avez une clé de partition avec une valeur de 1 de type bigint, elle est convertie en « 1 » de type chaîne. Cette conversion a toujours lieu, que la propriété de partition soit écrite ou non dans Cosmos DB.

## <a name="output-batch-size"></a>Taille de lot de sortie

Pour obtenir la taille maximale du message, consultez [Limites d’Azure Cosmos DB](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cosmos-db-limits). La taille des lots et la fréquence d’écriture sont ajustées dynamiquement en fonction des réponses Azure Cosmos DB. Stream Analytics n’impose aucune limite prédéterminée.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Azure Stream Analytics avec l’interface de ligne de commande Azure](quick-create-azure-cli.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide d’un modèle Resource Manager](quick-create-azure-resource-manager.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)
