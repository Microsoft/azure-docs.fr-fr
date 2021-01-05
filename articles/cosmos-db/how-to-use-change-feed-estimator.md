---
title: Utiliser l’estimateur de flux de modification - Azure Cosmos DB
description: Découvrez comment utiliser l’estimateur de flux de modification pour analyser la progression de votre processeur de flux de modification.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 08/15/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: a44557d15f437317c2b5fa659ab8d4ca3c208edf
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339833"
---
# <a name="use-the-change-feed-estimator"></a>Utilisation de l’estimateur de flux de modification
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit comment superviser la progression de vos instances de [processeur de flux de modification](./change-feed-processor.md) au fur et à mesure qu’elles lisent le flux de modification.

## <a name="why-is-monitoring-progress-important"></a>Pourquoi la supervision de la progression est-elle importante ?

Le processeur de flux de modification agit comme un pointeur qui avance dans votre [flux de modification](./change-feed.md) et fournit les modifications à une implémentation de délégué. 

Votre déploiement de processeur de flux de modification peut traiter les modifications à une vitesse particulière en fonction de ses ressources disponibles, telles que le CPU, la mémoire, le réseau, et ainsi de suite.

Si cette vitesse est inférieure à celle à laquelle vos modifications se produisent dans votre conteneur Azure Cosmos, votre processeur commencera à accumuler du retard.

L’identification de ce scénario aide à comprendre si nous devons mettre à l’échelle notre déploiement du processeur de flux de modification.

## <a name="implement-the-change-feed-estimator"></a>Utilisation de l’estimateur de flux de modification

Comme le [processeur de flux de modification](./change-feed-processor.md), l’estimateur de flux de modification fonctionne en tant que modèle Push. L’estimateur mesure la différence entre le dernier élément traité (défini par l’état du conteneur de baux) et la dernière modification dans le conteneur, et il transmet cette valeur à un délégué. Vous pouvez également personnaliser l’intervalle de prise de cette mesure avec une valeur par défaut de cinq secondes.

Par exemple, si votre processeur de flux de modification est défini comme suit :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartProcessorEstimator)]

La méthode correcte pour initialiser un estimateur afin de mesurer ce processeur consisterait à utiliser `GetChangeFeedEstimatorBuilder` comme suit :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartEstimator)]

Où le processeur et l’estimateur partagent le même `leaseContainer` et le même nom.

Les deux autres paramètres sont le délégué, qui recevra un nombre représentant **le nombre de modifications en attente de lecture** par le processeur, et l’intervalle de temps auquel vous souhaitez que cette mesure soit prise.

Voici un exemple de délégué qui reçoit l’estimation :

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=EstimationDelegate)]

Vous pouvez envoyer cette estimation à votre solution de supervision et l’utiliser pour comprendre le fonctionnement de votre progression au fil du temps.

> [!NOTE]
> L’estimateur de flux de modification n’a pas besoin d’être déployé dans le cadre de votre processeur de flux de modification, ni de faire partie du même projet. Il peut être indépendant et s’exécuter dans une instance complètement différente. Il doit simplement utiliser le même nom et la même configuration de bail.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md)
* [Exemples d’utilisation sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Exemples supplémentaires sur GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le processeur de flux de modification, consultez les articles suivants :

* [Vue d’ensemble du processeur de flux de modification](change-feed-processor.md)
* [Heure de début du processeur de flux de modification](./change-feed-processor.md#starting-time)