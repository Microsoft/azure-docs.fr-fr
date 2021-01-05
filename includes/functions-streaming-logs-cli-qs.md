---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/18/2020
ms.author: glenga
ms.openlocfilehash: 9213bdd8e63fc1e8ab7bdba8ac7a569be0dfe6be
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93424719"
---
Exécutez la commande suivante pour afficher les [journaux de streaming](../articles/azure-functions/functions-run-local.md#enable-streaming-logs) en temps quasi-réel :

```console
func azure functionapp logstream <APP_NAME> 
```

Dans une fenêtre de terminal distincte ou dans le navigateur, appelez à nouveau la fonction distante. Un journal détaillé de l’exécution de la fonction dans Azure est affiché dans le terminal. 