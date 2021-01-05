---
title: Azure Container Registry en tant que source Event Grid
description: Décrit les propriétés qui sont fournies pour les événements Container Registry avec Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: d216fe88ee6aaad33fbbe3b93b8c4f8a6e952a71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86113715"
---
# <a name="azure-container-registry-as-an-event-grid-source"></a>Azure Container Registry en tant que source Event Grid

Cet article fournit les propriétés et le schéma des événements Container Registry. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md).

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

Azure Container Registry émet les types d’événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.ContainerRegistry.ImagePushed | Déclenché lorsqu’une image est envoyée. |
| Microsoft.ContainerRegistry.ImageDeleted | Déclenché lorsqu’une image est supprimée. |
| Microsoft.ContainerRegistry.ChartPushed | Déclenché lors de l’envoi d’un chart Helm. |
| Microsoft.ContainerRegistry.ChartDeleted | Déclenché lors de la suppression d’un chart Helm. |

### <a name="example-event"></a>Exemple d’événement

L’exemple suivant montre le schéma d’un événement d’image envoyée (push) : 

```json
[{
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "aci-helloworld:v1",
  "eventType": "ImagePushed",
  "eventTime": "2018-04-25T21:39:47.6549614Z",
  "data": {
    "id": "31c51664-e5bd-416a-a5df-e5206bc47ed0",
    "timestamp": "2018-04-25T21:39:47.276585742Z",
    "action": "push",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "size": 3023,
      "digest": "sha256:213bbc182920ab41e18edc2001e06abcca6735d87782d9cef68abd83941cf0e5",
      "length": 3023,
      "repository": "aci-helloworld",
      "tag": "v1"
    },
    "request": {
      "id": "7c66f28b-de19-40a4-821c-6f5f6c0003a4",
      "host": "demo.azurecr.io",
      "method": "PUT",
      "useragent": "docker/18.03.0-ce go/go1.9.4 git-commit/0520e24 os/windows arch/amd64 UpstreamClient(Docker-Client/18.03.0-ce \\\\(windows\\\\))"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

Le schéma de l’événement de suppression d’une image est similaire :

```json
[{
  "id": "f06e3921-301f-42ec-b368-212f7d5354bd",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "aci-helloworld",
  "eventType": "ImageDeleted",
  "eventTime": "2018-04-26T17:56:01.8211268Z",
  "data": {
    "id": "f06e3921-301f-42ec-b368-212f7d5354bd",
    "timestamp": "2018-04-26T17:56:00.996603117Z",
    "action": "delete",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "digest": "sha256:213bbc182920ab41e18edc2001e06abcca6735d87782d9cef68abd83941cf0e5",
      "repository": "aci-helloworld"
    },
    "request": {
      "id": "aeda5b99-4197-409f-b8a8-ff539edb7de2",
      "host": "demo.azurecr.io",
      "method": "DELETE",
      "useragent": "python-requests/2.18.4"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

Le schéma pour un événement de chart envoyé est similaire au schéma pour un événement d’image envoyé, mais il n’inclut pas d’objet de demande :

```json
[{
  "id": "ea3a9c28-5b17-40f6-a500-3f02b6829277",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "mychart:1.0.0",
  "eventType": "Microsoft.ContainerRegistry.ChartPushed",
  "eventTime": "2019-03-12T22:16:31.5164086Z",
  "data": {
    "id":"ea3a9c28-5b17-40f6-a500-3f02b682927",
    "timestamp":"2019-03-12T22:16:31.0087496+00:00",
    "action":"chart_push",
    "target":{
      "mediaType":"application/vnd.acr.helm.chart",
      "size":25265,
      "digest":"sha256:7f060075264b5ba7c14c23672698152ae6a3ebac1c47916e4efe19cd624d5fab",
      "repository":"repo",
      "tag":"mychart-1.0.0.tgz",
      "name":"mychart",
      "version":"1.0.0"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

Le schéma pour un événement de chart supprimé est similaire au schéma pour un événement d’image supprimée, mais il n’inclut pas d’objet de demande :

```json
[{
  "id": "39136b3a-1a7e-416f-a09e-5c85d5402fca",
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerRegistry/registries/<name>",
  "subject": "mychart:1.0.0",
  "eventType": "Microsoft.ContainerRegistry.ChartDeleted",
  "eventTime": "019-03-12T22:42:08.7034064Z",
  "data": {
    "id":"ea3a9c28-5b17-40f6-a500-3f02b682927",
    "timestamp":"2019-03-12T22:42:08.3783775+00:00",
    "action":"chart_delete",
    "target":{
      "mediaType":"application/vnd.acr.helm.chart",
      "size":25265,
      "digest":"sha256:7f060075264b5ba7c14c23672698152ae6a3ebac1c47916e4efe19cd624d5fab",
      "repository":"repo",
      "tag":"mychart-1.0.0.tgz",
      "name":"mychart",
      "version":"1.0.0"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="event-properties"></a>Propriétés d’événement

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement de stockage Blob. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| id | string | ID de l’événement. |
| timestamp | string | Heure à laquelle l’événement s’est produit. |
| action | string | Action qui englobe l’événement fourni. |
| target | object | Cible de l’événement. |
| request | object | Requête ayant généré l’événement. |

L’objet cible comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| mediaType | string | Type MIME de l’objet référencé. |
| taille | entier | Nombre d’octets du contenu. Identique au champ Longueur. |
| digest | string | Résumé du contenu, tel que défini par la spécification d’API du Registre V2 HTTP. |
| length | entier | Nombre d’octets du contenu. Identique au champ Taille. |
| repository | string | Nom du référentiel. |
| tag | string | Nom de la balise. |
| name | string | Nom du chart. |
| version | string | Version du chart. |

L’objet de requête comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| id | string | ID de la requête qui a initié l’événement. |
| addr | string | Adresse IP ou nom d’hôte, et éventuellement port de la connexion cliente qui a lancé l’événement. Cette valeur correspond à RemoteAddr dans la requête HTTP standard. |
| host | string | Nom d’hôte accessible de l’extérieur pour l’instance du registre, tel que spécifié par l’en-tête d’hôte HTTP dans les requêtes entrantes. |
| method | string | Méthode de requête qui a généré l’événement. |
| useragent | string | En-tête d’agent utilisateur de la requête. |

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures
|Intitulé |Description  |
|---------|---------|
| [Démarrage rapide : Envoyer des événements de registre de conteneurs](../container-registry/container-registry-event-grid-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Montre comment utiliser Azure CLI pour envoyer des événements de registre de conteneurs. |


## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
