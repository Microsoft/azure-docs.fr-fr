---
title: À propos d’Azure ExpressRoute Direct
description: Découvrez les principales fonctionnalités d’Azure ExpressRoute Direct, ainsi que les informations nécessaires pour l’intégration à ExpressRoute Direct, comme les références (SKU) disponibles et les exigences techniques.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 08/12/2019
ms.author: duau
ms.openlocfilehash: 0365fd8d0d04b7d144ca6826d3d5eff976558eb3
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202148"
---
# <a name="about-expressroute-direct"></a>À propose de ExpressRoute Direct

ExpressRoute Direct vous offre la possibilité de vous connecter directement au réseau mondial Microsoft à partir d’emplacements de peering qui sont distribués stratégiquement dans le monde entier. ExpressRoute Direct offre une double connectivité de 100 Gbits/s ou 10 Gbits/squi prend en charge la connectivité Active/Active à grande échelle.

Les principales fonctionnalités offertes par ExpressRoute Direct sont les suivantes (sans s’y limiter) :

* Ingestion de données massive dans des services comme le stockage et Cosmos DB
* Isolation physique pour les secteurs qui sont réglementés, pour lesquels la connectivité isolée et dédiée est obligatoire, notamment : banque, secteur public et commerce de détail
* Contrôle précis de la distribution du circuit en fonction de l’unité commerciale

## <a name="onboard-to-expressroute-direct"></a>Intégration à ExpressRoute Direct

Vous devez commencer par inscrire votre abonnement pour pouvoir utiliser ExpressRoute Direct. Envoyez pour cela un e-mail à <ExpressRouteDirect@microsoft.com> avec votre ID d’abonnement et les informations suivantes :

* Scénarios que vous souhaitez réaliser avec **ExpressRoute Direct**
* Préférences d’emplacement : consultez l’article [Partenaires et emplacements de peering](expressroute-locations-providers.md) pour obtenir la liste complète de tous les emplacements
* Chronologie d'implémentation
* D’autres questions

## <a name="expressroute-using-a-service-provider-and-expressroute-direct"></a>ExpressRoute avec fournisseur de services et ExpressRoute Direct

| **ExpressRoute avec fournisseur de services** | **ExpressRoute Direct** | 
| --- | --- |
| Utilisation de fournisseurs de services pour permettre une intégration et une connectivité rapides dans l’infrastructure existante | Infrastructure de 100 Gbits/s / 10 Gbits/s et gestion complète de toutes les couches obligatoires
| Intégration avec des centaines de fournisseurs, y compris Ethernet et MPLS | Capacité directe dédiée pour les secteurs réglementés et ingestion massive de données |
| Références SKU de circuits de 50 Mbit/s à 10 Gbit/s | Possibilité pour le client de sélectionner une combinaison entre les références SKU de circuits suivantes sur ExpressRoute Direct 100 Gbits/s : <ul><li>5 Gbit/s</li><li>10 Gbits/s</li><li>40 Gbits/s</li><li>100 Gbits/s</li></ul> Possibilité pour le client de sélectionner une combinaison entre les références SKU de circuits suivantes sur ExpressRoute Direct 10 Gbits/s :<ul><li>1 Gbit/s</li><li>2 Gbit/s</li><li>5 Gbit/s</li><li>10 Gbits/s</li></ul>
| Optimisé pour monolocataire | Optimisé pour un seul locataire avec plusieurs unités commerciales et plusieurs environnements de travail

## <a name="expressroute-direct-circuits"></a>Circuits ExpressRoute Direct

Microsoft Azure ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. Avec ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, comme Microsoft Azure et Microsoft 365.

Chaque emplacement de peering dispose d’un accès au réseau global Microsoft, à n’importe quelle région d’une zone géopolitique par défaut et à toutes les régions du monde à l’aide d’un circuit premium.  

Dans la plupart des scénarios, la fonctionnalité se comporte comme des circuits qui utilisent un fournisseur de services ExpressRoute. Afin d’offrir un contrôle plus précis et de nouvelles fonctionnalités à l’aide d’ExpressRoute Direct, les circuits sont dotés de certaines fonctions clés.

## <a name="circuit-skus"></a>Références SKU de circuit

ExpressRoute Direct prend en charge des scénarios d’ingestion massive de données dans le Stockage Azure et d’autres services de Big Data. Désormais, les circuits ExpressRoute sur ExpressRoute Direct 100 Gbits/s prennent également en charge les références SKU de circuit **100 Gbits/s** et **40 Gbits/s**. La bande passante des paires de ports physiques est de **100 ou 10 Gbits/s**, et ils peuvent disposer de plusieurs circuits virtuels. Tailles des circuits :

| **ExpressRoute Direct 100 Gbits/s** | **ExpressRoute Direct 10 Gbits/s** | 
| --- | --- |
| **Bande passante souscrite** : 200 Gbits/s | **Bande passante souscrite** : 20 Gbits/s |
| <ul><li>5 Gbit/s</li><li>10 Gbits/s</li><li>40 Gbits/s</li><li>100 Gbits/s</li></ul> | <ul><li>1 Gbit/s</li><li>2 Gbit/s</li><li>5 Gbit/s</li><li>10 Gbits/s</li></ul>

## <a name="technical-requirements"></a>Exigences techniques

* Interfaces Microsoft Enterprise Edge Router (MSEE) :
    * Double port Ethernet 10 ou 100 Gigabits uniquement sur une paire de routeurs
    * Connectivité fibre optique LR monomode
    * IPv4 et IPv6
    * MTU IP 1 500 octets

* Connectivité commutateur/routeur couche 2/3 :
    * Doit prendre en charge la balise 1 802.1Q (Dot1Q) ou l’encapsulation à deux balises 802.1Q (QinQ)
    * EtherType = 0x8100
    * Vous devez ajouter la balise VLAN externe (STAG) en fonction de l’ID de VLAN spécifié par Microsoft - *Applicable uniquement sur QinQ*
    * Doit prendre en charge plusieurs sessions BGP (VLAN) par port et par appareil
    * Connectivité IPv4 et IPv6. *Pour IPv6, aucune sous-interface supplémentaire n’est créée. L’adresse IPv6 sera ajoutée à la sous-interface existante*. 
    * Facultatif : Prise en charge du protocole [Bidirectional Forwarding Detection (BFD)](./expressroute-bfd.md), qui est configurée par défaut sur l’ensemble des peerings privés et des circuits ExpressRoute

## <a name="vlan-tagging"></a>Balisage VLAN

ExpressRoute Direct prend en charge le balisage VLAN QinQ et Dot1Q.

* Grâce au **balisage VLAN QinQ**, il est possible d’isoler des domaines de routage individuellement pour chaque circuit ExpressRoute. Azure alloue dynamiquement une balise-S (S-Tag) non modifiable lors de la création du circuit. Chaque peering du circuit (privé et Microsoft) utilise une balise-C (C-Tag) unique en tant que VLAN. La balise-C (C-Tag) n’a pas besoin d’être unique sur les circuits des ports ExpressRoute Direct.

* Grâce au **balisage VLAN Dot1Q**, il est possible d’avoir un VLAN balisé unique pour chaque paire de ports ExpressRoute Direct. Une balise-C (C-Tag) utilisée sur un peering doit être unique sur tous les circuits et peerings de la paire de ports ExpressRoute Direct.

## <a name="workflow"></a>Workflow

[![workflow](./media/expressroute-erdirect-about/workflow1.png)](./media/expressroute-erdirect-about/workflow1.png#lightbox)

## <a name="sla"></a>Contrat SLA

ExpressRoute Direct fournit le même contrat de niveau de service professionnel avec des connexions redondantes Active/Active dans le réseau mondial Microsoft. L’infrastructure ExpressRoute est redondante, et la connectivité au réseau mondial Microsoft est redondante, différente, et s’adapte aux exigences du client. 

## <a name="next-steps"></a>Étapes suivantes

[Configurer ExpressRoute Direct](expressroute-howto-erdirect.md)