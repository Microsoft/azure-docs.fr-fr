---
title: Surveillance de l’infrastructure Azure
description: Découvrez les différents aspects de la surveillance de l’infrastructure du réseau de production Azure, tels que l’analyse des vulnérabilités.
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
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 7b75c9dc874a41d4221c55a8b00dd12d943e80fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87542940"
---
# <a name="azure-infrastructure-monitoring"></a>Surveillance de l’infrastructure Azure   

## <a name="configuration-and-change-management"></a>Gestion des configurations et des changements
Azure revoit et met à jour les paramètres de configuration et les configurations de référence du matériel, des logiciels et des appareils réseau chaque année. Les modifications sont développées, testées et approuvées avant le passage d’un environnement de développement et/ou de test à un environnement de production.

Les configurations de référence requises pour les services Azure sont revues par l’équipe de sécurité et de conformité Azure et par les équipes des services. La révision d’une équipe de service fait partie des tests avant le déploiement de leur service de production.

## <a name="vulnerability-management"></a>Gestion des vulnérabilités
La gestion des mises à jour de sécurité aide à protéger les systèmes des vulnérabilités connues. Azure utilise des systèmes de déploiement intégrés pour gérer la distribution et l’installation des mises à jour de sécurité destinées aux logiciels Microsoft. Azure peut également s’appuyer sur les ressources de Microsoft Security Response Center (MSRC). Le centre MSRC identifie, surveille et résout les incidents de sécurité et les vulnérabilités du cloud 24 heures sur 24, 7 jours sur 7.

## <a name="vulnerability-scanning"></a>Analyse des vulnérabilités
L’analyse des vulnérabilités est effectuée sur les systèmes d’exploitation serveur, les bases de données et les appareils réseau. Les recherches de vulnérabilités sont effectuées au moins tous les trimestres. Azure passe des contrats avec des évaluateurs indépendants pour effectuer des tests de pénétration de la frontière Azure. De plus, des exercices Red-Team sont régulièrement effectués, et les résultats sont utilisés pour apporter des améliorations à la sécurité.

## <a name="protective-monitoring"></a>Surveillance à des fins de protection
Le service de sécurité Azure a défini des critères pour une surveillance active. Les équipes de service configurent des outils de surveillance active conformes à ces critères. Les outils de surveillance active incluent Microsoft Monitoring Agent (MMA) et System Center Operations Manager. Ces outils sont configurés pour fournir des alertes en temps réel au personnel du service de sécurité Microsoft Azure quand une action immédiate est nécessaire.

## <a name="incident-management"></a>Gestion des incidents
Microsoft implémente un processus de gestion des incidents de sécurité pour permettre une réponse coordonnée aux incidents potentiels.

Si Microsoft prend connaissance d’un accès non autorisé à des données clientes stockées sur ses équipements ou dans ses installations, ou d’un accès non autorisé à ces équipements ou installations entraînant une perte, une divulgation ou une altération des données clients, Microsoft procède aux actions suivantes :

- Informer rapidement le client de l’incident de sécurité.
- Investiguer l’incident de sécurité dans les plus brefs délais et fournir aux clients des informations détaillées sur l’incident.
- Appliquer des procédures raisonnables permettant d’atténuer rapidement les effets de l’incident de sécurité et de réduire les dommages encourus.

Un cadre de gestion des incidents impliquant la définition de rôles et l’allocation de responsabilités a été défini. L’équipe de gestion des incidents de sécurité Azure est chargée de la gestion des incidents de sécurité, notamment du processus d’escalade. Elle doit également garantir l’implication d’équipes de spécialistes le cas échéant. Les responsables des opérations Azure sont chargés de superviser l’investigation et la résolution des incidents de sécurité et de confidentialité.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
