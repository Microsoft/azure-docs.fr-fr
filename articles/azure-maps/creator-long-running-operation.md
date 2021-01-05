---
title: API d’opération de longue durée Azure Maps
description: Découvrez le traitement en arrière-plan asynchrone de longue durée dans Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: f5fb7c8059c8b98e8ec514a4159e96f48db7b1ad
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906197"
---
# <a name="creator-preview-long-running-operation-api"></a>API d’opérations de longue durée Creator (préversion)

> [!IMPORTANT]
> Les services Azure Maps Creator sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Certaines API Azure Maps utilisent un [modèle de demande-réponse asynchrone](/azure/architecture/patterns/async-request-reply). Ce modèle permet à Azure Maps de fournir des services hautement disponibles et réactifs. Cet article explique l’implémentation spécifique d’Azure Map du traitement en arrière-plan asynchrone de longue durée.

## <a name="submitting-a-request"></a>Envoi d’une demande

Une application cliente démarre une opération de longue durée via un appel synchrone à une API HTTP. En général, cet appel présente la forme d’une requête HTTP POST. Quand une charge de travail asynchrone est correctement créée, l’API retourne un code d’état `202` HTTP, indiquant que la requête a été acceptée. Cette réponse contient un en-tête `Location` pointant vers un point de terminaison que le client peut sonder pour vérifier l’état de l’opération de longue durée.

### <a name="example-of-a-success-response"></a>Exemple de réponse de réussite

```HTTP
Status: 202 Accepted
Location: https://atlas.microsoft.com/service/operations/{operationId}

```

Si l’appel n’est pas validé, l’API retourne plutôt une réponse HTTP `400` pour une demande incorrecte. Le corps de la réponse fournit au client des informations supplémentaires sur la raison pour laquelle la demande n’est pas valide.

### <a name="monitoring-the-operation-status"></a>Surveillance de l’état de l’opération

Le point de terminaison d’emplacement fourni dans les en-têtes de réponse acceptée peut être sondé pour vérifier l’état de l’opération de longue durée. Le corps de la réponse de la demande d’état de l’opération contient toujours les propriétés `status` et `createdDateTime`. La propriété `status` indique l’état actuel de l’opération de longue durée. Les états possibles sont `"NotStarted"`, `"Running"`, `"Succeeded"` et `"Failed"`. La propriété `createdDateTime` indique l’heure de la demande initiale de démarrage de l’opération de longue durée. Quand l’état est `"NotStarted"` ou `"Running"`, un en-tête `Retry-After` est également fourni avec la réponse. L’en-tête `Retry-After`, mesuré en secondes, permet de déterminer à quel moment le prochain appel de sondage à l’API d’état de l’opération doit être effectué.

### <a name="example-of-running-a-status-response"></a>Exemple d’exécution d’une réponse d’état

```HTTP
Status: 200 OK
Retry-After: 30
{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Running"
}
```

## <a name="handling-operation-completion"></a>Gestion de la fin de l’opération

Une fois l’opération de longue durée terminée, l’état de la réponse est `"Succeeded"` ou `"Failed"`. Si une nouvelle ressource a été créée à partir d’une opération de longue durée, la réponse de réussite retourne un code d’état HTTP `201 Created`. La réponse contient également un en-tête `Location` qui pointe vers des métadonnées relatives à la ressource. Si aucune nouvelle ressource n’a été créée, la réponse de réussite retourne un code d’état HTTP `200 OK`. En cas d’échec, le code d’état de la réponse est également le code `200 OK`. Toutefois, le corps de la réponse contient une propriété `error`. Les données d’erreur adhèrent à la spécification d’erreur OData.

### <a name="example-of-success-response"></a>Exemple de réponse de réussite

```HTTP
Status: 201 Created
Location: "https://atlas.microsoft.com/tileset/{tileset-id}"
 {
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Succeeded",
    "resourceLocation": "https://atlas.microsoft.com/tileset/{tileset-id}"
}
```

### <a name="example-of-failure-response"></a>Exemple de réponse d’échec

```HTTP
Status: 200 OK

{
    "operationId": "c587574e-add9-4ef7-9788-1635bed9a87e",
    "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
    "status": "Failed",
    "error": {
        "code": "InvalidFeature",
        "message": "The provided feature is invalid.",
        "details": {
            "code": "NoGeometry",
            "message": "No geometry was provided with the feature."
        }
    }
}
```