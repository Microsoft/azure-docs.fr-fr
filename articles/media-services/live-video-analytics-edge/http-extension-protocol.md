---
title: Protocole d’extension HTTP – Azure
description: Dans cet article, vous allez apprendre à utiliser le protocole d’extension HTTP pour envoyer des messages entre le module Live Video Analytics et votre module d’IA ou de validation croisée (CV).
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 52c98231780a2776f4ff67992f29b247eccb8bc2
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97399143"
---
# <a name="http-extension-protocol"></a>Protocole d’extension HTTP

Live Video Analytics sur IoT Edge vous permet d’étendre les fonctionnalités de traitement de graphe multimédia via un [nœud d’extension de graphe](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/media-graph-extension-concept?branch=release-lva-dec-update). Si vous utilisez le processeur d’extension HTTP comme nœud d’extension, la communication entre le module Live Video Analytics et votre module d’IA ou de validation croisée (CV) se fait sur HTTP

Dans cet article, vous allez apprendre à utiliser le protocole d’extension HTTP pour envoyer des messages entre le module Live Video Analytics et votre module d’IA ou de validation croisée (CV). 

Le contrat HTTP est défini entre les deux composants suivants :

* Le serveur HTTP
* Le module Live Video Analytics sur IoT Edge, qui agit en tant que client HTTP

## <a name="request"></a>Requête

Les requêtes provenant du module Live Video Analytics et adressées à votre serveur HTTP sont les suivantes :

|Clé|Valeur|
|---|---|
|POST|`https://hostname/optional-path?optional-query`|
|Acceptation|application/json,  */*|
|Autorisation|Basic, Digest, Bearer (par le biais de la prise en charge des en-têtes personnalisés)|
|Content-Type|image/jpeg<br/>image/png<br/>image/bmp<br/>image/x-raw|
|Content-Length : Longueur du corps, en octets|
|User-Agent.|Azure Media Services|
|body|Octets d’image, encodés en binaire dans l’un des types de contenu pris en charge.|

### <a name="example"></a>Exemple

```
POST http://localhost:8080/inference HTTP/1.1
Host: localhost:8080
x-ms-client-request-id: d6050cd4-c9f2-42d3-9adc-53ba7e440f17
Content-Type: image/bmp
Content-Length: 519222

(Image Binary Content)
```

## <a name="response"></a>response

Les réponses de votre module au module Live Video Analytics doivent être les suivantes :

|Clé|Valeur|
|---|---|
|Codes d’état|200 OK - Résultats d’inférence trouvés<br/>204 Pas de contenu - Aucun contenu trouvé par l’IA<br/>400 Demande incorrecte - Inattendue<br/>500 Erreur interne du serveur - Inattendue<br/>503 Serveur occupé – AMS se retire en fonction de l’en-tête « Retry-after » ou d’un intervalle de temps par défaut en cas d’en-tête non prédéfini.|
|Content-Type|application/json|
|Content-Length|Longueur du corps, en octets|
|body|Objet JSON avec une seule propriété « inferences ».|

### <a name="example"></a>Exemple

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 468
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 17 Apr 2020 04:44:01 GMT

{
  "inferences": [
    {
      "type": "entity",
      "entity": {
        "tag": { "value": "car", "confidence": 0.9048132 },
        "box": { "l": 0.42681578, "t": 0.47660735, "w": 0.019501392, "h": 0.020954132 }
      }
    },
    {
      "type": "entity",
      "entity": {
        "tag": { "value": "car", "confidence": 0.8953932 },
        "box": { "l": 0.55083525, "t": 0.4843858, "w": 0.046550274, "h": 0.046502113 }
      }
    }    
  ]
}
```

Il est fortement recommandé de retourner les réponses à l’aide de documents JSON valides, conformes au schéma pré-établi défini comme étant le [modèle d’objet du schéma des métadonnées d’inférence](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/inference-metadata-schema?branch=release-lva-dec-update). L’interopérabilité avec d’autres composants et les futures fonctionnalités ajoutées au module Live Video Analytics est ainsi mieux garantie.

Si votre module retourne une réponse dans laquelle le type de contenu n’est pas « application/JSON », Live Video Analytics encode le message sous forme de contenu en base 64 et le sérialise en tant que charge utile JSON opaque.

Si votre module retourne une réponse avec le type de contenu « application/JSON », mais que le schéma JSON ne suit pas le schéma des métadonnées d’inférence décrit ci-dessous, la charge utile du message est transférée par le biais du pipeline, mais l’interopérabilité est réduite. Pour des informations détaillées et à jour sur le schéma des métadonnées d’inférence, consultez [cette page](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/inference-metadata-schema?branch=release-lva-dec-update).

> [!NOTE]
> Si votre module ne produit aucun résultat, il doit retourner le code d’état HTTP 204 (Pas de contenu) avec un corps de réponse vide. Live Video Analytics va l’interpréter comme un résultat vide et ne va pas transférer l’événement dans le pipeline.


## <a name="next-steps"></a>Étapes suivantes

[Protocole d’extension gRPC](./grpc-extension-protocol.md)
