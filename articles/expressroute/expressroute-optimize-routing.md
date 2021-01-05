---
title: 'Azure ExpressRoute : Optimiser le routage'
description: Cette page fournit des détails sur l’optimisation du routage lorsque vous avez le choix entre plusieurs circuits ExpressRoute de connexion entre Microsoft et votre réseau professionnel.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 07/11/2019
ms.author: duau
ms.openlocfilehash: f35f1d390762d3f83176d7b36db8959dc5ed0157
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204875"
---
# <a name="optimize-expressroute-routing"></a>Optimiser le routage ExpressRoute
En présence de plusieurs circuits ExpressRoute, vous pouvez vous connecter à Microsoft par le biais de plusieurs chemins d’accès. Par conséquent, le routage pourrait ne pas être optimal. Autrement dit, votre trafic peut emprunter un chemin d’accès plus long pour atteindre Microsoft ; Microsoft peut faire de même pour atteindre votre réseau. Plus le chemin d’accès réseau est long, plus la latence est élevée. La latence a un impact direct sur les performances d’application ainsi que sur l’expérience utilisateur. Cet article aborde ce problème et explique comment optimiser le routage à l’aide des technologies de routage standard.

## <a name="path-selection-on-microsoft-and-public-peerings"></a>Sélection du chemin sur les peerings Microsoft et publics
Il est important de s’assurer que lors de l’utilisation du peering Microsoft ou public, le trafic transite par le chemin souhaité si vous disposez d’un ou de plusieurs circuits ExpressRoute, ainsi que de chemins d’accès à Internet via un échange Internet (IX) ou un fournisseur de services Internet (ISP). BGP utilise un algorithme de sélection de chemin optimal basé sur un certain nombre de facteurs, notamment la correspondance de préfixe la plus longue. Pour s’assurer que le trafic destiné à Azure via le peering Microsoft ou public traverse le chemin ExpressRoute, les clients doivent implémenter l’attribut *Local Preference* pour s’assurer que le chemin est toujours préféré sur ExpressRoute. 

> [!NOTE]
> La préférence locale par défaut est généralement 100. Les préférences locales plus élevées sont privilégiées. 
>
>

Examinez l’exemple de scénario suivant :

![Diagramme illustrant le scénario ExpressRoute n° 1 : routage non optimal entre le client et Microsoft](./media/expressroute-optimize-routing/expressroute-localPreference.png)

Dans l’exemple ci-dessus, pour préférer les chemins ExpressRoute, configurez la préférence locale comme suit. 

**Configuration Cisco IOS-XE du point de vue R1 :**

- R1(config)#route-map prefer-ExR permit 10
- R1(config-route-map)#set local-preference 150

- R1(config)#router BGP 345
- R1(config-router)#neighbor 1.1.1.2 remote-as 12076
- R1(config-router)#neighbor 1.1.1.2 activate
- R1(config-router)#neighbor 1.1.1.2 route-map prefer-ExR in

**Configuration Junos du point de vue R1 :**

- user@R1# set protocols bgp group ibgp type internal
- user@R1# set protocols bgp group ibgp local-preference 150



## <a name="suboptimal-routing-from-customer-to-microsoft"></a>Routage non optimal entre le client et Microsoft
Voici un exemple de problème de routage. Imaginez que vous disposez de deux bureaux aux États-Unis, un à Los Angeles et un à New York. Vos bureaux sont connectés sur un réseau étendu (WAN), qui peut être votre propre réseau principal ou le VPN IP de votre fournisseur de services. Vous avez deux circuits ExpressRoute également connectés sur le réseau étendu, l’un dans USA Ouest et l’autre dans USA Est. Vous avez bien évidemment deux chemins d’accès pour vous connecter au réseau Microsoft. Imaginez à présent que vous déployiez un service Azure (par exemple, Azure App Service) dans les régions USA Ouest et USA Est. Vous avez pour objectif de connecter vos utilisateurs de Los Angeles à la région Azure USA Ouest et les utilisateurs de New York à la région Azure USA Est, car selon votre administrateur de service, les utilisateurs de chaque bureau doivent accéder aux services Azure les plus proches pour une expérience optimale. Malheureusement, le plan fonctionne bien pour les utilisateurs de la côte Est, mais pas pour ceux de la côte Ouest. La cause du problème est la suivante. Sur chaque circuit ExpressRoute, nous publions le préfixe dans la région Azure USA Est (23.100.0.0/16) et dans la région Azure USA Ouest (13.100.0.0/16). Si vous ne savez pas à quelle région correspond chaque préfixe, vous ne pourrez pas le traiter différemment. Votre réseau WAN peut penser que les deux préfixes sont plus proches des régions USA Est et USA Ouest et donc router les utilisateurs des deux bureaux vers le circuit ExpressRoute dans la région USA Est. Au final, vous risquez d’avoir des utilisateurs mécontents dans vos bureaux de Los Angeles.

![Scénario ExpressRoute n° 1 : routage non optimal entre le client et Microsoft](./media/expressroute-optimize-routing/expressroute-case1-problem.png)

### <a name="solution-use-bgp-communities"></a>Solution : utilisez des communautés BGP
Pour optimiser le routage pour les utilisateurs des deux bureaux, vous devez savoir faire la différence entre le préfixe de la région Azure USA Ouest et celui de la région Azure USA Est. Nous encodons ces informations à l’aide des [valeurs de communauté BGP](expressroute-routing.md). Nous avons affecté une valeur de Communauté BGP unique à chaque région Azure, par exemple : « 12076:51004 » pour USA Est et « 12076:51006 » pour USA Ouest. À présent que vous savez à quelles régions Azure correspondent les préfixes, vous pouvez configurer les préférences de circuit ExpressRoute. Étant donné que nous utilisons le protocole BGP pour échanger des informations de routage, vous pouvez utiliser les préférences locales du protocole BGP pour influencer le routage. Dans notre exemple, vous pouvez affecter, dans la région USA Ouest, une valeur de préférence locale supérieure à celle de la région USA Est (13.100.0.0/16) et vice-versa dans la région USA Est (23.100.0.0/16). Cette configuration permet de garantir que, lorsque les deux chemins d’accès à Microsoft sont disponibles, vos utilisateurs de Los Angeles prendront le circuit ExpressRoute dans la région USA Ouest pour se connecter à Azure dans cette région, tandis que vos utilisateurs de New York prendront le circuit ExpressRoute dans la région USA Est vers Azure dans cette région. Le routage est optimisé des deux côtés. 

![Solution ExpressRoute n° 1 : utilisez des communautés BGP](./media/expressroute-optimize-routing/expressroute-case1-solution.png)

> [!NOTE]
> La même technique peut être appliquée pour le routage entre le client et le réseau virtuel Azure, en utilisant les préférences locales. Nous ne balisons pas la valeur de communauté BGP sur les préfixes publiés à partir d’Azure sur votre réseau. Toutefois, étant donné que vous savez à quel déploiement de réseau virtuel correspond quel bureau, vous pouvez configurer vos routeurs en conséquence pour privilégier un circuit ExpressRoute plutôt qu’un autre.
>
>

## <a name="suboptimal-routing-from-microsoft-to-customer"></a>Routage non optimal entre Microsoft et le client
Voici un autre exemple dans lequel les connexions en provenance de Microsoft prennent un chemin plus long pour atteindre votre réseau. Dans ce cas, vous utilisez des serveurs Exchange locaux et Exchange Online dans un [environnement hybride](/exchange/exchange-hybrid). Vos bureaux sont connectés à un réseau étendu. Vous publiez vers Microsoft les préfixes des serveurs locaux de vos deux bureaux par le biais des deux circuits ExpressRoute. Dans certains cas, tels que la migration de boîte aux lettres, Exchange Online lance les connexions aux serveurs locaux. Malheureusement, la connexion à votre bureau de Los Angeles est routée au circuit ExpressRoute dans la région USA Est, puis retraverse l’ensemble du pays avant d’atteindre la côte ouest. L’origine du problème est similaire au premier cas. En l’absence d’indicateur, le réseau Microsoft ne peut pas déterminer quel préfixe est le plus proche de la côte Ouest et de la côte Est. Malheureusement, il choisit le mauvais chemin vers votre bureau de Los Angeles.

![Scénario ExpressRoute n° 2 : routage non optimal entre Microsoft et le client](./media/expressroute-optimize-routing/expressroute-case2-problem.png)

### <a name="solution-use-as-path-prepending"></a>Solution : ajoutez le préfixe AS PATH
Il existe deux solutions à ce problème. La première consiste simplement à publier votre préfixe local pour le bureau de Los Angeles (177.2.0.0/31) sur le circuit ExpressRoute dans la région USA Ouest et celui pour le bureau de New York (177.2.0.2/31) sur le circuit ExpressRoute dans la région USA Est. Par conséquent, Microsoft ne dispose que d’un seul chemin d’accès pour se connecter à chacun de vos sites. Il n’existe aucune ambiguïté et le routage est optimisé. Avec cette conception, vous devez réfléchir à votre stratégie de basculement. Dans le cas où le chemin d’accès à Microsoft via ExpressRoute est interrompu, vous devez vous assurer qu’Exchange Online peut toujours se connecter à vos serveurs locaux. 

La deuxième solution est de continuer à publier les deux préfixes sur les deux circuits ExpressRoute, tout en nous indiquant quel préfixe correspond à chacun de vos bureaux. Étant donné que nous prenons en charge l’ajout de préfixe AS PATH BGP, vous pouvez configurer l’AS PATH pour influencer le routage. Dans cet exemple, vous pouvez allonger le chemin AS PATH pour 172.2.0.0/31 dans la région USA Est pour que nous privilégiions le circuit ExpressRoute dans la région USA Ouest pour le trafic destiné à ce préfixe ; notre réseau pensera que le chemin d’accès pour ce préfixe est plus court dans l’Ouest. De même, vous pouvez allonger le chemin AS PATH pour 172.2.0.2/31 dans la région USA Ouest pour que nous privilégiions le circuit ExpressRoute dans la région USA Est. Le routage est optimisé pour les deux bureaux. Grâce à cette conception, si un circuit ExpressRoute est défaillant, Exchange Online peut toujours vous joindre via un autre circuit ExpressRoute ainsi que par le biais de votre réseau étendu. 

> [!IMPORTANT]
> Nous supprimons les numéros AS privés dans le chemin AS PATH pour les préfixes reçus dans le cadre de l’appairage Microsoft lors d’un appairage avec un numéro AS privé. Vous devez effectuer un appairage avec un numéro AS public et ajouter des numéros AS publics au chemin AS PATH pour influencer le routage pour l’appairage Microsoft.
> 
> 

![Solution ExpressRoute n° 2 : ajoutez le préfixe AS PATH](./media/expressroute-optimize-routing/expressroute-case2-solution.png)

> [!NOTE]
> Bien que les exemples fournis ici s’appliquent aux peerings Microsoft et publics, nous prenons en charge les mêmes fonctionnalités pour le peering privé. En outre, le préfixe AS Path fonctionne au sein d’un même circuit ExpressRoute pour influencer la sélection des chemins principaux et secondaires.
> 
> 

## <a name="suboptimal-routing-between-virtual-networks"></a>Routage non optimal entre des réseaux virtuels
Avec ExpressRoute, vous pouvez établir une communication réseau virtuel-réseau virtuel (également appelée « VNet ») en les reliant à un circuit ExpressRoute. Lorsque vous les reliez à plusieurs circuits ExpressRoute, un routage non optimal peut se produire entre les réseaux virtuels. Prenons un exemple. Vous disposez de deux circuits ExpressRoute, l’un dans USA Ouest et l’autre dans USA Est. Dans chaque région, vous avez deux réseaux virtuels. Vos serveurs web sont déployés sur un réseau virtuel et les serveurs d’applications sur l’autre. À des fins de redondance, vous reliez les deux réseaux virtuels de chaque région à la fois au circuit ExpressRoute local et au circuit ExpressRoute distant. Comme illustré ci-dessous, chaque réseau virtuel est doté de deux chemins d’accès menant vers l’autre réseau virtuel. Les réseaux virtuels ne font pas la distinction entre le circuit ExpressRoute local et le circuit ExpressRoute distant. Par conséquent, étant donné qu’ils appliquent une stratégie de routage ECMP (Equal-cost multi-path routing) pour équilibrer la charge du trafic entre les réseaux virtuels, certains flux de trafic emprunteront le chemin le plus long et seront routés au niveau du circuit ExpressRoute distant.

![Scénario ExpressRoute n° 3 : routage non optimal entre réseaux virtuels](./media/expressroute-optimize-routing/expressroute-case3-problem.png)

### <a name="solution-assign-a-high-weight-to-local-connection"></a>Solution : attribuer plus de poids à la connexion locale
La solution est simple. Étant donné que vous savez où se trouvent les réseaux virtuels et les circuits, vous pouvez nous indiquer le chemin à privilégier pour chaque réseau virtuel. Spécifiquement pour cet exemple, vous attribuez plus de poids à la connexion locale qu’à la connexion à distance (consultez l’exemple de configuration [ici](expressroute-howto-linkvnet-arm.md#modify-a-virtual-network-connection)). Lorsqu’un réseau virtuel reçoit le préfixe de l’autre réseau virtuel sur plusieurs connexions, il préfère la connexion avec le plus de poids pour envoyer le trafic destiné à ce préfixe.

![Solution ExpressRoute n° 3 : attribuer plus de poids à la connexion locale](./media/expressroute-optimize-routing/expressroute-case3-solution.png)

> [!NOTE]
> Vous pouvez également modifier entre le réseau virtuel et votre réseau sur site, si vous disposez de plusieurs circuits ExpressRoute, en configurant le poids attribué à une connexion au lien d’ajouter un préfixe AS PATH, technique décrite dans le second scénario ci-dessus. Pour chaque préfixe, nous examinerons toujours le poids attribué à la connexion avant la longueur du chemin AS pour décider du mode d’envoi du trafic.
>
>