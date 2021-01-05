---
title: Travailler à distance à l’aide des services de mise en réseau Azure
description: Cette page décrit comment vous pouvez utiliser les services de mise en réseau Azure qui sont disponibles pour activer l’utilisation à distance et comment atténuer les problèmes de trafic résultant d’un nombre accru de personnes travaillant à distance.
services: networking
author: rambk
ms.service: virtual-network
ms.topic: article
ms.date: 03/26/2020
ms.author: rambala
ms.openlocfilehash: e0e5806ec59cd2d75111db213d8511488d043eec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80982852"
---
# <a name="working-remotely-using-azure-networking-services"></a>Travailler à distance à l’aide des services de mise en réseau Azure

>[!NOTE]
> Cet article décrit comment vous pouvez tirer parti des services de mise en réseau d’Azure, des réseaux Microsoft et de l’écosystème de partenaires Azure pour travailler à distance et atténuer les problèmes de réseau auxquels vous pourriez faire face dans le contexte de l’épidémie de COVID-19.

Cet article décrit les options disponibles pour les organisations afin de configurer l’accès à distance à leurs utilisateurs ou de compléter leurs solutions existantes avec une capacité supplémentaire pendant les périodes avec des pics d’utilisation. Les architectes réseau sont confrontés aux défis suivants :

- Répondre à une augmentation de l’utilisation du réseau.
- Fournir une connectivité fiable et sécurisée à un plus grand nombre d’employés de leur entreprise et de leurs clients.
- Fournir la connectivité à des sites distants dans le monde entier.

Tous les réseaux (par exemple, les réseaux étendus privés et les réseaux de base d’entreprise) ne subissent pas de congestion lors des pics de charge de travail à distance. Les goulots d’étranglement sont généralement signalés uniquement dans les réseaux à haut débit et les passerelles VPN des réseaux locaux des entreprises.

Les planificateurs de réseau peuvent soulager les goulots d’étranglement et réduire la congestion du réseau en gardant à l’esprit que les différents types de trafic nécessitent des priorités de traitement réseau différentes et une redirection/distribution de charge intelligente. Par exemple, le trafic de télémédecine en temps réel ou d’interaction médecin-patient est de haute importance et sensible aux retards/instabilités. En revanche, la réplication du même trafic entre stockages n’est pas sensible au délai. Le premier type de trafic précédent doit être acheminé via le chemin réseau le plus optimal avec une qualité de service supérieure. Il est en revanche acceptable d’acheminer le second type de trafic via un itinéraire non optimal.



## <a name="sharing-our-best-practices---azure-network-is-designed-for-elasticity-and-high-availability"></a>Partage de nos meilleures pratiques : le réseau Azure est conçu pour l’élasticité et la haute disponibilité

Azure est conçu pour résister à des modifications soudaines de l’utilisation des ressources et peut s’avérer très utile pendant les périodes avec des pics d’utilisation. En outre, Microsoft gère et exploite un des réseaux les plus étendus au monde. Le réseau de Microsoft a été conçu pour une haute disponibilité pouvant résister à différents types de défaillances : d’un seul élément réseau à la panne d’une région entière.

Le réseau Microsoft est conçu pour répondre aux exigences et offrir des performances optimales pour différents types de trafic réseau, notamment le trafic multimédia sensible aux retards pour Skype et Teams, CDN, l’analyse de Big Data en temps réel, le stockage Azure, Bing et Xbox. Pour fournir des performances optimales pour différents types de trafic, le réseau Microsoft attire tout le trafic qui est destiné à ou qui souhaite transiter via ses ressources le plus près possible de l’origine du trafic.

>[!NOTE] 
>L’utilisation des fonctionnalités de mise en réseau Azure décrites ci-dessous s’appuie sur le comportement d’attraction du trafic du réseau global Microsoft pour fournir une meilleure expérience réseau aux clients. Le comportement d’attraction du trafic du réseau Microsoft permet de décharger le trafic le plus rapidement possible à partir des premiers/derniers réseaux qui peuvent rencontrer une congestion pendant les périodes avec des pics d’utilisation.
>

## <a name="enable-employees-to-work-remotely"></a>Permettre aux employés de travailler à distance

La passerelle VPN Azure prend en charge les connexions VPN de point à site (P2S) et de site à site (S2S). À l’aide de la passerelle VPN Azure, vous pouvez mettre à l’échelle les connexions de vos employés pour qu’ils accèdent en toute sécurité à vos ressources Azure déployées et à vos ressources locales. Pour plus d’informations, consultez [Comment permettre aux utilisateurs de travailler à distance](../vpn-gateway/work-remotely-support.md). 

Si vous utilisez le protocole SSTP (Secure Sockets Tunneling Protocol), le nombre de connexions simultanées est limité à 128. Pour obtenir un plus grand nombre de connexions, nous vous suggérons de passer à OpenVPN ou IKEv2. Pour plus d’informations, consultez [Transition vers le protocole OpenVPN ou IKEv2 à partir de SSTP](../vpn-gateway/ikev2-openvpn-from-sstp.md
).

Pour accéder à vos ressources déployées dans Azure, les développeurs distants peuvent utiliser la solution Azure Bastion au lieu d’une connexion VPN pour obtenir un accès sécurisé à l’interpréteur de commandes (RDP ou SSH) sans nécessiter d’adresses IP publiques sur les machines virtuelles en cours d’accès. Pour plus d’informations, consultez [Travailler à distance à l’aide d’Azure Bastion](../bastion/work-remotely-support.md).

Pour l’agrégation d’une connexion VPN à grande échelle, pour prendre en charge des connexions entre ressources situées dans des emplacements globaux locaux différents, dans différents réseaux virtuels hub et spoke et pour optimiser l’utilisation de plusieurs réseaux haut débit à distance, vous pouvez utiliser Azure Virtual WAN. Pour plus d’informations, consultez [Vous avez du mal à répondre aux besoins de votre entreprise ? C’est là qu’Azure Virtual WAN peut vous aider](../virtual-wan/work-remotely-support.md).

Une autre façon de prendre en charge votre personnel à distance consiste à déployer une infrastructure de bureau virtuel (VDI) hébergée dans votre réseau virtuel Azure, et sécurisée avec un pare-feu Azure. Par exemple, Windows Virtual Desktop (WVD) est un service de virtualisation de bureau et d’application qui s’exécute dans Azure. Avec Windows Virtual Desktop, vous pouvez configurer un environnement évolutif et flexible dans votre abonnement Azure sans avoir à exécuter d’autres serveurs de passerelle. Vous êtes uniquement responsable des machines virtuelles WVD de votre réseau virtuel. Pour plus d’informations, consultez [Prise en charge du travail à distance par le pare-feu Azure](../firewall/remote-work-support.md). 

Azure propose également un ensemble complet de partenaires d’écosystème. Les appliances virtuelles réseau de nos partenaires sur Azure peuvent également vous aider à mettre à l’échelle la connectivité VPN. Pour plus d’informations, consultez [Considérations relatives aux appliances virtuelles réseau (NVA) pour le travail à distance](../vpn-gateway/nva-work-remotely-support.md).

## <a name="extend-employees-connection-to-access-globally-distributed-resources"></a>Étendre la connexion des employés pour qu’ils puissent accéder aux ressources distribuées à l’échelle mondiale

Les services Azure suivants peuvent permettre à vos employés d’accéder à vos ressources distribuées à l’échelle mondiale. Vos ressources peuvent se trouver dans l’une des régions Azure, sur des réseaux locaux ou même dans d’autres clouds publics ou privés. 

- **Peering de réseaux virtuels Azure** : Si vous déployez vos ressources dans plusieurs régions Azure et/ou si vous regroupez la connectivité des employés travaillant à distance à l’aide de plusieurs réseaux virtuels, vous pouvez établir une connectivité entre les différents réseaux virtuels Azure à l’aide du peering de réseaux virtuels. Pour en savoir plus, consultez [Peering de réseaux virtuels][VNet-peer].

- **Solution basée sur un VPN Azure** : Pour vos employés distants connectés à Azure via un VPN P2S ou S2S, vous pouvez activer l’accès aux réseaux locaux en configurant le VPN S2S entre vos réseaux locaux et la passerelle VPN Azure. Pour plus d'informations, consultez [Créer une connexion de site à site][S2S].

- **ExpressRoute** : À l’aide du peering privé ExpressRoute, vous pouvez activer la connectivité privée entre vos déploiements Azure et une infrastructure locale ou votre infrastructure dans une installation de colocalisation. ExpressRoute, via le peering Microsoft, permet également d’accéder aux points de terminaison publics de Microsoft à partir de votre réseau local. Les connexions ExpressRoute ne sont pas établies par le biais de l'Internet public. Elles offrent une connectivité sûre, de la fiabilité et plus de débit, ainsi que des latences inférieures et cohérentes par rapport aux connexions classiques sur Internet. Pour plus d’informations, voir [Appairage ExpressRoute][ExR]. En tirant parti de votre fournisseur de réseau existant qui fait déjà partie de notre [écosystème de partenaires ExpressRoute][ExR-eco], vous pouvez réduire le temps nécessaire pour obtenir des connexions à large bande passante à Microsoft.  Avec [ExpressRoute Direct][ExR-D], vous pouvez connecter directement votre réseau local à la dorsale principale Microsoft. ExpressRoute Direct offre deux options de débit de ligne différentes avec 10 Gbit/s ou 100 Gbit/s. 

- **Azure Virtual WAN** : Azure Virtual WAN permet une interopérabilité transparente entre vos connexions VPN et les circuits ExpressRoute. Comme mentionné précédemment, Azure Virtual WAN prend également en charge les connexions entre n’importe quelles ressources situées dans des emplacements locaux différents à l’échelle globale, dans différents réseaux virtuels hub et spoke régionaux

## <a name="scale-customer-connectivity-to-frontend-resources"></a>Mettre à l’échelle la connectivité client aux ressources front-end

Pendant les périodes où davantage de personnes sont en ligne, de nombreux sites web d’entreprise notent une augmentation du trafic client. Azure Application Gateway peut vous aider à gérer cette charge de travail front-end accrue. Pour plus d’informations, consultez [Prendre en charge le trafic élevé Application Gateway](../application-gateway/high-traffic-support.md).

## <a name="microsoft-support-for-multi-cloud-traffic"></a>Prise en charge par Microsoft du trafic multi-cloud

Pour vos déploiements dans d’autres clouds publics, Microsoft peut fournir une connectivité globale. Azure Virtual WAN, VPN ou ExpressRoute peuvent vous aider à cet égard. Pour étendre la connectivité d’Azure à d’autres clouds, vous pouvez configurer un VPN S2S entre les deux clouds. Vous pouvez également établir une connectivité d’Azure vers d’autres clouds publics à l’aide d’ExpressRoute. Le cloud Oracle fait partie de l’écosystème de partenaires ExpressRoute. Vous pouvez [configurer une interconnexion directe entre Azure et Oracle Cloud Infrastructure][Az-OCI]. La plupart des fournisseurs de services qui font partie de l’écosystème de partenaires ExpressRoute offrent également une connectivité privée à d’autres clouds publics. En tirant parti de ces fournisseurs de services, vous pouvez établir une connectivité privée entre vos déploiements dans Azure et d’autres clouds via ExpressRoute.

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants expliquent comment différentes fonctionnalités de mise en réseau Azure peuvent être utilisées pour mettre à l’échelle les utilisateurs pour travailler à distance :

| **Article** | **Description** |
| --- | --- |
| [Comment permettre aux utilisateurs de travailler à distance](../vpn-gateway/work-remotely-support.md) | Passez en revue les options disponibles pour configurer l’accès à distance pour les utilisateurs ou compléter leurs solutions existantes avec des capacités supplémentaires pour votre organisation.|
| [Vous avez du mal à répondre aux besoins de votre entreprise ? C’est là qu’Azure Virtual WAN peut vous aider](../virtual-wan/work-remotely-support.md) | Utilisez Azure Virtual WAN pour répondre aux besoins de votre organisation en termes de connectivité à distance.|
| [Prendre en charge le trafic élevé Application Gateway](../application-gateway/high-traffic-support.md) | Utilisez Application Gateway avec le pare-feu d’applications Web (WAF) pour disposer d’un moyen scalable et sécurisé de gérer le trafic vers vos applications web. |
| [Considérations relatives aux appliances virtuelles réseau (NVA) pour le travail à distance](../vpn-gateway/nva-work-remotely-support.md)|Consultez les instructions relatives à l’utilisation d’appliances virtuelles réseau dans Azure pour fournir des solutions d’accès à distance. |
| [Passer au protocole OpenVPN ou à IKEv2 à partir de SSTP](https://go.microsoft.com/fwlink/?linkid=2124112) | Surmontez la limite de 128 connexions simultanées de SSTP en passant au protocole OpenVPN ou IKEv2.|
| [Travail à distance à l’aide d’Azure Bastion](../bastion/work-remotely-support.md) | Offre une connectivité RDP/SSH sans interruption et sécurisée aux machines virtuelles au sein du réseau virtuel Azure, directement dans le portail Azure, sans utiliser d’adresse IP publique. |
| [Utilisation d’Azure ExpressRoute pour créer une connectivité hybride pour prendre en charge les utilisateurs distants](../expressroute/work-remotely-support.md) | Utilisez ExpressRoute pour la connectivité hybride afin de permettre aux utilisateurs de votre organisation de travailler à distance.|
| [Prise en charge du travail à distance par le pare-feu Azure](../firewall/remote-work-support.md)|Protégez vos ressources de réseau virtuel Azure à l’aide du Pare-feu Azure. |

<!--Link References-->
[VNet-peer]: https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview
[S2S]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[ExR]: https://docs.microsoft.com/azure/expressroute/expressroute-introduction
[ExR-eco]: https://docs.microsoft.com/azure/expressroute/expressroute-locations
[ExR-D]: https://docs.microsoft.com/azure/expressroute/expressroute-erdirect-about
[Az-OCI]: https://docs.microsoft.com/azure/virtual-machines/workloads/oracle/configure-azure-oci-networking
