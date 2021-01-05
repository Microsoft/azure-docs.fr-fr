---
title: Définir des alertes pour les mesures Azure Event Grid et les opérations du journal d’activité
description: Cet article explique comment créer des alertes sur les mesures Azure Event Grid et les opérations du journal d’activité.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 615ebef749be49822a09470eb0d47d8eb31b9a5a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86119138"
---
# <a name="set-alerts-on-azure-event-grid-metrics-and-activity-logs"></a>Définir des alertes sur les mesures Azure Event Grid et les journaux d’activité
Cet article explique comment créer des alertes sur les mesures Azure Event Grid et les opérations du journal d’activité. Vous pouvez créer des alertes sur les mesures de publication et de livraison pour les ressources Azure Event Grid (rubriques et domaines). Pour les rubriques du système, [créez des alertes à l’aide de la page **Mesures**](#create-alerts-using-the-metrics-page).

## <a name="create-alerts-on-dead-lettered-events"></a>Créer des alertes sur les événements de lettres mortes
La procédure suivante vous montre comment créer une alerte sur une métrique **événements de lettres mortes** pour une rubrique personnalisée. Dans cet exemple, un e-mail est envoyé au propriétaire du groupe de ressources Azure lorsque le nombre d’événements de lettres mortes pour une rubrique dépasse 10. 

1. Sur la page **Rubrique Event Grid** de votre rubrique, sélectionnez **Alertes** dans le menu de gauche, puis sélectionnez **+Nouvelle règle d’alerte**. 

    :::image type="content" source="./media/monitor-event-delivery/new-alert-button.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::
2. Dans la page **Créer une règle d’alerte**, vérifiez que votre rubrique est sélectionnée pour la ressource. Puis cliquez sur **Sélectionner une condition**. 

    :::image type="content" source="./media/monitor-event-delivery/alert-select-condition.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::    
3. Sur la page **Configurer la logique du signal**, procédez comme suit :
    1. Sélectionnez une métrique ou une entrée de journal d’activité. Dans cet exemple, **Événements de lettres mortes** est sélectionné. 

        :::image type="content" source="./media/monitor-event-delivery/select-dead-lettered-events.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::        
    2. Sélectionnez les dimensions (facultatif). 
        
        :::image type="content" source="./media/monitor-event-delivery/configure-signal-logic.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::        

        > [!NOTE]
        > Vous pouvez sélectionner le bouton **+** pour **EventSubscriptionName** pour spécifier un nom d’abonnement aux événements pour filtrer les événements. 
    3. Faites défiler vers le bas. Dans la section **Logique d’alerte**, sélectionnez un **Opérateur**, un **Type d’agrégation**, entrez une **Valeur de seuil**, puis sélectionnez **Terminé**. Dans cet exemple, une alerte est déclenchée lorsque le nombre total d’événements de lettres mortes est supérieur à 10. 
    
        :::image type="content" source="./media/monitor-event-delivery/alert-logic.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::                
4. De retour dans la page **Créer une règle d’alerte**, cliquez sur **Sélectionner un groupe d’actions**.

    :::image type="content" source="./media/monitor-event-delivery/select-action-group-button.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::
5. Sélectionnez **Créer un groupe d’actions** dans la barre d’outils pour créer un nouveau groupe d’actions. Vous pouvez également sélectionner un groupe d'actions existant.        
6. Dans la page **Ajouter un groupe d’actions**, effectuez les étapes suivantes :
    1. Entrez le **nom du groupe d’actions**.
    1. Entrez le **nom court** du groupe d’actions.
    1. Sélectionnez un **abonnement Azure** où créer le groupe d'actions désiré.
    1. Sélectionnez le **groupe de ressources Azure** dans lequel vous souhaitez créer le groupe d'actions.
    1. Entrez le **nom de l’action**. 
    1. Sélectionner le **type d’action**. Dans cet exemple, **Rôle de l’e-mail Azure Resource Manager** est sélectionné, en particulier le rôle **Propriétaires**. 
    1. Cliquez sur **OK** pour fermer la page. 
    
        :::image type="content" source="./media/monitor-event-delivery/add-action-group-page.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::                   
7. De retour dans la page **Créer une règle d’alerte**, entrez un nom pour la règle d'alerte, puis sélectionnez **Créer une règle d’alerte**.

    :::image type="content" source="./media/monitor-event-delivery/alert-rule-name.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::  
8. Maintenant, dans la page **Alertes** de la rubrique, vous voyez un lien pour gérer les règles d’alerte si aucune alerte n’est encore présente. S’il existe des alertes, sélectionnez **Règles d’alerte manager** dans la barre d’outils.  

    :::image type="content" source="./media/monitor-event-delivery/manage-alert-rules.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::

## <a name="create-alerts-on-other-metrics-or-activity-log-operations"></a>Créer des alertes sur d’autres mesures ou opérations de journal d’activité
La section précédente vous a montré comment créer des alertes sur les événements de lettres mortes. Ces étapes de création des alertes sur d’autres mesures ou opérations de journal d’activité sont identiques. 

Par exemple, pour créer une alerte sur un événement d’échec de livraison, sélectionnez **Événements d’échec de livraison** dans la page **Configurer la logique de signal**. 

:::image type="content" source="./media/set-alerts/delivery-failed-events.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::


## <a name="create-alerts-using-the-metrics-page"></a>Créer des alertes à l’aide de la page Mesures
Vous pouvez également créer des alertes à l’aide de la page **Mesures**. Les étapes sont identiques. Pour les rubriques système, vous pouvez utiliser uniquement la page **Mesures** pour créer des alertes, car la page **Alertes** n’est pas disponible. 

:::image type="content" source="./media/monitor-event-delivery/metric-page-create-alert-button.png" alt-text="Page Alertes : bouton Nouvelle règle d’alerte":::   
    

> [!NOTE]
> Cet article ne couvre pas toutes les étapes et combinaisons que vous pouvez utiliser pour créer une alerte. Pour obtenir une vue d’ensemble des alertes, consultez [Vue d’ensemble des alertes](../azure-monitor/platform/alerts-metric.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
