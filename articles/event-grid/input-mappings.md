---
title: Mapper un champ personnalisé au schéma Azure Event Grid
description: Cet article explique comment convertir votre schéma personnalisé en schéma Azure Event Grid lorsque vos données d’événement ne correspondent pas au schéma Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 34381782c9337631b0aa04e47eb5897a8071139a
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109196"
---
# <a name="map-custom-fields-to-event-grid-schema"></a>Mapper des champs personnalisés au schéma Event Grid

Si vos données d’événement ne correspondent pas au [schéma Event Grid](event-schema.md) attendu, vous pouvez toujours utiliser Event Grid pour acheminer l’événement vers les abonnés. Cet article décrit comment mapper votre schéma au schéma Event Grid.

## <a name="original-event-schema"></a>Schéma d’événement d’origine

Supposons que vous avez une application qui envoie des événements dans le format suivant :

```json
[
  {
    "myEventTypeField":"Created",
    "resource":"Users/example/Messages/1000",
    "resourceData":{"someDataField1":"SomeDataFieldValue"}
  }
]
```

Bien que ce format ne corresponde pas au schéma requis, Event Grid vous permet de mapper vos champs au schéma. Ou bien, vous pouvez recevoir les valeurs dans le schéma d’origine.

## <a name="create-custom-topic-with-mapped-fields"></a>Créer une rubrique personnalisée avec des champs mappés

Quand vous créez une rubrique personnalisée, spécifiez comment mapper les champs à partir de l’événement d’origine au schéma de grille d’événement. Vous disposez de trois valeurs pour personnaliser le mappage :

* La valeur du **schéma d’entrée** spécifie le type de schéma. Le schéma CloudEvents, le schéma d’événement personnalisé et le schéma Event Grid sont les options disponibles. La valeur par défaut est le schéma Event Grid. Quand vous créez un mappage personnalisé entre votre schéma et le schéma de grille d’événement, utilisez le schéma d’événement personnalisé. Quand les événements sont au format CloudEvents, utilisez le schéma CloudEvents.

* Le paramètre des **valeurs par défaut de mappage** spécifie les valeurs par défaut des champs dans le schéma Event Grid. Vous pouvez définir des valeurs par défaut pour `subject`, `eventtype` et `dataversion`. En règle générale, vous utilisez ce paramètre quand votre schéma personnalisé ne contient aucun champ correspondant à l’un de ces trois champs. Par exemple, vous pouvez spécifier que le paramètre dataversion est toujours défini sur **1.0**.

* La valeur des **champs de mappage** mappe les champs à partir de votre schéma au schéma de grille d’événement. Vous spécifiez les valeurs sous la forme de paires clé/valeur séparées par des espaces. En guise de nom de clé, utilisez le nom du champ de la grille d’événement. En guise de valeur, utilisez le nom de votre champ. Vous pouvez utiliser des noms de clé pour `id`, `topic`, `eventtime`, `subject`, `eventtype` et `dataversion`.

Pour créer une rubrique personnalisée Azure CLI, utilisez :

```azurecli-interactive
az eventgrid topic create \
  -n demotopic \
  -l eastus2 \
  -g myResourceGroup \
  --input-schema customeventschema \
  --input-mapping-fields eventType=myEventTypeField \
  --input-mapping-default-values subject=DefaultSubject dataVersion=1.0
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
New-AzEventGridTopic `
  -ResourceGroupName myResourceGroup `
  -Name demotopic `
  -Location eastus2 `
  -InputSchema CustomEventSchema `
  -InputMappingField @{eventType="myEventTypeField"} `
  -InputMappingDefaultValue @{subject="DefaultSubject"; dataVersion="1.0" }
```

## <a name="subscribe-to-event-grid-topic"></a>S’abonner à une rubrique de grille d’événement

Durant l’abonnement à la rubrique personnalisée, vous spécifiez le schéma que vous souhaitez utiliser pour recevoir les événements. Vous spécifiez le schéma CloudEvents, le schéma d’événement personnalisé ou le schéma Event Grid. La valeur par défaut est le schéma Event Grid.

L’exemple suivant illustre un abonnement à une rubrique Event Grid et l’utilisation du schéma Event Grid. Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
topicid=$(az eventgrid topic show --name demoTopic -g myResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name eventsub1 \
  --event-delivery-schema eventgridschema \
  --endpoint <endpoint_URL>
```

L’exemple suivant utilise le schéma d’entrée de l’événement :

```azurecli-interactive
az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name eventsub2 \
  --event-delivery-schema custominputschema \
  --endpoint <endpoint_URL>
```

L’exemple suivant illustre un abonnement à une rubrique Event Grid et l’utilisation du schéma Event Grid. Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName myResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName eventsub1 `
  -EndpointType webhook `
  -Endpoint <endpoint-url> `
  -DeliverySchema EventGridSchema
```

L’exemple suivant utilise le schéma d’entrée de l’événement :

```azurepowershell-interactive
New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName eventsub2 `
  -EndpointType webhook `
  -Endpoint <endpoint-url> `
  -DeliverySchema CustomInputSchema
```

## <a name="publish-event-to-topic"></a>Publier un événement sur la rubrique

Vous êtes maintenant prêt à envoyer un événement à la rubrique personnalisée et à voir le résultat du mappage. Le script suivant permet de publier un événement dans [l’exemple de schéma](#original-event-schema) :

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
endpoint=$(az eventgrid topic show --name demotopic -g myResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name demotopic -g myResourceGroup --query "key1" --output tsv)

event='[ { "myEventTypeField":"Created", "resource":"Users/example/Messages/1000", "resourceData":{"someDataField1":"SomeDataFieldValue"} } ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
$endpoint = (Get-AzEventGridTopic -ResourceGroupName myResourceGroup -Name demotopic).Endpoint
$keys = Get-AzEventGridTopicKey -ResourceGroupName myResourceGroup -Name demotopic

$htbody = @{
    myEventTypeField="Created"
    resource="Users/example/Messages/1000"
    resourceData= @{
        someDataField1="SomeDataFieldValue"
    }
}

$body = "["+(ConvertTo-Json $htbody)+"]"
Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

À présent, examinez votre point de terminaison WebHook. Les deux abonnements ont remis des événements dans des schémas différents.

Le premier abonnement a utilisé le schéma de grille d’événement. Le format de l’événement remis est :

```json
{
  "id": "aa5b8e2a-1235-4032-be8f-5223395b9eae",
  "eventTime": "2018-11-07T23:59:14.7997564Z",
  "eventType": "Created",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.EventGrid/topics/demotopic",
  "subject": "DefaultSubject",
  "data": {
    "myEventTypeField": "Created",
    "resource": "Users/example/Messages/1000",
    "resourceData": {
      "someDataField1": "SomeDataFieldValue"
    }
  }
}
```

Ces champs contiennent les mappages à partir de la rubrique personnalisée. **myEventTypeField** est mappé à **EventType**. Les valeurs par défaut pour **DataVersion** et **Subject** sont utilisées. L’objet **Data** contient les champs de schéma d’événement d’origine.

Le second abonnement a utilisé le schéma d’événement d’entrée. Le format de l’événement remis est :

```json
{
  "myEventTypeField": "Created",
  "resource": "Users/example/Messages/1000",
  "resourceData": {
    "someDataField1": "SomeDataFieldValue"
  }
}
```

Notez que les champs d’origine ont été remis.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
