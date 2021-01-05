---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 09/03/2020
ms.author: msmbaldwin
ms.openlocfilehash: 5e912b76c2ef68aa01dae57d1b42abc386a8e67b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89482123"
---
1. Utilisez la commande `az group create` pour créer un groupe de ressources :

    ```azurecli
    az group create --name KeyVault-PythonQS-rg --location eastus
    ```

    Si vous préférez, vous pouvez remplacer « eastus » par un emplacement plus proche de vous.

1. Utilisez `az keyvault create` pour créer le coffre de clés :

    ```azurecli
    az keyvault create --name <your-unique-keyvault-name> --resource-group KeyVault-PythonQS-rg
    ```

    Remplacez `<your-unique-keyvault-name>` par un nom unique à l’échelle d’Azure. La pratique courante consiste à utiliser son nom ou le nom de son entreprise et à ajouter des chiffres ou des identificateurs. 

1. Créez une variable d’environnement qui fournit le nom du coffre de clés au code :

    # <a name="cmd"></a>[cmd](#tab/cmd)

    ```cmd
    set KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    export KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```

    ---
