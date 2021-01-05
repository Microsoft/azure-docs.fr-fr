---
title: Concepts relatifs à Azure Load Balancer
description: Vue d’ensemble des concepts relatifs à Azure Load Balancer
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/13/2020
ms.author: allensu
ms.openlocfilehash: 3f8c288f950f34e1764c50e8eb74a8a73b39b3d7
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698527"
---
# <a name="azure-load-balancer-algorithm"></a>Algorithme Azure Load Balancer

L’équilibreur de charge offre plusieurs fonctionnalités pour les applications UDP et TCP.

## <a name="load-balancing-algorithm"></a>Algorithme d’équilibrage de charge

Vous pouvez créer une règle d’équilibrage de charge pour répartir le trafic entre le front-end et un pool de back-ends. Azure Load Balancer utilise un algorithme de hachage pour la distribution des flux entrants (pas d’octets). L’équilibreur de charge réécrit les en-têtes des flux dans les instances du pool de back-ends. Un serveur est disponible pour recevoir les nouveaux flux quand une sonde d’intégrité indique un point de terminaison de serveur principal sain.

Par défaut, l’équilibreur de charge utilise un hachage à cinq tuples.

Le hachage comprend les éléments suivants :

- **Une adresse IP source**
- **Port source**
- **Une adresse IP de destination**
- **Port de destination**
- **Un numéro de protocole IP pour mapper des flux sur les serveurs disponibles**

L’affinité avec une adresse IP source est créée à l’aide d’un hachage à deux ou trois tuples. Les paquets d’un même flux arrivent sur la même instance derrière le front-end soumis à l’équilibrage de charge.

Le port source change quand un client démarre un nouveau flux à partir de la même adresse IP source. Ainsi, le hachage à cinq tuples peut provoquer l’acheminement du trafic vers un autre point de terminaison back-end.
Pour plus d’informations, consultez [Configuration du mode de distribution pour Azure Load Balancer](./load-balancer-distribution-mode.md).

L’image suivante montre la distribution basée sur le hachage :

![Distribution basée sur le hachage](./media/load-balancer-overview/load-balancer-distribution.png)

*Figure : Distribution basée sur le hachage*

## <a name="application-independence-and-transparency"></a>Indépendance d’application et transparence

L’équilibreur de charge n’interagit pas directement avec TCP, UDP ou la couche Application. Tous les scénarios d’application TCP ou UDP peuvent être pris en charge. L’équilibreur de charge ne ferme pas de flux, n’en démarre pas ou n’interagit pas avec la charge utile du flux. L’équilibreur de charge ne fournit pas de fonctionnalité de passerelle de couche Application. Les liaisons de protocole se produisent toujours directement entre le client et l’instance de pool principal. La réponse à un flux entrant provient toujours d’une machine virtuelle. Lorsque le flux arrive sur la machine virtuelle, l’adresse IP source d’origine est également conservée.

- Chaque point de terminaison obtient une réponse d’une machine virtuelle. Par exemple, l’établissement d’une liaison TCP se fait entre le client et la machine virtuelle du back-end sélectionnée. La réponse à une demande d’un serveur frontal est une réponse générée par la machine virtuelle du serveur principal. Quand vous validez correctement la connectivité à un front-end, vous validez la connectivité de bout en bout jusqu’à au moins une machine virtuelle du back-end.
- Les charges utiles d’application sont transparentes pour l’équilibreur de charge. Toutes les applications UDP ou TCP peuvent être prises en charge.
- Étant donné que l’équilibreur de charge n’interagit pas avec la charge utile TCP et fournit le déchargement TLS, vous pouvez générer des scénarios chiffrés complets. L’utilisation de l’équilibreur de charge permet d’obtenir de vastes systèmes de scale-out pour les applications TLS en mettant fin à la connexion TLS sur la machine virtuelle. Par exemple, la capacité de création de clés pour votre session TLS est uniquement limitée par le type et le nombre de machines virtuelles que vous ajoutez au pool du serveur principal.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur les [composants](components.md) qui constituent Azure Load Balancer.
- Consultez [Créer un service Standard Load Balancer public](quickstart-load-balancer-standard-public-portal.md) pour bien démarrer avec Load Balancer : créez un équilibreur de charge, créez des machines virtuelles avec une extension IIS personnalisée installée et équilibrez la charge de l’application web entre les machines virtuelles.
- Découvrez les [connexions sortantes d’Azure Load Balancer](load-balancer-outbound-connections.md).
- En savoir plus sur [Azure Load Balancer](load-balancer-overview.md).
- Découvrez les [sondes d’intégrité](load-balancer-custom-probe-overview.md).
- En savoir plus sur les [Diagnostics Load Balancer Standard](load-balancer-standard-diagnostics.md).
- En savoir plus sur les [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).