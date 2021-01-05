---
title: 'Architecture : Architecture du réseau de transit global'
titleSuffix: Azure Virtual WAN
description: Découvrez comment Azure Virtual WAN permet une architecture du réseau de transit global en fournissant une connectivité omniprésente, universelle, entre des charges de travail cloud.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: cherylmc
ms.openlocfilehash: 59e60dadda7c0de37cfabadbc36ca53bc3c2b336
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563730"
---
# <a name="global-transit-network-architecture-and-virtual-wan"></a>Architecture du réseau de transit global Virtual WAN

Les entreprises modernes ont besoin d’une connectivité omniprésente entre les applications hyper-distribuées, les données et les utilisateurs dans le cloud et en local. L’architecture du réseau de transit global est adoptée par les entreprises pour consolider, connecter et contrôler l’empreinte informatique de l’entreprise mondiale moderne tournée vers le cloud.

L’architecture du réseau de transit global est basée sur un modèle classique de connectivité en étoile où le « hub » du réseau hébergé sur le cloud permet une connectivité transitive entre les terminaux qui peut être répartie sur différents types de « rayons ».

Dans ce modèle, un rayon peut être :
* Un réseau virtuel (VPN)
* Site de branche physique
* Utilisateur distant
* Internet

![Hub-and-spoke](./media/virtual-wan-global-transit-network-architecture/figure1.png)

**Figure 1 : Réseau hub-and-spoke de transit global**

La figure 1 montre la vue logique du réseau de transit global où les utilisateurs répartis géographiquement, les sites physiques et les réseaux virtuels sont interconnectés via un hub réseau hébergé dans le cloud. Cette architecture permet une connectivité de transit à un seul tronçon logique entre les points de terminaison du réseau.

## <a name="global-transit-network-with-virtual-wan"></a><a name="globalnetworktransit"></a>Réseau de transit global avec Virtual WAN

Azure Virtual WAN est un service réseau cloud managé par Microsoft. Tous les composants réseau qui composent ce service sont hébergés et managés par Microsoft. Pour plus d’informations sur Azure Virtual WAN, consultez [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).

Azure Virtual WAN permet une architecture de réseau de transit global en fournissant une connectivité omniprésente et universelle entre les groupes distribués mondialement de charges de travail cloud dans les réseaux virtuels, les sites de succursale, les applications SaaS et PaaS, et les utilisateurs.

![WAN virtuel Azure](./media/virtual-wan-global-transit-network-architecture/figure2.png)

**Figure 2 : Réseau de transit global et Virtual WAN**

Dans l’architecture Azure Virtual WAN, les hubs WAN virtuels sont provisionnés dans des régions Azure, auxquelles vous pouvez choisir de connecter vos branches, réseaux virtuels et utilisateurs distants. Les sites de succursale physiques sont connectés au hub via des connexions ExpressRoute Premium ou Standard ou des connexions VPN site à site, les réseaux virtuels sont connectés au hub par des connexions de réseau virtuel et les utilisateurs distants peuvent se connecter directement au hub à l’aide d’un VPN utilisateur (VPN point à site). Virtual WAN prend également en charge la connexion de réseaux virtuels inter-régions qui permet de connecter un réseau virtuel dans une région à un hub WAN virtuel situé dans une autre région.

Vous pouvez établir un WAN virtuel en créant un seul hub WAN virtuel dans la région qui possède le plus grand nombre de rayons (branches, réseaux virtuels, utilisateurs), puis en connectant à ce hub les rayons qui se trouvent dans d’autres régions. Cette approche est conseillée quand l’empreinte informatique de l’entreprise se limite principalement à une région avec quelques rayons à distance.  
  
## <a name="hub-to-hub-connectivity"></a><a name="hubtohub"></a>Connectivité de hub à hub

L’empreinte cloud d’une entreprise peut s’étendre sur plusieurs régions du cloud et, dans ce cas, il est primordial (du point de vue de la latence) d’accéder au cloud à partir d’une région la plus proche possible de son site physique et de ses utilisateurs. Un des principes clés de l’architecture du réseau de transit global consiste à permettre une connectivité inter-régions entre tous les points de terminaison des réseaux cloud et locaux. Cela signifie que le trafic à partir d’une branche connectée au cloud dans une région peut atteindre une autre branche ou un réseau virtuel dans une région différente grâce à la connectivité de hub à hub fournie par [Azure Global Network](https://azure.microsoft.com/global-infrastructure/global-network/).

![inter-régions](./media/virtual-wan-global-transit-network-architecture/figure3.png)

**Figure 3 : Connectivité inter-régions de Virtual WAN**

Quand plusieurs hubs sont activés dans un seul WAN virtuel, les hubs sont automatiquement interconnectés par des liaisons de hub à hub, ce qui fournit une connectivité globale entre les branches et les réseaux virtuels répartis dans des régions différentes. 

De plus, tous les hubs qui font partie du même WAN virtuel peuvent être associés à des stratégies de sécurité et d’accès différentes selon les régions. Pour plus d’informations, consultez [Contrôle de la sécurité et de la stratégie](#security), plus loin dans cet article.

## <a name="any-to-any-connectivity"></a><a name="anytoany"></a>Connexion universelle

L’architecture du réseau de transit global permet une connectivité universelle via des hubs WAN virtuels. Cette architecture élimine ou réduit le besoin d’un maillage complet ou partiel de la connectivité entre les rayons, qui sont plus complexes à créer et gérer. En outre, comparativement aux réseaux maillés, le contrôle du routage dans le réseau en étoile est plus facile à configurer et gérer.

La connectivité universelle (dans le contexte d’une architecture globale) permet à une entreprise d’interconnecter ses utilisateurs, succursales, centres de données, réseaux virtuels et applications répartis dans le monde entier via un ou plusieurs hubs de « transit ». Azure Virtual WAN agit en tant que système de transit global.

![universel](./media/virtual-wan-global-transit-network-architecture/figure4.png)

**Figure 4 : Chemins d’accès du trafic Virtual WAN**

Azure Virtual WAN prend en charge les chemins d’accès de connectivité de transit global suivants : Les lettres entre parenthèses renvoient à la Figure 4.

* Branche à réseau virtuel (a)
* Branche à branche (b)
  * ExpressRoute Global Reach et Virtual WAN
* Utilisateur distant à réseau virtuel (c)
* Utilisateur distant à branche (d)
* Réseau virtuel à réseau virtuel (e)
* Branche à hub - Hub à branche (f)
* Branche à hub - Hub à réseau virtuel (g)
* Réseau virtuel à hub - Hub à réseau virtuel (h)

### <a name="branch-to-vnet-a-and-branch-to-vnet-cross-region-g"></a>Branche à réseau virtuel (a) et Branche à réseau virtuel inter-régions (g)

Branche à réseau virtuel est le principal chemin d’accès pris en charge par Azure Virtual WAN. Ce chemin d’accès vous permet de connecter des branches aux charges de travail d’entreprise Azure IAAS déployées dans Azure VNets. Les branches peuvent être connectées au WAN virtuel via ExpressRoute ou un VPN site à site. Le trafic transite vers des réseaux virtuels qui sont connectés aux hubs WAN virtuels par le biais de connexions de réseau virtuel. Le [transit par passerelle](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity) explicite n’est pas requis pour Virtual WAN, car Virtual WAN active automatiquement le transit par passerelle vers le site de succursale. Pour plus d’informations sur la connexion d’un CPE SD-WAN à Virtual WAN, consultez [Partenaires Virtual WAN](virtual-wan-configure-automation-providers.md).

### <a name="expressroute-global-reach-and-virtual-wan"></a>ExpressRoute Global Reach et Virtual WAN

ExpressRoute vous permet de connecter vos réseaux locaux à Microsoft Cloud de manière privée et résiliente. Virtual WAN prend en charge les connexions de circuits ExpressRoute. La connexion d’un site de succursale à Virtual WAN avec ExpressRoute nécessite 1) un circuit Premium ou Standard et 2) que ce circuit se trouve dans un emplacement doté de Global Reach.

ExpressRoute Global Reach est une fonctionnalité de module complémentaire pour ExpressRoute. Avec Global Reach, vous pouvez associer des circuits ExpressRoute afin de constituer un réseau privé entre vos réseaux locaux. Les branches connectées à Azure Virtual WAN à l’aide d’ExpressRoute nécessitent ExpressRoute Global Reach pour communiquer entre elles.

Dans ce modèle, chaque branche connectée au hub WAN virtuel à l’aide d’ExpressRoute peut se connecter à des réseaux virtuels utilisant le chemin « Branche à réseau virtuel ». Le trafic de branche à branche ne transite pas par le hub, car ExpressRoute Global Reach offre un meilleur chemin d’accès sur Azure WAN.

### <a name="branch-to-branch-b-and-branch-to-branch-cross-region-f"></a>Branche à branche (b) et Branche à branche inter-régions (f)

Les branches peuvent être connectées à un hub WAN virtuel par des circuits ExpressRoute et/ou des connexions VPN de site à site. Vous pouvez connecter les branches au hub WAN virtuel qui se trouve dans la région la plus proche de la branche.

Cette option permet aux entreprises de tirer parti de la dorsale principale d’Azure pour connecter des branches. Toutefois, même si cette fonctionnalité est disponible, vous devez comparer les avantages de la connexion des branches via Azure Virtual WAN à l’utilisation d’un réseau WAN privé.  

> [!NOTE]
> La désactivation de la connectivité branche à branche dans Virtual WAN - Le service Virtual WAN peut être configuré pour désactiver la connectivité branche à branche. Cette configuration bloque la propagation des routes entre les sites connectés par VPN (S2S et P2S) et par ExpressRoute. Cette configuration n’affecte pas la connectivité et la propagation de routes de branche à réseau virtuel et de réseau virtuel à réseau virtuel. Pour configurer ce paramètre à l’aide du portail Azure : Dans le menu Configuration de Virtual WAN, choisissez Paramètre : Branche à branche - Désactivé. 

### <a name="remote-user-to-vnet-c"></a>Utilisateur distant à réseau virtuel (c)

Vous pouvez activer l’accès à distance direct et sécurisé à Azure à l’aide d’une connexion point à site entre un utilisateur distant et un WAN virtuel. Les utilisateurs distants d’entreprise n’ont plus besoin d’accéder au cloud en utilisant un VPN d’entreprise.

### <a name="remote-user-to-branch-d"></a>Utilisateur distant à branche (d)

Le chemin d’accès utilisateur distant à branche permet aux utilisateurs distants qui utilisent une connexion point à site sur Azure d’accéder aux charges de travail locales et aux applications en transit à travers le cloud. Ce chemin d’accès offre aux utilisateurs distants la flexibilité d’accéder aux charges de travail qui sont déployées à la fois dans Azure et en local. Les entreprises peuvent activer un service central d’accès distant sécurisé basé sur le cloud dans Azure Virtual WAN.

### <a name="vnet-to-vnet-transit-e-and-vnet-to-vnet-cross-region-h"></a>Réseau virtuel à réseau virtuel (e) et Réseau virtuel à réseau virtuel inter-régions (h)

Le transit de réseau virtuel à réseau virtuel permet de relier des réseaux virtuels entre eux afin d’interconnecter les applications multiniveau qui sont distribuées sur plusieurs réseaux virtuels. Vous pouvez également connecter des réseaux virtuels entre eux par le biais du peering de réseaux virtuels. Cette approche est plus adaptée dans certains scénarios où le transit via le hub VWAN n’est pas nécessaire.


## <a name="force-tunneling-and-default-route-in-azure-virtual-wan"></a><a name="DefaultRoute"></a>Forcer le tunneling et la route par défaut dans Azure Virtual WAN

Le tunneling forcé peut être activé en configurant l’option de route par défaut sur une connexion VPN, ExpressRoute ou réseau virtuel dans Virtual WAN.

Le hub virtuel propage l’itinéraire par défaut appris à une connexion de réseau virtuel/VPN site à site/ExpressRoute si l’indicateur par défaut est « Activé » sur la connexion. 

Cet indicateur est visible lorsque l’utilisateur modifie une connexion de réseau virtuel, une connexion VPN ou une connexion ExpressRoute. Par défaut, cet indicateur est désactivé lorsqu’un site ou un circuit ExpressRoute est connecté à un hub. Il est activé par défaut lorsqu’une connexion de réseau virtuel est ajoutée pour connecter un réseau virtuel à un hub virtuel. L’itinéraire par défaut ne provient pas du hub Virtual WAN ; il est propagé s’il est déjà appris par le hub Virtual WAN suite au déploiement d’un pare-feu dans le hub, ou si le tunneling forcé est activé sur un autre site connecté.

## <a name="security-and-policy-control"></a><a name="security"></a>Contrôle de la sécurité et de la stratégie

Les hubs Azure Virtual WAN interconnectent tous les points de terminaison réseau sur le réseau hybride et peuvent potentiellement voir l’ensemble du trafic sur le réseau de transit. Il est possible de changer des hubs Virtual WAN en hubs virtuels sécurisés en déployant le Pare-feu Azure à l’intérieur des hubs VWAN pour contrôler la stratégie, l’accès et la sécurité dans le cloud. L’orchestration des Pare-feu Azure dans les hubs WAN virtuels peut être effectuée par Azure Firewall Manager.

[Azure Firewall Manager](../firewall-manager/index.yml) fournit les fonctionnalités nécessaires pour gérer et adapter la sécurité des réseaux de transit global. Avec Azure Firewall Manager, vous pouvez gérer de manière centralisée le routage, les stratégies globales, les services de sécurité Internet avancée par le biais d’un tiers et du Pare-feu Azure.

![hub virtuel sécurisé avec le Pare-feu Azure](./media/virtual-wan-global-transit-network-architecture/figure5.png)

**Figure 5 : Hub virtuel sécurisé avec le Pare-feu Azure**

> [!NOTE]
> L’inter-hub avec pare-feu n’est actuellement pas pris en charge. Le trafic entre les hubs passera directement en ignorant le pare-feu Azure dans chaque hub.

Le Pare-feu Azure dans le hub WAN virtuel prend en charge les chemins de connectivité suivants pour le transit de sécurisé global. Les lettres entre parenthèses renvoient à la Figure 5.

* Transit sécurisé de réseau virtuel à réseau virtuel (e)
* Réseau virtuel à Internet ou un service de sécurité tiers (i)
* Branche à Internet ou un service de sécurité tiers (j)

### <a name="vnet-to-vnet-secured-transit-e"></a>Transit sécurisé de réseau virtuel à réseau virtuel (e)

Le transit sécurisé de réseau virtuel à réseau virtuel permet d’interconnecter des réseaux virtuels par le biais du Pare-feu Azure dans le hub WAN virtuel.

### <a name="vnet-to-internet-or-third-party-security-service-i"></a>Réseau virtuel à Internet ou un service de sécurité tiers (i)

Le scénario Réseau virtuel à Internet permet de se connecter à Internet par le biais du Pare-feu Azure dans le hub Virtual WAN. Le trafic vers Internet via les services de sécurité tiers pris en charge n’est pas transmis via le Pare-feu Azure. Vous pouvez configurer le chemin d’accès du réseau virtuel à Internet par le biais d’un service de sécurité tiers pris en charge avec Azure Firewall Manager.  

### <a name="branch-to-internet-or-third-party-security-service-j"></a>Branche à Internet ou un service de sécurité tiers (j)
Le scénario Branche à Internet permet aux branches de se connecter à Internet via le Pare-feu Azure dans le hub Virtual WAN. Le trafic vers Internet via les services de sécurité tiers pris en charge n’est pas transmis via le Pare-feu Azure. Vous pouvez configurer le chemin d’accès Branche à Internet par le biais d’un service de sécurité tiers pris en charge avec Azure Firewall Manager. 

### <a name="how-do-i-enable-default-route-00000-in-a-secured-virtual-hub"></a>Comment faire activer l’itinéraire par défaut (0.0.0.0/0) dans un hub virtuel sécurisé

Le Pare-feu Azure déployé dans un hub Virtual WAN (hub virtuel sécurisé) peut être configuré en tant que routeur par défaut vers Internet ou comme fournisseur de sécurité approuvé pour toutes les branches (connectées par VPN ou Express Route), les réseaux virtuels des rayons et les utilisateurs (connectés via un VPN P2S). Cette configuration doit être effectuée à l’aide d’Azure Firewall Manager.  Consultez Acheminer le trafic vers votre hub pour configurer tout le trafic venant des branches (y compris les utilisateurs) et des réseaux virtuels vers Internet via le Pare-feu Azure. 

Il s’agit d’une configuration en deux étapes :

1. Configurer le routage du trafic Internet à l’aide du menu des paramètres de route du hub virtuel sécurisé. Configurez les réseaux virtuels et les branches qui peuvent envoyer le trafic vers Internet via le pare-feu.

2. Configurez les connexions (réseau virtuel et branche) qui peuvent acheminer le trafic vers Internet (0.0.0.0/0) via le Pare-feu Azure dans le hub ou le fournisseur de sécurité approuvé. Cette étape permet de s’assurer que la route par défaut est propagée vers les branches sélectionnées et les réseaux virtuels reliés au hub Virtual WAN via les connexions. 

### <a name="force-tunneling-traffic-to-on-premises-firewall-in-a-secured-virtual-hub"></a>Forcer le tunneling du trafic vers le pare-feu local dans un hub virtuel sécurisé

S’il existe déjà une route par défaut apprise (via BGP) par le hub virtuel depuis une des branches (VPN ou sites ER), cette route par défaut est remplacée par la route par défaut apprise depuis le paramètre Azure Firewall Manager. Dans ce cas, tout le trafic qui entre dans le hub en provenance de réseaux virtuels et de branches et destiné à Internet est acheminé vers le Pare-feu Azure ou le fournisseur de sécurité approuvé.

> [!NOTE]
> Il n’existe actuellement aucune option permettant de sélectionner un pare-feu local ou le Pare-feu Azure (et un fournisseur de sécurité approuvé) pour le trafic destiné à Internet et provenant des réseaux virtuels, des branches ou des utilisateurs. La route par défaut apprise par le biais du paramètre Azure Firewall Manager est toujours préférée par rapport à la route par défaut apprise de l’une des branches.


## <a name="next-steps"></a>Étapes suivantes

Créez une connexion à l’aide de Virtual WAN et déployez le Pare-feu Azure dans les hubs VWAN.

* [Connexions de site à site à l’aide de Virtual WAN](virtual-wan-site-to-site-portal.md)
* [Connexions ExpressRoute à l’aide du Virtual WAN](virtual-wan-expressroute-portal.md)
* [Azure Firewall Manager pour déployer le Pare-feu Azure dans un WAN virtuel](../firewall-manager/index.yml)