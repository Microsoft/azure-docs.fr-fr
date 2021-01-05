---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 894a89126d1ee3ed909134f3e0dd914166568654
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90606552"
---
Vous pouvez voir la file d’attente dans le [portail Azure](../articles/storage/queues/storage-quickstart-queues-portal.md) ou dans l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com/). Vous pouvez également voir la file d’attente dans Azure CLI, en effectuant les étapes suivantes :

1. Ouvrez le fichier *local.setting.json* du projet de fonction, et copiez la valeur de la chaîne de connexion. Dans une fenêtre de terminal ou une fenêtre Commande, exécutez la commande suivante pour créer une variable d’environnement nommée `AZURE_STORAGE_CONNECTION_STRING`, en collant votre chaîne de connexion spécifique à la place de `<MY_CONNECTION_STRING>`. (Cette variable d’environnement signifie que vous n’avez pas besoin de fournir la chaîne de connexion pour les prochaines commandes à l’aide de l’argument `--connection-string`.)

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    export AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    $env:AZURE_STORAGE_CONNECTION_STRING = "<MY_CONNECTION_STRING>"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/cmd)
    
    ```azurecli
    set AZURE_STORAGE_CONNECTION_STRING="<MY_CONNECTION_STRING>"
    ```
    
    ---
    
1. (Facultatif) Utilisez la commande [`az storage queue list`](/cli/azure/storage/queue#az-storage-queue-list) pour voir les files d’attente de stockage dans votre compte. La sortie de cette commande doit inclure une file d’attente nommée `outqueue`, qui a été créée au moment où la fonction a écrit son premier message dans cette file d’attente.
    
    ```azurecli
    az storage queue list --output tsv
    ```

1. Utilisez la commande [`az storage message get`](/cli/azure/storage/message#az-storage-message-get) pour lire le message de cette file d’attente, qui doit être le premier nom que vous avez utilisé quand vous avez précédemment testé la fonction. La commande lit et supprime le premier message dans la file d’attente. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    echo `echo $(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(az storage message get --queue-name outqueue -o tsv --query '[].{Message:content}')))
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/cmd)
    
    ```azurecli
    az storage message get --queue-name outqueue -o tsv --query [].{Message:content} > %TEMP%out.b64 && certutil -decode -f %TEMP%out.b64 %TEMP%out.txt > NUL && type %TEMP%out.txt && del %TEMP%out.b64 %TEMP%out.txt /q
    ```

    Ce script utilise certutil pour décoder la collection de messages encodés en base64 dans un fichier temporaire local. S’il n’y a pas de sortie, essayez de supprimer `> NUL` du script pour arrêter la suppression de la sortie certutil, en cas d’erreur. 
    
    ---
    
    Étant donné que le corps du message stocké est [encodé en base64](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#encoding), le message doit être décodé préalablement à son affichage. Une fois que vous avez exécuté `az storage message get`, le message est supprimé de la file d’attente. S’il n’y avait qu’un seul message dans `outqueue`, vous ne récupérez pas de message à la deuxième exécution de cette commande et vous obtenez une erreur à la place.
