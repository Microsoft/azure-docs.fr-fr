---
title: Fonctionnalités réseau
description: Découvrez les fonctionnalités réseau d’Azure App Service, ainsi que les fonctionnalités nécessaires à la sécurité et autres.
author: ccompy
ms.assetid: 5c61eed1-1ad1-4191-9f71-906d610ee5b7
ms.topic: article
ms.date: 10/18/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 5d950598e4a0af86ac37b53722e80eb4ef0a71a4
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183054"
---
# <a name="app-service-networking-features"></a>Fonctionnalités de mise en réseau App Service

Vous pouvez déployer des applications dans Azure App Service de plusieurs façons. Par défaut, les applications hébergées dans App Service sont accessibles directement via Internet et peuvent atteindre uniquement des points de terminaison hébergés sur Internet. Toutefois, pour bon nombre d’applications, vous devez contrôler le trafic réseau entrant et sortant. App Service inclut plusieurs fonctionnalités destinées à vous aider à répondre à ces besoins. Le défi consiste à savoir quelle fonctionnalité utiliser pour résoudre un problème donné. Cet article vous aidera à déterminer la fonctionnalité à utiliser, sur la base d’exemples de cas d’usage.

Il existe deux types de déploiement principaux pour Azure App Service : 
- Le service public mutualisé héberge des plans App Service dans les niveaux tarifaires Gratuit, Partagé, De base, Standard, Premium, PremiumV2 et PremiumV3. 
- L’environnement Azure App Service Environment (ASE) de locataire unique héberge les plans App Service de référence SKU Isolé directement dans votre réseau virtuel Azure. 

Les fonctionnalités que vous utilisez varient selon que vous êtes dans le service mutualisé ou dans un environnement ASE. 

## <a name="multitenant-app-service-networking-features"></a>Fonctionnalités réseau App Service mutualisées 

Azure App Service est un système distribué. Les rôles qui gèrent les requêtes HTTP/HTTPS entrantes sont appelés *serveurs frontaux*. Les rôles qui hébergent la charge de travail du client sont appelés *rôles de travail*. Tous les rôles d’un déploiement App Service existent dans un réseau mutualisé. Comme il existe de nombreux clients différents dans la même unité d’échelle App Service, vous ne pouvez pas connecter le réseau App Service directement à votre réseau. 

Au lieu de connecter les réseaux, vous avez besoin de fonctionnalités pour gérer les différents aspects de la communication de l’application. Les fonctionnalités qui gèrent les demandes *vers* votre application ne peuvent pas être utilisées pour résoudre des problèmes lorsque des appels sont effectués *depuis* votre application. De même, les fonctionnalités qui résolvent des problèmes pour les appels depuis votre application ne peuvent pas être utilisées pour résoudre des problèmes vers votre application.  

| Fonctionnalités en entrée | Fonctionnalités en sortie |
|---------------------|-------------------|
| Adresse attribuée par l’application | les connexions hybrides |
| Restrictions d'accès | Intégration au réseau virtuel avec passerelle obligatoire |
| Points de terminaison de service | Intégration au réseau virtuel |
| Instances Private Endpoint ||

À part les exceptions notées, vous pouvez utiliser toutes ces fonctionnalités ensemble. Vous pouvez combiner les fonctionnalités pour résoudre vos problèmes.

## <a name="use-cases-and-features"></a>Cas d’usage et fonctionnalités

Pour tout cas d’usage, il peut y avoir plusieurs façons de résoudre le problème. Le choix de la meilleure fonctionnalité va parfois au-delà du cas d’usage. Les cas d’usage en entrée suivants suggèrent comment utiliser des fonctionnalités réseau App Service pour résoudre des problèmes en lien avec le contrôle du trafic vers votre application :
 
| Cas d’usage en entrée | Fonctionnalité |
|---------------------|-------------------|
| Prendre en charge les besoins SSL en fonction des adresses IP pour votre application | Adresse attribuée par l’application |
| Prendre en charge une adresse entrante dédiée non partagée pour votre application | Adresse attribuée par l’application |
| Restreindre l’accès à votre application à partir d’un ensemble d’adresses bien définies | Restrictions d'accès |
| Restreindre l’accès à votre application à partir des ressources d’un réseau virtuel | Points de terminaison de service </br> ASE ILB </br> Instances Private Endpoint |
| Exposer votre application sur une adresse IP privée dans votre réseau virtuel | ASE ILB </br> Instances Private Endpoint </br> Adresse IP privée pour le trafic entrant sur une instance Application Gateway avec des points de terminaison de service |
| Protéger votre application avec un pare-feu d’applications web (WAF) | Application Gateway et ASE ILB </br> Application Gateway avec des points de terminaison privés </br> Application Gateway avec des points de terminaison de service </br> Azure Front Door avec des restrictions d’accès |
| Équilibrer la charge du trafic vers vos applications dans différentes régions | Azure Front Door avec des restrictions d’accès | 
| Équilibrer la charge du trafic dans la même région | [Application Gateway avec des points de terminaison de service][appgwserviceendpoints] | 

Les cas d’usage en sortie suivants suggèrent comment utiliser les fonctionnalités réseau d’App Service pour répondre aux besoins d’accès sortant de votre application :

| Cas d’usage en sortie | Fonctionnalité |
|---------------------|-------------------|
| Accéder à des ressources dans un réseau virtuel Azure situé dans la même région | Intégration au réseau virtuel </br> ASE |
| Accéder à des ressources dans un réseau virtuel Azure situé dans une région différente | Intégration au réseau virtuel avec passerelle obligatoire </br> ASE et homologation de réseau virtuel mondial |
| Accéder à des ressources sécurisées avec des points de terminaison de service | Intégration au réseau virtuel </br> ASE |
| Accéder à des ressources dans un réseau privé non connecté à Azure | les connexions hybrides |
| Accéder à des ressources via des circuits Azure ExpressRoute | Intégration au réseau virtuel </br> ASE | 
| Sécuriser le trafic sortant à partir de votre application web | Intégration au réseau virtuel et groupes de sécurité réseau </br> ASE | 
| Router le trafic sortant à partir de votre application web | Intégration au réseau virtuel et tables de route </br> ASE | 


### <a name="default-networking-behavior"></a>Comportement de mise en réseau par défaut

Des unités d’échelle Azure App Service prennent en charge de nombreux clients dans chaque déploiement. Les plans de référence SKU Gratuit et Partagé hébergent des charges de travail client sur des rôles de travail mutualisés. Les plans De base et supérieurs hébergent les charges de travail client dédiées sur un seul plan App Service. Si vous avez un plan App Service Standard, toutes les applications dans ce plan s’exécutent sur le même rôle de travail. Si vous effectuez un scale-out du rôle de travail, toutes les applications de ce plan App Service sont répliquées sur un nouveau rôle de travail pour chaque instance de votre plan App Service. 

#### <a name="outbound-addresses"></a>Adresses sortantes

Les machines virtuelles de travail sont décomposées en grande partie par les plans App Service. Les plans Gratuit, Partagé, De base, Standard et Premium utilisent tous le même type de machine virtuelle de travail. Le plan PremiumV2 utilise un autre type de machine virtuelle. Le plan PremiumV3 utilise encore un autre type de machine virtuelle. Lorsque vous modifiez la famille de machines virtuelles, vous recevez un autre ensemble d’adresses sortantes. Si vous passez d’un plan Standard à un plan Premiumv2, vos adresses sortantes changent. Si vous passez d’un plan Premiumv2 à un plan Premiumv3, vos adresses sortantes changent. Dans certaines unités d’échelle plus anciennes, les adresses tant entrantes que sortantes changent lorsque vous effectuez une mise à l’échelle d’un plan Standard vers un plan PremiumV2. 

Un certain nombre d’adresses sont utilisées pour des appels sortants. Les adresses sortantes utilisées par votre application pour effectuer des appels sortants sont répertoriées dans les propriétés de votre application. Ces adresses sont partagées par toutes les applications qui s’exécutent sur la même famille de machines virtuelles de travail dans le déploiement App Service. Si vous souhaitez voir toutes les adresses que votre application peut utiliser dans une unité d’échelle, il existe une propriété appelée `possibleOutboundAddresses` qui les répertorie. 

![Capture d’écran montrant les propriétés de l’application.](media/networking-features/app-properties.png)

App Service a plusieurs points de terminaison qui sont utilisés pour gérer le service.  Ces adresses sont publiées dans un document distinct, et figurent également dans la balise de service d’adresse IP `AppServiceManagement`. La balise `AppServiceManagement` est utilisée uniquement dans les environnements ASE où vous devez autoriser un tel trafic. Les adresses App Service entrantes sont suivies dans la balise de service d’adresse IP `AppService`. Il n’y a aucune balise de service d’adresse IP contenant les adresses sortantes qu’App Service utilise. 

![Diagramme montrant le trafic entrant et sortant d’App Service.](media/networking-features/default-behavior.png)

### <a name="app-assigned-address"></a>Adresse attribuée par l’application 

La fonctionnalité d’adresse attribuée par l’application est une ramification de la fonctionnalité SSL basée sur IP. Vous y accédez en configurant SSL avec votre application. Vous pouvez utiliser cette fonctionnalité pour les appels SSL basés sur IP. Vous pouvez également l’utiliser pour fournir à votre application une adresse dont elle seule dispose. 

![Diagramme illustrant l’adresse attribuée par l’application.](media/networking-features/app-assigned-address.png)

Lorsque vous utilisez une adresse attribuée par l’application, votre trafic traverse toujours les mêmes rôles de serveur frontal qui gèrent tout le trafic entrant dans l’unité d’échelle App Service. Toutefois, l’adresse attribuée à votre application est utilisée uniquement par celle-ci. Cas d’usage de cette fonctionnalité :

* Prendre en charge les besoins SSL basés sur IP pour votre application.
* Définir une adresse dédiée non partagée pour votre application.

Pour découvrir comment définir une adresse sur votre application, consultez [Ajouter un certificat TLS/SSL dans Azure App Service][appassignedaddress]. 

### <a name="access-restrictions"></a>Restrictions d'accès 

Les restrictions d’accès vous permettent de filtrer les demandes *entrantes*. L’action de filtrage a lieu sur les rôles front-end situés en amont des déploiements des rôles de travail où vos applications s’exécutent. Étant donné que les rôles de serveur frontal sont en amont des rôles de travail, vous pouvez considérer les restrictions d’accès comme une protection de vos applications au niveau du réseau. 

Cette fonctionnalité vous permet de créer une liste de règles d’autorisation et de refus qui sont évaluées par ordre de priorité. Elle est similaire à la fonctionnalité de groupe de sécurité réseau (NSG) dans Azure Networking. Vous pouvez utiliser cette fonctionnalité dans un environnement ASE ou dans le service mutualisé. Lorsque vous l’utilisez avec un ASE ILB ou point de terminaison privé, vous pouvez restreindre l’accès à partir de blocs d’adresses privées.
> [!NOTE]
> Vous pouvez configurer Jusqu’à 512 règles de restrictions par application. 

![Diagramme illustrant les restrictions d’accès.](media/networking-features/access-restrictions.png)

#### <a name="ip-based-access-restriction-rules"></a>Règles de restrictions d’accès basées sur IP

La fonctionnalité des restrictions d’accès basées sur IP est utile quand vous souhaitez limiter les adresses IP qui peuvent être utilisées pour atteindre votre application. IPv4 et IPv6 sont pris en charge. Cas d’usage de cette fonctionnalité :
* Restreindre l’accès à votre application à partir d’un ensemble d’adresses bien définies. 
* Restreindre l’accès au trafic provenant d’un service d’équilibrage de charge tel qu’Azure Front Door. Si vous souhaitez verrouiller votre trafic entrant vers Azure Front Door, créez des règles pour autoriser le trafic à partir de 147.243.0.0/16 et 2a01:111:2050::/44. 

Pour découvrir comment activer cette fonctionnalité, voir [Restrictions d’accès dans Azure App Service][iprestrictions].

#### <a name="access-restriction-rules-based-on-service-endpoints"></a>Règles de restriction d’accès basées sur des points de terminaison de service 

Les points de terminaison de service vous permettent de verrouiller l’accès *entrant* à votre application, de façon que l’adresse source doive faire partie d’un ensemble de sous-réseaux que vous sélectionnez. Cette fonctionnalité opère conjointement avec les restrictions d’accès IP. Les points de terminaison de service ne sont pas compatibles avec le débogage à distance. Si vous voulez utiliser le débogage à distance avec votre application, votre client ne peut pas se trouver dans un sous-réseau dans lequel des points de terminaison de service sont activés. Le processus de définition de points de terminaison de service est similaire au processus de définition de restrictions d’accès IP. Vous pouvez créer une liste d’autorisation/de refus des règles d’accès qui inclut des adresses publiques, ainsi que des sous-réseaux dans vos réseaux virtuels. 

Cas d’usage de cette fonctionnalité :

* Configurer une passerelle applicative avec votre application pour verrouiller le trafic entrant vers votre application.
* Limiter l’accès à votre application aux ressources de votre réseau virtuel. Celles-ci peuvent inclure des machines virtuelles, des environnements ASE, voire d’autres applications qui utilisent une intégration au réseau virtuel. 

![Diagramme illustrant l’utilisation de points de terminaison de service avec Application Gateway.](media/networking-features/service-endpoints-appgw.png)

Pour en savoir plus sur la configuration des points de terminaison de service avec votre application, consultez [Restrictions d’accès dans Azure App Service][serviceendpoints].

### <a name="private-endpoint"></a>Point de terminaison privé

Un point de terminaison privé est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à votre application web via une liaison privée Azure. Un point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, ce qui a pour effet d’introduire l’application web dans votre réseau virtuel. Cette fonctionnalité s’applique uniquement aux flux *entrants* dans votre application web.
Pour plus d’informations, consultez [Utilisation de points de terminaison privés pour une application web Azure][privateendpoints].

Cas d’usage de cette fonctionnalité :

* Limiter l’accès à votre application à partir de ressources d’un réseau virtuel. 
* Exposer votre application sur une adresse IP privée dans votre réseau virtuel. 
* Protéger votre application avec un pare-feu d’applications web (WAF).

Les points de terminaison privés empêchent l’exfiltration de données, car la seule chose que vous pouvez atteindre sur le point de terminaison privé est l’application avec laquelle il est configuré. 
 
### <a name="hybrid-connections"></a>les connexions hybrides

La fonctionnalité Connexions hybrides App Service permet à vos applications d’effectuer des appels *sortants* vers des points de terminaison TCP spécifiés. Le point de terminaison peut être local, dans un réseau virtuel, ou dans n’importe quel emplacement autorisant le trafic sortant vers Azure sur le port 443. Pour utiliser cette fonctionnalité, vous devez installer un agent de relais appelé Hybrid Connection Manager sur un serveur Windows Server 2012 ou un hôte plus récent. L’agent de relais Hybrid Connection Manager doit être en mesure d’atteindre Azure Relay sur le port 443. Vous pouvez télécharger Hybrid Connection Manager à partir de l’interface utilisateur des connexions hybrides d’App Service dans le portail. 

![Diagramme montrant le flux réseau de connexions hybrides.](media/networking-features/hybrid-connections.png)

Les connexions hybrides d’App Service reposent sur la capacité de connexions hybrides d’Azure Relay. App Service utilise une forme spécialisée de la fonctionnalité qui prend uniquement en charge les appels sortants de votre application vers un port et un hôte TCP. Cet hôte et ce port doivent être résolus uniquement sur l’hôte sur lequel Hybrid Connection Manager est installé. 

Lorsque l’application, dans App Service, effectue une recherche DNS sur l’hôte et le port définis dans votre connexion hybride, le trafic est redirigé automatiquement pour traverser la connexion hybride et sortir de Hybrid Connection Manager. Pour plus d’informations, consultez [Connexions hybrides d’App Service][hybridconn].

Cette fonctionnalité est couramment utilisée pour :

* Accéder aux ressources dans des réseaux privés qui ne sont pas connectés à Azure via un VPN ou un circuit ExpressRoute.
* Prendre en charge la migration d’applications locales vers App Service sans avoir à déplacer les bases de données associées.  
* Fournir un accès avec une sécurité améliorée à un hôte et un port uniques par connexion hybride. La plupart des fonctionnalités réseau ouvrent l’accès à un réseau. Avec des connexions hybrides, vous ne pouvez atteindre que l’hôte et le port uniques.
* Couvrir des scénarios non couverts par d’autres méthodes de connectivité sortante.
* Effectuer du développement dans App Service de manière à permettre aux applications d’utiliser facilement des ressources locales. 

Étant donné que cette fonctionnalité permet d’accéder aux ressources locales sans un trou dans le pare-feu de trafic entrant, elle est populaire auprès des développeurs. Les autres fonctionnalités réseau d’App Service sortantes ont trait au réseau virtuel Azure. Les connexions hybrides ne dépendent pas d’un passage par un réseau virtuel. Elles peuvent être utilisées pour couvrir un éventail plus vaste de besoins de réseau. 

Notez que les connexions hybrides d’App Service ignorent ce que vous en faites. Vous pouvez les utiliser pour accéder à une base de données, à un service web ou à un socket TCP arbitraire sur un ordinateur mainframe. La fonctionnalité crée essentiellement des tunnels pour les paquets TCP. 

Les connexions hybrides sont populaires pour le développement, mais sont également utilisées dans des applications de production. Elles sont très utiles pour accéder à un service web ou à une base de données, mais ne conviennent pas pour des situations impliquant la création de nombreuses connexions. 

### <a name="gateway-required-vnet-integration"></a>Intégration au réseau virtuel avec passerelle obligatoire 

L’intégration au réseau virtuel App Service avec passerelle obligatoire permet à votre application d’envoyer des requêtes *sortantes* à un réseau virtuel Azure. La fonctionnalité fonctionne en connectant l’hôte sur lequel votre application s’exécute à une passerelle de réseau virtuel sur votre réseau virtuel en utilisant une connexion VPN point à site. Lorsque vous configurez la fonctionnalité, votre application obtient une des adresses de point à site attribuées à chaque instance. Cette fonctionnalité vous permet d’accéder aux ressources dans des réseaux virtuels classiques ou Azure Resource Manager dans toute région. 

![Diagramme illustrant l’intégration au réseau virtuel avec passerelle obligatoire.](media/networking-features/gw-vnet-integration.png)

Cette fonctionnalité résout le problème d’accès aux ressources dans d’autres réseaux virtuels. Elle permet même de se connecter via un réseau virtuel à d’autres réseaux virtuels ou à des réseaux locaux. Elle ne fonctionne pas avec des réseaux virtuels connectés via ExpressRoute, mais fonctionne avec des réseaux connectés via un VPN site à site. Il est généralement inapproprié d’utiliser cette fonctionnalité à partir d’une application dans un environnement ASE, car celui-ci est déjà dans votre réseau virtuel. Cas d’usage de cette fonctionnalité :

* Accéder aux ressources sur des adresses IP privées dans vos réseaux virtuels Azure. 
* Accéder aux ressources locales s’il existe un VPN site à site. 
* Accéder aux ressources de réseaux virtuels appairés. 

Lorsque cette fonctionnalité est activée, votre application utilise le serveur DNS avec lequel le réseau virtuel de destination est configuré. Pour plus d’informations sur cette fonctionnalité, consultez [Intégration au réseau virtuel App Service][vnetintegrationp2s]. 

### <a name="vnet-integration"></a>Intégration au réseau virtuel

L’intégration au réseau virtuel avec passerelle obligatoire est utile, mais elle ne résout pas le problème d’accès aux ressources via ExpressRoute. En plus de devoir transiter par des connexions ExpressRoute, il est nécessaire que les applications soient en mesure d’effectuer des appels à des services sécurisés par point de terminaison de service. Une autre fonctionnalité d’intégration au réseau virtuel peut répondre à ces besoins. 

La nouvelle fonctionnalité Intégration au réseau virtuel vous permet de placer le back end de votre application dans un sous-réseau d’un réseau virtuel Resource Manager situé dans la même région que votre application. Cette fonctionnalité n’est pas disponible à partir d’un environnement ASE, qui se trouve déjà dans un réseau virtuel. Cas d’usage de cette fonctionnalité :

* Accéder à des ressources de réseaux virtuels Resource Manager dans la même région.
* Accéder à des ressources sécurisées avec des points de terminaison de service. 
* Accéder à des ressources accessibles via des connexions ExpressRoute ou VPN.
* Aider à sécuriser tout le trafic sortant. 
* Forcer le tunneling de tout le trafic sortant. 

![Diagramme illustrant l’intégration au réseau virtuel.](media/networking-features/vnet-integration.png)

Pour plus d’informations, consultez [Intégration au réseau virtuel App Service][vnetintegration].

### <a name="app-service-environment"></a>Environnement App Service 

Un environnement ASE est un déploiement monolocataire d’Azure App Service, qui s’exécute dans votre réseau virtuel. Cas d’usage de cette fonctionnalité :

* Accéder aux ressources de votre réseau virtuel.
* Accéder à des ressources via ExpressRoute.
* Exposer vos applications avec une adresse privée dans votre réseau virtuel. 
* Accéder à des ressources via des points de terminaison de service. 

Avec un environnement ASE, vous n’avez pas besoin d’utiliser des fonctionnalités telles que l’intégration au réseau virtuel ou des points de terminaison de service, car l’ASE est déjà dans votre réseau virtuel. Si vous souhaitez accéder à des ressources telles que SQL ou Stockage Azure sur des points de terminaison de service, activez les points de terminaison de service sur le sous-réseau ASE. Si vous souhaitez accéder à des ressources dans le réseau virtuel, vous n’avez pas besoin d’effectuer de configuration supplémentaire. Si vous souhaitez accéder à des ressources via ExpressRoute, vous êtes déjà dans le réseau virtuel et n’avez pas besoin de configurer quoi que ce soit sur l’environnement ASE ou les applications qu’il contient. 

Étant donné que les applications dans un ASE ILB peuvent être exposées sur une adresse IP privée, vous pouvez facilement ajouter des appareils de pare-feu d’applications web pour exposer uniquement les applications que vous souhaitez sur Internet tout en sécurisant le reste. Cette fonctionnalité peut faciliter le développement d’applications multiniveaux. 

Certaines choses ne sont actuellement pas possibles à partir du service mutualisé, mais le sont à partir d’un environnement ASE. Voici quelques exemples :

* Exposer vos applications sur une adresse IP privée.
* Sécuriser l’ensemble du trafic sortant avec des contrôles réseau qui ne font pas partie de votre application.
* Héberger vos applications dans un service monolocataire. 
* Effectuer un scale-up vers beaucoup plus d’instances que ne le permet le service mutualisé. 
* Charger des certificats clients d’autorité de certification privée à l’usage de vos applications avec des points de terminaison sécurisés par l’autorité de certification privée.
* Forcer l’utilisation du protocole TLS 1.1 pour toutes les applications hébergées dans le système, sans possibilité de le désactiver au niveau de l’application. 
* Fournir une adresse sortante dédiée pour toutes les applications dans votre environnement ASE qui ne sont pas partagées avec des clients. 

![Diagramme illustrant un environnement ASE dans un réseau virtuel.](media/networking-features/app-service-environment.png)

L’environnement ASE offre la meilleure solution en ce qui concerne l’hébergement d’applications dédiées et isolées, mais soulève également quelques problèmes de gestion. Voici quelques points à prendre en compte avant d’utiliser un environnement ASE opérationnel :
 
 * Un environnement ASE s’exécute à l’intérieur de votre réseau virtuel, mais présente des dépendances extérieures au réseau virtuel. Ces dépendances doivent être autorisées. Pour plus d’informations, consultez [Considérations relatives à la mise en réseau pour un environnement App Service][networkinfo].
 * Un environnement ASE n’évolue pas immédiatement comme le service mutualisé. Vous devez anticiper les besoins de mise à l’échelle, plutôt que d’effectuer une mise à l’échelle quand il est trop tard. 
 * Un environnement ASE a un coût initial supérieur. Afin de tirer le meilleur parti de votre ASE, vous devez planifier l’ajout de nombreuses charges de travail dans un seul ASE au lieu de ne l’utiliser que pour quelques tâches.
 * Les applications dans un ASE ne peuvent pas restreindre de manière sélective l’accès à certaines applications dans l’ASE et pas à d’autres.
 * Un environnement ASE se trouve dans un sous-réseau, et des règles de réseau s’appliquent à tout le trafic vers et depuis cet environnement. Si vous souhaitez attribuer des règles de trafic entrant pour une seule application, utilisez des restrictions d’accès. 

## <a name="combining-features"></a>Combinaison de fonctionnalités 

Les fonctionnalités indiquées pour le service mutualisé peuvent être utilisées ensemble pour traiter des cas d’usage plus complexes. Deux cas d’usage plus courants sont décrits ici, mais ils servent uniquement d’exemples. En comprenant le fonctionnement des différentes fonctionnalités, vous pouvez répondre à pratiquement tous les besoins architecturaux de votre système.

### <a name="place-an-app-into-a-virtual-network"></a>Placer une application dans un réseau virtuel

Vous vous demandez peut-être comment placer une application dans un réseau virtuel. Si vous placez votre application dans un réseau virtuel, les points de terminaison entrants et sortants pour l’application se trouvent dans le réseau virtuel. Un environnement ASE est la meilleure façon de résoudre ce problème. Toutefois, vous pouvez répondre à la plupart de vos besoins au sein du service mutualisée en combinant des fonctionnalités. Par exemple, vous pouvez héberger des applications intranet uniquement avec des adresses entrantes et sortantes privées comme suit :

* En créant une passerelle applicative avec des adresse entrante et sortante privées.
* En sécurisant le trafic entrant vers votre application avec des points de terminaison de service. 
* En utilisant la nouvelle fonctionnalité d’intégration au réseau virtuel de façon à ce que le back end de votre application se trouve dans votre réseau virtuel. 

Ce style de déploiement ne vous donne pas d’adresse dédiée pour le trafic sortant vers Internet, ou la possibilité de verrouiller tout le trafic sortant de votre application. Il vous apporte essentiellement ce que vous ne pourriez obtenir autrement qu’avec un environnement ASE. 

### <a name="create-multitier-applications"></a>Créer des applications multiniveaux

Une application multiniveau est une application dans laquelle les applications back end d’API sont accessibles uniquement à partir du niveau frontal. Deux méthodes permettent de créer une application multiniveau. Toutes deux commencent par utiliser une intégration au réseau virtuel pour connecter votre application web frontale à un sous-réseau dans un réseau virtuel. Cela permet à votre application web d’effectuer des appels dans votre réseau virtuel. Une fois votre application frontale est connectée au réseau virtuel, vous devez choisir comment verrouiller l’accès à votre application d’API. Vous pouvez :

* Héberger l’application frontale et l’application d’API dans le même ASE ILB, et exposer l’application frontale sur Internet en utilisant une passerelle applicative.
* Héberger le serveur frontal dans le service mutualisé et le back end dans un ASE ILB.
* Héberger le serveur frontal et l’application d’API dans le service mutualisé.

Si vous hébergez l’application frontale et l’application d’API pour une application multiniveau, vous pouvez :

- Exposer votre application d’API à l’aide de points de terminaison privés dans votre réseau virtuel :

  ![Diagramme illustrant l’utilisation de points de terminaison privés dans une application à deux niveaux.](media/networking-features/multi-tier-app-private-endpoint.png)

- Utiliser des points de terminaison de service pour vous assurer que le trafic entrant dans votre application d’API provienne uniquement du sous-réseau qu’utilise votre application web frontale :

  ![Diagramme illustrant l’utilisation de points de terminaison de service pour sécuriser une application.](media/networking-features/multi-tier-app.png)

Voici quelques éléments à prendre en considération pour vous aider à choisir la méthode à utiliser :

* Lorsque vous utilisez des points de terminaison de service, vous devez uniquement sécuriser le trafic vers votre application d’API sur le sous-réseau d’intégration. Cela sécurise l’application d’API, mais pourrait offrir une possibilité d’exfiltration de données de votre application frontale vers d’autres applications dans App Service.
* Lorsque vous utilisez des points de terminaison privés, vous avez deux sous-réseaux en lecture, ce qui ajoute de la complexité. En outre, le point de terminaison privé est une ressource de niveau supérieur et ajoute une surcharge de gestion. L’avantage de l’utilisation de points de terminaison privés est qu’ils éliminent la possibilité d’exfiltration de données. 

Les deux méthodes fonctionnent avec plusieurs serveurs frontaux. À petite échelle, les points de terminaison de service sont plus faciles à utiliser, car il vous suffit d’activer les points de terminaison de service pour l’application d’API sur le sous-réseau d’intégration frontal. À mesure que vous ajoutez des applications frontales, vous devez ajuster chaque application d’API pour inclure des points de terminaison de service avec le sous-réseau d’intégration. Lorsque vous utilisez des points de terminaison privés, vous avez plus de complexité, mais vous n’avez rien à modifier sur vos applications d’API après avoir défini un point de terminaison privé. 

### <a name="line-of-business-applications"></a>applications métier ;

Les applications métier sont des applications internes qui ne sont normalement pas exposées pour un accès depuis Internet. Ces applications sont appelées à partir de réseaux d’entreprise où l’accès peut être strictement contrôlé. Si vous utilisez un ASE ILB, il est facile d’héberger vos applications métier. Si vous utilisez le service mutualisé, vous pouvez utiliser des points de terminaison privés ou des points de terminaison de service associés à une passerelle applicative. Il existe deux raisons d’utiliser une passerelle applicative avec des points de terminaison de service plutôt que des points de terminaison privés :
* Vous avez besoin d’une protection WAF sur vos applications métier.
* Vous souhaitez équilibrer la charge sur plusieurs instances de vos applications métier.

Si aucun de ces besoins ne s’applique, il est préférable d’utiliser des points de terminaison privés. Avec les points de terminaison privés disponibles dans App Service, vous pouvez exposer vos applications sur des adresses privées de votre réseau virtuel. Le point de terminaison privé que vous placez dans votre réseau virtuel peut être atteint via des connexions ExpressRoute et VPN. 

La configuration de points de terminaison privés expose vos applications sur une adresse privée, mais vous devez configurer un DNS pour atteindre cette adresse en local. Pour que cette configuration fonctionne, vous devez transférer la zone Azure DNS privée contenant vos points de terminaison privés vers vos serveurs DNS locaux. Les zones Azure DNS privées ne prennent pas en charge le transfert de zone, mais vous pouvez le prendre en charge à l’aide d’un serveur DNS dédié à cette fin. Le modèle de [Redirecteur DNS](https://azure.microsoft.com/resources/templates/301-dns-forwarder/) facilite le transfert de votre zone Azure DNS privée vers vos serveurs DNS locaux.

## <a name="app-service-ports"></a>Ports App Service

Si vous analysez App Service, vous trouverez plusieurs ports exposés pour les connexions entrantes. Il n’existe aucun moyen de bloquer ou de contrôler l’accès à ces ports dans le service mutualisé. Voici la liste des ports exposés :

| Utilisation | Port ou ports |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  Gestion | 454, 455 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Débogage distant de Visual Studio  |  4020, 4022, 4024 |
|  Service Web Deploy | 8172 |
|  Utilisation de l’infrastructure | 7654, 1221 |

<!--Links-->
[appassignedaddress]: ./configure-ssl-certificate.md
[iprestrictions]: ./app-service-ip-restrictions.md
[serviceendpoints]: ./app-service-ip-restrictions.md
[hybridconn]: ./app-service-hybrid-connections.md
[vnetintegrationp2s]: ./web-sites-integrate-with-vnet.md
[vnetintegration]: ./web-sites-integrate-with-vnet.md
[networkinfo]: ./environment/network-info.md
[appgwserviceendpoints]: ./networking/app-gateway-with-service-endpoints.md
[privateendpoints]: ./networking/private-endpoint.md