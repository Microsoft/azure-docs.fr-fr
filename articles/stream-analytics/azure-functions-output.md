---
title: Sortie Azure Functions d’Azure Stream Analytics
description: Cet article décrit les fonctions Azure en tant que sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: ffaf66cbb5d9fa0b294f5749a1923684f16c2979
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94488086"
---
# <a name="azure-functions-output-from-azure-stream-analytics"></a>Sortie Azure Functions d’Azure Stream Analytics

Azure Functions est un service de calcul serverless que vous pouvez utiliser pour exécuter du code à la demande sans configurer ou gérer l’infrastructure explicitement. Grâce à ce service, vous pouvez implémenter le code qui est déclenché par les événements qui se produisent dans Azure ou des services partenaires. Comme Azure Functions peut répondre à des déclencheurs, il constitue l’outil de sortie logique pour Azure Stream Analytics. Cet adaptateur de sortie permet aux utilisateurs de connecter Stream Analytics à Azure Functions et d’exécuter un script ou un fragment de code en réaction à différents événements.

La sortie Azure Functions de Stream Analytics n’est pas disponible dans les régions Azure - Chine 21Vianet et Azure - Allemagne (T-Systems International). Il n’est pas non plus possible d’établir une connexion à Azure Functions au sein d’un réseau virtuel à partir d’une tâche Stream Analytics qui s’exécute dans un cluster multilocataire.

Azure Stream Analytics appelle Azure Functions via des déclencheurs HTTP. L’adaptateur de sortie Azure Functions est disponible, avec les propriétés configurables suivantes :

| Nom de la propriété | Description |
| --- | --- |
| Conteneur de fonctions |Nom de votre application Azure Functions. |
| Fonction |Nom de la fonction dans votre application Azure Functions. |
| Clé |Si vous souhaitez utiliser une fonction Azure à partir d’un autre abonnement, vous pouvez le faire en fournissant la clé permettant d’accéder à votre fonction. |
| Taille de lot maximale |Propriété qui peut être utilisée pour définir la taille maximale de chaque lot de sortie qui est envoyé à votre fonction Azure. L’unité d’entrée est exprimée en octets. Par défaut, cette valeur est définie sur 262 144 Ko (256 Ko). |
| Nombre maximal de lots  |Propriété qui vous permet de spécifier le nombre maximal d’événements dans chaque lot envoyé à Azure Functions. La valeur par défaut est 100. |

Azure Stream Analytics attend l’état HTTP 200 de l’application Functions pour les lots traités avec succès.

Lorsqu’Azure Stream Analytics reçoit une erreur 413 (qui indique que l’entité de requête HTTP est trop volumineuse) de la part d’une fonction Azure, il réduit la taille des lots envoyés à Azure Functions. Dans le code de votre fonction Azure, utilisez cette exception pour vous assurer qu'Azure Stream Analytics n'envoie pas de lots trop volumineux. Vérifiez également que les valeurs de taille et de nombre de lots maximum utilisées dans la fonction correspondent à celles qui ont été saisies dans le portail Stream Analytics.

> [!NOTE]
> Lors du test de la connexion, Stream Analytics envoie un lot vide à Azure Functions pour s’assurer que la connexion entre les deux fonctionne. Assurez-vous que votre Function App gère les demandes de lots vides pour vérifier si la connexion de test est bien établie.

De plus, si aucun événement n’est signalé dans le temps imparti, aucune sortie n’est générée. De ce fait, la fonction **computeResult** n’est pas appelée. Ce comportement est cohérent avec les fonctions d’agrégation fenêtrées intégrées.

## <a name="partitioning"></a>Partitionnement

La clé de partition est basée sur la clause PARTITION BY dans la requête. Le nombre d’enregistreurs de sortie suit le partitionnement d’entrée de [requêtes entièrement mises en parallèle ](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Taille de lot de sortie

Taille de lot par défaut : 262 144 Ko (256 Ko). Nombre d’événements par défaut par lot : 100. La taille de lot, configurable, peut être augmentée ou diminuée dans les options de sortie de Stream Analytics.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Azure Stream Analytics avec l’interface de ligne de commande Azure](quick-create-azure-cli.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide d’un modèle Resource Manager](quick-create-azure-resource-manager.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)