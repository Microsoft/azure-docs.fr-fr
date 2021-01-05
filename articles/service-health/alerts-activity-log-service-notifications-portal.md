---
title: Recevoir des alertes de journal d’activité sur les notifications de service Azure avec le portail Azure
description: Découvrez comment configurer des alertes de journal d’activité pour les notifications sur l’intégrité du service à l’aide du portail Azure.
ms.topic: conceptual
ms.date: 06/27/2019
ms.openlocfilehash: 88fbdeeb8bdcc239f1591e053aaf4fb1c36b9b19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91289784"
---
# <a name="create-activity-log-alerts-on-service-notifications-using-the-azure-portal"></a>Créer des alertes de journal d’activité sur les notifications de service à l’aide d’un portail Azure
## <a name="overview"></a>Vue d’ensemble

Cet article vous indique comment configurer des alertes de journal d’activité pour les notifications sur l’intégrité du service à l’aide du portail Azure.  

Les notifications sur l’intégrité du service sont stockées dans le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Dans la mesure où le volume d’informations stockées dans le journal d’activité est important, il existe une interface utilisateur distincte qui permet de faciliter l’affichage et la configuration d’alertes sur ces notifications. 

Vous pouvez recevoir une alerte lorsqu’Azure envoie des notifications sur l’état du service sur votre abonnement Azure. Vous pouvez configurer l’alerte en fonction des éléments suivants :

- la classe de la notification sur l’intégrité du service (problèmes de service, maintenance planifiée, avis d’intégrité, avis de sécurité) ;
- l’abonnement affecté ;
- le ou les services affectés ;
- la ou les régions affectées ;

> [!NOTE]
> Les notifications d’intégrité de service n’envoient pas d’alerte pour les événements d’intégrité des ressources.

Vous pouvez également configurer à qui l’alerte doit être envoyée :

- sélectionner un groupe d’actions existant ;
- créer un nouveau groupe d’actions (qui peut être utilisé pour les alertes futures).

Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

Pour plus d’informations sur le mode de configuration des alertes de notification sur l’intégrité du service à l’aide de modèles Azure Resource Manager, consultez [Modèles Resource Manager](../azure-monitor/platform/alerts-activity-log.md).

### <a name="watch-a-video-on-setting-up-your-first-azure-service-health-alert"></a>Regarder une vidéo expliquant comment configurer votre première alerte Azure Service Health

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OaXt]

## <a name="create-service-health-alert-using-azure-portal"></a>Créer une alerte d’intégrité de service avec le portail Azure
1. Dans le [portail](https://portal.azure.com), sélectionnez **Intégrité du service**.

    ![Service Intégrité du service](media/alerts-activity-log-service-notifications/home-servicehealth.png)

1. Dans la section **Alertes**, sélectionnez **Alertes d’intégrité**.

    ![Onglet Alertes d’intégrité](media/alerts-activity-log-service-notifications/alerts-blades-sh.png)

1. Sélectionnez **Ajouter une alerte d’intégrité de service** et renseignez les champs.

    ![Commande Créer une alerte d’intégrité du service](media/alerts-activity-log-service-notifications/service-health-alert.png)

1. Sélectionnez l’**Abonnement**, les **Services** et les **Régions** pour lesquels vous voulez être alerté.

    [![Boîte de dialogue « Ajouter une alerte de journal d’activité »](./media/alerts-activity-log-service-notifications/activity-log-alert-new-ux.png)](./media/alerts-activity-log-service-notifications/activity-log-alert-new-ux.png#lightbox)

> [!NOTE]
>Cet abonnement est utilisé pour enregistrer l’alerte de journal d’activité. La ressource d’alerte est déployée pour cet abonnement et surveille les événements du journal d’activité.

5. Choisissez les **types d’événements** pour lesquels vous voulez recevoir des alertes : *Problème de service*, *Maintenance planifiée*, *Avis d’intégrité* et *Avis de sécurité*.

6. Cliquez sur **Sélectionner un groupe d’actions** pour choisir un groupe d’actions existant ou créer un nouveau groupe d’actions. Pour plus d’informations sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions dans le portail Azure](../azure-monitor/platform/action-groups.md).


7. Définissez les détails de l’alerte en entrant le **nom de la règle d’alerte** et une **description**.

8. Sélectionnez le **groupe de ressources** dans lequel vous souhaitez enregistrer l’alerte.



Dans quelques minutes, l’alerte est active et commence à se déclencher en fonction des conditions que vous avez spécifiées lors de la création.

Découvrez comment [configurer des notifications de Webhook pour les systèmes de gestion de problème existants](service-health-alert-webhook-guide.md). Pour plus d’informations sur le schéma de webhook sur les alertes de journal d’activité, consultez [Webhooks pour les alertes du journal d’activité Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).


## <a name="next-steps"></a>Étapes suivantes
- Découvrez les [meilleures pratiques de configuration d’alertes Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa).
- Découvrez comment [configurer des notifications Push mobiles pour Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw).
- Découvrez comment [configurer des notifications de Webhook pour les systèmes de gestion de problème existants](service-health-alert-webhook-guide.md).
- En savoir plus sur les [notifications sur l’intégrité du service](service-notifications.md).
- En savoir plus sur la [limitation du débit des notifications](../azure-monitor/platform/alerts-rate-limiting.md).
- Consultez le [schéma webhook des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts-webhook.md).
- Obtenir une [vue d’ensemble des alertes du journal d’activité](../azure-monitor/platform/alerts-overview.md) et découvrir comment recevoir des alertes.
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).
