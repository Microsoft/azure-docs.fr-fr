---
title: Points de terminaison privés managés
description: Article qui explique ce que sont les points de terminaison privés managés dans Azure Synapse Analytics.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: security
ms.date: 10/16/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 5258b1eab48f71d8d17f52849b5e57b467e7a2da
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460390"
---
# <a name="synapse-managed-private-endpoints"></a>Points de terminaison privés managés Synapse

Cet article explique ce que sont les points de terminaison privés managés dans Azure Synapse Analytics.

## <a name="managed-private-endpoints"></a>Points de terminaison privés managés

Les points de terminaison privés managés sont des points de terminaison privés créés sur le Réseau virtuel Microsoft Azure d’espace de travail managé qui établissent une liaison privée vers des ressources Azure. Azure Synapse gère ces points de terminaison privés à votre place.

Azure Synapse prend en charge les liaisons privées. Une liaison privée vous permet d’accéder à des services Azure (tels que Stockage Azure et Azure Cosmos DB) et à des services clients/partenaires hébergés par Azure à partir de votre Réseau virtuel Azure de manière sécurisée.

Quand vous utilisez une liaison privée, le trafic entre votre Réseau virtuel et l’espace de travail transite intégralement par le réseau principal de Microsoft. Une liaison privée assure une protection contre les risques liés à l’exfiltration des données. Vous établissez une liaison privée vers une ressource en créant un point de terminaison privé.

Le point de terminaison privé utilise une adresse IP privée de votre Réseau virtuel pour placer de fait le service dans votre Réseau virtuel. Les points de terminaison privés sont mappés à une ressource spécifique dans Azure, et non à l’ensemble du service. Les clients peuvent limiter la connectivité à une ressource spécifique approuvée par leur organisation. 

Apprenez-en davantage sur [les liaisons privées et les points de terminaison privés](https://docs.microsoft.com/azure/private-link/).

>[!IMPORTANT]
>Les points de terminaison privés managés sont uniquement pris en charge dans les espaces de travail Azure Synapse avec un Réseau virtuel d’espace de travail managé.

>[!NOTE]
>Tout le trafic sortant en provenance du Réseau virtuel d’espace de travail managé via des points de terminaison privés sera bloqué à l’avenir. Nous vous recommandons de créer des points de terminaison privés managés pour vous connecter à toutes vos sources de données Azure externes à l’espace de travail. 

Une connexion de point de terminaison privé est créée dans un état « en attente » quand vous créez un point de terminaison privé managé dans Azure Synapse. Un workflow d’approbation est démarré. Le propriétaire de la ressource de liaison privée est responsable de l’approbation ou du refus de la connexion.

Si le propriétaire approuve la connexion, la liaison privée est établie. Toutefois, si le propriétaire n’approuve pas la connexion, le lien privé n’est pas établi. Dans les deux cas, le point de terminaison privé managé est mis à jour avec l’état de la connexion.

Seule une instance de point de terminaison privé managé dans un état approuvé peut envoyer du trafic vers une ressource de liaison privée donnée.

## <a name="managed-private-endpoints-for-dedicated-sql-pool-and-serverless-sql-pool"></a>Points de terminaison privés managés pour un pool SQL dédié et un pool SQL serverless

Le pool SQL dédié et le pool SQL serverless sont des fonctionnalités d’analytique dans votre espace de travail Azure Synapse. Ces fonctionnalités utilisent une infrastructure multilocataire qui n’est pas déployée sur le [Réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md).

Quand un espace de travail est créé, Azure Synapse crée deux points de terminaison privés managés dans l’espace de travail, un pour le pool SQL dédié et un pour le pool SQL serverless. 

Ces deux points de terminaison privés managés sont listés dans Synapse Studio. Sélectionnez **Gérer** dans la zone de navigation de gauche, puis sélectionnez **Points de terminaison privés managés** pour les afficher dans Studio.

Le point de terminaison privé managé qui cible le pool SQL est appelé *synapse-ws-sql--\<workspacename\>* et celui qui cible le pool SQL serverless est appelé *synapse-ws-sqlOnDemand--\<workspacename\>* .

![Points de terminaison privés managés pour un pool SQL dédié et un pool SQL serverless](./media/synapse-workspace-managed-private-endpoints/managed-pe-for-sql-1.png)

Ces deux points de terminaison privés managés sont créés automatiquement pour vous quand vous créez votre espace de travail Azure Synapse. Vous n’êtes pas facturé pour ces deux points de terminaison privés managés.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, passez à l’article [Créer des points de terminaison privés managés dans vos sources de données](./how-to-create-managed-private-endpoints.md).
