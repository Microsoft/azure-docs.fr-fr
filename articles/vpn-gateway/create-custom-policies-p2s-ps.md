---
title: 'Créer et définir des stratégies IPsec personnalisées pour une configuration point à site : PowerShell'
titleSuffix: Azure VPN Gateway
description: Cet article vous aide à créer et à définir des stratégies IPsec personnalisées pour les configurations de passerelle VPN P2S
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/09/2020
ms.author: cherylmc
ms.openlocfilehash: a77a870304e20c179e22b4a4ffe404315894d8a0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91743690"
---
# <a name="create-and-set-custom-ipsec-policies-for-point-to-site-preview"></a>Créer et définir des stratégies IPsec personnalisées pour une configuration point à site (préversion)

Si votre environnement nécessite une stratégie IPsec personnalisée pour le chiffrement, vous pouvez facilement configurer un objet de stratégie avec les paramètres requis. Cet article vous permet de créer un objet de stratégie personnalisée, puis de le définir en utilisant PowerShell.

## <a name="before-you-begin"></a>Avant de commencer

### <a name="prerequisites"></a>Prérequis

Vérifiez que votre environnement répond aux prérequis suivants :

* Vous avez un réseau VPN point à site déjà configuré. Si ce n’est pas le cas, configurez-en un en utilisant les étapes de l’article **Créer un VPN point à site** avec [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md) ou avec le [portail Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell.md)]

## <a name="1-set-variables"></a><a name="signin"></a>1. Définition des variables

Déclarez les variables que vous souhaitez utiliser. Utilisez l’exemple ci-dessous en remplaçant les valeurs par les vôtres là où c’est nécessaire. Si vous fermez votre session PowerShell/Cloud Shell au cours de l’exercice, copiez et collez à nouveau les valeurs pour redéclarer les variables.

  ```azurepowershell-interactive
  $RG = "TestRG"
  $GWName = "VNet1GW"
  ```

## <a name="2-create-policy-object"></a><a name="create"></a>2. Créer un objet de stratégie

Créez un objet de stratégie IPsec personnalisée. Vous pouvez ajuster les valeurs pour les faire correspondre aux critères nécessaires.

```azurepowershell-interactive
$vpnclientipsecpolicy = New-AzVpnClientIpsecPolicy -IpsecEncryption AES256 -IpsecIntegrity SHA256 -SALifeTime 86471 -SADataSize 429496 -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup2 -PfsGroup PFS2
```

## <a name="3-update-gateway-and-set-policy"></a><a name="update"></a>3. Mettre à jour la passerelle et définir la stratégie

Dans cette étape, mettez à jour votre passerelle VPN P2S existante et définissez la stratégie IPsec.

```azurepowershell-interactive
$gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gateway -VpnClientIpsecPolicy $vpnclientipsecpolicy
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les configurations P2S, consultez l’article [À propos des VPN point à site](point-to-site-about.md).
