---
title: Arrêter la surveillance de votre cluster Azure Kubernetes Service | Microsoft Docs
description: Cet article explique comment arrêter la surveillance de votre cluster Azure AKS avec Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 08/19/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 99c4ecb6c8b77f1576f25816ba486ed262249c32
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695704"
---
# <a name="how-to-stop-monitoring-your-azure-kubernetes-service-aks-with-azure-monitor-for-containers"></a>Guide pratique pour arrêter la supervision de votre cluster Azure Kubernetes Service (AKS) avec Azure Monitor pour conteneurs

Après avoir activé la supervision de votre cluster AKS, vous pouvez l'arrêter si vous décidez de ne plus le superviser ce cluster. Cet article explique comment procéder avec Azure CLI ou avec les modèles Azure Resource Manager fournis.  


## <a name="azure-cli"></a>Azure CLI

Utilisez la commande [az aks disable-addons](/cli/azure/aks?view=azure-cli-latest#az-aks-disable-addons) pour désactiver Azure Monitor pour conteneurs. La commande supprime l’agent des nœuds du cluster, mais ne supprime pas la solution ou les données déjà collectées et stockées dans votre ressource Azure Monitor.  

```azurecli
az aks disable-addons -a monitoring -n MyExistingManagedCluster -g MyExistingManagedClusterRG
```

Pour réactiver la supervision de votre cluster, consultez [Activer la supervision à l’aide d’Azure CLI](container-insights-enable-new-cluster.md#enable-using-azure-cli).

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Deux modèles Azure Resource Manager sont fournis pour prendre en charge la suppression cohérente et répétée des ressources de la solution dans votre groupe de ressources. Un des deux est un modèle JSON spécifiant la configuration pour arrêter la supervision, tandis que l’autre contient des valeurs de paramètres que vous configurez pour spécifier l’ID de ressource du cluster AKS ainsi que le groupe de ressources dans lequel le cluster est déployé.

Si vous n’êtes pas familiarisé avec le déploiement de ressources à l’aide d’un modèle, consultez les rubriques suivantes :
* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)
* [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../../azure-resource-manager/templates/deploy-cli.md)

>[!NOTE]
>Le modèle doit être déployé dans le même groupe de ressources du cluster. Si vous omettez d’autres propriétés ou modules complémentaires lors de l’utilisation de ce modèle, ils peuvent être supprimés du cluster. Par exemple, *enableRBAC* pour les stratégies Kubernetes RBAC implémentées dans votre cluster ou *aksResourceTagValues* si des balises sont spécifiées pour le cluster AKS.  
>

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI 2.0.27 ou version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

### <a name="create-template"></a>Créer un modèle

1. Copiez et collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
           "type": "string",
           "metadata": {
             "description": "AKS Cluster Resource ID"
           }
       },
      "aksResourceLocation": {
        "type": "string",
        "metadata": {
           "description": "Location of the AKS resource e.g. \"East US\""
         }
       },
    "aksResourceTagValues": {
      "type": "object",
      "metadata": {
        "description": "Existing all tags on AKS Cluster Resource"
        }
      }
     },
    "resources": [
      {
        "name": "[split(parameters('aksResourceId'),'/')[8]]",
        "type": "Microsoft.ContainerService/managedClusters",
        "location": "[parameters('aksResourceLocation')]",
        "tags": "[parameters('aksResourceTagValues')]",
        "apiVersion": "2018-03-31",
        "properties": {
          "mode": "Incremental",
          "id": "[parameters('aksResourceId')]",
          "addonProfiles": {
            "omsagent": {
              "enabled": false,
              "config": null
            }
           }
         }
       }
      ]
    }
    ```

2. Enregistrez ce fichier en tant que **OptOutTemplate.json** dans un dossier local.

3. Collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
          "value": "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
        },
        "aksResourceLocation": {
          "value": "<aksClusterRegion>"
        },
        "aksResourceTagValues": {
          "value": {
            "<existing-tag-name1>": "<existing-tag-value1>",
            "<existing-tag-name2>": "<existing-tag-value2>",
            "<existing-tag-nameN>": "<existing-tag-valueN>"
          }
        }
      }
    }
    ```

4. Modifiez **aksResourceId** et **aksResourceLocation** avec les valeurs du cluster AKS, que vous pouvez trouver dans la page **Propriétés** du cluster sélectionné.

    ![Page des propriétés du conteneur](media/container-insights-optout/container-properties-page.png)

    Lorsque vous êtes dans la page **Propriétés**, copiez également l’**ID de ressource de l’espace de travail**. Cette valeur est nécessaire si vous décidez ultérieurement de supprimer l’espace de travail Log Analytics. La suppression de l’espace de travail Log Analytics n’est pas effectuée dans le cadre de ce processus.

    Modifiez les valeurs d’**aksResourceTagValues** pour qu’elles correspondent aux valeurs de balises existantes spécifiées pour le cluster AKS.

5. Enregistrez ce fichier en tant que **OptOutParam.json** dans un dossier local.

6. Vous êtes prêt à déployer ce modèle.

### <a name="remove-the-solution-using-azure-cli"></a>Supprimer la solution à l’aide d’Azure CLI

Exécutez la commande suivante avec Azure CLI sur Linux pour supprimer la solution et effacer la configuration sur votre cluster AKS.

```azurecli
az login   
az account set --subscription "Subscription Name"
az deployment group create --resource-group <ResourceGroupName> --template-file ./OptOutTemplate.json --parameters @./OptOutParam.json  
```

Le changement de configuration peut prendre quelques minutes. Lorsqu’il est terminé, un message similaire au suivant s’affiche avec les résultats :

```output
ProvisioningState       : Succeeded
```

### <a name="remove-the-solution-using-powershell"></a>Supprimer la solution à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Exécutez les commandes PowerShell suivantes dans le dossier contenant le modèle pour supprimer la solution et effacer la configuration de votre cluster AKS.    

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName <yourSubscriptionName>
New-AzResourceGroupDeployment -Name opt-out -ResourceGroupName <ResourceGroupName> -TemplateFile .\OptOutTemplate.json -TemplateParameterFile .\OptOutParam.json
```

Le changement de configuration peut prendre quelques minutes. Lorsqu’il est terminé, un message similaire au suivant s’affiche avec les résultats :

```output
ProvisioningState       : Succeeded
```


## <a name="next-steps"></a>Étapes suivantes

Si l’espace de travail a été créé uniquement pour prendre en charge la surveillance du cluster et qu’il n’est plus nécessaire, vous devez le supprimer manuellement. Si vous n’êtes pas familiarisé avec la suppression d’un espace de travail, consultez la rubrique [Supprimer un espace de travail Azure Log Analytics avec le portail Azure](../platform/delete-workspace.md). N’oubliez pas l’**ID de ressource de l’espace de travail** copiée précédemment à l’étape 4 car vous en aurez besoin.
