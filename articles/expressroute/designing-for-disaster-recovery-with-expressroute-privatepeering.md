---
title: 'Azure ExpressRoute : Conception pour la récupération d’urgence'
description: Cette page fournit des conseils sur l’architecture pour une reprise d’activité lors de l’utilisation d’Azure ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 05/25/2019
ms.author: duau
ms.openlocfilehash: 2a5730cd75ccb76d25897e9109555113f7355c2f
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92202411"
---
# <a name="designing-for-disaster-recovery-with-expressroute-private-peering"></a>Conception pour une reprise d’activité avec le peering privé ExpressRoute

ExpressRoute est conçu pour la haute disponibilité afin de fournir à l’opérateur une connectivité de réseau privé de qualité aux ressources Microsoft. En d’autres termes, il n’existe aucun point de défaillance unique dans le chemin d’accès ExpressRoute au sein du réseau de Microsoft. Pour des considérations de conception visant à optimiser la disponibilité d’un circuit ExpressRoute, consultez [Conception pour une haute disponibilité avec ExpressRoute][HA].

Toutefois, prenant en considération l’adage populaire de Murphy, selon lequel *si quelque chose peut mal tourner, c’est ce qui va arriver*, nous nous concentrons dans cet article sur des solutions qui vont au-delà des défaillances qui peuvent être traitées à l’aide d’un simple circuit ExpressRoute. En d’autres termes, dans cet article, nous allons nous intéresser à l’architecture des réseaux du point de vue de la création d’une connectivité réseau back-end robuste favorisant la reprise d’activité à l’aide de circuits ExpressRoute géoredondants.

>[!NOTE]
>Les concepts décrits dans cet article s’appliquent tout autant lorsqu’un circuit ExpressRoute est créé sous Virtual WAN ou à l’extérieur de celui-ci.
>

## <a name="need-for-redundant-connectivity-solution"></a>Nécessité d’une solution de connectivité redondante

Certaines situations peuvent favoriser la dégradation d’un service régional entier (que ce soit au niveau de Microsoft, des fournisseurs de services réseau, des clients ou d’autres fournisseurs de services cloud). Une catastrophe naturelle peut-être la cause première de ce type d’impact sur un service à l’échelle régionale. Pour la continuité des activités et les applications stratégiques, il est donc important de planifier la reprise d’activité.   

Que vous exécutiez vos applications stratégiques dans une région Azure, localement ou n’importe où ailleurs, vous pouvez utiliser une autre région Azure comme site de basculement. Les articles suivants abordent la reprise d’activité du point de vue des applications et de l’accès front-end :

- [Reprise d’activité à l’échelle de l’entreprise][Enterprise DR]
- [Reprise d’activité des TPE/PME avec Azure Site Recovery][SMB DR]

Si vous vous appuyez sur la connectivité ExpressRoute entre votre réseau local et Microsoft pour vos opérations stratégiques, votre plan de reprise d’activité doit également inclure une connectivité réseau géoredondante. 

## <a name="challenges-of-using-multiple-expressroute-circuits"></a>Défis liés à l’utilisation de plusieurs circuits ExpressRoute

Quand vous interconnectez le même ensemble de réseaux à l’aide de plusieurs connexions, vous introduisez des chemins parallèles entre les réseaux. Quand ils ne sont pas correctement conçus, les chemins parallèles peuvent engendrer un routage asymétrique. Si le chemin comporte des entités avec état (par exemple, NAT, pare-feu), le routage asymétrique risque de bloquer le flux de trafic.  En règle générale, sur le chemin de peering privé ExpressRoute, vous ne rencontrez pas d’entités avec état, telles que NAT ou un pare-feu. Ainsi, le routage asymétrique sur le peering privé ExpressRoute ne bloque pas nécessairement le flux de trafic.
 
Toutefois, si vous équilibrez la charge du trafic entre des chemin parallèles géoredondants, qu’il existe ou non des entités avec état, vous pouvez observer des performances réseau incohérentes. Dans cet article, nous allons aborder la façon de relever ces défis.

## <a name="small-to-medium-on-premises-network-considerations"></a>Considérations relatives aux réseaux locaux petits ou moyens

Penchons-nous sur l’exemple de réseau illustré dans le diagramme suivant. Dans l’exemple, une connectivité ExpressRoute géoredondante est établie entre un emplacement local de Contoso et un réseau virtuel de Contoso dans une région Azure. Dans le diagramme, la ligne verte pleine indique le chemin préféré (via ExpressRoute 1), tandis que la ligne verte en pointillé représente le chemin de secours (via ExpressRoute 2).

[![1]][1]

Quand vous concevez une connectivité ExpressRoute pour la reprise d’activité, vous devez envisager ce qui suit :

- Utiliser des circuits ExpressRoute géoredondants
- Utiliser divers réseaux de fournisseur de service pour le circuit ExpressRoute différent
- Concevoir chaque circuit ExpressRoute pour la [haute disponibilité][HA]
- Terminer le circuit ExpressRoute différent à un emplacement différent sur le réseau du client

Par défaut, si vous publiez des routes identiques sur tous les chemins ExpressRoute, Azure équilibre la charge liée au trafic local entre tous les chemins ExpressRoute à l’aide du routage multichemin à coût égal (ECMP).

Toutefois, avec les circuits ExpressRoute géoredondants, nous devons tenir compte de la différence des performances réseau d’un chemin réseau à l’autre (en particulier du point de vue de la latence du réseau). Pour obtenir des performances réseau plus cohérentes pendant un fonctionnement normal, vous pouvez préférer le circuit ExpressRoute qui offre la latence minimale.

Vous pouvez influer sur Azure afin qu’il préfère un circuit ExpressRoute à un autre en utilisant l’une des techniques suivantes (listées par ordre d’efficacité) :

- Publier une route plus spécifique via le circuit ExpressRoute préféré par rapport aux autres circuits ExpressRoute
- Configurer une pondération de connexion supérieure sur la connexion qui relie le réseau virtuel au circuit ExpressRoute préféré
- Publier les routes via un circuit ExpressRoute moins préféré avec un chemin AS plus long (ajout au chemin AS)

### <a name="more-specific-route"></a>Route plus spécifique

Le diagramme suivant montre comment influer sur la sélection d’un chemin ExpressRoute à l’aide d’une publication de route plus spécifique. Dans l’exemple illustré, la plage d’adresses IP /24 du réseau local Contoso est publiée sous la forme de deux plages d’adresses /25 via le chemin d’accès préféré (ExpressRoute 1) et d’une plage /24 via le chemin de secours (ExpressRoute 2).

[![2]][2]

/25 étant plus spécifique, par rapport à /24, Azure envoie le trafic destiné à 10.1.11.0/24 via ExpressRoute 1 dans un état normal. Si les deux connexions d’ExpressRoute 1 tombent en panne, le réseau virtuel voit la publication de la route 10.1.11.0/24 uniquement par le biais d’ExpressRoute 2 ; ainsi, le circuit de secours est utilisé dans cet état d’échec.

### <a name="connection-weight"></a>Pondération de connexion

La capture d’écran suivante illustre la configuration de la pondération d’une connexion ExpressRoute via le portail Azure.

[![3]][3]

Le diagramme suivant montre comment influer sur la sélection d’un chemin ExpressRoute à l’aide de la pondération de connexion. La pondération de connexion par défaut est 0. Dans l’exemple ci-dessous, la pondération de connexion pour ExpressRoute 1 est configurée sur la valeur 100. Quand un réseau virtuel reçoit un préfixe de route publié par le biais de plusieurs circuits ExpressRoute, il préfère la connexion ayant la pondération la plus élevée.

[![4]][4]

Si les deux connexions d’ExpressRoute 1 tombent en panne, le réseau virtuel voit la publication de la route 10.1.11.0/24 uniquement par le biais d’ExpressRoute 2 ; ainsi, le circuit de secours est utilisé dans cet état d’échec.

### <a name="as-path-prepend"></a>Ajout au chemin AS

Le diagramme suivant montre comment influer sur la sélection d’un chemin ExpressRoute à l’aide d’un ajout au chemin AS. Dans le diagramme, la publication des routes via ExpressRoute 1 indique le comportement par défaut du mode eBGP. Sur la publication des routes via ExpressRoute 2, le numéro ASN du réseau local est ajouté au chemin AS de la route. Quand la même route est reçue via plusieurs circuits ExpressRoute, selon le processus de sélection de route eBGP, le réseau virtuel préfère la route ayant le chemin AS le plus court. 

[![5]][5]

Si les deux connexions d’ExpressRoute 1 tombent en panne, le réseau virtuel voit la publication de la route 10.1.11.0/24 uniquement par le biais d’ExpressRoute 2. Ainsi, le chemin AS plus long devient superflu. Le circuit de secours est donc utilisé dans cet état d’échec.

En utilisant l’une ou l’autre des techniques, si vous amenez Azure à préférer un de vos chemins ExpressRoute, vous devez vous assurer que le réseau local préfère également le même chemin ExpressRoute pour le trafic lié à Azure afin d’éviter les flux asymétriques. En règle générale, la valeur de préférence locale est utilisée pour amener le réseau local à préférer un circuit ExpressRoute. La préférence locale est une métrique iBGP (BGP interne). La route BGP ayant la valeur de préférence locale la plus élevée est préférée.

> [!IMPORTANT]
> Quand vous utilisez certains circuits ExpressRoute en guise de circuits de secours, vous devez les gérer activement et tester régulièrement l’opération de basculement. 
> 

## <a name="large-distributed-enterprise-network"></a>Grand réseau d’entreprise distribué

Quand vous avez un grand réseau d’entreprise distribué, vous êtes susceptible d’avoir plusieurs circuits ExpressRoute. Dans cette section, nous allons voir comment concevoir la reprise d’activité à l’aide de circuits ExpressRoute en mode actif-actif, sans avoir besoin de circuits de secours supplémentaires. 

Penchons-nous sur l’exemple illustré dans le diagramme suivant. Dans l’exemple, Contoso dispose de deux emplacements locaux connectés à deux déploiements IaaS Contoso dans deux régions Azure différentes via des circuits ExpressRoute dans deux emplacements de peering différents. 

[![6]][6]

La façon dont nous architecturons la reprise d’activité a un impact sur la façon dont est routé le trafic depuis les régions vers les emplacements (région 1/région 2 vers emplacement 2/emplacement1). Prenons l’exemple de deux architectures de reprise qui routent différemment le trafic depuis les régions vers les emplacements.

### <a name="scenario-1"></a>Scénario 1

Dans le premier scénario, nous concevons la reprise d’activité afin que tout le trafic entre une région Azure et un réseau local emprunte le circuit ExpressRoute local dans l’état stable. Si le circuit ExpressRoute local échoue, le circuit ExpressRoute distant est utilisé pour tous les flux de trafic entre Azure et le réseau local.

Le scénario 1 est illustré dans le diagramme suivant. Dans le diagramme, les lignes vertes indiquent les chemins pour le flux de trafic entre les réseaux VNet1 et locaux. Les lignes bleues indiquent les chemins pour le flux de trafic entre les réseaux VNet2 et locaux. Les lignes pleines indiquent le chemin souhaité dans l’état stable, tandis que les lignes en pointillés indiquent le chemin du trafic en cas de défaillance du circuit ExpressRoute correspondant par lequel transite le flux de trafic dans l’état stable. 

[![7]][7]

Vous pouvez concevoir le scénario au moyen de la pondération de connexion afin que les réseaux virtuels préfèrent la connexion au circuit ExpressRoute de l’emplacement de peering local pour le trafic lié au réseau local. Pour compléter la solution, vous devez garantir un flux de trafic inverse symétrique. Vous pouvez utiliser la préférence locale sur la session iBGP entre vos routeurs BGP (sur lesquels les circuits ExpressRoute sont terminés du côté local) pour préférer un circuit ExpressRoute. La solution est illustrée dans le diagramme suivant. 

[![8]][8]

### <a name="scenario-2"></a>Scénario 2

Le scénario 2 est illustré dans le diagramme suivant. Dans le diagramme, les lignes vertes indiquent les chemins pour le flux de trafic entre les réseaux VNet1 et locaux. Les lignes bleues indiquent les chemins pour le flux de trafic entre les réseaux VNet2 et locaux. Dans l’état stable (lignes pleines dans le diagramme), tout le trafic entre les réseaux virtuels et les emplacements emprunte essentiellement l’infrastructure Microsoft dorsale, et ne transite par l’interconnexion entre les emplacements locaux que si un circuit ExpressRoute se trouve en état d’échec (lignes en pointillés dans le diagramme).

[![9]][9]

La solution est illustrée dans le diagramme suivant. Comme illustré, vous pouvez concevoir le scénario en utilisant une route plus spécifique (option 1) ou l’ajout au chemin AS (option 2) pour influer sur la sélection du chemin pour les réseaux virtuels. Pour influer sur la sélection des routes des réseaux locaux pour le trafic lié à Azure, vous devez configurer l’interconnexion entre l’emplacement local comme étant moins préférable. La façon dont vous configurez le lien d’interconnexion comme étant préférable varie selon le protocole de routage utilisé au sein du réseau local. Vous pouvez utiliser la préférence locale avec iBGP ou une métrique avec IGP (OSPF ou IS-IS).

[![10]][10]


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons abordé la conception de la reprise d’activité d’une connectivité de peering privé dans les circuits ExpressRoute. Les articles suivants abordent la reprise d’activité du point de vue des applications et de l’accès front-end :

- [Reprise d’activité à l’échelle de l’entreprise][Enterprise DR]
- [Reprise d’activité des TPE/PME avec Azure Site Recovery][SMB DR]

<!--Image References-->
[1]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/one-region.png "Considérations relatives aux réseaux locaux petits ou moyens"
[2]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/specificroute.png "Influer sur la sélection du chemin en utilisant des routes plus spécifiques"
[3]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/configure-weight.png "Configurer la pondération de connexion par le biais du portail Azure"
[4]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/connectionweight.png "Influer sur la sélection du chemin en utilisant la pondération de connexion"
[5]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/aspath.png "Influer sur la sélection du chemin en utilisant l’ajout au chemin AS"
[6]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region.png "Considérations relatives aux grands réseaux locaux distribués"
[7]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-arch1.png "Scénario 1"
[8]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-sol1.png "Circuits ExpressRoute en mode actif-actif - Solution 1"
[9]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-arch2.png "Scénario 2"
[10]: ./media/designing-for-disaster-recovery-with-expressroute-pvt/multi-region-sol2.png "Circuits ExpressRoute en mode actif-actif - Solution 2"

<!--Link References-->
[HA]: ./designing-for-high-availability-with-expressroute.md
[Enterprise DR]: https://azure.microsoft.com/solutions/architecture/disaster-recovery-enterprise-scale-dr/
[SMB DR]: https://azure.microsoft.com/solutions/architecture/disaster-recovery-smb-azure-site-recovery/
[con wgt]: ./expressroute-optimize-routing.md#solution-assign-a-high-weight-to-local-connection
[AS Path Pre]: ./expressroute-optimize-routing.md#solution-use-as-path-prepending