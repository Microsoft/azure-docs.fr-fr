---
title: Qu’est-ce qu’Azure Arc enabled PostgreSQL Hyperscale ?
description: Qu’est-ce qu’Azure Arc enabled PostgreSQL Hyperscale ?
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 7e8746d9b29b0b4af30ba799628328acc3fd8bc0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930173"
---
# <a name="what-is-azure-arc-enabled-postgresql-hyperscale"></a>Qu’est-ce qu’Azure Arc enabled PostgreSQL Hyperscale ?

Azure Arc enabled PostgreSQL Hyperscale est un des services de base de données disponibles dans le cadre des services de données activés pour Azure Arc. Azure Arc vous permet d’exécuter des services de données Azure localement, à la périphérie et dans des clouds publics en utilisant Kubernetes et l’infrastructure de votre choix. La proposition de valeur des services de données activés pour Azure Arc s’articule autour des éléments suivants :
- Toujours à jour
- Mise à l’échelle élastique
- Provisionnement en libre-service
- Gestion unifiée
- Prise en charge de scénarios déconnectés

Vous pouvez approfondir le sujet en lisant :
- [Présentation des services de données activés pour Azure Arc](overview.md)
- [Modes et spécifications de la connectivité](connectivity.md)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="compare-solutions"></a>Comparer les solutions

Cette section décrit en quoi Azure Arc enabled PostgreSQL Hyperscale diffère d’Azure Database pour PostgreSQL Hyperscale (Citus) ?

## <a name="azure-database-for-postgresql-hyperscale-citus"></a>Hyperscale (Citus) - Azure Database pour PostgreSQL

:::image type="content" source="media/postgres-hyperscale/postgresql-hyperscale.png" alt-text="Azure SQL Database pour PostgreSQL Hyperscale (Citus)":::

Il s’agit du facteur de forme hyperscale du moteur de base de données Postgres disponible en tant que service dans Azure (PaaS). Il s’appuie sur l’extension Citus qui permet l’expérience hyperscale. Dans ce facteur de forme, le service s’exécute dans les centres de données Microsoft et il est exploité par Microsoft.

## <a name="azure-arc-enabled-postgresql-hyperscale"></a>Azure Arc enabled PostgreSQL Hyperscale

:::image type="content" source="media/postgres-hyperscale/postgresql-hyperscale-arc.png" alt-text="Azure SQL Database pour PostgreSQL Hyperscale (Citus)":::

Il s’agit du facteur de forme hyperscale du moteur de base de données Postgres disponible avec les services de données activés pour Azure Arc. Il s’appuie également sur l’extension Citus qui permet l’expérience hyperscale. Dans ce facteur de forme, nos clients fournissent l’infrastructure qui héberge les systèmes et les exploitent.

## <a name="next-steps"></a>Étapes suivantes
- **Créer**
   > **Vous voulez juste l’essayer ?**  
   > Démarrez rapidement avec [Démarrage rapide d'Azure Arc](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) sur Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) ou sur une machine virtuelle Azure.

   - **Créer :**
      - [Installer les outils clients](install-client-tools.md)
      - [Créer le contrôleur de données Azure Arc](create-data-controller.md) (nécessite l'installation préalable des outils clients)
      - [Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc](create-postgresql-hyperscale-server-group.md) (nécessite la création préalable d'un contrôleur de données Azure Arc)
- [**Découvrir plus d’informations sur les services de données activés pour Azure Arc**](https://azure.microsoft.com/services/azure-arc/hybrid-data-services)
- [**Découvrir plus d’informations sur Azure Arc**](https://aka.ms/azurearc)
