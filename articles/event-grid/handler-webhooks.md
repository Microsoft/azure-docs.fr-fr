---
title: Webhooks en tant que gestionnaires d’événements pour les événements Azure Event Grid
description: Décrit comment utiliser des webhooks en tant que gestionnaires d’événements pour des événements Azure Event Grid. Les runbooks et applications logiques Azure Automation sont pris en charge en tant que gestionnaires d’événements via des webhooks.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: aa20c75789a18b93f787134dffe165e60ff1ab6f
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91875799"
---
# <a name="webhooks-automation-runbooks-logic-apps-as-event-handlers-for-azure-event-grid-events"></a>Webhooks, runbooks Automation, applications logiques en tant que gestionnaires d’événements pour les événements Azure Event Grid
Un gestionnaire d’événements désigne l’endroit où l’événement est envoyé. Le gestionnaire effectue des actions supplémentaires pour traiter l’événement. Plusieurs services Azure sont automatiquement configurés pour gérer les événements. Vous pouvez également utiliser n’importe quel webhook pour la gestion des événements. À cette fin, il n’est pas nécessaire que le webhook soit hébergé dans Azure. Event Grid ne prend en charge que les points de terminaison Webhook HTTPS.

> [!NOTE]
> - Les runbooks et applications logiques Azure Automation sont pris en charge en tant que gestionnaires d’événements via des webhooks. 
> - Même si vous pouvez utiliser **Webhook** en tant que **type de point de terminaison** pour configurer une fonction Azure en tant que gestionnaire d’événements, utilisez **Azure Function** comme type de point de terminaison. Pour en savoir plus, consultez [Fonction Azure en tant que gestionnaire d’événements](handler-functions.md).

## <a name="webhooks"></a>webhooks
Consultez les articles suivants pour obtenir une vue d’ensemble et des exemples d’utilisation de webhooks en tant que gestionnaires d’événements. 

|Intitulé  |Description  |
|---------|---------|
| Démarrage rapide : Créer et acheminer des événements personnalisés avec [Azure CLI](custom-event-quickstart.md), [PowerShell](custom-event-quickstart-powershell.md) et le [portail](custom-event-quickstart-portal.md). | Montre comment envoyer des événements personnalisés à un webhook. |
| Démarrage rapide : Acheminer des événements de stockage Blob vers un point de terminaison web avec [Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json), [PowerShell](../storage/blobs/storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fevent-grid%2ftoc.json) et le [portail](blob-event-quickstart-portal.md). | Montre comment envoyer des événements de stockage d’objets blob à un webhook. |
| [Démarrage rapide : Envoyer des événements de registre de conteneurs](../container-registry/container-registry-event-grid-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Montre comment utiliser Azure CLI pour envoyer des événements de registre de conteneurs. |
| [Vue d’ensemble : Recevoir des événements sur un point de terminaison HTTP](receive-events.md) | Décrit comment valider un point de terminaison HTTP pour recevoir des événements à partir d’un abonnement à des événements et désérialiser des événements. |


## <a name="azure-automation"></a>Azure Automation
Vous pouvez traiter des événements à l’aide de runbooks Azure Automation. Le traitement d’événements à l’aide de runbooks automatisés est pris en charge via des webhooks. Vous créez un webhook pour le runbook, puis utilisez le gestionnaire de webhook. Pour un exemple, consultez le tutoriel suivant : 

|Intitulé  |Description  |
|---------|---------|
|[Tutoriel : Intégrer Azure Automation à Event Grid et Microsoft Teams](ensure-tags-exists-on-new-virtual-machines.md) |Créez une machine virtuelle, qui envoie un événement. L’événement déclenche un runbook Automation qui balise la machine virtuelle et déclenche un message qui est envoyé à un canal Microsoft Teams. |


## <a name="logic-apps"></a>Logic Apps
Utilisez **Logic Apps** pour implémenter des processus d’entreprise afin de traiter des événements Event Grid. Dans ce scénario, vous ne créez pas explicitement de webhook. Le webhook est automatiquement créé pour vous lorsque vous configurez l’application logique pour gérer des événements à partir d’Event Grid. Pour des exemples, consultez les tutoriels suivants : 

|Intitulé  |Description  |
|---------|---------|
| [Tutoriel : Surveiller les modifications d'une machine virtuelle avec Azure Event Grid et Azure Logic Apps](monitor-virtual-machine-changes-event-grid-logic-app.md) | Une application logique surveille les modifications apportées à une machine virtuelle et envoie des e-mails à ce sujet. |
| [Tutoriel : Envoyer des notifications par e-mail sur des événements Azure IoT Hub à l’aide de Logic Apps](publish-iot-hub-events-to-logic-apps.md) | Une application logique envoie un e-mail de notification chaque fois qu’un appareil est ajouté à votre hub IoT. |
| [Tutoriel : Répondre aux événements Azure Service Bus reçus via Azure Event Grid à l’aide d’Azure Functions et d’Azure Logic Apps](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Event Grid envoie des messages à partir de la rubrique Service Bus à l’application de fonction et à l’application logique. |

## <a name="rest-example-for-put"></a>Exemple REST (pour PUT)

```json
{
    "properties": 
    {
        "destination": 
        {
            "endpointType": "WebHook",
            "properties": 
            {
                "endpointUrl": "<WEB HOOK URL>",
                "maxEventsPerBatch": 1,
                "preferredBatchSizeInKilobytes": 64
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des gestionnaires d’événements pris en charge, consultez l’article [Gestionnaires d’événements](event-handlers.md). 
