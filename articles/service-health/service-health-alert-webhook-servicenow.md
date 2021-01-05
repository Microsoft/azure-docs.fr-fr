---
title: Envoyer des alertes sur l’intégrité de service avec ServiceNow
description: Obtenir des notifications personnalisées sur les événements d’intégrité du service sur votre instance ServiceNow.
ms.topic: conceptual
ms.date: 06/10/2019
ms.custom: devx-track-js
ms.openlocfilehash: f120e9d950cc349d0331a476dbfbfe9d7e599f0c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000638"
---
# <a name="send-azure-service-health-alerts-with-servicenow-using-webhooks"></a>Envoyer des alertes Azure Service Health avec ServiceNow à l’aide de webhooks

Cet article vous explique comment intégrer les alertes sur l’intégrité du service Azure avec ServiceNow à l’aide d’un Webhook. Après avoir configuré l’intégration de Webhook avec votre instance ServiceNow, vous obtenez des alertes via votre infrastructure de notification existante lorsque des problèmes liés au service Azure vous affectent. Chaque fois qu’une alerte sur l’intégrité du service Azure se déclenche, celle-ci appelle un Webhook via l’API REST de script de ServiceNow.

## <a name="creating-a-scripted-rest-api-in-servicenow"></a>Création d’une API REST de script dans ServiceNow

1.  Vérifiez que vous êtes inscrit et connecté à votre compte [ServiceNow](https://www.servicenow.com/).

1.  Accédez à la section **Services web du système** dans ServiceNow, puis sélectionnez **API REST de script**.

    ![Section Service web de script dans ServiceNow](./media/webhook-alerts/servicenow-sws-section.png)

1.  Sélectionnez **Nouveau** pour créer un service REST de script.
 
    ![Bouton Nouvel API REST de script dans ServiceNow](./media/webhook-alerts/servicenow-new-button.png)

1.  Ajoutez un **nom** à votre API REST et définissez l’**ID de l’API** sur `azureservicehealth`.

1.  Sélectionnez **Envoyer**.

    ![Paramètres de l’API REST dans ServiceNow](./media/webhook-alerts/servicenow-restapi-settings.png)

1.  Sélectionnez l’API REST créée, puis, sous l’onglet **Ressources** sélectionnez **Nouveau**.

    ![Onglet Ressource dans ServiceNow](./media/webhook-alerts/servicenow-resources-tab.png)

1.  **Nommez** votre nouvelle ressource `event` et modifiez la **méthode HTTP** pour la faire passer à `POST`.

1.  Dans la section **Script**, ajoutez le code JavaScript suivant :

    >[!NOTE]
    >Vous devez mettre à jour les valeurs `<secret>`,`<group>` et `<email>` dans le script ci-dessous.
    >* `<secret>` doit être une chaîne aléatoire, comme un GUID
    >* `<group>` doit être le groupe ServiceNow auquel vous voulez attribuer l’incident
    >* `<email>` doit être la personne spécifique à laquelle vous voulez attribuer l’incident (facultatif)
    >

    ```javascript
    (function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
        var apiKey = request.queryParams['apiKey'];
        var secret = '<secret>';
        if (apiKey == secret) {
            var event = request.body.data;
            var responseBody = {};
            if (event.data.context.activityLog.operationName == 'Microsoft.ServiceHealth/incident/action') {
                var inc = new GlideRecord('incident');
                var incidentExists = false;
                inc.addQuery('number', event.data.context.activityLog.properties.trackingId);
                inc.query();
                if (inc.hasNext()) {
                    incidentExists = true;
                    inc.next();
                } else {
                    inc.initialize();
                }
                var short_description = "Azure Service Health";
                if (event.data.context.activityLog.properties.incidentType == "Incident") {
                    short_description += " - Service Issue - ";
                } else if (event.data.context.activityLog.properties.incidentType == "Maintenance") {
                    short_description += " - Planned Maintenance - ";
                } else if (event.data.context.activityLog.properties.incidentType == "Informational" || event.data.context.activityLog.properties.incidentType == "ActionRequired") {
                    short_description += " - Health Advisory - ";
                }
                short_description += event.data.context.activityLog.properties.title;
                inc.short_description = short_description;
                inc.description = event.data.context.activityLog.properties.communication;
                inc.work_notes = "Impacted subscription: " + event.data.context.activityLog.subscriptionId;
                if (incidentExists) {
                    if (event.data.context.activityLog.properties.stage == 'Active') {
                        inc.state = 2;
                    } else if (event.data.context.activityLog.properties.stage == 'Resolved') {
                        inc.state = 6;
                    } else if (event.data.context.activityLog.properties.stage == 'Closed') {
                        inc.state = 7;
                    }
                    inc.update();
                    responseBody.message = "Incident updated.";
                } else {
                    inc.number = event.data.context.activityLog.properties.trackingId;
                    inc.state = 1;
                    inc.impact = 2;
                    inc.urgency = 2;
                    inc.priority = 2;
                    inc.assigned_to = '<email>';
                    inc.assignment_group.setDisplayValue('<group>');
                    var subscriptionId = event.data.context.activityLog.subscriptionId;
                    var comments = "Azure portal Link: https://app.azure.com/h";
                    comments += "/" + event.data.context.activityLog.properties.trackingId;
                    comments += "/" + subscriptionId.substring(0, 3) + subscriptionId.slice(-3);
                    var impactedServices = JSON.parse(event.data.context.activityLog.properties.impactedServices);
                    var impactedServicesFormatted = "";
                    for (var i = 0; i < impactedServices.length; i++) {
                        impactedServicesFormatted += impactedServices[i].ServiceName + ": ";
                        for (var j = 0; j < impactedServices[i].ImpactedRegions.length; j++) {
                            if (j != 0) {
                                impactedServicesFormatted += ", ";
                            }
                            impactedServicesFormatted += impactedServices[i].ImpactedRegions[j].RegionName;
                        }

                        impactedServicesFormatted += "\n";

                    }
                    comments += "\n\nImpacted Services:\n" + impactedServicesFormatted;
                    inc.comments = comments;
                    inc.insert();
                    responseBody.message = "Incident created.";
                }
            } else {
                responseBody.message = "Hello from the other side!";
            }
            response.setBody(responseBody);
        } else {
            var unauthorized = new sn_ws_err.ServiceError();
            unauthorized.setStatus(401);
            unauthorized.setMessage('Invalid apiKey');
            response.setError(unauthorized);
        }
    })(request, response);
    ```

1.  Dans l’onglet sécurité, décochez la case **Authentification requise** et sélectionnez **Envoyer**. Le `<secret>` défini protège cette API.

    ![Case à cocher Authentification requise dans ServiceNow](./media/webhook-alerts/servicenow-resource-settings.png)

1.  Lorsque vous revenez à la section de l’API REST de script, vous devez rechercher le **chemin d’accès à l’API de base** pour votre nouvelle API REST :

     ![Chemin d’accès à l’API de base dans ServiceNow](./media/webhook-alerts/servicenow-base-api-path.png)

1.  Votre URL d’intégration complète ressemble à ce qui suit :

    ```http
    https://<yourInstanceName>.service-now.com/<baseApiPath>?apiKey=<secret>
    ```

## <a name="create-an-alert-using-servicenow-in-the-azure-portal"></a>Créer une alerte à l’aide de ServiceNow dans le portail Azure
### <a name="for-a-new-action-group"></a>Pour un nouveau groupe d’action :
1. Suivez les étapes 1 à 8 de [cet article](./alerts-activity-log-service-notifications-portal.md) pour créer une alerte avec un nouveau groupe d’actions.

1. À définir dans la liste des **Actions** :

    a. **Type d’action :** *Webhook*

    b. **Détails :** **URL d’intégration** ServiceNow précédemment enregistrée.

    c. **Nom :** nom, alias ou identificateur du webhook.

1. Quand vous avez terminé, sélectionnez **Enregistrer** pour créer l’alerte.

### <a name="for-an-existing-action-group"></a>Pour un groupe d’actions existant :
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Moniteur**.

1. Dans la section **Paramètres**, sélectionnez **Groupes d’actions**.

1. Recherchez et sélectionnez le groupe d’actions que vous souhaitez modifier.

1. À ajouter à la liste des **Actions** :

    a. **Type d’action :** *Webhook*

    b. **Détails :** **URL d’intégration** ServiceNow précédemment enregistrée.

    c. **Nom :** nom, alias ou identificateur du webhook.

1. Quand vous avez terminé, sélectionnez **Enregistrer** pour mettre à jour le groupe d’actions.

## <a name="testing-your-webhook-integration-via-an-http-post-request"></a>Tester l’intégration à Webhook via une demande HTTP POST
1. Créez la charge utile d’intégrité du service que vous souhaitez envoyer. Vous trouverez un exemple de charge utile du Webhook d’intégrité du service dans la page [Webhook pour des alertes du journal d’activité Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).

1. Créez une requête HTTP POST comme suit :

    ```
    POST        https://<yourInstanceName>.service-now.com/<baseApiPath>?apiKey=<secret>

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
1. Vous devez recevoir une réponse `200 OK` avec le message indiquant que l’incident est créé.

1. Accédez à [ServiceNow](https://www.servicenow.com/) pour confirmer que votre intégration a été définie avec succès.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [configurer des notifications de Webhook pour les systèmes de gestion de problème existants](service-health-alert-webhook-guide.md).
- Consultez le [schéma webhook des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts-webhook.md). 
- En savoir plus sur les [notifications sur l’intégrité du service](./service-notifications.md).
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).
