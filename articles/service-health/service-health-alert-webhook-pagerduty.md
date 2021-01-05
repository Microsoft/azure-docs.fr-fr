---
title: Envoyer des alertes sur l’intégrité de service Azure avec PagerDuty
description: Obtenir des notifications personnalisées sur les événements d’intégrité du service sur votre instance PagerDuty.
ms.topic: conceptual
ms.date: 06/10/2019
ms.openlocfilehash: 5c70af2ad2ded919cda0b174705d1d3245eac627
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87091536"
---
# <a name="send-azure-service-health-alerts-with-pagerduty-using-webhooks"></a>Envoyer des alertes sur l’intégrité du service Azure avec PagerDuty à l’aide de webhooks

Cet article vous explique comment configurer les notifications sur l’intégrité du service Azure via PagerDuty à l’aide d’un Webhook. À l’aide du type d’intégration Microsoft Azure personnalisé [PagerDuty](https://www.pagerduty.com/), vous pouvez facilement ajouter des alertes sur l’intégrité du service à vos services PagerDuty nouveaux ou existants.

## <a name="creating-a-service-health-integration-url-in-pagerduty"></a>Création d’une URL d’intégration de service d’intégrité dans PagerDuty
1.  Vérifiez que vous êtes inscrit et connecté à votre compte [PagerDuty](https://www.pagerduty.com/).

1.  Accédez à la section **Services** dans PagerDuty.

    ![Section Services dans PagerDuty](./media/webhook-alerts/pagerduty-services-section.png)

1.  Sélectionnez **Ajouter un nouveau service** ou ouvrez un service existant que vous avez configuré.

1.  Dans les **paramètres d’intégration**, sélectionnez les éléments suivants :

    a. **Type d’intégration** : Microsoft Azure

    b. **Nom de l’intégration** : \<Name\>

    ![Paramètres d’intégration dans PagerDuty](./media/webhook-alerts/pagerduty-integration-settings.png)

1.  Renseignez tous les autres champs obligatoires et sélectionnez **Ajouter**.

1.  Ouvrez cette nouvelle intégration et copiez et enregistrez l’**URL d’intégration**.

    ![URL d’intégration dans PagerDuty](./media/webhook-alerts/pagerduty-integration-url.png)

## <a name="create-an-alert-using-pagerduty-in-the-azure-portal"></a>Créer une alerte à l’aide de PagerDuty dans le portail Azure
### <a name="for-a-new-action-group"></a>Pour un nouveau groupe d’action :
1. Suivez les étapes 1 à 8 de [Créer une alerte sur une notification sur l’intégrité du service pour un nouveau groupe d’actions à l’aide du portail Azure](./alerts-activity-log-service-notifications-portal.md).

1. À définir dans la liste des **Actions** :

    a. **Type d’action :** *Webhook*

    b. **Détails :** l’**URL d’intégration** PagerDuty précédemment enregistrée.

    c. **Nom :** nom, alias ou identificateur du webhook.

1. Quand vous avez terminé, sélectionnez **Enregistrer** pour créer l’alerte.

### <a name="for-an-existing-action-group"></a>Pour un groupe d’actions existant :
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Moniteur**.

1. Dans la section **Paramètres**, sélectionnez **Groupes d’actions**.

1. Recherchez et sélectionnez le groupe d’actions que vous souhaitez modifier.

1. À ajouter à la liste des **Actions** :

    a. **Type d’action :** *Webhook*

    b. **Détails :** l’**URL d’intégration** PagerDuty précédemment enregistrée.

    c. **Nom :** nom, alias ou identificateur du webhook.

1. Quand vous avez terminé, sélectionnez **Enregistrer** pour mettre à jour le groupe d’actions.

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>Tester l’intégration à Webhook via une demande HTTP POST
1. Créez la charge utile d’intégrité du service que vous souhaitez envoyer. Vous trouverez un exemple de charge utile du Webhook d’intégrité du service dans la page [Webhook pour des alertes du journal d’activité Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).

1. Créez une requête HTTP POST comme suit :

    ```
    POST        https://events.pagerduty.com/integration/<IntegrationKey>/enqueue

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. Vous devez recevoir un `202 Accepted` avec un message contenant votre ID d’événement.

1. Accédez à [PagerDuty](https://www.pagerduty.com/) pour confirmer que votre intégration a été définie avec succès.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [configurer des notifications de Webhook pour les systèmes de gestion de problème existants](service-health-alert-webhook-guide.md).
- Consultez le [schéma webhook des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts-webhook.md). 
- En savoir plus sur les [notifications sur l’intégrité du service](./service-notifications.md).
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).
