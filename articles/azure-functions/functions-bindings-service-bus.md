---
title: Liaisons Azure Service Bus pour Azure Functions
description: Apprenez à envoyer des déclencheurs et des liaisons Azure Service Bus dans Azure Functions.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: fasttrack-edit
ms.openlocfilehash: b32f16d170df9963960862bc82aef1a4baf13896
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104442"
---
# <a name="azure-service-bus-bindings-for-azure-functions"></a>Liaisons Azure Service Bus pour Azure Functions

Azure Functions s’intègre avec [Azure Service Bus](https://azure.microsoft.com/services/service-bus) via [des déclencheurs et des liaisons](./functions-triggers-bindings.md). L’intégration avec Service Bus vous permet de créer des fonctions qui réagissent et envoient des messages de file d’attente ou de rubrique.

| Action | Type |
|---------|---------|
| Exécuter une fonction durant la création d’un message de file d’attente ou de rubrique Service Bus | [Déclencheur](./functions-bindings-service-bus-trigger.md) |
| Envoyer des messages Azure Service Bus |[Liaison de sortie](./functions-bindings-service-bus-output.md) |

## <a name="add-to-your-functions-app"></a>Ajouter à votre application de fonction

> [!NOTE]
> Actuellement, la liaison Service Bus ne prend pas en charge l’authentification à l’aide d’une identité managée. Utilisez plutôt une [signature d’accès partagé Service Bus](../service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature).

### <a name="functions-2x-and-higher"></a>Functions 2.x et versions ultérieures

Pour utiliser le déclencheur et les liaisons, vous devez référencer le package approprié. Le package NuGet est utilisé pour les bibliothèques de classes .NET, tandis que le bundle d’extensions est utilisé pour tous les autres types d’applications.

| Langage                                        | Ajouter via...                                   | Notes 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | L’installation du [package NuGet], version 4.x | |
| Script C#, Java, JavaScript, Python, PowerShell | L’inscription du [bundle d’extensions]          | Il est recommandé d’utiliser l’[extension Azure Tools] avec Visual Studio Code. |
| Script C# (en ligne uniquement dans le portail Azure)         | Ajout d’une liaison                            | Pour mettre à jour des extensions de liaison existantes sans avoir à republier votre application de fonction, voir [Mettre à jour vos extensions]. |

[Package NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.ServiceBus/
[core tools]: ./functions-run-local.md
[Bundle d’extensions]: ./functions-bindings-register.md#extension-bundles
[Mettre à jour vos extensions]: ./functions-bindings-register.md
[Extension Azure Tools]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Les applications Functions 1.x ont automatiquement une référence au package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), version 2.x.

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction durant la création d’un message de file d’attente ou de rubrique Service Bus (déclencheur)](./functions-bindings-service-bus-trigger.md)
- [Envoyer des messages Azure Service Bus à partir d’Azure Functions (liaison de sortie)](./functions-bindings-service-bus-output.md)