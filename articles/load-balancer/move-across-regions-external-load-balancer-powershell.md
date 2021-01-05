---
title: Déplacer un équilibreur de charge externe Azure vers une autre région Azure à l’aide d’Azure PowerShell
description: Utilisez un modèle Azure Resource Manager pour déplacer un équilibreur de charge externe Azure d’une région Azure vers une autre à l’aide d’Azure PowerShell.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.openlocfilehash: e43d8f1050f6b2b458c0926c674c05f7f18edc63
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018509"
---
# <a name="move-azure-external-load-balancer-to-another-region-using-azure-powershell"></a>Déplacer un équilibreur de charge externe Azure vers une autre région à l’aide d’Azure PowerShell

Il existe différents scénarios dans lesquels vous pourriez souhaiter déplacer votre équilibreur de charge externe existant d’une région à une autre. Par exemple, vous souhaiterez peut-être créer un équilibreur de charge externe avec la même configuration à des fins de test. Vous pouvez également déplacer un équilibreur de charge externe vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Les équilibreurs de charge externes Azure ne peuvent pas être déplacés d’une région à une autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et l’adresse IP publique d’un équilibreur de charge externe.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’équilibreur de charge et l’adresse IP publique vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant les modèles dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Exporter des groupes de ressources vers des modèles](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates)


## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’équilibreur de charge externe se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les équilibreurs de charge externes Azure ne peuvent pas être déplacés entre les régions.  Vous devez associer le nouvel équilibreur de charge à des ressources de la région cible.

- Pour exporter une configuration d’équilibreur de charge externe et déployer un modèle afin de créer un équilibreur de charge externe dans une autre région, vous devez disposer au moins du rôle Contributeur de réseaux.
   
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau, les adresses IP publiques et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des équilibreurs de charge externes dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’équilibreurs de charge pour ce processus.  Voir [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes expliquent comment préparer l’équilibreur de charge externe pour le déplacement à l’aide d’un modèle Resource Manager, et déplacer la configuration de l’équilibreur de charge externe vers la région cible à l’aide d'Azure PowerShell.  Dans le cadre de ce processus, la configuration des adresses IP publiques de l’équilibreur de charge externe doit être incluse et effectuée avant de déplacer l’équilibreur de charge externe.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-public-ip-template-and-deploy-from-azure-powershell"></a>Exporter le modèle d’adresse IP publique et le déployer à partir d'Azure PowerShell

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```
2. Obtenez l’ID de ressource de l’adresse IP publique que vous souhaitez déplacer vers la région cible, puis placez-la dans une variable à l’aide de [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourcePubIPID = (Get-AzPublicIPaddress -Name <source-public-ip-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez l'adresse IP publique source vers un fichier .json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceVNETID -IncludeParameterDefaultValue
   ```

4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
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
  
7. Pour obtenir les codes d’emplacement des régions, vous pouvez utiliser l’applet de commande Azure PowerShell [Get-AzLocation](/powershell/module/az.resources/get-azlocation?view=azps-1.8.0) en exécutant la commande suivante :

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

         Pour plus d’informations sur les différences entre les adresses IP publiques des références SKU basic et standard, consultez [Créer, modifier ou supprimer une adresse IP publique](../virtual-network/virtual-network-public-ip-address.md).

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

        Pour plus d’informations sur les méthodes d’allocation et les valeurs de délai d’inactivité, consultez [Créer, modifier ou supprimer une adresse IP publique](../virtual-network/virtual-network-public-ip-address.md).


9. Enregistrez le fichier **\<resource-group-name>.json**.

10. Créez un groupe de ressources dans la région cible pour l'adresse IP publique cible à déployer à l’aide de [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0).
    
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <target-resource-group-name> -location <target-region>
    ```
11. Déployez le fichier **\<resource-group-name>.json** modifié sur le groupe de ressources créé à l’étape précédente à l’aide de [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-2.6.0) :

    ```azurepowershell-interactive

    New-AzResourceGroupDeployment -ResourceGroupName <target-resource-group-name> -TemplateFile <source-resource-group-name>.json
    
    ```

12. Pour vérifier que les ressources ont été créées dans la région cible, utilisez [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup?view=azps-2.6.0) et [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress?view=azps-2.6.0) :
    
    ```azurepowershell-interactive

    Get-AzResourceGroup -Name <target-resource-group-name>

    ```

    ```azurepowershell-interactive

    Get-AzPublicIPAddress -Name <target-publicip-name> -ResourceGroupName <target-resource-group-name>

    ```

### <a name="export-the-external-load-balancer-template-and-deploy-from-azure-powershell"></a>Exporter le modèle d’équilibreur de charge externe et le déployer à partir d’Azure PowerShell

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) et suivez les instructions qui s'affichent à l’écran :
    
    ```azurepowershell-interactive
    Connect-AzAccount
    ```

2. Obtenez l’ID de ressource de l’équilibreur de charge externe que vous souhaitez déplacer vers la région cible, puis placez-le dans une variable à l’aide de [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $sourceExtLBID = (Get-AzLoadBalancer -Name <source-external-lb-name> -ResourceGroupName <source-resource-group-name>).Id

    ```
3. Exportez la configuration de l’équilibreur de charge externe source vers un fichier. json dans le répertoire où vous exécutez la commande [Export-AzResourceGroup](/powershell/module/az.resources/export-azresourcegroup?view=azps-2.6.0) :
   
   ```azurepowershell-interactive
   Export-AzResourceGroup -ResourceGroupName <source-resource-group-name> -Resource $sourceExtLBID -IncludeParameterDefaultValue
   ```
4. Le fichier téléchargé est nommé d’après le groupe de ressources à partir duquel la ressource a été exportée.  Recherchez le fichier **\<resource-group-name>.json** qui a été exporté à partir de la commande et ouvrez-le dans l’éditeur de votre choix :
   
   ```azurepowershell
   notepad.exe <source-resource-group-name>.json
   ```

5. Pour modifier le paramètre du nom de l’équilibreur de charge externe, remplacez la propriété **defaultValue** du nom de l’équilibreur de charge externe source par le nom de votre équilibreur de charge externe cible, en vous assurant que le nom est entre guillemets :

    ```json
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "loadBalancers_myLoadbalancer_ext_name": {
        "defaultValue": "<target-external-lb-name>",
        "type": "String"
            },
        "publicIPAddresses_myPubIP_in_externalid": {
        "defaultValue": "<target-publicIP-resource-ID>",
        "type": "String"
            },

    ```

6.  Pour modifier la valeur de l’adresse IP publique cible qui a été déplacée ci-dessus, vous devez d’abord obtenir l’ID de ressource, puis le copier et le coller dans le fichier **\<resource-group-name>.json**.  Pour obtenir l’ID, utilisez [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress?view=azps-2.6.0) :

    ```azurepowershell-interactive
    $targetPubIPID = (Get-AzPublicIPaddress -Name <target-public-ip-name> -ResourceGroupName <target-resource-group-name>).Id
    ```
    Entrez la variable et appuyez sur Entrée pour afficher l’ID de ressource.  Mettez en surbrillance le chemin de l’ID et copiez-le dans le Presse-papiers :

    ```powershell
    PS C:\> $targetPubIPID
    /subscriptions/7668d659-17fc-4ffd-85ba-9de61fe977e8/resourceGroups/myResourceGroupLB-Move/providers/Microsoft.Network/publicIPAddresses/myPubIP-in-move
    ```

7.  Dans le fichier **\<resource-group-name>.json**, collez l’**ID de ressource** issu de la variable à la place de **defaultValue** dans le deuxième paramètre pour l’ID externe d'adresse IP publique, en veillant à placer le chemin entre guillemets :

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            "loadBalancers_myLoadbalancer_ext_name": {
            "defaultValue": "<target-external-lb-name>",
            "type": "String"
            },
            "publicIPAddresses_myPubIP_in_externalid": {
            "defaultValue": "<target-publicIP-resource-ID>",
            "type": "String"
            },

    ```

8.  Si vous avez configuré la traduction d’adresses réseau pour le trafic sortant et des règles de trafic sortant pour l’équilibreur de charge, une troisième entrée sera présente dans ce fichier pour l’ID externe de l’adresse IP publique sortante.  Répétez les étapes ci-dessus dans la **région cible** pour obtenir l’ID de l’adresse IP publique sortante et collez cette entrée dans le fichier **\<resource-group-name>.json** :

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "loadBalancers_myLoadbalancer_ext_name": {
                "defaultValue": "<target-external-lb-name>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_in_externalid": {
                "defaultValue": "<target-publicIP-resource-ID>",
                "type": "String"
            },
                "publicIPAddresses_myPubIP_out_externalid": {
                "defaultValue": "<target-publicIP-outbound-resource-ID>",
                "type": "String"
            }
        },
    ```

10. Pour modifier la région cible où la configuration de l’équilibreur de charge interne sera déplacée, modifiez la propriété **location** sous **resources** dans le fichier **\<resource-group-name>.json** :

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-external-lb-region>",
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
    
    * **Référence SKU** : vous pouvez permuter la référence SKU de l’équilibreur de charge externe dans la configuration entre les valeurs basic et standard en modifiant la propriété **sku** > **name** dans le fichier **\<resource-group-name>.json** :

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-external-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Pour plus d’informations sur les différences entre les équilibreurs de charge des références SKU basic et standard, consultez [Présentation d’Azure Standard Load Balancer](./load-balancer-overview.md).

    * **Règles d’équilibrage de charge** : vous pouvez ajouter ou supprimer des règles d’équilibrage de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **loadBalancingRules** du fichier **\<resource-group-name>.json** :

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

    * **Sondes** : vous pouvez ajouter ou supprimer une sonde pour l’équilibreur de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **probes** du fichier **\<resource-group-name>.json** :

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

    * **Règle NAT de trafic entrant**  : vous pouvez ajouter ou supprimer des règles NAT de trafic entrant pour l’équilibreur de charge en ajoutant ou en supprimant des entrées dans la section **inboundNatRules** du fichier **\<resource-group-name>.json** :

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
        Pour terminer l’ajout ou la suppression d’une règle NAT de trafic entrant, il faut que la règle soit présente ou supprimée en tant que propriété **type** à la fin du fichier **\<resource-group-name>.json** :

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

    * **Règles de trafic sortant** : vous pouvez ajouter ou supprimer des règles de trafic sortant dans la configuration en modifiant la propriété **outboundRules** dans le fichier **\<resource-group-name>.json** :

        ```json
        "outboundRules": [
                    {
                        "name": "myOutboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "allocatedOutboundPorts": 10000,
                            "protocol": "All",
                            "enableTcpReset": false,
                            "idleTimeoutInMinutes": 15,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolOutbound')]"
                            },
                            "frontendIPConfigurations": [
                                {
                                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPoutbound')]"
                                }
                            ]
                        }
                    }
                ]
        ```

         Pour plus d’informations sur les règles de trafic sortant, consultez [Règles de trafic sortant dans Load Balancer](./load-balancer-outbound-connections.md#outboundrules).

13. Enregistrez le fichier **\<resource-group-name>.json**.
    
10. Créez un groupe de ressources dans la région cible pour l’équilibreur de charge externe cible à déployer à l’aide de [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup?view=azps-2.6.0). Le groupe de ressources existant ci-dessus peut également être réutilisé dans le cadre de ce processus :
    
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

Après le déploiement, si vous souhaitez recommencer ou ignorer l'adresse IP publique et l’équilibreur de charge dans la cible, supprimez le groupe de ressources qui a été créé dans la cible, et l'adresse IP publique et l’équilibreur de charge déplacés seront supprimés.  Pour supprimer le groupe de ressources, utilisez [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) :

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du groupe de sécurité réseau, supprimez le groupe de ressources ou le groupe de sécurité réseau source avec [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup?view=azps-2.6.0) ou [Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress?view=azps-2.6.0) et [Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer?view=azps-2.6.0).

```azurepowershell-interactive

Remove-AzResourceGroup -Name <resource-group-name>

```

``` azurepowershell-interactive

Remove-AzLoadBalancer -name <load-balancer> -ResourceGroupName <resource-group-name>

Remove-AzPublicIpAddress -Name <public-ip> -ResourceGroupName <resource-group-name>


```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un groupe de sécurité réseau Azure d’une région à une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)