---
title: 'Virtual WAN : Créer une table de routes de hub virtuel sur des appliances réseau virtuelles : Azure PowerShell'
description: Table d’itinéraires de hub virtuel Azure Virtual WAN pour diriger le trafic vers une appliance réseau virtuelle.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to work with routing tables for NVA.
ms.openlocfilehash: 40154a9c3cefed69bd3f1639153099b944da79b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91267123"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>Créer une table d’itinéraires de hub virtuel pour diriger le trafic vers une appliance réseau virtuelle

Dans cet article, découvrez comment diriger le trafic à partir d’un hub virtuel vers une appliance réseau virtuelle. 

![Diagramme WAN virtuel](./media/virtual-wan-route-table-nva/vwanroute.png)

Dans cet article, vous apprendrez comment :

* Créer un WAN
* Créer un hub
* Créer des connexions de réseau virtuel du hub
* Créer un itinéraire de hub
* Créer une table de routage
* Appliquer la table d’itinéraires

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vérifiez que vous respectez les critères suivants :

* Vous avez une appliance virtuelle réseau (NVA). Il s’agit d’un logiciel tiers de votre choix qui est généralement provisionné à partir de la Place de marché Azure dans un réseau virtuel.
* Vous disposez d’une adresse IP privée assignée à l’interface de l’appliance réseau virtuelle. 
* L’appliance réseau virtuelle ne peut pas être déployée dans le hub virtuel. Elle doit être déployée dans un réseau virtuel distinct. Dans cet article, le réseau virtuel NVA est appelé « DMZ VNet ».
* Un ou plusieurs réseaux virtuels peuvent être connectés au réseau « DMZ VNet ». Dans cet article, ce réseau virtuel est appelé « Indirect spoke VNet ». Ces réseaux virtuels peuvent être connectés au réseau DMZ VNet à l’aide du peering.
* Vérifiez que les deux réseaux virtuels sont créés. Ils seront utilisés en tant que réseaux virtuels spokes. Dans cet article, le réseau virtuel spoke bénéficie des espaces d’adressage 10.0.2.0/24 et 10.0.3.0/24. Si vous avez besoin d’informations sur la création d’un réseau virtuel, consultez l’article [Créer un réseau virtuel à l’aide de PowerShell](../virtual-network/quick-create-powershell.md).
* Veillez à ce qu’aucune passerelle de réseau virtuel n’existe dans les réseaux virtuels.

## <a name="1-sign-in"></a><a name="signin"></a>1. Se connecter

Assurez-vous d’avoir installé la dernière version des cmdlets PowerShell de Resource Manager. Pour plus d’informations sur l’installation des applets de commande PowerShell, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/install-az-ps). Ceci est important, car les versions antérieures des cmdlets ne contiennent pas les valeurs actuelles dont vous avez besoin pour cet exercice.

1. Ouvrez la console PowerShell avec des privilèges élevés, puis connectez-vous à votre compte Azure. Ce cmdlet vous invite à entrer vos informations d’identification. Une fois que vous êtes connecté, vos paramètres de compte sont téléchargés afin de les mettre à disposition d’Azure PowerShell.

   ```powershell
   Connect-AzAccount
   ```
2. Obtenez la liste de vos abonnements Azure.

   ```powershell
   Get-AzSubscription
   ```
3. Spécifiez l’abonnement à utiliser.

   ```powershell
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```

## <a name="2-create-resources"></a><a name="rg"></a>2. Créer des ressources

1. Créez un groupe de ressources.

   ```powershell
   New-AzResourceGroup -Location "West US" -Name "testRG"
   ```
2. Créez un WAN virtuel.

   ```powershell
   $virtualWan = New-AzVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "West US"
   ```
3. Créez un hub virtuel.

   ```powershell
   New-AzVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "westushub" -AddressPrefix "10.0.1.0/24" -Location "West US"
   ```

## <a name="3-create-connections"></a><a name="connections"></a>3. Créer des connexions

Créez des connexions de réseau virtuel du hub à partir des réseaux Indirect Spoke VNet et DMZ VNet vers le hub virtuel.

  ```powershell
  $remoteVirtualNetwork1= Get-AzVirtualNetwork -Name "indirectspoke1" -ResourceGroupName "testRG"
  $remoteVirtualNetwork2= Get-AzVirtualNetwork -Name "indirectspoke2" -ResourceGroupName "testRG"
  $remoteVirtualNetwork3= Get-AzVirtualNetwork -Name "dmzvnet" -ResourceGroupName "testRG"

  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection1" -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection2" -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection3" -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

## <a name="4-create-a-virtual-hub-route"></a><a name="route"></a>4. Créer un itinéraire de hub virtuel

Dans cet article, le réseau Indirect Spoke VNet bénéficie des espaces d’adressage 10.0.2.0/24 et 10.0.3.0/24 et l’adresse IP privée de l’interface du réseau DMZ VNet est 10.0.4.5.

```powershell
$route1 = New-AzVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

## <a name="5-create-a-virtual-hub-route-table"></a><a name="applyroute"></a>5. Créer une table d’itinéraires de hub virtuel

Créez une table d’itinéraires de hub virtuel, puis appliquez-y l’itinéraire créé.
 
```powershell
$routeTable = New-AzVirtualHubRouteTable -Route @($route1)
```

## <a name="6-commit-the-changes"></a><a name="commit"></a>6. Valider les modifications

Validez les modifications pour le hub virtuel.

```powershell
Update-AzVirtualHub -ResourceGroupName "testRG" -Name "westushub" -RouteTable $routeTable
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez la page [Vue d’ensemble de Virtual WAN](virtual-wan-about.md).
