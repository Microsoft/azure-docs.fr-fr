---
title: Azure Virtual WAN et travail à distance
description: Cette page décrit comment tirer parti d’Azure Virtual WAN pour travailler à distance dans le contexte de la pandémie de COVID-19.
services: virtual-wan
author: reyandap
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 03/22/2020
ms.author: cherylmc
ms.openlocfilehash: fc048f3da3156f5e17cfa32479b834b7320a60a2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023486"
---
# <a name="azure-virtual-wan-and-supporting-remote-work"></a>Azure Virtual WAN et prise en charge du travail à distance

>[!NOTE]
>Cet article décrit comment vous pouvez tirer parti d’Azure Virtual WAN, d’Azure, du réseau Microsoft et de l’écosystème de partenaires Azure pour travailler à distance et atténuer les problèmes de réseau auxquels vous êtes confronté dans le contexte de la crise de COVID-19.
>

Vous vous démenez pour fournir une connectivité à des utilisateurs distants ?
Vous avez soudainement besoin de prendre en charge une augmentation du nombre d’utilisateurs au-delà de ce que vous aviez prévu ?
Vous avez besoin que l’utilisateur puisse se connecter à partir de chez lui et qu’en plus d’accéder au cloud, il puisse atteindre l’emplacement local ?
Vous avez besoin que vos utilisateurs distants puissent accéder à des ressources situées derrière un WAN privé ?
Vous avez besoin que les utilisateurs puissent accéder aux ressources à l’intérieur du cloud sans devoir configurer la connectivité entre régions ?
Étant donné que cette pandémie planétaire génère autour de nous des changements sans précédent, l’équipe Azure Virtual WAN est là pour répondre à vos besoins de connectivité.

Azure Virtual WAN est un service de mise en réseau qui combine un grand nombre de fonctionnalités de mise en réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Ces fonctionnalités sont les suivantes : connectivité de branche (via une automatisation de la connectivité d’appareils partenaires d’Azure Virtual WAN, tels que SD-WAN ou un CPE VPN), connectivité VPN site à site, connectivité VPN d’utilisateurs distants (point à site), connectivité privée (ExpressRoute), connectivité intra-cloud (connectivité transitive pour les réseaux virtuels), interconnectivité ExpressRoute de VPN, routage, Pare-feu Azure, chiffrement de connexion privée, etc. Tous ces cas d’usage ne doivent pas nécessairement être réunis pour commencer à utiliser Azure Virtual WAN. Vous pouvez commencer avec un seul cas d’usage et ajuster votre réseau au fur et à mesure de son évolution.

![Diagramme WAN virtuel](./media/virtual-wan-about/virtualwan1.png)

À propos des utilisateurs distants, découvrez ce dont vous avez besoin pour que votre réseau soit opérationnel :

## <a name="set-up-remote-user-connectivity"></a><a name="connectivity"></a>Configurer la connectivité de l’utilisateur distant

Vous pouvez vous connecter à vos ressources Azure via une connexion IPsec/IKE (IKEv2) ou OpenVPN. Ce type de connexion requiert un client VPN configuré pour l’utilisateur client. Ce client peut être le [Client VPN Azure](https://go.microsoft.com/fwlink/?linkid=2117554), un client OpenVPN ou tout client prenant en charge IKEv2. Pour plus d'informations, consultez [Créer une connexion point à site](virtual-wan-point-to-site-portal.md).

## <a name="connectivity-from-the-remote-user-to-on-premises"></a><a name="remote user connectivity"></a>Connectivité de l’utilisateur distant au site local

Deux options s’offrent à vous :

* Configurer une connectivité site à site avec n’importe quel périphérique VPN existant. Lorsque vous connectez le périphérique VPN IPsec au hub Azure Virtual WAN, l’interconnexion entre le VPN utilisateur point à site (utilisateur distant) et le VPN site à site est automatique. Pour plus d’informations sur la configuration d’un VPN site à site à partir de votre périphérique VPN local vers Azure Virtual WAN, consultez [Créer une connexion site à site à l’aide de Virtual WAN](virtual-wan-site-to-site-portal.md).

* Connecter votre circuit ExpressRoute au hub Azure Virtual WAN. La connexion d’un circuit ExpressRoute nécessite le déploiement d’une passerelle ExpressRoute dans Azure Virtual WAN. Dès que vous en avez déployé une, l’interconnexion entre l’utilisateur VPN point à site et l’utilisateur ExpressRoute est automatique. Pour créer la connexion ExpressRoute, consultez [Créer une connexion ExpressRoute à l’aide de Virtual WAN](virtual-wan-expressroute-portal.md). Vous pouvez utiliser un circuit ExpressRoute existant pour vous connecter à un Azure Virtual WAN.

## <a name="existing-basic-virtual-wan-customer"></a><a name="basic vWAN"></a>Client Azure Virtual WAN de base existant

Le Virtual WAN de base fournit uniquement un VPN site à site. Pour que les utilisateurs distants puissent se connecter, vous devez mettre à niveau le WAN virtuel vers un Virtual WAN Standard. Pour connaître la procédure de mise à niveau d’un WAN virtuel, consultez [Procéder à la mise à niveau d’un WAN virtuel De base en WAN virtuel Standard](upgrade-virtual-wan.md)

## <a name="additional-information"></a><a name="other considerations"></a>Informations supplémentaires

Un Virtual WAN prend en charge un hub par région/emplacement. Pour plus d’informations sur les emplacements, consultez l’article [Partenaires et emplacements Virtual WAN](virtual-wan-locations-partners.md). Chaque concentrateur prend en charge jusqu’à 10 000 connexions d’utilisateurs distants, 1 000 connexions de branche, quatre circuits ExpressRoute et 500 connexions de réseau virtuel. Lorsque vous mettez à l’échelle les utilisateurs distants, si vous avez des questions, n’hésitez pas à rechercher de l’aide en envoyant un e-mail à azurevirtualwan@microsoft.com. Si vous avez besoin d’un support technique, veillez à ouvrir un ticket de support à partir du portail Azure pour obtenir de l’aide.

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

[!INCLUDE [Virtual WAN FAQ](../../includes/virtual-wan-faq-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Virtual WAN, consultez l’article [Vue d’ensemble de Virtual WAN](virtual-wan-about.md)