---
title: Créer des alertes de journal d’activité pour les labos dans Azure DevTest Labs
description: Cet article fournit la procédure de création d’alertes de journal d’activité pour un labo dans Azure DevTest Labs.
ms.topic: how-to
ms.date: 07/10/2020
ms.openlocfilehash: d5886ea26ddbeb07efc23d61d3197860620eebf3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90526355"
---
# <a name="create-activity-log-alerts-for-labs-in-azure-devtest-labs"></a>Créer des alertes de journal d’activité pour les labos dans Azure DevTest Labs
Cet article explique comment créer des alertes de journal d’activité pour les labos dans Azure DevTest Labs (par exemple, quand une machine virtuelle est créée ou supprimée).

## <a name="create-alerts"></a>Créez des alertes
Dans cet exemple, vous créez une alerte pour toutes les opérations d’administration sur un labo avec une action qui envoie un e-mail aux propriétaires d’abonnements. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre de recherche du portail Azure, tapez **Superviser**, puis sélectionnez **Superviser** dans la liste des résultats. 

    :::image type="content" source="./media/activity-logs/search-monitor.png" alt-text="Rechercher Superviser":::        
1. Sélectionnez **Alertes** dans le menu de gauche, puis **Nouvelle règle d’alerte** dans la barre d’outils. 

    :::image type="content" source="./media/activity-logs/alerts-page.png" alt-text="Rechercher Superviser":::    
1. Dans la page **Créer une règle d’alerte**, cliquez sur **Sélectionner une ressource**. 

    :::image type="content" source="./media/activity-logs/select-resource-link.png" alt-text="Rechercher Superviser":::        
1. Sélectionnez **DevTest Labs** pour **Filtrer par type de ressource**, votre labo dans la liste, puis **Terminé**.

    :::image type="content" source="./media/activity-logs/select-lab-resource.png" alt-text="Rechercher Superviser":::
1. De retour dans la page **Créer une règle d’alerte**, cliquez sur **Sélectionner une condition**. 

    :::image type="content" source="./media/activity-logs/select-condition-link.png" alt-text="Rechercher Superviser":::    
1. Dans la page **Configurer la logique du signal**, sélectionnez un signal pris en charge par DevTest Labs. 

    :::image type="content" source="./media/activity-logs/select-signal.png" alt-text="Rechercher Superviser":::
1. Filtrez par **niveau d’événement** (Détaillé, Information, Avertissement, Erreur, Critique, Tout), **état** (Échec, Démarré, Opération réussie) et **qui a lancé** l’événement. 
1. Sélectionnez **Terminé** pour terminer la configuration de la condition. 

    :::image type="content" source="./media/activity-logs/configure-signal-logic-done.png" alt-text="Rechercher Superviser":::
1. Vous avez spécifié l’étendue (labo) et la condition pour l’alerte. À présent, vous devez spécifier un groupe d’actions avec des actions à exécuter lorsque la condition est remplie. De retour dans la page **Créer une règle d’alerte**, choisissez **Sélectionner un groupe d’actions**. 

    :::image type="content" source="./media/activity-logs/select-action-group-link.png" alt-text="Rechercher Superviser":::
1. Sélectionnez le lien **Créer un groupe d’actions** dans la barre d’outils. 

    :::image type="content" source="./media/activity-logs/create-action-group-link.png" alt-text="Rechercher Superviser":::
1. Dans la page **Ajouter un groupe d’actions**, effectuez les étapes suivantes :
    1. Entrez le **nom** du groupe d’actions.
    1. Entrez le **nom court** du groupe d’actions. 
    1. Sélectionnez le **groupe de ressources** dans lequel vous souhaitez créer l’alerte. 
    1. Entrez le **nom de l’action**. 
    1. Sélectionnez le **type d’action** (dans cet exemple, **Envoyer un e-mail au rôle Azure Resource Manager**). 

        :::image type="content" source="./media/activity-logs/add-action-group.png" alt-text="Rechercher Superviser":::
    1. Dans la page **Envoyer un e-mail au rôle Azure Resource Manager**, sélectionnez le rôle. Dans cet exemple, il s’agit de **Propriétaire**. Ensuite, sélectionnez **OK**. 

        :::image type="content" source="./media/activity-logs/select-role.png" alt-text="Rechercher Superviser":::            
    1. Sélectionnez **OK** dans la page **Ajouter un groupe d’actions**. 
1. Maintenant, dans la page **Créer une règle d’alerte**, entrez le nom de la règle d’alerte, puis sélectionnez **OK**. 

    :::image type="content" source="./media/activity-logs/create-alert-rule-done.png" alt-text="Rechercher Superviser":::

## <a name="view-alerts"></a>Afficher les alertes 
1. Les alertes s’affichent dans **Alertes** pour toutes les opérations d’administration (dans cet exemple). Les alertes peuvent prendre un certain temps pour s’afficher. 

    :::image type="content" source="./media/activity-logs/alerts.png" alt-text="Rechercher Superviser":::
1. Si vous sélectionnez un nombre dans une colonne (par exemple : **Nombre total d’alertes**), vous voyez les alertes qui ont été déclenchées. 

    :::image type="content" source="./media/activity-logs/all-alerts.png" alt-text="Rechercher Superviser":::
1. Si vous sélectionnez une alerte, vous voyez les détails la concernant. 

    :::image type="content" source="./media/activity-logs/alert-details.png" alt-text="Rechercher Superviser":::
1. Dans cet exemple, vous recevez également un e-mail avec du contenu comme illustré dans l’exemple suivant : 

    :::image type="content" source="./media/activity-logs/alert-email.png" alt-text="Rechercher Superviser":::

## <a name="next-steps"></a>Étapes suivantes
- Pour en savoir plus sur la création de groupes d’actions avec différents types d’actions, consultez [Créer et gérer des groupes d’actions dans le portail Azure](../azure-monitor/platform/action-groups.md).
- Pour en savoir plus sur les journaux d’activité, consultez [Journal d’activité Azure](../azure-monitor/platform/activity-log.md).
- Pour en savoir plus sur la définition d’alertes sur les journaux d’activité, consultez [Alertes sur le journal d’activité](../azure-monitor/platform/activity-log-alerts.md).

