---
title: Architecture réseau v1
description: Présentation de l’architecture de la topologie réseau des environnements App Service. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: 13d03a37-1fe2-4e3e-9d57-46dfb330ba52
ms.topic: article
ms.date: 10/04/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: b1b866f3be789c59eea38c5c22b5557d557440be
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84701811"
---
# <a name="network-architecture-overview-of-app-service-environments"></a>Présentation de l'architecture réseau des environnements App Service
Les environnements App Service sont toujours créés dans un sous-réseau d’un [réseau virtuel][virtualnetwork]. Les applications exécutées dans un environnement App Service peuvent communiquer avec des points de terminaison privés au sein de la même topologie de réseau virtuel.  Dans la mesure où les clients peuvent verrouiller des parties de leur infrastructure de réseau virtuel, il est important de comprendre les types de flux de communication réseau qui se produisent avec un environnement App Service.

## <a name="general-network-flow"></a>Flux réseau général
Lorsqu’un environnement App Service (ASE) utilise une adresse IP virtuelle publique (VIP) pour les applications, tout le trafic entrant arrive sur cette adresse IP virtuelle publique.  Cela inclut le trafic HTTP et HTTPS pour les applications, ainsi que le reste du trafic pour les opérations de gestion Azure, les fonctionnalités de débogage à distance et FTP.  Pour obtenir une liste complète des ports spécifiques (obligatoires et facultatifs) disponibles sur l'adresse VIP publique, consultez l’article sur le [contrôle du trafic entrant][controllinginboundtraffic] dans un environnement App Service. 

Les environnements App Service prennent également en charge les applications en cours d’exécution et uniquement liées à une adresse interne de réseau virtuel, également appelée adresse ILB (équilibrage de charge interne).  Sur un ASE avec équilibrage de charge interne, le trafic HTTP et HTTPS des applications et des appels de débogage à distance arrive sur l’adresse d’équilibrage de charge interne.  Dans la plupart des configurations d’équilibrage de charge interne ASE, le trafic FTP/FTPS arrive également sur l’adresse d’équilibrage de charge interne.  Toutefois les opérations de gestion Azure continuent de transiter par les ports 454/455 sur l’adresse IP virtuelle publique d’un ASE avec équilibrage de charge interne.

Le diagramme suivant montre une vue d’ensemble des différents flux réseau entrants et sortants d’un environnement App Service où les applications sont liées à une adresse IP virtuelle publique :

![Flux de réseau généraux][GeneralNetworkFlows]

Un environnement App Service peut communiquer avec plusieurs points de terminaison client privés.  Par exemple, les applications exécutées dans l'environnement App Service peuvent se connecter au(x) serveur(s) de base de données en cours d'exécution sur des machines virtuelles IaaS dans la même topologie de réseau virtuel.

> [!IMPORTANT]
> D’après le schéma du réseau, les « Other Computer Resources » (« Autres ressources de l’ordinateur ») sont déployées dans un sous-réseau différent de l’environnement App Service. Déployer des ressources dans le même sous-réseau avec l’ASE bloque la connectivité depuis l’ASE à ces ressources (à l’exception de certains routages intra-ASE). Il est préférable de les déployer dans un autre sous-réseau (dans le même réseau virtuel). L’environnement App Service est alors en mesure de se connecter. Aucune configuration supplémentaire n’est nécessaire.
> 
> 

Les environnements App Service communiquent également avec les ressources BD SQL et stockage Azure nécessaires pour la gestion et l'exploitation d'un environnement App Service.  Certaines des ressources de stockage et Sql avec lesquelles un environnement App Service communique se trouvent dans la même région que l'environnement App Service, tandis que d'autres sont situées dans des régions Azure distantes.  Par conséquent, une connectivité sortante à Internet est toujours requise pour qu'un environnement App Service fonctionne correctement. 

Étant donné qu'un environnement App Service est déployé dans un sous-réseau, vous pouvez utiliser des groupes de sécurité réseau pour contrôler le trafic entrant vers le sous-réseau.  Pour plus d’informations sur la façon de contrôler le trafic entrant vers un environnement App Service, consultez l’[article][controllinginboundtraffic] suivant.

Pour plus d’informations sur la façon d’autoriser la connectivité Internet sortante à partir d’un environnement App Service, consultez l’article suivant sur l’utilisation d’[ExpressRoute][ExpressRoute].  La même approche que celle décrite dans cet article s'applique avec une connectivité de site à site et l'utilisation du tunneling forcé.

## <a name="outbound-network-addresses"></a>Adresses réseau sortantes
Lorsqu'un environnement App Service effectue des appels sortants, une adresse IP est toujours associée aux appels sortants.  L'adresse IP spécifique utilisée varie suivant si le point de terminaison appelé se trouve dans la topologie de réseau virtuel ou en dehors de la topologie de réseau virtuel.

Si le point de terminaison appelé est **en dehors** de la topologie de réseau virtuel, alors l'adresse sortante (également appelé adresse NAT sortante) utilisée est l'adresse VIP publique de l'environnement App Service.  Cette adresse se trouve dans l'interface utilisateur du portail de l'environnement App Service, dans le panneau Propriétés.

![Adresse IP sortante][OutboundIPAddress]

Il est également possible de déterminer cette adresse pour les ASE disposant uniquement d’une adresse VIP publique en créant une application dans l'environnement App Service, puis en effectuant une opération *nslookup* sur l'adresse de l'application. L'adresse IP obtenue est à la fois l'adresse IP virtuelle publique et l'adresse NAT sortante de l'environnement App Service.

Si le point de terminaison appelé est **dans** la topologie de réseau virtuel, l'adresse sortante de l'application appelante sera l'adresse IP interne de la ressource de calcul individuelle exécutant l'application.  Toutefois, il n'existe pas de mappage persistant des adresses IP internes du réseau virtuel aux applications.  Les applications peuvent se déplacer entre différentes ressources de calcul et le pool de ressources de calcul disponibles dans un environnement App Service peut changer en raison d'opérations de mise à l'échelle.

Toutefois, dans la mesure où un environnement App Service est toujours situé dans un sous-réseau, vous pouvez être sûr que l'adresse IP interne d'une ressource de calcul exécutant une application sera toujours comprise dans la plage CIDR du sous-réseau.  Par conséquent, si des listes de contrôle d'accès précises ou des groupes de sécurité réseau sont utilisés pour sécuriser l'accès à d'autres points de terminaison dans le réseau virtuel, l'accès doit être accordé à la plage du sous-réseau contenant l'environnement App Service.

Le diagramme suivant illustre ces concepts plus en détail :

![Adresses réseau sortantes][OutboundNetworkAddresses]

Dans le diagramme ci-dessus :

* Étant donné que l'adresse VIP publique de l'environnement App Service est 192.23.1.2, cela correspond à l'adresse IP sortante utilisée lors des appels aux points de terminaison « Internet ».
* La plage CIDR du sous-réseau conteneur pour l'environnement App Service est 10.0.1.0/26.  Les autres points de terminaison dans la même infrastructure de réseau virtuel verront les appels provenant d'applications comme émanant de quelque part dans cette plage d'adresses.

## <a name="calls-between-app-service-environments"></a>Appels entre les environnements App Service
Un scénario plus complexe peut se produire si vous déployez plusieurs environnements App Service dans le même réseau virtuel et effectuez des appels sortants à partir d’un environnement App Service vers un autre environnement App Service.  Ces types d’appels entre les environnements App Service seront également traités comme des appels « Internet ».

Le diagramme suivant illustre un exemple d’architecture en couches avec des applications dans un environnement App Service (par exemple,  des applications web de « porte d’entrée ») appelant des applications dans un environnement App Service (par exemple, des applications d'API internes principales qui ne sont pas destinées à être accessibles via Internet). 

![Appels entre les environnements App Service][CallsBetweenAppServiceEnvironments] 

Dans l'exemple ci-dessus, l'environnement App Service « ASE One » possède l’adresse IP sortante 192.23.1.2.  Si une application en cours d'exécution sur cet environnement App Service effectue un appel sortant vers une application s'exécutant sur un second environnement App Service (« ASE Two ») situé sur le même réseau virtuel, l'appel sortant sera traité comme un appel « Internet ».  Par conséquent, le trafic réseau arrivant sur le deuxième environnement App Service s’affichera comme un appel provenant de l’adresse 192.23.1.2 (et non pas la plage d'adresses de sous-réseau du premier environnement App Service).

Même si les appels entre les différents environnements App Service sont traités comme des appels « Internet », lorsque les deux environnements App Service se trouvent dans la même région Azure, le trafic réseau reste sur le réseau Azure régional et ne circule pas physiquement sur le réseau Internet public.  Par conséquent, vous pouvez utiliser un groupe de sécurité réseau sur le sous-réseau du deuxième environnement App Service pour autoriser uniquement les appels entrants à partir du premier environnement App Service (dont l’adresse IP sortante est 192.23.1.2), afin de garantir une communication sécurisée entre les environnements App Service.

## <a name="additional-links-and-information"></a>Informations et liens supplémentaires
Vous trouverez plus d’informations sur les ports entrants utilisés par les environnements App Service et l’utilisation de groupes de sécurité réseau pour contrôler le trafic entrant [ici][controllinginboundtraffic].

Cet [article][ExpressRoute] contient des informations sur l’utilisation d’itinéraires définis par l’utilisateur pour accorder un accès Internet sortant à des environnements App Service. 

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/services/virtual-network/
[controllinginboundtraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[ExpressRoute]:  app-service-app-service-environment-network-configuration-expressroute.md

<!-- IMAGES -->
[GeneralNetworkFlows]: ./media/app-service-app-service-environment-network-architecture-overview/NetworkOverview-1.png
[OutboundIPAddress]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundIPAddress-1.png
[OutboundNetworkAddresses]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundNetworkAddresses-1.png
[CallsBetweenAppServiceEnvironments]: ./media/app-service-app-service-environment-network-architecture-overview/CallsBetweenEnvironments-1.png

