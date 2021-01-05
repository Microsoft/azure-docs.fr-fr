---
title: 'Démarrage rapide : Créer un espace de travail Synapse en utilisant Azure CLI'
description: Créez un espace de travail Azure Synapse à l’aide d’Azure CLI en suivant les étapes décrites dans ce guide.
services: synapse-analytics
author: alehall
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 08/25/2020
ms.author: alehall
ms.reviewer: jrasnick
ms.openlocfilehash: 46ab1df1b776bf8dc9d6d917e5394c3efeec0de4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659384"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-cli"></a>Démarrage rapide : Créer un espace de travail Azure Synapse avec Azure CLI

L’interface de ligne de commande Azure (Azure CLI) est l’expérience de ligne de commande d’Azure pour gérer les ressources Azure. Vous pouvez l’utiliser dans votre navigateur avec Azure Cloud Shell. Vous pouvez également l’installer sur Windows, Linux ou MacOS, et l’exécuter à partir de la ligne de commande.

Dans ce guide de démarrage rapide, vous allez apprendre à créer un espace de travail Synapse en utilisant l’interface Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Téléchargez et installez [jq](https://stedolan.github.io/jq/download/), un processeur JSON en ligne de commande léger et flexible
- [Compte de stockage Azure Data Lake Storage Gen2](../storage/common/storage-account-create.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

    > [!IMPORTANT]
    > L’espace de travail Azure Synapse doit pouvoir lire et écrire dans le compte ADLS Gen2 sélectionné. De plus, pour tout compte de stockage que vous liez en tant que compte de stockage principal, vous devez avoir activé l’**espace de noms hiérarchique** lors de la création du compte de stockage, et tel que décrit dans la page [Créer un compte de stockage](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal#create-a-storage-account). 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-an-azure-synapse-workspace-using-the-azure-cli"></a>Créer un espace de travail Azure Synapse à l’aide de l’interface Azure CLI

1. Définissez les variables d’environnement nécessaires qui permettent de créer des ressources pour l’espace de travail Azure Synapse.

    | Nom de variable d'environnement | Description |
    |---|---|---|
    |StorageAccountName| Nom de votre compte de stockage ADLS Gen2 existant.|
    |StorageAccountResourceGroup| Nom du groupe de ressources de votre compte de stockage ADLS Gen2 existant. |
    |FileShareName| Nom de votre système de fichiers de stockage existant.|
    |SynapseResourceGroup| Choisissez un nouveau nom pour votre groupe de ressources Azure Synapse. |
    |Région| Choisissez une des [régions Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
    |SynapseWorkspaceName| Choisissez un nom unique pour votre nouvel espace de travail Azure Synapse. |
    |SqlUser| Choisissez une valeur pour un nouveau nom d’utilisateur.|
    |SqlPassword| Choisissez un mot de passe sécurisé.|
    |||

2. Créez un groupe de ressources en tant que conteneur pour votre espace de travail Azure Synapse :
    ```azurecli
    az group create --name $SynapseResourceGroup --location $Region
    ```
3. Récupérez la clé du compte de stockage ADLS Gen 2 :
    ```azurecli
    StorageAccountKey=$(az storage account keys list \
      --account-name $StorageAccountName \
      | jq -r '.[0] | .value')
    ```
4. Récupérez l’URL du point de terminaison du stockage ADLS Gen 2 :
    ```azurecli
    StorageEndpointUrl=$(az storage account show \
      --name $StorageAccountName \
      --resource-group $StorageAccountResourceGroup \
      | jq -r '.primaryEndpoints | .dfs')
    ```

5. (Facultatif) Vous pouvez toujours vérifier quels sont vos clé et point de terminaison du compte de stockage ADLS Gen2 :
    ```azurecli
    echo "Storage Account Key: $StorageAccountKey"
    echo "Storage Endpoint URL: $StorageEndpointUrl"
    ```

6. Créer un espace de travail Azure Synapse :
    ```azurecli
    az synapse workspace create \
      --name $SynapseWorkspaceName \
      --resource-group $SynapseResourceGroup \
      --storage-account $StorageAccountName \
      --file-system $FileShareName \
      --sql-admin-login-user $SqlUser \
      --sql-admin-login-password $SqlPassword \
      --location $Region
    ```

7. Obtenez l’URL dev et web de l’espace de travail Azure Synapse :
    ```azurecli
    WorkspaceWeb=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .web')

    WorkspaceDev=$(az synapse workspace show --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup | jq -r '.connectivityEndpoints | .dev')
    ```

8. Créez une règle de pare-feu pour autoriser votre accès à l’espace de travail Azure Synapse depuis votre machine :

    ```azurecli
    ClientIP=$(curl -sb -H "Accept: application/json" "$WorkspaceDev" | jq -r '.message')
    ClientIP=${ClientIP##'Client Ip address : '}
    echo "Creating a firewall rule to enable access for IP address: $ClientIP"

    az synapse workspace firewall-rule create --end-ip-address $ClientIP --start-ip-address $ClientIP --name "Allow Client IP" --resource-group $SynapseResourceGroup --workspace-name $SynapseWorkspaceName
    ```

9. Ouvrez l’adresse URL web de l’espace de travail Azure Synapse, stockée dans la variable d’environnement `WorkspaceWeb`, pour accéder à votre espace de travail :

    ```azurecli
    echo "Open your Azure Synapse Workspace Web URL in the browser: $WorkspaceWeb"
    ```
    
    [ ![Espace de travail Azure Synapse sur le web](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png) ](media/quickstart-create-synapse-workspace-cli/create-workspace-cli-1.png#lightbox)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Effectuez les étapes ci-dessous pour supprimer l’espace de travail Azure Synapse.
> [!WARNING]
> Le fait de supprimer un espace de travail Azure Synapse entraîne la suppression des moteurs d’analytique et des données stockées dans la base de données des pools SQL et des métadonnées de l’espace de travail. Il n’est plus possible de se connecter aux points de terminaison SQL ni aux points de terminaison Apache Spark. Tous les artefacts de code seront supprimés (requêtes, notebooks, définitions de travaux et pipelines).
>
> Le fait de supprimer l’espace de travail **n’affecte pas** les données dans le Data Lake Store Gen2 lié à l’espace de travail.

Pour supprimer l’espace de travail Azure Synapse, exécutez la commande suivante :

```azurecli
az synapse workspace delete --name $SynapseWorkspaceName --resource-group $SynapseResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez ensuite [créer des pools SQL](quickstart-create-sql-pool-studio.md) ou [créer des pools Apache Spark](quickstart-create-apache-spark-pool-studio.md) pour commencer à analyser et à explorer vos données.
