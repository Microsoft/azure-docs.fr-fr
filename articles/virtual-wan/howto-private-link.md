---
title: Partager un service de liaison privée entre des réseaux étendus virtuels
titleSuffix: Azure Virtual WAN
description: Étapes de configuration d’Azure Private Link dans Virtual WAN
services: virtual-wan
author: erjosito
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: jomore
ms.custom: fasttrack-new
ms.openlocfilehash: cc8e7314c941035207ecf809a9d85ef46bd58379
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913753"
---
# <a name="use-private-link-in-virtual-wan"></a>Utiliser Azure Private Link dans Azure Virtual WAN

[Azure Private Link](../private-link/private-link-overview.md) est une technologie qui vous permet de connecter des offres PaaS Azure à l’aide d’une connectivité d’adresse IP privée en exposant des [points de terminaison privés](../private-link/private-endpoint-overview.md). Grâce à Azure Virtual WAN, vous pouvez déployer un point de terminaison privé dans l’un des réseaux virtuels connectés à un hub virtuel. Cela permet de se connecter à un autre réseau virtuel ou à une autre branche connectée au même réseau étendu virtuel.

## <a name="before-you-begin"></a>Avant de commencer

Les étapes décrites dans cet article supposent que vous avez déjà déployé un réseau étendu virtuel avec un ou plusieurs hubs, ainsi qu’au moins deux réseaux virtuels connectés à Azure Virtual WAN.

Pour créer un nouveau réseau étendu virtuel et un nouveau hub, suivez les étapes décrites dans les articles suivants :

* [Créer un réseau étendu virtuel](virtual-wan-site-to-site-portal.md#openvwan)
* [Créer un hub](virtual-wan-site-to-site-portal.md#hub)
* [Connecter un réseau virtuel à un hub](virtual-wan-site-to-site-portal.md#hub)

## <a name="create-a-private-link-endpoint"></a><a name="endpoint"></a>Créer un point de terminaison de liaison privée

Vous pouvez créer un point de terminaison de liaison privée pour de nombreux services différents. Dans cet exemple, nous utiliserons Azure SQL Database. Vous trouverez plus d’informations sur la création d’un point de terminaison privé pour une base de données Azure SQL dans [Démarrage rapide : Créer un point de terminaison privé en utilisant le portail Azure](../private-link/create-private-endpoint-portal.md). L’illustration suivante montre la configuration réseau de la base de données Azure SQL :

:::image type="content" source="./media/howto-private-link/create-private-link.png" alt-text="créer une liaison privée" lightbox="./media/howto-private-link/create-private-link.png":::

Après avoir créé la base de données Azure SQL, vous pouvez vérifier l’adresse IP du point de terminaison privé en parcourant vos points de terminaison privés :

:::image type="content" source="./media/howto-private-link/endpoints.png" alt-text="points de terminaison privés" lightbox="./media/howto-private-link/endpoints.png":::

En cliquant sur le point de terminaison privé que nous avons créé, vous devez voir son adresse IP privée, ainsi que son nom de domaine complet (FQDN). Notez que le point de terminaison privé a une adresse IP comprise dans la plage du réseau virtuel où il a été déployé (10.1.3.0/24) :

:::image type="content" source="./media/howto-private-link/sql-endpoint.png" alt-text="point de terminaison SQL" lightbox="./media/howto-private-link/sql-endpoint.png":::

## <a name="verify-connectivity-from-the-same-vnet"></a><a name="connectivity"></a>Vérifier la connectivité provenant du même réseau virtuel

Dans cet exemple, nous allons vérifier la connectivité à la base de données Azure SQL à partir d’une machine virtuelle Ubuntu sur laquelle les outils MS SQL sont installés. La première étape consiste à vérifier que la résolution DNS fonctionne et que le nom de domaine complet de la base de données Azure SQL est résolu en une adresse IP privée, dans le même réseau virtuel où le point de terminaison privé a été déployé (10.1.3.0/24) :

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Comme vous pouvez le voir dans la sortie précédente, le nom de domaine complet `wantest.database.windows.net` est mappé à `wantest.privatelink.database.windows.net`, que la zone DNS privée créée avec le point de terminaison privé résoudra en l’adresse IP privée `10.1.3.228`. L’examen de la zone DNS privée confirmera qu’il existe un enregistrement A pour le point de terminaison privé mappé à l’adresse IP privée :

:::image type="content" source="./media/howto-private-link/dns-zone.png" alt-text="Zone DNS" lightbox="./media/howto-private-link/dns-zone.png":::

Après avoir vérifié la résolution DNS correcte, nous pouvons tenter de nous connecter à la base de données :

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.3.75
```

Comme vous pouvez le voir, nous utilisons une requête SQL spéciale qui nous donne l’adresse IP source que le serveur SQL voit à partir du client. Dans ce cas, le serveur voit le client avec son adresse IP privée (`10.1.3.75`), ce qui signifie que le trafic transite directement du réseau virtuel au point de terminaison privé.

Notez que vous devez définir les variables `username` et `password` pour qu’elles correspondent aux informations d’identification définies dans la base de données Azure SQL et que les exemples de ce guide fonctionnent.

## <a name="connect-from-a-different-vnet"></a><a name="vnet"></a>Se connecter à partir d’un autre réseau virtuel

Maintenant qu’un réseau virtuel dans Azure Virtual WAN dispose d’une connectivité au point de terminaison privé, tous les autres réseaux virtuels et toutes les branches connectées à Virtual WAN peuvent également y accéder. Vous devez fournir une connectivité à l’aide de l’un des modèles pris en charge par Azure Virtual WAN, comme le [scénario Any-To-Any](scenario-any-to-any.md) ou le [scénario Réseaux virtuels des services partagés](scenario-shared-services-vnet.md), pour ne citer que deux exemples.

Une fois que vous disposez d’une connectivité entre le réseau virtuel ou la branche et le réseau virtuel dans lequel le point de terminaison privé a été déployé, vous devez configurer la résolution DNS :

* Si vous vous connectez au point de terminaison privé à partir d’un réseau virtuel, vous pouvez utiliser la même zone privée que celle qui a été créée avec la base de données Azure SQL.
* Si vous vous connectez au point de terminaison privé à partir d’une branche (VPN site à site, VPN point à site ou ExpressRoute), vous devez utiliser la résolution DNS locale.

Dans cet exemple, nous allons nous connecter à partir d’un autre réseau virtuel. Nous allons tout d’abord attacher la zone DNS privée au nouveau réseau virtuel afin que ses charges de travail puissent résoudre le nom de domaine complet de la base de données Azure SQL en adresse IP privée. Pour ce faire, vous pouvez lier la zone DNS privée au nouveau réseau virtuel :

:::image type="content" source="./media/howto-private-link/dns-link.png" alt-text="lien DNS" lightbox="./media/howto-private-link/dns-link.png":::

Maintenant, toute machine virtuelle du réseau virtuel attaché doit résoudre correctement le FQDN de la base de données Azure SQL en adresse IP privée de la liaison privée :

```bash
$ nslookup wantest.database.windows.net
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
wantest.database.windows.net    canonical name = wantest.privatelink.database.windows.net.
Name:   wantest.privatelink.database.windows.net
Address: 10.1.3.228
```

Afin de vérifier que ce réseau virtuel (10.1.1.0/24) dispose d’une connectivité au réseau virtuel d’origine dans lequel le point de terminaison privé a été configuré (10.1.3.0/24), vous pouvez vérifier la table de routage en vigueur dans toute machine virtuelle du réseau virtuel :

:::image type="content" source="./media/howto-private-link/effective-routes.png" alt-text="itinéraires effectifs" lightbox="./media/howto-private-link/effective-routes.png":::

Comme vous pouvez le voir, il existe un itinéraire pointant vers le réseau virtuel 10.1.3.0/24 injecté par les passerelles du réseau virtuel dans Azure Virtual WAN. Nous pouvons à présent tester la connectivité à la base de données :

```bash
$ query="SELECT CONVERT(char(15), CONNECTIONPROPERTY('client_net_address'));"
$ sqlcmd -S wantest.database.windows.net -U $username -P $password -Q "$query"

10.1.1.75
```

Dans cet exemple, nous avons vu comment la création d’un point de terminaison privé dans l’un des réseaux virtuels attachés à un réseau étendu virtuel permet de se connecter au reste des réseaux virtuels et des branches du réseau étendu virtuel.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
