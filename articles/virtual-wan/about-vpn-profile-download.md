---
title: 'Azure Virtual WAN : Profils clients VPN utilisateurs'
description: Cela vous aide à utiliser le fichier de profil client.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: 116ec518f7788e620fc6bd177e535c8f6af99d10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91267769"
---
# <a name="working-with-user-vpn-client-profiles"></a>Utilisation de profils clients VPN utilisateurs

Le fichier de profil téléchargé contient les informations nécessaires à la configuration d’une connexion VPN. Cet article permet d’obtenir et de comprendre les informations nécessaires pour un profil client VPN utilisateur.

[!INCLUDE [client profiles](../../includes/vpn-gateway-vwan-vpn-profile-download.md)]

* Le dossier **OpenVPN** contient le profil *ovpn* qui doit être modifié pour inclure la clé et le certificat. Pour plus d’informations, consultez [Configuration de clients OpenVPN](../virtual-wan/howto-openvpn-clients.md#windows).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le VPN utilisateur Virtual WAN, consultez [Création d’une connexion VPN utilisateur](virtual-wan-point-to-site-portal.md).
