---
title: Vue d’ensemble des versions de Durable Functions - Azure Functions
description: En savoir plus sur les versions de Durable Functions.
author: cgillum
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: azfuncdf
ms.openlocfilehash: d6662259494bba5747e01c4574186e9030112247
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88719408"
---
# <a name="durable-functions-versions-overview"></a>Vue d’ensemble des versions de Durable Functions

*Fonctions durables* est une extension d[’Azure Functions](../functions-overview.md) et d[’Azure WebJobs](../../app-service/webjobs-create.md) qui vous permet d’écrire des fonctions avec état dans un environnement sans serveur. L’extension gère l’état, les points de contrôle et les redémarrages à votre place. Si vous n’êtes pas encore familiarisé avec Durable Functions, consultez la [documentation générale](durable-functions-overview.md).

## <a name="new-features-in-2x"></a>Nouvelles fonctionnalités de la version 2.x

Cette section décrit les fonctionnalités de Durable Functions ajoutées à la version 2.x.

### <a name="durable-entities"></a>Entités durables

Dans Durable Functions 2.x, nous avons introduit un nouveau concept de [fonctions d’entité](durable-functions-entities.md).

Les fonctions d’entité définissent les opérations pour la lecture et la mise à jour de petits éléments d’état, connus sous le nom *d’entités durables*. Comme les fonctions d’orchestrateur, les fonctions d’entité sont des fonctions ayant un type spécial de déclencheur, *déclencheur d’entité*. Contrairement aux fonctions d’orchestrateur, les fonctions d’entité n’ont pas de contraintes code spécifiques. Les fonctions d’entité gèrent également l’état explicitement plutôt que de représenter implicitement l’état via le flux de contrôle.

Pour plus d’informations, consultez l’article [Entités durables](durable-functions-entities.md).

### <a name="durable-http"></a>Fonctionnalité HTTP durable

Dans Durable Functions v2.x, nous avons introduit une nouvelle fonctionnalité [HTTP durable](durable-functions-http-features.md#consuming-http-apis) qui vous permet d’effectuer les opérations suivantes :

* Appeler les API HTTP directement à partir des fonctions d’orchestration (avec certaines limitations documentées).
* Implémenter une interrogation d’état HTTP 202 automatique côté client.
* Bénéficier d’une prise en charge intégrée des [identités managées Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Pour plus d’informations, consultez cet article sur les [fonctionnalités HTTP](durable-functions-http-features.md#consuming-http-apis).

## <a name="migrate-from-1x-to-2x"></a>Migrer de la version 1.x à 2.x

Cette section décrit comment migrer votre extension Durable Functions existante de la version 1.x vers la version 2.x pour tirer parti des nouvelles fonctionnalités.

### <a name="upgrade-the-extension"></a>Mettre à niveau l’extension

Installez la dernière version 2.x de l’extension de liaisons Durable Functions dans votre projet.

#### <a name="javascript-python-and-powershell"></a>JavaScript, Python, et PowerShell

Durable Functions 2.x est disponible dans la version 2.x du [pack d’extension Azure Functions](../functions-bindings-register.md#extension-bundles).

Pour mettre à jour la version du pack d’extension dans votre projet, ouvrez host.json et mettez à jour la section `extensionBundle` pour utiliser la version 2.x (`[2.*, 3.0.0)`).

```json
{
    "version": "2.0",
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[2.*, 3.0.0)"
    }
}
```

#### <a name="net"></a>.NET

Mettez à jour votre projet .NET pour utiliser la dernière version de l’[extension de liaisons Durable Functions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask).

Pour en savoir plus, consultez [Inscrire des extensions de liaison Azure Functions](../functions-bindings-register.md#local-csharp).

### <a name="update-your-code"></a>Mettre à jour votre code

Durable Functions 2.x introduit plusieurs changements cassants. Les applications Durable Functions 1.x ne sont pas compatibles avec Durable Functions 2.x sans modification de code. Cette section répertorie quelques-unes des modifications que vous devez effectuer lors de la mise à niveau de vos fonctions version 1.x vers la version 2.x.

#### <a name="hostjson-schema"></a>Schéma Host.json

Durable Functions 2.x utilise un nouveau schéma host.json. Les principales modifications apportées par rapport à la version 1.x sont les suivantes :

* `"storageProvider"` (et la sous-section `"azureStorage"`) pour la configuration spécifique au stockage.
* `"tracing"` pour la configuration du suivi et de la journalisation.
* `"notifications"` (et la sous-section `"eventGrid"`) pour la configuration de notification de grille d’événement.

Pour plus d’informations, consultez la [documentation de référence pour host.json de Durable Functions](durable-functions-bindings.md#durable-functions-2-0-host-json).

#### <a name="default-taskhub-name-changes"></a>Modification du nom du hub de tâches par défaut

Dans la version 1.x, si un nom de hub de tâches n’a pas été spécifié dans le fichier host.json, la valeur par défaut est « DurableFunctionsHub ». Dans la version 2.x, le nom du hub de tâches par défaut est désormais dérivé du nom de l’application de fonction. Pour cette raison, si vous n’avez pas spécifié de nom de hub de tâches lors de la mise à niveau vers 2.x, votre code fonctionnera avec le nouveau hub de tâches, et toutes les orchestrations en cours ne disposeront plus d’une application qui les traite. Pour contourner ce problème, vous pouvez définir explicitement le nom de votre hub de tâches sur la valeur par défaut « DurableFunctionsHub » de la version 1.x, ou vous pouvez suivre notre [guide de déploiement sans interruption de service](durable-functions-zero-downtime-deployment.md) pour plus d’informations sur la gestion des changements cassants pour les orchestrations en cours.

#### <a name="public-interface-changes-net-only"></a>Modifications de l’interface publique (.NET uniquement)

Dans la version 1.x, les divers objets de _contexte_ pris en charge par Durable Functions avaient des classes abstraites prévues pour une utilisation lors des tests unitaires. Dans le cadre de Durable Functions 2.x, ces classes de base abstraites sont remplacées par des interfaces.

Le tableau suivant représente les principales modifications :

| 1.x | 2.x |
|----------|----------|
| `DurableOrchestrationClientBase` | `IDurableOrchestrationClient` ou `IDurableClient` |
| `DurableOrchestrationContext` ou `DurableOrchestrationContextBase` | `IDurableOrchestrationContext` |
| `DurableActivityContext` ou `DurableActivityContextBase` | `IDurableActivityContext` |
| `OrchestrationClientAttribute` | `DurableClientAttribute` |

Dans le cas où une classe de base abstraite contenait des méthodes virtuelles, ces méthodes virtuelles ont été remplacées par les méthodes d’extension définies dans `DurableContextExtensions`.

#### <a name="functionjson-changes-javascript-and-c-script"></a>Modifications de function.json (script JavaScript et C#)

Dans Durable Functions 1.x, la liaison du client d’orchestration utilise un `type` de `orchestrationClient`. La version 2.x utilise `durableClient` à la place.

#### <a name="raise-event-changes"></a>Changements liés au déclenchement d’événements

Dans Durable Functions 1.x, l’appel de l’API de [déclenchement d’événement](durable-functions-external-events.md#send-events) et la spécification d’une instance qui n’existait pas a occasionné un échec silencieux. À partir de 2.x, le déclenchement d’un événement pour une orchestration inexistante entraîne une exception.
