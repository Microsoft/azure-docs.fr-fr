---
title: Présentation des services de données avec Azure Arc
description: Présente les services de données avec Azure Arc
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 09/22/2020
ms.topic: overview
ms.openlocfilehash: ef16a4c0b6ea40fb4934307916f1fe79ccea72f9
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609137"
---
# <a name="what-are-azure-arc-enabled-data-services-preview"></a>Présentation des services de données avec Azure Arc (préversion)

Azure Arc vous permet d'exécuter des services de données Azure localement, à la périphérie et dans des clouds publics à l'aide de Kubernetes et de l'infrastructure de votre choix.

Actuellement, les services de données avec Azure Arc suivants sont disponibles en préversion :

- Instance managée SQL
- PostgreSQL Hyperscale

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="always-current"></a>Toujours à jour

Les services de données avec Azure Arc, tels que SQL Managed Instance avec Azure Arc et PostgreSQL Hyperscale avec Azure Arc, bénéficient de mises à jour fréquentes contenant notamment des correctifs de maintenance et de nouvelles fonctionnalités semblables à l'expérience offerte par Azure. Des mises à jour de Microsoft Container Registry vous sont fournies et vous pouvez définir les cadences de déploiement conformément à vos stratégies. Ainsi, les bases de données locales restent à jour tandis que vous gardez le contrôle. Dans la mesure où les services de données avec Azure Arc sont fournis sur abonnement, vos bases de données ne sont jamais confrontées à aucun problème de fin de support.

## <a name="elastic-scale"></a>Mise à l’échelle élastique

L'élasticité de type cloud disponible localement vous permet d'effectuer dynamiquement un scale-up ou un scale-down de vos bases de données en fonction de la capacité de votre infrastructure, de la même manière que dans Azure. Cette fonctionnalité peut convenir aux scénarios d'augmentation du trafic qui se caractérisent par des besoins volatils, notamment aux scénarios nécessitant l'ingestion et l'interrogation de données en temps réel, quelle que soit l'échelle, avec un temps de réponse inférieur à la seconde. En outre, vous pouvez effectuer un scale-out des instances de base de données à l'aide de l'option de déploiement hyperscale unique d'Azure Database pour PostgreSQL Hyperscale. Cette fonctionnalité permet aux charges de travail de données de bénéficier d'une optimisation supplémentaire en termes de capacité grâce à des lectures et à des écritures de scale-*out* uniques.

## <a name="self-service-provisioning"></a>Approvisionnement en libre-service

Azure Arc offre également d'autres avantages liés au cloud, tels qu'un déploiement rapide et une automatisation à grande échelle. Grâce à l'orchestration basée sur Kubernetes, vous pouvez déployer une base de données en quelques secondes via l'interface graphique utilisateur (GUI) ou l'interface de ligne de commande (CLI).

## <a name="unified-management"></a>Gestion unifiée

À l’aide d’outils que vous connaissez déjà, comme le portail Azure, d’Azure Data Studio et de l’[!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)], vous pouvez désormais profiter d’une vue unifiée sur toutes les ressources de données déployées avec Azure Arc. Vous pouvez non seulement visualiser et gérer différentes bases de données relationnelles au sein de votre environnement et d'Azure, mais également bénéficier de journaux et de données de télémétrie à partir d'API Kubernetes pour analyser la capacité et l'intégrité de l'infrastructure sous-jacente. Outre les fonctionnalités localisées d'analytique des journaux d'activité et d'analyse des performances, vous pouvez désormais tirer parti d'Azure Monitor pour obtenir des insights opérationnels complets sur l'ensemble de votre patrimoine.

## <a name="disconnected-scenario-support"></a>Prise en charge de scénarios déconnectés

De nombreux services tels que l'approvisionnement en libre-service, les sauvegardes/restaurations automatisées et la supervision peuvent être exécutés localement dans votre infrastructure, avec ou sans connexion directe à Azure. La connexion directe à Azure ouvre la voie à des options supplémentaires d'intégration à d'autres services Azure tels qu'Azure Monitor, et la possibilité d'utiliser le portail Azure et les API Azure Resource Manager n'importe où dans le monde pour gérer vos services de données avec Azure Arc.

## <a name="next-steps"></a>Étapes suivantes

> **Vous voulez juste essayer ?**  
> Démarrez rapidement avec [Démarrage rapide d’Azure Arc](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) sur Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) ou sur une machine virtuelle Azure.

[Installer les outils clients](install-client-tools.md)

[Créer le contrôleur de données Azure Arc](create-data-controller.md) (nécessite l’installation préalable des outils clients)

[Créer une instance gérée Azure SQL sur Azure Arc](create-sql-managed-instance.md) (nécessite la création préalable d’un contrôleur de données Azure Arc)

[Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc](create-postgresql-hyperscale-server-group.md) (nécessite la création préalable d’un contrôleur de données Azure Arc)
