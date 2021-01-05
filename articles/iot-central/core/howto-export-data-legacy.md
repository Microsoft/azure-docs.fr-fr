---
title: Exporter des données à partir d'Azure IoT Central (fonctionnalité héritée) | Microsoft Docs
description: Comment exporter des données depuis votre application Azure IoT Central vers Azure Event Hubs, Azure Service Bus et Stockage Blob Azure
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 06/25/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: 9e5f4fd14f56f0a2dff45dd2650ea552b07fecd7
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94987352"
---
# <a name="export-iot-data-to-cloud-destinations-using-data-export-legacy"></a>Exporter des données IoT vers des destinations cloud à l'aide de la fonctionnalité d'exportation de données (héritée)

> [!Note]
> Cet article décrit la fonctionnalité d'exportation de données héritée d'IoT Central.
>
> - Pour plus d'informations sur la nouvelle fonctionnalité d'exportation de données, consultez la section [Exporter des données IoT vers des destinations cloud à l'aide de la fonctionnalité d'exportation de données](./howto-export-data.md).
> - Pour en savoir plus sur les différences entre la fonctionnalité d'exportation de données disponible en préversion et la fonctionnalité d'exportation de données héritée, consultez le [tableau de comparaison](./howto-export-data.md#comparison-of-legacy-data-export-and-data-export).

Cet article explique comment utiliser la fonctionnalité d’exportation de données dans Azure IoT Central. Cette fonctionnalité vous permet d’exporter vos données en continu vers des instances **Azure Event Hubs**, **Azure Service Bus** ou **Stockage Blob Azure**. L’exportation des données utilise le format JSON et peut inclure des informations de télémétrie, ainsi que des informations sur les appareils et les modèles d’appareil. Utilisez les données exportées pour :

- Insights et analytiques à chaud. Cette option inclut le déclenchement de règles personnalisées dans Azure Stream Analytics, le déclenchement de workflows personnalisés dans Azure Logic Apps ou la transmission des données à transformer via Azure Functions.
- Analytique des chemins à froid, comme les modèles d’apprentissage dans Azure Machine Learning ou l’analyse de tendances à long terme dans Microsoft Power BI.

> [!Note]
> Quand vous activez l’exportation de données, vous obtenez seulement les données à partir de ce moment. Pour le moment, vous ne pouvez pas récupérer les données d’une période pendant laquelle l’exportation de données était désactivée. Pour conserver un historique des données plus étendu, activez l’exportation de données au plus tôt.

## <a name="prerequisites"></a>Prérequis

Vous devez être administrateur de votre application IoT Central, ou disposer d’autorisations d’exportation de données.

## <a name="set-up-export-destination"></a>Définir la destination de l’exportation

Votre destination d’exportation doit exister avant la configuration de l’exportation de données.

### <a name="create-event-hubs-namespace"></a>Créer un espace de noms Event Hubs

En l’absence d’espace de noms Event Hubs existant vers lequel exporter, suivez ces étapes :

1. Créez un [espace de noms Event Hubs dans le Portail Azure](https://ms.portal.azure.com/#create/Microsoft.EventHub). Pour en savoir plus, consultez les [documents sur Azure Event Hubs](../../event-hubs/event-hubs-create.md).

2. Choisissez un abonnement. Vous pouvez exporter des données vers des abonnements autres que celui de votre application IoT Central. Vous vous connecterez à l’aide d’une chaîne de connexion dans ce cas.

3. Créez un Event Hub dans votre espace de noms Event Hubs. Accédez à votre espace de noms, puis sélectionnez **+ Event Hub**, en haut, pour créer une instance Event Hub.

### <a name="create-service-bus-namespace"></a>Créer un espace de noms Service Bus

En l’absence d’espace de noms Service Bus vers lequel exporter, suivez ces étapes :

1. Créez un [espace de noms Service Bus dans le Portail Azure](https://ms.portal.azure.com/#create/Microsoft.ServiceBus.1.0.5). Pour en savoir plus, consultez les [documents sur Azure Service Bus](../../service-bus-messaging/service-bus-create-namespace-portal.md).
2. Choisissez un abonnement. Vous pouvez exporter des données vers des abonnements autres que celui de votre application IoT Central. Vous vous connecterez à l’aide d’une chaîne de connexion dans ce cas.

3. Pour créer une file d’attente ou une rubrique de destination d’exportation, accédez à votre espace de noms Service Bus, puis sélectionnez **+ File d’attente** ou **+ Rubrique**, en haut.

Lorsque vous choisissez un Service Bus comme destination d’exportation, les sessions et la détection des doublons ne doivent pas être activées dans les files d’attente et les rubriques. Si l’une de ces options est activée, certains messages n’arriveront pas dans votre file d’attente ou votre rubrique.

### <a name="create-storage-account"></a>Créer un compte de stockage

En l’absence de compte de stockage Azure vers lequel exporter, suivez ces étapes :

1. Créez un [compte de stockage sur le Portail Azure](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM). Découvrez comment créer des [comptes de stockage Blob Azure](../../storage/blobs/storage-quickstart-blobs-portal.md) ou des [comptes de stockage Azure Data Lake Storage v2](../../storage/common/storage-account-create.md). L’exportation de données peut uniquement écrire des données dans des comptes de stockage qui prennent en charge les objets blob de blocs. La liste suivante répertorie les types de compte de stockage compatibles connus :

    |Niveau de performances|Type de compte|
    |-|-|
    |standard|Usage général v2|
    |standard|Usage général v1|
    |standard|Stockage d'objets blob|
    |Premium|Stockage d’objets blob de blocs|

2. Créez un conteneur dans votre compte de stockage. Accédez à votre compte de stockage. Sous **Service blob**, sélectionnez **Parcourir les objets blob**. Sélectionnez **+ Conteneur**, en haut, pour créer un conteneur.

## <a name="set-up-data-export"></a>Configurer l’exportation de données

Maintenant que vous avez une destination pour exporter les données, suivez ces étapes pour configurer l’exportation des données.

1. Connectez-vous à votre application IoT Central.

2. Dans le volet gauche, sélectionnez **Exportation de données**.

    > [!Tip]
    > Si vous ne voyez pas **Exportation de données** dans le volet gauche, cela signifie que vous n’avez pas les autorisations nécessaires pour configurer l’exportation de données dans votre application. Contactez un administrateur pour configurer l’exportation de données.

3. Sélectionner le bouton **+ New**. Choisissez **Stockage Blob Azure**, **Azure Event Hubs**, **File d’attente Azure Service Bus** ou **Rubrique Azure Service Bus** en tant que destination de l’exportation. Le nombre maximal d’exportations par application est de cinq.

4. Entrez le nom de l’exportation. Dans la zone de liste déroulante, sélectionnez votre **espace de noms**, ou **entrez une chaîne de connexion**.

    - Vous verrez seulement les comptes de stockage, les espaces de noms Event Hubs et les espaces de noms Service Bus se trouvant dans le même abonnement que votre application IoT Central. Si vous souhaitez effectuer l’exportation vers une destination en dehors de cet abonnement, choisissez **Entrer une chaîne de connexion**, puis consultez l’étape 6.
    - Pour les applications créées à l’aide du plan Gratuit, le seul moyen de configurer l’exportation de données est d’utiliser une chaîne de connexion. Les applications du plan Gratuit ne sont associées à aucun abonnement Azure.

    ![Créer un Event Hub](media/howto-export-data-legacy/export-event-hub.png)

5. Choisissez un conteneur, un Event Hub, une file d’attente ou une rubrique dans la zone de liste déroulante.

6. (Facultatif) Si vous avez choisi **Entrer une chaîne de connexion**, une nouvelle zone vous permettant de coller votre chaîne de connexion s’affiche. Pour obtenir votre chaîne de connexion :

    - Pour Event Hubs ou Service Bus, accédez à l’espace de noms correspondant dans le Portail Azure :
        - Pour utiliser une chaîne de connexion pour tout l’espace de noms :
            1. Sous **Paramètres**, sélectionnez **Stratégies d’accès partagé**
            2. Créez une clé, ou choisissez une clé existante qui dispose des autorisations **Envoyer**.
            3. Copiez la chaîne de connexion primaire ou secondaire
        - Pour utiliser la chaîne de connexion d’une instance de hub d’événements, d’une file d’attente Service Bus ou d’une rubrique Service Bus spécifique, accédez à **Entités > Hubs d’événements** ou **Entités > Files d’attente** ou **Entités > Rubriques**. Choisissez une instance spécifique, puis suivez les mêmes étapes que ci-dessus pour obtenir une chaîne de connexion.
    - Pour votre compte de stockage, accédez à celui-ci dans le Portail Azure :
        - Seules les chaînes de connexion de l’intégralité du compte de stockage sont prises en charge. Les chaînes de connexion dont l'étendue se limite à un seul conteneur ne sont pas prises en charge.
          1. Sous **Paramètres**, sélectionnez **Clés d’accès**.
          2. Copiez soit la chaîne de connexion key1 soit la chaîne de connexion key2

    Collez le contenu de la chaîne de connexion. Entrez l'instance ou le **nom du conteneur** en respectant la casse.

7. Sous **Données à exporter**, choisissez les types de données à exporter en définissant le type avec la valeur **Activé**.

8. Pour activer l’exportation de données, vérifiez que le bouton bascule sous **Activé** indique **activé**. Sélectionnez **Enregistrer**.

9. Après quelques minutes, vos données s’affichent à la destination choisie.

## <a name="export-contents-and-format"></a>Exporter le contenu et le format

Les données de télémétrie exportées contiennent l’intégralité du message que vos appareils ont envoyé à IoT Central, et non uniquement les valeurs de télémétrie. Les données des appareils exportés contiennent les modifications apportées aux propriétés et aux métadonnées de tous les appareils, et les modèles d’appareil exportés contiennent les modifications apportées à tous les modèles d’appareil.

Pour Event Hubs et Service Bus, les données sont exportées en quasi-temps réel. Les données résident dans la propriété `body` au format JSON. Consultez les exemples ci-dessous.

Pour le stockage Blob, les données sont exportées une fois par minute et chaque fichier contient le lot de modifications apportées depuis la dernière exportation de fichier. Les données exportées sont placées dans trois dossiers au format JSON. Les chemins d’accès par défaut dans votre compte de stockage sont les suivants :

- Données de télémétrie : _{conteneur}/{id-app}/telemetry/{AAAA}/{MM}/{jj}/{hh}/{mm}/{nom-fichier}_
- Appareils : _{conteneur}/{id-app}/devices/{AAAA}/{MM}/{jj}/{hh}/{mm}/{nom-fichier}_
- Modèles d’appareil : _{conteneur}/deviceTemplates/{AAAA}/{MM}/{jj}/{hh}/{mm}/{nom-fichier}_

Pour parcourir les fichiers exportés dans le Portail Azure, accédez au fichier puis sélectionnez l’onglet **Modifier le blob**.

## <a name="telemetry"></a>Télémétrie

Pour Event Hubs et Service Bus, IoT Central exporte rapidement un nouveau message après l’avoir reçu d’un appareil. Chaque message exporté contient le message complet envoyé par l’appareil dans la propriété « body » au format JSON.

Pour le stockage Blob, les messages sont regroupés par lot et exportés une fois par minute. Les fichiers exportés utilisent le même format que les fichiers de messages exportés par [le routage des messages IoT Hub](../../iot-hub/tutorial-routing.md) vers Stockage Blob.

> [!NOTE]
> Pour le stockage Blob, vérifiez que vos appareils envoient bien des messages avec `contentType: application/JSON` et `contentEncoding:utf-8` (ou `utf-16`, `utf-32`). Pour obtenir un exemple, consultez la [documentation IoT Hub](../../iot-hub/iot-hub-devguide-routing-query-syntax.md#message-routing-query-based-on-message-body).

L’appareil qui a envoyé les données de télémétrie est représenté par l’ID d’appareil (voir les sections suivantes). Pour obtenir les noms des appareils, exportez les données de ceux-ci et corrélez chaque message en utilisant le **connectionDeviceId** qui correspond au **deviceId** du message de l’appareil.

L’exemple suivant montre un message reçu d’une file d’attente ou rubrique Event Hub ou Service Bus :

```json
{
  "temp":81.129693132351775,
  "humid":59.488071477541247,
  "EventProcessedUtcTime":"2020-04-07T09:41:15.2877981Z",
  "PartitionId":0,
  "EventEnqueuedUtcTime":"2020-04-07T09:38:32.7380000Z"
}
```

Ce message n’inclut pas l’ID d’appareil de l’appareil émetteur.

Pour récupérer l’ID de l’appareil à partir des données du message dans une requête Azure Stream Analytics, utilisez la fonction [GetMetadataPropertyValue](/stream-analytics-query/getmetadatapropertyvalue). Par obtenir un exemple, consultez la requête [Étendre Azure IoT Central avec des règles personnalisées à l’aide de Stream Analytics, d’Azure Functions et de SendGrid](./howto-create-custom-rules.md).

Pour récupérer l’ID d’appareil dans un espace de travail Azure Databricks ou Apache Spark, utilisez [systemProperties](https://github.com/Azure/azure-event-hubs-spark/blob/master/docs/structured-streaming-eventhubs-integration.md). Pour obtenir un exemple, consultez l’espace de travail Databricks dans [Étendre Azure IoT Central avec des analyses personnalisées à l’aide d’Azure Databricks](./howto-create-custom-analytics.md).

L’exemple suivant montre un enregistrement exporté vers le stockage Blob :

```json
{
  "EnqueuedTimeUtc":"2019-09-26T17:46:09.8870000Z",
  "Properties":{

  },
  "SystemProperties":{
    "connectionDeviceId":"<deviceid>",
    "connectionAuthMethod":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
    "connectionDeviceGenerationId":"637051167384630591",
    "contentType":"application/json",
    "contentEncoding":"utf-8",
    "enqueuedTime":"2019-09-26T17:46:09.8870000Z"
  },
  "Body":{
    "temp":49.91322758395974,
    "humid":49.61214852573155,
    "pm25":25.87332214661367
  }
}
```

## <a name="devices"></a>Appareils

Chaque message ou enregistrement d’instantané représente un ou plusieurs changements apportés à un appareil, ainsi que ses propriétés d’appareil et ses propriétés cloud, depuis le dernier message exporté. Le message contient les éléments suivants :

- `id` de l’appareil dans IoT Central
- `displayName` de l’appareil
- L’ID du modèle d’appareil dans `instanceOf`
- L’indicateur `simulated`, true si l’appareil est un appareil simulé
- L’indicateur `provisioned`, true si l’appareil a bien été provisionné
- L’indicateur `approved`, true si l’appareil a reçu une approbation pour l’envoi de données
- Valeurs de propriétés
- Les `properties`, y compris les valeurs de propriétés d’appareil et de cloud

Les appareils supprimés ne sont pas exportés. Actuellement, il n’y a aucun indicateur dans les messages exportés pour les appareils supprimés.

Pour Event Hubs et Service Bus, IoT Central envoie des messages contenant les données de l’appareil à la file d’attente ou à la rubrique Event Hub ou Service Bus en quasi-temps réel.

Pour le stockage Blob, un nouvel instantané contenant toutes les modifications apportées depuis la dernière écriture est exporté une fois par minute.

L’exemple de message suivant affiche des informations concernant les données d’appareils et de propriétés dans la file d’attente ou la rubrique Event Hub ou Service Bus :

```json
{
  "body":{
    "id": "<device Id>",
    "etag": "<etag>",
    "displayName": "Sensor 1",
    "instanceOf": "<device template Id>",
    "simulated": false,
    "provisioned": true,
    "approved": true,
    "properties": {
        "sensorComponent": {
            "setTemp": "30",
            "fwVersion": "2.0.1",
            "status": { "first": "first", "second": "second" },
            "$metadata": {
                "setTemp": {
                    "desiredValue": "30",
                    "desiredVersion": 3,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 3
                },
                "fwVersion": { "ackVersion": 3 },
                "status": {
                    "desiredValue": {
                        "first": "first",
                        "second": "second"
                    },
                    "desiredVersion": 2,
                    "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                    "ackVersion": 2
                }
            },
            
        }
    },
    "installDate": { "installDate": "2020-02-01" }
},
  "annotations":{
    "iotcentral-message-source":"devices",
    "x-opt-partition-key":"<partitionKey>",
    "x-opt-sequence-number":39740,
    "x-opt-offset":"<offset>",
    "x-opt-enqueued-time":1539274959654
  },
  "partitionKey":"<partitionKey>",
  "sequenceNumber":39740,
  "enqueuedTimeUtc":"2020-02-01T18:14:49.3820326Z",
  "offset":"<offset>"
}
```

Cet instantané est un exemple de message qui affiche les données des appareils et des propriétés dans le stockage Blob. Les fichiers exportés contiennent une seule ligne par enregistrement.

```json
{
  "id": "<device Id>",
  "etag": "<etag>",
  "displayName": "Sensor 1",
  "instanceOf": "<device template Id>",
  "simulated": false,
  "provisioned": true,
  "approved": true,
  "properties": {
      "sensorComponent": {
          "setTemp": "30",
          "fwVersion": "2.0.1",
          "status": { "first": "first", "second": "second" },
          "$metadata": {
              "setTemp": {
                  "desiredValue": "30",
                  "desiredVersion": 3,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 3
              },
              "fwVersion": { "ackVersion": 3 },
              "status": {
                  "desiredValue": {
                      "first": "first",
                      "second": "second"
                  },
                  "desiredVersion": 2,
                  "desiredTimestamp": "2020-02-01T17:15:08.9284049Z",
                  "ackVersion": 2
              }
          },
          
      }
  },
  "installDate": { "installDate": "2020-02-01" }
}
```

## <a name="device-templates"></a>Modèles d’appareil

Chaque message ou enregistrement d’instantané représente un ou plusieurs changements apportés à un modèle d’appareil publié depuis le dernier message exporté. Les informations envoyées dans chaque message ou enregistrement incluent :

- `id` du modèle d’appareil qui correspond à `instanceOf` du flux d’appareil ci-dessus
- `displayName` du modèle d’appareil
- L’appareil `capabilityModel`, y compris ses `interfaces`, et les définitions de télémétrie, de propriétés et de commandes
- Définitions `cloudProperties`
- Remplacements et valeurs initiales, alignés sur le `capabilityModel`

Les modèles d’appareils supprimés ne sont pas exportés. Actuellement, il n’y a aucun indicateur dans les messages exportés pour les modèles d’appareil supprimé.

Pour Event Hubs et Service Bus, IoT Central envoie des messages contenant des données de modèle d’appareil à la file d’attente ou à la rubrique Event Hub ou Service Bus en quasi-temps réel.

Pour le stockage Blob, un nouvel instantané contenant toutes les modifications apportées depuis la dernière écriture est exporté une fois par minute.

Cet exemple montre un message concernant les données de modèles d’appareil dans la file d’attente ou rubrique Event Hub ou Service Bus :

```json
{
  "body":{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "sensorComponent",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorConfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  },
    "annotations":{
      "iotcentral-message-source":"deviceTemplates",
      "x-opt-partition-key":"<partitionKey>",
      "x-opt-sequence-number":25315,
      "x-opt-offset":"<offset>",
      "x-opt-enqueued-time":1539274985085
    },
    "partitionKey":"<partitionKey>",
    "sequenceNumber":25315,
    "enqueuedTimeUtc":"2019-10-02T16:23:05.085Z",
    "offset":"<offset>"
  }
}
```

Cet exemple d’instantané montre un message contenant les données d’appareils et de propriétés dans le stockage Blob. Les fichiers exportés contiennent une seule ligne par enregistrement.

```json
{
      "id": "<device template id>",
      "etag": "<etag>",
      "types": ["DeviceModel"],
      "displayName": "Sensor template",
      "capabilityModel": {
          "@id": "<capability model id>",
          "@type": ["CapabilityModel"],
          "contents": [],
          "implements": [
              {
                  "@id": "<component Id>",
                  "@type": ["InterfaceInstance"],
                  "name": "Sensor component",
                  "schema": {
                      "@id": "<interface Id>",
                      "@type": ["Interface"],
                      "displayName": "Sensor interface",
                      "contents": [
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry"],
                              "displayName": "Humidity",
                              "name": "humidity",
                              "schema": "double"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Telemetry", "SemanticType/Event"],
                              "displayName": "Error event",
                              "name": "error",
                              "schema": "integer"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Set temperature",
                              "name": "setTemp",
                              "writable": true,
                              "schema": "integer",
                              "unit": "Units/Temperature/fahrenheit",
                              "initialValue": "30"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Firmware version read only",
                              "name": "fwversion",
                              "schema": "string"
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Property"],
                              "displayName": "Display status",
                              "name": "status",
                              "writable": true,
                              "schema": {
                                  "@id": "urn:testInterface:status:obj:ka8iw8wka:1",
                                  "@type": ["Object"]
                              }
                          },
                          {
                              "@id": "<id>",
                              "@type": ["Command"],
                              "request": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Configuration",
                                  "name": "config",
                                  "schema": "string"
                              },
                              "response": {
                                  "@id": "<id>",
                                  "@type": ["SchemaField"],
                                  "displayName": "Response",
                                  "name": "response",
                                  "schema": "string"
                              },
                              "displayName": "Configure sensor",
                              "name": "sensorconfig"
                          }
                      ]
                  }
              }
          ],
          "displayName": "Sensor capability model"
      },
      "solutionModel": {
          "@id": "<id>",
          "@type": ["SolutionModel"],
          "cloudProperties": [
              {
                  "@id": "<id>",
                  "@type": ["CloudProperty"],
                  "displayName": "Install date",
                  "name": "installDate",
                  "schema": "dateTime",
                  "valueDetail": {
                      "@id": "<id>",
                      "@type": ["ValueDetail/DateTimeValueDetail"]
                  }
              }
          ]
      }
  }
```

## <a name="data-format-change-notice"></a>Avis de modification du format de données

> [!Note]
> Le format des données des flux de télémétrie n’est pas affecté par cette modification. Seuls les flux de données des appareils et des modèles d’appareils sont affectés.

Si vous avez déjà une exportation de données dans votre application en préversion, dans laquelle les flux *Appareils* et *Modèles d’appareil* sont activés, mettez à jour votre exportation d’ici le **30 juin 2020**. Cette condition s’applique aux exportations vers le stockage Blob Azure, Azure Event Hubs et Azure Service Bus.

À partir du 3 février 2020, toutes les nouvelles exportations des applications où sont activés les appareils et les modèles d’appareils auront le format de données décrit plus haut. Toutes les exportations créées avant cette date sont conservées dans l’ancien format de données jusqu’au 30 juin 2020. Après cela, ces exportations seront automatiquement migrées au nouveau format de données. Le nouveau format de données correspond aux objets de type [appareil](/rest/api/iotcentral/devices/get), [propriété d’appareil](/rest/api/iotcentral/devices/getproperties), [propriété cloud d’appareil](/rest/api/iotcentral/devices/getcloudproperties) et [modèle d’appareil](/rest/api/iotcentral/devicetemplates/get) dans l’API publique IoT Central.

Pour les **appareils**, les différences notables entre l’ancien format de données et le nouveau sont les suivantes :
- L’`@id` a été supprimé pour les appareils, `deviceId` a été remplacé par `id` 
- L’indicateur `provisioned` a été ajouté pour décrire l’état de provisionnement de l’appareil
- L’indicateur `approved` a été ajouté pour décrire l’état d’approbation de l’appareil
- Les `properties`, y compris les propriétés d’appareil et les propriétés cloud, correspondent à des entités de l’API publique

Pour les **modèles d’appareil**, les différences notables entre l’ancien format de données et le nouveau sont les suivantes :

- L’`@id` des modèles d’appareil a été remplacé par `id`
- Le `@type` des modèles d’appareil a été remplacé par `types` et correspond désormais à un tableau

### <a name="devices-format-deprecated-as-of-3-february-2020"></a>Appareils (format déprécié à partir du 3 février 2020)

```json
{
  "@id":"<id-value>",
  "@type":"Device",
  "displayName":"Airbox",
  "data":{
    "$cloudProperties":{
        "Color":"blue"
    },
    "EnvironmentalSensor":{
      "thsensormodel":{
        "reported":{
          "value":"Neque quia et voluptatem veritatis assumenda consequuntur quod.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "pm25sensormodel":{
        "reported":{
          "value":"Aut alias odio.",
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    },
    "urn_azureiot_DeviceManagement_DeviceInformation":{
      "totalStorage":{
        "reported":{
          "value":27900.9730905171,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      },
      "totalMemory":{
        "reported":{
          "value":4667.82916715811,
          "$lastUpdatedTimestamp":"2019-09-30T20:35:43.8478978Z"
        }
      }
    }
  },
  "instanceOf":"<template-id>",
  "deviceId":"<device-id>",
  "simulated":true
}
```

### <a name="device-templates-format-deprecated-as-of-3-february-2020"></a>Modèles d’appareil (format déprécié à partir du 3 février 2020)

```json
{
  "@id":"<template-id>",
  "@type":"DeviceModelDefinition",
  "displayName":"Airbox",
  "capabilityModel":{
    "@id":"<id>",
    "@type":"CapabilityModel",
    "implements":[
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"EnvironmentalSensor",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "comment":"Requires temperature and humidity sensors.",
          "description":"Provides functionality to report temperature, humidity. Provides telemetry, commands and read-write properties",
          "displayName":"Environmental Sensor",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current temperature on the device",
              "displayName":"Temperature",
              "name":"temp",
              "schema":"double",
              "unit":"Units/Temperature/celsius",
              "valueDetail":{
                "@id":"<id>",
                "@type":"ValueDetail/NumberValueDetail",
                "minValue":{
                  "@value":"50"
                }
              },
              "visualizationDetail":{
                "@id":"<id>",
                "@type":"VisualizationDetail"
              }
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current humidity on the device",
              "displayName":"Humidity",
              "name":"humid",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Telemetry",
              "description":"Current PM2.5 on the device",
              "displayName":"PM2.5",
              "name":"pm25",
              "schema":"integer"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"T&H Sensor Model Name",
              "displayName":"T&H Sensor Model",
              "name":"thsensormodel",
              "schema":"string"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "description":"PM2.5 Sensor Model Name",
              "displayName":"PM2.5 Sensor Model",
              "name":"pm25sensormodel",
              "schema":"string"
            }
          ]
        }
      },
      {
        "@id":"<id>",
        "@type":"InterfaceInstance",
        "name":"urn_azureiot_DeviceManagement_DeviceInformation",
        "schema":{
          "@id":"<id>",
          "@type":"Interface",
          "displayName":"Device information",
          "contents":[
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available storage on the device in kilobytes. Ex. 20480000 kilobytes.",
              "displayName":"Total storage",
              "name":"totalStorage",
              "displayUnit":"kilobytes",
              "schema":"long"
            },
            {
              "@id":"<id>",
              "@type":"Property",
              "comment":"Total available memory on the device in kilobytes. Ex. 256000 kilobytes.",
              "displayName":"Total memory",
              "name":"totalMemory",
              "displayUnit":"kilobytes",
              "schema":"long"
            }
          ]
        }
      }
    ],
    "displayName":"AAEONAirbox52"
  },
  "solutionModel":{
    "@id":"<id>",
    "@type":"SolutionModel",
    "cloudProperties":[
      {
        "@id":"<id>",
        "@type":"CloudProperty",
        "displayName":"Color",
        "name":"Color",
        "schema":"string",
        "valueDetail":{
          "@id":"<id>",
          "@type":"ValueDetail/StringValueDetail"
        },
        "visualizationDetail":{
          "@id":"<id>",
          "@type":"VisualizationDetail"
        }
      }
    ]
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment exporter vos données vers Azure Event Hubs, Azure Service Bus ou Stockage Blob Azure, passez à l’étape suivante :

> [!div class="nextstepaction"]
> [Guide pratique pour exécuter des tâches analytiques personnalisées avec Databricks](./howto-create-custom-analytics.md)