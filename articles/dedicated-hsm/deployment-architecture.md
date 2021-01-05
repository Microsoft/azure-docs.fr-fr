---
title: Architecture de déploiement - Module HSM dédié Azure | Microsoft Docs
description: Considérations de base relatives à la conception lors de l’utilisation d’un module HSM dédié Azure en tant que partie d’une architecture d’application
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 6a0767b077886337331f24b15715247006f3fe2c
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888893"
---
# <a name="azure-dedicated-hsm-deployment-architecture"></a>Architecture de déploiement Azure Dedicated HSM

Un HSM dédié Azure permet le stockage de clés de chiffrement dans Azure. Il satisfait à des exigences de sécurité strictes. Les clients tireront un bénéfice de l’utilisation du HSM dédié Azure s’ils :

* doivent satisfaire à la certification FIPS 140-2 niveau 3 ;
* ont besoin d’un accès exclusif au HSM ;
* doivent contrôler complètement leurs appareils.

Les HSM sont répartis entre les centres de données de Microsoft et peuvent être facilement approvisionnés en tant que paire d’appareils comme base d’une solution hautement disponible. Ils peuvent également être déployés dans différentes régions pour obtenir une solution résiliente d’urgence. Pour savoir dans quelles régions Dedicated HSM est actuellement disponible, consultez la [page Produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=azure-dedicated-hsm). 

Chacune de ces régions dispose de racks HSM déployés dans deux centres de données indépendants ou dans au moins deux zones de disponibilité indépendantes. Par exemple, la région Asie Sud-Est compte trois zones de disponibilité tandis que la région USA Est 2 en compte deux. Au total, huit régions d'Europe, d'Asie et des États-Unis proposent le service Dedicated HSM, et ce nombre évolue à mesure que nous ajoutons de nouveaux racks HSM dans de nouvelles régions. Pour plus d’informations sur les régions Azure, consultez les [informations sur les régions Azure](https://azure.microsoft.com/global-infrastructure/regions/).
L’emplacement/la latence, une haute disponibilité et la prise en charge d’autres applications distribuées font partie des facteurs de conception pour une solution HSM dédié.

## <a name="device-location"></a>Emplacement de l’appareil

L’emplacement optimal de l’appareil HSM est le plus près possible des applications effectuant des opérations de chiffrement. La latence dans la région est censée être en millisecondes à un chiffre. La latence entre les régions peut être 5 à 10 fois supérieure.

## <a name="high-availability"></a>Haute disponibilité

Pour profiter de la haute disponibilité, un client doit utiliser deux appareils HSM dans une région, ceux-ci étant configurés comme une paire à haute disponibilité à l'aide du logiciel Thanles. Ce type de déploiement garantit la disponibilité des clés si un seul appareil rencontre un problème empêchant le traitement des opérations de clé. Il réduit également considérablement les risques lors d’une maintenance de réparation/de correction comme le remplacement de l’alimentation électrique. Lors d’une conception, il est important de tenir compte de tous les types de défaillances au niveau régional. Des défaillances au niveau régional peuvent survenir en cas de catastrophes naturelles telles que des ouragans, des inondations ou des tremblements de terre. Ces types d’événements doivent être atténués en approvisionnant les appareils HSM dans une autre région. Les appareils déployés dans une autre région peuvent être couplés via la configuration du logiciel Thales. Ceci signifie que le déploiement minimal pour une solution à haute disponibilité et résiliente après sinistre est de quatre appareils HSM dans deux régions. La redondance locale et la redondance entre les régions peuvent être utilisées comme ligne de base pour ajouter d’autres déploiements d’appareils HSM afin de prendre en charge la latence, la capacité, ou bien pour satisfaire à d’autres exigences spécifiques à l’application.

## <a name="distributed-application-support"></a>Prise en charge des applications distribuées

Les appareils HSM dédié sont généralement déployés pour prendre en charge des applications devant effectuer des opérations de récupération et de stockage de clés. Les appareils HSM dédié ont 10 partitions pour la prise en charge indépendante des applications. L’emplacement de l’appareil doit être basé sur une vue holistique de toutes les applications qui doivent utiliser le service.

## <a name="next-steps"></a>Étapes suivantes

Une fois l'architecture de déploiement déterminée, la plupart des activités de configuration permettant d'implémenter cette architecture seront fournies par Thales. Ceci inclut la configuration de l’appareil ainsi que les scénarios d’intégration des applications. Pour plus d'informations, utilisez le portail de [support client de Thales](https://supportportal.gemalto.com/csm/) et téléchargez les guides d'administration et de configuration. Le site du partenaire Microsoft met à votre disposition tout un éventail de guides d’intégration.
Il est recommandé de bien comprendre tous les concepts clés du service, comme la haute disponibilité et la sécurité, par exemple, avant le provisionnement des appareils ou la conception ou bien le déploiement des applications.
Quelques rubriques conceptuelles supplémentaires :

* [Haute disponibilité](high-availability.md)
* [Sécurité physique](physical-security.md)
* [Mise en réseau](networking.md)
* [Prise en charge](supportability.md)
* [Surveillance](monitoring.md)
