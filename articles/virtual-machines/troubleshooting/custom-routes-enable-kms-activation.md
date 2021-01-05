---
title: Utiliser des itinéraires personnalisés Azure pour permettre l’activation KMS avec le tunneling forcé | Microsoft Docs
description: Explique comment utiliser des itinéraires personnalisés Azure pour permettre l’activation KMS avec le tunneling forcé dans Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 12/20/2018
ms.author: genli
ms.openlocfilehash: 1c2050969e95b521554bba100b688add3a987a80
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86526739"
---
# <a name="windows-activation-fails-in-forced-tunneling-scenario"></a>L’activation de Windows échoue dans un scénario de tunneling forcé

Cet article explique comment résoudre le problème d’activation KMS que vous pouvez rencontrer si vous avez activé le tunneling forcé dans des scénarios ExpressRoute ou de connexion VPN de site à site.

## <a name="symptom"></a>Symptôme

Vous activez [le tunneling forcé](../../vpn-gateway/vpn-gateway-forced-tunneling-rm.md) sur les sous-réseaux de réseau virtuel Azure pour rediriger tout le trafic Internet vers votre réseau local. Dans ce scénario, les machines virtuelles Azure qui exécutent Windows ne parviennent pas à activer Windows.

## <a name="cause"></a>Cause

Les machines virtuelles Windows Azure doivent se connecter au serveur Azure KMS pour l’activation des fenêtres. L’activation exige que la demande d’activation provienne d’une adresse IP publique Azure. Dans le scénario de tunneling forcé, l’activation échoue parce que la demande d’activation provient de votre réseau local plutôt que d’une adresse IP publique Azure.

## <a name="solution"></a>Solution

Pour résoudre ce problème, utilisez l’itinéraire personnalisé Azure pour acheminer le trafic d’activation vers le serveur Azure KMS.

23.102.135.246 est l’adresse IP du serveur KMS pour le cloud Azure Global. Son nom DNS est kms.core.windows.net. Si vous utilisez d’autres plateformes Azure telles qu’Azure Allemagne, vous devez utiliser l’adresse IP du serveur KMS correspondant. Pour plus d’informations, voir le tableau suivant :

|Plateforme| DNS DE KMS|ADRESSE IP DE KMS|
|------|-------|-------|
|Azure Global|kms.core.windows.net|23.102.135.246|
|Azure Germany|kms.core.cloudapi.de|51.4.143.248|
|Azure US Government|kms.core.usgovcloudapi.net|23.97.0.13|
|Azure China 21Vianet|kms.core.chinacloudapi.cn|42.159.7.249|


Pour ajouter l’itinéraire personnalisé, procédez comme suit :

### <a name="for-resource-manager-vms"></a>Pour les machines virtuelles Resource Manager

 

> [!NOTE] 
> L’activation utilise des adresses IP publiques et sera affectée par une configuration d’équilibreur de charge SKU standard. Examinez attentivement [les connexions sortantes dans Azure](../../load-balancer/load-balancer-outbound-connections.md) pour en savoir plus sur la configuration requise.

1. Ouvrez Azure PowerShell, puis [connectez-vous à votre abonnement Azure](/powershell/azure/authenticate-azureps).
2. Exécutez les commandes suivantes :

    ```powershell
    # First, get the virtual network that hosts the VMs that have activation problems. In this case, we get virtual network ArmVNet-DM in Resource Group ArmVNet-DM:

    $vnet = Get-AzVirtualNetwork -ResourceGroupName "ArmVNet-DM" -Name "ArmVNet-DM"

    # Next, create a route table and specify that traffic bound to the KMS IP (23.102.135.246) will go directly out:

    $RouteTable = New-AzRouteTable -Name "ArmVNet-DM-KmsDirectRoute" -ResourceGroupName "ArmVNet-DM" -Location "centralus"

    Add-AzRouteConfig -Name "DirectRouteToKMS" -AddressPrefix 23.102.135.246/32 -NextHopType Internet -RouteTable $RouteTable

    Set-AzRouteTable -RouteTable $RouteTable

    # Next, attach the route table to the subnet that hosts the VMs

    Set-AzVirtualNetworkSubnetConfig -Name "Subnet01" -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/24" -RouteTable $RouteTable

    Set-AzVirtualNetwork -VirtualNetwork $vnet
    ```
3. Localisez la machine virtuelle qui rencontre des problèmes d’activation. Utilisez [PsPing](/sysinternals/downloads/psping) pour tester si elle peut atteindre le serveur KMS :

    ```console
    psping kms.core.windows.net:1688
    ```

4. Essayez d’activer Windows pour voir si le problème est résolu.

### <a name="for-classic-vms"></a>Pour les machines virtuelles Classic

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

1. Ouvrez Azure PowerShell, puis [connectez-vous à votre abonnement Azure](/powershell/azure/authenticate-azureps).
2. Exécutez les commandes suivantes :

    ```powershell
    # First, create a new route table:
    New-AzureRouteTable -Name "VNet-DM-KmsRouteGroup" -Label "Route table for KMS" -Location "Central US"

    # Next, get the route table that was created:
    $rt = Get-AzureRouteTable -Name "VNet-DM-KmsRouteTable"

    # Next, create a route:
    Set-AzureRoute -RouteTable $rt -RouteName "AzureKMS" -AddressPrefix "23.102.135.246/32" -NextHopType Internet

    # Apply the KMS route table to the subnet that hosts the problem VMs (in this case, we apply it to the subnet that's named Subnet-1):
    Set-AzureSubnetRouteTable -VirtualNetworkName "VNet-DM" -SubnetName "Subnet-1" 
    -RouteTableName "VNet-DM-KmsRouteTable"
    ```

3. Localisez la machine virtuelle qui rencontre des problèmes d’activation. Utilisez [PsPing](/sysinternals/downloads/psping) pour tester si elle peut atteindre le serveur KMS :

    ```console
    psping kms.core.windows.net:1688
    ```

4. Essayez d’activer Windows pour voir si le problème est résolu.

## <a name="next-steps"></a>Étapes suivantes

- [Clés de configuration de Client KMS](/windows-server/get-started/kmsclientkeys)
- [Examiner et sélectionner des méthodes d’activation](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj134256(v=ws.11))
