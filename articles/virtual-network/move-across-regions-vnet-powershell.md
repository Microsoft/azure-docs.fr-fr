---
title: Déplacer un réseau virtuel Azure vers une autre région Azure en utilisant Azure PowerShell
description: Déplacez un réseau virtuel Azure d’une région Azure vers une autre en utilisant un modèle Resource Manager et Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/26/2019
ms.author: allensu
ms.openlocfilehash: e13164c3ec6049a8ae3954528a02d20e313dd883
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008121"
---
# <a name="move-an-azure-virtual-network-to-another-region-by-using-azure-powershell"></a>Déplacer un réseau virtuel Azure vers une autre région en utilisant Azure PowerShell

Il existe différents scénarios pour déplacer un réseau virtuel Azure existant d’une région à une autre. Par exemple, vous voulez créer un réseau virtuel avec la même configuration pour des tests et la même disponibilité que votre réseau virtuel existant. Vous pourriez aussi déplacer un réseau virtuel de production vers une autre région dans le cadre de la planification de la reprise d’activité.

Vous pouvez utiliser un modèle Azure Resource Manager pour terminer la migration du réseau virtuel vers une autre région. Pour ce faire, exportez le réseau virtuel vers un modèle, modifiez les paramètres pour qu’ils correspondent à la région de destination, puis déployez le modèle dans la nouvelle région. Pour plus d’informations sur les modèles Resource Manager, consultez [Exporter des groupes de ressources vers des modèles](https://docs.microsoft.com/azure/azure-resource-manager/manage-resource-groups-powershell#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que votre réseau virtuel Azure se trouve dans la région Azure à partir de laquelle vous voulez effectuer le déplacement.

- Pour exporter un réseau virtuel et déployer un modèle pour créer un réseau virtuel dans une autre région, vous avez besoin du rôle Contributeur de réseaux ou d’un rôle supérieur.

- Les appairages de réseaux virtuels ne sont pas recréés et échouent s’ils sont toujours présents dans le modèle. Avant d’exporter le modèle, vous devez supprimer les pairs du réseau virtuel. Vous pouvez ensuite les rétablir après le déplacement du réseau virtuel.
    
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les adresses IP publiques.

- Vérifiez que votre abonnement Azure vous permet de créer des réseaux virtuels dans la région cible. Pour activer le quota nécessaire, contactez le support technique.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout de réseaux virtuels pour ce processus. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-for-the-move"></a>Préparer le déplacement
Dans cette section, vous préparez le réseau virtuel pour le déplacement en utilisant un modèle Resource Manager. Vous déplacez ensuite le réseau virtuel vers la région cible en utilisant des commandes Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour exporter le réseau virtuel et déployer le réseau virtuel cible en utilisant PowerShell, procédez comme suit :

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0), puis suivez les instructions qui s’affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

1. Obtenez l’ID de ressource du réseau virtuel que vous voulez déplacer vers la région cible, puis placez-le dans une variable en utilisant [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourceVNETID = (Get-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>).Id
    ```

1. Exportez le réseau virtuel source vers un fichier .json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

1. Le fichier téléchargé a le même nom que le groupe de ressources à partir duquel la ressource a été exportée. Recherchez le fichier *\<resource-group-name>.json*, que vous avez exporté avec la commande, puis ouvrez-le dans votre éditeur :
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

1. Pour modifier le paramètre du nom du réseau virtuel, remplacez la valeur de la propriété **defaultValue** du nom du réseau virtuel source par le nom de votre réseau virtuel cible. Veillez à placer le nom entre guillemets.
    
    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentmyResourceGroupVNET.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_myVNET1_name": {
        "defaultValue": "<target-virtual-network-name>",
        "type": "String"
        }
    ```

1. Pour modifier la région cible où le réseau virtuel sera déplacé, changez la propriété **location** sous resources :

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```
  
1. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](https://docs.microsoft.com/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    ```

1. (Facultatif) Vous pouvez aussi changer d’autres paramètres dans le fichier *\<resource-group-name>.json*, en fonction de vos besoins :

    * **Espace d’adressage** : Avant d’enregistrer le fichier, vous pouvez modifier l’espace d’adressage du réseau virtuel en modifiant la section **resources** > **addressSpace** et en changeant la propriété **addressPrefixes** :

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },
        ```

    * **Sous-réseau** : Vous pouvez changer ou ajouter le nom du sous-réseau et l’espace d’adressage du sous-réseau en modifiant la section **subnets** du fichier. Vous pouvez changer le nom du sous-réseau en modifiant la propriété **name**. Vous pouvez changer l’espace d’adressage du sous-réseau en modifiant la propriété **addressPrefix** :

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

        Pour changer le préfixe d’adresse, modifiez-le à deux endroits : dans le code de la section précédente et dans la section **type** du code suivant. Modifiez la propriété **addressPrefix** dans le code suivant pour qu’elle corresponde à la propriété **addressPrefix** dans le code de la section précédente.

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

1. Enregistrez le fichier *\<resource-group-name>.json*.

1. Créez un groupe de ressources dans la région cible pour le réseau virtuel cible à déployer en utilisant [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0) :
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
1. Déployez le fichier *\<resource-group-name>.json* modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    ```

1. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>
    ```

    ```azurepowershell-interactive

    Get-AzVirtualNetwork -Name <target-virtual-network-name> -ResourceGroupName <target-resource-group-name>
    ```

## <a name="delete-the-virtual-network-or-resource-group"></a>Supprimer le réseau virtuel ou le groupe de ressources 

Une fois que vous avez déployé le réseau virtuel , pour recommencer ou supprimer le réseau virtuel dans la région cible, supprimez le groupe de ressources que vous avez créé dans la région cible : le réseau virtuel déplacé est alors supprimé. 

Pour supprimer le groupe de ressources, utilisez [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>
```

## <a name="clean-up"></a>Nettoyer

Pour valider vos changements et effectuer le déplacement du réseau virtuel, effectuez une des actions suivantes :

* Supprimez le groupe de ressources en utilisant la commande [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

    ```azurepowershell-interactive

    Remove-AzResourceGroup -Name <source-resource-group-name>
    ```

* Supprimez le réseau virtuel source en utilisant [Remove-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/remove-azvirtualnetwork?view=azps-2.6.0) :  
    ``` azurepowershell-interactive

    Remove-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un réseau virtuel d’une région à une autre en utilisant PowerShell, puis vous avez nettoyé les ressources sources non nécessaires. Pour plus d’informations sur le déplacement de ressources entre régions et la reprise d’activité dans Azure, consultez :

- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
