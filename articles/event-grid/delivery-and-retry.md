---
title: Distribution et nouvelle tentative de distribution avec Azure Event Grid
description: Décrit comment Azure Event Grid distribue des événements et gère les messages qui n’ont pas été distribués.
ms.topic: conceptual
ms.date: 10/29/2020
ms.openlocfilehash: 51473cf457a1c713e6694edd23c344be8c4d439e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463242"
---
# <a name="event-grid-message-delivery-and-retry"></a>Distribution et nouvelle tentative de distribution de messages avec Azure Grid

Cet article décrit comment Azure Event Grid gère les événements en l’absence d’accusé de réception d’une distribution.

Event Grid assure une distribution fiable. Il distribue chaque message **au moins une fois** pour chaque abonnement. Les événements sont envoyés immédiatement au point de terminaison inscrit de chaque abonnement. Si un point de terminaison n’accuse pas réception d’un événement, Event Grid effectue une nouvelle tentative de distribution.

> [!NOTE]
> Event Grid ne garantit pas l’ordre de distribution des événements, de sorte que l’abonné peut les recevoir dans le désordre. 

## <a name="batched-event-delivery"></a>Livraison d’événements par lot

Par défaut, Event Grid envoie chaque événement individuellement aux abonnés. L’abonné reçoit un tableau ne comprenant qu’un seul événement. Vous pouvez configurer Event Grid pour que les événements soient livrés par lot afin d’améliorer les performances HTTP dans les scénarios à débit élevé.

La livraison par lot a deux paramètres :

* **Nombre maximum d’événements par lot** : nombre maximum d’événements qu’Event Grid livrera par lot. Ce nombre ne sera jamais dépassé, mais moins d’événements peuvent être livrés si aucun autre événement n’est disponible au moment de la publication. Event Grid ne retarde pas la livraison des événements pour créer un lot si moins d’événements sont disponibles. Doit être compris entre 1 et 5 000.
* **Taille de lot préférée en kilo-octets** : plafond cible pour la taille de lot en kilo-octets. Comme pour le nombre maximum d’événements, la taille du lot peut être plus petite si aucun autre événement n’est disponible au moment de la publication. Il est possible qu’un lot soit plus grand que la taille de lot préférée *si* un événement unique est plus volumineux que la taille préférée. Par exemple, si la taille préférée est de 4 Ko et qu’un événement de 10 Ko est envoyé (push) à Event Grid, l’événement de 10 Ko sera tout de même livré dans son propre lot plutôt que d’être abandonné.

La livraison en lot est configurée sur la base d’un abonnement par événement via le portail, l’interface CLI, PowerShell ou les Kits de développement logiciel (SDK).

### <a name="azure-portal"></a>Portail Azure : 
![Paramètres de livraison en lot](./media/delivery-and-retry/batch-settings.png)

### <a name="azure-cli"></a>Azure CLI
Lorsque vous créez un abonnement aux événements, utilisez les paramètres suivants : 

- **max-events-per-batch** : nombre maximum d’événements dans un lot. Doit être un nombre compris entre 1 et 5000.
- **preferred-batch-size-in-kilobytes**  taille de lot préférée en kilo-octets. Doit être un nombre compris entre 1 et 1024.

```azurecli
storageid=$(az storage account show --name <storage_account_name> --resource-group <resource_group_name> --query id --output tsv)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --resource-id $storageid \
  --name <event_subscription_name> \
  --endpoint $endpoint \
  --max-events-per-batch 1000 \
  --preferred-batch-size-in-kilobytes 512
```

Pour plus d’informations sur l’utilisation d’Azure CLI avec Event Grid, consultez [Acheminer des événements de stockage vers un point de terminaison web avec Azure CLI](../storage/blobs/storage-blob-event-quickstart.md).

## <a name="retry-schedule-and-duration"></a>Planification d’un nouvel essai et durée

Quand EventGrid reçoit une erreur lors d’une tentative de remise d’événement, EventGrid décide s’il doit retenter la remise ou mettre l’événement dans la file d’attente de lettres mortes ou supprimer l’événement en fonction du type d’erreur. 

Si l’erreur retournée par le point de terminaison abonné est liée à une erreur de configuration qui ne peut pas être corrigée avec des nouvelles tentatives (par exemple, si le point de terminaison est supprimé), EventGrid met l’événement dans la file d’attente de lettres mortes ou supprime l’événement si la file d’attente de lettres mortes n’est pas configurée.

Voici les types de points de terminaison pour lesquels aucune nouvelle tentative ne se produit :

| Type de point de terminaison | Codes d’erreur |
| --------------| -----------|
| Ressources Azure | 400 Requête incorrecte, 413 Entité de la requête trop grande, 403 Interdit | 
| webhook | 400 Requête incorrecte, 413 Entité de la requête trop grande, 403 Interdit, 404 Introuvable, 401 Non autorisé |
 
> [!NOTE]
> Si la file d’attente de lettres mortes n’est pas configurée pour le point de terminaison, les événements sont supprimés lorsque les erreurs ci-dessus se produisent. Par conséquent, envisagez de configurer la file d’attente de lettres mortes si vous ne souhaitez pas que ces types d’événements soient supprimés.

Si l’erreur retournée par le point de terminaison abonné ne figure pas dans la liste ci-dessus, EventGrid effectue la nouvelle tentative à l’aide des stratégies décrites ci-dessous :

Event Grid attend une réponse pendant 30 secondes après la distribution d’un message. Après 30 secondes, si le point de terminaison n’a pas répondu, le message est mis en file d’attente en vue d’une nouvelle tentative. Event Grid utilise une stratégie de nouvelle tentative d’interruption exponentielle pour la distribution des événements. Dans la mesure du possible, Event Grid tente une nouvelle livraison selon la planification suivante :

- 10 secondes
- 30 secondes
- 1 minute
- 5 minutes
- 10 minutes
- 30 minutes
- 1 heure
- Toutes les heures jusqu’à 24 heures

Si le point de terminaison répond dans les 3 minutes, Event Grid tente de supprimer l’événement de la file d’attente de nouvelle tentative dans la mesure du possible, mais des doublons peuvent toujours être reçus.

Event Grid ajoute une légère randomisation à toutes les étapes de nouvelle tentative et peut ignorer de façon opportuniste certaines nouvelles tentatives si un point de terminaison est systématiquement non sain, inactif pendant une longue période ou semble être surchargé.

Pour un comportement déterministe, définissez la durée de l’événement de durée de vie et de nombre maximum de tentatives de remise dans les [stratégies de nouvelle tentative d’abonnement](manage-event-delivery.md).

Par défaut, Event Grid fait expirer tous les événements qui ne sont pas distribués dans les 24 heures. Vous pouvez [personnaliser la stratégie de nouvelle tentative](manage-event-delivery.md) lors de la création d’un abonnement à un événement. Vous fournissez le nombre maximal de tentatives de remise (par défaut, 30) et la durée de vie de l’événement (par défaut, 1 440 minutes).

## <a name="delayed-delivery"></a>Livraison retardée

En cas d'échec de livraison d'un point de terminaison, Event Grid commence à retarder la livraison et retente les événements sur ce point de terminaison. Par exemple, si les dix premiers événements publiés sur un point de terminaison échouent, Event Grid supposera que le point de terminaison rencontre des problèmes et retardera toutes les tentatives suivantes *et les nouvelles* livraisons pendant un certain laps de temps, parfois même pendant plusieurs heures.

La livraison retardée a pour objectif de protéger les points de terminaison non sains, ainsi que le système Event Grid. Sans temporisation et retard de livraison sur les points de terminaison non sains, la stratégie de nouvelle tentative d'Event Grid peut aisément saturer un système.

## <a name="dead-letter-events"></a>Événements de lettres mortes
Lorsque Event Grid ne peut pas remettre un événement dans un laps de temps donné ou après avoir essayé de remettre l’événement un certain nombre de fois, il peut envoyer l’événement non remis à un compte de stockage. Ce processus est appelé **mise en file d’attente de lettres mortes**. Event Grid met un événement en file d’attente de lettres mortes lorsque **l’une des conditions suivantes** est remplie. 

- L’événement n’est pas remis dans la période de **durée de vie**. 
- Le **nombre de tentatives** de livraison de l’événement a dépassé la limite.

Si l’une des conditions est remplie, l’événement est abandonné ou mis en file d’attente de lettres mortes.  Par défaut, Event Grid n’active pas cette fonctionnalité. Pour l’activer, vous devez spécifier le compte de stockage dans lequel les événements non remis seront conservés au moment de créer l’abonnement aux événements. Les événements sont extraits de ce compte de stockage pour résoudre les remises.

Event Grid envoie un événement à l’emplacement des lettres mortes lorsqu’il a effectué toutes ses nouvelles tentatives. Si Event Grid reçoit un code de réponse 400 (requête incorrecte) ou 413 (entité de requête trop grande), il envoie immédiatement l’événement au point de terminaison des lettres mortes. Ces codes de réponse indiquent que la diffusion de l’événement va échouer.

L’expiration de la durée de vie est vérifiée uniquement lors de la prochaine tentative de livraison planifiée. Par conséquent, même si la durée de vie expire avant la prochaine tentative de livraison planifiée, l’expiration de l’événement est vérifiée uniquement au moment de la prochaine livraison, puis devient lettre morte par la suite. 

Un délai de cinq minutes s’écoule entre la dernière tentative de remise d’un événement et le moment de sa remise à l’emplacement des lettres mortes. Ce décalage est destiné à réduire le nombre d’opérations de stockage d’objets blob. Si l’emplacement des lettres mortes est indisponible pendant quatre heures, l’événement est abandonné.

Avant de définir l’emplacement des lettres mortes, vous devez disposer d’un compte de stockage avec un conteneur. Vous devez indiquer le point de terminaison de ce conteneur au moment de créer l’abonnement aux événements. Le point de terminaison se présente sous la forme suivante : `/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>/blobServices/default/containers/<container-name>`

Vous souhaiterez peut-être être averti lorsqu’un événement a été envoyé à l’emplacement des lettres mortes. Pour répondre aux événements non remis à l’aide d’Event Grid, [créez un abonnement aux événements](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) pour le stockage Blob de lettres mortes. Chaque fois que votre stockage Blob de lettres mortes reçoit un événement non remis, Event Grid notifie votre gestionnaire. Le gestionnaire répond par les mesures que vous voulez prendre pour réconcilier les événements non remis. Pour savoir comment configurer un emplacement de lettres mortes et des stratégies de nouvelle tentative, consultez [Lettres mortes et stratégies de nouvelle tentative](manage-event-delivery.md).

## <a name="delivery-event-formats"></a>Formats d’événement de livraison
Cette section fournit des exemples d’événements et des événements de lettres mortes dans différents formats de schéma de livraison (schéma Event Grid, schéma CloudEvents 1.0 et schéma personnalisé). Pour plus d’informations sur ces formats, consultez les articles [Schéma Event Grid](event-schema.md) et [Schéma CloudEvents v1.0](cloud-event-schema.md). 

### <a name="event-grid-schema"></a>Schéma Event Grid

#### <a name="event"></a>Événement 
```json
{
    "id": "93902694-901e-008f-6f95-7153a806873c",
    "eventTime": "2020-08-13T17:18:13.1647262Z",
    "eventType": "Microsoft.Storage.BlobCreated",
    "dataVersion": "",
    "metadataVersion": "1",
    "topic": "/subscriptions/000000000-0000-0000-0000-00000000000000/resourceGroups/rgwithoutpolicy/providers/Microsoft.Storage/storageAccounts/myegteststgfoo",
    "subject": "/blobServices/default/containers/deadletter/blobs/myBlobFile.txt",    
    "data": {
        "api": "PutBlob",
        "clientRequestId": "c0d879ad-88c8-4bbe-8774-d65888dc2038",
        "requestId": "93902694-901e-008f-6f95-7153a8000000",
        "eTag": "0x8D83FACDC0C3402",
        "contentType": "text/plain",
        "contentLength": 0,
        "blobType": "BlockBlob",
        "url": "https://myegteststgfoo.blob.core.windows.net/deadletter/myBlobFile.txt",
        "sequencer": "00000000000000000000000000015508000000000005101c",
        "storageDiagnostics": { "batchId": "cfb32f79-3006-0010-0095-711faa000000" }
    }
}
```

#### <a name="dead-letter-event"></a>Événement de lettres mortes

```json
{
    "id": "93902694-901e-008f-6f95-7153a806873c",
    "eventTime": "2020-08-13T17:18:13.1647262Z",
    "eventType": "Microsoft.Storage.BlobCreated",
    "dataVersion": "",
    "metadataVersion": "1",
    "topic": "/subscriptions/0000000000-0000-0000-0000-000000000000000/resourceGroups/rgwithoutpolicy/providers/Microsoft.Storage/storageAccounts/myegteststgfoo",
    "subject": "/blobServices/default/containers/deadletter/blobs/myBlobFile.txt",    
    "data": {
        "api": "PutBlob",
        "clientRequestId": "c0d879ad-88c8-4bbe-8774-d65888dc2038",
        "requestId": "93902694-901e-008f-6f95-7153a8000000",
        "eTag": "0x8D83FACDC0C3402",
        "contentType": "text/plain",
        "contentLength": 0,
        "blobType": "BlockBlob",
        "url": "https://myegteststgfoo.blob.core.windows.net/deadletter/myBlobFile.txt",
        "sequencer": "00000000000000000000000000015508000000000005101c",
        "storageDiagnostics": { "batchId": "cfb32f79-3006-0010-0095-711faa000000" }
    },

    "deadLetterReason": "MaxDeliveryAttemptsExceeded",
    "deliveryAttempts": 1,
    "lastDeliveryOutcome": "NotFound",
    "publishTime": "2020-08-13T17:18:14.0265758Z",
    "lastDeliveryAttemptTime": "2020-08-13T17:18:14.0465788Z" 
}
```

### <a name="cloudevents-10-schema"></a>Schéma CloudEvents 1.0

#### <a name="event"></a>Événement

```json
{
    "id": "caee971c-3ca0-4254-8f99-1395b394588e",
    "source": "mysource",
    "dataversion": "1.0",
    "subject": "mySubject",
    "type": "fooEventType",
    "datacontenttype": "application/json",
    "data": {
        "prop1": "value1",
        "prop2": 5
    }
}
```

#### <a name="dead-letter-event"></a>Événement de lettres mortes

```json
{
    "id": "caee971c-3ca0-4254-8f99-1395b394588e",
    "source": "mysource",
    "dataversion": "1.0",
    "subject": "mySubject",
    "type": "fooEventType",
    "datacontenttype": "application/json",
    "data": {
        "prop1": "value1",
        "prop2": 5
    },

    "deadletterreason": "MaxDeliveryAttemptsExceeded",
    "deliveryattempts": 1,
    "lastdeliveryoutcome": "NotFound",
    "publishtime": "2020-08-13T21:21:36.4018726Z",
}
```

### <a name="custom-schema"></a>Schéma personnalisé

#### <a name="event"></a>Événement

```json
{
    "prop1": "my property",
    "prop2": 5,
    "myEventType": "fooEventType"
}

```

#### <a name="dead-letter-event"></a>Événement de lettres mortes
```json
{
    "id": "8bc07e6f-0885-4729-90e4-7c3f052bd754",
    "eventTime": "2020-08-13T18:11:29.4121391Z",
    "eventType": "myEventType",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/00000000000-0000-0000-0000-000000000000000/resourceGroups/rgwithoutpolicy/providers/Microsoft.EventGrid/topics/myCustomSchemaTopic",
    "subject": "subjectDefault",
  
    "deadLetterReason": "MaxDeliveryAttemptsExceeded",
    "deliveryAttempts": 1,
    "lastDeliveryOutcome": "NotFound",
    "publishTime": "2020-08-13T18:11:29.4121391Z",
    "lastDeliveryAttemptTime": "2020-08-13T18:11:29.4277644Z",
  
    "data": {
        "prop1": "my property",
        "prop2": 5,
        "myEventType": "fooEventType"
    }
}
```


## <a name="message-delivery-status"></a>État de distribution du message

Event Grid utilise les codes de réponse HTTP pour accuser réception des événements. 

### <a name="success-codes"></a>Codes de réussite

Event Grid considère **uniquement** les codes de réponse HTTP suivants en tant que livraisons réussies. Tous les autres codes d’état sont considérés en tant que livraisons ayant échoué et font l'objet de nouvelles tentatives ou de lettres mortes, le cas échéant. Lorsqu'il reçoit un code d'état réussi, Event Grid considère la livraison comme terminée.

- 200 OK
- 201 Créé
- 202 Accepté
- 203 Informations ne faisant pas autorité
- 204 Pas de contenu

### <a name="failure-codes"></a>Codes d’échec

Tous les codes ne figurant pas dans les codes ci-dessus (200 à 204) sont considérés comme ayant échoué et feront l’objet de nouvelles tentatives (le cas échéant). Certains sont associés à des stratégies de nouvelle tentative spécifiques (voir ci-dessous). Toutes les autres suivent le modèle de temporisation exponentielle standard. Il est important de garder à l’esprit qu’en raison de la nature hautement parallélisée de l’architecture d'Event Grid, le comportement d'une nouvelle tentative n'est pas déterministe. 

| Code d’état | Comportement pour les nouvelles tentatives |
| ------------|----------------|
| 400 Demande incorrecte | Aucune nouvelle tentative |
| 401 Non autorisé | Nouvelle tentative au bout de 5 minutes pour les points de terminaison des ressources Azure |
| 403 Interdit | Aucune nouvelle tentative |
| 404 Introuvable | Nouvelle tentative au bout de 5 minutes pour les points de terminaison des ressources Azure |
| 408 Délai d’expiration de la requête | Nouvelle tentative après 2 minutes ou plus |
| 413 Entité de demande trop grande | Aucune nouvelle tentative |
| 503 Service indisponible | Nouvelle tentatives après 30 secondes ou plus |
| Tous les autres | Nouvelle tentatives après 10 secondes ou plus |


## <a name="next-steps"></a>Étapes suivantes

* Pour afficher l’état des remises des événements, consultez [Surveiller la remise des messages Event Grid](monitor-event-delivery.md).
* Pour personnaliser les options de diffusion d’événements, consultez [Stratégies de lettres mortes et de nouvelles tentatives](manage-event-delivery.md).
