---
title: Publier et s’abonner avec Azure IoT Edge | Microsoft Docs
description: Utiliser le répartiteur IoT Edge MQTT pour publier des messages et s’y abonner
services: iot-edge
keywords: ''
author: kgremban
ms.author: kgremban
ms.reviewer: ebertra
ms.date: 11/09/2020
ms.topic: conceptual
ms.service: iot-edge
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 005830575ba7f45d30fed71a73e7a419e4d98220
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922583"
---
# <a name="publish-and-subscribe-with-azure-iot-edge"></a>Publier et s’abonner avec Azure IoT Edge

Vous pouvez utiliser le répartiteur IoT Edge MQTT pour publier des messages et vous y abonner. Cet article explique comment se connecter à ce répartiteur, publier et des messages et s’y abonner sur des rubriques définies par l’utilisateur et utiliser les primitives de messagerie IoT Hub. Le répartiteur IoT Edge MQTT est intégré au hub IoT Edge. Pour plus d’informations, consultez [les fonctionnalités de répartition du hub IoT Edge](iot-edge-runtime.md).

> [!NOTE]
> Le répartiteur IoT Edge MQTT est actuellement disponible en préversion publique.

## <a name="pre-requisites"></a>Conditions préalables

- Un compte Azure avec un abonnement valide
- [Azure CLI](/cli/azure/) avec l’extension CLI `azure-iot` installée. Pour plus d’informations, consultez [les étapes d’installation de l’extension Azure IoT pour Azure CLI](/cli/azure/azure-cli-reference-for-iot).
- Un **hub IoT** avec la référence SKU F1, S1, S2 ou S3.
- Disposer d’un **appareil IoT Edge avec la version 1.2 ou supérieure**. Étant donné que le répartiteur IoT Edge MQTT est actuellement en préversion publique, définissez les variables d’environnement suivantes sur true dans le conteneur edgeHub pour activer le répartiteur MQTT :

   | Nom | Valeur |
   | - | - |
   | `experimentalFeatures__enabled` | `true` |
   | `experimentalFeatures__mqttBrokerEnabled` | `true` |

- Les **clients Mosquitto** installés sur l’appareil IoT Edge. Cet article utilise les clients Mosquitto populaires [MOSQUITTO_PUB](https://mosquitto.org/man/mosquitto_pub-1.html) et [MOSQUITTO_SUB](https://mosquitto.org/man/mosquitto_sub-1.html). D’autres clients MQTT peuvent être utilisés à la place. Pour installer les clients Mosquitto sur un appareil Ubuntu, exécutez la commande suivante :

    ```cmd
    sudo apt-get update && sudo apt-get install mosquitto-clients
    ```

    N’installez pas le serveur Mosquitto, car il peut provoquer le blocage des ports MQTT (8883 et 1883) et un conflit avec le hub IoT Edge.

## <a name="connect-to-iot-edge-hub"></a>Connexion au hub IoT Edge

La connexion au hub IoT Edge suit les mêmes étapes que celles décrites dans [l’article sur la connexion à IoT Hub avec un client MQTT générique](../iot-hub/iot-hub-mqtt-support.md) ou dans [l’article de la description conceptuelle du hub IoT Edge](iot-edge-runtime.md). Ces étapes sont les suivantes :

1. Le client MQTT peut établir une *connexion sécurisée* avec le hub IoT Edge à l’aide du protocole TLS (Transport Layer Security)
2. Le client MQTT *s’authentifie* au hub IoT Edge
3. Le hub IoT Edge *autorise* le client MQTT selon sa stratégie d’autorisation

### <a name="secure-connection-tls"></a>Connexion sécurisée (TLS)

Le protocole TLS (Transport Layer Security) est utilisé pour établir une communication chiffrée entre le client et le hub IoT Edge.

Pour désactiver TLS, utilisez le port 1883 (MQTT) et liez le conteneur edgeHub au port 1883.

Pour activer TLS, si un client se connecte sur le port 8883 (MQTTS) au répartiteur MQTT, un canal TLS est initié. Le répartiteur envoie sa chaîne de certificats que le client doit valider. Pour valider la chaîne de certificats, le certificat racine du répartiteur MQTT doit être installé en tant que certificat approuvé sur le client. Si le certificat racine n’est pas approuvé, la bibliothèque cliente est rejetée par le répartiteur MQTT avec une erreur de vérification du certificat. Les étapes à suivre pour installer ce certificat racine du répartiteur sur le client sont les mêmes que celles décrites dans le cas de la [passerelle transparente](how-to-create-transparent-gateway.md) et sont décrites dans la documentation de [préparation d’un appareil en aval](how-to-connect-downstream-device.md#prepare-a-downstream-device).

### <a name="authentication"></a>Authentification

Pour qu’un client MQTT s’authentifie lui-même, il doit d’abord envoyer un paquet CONNECT au répartiteur MQTT pour établir une connexion dans son nom. Ce paquet fournit trois éléments d’informations d’authentification : `client identifier`, `username` et `password` :

- Le champ `client identifier` est le nom de l’appareil ou du module dans IoT Hub. Il utilise la syntaxe suivante :

  - Pour un appareil : `<device_name>`

  - Pour un module : `<device_name>/<module_name>`

   Pour se connecter au répartiteur MQTT, un appareil ou un module doit être inscrit dans IoT Hub.

   Le répartiteur n’autorise pas les connexions de plusieurs clients à l’aide des mêmes informations d’identification. Le répartiteur déconnecte le client déjà connecté si un deuxième client se connecte à l’aide des mêmes informations d’identification.

- Le champ `username` est dérivé du nom de l’appareil ou du module, et du nom IoTHub auquel appartient l’appareil à l’aide de la syntaxe suivante :

  - Pour un appareil : `<iot_hub_name>.azure-devices.net/<device_name>/?api-version=2018-06-30`

  - Pour un module : `<iot_hub_name>.azure-devices.net/<device_name>/<module_name>/?api-version=2018-06-30`

- Le champ `password` du paquet CONNECT dépend du mode d’authentification :

  - Quand vous utilisez l’[authentification avec des clés symétriques](how-to-authenticate-downstream-device.md#symmetric-key-authentication), le champ `password` est un jeton SAS.
  - Quand vous utilisez l’[authentification par certificat X.509 autosigné](how-to-authenticate-downstream-device.md#x509-self-signed-authentication), le champ `password` n’est pas présent. Dans ce mode d’authentification, un canal TLS est requis. Le client doit se connecter au port 8883 pour établir une connexion TLS. Pendant l’établissement d’une liaison TLS, le répartiteur MQTT demande un certificat client. Ce certificat est utilisé pour vérifier l’identité du client et, par conséquent, le champ `password` n’est pas nécessaire plus tard lors de l’envoi du paquet CONNECT. L’envoi d’un certificat client et du champ de mot de passe provoque une erreur et la connexion est fermée. Les bibliothèques MQTT et les bibliothèques clientes TLS ont généralement un moyen d’envoyer un certificat client lors de l’initialisation d’une connexion. Vous pouvez voir un exemple pas à pas dans la section [Utilisation du certificat X509 pour l’authentification du client](how-to-authenticate-downstream-device.md#x509-self-signed-authentication).

Les modules déployés par IoT Edge utilisent [l’authentification avec des clés symétriques](how-to-authenticate-downstream-device.md#symmetric-key-authentication) et peuvent appeler [l’API de charge de travail IoT Edge](https://github.com/Azure/iotedge/blob/40f10950dc65dd955e20f51f35d69dd4882e1618/edgelet/workload/README.md) locale pour obtenir un jeton SAS de manière programmatique, même en mode hors connexion.

### <a name="authorization"></a>Autorisation

Une fois qu’un client MQTT est authentifié sur le hub IoT Edge, il doit être autorisé à se connecter. Une fois connecté, il doit être autorisé à publier dans des rubriques spécifiques ou à s’y abonner. Ces autorisations sont accordées par le hub IoT Edge en fonction de sa stratégie d’autorisation. La stratégie d’autorisation est un ensemble d’instructions exprimé sous la forme d’une structure JSON qui est envoyée au hub IoT Edge via son jumeau. Modifiez un jumeau de hub IoT Edge pour configurer sa stratégie d’autorisation.

> [!NOTE]
> Pour la préversion publique, la modification des stratégies d’autorisation du répartiteur MQTT est uniquement disponible via Visual Studio, Visual Studio Code ou l’interface de ligne de commande Azure. Actuellement, le portail Azure ne prend pas en charge la modification du jumeau du hub IoT Edge et de sa stratégie d’autorisation.

Chaque instruction de la stratégie d’autorisation est une combinaison des effets `identities`, `allow` ou `deny`, de `operations` et `resources` :

- `identities` décrit l’objet de la stratégie. Il doit correspondre au `client identifier` envoyé par les clients dans leur paquet CONNECT.
- Les effets `allow` ou `deny` définissent s’il faut autoriser ou refuser des opérations.
- `operations` définit les actions à autoriser. `mqtt:connect`, `mqtt:publish` et `mqtt:subscribe` sont les trois actions prises en charge aujourd’hui.
- `resources` définit l’objet de la stratégie. Il peut s’agir d’une rubrique ou d’un modèle de rubrique défini avec des [caractères génériques MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107).

Voici un exemple de stratégie d’autorisation qui, explicitement, n’autorise pas le client « rogue_client » à se connecter, autorise tous les clients Azure IoT à se connecter et autorise « sensor_1 » à publier dans la rubrique `events/alerts`.

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "routes":{
            "Route1":"FROM /messages/* INTO $upstream"
         },
         "storeAndForwardConfiguration":{
            "timeToLiveSecs":7200
         },
         "mqttBroker":{
            "authorizations":[
               {
                  "identities":[
                     "rogue_client"
                  ],
                  "deny":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities":[
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities":[
                     "sensor_1"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:publish"
                        ],
                        "resources":[
                           "events/alerts"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

Voici quelques éléments à prendre en compte lors de l’écriture de votre stratégie d’autorisation :

- Elle requiert le schéma de jumeau `$edgeHub` version 1.2
- Par défaut, toutes les opérations sont refusées.
- Les instructions d’autorisation sont évaluées dans l’ordre dans lequel elles apparaissent dans la définition JSON. Celle-ci commence par examiner `identities` puis sélectionne les premières instructions Allow ou Deny qui correspondent à la demande. En cas de conflit entre les instructions Allow et Deny, l’instruction Deny gagne.
- Plusieurs variables (par exemple, substitutions) peuvent être utilisées dans la stratégie d’autorisation :
    - `{{iot:identity}}` représente l’identité du client actuellement connecté. Par exemple, une identité d’appareil comme `myDevice` ou une identité de module comme `myEdgeDevice/SampleModule`.
    - `{{iot:device_id}}` représente l’identité de l’appareil actuellement connecté. Par exemple, une identité d’appareil comme `myDevice` ou l’identité de l’appareil où un module s’exécute comme `myEdgeDevice`.
    - `{{iot:module_id}}` représente l’identité du module actuellement connecté. Cette variable est vide pour les appareils connectés, ou une identité de module comme `SampleModule`.
    - `{{iot:this_device_id}}` représente l’identité de l’appareil IoT Edge exécutant la stratégie d’autorisation. Par exemple : `myIoTEdgeDevice`.

Les rubriques d’autorisations pour le hub IoT sont traitées de manière légèrement différente que les rubriques définies par l’utilisateur. Voici les points importants à retenir :

- Les appareils ou les modules Azure IoT ont besoin d’une règle d’autorisation explicite pour se connecter au répartiteur MQTT du hub IoT Edge. Une stratégie d’autorisation de connexion par défaut est fournie ci-dessous.
- Les appareils ou les modules Azure IoT peuvent accéder à leurs propres rubriques de hub IoT par défaut sans règle d’autorisation explicite. Toutefois, les autorisations proviennent de relations parent/enfant dans ce cas, et ces relations doivent être définies. Les modules IoT Edge sont automatiquement définis comme enfants de leur appareil IoT Edge, mais les appareils doivent être explicitement définis comme enfants de leur passerelle IoT Edge.

Voici une stratégie d’autorisation par défaut qui peut être utilisée pour permettre à tous les modules ou appareils Azure IoT de se **connecter** au répartiteur :

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "mqttBroker":{
            "authorizations":[
               {
                  "identities": [
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

Maintenant que vous savez comment vous connecter au répartiteur IoT Edge MQTT, voyons comment il peut être utilisé pour publier des messages et s’y abonner, premièrement dans les rubriques définies par l’utilisateur, puis dans les rubriques du hub IoT et enfin sur un autre répartiteur MQTT.

## <a name="publish-and-subscribe-on-user-defined-topics"></a>Publier dans des rubriques définies par l’utilisateur et s’y abonner

Dans cet article, vous allez utiliser un client nommé **sub_client** qui s’abonne à une rubrique et un autre client nommé **pub_client** qui publie dans une rubrique. Nous allons utiliser [l’authentification par clé symétrique](how-to-authenticate-downstream-device.md#symmetric-key-authentication), mais vous pouvez le faire avec [l’authentification par certificat X.509 auto-signé](how-to-authenticate-downstream-device.md#x509-self-signed-authentication) ou [l’authentification par certificat X.509 signé par autorité de certification](./how-to-authenticate-downstream-device.md#x509-ca-signed-authentication).

### <a name="create-publisher-and-subscriber-clients"></a>Créer des clients Serveur de publication et Abonné

Créez deux appareils IoT dans IoT Hub et obtenez leur mot de passe. À l’aide d’Azure CLI de votre terminal :

1. Créez deux appareils IoT dans IoT Hub, apparentez-les à votre appareil IoT Edge :

    ```azurecli-interactive
    az iot hub device-identity create --device-id  sub_client --hub-name <iot_hub_name> --pd <edge_device_id>
    az iot hub device-identity create --device-id  pub_client --hub-name <iot_hub_name> --pd <edge_device_id>
    ```

2. Obtenez leurs mots de passe en générant un jeton SAS :

    - Pour un appareil :
    
       ```azurecli-interactive
       az iot hub generate-sas-token -n <iot_hub_name> -d <device_name> --key-type primary --du 3600
       ```
    
       où 3600 est la durée du jeton SAS en secondes (par exemple, 3600 = 1 heure).
    
    - Pour un module :
    
       ```azurecli-interactive
       az iot hub generate-sas-token -n <iot_hub_name> -d <device_name> -m <module_name> --key-type primary --du 3600
       ```
    
       où 3600 est la durée du jeton SAS en secondes (par exemple, 3600 = 1 heure).

3. Copiez le jeton SAS, qui est la valeur correspondant à la clé « sas » dans la sortie. Voici un exemple de sortie de la commande Azure CLI ci-dessus :

    ```
    {
       "sas": "SharedAccessSignature sr=example.azure-devices.net%2Fdevices%2Fdevice_1%2Fmodules%2Fmodule_a&sig=H5iMq8ZPJBkH3aBWCs0khoTPdFytHXk8VAxrthqIQS0%3D&se=1596249190"
    }
    ```

### <a name="authorize-publisher-and-subscriber-clients"></a>Autoriser des clients Serveur de publication et Abonné

Pour autoriser le serveur de publication et l’abonné, modifiez le jumeau du hub IoT Edge à l’aide d’Azure CLI, de Visual Studio ou de Visual Studio Code pour inclure la stratégie d’autorisation suivante :

```json
{
   "$edgeHub":{
      "properties.desired":{
         "schemaVersion":"1.2",
         "mqttBroker":{
            "authorizations":[
               {
                  "identities": [
                     "{{iot:identity}}"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:connect"
                        ]
                     }
                  ]
               },
               {
                  "identities": [
                     "<iot_hub_name>.azure-devices.net/sub_client"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:subscribe"
                        ],
                        "resources":[
                           "test_topic"
                        ]
                     }
                  ],
               },
               {
                  "identities": [
                     "<iot_hub_name>.azure-devices.net/pub_client"
                  ],
                  "allow":[
                     {
                        "operations":[
                           "mqtt:publish"
                        ],
                        "resources":[
                           "test_topic"
                        ]
                     }
                  ]
               }
            ]
         }
      }
   }
}
```

### <a name="symmetric-keys-authentication-without-tls"></a>Authentification avec des clés symétriques sans TLS

#### <a name="subscribe"></a>S’abonner

Connectez votre client MQTT **sub_client** au répartiteur MQTT et abonnez-vous à `test_topic` en exécutant la commande suivante sur votre appareil IoT Edge :

```bash
mosquitto_sub \
    -t "test_topic" \
    -i "sub_client" \
    -u "<iot_hub_name>.azure-devices.net/sub_client/?api-version=2018-06-30" \
    -P "<sas_token>" \
    -h "<edge_device_address>" \
    -V mqttv311 \
    -p 1883
```

`<edge_device_address>` = `localhost` dans cet exemple puisque le client s’exécute sur le même appareil qu’IoT Edge.

Notez que le port 1883 (MQTT), sans TLS, est utilisé dans ce premier exemple. Un autre exemple avec le port 8883 (MQTTS), avec TLS activé, est présenté dans la section suivante.

Le client MQTT **sub_client** est maintenant démarré et attend les messages entrants sur `test_topic`.

#### <a name="publish"></a>Publish

Connectez votre client MQTT **pub_client** au répartiteur MQTT et publiez un message sur le même `test_topic` que ci-dessus en exécutant la commande suivante sur votre appareil IoT Edge à partir d’un autre terminal :

```bash
mosquitto_pub \
    -t "test_topic" \
    -i "pub_client" \
    -u "<iot_hub_name>.azure-devices.net/pub_client/?api-version=2018-06-30" \
    -P "<sas_token>" \
    -h "<edge_device_address>" \
    -V mqttv311 \
    -p 1883 \
    -m "hello"
```

`<edge_device_address>` = `localhost` dans cet exemple puisque le client s’exécute sur le même appareil qu’IoT Edge.

Lors de l’exécution de la commande, le client MQTT **sub_client** reçoit le message « hello ».

### <a name="symmetric-keys-authentication-with-tls"></a>Authentification avec des clés symétriques avec TLS

Pour activer TLS, le port doit passer de 1883 (MQTT) à 8883 (MQTTS) et les clients doivent disposer du certificat racine du répartiteur MQTT pour pouvoir valider la chaîne de certificats envoyée par le répartiteur MQTT. Pour ce faire, suivez les étapes décrites dans la section [Connexion sécurisée (TLS)](#secure-connection-tls).

Étant donné que les clients s’exécutent sur le même appareil que le répartiteur MQTT dans l’exemple ci-dessus, les mêmes étapes s’appliquent à l’activation du protocole TLS. Il faut simplement modifier le numéro de port de 1883 (MQTT) à 8883 (MQTTS).

## <a name="publish-and-subscribe-on-iot-hub-topics"></a>Publier dans des rubriques IoT Hub et s’y abonner

Les [SDK d’appareil Azure IoT](https://github.com/Azure/azure-iot-sdks) permettent déjà aux clients d’effectuer des opérations IoT Hub, mais ils n’autorisent pas la publication/l’abonnement sur des rubriques définies par l’utilisateur. Les opérations IoT Hub peuvent être effectuées à l’aide de n’importe quel client MQTT à l’aide de la sémantique de publication et d’abonnement tant que les protocoles de primitives IoT Hub sont respectés. Nous allons examiner les spécificités pour comprendre le fonctionnement de ces protocoles.

### <a name="send-telemetry-data-to-iot-hub"></a>Envoyer des données de télémétrie à IoT Hub

L’envoi de données de télémétrie à IoT Hub est similaire à la publication sur une rubrique définie par l’utilisateur, mais en utilisant une rubrique IoT Hub spécifique :

- Pour un appareil, les données de télémétrie sont envoyées sur la rubrique : `devices/<device_name>/messages/events`
- Pour un module, les données de télémétrie sont envoyées sur la rubrique : `devices/<device_name>/<module_name>/messages/events`

En outre, créez un itinéraire comme `FROM /messages/* INTO $upstream` pour envoyer des données de télémétrie à partir du répartiteur IoT Edge MQTT vers IoT Hub. Pour en savoir plus sur le routage, consultez [Déclarer des itinéraires](module-composition.md#declare-routes).

### <a name="get-twin"></a>Obtenir un jumeau

L’obtention du jumeau de l’appareil/du module n’est pas un modèle MQTT classique. Le client doit émettre une demande concernant le jumeau. Celle-ci sera servie par IoT Hub.

Pour pouvoir recevoir des jumeaux, le client doit s’abonner à une rubrique `$iothub/twin/res/#` IoT Hub spécifique. Ce nom de rubrique est hérité d’IoT Hub et tous les clients doivent s’abonner à la même rubrique. Cela ne signifie pas que les appareils ou les modules reçoivent le jumeau l’un de l’autre. IoT Hub et le hub IoT Edge savent quel jumeau doit être livré et où, même si tous les appareils écoutent le même nom de rubrique. 

Une fois l’abonnement effectué, le client doit demander le jumeau en publiant un message sur une rubrique `$iothub/twin/GET/?rid=<request_id>/#` IoT Hub spécifique où `<request_id>` est un identificateur arbitraire. IoT Hub envoie ensuite sa réponse avec les données demandées sur la rubrique `$iothub/twin/res/200/?rid=<request_id>`, à laquelle le client s’abonne. C’est ainsi qu’un client peut jumeler ses demandes aux réponses.

### <a name="receive-twin-patches"></a>Recevoir des correctifs du jumeau

Pour recevoir des correctifs du jumeau, un client doit s’abonner à une rubrique `$iothub/twin/PATCH/properties/desired/#` IoT Hub spécifique. Une fois l’abonnement effectué, le client reçoit les correctifs du jumeau envoyés par IoT Hub sur cette rubrique. 

### <a name="receive-direct-methods"></a>Recevoir des méthodes directes

La réception d’une méthode directe est similaire à la réception des jumeaux complets, en ajoutant le fait que le client doit confirmer qu’il a reçu l’appel. Tout d’abord, le client s’abonne à la rubrique spéciale de hub IoT `$iothub/methods/POST/#`. Ensuite, une fois qu’une méthode directe est reçue sur cette rubrique, le client doit extraire l’identificateur de la demande `rid` à partir de la sous-rubrique sur laquelle la méthode directe est reçue et enfin publier un message de confirmation sur la rubrique `$iothub/methods/res/200/<request_id>` IoT Hub spéciale.

### <a name="send-direct-methods"></a>Envoyer des méthodes directes

L’envoi d’une méthode directe est un appel HTTP qui ne s’effectue pas par le biais du répartiteur MQTT. Pour envoyer une méthode directe à IoT Hub, consultez [Comprendre et appeler des méthodes directes](../iot-hub/iot-hub-devguide-direct-methods.md). Pour envoyer une méthode directe localement à un autre module, consultez cet [exemple d’appel de méthode directe du SDK C# Azure IoT](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/src/ModuleClient.cs#L597).

## <a name="publish-and-subscribe-between-mqtt-brokers"></a>Publier et s’abonner entre des répartiteurs MQTT

Pour connecter deux répartiteurs MQTT, le hub IoT Edge inclut un pont MQTT. Un pont MQTT est couramment utilisé pour connecter un répartiteur MQTT en cours d’exécution à un autre répartiteur MQTT. Seul un sous-ensemble du trafic local est envoyé (push) vers un autre répartiteur.

> [!NOTE]
> Actuellement, le pont du hub IoT Edge ne peut être utilisé qu’entre des appareils IoT Edge imbriqués. Il ne peut pas être utilisée pour envoyer des données à IoT Hub, car IoT Hub n’est pas un répartiteur MQTT complet. Pour en savoir plus sur la prise en charge des fonctionnalités du répartiteur IoT Hub MQTT, consultez [Communication avec votre IoT Hub à l’aide du protocole MQTT](../iot-hub/iot-hub-mqtt-support.md). Pour en savoir plus sur l’imbrication d’appareils IoT Edge, consultez [Connecter un appareil IoT Edge en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md#configure-iot-edge-on-devices) 

Dans une configuration imbriquée, le pont MQTT du hub IoT Edge agit en tant que client du répartiteur MQTT parent. Donc, les règles d’autorisation doivent être définies sur l’élément EdgeHub parent pour autoriser l’élément EdgeHub enfant à publier sur des rubriques définies par l’utilisateur spécifiques pour lesquelles le pont est configuré et à s’y abonner.

Le pont IoT Edge MQTT est configuré via une structure JSON qui est envoyée au hub IoT Edge via son jumeau. Modifiez un jumeau de hub IoT Edge pour configurer son pont MQTT.

> [!NOTE]
> Pour la préversion publique, la configuration du pont MQTT est uniquement disponible via Visual Studio, Visual Studio Code ou Azure CLI. Actuellement, le portail Azure ne prend pas en charge la modification du jumeau du hub IoT Edge et de sa configuration de pont MQTT.

Le pont MQTT peut être configuré pour connecter un répartiteur MQTT de hub IoT Edge à plusieurs répartiteurs externes. Pour chaque répartiteur externe, les paramètres suivants sont requis :

- `endpoint` est l’adresse du répartiteur MQTT distant auquel se connecter. Seuls les appareils IoT Edge parents sont actuellement pris en charge et sont définis par la variable `$upstream`.
- `settings` définit les rubriques à relier à un point de terminaison. Il peut y avoir plusieurs paramètres par point de terminaison et les valeurs suivantes sont utilisées pour le configurer :
    - `direction` : `in` pour s’abonner aux rubriques du répartiteur distant ou `out` pour publier sur les rubriques du répartiteur distant
    - `topic` : modèle de rubrique principal à mettre en correspondance. Les [caractères génériques MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107) peuvent être utilisés pour définir ce modèle. Des préfixes différents peuvent être appliqués à ce modèle de rubrique sur le répartiteur local et le répartiteur distant.
    - `outPrefix` : Préfixe appliqué au modèle `topic` sur le répartiteur à distance.
    - `inPrefix` : Préfixe appliqué au modèle `topic` sur le répartiteur local.

Voici un exemple de configuration de pont IoT Edge MQTT qui republie tous les messages reçus sur les rubriques `alerts/#` d’un appareil IoT Edge parent vers un appareil IoT Edge enfant sur les mêmes rubriques, et republie tous les messages envoyés sur les rubriques `/local/telemetry/#` d’un appareil IoT Edge enfant sur un appareil IoT Edge parent sur des rubriques `/remote/messages/#`.

```json
{
    "schemaVersion": "1.2",
    "mqttBroker": {
        "bridges": [{
            "endpoint": "$upstream",
            "settings": [{
                    "direction": "in",
                    "topic": "alerts/#"
                },
                {
                    "direction": "out",
                    "topic": "",
                    "inPrefix": "/local/telemetry",
                    "outPrefix": "/remote/messages"
                }
            ]
        }]
    }
}
```
Autres remarques sur le pont MQTT du hub IoT Edge :
- Le protocole MQTT est automatiquement utilisé comme protocole en amont quand le répartiteur MQTT est utilisé et qu’IoT Edge est utilisé dans une configuration imbriquée, par exemple avec un élément `parent_hostname` spécifié. Pour en savoir plus sur les protocoles en amont, consultez [Communication cloud](iot-edge-runtime.md#cloud-communication). Pour en savoir plus sur les configurations imbriquées, consultez [Connecter un appareil IoT Edge en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md#configure-iot-edge-on-devices).

## <a name="next-steps"></a>Étapes suivantes

[Comprendre le hub IoT Edge](iot-edge-runtime.md#iot-edge-hub)