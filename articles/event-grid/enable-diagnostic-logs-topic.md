---
title: Azure Event Grid – Activer les journaux de diagnostic pour des rubriques ou domaines
description: Cet article fournit des instructions pas à pas sur l’activation des journaux de diagnostic pour une rubrique Azure Event Grid.
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: ff00c1438c49cbc9f9e67eba0cf0acef7991a5a4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576449"
---
#  <a name="enable-diagnostic-logs-for-azure-event-grid-topics-or-domains"></a>Activer les journaux de diagnostic pour des rubriques ou domaines Azure Event Grid
Cet article fournit des instructions pas à pas pour activer les paramètres de diagnostic pour des rubriques ou domaines Event Grid.  Ces paramètres vous permettent de capturer et d’afficher des journaux d’**échec de publication et de remise**. 

## <a name="prerequisites"></a>Prérequis

- Rubrique Event Grid approvisionnée
- Destination approvisionnée pour la capture de journaux de diagnostic Il peut s’agir de l’une des destinations suivantes au même emplacement que la rubrique Event Grid :
    - Compte Azure Storage
    - Event Hub
    - Espace de travail Log Analytics

## <a name="enable-diagnostic-logs-for-a-custom-topic"></a>Activer les journaux de diagnostic pour une rubrique personnalisée

> [!NOTE]
> La procédure suivante fournit des instructions pas à pas pour l’activation des journaux de diagnostic pour une rubrique. Les étapes d’activation des journaux de diagnostic pour un domaine sont très similaires. À l’étape 2, accédez à l’Event Grid **domaine** dans le portail Azure.  

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à la rubrique Event Grid pour laquelle vous souhaitez activer les paramètres de journal de diagnostic. 
    1. Dans la barre de recherche en haut, recherchez **Rubriques Event Grid**. 
    
        ![Rechercher des rubriques personnalisées](./media/enable-diagnostic-logs-topic/search-custom-topics.png)
    1. Sélectionnez la **rubrique** dont vous souhaitez configurer les paramètres de diagnostic. 
1. Sous **Supervision**, sélectionnez **Paramètres de diagnostic** dans le menu de gauche.
1. Dans la page **Paramètres de diagnostic**, sélectionnez **Ajouter un nouveau paramètre de diagnostic**. 
    
    ![Bouton Ajouter un paramètre de diagnostic](./media/enable-diagnostic-logs-topic/diagnostic-settings-add.png)
5. Donnez un **nom** au paramètre de diagnostic. 
6. Sélectionnez les options **DeliveryFailures** et **PublishFailures** dans la section **Journal**. 
    ![Sélectionner les défaillances](./media/enable-diagnostic-logs-topic/log-failures.png)
7. Activez une ou plusieurs destinations de capture pour les journaux, puis configurez-les en sélectionnant une ressource de capture créée précédemment. 
    - Si vous sélectionnez **Archiver dans un compte de stockage**, sélectionnez **Compte de stockage - Configurer**, puis le compte de stockage dans votre abonnement Azure. 

        ![Capture d’écran montrant la page « Paramètres de diagnostic » avec l’option « Archiver dans un compte de stockage Azure » activée et un compte de stockage sélectionné.](./media/enable-diagnostic-logs-topic/archive-storage.png)
    - Si vous sélectionnez **Diffuser vers un hub d'événements**, sélectionnez **Hub d'événements - Configurer**, puis l’espace de noms Event Hubs, le hub d'événements et la stratégie d’accès. 
        ![Screenshot that shows the "Diagnostic settings" page with "Stream to an event hub" checked.](./media/enable-diagnostic-logs-topic/archive-event-hub.png)
    - Si vous sélectionnez **Envoyer à Log Analytics**, sélectionnez un espace de travail Log Analytics.
        ![Screenshot that shows the "Diagnostic settings" page with "Send to Log Analytics" checked.](./media/enable-diagnostic-logs-topic/send-log-analytics.png)
8. Sélectionnez **Enregistrer**. Ensuite, sélectionnez **X** dans l’angle droit pour fermer la page. 
9. À présent, dans la page **Paramètres de diagnostic**, vérifiez la présence d'une nouvelle entrée dans la table **Paramètres de diagnostic**. 
    ![Screenshot that shows the "Diagnostic settings" page with a new entry highlighted in the "Diagnostics settings" table.](./media/enable-diagnostic-logs-topic/diagnostic-setting-list.png)

     Vous pouvez également activer la collecte de toutes les métriques de la rubrique. 

## <a name="enable-diagnostic-logs-for-a-system-topic"></a>Activer les journaux de diagnostic pour une rubrique système

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à la rubrique Event Grid pour laquelle vous souhaitez activer les paramètres de journal de diagnostic. 
    1. Dans la barre de recherche en haut, recherchez **rubriques système Event Grid**. 
    
        ![Rechercher des rubriques système](./media/enable-diagnostic-logs-topic/search-system-topics.png)
    1. Sélectionnez la **rubrique système** dont vous souhaitez configurer les paramètres de diagnostic. 
    
        ![Sélectionner une rubrique système](./media/enable-diagnostic-logs-topic/select-system-topic.png)
3. Sélectionnez **Paramètres de diagnostic** sous **Supervision** dans le volet gauche, puis sélectionnez **Ajouter un paramètre de diagnostic**. 

    ![Ajouter des paramètres de diagnostic - Bouton](./media/enable-diagnostic-logs-topic/system-topic-add-diagnostic-settings-button.png)
4. Donnez un **nom** au paramètre de diagnostic. 
7. Sélectionnez **DeliveryFailures** dans la section **Journal**. 
    ![Sélectionner les échecs de livraison](./media/enable-diagnostic-logs-topic/system-topic-select-delivery-failures.png)
6. Activez une ou plusieurs destinations de capture pour les journaux, puis configurez-les en sélectionnant une ressource de capture créée précédemment. 
    - Si vous sélectionnez **Envoyer à Log Analytics**, sélectionnez un espace de travail Log Analytics.
        ![Envoyer à Log Analytics](./media/enable-diagnostic-logs-topic/system-topic-select-log-workspace.png) 
    - Si vous sélectionnez **Archiver dans un compte de stockage**, sélectionnez **Compte de stockage - Configurer**, puis le compte de stockage dans votre abonnement Azure. 

        ![Archiver dans un compte de stockage Azure](./media/enable-diagnostic-logs-topic/system-topic-select-storage-account.png)
    - Si vous sélectionnez **Diffuser vers un hub d'événements**, sélectionnez **Hub d'événements - Configurer**, puis l’espace de noms Event Hubs, le hub d'événements et la stratégie d’accès. 
        ![Diffuser vers un hub d’événements](./media/enable-diagnostic-logs-topic/system-topic-select-event-hub.png)
8. Sélectionnez **Enregistrer**. Ensuite, sélectionnez **X** dans l’angle droit pour fermer la page. 
9. À présent, dans la page **Paramètres de diagnostic**, vérifiez la présence d'une nouvelle entrée dans la table **Paramètres de diagnostic**. 
    ![Paramètre de diagnostic dans la liste](./media/enable-diagnostic-logs-topic/system-topic-diagnostic-settings-targets.png)

     Vous pouvez également activer la collecte de toutes les **métriques** de la rubrique système.

    ![Rubrique système - Activer toutes les métriques](./media/enable-diagnostic-logs-topic/system-topics-metrics.png)

## <a name="view-diagnostic-logs-in-azure-storage"></a>Afficher les journaux de diagnostic dans Stockage Azure 

1. Une fois que vous avez activé un compte de stockage comme destination de capture, Event Grid commence à émettre des journaux de diagnostic. Vous devez voir les nouveaux conteneurs nommés **Insights-logs-deliveryfailures** et **Insights-logs-publishfailures** dans le compte de stockage. 

    ![Conteneurs de stockage pour les journaux de diagnostic](./media/enable-diagnostic-logs-topic/storage-containers.png)
2. Lorsque vous naviguez dans l’un des conteneurs, vous aboutissez à un blob au format JSON. Le fichier contient des entrées de journal pour un échec de remise ou de publication. Le chemin de navigation représente le **ResourceId** de la rubrique Event Grid et l’horodateur (niveau minute) en ce qui concerne le moment auquel les entrées de journal ont été émises. À la fin, le fichier blob/JSON téléchargeable est conforme au schéma décrit dans la section suivante. 

    [ ![Fichier JSON dans le stockage](./media/enable-diagnostic-logs-topic/select-json.png) ](./media/enable-diagnostic-logs-topic/select-json.png)
3. Le contenu du fichier JSON devrait être similaire à l’exemple suivant : 

    ```json
    {
        "time": "2019-11-01T00:17:13.4389048Z",
        "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID /RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME ",
        "eventSubscriptionName": "SAMPLEDESTINATION",
        "category": "DeliveryFailures",
        "operationName": "Deliver",
        "message": "Message:outcome=NotFound, latencyInMs=2635, id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx, systemId=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx, state=FilteredFailingDelivery, deliveryTime=11/1/2019 12:17:10 AM, deliveryCount=0, probationCount=0, deliverySchema=EventGridEvent, eventSubscriptionDeliverySchema=EventGridEvent, fields=InputEvent, EventSubscriptionId, DeliveryTime, State, Id, DeliverySchema, LastDeliveryAttemptTime, SystemId, fieldCount=, requestExpiration=1/1/0001 12:00:00 AM, delivered=False publishTime=11/1/2019 12:17:10 AM, eventTime=11/1/2019 12:17:09 AM, eventType=Type, deliveryTime=11/1/2019 12:17:10 AM, filteringState=FilteredWithRpc, inputSchema=EventGridEvent, publisher=DIAGNOSTICLOGSTEST-EASTUS.EASTUS-1.EVENTGRID.AZURE.NET, size=363, fields=Id, PublishTime, SerializedBody, EventType, Topic, Subject, FilteringHashCode, SystemId, Publisher, FilteringTopic, TopicCategory, DataVersion, MetadataVersion, InputSchema, EventTime, fieldCount=15, url=sb://diagnosticlogstesting-eastus.servicebus.windows.net/, deliveryResponse=NotFound: The messaging entity 'sb://diagnosticlogstesting-eastus.servicebus.windows.net/eh-diagnosticlogstest' could not be found. TrackingId:c98c5af6-11f0-400b-8f56-c605662fb849_G14, SystemTracker:diagnosticlogstesting-eastus.servicebus.windows.net:eh-diagnosticlogstest, Timestamp:2019-11-01T00:17:13, referenceId: ac141738a9a54451b12b4cc31a10dedc_G14:"
    }
    ```
## <a name="next-steps"></a>Étapes suivantes
Pour obtenir le schéma de journal et d’autres informations conceptuelles sur les journaux de diagnostic pour les rubriques ou les domaines, consultez [Journaux de diagnostic](diagnostic-logs.md).
