---
title: Déplacer un Azure Load Balancer interne vers une autre région Azure à l’aide d’Azure PowerShell
description: Utilisez un modèle Azure Resource Manager pour déplacer un Azure Load Balancer interne d’une région Azure vers une autre à l’aide d’Azure PowerShell.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.openlocfilehash: 73a9356de555e33996b92f05c3bbbabb651f1c9f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014225"
---
# <a name="move-azure-internal-load-balancer-to-another-region-using-powershell"></a>Déplacer un Azure Load Balancer interne vers une autre région à l’aide de PowerShell

Il existe différents scénarios dans lesquels vous pourriez souhaiter déplacer votre équilibreur de charge interne existant d’une région à une autre. Par exemple, vous souhaiterez peut-être créer un équilibreur de charge interne avec la même configuration à des fins de test. Vous pouvez également déplacer un équilibreur de charge interne vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Les équilibreurs de charge internes Azure ne peuvent pas être déplacés d’une région à l’autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et le réseau virtuel d’un équilibreur de charge interne.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’équilibreur de charge et le réseau virtuel vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant les modèles dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Exporter des groupes de ressources vers des modèles](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)


## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’équilibreur de charge interne se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les équilibreurs de charge internes Azure ne peuvent pas être déplacés entre les régions.  Vous devez associer le nouvel équilibreur de charge à des ressources de la région cible.

- Pour exporter une configuration d’équilibreur de charge interne et déployer un modèle afin de créer un équilibreur de charge interne dans une autre région, vous devez disposer au moins du rôle Contributeur de réseaux.
   
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau, les machines virtuelles et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des équilibreurs de charge internes dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’équilibreurs de charge pour ce processus.  Voir [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer l’équilibreur de charge interne pour le déplacement à l’aide d’un modèle Resource Manager, et déplacer la configuration de l’équilibreur de charge interne vers la région cible à l’aide d’Azure PowerShell.  Dans le cadre de ce processus, la configuration du réseau virtuel de l’équilibreur de charge interne doit être incluse et doit être effectuée avant de déplacer l’équilibreur de charge interne.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-virtual-network-template-and-deploy-from-azure-powershell"></a>Exporter le modèle de réseau virtuel et le déployer à partir d’Azure PowerShell

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
2.  Obtenez l’ID de ressource du réseau virtuel que vous souhaitez déplacer vers la région cible, puis placez-le dans une variable à l’aide de [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourceVNETID = (Get-AzVirtualNetwork -Name <source-virtual-network-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez le réseau virtuel source vers un fichier .json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Pour modifier le paramètre du nom du réseau virtuel, remplacez la valeur de la propriété **defaultValue** du nom du réseau virtuel source par le nom de votre réseau virtuel cible, en vous assurant que le nom est entre guillemets :
    
    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentmyResourceGroupVNET.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_myVNET1_name": {
        "defaultValue": "<target-virtual-network-name>",
        "type": "String"
        }
    ```

6.  Pour modifier la région cible où le réseau virtuel sera déplacé, changez la propriété **location** sous Ressources :

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
  
7. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
8.  Vous pouvez également changer d’autres paramètres dans le fichier **\<resource-group-name>.json** ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Espace d’adressage** : l’espace d’adressage du réseau virtuel peut être modifié avant l’enregistrement en modifiant la section **resources** > **addressSpace** et en modifiant la propriété **addressPrefixes** dans le fichier **\<resource-group-name>.json** :

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

    * **Sous-réseau** : le nom du sous-réseau et l’espace d’adressage du sous-réseau peuvent être modifiés ou ajoutés en modifiant la section **subnets** du fichier **\<resource-group-name>.json**. Le nom du sous-réseau peut être changé en modifiant la propriété **name**. Vous pouvez modifier l’espace d’adressage du sous-réseau en modifiant la propriété **addressPrefix** dans le fichier **\<resource-group-name>.json** :

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

         Dans le fichier **\<resource-group-name>.json**, pour modifier le préfixe d’adresse, vous devez le modifier à deux emplacements, la section ci-dessus et la section **type** indiquée ci-dessous.  Modifiez la propriété **addressPrefix** pour qu’elle corresponde à celle ci-dessus :

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

9.  Enregistrez le fichier **\<resource-group-name>.json**.

10. Créez un groupe de ressources dans la région cible pour le réseau virtuel cible à déployer à l’aide de [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0)
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
    
11. Déployez le fichier **\<resource-group-name>.json** modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```
12. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzVirtualNetwork -Name <target-virtual-network-name> -ResourceGroupName <target-resource-group-name>

    ```
### <a name="export-the-internal-load-balancer-template-and-deploy-from-azure-powershell"></a>Exporter le modèle d’équilibreur de charge interne et le déployer à partir d’Azure PowerShell

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenez l’ID de ressource de l’équilibreur de charge interne que vous souhaitez déplacer vers la région cible, puis placez-le dans une variable à l’aide de [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourceIntLBID = (Get-AzLoadBalancer -Name <source-internal-lb-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez la configuration de l’équilibreur de charge interne source vers un fichier. json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceIntLBID -IncludeParameterDefaultValue
   ```
4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Pour modifier le paramètre du nom de l’équilibreur de charge interne, remplacez la propriété **defaultValue** du nom de l’équilibreur de charge interne source par le nom de votre équilibreur de charge interne cible, en vous assurant que le nom est entre guillemets :

    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_externalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```
 
6. Pour modifier la valeur du réseau virtuel cible qui a été déplacé ci-dessus, vous devez d’abord obtenir l’ID de ressource, puis le copier et le coller dans le fichier **\<resource-group-name>.json**.  Pour obtenir l’ID, utilisez [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
    $targetVNETID = (Get-AzVirtualNetwork -Name <target-vnet-name> -ResourceGroupName <target-resource-group-name>).Id
    ```
    Entrez la variable et appuyez sur Entrée pour afficher l’ID de ressource.  Mettez en surbrillance le chemin de l’ID et copiez-le dans le Presse-papiers :

    ```powershell
    PS C:\> $targetVNETID
    /subscriptions/7668d659-17fc-4ffd-85ba-9de61fe977e8/resourceGroups/myResourceGroupVNET-Move/providers/Microsoft.Network/virtualNetworks/myVNET2-Move
    ```

7.  Dans le fichier **\<resource-group-name>.json**, collez l’**ID de ressource** de la variable à la place de **defaultValue** dans le deuxième paramètre pour l’ID de réseau virtuel cible, en veillant à placer le chemin entre guillemets :
   
    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_externalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```

8. Pour modifier la région cible où la configuration de l’équilibreur de charge interne sera déplacée, modifiez la propriété **location** sous **resources** dans le fichier **\<resource-group-name>.json** :

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-internal-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

11. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

    ```azurepowershell-interactive

    Get-AzLocation | format-table
    
    ```
12. Vous pouvez également changer d’autres paramètres dans le modèle ; ces paramètres sont facultatifs en fonction de vos besoins :
    
    * **Référence SKU** : vous pouvez permuter la référence SKU de l’équilibreur de charge interne dans la configuration entre les valeurs basic et standard en modifiant la propriété **sku** > **name** dans le fichier **\<resource-group-name>.json** :

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-internal-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Pour plus d’informations sur les différences entre les équilibreurs de charge des références SKU basic et standard, consultez [Présentation d’Azure Standard Load Balancer](./load-balancer-overview.md).

    * **Règles d’équilibrage de charge** : vous pouvez ajouter ou supprimer des règles d’équilibrage de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **loadBalancingRules** du fichier **\<resource-group-name>.json** :

        ```json
        "loadBalancingRules": [
                    {
                        "name": "myInboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false,
                            "loadDistribution": "Default",
                            "disableOutboundSnat": true,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolInbound')]"
                            },
                            "probe": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/probes/myHTTPProbe')]"
                            }
                        }
                    }
                ]
        ```
       Pour plus d’informations sur les règles d’équilibrage de charge, consultez [Qu’est-ce qu’Azure Load Balancer ?](./load-balancer-overview.md).

    * **Sondes** : vous pouvez ajouter ou supprimer une sonde pour l’équilibreur de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **probes** du fichier **\<resource-group-name>.json** :

        ```json
        "probes": [
                    {
                        "name": "myHTTPProbe",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "Http",
                            "port": 80,
                            "requestPath": "/",
                            "intervalInSeconds": 15,
                            "numberOfProbes": 2
                        }
                    }
                ],
        ```
       Pour plus d’informations sur les sondes d’intégrité Azure Load Balancer, consultez [Sondes d’intégrité Load Balancer](./load-balancer-custom-probe-overview.md).

    * **Règle NAT de trafic entrant**  : vous pouvez ajouter ou supprimer des règles NAT de trafic entrant pour l’équilibreur de charge en ajoutant ou en supprimant des entrées dans la section **inboundNatRules** du fichier **\<resource-group-name>.json** :

        ```json
        "inboundNatRules": [
                    {
                        "name": "myInboundNATRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 4422,
                            "backendPort": 3389,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false
                        }
                    }
                ]
        ```
        Pour terminer l’ajout ou la suppression d’une règle NAT de trafic entrant, il faut que la règle soit présente ou supprimée en tant que propriété **type** à la fin du fichier **\<resource-group-name>.json** :

        ```json
        {
            "type": "Microsoft.Network/loadBalancers/inboundNatRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('loadBalancers_myLoadBalancer_name'), '/myInboundNATRule')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "frontendIPConfiguration": {
                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                },
                "frontendPort": 4422,
                "backendPort": 3389,
                "enableFloatingIP": false,
                "idleTimeoutInMinutes": 4,
                "protocol": "Tcp",
                "enableTcpReset": false
            }
        }
        ```
        Pour plus d’informations sur les règles NAT de trafic entrant, consultez [Qu’est-ce qu’Azure Load Balancer ?](./load-balancer-overview.md)
    
13. Enregistrez le fichier **\<resource-group-name>.json**.
    
10. Créez un groupe de ressources dans la région cible pour l’équilibreur de charge interne cible à déployer à l’aide de [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0). Le groupe de ressources existant ci-dessus peut également être réutilisé dans le cadre de ce processus :
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Déployez le fichier **\<resource-group-name>.json** modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzLoadBalancer -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

## <a name="discard"></a>Abandonner 

Après le déploiement, si vous souhaitez recommencer ou ignorer le réseau virtuel et l’équilibreur de charge dans la cible, supprimez le groupe de ressources qui a été créé dans la cible, et le réseau virtuel et l’équilibreur de charge déplacés seront supprimés.  Pour supprimer le groupe de ressources, utilisez [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du groupe de sécurité réseau, supprimez le groupe de ressources ou le groupe de sécurité réseau source avec [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) ou [Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork?view=azps-2.6.0) et [Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer?view=azps-2.6.0).

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

``` azurepowershell-interactive

Remove-AzLoadBalancer -name <load-balancer> -ResourceGroupName <resource-group-name>

Remove-AzVirtualNetwork -Name <virtual-network-name> -ResourceGroupName <resource-group-name>


```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un équilibreur de charge interne Azure d’une région à une autre et vous avez nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)