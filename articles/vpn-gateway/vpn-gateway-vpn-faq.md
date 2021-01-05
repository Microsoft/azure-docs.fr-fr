---
title: FAQ sur la passerelle VPN Azure
description: Le FAQ sur la passerelle VPN FAQ sur les connexions entre différents locaux de Microsoft Azure Virtual Network, les connexions de configuration hybride et les passerelles VPN.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 7e59c8ecc0d7af341ddc1ea79aa42460e00fa444
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89419773"
---
# <a name="vpn-gateway-faq"></a>FAQ sur la passerelle VPN

## <a name="connecting-to-virtual-networks"></a><a name="connecting"></a>Connexion à des réseaux virtuels

### <a name="can-i-connect-virtual-networks-in-different-azure-regions"></a>Puis-je me connecter à des réseaux virtuels dans différentes régions Azure ?

Oui. En fait, il n'existe aucune contrainte de région. Un réseau virtuel peut se connecter à un autre réseau virtuel dans la même région ou dans une autre région Azure. 

### <a name="can-i-connect-virtual-networks-in-different-subscriptions"></a>Puis-je me connecter à des réseaux virtuels avec différents abonnements ?

Oui.

### <a name="can-i-connect-to-multiple-sites-from-a-single-virtual-network"></a>Puis-je me connecter à plusieurs sites à partir d'un seul réseau virtuel ?

Vous pouvez vous connecter à plusieurs sites à l'aide de Windows PowerShell et des API REST Azure. Consultez la rubrique FAQ sur la [Connectivité multisite et de réseau virtuel à réseau virtuel](#V2VMulti) .

### <a name="is-there-an-additional-cost-for-setting-up-a-vpn-gateway-as-active-active"></a>La configuration d’une passerelle VPN en mode actif-actif engendre-t-elle des coûts supplémentaires ?

Non. 

### <a name="what-are-my-cross-premises-connection-options"></a>Quelles sont mes options de connexion entre différents locaux ?

Les connexions intersites suivantes sont prises en charge :

* Site à site : connexion VPN sur IPsec (IKE v1 et IKE v2). Ce type de connexion requiert un périphérique VPN ou le service RRAS. Pour plus d’informations, consultez [Création d’une connexion de site à site dans le portail Azure](vpn-gateway-howto-site-to-site-resource-manager-portal.md).
* Point à site : connexion VPN sur SSTP (Secure Socket Tunneling Protocol) ou IKE v2. Cette connexion ne nécessite pas un périphérique VPN. Pour plus d’informations, consultez [Configuration d’une connexion point à site à un réseau virtuel à l’aide du portail Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md).
* Réseau virtuel à réseau virtuel : ce type de connexion est identique à une configuration site à site. La connexion de réseau virtuel à réseau virtuel est une connexion VPN sur IPsec (IKE v1 et IKE v2). Cette connexion ne nécessite pas un périphérique VPN. Pour plus d’informations, consultez [Configurer une connexion de réseau virtuel à réseau virtuel à l’aide du portail Azure](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).
* Multisite : il s’agit d’une variante d’une configuration site à site qui vous permet de connecter plusieurs sites locaux à un réseau virtuel. Pour plus d’informations, consultez [Ajouter une connexion de site à site à un réseau virtuel avec une connexion de passerelle VPN existante](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md).
* ExpressRoute : ExpressRoute est une connexion privée à Azure à partir de votre WAN, pas une connexion VPN qui passe par l’Internet public. Pour plus d’informations, consultez [Vue d’ensemble technique d’ExpressRoute](../expressroute/expressroute-introduction.md) et la [FAQ ExpressRoute](../expressroute/expressroute-faqs.md).

Pour plus d’informations sur les connexions à la passerelle VPN, consultez [À propos de la passerelle VPN](vpn-gateway-about-vpngateways.md).

### <a name="what-is-the-difference-between-a-site-to-site-connection-and-point-to-site"></a>Quelle est la différence entre une connexion site à site et point à site ?

Les configurations **Site à site** (tunnel VPN IPsec/IKE) se trouvent entre votre emplacement local et Azure. Elles vous permettent donc de vous connecter à partir de n’importe quel ordinateur se trouvant sur votre serveur local vers une machine virtuelle ou une instance de rôle au sein de votre réseau virtuel, selon le mode de configuration du routage et les autorisations que vous choisissez. Il s’agit d’une excellente solution pour une connexion entre différents locaux toujours disponibles et elle convient parfaitement aux configurations hybrides. Ce type de connexion s’appuie sur une appliance VPN IPsec (matériel ou application logicielle), qui doit être déployée à la périphérie de votre réseau. Pour créer ce type de connexion, vous devez disposer d’une adresse IPv4 externe.

Les configurations **Point à site** (VPN sur SSTP) vous permettent de vous connecter à partir d’un seul ordinateur depuis n’importe où et vers n’importe quel emplacement sur votre réseau virtuel. Ces connexions utilisent le client VPN fourni avec Windows. Dans le cadre de la configuration point à site, vous installez un certificat et un package de configuration client VPN qui contient les paramètres permettant à votre ordinateur de se connecter à une machine virtuelle ou à une instance de rôle au sein du réseau virtuel. Cela est très intéressant lorsque vous souhaitez vous connecter à un réseau virtuel mais que vous ne vous trouvez pas en local. Il s’agit également d’une solution intéressante lorsque vous n’avez pas accès au matériel VPN ou à une adresse IPv4 externe, tous deux étant nécessaires à une connexion site à site.

Vous pouvez configurer votre réseau virtuel pour utiliser la connexion site à site et point à site simultanément, à condition de créer votre connexion site à site à l’aide d’un type de réseau privé virtuel basé sur un itinéraire pour votre passerelle. Les types de réseau privé virtuel basé sur un itinéraire sont appelés des passerelles dynamiques dans le modèle de déploiement classique.

## <a name="virtual-network-gateways"></a><a name="gateways"></a>Passerelles de réseau virtuel

### <a name="is-a-vpn-gateway-a-virtual-network-gateway"></a>Une passerelle VPN est-elle une passerelle de réseau virtuel ?

Une passerelle VPN est un type de passerelle de réseau virtuel. Une passerelle VPN envoie le trafic chiffré entre votre réseau virtuel et votre emplacement local à travers une connexion publique. Vous pouvez également utiliser une passerelle VPN pour acheminer le trafic entre réseaux virtuels. Lorsque vous créez une passerelle VPN, vous utilisez la valeur -GatewayType « Vpn ». Pour plus d’informations, consultez [À propos des paramètres de configuration de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md).

### <a name="what-is-a-policy-based-static-routing-gateway"></a>Qu’est-ce qu’une passerelle basée sur une stratégie (routage statique) ?

Les passerelles basées sur des stratégies implémentent des VPN basés sur des stratégies. Les VPN basés sur des stratégies chiffrent et acheminent les paquets via les tunnels IPsec basés sur des combinaisons de préfixes d’adresses entre votre réseau local et le réseau virtuel Azure. La stratégie (ou le sélecteur de trafic) est généralement définie en tant que liste d'accès dans les configurations VPN.

### <a name="what-is-a-route-based-dynamic-routing-gateway"></a>Qu’est-ce qu’une passerelle basée sur l’itinéraire (routage dynamique) ?

Les passerelles basées sur des itinéraires implémentent les VPN basés sur des itinéraires. Les VPN basés sur l'itinéraire utilisent des « itinéraires » dans l'adresse IP de transfert ou la table de routage pour acheminer des paquets dans leurs interfaces de tunnel correspondantes. Les interfaces de tunnel chiffrent ou déchiffrent ensuite les paquets se trouvant dans et hors des tunnels. La stratégie ou le sélecteur de trafic pour les VPN basés sur l'itinéraire sont configurés comme universels (ou en caractères génériques).

### <a name="can-i-update-my-policy-based-vpn-gateway-to-route-based"></a>Puis-je mettre à jour ma passerelle VPN basée sur une stratégie en passerelle VPN basée sur l’itinéraire ?

Non. Vous ne pouvez pas modifier le type de passerelle de réseau virtuelle Azure. La passerelle doit être supprimée et recréée. Le processus dure environ 60 minutes. Ni l’adresse IP de la passerelle ni la clé prépartagée (PSK) ne sont conservées.
1. Supprimez toutes connexions associées à la passerelle que vous comptez supprimer.
1. Supprimez la passerelle :
   - [Azure portal](vpn-gateway-delete-vnet-gateway-portal.md)
   - [Azure PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
   - [Azure PowerShell - Classic](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
1. [Créez une passerelle du type souhaité et terminer la configuration VPN](vpn-gateway-howto-site-to-site-resource-manager-portal.md#VNetGateway).

### <a name="do-i-need-a-gatewaysubnet"></a>Ai-je besoin d’un « sous-réseau de passerelle » ?

Oui. Le sous-réseau de passerelle contient les adresses IP utilisées par les services de passerelle de réseau virtuel. Vous devez créer un sous-réseau de passerelle pour votre réseau virtuel afin de configurer une passerelle de réseau virtuel. Tous les sous-réseaux de passerelle doivent être nommés « GatewaySubnet » afin de fonctionner correctement. N’attribuez pas un autre nom au sous-réseau de passerelle, et ne déployez pas de machines virtuelles ou d’autres éléments vers le sous-réseau de passerelle.

Lorsque vous créez le sous-réseau de passerelle, vous spécifiez le nombre d’adresses IP que contient le sous-réseau. Les adresses IP dans le sous-réseau de passerelle sont allouées au service de passerelle. Certaines configurations nécessitent d’allouer davantage d’adresses IP aux services de passerelle. Assurez-vous que votre sous-réseau de passerelle contient suffisamment d’adresses IP pour pouvoir évoluer au rythme de votre croissance future et prendre en charge d’éventuelles nouvelles configurations de connexion. Bien qu’il soit possible de créer un sous-réseau de passerelle aussi petit que /29, nous vous recommandons de créer un sous-réseau de passerelle de taille /27 ou supérieure (/27, /26, /25, etc.). Prenez connaissance des recommandations relatives à la configuration que vous souhaitez créer et vérifiez que votre sous-réseau de passerelle suit ces recommandations.

### <a name="can-i-deploy-virtual-machines-or-role-instances-to-my-gateway-subnet"></a>Puis-je déployer des machines virtuelles ou des instances de rôle vers mon sous-réseau de passerelle ?

Non.

### <a name="can-i-get-my-vpn-gateway-ip-address-before-i-create-it"></a>Puis-je obtenir mon adresse IP de passerelle VPN avant de la créer ?

Les passerelles zonales et redondantes interzone (références SKU de passerelle avec _AZ_ dans leur nom) reposent sur une ressource IP publique Azure de _référence SKU Standard_. Les ressources IP publiques de référence SKU Standard Azure doivent utiliser une méthode d’allocation statique. Ainsi, vous disposez de l’adresse IP publique de votre passerelle VPN dès que vous créez la ressource IP publique de référence SKU Standard que vous prévoyez d’utiliser pour celle-ci.

Pour les passerelles non zonales et non redondantes interzone (références SKU de passerelle _sans_ _AZ_ dans leur nom), vous ne pouvez pas obtenir l’adresse IP de la passerelle VPN avant sa création. L’adresse IP change uniquement si vous supprimez et recréez votre passerelle VPN.

### <a name="can-i-request-a-static-public-ip-address-for-my-vpn-gateway"></a>Puis-je demander une adresse IP publique statique pour ma passerelle VPN ?

Comme indiqué ci-dessus, les passerelles zonales et redondantes interzone (références SKU de passerelle avec _AZ_ dans leur nom) reposent toutes deux sur une ressource IP publique Azure de _référence SKU standard_. Les ressources IP publiques de référence SKU Standard Azure doivent utiliser une méthode d’allocation statique.

Pour les passerelles non zonales et non redondantes interzone (références SKU de passerelle _sans_ _AZ_ dans leur nom), seule l’attribution d’adresse IP dynamique est prise en charge. Toutefois, cela ne signifie pas que l’adresse IP change après son affectation à votre passerelle VPN. L’adresse IP de la passerelle VPN change uniquement lorsque la passerelle est supprimée, puis recréée. L’adresse IP publique de la passerelle VPN ne change pas lors du redimensionnement, de la réinitialisation ou des autres opérations de maintenance/mise à niveau internes de votre passerelle VPN.

### <a name="how-does-my-vpn-tunnel-get-authenticated"></a>Comment mon tunnel VPN est-il authentifié ?

Azure VPN utilise l'authentification PSK (clé prépartagée). Nous générons une clé prépartagée (PSK) lorsque nous créons le tunnel VPN. Vous pouvez modifier la PSK générée automatiquement pour votre VPN avec la cmdlet PowerShell ou l’API REST de clé prépartagée définie.

### <a name="can-i-use-the-set-pre-shared-key-api-to-configure-my-policy-based-static-routing-gateway-vpn"></a>Puis-je utiliser l’API de clé prépartagée définie pour configurer ma passerelle VPN basée sur des itinéraires (routage statique) ?

Oui, l’applet de commande PowerShell et l’API de clé prépartagée définie permettent de configurer des réseaux VPN Azure basés sur des stratégies (statiques) et des réseaux VPN basés sur des itinéraires (dynamiques).

### <a name="can-i-use-other-authentication-options"></a>Puis-je utiliser les autres options d'authentification ?

Nous sommes limités à l'utilisation de clés prépartagées (PSK) pour l'authentification.

### <a name="how-do-i-specify-which-traffic-goes-through-the-vpn-gateway"></a>Comment puis-je indiquer quel trafic traverse la passerelle VPN ?

#### <a name="resource-manager-deployment-model"></a>Modèle de déploiement de Resource Manager

* PowerShell : utilisez le paramètre « AddressPrefix » pour spécifier le trafic de la passerelle réseau locale.
* Portail Azure : accédez à la zone Passerelle de réseau local > Configuration > Espace d’adressage.

#### <a name="classic-deployment-model"></a>Modèle de déploiement classique

* Portail Azure : accédez au réseau virtuel classique > Connexions VPN > Connexions VPN site à site > Nom du site local > Site local > Espace d’adressage du client. 

### <a name="can-i-configure-force-tunneling"></a>Puis-je configurer un tunneling forcé ?

Oui. Consultez [Configurer un tunneling forcé](vpn-gateway-about-forced-tunneling.md).

### <a name="can-i-use-nat-t-on-my-vpn-connections"></a>Puis-je utiliser NAT-T sur mes connexions VPN ?

Oui, NAT Traversal (NAT-T) est pris en charge. La passerelle VPN Azure n’effectue AUCUNE fonctionnalité de type NAT sur les paquets internes vers/à partir des tunnels IPsec.  Dans cette configuration, vérifiez que l’appareil local initie le tunnel IPSec.

### <a name="can-i-set-up-my-own-vpn-server-in-azure-and-use-it-to-connect-to-my-on-premises-network"></a>Puis-je configurer mon propre serveur VPN dans Azure et l’utiliser pour me connecter à mon réseau local ?

Oui, vous pouvez déployer vos propres serveurs ou passerelles VPN dans Azure depuis Azure Marketplace ou en créant vos propres routeurs VPN. Vous devez configurer des itinéraires définis par l’utilisateur dans votre réseau virtuel pour vous assurer que le trafic est acheminé correctement entre vos réseaux locaux et les sous-réseaux de votre réseau virtuel.

### <a name="why-are-certain-ports-opened-on-my-virtual-network-gateway"></a><a name="gatewayports"></a>Pourquoi certains ports sont-ils ouverts sur ma passerelle de réseau virtuel ?

Ils sont nécessaires pour la communication avec l’infrastructure Azure. Ils sont protégés (verrouillés) par des certificats Azure. Sans les certificats appropriés, les entités externes (notamment les clients de ces passerelles) ne pourront avoir aucun effet sur ces points de terminaison.

Une passerelle de réseau virtuel est, fondamentalement, un périphérique multirésident avec une carte réseau qui exploite le réseau privé du client et une autre carte réseau connectée au réseau public. Les entités de l’infrastructure Azure ne peuvent pas se connecter aux réseaux privés des clients pour des raisons de conformité. Elles doivent donc utiliser des points de terminaison publics pour la communication avec l’infrastructure. Les points de terminaison publics sont analysés régulièrement par l’audit de sécurité Azure.

### <a name="more-information-about-gateway-types-requirements-and-throughput"></a>Plus d'informations sur les types de passerelle, la configuration requise et le débit

Pour plus d’informations, consultez [À propos des paramètres de configuration de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md).

## <a name="site-to-site-connections-and-vpn-devices"></a><a name="s2s"></a>Connexions de site à site et périphériques VPN

### <a name="what-should-i-consider-when-selecting-a-vpn-device"></a>Que dois-je prendre en compte lors de la sélection d'un périphérique VPN ?

Nous avons validé un ensemble de périphériques VPN site à site standard en partenariat avec des fournisseurs de périphériques. Vous trouverez une liste de périphériques VPN compatibles, leurs instructions de configuration correspondantes ou des exemples, ainsi que les caractéristiques du périphérique dans l’article [À propos des périphériques VPN et des paramètres IPsec/IKE pour les connexions de passerelle VPN site à site](vpn-gateway-about-vpn-devices.md). Tous les périphériques des familles de périphériques répertoriées et reconnus comme compatibles doivent fonctionner avec Virtual Network. Pour aider à configurer votre périphérique VPN, reportez-vous à l'exemple de configuration de périphérique ou au lien qui correspond à la famille de périphériques appropriée.

### <a name="where-can-i-find-vpn-device-configuration-settings"></a>Où puis-je trouver les paramètres de configuration des périphériques VPN ?

[!INCLUDE [vpn devices](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

### <a name="how-do-i-edit-vpn-device-configuration-samples"></a>Comment dois-je modifier les exemples de configuration de périphérique VPN ?

Pour plus d’informations sur la modification des exemples de configuration des périphériques, consultez la page [Modifier les exemples](vpn-gateway-about-vpn-devices.md#editing).

### <a name="where-do-i-find-ipsec-and-ike-parameters"></a>Où puis-je trouver les paramètres IPsec et IKE ?

En ce qui concerne les paramètres IPsec/IKE, consultez la page [Paramètres](vpn-gateway-about-vpn-devices.md#ipsec).

### <a name="why-does-my-policy-based-vpn-tunnel-go-down-when-traffic-is-idle"></a>Pourquoi mon tunnel VPN basé sur une stratégie tombe-t-il en panne lorsque le trafic est inactif ?

Il s’agit du comportement attendu pour les passerelles VPN basées sur une stratégie (également appelé routage statique). Lorsque le trafic via le tunnel est inactif pendant plus de 5 minutes, le tunnel est détruit. Lorsque le trafic recommence à circuler dans les deux sens, le tunnel est immédiatement rétabli.

### <a name="can-i-use-software-vpns-to-connect-to-azure"></a>Puis-je utiliser le logiciel des VPN logiciels pour me connecter à Azure ?

Nous prenons en charge les serveurs de routage et d’accès distant (RRAS) Windows Server 2012 pour la configuration site à site entre différents locaux.

D’autres solutions VPN logicielles fonctionnent avec notre passerelle tant qu'elles sont conformes aux implémentations IPsec standard. Contactez le fournisseur du logiciel pour obtenir des instructions de configuration et de prise en charge.

## <a name="how-do-i-change-the-authentication-type-for-my-point-to-site-connections"></a>Comment faire modifier le type d’authentification pour mes connexions de point à site ?

Vous pouvez modifier la méthode d’authentification pour vos connexions point à site en accédant à la section **Configuration de point à site** sous la passerelle VPN et en cochant la case d’option souhaitée. Les options actuelles sont **Certificat Azure, Authentification RADIUS et Azure Active Directory**. Notez que les clients actuels **peuvent ne pas être en mesure de se connecter** après la modification jusqu’à ce que le nouveau profil ait été téléchargé et configuré sur le client.

## <a name="point-to-site-using-native-azure-certificate-authentication"></a><a name="P2S"></a>Point à site utilisant l’authentification par certificat Azure native

Cette section s’applique au modèle de déploiement Resource Manager.

[!INCLUDE [P2S Azure cert](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="point-to-site-using-radius-authentication"></a><a name="P2SRADIUS"></a>Point à site utilisant l’authentification RADIUS

Cette section s’applique au modèle de déploiement Resource Manager.

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="vnet-to-vnet-and-multi-site-connections"></a><a name="V2VMulti"></a>Connexions multi-sites et réseau virtuel à réseau virtuel

[!INCLUDE [vpn-gateway-vnet-vnet-faq-include](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

### <a name="can-i-use-azure-vpn-gateway-to-transit-traffic-between-my-on-premises-sites-or-to-another-virtual-network"></a>Puis-je utiliser une passerelle VPN Azure pour faire transiter le trafic entre mes sites locaux ou vers un autre réseau virtuel ?

**Modèle de déploiement de Resource Manager**<br>
Oui. Pour en savoir plus, voir [BGP](#bgp).

**Modèle de déploiement classique**<br>
 Le transit du trafic via la passerelle VPN Azure est possible à l’aide du modèle de déploiement Classic, mais il s’appuie sur les espaces d’adressage définis de manière statique dans le fichier de configuration réseau. BGP n’est pas encore pris en charge avec les réseaux virtuels Azure et les passerelles VPN à l’aide du modèle de déploiement Classic. Sans BGP, la définition manuelle des espaces d’adressage de transit peut entraîner des erreurs et n’est pas recommandée.

### <a name="does-azure-generate-the-same-ipsecike-pre-shared-key-for-all-my-vpn-connections-for-the-same-virtual-network"></a>Azure génère-t-il la même clé prépartagée IPsec/IKE pour toutes les connexions VPN d'un même réseau virtuel ?

Non, Azure génère par défaut différentes clés prépartagées pour différentes connexions VPN. Toutefois, vous pouvez utiliser l’API REST de clé de passerelle VPN définie ou la cmdlet PowerShell pour définir la valeur clé de votre choix. La clé DOIT être composé de caractères ASCII imprimables.

### <a name="do-i-get-more-bandwidth-with-more-site-to-site-vpns-than-for-a-single-virtual-network"></a>Puis-je obtenir plus de bande passante avec un plus grand nombre de réseaux VPN site à site que si j’utilise un seul réseau virtuel ?

Non, tous les tunnels VPN, y compris les VPN point à site, partagent la même passerelle VPN Azure et la bande passante disponible.

### <a name="can-i-configure-multiple-tunnels-between-my-virtual-network-and-my-on-premises-site-using-multi-site-vpn"></a>Puis-je configurer plusieurs tunnels entre mon réseau virtuel et mon site local à l'aide d’un VPN multisite ?

Oui, mais vous devez configurer BGP sur les deux tunnels, au même emplacement.

### <a name="can-i-use-point-to-site-vpns-with-my-virtual-network-with-multiple-vpn-tunnels"></a>Puis-je utiliser des réseaux VPN point à site avec mon réseau virtuel comportant plusieurs tunnels VPN ?

Oui, les réseaux VPN point à site peuvent être utilisés avec les passerelles VPN se connectant à plusieurs sites locaux et à d’autres réseaux virtuels.

### <a name="can-i-connect-a-virtual-network-with-ipsec-vpns-to-my-expressroute-circuit"></a>Puis-je me connecter à un réseau virtuel avec les VPN IPsec sur mon circuit ExpressRoute ?

Oui, ceci est pris en charge. Pour plus d’informations, consultez [Configurer des connexions ExpressRoute et VPN de site à site pour qu’elles coexistent pour un réseau virtuel](../expressroute/expressroute-howto-coexist-classic.md).

## <a name="ipsecike-policy"></a><a name="ipsecike"></a>Stratégie IPsec/IKE

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]


## <a name="bgp"></a><a name="bgp"></a>BGP

[!INCLUDE [vpn-gateway-faq-bgp-include](../../includes/vpn-gateway-faq-bgp-include.md)]

## <a name="cross-premises-connectivity-and-vms"></a><a name="vms"></a>Connectivité entre locaux et machines virtuelles

### <a name="if-my-virtual-machine-is-in-a-virtual-network-and-i-have-a-cross-premises-connection-how-should-i-connect-to-the-vm"></a>Si ma machine virtuelle est dans un réseau virtuel et que j'ai une connexion entre différents locaux, comment dois-je me connecter à la machine virtuelle ?

Vous disposez de plusieurs options. Si vous avez activé le protocole RDP pour votre machine virtuelle, vous pouvez vous connecter à cette dernière à l’aide de l’adresse IP privée. Dans ce cas, vous allez spécifier l’adresse IP privée et le port auquel vous souhaitez vous connecter (généralement 3389). Vous devez configurer le port sur votre machine virtuelle pour le trafic.

Vous pouvez également vous connecter à votre machine virtuelle en adresse IP privée à partir d’une autre machine virtuelle qui se trouve sur le même réseau virtuel. Vous ne pouvez pas vous connecter en RDP à votre machine virtuelle à l’aide d’une adresse IP privée si vous vous connectez à partir d’un emplacement en dehors de votre réseau virtuel. Par exemple, si vous avez configuré un réseau virtuel point à site et que vous n’établissez pas de connexion à partir de votre ordinateur, vous ne pouvez pas vous connecter à la machine virtuelle en adresse IP privée.

### <a name="if-my-virtual-machine-is-in-a-virtual-network-with-cross-premises-connectivity-does-all-the-traffic-from-my-vm-go-through-that-connection"></a>Si ma machine virtuelle est dans un réseau virtuel avec connectivité entre différents locaux, tout le trafic de ma machine virtuelle passe-t-il par cette connexion ?

Non. Seul le trafic qui possède une IP de destination contenue dans les plages d'adresse IP du réseau local du réseau virtuel que vous avez spécifié passera par la passerelle de réseau virtuel. Le trafic avec une IP de destination qui se trouve dans le réseau virtuel reste dans le réseau virtuel. Tout autre trafic est envoyé par le biais de l'équilibreur de charge aux réseaux publics, ou si le tunneling forcé est utilisé, il est envoyé via la passerelle VPN Azure.

### <a name="how-do-i-troubleshoot-an-rdp-connection-to-a-vm"></a>Résolution d’une connexion RDP à une machine virtuelle

[!INCLUDE [Troubleshoot VM connection](../../includes/vpn-gateway-connect-vm-troubleshoot-include.md)]


## <a name="virtual-network-faq"></a><a name="faq"></a>Forum aux questions sur le réseau virtuel

Des informations supplémentaires sur les réseaux virtuels sont disponibles dans le [Forum Aux Questions sur les réseaux virtuels](../virtual-network/virtual-networks-faq.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la passerelle VPN, consultez [À propos de la passerelle VPN](vpn-gateway-about-vpngateways.md).
* Pour plus d’informations sur les paramètres de configuration de la passerelle VPN, consultez [À propos des paramètres de configuration de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md).

**« OpenVPN » est une marque d’OpenVPN Inc.**
