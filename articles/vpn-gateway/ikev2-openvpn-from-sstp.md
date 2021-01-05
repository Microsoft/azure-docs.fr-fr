---
title: Passer au protocole OpenVPN ou IKEv2 à partir de SSTP | Passerelle VPN Azure
description: Cet article vous aide à comprendre les moyens de surmonter la limite de 128 connexions simultanées de SSTP.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: e2fa265e580bc0e752498284ed50e398b59423fd
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657139"
---
# <a name="transition-to-openvpn-protocol-or-ikev2-from-sstp"></a>Passer au protocole OpenVPN ou à IKEv2 à partir de SSTP

Une connexion par passerelle VPN point à site (P2S) vous permet de créer une connexion sécurisée à votre réseau virtuel à partir d’un ordinateur de client individuel. Une connexion P2S est établie en étant démarrée à partir de l’ordinateur client. Cet article s’applique au modèle de déploiement Resource Manager et présente les moyens de surmonter la limite de 128 connexions simultanées de SSTP en passant au protocole OpenVPN ou IKEv2.

## <a name="what-protocol-does-p2s-use"></a><a name="protocol"></a>Quel protocole est utilisé par le P2S ?

La connexion VPN point à site peut utiliser un des protocoles suivants :

* **Protocole &reg;OpenVPN**, un protocole VPN basé sur SSL/TLS. Une solution VPN SSL peut pénétrer des pare-feu puisque la plupart des pare-feu ouvrent le port de sortie TCP 443 utilisé par le protocole SSL. Vous pouvez utiliser OpenVPN pour vous connecter à partir d’appareils Android, iOS (versions 11.0 et versions ultérieures), Windows, Linux et Mac (OSX 10.13 et versions ultérieures).

* Le **Protocole SSTP (Secure Socket Tunneling Protocol)** est un protocole propriétaire VPN basé sur le protocole SSL. Une solution VPN SSL peut pénétrer des pare-feu puisque la plupart des pare-feu ouvrent le port de sortie TCP 443 utilisé par le protocole SSL. SSTP est pris en charge sur les appareils Windows uniquement. Azure prend en charge toutes les versions de Windows disposant de SSTP (Windows 7 et versions ultérieures). **SSTP prend en charge jusqu’à 128 connexions simultanées seulement, quelle que soit la référence SKU de la passerelle**.

* Un VPN IKEv2 est une solution VPN IPsec basée sur des normes. Un VPN IKEv2 peut être utilisé pour se connecter à partir d’appareils Mac (OSX 10.11 et versions ultérieures).


>[!NOTE]
>IKEv2 et OpenVPN pour P2S sont uniquement disponibles pour le modèle de déploiement Resource Manager. Ils ne sont pas disponibles pour le modèle de déploiement classique. La référence SKU de la passerelle De base ne prend pas en charge les protocoles OpenVPN ou IKEv2. Si vous utilisez la référence SKU De base, vous devez supprimer puis recréer une passerelle de réseau virtuel de référence SKU de production.
>

## <a name="migrating-from-sstp-to-ikev2-or-openvpn"></a>Migration de SSTP vers IKEv2 ou OpenVPN

Il peut arriver que vous vouliez prendre en charge plus de 128 connexions P2S simultanées à une passerelle VPN, mais que vous utilisiez SSTP. Dans ce cas, vous devez passer au protocole IKEv2 ou OpenVPN.

### <a name="option-1---add-ikev2-in-addition-to-sstp-on-the-gateway"></a>Option 1 - Ajouter IKEv2 en plus de SSTP sur la passerelle

Il s’agit de l’option la plus simple. SSTP et IKEv2 peuvent coexister sur la même passerelle et vous offrent un plus grand nombre de connexions simultanées. Vous pouvez simplement activer IKEv2 sur la passerelle existante et retélécharger le client.

L’ajout d’IKEv2 à une passerelle VPN SSTP existante n’affecte pas les clients existants. Vous pouvez configurer ceux-ci de manière à utiliser IKEv2 en petits lots ou simplement configurer les nouveaux clients pour qu’ils utilisent IKEv2. Si un client Windows est configuré pour SSTP et IKEv2, il va essayer de se connecter d’abord avec IKEV2 et,en cas d’échec, il va utiliser SSTP.

**IKEv2 utilise des ports UDP non standard. Vous devez donc vous assurer que ces ports ne sont pas bloqués sur le pare-feu de l’utilisateur. Les ports utilisés sont UDP 500 et 4500.**

Pour ajouter IKEv2 à une passerelle existante, accédez simplement à l’onglet « Configuration de point à site » sous la passerelle de réseau virtuel dans le portail et sélectionnez **IKEv2 et SSTP (SSL)** dans la liste déroulante.

![Capture d’écran montrant la page « Configuration de point à site » avec la liste déroulante « Type de tunnel », et l’option « IKEv2 et SSTP(SSL) » sélectionnée.](./media/ikev2-openvpn-from-sstp/sstptoikev2.png "IKEv2")


### <a name="option-2---remove-sstp-and-enable-openvpn-on-the-gateway"></a>Option 2 - Supprimer SSTP et activer OpenVPN sur la passerelle

SSTP et OpenVPN étant tous deux basés sur le protocole TLS, ils ne peuvent pas coexister sur la même passerelle. Si vous décidez de quitter SSTP pour OpenVPN, vous devez désactiver SSTP et activer OpenVPN sur la passerelle. Cette opération entraîne la perte de connectivité à la passerelle VPN pour les clients existants tant que le nouveau profil n’a pas été configuré sur le client.

Vous pouvez activer OpenVPN en parallèle de IKEv2 si vous le souhaitez. OpenVPN est basé sur TLS et utilise le port TCP 443 standard. Pour passer à OpenVPN, accédez simplement à l’onglet « Configuration de point à site » sous la passerelle de réseau virtuel dans le portail et sélectionnez **OpenVPN (SSL)** ou **IKEv2 et SSTP (SSL)** dans la liste déroulante.

![point à site](./media/ikev2-openvpn-from-sstp/sstptoopenvpn.png "OpenVPN")

Une fois la passerelle configurée, les clients existants ne pourront se connecter que lorsque vous aurez [déployé et configuré les clients OpenVPN](./vpn-gateway-howto-openvpn-clients.md).

Si vous utilisez Windows 10, vous pouvez également utiliser [Azure VPN Client pour Windows](./openvpn-azure-ad-client.md#to-download-the-azure-vpn-client)


## <a name="frequently-asked-questions"></a>Forum aux questions
### <a name="what-are-the-client-configuration-requirements"></a>Quelle est la configuration requise pour les clients ?

>[!NOTE]
>Pour les clients Windows, vous devez disposer des droits d’administrateur sur l’appareil client afin d’initialiser la connexion VPN à partir de l’appareil client vers Azure.
>

Les utilisateurs utilisent les clients VPN natifs sur les appareils Windows et Mac pour P2S. Azure fournit un fichier zip de configuration de client VPN qui contient les paramètres requis par ces clients natifs afin de se connecter à Azure.

* Pour les appareils Windows, la configuration du client VPN comprend un package d’installation que les utilisateurs installent sur leurs appareils.
* Pour les appareils Mac, elle contient un fichier mobileconfig que les utilisateurs installent sur leurs appareils.

Le fichier zip fournit également les valeurs de certains paramètres importants pour Azure que vous pouvez utiliser pour créer votre propre profil pour ces appareils. Ces valeurs incluent notamment l’adresse de passerelle VPN, les types de tunnel configurés, les itinéraires et le certificat racine pour la validation de la passerelle.

>[!NOTE]
>[!INCLUDE [TLS version changes](../../includes/vpn-gateway-tls-change.md)]
>

### <a name="which-gateway-skus-support-p2s-vpn"></a><a name="gwsku"></a>Quelles références SKU de passerelle prennent en charge les VPN P2S ?

[!INCLUDE [aggregate throughput sku](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

* Pour obtenir des recommandations sur les références SKU de passerelle, consultez [À propos des paramètres de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

>[!NOTE]
>La référence SKU de base ne prend pas en charge IKEv2 ou l’authentification RADIUS.
>

### <a name="what-ikeipsec-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="IKE/IPsec policies"></a>Quelles stratégies IKE/IPsec sont configurées sur les passerelles VPN pour P2S ?


**IKEv2**

| **Cipher** | **Intégrité** | **PRF** | **Groupe DH** |
|--|--|--|--|
| GCM_AES256 | GCM_AES256 | SHA384 | GROUP_24 |
| GCM_AES256 | GCM_AES256 | SHA384 | GROUP_14 |
| GCM_AES256 | GCM_AES256 | SHA384 | GROUP_ECP384 |
| GCM_AES256 | GCM_AES256 | SHA384 | GROUP_ECP256 |
| GCM_AES256 | GCM_AES256 | SHA256 | GROUP_24 |
| GCM_AES256 | GCM_AES256 | SHA256 | GROUP_14 |
| GCM_AES256 | GCM_AES256 | SHA256 | GROUP_ECP384 |
| GCM_AES256 | GCM_AES256 | SHA256 | GROUP_ECP256 |
| AES256 | SHA384 | SHA384 | GROUP_24 |
| AES256 | SHA384 | SHA384 | GROUP_14 |
| AES256 | SHA384 | SHA384 | GROUP_ECP384 |
| AES256 | SHA384 | SHA384 | GROUP_ECP256 |
| AES256 | SHA256 | SHA256 | GROUP_24 |
| AES256 | SHA256 | SHA256 | GROUP_14 |
| AES256 | SHA256 | SHA256 | GROUP_ECP384 |
| AES256 | SHA256 | SHA256 | GROUP_ECP256 |
| AES256 | SHA256 | SHA256 | GROUP_2 |

**IPsec**

| **Cipher** | **Intégrité** | **Groupe PFS** |
|--|--|--|
| GCM_AES256 | GCM_AES256 | GROUP_NONE |
| GCM_AES256 | GCM_AES256 | GROUP_24 |
| GCM_AES256 | GCM_AES256 | GROUP_14 |
| GCM_AES256 | GCM_AES256 | GROUP_ECP384 |
| GCM_AES256 | GCM_AES256 | GROUP_ECP256 |
| AES256 | SHA256 | GROUP_NONE |
| AES256 | SHA256 | GROUP_24 |
| AES256 | SHA256 | GROUP_14 |
| AES256 | SHA256 | GROUP_ECP384 |
| AES256 | SHA256 | GROUP_ECP256 |
| AES256 | SHA1 | GROUP_NONE |

### <a name="what-tls-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="TLS policies"></a>Quelles stratégies TLS sont configurées sur les passerelles VPN pour P2S ?
**TLS**

|**Stratégies** |
|---| 
|TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 |
|TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 |
|TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 |
|TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 |
|TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 |
|TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 |
|TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 |
|TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 |
|TLS_RSA_WITH_AES_128_GCM_SHA256 |
|TLS_RSA_WITH_AES_256_GCM_SHA384 |
|TLS_RSA_WITH_AES_128_CBC_SHA256 |
|TLS_RSA_WITH_AES_256_CBC_SHA256 |

### <a name="how-do-i-configure-a-p2s-connection"></a><a name="configure"></a>Comment configurer une connexion P2S ?

Une configuration P2S requiert quelques étapes spécifiques. Les articles suivants détaillent les étapes de la configuration P2S et incluent des liens pour configurer les appareils clients VPN :

* [Configurer une connexion P2S - Authentification RADIUS](point-to-site-how-to-radius-ps.md)

* [Configurer une connexion P2S - Authentification par certificat natif Azure](vpn-gateway-howto-point-to-site-rm-ps.md)

* [Configurer OpenVPN](vpn-gateway-howto-openvpn.md)

## <a name="next-steps"></a>Étapes suivantes

* [Configurer une connexion P2S - Authentification RADIUS](point-to-site-how-to-radius-ps.md)

* [Configurer une connexion P2S - Authentification par certificat natif Azure](vpn-gateway-howto-point-to-site-rm-ps.md)

**« OpenVPN » est une marque d’OpenVPN Inc.**