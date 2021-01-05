---
title: Comment arrêter la surveillance de votre cluster Azure Red Hat OpenShift v3 | Microsoft Docs
description: Cet article décrit comment arrêter la surveillance de votre cluster Azure Red Hat OpenShift avec Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 7e6ab46940ed29a98b3988c00c92d6c691d6e0f0
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695620"
---
# <a name="how-to-stop-monitoring-your-azure-red-hat-openshift-v3-cluster"></a>Comment arrêter la surveillance de votre cluster Azure Red Hat OpenShift v3

>[!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service en juin 2022.
>
> À partir d’octobre 2020, vous ne pourrez plus créer de clusters 3.11.
> Les clusters 3.11 existants continueront de fonctionner jusqu’en juin 2022, mais ne seront plus pris en charge après cette date.
>
> Suivez ce guide pour [Créer un cluster Azure Red Hat OpenShift 4](../../openshift/tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, [n’hésitez pas à nous contacter](mailto:aro-feedback@microsoft.com).

Une fois que vous avez activé la surveillance de votre cluster Azure Red Hat OpenShift 3.x, vous pouvez arrêter la surveillance à l’aide d’Azure Monitor pour conteneurs si vous décidez de ne plus le surveiller. Cet article explique comment procéder à cela en utilisant le modèle Azure Resource Manager fourni.  

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Deux modèles Azure Resource Manager sont fournis pour prendre en charge la suppression cohérente et répétée des ressources de la solution dans votre groupe de ressources. Un des deux est un modèle JSON qui spécifie la configuration pour arrêter la surveillance, tandis que l’autre contient des valeurs de paramètres que vous configurez pour spécifier l’ID de ressource du cluster OpenShift, ainsi que la région Azure dans lequel le cluster est déployé.

Si vous n’êtes pas familiarisé avec le déploiement de ressources à l’aide d’un modèle, consultez les rubriques suivantes :
* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)
* [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../../azure-resource-manager/templates/deploy-cli.md)

Si vous avez choisi d’utiliser Azure CLI, vous devez d’abord l’installer et l’utiliser localement. Vous devez exécuter Azure CLI version 2.0.65 ou une version ultérieure. Pour identifier votre version, exécutez `az --version`. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

### <a name="create-template"></a>Créer un modèle

1. Copiez et collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
         "aroResourceId": {
           "type": "string",
           "metadata": {
             "description": "ARO Cluster Resource ID"
          }
        },
        "aroResourceLocation": {
          "type": "string",
          "metadata": {
            "description": "Location of the aro cluster resource e.g. westcentralus"
          }
        }
      },
      "resources": [
        {
           "name": "[split(parameters('aroResourceId'),'/')[8]]",
           "type": "Microsoft.ContainerService/openShiftManagedClusters",
           "location": "[parameters('aroResourceLocation')]",
           "apiVersion": "2019-09-30-preview",
           "properties": {
             "mode": "Incremental",
             "id": "[parameters('aroResourceId')]",
             "monitorProfile": {
               "workspaceResourceID": null,
               "enabled": false
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
        "aroResourceId": {
          "value": "/subscriptions/<subscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.ContainerService/openShiftManagedClusters/<clusterName>"
        },
        "aroResourceLocation": {
          "value": "<azure region of the cluster e.g. westcentralus>"
        }
      }
    }
    ```

4. Modifiez les valeurs de **aroResourceId** et **aroResourceLocationn** avec les valeurs du cluster OpenShift, que vous pouvez trouver dans la page **Propriétés** du cluster sélectionné.

    ![Page des propriétés du conteneur](media/container-insights-optout-openshift/cluster-properties-page.png)

5. Enregistrez ce fichier en tant que **OptOutParam.json** dans un dossier local.

6. Vous êtes prêt à déployer ce modèle.

### <a name="remove-the-solution-using-azure-cli"></a>Supprimer la solution à l’aide d’Azure CLI

Exécutez la commande suivante avec Azure CLI sur Linux pour supprimer la solution et effacer la configuration sur votre cluster.

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

Exécutez les commandes PowerShell suivantes dans le dossier contenant le modèle pour supprimer la solution et effacer la configuration de votre cluster.    

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

Si l’espace de travail a été créé uniquement pour prendre en charge la surveillance du cluster et qu’il n’est plus nécessaire, vous devez le supprimer manuellement. Si vous n’êtes pas familiarisé avec la suppression d’un espace de travail, consultez la rubrique [Supprimer un espace de travail Azure Log Analytics](../platform/delete-workspace.md).
