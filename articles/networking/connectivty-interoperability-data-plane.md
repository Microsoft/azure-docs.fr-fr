---
title: 'Interopérabilité dans Azure : Analyse du plan de données'
description: Cet article fournit l’analyse du plan de données de l’initialisation (tearDown) de test que vous pouvez utiliser pour analyser l’interopérabilité entre ExpressRoute, un réseau privé virtuel (VPN) de site à site et le peering de réseau virtuel dans Azure.
documentationcenter: na
services: networking
author: rambk
manager: tracsman
ms.service: virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/18/2018
ms.author: rambala
ms.openlocfilehash: 635a8fc5409e18da9529763b06e4a531a36d0156
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86169202"
---
# <a name="interoperability-in-azure--data-plane-analysis"></a>Interopérabilité dans Azure : Analyse du plan de données

Cet article décrit l’analyse du plan de données de l’[initialisation (tearDown) de test][Setup]. Vous pouvez également consulter la [configuration de l’initialisation (tearDown) de test][Configuration] et l’[analyse du plan de contrôle][Control-Analysis] de l’initialisation (tearDown) de test.

L’analyse du plan de données examine le chemin emprunté par les paquets qui vont d’un réseau local (LAN ou réseau virtuel) à l’autre au sein d’une topologie. Le chemin des données entre deux réseaux locaux n’est pas nécessairement symétrique. Dans cet article, nous analysons donc le chemin de transfert d’un réseau local vers un autre réseau qui est distinct du chemin inverse.

## <a name="data-path-from-the-hub-vnet"></a>Chemin des données à partir du réseau virtuel hub

### <a name="path-to-the-spoke-vnet"></a>Chemin vers le réseau virtuel spoke

Le peering de réseau virtuel émule les fonctionnalités de pont réseau entre les deux réseaux virtuels qui sont appairés. La sortie de détermination d’itinéraire d’un réseau virtuel hub vers une machine virtuelle dans le réseau virtuel spoke est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.4

Tracing route to 10.11.30.4 over a maximum of 30 hops

  1     2 ms     1 ms     1 ms  10.11.30.4

Trace complete.
```

La figure suivante présente la vue graphique de la connexion entre le réseau virtuel hub et le réseau virtuel spoke du point de vue d’Azure Network Watcher :


![1][1]

### <a name="path-to-the-branch-vnet"></a>Chemin vers le réseau virtuel branch

La sortie de détermination d’itinéraire d’un réseau virtuel hub vers une machine virtuelle dans le réseau virtuel branch est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.68

Tracing route to 10.11.30.68 over a maximum of 30 hops

  1     1 ms     1 ms     1 ms  10.10.30.142
  2     *        *        *     Request timed out.
  3     2 ms     2 ms     2 ms  10.11.30.68

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est la passerelle VPN de la passerelle VPN Azure du réseau virtuel hub. Le deuxième tronçon est la passerelle VPN du réseau virtuel branch. L’adresse IP de la passerelle VPN du réseau virtuel branch n’est pas publiée dans le réseau virtuel hub. Le troisième tronçon est la machine virtuelle dans le réseau virtuel branch.

La figure suivante présente la vue graphique de la connexion entre le réseau virtuel hub et le réseau virtuel branch du point de vue de Network Watcher :

![2][2]

Pour la même connexion, la figure suivante montre le mode Grille dans Network Watcher :

![3][3]

### <a name="path-to-on-premises-location-1"></a>Chemin vers l’emplacement Location 1 local

La sortie de détermination d’itinéraire d’un réseau virtuel hub vers une machine virtuelle dans l’emplacement Location 1 local est illustrée ici :

```console
C:\Users\rb>tracert 10.2.30.10

Tracing route to 10.2.30.10 over a maximum of 30 hops

  1     2 ms     2 ms     2 ms  10.10.30.132
  2     *        *        *     Request timed out.
  3     *        *        *     Request timed out.
  4     2 ms     2 ms     2 ms  10.2.30.10

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle Azure ExpressRoute pour un routeur de périphérie d’entreprise (MSEE, Microsoft Enterprise Edge). Le deuxième et le troisième tronçons sont le routeur de périphérie du client (CE) et les adresses IP du réseau local Location 1 local. Ces adresses IP ne sont pas publiées dans le réseau virtuel hub. Le quatrième tronçon est la machine virtuelle dans l’emplacement Location 1 local.


### <a name="path-to-on-premises-location-2"></a>Chemin vers l’emplacement Location 2 local

La sortie de détermination d’itinéraire d’un réseau virtuel hub vers une machine virtuelle dans l’emplacement Location 2 local est illustrée ici :

```console
C:\Users\rb>tracert 10.1.31.10

Tracing route to 10.1.31.10 over a maximum of 30 hops

  1    76 ms    75 ms    75 ms  10.10.30.134
  2     *        *        *     Request timed out.
  3     *        *        *     Request timed out.
  4    75 ms    75 ms    75 ms  10.1.31.10

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle ExpressRoute vers un routeur MSEE. Le deuxième et le troisième tronçons sont le routeur CE et les adresses IP du réseau local Location 2 local. Ces adresses IP ne sont pas publiées dans le réseau virtuel hub. Le quatrième tronçon est la machine virtuelle sur l’emplacement Location 2 local.

### <a name="path-to-the-remote-vnet"></a>Chemin vers le réseau virtuel distant

La sortie de détermination d’itinéraire d’un réseau virtuel hub vers une machine virtuelle dans le réseau virtuel distant est illustrée ici :

```console
C:\Users\rb>tracert 10.17.30.4

Tracing route to 10.17.30.4 over a maximum of 30 hops

  1     2 ms     2 ms     2 ms  10.10.30.132
  2     *        *        *     Request timed out.
  3    69 ms    68 ms    69 ms  10.17.30.4

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle ExpressRoute vers un routeur MSEE. Le deuxième tronçon est l’adresse IP de la passerelle du réseau virtuel distant. La plage d’adresses IP du deuxième tronçon n’est pas publiée dans le réseau virtuel hub. Le troisième tronçon est la machine virtuelle sur le réseau virtuel distant.

## <a name="data-path-from-the-spoke-vnet"></a>Chemin des données à partir du réseau virtuel spoke

Le réseau virtuel spoke partage la vue du réseau virtuel hub. Par le biais du peering de réseau virtuel, le réseau virtuel spoke utilise la connectivité de la passerelle distante du réseau virtuel hub comme si sa connexion au réseau virtuel spoke était directe.

### <a name="path-to-the-hub-vnet"></a>Chemin vers le réseau virtuel hub

La sortie de détermination d’itinéraire du réseau virtuel spoke vers une machine virtuelle dans le réseau virtuel hub est illustrée ici :

```console
C:\Users\rb>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.10.30.4

Trace complete.
```

### <a name="path-to-the-branch-vnet"></a>Chemin vers le réseau virtuel branch

La sortie de détermination d’itinéraire du réseau virtuel spoke vers une machine virtuelle dans le réseau virtuel branch est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.68

Tracing route to 10.11.30.68 over a maximum of 30 hops

  1     1 ms    <1 ms    <1 ms  10.10.30.142
  2     *        *        *     Request timed out.
  3     3 ms     2 ms     2 ms  10.11.30.68

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est la passerelle VPN du réseau virtuel hub. Le deuxième tronçon est la passerelle VPN du réseau virtuel branch. L’adresse IP de la passerelle VPN du réseau virtuel branch n’est pas publiée dans le réseau virtuel hub/spoke. Le troisième tronçon est la machine virtuelle dans le réseau virtuel branch.

### <a name="path-to-on-premises-location-1"></a>Chemin vers l’emplacement Location 1 local

La sortie de détermination d’itinéraire du réseau virtuel spoke vers une machine virtuelle dans l’emplacement Location 1 local est illustrée ici :

```console
C:\Users\rb>tracert 10.2.30.10

Tracing route to 10.2.30.10 over a maximum of 30 hops

  1    24 ms     2 ms     3 ms  10.10.30.132
  2     *        *        *     Request timed out.
  3     *        *        *     Request timed out.
  4     3 ms     2 ms     2 ms  10.2.30.10

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle ExpressRoute du réseau virtuel hub à un routeur MSEE. Le deuxième et le troisième tronçons sont le routeur CE et les adresses IP du réseau local Location 1 local. Ces adresses IP ne sont pas publiées dans le réseau virtuel hub/spoke. Le quatrième tronçon est la machine virtuelle dans l’emplacement Location 1 local.

### <a name="path-to-on-premises-location-2"></a>Chemin vers l’emplacement Location 2 local

La sortie de détermination d’itinéraire du réseau virtuel spoke vers une machine virtuelle dans l’emplacement Location 2 local est illustrée ici :

```console
C:\Users\rb>tracert 10.1.31.10

Tracing route to 10.1.31.10 over a maximum of 30 hops

  1    76 ms    75 ms    76 ms  10.10.30.134
  2     *        *        *     Request timed out.
  3     *        *        *     Request timed out.
  4    75 ms    75 ms    75 ms  10.1.31.10

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle ExpressRoute du réseau virtuel hub à un routeur MSEE. Le deuxième et le troisième tronçons sont le routeur CE et les adresses IP du réseau local Location 2 local. Ces adresses IP ne sont pas publiées sur les réseaux virtuels hub/spoke. Le quatrième tronçon est la machine virtuelle dans l’emplacement Location 2 local.

### <a name="path-to-the-remote-vnet"></a>Chemin vers le réseau virtuel distant

La sortie de détermination d’itinéraire du réseau virtuel spoke vers une machine virtuelle dans le réseau virtuel distant est illustrée ici :

```console
C:\Users\rb>tracert 10.17.30.4

Tracing route to 10.17.30.4 over a maximum of 30 hops

  1     2 ms     1 ms     1 ms  10.10.30.133
  2     *        *        *     Request timed out.
  3    71 ms    70 ms    70 ms  10.17.30.4

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est le point de terminaison du tunnel de la passerelle ExpressRoute du réseau virtuel hub à un routeur MSEE. Le deuxième tronçon est l’adresse IP de la passerelle du réseau virtuel distant. La plage d’adresses IP du deuxième tronçon n’est pas publiée dans le réseau virtuel hub/spoke. Le troisième tronçon est la machine virtuelle sur le réseau virtuel distant.

## <a name="data-path-from-the-branch-vnet"></a>Chemin des données à partir du réseau virtuel branch

### <a name="path-to-the-hub-vnet"></a>Chemin vers le réseau virtuel hub

La sortie de détermination d’itinéraire du réseau virtuel branch vers une machine virtuelle dans le réseau virtuel hub est illustrée ici :

```console
C:\Windows\system32>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.11.30.100
  2     *        *        *     Request timed out.
  3     4 ms     3 ms     3 ms  10.10.30.4

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est la passerelle VPN du réseau virtuel branch. Le deuxième tronçon est la passerelle VPN du réseau virtuel hub. L’adresse IP de la passerelle VPN du réseau virtuel hub n’est pas publiée dans le réseau virtuel distant. Le troisième tronçon est la machine virtuelle sur le réseau virtuel hub.

### <a name="path-to-the-spoke-vnet"></a>Chemin vers le réseau virtuel spoke

La sortie de détermination d’itinéraire du réseau virtuel branch vers une machine virtuelle dans le réseau virtuel spoke est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.4

Tracing route to 10.11.30.4 over a maximum of 30 hops

  1     1 ms    <1 ms     1 ms  10.11.30.100
  2     *        *        *     Request timed out.
  3     4 ms     3 ms     2 ms  10.11.30.4

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est la passerelle VPN du réseau virtuel branch. Le deuxième tronçon est la passerelle VPN du réseau virtuel hub. L’adresse IP de la passerelle VPN du réseau virtuel hub n’est pas publiée dans le réseau virtuel distant. Le troisième tronçon est la machine virtuelle sur le réseau virtuel spoke.

### <a name="path-to-on-premises-location-1"></a>Chemin vers l’emplacement Location 1 local

La sortie de détermination d’itinéraire du réseau virtuel branch vers une machine virtuelle dans l’emplacement Location 1 local est illustrée ici :

```console
C:\Users\rb>tracert 10.2.30.10

Tracing route to 10.2.30.10 over a maximum of 30 hops

  1     1 ms    <1 ms    <1 ms  10.11.30.100
  2     *        *        *     Request timed out.
  3     3 ms     2 ms     2 ms  10.2.30.125
  4     *        *        *     Request timed out.
  5     3 ms     3 ms     3 ms  10.2.30.10

Trace complete.
```

Dans cette détermination d’itinéraire, le premier tronçon est la passerelle VPN du réseau virtuel branch. Le deuxième tronçon est la passerelle VPN du réseau virtuel hub. L’adresse IP de la passerelle VPN du réseau virtuel hub n’est pas publiée dans le réseau virtuel distant. Le troisième tronçon est le point de terminaison du tunnel VPN sur le routeur CE principal. Le quatrième tronçon est une adresse IP interne de l’emplacement Location 1 local. Cette adresse IP de réseau local n’est pas publiée en dehors du routeur CE. Le cinquième tronçon est la machine virtuelle de destination dans l’emplacement Location 1 local.

### <a name="path-to-on-premises-location-2-and-the-remote-vnet"></a>Chemin vers l’emplacement Location 2 local et le réseau virtuel distant

Comme nous l’avons indiqué dans l’analyse du plan de contrôle, le réseau virtuel branch n’a aucune visibilité sur l’emplacement Location 2 local ou sur le réseau virtuel distant selon la configuration réseau. Les résultats de test ping suivants le confirment : 

```console
C:\Users\rb>ping 10.1.31.10

Pinging 10.1.31.10 with 32 bytes of data:

Request timed out.
...
Request timed out.

Ping statistics for 10.1.31.10:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

C:\Users\rb>ping 10.17.30.4

Pinging 10.17.30.4 with 32 bytes of data:

Request timed out.
...
Request timed out.

Ping statistics for 10.17.30.4:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

## <a name="data-path-from-on-premises-location-1"></a>Chemin des données à partir de l’emplacement Location 1 local

### <a name="path-to-the-hub-vnet"></a>Chemin vers le réseau virtuel hub

La sortie de détermination d’itinéraire d’un emplacement Location 1 local vers une machine virtuelle dans le réseau virtuel hub est illustrée ici :

```console
C:\Users\rb>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.2.30.3
  2    <1 ms    <1 ms    <1 ms  192.168.30.0
  3    <1 ms    <1 ms    <1 ms  192.168.30.18
  4     *        *        *     Request timed out.
  5     2 ms     2 ms     2 ms  10.10.30.4

Trace complete.
```

Dans cette détermination d’itinéraire, les deux premiers tronçons font partie du réseau local. Le troisième tronçon est l’interface MSEE principale qui est accessible sur le routeur CE. Le quatrième tronçon est la passerelle ExpressRoute du réseau virtuel hub. La plage d’adresses IP de la passerelle ExpressRoute du réseau virtuel hub n’est pas publiée sur le réseau local. Le cinquième tronçon est la machine virtuelle de destination.

Network Watcher fournit uniquement une vue centrée sur Azure. D’un point de vue local, nous utilisons Azure Network Performance Monitor. Network Performance Monitor fournit des agents que vous pouvez installer sur des serveurs dans des réseaux en dehors d’Azure pour l’analyse du chemin des données.

La figure suivante présente la vue de la topologie de la connectivité de la machine virtuelle Location 1 locale à la machine virtuelle sur le réseau virtuel hub via ExpressRoute :

![4][4]

Comme indiqué précédemment, l’initialisation (tearDown) de test utilise un VPN de site à site comme connectivité de secours pour ExpressRoute entre l’emplacement Location 1 local et le réseau virtuel hub. Pour tester le chemin des données de sauvegarde, nous allons provoquer un échec de la liaison ExpressRoute entre le routeur CE principal de l’emplacement Location 1 local et le composant MSEE correspondant. Pour provoquer l’échec d’une liaison ExpressRoute, arrêtez l’interface CE qui est accessible sur le composant MSEE :

```console
C:\Users\rb>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.2.30.3
  2    <1 ms    <1 ms    <1 ms  192.168.30.0
  3     3 ms     2 ms     3 ms  10.10.30.4

Trace complete.
```

La figure suivante présente la vue de la topologie de la connectivité de la machine virtuelle Location 1 locale à la machine virtuelle sur le réseau virtuel hub via la connectivité VPN de site à site quand la connectivité ExpressRoute est à l’arrêt :

![5][5]

### <a name="path-to-the-spoke-vnet"></a>Chemin vers le réseau virtuel spoke

La sortie de détermination d’itinéraire d’un emplacement Location 1 local vers une machine virtuelle dans le réseau virtuel spoke est illustrée ici :

Rétablissons la connectivité principale ExpressRoute pour effectuer l’analyse du chemin des données vers le réseau virtuel spoke :

```console
C:\Users\rb>tracert 10.11.30.4

Tracing route to 10.11.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.2.30.3
  2    <1 ms    <1 ms    <1 ms  192.168.30.0
  3    <1 ms    <1 ms    <1 ms  192.168.30.18
  4     *        *        *     Request timed out.
  5     3 ms     2 ms     2 ms  10.11.30.4

Trace complete.
```

Établissons la connectivité ExpressRoute 1 principale pour le reste de l’analyse du chemin des données.

### <a name="path-to-the-branch-vnet"></a>Chemin vers le réseau virtuel branch

La sortie de détermination d’itinéraire d’un emplacement Location 1 local vers une machine virtuelle dans le réseau virtuel branch est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.68

Tracing route to 10.11.30.68 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.2.30.3
  2    <1 ms    <1 ms    <1 ms  192.168.30.0
  3     3 ms     2 ms     2 ms  10.11.30.68

Trace complete.
```

### <a name="path-to-on-premises-location-2"></a>Chemin vers l’emplacement Location 2 local

Comme nous l’avons indiqué dans l’[analyse du plan de contrôle][Control-Analysis], l’emplacement Location 1 local n’a aucune visibilité sur l’emplacement Location 2 local selon la configuration réseau. Les résultats de test ping suivants le confirment : 

```console
C:\Users\rb>ping 10.1.31.10

Pinging 10.1.31.10 with 32 bytes of data:

Request timed out.
...
Request timed out.

Ping statistics for 10.1.31.10:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

### <a name="path-to-the-remote-vnet"></a>Chemin vers le réseau virtuel distant

La sortie de détermination d’itinéraire d’un emplacement Location 1 local vers une machine virtuelle dans le réseau virtuel distant est illustrée ici :

```console
C:\Users\rb>tracert 10.17.30.4

Tracing route to 10.17.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.2.30.3
  2     2 ms     5 ms     7 ms  192.168.30.0
  3    <1 ms    <1 ms    <1 ms  192.168.30.18
  4     *        *        *     Request timed out.
  5    69 ms    70 ms    69 ms  10.17.30.4

Trace complete.
```

## <a name="data-path-from-on-premises-location-2"></a>Chemin des données à partir de l’emplacement Location 2 local

### <a name="path-to-the-hub-vnet"></a>Chemin vers le réseau virtuel hub

La sortie de détermination d’itinéraire d’un emplacement Location 2 local vers une machine virtuelle dans le réseau virtuel hub est illustrée ici :

```console
C:\Windows\system32>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    <1 ms    <1 ms    <1 ms  10.1.31.3
  2    <1 ms    <1 ms    <1 ms  192.168.31.4
  3    <1 ms    <1 ms    <1 ms  192.168.31.22
  4     *        *        *     Request timed out.
  5    75 ms    74 ms    74 ms  10.10.30.4

Trace complete.
```

### <a name="path-to-the-spoke-vnet"></a>Chemin vers le réseau virtuel spoke

La sortie de détermination d’itinéraire d’un emplacement Location 2 local vers une machine virtuelle dans le réseau virtuel spoke est illustrée ici :

```console
C:\Windows\system32>tracert 10.11.30.4

Tracing route to 10.11.30.4 over a maximum of 30 hops
  1    <1 ms    <1 ms     1 ms  10.1.31.3
  2    <1 ms    <1 ms    <1 ms  192.168.31.0
  3    <1 ms    <1 ms    <1 ms  192.168.31.18
  4     *        *        *     Request timed out.
  5    75 ms    74 ms    74 ms  10.11.30.4

Trace complete.
```

### <a name="path-to-the-branch-vnet-on-premises-location-1-and-the-remote-vnet"></a>Chemin du réseau virtuel branch, de l’emplacement Location 1 local et du réseau virtuel distant

Comme nous l’avons indiqué dans l’[analyse du plan de contrôle][Control-Analysis], l’emplacement Location 1 local n’a aucune visibilité sur le réseau virtuel branche, l’emplacement Location 1 local ou le réseau virtuel distant selon la configuration réseau. 

## <a name="data-path-from-the-remote-vnet"></a>Chemin des données à partir du réseau virtuel distant

### <a name="path-to-the-hub-vnet"></a>Chemin vers le réseau virtuel hub

La sortie de détermination d’itinéraire d’un réseau virtuel distant vers une machine virtuelle dans le réseau virtuel hub est illustrée ici :

```console
C:\Users\rb>tracert 10.10.30.4

Tracing route to 10.10.30.4 over a maximum of 30 hops

  1    65 ms    65 ms    65 ms  10.17.30.36
  2     *        *        *     Request timed out.
  3    69 ms    68 ms    68 ms  10.10.30.4

Trace complete.
```

### <a name="path-to-the-spoke-vnet"></a>Chemin vers le réseau virtuel spoke

La sortie de détermination d’itinéraire du réseau virtuel distant vers une machine virtuelle dans le réseau virtuel spoke est illustrée ici :

```console
C:\Users\rb>tracert 10.11.30.4

Tracing route to 10.11.30.4 over a maximum of 30 hops

  1    67 ms    67 ms    67 ms  10.17.30.36
  2     *        *        *     Request timed out.
  3    71 ms    69 ms    69 ms  10.11.30.4

Trace complete.
```

### <a name="path-to-the-branch-vnet-and-on-premises-location-2"></a>Chemin du réseau virtuel branch et de l’emplacement Location 2 local

Comme nous l’avons indiqué dans l’[analyse du plan de contrôle][Control-Analysis], le réseau virtuel distant n’a aucune visibilité sur le réseau virtuel branche ni sur l’emplacement Location 2 local selon la configuration réseau. 

### <a name="path-to-on-premises-location-1"></a>Chemin vers l’emplacement Location 1 local

La sortie de détermination d’itinéraire du réseau virtuel distant vers une machine virtuelle dans l’emplacement Location 1 local est illustrée ici :

```console
C:\Users\rb>tracert 10.2.30.10

Tracing route to 10.2.30.10 over a maximum of 30 hops

  1    67 ms    67 ms    67 ms  10.17.30.36
  2     *        *        *     Request timed out.
  3     *        *        *     Request timed out.
  4    69 ms    69 ms    69 ms  10.2.30.10

Trace complete.
```

## <a name="expressroute-and-site-to-site-vpn-connectivity-in-tandem"></a>Connectivité ExpressRoute et VPN de site à site en tandem

###  <a name="site-to-site-vpn-over-expressroute"></a>VPN de site à site sur ExpressRoute

Vous pouvez configurer un VPN de site à site à l’aide du peering Microsoft ExpressRoute pour échanger des données de façon privée entre votre réseau local et vos réseaux virtuels Azure. Avec cette configuration, vous pouvez échanger des données en garantissant confidentialité, authenticité et intégrité. L’échange de données est également soumis à un système anti-relecture. Pour plus d’informations sur la configuration d’un VPN IPsec de site à site en mode tunnel via l’homologation Microsoft ExpressRoute, consultez l’article [Configurer un réseau VPN de site à site via le Peering Microsoft ExpressRoute][S2S-Over-ExR]. 

La principale limitation liée à la configuration d’un VPN de site à site qui utilise le peering Microsoft est le débit. Le débit sur le tunnel IPsec est limité par la capacité de la passerelle VPN. Le débit d’une passerelle VPN est inférieur au débit ExpressRoute. Dans ce scénario, le fait d’utiliser le tunnel IPsec pour un trafic très sécurisé et d’utiliser le peering privé pour toutes les autres catégories de trafic permet d’optimiser l’utilisation de la bande passante ExpressRoute.

### <a name="site-to-site-vpn-as-a-secure-failover-path-for-expressroute"></a>VPN de site à site en tant que chemin de basculement sécurisé pour ExpressRoute

ExpressRoute est utilisé comme une paire de circuits redondants pour garantir une haute disponibilité. Vous pouvez configurer une connectivité ExpressRoute géo-redondante dans différentes régions Azure. En outre, comme nous l’avons démontré dans notre initialisation (tearDown) de test, dans une région Azure, vous pouvez utiliser un VPN de site à site pour créer un chemin de basculement pour votre connectivité ExpressRoute. Quand les mêmes préfixes sont publiés sur ExpressRoute et un VPN de site à site, Azure choisit en priorité ExpressRoute. Pour éviter un routage asymétrique entre ExpressRoute et le VPN de site à site, la configuration réseau locale doit en faire autant en utilisant la connectivité ExpressRoute avant la connectivité VPN de site à site.

Pour plus d’informations sur la façon de configurer des connexions coexistantes pour ExpressRoute et un VPN de site à site, consultez [Coexistence de connexions ExpressRoute et de site à site][ExR-S2S-CoEx].

## <a name="extend-back-end-connectivity-to-spoke-vnets-and-branch-locations"></a>Étendre la connectivité de back-end à des réseaux virtuels spoke et à des emplacements branch

### <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>Connectivité de réseau virtuel spoke via le peering de réseaux virtuels

L’architecture des réseaux virtuels hub et spoke est largement utilisée. Le hub est un réseau virtuel dans Azure qui centralise la connectivité entre vos réseaux virtuels spoke et votre réseau local. Les spokes sont des réseaux virtuels appairés avec le hub et que vous pouvez utiliser pour isoler les charges de travail. Le trafic circule entre le centre de données local et le hub via une connexion ExpressRoute ou VPN. Pour plus d’informations sur l’architecture, consultez [Implémenter une topologie réseau hub-and-spoke dans Azure][Hub-n-Spoke].

Dans le peering de réseau virtuel dans une région, les réseaux virtuels spoke peuvent utiliser des passerelles de réseau virtuel hub (les passerelles VPN et ExpressRoute) pour communiquer avec des réseaux distants.

### <a name="branch-vnet-connectivity-by-using-site-to-site-vpn"></a>Connectivité de réseau virtuel branch à l’aide d’un VPN de site à site

Vous pouvez souhaiter que les réseaux virtuels branch, qui se trouvent dans différentes régions, et les réseaux locaux communiquent entre eux via un réseau virtuel hub. La solution Azure native pour cette configuration est la connectivité VPN de site à site à l’aide d’un VPN. Une autre solution consiste à utiliser une appliance virtuelle réseau (NVA) pour le routage dans le hub.

Pour plus d’informations, consultez [Qu’est-ce qu’une passerelle VPN ?][VPN] et [Déployer une appliance virtuelle réseau hautement disponible][Deploy-NVA].


## <a name="next-steps"></a>Étapes suivantes

Consultez le [FAQ ExpressRoute][ExR-FAQ] pour :
-   Connaître le nombre de circuits ExpressRoute que vous pouvez connecter à une passerelle ExpressRoute.
-   Connaître le nombre de passerelles ExpressRoute que vous pouvez connecter à un circuit ExpressRoute.
-   Découvrir les autres limites de mise à l’échelle d’ExpressRoute.


<!--Image References-->
[1]: ./media/backend-interoperability/HubVM-SpkVM.jpg "Vue Network Watcher de la connectivité d’un réseau virtuel hub à un réseau virtuel spoke"
[2]: ./media/backend-interoperability/HubVM-BranchVM.jpg "Vue Network Watcher de la connectivité d’un réseau virtuel hub à un réseau virtuel branche"
[3]: ./media/backend-interoperability/HubVM-BranchVM-Grid.jpg "Mode Grille Network Watcher de la connectivité d’un réseau virtuel hub à un réseau virtuel branche"
[4]: ./media/backend-interoperability/Loc1-HubVM.jpg "Vue Network Performance Monitor de la connectivité de la machine virtuelle Location 1 au réseau virtuel hub via ExpressRoute 1"
[5]: ./media/backend-interoperability/Loc1-HubVM-S2S.jpg "Vue Network Performance Monitor de la connectivité de la machine virtuelle Location 1 au réseau virtuel hub via un VPN de site à site"

<!--Link References-->
[Setup]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-preface
[Configuration]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-configuration
[ExpressRoute]: https://docs.microsoft.com/azure/expressroute/expressroute-introduction
[VPN]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways
[VNet]: https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal
[Configuration]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-configuration
[Control-Analysis]:https://docs.microsoft.com/azure/networking/connectivty-interoperability-control-plane
[Data-Analysis]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-data-plane
[ExR-FAQ]: https://docs.microsoft.com/azure/expressroute/expressroute-faqs
[S2S-Over-ExR]: https://docs.microsoft.com/azure/expressroute/site-to-site-vpn-over-microsoft-peering
[ExR-S2S-CoEx]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager
[Hub-n-Spoke]: https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke
[Deploy-NVA]: https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha
[VNet-Config]: https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-peering


