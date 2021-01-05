---
title: Liaisons Azure Event Grid pour Azure Functions
description: Découvrez comment gérer les événements Event Grid dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: fasttrack-edit
ms.openlocfilehash: 115ff1b59ecbe3f4fdb089c2bd61da955fba3984
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104544"
---
# <a name="azure-event-grid-bindings-for-azure-functions"></a>Liaisons Azure Event Grid pour Azure Functions

Cette référence explique comment gérer les événements [Event Grid](../event-grid/overview.md) dans Azure Functions. Pour plus d’informations sur la gestion des messages Event Grid dans un point de terminaison HTTP, consultez [Recevoir des événements sur un point de terminaison HTTP](../event-grid/receive-events.md).

Event Grid est un service Azure qui envoie des requêtes HTTP de notification au sujet des événements qui surviennent dans les *éditeurs*. L’éditeur d’un événement est le service ou la ressource qui est à l’origine de l’événement. Par exemple, un compte de Stockage Blob Azure est un éditeur, et [un chargement ou une suppression d’objet blob un événement](../storage/blobs/storage-blob-event-overview.md). [Certains services Azure intègrent la prise en charge de la publication d’événements sur Event Grid](../event-grid/overview.md#event-sources).

Les *gestionnaires* d’événements reçoivent et traitent les événements. Azure Functions est l’un des nombreux [services Azure qui intègrent la prise en charge de la gestion des événements Event Grid](../event-grid/overview.md#event-handlers). Dans cette référence, vous apprenez à utiliser un déclencheur Event Grid pour appeler une fonction quand un événement est reçu d’Event Grid et pour utiliser la liaison de sortie pour envoyer des événements à une [rubrique personnalisée Event Grid](../event-grid/post-to-custom-topic.md).

Si vous préférez, vous pouvez utiliser un déclencheur HTTP pour gérer les événements Event Grid ; consultez [Recevoir des événements sur un point de terminaison HTTP](../event-grid/receive-events.md). Actuellement, vous ne pouvez pas utiliser un déclencheur Event Grid pour une application Azure Functions quand l’événement est remis dans le [schéma CloudEvents](../event-grid/cloudevents-schema.md#azure-functions). Utilisez à la place un déclencheur HTTP.

| Action | Type |
|---------|---------|
| Exécuter une fonction lorsqu’un événement Event Grid est distribué | [Déclencheur](./functions-bindings-event-grid-trigger.md) |
| Envoie un événement Event Grid |[Liaison de sortie](./functions-bindings-event-grid-output.md) |

Le code de cette référence utilise par défaut la syntaxe .NET Core, utilisée dans les versions 2.x et ultérieures de Functions. Pour plus d'informations sur la syntaxe 1.x, consultez les [modèles Functions 1.x](https://github.com/Azure/azure-functions-templates/tree/v1.x/Functions.Templates/Templates).

## <a name="add-to-your-functions-app"></a>Ajouter à votre application Functions

### <a name="functions-2x-and-higher"></a>Functions 2.x et versions ultérieures

Pour utiliser le déclencheur et les liaisons, vous devez référencer le package approprié. Le package NuGet est utilisé pour les bibliothèques de classes .NET, tandis que le bundle d’extensions est utilisé pour tous les autres types d’applications.

| Langage                                        | Ajouter via...                                   | Notes 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | L’installation du [package NuGet], version 2.x | |
| Script C#, Java, JavaScript, Python, PowerShell | L’inscription du [bundle d’extensions]          | Il est recommandé d’utiliser l’[extension Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) avec Visual Studio Code. |
| Script C# (en ligne uniquement dans le portail Azure)         | Ajout d’une liaison                            | Pour mettre à jour des extensions de liaison existantes sans avoir à republier votre application de fonction, consultez [Mettre à jour vos extensions]. |

[core tools]: ./functions-run-local.md
[Bundle d’extensions]: ./functions-bindings-register.md#extension-bundles
[Package NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventGrid
[Mettre à jour vos extensions]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Les applications Functions 1.x ont automatiquement une référence au package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), version 2.x.

## <a name="next-steps"></a>Étapes suivantes
* [Exécuter une fonction lorsqu’un événement Event Grid est distribué](./functions-bindings-event-grid-trigger.md)
* [Distribuer un événement Event Grid](./functions-bindings-event-grid-trigger.md)