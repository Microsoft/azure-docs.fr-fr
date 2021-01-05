---
title: Déplacer un groupe de sécurité réseau Azure vers une autre région Azure à l’aide d’Azure PowerShell
description: Utilisez un modèle Azure Resource Manager pour déplacer un groupe de sécurité réseau Azure d’une région Azure vers une autre à l’aide d’Azure PowerShell.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/31/2019
ms.author: allensu
ms.openlocfilehash: 04abc051cec8a6fb38ce6aa8f5347ae06cb8bd1d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019750"
---
# <a name="move-azure-network-security-group-nsg-to-another-region-using-azure-powershell"></a>Déplacer un groupe de sécurité réseau Azure vers une autre région à l’aide d’Azure PowerShell

Il existe différents scénarios dans lesquels vous pouvez être amené à déplacer vos groupe de sécurité réseau existants d’une région à une autre. Par exemple, vous pouvez avoir besoin de créer un groupe de sécurité réseau avec la même configuration et les mêmes règles de sécurité à des fins de test. Vous pouvez également déplacer un groupe de sécurité réseau vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Les groupes de sécurité Azure ne peuvent pas être déplacés d’une région vers une autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et les règles de sécurité d’un groupe de sécurité réseau.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant le groupe de sécurité réseau vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant le modèle dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Exporter des groupes de ressources vers des modèles](https://docs.microsoft.com/azure/azure-resource-manager/manage-resource-groups-powershell#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que le groupe de sécurité réseau Azure se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les groupes de sécurité réseau Azure ne peuvent pas être déplacés entre les régions.  Vous devez associer le nouveau groupe de sécurité réseau à des ressources dans la région cible.

- Pour exporter une configuration de groupe de sécurité réseau et déployer un modèle afin de créer un groupe de sécurité réseau dans une autre région, vous devez disposer au minimum du rôle Contributeur de réseaux.
   
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les adresses IP publiques et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des groupes de sécurité réseau dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout de groupes de sécurité réseau pour ce processus.  Consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer le groupe de sécurité réseau pour le déplacement de la configuration et de la règle de sécurité à l’aide d’un modèle Resource Manager et comment déplacer la configuration et les règles de sécurité du groupe de sécurité réseau vers la région cible à l’aide d’Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-template-and-deploy-from-a-script"></a>Exporter le modèle et le déployer à partir d’un script

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenez l’ID de ressource du groupe de sécurité réseau que vous souhaitez déplacer vers la région cible, puis placez-le dans une variable à l’aide de [Get-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/get-aznetworksecuritygroup?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourceNSGID = (Get-AzNetworkSecurityGroup -Name <source-nsg-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez le groupe de sécurité réseau source vers un fichier .json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceNSGID -IncludeParameterDefaultValue
   ```

4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad <source-resource-group-name>.json
   ```

5. Pour modifier le paramètre du nom de groupe de sécurité réseau, remplacez la propriété **defaultValue** par le nom de votre groupe de sécurité réseau cible, en veillant à placer le nom entre guillemets :
    
    ```json
            {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "networkSecurityGroups_myVM1_nsg_name": {
            "defaultValue": "<target-nsg-name>",
            "type": "String"
            }
        }

    ```


6. Pour modifier la région cible où la configuration et les règles de sécurité du groupe de sécurité réseau seront déplacées, modifiez la propriété **location** sous **resources** :

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-06-01",
            "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
            "location": "<target-region>",
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78", 
             }
            }
    ```
  
7. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](https://docs.microsoft.com/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8. Vous pouvez également changer d’autres paramètres dans le fichier **\<resource-group-name>.json** ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Règles de sécurité** : vous pouvez modifier les règles déployées dans le groupe de sécurité réseau cible en ajoutant ou en supprimant des règles dans la section **securityRules** du fichier **\<resource-group-name>.json** :

        ```json
           "resources": [
                  {
                  "type": "Microsoft.Network/networkSecurityGroups",
                  "apiVersion": "2019-06-01",
                  "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
                  "location": "TARGET REGION",
                  "properties": {
                       "provisioningState": "Succeeded",
                       "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78",
                  "securityRules": [
                    {
                        "name": "RDP",
                        "etag": "W/\"c630c458-6b52-4202-8fd7-172b7ab49cf5\"",
                        "properties": {
                             "provisioningState": "Succeeded",
                             "protocol": "TCP",
                             "sourcePortRange": "*",
                             "destinationPortRange": "3389",
                             "sourceAddressPrefix": "*",
                             "destinationAddressPrefix": "*",
                             "access": "Allow",
                             "priority": 300,
                             "direction": "Inbound",
                             "sourcePortRanges": [],
                             "destinationPortRanges": [],
                             "sourceAddressPrefixes": [],
                             "destinationAddressPrefixes": []
                            }
                        ]
            }  
            
        ```

        Pour terminer l’ajout ou la suppression des règles dans le groupe de sécurité réseau cible, vous devez également modifier les types de règles personnalisées à la fin du fichier **\<resource-group-name>.json** conformément au format de l’exemple ci-dessous :

        ```json
           {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('networkSecurityGroups_myVM1_nsg_name'), '/Port_80')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_myVM1_nsg_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "protocol": "*",
                "sourcePortRange": "*",
                "destinationPortRange": "80",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 310,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
        ```

9. Enregistrez le fichier **\<resource-group-name>.json**.

10. Créez un groupe de ressources dans la région cible pour le groupe de sécurité réseau cible à déployer à l’aide de [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0) :
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
11. Déployez le fichier **\<resource-group-name>.json** modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/get-aznetworksecuritygroup?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzNetworkSecurityGroup -Name <target-nsg-name> -ResourceGroupName <target-resource-group-name>

    ```

## <a name="discard"></a>Abandonner 

Après le déploiement, si vous souhaitez recommencer ou ignorer le groupe de sécurité réseau dans la cible, supprimez le groupe de ressources qui a été créé dans la cible et le groupe de sécurité réseau déplacé sera supprimé.  Pour supprimer le groupe de ressources, utilisez [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <target-resource-group-name>

```

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du groupe de sécurité réseau, supprimez le groupe de ressources ou le groupe de sécurité réseau source avec [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) ou [Remove-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/remove-aznetworksecuritygroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <source-resource-group-name>

```

``` azurepowershell-interactive

Remove-AzNetworkSecurityGroup -Name <source-nsg-name> -ResourceGroupName <source-resource-group-name>

```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un groupe de sécurité réseau Azure d’une région à une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
