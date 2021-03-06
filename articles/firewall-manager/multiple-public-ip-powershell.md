---
title: Déployer un pare-feu Azure avec plusieurs adresses IP publiques à l’aide d’Azure PowerShell
description: Déployez un pare-feu avec plusieurs adresses IP publiques pour protéger un hub virtuel
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: how-to
ms.date: 07/09/2020
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 670e909d322cb5e1c9818508f19f102d3c836556
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/29/2021
ms.locfileid: "110700629"
---
# <a name="deploy-an-azure-firewall-with-multiple-public-ip-addresses"></a>Déployer un pare-feu Azure avec plusieurs adresses IP publiques

Si vous souhaitez protéger un hub virtuel à l’aide du pare-feu Azure, vous pouvez déployer ce pare-feu avec plusieurs adresses IP publiques avec Azure PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-the-firewall"></a>Déployer le pare-feu

Utilisez l’exemple Azure PowerShell suivant pour déployer un pare-feu Azure avec plusieurs adresses IP publiques afin de protéger un hub virtuel.

```azurepowershell
Select-AzSubscription -SubscriptionId <subscription ID> 

$rgName = <resource group name> 
$vHub = Get-AzVirtualHub -Name <hub name> 
$vHubId = $vHub.Id 
$fwpips = New-AzFirewallHubPublicIpAddress -Count 3
$hubIpAddresses = New-AzFirewallHubIpAddress -PublicIPs $fwpips 
$fw = New-AzFirewall -Name <firewall name> -ResourceGroupName $rgName `
     -Location <location> -Sku AZFW_Hub -HubIPAddresses $hubIpAddresses `
     -VirtualHubId $vHubId 
```

### <a name="update-a-public-ip-address"></a>Mettre à jour une IP publique

Vous pouvez utiliser Azure PowerShell pour mettre à jour l’IP publique d’un pare-feu Azure. L’exemple suivant supprime une adresse IP publique d’un pare-feu. Il commence avec trois adresses IP publiques.

```azurepowershell
Select-AzSubscription -SubscriptionId <subscription ID>

$azfw = get-azfirewall -Name <firewall name> -ResourceGroupName <resource group name>
$ip1 = New-AzFirewallPublicIpAddress -Address <first ip address to keep>
$ip2 = New-AzFirewallPublicIpAddress -Address <second ip address to keep>
$ipAddresses = $ip1,$ip2
$azfw.HubIPAddresses.publicIPs.Addresses = $ipAddresses
$azfw.HubIPAddresses.publicIPs.count = 2
Set-AzFirewall -AzureFirewall $azfw
```

## <a name="next-steps"></a>Étapes suivantes

[Qu’est-ce qu’un hub virtuel sécurisé ?](secured-virtual-hub.md)
