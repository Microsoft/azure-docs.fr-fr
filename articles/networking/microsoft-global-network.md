---
title: Réseau mondial Microsoft - Azure
description: Découvrez comment Microsoft crée et utilise l’un des plus grands réseaux principaux dans le monde, et pourquoi il est essentiel pour offrir une expérience cloud remarquable.
services: networking
documentationcenter: ''
author: KumudD
manager: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/13/2019
ms.author: kumud
ms.reviewer: ypitsch
ms.openlocfilehash: 05e966d3f4631e3c344bd97c0faec99cc3c6dd66
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91819900"
---
# <a name="microsoft-global-network"></a>Réseau Microsoft mondial

Microsoft possède et gère l’un des plus grands réseaux principaux du monde. Cette architecture mondiale et sophistiquée, couvrant plus de 165 000 km, connecte nos centres de données et nos clients. 
 
Tous les jours, des clients du monde entier se connectent et transfèrent des billions de demandes à Microsoft Azure, Bing, Dynamics 365, Microsoft 365, XBox et bien d’autres. Quel que soit leur type, les clients attendent de nos services une fiabilité et une réactivité instantanée. 
 
Le [réseau mondial Microsoft](https://azure.microsoft.com/global-infrastructure/global-network/) (WAN) est l’une des parties essentielles d’une très bonne expérience cloud. En connectant notre [centres de données](https://azure.microsoft.com/global-infrastructure/) Microsoft à 61 régions Azure et un grand maillage de nœuds de périphérie placés de manière stratégique dans le monde entier, notre réseau mondial offre à la fois la disponibilité, la capacité et la flexibilité permettant de répondre à toute demande.

![Réseau Microsoft mondial](./media/microsoft-global-network/microsoft-global-wan.png)
 
## <a name="get-the-premium-cloud-network"></a>Obtenir le réseau cloud Premium
 
Il est facile de choisir la [meilleure expérience possible](https://www.sdxcentral.com/articles/news/azure-tops-aws-gcp-in-cloud-performance-says-thousandeyes/2018/11/) quand vous utilisez le cloud Microsoft. À partir du moment où le trafic client entre sur notre réseau mondial par le biais de nos nœuds de périphérie disposés de façon stratégique, vos données transitent via des routes optimisées presque à la vitesse de la lumière. Cela garantit une latence optimale pour de meilleures performances. Ces nœuds de périphérie, tous interconnectés à plus de 4000 partenaires Internet uniques (pairs) par le biais des milliers de connexions dans plus de 175 endroits, servent de base à notre stratégie d’interconnexion. 
 
Si vous vous connectez de Londres à Tokyo, ou de Washington DC à Los Angeles, les performances réseau sont quantifiées et affectées par des éléments tels que la latence, l’instabilité, la perte de paquets et le débit.  Chez Microsoft, nous préférons utiliser des interconnexions directes plutôt que des liaisons de transit. Cela maintient le trafic des réponses symétrique et permet de garder les tronçons, les parties d’un appairage (peering) et les chemins aussi courts et aussi simples possible. 

Par exemple, si un utilisateur à Londres tente d’accéder à un service à Tokyo, le trafic Internet entre dans l’une de nos périphéries à Londres, passe par le réseau étendu (WAN) Microsoft via la France, nos chemins de Transarabie entre l’Europe et l’Inde, puis jusqu’au Japon où le service est hébergé. Le trafic des réponses est symétrique. Cela est parfois appelé [routage cold potato](https://en.wikipedia.org/wiki/Hot-potato_and_cold-potato_routing), ce qui signifie que le trafic reste sur le réseau Microsoft aussi longtemps que possible avant que nous le transmettions.  
  
Mais est-ce que cela concerne tout le trafic généré lors de l’utilisation de services Microsoft ? Oui, tout le trafic entre les centres de données, dans Microsoft Azure ou entre des services Microsoft comme des machines virtuelles, Microsoft 365, XBox, des bases de données SQL, le stockage et des réseaux virtuels, est routé dans notre réseau mondial et jamais via l’Internet public, afin de garantir des performances et une intégrité optimales.  
 
Des investissements considérables dans la fibre et la diversité entre les chemins métropolitains, terrestres et sous-marins sont essentiels pour maintenir un niveau du service élevé et constant tout en stimulant la croissance spectaculaire de notre cloud et de nos services en ligne. Nous avons récemment ajouté à notre réseau mondial notre câble sous-marin [MAREA](https://www.submarinecablemap.com/#/submarine-cable/marea), le premier système de ligne ouverte (OLS, Open Line System) sous-marin du secteur, entre Bilbao (Espagne) et Virginia Beach (Virginie, États-Unis), ainsi que le câble sous-marin [AEC](https://www.submarinecablemap.com/#/submarine-cable/aeconnect-1) entre New York (États-Unis) et Dublin (Irlande) et le câble sous-marin [NCP (New Cross-Pacifique)](https://www.submarinecablemap.com/#/submarine-cable/new-cross-pacific-ncp-cable-system) entre Tokyo (Japon) et Portland (Oregon, États-Unis). 
 

## <a name="our-network-is-your-network"></a>Notre réseau est votre réseau

Nous avons consacré vingt ans d’expérience ainsi que des investissements considérables dans le réseau afin de garantir en permanence des performances optimales. Les entreprises peuvent tirer pleinement parti de nos ressources réseau et générer des architectures de superposition avancées au-dessus. 
 
Microsoft Azure offre la gamme de services et de fonctionnalités la plus complète, permettant aux utilisateurs de rapidement et facilement créer, développer et répondre aux besoins en réseau où qu’ils soient. Notre famille de services de connectivité couvre le peering de réseaux virtuels entre des régions, des architectures de point à site et de site à site hybrides et dans le cloud, ainsi que des scénarios de transit IP à l’échelle mondiale.  Pour les entreprises cherchant à connecter leur propre centre de données ou réseau à Azure, ou les clients avec une ingestion massive de données ou des besoins de transit important, [ExpressRoute](../expressroute/expressroute-introduction.md) et [ExpressRoute Direct](../expressroute/expressroute-erdirect-about.md) proposent des options offrant jusqu’à 100 Gbits/s de bande passante, directement dans le réseau mondial de Microsoft à des emplacements de peering dans le monde entier.  
 
[ExpressRoute Global Reach](../expressroute/expressroute-global-reach.md) est conçu pour compléter l’implémentation du réseau étendu (WAN) de votre fournisseur de services et connecter vos sites locaux à travers le monde. Si vous exécutez une opération à l’échelle mondiale, vous pouvez utiliser ExpressRoute Global Reach conjointement avec vos fournisseurs de services locaux préférés pour connecter tous vos sites mondiaux à l’aide du réseau mondial Microsoft. Le développement de votre nouveau réseau dans le cloud (WAN) pour englober un grand nombre de sites de succursale peut être accompli à l’aide d’Azure Virtual WAN, qui offre la possibilité de connecter sans interruption vos succursales au réseau mondial de Microsoft avec des appareils SDWAN et VPN (autrement dit, Customer Premises Equipment ou CPE) avec une facilité d’utilisation intégrée, une connectivité automatisée et une gestion de la configuration. 
 
[Global VNet Peering](../virtual-network/virtual-network-peering-overview.md) permet aux clients de connecter sans interruption deux réseaux virtuels Azure ou plus de différentes régions. Une fois appairés, les réseaux virtuels apparaissent comme un seul réseau. Le trafic entre les machines virtuelles des réseaux virtuels appairés est routé par le biais de l’infrastructure principale de Microsoft, de façon assez semblable au trafic entre des machines virtuelles d’un même réseau virtuel (via des adresses IP privées uniquement). 
 

## <a name="well-managed-using-software-defined-innovation"></a>Gestion efficace à l’aide d’une innovation à définition logicielle

En exécutant l’un des clouds leaders sur le marché mondial, Microsoft a tiré beaucoup d’insights et d’expérience de la création et de la gestion d’une infrastructure mondiale hautes performances.  
 
Nous respectons un ensemble performant de principes de fonctionnement : 
 
- Utilisez le meilleur matériel de commutation sur les différents niveaux du réseau.  
- Déployez de nouvelles fonctionnalités sans aucun impact pour les utilisateurs finaux.  
- Déployez aussi rapidement que possible des mises à jour de manière sécurisée et fiable dans tout le parc. En heures plutôt qu’en semaines.  
- Utilisez une télémétrie approfondie à l’échelle du cloud et une atténuation des risques entièrement automatisée.  
- Utilisez une technologie réseau unifiée et à définition logicielle pour contrôler tous les éléments matériels présents dans le réseau.  Élimination de la duplication et réduction des échecs. 
 
Ces principes s’appliquent à toutes les couches du réseau, depuis l’interface réseau, la plateforme de commutation et les fonctions réseau du centre de données comme les équilibreurs de charge jusqu’au réseau étendu (WAN) avec notre plateforme d’ingénierie de trafic et nos réseaux optiques.  
 
La croissance exponentielle d’Azure et de son réseau a atteint un point où nous avons finalement compris qu’il n’était plus possible de s’appuyer sur l’intuition humaine pour gérer les opérations du réseau mondial. Pour répondre à la nécessité de valider les changements à long terme, moyen terme et court terme apportés au réseau, nous avons développé une plateforme permettant de mettre en miroir et d’émuler notre réseau de production de façon synthétique. La possibilité de créer des environnements en miroir et d’exécuter des millions de simulations nous permet de tester des changements logiciels et matériels avant de les valider sur notre plateforme de production et notre réseau. 

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus sur les services réseau fournis dans Azure](https://azure.microsoft.com/product-categories/networking/)
