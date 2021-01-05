---
title: Se connecter à Synapse Studio à l’aide de liaisons privées
description: Cet article explique comment vous connecter à votre Azure Synapse Studio à l’aide de liaisons privées.
author: NanditaV
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 12/01/2020
ms.author: NanditaV
ms.reviewer: jrasnick
ms.openlocfilehash: 2613a4fd931ad49a4f40a4221ea20e8c25f185fe
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501295"
---
# <a name="connect-to-azure-synapse-studio-using-azure-private-link-hubs"></a>Se connecter à Azure Synapse Studio à l’aide de hubs Azure Private Link 

Cet article explique comment des liaisons privées de hubs Azure Synapse Analytics Private Link sont utilisées pour la connexion sécurisée à Synapse Studio. 

## <a name="azure-private-link-hubs"></a>Hubs Azure Private Link 
Vous pouvez vous connecter en toute sécurité à Azure Synapse Studio à partir de votre réseau virtuel Azure à l’aide de liaisons privées. Les hubs de liaison privée Azure Synapse Analytics sont des ressources Azure qui jouent le rôle de connecteurs entre votre réseau sécurisé et l’expérience web de Synapse Studio. 

Il y a deux étapes pour vous connecter à Synapse Studio à l’aide de liaisons privées. Tout d’abord, vous devez créer une ressource de hubs de liaisons privées. Deuxièmement, vous devez créer un point de terminaison privé à partir de votre réseau virtuel Azure vers ce hub de liaisons privées. Vous pouvez ensuite utiliser des points de terminaison privés pour communiquer en toute sécurité avec Synapse Studio. Vous devez intégrer les points de terminaison privés à votre solution DNS, soit votre solution sur site, soit DNS privé Azure. 

## <a name="azure-private-links-hubs-and-azure-synapse-studio"></a>Hubs de liaisons privées Azure et Azure Synapse Studio
Vous pouvez utiliser une seule ressource de hub de liaison privée Azure Synapse pour vous connecter en privé à tous vos espaces de travail Azure Synapse Analytics à l’aide d’Azure Synapse Studio. Il n’est pas nécessaire que les espaces de travail se trouvent dans la même région que le hub de liaison privée Azure Synapse. La ressource de hub de liaison privée Azure Synapse peut également être utilisée pour les connexions à des espaces de travail Synapse dans des abonnements ou des locataires Azure AD différents.

Vous pouvez créer votre hub de liaison privée en recherchant des *hubs de liaisons privées Synapse* dans le portail Azure et en sélectionnant **Azure Synapse Analytics (hubs de liaisons privées)** dans Services. Pour plus d’informations, suivez les étapes décrites dans le guide sur la façon de [se connecter aux ressources de l’espace de travail à partir d’un réseau limité](./how-to-connect-to-workspace-from-restricted-network.md).

>[!NOTE]
>Les hubs de liaisons privées sont conçus pour charger en toute sécurité le contenu statique Synapse Studio sur des liaisons privées. Vous devez créer des **points de terminaison privés distincts** aux ressources auxquelles vous souhaitez vous connecter dans l’espace de travail, comme des pools SQL provisionnés/dédiés ou des pools Spark. 

## <a name="azure-private-links-hubs-and-azure-virtual-network"></a>Hubs de liaisons privées Azure et Réseau virtuel Azure
Vous devez connecter votre réseau virtuel Azure à la ressource de hub de liaison privée Synapse pour sécuriser la connexion de bout en bout à Synapse Studio. Pour ce faire, vous devez créer un point de terminaison privé à partir de votre réseau virtuel vers le hub de liaison privée que vous avez créé. Vous pouvez utiliser le portail Azure pour votre hub de liaison privée et accéder à la section Point de terminaison privé. Sélectionnez « + Point de terminaison privé » pour créer un nouveau point de terminaison privé qui se connecte à votre hub de liaison privée.

:::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-private-endpoint.png" alt-text="Créer un point de terminaison privé au hub de liaison privée":::

Veillez à choisir le type de ressource « Microsoft.Synapse/privateLinkHubs » sous l’onglet « Ressource ». :::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-resource-type.png" alt-text="Créer un point de terminaison privé au hub de liaison privée":::

Dans l’onglet « Configuration », sélectionnez « privatelink.azuresynapse.net » pour Zones DNS privé lors de l’intégration avec votre réseau virtuel et votre zone DNS privée.

:::image type="content" source="./media/synapse-private-link-hubs/synapse-private-links-dns-zones.png" alt-text="Créer un point de terminaison privé au hub de liaison privée":::

## <a name="next-steps"></a>Étapes suivantes

[Se connecter aux ressources de l’espace de travail à partir d’un réseau restreint](./how-to-connect-to-workspace-from-restricted-network.md)

En savoir plus sur le [Réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md)

En savoir plus sur les [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md)

[Créer des points de terminaison privés managés à vos sources de données](./how-to-create-managed-private-endpoints.md)

[Se connecter à un espace de travail Synapse à l’aide de points de terminaison privés](./how-to-connect-to-workspace-with-private-links.md)

