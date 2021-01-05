---
title: Contraintes de code d’orchestrateur durable - Azure Functions
description: Relecture des fonctions d’orchestration et contraintes de code pour Azure Durable Functions.
author: cgillum
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: azfuncdf
ms.openlocfilehash: dc301cf7149ad9fcd5bd5c02226afedc4df5e3ee
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94833093"
---
# <a name="orchestrator-function-code-constraints"></a>Contraintes de code des fonctions d’orchestrateur

Durable Functions est une extension d’[Azure Functions](../functions-overview.md) qui vous permet de créer des applications avec état. Vous pouvez utiliser une [fonction d’orchestrateur](durable-functions-orchestrations.md) pour orchestrer l’exécution d’autres fonctions durables dans une application de fonction. Les fonctions d’orchestrateur présentent un état, sont fiables et peuvent être de longue durée.

## <a name="orchestrator-code-constraints"></a>Contraintes du code d’orchestrateur

Les fonctions d’orchestrateur utilisent l’[approvisionnement en événements](/azure/architecture/patterns/event-sourcing) pour garantir une exécution fiable et maintenir l’état des variables locales. Le [comportement de réexécution](durable-functions-orchestrations.md#reliability) du code d’orchestrateur crée des contraintes sur le type de code qui peut être écrit dans une fonction d’orchestrateur. Par exemple, les fonctions d’orchestrateur doivent être *déterministes* : une fonction d’orchestrateur sera réexécutée à plusieurs reprises et doit générer le même résultat à chaque fois.

### <a name="using-deterministic-apis"></a>Utilisation des API déterministes

Cette section fournit des instructions simples vous permettant de vous assurer que votre code est déterministe.

Les fonctions d’orchestrateur peuvent appeler n’importe quelle API dans son langage cible. Toutefois, il est important qu’elles appellent uniquement des API déterministes. Une *API déterministe* est une API qui retourne toujours la même valeur pour la même entrée, quel que soit le moment ou la fréquence où elle est appelée.

Le tableau suivant présente des exemples d’API qui doivent être évitées, car elles *ne sont pas* déterministes. Ces restrictions s’appliquent uniquement aux fonctions d’orchestrateur. Les autres types de fonction n’ont pas de telles restrictions.

| Catégorie d’API | Motif | Solution de contournement |
| ------------ | ------ | ---------- |
| Dates et heures  | Les API qui retournent la date ou l’heure actuelle ne sont pas déterministes, car la valeur retournée est différente à chaque réexécution. | Utilisez la propriété [CurrentUtcDateTime](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationcontext.currentutcdatetime) dans .NET, l’API `currentUtcDateTime` dans JavaScript ou l’API `current_utc_datetime` dans Python, sans risque pour la réexécution. |
| GUID et UUID  | Les API qui retournent un GUID ou un UUID aléatoire ne sont pas déterministes, car la valeur générée est différente à chaque réexécution. | Utilisez [NewGuid](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationcontext.newguid) dans .NET ou `newGuid` dans JavaScript pour générer des GUID aléatoires de façon sûre. |
| Nombres aléatoires | Les API qui retournent des nombres aléatoires ne sont pas déterministes, car la valeur générée est différente à chaque réexécution. | Utilisez une fonction d’activité pour renvoyer des nombres aléatoires à une orchestration. Les valeurs de retour des fonctions d’activité sont toujours sûres pour la réexécution. |
| Liaisons | Les liaisons d’entrée et de sortie effectuent généralement des opérations d’E/S et ne sont pas déterministes. Même les liaisons du [client d’orchestration](durable-functions-bindings.md#orchestration-client) et du [client d’entité](durable-functions-bindings.md#entity-client) ne doivent pas être utilisées directement par une fonction d’orchestrateur. | Utilisez des liaisons d’entrée et de sortie dans les fonctions clientes ou d’activité. |
| Réseau | Les appels réseau impliquent des systèmes externes et ne sont pas déterministes. | Utilisez les fonctions d’activité pour effectuer des appels réseau. Si vous devez effectuer un appel HTTP à partir de votre fonction d’orchestrateur, vous pouvez également utiliser les [API HTTP durables](durable-functions-http-features.md#consuming-http-apis). |
| API de blocage | Les API de blocage comme `Thread.Sleep` en .NET ou autres peuvent entraîner des problèmes de performances et de mise à l’échelle pour les fonctions d’orchestrateur et doivent être évitées. Dans le plan de consommation Azure Functions, elles peuvent même entraîner des frais d’exécution inutiles. | Dans la mesure du possible, utilisez des alternatives aux API de blocage. Par exemple, utilisez `CreateTimer` pour introduire des délais lors de l’exécution de l’orchestration. Les retards de type [minuteur durable](durable-functions-timers.md) ne sont pas pris en compte dans la durée d’exécution d’une fonction d’orchestrateur. |
| API asynchrones | Le code de l’orchestrateur ne doit jamais lancer d’opération asynchrone, sauf en utilisant l’API `IDurableOrchestrationContext`, l’API `context.df` en JavaScript ou l’API `context` en Python. Par exemple, on ne peut pas utiliser `Task.Run`, `Task.Delay` et `HttpClient.SendAsync` en .NET, ou `setTimeout` et `setInterval` en JavaScript. Durable Task Framework exécute le code d’orchestrateur sur un thread unique. Il ne peut pas interagir avec d’autres threads pouvant être appelés par d’autres API asynchrones. | Une fonction d’orchestrateur doit effectuer uniquement des appels asynchrones durables. Tous les autres appels d’API asynchrones doivent être effectués par des fonctions d’activité. |
| Fonctions JavaScript asynchrones | Il n’est pas possible de déclarer des fonctions d’orchestrateur JavaScript comme `async`, car le runtime node.js ne garantit pas que les fonctions asynchrones sont déterministes. | Déclarez les fonctions d’orchestrateur JavaScript comme fonctions de générateur synchrones. |
| Coroutines Python | Vous ne pouvez pas déclarer des fonctions d’orchestrateur Python en tant que coroutines, c.-à-d. les déclarer avec le mot clé `async`, car la sémantique de coroutine ne s’aligne pas sur le modèle de réexécution Durable Functions. | Déclarez les fonctions d’orchestrateur Python comme des générateurs, ce qui signifie que vous devez vous attendre à ce que l’API `context` utilise `yield` au lieu de `await`.   |
| API de thread | Durable Task Framework exécute le code d’orchestrateur sur un thread unique et ne peut pas interagir avec d’autres threads. L’introduction de nouveaux threads dans l’exécution d’une orchestration peut entraîner des blocages ou des exécutions non déterministes. | Les fonctions d’orchestrateur ne doivent quasiment jamais utiliser des API de thread. Par exemple, dans .NET, évitez d’utiliser `ConfigureAwait(continueOnCapturedContext: false)`. Cela garantit la poursuite de l’exécution des tâches sur le `SynchronizationContext` d’origine de la fonction d’orchestrateur. Si des API de thread sont nécessaires, limitez leur utilisation aux fonctions d’activité. |
| Variables statiques | Évitez d’utiliser des variables statiques non constantes dans les fonctions d’orchestrateur, car leurs valeurs peuvent changer au fil du temps, générant un comportement d’exécution non déterministe. | Utilisez des constantes ou limitez l’utilisation des variables statiques aux fonctions d’activité. |
| Variables d'environnement | N’utilisez pas de variables d’environnement dans les fonctions d’orchestrateur. Leurs valeurs peuvent changer au fil du temps, générant un comportement d’exécution non déterministe. | Les variables d’environnement doivent être référencées uniquement à partir de fonctions clientes ou de fonctions d’activité. |
| Boucles infinies | Evitez les boucles infinies dans les fonctions de l’orchestrateur. Étant donné que Durable Task Framework enregistre l’historique d’exécution à mesure que la fonction d’orchestration s’exécute, une boucle infinie risquerait de laisser une instance d’orchestrateur sans mémoire suffisante. | Dans les scénarios de boucle infinie, utilisez des API comme `ContinueAsNew` en .NET, `continueAsNew` en JavaScript ou `continue_as_new` en Python pour relancer l’exécution de la fonction et ignorer l’historique des précédentes exécutions. |

Si l’application de ces contraintes peut sembler compliquée, elles sont, dans la pratique, faciles à suivre.

Durable Task Framework tente de détecter les violations des règles mentionnées ci-dessus. Si une violation est détectée, le framework lève une exception **NonDeterministicOrchestrationException**. Certaines violations peuvent toutefois échapper à ce comportement de détection. Vous ne devez donc pas en dépendre.

## <a name="versioning"></a>Gestion de version

Une orchestration durable peut s’exécuter en continu pendant des jours, des mois, des années voire même [indéfiniment](durable-functions-eternal-orchestrations.md). Toutes les mises à jour de code apportées aux applications Durable Functions qui affectent des orchestrations non terminées peuvent perturber le comportement de réexécution de l’orchestration. Il est donc important de planifier soigneusement les mises à jour du code. Pour obtenir une description plus détaillée de la façon de gérer les versions de votre code, consultez l’article sur le [contrôle de versions](durable-functions-versioning.md).

## <a name="durable-tasks"></a>Tâches durables

> [!NOTE]
> Cette section décrit en détail l’implémentation interne de Durable Task Framework. Vous pouvez utiliser des fonctions durables sans disposer de ces informations. Elles sont uniquement destinées à vous aider à mieux comprendre le comportement de réexécution.

Les tâches qui peuvent attendre en toute sécurité dans les fonctions d’orchestrateur sont parfois appelées des *tâches durables*. Durable Task Framework crée et gère ces tâches. Il peut s’agir, par exemple, des tâches retournées par **CallActivityAsync**, **WaitForExternalEvent** et **CreateTimer** dans les fonctions d’orchestrateur .NET.

Ces tâches durables sont gérées en interne par une liste d’objets `TaskCompletionSource` en .NET. Lors de la réexécution, ces tâches sont créées dans le cadre de l’exécution du code d’orchestrateur. Elles s’achèvent à mesure que le répartiteur énumère les événements d’historique correspondants.

Les tâches s’exécutent de manière synchrone à l’aide d’un seul thread tant que tout l’historique n’a pas été réexécuté. Les tâches durables non terminées à la fin de la réexécution de l’historique font l’objet d’actions appropriées. Par exemple, un message peut être empilé pour appeler une fonction d’activité.

La description du comportement d’exécution faite dans cette section aide à comprendre pourquoi une fonction d’orchestrateur ne peut pas utiliser `await` ou `yield` dans une tâche non durable. Il y a deux raisons à cela. Tout d’abord, le thread de répartiteur ne peut pas attendre la fin de la tâche. D’autre part, tout rappel par cette tâche risquerait d’altérer l’état de suivi de la fonction d’orchestrateur. Certaines vérifications de l’exécution sont mises en œuvre pour détecter de telles violations.

Pour plus d’informations sur la façon dont Durable Task Framework exécute les fonctions d’orchestrateur, consultez le [code source des tâches durables sur GitHub](https://github.com/Azure/durabletask). Consultez en particulier [TaskOrchestrationExecutor.cs](https://github.com/Azure/durabletask/blob/master/src/DurableTask.Core/TaskOrchestrationExecutor.cs) et [TaskOrchestrationContext.cs](https://github.com/Azure/durabletask/blob/master/src/DurableTask.Core/TaskOrchestrationContext.cs).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment appeler des orchestrations secondaires](durable-functions-sub-orchestrations.md)

> [!div class="nextstepaction"]
> [Découvrez comment gérer les versions](durable-functions-versioning.md)
