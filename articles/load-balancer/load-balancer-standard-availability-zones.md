---
title: Standard Load Balancer et zones de disponibilité Azure
titleSuffix: Azure Load Balancer
description: Dans ce parcours d’apprentissage, familiarisez-vous avec Azure Standard Load Balancer et les Zones de disponibilité.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2020
ms.author: allensu
ms.openlocfilehash: fca6b40f4bd871041d14d119eb44e8366d8562ee
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94700458"
---
# <a name="standard-load-balancer-and-availability-zones"></a>Référence Standard de Load Balancer et zones de disponibilité

Azure Standard Load Balancer prend en charge les scénarios des zones de disponibilité. Vous pouvez utiliser l’équilibreur de charge standard pour améliorer la disponibilité dans votre scénario en alignant les ressources et en les distribuant sur les zones. Les zones de disponibilité Azure associées à l’équilibreur de charge standard constituent un ensemble de fonctionnalités vaste et flexible capable de créer de nombreux scénarios différents.  Consultez ce document pour comprendre ces [concepts](#concepts) et obtenir des [conseils de conception](#design) de scénarios.

## <a name="availability-zones-concepts-applied-to-load-balancer"></a><a name="concepts"></a> Concepts des zones de disponibilité appliqués à Load Balancer

L’équilibreur de charge hérite la configuration de zone de ses composants : 

* Serveur frontal
* Règles
* Définition du pool principal

Dans le contexte des zones de disponibilité, le comportement et les propriétés d’une règle d’équilibreur de charge sont décrits comme redondants interzone ou zonaux.  Dans le contexte de l’équilibreur de charge, la redondance interzone implique toujours **plusieurs zones** tandis que l’adjectif zonal implique une isolation du service dans une **zone unique**. L’équilibreur Azure Load Balancer a deux types, public et interne. Les deux types d’équilibreurs de charge prennent en charge la redondance de zone et le déploiement zonal.  Les deux types d’équilibreurs de charge peuvent diriger le trafic entre les zones en fonction des besoins.

## <a name="frontend"></a>Serveur frontal

Le frontend d’équilibreur de charge est une configuration IP frontend qui fait référence soit à une adresse IP publique, soit à une adresse IP privée au sein du sous-réseau d’un réseau virtuel. Il forme le point de terminaison à charge équilibrée où votre service est exposé.

Une ressource d’équilibreur de charge peut contenir des règles avec des frontends zonaux et redondants interzone en même temps. Quand une adresse IP publique ou privée est garantie pour une zone, la zonalité (ou l’absence de celle-ci) n’est pas modifiable. Pour modifier ou omettre la zonalité d’un frontend d’adresse IP publique ou privée, recréez l’adresse IP dans la zone appropriée. 

Les zones de disponibilité ne modifient pas les contraintes pour plusieurs serveurs frontend. Pour plus d’informations sur cette fonctionnalité, consultez [Frontends multiples pour Azure Load Balancer](load-balancer-multivip-overview.md).

### <a name="zone-redundant"></a>Redondant interzone 

Dans une région avec des zones de disponibilité, un front-end d’équilibreur de charge standard peut être redondant interzone. Plusieurs zones peuvent traiter les transactions entrantes ou sortantes dans une région. Ce trafic est pris en charge par une adresse IP unique. Les schémas de redondance DNS ne sont pas nécessaires. 

Une adresse IP de serveur frontend unique va survivre à une défaillance de zone. L’adresse IP frontend peut être utilisée pour atteindre tous les membres du pool principal (non affectés), quelle que soit la zone. Une ou plusieurs zones de disponibilité peuvent échouer sans empêcher le chemin de données de survivre dans la mesure où une seule zone de la région reste saine. 

L’adresse IP frontend est servie simultanément par plusieurs déploiements d’infrastructure indépendants dans plusieurs zones de disponibilité. Toute nouvelle tentative ou tout rétablissement réussissent dans les autres zones non affectée par les défaillances de zone. 

<p align="center">
  <img src="./media/az-zonal/zone-redundant-lb-1.svg" alt="Figure depicts a zone-redundant standard load balancer directing traffic in three different zones to three different subnets in a zone redundant configuration." width="512" title="Service NAT de Réseau virtuel">
</p>

*Figure : Équilibreur de charge redondant dans une zone*

### <a name="zonal"></a>Zonal

Vous pouvez choisir de garantir un frontend dans une seule zone, ce qui donne un *frontend zonal*.  Dans ce scénario, tout flux entrant ou sortant est servi par une seule zone dans une région.  Votre frontend connaît le même sort que l’intégrité de la zone.  Le chemin de données n’est pas affecté par les défaillances des zones autres que celles dans lesquelles il a été garanti. Vous pouvez utiliser des frontends zonaux pour exposer une adresse IP par zone de disponibilité.  

En outre, vous pouvez utiliser des frontends zonaux directement pour les points de terminaison à charge équilibrée dans chaque zone. Vous pouvez également utiliser cette configuration pour exposer des points de terminaison à charge équilibrée par zone afin de surveiller individuellement chaque zone. Pour les points de terminaison publics, vous pouvez les intégrer à un produit d’équilibrage de charge DNS comme [Traffic Manager](../traffic-manager/traffic-manager-overview.md) et utiliser un nom DNS unique.


<p align="center">
  <img src="./media/az-zonal/zonal-lb-1.svg" alt="Figure depicts three zonal standard load balancers each directing traffic in a zone to three different subnets in a zonal configuration." width="512" title="Service NAT de Réseau virtuel">
</p>

*Figure : Équilibreur de charge zonal*

Pour fusionner ces concepts (le même backend à la fois redondant interzone et zonal), passez en revue [Frontends multiples pour Azure Load Balancer](load-balancer-multivip-overview.md).

Pour un frontend d’équilibreur de charge public, vous ajoutez un paramètre **zones** à l’adresse IP publique. Cette adresse IP publique est référencée par la configuration d’adresse IP frontend utilisée par la règle correspondante.

Pour un équilibreur de charge frontend interne, ajoutez un paramètre **zones** à la configuration IP de l’équilibreur de charge frontend interne. Un frontend zonal garantit une adresse IP dans un sous-réseau d’une zone spécifique.

## <a name="backend"></a>Backend

Azure Load Balancer fonctionne avec des instances de machines virtuelles. Ces instances peuvent être des instances autonomes, des groupes à haute disponibilité ou des groupes de machines virtuelles identiques. Toute machine virtuelle d’un réseau virtuel peut faire partie du pool principal, quelle que soit la zone pour laquelle la machine virtuelle est garantie.

Pour aligner et garantir votre frontend et votre back-end avec une zone unique, placez uniquement les machines virtuelles au sein de la même zone dans le pool principal correspondant.

Pour les machines virtuelles réparties dans plusieurs zones, placez des machines virtuelles de plusieurs zones dans le même pool principal. 

Quand vous utilisez des groupes de machines virtuelles identiques, placez-en un ou plusieurs dans le même pool backend. Les groupes identiques peuvent être dans une ou plusieurs zones.

## <a name="outbound-connections"></a>Connexions sortantes

La redondance interzone et zonale s’appliquent aux [connexions sortantes](load-balancer-outbound-connections.md). Une adresse IP publique redondante interzone utilisée pour les connexions sortantes est servie par toutes les zones. Une adresse IP publique zonale est servie uniquement par la zone dans laquelle elle est garantie. 

Les allocations de port SNAT des connexions sortantes survivent aux défaillances de zone et votre scénario continuera à fournir une connectivité SNAT sortante si elle n’est pas affectée par les défaillances de zone. La défaillance d’une zone peut nécessiter le rétablissement des connexions pour les scénarios redondants interzone si le trafic était assuré par une zone affectée. Les flux dans les zones autres que les zones concernées ne sont pas affectés.

L’algorithme de pré-allocation de port SNAT est le même avec ou sans zones de disponibilité.

## <a name="health-probes"></a>Sondes d’intégrité

Vos définitions de sonde d’intégrité existantes restent telles qu’elles sont sans zones de disponibilité. Toutefois, nous avons développé le modèle d’intégrité au niveau de l’infrastructure. 

Lorsque vous utilisez des serveurs frontends redondants interzone, l’équilibreur de charge étend son contrôle d’intégrité interne. L’équilibreur de charge sonde de façon indépendante la disponibilité de la machine virtuelle de chaque zone et coupe les chemins d’accès entre les zones qui ont échoué sans intervention.  

Les autres zones qui peuvent accéder à cette machine virtuelle peuvent continuer de servir la machine virtuelle à partir de leurs frontends respectifs. Pendant des événements de défaillance, chaque zone peut présenter des distributions différentes des nouveaux flux pendant qu’elle protège l’intégrité globale de votre service.

## <a name="design-considerations"></a><a name="design"></a> Remarques relatives à la conception

L’équilibreur de charge est flexible dans le contexte des zones de disponibilité. Vous pouvez choisir de vous aligner sur des zones ou d’appliquer la redondance interzone pour chaque règle. Plus de disponibilité peut aussi engendrer plus de complexité. Envisagez la conception de façon à obtenir des performances optimales.

### <a name="zone-redundancy"></a>Redondance interzone

Avec Load Balancer, il est simple d’avoir une adresse IP unique comme frontend redondant interzone. Une adresse IP redondante interzone peut servir une ressource zonale dans n’importe quelle zone.  L’adresse IP peut survivre à un ou plusieurs défaillances de zone tant qu’une zone reste fonctionnelle dans la région.  Inversement, un frontend zonal est une réduction du service à une seule zone et partage son sort avec la zone concernée.

La redondance dans une zone n’implique pas de chemin de données ni de plan de contrôle sans réponse ; il s’agit d’un plan de données. Les flux redondants interzone peuvent utiliser toutes les zones et les flux d’un client utilise toutes les zones saines dans une région. Dans le cas d’une défaillance de zone, les flux de trafic utilisant les zones saines ne sont pas affectés. 

Les flux de trafic qui font appel à une zone au moment de la défaillance de cette dernière peuvent être concernés, mais les applications peuvent récupérer. Le trafic se poursuit dans les zones saines de la région après retransmission, une fois qu’Azure a convergé autour de la défaillance de zone.

### <a name="cross-zone-boundaries"></a><a name="xzonedesign"></a> Limites entre les zones

Il est important de comprendre que dès lors qu’un service traverse des zones, vous partagez le sort non pas d’une seule zone mais éventuellement de plusieurs. Par conséquent, votre service n’a peut-être pas obtenu de disponibilité sur des déploiements non zonaux.

Évitez d’introduire involontairement des dépendances entre les zones qui invalident les gains de disponibilité quand vous utilisez des zones de disponibilité. Si votre application contient plusieurs composants critiques, assurez-vous de leur survie en cas de défaillance d’une zone.

Un seul composant critique de votre application peut affecter toute votre application s’il se trouve dans une seule zone, autre que les zones survivantes. Prévoyez la restauration de la zone et la façon dont votre application converge. Vous devez comprendre comment votre application réagit aux défaillances de certaines parties de l’application. Voyons quelques points importants et utilisons-les pour poser les questions qui surviennent quand vous réfléchissez à votre scénario.

- Si votre application comporte deux composants :

    * Adresse IP
    * Machine virtuelle avec disque managé

Elles sont configurées dans la zone 1 et dans la zone 2. Si la zone 1 connaît une défaillance, votre service ne survivra pas. Ne franchissez pas de zones avec des scénarios zonaux, sauf si vous comprenez pleinement que vous créez un mode de défaillance potentiellement dangereux. Ce scénario est autorisé pour assurer la flexibilité.

- Si votre application comporte deux composants :

    * Adresse IP
    * Machine virtuelle avec disque managé

Elles sont configurées pour être redondantes dans une zone et dans la zone 1. Votre service survivra aux défaillances de zone 2, de zone 3 ou les deux, sauf si la zone 1 a connu une défaillance. Toutefois, vous perdez une certaine capacité à raisonner sur l’intégrité de votre service si tout ce que vous observez, c’est l’accessibilité du frontend.  Envisagez de développer un modèle d’intégrité et de capacité plus étendu.  Vous pouvez utiliser les concepts de redondance interzone et de caractéristique zonale ensemble pour développer un insight et une facilité de gestion.

- Si votre application comporte deux composants :

    * Équilibreur de charge redondant interzone frontend
    * Groupe de machines virtuelles identiques entre zones dans trois zones

Vos ressources dans les zones non affectées par la défaillance seront disponibles. La capacité de votre service peut être détériorée au cours de la défaillance. Du point de vue de l’infrastructure, votre déploiement peut survivre à une ou plusieurs défaillances de zone. Ce scénario pose les questions suivantes :

  - Est-ce que vous comprenez comment votre application réagit face à ces défaillances et face à la détérioration de la capacité ?
  - Avez-vous besoin de mesures de protection pour votre service afin de forcer un basculement vers une paire de régions si nécessaire ?
  - Comment allez-vous surveiller, détecter et atténuer un tel scénario ? Vous pouvez utiliser le diagnostic de l’équilibreur de charge standard pour renforcer la surveillance des performances de votre service. Examinez ce qui est disponible et ce qui peut nécessiter une augmentation.

- Les zones permettent de comprendre et contenir les échecs plus facilement. Toutefois, une défaillance de zone n’est pas différente des autres défaillances en termes de délais d’attente, de nouvelles tentatives et d’algorithmes d’interruption. Azure Load Balancer fournit des chemins d’accès redondants interzones. L’équilibreur de charge essaie de récupérer rapidement, au niveau du paquet en temps réel. Des retransmissions ou des rétablissements peuvent se produire lors de l’apparition d’une défaillance. Il est important de comprendre comment votre application réagit face aux défaillances. Votre schéma d’équilibrage de charge survit, mais vous devez prévoir ce qui suit :

  - Lorsqu’une zone connaît une défaillance, votre service le comprend-il et en cas de perte de l’état, comment allez-vous récupérer ?
  - Quand une zone est rétablie, votre application comprend-elle comment converger en toute sécurité ?

Passez en revue les [modèles de conception cloud Azure](/azure/architecture/patterns/) pour améliorer la résilience de votre application aux scénarios de défaillance.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [zones de disponibilité](../availability-zones/az-overview.md)
- En savoir plus sur la [référence Standard de Load Balancer](./load-balancer-overview.md)
- Découvrir comment [équilibrer la charge des machines virtuelles dans une zone à l’aide de Load Balancer standard avec un serveur frontal zonal](./quickstart-load-balancer-standard-public-cli.md)
- Découvrir comment [équilibrer la charge des machines virtuelles dans des zones à l’aide de Load Balancer standard avec un serveur frontal redondant interzone](./quickstart-load-balancer-standard-public-cli.md)
- Découvrez plus d'informations sur les [modèles de conception cloud Azure](/azure/architecture/patterns/) pour améliorer la résilience de votre application aux scénarios de défaillance.