---
title: Disponibilité de l’infrastructure Azure - Sécurité Azure
description: Cet article fournit des informations sur ce que Microsoft fait pour sécuriser l’infrastructure Azure et garantir une disponibilité optimale des données des clients.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/28/2019
ms.author: terrylan
ms.openlocfilehash: c50c4faf47caf0a7519d61fdc8989ec9fd809d78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68727228"
---
# <a name="azure-infrastructure-availability"></a>Disponibilité de l’infrastructure Azure
Cet article fournit des informations sur ce que Microsoft fait pour sécuriser l’infrastructure Azure et garantir une disponibilité optimale des données des clients. Azure fournit une disponibilité robuste basée sur une redondance étendue obtenue grâce à la technologie de virtualisation.

## <a name="temporary-outages-and-natural-disaster"></a>Pannes temporaires et catastrophe naturelle
L’équipe Microsoft Cloud Infrastructure and Operations conçoit, génère, exploite et améliore la sécurité de l’infrastructure cloud. Cette équipe garantit que l’infrastructure Azure offre une disponibilité et une fiabilité élevées, une haute efficacité et une extensibilité intelligente. L’équipe fournit un cloud privé plus sécurisé et fiable.

Des onduleurs et d’énormes banques de batteries garantissent la continuité de la fourniture en électricité en cas de brève interruption d’alimentation. Des groupes électrogènes d’urgence fournissent une alimentation de secours en cas de panne prolongée et de maintenance planifiée. En cas de catastrophe naturelle, le centre de données dispose de réserves locales de carburant.

Des réseaux à fibre optique robustes et à grande vitesse connectent les centres de données avec d’autres hubs majeurs et avec les utilisateurs d’Internet. Des nœuds de calcul hébergent les charges de travail plus près des utilisateurs afin de réduire la latence, de fournir une géoredondance et d’accroître la résilience globale du service. Une équipe d’ingénieurs travaille 24 heures sur 24 afin de garantir la disponibilité permanente des services.

Microsoft garantit une haute disponibilité grâce à une surveillance avancée et une réponse aux incidents, à la prise en charge des services et à une capacité de basculement de secours. Les centres d’opérations Microsoft géographiquement distribués fonctionnent 24 heures sur 24, 7 jours sur 7, 365 jours par an. Le réseau Azure est l’un des plus grands au monde. Le réseau de distribution de contenu à fibre optique connecte les centres de données et les nœuds de périphérie afin de garantir la fiabilité et de hautes performances.

## <a name="disaster-recovery"></a>Récupération d'urgence
Azure préserve la durabilité de vos données à deux emplacements. Vous pouvez choisir l’emplacement du site de sauvegarde. Aux deux emplacements, Azure tient constamment à jour trois réplicas sains de vos données.

## <a name="database-availability"></a>Disponibilité des bases de données
Azure garantit qu’une base de données est accessible sur Internet par le biais d’une passerelle Internet avec une disponibilité de base de données soutenue. La surveillance évalue l’intégrité et l’état des bases de données actives toutes les cinq minutes.

## <a name="storage-availability"></a>Disponibilité du stockage
Azure offre un stockage par le biais d’un service de stockage hautement scalable et durable qui fournit des points de terminaison de connectivité. Cela signifie qu’une application peut accéder directement au service de stockage. Le service de stockage traite efficacement les requêtes de stockage entrantes avec une intégrité transactionnelle.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
