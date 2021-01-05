---
title: Migration de la passerelle VPN du modèle Classic vers le modèle Resource Manager | Microsoft Docs
description: Cette page offre une vue d’ensemble de la migration de la passerelle VPN du modèle Classic vers le modèle Resource Manager.
documentationcenter: na
services: vpn-gateway
author: amsriva
manager: rossort
editor: amsriva
ms.assetid: caa8eb19-825a-4031-8b49-18fbf3ebc04e
ms.service: vpn-gateway
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/06/2020
ms.author: amsriva
ms.openlocfilehash: c9d7fb8be1894ffa5f8c35e16e1ed3aa0949b3ff
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488202"
---
# <a name="vpn-gateway-classic-to-resource-manager-migration"></a>Migration de la passerelle VPN du modèle Classic vers le modèle Resource Manager
Les passerelles VPN peuvent désormais être migrées du modèle de déploiement Classic vers le modèle de déploiement Resource Manager. Vous pouvez découvrir plus en détail les [fonctionnalités et les avantages](../azure-resource-manager/management/overview.md) d’Azure Resource Manager. Dans cet article, nous expliquons en détail comment migrer des déploiements Classic vers le modèle plus récent basé sur Resource Manager. 

Les passerelles VPN sont migrées dans le cadre d’une migration de réseau virtuel classique vers Resource Manager. Cette migration porte sur un réseau virtuel à la fois. Il n’existe aucune exigence supplémentaire en termes d’outils ou de conditions préalables à la migration. Les étapes de migration sont identiques à celles de la procédure de migration de réseau virtuel existante et sont documentés à la [page de migration des ressources IaaS](../virtual-machines/migration-classic-resource-manager-ps.md). À aucun moment l’accès aux données n’est interrompu pendant la migration, et les charges de travail existantes continuent donc de fonctionner sans perte de connectivité locale. L’adresse IP publique associée à la passerelle VPN ne change pas pendant le processus de migration. Cela veut dire que vous n’avez pas besoin de reconfigurer votre routeur local à l’issue de la migration.  

Le modèle Resource Manager est différent du modèle Classic. Il est constitué de passerelles de réseau virtuel, de passerelles de réseau local et de ressources de connexion. Ces éléments représentent respectivement la passerelle VPN proprement dite, le site local représentant l’espace d’adressage local et la connectivité entre les deux. À l’issue de la migration, les passerelles ne sont pas disponibles dans le modèle Classic et toutes les opérations de gestion au niveau des passerelles de réseau virtuel, des passerelles de réseau local et des objets de connexion doivent être effectuées selon le modèle Resource Manager.

## <a name="supported-scenarios"></a>Scénarios pris en charge
La plupart des scénarios de connectivité VPN courants sont couverts par la migration du modèle Classic vers le modèle Resource Manager. Les scénarios pris en charge sont les suivants :

* Connectivité point à site
* Connectivité site à site avec la passerelle VPN connectée à un emplacement local
* Connectivité entre deux réseaux virtuels en utilisant des passerelles VPN
* Plusieurs réseaux virtuels connectés à un même emplacement local
* Connectivité multisite
* Réseaux virtuels compatibles avec le tunneling forcé

Les scénarios non pris en charge sont les suivants :  

* Réseau virtuel doté d’une passerelle ExpressRoute et d’une passerelle VPN.
* Scénarios de transit dans lesquels des extensions de machine virtuelle sont connectées à des serveurs locaux. Les limitations de connectivité VPN de transit sont détaillées ci-après.

> [!NOTE]
> La validation CIDR est plus stricte dans le modèle Resource Manager que dans le modèle Classic. Avant de commencer la migration, vérifiez que les plages d’adresses classiques données sont conformes au format CIDR valide. Le routage CIDR peut être validé à l’aide de l’un des validateurs CIDR courants. La migration de réseaux virtuels ou de sites locaux avec des plages CIDR non valides se traduirait par un état d’échec.
> 
> 

## <a name="vnet-to-vnet-connectivity-migration"></a>Migration de connectivité de réseau virtuel à réseau virtuel
Dans le modèle Classic, la connectivité de réseau virtuel à réseau virtuel était obtenue en créant une représentation sous forme de site local du réseau virtuel connecté. Les clients étaient contraints de créer deux sites locaux qui représentaient les deux réseaux virtuels qui devaient être connectés entre eux. Ceux-ci étaient ensuite connectés aux réseaux virtuels correspondants à l’aide d’un tunnel IPsec pour établir la connectivité entre les deux réseaux virtuels. Ce modèle présente des difficultés en termes de gestion dans le sens où les modifications apportées à la plage d’adresses d’un réseau virtuel doivent aussi être répercutées dans la représentation sous forme de site local correspondante. Dans le modèle Resource Manager, cette solution de contournement n’est plus nécessaire. La connexion entre les deux réseaux virtuels peut être directement obtenue en utilisant le type de connexion « Vnet2Vnet » dans la ressource de connexion. 

![Capture d’écran de la migration de réseau virtuel à réseau virtuel.](./media/vpn-gateway-migration/migration1.png)

Pendant la migration du réseau virtuel, nous détectons que l’entité connectée à la passerelle VPN du réseau virtuel actif est un autre réseau virtuel et faisons en sorte qu’à l’issue de la migration de ces deux réseaux virtuels, les deux sites locaux représentant l’autre réseau virtuel ne soient plus visibles. Le modèle classique à deux passerelles VPN, deux sites locaux et deux connexions entre les deux est transformé en modèle Resource Manager à deux passerelles VPN et deux connexions de type Vnet2Vnet.

## <a name="transit-vpn-connectivity"></a>Connectivité VPN de transit
Vous pouvez configurer des passerelles VPN dans une topologie de façon à établir une connectivité locale pour un réseau virtuel en le connectant à un autre réseau virtuel directement connecté en local. Il s’agit d’une connectivité VPN de transit dans laquelle les instances présentes dans le premier réseau virtuel sont connectées à des ressources locales par transit vers la passerelle VPN du réseau virtuel connecté directement connecté en local. Pour mettre en place cette configuration dans le modèle de déploiement Classic, vous deviez créer un site local avec des préfixes agrégés représentant à la fois le réseau virtuel connecté et l’espace d’adressage local. Ce site local représentatif est ensuite connecté au réseau virtuel pour obtenir une connectivité de transit. Ce modèle Classic présente aussi des défis comparables en matière de gestion, car les modifications apportées à la plage d’adresses locale doivent aussi être répercutées sur le site local représentant l’agrégat du réseau virtuel et de l’emplacement local. L’introduction de la prise en charge du protocole BGP dans les passerelles prises en charge par Resource Manager facilite la gestion dans la mesure où les passerelles connectées peuvent déterminer les itinéraires au niveau local sans modification manuelle des préfixes.

![Capture d’écran du scénario de routage de transit.](./media/vpn-gateway-migration/migration2.png)

Sachant que nous transformons la connectivité de réseau virtuel à réseau virtuel sans nécessiter de sites locaux, le scénario de transit perd la connectivité locale pour le réseau virtuel qui est indirectement connecté en local. Les effets de la perte de connectivité peuvent être atténués des deux façons suivantes, une fois la migration terminée : 

* Activation du protocole BGP sur les passerelles VPN qui sont connectées entre elles et en local. L’activation du protocole BGP a pour effet de restaurer la connectivité sans aucune autre modification de la configuration, car les itinéraires sont déterminés et annoncés entre les passerelles de réseau virtuel. Notez que l’option BGP est uniquement disponible sur les références (SKU) Standard et supérieures.
* Établissement d’une connexion explicite du réseau virtuel concerné vers la passerelle de réseau local représentant l’emplacement local. Cela passe aussi par la modification de la configuration du routeur local pour créer et configurer le tunnel IPsec.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous en savez plus sur la prise en charge de la migration de passerelle VPN, consultez [Migration de ressources IaaS d’un environnement Classic vers Resource Manager](../virtual-machines/migration-classic-resource-manager-ps.md) pour bien démarrer.