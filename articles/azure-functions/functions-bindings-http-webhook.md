---
title: Déclencheurs et liaisons HTTP d’Azure Functions
description: Apprenez à utiliser des déclencheurs et des liaisons HTTP dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.openlocfilehash: 5236a3be9f8908d886274764dfc0c0da5b3565bc
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104476"
---
# <a name="azure-functions-http-triggers-and-bindings-overview"></a>Vue d’ensemble des déclencheurs et des liaisons HTTP Azure Functions

Azure Functions peut être appelé par le biais de requêtes HTTP pour générer des API serverless et répondre aux [webhooks](https://en.wikipedia.org/wiki/Webhook).

| Action | Type |
|---------|---------|
| Exécuter une fonction à partir d’une requête HTTP | [Déclencheur](./functions-bindings-http-webhook-trigger.md) |
| Retourner une réponse HTTP à partir d’une fonction |[Liaison de sortie](./functions-bindings-http-webhook-output.md) |

Le code de cet article utilise par défaut la syntaxe .NET Core, utilisée dans les versions 2.x et ultérieures de Functions. Pour plus d'informations sur la syntaxe 1.x, consultez les [modèles Functions 1.x](https://github.com/Azure/azure-functions-templates/tree/v1.x/Functions.Templates/Templates).

## <a name="add-to-your-functions-app"></a>Ajouter à votre application de fonction

### <a name="functions-2x-and-higher"></a>Functions 2.x et versions ultérieures

Pour utiliser le déclencheur et les liaisons, vous devez référencer le package approprié. Le package NuGet est utilisé pour les bibliothèques de classes .NET, tandis que le bundle d’extensions est utilisé pour tous les autres types d’applications.

| Langage                                        | Ajouter via...                                   | Notes 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | L’installation du [package NuGet], version 3.x | |
| Script C#, Java, JavaScript, Python, PowerShell | L’inscription du [bundle d’extensions]          | Il est recommandé d’utiliser l’[extension Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) avec Visual Studio Code. |
| Script C# (en ligne uniquement dans le portail Azure)         | Ajout d’une liaison                            | Pour mettre à jour des extensions de liaison existantes sans avoir à republier votre application de fonction, voir [Mettre à jour vos extensions]. |

[core tools]: ./functions-run-local.md
[Bundle d’extensions]: ./functions-bindings-register.md#extension-bundles
[Package NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http
[Mettre à jour vos extensions]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Les applications Functions 1.x ont automatiquement une référence au package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), version 2.x.

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction à partir d’une requête HTTP](./functions-bindings-http-webhook-trigger.md)
- [Retourner une réponse HTTP à partir d'une fonction](./functions-bindings-http-webhook-output.md)