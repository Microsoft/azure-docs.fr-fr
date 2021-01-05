---
title: 'Azure ExpressRoute : Configuration requise QoS'
description: Cette page fournit des informations détaillées sur la configuration et la gestion de QoS. Skype Entreprise et les services vocaux sont évoqués.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: duau
ms.openlocfilehash: 7f14f43fa341df40ecd35340b7311e1acd18004c
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204705"
---
# <a name="expressroute-qos-requirements"></a>Configuration requise pour ExpressRoute QoS
Skype Entreprise a différentes charges de travail nécessitant un traitement QoS différencié. Si vous envisagez d’utiliser des services vocaux via ExpressRoute, vous devez respecter les conditions requises décrites ci-dessous.

![Diagramme qui montre les services vocaux passant par ExpressRoute.](./media/expressroute-qos/expressroute-qos.png)

> [!NOTE]
> La configuration requise pour QoS ne s’applique qu’au peering Microsoft. Les valeurs DSCP de votre trafic réseau reçues sur le peering public Azure et le peering privé Azure seront remises à 0. 
> 
> 

Le tableau suivant fournit la liste des marquages DSCP utilisés par Microsoft Teams et Skype Entreprise. Pour plus d’informations, voir [Gestion de QoS pour Skype Entreprise](/SkypeForBusiness/manage/network-management/qos/managing-quality-of-service-QoS) .

| **Classe de trafic** | **Traitement (marquage DSCP)** | **Charges de travail Microsoft Teams et Skype Entreprise** |
| --- | --- | --- |
| **Voice** |EF (46) |Skype / Microsoft Teams / Lync voice |
| **Interactive** |AF41 (34) |Vidéo, VBSS |
| |AF21 (18) |Partage d’application | 
| **Par défaut** |AF11 (10) |Transfert de fichiers |
| |CS0 (0) |Tout autre élément |

* Vous devez classer les charges de travail et marquer les valeurs DSCP appropriées. Suivez les instructions fournies [ici](/SkypeForBusiness/manage/network-management/qos/configuring-port-ranges-for-your-skype-clients#configure-quality-of-service-policies-for-clients-running-on-windows-10) sur la définition des marquages DSCP dans votre réseau.
* Vous devez configurer et prendre en charge plusieurs files d’attente QoS au sein de votre réseau. La voix doit être une classe autonome et recevoir le traitement Entity Framework spécifié dans la norme [RFC 3246](https://www.ietf.org/rfc/rfc3246.txt). 
* Vous pouvez choisir le mécanisme de mise en file d’attente, la stratégie de détection de congestion et l’allocation de bande passante par classe de trafic. Toutefois, le marquage DSCP pour Skype For Business Workloads doit être conservé. Si vous utilisez des marquages DSCP non répertoriés ci-dessus, par exemple AF31 (26), vous devez réécrire cette valeur DSCP 0 avant d’envoyer le paquet à Microsoft. Microsoft envoie uniquement les paquets marqués avec la valeur DSCP indiquée dans le tableau ci-dessus. 

## <a name="next-steps"></a>Étapes suivantes
* Consultez la configuration requise pour [Routage](expressroute-routing.md) et [NAT](expressroute-nat.md).
* Consultez les liens suivants pour configurer votre connexion ExpressRoute.
  
  * [Création d’un circuit ExpressRoute](expressroute-howto-circuit-classic.md)
  * [Configuration du routage](expressroute-howto-routing-classic.md)
  * [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-classic.md)