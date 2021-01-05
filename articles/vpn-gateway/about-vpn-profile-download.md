---
title: 'Passerelle VPN Azure : À propos des profils clients VPN P2S'
description: Cet article vous permet de rechercher les informations dont vous avez besoin pour un profil client VPN.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 6a09767a7992a5f902adea6f99e937f3fc6fa7fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90985935"
---
# <a name="about-p2s-vpn-client-profiles"></a>À propos des profils clients VPN P2S

Le fichier de profil téléchargé contient les informations nécessaires à la configuration d’une connexion VPN. Cet article vous aide à obtenir et à comprendre les informations nécessaires pour un profil client VPN.

[!INCLUDE [client profiles](../../includes/vpn-gateway-vwan-vpn-profile-download.md)]

* Le dossier **OpenVPN** contient le profil *ovpn* qui doit être modifié pour inclure la clé et le certificat. Pour plus d’informations, consultez [Configurer des clients OpenVPN pour la passerelle VPN Azure](vpn-gateway-howto-openvpn-clients.md#windows). Si vous sélectionnez l’authentification Azure AD sur la passerelle VPN, ce dossier sera absent du fichier zip. Au lieu de cela, accédez au dossier AzureVPN et recherchez azurevpnconfig.xml.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la fonctionnalité point à site, consultez l’article [À propos de la fonctionnalité point à site](point-to-site-about.md).
