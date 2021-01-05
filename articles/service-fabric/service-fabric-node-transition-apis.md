---
title: Démarrer et arrêter des nœuds de cluster
description: Découvrez comment utiliser l’injection d’erreurs pour tester une application Service Fabric en démarrant et en arrêtant des nœuds de cluster.
author: LMWF
ms.topic: conceptual
ms.date: 6/12/2017
ms.author: lemai
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c31040ec13084f9e4b08bbc9a347e4ad44975bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89021253"
---
# <a name="replacing-the-start-node-and-stop-node-apis-with-the-node-transition-api"></a>Remplacement des API de démarrage et d’arrêt de nœud par l’API de transition de nœud

## <a name="what-do-the-stop-node-and-start-node-apis-do"></a>À quoi servent les API de démarrage et d’arrêt de nœud ?

L’API Stop Node (managée : [StopNodeAsync()][stopnode], PowerShell: [Stop-ServiceFabricNode][stopnodeps]) arrête un nœud Service Fabric.  Un nœud Service Fabric est un processus, pas une machine virtuelle ou un ordinateur. La machine virtuelle ou l’ordinateur continuera à fonctionner.  Dans le reste du document, « nœud » désigne le nœud Service Fabric.  L’arrêt d’un nœud le fait passer à l’état *arrêté*, dans lequel il n’est pas membre du cluster et ne peut pas héberger de services, fonctionnant ainsi comme un nœud *en panne*.  Cet état est utile pour créer des erreurs dans le système afin de tester votre application.  L’API Start Node (managée : [StartNodeAsync()][startnode], PowerShell: [Start-ServiceFabricNode][startnodeps]]) inverse l’effet de l’API d’arrêt de nœud, qui rétablit le nœud à un état normal.

## <a name="why-are-we-replacing-these"></a>Pourquoi remplaçons-nous ces API ?

Comme indiqué précédemment, un nœud Service Fabric *arrêté* est un nœud qui a été volontairement ciblé à l’aide de l’API d’arrêt de nœud.  Un nœud *en panne* est un nœud qui est arrêté pour toute autre raison (par exemple, si la machine virtuelle ou l’ordinateur est éteint).  Avec l’API d’arrêt de nœud, le système ne fournit pas d’informations permettant de différencier les nœuds *arrêtés* des nœuds *en panne*.

De plus, certaines erreurs renvoyées par ces API ne sont pas aussi descriptives qu’elles pourraient l’être.  Par exemple, si vous appelez l’API d’arrêt de nœud sur un nœud déjà *arrêté*, vous verrez l’erreur *InvalidAddress*.  Cette expérience pourrait être améliorée.

La durée pendant laquelle un nœud est arrêté est « infinie » jusqu'à ce que l’API de démarrage de nœud soit appelée.  Nous avons découvert que cela pouvait provoquer des problèmes et des erreurs.  Par exemple, il est arrivé qu’un utilisateur appelle l’API d’arrêt de nœud sur un nœud, puis l’oublie.  Plus tard, il était difficile de savoir si le nœud était *en panne* ou *arrêté*.


## <a name="introducing-the-node-transition-apis"></a>Présentation des API de transition de nœud

Nous avons résolu les problèmes ci-dessus dans un nouvel ensemble d’API.  La nouvelle API Node Transition (managée : [StartNodeTransitionAsync()][snt]) peut être utilisée pour passer un nœud Service Fabric à l’état *arrêté* ou pour le faire passer de l’état *arrêté* à un état normal.  Veuillez noter que le mot « Start » dans le nom de l’API ne fait pas référence au démarrage d’un nœud.  Il fait référence au début d’une opération asynchrone que le système exécute pour faire passer le nœud à l’état *arrêté* ou à l’état démarré.

**Utilisation**

Si l’API de transition de nœud ne renvoie pas d’exception lorsqu’elle est appelée, cela signifie que le système a accepté l’opération asynchrone et autorise son exécution.  Un appel réussi n’implique pas que l’opération est terminée.  Pour obtenir des informations sur l’état actuel de l’opération, appelez l’API Node Transition Progress (managée : [GetNodeTransitionProgressAsync()][gntp]) avec le GUID utilisé lors de l’appel de l’API de transition de nœud pour cette opération.  L’API de progression de transition de nœud renvoie un objet NodeTransitionProgress.  La propriété State de cet objet indique l’état actuel de l’opération.  Si l’état est « Running », cela signifie que l’opération est en cours d’exécution.  Si l’état est « Completed », l’opération s’est terminée sans erreur.  Si l’état est « Faulted », un problème est survenu pendant l’exécution de l’opération.  La propriété Exception de la propriété Result indique quel était le problème.  Pour plus d’informations sur la propriété State, consultez https://docs.microsoft.com/dotnet/api/system.fabric.testcommandprogressstate. Pour obtenir des exemples de code, consultez la section « Exemple d’utilisation » ci-dessous.


**Distinction entre un nœud arrêté et un nœud en panne** Si un nœud est arrêté (*stopped*) à l’aide de l’API Node Transition, la sortie d’une requête de nœud (managée : [GetNodeListAsync()][nodequery], PowerShell: [Get-ServiceFabricNode][nodequeryps]) indiquera que ce nœud a une propriété *IsStopped* dont la valeur est true.  Cela est différent de la valeur de la propriété *NodeStatus*, qui indiquera *Down*.  Si la valeur de la propriété *NodeStatus* est *Down*, mais celle de *IsStopped* est false, cela signifie que le nœud n’a pas été arrêté à l’aide de l’API de transition de nœud et est *en panne* pour une autre raison.  Si la valeur de la propriété *IsStopped* est true et celle de la propriété *NodeStatus* est *Down*, cela signifie que le nœud a été arrêté à l’aide de l’API de transition de nœud.

Démarrer un nœud *arrêté* à l’aide de l’API de transition de nœud le fait de nouveau fonctionner en tant que membre normal du cluster.  La sortie de l’API de requête de nœud indiquera *IsStopped* comme false, et *NodeStatus* comme un élément qui n’est pas en panne (mais plutôt en fonctionnement).


**Durée limitée** Lors de l’utilisation de l’API de transition de nœud pour arrêter un nœud, un des paramètres requis, *stopNodeDurationInSeconds*, représente la durée en secondes pendant laquelle le nœud reste *arrêté*.  Cette valeur doit être dans la plage autorisée, c’est-à-dire entre 600 et 14 400.  Une fois ce délai expiré, le nœud va redémarrer automatiquement et revenir à l’état de fonctionnement.  Reportez-vous à l’exemple 1 ci-dessous pour obtenir un exemple d’utilisation.

> [!WARNING]
> Évitez de mélanger les API de transition de nœud et les API de démarrage et d’arrêt de nœud.  Il est recommandé d’utiliser uniquement l’API de transition de nœud.  > Si un nœud a déjà été arrêté à l’aide de l’API d’arrêt de nœud, il doit être démarré en utilisant l’API de démarrage de nœud avant d’utiliser les API de transition de nœud.

> [!WARNING]
> Il est impossible d’appeler plusieurs API de transition de nœud sur le même nœud en parallèle.  Dans une telle situation, l’API de transition de nœud renverra une FabricException avec une propriété ErrorCode ayant la valeur NodeTransitionInProgress.  Une fois qu’une transition de nœud a démarré sur un nœud spécifique, vous devez patienter jusqu'à ce que l’opération atteigne un état terminal (Completed, Faulted ou ForceCancelled) avant de démarrer une nouvelle transition sur le même nœud.  Les appels de transition de nœud parallèles sur des nœuds différents sont autorisés.


#### <a name="sample-usage"></a>Exemple d’utilisation


**Exemple 1** : l’exemple suivant utilise l’API de transition de nœud pour arrêter un nœud.

```csharp
        // Helper function to get information about a node
        static Node GetNodeInfo(FabricClient fc, string node)
        {
            NodeList n = null;
            while (n == null)
            {
                n = fc.QueryManager.GetNodeListAsync(node).GetAwaiter().GetResult();
                Task.Delay(TimeSpan.FromSeconds(1)).GetAwaiter();
            };

            return n.FirstOrDefault();
        }

        static async Task WaitForStateAsync(FabricClient fc, Guid operationId, TestCommandProgressState targetState)
        {
            NodeTransitionProgress progress = null;

            do
            {
                bool exceptionObserved = false;
                try
                {
                    progress = await fc.TestManager.GetNodeTransitionProgressAsync(operationId, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                    exceptionObserved = true;
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                    exceptionObserved = true;
                }

                if (!exceptionObserved)
                {
                    Console.WriteLine("Current state of operation '{0}': {1}", operationId, progress.State);

                    if (progress.State == TestCommandProgressState.Faulted)
                    {
                        // Inspect the progress object's Result.Exception.HResult to get the error code.
                        Console.WriteLine("'{0}' failed with: {1}, HResult: {2}", operationId, progress.Result.Exception, progress.Result.Exception.HResult);

                        // ...additional logic as required
                    }

                    if (progress.State == targetState)
                    {
                        Console.WriteLine("Target state '{0}' has been reached", targetState);
                        break;
                    }
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (true);
        }

        static async Task StopNodeAsync(FabricClient fc, string nodeName, int durationInSeconds)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            // Create a Guid
            Guid guid = Guid.NewGuid();

            // Create a NodeStopDescription object, which will be used as a parameter into StartNodeTransition
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, durationInSeconds);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStopDescription from above, which will stop the target node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    // This is retryable
                }
                catch (FabricTransientException fte)
                {
                    // This is retryable
                }

                // Backoff
                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Exemple 2** : l’exemple suivant démarre un nœud *arrêté*.  Il utilise des méthodes d’assistance du premier exemple.

```csharp
        static async Task StartNodeAsync(FabricClient fc, string nodeName)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = n.NodeInstanceId;

            // Create a NodeStartDescription object, which will be used as a parameter into StartNodeTransition
            NodeStartDescription description = new NodeStartDescription(guid, n.NodeName, nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Exemple 3** : l’exemple suivant illustre une utilisation incorrecte.  Cette utilisation est incorrecte car la propriété *stopDurationInSeconds* qu’il fournit est supérieure à la plage autorisée.  Étant donné que StartNodeTransitionAsync() échoue avec une erreur irrécupérable, l’opération n’a pas été acceptée et l’API de progression ne doit pas être appelée.  Cet exemple utilise des méthodes d’assistance du premier exemple.

```csharp
        static async Task StopNodeWithOutOfRangeDurationAsync(FabricClient fc, string nodeName)
        {
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();

            // Use an out of range value for stopDurationInSeconds to demonstrate error
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, 99999);

            try
            {
                await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
            }

            catch (FabricException e)
            {
                Console.WriteLine("Caught {0}", e);
                Console.WriteLine("ErrorCode {0}", e.ErrorCode);
                // Output:
                // Caught System.Fabric.FabricException: System.Runtime.InteropServices.COMException (-2147017629)
                // StopDurationInSeconds is out of range ---> System.Runtime.InteropServices.COMException: Exception from HRESULT: 0x80071C63
                // << Parts of exception omitted>>
                //
                // ErrorCode InvalidDuration
            }
        }
```

**Exemple 4** : l’exemple suivant montre les informations d’erreur qui seront renvoyées par l’API de progression de transition de nœud lorsque l’opération lancée par l’API de transition de nœud est acceptée, mais échoue en cours d’exécution.  Dans ce cas, elle échoue parce que l’API de transition de nœud tente de démarrer un nœud qui n’existe pas.  Cet exemple utilise des méthodes d’assistance du premier exemple.

```csharp
        static async Task StartNodeWithNonexistentNodeAsync(FabricClient fc)
        {
            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = 12345;

            // Intentionally use a nonexistent node
            NodeStartDescription description = new NodeStartDescription(guid, "NonexistentNode", nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.  In this case, it will end up in the Faulted state since the node does not exist.
            // When StartNodeTransitionProgressAsync()'s returned progress object has a State if Faulted, inspect the progress object's Result.Exception.HResult to get the error code.
            // In this case, it will be NodeNotFound.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Faulted).ConfigureAwait(false);
        }
```

[stopnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[stopnodeps]: /previous-versions/azure/mt125982(v=azure.100)
[startnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[startnodeps]: /previous-versions/azure/mt163520(v=azure.100)
[nodequery]: /dotnet/api/system.fabric.fabricclient.queryclient
[nodequeryps]: /powershell/module/servicefabric/get-servicefabricnode
[snt]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
[gntp]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
