---
title: Schéma CloudEvents v1.0 avec Azure Event Grid
description: Décrit comment utiliser le schéma CloudEvents v1.0 pour les événements dans Azure Event Grid. Le service prend en charge les événements dans l’implémentation JSON de CloudEvents.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: d17e92c28784ca31f3c9809c93e885b22c6a38d0
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91324176"
---
# <a name="cloudevents-v10-schema-with-azure-event-grid"></a>Schéma CloudEvents v1.0 avec Azure Event Grid

En plus de son [schéma d’événement par défaut](event-schema.md), Azure Event Grid prend en charge en mode natif les événements dans l’[implémentation JSON de CloudEvents v1.0](https://github.com/cloudevents/spec/blob/v1.0/json-format.md) et la [liaison de protocole HTTP](https://github.com/cloudevents/spec/blob/v1.0/http-protocol-binding.md). [CloudEvents](https://cloudevents.io/) est une [spécification ouverte](https://github.com/cloudevents/spec/blob/v1.0/spec.md) qui décrit les données d’événement.

CloudEvents simplifie l’interopérabilité grâce à un schéma d’événement commun pour la publication et la consommation des événements basés sur le cloud. Ce schéma permet l’utilisation d’outils uniformes, la mise en œuvre de méthodes de routage et de gestion des événements standard, ainsi que de méthodes universelles pour désérialiser le schéma d’événement externe. Avec un schéma commun, vous pouvez plus facilement intégrer le travail entre plusieurs plateformes.

Plusieurs [collaborateurs](https://github.com/cloudevents/spec/blob/master/community/contributors.md), dont Microsoft, travaillent à l’élaboration de CloudEvents par le biais de la [Cloud Native Computing Foundation](https://www.cncf.io/). Il est actuellement disponible en version 1.0.

Cet article décrit le schéma CloudEvents avec Event Grid.

## <a name="sample-event-using-cloudevents-schema"></a>Exemple d'événement utilisant le schéma CloudEvents

Voici un exemple d’événement Stockage Blob Azure dans le format CloudEvents :

``` JSON
{
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",  
    "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2019-11-18T15:13:39.4589254Z",
    "subject": "blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "dataschema": "#",
    "data": {
        "api": "PutBlockList",
        "clientRequestId": "4c5dd7fb-2c48-4a27-bb30-5361b5de920a",
        "requestId": "9aeb0fdf-c01e-0131-0922-9eb549000000",
        "eTag": "0x8D76C39E4407333",
        "contentType": "image/png",
        "contentLength": 30699,
        "blobType": "BlockBlob",
        "url": "https://gridtesting.blob.core.windows.net/testcontainer/{new-file}",
        "sequencer": "000000000000000000000000000099240000000000c41c18",
        "storageDiagnostics": {
            "batchId": "681fe319-3006-00a8-0022-9e7cde000000"
        }
    }
}
```

Une description détaillée des champs disponibles, de leurs types et définitions dans CloudEvents v1.0 est [disponible ici](https://github.com/cloudevents/spec/blob/v1.0/spec.md#required-attributes).

Les valeurs des en-têtes pour les événements remis dans le schéma CloudEvents et le schéma Event Grid sont identiques, à l’exception de `content-type`. Pour le schéma CloudEvents, cette valeur d’en-tête est `"content-type":"application/cloudevents+json; charset=utf-8"`. Pour le schéma Event Grid, cette valeur d’en-tête est `"content-type":"application/json; charset=utf-8"`.

## <a name="event-grid-for-cloudevents"></a>Event Grid pour CloudEvents

Vous pouvez utiliser Event Grid pour l’entrée et la sortie des événements dans le schéma CloudEvents. Vous pouvez utiliser CloudEvents pour des événements système, tels que les événements Stockage Blob et les événements IoT Hub, et des événements personnalisés. Il peut aussi transformer ces événements dans un sens ou dans l’autre.


| Schéma d’entrée       | Schéma de sortie
|--------------------|---------------------
| Format CloudEvents | Format CloudEvents
| Format Event Grid  | Format CloudEvents
| Format Event Grid  | Format Event Grid

Pour tous les schémas d’événement, Event Grid requiert une validation au moment de la publication sur une rubrique de grille d’événement et de la création d’un abonnement aux événements. Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](security-authentication.md).

## <a name="next-steps"></a>Étapes suivantes
Consultez [Utiliser le schéma CloudEvents v1.0 avec Event Grid](cloudevents-schema.md).  
