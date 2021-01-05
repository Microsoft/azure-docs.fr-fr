---
title: Déploiement sans temps d’arrêt pour Durable Functions
description: Apprenez à activer l’orchestration Durable Functions pour les déploiements sans temps d’arrêt.
author: tsushi
ms.topic: conceptual
ms.date: 10/10/2019
ms.author: azfuncdf
ms.custom: fasttrack-edit
ms.openlocfilehash: 2c96f2cc37c47c77b82ca86d5fd0295f0c66a896
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009481"
---
# <a name="zero-downtime-deployment-for-durable-functions"></a>Déploiement sans temps d’arrêt pour Durable Functions

Pour permettre un bon fonctionnement du [modèle d’exécution fiable](./durable-functions-orchestrations.md) de Durable Functions, les orchestrations doivent être déterministes, ce qui représente un point supplémentaire à prendre en compte quand vous déployez des mises à jour. Quand un déploiement contient des changements apportés aux signatures de fonction d’activité ou à la logique d’orchestrateur, les instances d’orchestration actives cessent de fonctionner correctement. Cette situation est particulièrement problématique pour les instances d’orchestration de longue durée, qui peuvent représenter des heures ou des jours de travail.

Pour éviter ces échecs, vous avez deux options : 
- Retardez votre déploiement jusqu’à ce que toutes les instances d’orchestration en cours d’exécution soient terminées.
- Assurez-vous que toutes les instances d’orchestration en cours d’exécution utilisent les versions existantes de vos fonctions. 

Le tableau suivant compare les trois stratégies principales qui permettent d’effectuer un déploiement sans temps d’arrêt pour Durable Functions : 

| Stratégie |  Quand l’utiliser | Avantages | Inconvénients |
| -------- | ------------ | ---- | ---- |
| [Contrôle de version](#versioning) |  Applications qui ne font pas l’objet de [changements cassants](durable-functions-versioning.md). | Simple à implémenter. |  Augmentation de la taille de l’application de fonction en mémoire et du nombre de fonctions.<br/>Duplication de code. |
| [Vérification de l’état sur un emplacement](#status-check-with-slot) | Système qui n’a pas d’orchestrations de longue durée de plus de 24 heures ou qui se chevauchent fréquemment. | Code base simple.<br/>Ne nécessite pas de gestion d’application de fonction supplémentaire. | Nécessite un compte de stockage supplémentaire ou une gestion du hub de tâches.<br/>Nécessite des périodes pendant lesquelles aucune orchestration n’est en cours d’exécution. |
| [Routage d’applications](#application-routing) | Système qui n’a pas de périodes pendant lesquelles les orchestrations ne sont pas exécutées, par exemple les périodes ayant des orchestrations de plus de 24 heures ou dont les orchestrations se chevauchent fréquemment. | Prend en charge les nouvelles versions des systèmes dont les orchestrations s’exécutent en permanence et comportent des changements cassants. | Nécessite un routeur d’applications intelligent.<br/>Peut atteindre la limite du nombre d’applications de fonction autorisées par votre abonnement. La valeur par défaut est 100. |

## <a name="versioning"></a>Contrôle de version

Définissez de nouvelles versions de vos fonctions, et laissez les anciennes versions dans votre application de fonction. Comme vous pouvez le voir dans le diagramme, la version d’une fonction devient partie intégrante de son nom. Dans la mesure où les versions précédentes des fonctions sont conservées, les instances d’orchestration actives peuvent continuer à les référencer. Pendant ce temps, les demandes de nouvelles instances d’orchestration appellent la dernière version, que votre fonction cliente d’orchestration peut référencer à partir d’un paramètre d’application.

![Stratégie de gestion de versions](media/durable-functions-zero-downtime-deployment/versioning-strategy.png)

Dans cette stratégie, chaque fonction doit être copiée, et ses références à d’autres fonctions doivent être mises à jour. Vous pouvez faciliter la tâche en écrivant un script. Voici un [exemple de projet](https://github.com/TsuyoshiUshio/DurableVersioning) avec un script de migration.

>[!NOTE]
>Cette stratégie utilise des emplacements de déploiement pour éviter les temps d’arrêt durant le déploiement. Pour plus d’informations sur la création d’emplacements de déploiement et sur leur utilisation, consultez [Emplacements de déploiement Azure Functions](../functions-deployment-slots.md).

## <a name="status-check-with-slot"></a>Vérification de l’état sur un emplacement

Pendant que la version actuelle de votre application de fonction s’exécute dans votre emplacement de production, déployez la nouvelle version de votre application de fonction sur votre emplacement de préproduction. Avant de permuter vos emplacements de production et de préproduction, vérifiez si des instances d’orchestration sont en cours d’exécution. Une fois que toutes les instances d’orchestration ont fini de s’exécuter, vous pouvez effectuer l’échange. Cette stratégie fonctionne quand vous avez des périodes prévisibles pendant lesquelles aucune instance d’orchestration n’est active. Il s’agit de la meilleure approche quand vos orchestrations ne sont pas longues et que leurs exécutions ne se chevauchent pas fréquemment.

### <a name="function-app-configuration"></a>Configuration de l’application de fonction

Utilisez la procédure suivante pour mettre en œuvre ce scénario.

1. [Ajoutez des emplacements de déploiement](../functions-deployment-slots.md#add-a-slot) à votre application de fonction pour la préproduction et la production.

1. Pour chaque emplacement, affectez au [paramètre d’application AzureWebJobsStorage](../functions-app-settings.md#azurewebjobsstorage) la chaîne de connexion d’un compte de stockage partagé. Cette chaîne de connexion de compte de stockage est utilisée par le runtime Azure Functions. Ce compte est utilisé par le runtime d’Azure Functions. Il gère les clés de la fonction.

1. Pour chaque emplacement, créez un paramètre d’application, par exemple, `DurableManagementStorage`. Définissez sa valeur en fonction de la chaîne de connexion de différents comptes de stockage. Ces comptes de stockage sont utilisés par l’extension Durable Functions pour l’[exécution fiable](./durable-functions-orchestrations.md). Utilisez un compte de stockage distinct pour chaque emplacement. Ne marquez pas ce paramètre en tant que paramètre d’emplacement de déploiement.

1. Dans la [section durableTask du fichier host.json](durable-functions-bindings.md#hostjson-settings) de votre application de fonction, spécifiez `connectionStringName` (Durable 2.x) ou `azureStorageConnectionStringName` (Durable 1.x) en tant que nom du paramètre d’application créé à l’étape 3.

Le diagramme suivant illustre la configuration décrite pour les emplacements de déploiement et les comptes de stockage. Dans ce scénario de prédéploiement potentiel, la version 2 d’une application de fonction s’exécute dans l’emplacement de production, alors que la version 1 reste dans l’emplacement de préproduction.

![Emplacements de déploiement et comptes de stockage](media/durable-functions-zero-downtime-deployment/deployment-slot.png)

### <a name="hostjson-examples"></a>Exemples host.json

Les fragments JSON suivants sont des exemples de paramètres de chaîne de connexion dans le fichier *host.json*.

#### <a name="functions-20"></a>Functions 2.0

```json
{
  "version": 2.0,
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub",
      "storageProvider": {
        "connectionStringName": "DurableManagementStorage"
      }
    }
  }
}
```

#### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "azureStorageConnectionStringName": "DurableManagementStorage"
  }
}
```

### <a name="cicd-pipeline-configuration"></a>Configuration du pipeline CI/CD

Configurez votre pipeline CI/CD pour le déploiement uniquement quand votre application de fonction n’a aucune instance d’orchestration en attente ou en cours d’exécution. Quand vous utilisez Azure Pipelines, vous pouvez créer une fonction qui vérifie ces conditions, comme dans l’exemple suivant :

```csharp
[FunctionName("StatusCheck")]
public static async Task<IActionResult> StatusCheck(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    var runtimeStatus = new List<OrchestrationRuntimeStatus>();

    runtimeStatus.Add(OrchestrationRuntimeStatus.Pending);
    runtimeStatus.Add(OrchestrationRuntimeStatus.Running);

    var result = await client.ListInstancesAsync(new OrchestrationStatusQueryCondition() { RuntimeStatus = runtimeStatus }, CancellationToken.None);
    return (ActionResult)new OkObjectResult(new { HasRunning = result.DurableOrchestrationState.Any() });
}
```

Configurez ensuite la porte de préproduction pour qu’elle attende la fin de l’exécution de toutes les orchestrations en cours. Pour plus d’informations, consultez [Contrôle du déploiement de mises en production à l’aide de portes](/azure/devops/pipelines/release/approvals/gates?view=azure-devops)

![Porte de déploiement](media/durable-functions-zero-downtime-deployment/deployment-gate.png)

Azure Pipelines vérifie si votre application de fonction exécute des instances d’orchestration avant le démarrage du déploiement.

![Porte de déploiement (en cours d’exécution)](media/durable-functions-zero-downtime-deployment/deployment-gate-2.png)

À présent, la nouvelle version de votre application de fonction doit être déployée sur l’emplacement de préproduction.

![Emplacements de préproduction](media/durable-functions-zero-downtime-deployment/deployment-slot-2.png)

Enfin, permutez les emplacements. 

Les paramètres d’application qui ne sont pas marqués en tant que paramètres d’emplacement de déploiement sont également permutés. Ainsi, l’application version 2 conserve sa référence au compte de stockage A. Dans la mesure où l’état d’orchestration est suivi dans le compte de stockage, les orchestrations exécutées sur l’application version 2 continuent de s’exécuter dans le nouvel emplacement, sans interruption.

![Emplacement de déploiement](media/durable-functions-zero-downtime-deployment/deployment-slot-3.png)

Si vous souhaitez utiliser le même compte de stockage pour les deux emplacements, changez les noms de vos hubs de tâches. Dans ce cas, vous devez gérer l’état de vos emplacements et les paramètres HubName de votre application. Pour en savoir plus, consultez [Hubs de tâches dans Durable Functions](durable-functions-task-hubs.md).

## <a name="application-routing"></a>Routage d’applications

Cette stratégie est la plus complexe. Toutefois, elle peut être utilisée pour les applications de fonction qui n’ont pas d’intervalles entre les exécutions d’orchestrations.

Pour cette stratégie, vous devez créer un *routeur d’applications* devant votre extension Durable Functions. Ce routeur peut être implémenté avec Durable Functions. Le routeur a la responsabilité d’effectuer les opérations suivantes :

* Déployez l’application de fonction.
* Gérer la version de Durable Functions. 
* Router les demandes d’orchestration vers les applications de fonction.

La première fois qu’une demande d’orchestration est reçue, le routeur effectue les tâches suivantes :

1. Il crée une application de fonction dans Azure.
2. Il déploie le code de votre application de fonction sur la nouvelle application de fonction dans Azure.
3. Il transmet la demande d’orchestration à la nouvelle application.

Le routeur gère l’état de la version du code d’application déployée sur l’application de fonction dans Azure.

![Routage d’applications (première fois)](media/durable-functions-zero-downtime-deployment/application-routing.png)

Le routeur dirige les demandes de déploiement et d’orchestration vers l’application de fonction appropriée selon la version envoyée avec la demande. Il ignore la version du correctif.

Quand vous déployez une nouvelle version de votre application sans changement cassant, vous pouvez incrémenter la version du correctif. Le routeur se déploie sur votre application de fonction existante et envoie les demandes relatives à l’ancienne et à la nouvelle version du code, qui sont routées vers la même application de fonction.

![Routage d’applications (aucun changement cassant)](media/durable-functions-zero-downtime-deployment/application-routing-2.png)

Quand vous déployez une nouvelle version de votre application avec un changement cassant, vous pouvez incrémenter la version majeure ou mineure. Le routeur d’applications crée ensuite une application de fonction dans Azure, s’y déploie et route vers celle-ci les demandes relatives à la nouvelle version de votre application. Dans le diagramme suivant, l’exécution d’orchestrations sur la version 1.0.1 de l’application continue de s’effectuer. Toutefois, les demandes relatives à la version 1.1.0 sont routées vers la nouvelle application de fonction.

![Routage d’applications (changement cassant)](media/durable-functions-zero-downtime-deployment/application-routing-3.png)

Le routeur supervise l’état des orchestrations sur la version 1.0.1, puis supprime les applications à la fin de l’exécution de toutes les orchestrations. 

### <a name="tracking-store-settings"></a>Suivi des paramètres de stockage

Chaque application de fonction doit utiliser des files d’attente de planification distinctes, éventuellement dans des comptes de stockage distincts. Si vous souhaitez interroger toutes les instances d’orchestration parmi toutes les versions de votre application, vous pouvez partager les tables d’instance et d’historique entre vos applications de fonction. Vous pouvez partager les tables en configurant les paramètres `trackingStoreConnectionStringName` et `trackingStoreNamePrefix` dans le fichier de [paramètres host.json](durable-functions-bindings.md#host-json) pour utiliser partout les mêmes valeurs.

Pour plus d’informations, consultez [Gérer des instances dans Durable Functions dans Azure](durable-functions-instance-management.md).

![Suivi des paramètres de stockage](media/durable-functions-zero-downtime-deployment/tracking-store-settings.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gestion de versions de Durable Functions](durable-functions-versioning.md)
