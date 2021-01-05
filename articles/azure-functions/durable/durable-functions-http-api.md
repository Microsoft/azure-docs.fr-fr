---
title: API HTTP dans Durable Functions - Azure Functions
description: Découvrez comment implémenter des API HTTP dans l’extension Fonctions durables d’Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: azfuncdf
ms.openlocfilehash: 4e4081ecca4714c713d105d363a83a4f96a0d3fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84697841"
---
# <a name="http-api-reference"></a>Référence sur l'API HTTP

L’extension Durable Functions expose un ensemble d’API HTTP intégrées qui peuvent être utilisées pour effectuer des tâches de gestion sur des [orchestrations](durable-functions-types-features-overview.md#orchestrator-functions), des [entités](durable-functions-types-features-overview.md#entity-functions) et des [hubs de tâches](durable-functions-task-hubs.md). Ces API HTTP sont des webhooks d’extensibilité qui sont autorisés par l’hôte Azure Functions mais gérés directement par l’extension Durable Functions.

Toutes les API HTTP implémentées par l’extension requièrent les paramètres suivants. Le type de données de tous les paramètres est `string`.

| Paramètre        | Type de paramètre  | Description |
|------------------|-----------------|-------------|
| **`taskHub`**    | Chaîne de requête    | Le nom du [hub de tâches](durable-functions-task-hubs.md). S’il n’est pas spécifié, le nom de hub de tâches de l’application de fonction en cours est supposé. |
| **`connection`** | Chaîne de requête    | Le **nom** de la chaîne de connexion du compte de stockage. Si elle n’est pas spécifiée, la chaîne de connexion par défaut pour l’application de la fonction est supposée. |
| **`systemKey`**  | Chaîne de requête    | La clé d’autorisation requise pour appeler l’API. |

`systemKey` est une clé d’autorisation automatiquement générée par l’hôte Azure Functions. Elle accorde l’accès aux API de l’extension Tâche durable et peut être gérée de la même façon que les [autres clés d’autorisation](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Key-management-API). Vous pouvez générer des URL qui contiennent les valeurs de chaîne de requête `taskHub`, `connection` et `systemKey` correctes à l’aide des API de [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client), telles que les API `CreateCheckStatusResponse` et `CreateHttpManagementPayload` dans .NET ou les API `createCheckStatusResponse` et `createHttpManagementPayload` dans JavaScript.

Les quelques sections suivantes présentent les API HTTP spécifiques prises en charge par l’extension et fournissent des exemples d’utilisation.

## <a name="start-orchestration"></a>Démarrer l’orchestration

Démarre l’exécution d’une nouvelle instance de la fonction d’orchestrateur spécifiée.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
POST /admin/extensions/DurableTaskExtension/orchestrators/{functionName}/{instanceId?}
     ?taskHub={taskHub}
     &connection={connectionName}
     &code={systemKey}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
POST /runtime/webhooks/durabletask/orchestrators/{functionName}/{instanceId?}
     ?taskHub={taskHub}
     &connection={connectionName}
     &code={systemKey}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ              | Type de paramètre  | Description |
|--------------------|-----------------|-------------|
| **`functionName`** | URL             | Nom de la fonction d’orchestrateur à démarrer. |
| **`instanceId`**   | URL             | Paramètre facultatif. L’ID de l’instance d’orchestration. S’il n’est pas spécifié, la fonction d’orchestrateur démarre avec un ID d’instance aléatoire. |
| **`{content}`**    | Contenu de la demande | facultatif. Entrée de la fonction d’orchestrateur au format JSON. |

### <a name="response"></a>response

Plusieurs valeurs de code d’état possibles peuvent être retournées.

* **HTTP 202 (acceptée)** : La fonction d’orchestrateur spécifiée a été planifiée pour commencer à s’exécuter. L’en-tête de réponse `Location` contient une URL pour interroger l’état de l’orchestration.
* **HTTP 400 (requête incorrecte)** : La fonction d’orchestrateur spécifiée n’existe pas, l’ID d’instance spécifié n’était pas valide ou le contenu de la demande n’était pas un contenu JSON valide.

Voici un exemple de demande qui démarre une fonction d’orchestrateur `RestartVMs` et inclut la charge utile d’objet JSON :

```http
POST /runtime/webhooks/durabletask/orchestrators/RestartVMs?code=XXX
Content-Type: application/json
Content-Length: 83

{
    "resourceGroup": "myRG",
    "subscriptionId": "111deb5d-09df-4604-992e-a968345530a9"
}
```

La charge utile de réponse pour les cas **HTTP 202** est un objet JSON avec les champs suivants :

| Champ                       | Description                          |
|-----------------------------|--------------------------------------|
| **`id`**                    |L’ID de l’instance d’orchestration. |
| **`statusQueryGetUri`**     |L’URL de l’état de l’instance d’orchestration. |
| **`sendEventPostUri`**      |L’URL « Raise event » de l’instance d’orchestration. |
| **`terminatePostUri`**      |L’URL « d’arrêt » de l’instance d’orchestration. |
| **`purgeHistoryDeleteUri`** |URL de « purge de l’historique » de l’instance d’orchestration. |
| **`rewindPostUri`**         |(préversion) URL de « rembobinage » de l’instance d’orchestration. |

Le type de données de tous les champs est `string`.

Voici un exemple de charge utile de réponse pour une instance d’orchestration avec l’ID `abc123` (mis en forme pour une meilleure lisibilité) :

```http
{
    "id": "abc123",
    "purgeHistoryDeleteUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/raiseEvent/{eventName}?code=XXX",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/terminate?reason={text}&code=XXX"
}
```

La réponse HTTP est destinée à être compatible avec le *modèle d’interrogation de consommateur*. Elle comprend également les en-têtes de réponse notables suivants :

* **Emplacement** : URL du point de terminaison de l’état. Cette URL contient la même valeur que le champ `statusQueryGetUri`.
* **Retry-After** : Nombre de secondes à attendre entre les opérations d’interrogation. La valeur par défaut est `10`.

Pour plus d’informations sur le modèle d’interrogation HTTP asynchrone, consultez la documentation sur le [suivi des opérations asynchrones HTTP](durable-functions-http-features.md#async-operation-tracking).

## <a name="get-instance-status"></a>Obtenir l’état de l’instance

Obtient l'état d'une instance d’orchestration spécifiée.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
GET /admin/extensions/DurableTaskExtension/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &showHistory=[true|false]
    &showHistoryOutput=[true|false]
    &showInput=[true|false]
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
GET /runtime/webhooks/durabletask/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &showHistory=[true|false]
    &showHistoryOutput=[true|false]
    &showInput=[true|false]
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ                   | Type de paramètre  | Description |
|-------------------------|-----------------|-------------|
| **`instanceId`**        | URL             | L’ID de l’instance d’orchestration. |
| **`showInput`**         | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `false`, la fonction n’est pas incluse dans la charge utile de la réponse.|
| **`showHistory`**       | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `true`, l’historique d’exécution de l’orchestration est inclus dans la charge utile de réponse.|
| **`showHistoryOutput`** | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `true`, les sorties de la fonction sont incluses dans l’historique d’exécution de l’orchestration.|
| **`createdTimeFrom`**   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées qui ont été créées pendant ou après l’horodatage ISO8601 donné.|
| **`createdTimeTo`**     | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées qui ont été créées pendant ou avant l’horodatage ISO8601 donné.|
| **`runtimeStatus`**     | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées selon leur état d’exécution. Pour obtenir la liste des valeurs d’état d’exécution possibles, consultez l’article [Interrogation des instances](durable-functions-instance-management.md). |

### <a name="response"></a>response

Plusieurs valeurs de code d’état possibles peuvent être retournées.

* **HTTP 200 (OK)** : L’instance spécifiée indique un état terminé.
* **HTTP 202 (acceptée)** : L’instance spécifiée est en cours d’exécution.
* **HTTP 400 (requête incorrecte)** : L’instance spécifiée a échoué ou a été arrêtée.
* **HTTP 404 (introuvable)** : L’instance spécifiée n’existe pas ou n’a pas démarré.
* **HTTP 500 (erreur interne du serveur)** : L’instance spécifiée a échoué avec une exception non prise en charge.

La charge utile de réponse pour les cas **HTTP 200** et **HTTP 202** est un objet JSON avec les champs suivants :

| Champ                 | Type de données | Description |
|-----------------------|-----------|-------------|
| **`runtimeStatus`**   | string    | L’état d’exécution de l’instance. Les valeurs sont *Running*, *Pending*, *Failed*, *Canceled*, *Terminated*, *Completed*. |
| **`input`**           | JSON      | Les données JSON utilisées pour initialiser l’instance. Ce champ est `null` si le paramètre de chaîne de requête `showInput` est défini sur `false`.|
| **`customStatus`**    | JSON      | Données JSON utilisées pour l’état d’orchestration personnalisé. Ce champ est `null` s’il n’est pas défini. |
| **`output`**          | JSON      | La sortie JSON de l’instance. Ce champ est `null` si l’instance n’est pas dans un état terminé. |
| **`createdTime`**     | string    | Heure à laquelle l’instance a été créée. Utilise la notation étendue ISO 8601. |
| **`lastUpdatedTime`** | string    | Heure du dernier état persistant de l’instance. Utilise la notation étendue ISO 8601. |
| **`historyEvents`**   | JSON      | Tableau JSON contenant l’historique d’exécution de l’orchestration. Ce champ est `null`, sauf si le paramètre de chaîne de requête `showHistory` a la valeur `true`. |

Voici un exemple de charge utile de réponse incluant l’historique et les sorties de l’activité d’exécution d’orchestration (mis en forme pour une meilleure lisibilité) :

```json
{
  "createdTime": "2018-02-28T05:18:49Z",
  "historyEvents": [
      {
          "EventType": "ExecutionStarted",
          "FunctionName": "E1_HelloSequence",
          "Timestamp": "2018-02-28T05:18:49.3452372Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Tokyo!",
          "ScheduledTime": "2018-02-28T05:18:51.3939873Z",
          "Timestamp": "2018-02-28T05:18:52.2895622Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello Seattle!",
          "ScheduledTime": "2018-02-28T05:18:52.8755705Z",
          "Timestamp": "2018-02-28T05:18:53.1765771Z"
      },
      {
          "EventType": "TaskCompleted",
          "FunctionName": "E1_SayHello",
          "Result": "Hello London!",
          "ScheduledTime": "2018-02-28T05:18:53.5170791Z",
          "Timestamp": "2018-02-28T05:18:53.891081Z"
      },
      {
          "EventType": "ExecutionCompleted",
          "OrchestrationStatus": "Completed",
          "Result": [
              "Hello Tokyo!",
              "Hello Seattle!",
              "Hello London!"
          ],
          "Timestamp": "2018-02-28T05:18:54.3660895Z"
      }
  ],
  "input": null,
  "customStatus": { "nextActions": ["A", "B", "C"], "foo": 2 },
  "lastUpdatedTime": "2018-02-28T05:18:54Z",
  "output": [
      "Hello Tokyo!",
      "Hello Seattle!",
      "Hello London!"
  ],
  "runtimeStatus": "Completed"
}
```

La réponse **HTTP 202** inclut également un en-tête de réponse **Location** qui fait référence à la même URL que le champ `statusQueryGetUri` mentionné précédemment.

## <a name="get-all-instances-status"></a>Obtenir l’état de toutes les instances

Vous pouvez également interroger l’état de toutes les instances en supprimant l’élément `instanceId` de la demande « Obtenir l’état de l’instance ». Dans le cas présent, les paramètres de base sont les mêmes que ceux de « Obtenir l’état de l’instance ». Les paramètres de chaîne de requête dédiés au filtrage sont également pris en charge.

Une chose à retenir est que `connection` et `code` sont facultatifs. Si vous disposez d’une authentification anonyme sur la fonction, le paramètre `code` n’est pas nécessaire.
Si vous ne souhaitez pas utiliser une autre chaîne de connexion de stockage que celle définie dans le paramètre d’application AzureWebJobsStorage, vous pouvez ignorer en toute sécurité le paramètre de chaîne de requête de connexion.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
GET /admin/extensions/DurableTaskExtension/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
    &showInput=[true|false]
    &top={integer}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
GET /runtime/webhooks/durableTask/instances?
    taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
    &showInput=[true|false]
    &top={integer}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ                   | Type de paramètre  | Description |
|-------------------------|-----------------|-------------|
| **`instanceId`**        | URL             | L’ID de l’instance d’orchestration. |
| **`showInput`**         | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `false`, la fonction n’est pas incluse dans la charge utile de la réponse.|
| **`showHistory`**       | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `true`, l’historique d’exécution de l’orchestration est inclus dans la charge utile de réponse.|
| **`showHistoryOutput`** | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `true`, les sorties de la fonction sont incluses dans l’historique d’exécution de l’orchestration.|
| **`createdTimeFrom`**   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées qui ont été créées pendant ou après l’horodatage ISO8601 donné.|
| **`createdTimeTo`**     | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées qui ont été créées pendant ou avant l’horodatage ISO8601 donné.|
| **`runtimeStatus`**     | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances retournées selon leur état d’exécution. Pour obtenir la liste des valeurs d’état d’exécution possibles, consultez l’article [Interrogation des instances](durable-functions-instance-management.md). |
| **`top`**               | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, limite le nombre d’instances retournées par la requête. |

### <a name="response"></a>response

Voici un exemple de charges utiles de réponse incluant l’état de l’orchestration (mises en forme pour une meilleure lisibilité) :

```json
[
    {
        "instanceId": "7af46ff000564c65aafbfe99d07c32a5",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:39Z",
        "lastUpdatedTime": "2018-06-04T10:46:47Z"
    },
    {
        "instanceId": "80eb7dd5c22f4eeba9f42b062794321e",
        "runtimeStatus": "Running",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:28Z",
        "lastUpdatedTime": "2018-06-04T15:18:38Z"
    },
    {
        "instanceId": "9124518926db408ab8dfe84822aba2b1",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2018-06-04T10:46:54Z",
        "lastUpdatedTime": "2018-06-04T10:47:03Z"
    },
    {
        "instanceId": "d100b90b903c4009ba1a90868331b11b",
        "runtimeStatus": "Pending",
        "input": null,
        "customStatus": null,
        "output": null,
        "createdTime": "2018-06-04T15:18:39Z",
        "lastUpdatedTime": "2018-06-04T15:18:39Z"
    }
]
```

> [!NOTE]
> Cette opération peut être très gourmande en E/S pour le stockage Azure s’il y a un grand nombre de lignes dans la table d’instances. Vous trouverez plus d’informations sur la table d’instances dans la documentation [Performances et mise à l’échelle dans Fonctions durables (Azure Functions)](durable-functions-perf-and-scale.md#instances-table).
>

Si vous obtenez plus de résultats, un jeton de continuation est retourné dans l’en-tête de réponse.  Le nom de l’en-tête est `x-ms-continuation-token`.

Si vous définissez la valeur du jeton de continuation dans l’en-tête de la demande suivante, vous pouvez obtenir la page de résultats suivante. Cet en-tête de demande est également nommé `x-ms-continuation-token`.

## <a name="purge-single-instance-history"></a>Vider l’historique d’une instance unique

Supprime l’historique et les artefacts associés d’une instance d’orchestration spécifiée.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
DELETE /admin/extensions/DurableTaskExtension/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connection}
    &code={systemKey}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
DELETE /runtime/webhooks/durabletask/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connection}
    &code={systemKey}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ             | Type de paramètre  | Description |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | L’ID de l’instance d’orchestration. |

### <a name="response"></a>response

Les valeurs de code d’état HTTP peuvent être également être retournées.

* **HTTP 200 (OK)** : L’historique d’instance a été vidé avec succès.
* **HTTP 404 (introuvable)** : L’instance spécifiée n’existe pas.

La charge utile de réponse pour le cas **HTTP 200** est un objet JSON présentant le champ suivant :

| Champ                  | Type de données | Description |
|------------------------|-----------|-------------|
| **`instancesDeleted`** | entier   | Nombre d’instances supprimées. Dans le cas d’une instance unique, cette valeur doit toujours être `1`. |

Voici un exemple de charge utile de réponse (mis en forme pour une meilleure lisibilité) :

```json
{
    "instancesDeleted": 1
}
```

## <a name="purge-multiple-instance-histories"></a>Vider l’historique de plusieurs instances

Vous pouvez également supprimer l’historique et les artefacts associés de plusieurs instances au sein d’un hub de tâches en supprimant `{instanceId}` de la demande « Vider l’historique d’une instance unique ». Pour vider l’historique de certaines instances spécifiques, utilisez les mêmes filtres que ceux décrits dans la demande « Obtenir l’état de toutes les instances ».

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
DELETE /admin/extensions/DurableTaskExtension/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
DELETE /runtime/webhooks/durabletask/instances
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &createdTimeFrom={timestamp}
    &createdTimeTo={timestamp}
    &runtimeStatus={runtimeStatus1,runtimeStatus2,...}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ                 | Type de paramètre  | Description |
|-----------------------|-----------------|-------------|
| **`createdTimeFrom`** | Chaîne de requête    | Filtre la liste des instances vidées qui ont été créées pendant ou après l’horodatage ISO8601 donné.|
| **`createdTimeTo`**   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances vidées qui ont été créées pendant ou avant l’horodatage ISO8601 donné.|
| **`runtimeStatus`**   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, filtre la liste des instances vidées selon l’état de leur runtime. Pour obtenir la liste des valeurs d’état d’exécution possibles, consultez l’article [Interrogation des instances](durable-functions-instance-management.md). |

> [!NOTE]
> Cette opération peut être très gourmande en E/S pour le stockage Azure s’il y a un grand nombre de lignes dans les instances et/ou les tables d’historique. Vous trouverez plus d’informations sur ces tables dans la documentation [Performances et mise à l’échelle dans Fonctions durables (Azure Functions)](durable-functions-perf-and-scale.md#instances-table).

### <a name="response"></a>response

Les valeurs de code d’état HTTP peuvent être également être retournées.

* **HTTP 200 (OK)** : L’historique d’instance a été vidé avec succès.
* **HTTP 404 (introuvable)** : Aucune instance correspondant à l’expression de filtre n’a été trouvée.

La charge utile de réponse pour le cas **HTTP 200** est un objet JSON présentant le champ suivant :

| Champ                   | Type de données | Description |
|-------------------------|-----------|-------------|
| **`instancesDeleted`**  | entier   | Nombre d’instances supprimées. |

Voici un exemple de charge utile de réponse (mis en forme pour une meilleure lisibilité) :

```json
{
    "instancesDeleted": 250
}
```

## <a name="raise-event"></a>Raise event

Envoie un message de notification d’événement à une instance d’orchestration en cours d’exécution.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/raiseEvent/{eventName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/{eventName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ             | Type de paramètre  | Description |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | L’ID de l’instance d’orchestration. |
| **`eventName`**   | URL             | Le nom de l’événement que l’instance d’orchestration cible attend. |
| **`{content}`**   | Contenu de la demande | La charge utile de l’événement au format JSON. |

### <a name="response"></a>response

Plusieurs valeurs de code d’état possibles peuvent être retournées.

* **HTTP 202 (acceptée)** : L’événement déclenché a été accepté pour traitement.
* **HTTP 400 (requête incorrecte)** : Le contenu de la requête n’était pas de type `application/json` ou n’était pas un objet JSON valide.
* **HTTP 404 (introuvable)** : L’instance spécifiée est introuvable.
* **HTTP 410 (absente)** : L’instance spécifiée est terminée ou a échoué et ne peut traiter aucun événement déclenché.

Voici un exemple de requête qui envoie la chaîne JSON `"incr"` à une instance en attente d’un événement nommé **operation** :

```http
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/raiseEvent/operation?taskHub=DurableFunctionsHub&connection=Storage&code=XXX
Content-Type: application/json
Content-Length: 6

"incr"
```

Les réponses pour cette API sont vides.

## <a name="terminate-instance"></a>Arrêter une instance

Arrête une instance d’orchestration en cours d’exécution.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/terminate
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/terminate
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

Les paramètres de la demande pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que le paramètre unique suivant.

| Champ             | Type de paramètre  | Description |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | L’ID de l’instance d’orchestration. |
| **`reason`**      | Chaîne de requête    | facultatif. Motif d’arrêt de l’instance d’orchestration. |

### <a name="response"></a>response

Plusieurs valeurs de code d’état possibles peuvent être retournées.

* **HTTP 202 (acceptée)** : La demande d’arrêt a été acceptée pour traitement.
* **HTTP 404 (introuvable)** : L’instance spécifiée est introuvable.
* **HTTP 410 (absente)** : L’instance spécifiée est terminée ou a échoué.

Voici un exemple de demande qui met fin à une instance en cours d’exécution et affiche la raison **buggy** (bogué) :

```
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/terminate?reason=buggy&taskHub=DurableFunctionsHub&connection=Storage&code=XXX
```

Les réponses pour cette API sont vides.

## <a name="rewind-instance-preview"></a>Instance de rembobinage (préversion)

Restaure une instance d’orchestration ayant échoué dans un état en cours d’exécution au travers de la relecture des dernières opérations ayant échoué.

### <a name="request"></a>Requête

Pour la version 1.x du runtime Functions, la demande est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
POST /admin/extensions/DurableTaskExtension/instances/{instanceId}/rewind
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

En version 2.x du runtime Functions, le format d’URL comporte les mêmes paramètres, mais présente un préfixe légèrement différent :

```http
POST /runtime/webhooks/durabletask/instances/{instanceId}/rewind
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &reason={text}
```

Les paramètres de la demande pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que le paramètre unique suivant.

| Champ             | Type de paramètre  | Description |
|-------------------|-----------------|-------------|
| **`instanceId`**  | URL             | L’ID de l’instance d’orchestration. |
| **`reason`**      | Chaîne de requête    | facultatif. Motif de rembobinage de l’instance d’orchestration. |

### <a name="response"></a>response

Plusieurs valeurs de code d’état possibles peuvent être retournées.

* **HTTP 202 (acceptée)** : La demande de rembobinage a été acceptée pour traitement.
* **HTTP 404 (introuvable)** : L’instance spécifiée est introuvable.
* **HTTP 410 (absente)** : L’instance spécifiée est terminée.

Voici un exemple de requête qui rembobine une instance ayant échoué et affiche la raison **corrigé** :

```http
POST /admin/extensions/DurableTaskExtension/instances/bcf6fb5067b046fbb021b52ba7deae5a/rewind?reason=fixed&taskHub=DurableFunctionsHub&connection=Storage&code=XXX
```

Les réponses pour cette API sont vides.

## <a name="signal-entity"></a>Signaler l’entité

Envoie un message d’opération unidirectionnel à une [entité durable](durable-functions-types-features-overview.md#entity-functions). Si l’entité n’existe pas, elle est créée automatiquement.

> [!NOTE]
> Les entités durables sont disponibles à compter de Durable Functions 2.0.

### <a name="request"></a>Requête

La requête HTTP est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
POST /runtime/webhooks/durabletask/entities/{entityName}/{entityKey}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &op={operationName}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ             | Type de paramètre  | Description |
|-------------------|-----------------|-------------|
| **`entityName`**  | URL             | Nom (type) de l’entité. |
| **`entityKey`**   | URL             | Clé (ID unique) de l’entité. |
| **`op`**          | Chaîne de requête    | facultatif. Nom de l’opération définie par l’utilisateur à appeler. |
| **`{content}`**   | Contenu de la demande | La charge utile de l’événement au format JSON. |

Voici un exemple de requête qui envoie un message « Add » défini par l’utilisateur à une entité `Counter` appelée `steps`. Le contenu du message est la valeur `5`. Si l’entité n’existe pas encore, elle est créée par cette requête :

```http
POST /runtime/webhooks/durabletask/entities/Counter/steps?op=Add
Content-Type: application/json

5
```

> [!NOTE]
> Par défaut, avec des [entités basées sur une classe dans .NET](durable-functions-dotnet-entities.md#defining-entity-classes), la spécification de la valeur `op` de `delete` a pour effet de supprimer l’état d’une entité. Toutefois, si l’entité définit une opération nommée `delete`, cette opération définie par l’utilisateur sera appelée à la place.

### <a name="response"></a>response

Cette opération a plusieurs réponses possibles :

* **HTTP 202 (acceptée)** : L’opération de signal a été acceptée pour traitement asynchrone.
* **HTTP 400 (requête incorrecte)** : Le contenu de la requête n’était pas de type `application/json` ou n’était pas un objet JSON valide ou bien sa valeur `entityKey` n’était pas valide.
* **HTTP 404 (introuvable)** : Le `entityName` spécifié est introuvable.

Une requête HTTP réussie ne contient pas de contenu dans la réponse. Une requête HTTP ayant échoué peut contenir des informations d’erreur au format JSON dans le contenu de la réponse.

## <a name="get-entity"></a>Obtenir l’entité

Obtient l’état de l’entité spécifiée.

### <a name="request"></a>Requête

La requête HTTP est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
GET /runtime/webhooks/durabletask/entities/{entityName}/{entityKey}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
```

### <a name="response"></a>response

Cette opération a deux réponses possibles :

* **HTTP 200 (OK)** : l’entité spécifiée existe.
* **HTTP 404 (introuvable)** : l’entité spécifiée est introuvable.

Une réponse réussie contient l’état sérialisé JSON de l’entité en tant que contenu.

### <a name="example"></a>Exemple
L’exemple suivant de requête HTTP obtient l’état d’une entité `Counter` existante nommée `steps` :

```http
GET /runtime/webhooks/durabletask/entities/Counter/steps
```

Si l’entité `Counter` contenait simplement un nombre d’étapes enregistrées dans un champ `currentValue`, le contenu de la réponse pourrait ressembler à ce qui suit (mise en forme pour plus de lisibilité) :

```json
{
    "currentValue": 5
}
```

## <a name="list-entities"></a>Lister les entités

Vous pouvez interroger plusieurs entités à l’aide de leur nom ou de la date de la dernière opération.

### <a name="request"></a>Requête

La requête HTTP est mise en forme comme suit (plusieurs lignes sont affichées par souci de clarté) :

```http
GET /runtime/webhooks/durabletask/entities/{entityName}
    ?taskHub={taskHub}
    &connection={connectionName}
    &code={systemKey}
    &lastOperationTimeFrom={timestamp}
    &lastOperationTimeTo={timestamp}
    &fetchState=[true|false]
    &top={integer}
```

Les paramètres de requête pour cette API incluent l’ensemble par défaut mentionné précédemment, ainsi que les paramètres uniques suivants :

| Champ                       | Type de paramètre  | Description |
|-----------------------------|-----------------|-------------|
| **`entityName`**            | URL             | facultatif. Lorsqu’elle est spécifiée, filtre la liste des entités retournées par nom (sans respect de la casse). |
| **`fetchState`**            | Chaîne de requête    | Paramètre facultatif. Si la valeur est définie sur `true`, l’état de l’entité sera inclus dans la charge utile de réponse. |
| **`lastOperationTimeFrom`** | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, il filtre la liste des entités retournées qui ont traité des opérations après l’horodatage ISO8601 donné. |
| **`lastOperationTimeTo`**   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, il filtre la liste des entités retournées qui ont traité des opérations avant l’horodatage ISO8601 donné. |
| **`top`**                   | Chaîne de requête    | Paramètre facultatif. Lorsqu’il est spécifié, il limite le nombre d’entités retournées par la requête. |


### <a name="response"></a>response

Une réponse HTTP 200 réussie contient un tableau d’entités sérialisé par du code JSON et, éventuellement, l’état de chaque entité.

Par défaut, l’opération retourne les 100 premières entités qui correspondent aux critères de la requête. L’appelant peut spécifier une valeur de paramètre de chaîne de requête pour que `top` retourne un autre nombre maximal de résultats. Si tous les résultats ne sont pas retournés, un jeton de continuation est également retourné dans l’en-tête de réponse. Le nom de l’en-tête est `x-ms-continuation-token`.

Si vous définissez la valeur du jeton de continuation dans l’en-tête de la demande suivante, vous pouvez obtenir la page de résultats suivante. Cet en-tête de demande est également nommé `x-ms-continuation-token`.

### <a name="example---list-all-entities"></a>Exemple - Liste de toutes les entités

L’exemple de requête HTTP suivant liste toutes les entités dans le hub de tâches :

```http
GET /runtime/webhooks/durabletask/entities
```

La réponse JSON peut se présenter de la façon suivante (elle a été mise en forme pour une meilleure lisibilité) :

```json
[
    {
        "entityId": { "key": "cats", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:45:44.6326361Z",
    },
    {
        "entityId": { "key": "dogs", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:01.9477382Z"
    },
    {
        "entityId": { "key": "mice", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:15.4626159Z"
    },
    {
        "entityId": { "key": "radio", "name": "device" },
        "lastOperationTime": "2019-12-18T21:46:18.2616154Z"
    },
]
```

### <a name="example---filtering-the-list-of-entities"></a>Exemple - Filtrage de la liste des entités

L’exemple de requête HTTP suivant liste uniquement les deux premières entités de type `counter` et récupère également leur état :

```http
GET /runtime/webhooks/durabletask/entities/counter?top=2&fetchState=true
```

La réponse JSON peut se présenter de la façon suivante (elle a été mise en forme pour une meilleure lisibilité) :

```json
[
    {
        "entityId": { "key": "cats", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:45:44.6326361Z",
        "state": { "value": 9 }
    },
    {
        "entityId": { "key": "dogs", "name": "counter" },
        "lastOperationTime": "2019-12-18T21:46:01.9477382Z",
        "state": { "value": 10 }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment utiliser Application Insights pour surveiller vos fonctions durables](durable-functions-diagnostics.md)