---
title: Vue d’ensemble d’Azure Peering Service
description: Découvrir une vue d’ensemble d’Azure Peering Service
services: peering-service
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: 198ba23920179e71e095e498ee2173d7f0111d42
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026727"
---
# <a name="azure-peering-service-overview"></a>Vue d’ensemble d’Azure Peering Service

Azure Peering Service est un service réseau qui améliore la connectivité entre le client et les services cloud Microsoft, comme Microsoft 365, Dynamics 365, les services SaaS, Azure ou les services Microsoft accessibles via l’Internet public. Microsoft a opéré en partenariat avec des fournisseurs de services Internet (ISP), des partenaires d’échange Internet (IXP) et des fournisseurs SDCI (Software-Defined Cloud Interconnexion) dans le monde entier pour fournir une connectivité publique fiable et hautement performante, avec un routage optimal entre le client et le réseau Microsoft.

Peering Service permet aux clients de sélectionner un fournisseur de services partenaire correctement connecté dans une région donnée. La connectivité publique est optimisée pour offrir une haute fiabilité et une latence minimale des services cloud à l’emplacement de l’utilisateur final.

![Connectivité distribuée au cloud Microsoft](./media/peering-service-about/peering-service-what.png)

Les clients peuvent également choisir une télémétrie du Peering Service, comme des mesures de latence de l’utilisateur vers le réseau Microsoft, une surveillance d’itinéraire BGP et des alertes de fuites et de détournements en inscrivant la connexion Peering Service dans le portail Azure. 

Pour utiliser Peering Service, les clients ne doivent pas s’inscrire auprès de Microsoft. La seule exigence consiste à contacter un [partenaire Peering Service](location-partners.md) pour obtenir le service. Pour choisir la télémétrie Peering Service, les clients doivent s’y inscrire via le portail Azure.

Pour obtenir des instructions sur l’inscription du Peering Service, consultez [inscrire Peering Service à l’aide du portail Azure](azure-portal.md). 

> [!NOTE]
> Cet article s’adresse aux architectes réseau chargés de la connectivité d’entreprise au cloud et à Internet.


## <a name="what-is-peering-service"></a>Qu’est-ce que Peering Service ?

Peering Service est :

- Un service IP qui utilise l’Internet public. 
- Une plateforme de collaboration avec des fournisseurs de services et un service à valeur ajoutée conçu pour offrir un routage optimal et fiable au client via des partenaires fournisseurs de services vers le cloud Microsoft sur le réseau public.

Peering Service n’est pas un produit de connectivité privé comme Azure ExpressRoute ou un produit VPN.

> [!NOTE]
> Pour plus d’informations sur ExpressRoute, consultez la [documentation sur ExpressRoute](../expressroute/index.yml).
>

## <a name="background"></a>Arrière-plan

Microsoft 365, Dynamics 365 et tous les autres services SaaS Microsoft sont hébergés dans plusieurs centres de donnés Microsoft, et accessibles à partir de n'importe quel emplacement géographique. Le réseau mondial de Microsoft dispose d’emplacements de point de présence (PoP, point-of-presence) Microsoft Edge dans le monde entier, où il peut se connecter à un utilisateur final via ses fournisseurs de services. 

Microsoft et ses fournisseurs de services partenaires garantissent que le trafic pour les préfixes inscrits auprès d’une connexion Peering Service transite par les emplacements PoP Microsoft Edge les plus proches sur le réseau Microsoft mondial. Microsoft s’assure que le trafic réseau sortant à partir des préfixes inscrits auprès de connexions Peering Service emprunte les emplacements de PoP Microsoft Edge les plus proches sur le réseau Microsoft mondial.

![Réseau Microsoft et connectivité publique](./media/peering-service-about/peering-service-background-final.png)

> [!NOTE]
> Pour plus d’informations sur le réseau Microsoft mondial, consultez [Réseau Microsoft mondial](../networking/microsoft-global-network.md).
>

## <a name="why-use-peering-service"></a>Pourquoi utiliser Peering Service ?

Les entreprises qui recherchent un accès Internet au cloud, envisagent une architecture SD-WAN ou une utilisation intensive des services SaaS Microsoft nécessitent une connectivité Internet robuste et hautement performante. Les clients peuvent opérer cette transition à l’aide de Peering Service. Microsoft et ses fournisseurs de services ont conclu un partenariat pour fournir une connectivité publique fiable et centrée sur les performances dans le cloud Microsoft. Voici quelques-unes des principales caractéristiques :

- meilleur routage public sur Internet vers Microsoft Azure Cloud Services pour des performances et une fiabilité optimales ;
- possibilité de sélectionner le fournisseur de services par défaut pour se connecter au cloud Microsoft ;
- informations sur le trafic, telles que le signalement de latence et la surveillance des préfixes ;
- tronçons réseau optimaux (tronçons AS) à partir du cloud Microsoft ;
- analyse et statistiques de l’itinéraire – événements pour les anomalies d’itinéraire ([BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol)) (détection des fuites ou détournements) et routage non optimal.

### <a name="robust-reliable-peering"></a>Appairage robuste et fiable

Peering Service utilise deux types de redondances :

- **Redondance locale**

   Microsoft et les fournisseurs de services s’interconnectent en plusieurs emplacements PoP Microsoft Edge pour assurer le Peering Service. Dans chaque emplacement, l’interconnexion doit prendre en charge le basculement entre deux routeurs.

   Chaque emplacement d’appairage est approvisionné en liens d’appairage redondants et diversifiés.

- **Géo-redondance**

   Microsoft s’est interconnecté avec des fournisseurs de services dans plusieurs emplacements métropolitains de sorte que, si l’un des nœuds Edge présente des performances dégradées, le trafic échangé avec Microsoft est acheminé via d’autres sites. Microsoft route le trafic dans son réseau global en utilisant des stratégies de routage basées sur SDN afin d’offrir des performances optimales.

    Ce type de redondance utilise le chemin de routage le plus court en choisissant toujours le PoP Microsoft Edge le plus proche de l’utilisateur final, et veille à que le client reste toujours à un tronçon (tronçon AS) de réseau de Microsoft.

   ![Géo-redondance](./media/peering-service-about/peering-service-geo-shortest.png)

### <a name="optimal-routing"></a>Routage optimal

La technique de routage suivante est privilégiée :

-  **Routage « cold potato »**

   La technique de routage « cold potato » à définition logicielle permet de contrôler le trafic réseau provenant du cloud Microsoft. Elle veille à ce que le trafic reste sur le réseau Microsoft mondial à haute capacité, à faible latence et extrêmement fiable, jusqu’à ce qu’il soit aussi proche que possible de la destination.
   
   Le routage qui n’utilise pas la technique de routage « cold potato » est appelé routage « hot potato ». Avec le routage « hot potato », le trafic provenant du cloud Microsoft transite sur Internet.

   ![Routage « hot potato »](./media/peering-service-about/peering-service-cold-potato.png)

### <a name="monitoring-platform"></a>Plateforme de supervision

   La supervision du service vise à analyser le trafic et le routage des clients, et présente les caractéristiques suivantes : 

-  **Détection des anomalies d’itinéraire BGP Internet**
          
   Ce service est utilisé pour détecter et signaler des événements d’anomalie d’itinéraire tels que des détournements vers des préfixes de client.

-  **Latence du client**

   Ce service surveille les performances de routage entre l’emplacement du client et Microsoft. 
   
   Les performances de routage sont mesurées en validant le temps d’aller-retour pris par le client pour atteindre le PoP Microsoft Edge. Les clients peuvent consulter les rapports de latence pour différents emplacements géographiques.

   La surveillance capture les événements en cas de dégradation du service.

   ![Plateforme de supervision pour Peering Service](media/peering-service-about/peering-service-latency-report.png)

### <a name="traffic-protection"></a>Protection du trafic

Le routage se produit uniquement via un chemin préféré défini lorsque le client s’inscrit auprès du Peering Service.

Microsoft garantit le routage du trafic via des chemins préférés, même en cas de détection d’une activité malveillante.

Le cas échéant, les anomalies d’itinéraires BGP sont signalées sur le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur les connexions Peering Service, consultez [Connexions Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).
- Pour rechercher un fournisseur de services partenaire, consultez [Partenaires et emplacements Peering Service](location-partners.md).
- Pour intégrer une connexion Peering Service, consultez [Modèle d’intégration de Peering Service](onboarding-model.md).
- Pour inscrire une connexion via le portail Azure, consultez [Inscrire une connexion Peering Service par le biais du portail Azure](azure-portal.md).
- Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).