---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/26/2019
ms.author: glenga
ms.openlocfilehash: 4fe159660421113e0f0ac0586ae7e4a22d5bcda7
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "77474197"
---
### <a name="query-the-storage-queue"></a>Interroger la file d’attente de stockage

Vous pouvez utiliser la commande [`az storage queue list`](/cli/azure/storage/queue#az-storage-queue-list) pour afficher les files d’attente de stockage dans votre compte, comme dans l’exemple suivant :

```azurecli-interactive
az storage queue list --output tsv
```

La sortie de cette commande comprend une file d’attente nommée `outqueue`, qui a été créée lors de l’exécution de la fonction.

Ensuite, utilisez la commande [`az storage message peek`](/cli/azure/storage/message#az-storage-message-peek) pour afficher les messages dans cette file d’attente, comme dans l’exemple suivant :

```azurecli-interactive
echo `echo $(az storage message peek --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
```

La chaîne retournée doit être la même que le message que vous avez envoyé pour tester la fonction.

> [!NOTE]  
> L’exemple précédent décode la chaîne retournée à partir de base64. En effet, les liaisons Stockage File d’attente écrivent dans Stockage Azure et lisent à partir de ce dernier sous forme de [chaînes base64](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#encoding).