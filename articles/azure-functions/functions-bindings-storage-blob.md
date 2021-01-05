---
title: Déclencheur et liaisons de Stockage Blob Azure pour Azure Functions
description: Découvrez comment utiliser le déclencheur et les liaisons Stockage Blob Azure dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: e56d1add36d4296526348d12d7c0b6eb03108f27
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104357"
---
# <a name="azure-blob-storage-bindings-for-azure-functions-overview"></a>Vue d’ensemble des liaisons de Stockage Blob Azure pour Azure Functions

Azure Functions s’intègre à [Stockage Azure](../storage/index.yml) via [des déclencheurs et des liaisons](./functions-triggers-bindings.md). L’intégration à Stockage Blob vous permet de créer des fonctions qui réagissent aux modifications de données d’objets Blob ainsi qu’à la lecture et à l’écriture de valeurs.

| Action | Type |
|---------|---------|
| Exécuter une fonction quand des données de stockage d’objets Blob sont modifiées | [Déclencheur](./functions-bindings-storage-blob-trigger.md) |
| Lire des données de stockage d’objets Blob dans une fonction | [Liaison d’entrée](./functions-bindings-storage-blob-input.md) |
| Autoriser une fonction à écrire des données de stockage d’objets Blob |[Liaison de sortie](./functions-bindings-storage-blob-output.md) |

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
[Package NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Mettre à jour vos extensions]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Functions 1.x

Les applications Functions 1.x ont automatiquement une référence au package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), version 2.x.

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction quand les données de stockage Blob changent](./functions-bindings-storage-blob-trigger.md)
- [Lire des données de stockage de blobs lors de l’exécution d’une fonction](./functions-bindings-storage-blob-input.md)
- [Écrire des données de stockage d’objets blob à partir d’une fonction](./functions-bindings-storage-blob-output.md)