---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 02/09/2020
ms.author: glenga
ms.openlocfilehash: 1fc37c6f93fba34944caa7a91c2a89ce5dcdc398
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "78201020"
---
::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
> [!TIP]
> Au démarrage, l’hôte télécharge et installe l’[extension de liaison de stockage](../articles/azure-functions/functions-bindings-storage-queue.md#functions-2x-and-higher) et d’autres extensions de liaison Microsoft. Cette installation a lieu, car les extensions de liaison sont activées par défaut dans le fichier *host.json* avec les propriétés suivantes :
>
> ```json
> {
>     "version": "2.0",
>     "extensionBundle": {
>         "id": "Microsoft.Azure.Functions.ExtensionBundle",
>         "version": "[1.*, 2.0.0)"
>     }
> }
> ```
>
> Si vous rencontrez des erreurs liées aux extensions de liaison, vérifiez que les propriétés ci-dessus sont présentes dans *host.json*.
::: zone-end  