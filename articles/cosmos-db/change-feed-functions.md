---
title: Comment utiliser le flux de modification Azure Cosmos DB avec Azure Functions
description: Utilisez Azure Functions pour vous connecter au flux de modification Azure Cosmos DB. Plus tard, vous pourrez créer des fonctions Azure réactives qui sont déclenchées à chaque nouvel événement.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/03/2019
ms.reviewer: sngun
ms.openlocfilehash: 7f4903cf29f15132db91e47d78efe5a556efd937
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340241"
---
# <a name="serverless-event-based-architectures-with-azure-cosmos-db-and-azure-functions"></a>Architectures basées sur des événements serverless avec Azure Cosmos DB et Azure Functions
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Functions fournit le moyen le plus simple pour vous connecter au [flux de modification](change-feed.md). Vous pouvez créer de petites fonctions Azure Functions réactives qui se déclencheront automatiquement sur chaque nouvel événement dans le flux de modification de votre conteneur Azure Cosmos.

:::image type="content" source="./media/change-feed-functions/functions.png" alt-text="Fonctions basées sur les événements serverless fonctionnant avec le déclencheur Azure Functions pour Cosmos DB" border="false":::

Avec le [déclencheur Azure Functions pour Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md), vous pouvez tirer parti de la mise à l’échelle du [processeur de flux de modification](./change-feed-processor.md) et de la fonctionnalité fiable de détection d’événement sans conserver d’[infrastructure Worker](./change-feed-processor.md). Concentrez-vous sur la logique de votre fonction Azure sans vous soucier du reste du pipeline source d’événements. Vous pouvez même combiner le déclencheur avec d’autres [liaisons Azure Functions](../azure-functions/functions-triggers-bindings.md#supported-bindings).

> [!NOTE]
> Actuellement, le déclencheur Azure Functions pour Cosmos DB est pris en charge avec l’API Core (SQL) uniquement.

## <a name="requirements"></a>Spécifications

Pour implémenter un flux serverless basé sur des événements, vous avez besoin des éléments suivants :

* **Conteneur supervisé :** le conteneur supervisé désigne le conteneur Azure Cosmos actuellement supervisé. Il stocke les données à partir desquelles le flux de modification est généré. Toutes les insertions, mises à jour apportées au conteneur supervisé sont répercutées dans le flux de modification du conteneur.
* **Conteneur de baux** : le conteneur de baux gère l’état entre plusieurs instances de fonction Azure serverless dynamiques et permet une mise à l’échelle dynamique. Ce conteneur de bail peut être créé manuellement ou automatiquement par le déclencheur Azure Functions pour Cosmos DB. Pour créer automatiquement le conteneur de bail, définissez l'indicateur *CreateLeaseCollectionIfNotExists* dans la [configuration](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration). Les conteneurs de baux partitionnés sont nécessaires pour avoir une définition de clé de partition `/id`.

## <a name="create-your-azure-functions-trigger-for-cosmos-db"></a>Créer votre déclencheur Azure Functions pour Cosmos DB

La création de votre fonction Azure avec un déclencheur Azure Functions pour Cosmos DB est désormais prise en charge sur toutes les intégrations de l’interface de commande Azure Functions IDE et CLI :

* [Extension Visual Studio](../azure-functions/functions-develop-vs.md) pour les utilisateurs Visual Studio.
* [Extension Visual Studio Code](/azure/developer/javascript/tutorial-vscode-serverless-node-01) pour les utilisateurs Visual Studio Code.
* Et enfin [outils Core CLI](../azure-functions/functions-run-local.md#create-func) pour une expérience sans dépendance d’IDE multiplateforme.

## <a name="run-your-trigger-locally"></a>Exécuter votre déclencheur localement

Vous pouvez exécuter votre [fonction Azure localement](../azure-functions/functions-develop-local.md) avec l’[émulateur Azure Cosmos DB](./local-emulator.md) pour créer et développer vos flux basés sur les événements serverless sans abonnement Azure et sans frais.

Si vous souhaitez tester des scénarios en direct dans le cloud, vous pouvez [tester gratuitement Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sans carte de crédit ni abonnement Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le flux de modification, consultez les articles suivants :

* [Présentation du flux de modification](change-feed.md)
* [Manières de lire le flux de modification](read-change-feed.md)
* [Utilisation de la bibliothèque du processeur de flux de modification](change-feed-processor.md)
* [Comment utiliser la bibliothèque du processeur de flux de modification](change-feed-processor.md)
* [Traitement de base de données serverless à l’aide d’Azure Cosmos DB et d’Azure Functions](serverless-computing-database.md)
