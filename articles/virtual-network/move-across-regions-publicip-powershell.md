---
title: Déplacer la configuration des adresses IP publiques Azure vers une autre région Azure via Azure PowerShell
description: Utilisez un modèle Azure Resource Manager pour déplacer la configuration d’adresses IP publiques d’une région Azure vers une autre à l’aide d’Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.topic: how-to
ms.date: 08/29/2019
ms.author: allensu
ms.openlocfilehash: 4f72c22ee26375e025af7b3a391fdd45187e7041
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006266"
---
# <a name="move-azure-public-ip-configuration-to-another-region-using-azure-powershell"></a>Déplacer la configuration d’adresses IP publiques Azure vers une autre région à l’aide d’Azure PowerShell

Il existe différents scénarios dans lesquels vous pouvez être amené à déplacer les configurations de vos adresses IP publiques Azure existantes d’une région à une autre. Par exemple, vous pouvez avoir besoin de créer une adresse IP publique avec la même configuration et la même référence SKU à des fins de test. Vous pouvez également déplacer une configuration d’adresses IP publiques vers une autre région dans le cadre de la planification de la récupération d’urgence.

**Les adresses IP publiques Azure sont spécifiques à une région et ne peuvent pas être déplacées d’une région à une autre.** Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante d’une adresse IP publique.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’adresse IP publique vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant le modèle dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Exporter des groupes de ressources vers des modèles](https://docs.microsoft.com/azure/azure-resource-manager/manage-resource-groups-powershell#export-resource-groups-to-templates)


## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’adresse IP publique Azure se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Vous ne pouvez pas déplacer les adresses IP publiques Azure entre les régions.  Vous devez associer la nouvelle adresse IP publique à des ressources dans la région cible.

- Pour exporter une configuration d’adresse IP publique et déployer un modèle pour créer une adresse IP publique dans une autre région, vous devez disposer au minimum du rôle Contributeur de réseaux.
   
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des adresses IP publiques dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’adresses IP publiques pour ce processus.  Consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer l’adresse IP publique pour le déplacement à l’aide d’un modèle Resource Manager et comment déplacer la configuration de l’adresse IP publique vers la région cible à l’aide d’Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-template-and-deploy-from-a-script"></a>Exporter le modèle et le déployer à partir d’un script

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenez l’ID de ressource de l’adresse IP publique que vous souhaitez déplacer vers la région cible, puis placez-la dans une variable à l’aide de [Get-AzPublicIPAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourcePubIPID = (Get-AzPublicIPaddress -Name <source-public-ip-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez le réseau virtuel source vers un fichier .json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

5. Pour modifier le paramètre du nom d’adresse IP publique, remplacez la propriété **defaultValue** par le nom de votre adresse IP publique cible, en veillant à placer le nom entre guillemets :
    
    ```json
        {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publicIPAddresses_myVM1pubIP_name": {
        "defaultValue": "<target-publicip-name>",
        "type": "String"
        }
    }

    ```

6. Pour modifier la région cible où l’adresse IP publique sera déplacée, changez la propriété **location** sous resources :

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
               }
               }
             ]             
    ```
  
7. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](https://docs.microsoft.com/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8. Vous pouvez également changer d’autres paramètres dans le modèle ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Référence SKU** : vous pouvez permuter la SKU de l’adresse IP publique dans la configuration entre les valeurs « basic » et « standard » en modifiant la propriété **sku** > **name** dans le fichier **\<resource-group-name>.json** :

         ```json
            "resources": [
                   {
                    "type": "Microsoft.Network/publicIPAddresses",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                    "location": "<target-region>",
                    "sku": {
                        "name": "Basic",
                        "tier": "Regional"
                    },
         ```

         Pour plus d’informations sur les différences entre les adresses IP publiques des références SKU basic et standard, consultez [Créer, modifier ou supprimer une adresse IP publique](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address).

    * **Méthode d’allocation d’adresse IP publique** et **Délai d’inactivité** : vous pouvez changer ces deux options dans le modèle en permutant la propriété **publicIPAllocationMethod** entre les valeurs **Dynamic** et **Static**. Vous pouvez changer le délai d’inactivité en affectant la valeur souhaitée à la propriété **idleTimeoutInMinutes**.  La valeur par défaut est **4** :

         ```json
         "resources": [
                {
                "type": "Microsoft.Network/publicIPAddresses",
                "apiVersion": "2019-06-01",
                "name": "[parameters('publicIPAddresses_myPubIP_name')]",
                "location": "<target-region>",
                  "sku": {
                  "name": "Basic",
                  "tier": "Regional"
                 },
                "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
                   }
                }            
         ```

        Pour plus d’informations sur les méthodes d’allocation et les valeurs de délai d’inactivité, consultez [Créer, modifier ou supprimer une adresse IP publique](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address).


9. Enregistrez le fichier **\<resource-group-name>.json**.

10. Créez un groupe de ressources dans la région cible pour l'adresse IP publique cible à déployer à l’aide de [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0).
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Déployez le fichier **\<resource-group-name>.json** modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzPublicIPAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzPublicIPAddress -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```
## <a name="discard"></a>Abandonner 

Après le déploiement, si vous souhaitez recommencer ou ignorer l’adresse IP publique dans la cible, supprimez le groupe de ressources qui a été créé dans la cible et l’adresse IP publique déplacée sera supprimée.  Pour supprimer le groupe de ressources, utilisez [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>

```

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du réseau virtuel, supprimez le réseau virtuel ou le groupe de ressources source avec [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) ou [Remove-AzPublicIPAddress](https://docs.microsoft.com/powershell/module/az.network/remove-azpublicipaddress?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <source-resource-group-name>

```

``` azurepowershell-interactive

Remove-AzPublicIpAddress -Name <source-publicip-name> -ResourceGroupName <resource-group-name>

```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé une adresse IP publique Azure d’une région à une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
