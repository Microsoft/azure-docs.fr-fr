---
title: Métriques et alertes pour le service NAT du réseau virtuel Azure
titleSuffix: Azure Virtual Network
description: Prenez connaissance des métriques et des alertes Azure Monitor disponibles pour le service NAT du réseau virtuel Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: nat
Customer intent: As an IT administrator, I want to understand available Azure Monitor metrics and alerts for Virtual Network NAT.
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/04/2020
ms.author: allensu
ms.openlocfilehash: e3c47a60a6cda074eba7b5c3292577c29f50c2ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87424049"
---
# <a name="azure-virtual-network-nat-metrics"></a>Métriques du service NAT du réseau virtuel Azure

Les ressources de la passerelle NAT du réseau virtuel Azure fournissent des métriques multidimensionnelles. Vous pouvez utiliser ces métriques pour observer l’opération et pour la [résolution des problèmes](troubleshoot-nat.md).  Il est possible de configurer des alertes pour des problèmes critiques comme l’épuisement des ressources SNAT.

<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway resource that consumes all IP addresses for a public IP prefix and directs that traffic to and from two subnets of virtual machines and a virtual machine scale set." width="256" title="Service NAT de Réseau virtuel pour le trafic sortant vers Internet">
</p>

*Figure : Service NAT de Réseau virtuel pour le trafic sortant vers Internet*

## <a name="metrics"></a>Mesures

Les ressources de passerelle NAT fournissent les métriques multidimensionnelles suivantes dans Azure Monitor :

| Métrique | Description | Agrégation recommandée | Dimensions |
|---|---|---|---|
| Octets | Octets entrants et sortants traités | SUM | Direction (In ; Out), Protocole (6 TCP ; 17 UDP) |
| Paquets | Paquets entrants et sortants traités | SUM | Direction (In ; Out), Protocole (6 TCP ; 17 UDP) |
| Paquets ignorés | Paquets ignorés par la passerelle NAT | SUM | / |
| Nombre de connexions SNAT | Transitions d’état par intervalle | SUM | État de la connexion, Protocole (6 TCP; 17 UDP) |
| Nombre total de connexions SNAT | Connexions SNAT actives actuelles (~ ports SNAT utilisés) | SUM | Protocole (6 TCP ; 17 UDP) |


## <a name="alerts"></a>Alertes

Les alertes relatives aux métriques peuvent être configurées dans Azure Monitor pour chacune des [métriques](#metrics) précédentes.

## <a name="limitations"></a>Limites

Resource Health n’est pas pris en charge.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur le service [Nat de Réseau virtuel](nat-overview.md).
* Découvrir la [ressource de passerelle NAT](nat-gateway-resource.md)
* Découvrir [Azure Monitor](../azure-monitor/overview.md)
* Découvrez la [résolution des problèmes liés aux ressources de passerelle NAT](troubleshoot-nat.md).
* [Utilisez UserVoice pour nous faire part des prochains développements dont vous aimeriez bénéficier concernant le service NAT de réseau virtuel](https://aka.ms/natuservoice).


