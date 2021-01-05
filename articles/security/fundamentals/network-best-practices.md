---
title: Meilleures pratiques pour la sécurité réseau – Microsoft Azure
description: Cet article détaille les meilleures pratiques en matière de sécurité réseau, sur la base de capacités Azure intégrées.
services: security
author: TerryLanfear
manager: barbkess
editor: TomShinder
ms.assetid: 7f6aa45f-138f-4fde-a611-aaf7e8fe56d1
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/02/2019
ms.author: TomSh
ms.openlocfilehash: 49c4f7888150446316a04710bc5de5e618cd66eb
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413307"
---
# <a name="azure-best-practices-for-network-security"></a>Meilleures pratiques Azure pour la sécurité réseau
Cet article présente l’ensemble des meilleures pratiques Azure pour améliorer votre sécurité réseau. Ces meilleures pratiques sont issues de notre expérience dans le domaine de la mise en réseau Azure, mais également de celle des clients, comme vous.

Cet article détaille les points suivants pour chaque bonne pratique :

* Nature de la bonne pratique
* Raison pour laquelle activer cette bonne pratique
* Conséquence possible en cas de non-utilisation de la bonne pratique
* Alternatives possibles à la meilleure pratique
* Comment apprendre à utiliser la bonne pratique

Ces meilleures pratiques reposent sur un consensus, ainsi que sur les ensembles de possibilités et de fonctionnalités de la plateforme Azure disponibles au moment de la rédaction de cet article. Les opinions et avis évoluent au fil du temps ; cet article sera régulièrement mis à jour de manière à tenir compte de ces changements.

## <a name="use-strong-network-controls"></a>Utiliser des contrôles réseau renforcés
Vous pouvez connecter des [machines virtuelles Azure](https://azure.microsoft.com/services/virtual-machines/) et des appliances à d’autres appareils en réseau, en les plaçant sur des [réseaux virtuels Azure](../../virtual-network/index.yml). Autrement dit, vous pouvez connecter des cartes d’interface réseau virtuel à un réseau virtuel afin de permettre des communications TCP/IP entre les appareils en réseau. Les machines virtuelles connectées à un réseau virtuel Azure peuvent se connecter à des appareils se trouvant sur le même réseau virtuel, sur des réseaux virtuels différents, sur Internet ou sur vos réseaux locaux.

Lors de la planification de votre réseau et de la sécurité de votre réseau, nous vous conseillons de centraliser :

- La gestion des fonctionnalités du réseau principal, comme ExpressRoute, le provisionnement du réseau et des sous-réseaux virtuels et l’adressage IP.
- La gouvernance des éléments de sécurité réseau, telles que les fonctionnalités d’appliance virtuelle du réseau, par exemple ExpressRoute, le provisionnement du réseau et des sous-réseaux virtuels et l’adressage IP.

Si vous utilisez un ensemble commun d’outils de gestion pour superviser votre réseau et la sécurité de votre réseau, vous obtenez une bonne visibilité dans les deux. Une stratégie de sécurité simple et unifiée réduit les erreurs, car elle facilite la compréhension humaine et la fiabilité de l’automatisation.

## <a name="logically-segment-subnets"></a>Segmentation logique des sous-réseaux
Les réseaux virtuels Azure sont similaires aux réseaux LAN de votre réseau local. Un réseau virtuel Azure repose sur un concept, celui de la création d’un réseau basé sur un espace d’adressage IP privé unique, au sein duquel vous pouvez placer toutes vos machines virtuelles Azure. Les espaces d’adressage IP privés disponibles se trouvent dans les plages des classes A (10.0.0.0/8), B (172.16.0.0/12) et C (192.168.0.0/16).

Meilleures pratiques pour segmenter logiquement les sous-réseaux :

**Bonne pratique**  : N’attribuez pas de règle d’autorisation avec de larges plages (autorisez, par exemple, de 0.0.0.0 à 255.255.255.255).  
**Détail**  : Assurez-vous que les procédures de résolution des problèmes découragent ou interdisent la configuration de ces types de règles. Ces règles d’autorisation induisent un sentiment de sécurité erroné : elles sont fréquemment trouvées et exploitées par les équipes rouges.

**Bonne pratique**  : Segmentez l’espace d’adressage plus volumineux en sous-réseaux.   
**Détail**  : Pour créer vos sous-réseaux, utilisez les principes de création de sous-réseau reposant sur [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

**Bonne pratique**  : Créez des contrôles d’accès réseau entre les sous-réseaux. Le routage entre les sous-réseaux se fait automatiquement. Il est donc inutile de configurer manuellement des tables de routage. Par défaut, il n’y a aucun contrôle d’accès réseau entre les sous-réseaux que vous créez sur un réseau virtuel Azure.   
**Détail**  : Utilisez un [groupe de sécurité réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md) pour vous protéger contre le trafic non sollicité dans les sous-réseaux Azure. Un groupe de sécurité réseau est un simple appareil d’inspection des paquets, avec état, qui applique la méthode basée sur les 5 tuples (adresse IP source, port source, adresse IP de destination, port de destination et protocole de couche 4) pour créer des règles visant à autoriser ou refuser le trafic réseau. Vous pouvez autoriser ou refuser le trafic vers et depuis une ou plusieurs adresses IP, ou entre des sous-réseaux entiers, dans les deux directions.

Lorsque vous utilisez des groupes de sécurité réseau pour le contrôle d’accès réseau entre les sous-réseaux, vous pouvez placer des ressources appartenant au même rôle ou à la même zone de sécurité dans leurs propres sous-réseaux.

**Bonne pratique**  : Évitez les petits réseaux et sous-réseaux virtuels pour garantir simplicité et flexibilité.   
**Détail**  : La plupart des organisations ajoutent plus de ressources qu’initialement prévu, et la réattribution d’adresses est laborieuse. L’utilisation de sous-réseaux de petite taille ajoute une valeur limitée à la sécurité, et le mappage d’un groupe de sécurité réseau à chaque sous-réseau ajoute une surcharge. Définissez les sous-réseaux largement pour être sûr de disposer de flexibilité pour leur croissance.

**Bonne pratique**  : Simplifiez la gestion des règles des groupes de sécurité réseau en définissant [Groupes de sécurité d’application](https://azure.microsoft.com/blog/applicationsecuritygroups/).  
**Détail**  : Définissez un groupe de sécurité d’application pour les listes d’adresses IP que vous pensez être susceptibles d’être modifiées à l’avenir, ou d’être utilisées sur plusieurs groupes de sécurité réseau. N’oubliez pas de nommer les groupes de sécurité d’application de façon explicite, pour que d’autres puissent comprendre leur contenu et leur finalité.

## <a name="adopt-a-zero-trust-approach"></a>Adoptez une approche Confiance Zéro
Les réseaux basés sur le périmètre fonctionnent sur l’hypothèse que tous les systèmes au sein d’un réseau peuvent être approuvés. Toutefois, les employés d’aujourd’hui accèdent aux ressources de leur entreprise depuis n’importe où, sur un grand nombre d’appareils et d’applications, ce qui rend les contrôles de sécurité du périmètre non pertinents. Les stratégies de contrôle d’accès, qui se concentrent uniquement sur qui peut accéder à une ressource, ne sont pas suffisantes. Afin de maîtriser l’équilibre entre sécurité et productivité, les administrateurs de sécurité doivent également tenir compte des *moyens* d’accès à une ressource.

Les réseaux doivent se développer à partir des défenses traditionnelles, car ils peuvent être vulnérables aux violations : un attaquant peut compromettre un point de terminaison unique au sein de la limite de confiance, puis rapidement déployer une brèche dans tout le réseau. Les réseaux de [Confiance Zéro](https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/) éliminent le concept d’approbation fondé sur l’emplacement réseau au sein d’un périmètre. À la place, les architectures de Confiance Zéro utilisent les revendications de confiance des appareils et des utilisateurs pour réguler l’accès aux ressources et aux données de l’organisation. Pour de nouvelles initiatives, adoptez les approches de Confiance Zéro qui valident l’approbation au moment de l’accès.

Les meilleures pratiques sont :

**Bonne pratique**  : Donnez l’accès conditionnel aux ressources en fonction de l’appareil, de l’identité, de l’assurance, de l’emplacement réseau, etc.  
**Détail**  : [L’accès conditionnel Azure AD](../../active-directory/conditional-access/overview.md) vous permet d’appliquer les contrôles d’accès appropriés en implémentant des décisions de contrôle d’accès automatisées, basées sur les conditions exigées. Pour plus informations, consultez [Gérer l’accès à la gestion Azure avec l’accès conditionnel](../../active-directory/conditional-access/howto-conditional-access-policy-azure-management.md).

**Bonne pratique**  : Activez l’accès au port uniquement après l’approbation du flux de travail.  
**Détail**  : Vous pouvez utiliser [l’accès juste-à-temps aux machines virtuelles dans Azure Security Center](../../security-center/security-center-just-in-time.md) pour verrouiller le trafic entrant vers vos machines virtuelles Azure, ce qui réduit l’exposition aux attaques et facilite la connexion aux machines virtuelles en cas de besoin.

**Bonne pratique**  : Accordez des autorisations temporaires pour effectuer des tâches privilégiées. De cette façon, les utilisateurs malveillants ou non autorisés ne peuvent pas accéder aux ressources une fois que les autorisations ont expiré. L’accès est accordé uniquement au moment où les utilisateurs en ont besoin.  
**Détail**  : Utilisez l’accès juste-à-temps dans Azure AD Privileged Identity Management ou dans une solution tierce pour accorder des autorisations en vue d’effectuer des tâches privilégiées.

La Confiance Zéro constitue la toute dernière évolution en matière de sécurité réseau. L’état des cyberattaques amène les organisations à adopter la mentalité « Assume Breach » (envisager les violations), mais cette démarche ne doit pas être limitative. Les réseaux de Confiance Zéro protègent les ressources et les données d’entreprise tout en garantissant aux organisations la possibilité de créer un espace de travail moderne, à l’aide de technologies qui donnent les moyens aux employés d’être productifs, à tout moment et en tous lieux, de quelque manière que ce soit.

## <a name="control-routing-behavior"></a>Contrôle du comportement de routage
Lorsque vous placez une machine virtuelle sur un réseau virtuel Azure, celle-ci peut se connecter à n’importe quelle autre machine virtuelle du même réseau virtuel, même si les autres machines virtuelles se trouvent sur des sous-réseaux différents. Ce type de communication est rendu possible par une collection d’itinéraires système qui sont activés par défaut. Grâce à ces itinéraires par défaut, les machines virtuelles placées sur le même réseau virtuel peuvent initier des connexions les unes avec les autres, ainsi qu’avec Internet (pour les communications sortantes vers Internet uniquement).

Même si les itinéraires système par défaut sont utiles dans de nombreux scénarios de déploiement, il se peut que vous souhaitiez personnaliser la configuration du routage pour vos déploiements. Vous pouvez configurer l’adresse du tronçon suivant afin d’atteindre des destinations spécifiques.

Nous vous recommandons de configurer des [itinéraires définis par l’utilisateur](../../virtual-network/virtual-networks-udr-overview.md) quand vous déployez une appliance de sécurité pour un réseau virtuel. Ce point est présenté dans une section ultérieure intitulée [Sécurisation de vos ressources critiques du service Azure pour vos réseaux virtuels uniquement](network-best-practices.md#secure-your-critical-azure-service-resources-to-only-your-virtual-networks).

> [!NOTE]
> Les itinéraires définis par l’utilisateur ne sont pas obligatoires ; les itinéraires système par défaut fonctionnent habituellement.
>
>

## <a name="use-virtual-network-appliances"></a>Utilisation d’appliances de réseau virtuel
Les groupes de sécurité réseau et le routage défini par l’utilisateur peuvent assurer un certain degré de sécurité réseau au niveau des couches réseau et transport du [modèle OSI](https://en.wikipedia.org/wiki/OSI_model). mais dans certaines situations, vous souhaiterez ou devrez activer la sécurité aux niveaux élevés de la pile. Le cas échéant, nous vous recommandons de déployer les appliances de sécurité de réseau virtuel fournies par les partenaires d’Azure.

Les appliances de sécurité réseau Azure peuvent offrir des niveaux de sécurité supérieurs à ceux des contrôles appliqués au niveau du réseau. Fonctionnalités de sécurité réseau des appliances de sécurité de réseau virtuel :


* Installation de pare-feu
* Détection et prévention des intrusions
* Gestion des vulnérabilités
* Contrôle des applications
* Détection des anomalies basée sur le réseau
* Filtrage web
* Antivirus
* Protection contre les botnets

Pour trouver les appliances de sécurité de réseau virtuel Azure disponibles, accédez à la [Place de marché Azure](https://azure.microsoft.com/marketplace/), puis recherchez les termes « sécurité » et « sécurité réseau ».

## <a name="deploy-perimeter-networks-for-security-zones"></a>Déploiement des réseaux de périmètre pour les zones de sécurité
Un [réseau de périmètre](/azure/architecture/vdc/networking-virtual-datacenter) (également appelé zone DMZ) est un segment de réseau logique ou physique qui fournit une couche de sécurité supplémentaire entre vos ressources et Internet. Les périphériques de contrôle d’accès réseau spécialisés situés à la périphérie d’un réseau de périmètre autorisent uniquement le trafic souhaité entrant dans votre réseau virtuel.

Les réseaux de périmètre sont utiles, car vous pouvez concentrer la gestion des contrôles d’accès réseau, la surveillance, la journalisation et la création de rapports sur les appareils situés à la périphérie de votre réseau virtuel Azure. Un réseau de périmètre est l’endroit où vous activez généralement la prévention de déni de service distribué (DDoS), les systèmes de détection et de prévention des intrusions (IDS/IPS), les règles et les stratégies de pare-feu, le filtrage web, les logiciels anti-programme malveillant du réseau, etc. Les appareils dédiés à la sécurité réseau se trouvent entre Internet et le réseau virtuel Azure, et disposent d’une interface sur les deux réseaux.

Même s’il s’agit là de la conception de base d’un réseau de périmètre, il existe de nombreuses autres conceptions, par exemple dos à dos, à triple hébergement et multirésidentes.

En prenant appui sur le concept de Confiance Zéro évoqué précédemment, nous vous recommandons de prévoir l’utilisation d’un réseau de périmètre pour tous les déploiements haute sécurité, afin d’améliorer le niveau de la sécurité réseau et le contrôle d’accès de vos ressources Azure. Vous pouvez utiliser Azure ou une solution tierce pour fournir une couche supplémentaire de sécurité entre vos ressources et internet :

- Contrôles natifs Azure. Le [Pare-feu Azure](../../firewall/overview.md) et le [Pare-feu d’applications web dans Application Gateway](../../application-gateway/features.md#web-application-firewall) offrent une sécurité de base moyennant un pare-feu entièrement avec état en tant que service, la haute disponibilité intégrée, l’extensibilité du cloud sans restriction, le filtrage du nom de domaine complet, la prise en charge d’ensembles de règles principales OWASP ainsi qu’une installation et une configuration simples.
- Offres de tiers. Recherchez sur la [Place de marché Azure](https://azuremarketplace.microsoft.com/) le pare-feu de nouvelle génération (NGFW) et d’autres offres de tiers qui fournissent des outils de sécurité courants et des niveaux de sécurité réseau grandement améliorés. La configuration peut être plus complexe, mais une offre de tiers peut vous permettre d’utiliser les fonctionnalités et les ensembles de compétences existants.

## <a name="avoid-exposure-to-the-internet-with-dedicated-wan-links"></a>Évitez toute exposition à Internet grâce à des liaisons réseau étendu dédiées
De nombreuses organisations ont opté pour l’informatique hybride. Avec un environnement informatique hybride, certaines des ressources informatiques de l’entreprise se trouvent sur Azure, et d’autres restent en local. Dans de nombreux cas, certains composants d’un service sont exécutés dans Azure, tandis que d’autres le sont localement.

Un scénario hybride propose généralement un certain type de connectivité entre locaux. La connectivité entre locaux permet à l’entreprise de relier ses réseaux locaux aux réseaux virtuels Azure. Deux solutions de connectivité entre locaux sont disponibles :

* [VPN de site à site](../../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md). Il s’agit d’une technologie établie, fiable et approuvée, mais la connexion s’effectue par Internet. La bande passante est limitée à un maximum d’environ 1,25 Gbits/s. Le VPN de site à site est une option souhaitable dans certains scénarios.
* **Azure ExpressRoute**. Nous vous recommandons d’utiliser [ExpressRoute](../../expressroute/expressroute-introduction.md) pour la connectivité entre locaux. ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. Grâce à ExpressRoute, vous pouvez établir des connexions aux services de cloud Microsoft, comme Azure, Microsoft 365 et Dynamics 365. ExpressRoute représente une liaison réseau étendu dédiée entre le site local et un fournisseur d’hébergement Microsoft Exchange. Comme il s’agit d’une connexion de télécommunications, vos données ne transitent pas par Internet. Elles ne sont donc pas exposées aux risques potentiels inhérents aux communications Internet.

L’emplacement de votre connexion ExpressRoute peut avoir une incidence sur la capacité du pare-feu, l’extensibilité, la fiabilité ainsi que sur la visibilité du trafic réseau. Vous devez déterminer l’emplacement où mettre fin à ExpressRoute dans les réseaux (locaux) existants. Vous pouvez :

- Mettre fin à l’extérieur du pare-feu (le paradigme du réseau de périmètre) si vous avez besoin d’une visibilité sur le trafic, si vous devez poursuivre une pratique existante d’isolation des centres de données ou si vous placez uniquement des ressources extranet sur Azure.
- Mettez fin à l’intérieur du pare-feu (le paradigme de l’extension réseau). Il s’agit de la suggestion par défaut. Dans tous les autres cas, nous vous conseillons de traiter Azure comme un nième centre de données.

## <a name="optimize-uptime-and-performance"></a>Optimisation de la durée active et des performances
Si un service est défaillant, les informations sont inaccessibles. Si les performances sont tellement médiocres que les données en sont inutilisables, vous pouvez considérer que ces dernières sont inaccessibles. Du point de vue de la sécurité, vous devez faire tout ce qui est en votre pouvoir pour garantir des performances et une durée de fonctionnement optimales pour vos services.

Pour optimiser la disponibilité et les performances, une méthode bien connue pour son efficacité est l’équilibrage de charge. L’équilibrage de charge est une méthode de répartition du trafic réseau entre les serveurs qui font partie d’un service. Ainsi, si votre service inclut plusieurs serveurs web frontaux, vous pouvez utiliser l’équilibrage de charge pour répartir le trafic entre ces derniers.

La répartition du trafic permet d’augmenter la disponibilité. En effet, si l’un des serveurs web est indisponible, l’équilibreur de charge arrête d’envoyer des données à ce serveur et redirige le trafic vers les serveurs actifs. L’équilibrage de charge améliore également les performances, car la surcharge du processeur, du réseau et de la mémoire associée au traitement des requêtes est répartie sur les différents serveurs avec équilibrage de charge.

Nous vous recommandons de tirer parti aussi souvent que possible de l’équilibrage de charge, selon les besoins de vos services. Voici des scénarios au niveau du réseau virtuel Azure et au niveau global, ainsi que des options d’équilibrage de charge pour chacun.

**Scénario**  : Vous disposez d’une application qui :

- Requiert des requêtes provenant d’une même session utilisateur/client pour atteindre la même machine virtuelle principale. Exemples : applications de panier d’achat et serveurs de courrier.
- Accepte uniquement une connexion sécurisée. La communication non chiffrée vers le serveur n’est donc pas une option acceptable.
- Exige le routage ou l’équilibrage de charge sur différents serveurs principaux des multiples requêtes HTTP sur une même connexion TCP de longue durée.

**Option d’équilibrage de charge**  : Utilisez [Azure Application Gateway](../../application-gateway/overview.md), un équilibreur de charge de trafic web HTTP. Application Gateway prend en charge le chiffrement TLS de bout en bout et la [terminaison TLS](../../application-gateway/overview.md) au niveau de la passerelle. Les serveurs web peuvent ensuite être libérés du traitement du chiffrement et du déchiffrement, et du trafic du contenu non chiffré vers les serveurs principaux.

**Scénario**  : Vous devez équilibrer la charge des connexions entrantes en provenance d’Internet entre vos serveurs situés au sein d’un réseau virtuel Azure. Ce sont les scénarios que vous rencontrez lorsque vous :

- Disposez d’applications sans état qui acceptent les demandes entrantes provenant d’Internet.
- N’exigez pas de sessions permanentes ni de déchargement TLS. Ces sessions correspondent à une méthode utilisée avec l’équilibrage de charge des applications pour obtenir l’affinité de serveur.

**Option d’équilibrage de charge**  : Utilisez le portail Azure pour [créer un équilibreur de charge externe](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) qui répartit les demandes entrantes sur plusieurs machines virtuelles afin de fournir un niveau de disponibilité plus élevé.

**Scénario**  : Vous devez équilibrer la charge des connexions des machines virtuelles qui ne sont pas sur Internet. Dans la plupart des cas, les connexions qui sont acceptées pour l’équilibrage de charge sont initiées par les appareils figurant sur un réseau virtuel Azure, par exemple des instances SQL Server ou des serveurs web internes.   
**Option d’équilibrage de charge**  : Utilisez le portail Azure pour [créer un équilibreur de charge interne](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) qui répartit les demandes entrantes sur plusieurs machines virtuelles afin de fournir un niveau de disponibilité plus élevé.

**Scénario**  : Vous recherchez un équilibrage de charge global, car vous :

- Disposez d’une solution cloud qui est largement distribuée dans plusieurs régions et qui nécessite le plus haut niveau de durée de fonctionnement (disponibilité) possible.
- Avez besoin du niveau de disponibilité le plus élevé possible pour vous assurer que votre service est disponible même si tout un centre de données ne l’est pas.

**Option d’équilibrage de charge**  : Utilisez Azure Traffic Manager. Grâce à ce service, vous pouvez équilibrer la charge des connexions vers vos services en fonction de l’emplacement de l’utilisateur.

Par exemple, si l’utilisateur qui envoie une requête à votre service se trouve dans un pays de l’Union européenne, la connexion est dirigée vers vos services qui se trouvent au sein d’un centre de données de cette zone. Cette étape de l’équilibrage de charge global assuré par Traffic Manager permet d’optimiser les performances, car la connexion au centre de données le plus proche est plus rapide que dans le cas de centres de données éloignés.

## <a name="disable-rdpssh-access-to-virtual-machines"></a>Désactivation de l’accès RDP/SSH aux machines virtuelles Azure
Il est possible d’atteindre les machines virtuelles Azure à l’aide des protocoles [RDP (Remote Desktop Protocol)](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) et [SSH (Secure Shell)](https://en.wikipedia.org/wiki/Secure_Shell). Ces protocoles permettent de gérer des machines virtuelles à partir d’emplacements distants. Ils sont souvent utilisés par les centres de données informatiques.

Cependant, ils peuvent être sources de problèmes de sécurité quand ils sont utilisés sur Internet. En effet, les attaquants peuvent recourir à des techniques de [force brute](https://en.wikipedia.org/wiki/Brute-force_attack) pour accéder aux machines virtuelles Azure. Lorsqu’ils y parviennent, ils peuvent utiliser votre machine virtuelle comme point de départ pour le piratage d’autres machines sur votre réseau virtuel, voire attaquer des appareils en réseau en dehors d’Azure.

Nous vous recommandons de désactiver l’accès direct des protocoles RDP et SSH à vos machines virtuelles Azure depuis Internet. Cela fait, vous disposez d’autres options vous permettant d’accéder à ces machines virtuelles à des fins de gestion à distance.

**Scénario**  : Autorisez un utilisateur unique à se connecter à un réseau virtuel Azure via Internet.   
**Option**  : L’expression [VPN de point à site](../../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md) est synonyme de connexion du client/serveur VPN pour un accès à distance. Une fois la connexion point à site établie, l’utilisateur peut avoir recours au protocole RDP ou SSH pour se connecter aux machines virtuelles situées sur le réseau virtuel Azure auquel cet utilisateur est connecté via le VPN point à site. Cela suppose que l’utilisateur dispose des autorisations requises pour atteindre ces machines virtuelles.

Le VPN point à site est plus sécurisé qu’une connexion RDP ou SSH directe, car l’utilisateur doit s’authentifier deux fois pour pouvoir se connecter à une machine virtuelle. L’utilisateur doit d’abord s’authentifier (et être autorisé) pour établir la connexion VPN point à site. Il doit ensuite s’authentifier (et être autorisé) pour établir la session RDP ou SSH.

**Scénario**  : Permettez aux utilisateurs de votre réseau local de se connecter aux machines virtuelles de votre réseau virtuel Azure.   
**Option**  : Un [VPN de site à site](../../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md) connecte un réseau dans son ensemble à un autre réseau, par le biais d’Internet. Vous pouvez utiliser un VPN de site à site pour connecter votre réseau local à un réseau virtuel Azure. Les utilisateurs de votre réseau local se connectent à l’aide du protocole RDP ou SSH via la connexion VPN de site à site. Vous n’avez pas à autoriser un accès RDP ou SSH direct via Internet.

**Scénario**  : Pour proposer une fonctionnalité similaire à la connexion VPN de site à site, utilisez une liaison réseau étendu dédiée.   
**Option**  : Utilisez [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/). Ce service fournit des fonctionnalités similaires au VPN de site à site. Les principales différences entre ces deux architectures sont les suivantes :

- La liaison réseau étendu (WAN) dédiée ne transite pas par Internet.
- Les liaisons WAN dédiées sont généralement plus stables et plus performantes.

## <a name="secure-your-critical-azure-service-resources-to-only-your-virtual-networks"></a>Sécurisation de vos ressources critiques du service Azure pour vos réseaux virtuels uniquement
Utilisez des points de terminaison de service de réseau virtuel pour étendre votre espace d’adressage privé de réseau virtuel et l’identité de votre réseau virtuel aux services Azure, via une connexion directe. Les points de terminaison permettent de sécuriser vos ressources critiques du service Azure pour vos réseaux virtuels uniquement. Le trafic à partir de votre réseau virtuel vers le service Azure reste toujours sur le réseau principal Microsoft Azure.

Les points de terminaison de service fournissent les avantages suivants :

- **Sécurité améliorée de vos ressources de service Azure** : Avec les points de terminaison de service, les ressources de service Azure peuvent être sécurisées pour votre réseau virtuel. La sécurisation des ressources du service pour un réseau virtuel renforce la sécurité grâce à la suppression complète de l’accès Internet public aux ressources et à l’autorisation du trafic seul à partir de votre réseau virtuel.
- **Routage optimal pour le trafic de service Azure à partir de votre réseau virtuel** : Tous les itinéraires dans votre réseau virtuel qui forcent le trafic Internet vers vos appliances locales et/ou virtuelles, aussi appelé tunneling forcé, forcent également le trafic de service Azure à prendre le même itinéraire que le trafic Internet. Les points de terminaison de service fournissent un routage optimal pour le trafic Azure.

  Les points de terminaison acheminent toujours le trafic de service directement à partir de votre réseau virtuel vers le service sur le réseau principal d’Azure. La conservation du trafic sur le réseau principal d’Azure vous permet de continuer l’audit et la surveillance du trafic Internet sortant à partir de vos réseaux virtuels, via le tunneling forcé, sans affecter le trafic de service. Découvrez d’autres informations sur les [itinéraires définis par l’utilisateur et le tunneling forcé](../../virtual-network/virtual-networks-udr-overview.md).

- **Une configuration simple et un temps de gestion réduit** : Les adresses IP publiques réservées dans vos réseaux virtuels ne sont désormais plus nécessaires pour sécuriser les ressources Azure via le pare-feu IP. Aucun NAT ou appareil de passerelle n’est requis pour configurer les points de terminaison de service. Les points de terminaison de service peuvent être configurés par un simple clic sur un sous-réseau. La conservation des points de terminaison ne requiert aucun traitement supplémentaire.

Pour en savoir plus sur les points de terminaison de service et sur les services et régions Azure pour lesquels ces points de terminaison sont disponibles, consultez [Points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md).

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.