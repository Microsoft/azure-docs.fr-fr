---
title: Orchestrations secondaires pour Fonctions durables - Azure
description: Guide pratique pour appeler des orchestrations à partir d’orchestrations dans l’extension Fonctions durables pour Azure Functions.
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: 5625bc2ddfa4b6f527ca16f19f33d257a1834d4b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85340811"
---
# <a name="sub-orchestrations-in-durable-functions-azure-functions"></a>Orchestrations secondaires dans Fonctions durables (Azure Functions)

En plus d’appeler des fonctions d’activité, les fonctions d’orchestrateur peuvent appeler d’autres fonctions d’orchestrateur. Il est par exemple possible de créer une grande orchestration à partir d’une bibliothèque de petites fonctions d’orchestrateur. Ou vous pouvez exécuter en parallèle plusieurs instances d’une fonction d’orchestrateur.

Une fonction d’orchestrateur peut en appeler une autre avec les méthodes `CallSubOrchestratorAsync` ou `CallSubOrchestratorWithRetryAsync` en .NET, ou les méthodes `callSubOrchestrator` ou `callSubOrchestratorWithRetry` en JavaScript. L’article [Gestion des erreurs et compensation](durable-functions-error-handling.md#automatic-retry-on-failure) fournit des informations supplémentaires sur la nouvelle tentative automatique.

Les fonctions d’orchestrateur secondaires se comportent comme des fonctions d’activité du point de vue de l’appelant. Elles peuvent retourner une valeur, lever une exception et être attendues par la fonction d’orchestrateur parente. 

> [!NOTE]
> Les sous-orchestrations sont actuellement prises en charge dans .NET et JavaScript.

## <a name="example"></a>Exemple

L’exemple suivant montre un scénario IoT (« Internet of Things ») dans lequel plusieurs appareils doivent être configurés. La fonction suivante représente le workflow de configuration qui doit être exécuté pour chaque appareil :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
public static async Task DeviceProvisioningOrchestration(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string deviceId = context.GetInput<string>();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    Uri sasUrl = await context.CallActivityAsync<Uri>("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    await context.CallActivityAsync("SendPackageUrlToDevice", Tuple.Create(deviceId, sasUrl));

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    await context.WaitForExternalEvent<bool>("DownloadCompletedAck");

    // Step 4: ...
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceId = context.df.getInput();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    const sasUrl = yield context.df.callActivity("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    yield context.df.callActivity("SendPackageUrlToDevice", { id: deviceId, url: sasUrl });

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    yield context.df.waitForExternalEvent("DownloadCompletedAck");

    // Step 4: ...
});
```

---

Cette fonction d’orchestrateur peut être utilisée en l’état pour la configuration unique d’un appareil, ou elle peut faire partie d’une orchestration plus importante. Dans ce cas, la fonction d’orchestrateur parente peut planifier des instances de `DeviceProvisioningOrchestration` à l’aide de l’API `CallSubOrchestratorAsync` (.NET) or `callSubOrchestrator` (JavaScript).

Voici un exemple montrant comment exécuter en parallèle plusieurs fonctions d’orchestrateur.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ProvisionNewDevices")]
public static async Task ProvisionNewDevices(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string[] deviceIds = await context.CallActivityAsync<string[]>("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    var provisioningTasks = new List<Task>();
    foreach (string deviceId in deviceIds)
    {
        Task provisionTask = context.CallSubOrchestratorAsync("DeviceProvisioningOrchestration", deviceId);
        provisioningTasks.Add(provisionTask);
    }

    await Task.WhenAll(provisioningTasks);

    // ...
}
```

> [!NOTE]
> Les exemples C# précédents portent sur Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceIds = yield context.df.callActivity("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    const provisioningTasks = [];
    var id = 0;
    for (const deviceId of deviceIds) {
        const child_id = context.df.instanceId+`:${id}`;
        const provisionTask = context.df.callSubOrchestrator("DeviceProvisioningOrchestration", deviceId, child_id);
        provisioningTasks.push(provisionTask);
        id++;
    }

    yield context.df.Task.all(provisioningTasks);

    // ...
});
```

---

> [!NOTE]
> Des sous-orchestrations doivent être définies dans la même application de fonction que l’orchestration parente. Si vous avez besoin d’appeler et d’attendre des orchestrations dans une autre application de fonction, envisagez d’utiliser la prise en charge intégrée des API HTTP et le modèle d’interrogation de consommateur HTTP 202. Pour plus d’informations, consultez la rubrique [Fonctionnalités HTTP](durable-functions-http-features.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment définir un état d’orchestration personnalisé](durable-functions-custom-orchestration-status.md)
