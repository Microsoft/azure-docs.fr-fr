---
title: Créer des espaces de travail avec Azure CLI
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser l’extension Azure CLI pour Machine Learning afin de créer un espace de travail Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: larryfr
author: Blackmist
ms.date: 09/30/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.openlocfilehash: 66a9c9d605911a9d3b30a55d47e16026e26e502a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447254"
---
# <a name="create-a-workspace-for-azure-machine-learning-with-azure-cli"></a>Créer un espace de travail pour Azure Machine Learning avec Azure CLI


Dans cet article, vous allez découvrir comment créer un espace de travail Azure Machine Learning à l’aide d’Azure CLI. Azure CLI fournit des commandes pour gérer les ressources Azure. L’extension Machine Learning de l’interface CLI fournit les commandes pour utiliser les ressources d’Azure Machine Learning.

## <a name="prerequisites"></a>Prérequis

* Un **abonnement Azure**. Si vous n’en avez pas, essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

* Pour utiliser les commandes CLI dans ce document à partir de votre **environnement local**, vous avez besoin [d’Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).

    Si vous utilisez [Azure Cloud Shell](https://azure.microsoft.com//features/cloud-shell/), l’interface CLI est accessible via le navigateur et réside dans le cloud.

## <a name="limitations"></a>Limites

[!INCLUDE [register-namespace](../../includes/machine-learning-register-namespace.md)]

## <a name="connect-the-cli-to-your-azure-subscription"></a>Connexion de la CLI à votre abonnement Azure

> [!IMPORTANT]
> Si vous utilisez Azure Cloud Shell, vous pouvez ignorer cette section. Le shell cloud vous authentifie automatiquement à l’aide du compte avec lequel vous vous connectez à votre abonnement Azure.

Il existe plusieurs façons de vous authentifier auprès de votre abonnement Azure à partir de l’interface CLI. La plus simple consiste à s’authentifier de manière interactive à l’aide d’un navigateur. Pour vous authentifier de manière interactive, ouvrez une ligne de commande ou un terminal et utilisez la commande suivante :

```azurecli-interactive
az login
```

Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fera et chargera une page de connexion par la même occasion. Dans le cas contraire, vous devez ouvrir un navigateur et suivre les instructions de la ligne de commande. Les instructions impliquent de naviguer vers [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et d’entrer un code d’autorisation.

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

Pour les autres méthodes d’authentification, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?preserve-view=true&view=azure-cli-latest).

## <a name="install-the-machine-learning-extension"></a>Installer l’extension Machine Learning

Pour installer l’extension Machine Learning, utilisez la commande suivante :

```azurecli-interactive
az extension add -n azure-cli-ml
```

## <a name="create-a-workspace"></a>Créer un espace de travail

L’espace de travail d’Azure Machine Learning s’appuie sur les entités ou services Azure suivants :

> [!IMPORTANT]
> Si vous ne spécifiez pas de service Azure existant, il en sera créé un automatiquement lors de la création de l’espace de travail. Vous devez toujours spécifier un groupe de ressources. Lorsque vous attachez votre propre compte de stockage, vérifiez qu’il répond aux critères suivants :
>
> * Le compte de stockage n’est _pas_ un compte Premium (Premium_LRS ou Premium_GRS).
> * Les fonctionnalités Azure Blob et Azure File sont activées.
> * L’espace de noms hiérarchique (ADLS Gen2) est désactivé.
>
> Ces exigences concernent uniquement le compte de stockage utilisé _par défaut_ par l’espace de travail.

| Service | Paramètre pour spécifier une instance existante |
| ---- | ---- |
| **Groupe de ressources Azure** | `-g <resource-group-name>`
| **Compte Stockage Azure** | `--storage-account <service-id>` |
| **Azure Application Insights** | `--application-insights <service-id>` |
| **Azure Key Vault** | `--keyvault <service-id>` |
| **Azure Container Registry** | `--container-registry <service-id>` |

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

L’espace de travail Azure Machine Learning doit être créé à l’intérieur d’un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un. Pour __créer un groupe de ressources__, utilisez la commande suivante. Remplacez `<resource-group-name>` par le nom à utiliser pour ce groupe de ressources. Remplacez `<location>` par la région Azure à utiliser pour ce groupe de ressources :

> [!TIP]
> Vous devez sélectionner une région où Azure Machine Learning est disponible. Pour plus d’informations, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service).

```azurecli-interactive
az group create --name <resource-group-name> --location <location>
```

La réponse de cette commande doit ressembler au JSON suivant :

```json
{
  "id": "/subscriptions/<subscription-GUID>/resourceGroups/<resourcegroupname>",
  "location": "<location>",
  "managedBy": null,
  "name": "<resource-group-name>",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

Pour plus d’informations sur l’utilisation des groupes de ressources, consultez [az group](/cli/azure/group?preserve-view=true&view=azure-cli-latest).

### <a name="automatically-create-required-resources"></a>Créer automatiquement les ressources nécessaires

Pour créer un nouvel espace de travail dans lequel les __services sont créés automatiquement__, utilisez la commande suivante :

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name>
```

> [!NOTE]
> Le nom de l’espace de travail n’est pas sensible à la casse.

Le résultat de cette commande doit ressembler au JSON suivant :

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

### <a name="virtual-network-and-private-endpoint"></a>Réseau virtuel et point de terminaison privé

> [!IMPORTANT]
> L’utilisation d’un espace de travail Azure Machine Learning avec un lien privé n’est pas disponible dans les régions Azure Government ou Azure China 21Vianet.

Si vous souhaitez restreindre l’accès à votre espace de travail à un réseau virtuel, vous pouvez utiliser les paramètres suivants :

* `--pe-name`: Nom du point de terminaison privé qui est créé.
* `--pe-auto-approval`: Si les connexions de point de terminaison privé à l’espace de travail doivent être approuvées automatiquement.
* `--pe-resource-group`: Groupe de ressources dans lequel créer le point de terminaison privé. Doit être le même groupe qui contient le réseau virtuel.
* `--pe-vnet-name`: Réseau virtuel existant dans lequel créer le point de terminaison privé.
* `--pe-subnet-name`: Nom du sous-réseau dans lequel créer le point de terminaison privé. La valeur par défaut est `default`.

Pour plus d’informations sur l’utilisation d’un point de terminaison privé et d’un réseau virtuel avec votre espace de travail, consultez [Vue d’ensemble de l’isolement et la confidentialité des réseaux virtuels](how-to-network-security-overview.md).

### <a name="customer-managed-key-and-high-business-impact-workspace"></a>Clé gérée par le client et espace de travail High Business Impact

Par défaut, les métadonnées de l’espace de travail sont stockées dans une instance d’Azure Cosmos DB gérée par Microsoft. Les données sont chiffrées avec des clés managées par Microsoft.

> [!NOTE]
> Azure Cosmos DB ne permet __pas__ de stocker des informations, telles que les performances du modèle, les informations consignées par les expériences ou les informations enregistrées à partir de vos modèles de déploiement. Pour plus d’informations sur la surveillance de ces éléments, consultez la section [Surveillance et journalisation](concept-azure-machine-learning-architecture.md) de l’article sur l’architecture et les concepts.

Au lieu d’utiliser la clé gérée par Microsoft, vous pouvez utiliser la clé de votre choix. Cela crée l’instance d’Azure Cosmos DB qui stocke les métadonnées dans votre abonnement Azure. Utilisez le paramètre `--cmk-keyvault` pour spécifier le coffre Azure Key Vault qui contient la clé, et `--resource-cmk-uri` pour spécifier l’URL de la clé dans le coffre.

Avant d’utiliser les paramètres `--cmk-keyvault` et `--resource-cmk-uri`, vous devez d’abord effectuer les actions suivantes :

1. Autorisez l’__application Azure Machine Learning__ (dans la gestion des identités et des accès) avec des autorisations de contributeur pour votre abonnement.
1. Suivez les étapes décrites dans [Configurer les clés gérées par le client](../cosmos-db/how-to-setup-cmk.md) pour :
    * Inscrire le fournisseur Azure Cosmos DB
    * Créer et configurer un coffre Azure Key Vault
    * Générer une clé

Vous n’avez pas besoin de créer manuellement l’instance d’Azure Cosmos DB, l’une sera créée pour vous lors de la création de l’espace de travail. Cette instance de Azure Cosmos DB sera créée dans un groupe de ressources distinct à l’aide d’un nom basé sur ce modèle : `<your-resource-group-name>_<GUID>`.

[!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

Pour limiter les données collectées par Microsoft sur votre espace de travail, utilisez le paramètre `--hbi-workspace`. 

> [!IMPORTANT]
> Sélectionner High Business Impact ne peut être effectué que lors de la création d’un espace de travail. Vous ne pouvez pas modifier ce paramètre une fois l’espace de travail créé.

Pour plus d’informations sur les clés gérées par le client et sur l’espace de travail High Business Impact, consultez [Sécurité Enterprise pour Azure Machine Learning](concept-data-encryption.md#encryption-at-rest).

### <a name="use-existing-resources"></a>Utiliser les ressources existantes

Pour créer un espace de travail qui utilise des ressources existantes, vous devez fournir l’ID des ressources. Utilisez les commandes suivantes pour obtenir l’ID des services :

> [!IMPORTANT]
> Vous n’êtes pas obligé de spécifier toutes les ressources existantes. Vous pouvez en spécifier une ou plusieurs. Par exemple, vous pouvez spécifier un compte de stockage existant et l’espace de travail créera les autres ressources.

+ **Compte Stockage Azure** : `az storage account show --name <storage-account-name> --query "id"`

    La réponse de cette commande est similaire au texte suivant et correspond à l’ID de votre compte de stockage :

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"`

    > [!IMPORTANT]
    > Si vous souhaitez utiliser un compte de stockage Azure existant, il ne doit pas s’agir d’un compte Premium (Premium_LRS ou Premium_GRS). Il ne peut pas non plus comporter d’espace de noms hiérarchique (utilisé avec Azure Data Lake Storage Gen2). Ni le stockage Premium ni l’espace de noms hiérarchique ne sont pris en charge avec le compte de stockage _par défaut_ de l’espace de travail. Ils peuvent en revanche être utilisés avec des comptes de stockage _autres que les comptes par défaut_.

+ **Azure Application Insights** :

    1. Installez l’extension Application Insights :

        ```azurecli-interactive
        az extension add -n application-insights
        ```

    2. Obtenez l’ID de votre service Application Insights :

        ```azurecli-interactive
        az monitor app-insights component show --app <application-insight-name> -g <resource-group-name> --query "id"
        ```

        La réponse de cette commande est similaire au texte suivant et correspond à l’ID de votre service Application Insights :

        `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>"`

+ **Azure Key Vault** : `az keyvault show --name <key-vault-name> --query "ID"`

    La réponse de cette commande est similaire au texte suivant et correspond à l’ID de votre coffre de clés :

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<key-vault-name>"`

+ **Azure Container Registry** : `az acr show --name <acr-name> -g <resource-group-name> --query "id"`

    La réponse de cette commande est similaire au texte suivant et correspond à l’ID du registre de conteneurs :

    `"/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<acr-name>"`

    > [!IMPORTANT]
    > Le [compte d’administrateur](../container-registry/container-registry-authentication.md#admin-account) doit être activé sur le registre de conteneurs pour qu’il puisse être utilisé avec un espace de travail Azure Machine Learning.

Une fois que vous disposez des ID des ressources que vous souhaitez utiliser avec l’espace de travail, utilisez la commande `az workspace create -w <workspace-name> -g <resource-group-name>` de base et ajoutez le(s) paramètre(s) et l’ID ou les ID pour les ressources existantes. Par exemple, la commande suivante crée un espace de travail qui utilise un registre de conteneurs existant :

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name> --container-registry "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<acr-name>"
```

Le résultat de cette commande doit ressembler au JSON suivant :

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

## <a name="list-workspaces"></a>Répertorier les espaces de travail

Pour afficher la liste de tous les espaces de travail pour votre abonnement Azure, utilisez la commande suivante :

```azurecli-interactive
az ml workspace list
```

Le résultat de cette commande doit ressembler au JSON suivant :

```json
[
  {
    "resourceGroup": "myresourcegroup",
    "subscriptionId": "<subscription-id>",
    "workspaceName": "myml"
  },
  {
    "resourceGroup": "anotherresourcegroup",
    "subscriptionId": "<subscription-id>",
    "workspaceName": "anotherml"
  }
]
```

Pour plus d’informations, consultez la documentation [az ml workspace list](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-list).

## <a name="get-workspace-information"></a>Obtenir des informations sur l’espace de travail

Pour obtenir des informations sur un espace de travail, utilisez la commande suivante :

```azurecli-interactive
az ml workspace show -w <workspace-name> -g <resource-group-name>
```

Le résultat de cette commande doit ressembler au JSON suivant :

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/application-insight-name>",
  "creationTime": "2019-08-30T18:55:03.1807976+00:00",
  "description": "",
  "friendlyName": "",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "tags": {},
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

Pour plus d’informations, consultez la documentation [az ml workspace show](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-show).

## <a name="update-a-workspace"></a>Mettre à jour un espace de travail

Utilisez la commande suivante pour mettre à jour un espace de travail :

```azurecli-interactive
az ml workspace update -w <workspace-name> -g <resource-group-name>
```

Le résultat de cette commande doit ressembler au JSON suivant :

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/application-insight-name>",
  "creationTime": "2019-08-30T18:55:03.1807976+00:00",
  "description": "",
  "friendlyName": "",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "tags": {},
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

Pour plus d’informations, consultez la documentation [az ml workspace update](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-update).

## <a name="share-a-workspace-with-another-user"></a>Partager un espace de travail avec un autre utilisateur

Pour partager un espace de travail avec un autre utilisateur de votre abonnement, utilisez la commande suivante :

```azurecli-interactive
az ml workspace share -w <workspace-name> -g <resource-group-name> --user <user> --role <role>
```

Pour plus d’informations sur le contrôle d’accès en fonction du rôle Azure (RBAC Azure) avec Azure Machine Learning, consultez [Gérer les utilisateurs et les rôles](how-to-assign-roles.md).

Pour plus d’informations, consultez la documentation [az ml workspace share](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-share).

## <a name="sync-keys-for-dependent-resources"></a>Synchroniser les clés pour les ressources dépendantes

Si vous modifiez les clés d’accès pour l’une des ressources utilisées par votre espace de travail, cela prend environ une heure de synchroniser l’espace de travail avec la nouvelle clé. Pour forcer l’espace de travail à synchroniser les nouvelles clés immédiatement, utilisez la commande suivante :

```azurecli-interactive
az ml workspace sync-keys -w <workspace-name> -g <resource-group-name>
```

Pour plus d’informations sur la modification des clés, consultez [Regénérer les clés d’accès au stockage](how-to-change-storage-access-key.md).

Pour plus d’informations, consultez la documentation [az ml workspace sync-keys](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-sync-keys).

## <a name="delete-a-workspace"></a>Supprimer un espace de travail

Pour supprimer un espace de travail une fois qu’il n’est plus nécessaire, utilisez la commande suivante :

```azurecli-interactive
az ml workspace delete -w <workspace-name> -g <resource-group-name>
```

> [!IMPORTANT]
> La suppression d’un espace de travail ne supprime pas le service Application Insight, le compte de stockage, le coffre de clés ou le registre de conteneurs utilisé par l’espace de travail.

Vous pouvez également supprimer le groupe de ressources, ce qui supprime l’espace de travail et toutes les autres ressources Azure dans le groupe de ressources. Pour supprimer le groupe de ressources, utilisez la commande suivante :

```azurecli-interactive
az group delete -g <resource-group-name>
```

Pour plus d’informations, consultez la documentation [az ml workspace delete](/cli/azure/ext/azure-cli-ml/ml/workspace?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-workspace-delete).

## <a name="troubleshooting"></a>Dépannage

### <a name="resource-provider-errors"></a>Erreurs du fournisseur de ressources

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Déplacement de l’espace de travail

> [!WARNING]
> Le déplacement de votre espace de travail Azure Machine Learning vers un autre abonnement, ou le déplacement de l’abonnement propriétaire vers un nouveau locataire, n’est pas pris en charge. En effet, cela peut provoquer des erreurs.

### <a name="deleting-the-azure-container-registry"></a>Suppression d'Azure Container Registry

L'espace de travail Azure Machine Learning utilise Azure Container Registry (ACR) pour certaines opérations. Il crée automatiquement une instance ACR dès qu'il en a besoin.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’extension Azure CLI pour Machine Learning, consultez la documentation [az ml](/cli/azure/ext/azure-cli-ml/ml?preserve-view=true&view=azure-cli-latest).