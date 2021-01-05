---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: 46cb129d18e082f836a688b95111c10c8e191b01
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511354"
---
## <a name="model-id-announcement"></a>Annonce de l’ID de modèle

Pour annoncer l’ID de modèle, l’appareil doit l’inclure dans les informations de connexion :

```csharp
DeviceClient.CreateFromConnectionString(
  connectionString,
  TransportType.Mqtt,
  new ClientOptions() { ModelId = modelId })
```

La nouvelle surcharge de `ClientOptions` est disponible dans toutes les méthodes `DeviceClient` utilisées pour initialiser une connexion.

> [!TIP]
> Pour les modules et IoT Edge, utilisez `ModuleClient` à la place de `DeviceClient`.

## <a name="dps-payload"></a>Charge utile DPS

Les appareils qui utilisent [Device Provisioning Service (DPS)](../articles/iot-dps/about-iot-dps.md) peuvent inclure le `modelId` à utiliser pendant le processus de provisionnement à l’aide de la charge utile JSON suivante.

```json
{
    "modelId" : "dtmi:com:example:Thermostat;1"
}
```

## <a name="implement-telemetry-properties-and-commands"></a>Implémentation des données de télémétrie, des propriétés et des commandes

Comme le décrit la section [Présentation des composants dans les modèles IoT Plug-and-Play](../articles/iot-pnp/concepts-components.md), les générateurs d’appareils doivent décider s’ils souhaitent utiliser des composants pour décrire leurs appareils, auquel cas ces derniers devront suivre les règles décrites dans cette section.

### <a name="telemetry"></a>Télémétrie

Aucun composant par défaut ne nécessite une propriété spéciale.

Lorsque vous utilisez des composants imbriqués, les appareils doivent définir une propriété de message avec le nom du composant :

```c#
public async Task SendComponentTelemetryValueAsync(string componentName, string serializedTelemetry)
{
  var message = new Message(Encoding.UTF8.GetBytes(serializedTelemetry));
  message.ComponentName = componentName;
  message.ContentType = "application/json";
  message.ContentEncoding = "utf-8";
  await client.SendEventAsync(message);
}
```

### <a name="read-only-properties"></a>Propriétés en lecture seule

Le signalement d’une propriété à partir du composant par défaut ne nécessite pas de construction spéciale :

```csharp
TwinCollection reportedProperties = new TwinCollection();
reportedProperties["maxTemperature"] = 38.7;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
      "maxTemperature" : 38.7
  }
}
```

Lorsque vous utilisez des composants imbriqués, les propriétés doivent être créées dans le nom du composant :

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
component["maxTemperature"] = 38.7;
component["__t"] = "c"; // marker to identify a component
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTemperature" : 38.7
     } 
  }
}
```

### <a name="writable-properties"></a>Propriétés accessibles en écriture

Ces propriétés peuvent être définies par l’appareil ou mises à jour par la solution. Si la solution met à jour une propriété, le client reçoit une notification sous la forme d’un rappel dans `DeviceClient` ou `ModuleClient`. Pour respecter les conventions IoT Plug-and-Play, l’appareil doit informer le service que la propriété a bien été reçue.

#### <a name="report-a-writable-property"></a>Signalement d’une propriété accessible en écriture

Quand un appareil rapporte une propriété accessible en écriture, il doit inclure les valeurs `ack` définies dans les conventions.

Pour signaler une propriété accessible en écriture à partir du composant par défaut :

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not readed from a desired property
ackProps["ad"] = "reported default value";
reportedProperties["targetTemperature"] = ackProps;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Pour signaler une propriété accessible en écriture à partir d’un composant imbriqué, le jumeau doit inclure un marqueur :

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
component["__t"] = "c"; // marker to identify a component
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not read from a desired property
ackProps["ad"] = "reported default value";
component["targetTemperature"] = ackProps;
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Abonnement aux mises à jour de propriétés souhaitées

Les services peuvent mettre à jour les propriétés souhaitées qui déclenchent une notification sur les appareils connectés. Cette notification inclut les propriétés souhaitées mises à jour, y compris le numéro de version identifiant la mise à jour. Les appareils doivent répondre avec le même message `ack` que les propriétés rapportées.

Un composant par défaut considère la propriété unique et crée la propriété `ack` rapportée avec la version reçue :

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) => 
{
  JValue targetTempJson = desired["targetTemperature"];
  double targetTemperature = targetTempJson.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200;
  ackProps["av"] = desired.Version; 
  ackProps["ad"] = "desired property received";
  reportedProperties["targetTemperature"] = ackProps;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

Le jumeau d’appareil montre la propriété dans les sections desired et reported :

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Un composant imbriqué reçoit les propriétés souhaitées encapsulées avec le nom du composant et doit signaler en retour la propriété rapportée `ack` :

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) =>
{
  JObject thermostatComponent = desired["thermostat1"];
  JToken targetTempProp = thermostatComponent["targetTemperature"];
  double targetTemperature = targetTempProp.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection component = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  component["__t"] = "c"; // marker to identify a component
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200; // using HTTP status codes
  ackProps["av"] = desired.Version; // not readed from a desired property
  ackProps["ad"] = "desired property received";
  component["targetTemperature"] = ackProps;
  reportedProperties["thermostat1"] = component;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

Le jumeau d’appareil d’un composant imbriqué montre les sections desired et reported de la façon suivante :

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Commandes

Un composant par défaut reçoit le nom de la commande telle qu’elle a été appelée par le service.

Un composant imbriqué reçoit le nom de la commande avec, pour préfixe, le nom du composant et le séparateur `*`.

```csharp
await client.SetMethodHandlerAsync("themostat*reboot", (MethodRequest req, object ctx) =>
{
  Console.WriteLine("REBOOT");
  return Task.FromResult(new MethodResponse(200));
},
null);
```

#### <a name="request-and-response-payloads"></a>Charge utile de demande et de réponse

Les commandes utilisent des types pour définir leur charge utile de demande et de réponse. Un appareil doit désérialiser le paramètre d’entrée entrant et sérialiser la réponse. L’exemple suivant montre comment implémenter une commande avec des types complexes définis dans les charges utiles :

```json
{
  "@type": "Command",
  "name": "start",
  "request": {
    "name": "startRequest",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "startPriority",
          "schema": "integer"
        },
        {
          "name": "startMessage",
          "schema" : "string"
        }
      ]
    }
  },
  "response": {
    "name": "startReponse",
    "schema": {
      "@type": "Object",
      "fields": [
        {
            "name": "startupTime",
            "schema": "integer" 
        },
        {
          "name": "startupMessage",
          "schema": "string"
        }
      ]
    }
  }
}
```

Les extraits de code suivants montrent comment un appareil implémente cette définition de commande, notamment les types utilisés pour permettre la sérialisation et la désérialisation :

```csharp
class startRequest
{
  public int startPriority { get; set; }
  public string startMessage { get; set; }
}

class startResponse
{
  public int startupTime { get; set; }
  public string startupMessage { get; set; }
}

// ... 

await client.SetMethodHandlerAsync("start", (MethodRequest req, object ctx) =>
{
  var startRequest = JsonConvert.DeserializeObject<startRequest>(req.DataAsJson);
  Console.WriteLine($"Received start command with priority ${startRequest.startPriority} and ${startRequest.startMessage}");

  var startResponse = new startResponse
  {
    startupTime = 123,
    startupMessage = "device started with message " + startRequest.startMessage
  };

  string responsePayload = JsonConvert.SerializeObject(startResponse);
  MethodResponse response = new MethodResponse(Encoding.UTF8.GetBytes(responsePayload), 200);
  return Task.FromResult(response);
},null);
```

> [!Tip]
> Le nom des demandes et des réponses n’est pas présent dans les charges utiles sérialisées transmises sur le réseau.
