---
title: 'Configuration d’OpenVPN sur la passerelle VPN Azure : PowerShell'
description: Découvrez comment utiliser PowerShell pour activer le protocole OpenVPN sur la passerelle VPN Azure pour un environnement de point à site.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 57bc53c28866b3f16e742c27b6a1600bfe6f44ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89418821"
---
# <a name="configure-openvpn-for-azure-point-to-site-vpn-gateway"></a>Configurer OpenVPN pour la passerelle VPN Azure de point à site

Cet article vous aide à configurer le **protocole OpenVPN®** sur la passerelle VPN Azure. Nous partons du principe que vous disposez déjà d’un environnement de point à site fonctionnel. Si ce n’est pas le cas, suivez les instructions de l’étape 1 pour créer un réseau VPN de point à site.



## <a name="1-create-a-point-to-site-vpn"></a><a name="vnet"></a>1. Créer un réseau VPN de point à site

Si vous ne disposez pas d’un environnement de point à site opérationnel, suivez les instructions de création. Consultez l’article [Créer un VPN de point à site](vpn-gateway-howto-point-to-site-resource-manager-portal.md) pour créer et configurer une passerelle VPN de point à site avec l’authentification par certificat native Azure. 

> [!IMPORTANT]
> La référence De base n’est pas prise en charge pour OpenVPN.

## <a name="2-enable-openvpn-on-the-gateway"></a><a name="enable"></a>2. Activer OpenVPN sur la passerelle

Activez OpenVPN sur votre passerelle. Assurez-vous que la passerelle est configurée de point à site (IKEv2 ou SSTP) avant d’exécuter les commandes suivantes :

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -ResourceGroupName $rgname -name $name
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -VpnClientProtocol OpenVPN
```

## <a name="next-steps"></a>Étapes suivantes

Pour configurer les clients pour OpenVPN, consultez [Configurer les clients OpenVPN](vpn-gateway-howto-openvpn-clients.md).

**« OpenVPN » est une marque d’OpenVPN Inc.**
