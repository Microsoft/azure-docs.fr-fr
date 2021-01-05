---
title: Créer des alertes d’intégrité de ressources avec le portail Azure
description: Créez des alertes via le portail Azure qui vous informent de l’indisponibilité de vos ressources Azure.
ms.topic: conceptual
ms.date: 6/23/2020
ms.openlocfilehash: 127aad1691f7bb6b6c64332eefde734a809ab75f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91540674"
---
# <a name="configure-resource-health-alerts-using-azure-portal"></a>Configurer des alertes d’intégrité de ressources avec le portail Azure

Cet article vous indique comment configurer des alertes de journal d’activité pour les notifications sur l’intégrité des ressources à l’aide du Portail Azure.

Azure Resource Health vous tient informé de l’état d’intégrité actuel et précédent de vos ressources Azure. Les alertes Azure Resource Health peuvent vous signaler quasiment en temps réel tout changement de l’état d’intégrité des ressources. La création par programmation d’alertes Resource Health permet aux utilisateurs de créer et personnaliser leurs alertes en bloc.

Les notifications sur l’intégrité des ressources sont stockées dans le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Dans la mesure où le volume d’informations stockées dans le journal d’activité peut être important, il existe une interface utilisateur distincte permettant de faciliter l’affichage et la configuration d’alertes sur ces notifications.
Vous pouvez recevoir une alerte lorsque les ressources Azure envoie des notifications sur l’état du service sur votre abonnement Azure. Vous pouvez configurer l’alerte en fonction des éléments suivants :

* l’abonnement affecté ;
* Type(s) de ressources affecté(s).
* Groupe(s) de ressources affecté(s).
* Ressource(s) affectée(s).
* État(s) d’événement des ressources affecté(s).
* État(s) des ressource(s) affecté(s).
* Motif(s) des ressources affecté(s).

Vous pouvez également configurer à qui l’alerte doit être envoyée :

* sélectionner un groupe d’actions existant ;
* créer un nouveau groupe d’actions (qui peut être utilisé pour les alertes futures).

Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

Pour plus d’informations sur le mode de configuration des alertes de notification sur l’intégrité des ressources à l’aide de modèles Azure Resource Manager, consultez [Modèles Resource Manager](./resource-health-alert-arm-template-guide.md).
Alerte d’intégrité de ressources avec le portail Azure

## <a name="resource-health-alert-using-azure-portal"></a>Alerte d’intégrité de ressources avec le portail Azure

1. Dans le [portail](https://portal.azure.com/) Azure, sélectionnez **Intégrité du service**.

    ![Sélection d’intégrité du service](./media/resource-health-alert-monitor-guide/service-health-selection.png)
2. Dans la section **Resource Health**, sélectionnez **Service Health**.
3. Sélectionnez **Ajouter une alerte d’intégrité des ressources** et renseignez les champs.
4. Sous cible de l’alerte, sélectionnez l’**abonnement**, les **types de ressources**, les **groupes de ressources** et les **ressources** pour lesquelles vous souhaitez être alerté.

    ![Sélection de la cible](./media/resource-health-alert-monitor-guide/alert-target.png)

5. Sous condition d’alerte, sélectionnez :
    1. Les **états d’événements** pour lesquels vous voulez recevoir des alertes. Niveau de gravité de l’événement : Actif, Résolu, En cours, Mis à jour
    2. Les **états des ressources** pour lesquels vous voulez recevoir des alertes. États des ressources de l’événement : Disponible, Non disponible, Inconnu, Détérioré
    3. Les **Motifs** pour lesquels vous voulez recevoir des alertes. Cause de l'événement : Initié par la plateforme, Initié par l’utilisateur ![Sélection des conditions d’alerte d’intégrité](./media/resource-health-alert-monitor-guide/alert-condition.png)
6. Sous Définir les détails de l’alerte, indiquez les informations suivantes :
    1. **Nom de la règle d’alerte** : nom de la nouvelle règle d’alerte.
    2. **Description** : Description de la nouvelle règle d’alerte.
    3. **Enregistrer l'alerte dans le groupe de ressources** : sélectionnez le groupe de ressources dans lequel vous souhaitez enregistrer cette nouvelle règle.
7. Dans le menu déroulant sous **Groupe d’actions**, spécifiez le groupe d’actions que vous souhaitez affecter à cette nouvelle règle d’alerte. Vous pouvez également [créer un groupe d’actions](../azure-monitor/platform/action-groups.md) et l’affecter à la nouvelle règle. Pour créer un groupe, sélectionnez + **Nouveau groupe**.
8. Pour activer les règles après les avoir créées, cliquez sur **Oui** dans l’option **Activer la règle lors de la création**.
9. Sélectionnez **Créer une règle d’alerte**.

La nouvelle règle d’alerte de journal d’activité est créée et un message de confirmation s’affiche en haut à droite de la fenêtre.
Vous pouvez activer, désactiver, modifier ou supprimer une règle. En savoir plus sur la [gestion des règles de journal d’activité](../azure-monitor/platform/alerts-activity-log.md#view-and-manage-in-the-azure-portal).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Resource Health :

* [Vue d’ensemble d’Azure Resource Health](Resource-health-overview.md)
* [Types de ressource et contrôles d’intégrité disponibles par le biais d’Azure Resource Health](resource-health-checks-resource-types.md)

Créer des alertes Service Health :

* [Configurer des alertes pour Service Health](./alerts-activity-log-service-notifications-portal.md) 
* [Schéma d’événement du journal d’activité Azure](../azure-monitor/platform/activity-log-schema.md)
* [Configurer les alertes Resource Health à l’aide de modèles Resource Manager](./resource-health-alert-arm-template-guide.md)
