---
title: 'Passerelle VPN Azure : Exigences de chiffrement'
description: En savoir plus sur la configuration des passerelles VPN Azure pour satisfaire les exigences de chiffrement pour les tunnels VPN S2S entre différents locaux et les connexions de réseau virtuel à réseau virtuel d’Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: article
ms.date: 12/02/2020
ms.author: yushwang
ms.openlocfilehash: 47d14c5ee7f6c4816bf15351e9cb28a2aaa72b4c
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546843"
---
# <a name="about-cryptographic-requirements-and-azure-vpn-gateways"></a>À propos des exigences de chiffrement et des passerelles VPN Azure

Cet article explique comment configurer des passerelles VPN Azure pour satisfaire vos exigences de chiffrement pour les tunnels VPN S2S entre différents locaux et les connexions de réseau virtuel à réseau virtuel dans Azure.

## <a name="about-ikev1-and-ikev2-for-azure-vpn-connections"></a>À propos des protocoles IKEv1 et IKEv2 pour les connexions Azure VPN

Jusqu’à présent, nous autorisions les connexions IKEv1 uniquement pour les références SKU De base et nous autorisions les connexions IKEv2 pour toutes les autres références SKU de passerelle VPN. À cause de la limitation des références SKU De base à une seule connexion autorisée et à d’autres limitations comme les performances, les clients qui utilisaient des appareils d’ancienne génération prenant uniquement en charge les protocoles IKEv1 avaient une expérience limitée. Pour améliorer l’expérience des clients utilisant les protocoles IKEv1, nous autorisons maintenant les connexions IKEv1 pour toutes les références SKU de passerelle VPN, à l’exception du SKU de base. Pour plus d’informations, consultez [Références SKU de passerelle VPN](./vpn-gateway-about-vpn-gateway-settings.md#gwsku).

![Connexions IKEv1 et IKEv2 pour les passerelles VPN Azure](./media/vpn-gateway-about-compliance-crypto/ikev1-ikev2-connections.png)

Lorsque des connexions IKEv1 et IKEv2 sont utilisées pour la même passerelle VPN, le transit entre ces deux connexions est activé automatiquement.

## <a name="about-ipsec-and-ike-policy-parameters-for-azure-vpn-gateways"></a>À propos des paramètres de stratégie IPsec et IKE pour les passerelles VPN Azure

La norme de protocole IPsec et IKE standard prend en charge un vaste éventail d’algorithmes de chiffrement dans différentes combinaisons. Si vous n’exigez pas une combinaison spécifique de paramètres et d’algorithmes de chiffrement, les passerelles VPN Azure utilisent un ensemble de propositions par défaut. Les jeux de stratégies par défaut ont été choisis pour optimiser l’interopérabilité avec un large éventail d’appareils VPN tiers dans des configurations par défaut. Par conséquent, les stratégies et le nombre de propositions ne peuvent pas couvrir toutes les combinaisons d’algorithmes de chiffrement disponibles et de forces de clé disponibles.

### <a name="default-policy"></a>Stratégie par défaut

La stratégie par défaut définie pour la passerelle VPN Azure est listée dans l’article : [À propos des périphériques VPN et des paramètres IPsec/IKE pour les connexions de passerelle VPN site à site](vpn-gateway-about-vpn-devices.md).

## <a name="cryptographic-requirements"></a>Exigences de chiffrement

Pour les communications qui exigent des paramètres ou des algorithmes de chiffrement spécifiques, généralement en raison d’exigences de conformité ou de sécurité, vous pouvez désormais configurer leurs passerelles VPN Azure pour utiliser une stratégie IPsec/IKE personnalisée avec des algorithmes de chiffrement et des puissances de clé spécifiques, à la place des ensembles de stratégies Azure par défaut.

Par exemple, les stratégies du mode principal IKEv2 pour les passerelles VPN Azure utilisent uniquement Diffie-Hellman du groupe 2 (1024 bits), alors que vous devez peut-être spécifier des groupes plus puissants à utiliser dans IKE, tels que le groupe 14 (2048 bits), le groupe 24 (groupe MODP de 2048 bits) ou des groupes ECP (groupes à courbe elliptique) 256 ou 384 bits (groupe 19 et groupe 20, respectivement). Des exigences similaires s’appliquent également aux stratégies de mode rapide IPsec.

## <a name="custom-ipsecike-policy-with-azure-vpn-gateways"></a>Stratégie IPsec/IKE personnalisée avec des passerelles VPN Azure

Les passerelles VPN Azure prennent désormais en charge la stratégie IPsec/IKE personnalisée par connexion. Vous pouvez choisir une combinaison spécifique d’algorithmes de chiffrement pour IPsec et IKE avec la force de clé souhaitée pour une connexion S2S ou de réseau virtuel à réseau virtuel, comme illustré dans l’exemple ci-dessous :

![stratégie IPSec/IKE](./media/vpn-gateway-about-compliance-crypto/ipsecikepolicy.png)

Vous pouvez créer une stratégie IPsec/IKE et l’appliquer à une connexion nouvelle ou existante.

### <a name="workflow"></a>Workflow

1. Créer des réseaux virtuels, des passerelles VPN ou des passerelles de réseau local pour votre topologie de connectivité, comme décrit dans d’autres documents de guide pratique
2. Créez une stratégie IPsec/IKE.
3. Vous pouvez appliquer la stratégie quand vous créez une connexion S2S ou de réseau virtuel à réseau virtuel
4. Si la connexion est déjà créée, vous pouvez appliquer ou mettre à jour la stratégie sur une connexion existante

## <a name="ipsecike-policy-faq"></a>Questions fréquentes (FAQ) relatives à la stratégie IPsec/IKE

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Consultez [Configurer la stratégie IPsec/IKE](vpn-gateway-ipsecikepolicy-rm-powershell.md) pour obtenir des instructions détaillées sur la configuration d’une stratégie IPsec/IKE personnalisée sur une connexion.

Consultez aussi [Connecter plusieurs appareils VPN en fonction de stratégies](vpn-gateway-connect-multiple-policybased-rm-ps.md) pour en savoir plus sur l’option UsePolicyBasedTrafficSelectors.
