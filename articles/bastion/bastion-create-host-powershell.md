---
title: Créer un hôte Bastion à l’aide d’Azure PowerShell | Microsoft Docs
description: Dans cet article, découvrez comment créer un hôte Azure Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 10/14/2020
ms.author: cherylmc
ms.openlocfilehash: ba2716613a0e950cbae5c65add410ac8a8b38955
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077723"
---
# <a name="create-an-azure-bastion-host-using-azure-powershell"></a>Créer un hôte Azure Bastion à l’aide d’Azure PowerShell

Dans cet article, découvrez comment créer un hôte Azure Bastion à l’aide de PowerShell. Une fois que vous avez provisionné le service Azure Bastion dans votre réseau virtuel, l’expérience fluide RDP/SSH est disponible pour toutes les machines virtuelles du même réseau virtuel. Le déploiement Azure Bastion est effectué par réseau virtuel et non par abonnement/compte ou machine virtuelle.

Vous pouvez éventuellement créer un hôte Azure Bastion à l’aide du [portail Azure](./tutorial-create-host-portal.md).

## <a name="prerequisites"></a>Prérequis

Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial).

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Créer un hôte Bastion

Cette section vous permet de créer une ressource Azure Bastion à l’aide d’Azure PowerShell.

1. Créez un réseau virtuel et un sous-réseau Azure Bastion associé. Vous devez créer un sous-réseau Azure Bastion à l’aide de la valeur de nom **AzureBastionSubnet**. Cette valeur permet à Azure de savoir dans quel sous-réseau déployer les ressources Bastion. Ceci est différent d’un sous-réseau de passerelle. Vous devez utiliser un sous-réseau d’au moins /27 ou plus grand (/27, /26, etc.). Créez le sous-réseau **AzureBastionSubnet** sans tables de routage ou délégations. Si vous utilisez des groupes de sécurité réseau sur le sous-réseau **AzureBastionSubnet**, reportez-vous à l’article [Utiliser des groupes de sécurité réseau](bastion-nsg.md).

   ```azurepowershell-interactive
   $subnetName = "AzureBastionSubnet"
   $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
   $vnet = New-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myBastionRG" -Location "westeurope" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
   ```

2. Créez une adresse IP publique pour Azure Bastion. Adresse IP publique de la ressource Bastion où RDP/SSH est accessible (sur le port 443). L’adresse IP publique doit être située dans la même région que la ressource Bastion que vous créez.

   ```azurepowershell-interactive
   $publicip = New-AzPublicIpAddress -ResourceGroupName "myBastionRG" -name "myPublicIP" -location "westeurope" -AllocationMethod Static -Sku Standard
   ```

3. Créez une ressource Azure Bastion dans le sous-réseau Azure Bastion de votre réseau virtuel. Il faut environ 5 minutes pour que la ressource Bastion soit créée et déployée.

   ```azurepowershell-interactive
   $bastion = New-AzBastion -ResourceGroupName "myBastionRG" -Name "myBastion" -PublicIpAddress $publicip -VirtualNetwork $vnet
   ```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations, lisez les [questions fréquentes (FAQ) sur Bastion](bastion-faq.md).
* Pour utiliser des groupes de sécurité réseau avec le sous-réseau Azure Bastion, consultez [Utiliser des groupes de sécurité réseau](bastion-nsg.md).
