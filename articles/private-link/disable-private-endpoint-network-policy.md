---
title: Désactiver les stratégies réseau pour les points de terminaison privés dans Azure
description: Découvrez comment désactiver les stratégies réseau pour les points de terminaison privés.
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.openlocfilehash: a4c04033f3fb5ff523f0d80bb6c978955f4c9ec2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84737510"
---
# <a name="disable-network-policies-for-private-endpoints"></a>Désactiver les stratégies réseau pour les points de terminaison privés

Les stratégies réseau, comme celles des groupes de sécurité réseau (NSG), ne sont pas applicables aux points de terminaison privés. Pour déployer un point de terminaison privé sur un sous-réseau donné, vous devez configurer un paramètre de désactivation explicite dans ce sous-réseau. Ce paramètre s’applique uniquement aux points de terminaison privés. Pour les autres ressources du sous-réseau, l’accès est contrôlé en fonction des règles de sécurité des groupes de sécurité réseau (NSG). 
 
Lorsque vous utilisez le portail pour créer un point de terminaison privé, ce paramètre est automatiquement désactivé dans le cadre du processus de création. Le déploiement à l’aide d’autres clients nécessite la modification de ce paramètre. Vous pouvez désactiver le paramètre à l’aide de Cloud Shell dans le portail Azure, à partir d’installations locales d’Azure PowerShell ou d’Azure CLI, ou à l’aide de modèles Azure Resource Manager.  
 
Les exemples suivants montrent comment désactiver `PrivateEndpointNetworkPolicies` pour le réseau virtuel nommé *myVirtualNetwork*, dont le sous-réseau *par défaut* est hébergé dans un groupe de ressources nommé *myResourceGroup*.

## <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell
Cette section explique comment désactiver les stratégies des points de terminaison privés d’un sous-réseau à l’aide d’Azure PowerShell.

```azurepowershell
$virtualNetwork= Get-AzVirtualNetwork `
  -Name "myVirtualNetwork" ` 
  -ResourceGroupName "myResourceGroup"  
   
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq 'default'} ).PrivateEndpointNetworkPolicies = "Disabled" 
 
$virtualNetwork | Set-AzVirtualNetwork 
```
## <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure
Cette section explique comment désactiver les stratégies des points de terminaison privés d’un sous-réseau à l’aide d’Azure CLI.
```azurecli
az network vnet subnet update \ 
  --name default \ 
  --resource-group myResourceGroup \ 
  --vnet-name myVirtualNetwork \ 
  --disable-private-endpoint-network-policies true
```
## <a name="using-a-template"></a>Utilisation d’un modèle
Cette section explique comment désactiver les stratégies des points de terminaison privés d’un sous-réseau à l’aide de modèles Azure Resource Manager.
```json
{ 
          "name": "myVirtualNetwork", 
          "type": "Microsoft.Network/virtualNetworks", 
          "apiVersion": "2019-04-01", 
          "location": "WestUS", 
          "properties": { 
                "addressSpace": { 
                     "addressPrefixes": [ 
                          "10.0.0.0/16" 
                        ] 
                  }, 
                  "subnets": [ 
                         { 
                                "name": "default", 
                                "properties": { 
                                    "addressPrefix": "10.0.0.0/24", 
                                    "privateEndpointNetworkPolicies": "Disabled" 
                                 } 
                         } 
                  ] 
          } 
} 
```
## <a name="next-steps"></a>Étapes suivantes
- Découvrez [Azure Private Endpoint](private-endpoint-overview.md).
 
