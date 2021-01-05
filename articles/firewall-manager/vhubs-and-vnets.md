---
title: Quelles sont les options d’architecture d’Azure Firewall Manager ?
description: Comparez et opposez l’utilisation d’une architecture de réseau virtuel hub à l’utilisation d’une architecture de hub virtuel sécurisé avec Azure Firewall Manager.
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: article
ms.date: 09/14/2020
ms.author: victorh
ms.openlocfilehash: 71ff23e749139087f24da406474403167dcc1c0d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90563146"
---
# <a name="what-are-the-azure-firewall-manager-architecture-options"></a>Quelles sont les options d’architecture d’Azure Firewall Manager ?

Azure Firewall Manager assure la gestion de la sécurité pour deux types d’architecture réseau :

- **hub virtuel sécurisé**

   Un [hub Azure Virtual WAN](../virtual-wan/virtual-wan-about.md#resources) est une ressource gérée par Microsoft qui vous permet de créer facilement des architectures « Hub and Spoke ». Quand des stratégies de sécurité et de routage sont associées à un hub, on parle de *[hub virtuel sécurisé](secured-virtual-hub.md)* . 
- **Réseau virtuel hub**

   Il s’agit d’un réseau virtuel Azure standard que vous créez et gérez vous-même. Quand des stratégies de sécurité sont associées à un hub de ce type, on parle de *réseau virtuel hub*. Pour l’instant, seule la stratégie de pare-feu Azure est prise en charge. Vous pouvez appairer des réseaux virtuels spoke qui contiennent vos services et serveurs de charge de travail. Vous pouvez également gérer les pare-feu dans des réseaux virtuels autonomes qui ne sont appairés à aucun réseau spoke.

## <a name="comparison"></a>Comparaison

Le tableau suivant compare ces deux options d’architecture et peut vous aider à déterminer celle qui convient le mieux aux critères de sécurité de votre organisation :


|  |**Réseau virtuel hub**|**Hub virtuel sécurisé**  |
|---------|---------|---------|
|**Ressource sous-jacente**     |Réseau virtuel|Hub Virtual WAN|
|**Hub and Spoke**     |Utiliser le peering de réseaux virtuels|Automatisé à l’aide d’une connexion au réseau virtuel hub|
|**Connectivité locale**     |Passerelle VPN jusqu’à 10 Gbits/s et 30 connexions S2S ; ExpressRoute|Passerelle VPN plus scalable jusqu’à 20 Gbits/s et 1 000 connexions S2S ; ExpressRoute|
|**Connectivité automatisée des branches à l’aide de SDWAN**      |Non pris en charge|Prise en charge|
|**Hubs par région**     |Plusieurs réseaux virtuels par région|Un seul hub virtuel par région. Plusieurs hubs possibles avec plusieurs réseaux étendus (WAN) virtuels|
|**Pare-feu Azure - Adresses IP publiques multiples**      |Fourni par le client|Généré automatiquement|
|**Zones de disponibilité du pare-feu Azure**     |Prise en charge|Pas encore disponible|
|**Sécurité Internet avancée avec la sécurité tierce comme partenaires de services**     |Connectivité VPN managée et établie par le client au service partenaire de votre choix|Automatisé par le biais du flux Fournisseur partenaire de sécurité et de l’expérience de gestion du partenaire|
|**Gestion centralisée des routes pour router le trafic vers le hub**     |Route définie par l’utilisateur et gérée par le client|Pris en charge à l’aide de BGP|
|**Prise en charge de plusieurs fournisseurs de sécurité**|Prise en charge du tunneling forcé configuré manuellement pour les pare-feu tiers|Prise en charge automatisée de deux fournisseurs de sécurité : Pare-feu Azure pour le filtrage du trafic privé et tiers pour le filtrage Internet|
|**Pare-feu d’applications web sur Application Gateway** |Pris en charge sur un réseau virtuel|Actuellement pris en charge sur un réseau spoke|
|**Appliance virtuelle réseau**|Pris en charge sur un réseau virtuel|Actuellement pris en charge sur un réseau spoke|
|**Prise en charge d’Azure DDoS Protection Standard**|Oui|Non|

## <a name="next-steps"></a>Étapes suivantes

- Passer en revue [Vue d’ensemble du déploiement d’Azure Firewall Manager](deployment-overview.md)
- En savoir plus sur les [hubs virtuels sécurisés](secured-virtual-hub.md)