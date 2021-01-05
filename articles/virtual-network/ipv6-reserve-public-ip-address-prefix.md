---
title: Réserver des plages d’adresses et des adresses IPv6 publiques dans un réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Découvrez comment réserver des plages d’adresses et des adresses IPv6 publiques dans un réseau virtuel Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: ac79e1eb5c4f7448dc17804cd8aac3cba582497e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88509941"
---
# <a name="reserve-public-ipv6-address-prefix"></a>Réserver un préfixe d’adresses IPv6 publiques
Le protocole IPv6 pour réseau virtuel Azure vous permet d’héberger des applications dans Azure avec une connectivité IPv6 et IPv4 tant au sein d’un réseau virtuel que vers et depuis Internet. En plus de réserver des adresses IPv6 individuelles, vous pouvez réserver des plages contiguës d’adresses IPv6 Azure (appelées préfixes IP) pour votre utilisation. Cet article explique comment créer des plages d’adresses et des adresses IP publiques IPv6 avec Azure PowerShell et l’interface CLI.


## <a name="create-a-single-reserved-ipv6-public-ip"></a>Créer une adresse IP publique IPv6 réservée unique

### <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell

Vous pouvez créer une adresse IP publique IPv6 unique (statique) réservée à l’aide d’Azure PowerShell avec [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) comme suit :

```azurepowershell
 $myOwnIPv6Address = New-AzPublicIpAddress `
 -name PIPv6_WestUS `
 -ResourceGroup MyRG `
 -Location "West US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6
 ```

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

 Vous pouvez créer une adresse IP publique IPv6 unique (statique) réservée à l’aide d’Azure CLI avec [az network public-ip create](/cli/azure/network/public-ip) comme suit :

```azurecli
 az network public-ip create \
 --name dsPublicIP_v6 \
 --resource-group UpgradeInPlace_CLI_RG1 \
 --location WestUS \
 --sku Standard  \
 --allocation-method static  \
 --version IPv6
```

## <a name="create-a-reserved-ipv6-prefix-range"></a>Créer un préfixe (plage) IPv6 réservé

Pour réserver un préfixe IPv6, ajoutez la famille d’adresses IP IPv6 à la même commande que celle utilisée pour créer des préfixes IPv4. Les commandes suivantes créent un préfixe de taille /125 (huit adresses IPv6).

### <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell

Vous pouvez créer une adresse IPv6 publique à l’aide d’Azure PowerShell avec [az network public-ip create](/powershell/module/az.network/new-azpublicipprefix) comme suit :
```azurepowershell
 $myOwnIPv6Prefix = New-AzPublicIpPrefix `
 -name IPv6PrefixWestUS `
 -ResourceGroupName MyRG `
 -Location "West US" `
 -Sku Standard `
 -IpAddressVersion IPv6 `
 -PrefixLength 125
```

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

Vous pouvez créer une adresse IPv6 publique à l’aide d’Azure CLI comme suit :

```azurecli
az network public-ip prefix create \
--name IPv6PrefixWestUS \
--resource-group MyRG \
--location WestUS \
--version IPv6 \
--length 125
```

## <a name="allocate-a-public-ip-address-from-a-reserved-ipv6-prefix"></a>Allouer une adresse IP publique à partir d’un préfixe IPv6 réservé

### <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell

 Pour créer une adresse IP publique IPv6 statique à partir d’un préfixe réservé, ajoutez l’argument `-PublicIpPrefix` lors de la création de l’adresse IP publique à l’aide d’Azure PowerShell. L’exemple suivant part du principe qu’un préfixe a été créé et stocké dans une variable PowerShell nommée *$myOwnIPv6Prefix*.

```azurepowershell
 $MyIPv6PublicIPFromMyReservedPrefix = New-AzPublicIpAddress \
 -name PIPv6_fromPrefix `
 -ResourceGroup DsStdLb04 `
 -Location "West Central US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6 `
 -PublicIpPrefix $myOwnIPv6Prefix
```

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

L’exemple suivant part du principe qu’un préfixe a été créé et stocké dans une variable CLI nommée *IPv6PrefixWestUS*.

```azurecli
az network public-ip create \
--name dsPublicIP_v6 \
--resource-group UpgradeInPlace_CLI_RG1 \
--location WestUS \
--sku Standard \
--allocation-method static \
--version IPv6 \
--public-ip-prefix  IPv6PrefixWestUS
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [préfixe d’adresses IPv6](ipv6-public-ip-address-prefix.md).
- En savoir plus sur les [adresses IPv6](ipv6-overview.md).
