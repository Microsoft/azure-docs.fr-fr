---
title: Guide pratique pour déclencher des actions complexes avec des alertes Azure Monitor
description: Découvrez comment créer une action d’application logique pour traiter des alertes Azure Monitor.
author: dkamstra
ms.author: dukek
ms.topic: conceptual
ms.date: 07/18/2018
ms.subservice: alerts
ms.openlocfilehash: f76d28018fdf55314593dabc44ef1e9a1dab9494
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995095"
---
# <a name="how-to-trigger-complex-actions-with-azure-monitor-alerts"></a>Guide pratique pour déclencher des actions complexes avec des alertes Azure Monitor

Cet article explique comment configurer une application logique pour qu’elle crée une conversation dans Microsoft Teams quand une alerte se déclenche.

## <a name="overview"></a>Vue d’ensemble

Quand une alerte Azure Monitor se déclenche, elle appelle un [groupe d’actions](./action-groups.md). Les groupes d’actions permettent de déclencher une ou plusieurs actions pour avertir les autres en cas d’alerte et y remédier.

Le processus général est le suivant :

-   Créez l’application logique pour le type d’alerte en question.

-   Importez un exemple de charge utile associé au type d’alerte en question dans l’application logique.

-   Définissez le comportement de l’application logique.

-   Copiez le point de terminaison HTTP de l’application logique dans un groupe d’actions Azure.

Le processus est similaire si vous souhaitez que l’application logique effectue une autre action.

## <a name="create-an-activity-log-alert-administrative"></a>Créer une alerte de journal d’activité : Administratif

1.  Sur le Portail Azure, sélectionnez **Créer une ressource** en haut à gauche.

2.  Recherchez et sélectionnez **Application logique**, puis **Créer**.

3.  Donnez un **Nom** à votre application logique, choisissez un **Groupe de ressources** et ainsi de suite.

    ![Créer une application logique](media/action-groups-logic-app/create-logic-app-dialog.png "Créer une application logique")

4.  Sélectionnez **Créer** pour créer l’application logique. Un message contextuel indique que l’application logique est créée. Sélectionnez **Lancer la ressource** pour ouvrir le **Concepteur d’applications logiques**.

5.  Sélectionnez le déclencheur : **Lors de la réception d’une requête HTTP**.

    ![Déclencheurs d’application logique](media/action-groups-logic-app/logic-app-triggers.png "Déclencheurs d’application logique")

6.  Sélectionnez **Modifier** pour changer de déclencheur de requête HTTP.

    ![Déclencheurs de requêtes HTTP](media/action-groups-logic-app/http-request-trigger-shape.png "Déclencheurs de requêtes HTTP")

7.  Sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

    ![Utiliser un exemple de charge utile](media/action-groups-logic-app/use-sample-payload-button.png "Utiliser un exemple de charge utile")

8.  Copiez et collez l’exemple de charge utile suivant dans la boîte de dialogue :

    ```json
        {
            "schemaId": "Microsoft.Insights/activityLogs",
            "data": {
                "status": "Activated",
                "context": {
                "activityLog": {
                    "authorization": {
                    "action": "microsoft.insights/activityLogAlerts/write",
                    "scope": "/subscriptions/…"
                    },
                    "channels": "Operation",
                    "claims": "…",
                    "caller": "logicappdemo@contoso.com",
                    "correlationId": "91ad2bac-1afa-4932-a2ce-2f8efd6765a3",
                    "description": "",
                    "eventSource": "Administrative",
                    "eventTimestamp": "2018-04-03T22:33:11.762469+00:00",
                    "eventDataId": "ec74c4a2-d7ae-48c3-a4d0-2684a1611ca0",
                    "level": "Informational",
                    "operationName": "microsoft.insights/activityLogAlerts/write",
                    "operationId": "61f59fc8-1442-4c74-9f5f-937392a9723c",
                    "resourceId": "/subscriptions/…",
                    "resourceGroupName": "LOGICAPP-DEMO",
                    "resourceProviderName": "microsoft.insights",
                    "status": "Succeeded",
                    "subStatus": "",
                    "subscriptionId": "…",
                    "submissionTimestamp": "2018-04-03T22:33:36.1068742+00:00",
                    "resourceType": "microsoft.insights/activityLogAlerts"
                }
                },
                "properties": {}
            }
        }
    ```

9. Le **Concepteur d’applications logiques** affiche une fenêtre indépendante rappelant que la requête envoyée à l’application logique doit définir l’en-tête **Content-Type** sur **application/json**. Fermez la fenêtre indépendante. L’alerte Azure Monitor définit l’en-tête.

    ![Définir l'en-tête Content-Type](media/action-groups-logic-app/content-type-header.png "Définir l'en-tête Content-Type")

10. Sélectionnez **+** **Nouvelle étape**, puis **Ajouter une action**.

    ![Ajouter une action](media/action-groups-logic-app/add-action.png "Ajouter une action")

11. Recherchez et sélectionnez le connecteur Microsoft Teams. Choisissez l’action **Microsoft Teams – Publier un message**.

    ![Action Microsoft Teams](media/action-groups-logic-app/microsoft-teams-actions.png "Action Microsoft Teams")

12. Configurez l’action Microsoft Teams. Le **Concepteur Logic Apps** vous invite à vous authentifier pour vous connecter à votre compte professionnel ou scolaire. Choisissez l’**ID de l’équipe** et l’**ID du canal** auxquels envoyer le message.

13. Configurez le message en utilisant une combinaison de texte statique et de références aux \<fields\> du contenu dynamique. Coupez et collez le texte suivant dans le champ **Message** :

    ```text
      Activity Log Alert: <eventSource>
      operationName: <operationName>
      status: <status>
      resourceId: <resourceId>
    ```

    Recherchez et remplacez ensuite les \<fields\> par des balises de contenu dynamique du même nom.

    > [!NOTE]
    > Il existe deux champs dynamiques nommés **status**. Ajoutez les deux au message. Utilisez celui du jeu de propriétés **activityLog** et supprimez l’autre. Placez le curseur sur le champ **status** pour voir la référence complète au champ, comme sur la capture d’écran suivante :

    ![Action Microsoft Teams : Poster un message](media/action-groups-logic-app/teams-action-post-message.png "Action Microsoft Teams : Envoyer un message")

14. En haut du **Concepteur d’applications logiques**, sélectionnez **Enregistrer** pour enregistrer votre application logique.

15. Ouvrez votre groupe d’actions existant et ajoutez une action pour référencer l’application logique. Si vous n’avez pas de groupe d’actions, voir [Créer et gérer des groupes d’actions sur le portail Azure](./action-groups.md) pour en créer un. N’oubliez pas d’enregistrer vos modifications.

    ![Mettre à jour le groupe d’actions](media/action-groups-logic-app/update-action-group.png "Mettre à jour le groupe d’actions")

La prochaine fois qu’une alerte appellera votre groupe d’actions, votre application logique sera appelée.

## <a name="create-a-service-health-alert"></a>Créer une alerte d’intégrité du service

Les entrées Azure Service Health font partie du journal d’activité. Le processus de création de l’alerte est semblable à celui de [création d’une alerte de journal d'activité](#create-an-activity-log-alert-administrative), mais avec quelques modifications :

- Les étapes 1 à 7 sont identiques.
- À l’étape 8, utilisez l’exemple de charge utile suivant pour le déclencheur de requête HTTP :

    ```json
    {
        "schemaId": "Microsoft.Insights/activityLogs",
        "data": {
            "status": "Activated",
            "context": {
                "activityLog": {
                    "channels": "Admin",
                    "correlationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "description": "…",
                    "eventSource": "ServiceHealth",
                    "eventTimestamp": "2018-04-03T22:44:43.7467716+00:00",
                    "eventDataId": "9ce152f5-d435-ee31-2dce-104228486a6d",
                    "level": "Informational",
                    "operationName": "Microsoft.ServiceHealth/incident/action",
                    "operationId": "e416ed3c-8874-4ec8-bc6b-54e3c92a24d4",
                    "properties": {
                        "title": "...",
                        "service": "...",
                        "region": "Global",
                        "communication": "...",
                        "incidentType": "Incident",
                        "trackingId": "...",
                        "impactStartTime": "2018-03-22T21:40:00.0000000Z",
                        "impactMitigationTime": "2018-03-22T21:41:00.0000000Z",
                        "impactedServices": "[{"ImpactedRegions"}]",
                        "defaultLanguageTitle": "...",
                        "defaultLanguageContent": "...",
                        "stage": "Active",
                        "communicationId": "11000001466525",
                        "version": "0.1.1"
                    },
                    "status": "Active",
                    "subscriptionId": "...",
                    "submissionTimestamp": "2018-04-03T22:44:50.8013523+00:00"
                }
            },
            "properties": {}
        }
    }
    ```

-  Les étapes 9 et 10 sont identiques.
-  De l’étape 11 à l’étape 14, suivez le processus ci-dessous :

   1. Sélectionnez **+** **Nouvelle étape**, puis **Ajouter une condition**. Définissez les conditions suivantes pour que l’application logique s’exécute uniquement lorsque les données d’entrée correspondent aux valeurs ci-dessous.  Lorsque vous entrez la valeur de version dans la zone de texte, placez-la entre guillemets (« 0.1.1 ») pour vous assurer qu’elle soit évaluée comme étant de type chaîne et non numérique.  Le système n’affiche pas les guillemets si vous revenez à la page, mais le code sous-jacent conserve le type chaîne.   
       - `schemaId == Microsoft.Insights/activityLogs`
       - `eventSource == ServiceHealth`
       - `version == "0.1.1"`

      ![« Condition de charge utile Service Health »](media/action-groups-logic-app/service-health-payload-condition.png "Condition de charge utile Service Health")

   1. Dans la condition **If true**, suivez les instructions des étapes 11 à 13 de [Créer une alerte de journal d’activité](#create-an-activity-log-alert-administrative) pour ajouter l’action Microsoft Teams.

   1. Définissez le message en utilisant une combinaison de HTML et de contenu dynamique. Coupez et collez le contenu suivant dans le champ **Message**. Remplacez les champs `[incidentType]`, `[trackingID]`, `[title]` et `[communication]` par des balises de contenu dynamique du même nom :

       ```html
       <p>
       <b>Alert Type:</b>&nbsp;<strong>[incidentType]</strong>&nbsp;
       <strong>Tracking ID:</strong>&nbsp;[trackingId]&nbsp;
       <strong>Title:</strong>&nbsp;[title]</p>
       <p>
       <a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.</a>
       </p>
       <p>[communication]</p>
       ```

       ![« Action postérieure condition vraie Service Health »](media/action-groups-logic-app/service-health-true-condition-post-action.png "Action postérieure condition vraie Service Health")

   1. Pour la condition **If false**, indiquez un message utile :

       ```html
       <p><strong>Service Health Alert</strong></p>
       <p><b>Unrecognized alert schema</b></p>
       <p><a href="https://ms.portal.azure.com/#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues">For details, log in to the Azure Service Health dashboard.\</a></p>
       ```

       ![« Action postérieure condition fausse Service Health »](media/action-groups-logic-app/service-health-false-condition-post-action.png "Action postérieure condition fausse Service Health")

- L’étape 15 est identique. Suivez les instructions pour enregistrer votre application logique et mettre à jour votre groupe d’actions.

## <a name="create-a-metric-alert"></a>Créer une alerte de métrique

Le processus de création d’une alerte de métrique est semblable à celui de [création d’une alerte de journal d'activité](#create-an-activity-log-alert-administrative), mais avec quelques modifications :

- Les étapes 1 à 7 sont identiques.
- À l’étape 8, utilisez l’exemple de charge utile suivant pour le déclencheur de requête HTTP :

    ```json
    {
    "schemaId": "AzureMonitorMetricAlert",
    "data": {
        "version": "2.0",
        "status": "Activated",
        "context": {
        "timestamp": "2018-04-09T19:00:07.7461615Z",
        "id": "...",
        "name": "TEST-VM CPU Utilization",
        "description": "",
        "conditionType": "SingleResourceMultipleMetricCriteria",
        "condition": {
            "windowSize": "PT15M",
            "allOf": [
                {
                    "metricName": "Percentage CPU",
                    "dimensions": [
                    {
                        "name": "ResourceId",
                        "value": "d92fc5cb-06cf-4309-8c9a-538eea6a17a6"
                    }
                ],
                "operator": "GreaterThan",
                "threshold": "5",
                "timeAggregation": "PT15M",
                "metricValue": 1.0
            }
            ]
        },
        "subscriptionId": "...",
        "resourceGroupName": "TEST",
        "resourceName": "test-vm",
        "resourceType": "Microsoft.Compute/virtualMachines",
        "resourceId": "...",
        "portalLink": "..."
        },
        "properties": {}
    }
    }
    ```

- Les étapes 9 et 10 sont identiques.
- De l’étape 11 à l’étape 14, suivez le processus ci-dessous :

  1. Sélectionnez **+** **Nouvelle étape**, puis **Ajouter une condition**. Définissez les conditions suivantes pour que l’application logique s’exécute uniquement lorsque les données d’entrée correspondent aux valeurs ci-dessous. Lorsque vous entrez la valeur de version dans la zone de texte, placez-la entre guillemets (« 2.0 ») pour vous assurer qu’elle soit évaluée comme étant de type chaîne et non numérique.  Le système n’affiche pas les guillemets si vous revenez à la page, mais le code sous-jacent conserve le type chaîne. 
     - `schemaId == AzureMonitorMetricAlert`
     - `version == "2.0"`
       
       ![« Condition de charge utile d’alerte de métrique »](media/action-groups-logic-app/metric-alert-payload-condition.png "Condition de charge utile d’alerte de métrique")

  1. Dans la condition **If true**, ajoutez une boucle **For each** et l’action Microsoft Teams. Définissez le message en utilisant une combinaison de HTML et de contenu dynamique.

      ![« Action postérieure condition vraie d’alerte métrique »](media/action-groups-logic-app/metric-alert-true-condition-post-action.png "Action postérieure condition vraie d’alerte métrique")

  1. Dans la condition **If false**, définissez une action Microsoft Teams pour signaler que l’alerte de métrique ne correspond pas aux attentes de l’application logique. Incluez la charge utile JSON. Notez comment faire référence au contenu dynamique `triggerBody` de l’expression `json()`.

      ![« Action postérieure condition fausse d’alerte métrique »](media/action-groups-logic-app/metric-alert-false-condition-post-action.png "Action postérieure condition fausse d’alerte métrique")

- L’étape 15 est identique. Suivez les instructions pour enregistrer votre application logique et mettre à jour votre groupe d’actions.

## <a name="calling-other-applications-besides-microsoft-teams"></a>Appel d’applications autres que Microsoft Teams
Logic Apps propose plusieurs connecteurs qui vous permettent de déclencher des actions dans un large éventail d’applications et de bases de données. Slack, SQL Server, Oracle, Salesforce en sont quelques exemples. Pour plus d’informations sur les connecteurs, consultez [Connecteurs Logic App](../../connectors/apis-list.md).  

## <a name="next-steps"></a>Étapes suivantes
* Dans une [vue d’ensemble des alertes du journal d’activité Azure](./alerts-overview.md), découvrez comment recevoir des alertes.  
* Découvrez comment [configurer des alertes lorsqu’une notification Azure Service Health est publiée](../../service-health/alerts-activity-log-service-notifications-portal.md).
* En savoir plus sur les [groupes d’actions](./action-groups.md).

