---
title: Utilisation du VPN site à site comme solution de secours pour le Peering privé Azure ExpressRoute | Microsoft Docs
description: Cette page fournit des conseils sur l’architecture d’une connexion de secours pour le Peering privé Azure ExpressRoute à l’aide d’un VPN site à site.
services: networking
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/05/2020
ms.author: duau
ms.openlocfilehash: 752edea8078cf55fc3965bdc7aa9e1b4269dee34
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207918"
---
# <a name="using-s2s-vpn-as-a-backup-for-expressroute-private-peering"></a>Utilisation du VPN site à site comme solution de secours pour le Peering privé ExpressRoute

Dans l’article intitulé [Conception pour une récupération d’urgence avec le Peering privé ExpressRoute][DR-PP], nous avons abordé la nécessité de disposer d’une solution de connectivité de secours pour la connectivité de Peering privé ExpressRoute et de l’utilisation de circuits ExpressRoute géoredondants à cet effet. Cet article explique comment tirer parti d’un VPN site à site (S2S) et le gérer en guise de solution de secours pour le Peering privé ExpressRoute. 

Contrairement aux circuits ExpressRoute géoredondants, vous pouvez utiliser la combinaison de récupération d’urgence ExpressRoute-VPN uniquement en mode actif-passif. L’une des principales difficultés de l’utilisation d’une connectivité réseau de secours en mode passif est que la connexion passive échoue souvent en même temps que la connexion principale. L’absence de maintenance active est la raison la plus fréquente des échecs de la connexion passive. Par conséquent, dans cet article, nous allons nous concentrer sur la manière de vérifier et d’entretenir activement la connectivité VPN S2S comme solution de secours d’un Peering privé ExpressRoute.

>[!NOTE] 
>Lorsqu’un itinéraire donné est publié via ExpressRoute et VPN, Azure préfère le routage via ExpressRoute.  
>

Dans cet article, nous allons voir comment vérifier la connectivité à la fois du point de vue d’Azure et du point de vue de la périphérie du réseau côté client. La possibilité de valider d’un côté comme de l’autre vous aidera, que vous gériez ou non les périphériques réseau côté client qui sont connectés par Peering avec les entités de réseau Microsoft. 

## <a name="example-topology"></a>Exemple de topologie

Dans notre configuration, nous disposons d’un réseau local connecté à un réseau virtuel hub Azure via un circuit ExpressRoute et une connexion VPN S2S. Le réseau virtuel hub Azure est à son tour connecté par Peering à un réseau virtuel spoke, comme indiqué dans le diagramme ci-dessous :

![1][1]

Dans la configuration, le circuit ExpressRoute se termine sur une paire de routeurs « Périphérie du client » (CE) localement. Le LAN local est connecté aux routeurs CE par le biais d’une paire de pare-feu qui fonctionnent en mode « leader-suiveur ». Le VPN S2S s’arrête directement sur les pare-feu.

Le tableau suivant répertorie les principaux préfixes d’adresse IP de la topologie :

| **Entité** | **Préfixe** |
| --- | --- |
| LAN local | 10.1.11.0/25 |
| Réseau virtuel hub Azure | 10.17.11.0/25 |
| Réseau virtuel spoke Azure | 10.17.11.128/26 |
| Serveur de test local | 10.1.11.10 |
| Machine virtuelle de test du réseau virtuel spoke | 10.17.11.132 |
| Sous-réseau P2P de la connexion principale ExpressRoute | 192.168.11.16/30 |
| Sous-réseau P2P de la connexion secondaire ExpressRoute | 192.168.11.20/30 |
| Adresse IP d’homologue BGP principale de la passerelle VPN | 10.17.11.76 |
| Adresse IP d’homologue BGP secondaire de la passerelle VPN | 10.17.11.77 |
| Adresse IP d’homologue BGP du VPN du pare-feu local | 192.168.11.88 |
| Adresse IP de l’interface du routeur CE principal vers le pare-feu | 192.168.11.0/31 |
| Adresse IP de l’interface du pare-feu vers le routeur CE principal | 192.168.11.1/31 |
| Adresse IP de l’interface du routeur CE secondaire vers le pare-feu | 192.168.11.2/31 |
| Adresse IP de l’interface du pare-feu vers le routeur CE secondaire | 192.168.11.3/31 |


Le tableau suivant répertorie les ASN de la topologie :

| **Système autonome** | **ASN** |
| --- | --- |
| Local | 65020 |
| Microsoft Enterprise Edge | 12076 |
| GW Réseau virtuel (ExR) | 65515 |
| GW Réseau virtuel (VPN) | 65515 |

## <a name="high-availability-without-asymmetricity"></a>Haute disponibilité sans asymétrie

### <a name="configuring-for-high-availability"></a>Configuration pour la haute disponibilité

[Configurer la coexistence de connexions de site à site et ExpressRoute][Conf-CoExist] explique comment configurer la coexistence du circuit ExpressRoute et des connexions VPN S2S. Comme nous l’avons vu dans [Conception pour une haute disponibilité avec ExpressRoute][HA], afin d’améliorer la haute disponibilité d’ExpressRoute, notre configuration gère la redondance du réseau (évite un point de défaillance unique) jusqu’aux points de terminaison. En outre, les connexions principales et secondaires des circuits ExpressRoute sont configurées pour fonctionner en mode actif-actif en publiant les préfixes locaux de la même façon via les deux connexions. 

La publication d’itinéraires locaux par le routeur CE principal via la connexion principale du circuit ExpressRoute est illustrée ci-dessous (commandes Juno) :

```console
user@SEA-MX03-01> show route advertising-protocol bgp 192.168.11.18 

Cust11.inet.0: 8 destinations, 8 routes (7 active, 0 holddown, 1 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

La publication d’itinéraires locaux par le routeur CE secondaire via la connexion secondaire du circuit ExpressRoute est illustrée ci-dessous (commandes Juno) :

```console
user@SEA-MX03-02> show route advertising-protocol bgp 192.168.11.22 

Cust11.inet.0: 8 destinations, 8 routes (7 active, 0 holddown, 1 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

Pour améliorer la haute disponibilité de la connexion de secours, le VPN S2S est également configuré en mode actif-actif. La configuration de la passerelle VPN Azure est indiquée ci-dessous. Notez que, dans le cadre de la configuration du VPN, les adresses IP d’homologue BGP de la passerelle (10.17.11.76 et 10.17.11.77) sont également répertoriées.

![2][2]

L’itinéraire local est publié par les pare-feu sur l’homologue BGP principal et secondaire de la passerelle VPN. Les publications d’itinéraire sont affichées ci-dessous (Junos) :

```console
user@SEA-SRX42-01> show route advertising-protocol bgp 10.17.11.76 

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I

{primary:node0}
user@SEA-SRX42-01> show route advertising-protocol bgp 10.17.11.77    

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.1.11.0/25            Self                                    I
```

>[!NOTE] 
>La configuration du VPN S2S en mode actif-actif fournit non seulement une haute disponibilité à votre connectivité réseau de secours pour la récupération d’urgence, mais lui fournit également un débit plus élevé. En d’autres termes, la configuration du VPN S2S en mode actif-actif est recommandée, car cela force la création de plusieurs tunnels sous-jacents.
>

### <a name="configuring-for-symmetric-traffic-flow"></a>Configuration du flux de trafic symétrique

Nous avons noté que lorsqu’un itinéraire local donné est publié via ExpressRoute et VPN S2S, Azure préfère le chemin ExpressRoute. Pour forcer Azure à préférer le chemin VPN S2S aux circuits ExpressRoute coexistants, vous devez publier des itinéraires plus spécifiques (préfixe plus long avec un masque de sous-réseau plus grand) via la connexion VPN. Notre objectif ici est d’utiliser les connexions VPN en guise de solution de secours uniquement. Par conséquent, le comportement de sélection du chemin d’accès par défaut d’Azure est conforme à notre objectif. 

Il est de notre responsabilité de veiller à ce que le trafic destiné à Azure à partir d’un emplacement local privilégie également le chemin ExpressRoute au VPN S2S. La préférence locale par défaut des routeurs CE et des pare-feu dans notre configuration locale est 100. Par conséquent, en configurant la préférence locale des itinéraires reçus via les Peerings privés ExpressRoute supérieurs à 100 (par exemple, 150), nous pouvons faire en sorte que le trafic destiné à Azure préfère le circuit ExpressRoute à l’état stable.

La configuration BGP du routeur CE principal qui termine la connexion principale du circuit ExpressRoute est illustrée ci-dessous. Notez que la préférence locale des itinéraires publiés via la session iBGP est configurée sur la valeur 150. De même, nous devons nous assurer que la préférence locale du routeur CE secondaire qui termine la connexion secondaire du circuit ExpressRoute est également configurée sur 150.

```console
user@SEA-MX03-01> show configuration routing-instances Cust11
description "Customer 11 VRF";
instance-type virtual-router;
interface xe-0/0/0:0.110;
interface ae0.11;
protocols {
  bgp {
    group ibgp {
        type internal;
        local-preference 150;
        neighbor 192.168.11.1;
    }
    group ebgp {
        peer-as 12076;
        bfd-liveness-detection {
            minimum-interval 300;
            multiplier 3;
        }
        neighbor 192.168.11.18;
    }
  }
}
```

La table de route des pare-feu locaux confirme (illustré ci-dessous) que, pour le trafic local destiné à Azure, le chemin d’accès préféré est via ExpressRoute à l’état stable.

```console
user@SEA-SRX42-01> show route table Cust11.inet.0 10.17.11.0/24

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.17.11.0/25      *[BGP/170] 2d 00:34:04, localpref 150
                      AS path: 12076 I, validation-state: unverified
                    > to 192.168.11.0 via reth1.11
                      to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 00:34:01, localpref 150
                      AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 21:12:13, localpref 100, from 10.17.11.76
                       AS path: 65515 I, validation-state: unverified
                    > via st0.118
                    [BGP/170] 2d 00:41:51, localpref 100, from 10.17.11.77
                       AS path: 65515 I, validation-state: unverified
                     > via st0.119
10.17.11.76/32     *[Static/5] 2d 21:12:16
                     > via st0.118
10.17.11.77/32     *[Static/5] 2d 00:41:56
                    > via st0.119
10.17.11.128/26    *[BGP/170] 2d 00:34:04, localpref 150
                       AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.0 via reth1.11
                       to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 00:34:01, localpref 150
                      AS path: 12076 I, validation-state: unverified
                     > to 192.168.11.2 via reth2.11
                    [BGP/170] 2d 21:12:13, localpref 100, from 10.17.11.76
                       AS path: 65515 I, validation-state: unverified
                    > via st0.118
                     [BGP/170] 2d 00:41:51, localpref 100, from 10.17.11.77
                       AS path: 65515 I, validation-state: unverified
                     > via st0.119
```

Dans la table de route ci-dessus, pour les routes de réseau virtuel hub et spoke (10.17.11.0/25 et 10.17.11.128/26), nous voyons que le circuit ExpressRoute est préféré aux connexions VPN. Les adresses 192.168.11.0 et 192.168.11.2 sont des adresses IP sur l’interface de pare-feu vers les routeurs CE.

## <a name="validation-of-route-exchange-over-s2s-vpn"></a>Validation de l’échange d’itinéraires via un VPN S2S

Plus haut dans cet article, nous avons vérifié la publication d’itinéraires locaux par les pare-feu sur l’homologue BGP principal et secondaire de la passerelle VPN. Par ailleurs, nous allons confirmer les itinéraires Azure reçus par les pare-feu de l’homologue BGP principal et secondaire de la passerelle VPN.

```console
user@SEA-SRX42-01> show route receive-protocol bgp 10.17.11.76 table Cust11.inet.0 

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  10.17.11.0/25           10.17.11.76                             65515 I
  10.17.11.128/26         10.17.11.76                             65515 I

{primary:node0}
user@SEA-SRX42-01> show route receive-protocol bgp 10.17.11.77 table Cust11.inet.0    

Cust11.inet.0: 14 destinations, 21 routes (14 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
  10.17.11.0/25           10.17.11.77                             65515 I
  10.17.11.128/26         10.17.11.77                             65515 I
```

De même, nous allons vérifier les préfixes d’itinéraire du réseau local reçus par la passerelle VPN Azure. 

```powershell
PS C:\Users\user> Get-AzVirtualNetworkGatewayLearnedRoute -ResourceGroupName SEA-Cust11 -VirtualNetworkGatewayName SEA-Cust11-VNet01-gw-vpn | where {$_.Network -eq "10.1.11.0/25"} | select Network, NextHop, AsPath, Weight

Network      NextHop       AsPath      Weight
-------      -------       ------      ------
10.1.11.0/25 192.168.11.88 65020        32768
10.1.11.0/25 10.17.11.76   65020        32768
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 192.168.11.88 65020        32768
10.1.11.0/25 10.17.11.77   65020        32768
10.1.11.0/25 10.17.11.69   12076-65020  32769
10.1.11.0/25 10.17.11.69   12076-65020  32769
```

Comme indiqué ci-dessus, les itinéraires de la passerelle VPN sont reçus par l’homologue BGP principal et secondaire de la passerelle VPN. La passerelle VPN offre également une visibilité sur les itinéraires reçus via les connexions ExpressRoute principale et secondaire (celles avec AS-path précédé de 12076). Pour confirmer les itinéraires reçus via des connexions VPN, nous devons connaître l’adresse IP locale d’homologue BGP des connexions. Dans la configuration que nous étudions, il s’agit de 192.168.11.88 et nous voyons bien les routes reçues.

Ensuite, nous allons vérifier les itinéraires publiés par la passerelle VPN Azure vers l’homologue BGP de pare-feu local (192.168.11.88).

```powershell
PS C:\Users\user> Get-AzVirtualNetworkGatewayAdvertisedRoute -Peer 192.168.11.88 -ResourceGroupName SEA-Cust11 -VirtualNetworkGatewayName SEA-Cust11-VNet01-gw-vpn |  select Network, NextHop, AsPath, Weight

Network         NextHop     AsPath Weight
-------         -------     ------ ------
10.17.11.0/25   10.17.11.76 65515       0
10.17.11.128/26 10.17.11.76 65515       0
10.17.11.0/25   10.17.11.77 65515       0
10.17.11.128/26 10.17.11.77 65515       0
```

Le fait de ne pas voir les échanges d’itinéraires indique un échec de la connexion. Consultez [Résolution des problèmes : la connexion VPN site à site Azure cesse de fonctionner][VPN Troubleshoot] pour plus d’informations sur la résolution des problèmes de connexion VPN.

## <a name="testing-failover"></a>Test du basculement

Maintenant que nous avons confirmé le succès des échanges d’itinéraires via la connexion VPN (plan de contrôle), nous sommes prêts à basculer le trafic (plan de données) de la connectivité ExpressRoute à la connectivité VPN. 

>[!NOTE] 
>Dans les environnements de production, les tests de basculement doivent être effectués pendant les fenêtres programmées de maintenance du réseau, car ils peuvent perturber le service.
>

Avant de procéder au basculement du trafic, traçons le chemin d’accès actuel dans notre configuration du serveur de test local jusqu’à la machine virtuelle de test dans le réseau virtuel spoke.

```console
C:\Users\PathLabUser>tracert 10.17.11.132

Tracing route to 10.17.11.132 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.1.11.1
  2    <1 ms    <1 ms    11 ms  192.168.11.0
  3    <1 ms    <1 ms    <1 ms  192.168.11.18
  4     *        *        *     Request timed out.
  5     6 ms     6 ms     5 ms  10.17.11.132

Trace complete.
```

Les sous-réseaux de connexion point à point ExpressRoute principal et secondaire de notre configuration sont, respectivement, 192.168.11.16/30 et 192.168.11.20/30. Dans la détermination de l’itinéraire ci-dessus, nous voyons à l’étape 3 que nous avons atteint 192.168.11.18, qui est l’adresse IP de l’interface du MSEE principal. La présence de l’interface MSEE confirme que, comme prévu, notre chemin d’accès actuel passe par ExpressRoute.

Comme indiqué dans la section [Réinitialiser les Peerings d’un circuit ExpressRoute][RST], utilisons les commandes PowerShell suivantes pour désactiver le Peering principal et secondaire du circuit ExpressRoute.

```powershell
$ckt = Get-AzExpressRouteCircuit -Name "expressroute name" -ResourceGroupName "SEA-Cust11"
$ckt.Peerings[0].State = "Disabled"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Le temps de basculement dépend du temps de convergence BGP. Dans notre configuration, le basculement prend quelques secondes (moins de 10). Après le basculement, la répétition de la détermination de l’itinéraire affiche le chemin d’accès suivant :

```console
C:\Users\PathLabUser>tracert 10.17.11.132

Tracing route to 10.17.11.132 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.1.11.1
  2     *        *        *     Request timed out.
  3     6 ms     7 ms     9 ms  10.17.11.132

Trace complete.
```

Le résultat de la détermination de l’itinéraire confirme que la connexion de secours via le VPN S2S est active et peut assurer la continuité du service en cas d’échec des connexions ExpressRoute principale et secondaire. Pour effectuer le test de basculement, réactivez les connexions ExpressRoute et normalisez le flux de trafic à l’aide de l’ensemble de commandes suivant.

```powershell
$ckt = Get-AzExpressRouteCircuit -Name "expressroute name" -ResourceGroupName "SEA-Cust11"
$ckt.Peerings[0].State = "Enabled"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Pour confirmer que le trafic est rétabli sur ExpressRoute, répétez la détermination de l’itinéraire et assurez-vous que ce dernier passe par le Peering privé ExpressRoute.

## <a name="next-steps"></a>Étapes suivantes

ExpressRoute est conçu pour la haute disponibilité sans point de défaillance unique au sein du réseau Microsoft. Un circuit ExpressRoute est toujours limité à une seule région géographique et à un seul fournisseur de services. Le VPN S2S peut être une bonne solution de secours passive pour la récupération d’urgence à un circuit ExpressRoute. Pour garantir la fiabilité d’une solution de connexion de secours passive, il est important d’effectuer régulièrement un entretien de la configuration passive et une validation de la connexion. Il est essentiel de ne pas laisser la configuration VPN devenir obsolète et de répéter régulièrement (par exemple, chaque trimestre) les étapes de test de validation et de basculement décrites dans cet article pendant la fenêtre de maintenance.

Pour activer la surveillance et les alertes basées sur les métriques de passerelle VPN, consultez [Configurer des alertes sur les métriques de passerelle VPN][VPN-alerts].

Pour accélérer la convergence BGP suite à un échec d’ExpressRoute, [configurez BFD sur ExpressRoute][BFD].

<!--Image References-->
[1]: ./media/use-s2s-vpn-as-backup-for-expressroute-privatepeering/topology.png "topologie étudiée"
[2]: ./media/use-s2s-vpn-as-backup-for-expressroute-privatepeering/vpn-gw-config.png "Configuration de GW VPN"

<!--Link References-->
[DR-PP]: ./designing-for-disaster-recovery-with-expressroute-privatepeering.md
[Conf-CoExist]: ./expressroute-howto-coexist-resource-manager.md
[HA]: ./designing-for-high-availability-with-expressroute.md
[VPN Troubleshoot]: ../vpn-gateway/vpn-gateway-troubleshoot-site-to-site-cannot-connect.md
[VPN-alerts]: ../vpn-gateway/vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md
[BFD]: ./expressroute-bfd.md
[RST]: ./expressroute-howto-reset-peering.md