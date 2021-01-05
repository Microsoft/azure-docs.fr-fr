---
title: Référence de schéma de webhook Registry
description: Informations de référence pour la charge utile JSON liée aux requêtes de webhook dans un registre de conteneurs Azure, qui sont générées quand des webhooks sont activés pour les événements de suppression ou d’envoi (push) d’artefacts
ms.topic: article
ms.date: 03/05/2019
ms.openlocfilehash: 8354ef9db24d5825238155ac567d5d829f9b0d7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74455960"
---
# <a name="azure-container-registry-webhook-reference"></a>Référence de webhook Azure Container Registry

Vous pouvez [configurer des webhooks](container-registry-webhook.md) pour votre registre de conteneurs qui génèrent des événements lorsque certaines actions sont effectuées. Par exemple, activer des webhooks qui sont déclenchés quand une image conteneur ou un graphique Helm est envoyé vers un Registre, ou supprimé. Lorsqu’un webhook est déclenché, Azure Container Registry envoie une requête HTTP ou HTTPS contenant des informations sur l’événement à un point de terminaison que vous spécifiez. Votre point de terminaison peut ensuite traiter le webhook et agir en conséquence.

Les sections suivantes détaillent le schéma de requêtes de webhook générées par des événements pris en charge. Les sections de l’événement contient le schéma de charge utile pour le type d’événement, une charge utile de requête par exemple, et un ou plusieurs exemples de commandes qui peuvent déclencher le webhook.

Pour plus d’informations sur la configuration des webhooks pour votre registre de conteneurs Azure, consultez [Utilisation de webhooks Azure Container Registry](container-registry-webhook.md).

## <a name="webhook-requests"></a>Requêtes de webhook

### <a name="http-request"></a>Demande HTTP

Un webhook déclenché envoie une requête `POST` HTTP au point de terminaison d’URL que vous avez spécifié lorsque vous avez configuré le webhook.

### <a name="http-headers"></a>En-têtes HTTP

Les requêtes de webhook incluent un `Content-Type` de `application/json` si vous n’avez pas spécifié un en-tête personnalisé `Content-Type` pour votre webhook.

Aucun en-tête, autre que ces en-têtes personnalisés que vous pouvez avoir spécifiés pour le webhook, n’est ajouté à la requête.

## <a name="push-event"></a>Événement push

Webhook déclenché lorsqu’une image conteneur est envoyée vers un référentiel.

### <a name="push-event-payload"></a>Charge utile d’événement push

|Élément|Type|Description|
|-------------|----------|-----------|
|`id`|String|ID de l’événement de webhook.|
|`timestamp`|DateTime|Heure à laquelle l’événement de webhook a été déclenché.|
|`action`|String|Action qui a déclenché l’événement de webhook.|
|[cible](#target)|Type complexe|Cible de l’événement qui a déclenché l’événement de webhook.|
|[requête](#request)|Type complexe|Requête qui a généré l’événement de webhook.|

### <a name="target"></a><a name="target"></a>cible

|Élément|Type|Description|
|------------------|----------|-----------|
|`mediaType`|String|Type MIME de l’objet référencé.|
|`size`|Int32|Nombre d’octets du contenu. Identique au champ Longueur.|
|`digest`|String|Résumé du contenu, tel que défini par la spécification d’API du Registre V2 HTTP.|
|`length`|Int32|Nombre d’octets du contenu. Identique au champ Taille.|
|`repository`|String|Nom du référentiel.|
|`tag`|String|Nom de la balise d’image.|

### <a name="request"></a><a name="request"></a>requête

|Élément|Type|Description|
|------------------|----------|-----------|
|`id`|String|ID de la requête qui a initié l’événement.|
|`host`|String|Nom d’hôte accessible de l’extérieur de l’instance du registre, tel que spécifié par l’en-tête d’hôte HTTP sur les requêtes entrantes.|
|`method`|String|Méthode de requête qui a généré l’événement.|
|`useragent`|String|En-tête d’agent utilisateur de la requête.|

### <a name="payload-example-image-push-event"></a>Exemple de charge utile : événement push d’image

```JSON
{
  "id": "cb8c3971-9adc-488b-xxxx-43cbb4974ff5",
  "timestamp": "2017-11-17T16:52:01.343145347Z",
  "action": "push",
  "target": {
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "size": 524,
    "digest": "sha256:xxxxd5c8786bb9e621a45ece0dbxxxx1cdc624ad20da9fe62e9d25490f33xxxx",
    "length": 524,
    "repository": "hello-world",
    "tag": "v1"
  },
  "request": {
    "id": "3cbb6949-7549-4fa1-xxxx-a6d5451dffc7",
    "host": "myregistry.azurecr.io",
    "method": "PUT",
    "useragent": "docker/17.09.0-ce go/go1.8.3 git-commit/afdb6d4 kernel/4.10.0-27-generic os/linux arch/amd64 UpstreamClient(Docker-Client/17.09.0-ce \\(linux\\))"
  }
}
```

Exemple de commande [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/) qui déclenche le webhook d’événement **push** d’image :

```bash
docker push myregistry.azurecr.io/hello-world:v1
```

## <a name="chart-push-event"></a>Événement push de graphique

Webhook déclenché quand un graphique Helm est envoyé vers un référentiel.

### <a name="chart-push-event-payload"></a>Charge utile d’événement push de graphique

|Élément|Type|Description|
|-------------|----------|-----------|
|`id`|String|ID de l’événement de webhook.|
|`timestamp`|DateTime|Heure à laquelle l’événement de webhook a été déclenché.|
|`action`|String|Action qui a déclenché l’événement de webhook.|
|[cible](#helm_target)|Type complexe|Cible de l’événement qui a déclenché l’événement de webhook.|

### <a name="target"></a><a name="helm_target"></a>cible

|Élément|Type|Description|
|------------------|----------|-----------|
|`mediaType`|String|Type MIME de l’objet référencé.|
|`size`|Int32|Nombre d’octets du contenu.|
|`digest`|String|Résumé du contenu, tel que défini par la spécification d’API du Registre V2 HTTP.|
|`repository`|String|Nom du référentiel.|
|`tag`|String|Nom de balise du graphique.|
|`name`|String|Nom du chart.|
|`version`|String|Version du chart.|

### <a name="payload-example-chart-push-event"></a>Exemple de charge utile : événement push de graphique

```JSON
{
  "id": "6356e9e0-627f-4fed-xxxx-d9059b5143ac",
  "timestamp": "2019-03-05T23:45:31.2614267Z",
  "action": "chart_push",
  "target": {
    "mediaType": "application/vnd.acr.helm.chart",
    "size": 25265,
    "digest": "sha256:xxxx8075264b5ba7c14c23672xxxx52ae6a3ebac1c47916e4efe19cd624dxxxx",
    "repository": "repo",
    "tag": "wordpress-5.4.0.tgz",
    "name": "wordpress",
    "version": "5.4.0.tgz"
  }
}
```

Exemple de commande [Azure CLI](/cli/azure/acr) qui déclenche le webhook d’événement **chart_push** :

```azurecli
az acr helm push wordpress-5.4.0.tgz --name MyRegistry
```

## <a name="delete-event"></a>Supprimer un événement

Webhook déclenché quand un référentiel d’image sou un manifeste est supprimé. Non déclenché lorsqu’une balise est supprimée.

### <a name="delete-event-payload"></a>Charge utile d’événement de suppression

|Élément|Type|Description|
|-------------|----------|-----------|
|`id`|String|ID de l’événement de webhook.|
|`timestamp`|DateTime|Heure à laquelle l’événement de webhook a été déclenché.|
|`action`|String|Action qui a déclenché l’événement de webhook.|
|[cible](#delete_target)|Type complexe|Cible de l’événement qui a déclenché l’événement de webhook.|
|[requête](#delete_request)|Type complexe|Requête qui a généré l’événement de webhook.|

### <a name="target"></a><a name="delete_target"></a> cible

|Élément|Type|Description|
|------------------|----------|-----------|
|`mediaType`|String|Type MIME de l’objet référencé.|
|`digest`|String|Résumé du contenu, tel que défini par la spécification d’API du Registre V2 HTTP.|
|`repository`|String|Nom du référentiel.|

### <a name="request"></a><a name="delete_request"></a> requête

|Élément|Type|Description|
|------------------|----------|-----------|
|`id`|String|ID de la requête qui a initié l’événement.|
|`host`|String|Nom d’hôte accessible de l’extérieur de l’instance du registre, tel que spécifié par l’en-tête d’hôte HTTP sur les requêtes entrantes.|
|`method`|String|Méthode de requête qui a généré l’événement.|
|`useragent`|String|En-tête d’agent utilisateur de la requête.|

### <a name="payload-example-image-delete-event"></a>Exemple de charge utile : événement de suppression d’image

```JSON
{
    "id": "afc359ce-df7f-4e32-xxxx-1ff8aa80927b",
    "timestamp": "2017-11-17T16:54:53.657764628Z",
    "action": "delete",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "digest": "sha256:xxxxd5c8786bb9e621a45ece0dbxxxx1cdc624ad20da9fe62e9d25490f33xxxx",
      "repository": "hello-world"
    },
    "request": {
      "id": "3d78b540-ab61-4f75-xxxx-7ca9ecf559b3",
      "host": "myregistry.azurecr.io",
      "method": "DELETE",
      "useragent": "python-requests/2.18.4"
    }
  }
```

Exemples de commandes [Azure CLI](/cli/azure/acr) qui déclenchent un webhook d’événement **supprimer** :

```azurecli
# Delete repository
az acr repository delete --name MyRegistry --repository MyRepository

# Delete image
az acr repository delete --name MyRegistry --image MyRepository:MyTag
```

## <a name="chart-delete-event"></a>Événement de suppression de graphique

Webhook déclenché quand un graphique Helm ou un référentiel est supprimé. 

### <a name="chart-delete-event-payload"></a>Charge utile d’événement de suppression de graphique

|Élément|Type|Description|
|-------------|----------|-----------|
|`id`|String|ID de l’événement de webhook.|
|`timestamp`|DateTime|Heure à laquelle l’événement de webhook a été déclenché.|
|`action`|String|Action qui a déclenché l’événement de webhook.|
|[cible](#chart_delete_target)|Type complexe|Cible de l’événement qui a déclenché l’événement de webhook.|

### <a name="target"></a><a name="chart_delete_target"></a> cible

|Élément|Type|Description|
|------------------|----------|-----------|
|`mediaType`|String|Type MIME de l’objet référencé.|
|`size`|Int32|Nombre d’octets du contenu.|
|`digest`|String|Résumé du contenu, tel que défini par la spécification d’API du Registre V2 HTTP.|
|`repository`|String|Nom du référentiel.|
|`tag`|String|Nom de balise du graphique.|
|`name`|String|Nom du chart.|
|`version`|String|Version du chart.|

### <a name="payload-example-chart-delete-event"></a>Exemple de charge utile : événement de suppression de graphique

```JSON
{
  "id": "338a3ef7-ad68-4128-xxxx-fdd3af8e8f67",
  "timestamp": "2019-03-06T00:10:48.1270754Z",
  "action": "chart_delete",
  "target": {
    "mediaType": "application/vnd.acr.helm.chart",
    "size": 25265,
    "digest": "sha256:xxxx8075264b5ba7c14c23672xxxx52ae6a3ebac1c47916e4efe19cd624dxxxx",
    "repository": "repo",
    "tag": "wordpress-5.4.0.tgz",
    "name": "wordpress",
    "version": "5.4.0.tgz"
  }
}
```

Exemple de commande [Azure CLI](/cli/azure/acr) qui déclenche le webhook d’événement **chart_delete** :

```azurecli
az acr helm delete wordpress --version 5.4.0 --name MyRegistry
```

## <a name="next-steps"></a>Étapes suivantes

[Utilisation de webhooks Azure Container Registry](container-registry-webhook.md)