---
title: Azure Front Door - Équilibrage de charge avec la suite de livraison d’application Azure | Microsoft Docs
description: Cet article explique comment Azure recommande l’équilibrage de charge avec sa suite de livraison d’application.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 50e047325ad17710794b1640715ab1938373fe85
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019461"
---
# <a name="load-balancing-with-azures-application-delivery-suite"></a>Équilibrage de charge avec la suite de livraison d’application Azure

## <a name="introduction"></a>Introduction
Microsoft Azure propose divers services globaux et régionaux destinés à gérer la distribution et l’équilibrage de la charge de votre trafic réseau : 

* Application Gateway
* Front Door 
* Load Balancer  
* Traffic Manager

Associée aux nombreuses régions Azure et à son architecture zonale, l’utilisation conjointe de ces services vous permet de créer des applications robustes, évolutives et à hautes performances.

:::image type="content" source="./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure1.png" alt-text="Suite de livraison d’application":::
 
Ces services sont répartis en deux catégories :
1. Les **services d’équilibrage de charge globaux** tels que Traffic Manager et Front Door distribuent le trafic de vos utilisateurs finaux sur vos back-ends régionaux, entre les clouds et même vos services locaux hybrides. L’équilibrage de charge global achemine le trafic vers votre backend de service le plus proche et réagit aux changements de fiabilité des services afin de maintenir une disponibilité continue et haute performance pour vos utilisateurs. 
1. Les **services d’équilibrage de charge régionaux** tels que Load Balancer et Application Gateway permettent de distribuer le trafic vers des machines virtuelles dans un réseau virtuel ou des points de terminaison de service dans une région.

Lorsque vous combinez ces services globaux et régionaux, votre application bénéficiera d’un trafic de bout en bout fiable et sécurisé qui est envoyé par vos utilisateurs finaux à vos services IaaS, PaaS ou locaux. Vous trouverez une description de chacun de ces services dans la section suivante.

## <a name="global-load-balancing"></a>Équilibrage de charge global
**Traffic Manager** assure un équilibrage de charge DNS global. Il examine les requêtes DNS entrantes et répond avec un backend intègre, suivant la stratégie de routage choisie par le client. Les différentes méthodes de routage disponibles sont les suivantes :
- **Le routage des performances envoie les demandes au backend le plus proche avec la latence la plus faible.
- Le **routage par priorité** permet de rediriger l’ensemble du trafic vers un backend, les autres backends jouant le rôle de backends de secours.
- Le **routage du trafic en tourniquet (round robin)** pondéré permet de répartir le trafic en fonction de la pondération associée à chaque backend.
- **Le routage géographique** garantit que les demandes provenant de régions géographiques spécifiques sont gérées par des backends mappés pour ces régions. (Par exemple, toutes les demandes en provenance d’Espagne doivent être dirigées vers la région Azure France centre)
- Le **routage de sous-réseau** vous permet de mapper des plages d’adresses IP aux backends, afin que les demandes entrantes pour ces adresses IP soient envoyées au backend spécifique. (Par exemple, tous les utilisateurs qui se connectent à partir de la plage d’adresses IP de votre siège social doivent obtenir un contenu Web différent de celui des utilisateurs généraux)

Le client se connecte directement à ce backend. Azure Traffic Manager détecte quand un backend est défectueux, et il redirige les clients vers une autre instance intègre. Pour plus d’informations sur ce service, consultez la [documentation Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md).

**Azure Front Door** fournit une accélération de site web dynamique, notamment l’équilibrage de charge HTTP global.  Il examine les requêtes HTTP entrantes et les achemine vers le backend ou la région la plus proche pour le nom d’hôte, le chemin d’URL et les règles configurées spécifiées.  
Front Door met fin aux requêtes HTTP à la périphérie du réseau Microsoft, et détecte de manière active les changements d’intégrité ou de latence d’application ou d’infrastructure.  Front Door achemine ensuite toujours le trafic vers le backend (sain) le plus rapide et disponible. Pour en savoir plus sur ce service, consultez les détails sur l’[architecture de routage](front-door-routing-architecture.md) et sur les [méthodes de routage du trafic](front-door-routing-methods.md) de Front Door.

## <a name="regional-load-balancing"></a>Équilibrage de charge régional
Application Gateway propose Application Delivery Controller (ADC) en tant que service, mettant ainsi plusieurs fonctionnalités d’équilibrage de charge de couche 7 à la disposition de votre application. Elle permet aux clients d’optimiser la productivité de la batterie de serveurs web en déchargeant une terminaison TLS gourmande en ressources du processeur vers la passerelle Application Gateway. Parmi les fonctionnalités supplémentaires de routage de couche 7, citons également la distribution en tourniquet (round robin) du trafic entrant, l’affinité de session basée sur les cookies, le routage basé sur le chemin des URL et la possibilité d’héberger plusieurs sites Web derrière une seule passerelle Application Gateway. Vous pouvez configurer Application Gateway en tant que point de terminaison accessible sur Internet, point de terminaison interne uniquement ou une combinaison des deux. Application Gateway est entièrement gérée par Azure, ce qui vous permet de bénéficier de son évolutivité et de sa haute disponibilité. Elle fournit un ensemble complet de fonctionnalités de diagnostics et de journalisation, pour une meilleure gérabilité.

Load Balancer fait partie intégrante de la pile Azure SDN, ce qui vous permet de bénéficier de services d’équilibrage de charge de couche 4 hautement performants et à faible latence pour tous les protocoles UDP et TCP. Vous pouvez configurer des points de terminaison publics ou à charge équilibrée interne en définissant des règles qui mappent les connexions entrantes aux pools de backend. Avec la surveillance de la détection d’intégrité au moyen du protocole TCP ou HTTPS, cela peut vous aider à gérer la disponibilité du service.

## <a name="choosing-a-global-load-balancer"></a>Choix d’un équilibreur de charge global
Quand vous devez choisir qui de Traffic Manager ou d’Azure Front Door utiliser comme équilibreur de charge global pour le routage global, vous devez examiner ce qui est similaire et ce qui est différent dans les deux services.   Ces deux services fournissent ce qui suit :
- **Géoredondance multiple :** si une région subit une panne, le trafic est acheminé vers la région la plus proche sans interruption et sans aucune intervention du propriétaire de l’application.
- **Routage vers la région plus proche :** le trafic est acheminé automatiquement vers la région la plus proche.

</br>Le tableau suivant décrit les différences entre Traffic Manager et Azure Front Door :</br>

| Traffic Manager | Azure Front Door |
| --------------- | ------------------------ |
|**N'importe quel protocole :** Comme Traffic Manager fonctionne au niveau de la couche DNS, vous pouvez acheminer tous les types de trafic réseau : HTTP, TCP, UDP, etc. | **Accélération HTTP :** avec Front Door, le trafic est en proxy à la périphérie du réseau Microsoft. Les requêtes HTTP/S verront des améliorations de la latence et du débit, ce qui réduit la latence de la négociation TLS.|
|**Routage local :** avec le routage au niveau d’une couche DNS, le trafic va toujours de point à point.  Le routage de votre filiale vers votre centre de données local peut prendre un chemin direct, même sur votre propre réseau à l’aide de Traffic Manager. | **Extensibilité indépendante :** comme Front Door fonctionne avec la requête HTTP, les requêtes pour différents chemins d’URL peuvent être routées vers différents pools de backends / services régionaux (microservices) en fonction de règles et de l’intégrité de chaque microservice d’application.|
|**Format de facturation :** la facturation basée sur DNS est mise à l’échelle en fonction du nombre d’utilisateurs et, pour les services à davantage d’utilisateurs, se stabilise afin de réduire le coût. |**Sécurité incluse :** Front Door active des règles telles que la limitation du débit et les listes ACL d’adresses IP pour vous permettre de protéger vos backends avant que le trafic n’atteigne votre application. 

</br>Nous recommandons aux clients d’utiliser Front Door pour leurs charges de travail HTTP en raison des avantages offerts par Front Door en termes de performances, d’opérabilité et de sécurité pour les charges de travail HTTP. Traffic Manager et Front Door peuvent être utilisés en parallèle afin de servir tout le trafic pour votre application. 

## <a name="building-with-azures-application-delivery-suite"></a>Création avec la suite de livraison d’application Azure 
Nous vous recommandons de faire en sorte que tous les services, sites Web et API soient géographiquement redondants afin qu’ils distribuent le trafic aux utilisateurs à partir de l’emplacement le plus proche dans la mesure du possible.  La combinaison de plusieurs services d’équilibrage de charge vous permet de créer une redondance géographique et zonale pour optimiser la fiabilité et les performances.

Dans le diagramme suivant, nous décrivons un exemple d’architecture qui utilise une combinaison de tous ces services pour créer un service Web global. L’architecte a utilisé Traffic Manager pour acheminer le trafic vers les serveurs backend pour la remise de fichiers et d’objets. Lors de l’utilisation de Front Door, pour acheminer globalement les chemins URL correspondant au modèle/store/* vers le service qu’ils ont migré vers App Service. Enfin, acheminer toutes les autres requêtes vers des passerelles Application Gateway régionales.

Dans chaque région de service IaaS, le développeur de l’application a décidé que toutes les URL correspondant au modèle /images /* seraient desservies par un pool dédié de machines virtuelles. Ce pool de machines virtuelles est différent du reste de la batterie de serveurs Web.

Par ailleurs, le pool de machines virtuelles par défaut affichant le contenu dynamique doit communiquer avec une base de données principale hébergée sur un cluster à haute disponibilité. Le déploiement est entièrement configuré via Azure Resource Manager.

Le diagramme suivant illustre l’architecture de ce scénario :

:::image type="content" source="./media/front-door-lb-with-azure-app-delivery-suite/application-delivery-figure2.png" alt-text="Architecture détaillée de la suite de livraison d’application":::

> [!NOTE]
> Cet exemple est l’une des innombrables configurations possibles offertes par les services d’équilibrage de charge Azure. Les services Traffic Manager, Front Door, Application Gateway et Load Balancer peuvent être combinés et associés pour répondre au mieux à vos besoins d’équilibrage de charge. Par exemple, s’il n’est pas nécessaire de faire appel au déchargement TLS/SSL ou au traitement de la couche 7, Load Balancer peut être utilisé à la place d’Application Gateway.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
