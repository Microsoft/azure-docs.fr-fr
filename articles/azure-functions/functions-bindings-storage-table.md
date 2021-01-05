---
title: Liaisons de stockage Table Azure pour Azure Functions
description: Comprendre comment utiliser les liaisons de stockage de table Azure dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 4f2b890dc60cd50b5fcaefabe8d418268b738c20
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92096723"
---
# <a name="azure-table-storage-bindings-for-azure-functions"></a>Liaisons de stockage Table Azure pour Azure Functions

Azure Functions s’intègre à [Stockage Azure](../storage/index.yml) via [des déclencheurs et des liaisons](./functions-triggers-bindings.md). L’intégration au Stockage Table vous permet de créer des fonctions qui lisent et écrivent des données de stockage de table.

| Action | Type |
|---------|---------|
| Lire des données de stockage de table dans une fonction | [Liaison d’entrée](./functions-bindings-storage-table-input.md) |
| Autoriser une fonction à écrire des données de stockage de table |[Liaison de sortie](./functions-bindings-storage-table-output.md) |

## <a name="packages---functions-2x-and-higher"></a>Packages – Functions 2.x et versions ultérieures

Les liaisons du Stockage Table sont fournies dans le package NuGet [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage), version 3.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables).

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="packages---functions-1x"></a>Packages - Functions 1.x

Les liaisons du Stockage Table sont fournies dans le package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs), version 2.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Table).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Lire des données de stockage de table dans une fonction exécutée](./functions-bindings-storage-table-input.md)
- [Écrire des données de stockage de table à partir d’une fonction](./functions-bindings-storage-table-output.md)
