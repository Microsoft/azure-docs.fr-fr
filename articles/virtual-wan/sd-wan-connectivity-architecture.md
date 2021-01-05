---
title: Architecture de connectivité Virtual WAN et SD-WAN
titleSuffix: Azure Virtual WAN
description: En savoir plus sur l’interconnexion d’un SD-WAN privé avec Azure Virtual WAN
services: virtual-wan
author: skishen525
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: sukishen
ms.openlocfilehash: ea9ddd05fe6fc745a3eefc29ab4f1d6aababc936
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94564699"
---
# <a name="sd-wan-connectivity-architecture-with-azure-virtual-wan"></a>Architecture de connectivité SD-WAN avec Azure Virtual WAN

Azure Virtual WAN est un service de mise en réseau qui regroupe de nombreux services de sécurité et de connectivité cloud avec une seule interface opérationnelle. Ces services incluent des connexions entre branches (VPN site à site), entre utilisateurs distants (VPN point à site) et privées (ExpressRoute), une connexion transitive intra-cloud pour réseaux virtuels, une interconnexion entre VPN et ExpressRoute, le routage, le Pare-feu Azure et le chiffrement des connexions privées.

Si Azure Virtual WAN est un réseau étendu, ou WAN, à définition logicielles (SD-WAN), il est également conçu pour permettre une interconnexion transparente avec des technologies et services SD-WAN locaux. De nombreux services de ce type sont proposés par notre écosystème de [Virtual WAN](virtual-wan-locations-partners.md) et des partenaires de services gérés [(MSP, Managed Services partners)](../networking/networking-partners-msp.md) d’Azure Networking. Les entreprises qui transforment leur WAN privé en SD-WAN disposent d’options lors de l’interconnexion de leur SD-WAN privé avec Azure Virtual WAN. Les entreprises peuvent choisir parmi les options suivantes :

* Modèle d’interconnexion directe
* Modèle d’interconnexion directe avec NVA-in-VWAN-hub
* Modèle d’interconnexion indirecte
* Modèle WAN hybride géré utilisant leur fournisseur de services gérés [MSP](../networking/networking-partners-msp.md) préféré

Dans tous ces cas, l’interconnexion de Virtual WAN avec SD-WAN est similaire sur le plan de la connectivité, mais peut varier sur les plans de l’orchestration et de l’exploitation.

## <a name="direct-interconnect-model"></a><a name="direct"></a>Modèle d’interconnexion directe

:::image type="content" source="./media/sd-wan-connectivity-architecture/direct.png" alt-text="Modèle d’interconnexion directe":::

Dans ce modèle d’architecture, l’équipement local du client côté branche SD-WAN est directement connecté à des concentrateurs Virtual WAN via des connexions IPsec. L’équipement local du client côté branche peut également être connecté à d’autres branches via le SD-WAN privé, ou tirer parti du Virtual WAN pour la connexion de branche à branche. Les branches qui ont besoin d’accéder à leurs charges de travail dans Azure pourront accéder directement et en toute sécurité à Azure via le ou les tunnels IPsec qui se terminent dans le ou les concentrateurs de Virtual WAN.

Les partenaires d’équipement local du client SD-WAN peuvent activer l’automatisation afin d’automatiser la connectivité IPsec souvent fastidieuse et sujette aux erreurs à partir de leurs appareils respectifs. L’automatisation permet au contrôleur SD-WAN de communiquer avec Azure via l’API Virtual WAN pour configurer les sites de Virtual WAN, ainsi que pour envoyer (push) la configuration de tunnel IPsec nécessaire aux équipements locaux côté branche. Pour obtenir une description de l’automatisation de l’interconnexion de Virtual WAN par différents partenaires de SD-WAN, consultez [Conseils d’automatisation](virtual-wan-configure-automation-providers.md).

L’équipement local SD-WAN reste l’endroit où l’optimisation du trafic et la sélection du chemin d’accès sont implémentées et appliquées. 

Dans ce modèle, il est possible que certaines optimisations du trafic propriétaire du fournisseur basées sur des caractéristiques de trafic en temps réel ne soient pas prises en charge parce que la connexion au Virtual WAN est basée sur IPsec, et que le VPN IPsec se termine sur la passerelle VPN de Virtual WAN. Par exemple, la sélection du chemin d’accès dynamique au niveau de l’équipement local du client côté branche est possible en raison de l’échange de diverses informations de paquet réseau entre un appareil de la branche et un autre nœud SD-WAN, identifiant ainsi le meilleur lien à utiliser pour le trafic hiérarchisé de façon dynamique au niveau de la branche. Cette fonctionnalité peut être utile dans des zones où l’optimisation jusqu’au dernier kilomètre (entre la branche et le POP Microsoft le plus proche) est requise.

Avec Virtual WAN, les utilisateurs peuvent obtenir une sélection de chemin d’accès Azure, qui est la sélection de chemin d’accès basée sur une stratégie entre plusieurs liens ISP de l’équipement local du client côté branche à des passerelles VPN de Virtual WAN. Virtual WAN permet de configurer plusieurs liens (chemins d’accès) à partir du même équipement SD-WAN local du client côté branche. Chaque lien représente une connexion à deux tunnels à partir d’une adresse IP publique unique d’équipement SD-WAN côté branche vers deux instances différentes de la passerelle VPN de d’Azure Virtual WAN. Les fournisseurs de SD-WAN peuvent implémenter le chemin d’accès optimal vers Azure, en fonction des stratégies de trafic définies par leur moteur de stratégies sur les liens d’équipement local du client. Côté Azure, toutes les connexions entrantes sont traitées de la même façon.

## <a name="direct-interconnect-model-with-nva-in-vwan-hub"></a><a name="direct"></a>Modèle d’interconnexion directe avec NVA-in-VWAN-hub

:::image type="content" source="./media/sd-wan-connectivity-architecture/direct-nva.png" alt-text="Modèle d’interconnexion directe avec NVA-in-VWAN-hub":::

Ce modèle d’architecture prend en charge le déploiement d’une [appliance virtuelle réseau (Network Virtual Appliance, NVA) tierce directement dans le hub virtuel](./about-nva-hub.md). Cela permet aux clients qui souhaitent connecter les CPE de leur filiale à une NVA de la même marque dans le hub virtuel de tirer parti des capacités SD-WAN propriétaires de bout en bout lors de la connexion à des charges de travail Azure. 

Plusieurs partenaires Virtual WAN ont collaboré pour fournir une expérience qui configure automatiquement la NVA dans le cadre du processus de déploiement. Une fois que la NVA a été approvisionnée dans le hub virtuel, toute configuration supplémentaire qui peut être requise pour la NVA doit être effectuée par le biais du portail des partenaires NVA ou de l’application de gestion. L’accès direct à la NVA n’est pas disponible. Les NVA disponibles au déploiement directement dans le hub Azure Virtual WAN sont conçues spécifiquement pour être utilisés dans le hub virtuel. Pour les partenaires qui prennent en charge la NVA dans le hub VWAN et leurs guides de déploiement, consultez l’article [Partenaires Virtual WAN](virtual-wan-locations-partners.md#partners-with-integrated-virtual-hub-offerings).

L’équipement local SD-WAN reste l’endroit où l’optimisation du trafic et la sélection du chemin d’accès sont implémentées et appliquées.
Dans ce modèle, l’optimisation du trafic propriétaire des fournisseurs basée sur les caractéristiques de trafic en temps réel est prise en charge car la connectivité à Virtual WAN s’effectue via la NVA SD-WAN dans le hub.

## <a name="indirect-interconnect-model"></a><a name="indirect"></a>Modèle d’interconnexion indirecte

:::image type="content" source="./media/sd-wan-connectivity-architecture/indirect.png" alt-text="Modèle d’interconnexion indirecte":::

Dans ce modèle d’architecture, des équipements SD-WAN locaux du client côté branche sont connectés indirectement à des concentrateurs Virtual WAN. Comme l’illustre la figure, un équipement SD-WAN local virtuel du client est déployé dans un réseau virtuel d’entreprise. Cet équipement virtuel est, à son tour, connecté aux concentrateurs Virtual WAN via IPsec. L’équipement local virtuel du client sert de passerelle SD-WAN dans Azure. Les branches qui ont besoin d’accéder à leurs charges de travail dans Azure peuvent le faire via la passerelle v-CPE.

Étant donné que la connexion à Azure transite via la passerelle v-CPE (appliance virtuelle réseau), tout le trafic échangé entre les réseaux virtuels de charge de travail Azure et d’autres branches SD-WAN transite par l’appliance virtuelle réseau. Dans ce modèle, l’utilisateur est responsable de la gestion et de l’exploitation de l’appliance virtuelle réseau SD-WAN, y compris de sa haute disponibilité, de son extensibilité et de son routage.
  
## <a name="managed-hybrid-wan-model"></a><a name="hybrid"></a>Modèle WAN hybride géré

:::image type="content" source="./media/sd-wan-connectivity-architecture/hybrid.png" alt-text="Modèle WAN hybride géré":::

Dans ce modèle d’architecture, les entreprises peuvent tirer parti d’un service SD-WAN géré offert par un partenaire fournisseur de services gérés (MSP). Ce modèle est similaire aux modèles direct ou indirect décrits ci-dessus. Toutefois, dans ce modèle, la conception, l’orchestration et les opérations du SD-WAN sont fournis par le fournisseur du SD-WAN.

[Les partenaires fournisseurs de services gérés (MSP) Azure Networking](../networking/networking-partners-msp.md) peuvent utiliser [Azure Lighthouse](https://azure.microsoft.com/services/azure-lighthouse/) pour implémenter le SD-WAN et le service Virtual WAN dans l’abonnement Azure du client entreprise, ainsi que pour exploiter le réseau étendu hybride de bout en bout pour le compte du client. Ces fournisseurs de services gérés peuvent également être en mesure d’implémenter Azure ExpressRoute dans le Virtual WAN, et de l’exploiter en tant que service managé de bout en bout.

## <a name="additional-information"></a>Informations supplémentaires

* [FAQ sur Include](virtual-wan-faq.md)
* [Résolution de connectivité à distance](work-remotely-support.md)