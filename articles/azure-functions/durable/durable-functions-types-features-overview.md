---
title: Types de fonctions dans Azure Durable Functions
description: Découvrez les types de fonctions et les rôles qui permettent la communication de fonction à fonction dans le cadre d’une orchestration Durable Functions dans Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 08/22/2019
ms.author: azfuncdf
ms.openlocfilehash: 35ef9d8731e169e890f5985ce01215fec5d6e3de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84697705"
---
# <a name="durable-functions-types-and-features"></a>Types et fonctionnalités de Durable Functions

Durable Functions est une extension d’[Azure Functions](../functions-overview.md). Vous pouvez utiliser Durable Functions pour effectuer une orchestration avec état d’une exécution de fonction. Une application de fonction durable est une solution composée de différentes fonctions Azure. Les fonctions peuvent jouer différents rôles dans le cadre d’une orchestration de fonction durable. 

Il existe actuellement 4 types de fonctions durables dans Azure Functions : les fonctions d’activité, les fonctions d’orchestrateur, les fonctions d’entité et les fonctions clientes. Le reste de cette section décrit en détail les types de fonctions impliqués dans une orchestration.

## <a name="orchestrator-functions"></a>Fonctions d’orchestrateur

Les fonctions d’orchestrateur décrivent la façon dont les actions sont exécutées et l’ordre dans lequel elles sont exécutées. Les fonctions d’orchestrateur décrivent l’orchestration dans le code (C# ou JavaScript), comme indiqué dans les [modèles d’application Durable Functions](durable-functions-overview.md#application-patterns). Une orchestration peut avoir de nombreux types d’actions différents, comme des [fonctions d’activité](#activity-functions), des [orchestrations secondaires](durable-functions-orchestrations.md#sub-orchestrations), l’[attente d’événements externes](durable-functions-orchestrations.md#external-events), [HTTP](durable-functions-http-features.md) et des [minuteurs](durable-functions-orchestrations.md#durable-timers). Les fonctions d’orchestrateur peuvent également interagir avec des [fonctions d’entité](#entity-functions).

> [!NOTE]
> Les fonctions d’orchestrateur sont écrites à l’aide de code ordinaire, mais des conditions strictes s’appliquent à l’écriture du code. Plus précisément, le code de la fonction d’orchestrateur doit être *déterministe*. Le non-respect de ces conditions de déterminisme peut entraîner l’échec de l’exécution des fonctions d’orchestrateur. Vous trouverez des informations détaillées sur ces conditions et sur la façon de les contourner dans la rubrique sur les [contraintes de code](durable-functions-code-constraints.md).

Pour plus d’informations sur les fonctions d’orchestrateur et leurs fonctionnalités, consultez l’article [Orchestrations durables](durable-functions-orchestrations.md).

## <a name="activity-functions"></a>Fonctions d’activité

Les fonctions d’activité sont à la base de l’orchestration de fonction durable. Les fonctions d’activité correspondent aux fonctions et aux tâches qui sont orchestrées dans le processus. Par exemple, vous pouvez créer une fonction d’orchestrateur pour traiter une commande. Ces tâches impliquent la vérification de l’inventaire, la facturation du client et l’organisation de l’expédition. Chacune de ces tâches correspond à une fonction d’activité distincte. Ces fonctions d’activité peuvent être exécutées en série, en parallèle ou dans une combinaison de ces deux modes.

À la différence des fonctions d’orchestrateur, les fonctions d’activité ne sont pas limitées quant au type de travail qu’elles vous permettent d’effectuer. Les fonctions d’activité sont fréquemment utilisées pour effectuer des appels réseau ou exécuter des opérations gourmandes en processeur. Une fonction d’activité peut également retourner des données à la fonction d’orchestrateur. L’infrastructure Durable Task Framework garantit que chaque fonction d’activité appelée est exécutée *au moins une fois* au cours de l’exécution d’une orchestration.

> [!NOTE]
> Comme les fonctions d’activité garantissent uniquement une exécution *au moins une fois*, nous vous recommandons de rendre *idempotente* votre logique de fonction d’activité chaque fois que cela est possible.

Utilisez un [déclencheur d’activité](durable-functions-bindings.md#activity-trigger) pour définir une fonction d’activité. Les fonctions .NET reçoivent `DurableActivityContext` en tant que paramètre. Vous pouvez également lier le déclencheur à un autre objet sérialisable au format JSON pour transmettre des entrées à la fonction. En JavaScript, vous pouvez accéder à l’entrée via la propriété `<activity trigger binding name>` de l’[objet `context.bindings`](../functions-reference-node.md#bindings). Une seule valeur peut être passée aux fonctions d’activité. Pour passer plusieurs valeurs, vous devez utiliser des tuples, des tableaux ou des types complexes.

> [!NOTE]
> Vous ne pouvez déclencher une fonction d’activité qu’à partir d’une fonction d’orchestrateur.

## <a name="entity-functions"></a>Fonctions d’entité

Les fonctions d’entité définissent les opérations pour la lecture et la mise à jour de petits éléments d’état. Nous faisons souvent référence à ces entités avec état en tant qu’*entités durables*. Comme les fonctions d’orchestrateur, les fonctions d’entité sont des fonctions ayant un type spécial de déclencheur, *déclencheur d’entité*. Elles peuvent également être appelées à partir de fonctions clientes ou de fonctions d’orchestrateur. Contrairement aux fonctions d’orchestrateur, les fonctions d’entité n’ont pas de contraintes code spécifiques. Les fonctions d’entité gèrent également l’état explicitement plutôt que de représenter implicitement l’état via le flux de contrôle.

> [!NOTE]
> Les fonctions d’entité et les fonctionnalités associées sont uniquement disponibles dans Durable Functions 2.0 et versions ultérieures.

Pour plus d’informations sur les fonctions d’entité, consultez l’article [Entités durables](durable-functions-entities.md).

## <a name="client-functions"></a>Fonctions clientes

Les fonctions d’orchestrateur sont déclenchées par une [liaison de déclencheur d’orchestration](durable-functions-bindings.md#orchestration-trigger) et les fonctions d’entité sont déclenchées par une [liaison de déclencheur d’entité](durable-functions-bindings.md#entity-trigger). Ces deux déclencheurs fonctionnent en réagissant aux messages mis en file d’attente dans un [hub de tâches](durable-functions-task-hubs.md). Le principal moyen de fournir ces messages est d’utiliser une [liaison de client d’orchestrateur](durable-functions-bindings.md#orchestration-client) ou une [liaison de client d’entité](durable-functions-bindings.md#entity-client) à partir d’une *fonction cliente*. Toute fonction non-orchestrateur peut être une *fonction cliente*. Par exemple, vous pouvez déclencher l’orchestrateur à partir d’une fonction déclenchée par HTTP, d’une fonction déclenchée par Azure Event Hub, etc. Ce qui fait d’une fonction une *fonction cliente* est son utilisation de la liaison de sortie du client durable.

> [!NOTE]
> Contrairement aux autres types de fonction, les fonctions d’orchestrateur et d’entité ne peuvent pas être déclenchées directement à l’aide des boutons du portail Azure. Si vous souhaitez tester une fonction d’orchestrateur ou d’entité dans le portail Azure, vous devez exécuter à la place une *fonction cliente* qui démarre une fonction d’orchestrateur ou d’entité dans le cadre de son implémentation. Pour la plus simple expérience de test, une fonction de *déclencheur manuel* est recommandée.

En plus de déclencher des fonctions d’orchestrateur ou d’entité, la liaison de *client durable* peut être utilisée pour interagir avec des orchestrations et des entités en cours d’exécution. Par exemple, les orchestrations peuvent être interrogées, arrêtées et faire l’objet d’événements déclenchés. Pour plus d’informations sur la gestion des orchestrations et des entités, consultez l’article [Gestion d’instance](durable-functions-instance-management.md).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer, créez votre première fonction durable en [C#](durable-functions-create-first-csharp.md) ou en [JavaScript](quickstart-js-vscode.md).

> [!div class="nextstepaction"]
> [En savoir plus sur les orchestrations Durable Functions](durable-functions-orchestrations.md)