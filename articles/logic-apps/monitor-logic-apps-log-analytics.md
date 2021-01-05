---
title: Superviser les applications logiques à l'aide des journaux d'activité Azure Monitor
description: Détecter les problèmes liés aux applications logiques en configurant les journaux d'activité Azure Monitor et en collectant des données de diagnostic pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 09/24/2020
ms.openlocfilehash: 5df596560e97ea9dba38dca4d4ca58e38caabd37
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333951"
---
# <a name="set-up-azure-monitor-logs-and-collect-diagnostics-data-for-azure-logic-apps"></a>Configurer les journaux d'activité Azure Monitor et collecter des données de diagnostic pour Azure Logic Apps

Pour obtenir des informations de débogage plus détaillées sur vos applications logiques lors de l'exécution, vous pouvez configurer et utiliser les [journaux Azure Monitor](../azure-monitor/platform/data-platform-logs.md) afin d'enregistrer et de stocker les informations relatives aux données et événements d'exécution, comme les événements de déclenchement, les événements d'exécution et les événements d'action, dans un [espace de travail Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace). [Azure Monitor](../azure-monitor/overview.md) vous permet de superviser vos environnements cloud et locaux afin de vous aider à garantir plus facilement leur disponibilité et leurs performances. Les journaux Azure Monitor vous permettent de créer des [requêtes de journal](../azure-monitor/log-query/log-query-overview.md) afin de collecter et de visualiser ces informations. Vous pouvez également [utiliser ces données de diagnostic avec d'autres services Azure](#extend-data), tels que Stockage Azure et Azure Event Hubs.

Pour configurer la journalisation de votre application logique, vous pouvez [activer Log Analytics au moment de la création de l'application](#logging-for-new-logic-apps). Pour les applications logiques existantes, vous pouvez aussi [installer la solution Logic Apps Management](#install-management-solution) dans votre espace de travail Log Analytics. Cette solution fournit des informations agrégées sur les exécutions de votre application logique et inclut des détails spécifiques comme l'état, la durée d'exécution, l'état de la nouvelle soumission et les ID de corrélation. Ensuite, pour activer la journalisation et créer des requêtes propres à ces informations, [configurez les journaux Azure Monitor](#set-up-resource-logs).

Cet article explique comment activer Log Analytics sur les nouvelles applications logiques et les applications logiques existantes, comment installer et configurer la solution Logic Apps Management et comment configurer et créer des requêtes pour les journaux d’activité Azure Monitor.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez disposer d'un [espace de travail Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace) Si vous n'en avez pas, apprenez à [créer un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md).

<a name="logging-for-new-logic-apps"></a>

## <a name="enable-log-analytics-for-new-logic-apps"></a>Activer Log Analytics pour les nouvelles applications logiques

Vous pouvez activer Log Analytics au moment de la création de votre application logique.

1. Sur le [portail Azure](https://portal.azure.com), dans le volet où vous fournissez les informations nécessaires à la création de votre application logique, procédez comme suit :

   1. Sous **Log Analytics**, sélectionnez **Activer**.

   1. Dans la liste **d’espaces de travail Log Analytics**, sélectionnez l’espace de travail vers lequel vous souhaitez envoyer les données concernant les exécutions de votre application logique.

      ![Spécifier les informations de l’application logique](./media/monitor-logic-apps-log-analytics/create-logic-app-details.png)

      Une fois cette étape terminée, Azure crée l’application logique qui est désormais associée à votre espace de travail Log Analytics. De plus, cette étape installe automatiquement la solution Logic Apps Management dans votre espace de travail Log Analytics.

1. Sélectionnez **Créer** lorsque vous avez terminé.

1. Après avoir exécuté votre application logique, [suivez ces étapes](#view-logic-app-runs) pour afficher ses exécutions.

<a name="install-management-solution"></a>

## <a name="install-logic-apps-management-solution"></a>Installer la solution Logic Apps Management

Si vous avez activé Log Analytics lors de la création de votre application logique, ignorez cette étape car vous disposez déjà de la solution Logic Apps Management dans votre espace de travail Log Analytics.

1. Dans la zone de recherche du [portail Azure](https://portal.azure.com), entrez `log analytics workspaces`, puis sélectionnez **Espaces de travail Log Analytics**.

   ![Sélectionnez « Espaces de travail Log Analytics »](./media/monitor-logic-apps-log-analytics/find-select-log-analytics-workspaces.png)

1. Sous **Espaces de travail Log Analytics**, sélectionnez votre espace de travail.

   ![Sélectionnez votre espace de travail Log Analytics.](./media/monitor-logic-apps-log-analytics/select-log-analytics-workspace.png)

1. Dans le volet **Vue d'ensemble**, sous **Bien démarrer avec Log Analytics** > **Configurer des solutions de supervision**, sélectionnez **Afficher les solutions**.

   ![Dans le volet Vue d'ensemble, sélectionnez « Afficher les solutions ».](./media/monitor-logic-apps-log-analytics/log-analytics-workspace.png)

1. Sous **Vue d’ensemble**, sélectionnez **Ajouter**.

   ![Dans le volet Vue d'ensemble, ajoutez une nouvelle solution.](./media/monitor-logic-apps-log-analytics/add-logic-apps-management-solution.png)

1. Une fois la **Place de marché** ouverte, dans la zone de recherche, entrez `logic apps management`, puis sélectionnez **Logic Apps Management**.

   ![Sur la Place de marché, sélectionnez « Logic Apps Management ».](./media/monitor-logic-apps-log-analytics/select-logic-apps-management.png)

1. Dans le volet de description de la solution, sélectionnez **Créer**.

   ![Sélectionnez « Créer » pour ajouter la solution « Logic Apps Management ».](./media/monitor-logic-apps-log-analytics/create-logic-apps-management-solution.png)

1. Vérifiez et confirmez l’espace de travail Log Analytics dans lequel vous souhaitez installer la solution, puis sélectionnez à nouveau **Créer**.

   ![Sélectionnez « Créer » pour « Logic Apps Management »](./media/monitor-logic-apps-log-analytics/confirm-log-analytics-workspace.png)

   Une fois qu’Azure a déployé la solution sur le groupe de ressources Azure qui contient votre espace de travail Log Analytics, la solution apparaît dans le volet récapitulatif de votre espace de travail.

   ![Volet récapitulatif de l'espace de travail](./media/monitor-logic-apps-log-analytics/workspace-summary-pane-logic-apps-management.png)

<a name="set-up-resource-logs"></a>

## <a name="set-up-azure-monitor-logs"></a>Configurer les journaux Azure Monitor

Lorsque vous stockez des informations sur les événements et données d'exécution dans les [journaux d'activité Azure Monitor](../azure-monitor/platform/data-platform-logs.md), vous pouvez créer des [requêtes de journal](../azure-monitor/log-query/log-query-overview.md). Celles-ci vous aideront à rechercher et consulter ces informations.

> [!NOTE]
> Une fois que vous avez activé les paramètres de diagnostic, les données de diagnostic peuvent ne pas circuler pendant 30 minutes jusqu’aux journaux de la destination spécifiée, par exemple Log Analytics, Event Hub ou un compte de stockage. Ce retard signifie que les données de diagnostic de cette période pourraient ne pas exister et ne pas être consultables. Les événements terminés et les [propriétés suivies](#extend-data) peuvent ne pas apparaître dans votre espace de travail Log Analytics pendant 10 à 15 minutes.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre application logique.

1. Dans le menu de votre application logique, sous **Supervision**, sélectionnez **Paramètres de diagnostic** > **Ajouter un paramètre de diagnostic**.

   ![Sous « Supervision », sélectionnez « Paramètres de diagnostic » > « Ajouter un paramètre de diagnostic ».](./media/monitor-logic-apps-log-analytics/logic-app-diagnostics.png)

1. Pour créer le paramètre, procédez comme suit :

   1. Donnez un nom au paramètre.

   1. Sélectionnez **Envoyer à Log Analytics**.

   1. Dans **Abonnement**, sélectionnez l'abonnement Azure associé à votre espace de travail Log Analytics.

   1. Dans **Espace de travail Log Analytics**, sélectionnez l'espace de travail que vous souhaitez utiliser.

   1. Sous **journal**, sélectionnez la catégorie **WorkflowRuntime**, qui spécifie la catégorie d'événement que vous souhaitez enregistrer.

   1. Pour sélectionner toutes les métriques, sous **métrique**, sélectionnez **AllMetrics**.

   1. Quand vous avez terminé, sélectionnez **Enregistrer**.

   Par exemple :

   ![Sélectionner l’espace de travail Log Analytics et les données à journaliser](./media/monitor-logic-apps-log-analytics/send-diagnostics-data-log-analytics-workspace.png)

<a name="view-logic-app-runs"></a>

## <a name="view-logic-app-runs-status"></a>Consulter l'état des exécutions des applications logiques

Après les exécutions de votre application logique, vous pouvez consulter les données relatives à ces exécutions dans votre espace de travail Log Analytics.

1. Sur le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre espace de travail Log Analytics.

1. Dans le menu de votre espace de travail, sélectionnez **Résumé de l'espace de travail** > **Logic Apps Management**.

   ![État et nombre d’exécutions de l’application logique](./media/monitor-logic-apps-log-analytics/logic-app-runs-summary.png)

   > [!NOTE]
   > Si la vignette Logic Apps Management n'affiche pas de résultats immédiatement après une exécution, sélectionnez **Actualiser** ou attendez un peu avant de réessayer.

   Les exécutions de votre application logique y sont regroupées par nom ou par état d’exécution. Cette page affiche également des détails sur les échecs dans les actions ou déclencheurs pour les exécutions d’une application logique.

   ![Récapitulatif des états pour les exécutions de votre application logique](./media/monitor-logic-apps-log-analytics/logic-app-runs-summary-details.png)

1. Pour afficher toutes les exécutions d'une application logique ou un état, sélectionnez la ligne correspondant à cette application logique ou à cet état.

   Voici un exemple qui montre toutes les exécutions d’une application logique :

   ![Voir les exécutions et l’état des applications logiques](./media/monitor-logic-apps-log-analytics/logic-app-run-details.png)

   > [!NOTE]
   > La capacité **Renvoyer** de cette page n’est actuellement pas disponible.

   Pour les actions pour lesquelles vous avez [configuré des propriétés suivies](#extend-data), vous pouvez également visualiser ces propriétés en sélectionnant **Afficher** dans la colonne **Propriétés suivies**. Pour rechercher les propriétés suivies, utilisez le filtre de colonne.

   ![Afficher les propriétés suivies pour une application de logique](./media/monitor-logic-apps-log-analytics/logic-app-tracked-properties.png)

1. Pour filtrer vos résultats, vous pouvez effectuer un filtrage côté client et côté serveur.

   * **Filtre côté client** : Pour chaque colonne, sélectionnez les filtres que vous souhaitez, par exemple :

     ![Exemples de filtres de colonne](./media/monitor-logic-apps-log-analytics/filters.png)

   * **Filtre côté serveur** : Pour sélectionner une fenêtre de temps spécifique ou pour limiter le nombre d’exécutions affichées, utilisez la commande d’étendue située en haut de la page. Par défaut, vous ne pouvez afficher que 1 000 enregistrements à la fois.

     ![Modifier la fenêtre de temps](./media/monitor-logic-apps-log-analytics/change-interval.png)

1. Pour voir toutes les actions et leurs détails pour une exécution spécifique, sélectionnez la ligne correspondant à une exécution d’application logique.

   Voici un exemple qui montre toutes les actions et tous les déclencheurs pour une exécution d’application logique spécifique :

   ![Afficher les actions d’une exécution d’application logique](./media/monitor-logic-apps-log-analytics/logic-app-action-details.png)

<!-------------
   * **Resubmit**: You can resubmit one or more logic apps runs that failed, succeeded, or are still running. Select the check boxes for the runs that you want to resubmit, and then select **Resubmit**.

     ![Resubmit logic app runs](./media/monitor-logic-apps-log-analytics/logic-app-resubmit.png)
--------------->

<a name="extend-data"></a>

## <a name="send-diagnostic-data-to-azure-storage-and-azure-event-hubs"></a>Envoyer des données de diagnostic à Stockage Azure et Azure Event Hubs

Avec les journaux Azure Monitor, vous pouvez étendre le mode d’utilisation des données de diagnostic de votre application logique avec d’autres services Azure, par exemple :

* [Archiver des journaux de ressources Azure sur un compte de stockage](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)
* [Diffuser en continu les journaux de plateforme Azure vers Azure Event Hubs](../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs)

Vous pouvez ensuite obtenir une surveillance en temps réel en utilisant les ressources de télémétrie et d’analyse d’autres services, tels que [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) et [Power BI](../azure-monitor/platform/powerbi.md). Par exemple :

* [Diffuser les données d’Event Hubs vers Stream Analytics](../stream-analytics/stream-analytics-define-inputs.md)
* [Analyser les données de diffusion avec Stream Analytics et créer un tableau de bord analytique en temps réel dans Power BI](../stream-analytics/stream-analytics-power-bi-dashboard.md)

En fonction des emplacements où vous souhaitez envoyer les données de diagnostic, veillez au préalable à [créer un compte de stockage Azure](../storage/common/storage-account-create.md) ou à [créer un hub d'événements Azure](../event-hubs/event-hubs-create.md). Vous pouvez ensuite sélectionner les destinations où vous souhaitez envoyer ces données. Les périodes de conservation s'appliquent uniquement lorsque vous utilisez un compte de stockage.

![Envoyer des données à un compte de stockage ou à un hub d’événements Azure](./media/monitor-logic-apps-log-analytics/diagnostics-storage-event-hub-log-analytics.png)

<a name="diagnostic-event-properties"></a>

## <a name="azure-monitor-diagnostics-events"></a>Événements de diagnostic Azure Monitor

Chaque événement de diagnostic comprend des détails sur votre application logique et l’événement, par exemple, l’état, l’heure de début, l’heure de fin, etc. Pour configurer par programmation la supervision, le suivi et la journalisation, vous pouvez utiliser ces informations avec l'[API REST d'Azure Logic Apps](/rest/api/logic) et l'[API REST d'Azure Monitor](../azure-monitor/platform/metrics-supported.md#microsoftlogicworkflows). Vous pouvez également utiliser les propriétés `clientTrackingId` et `trackedProperties`, qui apparaissent dans 

* `clientTrackingId` : si cet ID n’est pas fourni, Azure le génère automatiquement et met en corrélation les événements de l’exécution d’une application logique, y compris les flux de travail imbriqués appelés par l’application logique. Vous pouvez manuellement spécifier cet ID dans un déclencheur en ajoutant à la demande de déclenchement un en-tête `x-ms-client-tracking-id` comportant votre valeur d'ID personnalisé. Vous pouvez utiliser un déclencheur de demande, un déclencheur HTTP ou un déclencheur de webhook.

* `trackedProperties` : Pour suivre les entrées ou les sorties dans les données de diagnostic, vous pouvez ajouter une section `trackedProperties` à une action soit à l'aide du Concepteur d'application logique, soit directement dans la définition JSON de votre application logique. Les propriétés suivies ne peuvent suivre les entrées ou sorties que d’une action unique, mais vous pouvez utiliser les propriétés `correlation` d’événements pour mettre en corrélation les actions d’une exécution. Pour effectuer le suivi d'une ou plusieurs propriétés, ajoutez la section `trackedProperties` et les propriétés souhaitées à la définition de l'action.

  Voici un exemple montrant que la définition de l'action **Initialiser la variable** inclut des propriétés suivies provenant de l'entrée de l'action, sachant que l'entrée est un tableau et non un enregistrement.

  ``` json
  {
     "Initialize_variable": {
        "type": "InitializeVariable",
        "inputs": {
           "variables": [
              {
                 "name": "ConnectorName", 
                 "type": "String", 
                 "value": "SFTP-SSH" 
              }
           ]
        },
        "runAfter": {},
        "trackedProperties": { 
           "myTrackedPropertyName": "@action().inputs.variables[0].value"
        }
     }
  }
  ```

  Cet exemple comporte plusieurs propriétés suivies :

  ``` json
  "HTTP": {
     "type": "Http",
     "inputs": {
        "body": "@triggerBody()",
        "headers": {
           "Content-Type": "application/json"
        },
        "method": "POST",
        "uri": "http://store.fabrikam.com",
     },
     "runAfter": {},
     "trackedProperties": {
        "myActionHTTPStatusCode": "@action()['outputs']['statusCode']",
        "myActionHTTPValue": "@action()['outputs']['body']['<content>']",
        "transactionId": "@action()['inputs']['body']['<content>']"
     }
  }
  ```

Cet exemple montre que l'événement `ActionCompleted` comprend les attributs `clientTrackingId` et `trackedProperties` :

```json
{
   "time": "2016-07-09T17:09:54.4773148Z",
   "workflowId": "/subscriptions/XXXXXXXXXXXXXXX/resourceGroups/MyResourceGroup/providers/Microsoft.Logic/workflows/MyLogicApp",
   "resourceId": "/subscriptions/<subscription-ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Logic/workflows/MyLogicApp/runs/<run-ID>/actions/Http",
   "category": "WorkflowRuntime",
   "level": "Information",
   "operationName": "Microsoft.Logic/workflows/workflowActionCompleted",
   "properties": {
      "$schema": "2016-06-01",
      "startTime": "2016-07-09T17:09:53.4336305Z",
      "endTime": "2016-07-09T17:09:53.5430281Z",
      "status": "Succeeded",
      "code": "OK",
      "resource": {
         "subscriptionId": "<subscription-ID>",
         "resourceGroupName": "MyResourceGroup",
         "workflowId": "<logic-app-workflow-ID>",
         "workflowName": "MyLogicApp",
         "runId": "08587361146922712057",
         "location": "westus",
         "actionName": "Http"
      },
      "correlation": {
         "actionTrackingId": "e1931543-906d-4d1d-baed-dee72ddf1047",
         "clientTrackingId": "<my-custom-tracking-ID>"
      },
      "trackedProperties": {
         "myTrackedPropertyName": "<value>"
      }
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Créer des requêtes de supervision et de suivi](../logic-apps/create-monitoring-tracking-queries.md)
* [Superviser les messages B2B avec les journaux d’activité Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)
