---
title: Afficher des métriques Azure Event Grid et définir des alertes
description: Cet article explique comment utiliser le portail Azure pour afficher des métriques des rubriques et abonnements de Azure Event Grid, et créer des alertes sur ces éléments.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 518d34d39e6fbecc408fe9a44d899fe4745d60d0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008954"
---
# <a name="monitor-event-grid-message-delivery"></a>Surveiller la remise des messages Event Grid 
Cet article explique comment utiliser le portail pour afficher des métriques des rubriques et abonnements Event Grid et créer des alertes sur ces éléments. 

## <a name="metrics"></a>Mesures

Le portail affiche les métriques pour l’état de la remise des messages d’événement.

Voici quelques-unes des mesures disponibles pour les rubriques :

* **Publication réussie** : événement correctement envoyé à la rubrique et traité avec une réponse 2xx.
* **Échec de la publication** : événement envoyé à la rubrique mais rejeté avec un code d’erreur.
* **Sans correspondance** : événement correctement publié dans la rubrique, mais sans correspondance avec un abonnement aux événements. L’événement a été supprimé.

Pour les abonnements, voici quelques-unes des mesures :

* **Remise réussie** : l’événement a été correctement remis au point de terminaison de l’abonnement et a reçu une réponse 2xx.
* **Échec de la remise** : chaque fois que le service tente d’effectuer une remise et que le gestionnaire d’événements ne retourne pas de code 2xx de réussite, le compteur **Échec de la remise** est incrémenté. Si le service tente de remettre le même événement plusieurs fois et échoue, le compteur **Échec de la remise** est incrémenté à chaque nouvel échec.
* **Événements expirés** : l’événement n’a pas été remis et toutes les nouvelles tentatives ont été envoyées. L’événement a été supprimé.
* **Événements correspondants** : l’événement dans la rubrique a été mis en correspondance par l’abonnement aux événements.

    > [!NOTE]
    > Pour obtenir la liste complète des métriques, consultez [Métriques prises en charge par Azure Event Grid](metrics.md).

## <a name="view-custom-topic-metrics"></a>Afficher des métriques d’une rubrique personnalisée

Si vous avez publié une rubrique personnalisée, vous pouvez afficher ses métriques. 

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la barre de recherche de la rubrique, tapez **Rubriques Event Grid**, puis sélectionnez **Rubriques Event Grid** dans la liste déroulante. 

    Rechercher et sélectionner :::image type="content" source="./media/custom-event-quickstart-portal/select-event-grid-topics.png" alt-text="Rubriques Event Grid":::
3. Sélectionnez votre rubrique personnalisée dans la liste des rubriques. 

    :::image type="content" source="./media/monitor-event-delivery/select-custom-topic.png" alt-text="Sélectionnez votre rubrique personnalisée":::
4. Affichez les métriques de la rubrique d’événement personnalisé sur la page **Rubrique Event Grid**. Dans l’image suivante, la section **Essentials** qui affiche le groupe de ressources, l’abonnement, etc. est réduite. 

    :::image type="content" source="./media/monitor-event-delivery/custom-topic-metrics.png" alt-text="Afficher les métriques d’événement":::

Vous pouvez créer des graphiques avec des métriques prises en charge à l’aide de l’onglet **Métriques** de la page **Rubrique Event Grid**.

:::image type="content" source="./media/monitor-event-delivery/topics-metrics-page.png" alt-text="Rubrique - Page des métriques":::

Pour plus d’informations sur les métriques, consultez [Métriques dans Azure Monitor](../azure-monitor/platform/data-platform-metrics.md)

Par exemple, consultez le graphique des métriques pour la métrique **Événements publiés**.

:::image type="content" source="./media/monitor-event-delivery/custom-topic-metrics-example.png" alt-text="Métrique des événements publiés":::


## <a name="view-subscription-metrics"></a>Afficher les métriques des abonnements
1. Accédez à la page **Rubrique Event Grid** en suivant les étapes de la section précédente. 
2. Sélectionnez l’abonnement dans le volet inférieur, comme indiqué dans l’exemple suivant. 

    :::image type="content" source="./media/monitor-event-delivery/select-event-subscription.png" alt-text="Sélectionner l’abonnement aux événements":::    

    Vous pouvez également rechercher des **abonnements Event Grid** dans la barre de recherche du portail Azure, sélectionnez **Type de rubrique**, **Abonnement** et **Emplacement** pour voir un abonnement aux événements. 

    :::image type="content" source="./media/monitor-event-delivery/event-subscriptions-page.png" alt-text="Sélectionner l’abonnement aux événements à partir de la page Abonnements Event Grid":::        

    Pour les rubriques personnalisées, sélectionnez **Rubriques Event Grid** comme **Type de rubrique**. Pour les rubriques système, sélectionnez le type de la ressource Azure, par exemple **Comptes de stockage (blob, GPv2)** . 
3. Consultez les métriques d’un abonnement sur sa page d’accueil sous forme de graphique. Vous pouvez voir les métriques **Général**, **Erreur**, **Latence** et **Lettres mortes** pour la dernière heure, les 6 dernières heures, les 12 dernières heures, une journée, 7 jours ou 30 jours. 

    :::image type="content" source="./media/monitor-event-delivery/subscription-home-page-metrics.png" alt-text="Métriques sur la page d’accueil de l’abonnement":::    

## <a name="view-system-topic-metrics"></a>Afficher les métriques de rubrique système

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la barre de recherche de la rubrique, tapez **Rubriques système Event Grid**, puis sélectionnez **Rubriques système Event Grid** dans la liste déroulante. 

    :::image type="content" source="./media/monitor-event-delivery/search-system-topics.png" alt-text="Rechercher et sélectionner des rubriques système Event Grid":::
3. Sélectionnez votre rubrique système dans la liste des rubriques. 

    :::image type="content" source="./media/monitor-event-delivery/select-system-topic.png" alt-text="Sélectionner votre rubrique système":::
4. Affichez les métriques de la rubrique système sur la page **Rubrique système Event Grid**. Dans l’image suivante, la section **Essentials** qui affiche le groupe de ressources, l’abonnement, etc. est réduite. 

    :::image type="content" source="./media/monitor-event-delivery/system-topic-overview-metrics.png" alt-text="Afficher les métriques de rubrique système sur la page de présentation":::

Vous pouvez créer des graphiques avec des métriques prises en charge à l’aide de l’onglet **Métriques** de la page **Rubrique Event Grid**.

:::image type="content" source="./media/monitor-event-delivery/system-topic-metrics-page.png" alt-text="Rubrique système - Page des métriques":::

Pour plus d’informations sur les métriques, consultez [Métriques dans Azure Monitor](../azure-monitor/platform/data-platform-metrics.md)


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- Pour apprendre à créer des alertes sur des métriques et des opérations du journal d’activité, consultez [Définir des alertes](set-alerts.md).
- Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
