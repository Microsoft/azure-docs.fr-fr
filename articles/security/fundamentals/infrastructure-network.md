---
title: Architecture réseau Azure
description: Cet article fournit une description générale du réseau d’infrastructures Microsoft Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/08/2020
ms.author: terrylan
ms.openlocfilehash: 3b047489f9cfa3623c11e324cf58114b707c10b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89567863"
---
# <a name="azure-network-architecture"></a>Architecture réseau Azure
L’architecture réseau Azure assure la connectivité entre Internet et les centres de données Azure. Toute charge de travail déployée (IaaS, PaaS et SaaS) sur Azure s’appuie sur le réseau du centre de données Azure.

## <a name="network-topology"></a>Topologie du réseau
L’architecture réseau d’un centre de données Azure se compose des éléments suivants :

- Réseau de périphérie
- Réseau étendu
- Réseau de passerelles régionales
- Réseau du centre de données

![Diagramme du réseau Azure](./media/infrastructure-network/network-arch.png)

## <a name="network-components"></a>Composants réseau
Brève description des composants réseau.

- Réseau de périphérie

   - Point de démarcation entre le réseau Microsoft et d’autres réseaux (par exemple Internet, réseau d’entreprise)
   - Assure l’appairage Internet et [ExpressRoute](../../expressroute/expressroute-introduction.md) dans Azure

- Réseau étendu

   - Réseau principal intelligent Microsoft couvrant le monde entier
   - Assure la connectivité entre les [régions Azure](https://azure.microsoft.com/global-infrastructure/geographies/)

- Passerelles régionales

   - Point de regroupement pour tous les centres de données dans une région Azure
   - Assure une connectivité élevée entre les centres de données au sein d’une région Azure (par exemple, plusieurs centaines de térabits par centre de données)

- Réseau du centre de données

   - Assure la connectivité entre les serveurs dans le centre de données avec une bande passante sursouscrite faible

Les composants réseau ci-dessus sont conçus pour offrir une souplesse maximale afin de prendre en charge les activités du cloud toujours actives et toujours disponibles. La redondance est conçue et intégrée au réseau de l’aspect physique au protocole de contrôle.

## <a name="datacenter-network-resiliency"></a>Résilience réseau du centre de données
Nous allons illustrer le principe de conception de la résilience à l’aide du réseau de centre de données.

Le réseau de centre de données est une version modifiée d’un [réseau Clos](https://en.wikipedia.org/wiki/Clos_network), fournissant une bande passante de bissection élevée pour le trafic à l’échelle du cloud. Le réseau est construit en utilisant un grand nombre d’appareils standard pour réduire l’impact causé par une défaillance matérielle individuelle. Ces appareils sont situés de façon stratégique à des emplacements physiques différents avec un système d’alimentation et de refroidissement distinct pour réduire l’impact d’un événement d’environnement.  Sur le plan de contrôle, tous les appareils réseau s’exécutent en mode de routage de la couche 3 du modèle OSI, ce qui élimine le problème historique de la boucle de trafic. Tous les chemins entre les différents niveaux sont actifs pour assurer une redondance et une bande passante élevées avec le routage ECMP (Equal-Cost Multi-Path).

Le diagramme suivant montre que le réseau de centre de données est construit par différents niveaux d’appareils réseau. Les barres du diagramme représentent des groupes d’appareils réseau qui assurent la redondance et la connectivité à bande passante élevée.

![Réseau du centre de données](./media/infrastructure-network/datacenter-network.png)

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
