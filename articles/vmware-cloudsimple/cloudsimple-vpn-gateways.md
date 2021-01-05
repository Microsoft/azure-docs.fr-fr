---
title: Azure VMware Solution by CloudSimple - Passerelles VPN
description: Découvrez les passerelles VPN de site à site et de point à site CloudSimple utilisées pour envoyer du trafic chiffré entre une région CloudSimple et d’autres ressources.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 6e013bb96990a8f3a0ef7d3a58529b200919e276
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88140630"
---
# <a name="vpn-gateways-overview"></a>Vue d’ensemble des passerelles VPN

Une passerelle VPN est utilisée pour envoyer le trafic chiffré entre un réseau de la région CloudSimple à un emplacement local ou un ordinateur via l’Internet public.  Une région ne peut posséder qu’une seule passerelle VPN qui prend en charge plusieurs connexions. Lorsque vous créez plusieurs connexions à la même passerelle VPN, tous les tunnels VPN partagent la bande passante de passerelle disponible.

CloudSimple fournit deux types de passerelles VPN :

* Passerelle VPN de site à site
* Passerelle VPN de point à site

## <a name="site-to-site-vpn-gateway"></a>Passerelle VPN de site à site

Une passerelle VPN de site à site est utilisée pour envoyer le trafic chiffré entre un réseau de la région CloudSimple et un centre de données local. Utilisez cette connexion pour définir la plage de sous-réseaux/CIDR, pour le trafic entre votre réseau local et le réseau de la région CloudSimple.

La passerelle VPN vous permet d’utiliser les services en local sur votre cloud privé et des services sur votre cloud privé, à partir du réseau local.  CloudSimple fournit un serveur VPN basé sur la stratégie pour établir une connexion depuis votre réseau local.

Les cas d’utilisation de VPN de site à site sont les suivants :

* Accessibilité de votre cloud privé vCenter à partir de toute station de travail dans votre réseau local.
* Utilisation de votre Active Directory local comme source d’identité vCenter.
* Transfert pratique de modèles de machines virtuelles, ISO et autres fichiers depuis vos ressources locales vers votre cloud privé vCenter.
* Accessibilité des charges de travail en cours d’exécution sur votre cloud privé à partir de votre réseau local.

![Topologie d’une connexion VPN de site à site](media/cloudsimple-site-to-site-vpn-connection.png)

### <a name="cryptographic-parameters"></a>Paramètres de chiffrement

Une connexion VPN de site à site utilise les paramètres de chiffrement par défaut suivants pour établir une connexion sécurisée.  Lorsque vous créez une connexion à partir de votre appareil VPN local, utilisez un des paramètres suivants pris en charge par votre passerelle VPN.

#### <a name="phase-1-proposals"></a>Propositions de phase 1

| Paramètre | Proposition 1 | Proposition 2 | Proposition 3 |
|-----------|------------|------------|------------|
| Version IKE | IKEv1 | IKEv1 | IKEv1 |
| Chiffrement | AES 128 | AES 256 | AES 256 |
| Algorithme de hachage| SHA 256 | SHA 256 | SHA 1 |
| Groupe Diffie Hellman (groupe DH) | 2 | 2 | 2 |
| Durée de vie | 28 800 secondes | 28 800 secondes | 28 800 secondes |
| Taille des données | 4 Go | 4 Go | 4 Go |

#### <a name="phase-2-proposals"></a>Propositions de phase 2

| Paramètre | Proposition 1 | Proposition 2 | Proposition 3 |
|-----------|------------|------------|------------|
| Chiffrement | AES 128 | AES 256 | AES 256 |
| Algorithme de hachage| SHA 256 | SHA 256 | SHA 1 |
| Groupe PFS (Perfect Forward Secrecy) | None | None | None |
| Durée de vie | 1 800 secondes | 1 800 secondes | 1 800 secondes |
| Taille des données | 4 Go | 4 Go | 4 Go |


> [!IMPORTANT]
> Définissez le clamping du MSS TCP sur 1200 sur votre périphérique VPN. Dans le cas où vos périphériques VPN ne prendraient pas en charge le réglage de la taille maximale de segment, vous pouvez à la place définir l’unité de transmission maximale dans l’interface de tunnel sur 1240 octets.

## <a name="point-to-site-vpn-gateway"></a>Passerelle VPN de point à site

Une passerelle VPN de point à site est utilisée pour envoyer le trafic chiffré entre un réseau de la région CloudSimple et l’ordinateur d’un client.  Il s’agit du moyen le plus facile d’accéder à votre réseau cloud privé, notamment à votre cloud privé vCenter et vos machines virtuelles de charge de travail.  Utilisez la connectivité VPN de point à site si vous vous connectez au cloud privé à distance.

## <a name="next-steps"></a>Étapes suivantes

* [Configurer la passerelle VPN](vpn-gateway.md)
