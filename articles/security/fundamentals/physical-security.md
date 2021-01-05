---
title: Sécurité physique des centres de données Azure - Microsoft Azure | Microsoft Docs
description: Cet article décrit les actions entreprises par Microsoft pour sécuriser les centres de données Azure, notamment l’infrastructure physique, la sécurité et les offres de conformité.
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
ms.date: 07/10/2020
ms.author: terrylan
ms.openlocfilehash: 3eac3c9abda74f0414fc97512f011d235d3991f5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87076328"
---
# <a name="azure-facilities-premises-and-physical-security"></a>Sécurité locale et physique des centres de données Azure
Cet article décrit ce que Microsoft fait pour sécuriser l’infrastructure Azure.

## <a name="datacenter-infrastructure"></a>Infrastructure de centre de données
Azure est composé d’une [infrastructure de centres de données distribuée à l’échelle mondiale](https://azure.microsoft.com/global-infrastructure/) qui gère des milliers de services en ligne et couvre plus de 100 installations de haute sécurité dans le monde entier.

L’infrastructure est conçue pour rapprocher les applications des utilisateurs dans le monde entier, en préservant la résidence des données et en offrant aux clients des options complètes de conformité et de résilience. Azure est présent dans 58 régions du monde et disponible dans 140 pays/régions.

Une région est un ensemble de centres de données qui sont interconnectés dans un réseau massif et résilient. Le réseau comprend la distribution de contenu, l’équilibrage de charge, la redondance et le [chiffrement de la couche de liaison de données par défaut](encryption-overview.md#encryption-of-data-in-transit) pour tout le trafic Azure au sein d’une région ou le déplacement entre les régions. En proposant plus de régions dans le monde que les autres fournisseurs de cloud, Azure offre aux clients la possibilité de déployer des applications là où ils en ont besoin.

Les régions Azure sont organisées en zones géographiques. Une zone géographique Azure garantit que les conditions de résidence, de souveraineté, de conformité et de résilience des données sont respectées dans les limites géographiques.

Les zones géographiques permettent aux clients ayant des besoins spécifiques en conformité et résidence des données de conserver leurs données et leurs applications à proximité. Les zones géographiques sont tolérantes aux pannes et peuvent résister à une défaillance complète de la région, car elles sont connectées à l’infrastructure réseau haute capacité dédiée.

Les zones de disponibilité sont des emplacements physiquement séparés au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un refroidissement et d’un réseau indépendants. Les zones de disponibilité vous permettent d’exécuter des applications critiques avec une haute disponibilité et une réplication de latence faible.

La figure suivante montre comment l’infrastructure mondiale d’Azure associe une région à des zones de disponibilité dans la même limite de résidence des données pour bénéficier d’une haute disponibilité, d’une reprise d’activité après sinistre et d’une sauvegarde.

![Diagramme montrant la limite de résidence des données](./media/physical-security/data-residency-boundary.png)

Des centres de données distribués géographiquement permettent à Microsoft d’être proche des clients pour réduire la latence du réseau et rendre possibles les basculements et les sauvegardes géoredondantes.

## <a name="physical-security"></a>Sécurité physique
Microsoft conçoit, crée et utilise des centres de données de manière à assurer un contrôle strict de l’accès physique aux zones où vos données sont stockées. Microsoft comprend l’importance de protéger les données de ses clients et s’engage à contribuer à la sécurisation des centres de données qui contiennent vos données. Microsoft compte une division entière dédiée à la conception, à la création et au fonctionnement des installations physiques qui gèrent Azure. Cette équipe est investie dans la conservation d’une sécurité physique à la pointe.

Microsoft adopte une approche en couche de la sécurité physique, pour réduire les risques que des utilisateurs non autorisés puissent avoir un accès physique aux données et aux ressources des centres de données. Les centres de données gérés par Microsoft présentent des couches de protection complètes : approbation de l’accès au périmètre de l’installation, au périmètre du bâtiment, à l’intérieur du bâtiment et à l’étage du centre de données. Les couches de la sécurité physique sont :

- **Demande d’accès et approbation.** Vous devez demander l’accès avant d’arriver au centre de données. Vous êtes obligé de fournir une justification professionnelle valide de votre visite, par exemple pour des raisons de conformité ou d’audit. Toutes les demandes sont approuvées au cas par cas par les employés de Microsoft. La gestion des accès au cas par cas permet de garder le nombre de personnes qui ont besoin d’effectuer une tâche dans les centres de données au strict minimum. Une fois qu’une personne obtient une autorisation de Microsoft, elle n’a accès qu’à la zone du centre de données requise, pour laquelle sa justification a été approuvée. Les autorisations sont limitées dans le temps et expirent.

- **Périmètre de l’installation.** Quand vous arrivez dans un centre de données, vous êtes obligé de passer par un point d’accès bien défini. En règle générale, des clôtures hautes en acier et en béton couvrent chaque centimètre du périmètre. Des caméras sont postées autour des centres de données et une équipe de sécurité visionne les vidéos en permanence.

- **Entrée du bâtiment.** L’entrée des centres de données est surveillée par des professionnels de la sécurité qui ont suivi une formation stricte et dont les antécédents ont été vérifiés. Ces professionnels de la sécurité patrouillent aussi régulièrement dans le centre de données tout en contrôlant les vidéos des caméras qui se trouvent à l’intérieur en permanence.

- **À l’intérieur du bâtiment.** Après être entré dans le bâtiment, vous devez passer une authentification à deux facteurs avec biométrie pour pouvoir vous déplacer dans le centre de données. Si votre identité est validée, vous pouvez entrer dans la partie du centre de données pour laquelle votre accès a été approuvé. Vous pouvez y rester uniquement pendant la durée approuvée.

- **Étage du centre de données** Vous n’êtes autorisé à vous rendre qu’à l’étage pour lequel votre accès a été approuvé. Vous êtes obligé de passer sous un portique de détection de métaux. Pour réduire le risque que des données non autorisées entrent ou sortent du centre de données sans notre connaissance, seuls les appareils approuvés sont admis à l’étage du centre de données. Par ailleurs, des caméras vidéo surveillent les deux côtés de chaque rack de serveurs. Quand vous quittez l’étage du centre de données, vous devez repasser par le portique de détection de métaux. Pour quitter le centre de données, vous devez passer par un autre scan de sécurité.

Les visiteurs doivent rendre leurs badges quand ils quittent un bâtiment Microsoft.

## <a name="physical-security-reviews"></a>Révisions de la sécurité physique
Les révisions de la sécurité physique des bâtiments sont effectuées régulièrement pour s’assurer que les centres de données répondent aux critères de sécurité d’Azure. Le personnel du fournisseur d’hébergement du centre de données ne fournit aucune gestion des services Azure. Il ne peut pas se connecter aux systèmes Azure n’a pas d’accès physique à la salle et aux cages de collocation Azure.

## <a name="data-bearing-devices"></a>Appareils contenant des données
Microsoft utilise les procédures de bonne pratique et une solution d’effacement [conforme à la norme NIST 800-88](https://csrc.nist.gov/publications/detail/sp/800-88/archive/2006-09-01). Pour les disques durs qui ne peuvent pas être effacés, une procédure de destruction est utilisée pour les détruire et rendre la récupération des informations impossible. Le processus de destruction peut être une désintégration, un déchiquetage, une pulvérisation ou une incinération. Nous déterminons les moyens d’élimination en fonction du type de ressource. Les enregistrements des destructions sont conservés.  

## <a name="equipment-disposal"></a>Élimination de l’équipement
À la fin du cycle de vie d’un système, le personnel d’exploitation de Microsoft suit les procédures rigoureuses de traitement des données et d’élimination de matériel afin de s’assurer qu’aucun matériel susceptible de contenir les données d’un client ne soit accessible à des tiers non approuvés. Nous utilisons une approche d’effacement sécurisée pour les disques durs qui la prennent en charge. Pour les disques durs qui ne peuvent pas être effacés, un processus de destruction est utilisé pour les détruire et rendre la récupération des informations impossible. Le processus de destruction peut être une désintégration, un déchiquetage, une pulvérisation ou une incinération. Nous déterminons les moyens d’élimination en fonction du type de ressource. Les enregistrements des destructions sont conservés. Tous les services Microsoft Azure utilisent des services de gestion des supports de stockage et d’élimination approuvés.

## <a name="compliance"></a>Conformité
L’infrastructure Azure est conçue et gérée pour répondre à un large éventail de normes de conformité internationales et spécifiques au secteur, telles que ISO 27001, HIPAA, FedRAMP, SOC 1 et SOC 2. Les normes propres aux pays ou régions sont également respectées, comme l’IRAP en Australie, UK G-Cloud et la MTCS à Singapour. Des audits rigoureux de tiers, comme ceux effectués par le British Standards Institute, vérifient l’adhérence d’Azure aux contrôles de sécurité stricts imposés par ces normes.

Pour avoir une liste complète des normes de conformité observées par Azure, consultez les [Offres de conformité](https://www.microsoft.com/trustcenter/compliance/complianceofferings).

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)


