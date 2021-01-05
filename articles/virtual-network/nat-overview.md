---
title: Présentation du service NAT de Réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Vue d’ensemble des fonctionnalités, des ressources, de l’architecture et de l’implémentation du service NAT de Réseau virtuel. Découvrez le fonctionnement du service NAT de Réseau virtuel et comment utiliser des ressources de passerelle NAT dans le cloud.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: nat
Customer intent: As an IT administrator, I want to learn more about Virtual Network NAT, its NAT gateway resources, and what I can use them for.
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: allensu
ms.openlocfilehash: 75a2bb187b2ed7a234e99d8cd293cb30148bcb1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91667195"
---
# <a name="what-is-virtual-network-nat"></a>Qu’est-ce que le service NAT de Réseau virtuel ?

Le service NAT (traduction d’adresses réseau) de Réseau virtuel simplifie la connectivité Internet sortante uniquement pour les réseaux virtuels. Quand il est configuré sur un sous-réseau, toute la connectivité sortante utilise vos adresses IP publiques statiques spécifiées.  Une connectivité sortante est possible sans équilibreur de charge ni adresses IP publiques directement attachées aux machines virtuelles. NAT est complètement managé et hautement résilient.

<!-- 
<img src="./media/nat-overview/flow-map.svg" width="270" align="center">
![Virtual Network NAT](./media/nat-overview/flow-map.svg)
-->

<p align="center">
  <img src="./media/nat-overview/flow-map.svg" alt="Figure depicts a NAT receiving traffic from internal subnets and directing it to a public IP (PIP) and an IP prefix." width="256" title="Service NAT de Réseau virtuel">
</p>



*Figure : Service NAT de Réseau virtuel*

## <a name="static-ip-addresses-for-outbound-only"></a>Adresses IP statiques pour le trafic sortant uniquement

Une connectivité sortante peut être définie pour chaque sous-réseau avec NAT.  Plusieurs sous-réseaux au sein du même réseau virtuel peuvent avoir différents services NAT. Un sous-réseau est configuré en spécifiant la ressource de passerelle NAT à utiliser. Tous les flux sortants UDP et TCP en provenance de toute instance de machine virtuelle utilisent la traduction d’adresses réseau (NAT). 

NAT est compatible avec les ressources d’adresses IP publiques ou les ressources de préfixes d’adresses IP publiques de la référence SKU standard, ou avec une combinaison des deux.  Vous pouvez utiliser directement un préfixe d’adresse IP publique ou distribuer les adresses IP publiques du préfixe entre plusieurs ressources de passerelle NAT. NAT nettoie tout le trafic vers la plage d’adresses IP du préfixe.  Tout filtrage des adresses IP de vos déploiements est à présent très simple.

Tout le trafic sortant du sous-réseau est traité par NAT automatiquement sans aucune configuration par le client.  Les routes définies par l’utilisateur ne sont pas nécessaires. NAT est prioritaire sur les autres scénarios de trafic sortant et remplace la destination Internet par défaut d’un sous-réseau.

## <a name="on-demand-snat-with-multiple-ip-addresses-for-scale"></a>SNAT à la demande avec plusieurs adresses IP à mettre à l’échelle

NAT utilise la « traduction d’adresses réseau de port » (PNAT ou PAT) et est recommandé pour la plupart des charges de travail. Les charges de travail dynamiques ou divergentes peuvent être facilement adaptées à l’allocation de flux de trafic sortant à la demande. Sont ainsi évités une planification préalable, une préallocation et un surprovisionnement des ressources sortantes. Les ressources de port SNAT sont partagées et disponibles sur tous les sous-réseaux à l’aide d’une ressource de passerelle NAT spécifique. Elles sont fournies en cas de besoin.

Une adresse IP publique attachée à NAT fournit jusqu’à 64 000 flux simultanés pour respectivement UDP et TCP. Vous pouvez commencer avec une seule adresse IP et évoluer jusqu’à 16 adresses IP à l’aide d’adresses IP publiques ou de préfixes IP publics, ou les deux.  Une ressource de passerelle NAT utilisera toutes les adresses IP associées à la ressource pour les connexions sortantes à partir de tous les sous-réseaux configurés avec la même ressource de passerelle NAT.

NAT autorise la création de flux depuis le réseau virtuel vers Internet. Le trafic de retour provenant d’Internet est uniquement autorisé en réponse à un flux actif.

Contrairement au service SNAT de flux sortant d’équilibreur de charge, NAT ne présente aucune restriction quant à l’adresse IP privée d’une instance de machine virtuelle pouvant établir des connexions sortantes.  Les configurations IP principales et secondaires peuvent créer une connexion Internet sortante avec NAT.

## <a name="coexistence-of-inbound-and-outbound"></a>Coexistence des trafics entrant et sortant

NAT est compatible avec les ressources SKU standard suivantes :

- Équilibrage de charge
- Adresse IP publique
- Préfixe d’adresse IP publique

Utilisées avec NAT, ces ressources fournissent une connectivité Internet entrante à vos sous-réseaux. NAT fournit toute la connectivité Internet sortante à partir de vos sous-réseaux.

NAT et les fonctionnalités de la référence SKU standard compatibles connaissent la direction dans laquelle le flux a démarré. Des scénarios de trafic entrant et sortant peuvent coexister. Ces scénarios reçoivent les traductions d’adresses réseau appropriées, car ces fonctionnalités connaissent la direction du flux. 

<!-- 
<img src="./media/nat-overview/flow-direction4.svg" width="500" align="center">
![Virtual Network NAT flow direction](./media/nat-overview/flow-direction4.svg)
-->
<p align="center">
  <img src="./media/nat-overview/flow-direction4.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with an instance-level public IP and a public load balancer." width="512" title="Direction du flux NAT de Réseau virtuel">
</p>

*Figure : Direction du flux NAT de Réseau virtuel*

## <a name="fully-managed-highly-resilient"></a>Complètement managé et hautement résilient

NAT a déjà fait l’objet d’un scale-out complet dès le départ. Aucune opération de montée en puissance ni de scale-out n’est nécessaire.  Azure gère le fonctionnement de NAT pour vous.  NAT a toujours plusieurs domaines d’erreur et peut supporter plusieurs défaillances sans interruption de service.

## <a name="tcp-reset-for-unrecognized-flows"></a>Réinitialisation TCP pour les flux non reconnus

Le côté privé de NAT envoie des paquets de réinitialisation TCP pour les tentatives de communication sur une connexion TCP qui n’existe pas. Par exemple, des connexions ont atteint le délai d’inactivité. Le paquet reçu suivant retourne une réinitialisation TCP à l’adresse IP privée pour signaler et forcer la fermeture de la connexion.

Le côté public de NAT ne génère pas de paquets de réinitialisation TCP ni tout autre trafic.  Seul le trafic produit par le réseau virtuel du client est émis.

## <a name="configurable-tcp-idle-timeout"></a>Délai d’inactivité TCP configurable

Un délai d’inactivité TCP par défaut de 4 minutes est utilisé et peut être augmenté à 120 minutes maximum. Toute activité sur un flux peut également réinitialiser le délai d’inactivité, y compris les conservations de connexion active TCP.

## <a name="regional-or-zone-isolation-with-availability-zones"></a>Isolement régional ou zonal avec des zones de disponibilité

Par défaut, le service NAT est régional. Quand vous créez des scénarios de [zones de disponibilité](../availability-zones/az-overview.md), NAT peut être isolé dans une zone spécifique (déploiement zonal).

<!-- 
<img src="./media/nat-overview/az-directions.svg" width="500" align="center">
![Virtual Network NAT with availability zones](./media/nat-overview/az-directions.svg)
-->

<p align="center">
  <img src="./media/nat-overview/az-directions.svg" alt="Figure depicts three zonal stacks, each of which contains a NAT gateway and a subnet." width="512" title="Service NAT de Réseau virtuel avec des zones de disponibilité">
</p>

*Figure : Service NAT de Réseau virtuel avec des zones de disponibilité*

## <a name="multi-dimensional-metrics-for-observability"></a>Métriques multidimensionnelles pour l’observabilité

Vous pouvez superviser le fonctionnement de votre NAT par le biais de métriques multidimensionnelles exposées dans Azure Monitor. Ces métriques peuvent servir à observer l’utilisation et à résoudre les problèmes.  Les ressources de passerelle NAT exposent les métriques suivantes :
- Octets
- Paquets
- Paquets ignorés
- Nombre total de connexions SNAT
- Transitions d’état de connexion SNAT par intervalle.

<!-- "ADD when PM is done" Learn more about [NAT gateway metrics](./nat-gateway-metrics.md) -->

## <a name="sla"></a>Contrat SLA

En disponibilité générale, le chemin de données NAT est au moins disponible à 99,9 %.

## <a name="pricing"></a>Tarifs

Pour plus d'informations sur les tarifs, consultez [Tarification des réseaux virtuels](https://azure.microsoft.com/pricing/details/virtual-network).

## <a name="availability"></a>Disponibilité

Le service NAT de réseau virtuel et la ressource de passerelle NAT sont disponibles dans toutes les [régions](https://azure.microsoft.com/global-infrastructure/regions/) des clouds Azure.

## <a name="suggestions"></a>Suggestions

Nous aimerions savoir comment nous pouvons améliorer le service. Proposez-nous de nouvelles fonctionnalités et votez pour celles que vous préférez en nous contactant sur [UserVoice for NAT](https://aka.ms/natuservoice).

## <a name="limitations"></a>Limites

* NAT est compatible avec des ressources d’adresses IP publiques, de préfixes d’adresses IP publiques et d’équilibreur de charge de la référence SKU standard. Les ressources de base (par exemple, un équilibreur de charge de base) et tous les produits qui en dérivent ne sont pas compatibles avec NAT.  Les ressources de base doivent être placées sur un sous-réseau non configuré avec NAT.
* La famille d’adresses IPv4 est prise en charge.  NAT n’interagit pas avec la famille d’adresses IPv6.  NAT ne peut pas être déployé sur un sous-réseau avec un préfixe IPv6.
* NAT ne peut pas s’étendre sur plusieurs réseaux virtuels.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur la [ressource de passerelle NAT](./nat-gateway-resource.md).
* [Utilisez UserVoice pour nous faire part des prochains développements dont vous aimeriez bénéficier concernant le service NAT de réseau virtuel](https://aka.ms/natuservoice).
