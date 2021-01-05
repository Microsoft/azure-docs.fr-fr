---
title: Utiliser des méthodes directes dans Live Video Analytics sur IoT Edge – Azure
description: Live Video Analytics sur IoT Edge expose plusieurs méthodes directes. Les méthodes directes sont basées sur les conventions décrites dans cette rubrique.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 8b5c16dc72beed4ec757e48461a2fc194c113f8d
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656238"
---
# <a name="direct-methods"></a>Méthodes directes

Live Video Analytics sur IoT Edge expose plusieurs méthodes directes qui peuvent être appelées à partir d’IoT Hub. Les méthodes directes représentent une interaction de demande-réponse avec un appareil, similaire à un appel HTTP, dans la mesure où elles réussissent ou échouent immédiatement (après un délai d’attente spécifié par l’utilisateur). Cette approche est utile pour les scénarios où le plan d’action immédiate est différent selon que l’appareil a été en mesure ou non de répondre. Pour plus d’informations, consultez [Comprendre et appeler des méthodes directes à partir d’IoT Hub](../../iot-hub/iot-hub-devguide-direct-methods.md).

Cette rubrique décrit ces méthodes et conventions.

## <a name="conventions"></a>Conventions

Les méthodes directes sont basées sur les conventions suivantes :

1. Les méthodes directes ont une version spécifiée au format MAJOR.MINOR (comme indiqué dans l’exemple suivant) :

    ```
    {
        "methodName": "GraphTopologySet",
        "payload": {
            // API version matches matches module MAJOR.MINOR version
            "@apiVersion": "1.0"
            //payload here…
        }
    }
    ```

2. Une version donnée du module Live Video Analytics sur IoT Edge prendra en charge toutes les méthodes directes jusqu’à sa version actuelle. Par exemple, le module version 1.3 prend en charge les méthodes directes compatibles avec les versions 1.3, 1.2, 1.1 et 1.0.
3. Toutes les méthodes directes sont synchrones.
4. Les résultats d’erreur suivent le schéma d’erreur OData.
5. Les noms doivent respecter les contraintes suivantes :
    
    * Uniquement des caractères alphanumériques et des tirets, à condition qu’ils ne commencent pas ni ne se terminent par un tiret
    * Pas d’espaces
    * 32 caractères maximum

### <a name="example"></a>Exemple

```
{
  "status": 400,
  "payload": {
    "error": {
      "code": "BadRequest",
      "message": "Graph instance is not found."
    }
  }
}
```    

### <a name="top-level-error-codes"></a>Codes d’erreur de premier niveau     

|Statut |Code   |Message|
|---|---|---|
|400|   BadRequest| La demande n’est pas valide.|
|400|   InvalidResource|    La ressource n’est pas valide.|
|400|   InvalidVersion| La version de l’API n’est pas valide.|
|402|   ConnectivityRequired    |Le module Edge requiert une connectivité cloud pour fonctionner correctement.|
|404|   NotFound    |Ressource introuvable|
|409|   Conflit|   Conflit d’opérations.|
|500|   InternalServerError|    Erreur interne du serveur|
|503|   ServerBusy| Serveur occupé.|

### <a name="detailed-error-codes"></a>Codes d’erreur détaillés

Des erreurs de validation détaillées, telles que les validations de module de graphique, sont ajoutées en tant que détails de l’erreur :

```
{
  "status": 400,
  "payload": {
    "error": {
      "code": "InvalidResource",
      "message": "The resource format is invalid.",
      "details": [
        {
          "code": "RtspEndpointUrlInvalidScheme",
          "target": "$.Properties.Sources[0]",
          "message": "Uri scheme 'http' is not valid for an RTSP source endpoint. Valid values are: rtsp"
        },
        {
          "code": "PropertyValueInvalidPattern",
          "target": "$.Properties.Sinks[0].AssetNamePattern",
          "message": "The value must match the regular expression '^[^<>%&:\\\\\\/?*+.']{1,260}$'"
        }
      ]
    }
  }
}
}
```

|Statut|    Code détaillé   |Description|
|---|---|---|
|400|   GraphValidationError|   Erreurs graphiques générales telles que les cycles ou le partitionnement, etc.|
|400|   ModuleValidationError|  Erreurs de validation spécifiques au module.|
|409|   GraphTopologyInUse| La topologie de graphe est toujours référencée par une ou plusieurs instances de graphe.|
|409|   OperationNotAllowedInState| L’opération demandée ne peut pas être effectuée à l’état actuel.|
|409|   ResourceValidationError|    La ressource référencée (par exemple, Asset) n’est pas dans un état valide.|

## <a name="details"></a>Détails  

### <a name="graphtopologyget"></a>GraphTopologyGet

Cette méthode directe récupère une topologie de graphe unique.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphTopologyGet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": {
        "name": "CaptureAndRecord",
        "properties": {
            // Complete graph topology
        }
    }
}
```

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Entité trouvée | 200 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Entité introuvable | 404 |  |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphtopologyset"></a>GraphTopologySet

Crée une topologie unique s’il n’existe pas de topologie avec le nom donné ou met à jour une topologie portant le même nom.

Aspects clés :

* La topologie peut être mise à jour librement, car aucun graphe n’y fait référence.
* La topologie peut être mise à jour librement si tous les graphes y faisant référence sont désactivés, à condition que :

    * Les paramètres récemment ajoutés ont des valeurs par défaut.
    * Les paramètres supprimés ne sont référencés par aucun graphe.
* Les mises à jour de la topologie ne sont pas autorisées si des graphes sont actifs.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphTopologySet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord",
        "properties": {
            // Desired graph topology properties
        }
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 201,
    "payload": {
        "name": " CaptureAndRecord ",
        "properties": {
            // Complete graph topology
        }
    }
}
```

#### <a name="status-codes"></a>Codes d’état

|Condition  |Code d’état    |Code d’erreur détaillé|
|---|---|---|
Entité existante mise à jour |200|   N/A|
Nouvelle entité créée  |201|   N/A|
Erreurs utilisateur générales |Plage 400  ||
Erreurs de validation du graphe |400    |GraphValidationError|
Erreurs de validation du module|   400 |ModuleValidationError|
Erreurs de serveur générales   |Plage 500  ||

### <a name="graphtopologydelete"></a>GraphTopologyDelete

Supprime une topologie de graphe unique.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphTopologyDelete",
    "payload": {
        "@apiVersion": "1.0",
        "name": "CaptureAndRecord"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Entité supprimée | 200 | N/A |
| Entité introuvable | 204 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| La topologie de graphe est référencée par une ou plusieurs instances de graphe | 409 | GraphTopologyInUse |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphtopologylist"></a>GraphTopologyList

Récupère une liste de toutes les topologies de graphe qui correspondent aux critères de filtre.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphTopologyList",
    "payload": {
        // Required
        "@apiVersion": "1.0",
        
        // Optional
        "@continuationToken": "xxxxx",    
        "@query": "$orderby=name asc"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": {
        "@continuationToken": "xxxx",
        "value": [
            {
                "name": "Capture & Record",
                // ...
            },
            {
                "name": "Capture, Analyze & Record",
                // ...
            }
        ]
    }
}
```

#### <a name="filter-support"></a>Prise en charge des filtres

|Opération |Champ(s)    |Opérateurs|
|---|---|---|
|$orderby|name  |asc|


#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Succès | 200 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphinstanceget"></a>GraphInstanceGet

Récupère une instance de graphe unique :

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphInstanceGet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": {
        "name": "Camera001",
        "properties": {
            // Complete graph instance
        }
    }
}
```

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Entité trouvée | 200 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Entité introuvable | 404 |  |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphinstanceset"></a>GraphInstanceSet

Crée une instance de graphe unique s’il n’existe pas d’instance avec le nom donné ou met à jour une instance portant le même nom.

Aspects clés :

* L’instance de graphe peut être mise à jour librement à l’état « Désactivé ».

    * Le graphe est revalidé à chaque mise à jour.
* Les mises à jour de l’instance de graphe sont partiellement restreintes lorsque le graphe n’est pas à l’état « Inactif ».
* Les mises à jour de l’instance de graphe ne sont pas autorisées sur les graphes actifs.

#### <a name="request"></a>Requête

```
{
"methodName": "GraphInstanceSet",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001",
        "properties": {
            // Desired graph instance properties
        }
    }
}
```

#### <a name="response"></a>response

````
{
    "status": 201,
    "payload": {
        "name": " Camera001",
        "properties": {
            // Complete graph instance
        }   
    }
}
````

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Entité existante mise à jour | 200 | N/A |
| Nouvelle entité créée | 201 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Erreurs de validation du graphe | 400 | GraphValidationError |
| Erreurs de validation du module | 400 | ModuleValidationError |
| Erreurs de validation des ressources | 409 | ResourceValidationError |
| Erreurs de serveur générales | Plage 500 |  |  |

### <a name="graphinstancedelete"></a>GraphInstanceDelete

Supprime une instance de graphe unique.

Aspects clés :

* Seuls les graphes désactivés peuvent être supprimés.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphInstanceDelete",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Graphe correctement supprimé | 200 | N/A |
| Graphe introuvable | 204 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Le graphe n’est pas à l’état « Arrêté » | 409 | OperationNotAllowedInState |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphinstancelist"></a>GraphInstanceList

Cette méthode est similaire à la méthode GraphTopologyList. Elle permet d’énumérer les instances de graphe.
Récupère une liste de toutes les instances de graphe qui correspondent aux critères de filtre.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphInstanceList",
    "payload": {
        // Required
        "@apiVersion": "1.0",
        
        // Optional
        "@continuationToken": "xxxxx",    
        "@query": "$orderby=name asc"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": {
        "@continuationToken": "xxxx",
        "value": [
            {
                "name": "Capture & Record",
                // ...
            },
            {
                "name": "Capture, Analyze & Record",
                // ...
            }
        ]
    }
}
```

#### <a name="filter-support"></a>Prise en charge des filtres

|Opération  |   Champ(s)|   Opérateurs|
|---|---|---|
|$orderby|  name|   asc|

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Succès | 200 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphinstanceactivate"></a>GraphInstanceActivate

Active une instance de graphe unique. 

Aspects clés

* La méthode retourne des résultats uniquement lorsque le graphe est activé. 
* Le graphe suppose l’état « Activation » lors de son activation.

    * Une opération List/Get sur le graphe retourne le graphe à l’état approprié.
* Idempotence :

    * Le démarrage d’un graphe à l’état « Activation » se comporte de la même façon que si le graphe était désactivé (c’est-à-dire, bloque l’appel jusqu’à l’activation du graphique).
    * L’activation d’un graphe à l’état « Actif » est immédiatement retournée.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphInstanceActivate",
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": null
}
```

#### <a name="status-codes"></a>Codes d’état

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Graphe activé | 200 | N/A |
| Nouvelle entité créée | 201 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Erreurs de validation du module | 400 | ModuleValidationError |
| Erreurs de validation des ressources | 409 | ResourceValidationError |
| Le graphe est à l’état Désactivation | 409 | OperationNotAllowedInState |
| Erreurs de serveur générales | Plage 500 |  |

### <a name="graphinstancedeactivate"></a>GraphInstanceDeactivate

Désactive une instance de graphe unique. La désactivation d’un graphe désactive normalement le traitement des médias et garantit que tous les événements et médias stockés de manière transitoire sur Edge sont validés dans le cloud, le cas échéant.

Aspects clés :

* La méthode retourne des résultats uniquement lorsque le graphe est désactivé.
* Le graphe suppose l’état « Désactivation » lors de sa désactivation.

    * Une opération List/Get sur le graphe retourne le graphe à l’état approprié.
    * L’arrêt se termine uniquement lorsque tous les médias ont été chargés dans le cloud.
* Idempotence :

    * La désactivation d’un graphe à l’état « Désactivation » se comporte de la même façon que si le graphe était désactivé (c’est-à-dire, bloque l’appel jusqu’à la désactivation du graphique).
    * La désactivation d’un graphe à l’état « Inactif » est immédiatement retournée.

#### <a name="request"></a>Requête

```
{
    "methodName": "GraphInstanceDeactivate",
    // A high response timeout is recommended here since there might be data movement    // being executed as part of this operation
    "responseTimeoutInSeconds": 300,
    "payload": {
        "@apiVersion": "1.0",
        "name": "Camera001"
    }
}
```

#### <a name="response"></a>response

```
{
    "status": 200,
    "payload": null
}
```

| Condition | Code d’état | Code d’erreur détaillé |
|--|--|--|
| Graphe activé | 200 | N/A |
| Nouvelle entité créée | 201 | N/A |
| Erreurs utilisateur générales | Plage 400 |  |
| Le graphe est à l’état Activation | 409 | OperationNotAllowedInState |
| Erreurs de serveur générales | Plage 500 |  |

## <a name="next-steps"></a>Étapes suivantes

[Schéma de configuration de jumeau de module](module-twin-configuration-schema.md)
