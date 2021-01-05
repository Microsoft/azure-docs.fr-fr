---
title: Facturation Durable Functions - Azure Functions
description: Découvrez les comportements internes de Durable Functions et son incidence sur la facturation pour Azure Functions.
author: cgillum
ms.topic: overview
ms.date: 08/31/2019
ms.author: azfuncdf
ms.openlocfilehash: 504ef93a0002895bc5662d95ad269c8593170ee2
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "74233012"
---
# <a name="durable-functions-billing"></a>Facturation Durable Functions

[Durable Functions](durable-functions-overview.md) suit le même modèle de facturation qu’Azure Functions. Pour plus d’informations, consultez [Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

Lors de l’exécution de fonctions d’orchestrateur dans le [plan de consommation](../functions-scale.md#consumption-plan) Azure Functions, vous devez avoir conscience de certains comportements de facturation. Les sections suivantes décrivent ces comportements et leur incidence plus en détail.

## <a name="orchestrator-function-replay-billing"></a>Facturation de la relecture de fonctions orchestrator

Les [fonctions d’orchestrateur](durable-functions-orchestrations.md) peuvent être relues plusieurs fois pendant toute la durée de vie d’une orchestration. Chaque relecture est considérée par le runtime Azure Functions comme un appel de fonction distinct. C’est pourquoi, dans le plan de Consommation Azure Functions, vous êtes facturé pour chaque relecture d’une fonction d’orchestrateur. Les autres types de plans ne facturent pas la relecture des fonctions d’orchestrateur.

## <a name="awaiting-and-yielding-in-orchestrator-functions"></a>Attente et interruption dans les fonctions orchestrator

Quand une fonction d’orchestrateur attend qu’une action asynchrone se termine en utilisant **await** en C# ou **yield** en JavaScript, le runtime considère cette exécution particulière comme terminée. La facturation de la fonction d’orchestrateur s’arrête à ce stade. Elle ne reprend qu’à la relecture de la fonction d’orchestrateur suivante. Vous n’êtes pas facturé pour le temps d’attente ou d’interruption dans une fonction orchestrator.

> [!NOTE]
> Les fonctions qui appellent d’autres fonctions sont considérées par certains comme un anti-modèle. Cela est dû à un problème appelé _double facturation_. Quand une fonction appelle une autre fonction directement, elles s’exécutent toutes les deux en même temps. La fonction appelée exécute du code activement pendant que la fonction appelante attend une réponse. Dans ce cas, vous devez payer pour le temps passé par la fonction appelante à attendre que la fonction appelée s’exécute.
>
> Il n’existe pas de double facturation dans les fonctions d’orchestrateur. La facturation de la fonction d’orchestrateur s’arrête pendant qu’elle attend le résultat d’une fonction d’activité ou d’une fonction de sous-orchestration.

## <a name="durable-http-polling"></a>Interrogation HTTP durable

Les fonctions d’orchestrateur peuvent effectuer des appels HTTP longs à des points de terminaison externes, comme décrit dans l’article [Fonctionnalités HTTP](durable-functions-http-features.md). La méthode **CallHttpAsync** en C# et la méthode **callHttp** en JavaScript peuvent interroger en interne un point de terminaison HTTP tout en suivant le [modèle 202 asynchrone](durable-functions-http-features.md#http-202-handling).

Il n’existe actuellement pas de facturation directe pour les opérations d’interrogation HTTP internes. Toutefois, l’interrogation interne peut entraîner la relecture périodique de la fonction d’orchestrateur. Ces relectures de fonction internes font l’objet d’une facturation de frais standard.

## <a name="azure-storage-transactions"></a>Transactions de stockage Azure

Durable Functions utilise par défaut le Stockage Azure pour maintenir l’état persistant, traiter les messages et gérer les partitions par le biais de baux d’objets blob. Comme vous détenez ce compte de stockage, tous les coûts de transaction sont facturés à votre abonnement Azure. Pour plus d’informations sur les artefacts du Stockage Azure utilisés par Durable Functions, consultez l’article [Hubs de tâches](durable-functions-task-hubs.md).

Plusieurs facteurs contribuent aux coûts du Stockage Azure réels engendrés par votre application Durable Functions :

* Une application de fonction est associée à un seul hub de tâches, qui partage un ensemble de ressources de Stockage Azure. Ces ressources sont utilisées par toutes les fonctions durables dans une application de fonction. Le nombre réel de fonctions dans l’application de fonction n’a aucune incidence sur les coûts des transactions du Stockage Azure.
* Chaque instance d’application de fonction interroge en interne plusieurs files d’attente dans le compte de stockage à l’aide d’un algorithme d’interrogation avec interruption exponentielle. Une instance d’application inactive interroge les files d’attente moins souvent qu’une application active, ce qui réduit les coûts de transaction. Pour plus d’informations sur le comportement d’interrogation de file d’attente Durable Functions, consultez la [section relative à l’interrogation de file d’attente dans l’article Performances et mise à l’échelle](durable-functions-perf-and-scale.md#queue-polling).
* En cas d’exécution dans les plans Consommation Azure Functions ou Premium, le [contrôleur de mise à l’échelle Azure Functions](../functions-scale.md#how-the-consumption-and-premium-plans-work) interroge régulièrement toutes les files d’attente de hub de tâches en arrière-plan. En cas d’échelle faible à modérée d’une application de fonction, une seule instance de contrôleur d’échelle interroge ces files d’attente. Si l’application de fonction est étendue à un grand nombre d’instances, d’autres instances de contrôleur d’échelle peuvent être ajoutées. Ces instances de contrôleur d’échelle supplémentaires peuvent augmenter les coûts totaux de transaction de file d’attente.
* Chaque instance d’application de fonction est en concurrence pour un ensemble de baux d’objets blob. Ces instances effectuent régulièrement des appels au service BLOB Azure pour renouveler les baux détenus ou tenter d’acquérir de nouveaux baux. Le nombre de partitions configurées du hub de tâches détermine le nombre de baux d’objets blob. Un scale-out vers un plus grand nombre d’instances d’application de fonction augmente probablement les coûts de transaction du Stockage Azure associés à ces opérations de bail.

Vous trouverez de plus amples informations sur les tarifs du Stockage Azure dans la documentation sur la [tarification Stockage Azure](https://azure.microsoft.com/pricing/details/storage/). 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la tarification Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
