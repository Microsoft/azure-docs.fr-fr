---
title: Configurer des clients OpenVPN pour Azure Virtual WAN
description: Étapes de configuration des clients OpenVPN pour Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: 05502b344b1224dff5d12f95b96b05baace98970
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94490999"
---
# <a name="configure-an-openvpn-client-for-azure-virtual-wan"></a>Configurer un client OpenVPN pour Azure Virtual WAN

Cet article vous permet de configurer des clients pour l’utilisation du **protocole&reg; OpenVPN**. Vous pouvez également utiliser Azure VPN Client pour Windows 10 pour vous connecter via le protocole OpenVPN. Vous trouverez plus d’instructions [ici](openvpn-azure-ad-client.md)

## <a name="before-you-begin"></a>Avant de commencer

Créez une configuration VPN (point à site) utilisateur. Veillez à sélectionner « OpenVPN » comme type de tunnel. Pour les étapes, consultez [Créer une configuration P2S pour Azure Virtual WAN](virtual-wan-point-to-site-portal.md#p2sconfig).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le VPN utilisateur (point à site), consultez [Création de connexions VPN utilisateur](virtual-wan-point-to-site-portal.md).

**« OpenVPN » est une marque d’OpenVPN Inc.**
