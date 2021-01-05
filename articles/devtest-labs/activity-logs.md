---
title: Journaux d'activité dans Azure DevTest Labs | Microsoft Docs
description: Cet article fournit la procédure d’affichage des journaux d’activité pour Azure DevTest Labs.
ms.topic: how-to
ms.date: 07/10/2020
ms.openlocfilehash: 24b5eb25f1c8c9044ad6e77545381a43a11714f9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094123"
---
# <a name="view-activity-logs-for-labs-in-azure-devtest-labs"></a>Afficher les journaux d’activité pour les laboratoires dans Azure DevTest Labs 
Une fois que vous avez créé un ou plusieurs laboratoires, vous voulez probablement contrôler qui accède à ces derniers, les modifie ou les gère, par quel moyen et quand. Azure DevTest Labs utilise Azure Monitor, en particulier les **journaux d’activité**, pour fournir des informations à ces opérations par rapport aux laboratoires. 

Cet article explique comment afficher les journaux d’activité d’un laboratoire dans Azure DevTest Labs.

## <a name="view-activity-log-for-a-lab"></a>Afficher le journal d’activité d’un laboratoire

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la section **DEVOPS**. Si vous sélectionnez * (étoile) à côté de **DevTest Labs** dans la section **DEVOPS**, cette action ajoute **DevTest Labs** au menu de navigation de gauche afin que vous puissiez y accéder facilement la prochaine fois. Vous pouvez ensuite sélectionner **DevTest Labs** dans le menu de navigation de gauche.

    ![Tous les services - Sélectionner DevTest Labs](./media/devtest-lab-create-lab/all-services-select.png)
1. Dans la liste de laboratoires, sélectionnez votre laboratoire.
1. Sur la page d’accueil du laboratoire, sélectionnez **Configuration et stratégies** dans le menu de gauche. 

    :::image type="content" source="./media/activity-logs/configuration-policies-link.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::
1. Sur la page **Configuration et stratégies**, sélectionnez **Journaux d’activité** dans le menu de gauche, sous **Gérer**. Vous devriez voir des entrées pour les opérations effectuées sur le laboratoire. 

    :::image type="content" source="./media/activity-logs/activity-log.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::    
1. Sélectionnez un événement pour en afficher les détails. Sur la page **Résumé**, vous pouvez voir des informations telles que le nom de l’opération, l’horodatage et la personne qui a effectué l’opération. 
    
    :::image type="content" source="./media/activity-logs/stop-vm-event.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::        
1. Basculez vers l’onglet **JSON** pour afficher plus de détails. Dans l’exemple suivant, vous pouvez voir le nom de la machine virtuelle et l’opération effectuée sur la machine virtuelle (arrêtée).

    :::image type="content" source="./media/activity-logs/stop-vm-event-json.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::           
1. Basculez vers l’onglet **Historique des changements (préversion)** pour afficher l’historique des modifications. Dans l’exemple suivant, vous voyez la modification qui a été apportée sur la machine virtuelle. 

    :::image type="content" source="./media/activity-logs/change-history.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::             
1. Sélectionnez la modification dans la liste d’historique des changements pour afficher plus de détails sur la modification. 

    :::image type="content" source="./media/activity-logs/change-details.png" alt-text="Sélectionnez Configuration et stratégies dans le menu de gauche":::             

Pour plus d’informations à propos des Journaux d’activité, consultez [Journal d’activité Azure](../azure-monitor/platform/activity-log.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la définition d’**alertes** sur les journaux d’activité, consultez [Créer des alertes](create-alerts.md).
- Pour en savoir plus sur les journaux d’activité, consultez [Journal d’activité Azure](../azure-monitor/platform/activity-log.md).

