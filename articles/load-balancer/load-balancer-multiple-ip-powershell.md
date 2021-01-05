---
title: Équilibrage de charge sur plusieurs configurations IP - Azure CLI
titleSuffix: Azure Load Balancer
description: Dans cet article, vous en apprendrez plus sur l’équilibrage de charge sur des configurations IP principales et secondaires à l’aide d’Azure CLI.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: a78bfc2b4f0c372c915647c0afa40263079af8e5
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746059"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-powershell"></a>Équilibrage de charge sur plusieurs configurations IP avec PowerShell

> [!div class="op_single_selector"]
> * [Portail](load-balancer-multiple-ip.md)
> * [INTERFACE DE LIGNE DE COMMANDE](load-balancer-multiple-ip-cli.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)


Cet article décrit comment utiliser Azure Load Balancer avec plusieurs adresses IP sur une interface réseau secondaire (carte réseau). Avec ce scénario, nous disposons de deux machines virtuelles exécutant Windows, chacune avec une carte réseau principale et secondaire. Chacune des cartes réseau secondaires dispose de deux configurations IP. Chaque machine virtuelle héberge les deux sites web contoso.com et fabrikam.com. Chaque site web est lié à l’une des configurations IP sur la carte réseau secondaire. Nous utilisons Azure Load Balancer pour exposer deux adresses IP frontales, une par site web, afin de distribuer le trafic à la configuration IP correspondante pour le site web. Ce scénario utilise le même numéro de port sur les deux serveurs frontaux, ainsi que les deux adresses IP de pool principal.

![Image du scénario LB](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>Étapes pour équilibrer la charge sur plusieurs configurations IP

Pour accomplir le scénario décrit dans cet article, suivez les étapes ci-dessous :

1. Installez Azure PowerShell. Pour plus d’informations sur l’installation de la version la plus récente d’Azure PowerShell, la sélection de votre abonnement et la connexion à votre compte, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).
2. Créez un groupe de ressources à l’aide des paramètres suivants :

    ```powershell
    $location = "westcentralus".
    $myResourceGroup = "contosofabrikam"
    ```

    Pour plus d’informations, voir l’Étape 2 de [Créer un groupe de ressources](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json).

3. [Créez un groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md?toc=%2fazure%2fload-balancer%2ftoc.json) pour contenir vos machines virtuelles. Pour ce scénario, utilisez la commande suivante :

    ```powershell
    New-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "West Central US"
    ```

4. Suivez les instructions des étapes 3 à 5 de l’article [Créer une machine virtuelle Windows](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json) pour préparer la création d’une machine virtuelle avec une seule carte réseau. Exécutez l’étape 6.1, puis procédez comme suit au lieu de suivre l’étape 6.2 :

    ```powershell
    $availset = Get-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
    New-AzVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id
    ```

    Accomplissez ensuite les étapes 6.3 à 6.8 de l’article [Créer une machine virtuelle Windows](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json).

5. Ajoutez une deuxième configuration IP à chacune des machines virtuelles. Suivez les instructions de l’article [Affecter plusieurs adresses IP aux machines virtuelles](../virtual-network/virtual-network-multiple-ip-addresses-powershell.md#add). Utilisez les paramètres de configuration suivants :

    ```powershell
    $NicName = "VM1-NIC2"
    $RgName = "contosofabrikam"
    $NicLocation = "West Central US"
    $IPConfigName4 = "VM1-ipconfig2"
    $Subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet
    ```

    Pour les besoins de ce didacticiel, il est inutile d’associer les configurations IP secondaires à des adresses IP publiques. Modifiez la commande pour supprimer la partie d’association d’adresse IP publique.

6. Exécutez à nouveau les étapes 4 à 6 de cet article pour la machine virtuelle VM2. Veillez à remplacer le nom de la machine virtuelle par VM2 lors de cette opération. Notez que vous n’avez pas besoin créer un réseau virtuel pour la deuxième machine virtuelle. Vous pouvez ou non créer un sous-réseau, selon votre cas d’utilisation.

7. Créez deux adresses IP publiques et stockez-les dans les variables appropriées, comme indiqué :

    ```powershell
    $publicIP1 = New-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel contoso
    $publicIP2 = New-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel fabrikam

    $publicIP1 = Get-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
    $publicIP2 = Get-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam
    ```

8. Créez deux configurations IP frontales :

    ```powershell
    $frontendIP1 = New-AzLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
    $frontendIP2 = New-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2
    ```

9. Créer vos pools d’adresses principaux, une sonde et vos règles d’équilibrage de charge :

    ```powershell
    $beaddresspool1 = New-AzLoadBalancerBackendAddressPoolConfig -Name contosopool
    $beaddresspool2 = New-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool

    $healthProbe = New-AzLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

    $lbrule1 = New-AzLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    $lbrule2 = New-AzLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    ```

10. Une fois les ressources créées, créez votre équilibreur de charge :

    ```powershell
    $mylb = New-AzLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'West Central US' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
    ```

11. Ajoutez le deuxième pool d’adresses principal et la configuration IP frontale à l’équilibrage de charge que vous venez de créer :

    ```powershell
    $mylb = Get-AzLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzLoadBalancer

    $mylb | Add-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzLoadBalancer
    
    Add-AzLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzLoadBalancer
    ```

12. Les commandes ci-dessous obtiennent les cartes réseau, puis ajoutent les deux configurations IP de chaque carte réseau secondaire au pool d’adresses principales de l’équilibrage de charge :

    ```powershell
    $nic1 = Get-AzNetworkInterface -Name "VM1-NIC2" -ResourceGroupName "MyResourcegroup";
    $nic2 = Get-AzNetworkInterface -Name "VM2-NIC2" -ResourceGroupName "MyResourcegroup";

    $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
    $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

    $mylb = $mylb | Set-AzLoadBalancer

    $nic1 | Set-AzNetworkInterface
    $nic2 | Set-AzNetworkInterface
    ```

13. Enfin, vous devez configurer les enregistrements de ressource DNS pour qu’ils pointent sur l’adresse IP frontale respective de l’équilibrage de charge. Vous pouvez héberger vos domaines dans Azure DNS. Pour plus d’informations sur l’utilisation d’Azure DNS avec un équilibrage de charge, voir [Utiliser Azure DNS avec d’autres services Azure](../dns/dns-for-azure-services.md).

## <a name="next-steps"></a>Étapes suivantes
- Pour en savoir plus sur la combinaison de services d’équilibrage de charge dans Azure, consultez [Utilisation des services d’équilibrage de charge dans Azure](../traffic-manager/traffic-manager-load-balancing-azure.md).
- Pour savoir comment gérer et dépanner l’équilibreur de charge à l’aide de différents types de journaux d’activité dans Azure, consultez [Journaux d’activité Azure Monitor pour Azure Load Balancer](../load-balancer/load-balancer-monitor-log.md).
