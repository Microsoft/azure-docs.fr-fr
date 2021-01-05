---
title: Fonction Azure en tant que gestionnaire d’événements pour des événements Azure Event Grid
description: Décrit comment utiliser des fonctions Azure en tant que gestionnaires d’événements pour des événements Event Grid.
ms.topic: conceptual
ms.date: 09/18/2020
ms.openlocfilehash: 9e04fd3e04dab7a50940c2a4a799a56d447fbb6e
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92145757"
---
# <a name="azure-function-as-an-event-handler-for-event-grid-events"></a>Fonction Azure en tant que gestionnaire d’événements pour des événements Event Grid

Un gestionnaire d’événements désigne l’endroit où l’événement est envoyé. Le gestionnaire effectue une action pour traiter l’événement. Plusieurs services Azure sont automatiquement configurés pour gérer des événements, et **Azure Functions** est l’un d’eux. 


Pour utiliser une fonction Azure comme gestionnaire d’événements, suivez l’une des approches suivantes : 

-   Utilisez un [déclencheur Event Grid](../azure-functions/functions-bindings-event-grid-trigger.md).  Spécifiez **Fonction Azure** pour le **type de point de terminaison**. Ensuite, spécifiez l’application Azure Functions et la fonction qui gérera les événements. 
-   Utilisez un [déclencheur HTTP](../azure-functions/functions-bindings-http-webhook.md).  Spécifiez **Webhook** comme **Type de point de terminaison**. Ensuite, spécifiez l’URL de la fonction Azure qui gérera les événements. 

Nous vous recommandons d’utiliser la première approche (déclencheur Event Grid), car elle présente les avantages suivants par rapport à la deuxième approche :
-   Event Grid valide automatiquement les déclencheurs Event Grid. Dans le cas des déclencheurs HTTP génériques, vous devez implémenter vous-même la [réponse de validation](webhook-event-delivery.md).
-   Event Grid ajuste automatiquement la vitesse à laquelle les événements sont transmis à une fonction déclenchée par un événement Event Grid en fonction de la vitesse perçue à laquelle la fonction peut traiter les événements. Cette fonctionnalité de correspondance de vitesse permet d’éviter les erreurs de remise qui résultent de l’incapacité d’une fonction à traiter les événements, car la vitesse de traitement des événements de la fonction peut varier dans le temps. Pour améliorer l’efficacité à un débit élevé, activez le traitement par lot sur l’abonnement aux événements. Pour plus d’informations, consultez [Activer le traitement par lot](#enable-batching).

    > [!NOTE]
    > Actuellement, vous ne pouvez pas utiliser un déclencheur Event Grid pour une application Azure Functions quand l’événement est remis dans le schéma **CloudEvents**. Utilisez à la place un déclencheur HTTP.

## <a name="tutorials"></a>Tutoriels

|Intitulé  |Description  |
|---------|---------|
| [Démarrage rapide : Gérer les événements avec une fonction](custom-event-to-function.md) | Envoie un événement personnalisé à une fonction pour traitement. |
| [Tutoriel : Automatiser le redimensionnement des images chargées à l’aide d’Event Grid](resize-images-on-storage-blob-upload-event.md) | Les utilisateurs chargent les images par le biais de l’application web sur le compte de stockage. Quand un objet blob de stockage est créé, Event Grid envoie un événement à l’application de fonction, qui redimensionne l’image chargée. |
| [Tutoriel : Diffuser en continu des Big Data dans un entrepôt de données](event-grid-event-hubs-integration.md) | Quand Event Hubs crée un fichier Capture, Event Grid envoie un événement à une application de fonction. L’application récupère le fichier Capture et migre les données vers un entrepôt de données. |
| [Tutoriel : Exemples d’intégration d’Azure Service Bus et d’Azure Event Grid](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Event Grid envoie des messages à partir de la rubrique Service Bus à une application de fonction et à une application logique. |

## <a name="rest-example-for-put"></a>Exemple REST (pour PUT)

```json
{
    "properties": 
    {
        "destination": 
        {
            "endpointType": "AzureFunction",
            "properties": 
            {
                "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Web/sites/<FUNCTION APP NAME>/functions/<FUNCTION NAME>",
                "maxEventsPerBatch": 10,
                "preferredBatchSizeInKilobytes": 6400
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

## <a name="enable-batching"></a>Activer le traitement par lot
Pour obtenir un débit plus élevé, activez le traitement par lot sur l’abonnement. Si vous utilisez le portail Azure, vous pouvez définir le nombre maximal d’événements par lot et la taille de lot par défaut en kilo-octets au moment de la création d’un abonnement ou après sa création. 

Pour configurer les paramètres de lot, vous pouvez utiliser le portail Azure, PowerShell, CLI ou le modèle Resource Manager. 

### <a name="azure-portal"></a>Portail Azure
Lors de la création d’un abonnement dans l’interface utilisateur, sur la page **Créer un abonnement aux événements**, basculez sur l’onglet **Fonctionnalités avancées** et définissez des valeurs pour **Nb max. d’événements par lot** et **Taille de lot par défaut en kilo-octets**. 
    
:::image type="content" source="./media/custom-event-to-function/enable-batching.png" alt-text="Activer le traitement par lot au moment de la création d’un abonnement":::

Vous pouvez mettre à jour ces valeurs pour un abonnement existant sous l’onglet **Fonctionnalités** de la page **Rubrique Event Grid**. 

:::image type="content" source="./media/custom-event-to-function/features-batch-settings.png" alt-text="Activer le traitement par lot au moment de la création d’un abonnement":::

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager
Vous pouvez définir **maxEventsPerBatch** et **preferredBatchSizeInKilobytes** dans un modèle Resource Manager. Pour plus d’informations, consultez [Référence sur le modèle Microsoft.EventGrid eventSubscriptions](/azure/templates/microsoft.eventgrid/eventsubscriptions).

### <a name="azure-cli"></a>Azure CLI
Vous pouvez utiliser la commande [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az_eventgrid_event_subscription_create&preserve-view=true) ou [az eventgrid event-subscription update](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az_eventgrid_event_subscription_update&preserve-view=true) pour configurer les paramètres de traitement par lot en utilisant les paramètres suivants : `--max-events-per-batch` ou `--preferred-batch-size-in-kilobytes`.

### <a name="azure-powershell"></a>Azure PowerShell
Vous pouvez utiliser la cmdlet [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) ou [Update-AzEventGridSubscription](/powershell/module/az.eventgrid/update-azeventgridsubscription) pour configurer les paramètres de traitement par lot en utilisant les paramètres suivants : `-MaxEventsPerBatch` ou `-PreferredBatchSizeInKiloBytes`.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des gestionnaires d’événements pris en charge, consultez l’article [Gestionnaires d’événements](event-handlers.md).