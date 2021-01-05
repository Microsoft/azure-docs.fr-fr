---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/27/2019
ms.author: glenga
ms.openlocfilehash: d697334fe56fb9133a06cee79067c60bc3a37281
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010479"
---
Le moyen le plus simple d’installer les extensions de liaison consiste à activer les [offres groupées d’extension](../articles/azure-functions/functions-bindings-register.md#extension-bundles). Lorsque vous activez les offres groupées, un ensemble prédéfini de packages d’extension sont automatiquement installés.

Pour activer les offres groupées d’extension, ouvrez le fichier host.json et mettez à jour son contenu pour qu’il corresponde au code suivant :

```json
{
    "version": "2.0",
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[1.*, 2.0.0)"
    }
}
```