---
title: 'Didacticiel – Azure ExpressRoute : Ajouter une passerelle à un réseau virtuel – Azure PowerShell'
description: Ce tutoriel explique comment ajouter une passerelle de réseau virtuel à un réseau virtuel Resource Manager déjà créé pour ExpressRoute à l’aide d’Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/05/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 9f01961ec7c7f8e0a4e2d72e28e6def50e93ad5d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91854305"
---
# <a name="tutorial-configure-a-virtual-network-gateway-for-expressroute-using-powershell"></a>Tutoriel : Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide de PowerShell
> [!div class="op_single_selector"]
> * [Resource Manager - Portail Azure](expressroute-howto-add-gateway-portal-resource-manager.md)
> * [Resource Manager - PowerShell](expressroute-howto-add-gateway-resource-manager.md)
> * [Classic - PowerShell](expressroute-howto-add-gateway-classic.md)
> * [Vidéo - portail Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-vpn-gateway-for-your-virtual-network)
> 

Ce tutoriel vous aide à ajouter, redimensionner et supprimer une passerelle de réseau virtuel pour un réseau virtuel préexistant. Les étapes de cette configuration s’appliquent aux réseaux virtuels qui ont été créés à l’aide du modèle de déploiement Resource Manager pour une configuration ExpressRoute. Pour en savoir plus, consultez [À propos des passerelles de réseau virtuel pour ExpressRoute](expressroute-about-virtual-network-gateways.md).

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - créer un sous-réseau de passerelle ;
> - créer une passerelle de réseau virtuel.

## <a name="prerequisites"></a>Prérequis

### <a name="configuration-reference-list"></a>Liste de référence de configuration

Les étapes de cette tâche utilisent un réseau virtuel basé sur les valeurs figurant dans la liste de référence de configuration suivante. Les noms et paramètres supplémentaires sont également présentés dans cette liste. Nous n’utilisons pas cette liste directement dans la procédure, mais nous ajoutons des variables en fonction des valeurs dans cette liste. Vous pouvez copier la liste et l’utiliser en tant que référence, en remplaçant les valeurs par les vôtres.

| Paramètre                   | Valeur                                              |
| ---                       | ---                                                |
| Nom du réseau virtuel | *TestVNet* |    
| Espace d’adressage du réseau virtuel | *192.168.0.0/16* |
| Groupe de ressources | *TestRG* |
| Nom du sous-réseau 1 | *FrontEnd* |   
| Espace d’adressage du sous-réseau 1 | *192.168.1.0/24* |
| Nom du sous-réseau 1 | *FrontEnd* |
| Nom du sous-réseau de passerelle | *GatewaySubnet* |    
| Espace d’adressage du sous-réseau de passerelle | *192.168.200.0/26* |
| Région | *USA Est* |
| Nom de la passerelle | *GW* |   
| Nom d’IP de passerelle | *GWIP* |
| Nom de configuration IP de passerelle | *gwipconf* |
| Type | *ExpressRoute* |
| Nom d’IP publique de passerelle  | *gwpip* |

## <a name="add-a-gateway"></a>Ajout d’une passerelle

1. Pour vous connecter à Azure, exécutez `Connect-AzAccount`.

1. Déclarez vos variables pour cet exercice. Veillez à modifier l’exemple pour refléter les paramètres que vous souhaitez utiliser.

   ```azurepowershell-interactive 
   $RG = "TestRG"
   $Location = "East US"
   $GWName = "GW"
   $GWIPName = "GWIP"
   $GWIPconfName = "gwipconf"
   $VNetName = "TestVNet"
   ```
1. Stockez l’objet de réseau virtuel en tant que variable.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   ```
1. Ajoutez un sous-réseau de passerelle à votre réseau virtuel. Le sous-réseau de passerelle doit être nommé « GatewaySubnet ». Le sous-réseau de passerelle doit être d’une taille de /27 ou supérieure (/26,/25, etc.).

   ```azurepowershell-interactive
   Add-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26
   ```
1. Définissez la configuration.

   ```azurepowershell-interactive
   $vnet = Set-AzVirtualNetwork -VirtualNetwork $vnet
   ```
1. Stockez le sous-réseau de passerelle en tant que variable.

   ```azurepowershell-interactive
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
   ```
1. Demandez une adresse IP publique L’adresse IP est demandée avant de créer la passerelle. Vous ne pouvez pas spécifier l’adresse IP que vous souhaitez utiliser, car elle est attribuée de façon dynamique. Vous utiliserez cette adresse IP dans la section de configuration suivante. La méthode AllocationMethod doit être dynamique.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName  -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   ```
1. Créez la configuration de votre passerelle. La configuration de la passerelle définit le sous-réseau et l’adresse IP publique à utiliser. Dans cette étape, vous spécifiez la configuration qui sera utilisée lorsque vous créerez la passerelle. Utilisez l’exemple suivant pour créer la configuration de votre passerelle.

   ```azurepowershell-interactive
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```
1. Créez la passerelle. Dans cette étape, le type **-GatewayType** est particulièrement important. Vous devez utiliser la valeur **ExpressRoute**. Après l’exécution de ces cmdlets, la création de la passerelle peut prendre 45 minutes ou plus.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard
   ```

## <a name="verify-the-gateway-was-created"></a>Vérification de la création de la passerelle
Utilisez les commandes suivantes pour vérifier que la passerelle a été créée :

```azurepowershell-interactive
Get-AzVirtualNetworkGateway -ResourceGroupName $RG
```

## <a name="resize-a-gateway"></a>Redimensionner une passerelle
Il existe un certain nombre de [Références (SKU) de passerelle](expressroute-about-virtual-network-gateways.md). Vous pouvez utiliser la commande suivante pour modifier la référence de passerelle à tout moment.

> [!IMPORTANT]
> Cette commande ne fonctionne pas pour la passerelle UltraPerformance. Pour modifier votre passerelle en passerelle UltraPerformance, commencez par supprimer la passerelle ExpressRoute, puis créez une passerelle UltraPerformance. Pour mettre à niveau votre passerelle à partir d’une passerelle UltraPerformance, commencez par supprimer la passerelle UltraPerformance, puis créez-en une.
> 

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
Resize-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance
```

## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer la passerelle, utilisez la commande suivante :

```azurepowershell-interactive
Remove-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
```

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez créé la passerelle de réseau virtuel, vous pouvez lier votre réseau virtuel à un circuit ExpressRoute. 

> [!div class="nextstepaction"]
> [Lier un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)
