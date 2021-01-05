---
title: 'Travail à distance avec P2S : Passerelle VPN Azure'
description: Cette page décrit comment vous pouvez tirer parti de la passerelle VPN pour travailler à distance dans le contexte de la pandémie du COVID-19.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: 028250e25f4d67628c08970e25b4621e78526b6a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91440836"
---
# <a name="remote-work-using-azure-vpn-gateway-point-to-site"></a>Travail à distance à l’aide d’une connexion point à site par passerelle VPN Azure

>[!NOTE]
>Cet article décrit comment vous pouvez tirer parti d’une Passerelle VPN Azure, d’Azure, du réseau Microsoft et de l’écosystème de partenaires Azure pour travailler à distance et atténuer les problèmes de réseau auxquels vous faites face dans le contexte de la crise de COVID-19.
>

Cet article décrit les options disponibles pour les organisations afin de configurer l’accès à distance de leurs utilisateurs ou de compléter leurs solutions existantes avec une capacité supplémentaire pendant l’épidémie de COVID-19.

La solution point à site Azure est basée sur le cloud et peut être provisionnée rapidement pour répondre à la demande croissante des utilisateurs désireux de travailler à partir de leur domicile. Elle peut monter en puissance facilement et se désactiver tout aussi facilement et rapidement lorsque la capacité augmentée n’est plus nécessaire.

## <a name="about-point-to-site-vpn"></a><a name="p2s"></a>À propos du VPN de point à site

Une connexion par passerelle VPN point à site (P2S) vous permet de créer une connexion sécurisée à votre réseau virtuel à partir d’un ordinateur de client individuel. Une connexion P2S est établie en étant démarrée à partir de l’ordinateur client. Cette solution est utile pour les télétravailleurs désireux de se connecter à des réseaux virtuels Azure ou à des centres de données locaux à partir d’un emplacement distant tel que leur domicile ou une salle de conférence. Cet article explique comment permettre aux utilisateurs de travailler à distance dans diverses circonstances.

Le tableau ci-dessous présente les systèmes d’exploitation clients et les options d’authentification disponibles. Il serait utile de sélectionner la méthode d’authentification en fonction du système d’exploitation client déjà utilisé. Par exemple, sélectionnez OpenVPN avec l’authentification basée sur les certificats si vous avez plusieurs systèmes d’exploitation clients qui doivent se connecter. Notez également qu’un VPN point à site est pris en charge uniquement sur des passerelles VPN basées sur un itinéraire.

![Capture d'écran représentant les systèmes d'exploitation clients et les options d'authentification disponibles.](./media/working-remotely-support/ostable.png "Système d''exploitation")

## <a name="scenario-1---users-need-access-to-resources-in-azure-only"></a><a name="scenario1"></a>Scénario 1 : les utilisateurs doivent accéder à des ressources seulement dans Azure

Dans ce scénario, les utilisateurs distants doivent accéder uniquement à des ressources qui se trouvent dans Azure.

![Diagramme illustrant un scénario point à site pour des utilisateurs qui ont uniquement besoin d'accéder aux ressources disponibles dans Azure.](./media/working-remotely-support/scenario1.png "Scénario 1")

À un niveau élevé, les étapes suivantes sont nécessaires pour permettre aux utilisateurs de se connecter aux ressources Azure en toute sécurité :

1. Créer une passerelle de réseau virtuel (s’il n’en existe pas).
2. Configurer un VPN point à site sur la passerelle.

   * Pour l’authentification par certificat, suivez [ce lien](vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw).
   * Pour OpenVPN, suivez [ce lien](vpn-gateway-howto-openvpn.md).
   * Pour l’authentification Azure AD, suivez [ce lien](openvpn-azure-ad-tenant.md).
   * Pour résoudre les problèmes de connexion de point à site, suivez [ce lien](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
3. Télécharger et distribuer la configuration de client VPN.
4. Distribuer les certificats (si l’authentification par certificat est sélectionnée) aux clients.
5. Se connecter au VPN Azure.

## <a name="scenario-2---users-need-access-to-resources-in-azure-andor-on-prem-resources"></a><a name="scenario2"></a>Scénario 2 : les utilisateurs doivent accéder à des ressources dans Azure et/ou localement

Dans ce scénario, les utilisateurs distants doivent accéder à des ressources qui se trouvent dans Azure et dans des centres de données locaux.

![Diagramme illustrant un scénario point à site pour des utilisateurs qui ont besoin d'accéder aux ressources disponibles dans Azure.](./media/working-remotely-support/scenario2.png "Scénario 2")

À un niveau élevé, les étapes suivantes sont nécessaires pour permettre aux utilisateurs de se connecter aux ressources Azure en toute sécurité :

1. Créer une passerelle de réseau virtuel (s’il n’en existe pas).
2. Configurer un VPN point à site sur la passerelle (voir [Scénario 1](#scenario1)).
3. Configurer un tunnel site à site sur la passerelle de réseau virtuel Azure avec BGP activé.
4. Configurer l’appareil local pour se connecter à la passerelle de réseau virtuel Azure.
5. Télécharger le profil point à site à partir du portail Azure et le distribuer aux clients

Pour découvrir comment configurer un tunnel VPN site à site, suivez [ce lien](vpn-gateway-howto-site-to-site-resource-manager-portal.md).

## <a name="faq-for-native-azure-certificate-authentication"></a><a name="faqcert"></a>Forum Aux Questions (FAQ) sur l’authentification par certificat Azure native

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="faq-for-radius-authentication"></a><a name="faqradius"></a>Forum Aux Questions (FAQ) sur l’authentification RADIUS

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Configurer une connexion P2S - Authentification Azure AD](openvpn-azure-ad-tenant.md)

* [Configurer une connexion P2S - Authentification RADIUS](point-to-site-how-to-radius-ps.md)

* [Configurer une connexion P2S - Authentification par certificat natif Azure](vpn-gateway-howto-point-to-site-rm-ps.md)

**« OpenVPN » est une marque d’OpenVPN Inc.**