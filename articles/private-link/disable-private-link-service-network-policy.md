---
title: 'Désactiver les stratégies réseau pour l’adresse IP source du service Azure Private Link '
description: Découvrez comment désactiver les stratégies réseau pour le service Azure Private Link
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.openlocfilehash: 1062f126da8be6b37f6b52eee520425b3edcde16
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84744338"
---
# <a name="disable-network-policies-for-private-link-service-source-ip"></a>Désactiver les stratégies réseau pour l’adresse IP source du service Private Link

Lorsque vous choisissez une adresse IP source pour votre service Private Link, vous devez configurer un paramètre de désactivation `privateLinkServiceNetworkPolicies` explicite dans le sous-réseau. Ce paramètre s’applique uniquement à l’adresse IP privée que vous avez choisie comme adresse IP source pour le service Private Link. Pour les autres ressources du sous-réseau, l’accès est contrôlé en fonction des règles de sécurité des groupes de sécurité réseau (NSG). 
 
Lorsque vous utilisez un client Azure (PowerShell, CLI ou des modèles), vous devez effectuer une étape supplémentaire qui consiste à modifier cette propriété. Vous pouvez désactiver la stratégie à l’aide de Cloud Shell dans le portail Azure, à partir d’installations locales d’Azure PowerShell ou d’Azure CLI, ou à l’aide de modèles Azure Resource Manager.  
 
Suivez les étapes ci-dessous afin de désactiver les stratégies réseau du service Private Link pour le réseau virtuel nommé *myVirtualNetwork*, dont le sous-réseau *par défaut* est hébergé dans un groupe de ressources nommé *myResourceGroup*. 

## <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell
Cette section explique comment désactiver les stratégies des points de terminaison privés d’un sous-réseau à l’aide d’Azure PowerShell.
Dans le code, remplacez « default » par le nom du sous-réseau virtuel.

```azurepowershell
$virtualSubnetName = "default"
$virtualNetwork= Get-AzVirtualNetwork `
  -Name "myVirtualNetwork" ` 
  -ResourceGroupName "myResourceGroup"
   
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq $virtualSubnetName} ).privateLinkServiceNetworkPolicies = "Disabled"  
 
$virtualNetwork | Set-AzVirtualNetwork 
```
## <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure
Cette section explique comment désactiver les stratégies des points de terminaison privés d’un sous-réseau à l’aide d’Azure CLI.
```azurecli
az network vnet subnet update \ 
  --name default \ 
  --resource-group myResourceGroup \ 
  --vnet-name myVirtualNetwork \ 
  --disable-private-link-service-network-policies true 
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
                        "privateLinkServiceNetworkPolicies": "Disabled" 
                    } 
                } 
        ] 
    } 
} 
 
```
## <a name="next-steps"></a>Étapes suivantes
- Découvrez [Azure Private Endpoint](private-endpoint-overview.md).
 
