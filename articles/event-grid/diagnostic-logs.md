---
title: Azure Event Grid – Journaux de diagnostic pour des rubriques ou domaines
description: Cet article fournit des informations conceptuelles sur les journaux de diagnostic pour une rubrique ou un domaine Azure Event Grid.
ms.topic: conceptual
ms.date: 12/03/2020
ms.openlocfilehash: 36ade14932b5d25c7a1fe05632da671de68ba3bb
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574984"
---
#  <a name="diagnostic-logs-for-azure-event-grid-topicsdomains"></a>Journaux de diagnostic pour des rubriques/domaines Azure Event Grid
Les paramètres de diagnostic permettent aux utilisateurs d’Event Grid de capturer et d’afficher les journaux d’**échec de publication et de remise** dans un compte de stockage, un Event Hub ou espace de travail Log Analytics. Cet article fournit un schéma pour les journaux et un exemple d’entrée de journal.


## <a name="schema-for-publishdelivery-failure-logs"></a>Schéma pour les journaux d’échec de publication/remise

| Nom de la propriété | Type de données | Description |
| ------------- | --------- | ----------- | 
| Temps | DateTime | Heure de génération de l’entrée du journal <p>**Exemple de valeur :**  01-29-2020 09:52:02.700</p> |
| EventSubscriptionName | String | Nom de l’abonnement aux événements <p>**Exemple de valeur :** "EVENTSUB1"</p> <p>Cette propriété existe uniquement pour les journaux d’échec de remise.</p>  |
| Category | String | Nom de la catégorie de journal. <p>**Exemples de valeurs :** « DeliveryFailures » ou « PublishFailures » | 
| NomOpération | String | Nom de l’opération à l’origine de l’échec.<p>**Exemples de valeurs :** « Deliver » pour les échecs de remise. |
| Message | String | Message du journal destiné à l’utilisateur expliquant la raison de l’échec et fournissant d’autres détails. |
| ResourceId | String | ID de ressource pour la ressource de rubrique/domaine<p>**Exemples de valeurs :** `/SUBSCRIPTIONS/SAMPLE-SUBSCRIPTION-ID/RESOURCEGROUPS/SAMPLE-RESOURCEGROUP/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/TOPIC1` |

## <a name="example"></a>Exemple

```json
{
    "time": "2019-11-01T00:17:13.4389048Z",
    "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID /RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME ",
    "eventSubscriptionName": "SAMPLEDESTINATION",
    "category": "DeliveryFailures",
    "operationName": "Deliver",
    "message": "Message:outcome=NotFound, latencyInMs=2635, id=xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx, systemId=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx, state=FilteredFailingDelivery, deliveryTime=11/1/2019 12:17:10 AM, deliveryCount=0, probationCount=0, deliverySchema=EventGridEvent, eventSubscriptionDeliverySchema=EventGridEvent, fields=InputEvent, EventSubscriptionId, DeliveryTime, State, Id, DeliverySchema, LastDeliveryAttemptTime, SystemId, fieldCount=, requestExpiration=1/1/0001 12:00:00 AM, delivered=False publishTime=11/1/2019 12:17:10 AM, eventTime=11/1/2019 12:17:09 AM, eventType=Type, deliveryTime=11/1/2019 12:17:10 AM, filteringState=FilteredWithRpc, inputSchema=EventGridEvent, publisher=DIAGNOSTICLOGSTEST-EASTUS.EASTUS-1.EVENTGRID.AZURE.NET, size=363, fields=Id, PublishTime, SerializedBody, EventType, Topic, Subject, FilteringHashCode, SystemId, Publisher, FilteringTopic, TopicCategory, DataVersion, MetadataVersion, InputSchema, EventTime, fieldCount=15, url=sb://diagnosticlogstesting-eastus.servicebus.windows.net/, deliveryResponse=NotFound: The messaging entity 'sb://diagnosticlogstesting-eastus.servicebus.windows.net/eh-diagnosticlogstest' could not be found. TrackingId:c98c5af6-11f0-400b-8f56-c605662fb849_G14, SystemTracker:diagnosticlogstesting-eastus.servicebus.windows.net:eh-diagnosticlogstest, Timestamp:2019-11-01T00:17:13, referenceId: ac141738a9a54451b12b4cc31a10dedc_G14:"
}
```

## <a name="next-steps"></a>Étapes suivantes
Pour savoir comment activer les journaux de diagnostic pour les rubriques ou les domaines, consultez [Activer les journaux de diagnostic](enable-diagnostic-logs-topic.md).
