---
title: Configuration IP flottante d’Azure Load Balancer
description: Vue d’ensemble de l’IP flottante dans Azure Load Balancer
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
ms.openlocfilehash: 01cca2f2233ed5cdfb3003bb44c40f481bcf9bda
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94699404"
---
# <a name="azure-load-balancer-floating-ip-configuration"></a>Configuration IP flottante d’Azure Load Balancer

L’équilibreur de charge offre plusieurs fonctionnalités pour les applications UDP et TCP.

## <a name="floating-ip"></a>IP flottante

Dans certains scénarios d’application, il est préférable ou nécessaire que plusieurs instances d’application utilisent le même port sur une machine virtuelle du pool principal. Des exemples courants de réutilisation de port incluent le clustering pour la haute disponibilité, les appliances réseau virtuelles, ainsi que l’exposition de plusieurs points de terminaison TLS sans nouveau chiffrement. Si vous souhaitez réutiliser le port principal pour plusieurs règles, vous devez activer la fonctionnalité d’adresse IP flottante dans la définition des règles.

**Adresse IP flottante** est le terme Azure désignant une partie de ce que l’on appelle le « retour direct du serveur » (Direct Server Return, DSR). La configuration DSR comprend deux parties :

- une topologie de flux ;
- un schéma de mappage d’adresses IP.

Du point de vue de la plate-forme, Azure Load Balancer fonctionne toujours dans une topologie de flux DSR, que l’adresse IP flottante soit activée ou non. Cela signifie que la partie sortante d’un flux est toujours correctement réécrite pour retourner directement à l’origine.
À défaut d’adresse IP flottante, Azure expose un schéma de mappage d’adresse IP d’équilibrage de charge traditionnel pour faciliter l’utilisation (adresse IP des instances de machine virtuelle). L’activation de l’adresse IP flottante modifie le mappage de l’adresse IP à l’adresse IP frontale de l’équilibreur de charge afin de permettre davantage de flexibilité. En savoir plus [ici](load-balancer-multivip-overview.md).

## <a name="limitations"></a><a name = "limitations"></a>Limitations

- Une adresse IP flottante n’est actuellement pas prise en charge sur des configurations IP secondaires pour des scénarios d’équilibrage de charge

## <a name="next-steps"></a>Étapes suivantes

- Pour bien démarrer avec les équilibreurs de charge, consultez [Créer un équilibreur de charge standard public](quickstart-load-balancer-standard-public-portal.md).
- Découvrez les [connexions sortantes d’Azure Load Balancer](load-balancer-outbound-connections.md).
- En savoir plus sur [Azure Load Balancer](load-balancer-overview.md).
- Découvrez les [sondes d’intégrité](load-balancer-custom-probe-overview.md).
- En savoir plus sur les [Diagnostics Load Balancer Standard](load-balancer-standard-diagnostics.md).
- En savoir plus sur les [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).