---
title: Abonnements Azure Event Grid via le portail
description: Cet article explique comment créer des abonnements Event Grid pour les sources prises en charge, telles que Stockage Blob Azure, à l’aide du Portail Azure.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: e80e2243c93ab38187646256f567d6ab73c40100
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995061"
---
# <a name="subscribe-to-events-through-portal"></a>S’abonner à des événements via le portail

Cet article décrit comment créer des abonnements Event Grid via le portail.

## <a name="create-event-subscriptions"></a>Créer des abonnements aux événements

Pour créer un abonnement Event Grid pour une des [sources d’événements](overview.md#event-sources) prises en charge, procédez selon les étapes suivantes. Cet article explique comment créer un abonnement Event Grid pour un abonnement Azure.

1. Sélectionnez **Tous les services**.

   ![Sélectionner Tous les services](./media/subscribe-through-portal/select-all-services.png)

1. Recherchez **Abonnements Event Grid**  et sélectionnez cette option parmi celles disponibles.

   ![La capture d’écran montre la recherche dans le portail Azure avec les abonnements Event Grid sélectionnés.](./media/subscribe-through-portal/search.png)

1. Sélectionnez **+ Abonnement aux événements**.

   ![Ajouter un abonnement](./media/subscribe-through-portal/add-subscription.png)

1. Sélectionnez le type d’abonnement que vous voulez créer. Par exemple, pour vous abonner à des événements pour votre abonnement Azure, sélectionnez **Abonnements Azure** et l’abonnement cible.

   ![Sélectionner un abonnement Azure](./media/subscribe-through-portal/azure-subscription.png)

1. Pour vous abonner à tous les types d’événements pour cette source d’événements, conservez l’option **S’abonner à tous les types d’événements** activée. Sinon, sélectionnez les types d’événements pour cet abonnement.

   ![Sélectionner des types d’événements](./media/subscribe-through-portal/select-event-types.png)

1. Spécifiez des détails supplémentaires sur l’abonnement aux événements, comme le point de terminaison pour la gestion des événements et un nom d’abonnement.

   ![Capture d’écran montrant les sections « Détails du point de terminaison » et « Détails de l’abonnement aux événements » avec une valeur de nom d’abonnement entrée.](./media/subscribe-through-portal/provide-subscription-details.png)

1. Pour activer le lettrage mort et personnaliser les stratégies de nouvelle tentative, sélectionnez **Fonctionnalités supplémentaires**.

   ![Sélectionner des fonctionnalités supplémentaires](./media/subscribe-through-portal/select-additional-features.png)

1. Sélectionnez un conteneur à utiliser pour stocker les événements qui ne sont pas livrés et définissez le mode d’envoi des nouvelles tentatives.

   ![Activer les stratégies de lettres mortes et de nouvelles tentatives](./media/subscribe-through-portal/set-deadletter-retry.png)

1. Quand vous avez terminé, sélectionnez **Créer**.

## <a name="create-subscription-on-resource"></a>Créer un abonnement sur une ressource

Certaines sources d’événements prennent en charge la création d’un abonnement aux événements via l’interface du portail pour cette ressource. Sélectionnez la source d’événements et recherchez **Événements** dans le volet gauche.

![Spécifier les détails de l’abonnement](./media/subscribe-through-portal/resource-events.png)

Le portail présente des options pour la création d’un abonnement aux événements qui se rapportent à cette source.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
