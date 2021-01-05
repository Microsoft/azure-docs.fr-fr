---
title: Requête sur le routage des messages IoT Hub | Microsoft Docs
description: Découvrez plus en détail le langage de requête de routage des messages IoT Hub que vous pouvez utiliser pour appliquer des requêtes riches à des messages afin de recevoir les données qui vous intéressent.
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: asrastog
ms.custom:
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 83c290adea02915db1dc52bd359b4d3165611522
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547705"
---
# <a name="iot-hub-message-routing-query-syntax"></a>Syntaxe des requêtes pour le routage des messages IoT Hub

Le routage de messages permet aux utilisateurs d’acheminer différents types de données, notamment des messages de télémétrie d’appareil, des événements de cycle de vie d’appareil et des événements de modification de jumeau d’appareil, vers différents points de terminaison. Vous pouvez également appliquer des requêtes élaborées à ces données avant de les router, afin de recevoir les données qui vous intéressent. Cet article décrit le langage des requêtes du routage de messages IoT Hub, et fournit quelques modèles courants de requête.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Le routage de messages vous permet d’interroger les propriétés et le corps d’un message, ainsi que les étiquettes et les propriétés d’un jumeau d’appareil. Si le corps du message n’est pas un fichier JSON, le routage de messages peut toujours acheminer le message, mais les requêtes ne sont pas applicables à ce corps de message.  Les requêtes sont décrites comme des expressions booléennes, où une valeur booléenne True fait réussir la requête qui achemine toutes les données entrantes, et où une valeur booléenne False fait échouer la requête qui n’achemine aucune donnée. Si l’expression prend la valeur Null ou Undefined, elle est considérée comme False et une erreur est générée dans les [journaux de ressources des routes](monitor-iot-hub-reference.md#routes) d’IoT Hub en cas d’échec. La syntaxe des requêtes doit être correcte pour que l’itinéraire soit enregistré et évalué.  

## <a name="message-routing-query-based-on-message-properties"></a>Requête de routage de messages en fonction des propriétés de message 

IoT Hub définit un [format commun](iot-hub-devguide-messages-construct.md) pour tous les envois de messages appareil-à-cloud à des fins d'interopérabilité entre les protocoles. Le message IoT Hub suppose la représentation JSON suivante du message. Les propriétés système sont ajoutées pour tous les utilisateurs et identifient le contenu du message. Les utilisateurs peuvent ajouter de façon sélective des propriétés de l’application au message. La messagerie appareil-à-cloud IoT Hub ne respectant pas la casse, nous recommandons d’utiliser des noms de propriété uniques. Par exemple, si vous avez plusieurs propriétés portant le même nom, IoT Hub n’envoie qu’une seule de ces propriétés.  

```json
{ 
  "message": { 
    "systemProperties": { 
      "contentType": "application/json", 
      "contentEncoding": "UTF-8", 
      "iothub-message-source": "deviceMessages", 
      "iothub-enqueuedtime": "2017-05-08T18:55:31.8514657Z" 
    }, 
    "appProperties": { 
      "processingPath": "{cold | warm | hot}", 
      "verbose": "{true, false}", 
      "severity": 1-5, 
      "testDevice": "{true | false}" 
    }, 
    "body": "{\"Weather\":{\"Temperature\":50}}" 
  } 
} 
```

### <a name="system-properties"></a>Propriétés système

Les propriétés système permettent d’identifier le contenu et la source des messages. 

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| contentType | string | L’utilisateur spécifie le type de contenu du message. Pour autoriser la requête sur le corps du message, cette valeur doit être définie sur application/JSON. |
| contentEncoding | string | L’utilisateur spécifie le type d’encodage du message. Les valeurs autorisées sont UTF-8, UTF-16, UTF-32 si la valeur contentType est définie sur application/JSON. |
| iothub-connection-device-id | string | Cette valeur est définie par IoT Hub et identifie l’ID de l’appareil. Pour interroger, utilisez `$connectionDeviceId`. |
| iothub-enqueuedtime | string | Cette valeur est définie par IoT Hub et représente l’heure (UTC) réelle de la mise en file d’attente du message. Pour interroger, utilisez `enqueuedTime`. |
| dt-dataschema | string |  Cette valeur est définie par IoT Hub sur les messages appareil-à-cloud. Elle contient l’ID du modèle d’appareil défini dans la connexion de l’appareil. Pour interroger, utilisez `$dt-dataschema`. |
| dt-subject | string | Nom du composant qui envoie les messages appareil-à-cloud. Pour interroger, utilisez `$dt-subject`. |

Comme expliqué dans l’article [Messages IoT Hub](iot-hub-devguide-messages-construct.md), il existe des propriétés système supplémentaires dans un message. Outre les propriétés indiquées dans le tableau précédent, vous pouvez également interroger **connectionDeviceId** et **connectionModuleId**.

### <a name="application-properties"></a>Propriétés de l’application

Les propriétés de l’application sont des chaînes définies par l’utilisateur qui peuvent être ajoutées au message. Ces champs sont facultatifs.  

### <a name="query-expressions"></a>Expressions de requête

Une requête sur les propriétés système d’un message doit avoir pour préfixe le symbole `$`. Les requêtes sur les propriétés de l’application sont accessibles par leur nom, dont le préfixe ne doit pas contenir de symbole `$`. Si le nom d’une propriété de l’application commence par `$`, IoT Hub recherche ce nom dans les propriétés système, où il est introuvable, puis le cherche dans les propriétés de l’application. Par exemple : 

Pour interroger la propriété système contentEncoding 

```sql
$contentEncoding = 'UTF-8'
```

Pour interroger la propriété de l’application processingPath :

```sql
processingPath = 'hot'
```

Pour combiner ces requêtes, vous pouvez utiliser des fonctions et des expressions booléennes :

```sql
$contentEncoding = 'UTF-8' AND processingPath = 'hot'
```

La liste complète des fonctions et des opérateurs pris en charge est affichée dans [Expression et conditions](iot-hub-devguide-query-language.md#expressions-and-conditions).

## <a name="message-routing-query-based-on-message-body"></a>Requête de routage de messages en fonction du corps de message

Pour permettre l’interrogation du corps de message, le message doit se trouver dans un fichier JSON encodé en UTF-8, UTF-16 ou UTF-32. `contentType` doit être définie sur `application/JSON`, et `contentEncoding` sur un des encodages UTF pris en charge dans la propriété système. Si ces propriétés ne sont pas spécifiées, IoT Hub ne tient pas compte de l’expression de requête sur le corps de message. 

L’exemple suivant montre comment créer un message avec un corps JSON correctement formé et encodé : 

```javascript
var messageBody = JSON.stringify(Object.assign({}, {
    "Weather": {
        "Temperature": 50,
        "Time": "2017-03-09T00:00:00.000Z",
        "PrevTemperatures": [
            20,
            30,
            40
        ],
        "IsEnabled": true,
        "Location": {
            "Street": "One Microsoft Way",
            "City": "Redmond",
            "State": "WA"
        },
        "HistoricalData": [
            {
                "Month": "Feb",
                "Temperature": 40
            },
            {
                "Month": "Jan",
                "Temperature": 30
            }
        ]
    }
}));

// Encode message body using UTF-8  
var messageBytes = Buffer.from(messageBody, "utf8");

var message = new Message(messageBytes);

// Set message body type and content encoding 
message.contentEncoding = "utf-8";
message.contentType = "application/json";

// Add other custom application properties   
message.properties.add("Status", "Active");

deviceClient.sendEvent(message, (err, res) => {
    if (err) console.log('error: ' + err.toString());
    if (res) console.log('status: ' + res.constructor.name);
});
```

> [!NOTE] 
> Cet exemple montre comment gérer l’encodage du corps en JavaScript. Si vous voulez voir un exemple en C#, téléchargez les [exemples C# Azure IOT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Décompressez le fichier master.zip. Le fichier Program.cs de la solution Visual Studio *SimulatedDevice* montre comment encoder et envoyer des messages à un hub IoT. Il s’agit du même exemple que celui utilisé pour tester le routage des messages, comme expliqué dans le [tutoriel Routage des messages](tutorial-routing.md). En bas de Program.cs, il a également une méthode pour lire l’un des fichiers encodés, le décoder et le réécrire au format ASCII afin que vous puissiez le lire. 


### <a name="query-expressions"></a>Expressions de requête

Une requête sur le corps de message doit avoir pour préfixe `$body`. Vous pouvez utiliser une référence au corps, une référence au tableau du corps ou plusieurs références au corps dans l’expression de requête. Votre expression de requête peut également combiner une référence au corps avec une référence aux propriétés système de message, et aux propriétés de l’application de messagerie. Par exemple, toutes les expressions de requête suivantes sont valides : 

```sql
$body.Weather.HistoricalData[0].Month = 'Feb' 
```

```sql
$body.Weather.Temperature = 50 AND $body.Weather.IsEnabled 
```

```sql
length($body.Weather.Location.State) = 2 
```

```sql
$body.Weather.Temperature = 50 AND processingPath = 'hot'
```

## <a name="message-routing-query-based-on-device-twin"></a>Requête de routage de messages en fonction du jumeau d’appareil 

Le routage de messages vous permet d’interroger les étiquettes et les propriétés d’un [Jumeau d’appareil](iot-hub-devguide-device-twins.md), qui sont des objets JSON. L’interrogation d’un jumeau de module est également prise en charge. Un exemple d’étiquettes et de propriétés de jumeau d’appareil est détaillé ci-dessous.

```JSON
{
    "tags": { 
        "deploymentLocation": { 
            "building": "43", 
            "floor": "1" 
        } 
    }, 
    "properties": { 
        "desired": { 
            "telemetryConfig": { 
                "sendFrequency": "5m" 
            }, 
            "$metadata" : {...}, 
            "$version": 1 
        }, 
        "reported": { 
            "telemetryConfig": { 
                "sendFrequency": "5m", 
                "status": "success" 
            },
            "batteryLevel": 55, 
            "$metadata" : {...}, 
            "$version": 4 
        } 
    } 
} 
```

### <a name="query-expressions"></a>Expressions de requête

Une requête sur le jumeau de message doit avoir pour préfixe `$twin`. Votre expression de requête peut également combiner une référence à une étiquette ou à une propriété de jumeau avec une référence au corps, aux propriétés système de message et aux propriétés de l’application de messagerie. La requête ne respectant pas la casse, nous recommandons d’utiliser des noms uniques dans les balises et les propriétés. Cela s’applique aux jumeaux d'appareil ainsi qu'aux jumeaux de module. Évitez également l’utilisation de `twin`, `$twin`, `body` ou `$body`, comme noms de propriété. Par exemple, toutes les expressions de requête suivantes sont valides : 

```sql
$twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$body.Weather.Temperature = 50 AND $twin.properties.desired.telemetryConfig.sendFrequency = '5m'
```

```sql
$twin.tags.deploymentLocation.floor = 1 
```

Une requête de routage sur le corps ou le jumeau d’appareil avec un point dans le nom de la charge utile ou de la propriété n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [routage de messages](iot-hub-devguide-messages-d2c.md)
* Essayer le [tutoriel sur le routage de messages](tutorial-routing.md)
