---
title: Configurer un tunnel utilisateur VPN Always On
titleSuffix: Azure VPN Gateway
description: Cet article explique comment configurer un tunnel utilisateur VPN Always On pour votre passerelle VPN
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 99d21222a62ed6b27a6a1b2a73b704f4cb26457b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89435794"
---
# <a name="configure-an-always-on-vpn-user-tunnel"></a>Configurer un tunnel utilisateur VPN Always On

[!INCLUDE [intro](../../includes/vpn-gateway-vwan-always-on-intro.md)]

## <a name="configure-the-gateway"></a>Configurer la passerelle

 Suivez les instructions de l’article [Configurer une connexion VPN point à site](vpn-gateway-howto-point-to-site-resource-manager-portal.md) pour configurer la passerelle VPN afin d’utiliser IKEv2 et l’authentification basée sur les certificats.

## <a name="configure-a-user-tunnel"></a>Configurer un tunnel utilisateur

[!INCLUDE [user configuration](../../includes/vpn-gateway-vwan-always-on-user.md)]

## <a name="to-remove-a-profile"></a>Pour supprimer un profil

Pour supprimer un profil, effectuez les étapes suivantes :

1. Exécutez la commande suivante :

   ```powershell
   C:\> Remove-VpnConnection UserTest  
   ```

1. Arrêtez la connexion et désactivez la case à cocher **Se connecter automatiquement**.

   ![Nettoyage](./media/vpn-gateway-howto-always-on-user-tunnel/disconnect.jpg)

## <a name="next-steps"></a>Étapes suivantes

Pour résoudre les problèmes de connexion qui peuvent se produire, consultez l’article [Résolution des problèmes : problèmes de connexion point à site Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
