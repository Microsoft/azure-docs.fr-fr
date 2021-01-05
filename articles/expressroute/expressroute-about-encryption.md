---
title: 'Azure ExpressRoute : À propos du chiffrement'
description: Découvrez ce qu’est le chiffrement ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: duau
ms.openlocfilehash: 693d2304324bdfcac298b3e20ddd0d882a16533c
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92899861"
---
# <a name="expressroute-encryption"></a>Chiffrement ExpressRoute
 
ExpressRoute prend en charge deux technologies de chiffrement pour garantir la confidentialité et l’intégrité des données qui transitent entre votre réseau et le réseau de Microsoft.

## <a name="point-to-point-encryption-by-macsec-faq"></a>Questions fréquentes sur le chiffrement point à point de MACsec
MACsec est une [norme IEEE](https://1.ieee802.org/security/802-1ae/). Cette technologie chiffre les données au niveau de la couche MAC (Media Access Control) ou de la couche réseau 2. Vous pouvez utiliser MACsec pour chiffrer les liens physiques entre vos appareils réseau et les appareils réseau de Microsoft quand vous vous connectez à Microsoft par le biais d’[ExpressRoute Direct](expressroute-erdirect-about.md). MACsec est désactivé sur les ports ExpressRoute Direct par défaut. Vous apportez votre propre clé MACsec pour le chiffrement et la stockez dans [Azure Key Vault](../key-vault/general/overview.md). Vous déterminez à quel moment permuter la clé. Consultez d’autres questions fréquentes ci-après.
### <a name="can-i-enable-macsec-on-my-expressroute-circuit-provisioned-by-an-expressroute-provider"></a>Puis-je activer MACsec sur mon circuit ExpressRoute provisionné par un fournisseur ExpressRoute ?
Non. MACsec chiffre tout le trafic sur un lien physique avec une clé appartenant à une entité unique (le client). Il est donc disponible uniquement sur ExpressRoute Direct.
### <a name="can-i-encrypt-some-of-the-expressroute-circuits-on-my-expressroute-direct-ports-and-leave-other-circuits-on-the-same-ports-unencrypted"></a>Puis-je chiffrer certains circuits ExpressRoute sur mes ports ExpressRoute Direct et ne pas chiffrer les autres circuits sur les mêmes ports ? 
Non. Une fois que vous avez activé MACsec, tout le trafic de contrôle réseau, comme le trafic de données BGP, et le trafic de données client sont chiffrés. 
### <a name="when-i-enabledisable-macsec-or-update-macsec-key-will-my-on-premises-network-lose-connectivity-to-microsoft-over-expressroute"></a>Quand j’active/je désactive MACsec ou quand je mets à jour la clé MACsec, mon réseau local perd-il la connectivité à Microsoft sur ExpressRoute ?
Oui. Pour la configuration de MACsec, nous prenons en charge le mode clé prépartagée uniquement. Vous devez donc mettre à jour la clé à la fois sur vos appareils et sur ceux de Microsoft (via notre API). Cette modification n’étant pas atomique, vous perdez la connectivité si les clés de chaque côté ne correspondent pas. Nous vous recommandons fortement de planifier une fenêtre de maintenance pour la modification de la configuration. Pour réduire le temps d’arrêt, nous vous suggérons de mettre à jour la configuration sur un lien d’ExpressRoute Direct à la fois, après avoir basculé le trafic réseau sur l’autre lien.  
### <a name="will-traffic-continue-to-flow-if-theres-a-mismatch-in-macsec-key-between-my-devices-and-microsofts"></a>Le trafic continue-t-il si la clé MACsec sur mes appareils ne correspond pas à celle sur les appareils de Microsoft ?
Non. Si MACsec est activé et que les clés ne correspondent pas, vous perdez la connectivité à Microsoft. Autrement dit, nous ne revenons pas à une connexion non chiffrée, qui exposerait vos données. 
### <a name="will-enabling-macsec-on-expressroute-direct-degrade-network-performance"></a>L’activation de MACsec sur ExpressRoute Direct dégrade-t-elle les performances du réseau ?
Le chiffrement/déchiffrement MACsec s’effectue au niveau du matériel sur les routeurs que nous utilisons. Il n’y a pas d’impact sur les performances de notre côté. Toutefois, vérifiez auprès du fournisseur du service réseau si MACsec a un impact sur les performances pour les appareils que vous utilisez.
### <a name="which-cipher-suites-are-supported-for-encryption"></a>Quelles sont les suites de chiffrement prises en charge pour le chiffrement ?
Nous prenons uniquement en charge la version d’[Extended Packet Numbering](https://1.ieee802.org/security/802-1aebw/) d’AES-128 et d’AES-256. Vous devez en outre désactiver [Secure Channel Identifier (SCI)](https://wikipedia.org/wiki/IEEE_802.1AE) dans la configuration MACsec sur votre appareil. 

## <a name="end-to-end-encryption-by-ipsec-faq"></a>Questions fréquentes sur le chiffrement de bout en bout d’IPsec
IPsec est une [norme IETF](https://tools.ietf.org/html/rfc6071). Cette technologie chiffre les données au niveau des adresses IP (Internet Protocol) ou de la couche réseau 3. Vous pouvez utiliser IPsec pour chiffrer une connexion de bout en bout entre votre réseau local et votre réseau virtuel (VNET) sur Azure. Consultez d’autres questions fréquentes ci-après.
### <a name="can-i-enable-ipsec-in-addition-to-macsec-on-my-expressroute-direct-ports"></a>Puis-je activer IPsec en plus de MACsec sur mes ports ExpressRoute Direct ?
Oui. MACsec sécurise les connexions physiques entre vos appareils et ceux de Microsoft. IPsec sécurise la connexion de bout en bout entre votre réseau local et vos réseaux virtuels sur Azure. Vous pouvez activer ces deux technologies indépendamment. 
### <a name="can-i-use-azure-vpn-gateway-to-set-up-the-ipsec-tunnel-over-azure-private-peering"></a>Puis-je utiliser la passerelle VPN Azure pour configurer le tunnel IPsec sur le peering privé Azure ?
Oui. Si vous adoptez un Azure Virtual WAN, vous pouvez suivre [ces étapes](../virtual-wan/vpn-over-expressroute.md) pour chiffrer la connexion de bout en bout. Si vous avez un réseau virtuel Azure traditionnel, vous pouvez suivez [ces étapes](../vpn-gateway/site-to-site-vpn-private-peering.md) pour configurer un tunnel IPsec entre la passerelle VPN Azure et votre passerelle VPN locale.
### <a name="what-is-the-throughput-i-will-get-after-enabling-ipsec-on-my-expressroute-connection"></a>Après l’activation d’IPsec sur ma connexion ExpressRoute, quel sera le débit ?
Si vous utilisez la passerelle VPN Azure, consultez les [résultats des performances](../vpn-gateway/vpn-gateway-about-vpngateways.md). Si vous utilisez une passerelle VPN tierce, contactez le fournisseur pour obtenir les résultats des performances.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la configuration de MACsec, consultez [Configurer MACsec](expressroute-howto-macsec.md).

Pour plus d’informations sur la configuration d’IPsec, consultez [Configurer IPsec](site-to-site-vpn-over-microsoft-peering.md).
