---
title: Configurer OPC Publisher - Azure | Microsoft Docs
description: Cet article explique comment configurer OPC Publisher pour spécifier les modifications de données des nœuds OPC UA, les événements OPC UA à publier, ainsi que le format de télémétrie.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom: devx-track-csharp
ms.openlocfilehash: 405fa04a3b6af7f0c87888aa48aba9bdffeaf282
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366953"
---
# <a name="configure-opc-publisher"></a>Configurer OPC Publisher

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Vous pouvez configurer OPC Publisher pour spécifier ce qui suit :

- Données de nœud OPC UA à publier.
- Événements OPC UA à publier.
- Format de télémétrie.

Vous pouvez configurer OPC Publisher à l’aide de fichiers de configuration ou d’appels de méthode.

## <a name="use-configuration-files"></a>Utiliser des fichiers de configuration

Cette section décrit les options de configuration de la publication de nœuds OPC UA avec les fichiers de configuration.

### <a name="use-a-configuration-file-to-configure-publishing-data-changes"></a>Utiliser un fichier de configuration pour configurer les modifications des données de publication

La façon la plus simple de configurer les nœuds OPC UA à publier est d’utiliser un fichier de configuration. Le format du fichier de configuration est documenté dans [publishednodes.json](https://github.com/Azure/iot-edge-opc-publisher/blob/master/opcpublisher/publishednodes.json) dans le référentiel.

La syntaxe du fichier de configuration a changé au fil du temps. OPC Publisher lit toujours les anciens formats, mais les convertit dans le dernier format lorsqu’il conserve la configuration.

L’exemple suivant montre le format du fichier de configuration :

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62541/Quickstarts/ReferenceServer",
    "UseSecurity": true,
    "OpcNodes": [
      {
        "Id": "i=2258",
        "OpcSamplingInterval": 2000,
        "OpcPublishingInterval": 5000,
        "DisplayName": "Current time"
      }
    ]
  }
]
```

### <a name="use-a-configuration-file-to-configure-publishing-events"></a>Utiliser un fichier de configuration pour configurer la publication d’événements

Pour publier les événements OPC UA, vous utilisez le même fichier de configuration que pour les modifications de données.

L’exemple suivant montre comment configurer la publication des événements générés par le serveur [SimpleEvents](https://github.com/OPCFoundation/UA-.NETStandard-Samples/tree/master/Workshop/SimpleEvents/Server). Le serveur SimpleEvents se trouve dans le [référentiel OPC Foundation](https://github.com/OPCFoundation/UA-.NETStandard-Samples) :

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62563/Quickstarts/SimpleEventsServer",
    "OpcEvents": [
      {
        "Id": "i=2253",
        "DisplayName": "SimpleEventServerEvents",
        "SelectClauses": [
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "EventId"
            ]
          },
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "Message"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CycleId"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CurrentStep"
            ]
          }
        ],
        "WhereClause": [
          {
            "Operator": "OfType",
            "Operands": [
              {
                "Literal": "ns=2;i=235"
              }
            ]
          }
        ]
      }
    ]
  }
]
```

## <a name="use-method-calls"></a>Utiliser des appels de méthode

Cette section décrit les appels de méthode que vous pouvez utiliser pour configurer OPC Publisher.

### <a name="configure-using-opc-ua-method-calls"></a>Configurer à l’aide des appels de méthode OPC UA

OPC Publisher inclut un serveur OPC UA, accessible sur le port 62222. Si le nom d’hôte est **publisher** , l’URI du point de terminaison est : `opc.tcp://publisher:62222/UA/Publisher`.

Ce point de terminaison présente les quatre méthodes suivantes :

- PublishNode
- UnpublishNode
- GetPublishedNodes
- IoT HubDirectMethod

### <a name="configure-using-iot-hub-direct-method-calls"></a>Configurer à l’aide d’appels de méthode directs IoT Hub

OPC Publisher implémente les appels de méthode directs IoT Hub suivants :

- PublishNodes
- UnpublishNodes
- UnpublishAllNodes
- GetConfiguredEndpoints
- GetConfiguredNodesOnEndpoint
- GetDiagnosticInfo
- GetDiagnosticLog
- GetDiagnosticStartupLog
- ExitApplication
- GetInfo

Le format de la charge utile JSON de la requête de méthode et des réponses est défini dans [opcpublisher/HubMethodModel.cs](https://github.com/Azure/iot-edge-opc-publisher/tree/master/opcpublisher).

Si vous appelez une méthode inconnue sur le module, elle répond avec une chaîne indiquant que la méthode n’est pas implémentée. Vous pouvez appeler une méthode inconnue comme moyen de « pinger » le module.

### <a name="configure-username-and-password-for-authentication"></a>Configurer le nom d’utilisateur et le mot de passe pour l’authentification

Le mode d’authentification peut être défini par le biais d’appels directs de méthode IoT Hub. La charge utile doit contenir la propriété **OpcAuthenticationMode** et le nom d’utilisateur et le mot de passe :

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "UsernamePassword",
    "Username": "<Username>",
    "Password": "<Password>"
    ...
}
```

Le mot de passe est chiffré par le client Workload IoT Hub et stocké dans la configuration de l’éditeur. Pour que l’authentification redevienne anonyme, utilisez la méthode avec la charge utile suivante :

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "Anonymous"
    ...
}
```

Si la propriété **OpcAuthenticationMode** n’est pas définie dans la charge utile, les paramètres d’authentification restent inchangés dans la configuration.

## <a name="configure-telemetry-publishing"></a>Configurer la publication de données de télémétrie

Lorsque OPC Publisher reçoit une notification de changement de valeur dans un nœud publié, il génère un message au format JSON qui est envoyé à IoT Hub.

Vous pouvez configurer le contenu de ce message au format JSON à l’aide d’un fichier de configuration. Si aucun fichier de configuration n’est spécifié avec l’option `--tc`, une configuration par défaut compatible avec [l’accélérateur de solution de fabrique connecté](https://github.com/Azure/azure-iot-connected-factory) est utilisée.

Si OPC Publisher est configuré pour les messages par lots, ils sont envoyés en tant que tableau JSON valide.

La télémétrie est dérivée des sources suivantes :

- Configuration du nœud OPC Publisher pour le nœud
- Objet **MonitoredItem** de la pile OPC UA pour laquelle OPC Publisher a reçu une notification.
- L’argument est passé à cette notification, qui fournit des détails sur la modification de la valeur des données.

Les données de télémétrie placées dans le message au format JSON est une sélection de propriétés importantes de ces objets. Si vous avez besoin de plus de propriétés, vous devez changer la base de code OPC Publisher.

La syntaxe du fichier de configuration est la suivante :

```json
// The configuration settings file consists of two objects:
// 1) The 'Defaults' object, which defines defaults for the telemetry configuration
// 2) An array 'EndpointSpecific' of endpoint specific configuration
// Both objects are optional and if they are not specified, then publisher uses
// its internal default configuration, which generates telemetry messages compatible
// with the Microsoft Connected factory Preconfigured Solution (https://github.com/Azure/azure-iot-connected-factory).

// A JSON telemetry message for Connected factory looks like:
//  {
//      "NodeId": "i=2058",
//      "ApplicationUri": "urn:myopcserver",
//      "DisplayName": "CurrentTime",
//      "Value": {
//          "Value": "10.11.2017 14:03:17",
//          "SourceTimestamp": "2017-11-10T14:03:17Z"
//      }
//  }

// The 'Defaults' object in the sample below, are similar to what publisher is
// using as its internal default telemetry configuration.
{
    "Defaults": {
        // The first two properties ('EndpointUrl' and 'NodeId' are configuring data
        // taken from the OpcPublisher node configuration.
        "EndpointUrl": {

            // The following three properties can be used to configure the 'EndpointUrl'
            // property in the JSON message send by publisher to IoT Hub.

            // Publish controls if the property should be part of the JSON message at all.
            "Publish": false,

            // Pattern is a regular expression, which is applied to the actual value of the
            // property (here 'EndpointUrl').
            // If this key is omitted (which is the default), then no regex matching is done
            // at all, which improves performance.
            // If the key is used you need to define groups in the regular expression.
            // Publisher applies the regular expression and then concatenates all groups
            // found and use the resulting string as the value in the JSON message to
            //sent to IoT Hub.
            // This example mimics the default behaviour and defines a group,
            // which matches the conplete value:
            "Pattern": "(.*)",
            // Here some more exaples for 'Pattern' values and the generated result:
            // "Pattern": "i=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of '2058'to be sent by publisher
            // "Pattern": "(i)=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of 'i2058' to be sent by publisher

            // Name allows you to use a shorter string as property name in the JSON message
            // sent by publisher. By default the property name is unchanged and will be
            // here 'EndpointUrl'.
            // The 'Name' property can only be set in the 'Defaults' object to ensure
            // all messages from publisher sent to IoT Hub have a similar layout.
            "Name": "EndpointUrl"

        },
        "NodeId": {
            "Publish": true,

            // If you set Defaults.NodeId.Name to "ni", then the "NodeId" key/value pair
            // (from the above example) will change to:
            //      "ni": "i=2058",
            "Name": "NodeId"
        },

        // The MonitoredItem object is configuring the data taken from the MonitoredItem
        // OPC UA object for published nodes.
        "MonitoredItem": {

            // If you set the Defaults.MonitoredItem.Flat to 'false', then a
            // 'MonitoredItem' object will appear, which contains 'ApplicationUri'
            // and 'DisplayNode' proerties:
            //      "NodeId": "i=2058",
            //      "MonitoredItem": {
            //          "ApplicationUri": "urn:myopcserver",
            //          "DisplayName": "CurrentTime",
            //      }
            // The 'Flat' property can only be used in the 'MonitoredItem' and
            // 'Value' objects of the 'Defaults' object and will be used
            // for all JSON messages sent by publisher.
            "Flat": true,

            "ApplicationUri": {
                "Publish": true,
                "Name": "ApplicationUri"
            },
            "DisplayName": {
                "Publish": true,
                "Name": "DisplayName"
            }
        },
        // The Value object is configuring the properties taken from the event object
        // the OPC UA stack provided in the value change notification event.
        "Value": {
            // If you set the Defaults.Value.Flat to 'true', then the 'Value'
            // object will disappear completely and the 'Value' and 'SourceTimestamp'
            // members won't be nested:
            //      "DisplayName": "CurrentTime",
            //      "Value": "10.11.2017 14:03:17",
            //      "SourceTimestamp": "2017-11-10T14:03:17Z"
            // The 'Flat' property can only be used for the 'MonitoredItem' and 'Value'
            // objects of the 'Defaults' object and will be used for all
            // messages sent by publisher.
            "Flat": false,

            "Value": {
                "Publish": true,
                "Name": "Value"
            },
            "SourceTimestamp": {
                "Publish": true,
                "Name": "SourceTimestamp"
            },
            // 'StatusCode' is the 32 bit OPC UA status code
            "StatusCode": {
                "Publish": false,
                "Name": "StatusCode"
                // 'Pattern' is ignored for the 'StatusCode' value
            },
            // 'Status' is the symbolic name of 'StatusCode'
            "Status": {
                "Publish": false,
                "Name": "Status"
            }
        }
    },

    // The next object allows to configure 'Publish' and 'Pattern' for specific
    // endpoint URLs. Those will overwrite the ones specified in the 'Defaults' object
    // or the defaults used by publisher.
    // It is not allowed to specify 'Name' and 'Flat' properties in this object.
    "EndpointSpecific": [
        // The following shows how a endpoint specific configuration can look like:
        {
            // 'ForEndpointUrl' allows to configure for which OPC UA server this
            // object applies and is a required property for all objects in the
            // 'EndpointSpecific' array.
            // The value of 'ForEndpointUrl' must be an 'EndpointUrl' configured in
            // the publishednodes.json confguration file.
            "ForEndpointUrl": "opc.tcp://<your_opcua_server>:<your_opcua_server_port>/<your_opcua_server_path>",
            "EndpointUrl": {
                // We overwrite the default behaviour and publish the
                // endpoint URL in this case.
                "Publish": true,
                // We are only interested in the URL part following the 'opc.tcp://' prefix
                // and define a group matching this.
                "Pattern": "opc.tcp://(.*)"
            },
            "NodeId": {
                // We are not interested in the configured 'NodeId' value, 
                // so we do not publish it.
                "Publish": false
                // No 'Pattern' key is specified here, so the 'NodeId' value will be
                // taken as specified in the publishednodes configuration file.
            },
            "MonitoredItem": {
                "ApplicationUri": {
                    // We already publish the endpoint URL, so we do not want
                    // the ApplicationUri of the MonitoredItem to be published.
                    "Publish": false
                },
                "DisplayName": {
                    "Publish": true
                }
            },
            "Value": {
                "Value": {
                    // The value of the node is important for us, everything else we
                    // are not interested in to keep the data ingest as small as possible.
                    "Publish": true
                },
                "SourceTimestamp": {
                    "Publish": false
                },
                "StatusCode": {
                    "Publish": false
                },
                "Status": {
                    "Publish": false
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à configurer OPC Publisher, l’étape suivante suggérée consiste à apprendre comment [Exécuter OPC Publisher](howto-opc-publisher-run.md).
