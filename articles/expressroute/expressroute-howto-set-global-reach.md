---
title: 'Azure ExpressRoute : Configurer Global Reach'
description: Cet article vous aide à associer des circuits ExpressRoute afin de constituer un réseau privé entre vos réseaux locaux et d’activer Global Reach.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.openlocfilehash: 2365317d83a4c11fa17cb5c449a25b70da17c2eb
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368364"
---
# <a name="configure-expressroute-global-reach"></a>Configurer ExpressRoute Global Reach

Cet article vous permet de configurer ExpressRoute Global Reach à l’aide de PowerShell. Pour plus d’informations, consultez [ExpressRoute Global Reach](expressroute-global-reach.md).

 ## <a name="before-you-begin"></a>Avant de commencer

Avant de commencer la configuration, vérifiez les éléments suivants :

* Vous connaissez les [flux de travail](expressroute-workflows.md) de provisionnement du circuit ExpressRoute.
* Vos circuits ExpressRoute sont à l’état provisionné.
* Le peering privé Azure est configuré sur vos circuits ExpressRoute.
* Si vous souhaitez exécuter PowerShell localement, vérifiez que la dernière version d’Azure PowerShell est installée sur votre ordinateur.

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="identify-circuits"></a>Identifier les circuits

1. Pour commencer la configuration, connectez-vous à votre compte Azure et sélectionnez l’abonnement que vous souhaitez utiliser.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]
2. Identifiez les circuits ExpressRoute que vous souhaitez utiliser. Vous pouvez activer ExpressRoute Global Reach entre le peering privé de deux circuits ExpressRoute à condition qu’ils soient dans les pays/régions pris en charge et qu’ils aient été créés à différents endroits de peering. 

   * Si votre abonnement comprend les deux circuits, vous pouvez choisir celui de votre choix pour exécuter la configuration dans les sections suivantes.
   * Si les deux circuits se trouvent dans des abonnements Azure différents, vous devez avoir l’autorisation d’un abonnement Azure, puis transférer la clé d’autorisation lorsque vous exécutez la commande de configuration dans l’autre abonnement Azure.

## <a name="enable-connectivity"></a>Activez la connectivité.

Activez la connectivité entre vos réseaux locaux. Il existe des ensembles distincts d’instructions pour les circuits qui sont dans le même abonnement Azure et ceux qui sont dans des abonnements différents.

### <a name="expressroute-circuits-in-the-same-azure-subscription"></a>Circuits ExpressRoute dans le même abonnement Azure

1. Utilisez les commandes suivantes pour obtenir le circuit 1 et le circuit 2. Les deux circuits se trouvent dans le même abonnement.

   ```azurepowershell-interactive
   $ckt_1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
   $ckt_2 = Get-AzExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
   ```
2. Exécutez la commande suivante sur le circuit 1 et transférez l’ID du peering privé du circuit 2. Lors de l'exécution de la commande, notez ce qui suit :

   * L'ID du peering privé ressemble à l’exemple suivant : 

     ```
     /subscriptions/{your_subscription_id}/resourceGroups/{your_resource_group}/providers/Microsoft.Network/expressRouteCircuits/{your_circuit_name}/peerings/AzurePrivatePeering
     ```
   * *- AddressPrefix* doit correspondre à un sous-réseau /29 IPv4, par exemple « 10.0.0.0/29 ». Nous utilisons les adresses IP de ce sous-réseau pour établir la connexion entre les deux circuits ExpressRoute. Vous ne devez pas utiliser ces adresses dans vos réseaux virtuels Azure ou vos réseaux locaux.

     ```azurepowershell-interactive
     Add-AzExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering $ckt_2.Peerings[0].Id -AddressPrefix '__.__.__.__/29'
     ```
3. Enregistrez la configuration sur le circuit 1 comme suit :

   ```azurepowershell-interactive
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
   ```

Une fois l’opération terminée, vous disposerez d’une connectivité entre vos réseaux locaux des deux côtés par le biais des deux circuits ExpressRoute.

### <a name="expressroute-circuits-in-different-azure-subscriptions"></a>Circuits ExpressRoute dans différents abonnements Azure

Si les deux circuits ne se trouvent pas dans le même abonnement Azure, vous devez disposer d’une autorisation. Dans la configuration suivante, l’autorisation est générée dans l’abonnement du circuit 2 et la clé d’autorisation est transférée au circuit 1.

1. Générez une clé d’autorisation.

   ```azurepowershell-interactive
   $ckt_2 = Get-AzExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
   Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $ckt_2 -Name "Name_for_auth_key"
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_2
   ```

   Notez l’ID du peering privé du circuit 2, ainsi que la clé d’autorisation.
2. Exécutez la commande suivante sur le circuit 1. Transférez l’ID du peering privé du circuit 2, ainsi que la clé d’autorisation.

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering "circuit_2_private_peering_id" -AddressPrefix '__.__.__.__/29' -AuthorizationKey '########-####-####-####-############'
   ```
3. Enregistrez la configuration sur le circuit 1.

   ```azurepowershell-interactive
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
   ```

Une fois l’opération terminée, vous disposerez d’une connectivité entre vos réseaux locaux des deux côtés par le biais des deux circuits ExpressRoute.

## <a name="verify-the-configuration"></a>Vérifier la configuration

Utilisez la commande suivante pour vérifier la configuration sur le circuit où elle a été effectuée (par exemple, le circuit 1 dans l'exemple précédent).
```azurepowershell-interactive
$ckt_1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
```

Si vous exécutez simplement *$ckt_1* dans PowerShell, vous voyez s’afficher la valeur *CircuitConnectionStatus* dans le résultat. Cela indique si la connectivité est établie, « Connecté » ou « Déconnecté ». 

## <a name="disable-connectivity"></a>Désactiver la connectivité

Pour désactiver la connectivité entre vos réseaux locaux, exécutez les commandes sur le circuit où la configuration a été effectuée (le circuit 1 dans l’exemple précédent).

```azurepowershell-interactive
$ckt_1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
Remove-AzExpressRouteCircuitConnectionConfig -Name "Your_connection_name" -ExpressRouteCircuit $ckt_1
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
```

Vous pouvez exécuter l’opération Get pour vérifier l’état.

Une fois l’opération précédente terminée, la connexion entre vos réseaux locaux via vos circuits ExpressRoute n’est plus établie.

## <a name="next-steps"></a>Étapes suivantes
1. [En savoir plus sur ExpressRoute Global Reach](expressroute-global-reach.md)
2. [Vérifier la connectivité ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
3. [Lier un circuit ExpressRoute à un réseau virtuel Azure](expressroute-howto-linkvnet-arm.md)
