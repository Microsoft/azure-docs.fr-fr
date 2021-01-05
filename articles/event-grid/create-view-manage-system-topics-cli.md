---
title: Créer, afficher et gérer des rubriques système Azure Event Grid avec l’interface CLI
description: Cet article explique comment utiliser Azure CLI pour créer, afficher et supprimer des rubriques système.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 354afb89b145e288f525e40ad700e8f8a67c6dad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86115041"
---
# <a name="create-view-and-manage-event-grid-system-topics-using-azure-cli"></a>Créer, afficher et gérer des rubriques système Event Grid avec l’interface Azure CLI
Cet article vous montre comment créer et gérer des rubriques système dans l’interface Azure CLI. Pour une présentation des rubriques système, consultez [Rubriques système](system-topics.md).

## <a name="install-extension-for-azure-cli"></a>Installer l'extension pour l'interface de ligne de commande Azure
L'interface de ligne de commande Azure requiert l'[extension Event Grid](/cli/azure/azure-cli-extensions-list).

Dans Cloud Shell :

- Si vous avez installé l’extension précédemment, mettez-la à jour : `az extension update -n eventgrid`
- Si vous n’avez pas installé l’extension précédemment, faites-le : `az extension add -n eventgrid`

Pour une installation locale :

1. [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli). Vérifiez que la dernière version est installée en exécutant `az --version`.
2. Désinstallez les précédentes versions de l’extension : `az extension remove -n eventgrid`
3. Installez l’extension eventgrid avec `az extension add -n eventgrid`

## <a name="create-a-system-topic"></a>Créer une rubrique système

- Pour créer d’abord une rubrique système sur une source Azure, puis créer un abonnement d’événement pour cette rubrique, consultez les rubriques de référence suivantes :
    - [az eventgrid system-topic create](/cli/azure/ext/eventgrid/eventgrid/system-topic?view=azure-cli-latest#ext-eventgrid-az-eventgrid-system-topic-create)

        ```azurecli-interactive
        # Get the ID of the Azure source (for example: Azure Storage account)
        storageid=$(az storage account show \
                --name <AZURE STORAGE ACCOUNT NAME> \
                --resource-group <AZURE RESOURCE GROUP NAME> \
                    --query id --output tsv)
    
        # Create the system topic on the Azure source (example: Azure Storage account)
        az eventgrid system-topic create \
            -g <AZURE RESOURCE GROUP NAME> \
            --name <SPECIFY SYSTEM TOPIC NAME> \
            --location <LOCATION> \
            --topic-type microsoft.storage.storageaccounts \
            --source $storageid
        ```           

        Pour obtenir la liste des valeurs de `topic-type` que vous pouvez utiliser pour créer une rubrique système, exécutez la commande suivante. Ces valeurs de rubrique représentent les sources d’événements qui prennent en charge la création de rubriques système. Ignorez `Microsoft.EventGrid.Topics` et `Microsoft.EventGrid.Domains` de la liste. 

        ```azurecli-interactive
        az eventgrid topic-type  list --output json | grep -w id
        ```
    - [az eventgrid system-topic event-subscription create](/cli/azure/ext/eventgrid/eventgrid/system-topic/event-subscription?view=azure-cli-latest#ext-eventgrid-az-eventgrid-system-topic-event-subscription-create)

        ```azurecli-interactive
        az eventgrid system-topic event-subscription create --name <SPECIFY EVENT SUBSCRIPTION NAME> \
            -g rg1 --system-topic-name <SYSTEM TOPIC NAME> \
            --endpoint <ENDPOINT URL>         
        ```
- Pour créer une rubrique système (implicitement) lors de la création d’un abonnement aux événements pour une source Azure, utilisez la méthode [az eventgrid event-subscription create](/cli/azure/ext/eventgrid/eventgrid/event-subscription?view=azure-cli-latest#ext-eventgrid-az-eventgrid-event-subscription-create). Voici un exemple :
    
    ```azurecli-interactive
    storageid=$(az storage account show --name <AZURE STORAGE ACCOUNT NAME> --resource-group <AZURE RESOURCE GROUP NAME> --query id --output tsv)
    endpoint=<ENDPOINT URL>

    az eventgrid event-subscription create \
      --source-resource-id $storageid \
      --name <EVENT SUBSCRIPTION NAME> \
      --endpoint $endpoint
    ```
    Pour suivre un tutoriel avec des instructions pas à pas, consultez [S’abonner à un compte de stockage](../storage/blobs/storage-blob-event-quickstart.md?toc=%2Fazure%2Fevent-grid%2Ftoc.json#subscribe-to-your-storage-account).

## <a name="view-all-system-topics"></a>Afficher toutes les rubriques système
Pour afficher toutes les rubriques et les détails relatifs à une rubrique système sélectionnée, utilisez les commandes suivantes :

- [az eventgrid system-topic list](/cli/azure/ext/eventgrid/eventgrid/system-topic?view=azure-cli-latest#ext-eventgrid-az-eventgrid-system-topic-list)

    ```azurecli-interactive
    az eventgrid system-topic list   
     ```
- [az eventgrid system-topic show](/cli/azure/ext/eventgrid/eventgrid/system-topic?view=azure-cli-latest#ext-eventgrid-az-eventgrid-system-topic-show)

    ```azurecli-interactive
    az eventgrid system-topic show -g <AZURE RESOURCE GROUP NAME> -n <SYSTEM TOPIC NAME>     
     ```

## <a name="delete-a-system-topic"></a>Supprime une rubrique système
Pour supprimer une rubrique système, utilisez la commande suivante : 

- [az eventgrid system-topic delete](/cli/azure/ext/eventgrid/eventgrid/system-topic?view=azure-cli-latest#ext-eventgrid-az-eventgrid-system-topic-delete)

    ```azurecli-interactive
    az eventgrid system-topic delete -g <AZURE RESOURCE GROUP NAME> --name <SYSTEM TOPIC NAME>   
     ```

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les rubriques système et les types de rubriques pris en charge par Azure Event Grid, consultez la section [Rubriques système dans Azure Event Grid](system-topics.md). 