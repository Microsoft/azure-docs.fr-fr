---
title: Présentation des jumeaux d’appareil Azure IoT Hub | Microsoft Docs
description: Guide du développeur - Utiliser des jumeaux d’appareil pour synchroniser les données d’état et de configuration entre IoT Hub et vos appareils
author: nehsin
manager: philmea
ms.author: nehsin
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom:
- mqtt
- 'Role: Cloud Development'
ms.openlocfilehash: b83faecb16ac09a47a0ade25474f7a5b3ecd4296
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400926"
---
# <a name="understand-and-use-device-twins-in-iot-hub"></a>Comprendre et utiliser les jumeaux d’appareil IoT Hub

Les *jumeaux d’appareil* sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). Azure IoT Hub conserve un jumeau d’appareil pour chaque appareil que vous y connectez. 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Cet article aborde les points suivants :

* La structure du jumeau d’appareil : *tags* (balises), propriétés *desired* (souhaitées) et *reported* (signalées).
* Les opérations que les applications d’appareil et back-ends peuvent effectuer sur des jumeaux d’appareil.

Vous pouvez utiliser des jumeaux d’appareil pour répondre aux besoins suivants :

* Stocker les métadonnées spécifiques à l’appareil dans le cloud, par exemple, l’emplacement de déploiement d’un distributeur automatique.

* Signaler les informations d’état actuel, telles que les capacités disponibles et les conditions, à partir de votre application pour appareil, par exemple, un appareil connecté à votre hub IoT via un réseau mobile ou Wi-Fi.

* Synchroniser l’état des flux de travail de longue durée entre une application pour appareil et un back-end, par exemple, lorsque le back-end de solution spécifie une nouvelle version de microprogramme à installer et que l’application pour appareil rapporte les différentes étapes du processus de mise à jour.

* Interroger les métadonnées, la configuration ou l’état de vos appareils

Pour des conseils sur l’utilisation des propriétés rapportées, des messages appareil-à-cloud ou du chargement de fichiers, consultez [Recommandations sur les communications appareil-à-cloud](iot-hub-devguide-d2c-guidance.md).

Pour des conseils sur l’utilisation des propriétés souhaitées, des méthodes directes ou des messages cloud-à-appareil, consultez [Recommandations sur les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

## <a name="device-twins"></a>Jumeaux d’appareil

Les jumeaux d’appareil stockent des informations relatives aux appareils, dont l’utilité est la suivante :

* Ils permettent à un appareil et à un back-end de synchroniser les conditions et la configuration de l’appareil.

* Ils permettent à un back-end de solution d’interroger et de cibler des opérations de longue durée.

Le cycle de vie d’un jumeau d’appareil est lié à l’[identité d’appareil](iot-hub-devguide-identity-registry.md) correspondante. Des jumeaux d’appareil sont implicitement créés et supprimés lors de la création ou de la suppression d’une identité d’appareil dans IoT Hub.

Un jumeau d’appareil est un document JSON incluant les éléments suivants :

* **Tags** (balises). Une section du document JSON accessible en lecture et en écriture par le serveur principal de solution. Les balises ne sont pas visibles pour des applications pour appareil.

* **Propriétés souhaitées (Desired)** . Utilisées en même temps que les propriétés signalées pour synchroniser une configuration ou une condition d’appareil. Le serveur principal de solution peut définir les propriétés souhaitées, et l’application d’appareil peut les lire. L’application d’appareil peut également recevoir des notifications sur les changements des propriétés souhaitées.

* **Propriétés signalées (Reported)** . Utilisées en même temps que les propriétés souhaitées pour synchroniser une configuration ou une condition d’appareil. L’application d’appareil peut définir les propriétés signalées, et le serveur principal de solution peut les lire et les interroger.

* **Propriétés d’identité des appareils**. La racine du document JSON du jumeau d’appareil contient les propriétés en lecture seule de l’identité d’appareil correspondante stockées dans le [registre des identités](iot-hub-devguide-identity-registry.md). Les propriétés `connectionStateUpdatedTime` et `generationId` ne sont pas incluses.

![Capture d’écran des propriétés de jumeau d’appareil](./media/iot-hub-devguide-device-twins/twin.png)

L’exemple suivant montre un document JSON de jumeau d’appareil :

```json
{
    "deviceId": "devA",
    "etag": "AAAAAAAAAAc=", 
    "status": "enabled",
    "statusReason": "provisioned",
    "statusUpdateTime": "0001-01-01T00:00:00",
    "connectionState": "connected",
    "lastActivityTime": "2015-02-30T16:24:48.789Z",
    "cloudToDeviceMessageCount": 0, 
    "authenticationType": "sas",
    "x509Thumbprint": {     
        "primaryThumbprint": null, 
        "secondaryThumbprint": null 
    }, 
    "version": 2, 
    "tags": {
        "$etag": "123",
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

L’objet racine contient les propriétés d’identité des appareils et des objets conteneur pour `tags` et les propriétés `reported` et `desired`. Le conteneur `properties` contient des éléments en lecture seule (`$metadata`, `$etag` et `$version`) décrits dans les sections [Métadonnées de jumeau d’appareil](iot-hub-devguide-device-twins.md#device-twin-metadata) et [Accès concurrentiel optimiste](iot-hub-devguide-device-twins.md#optimistic-concurrency).

### <a name="reported-property-example"></a>Exemple de propriété signalée (Reported)

Dans l’exemple précédent, le jumeau d’appareil contient une propriété `batteryLevel` qui est signalée par l’application d’appareil. Cette propriété permet d’interroger des appareils et d’agir sur ceux-ci en fonction du dernier niveau signalé de charge de la batterie. D’autres exemples incluent une application d’appareil signalant des capacités d’appareil ou des options de connectivité.

> [!NOTE]
> Les propriétés signalées simplifient les scénarios où le serveur principal de solution s’intéresse à la dernière valeur connue d’une propriété. Si le back-end de la solution doit traiter une télémétrie d’appareil se présentant sous la forme de séquences d’événements horodatés, telles que des séries chronologiques, utilisez des [messages appareil-à-cloud](iot-hub-devguide-messages-d2c.md).

### <a name="desired-property-example"></a>Exemple de propriété souhaitée

Dans l’exemple précédent, les propriétés souhaitées et signalées du jumeau d’appareil `telemetryConfig` sont utilisées par le serveur principal d’application et l’application d’appareil pour synchroniser la configuration de la télémétrie pour cet appareil. Par exemple :

1. Le serveur principal de solution définit la propriété souhaitée avec la valeur de configuration souhaitée. Voici la partie du document contenant la propriété souhaitée (Desired) définie :

   ```json
   "desired": {
       "telemetryConfig": {
           "sendFrequency": "5m"
       },
       ...
   },
   ```

2. L’application d’appareil est informée de la modification, immédiatement si elle est connectée ou à la première de la reconnexion. L’application d’appareil signale ensuite la configuration mise à jour (ou une condition d’erreur à l’aide de la propriété `status`). Voici la partie contenant les propriétés signalées :

   ```json
   "reported": {
       "telemetryConfig": {
           "sendFrequency": "5m",
           "status": "success"
       }
       ...
   }
   ```

3. Le back-end de la solution peut suivre les résultats de l’opération de configuration sur de nombreux appareils en [interrogeant](iot-hub-devguide-query-language.md) les jumeaux d’appareil.

> [!NOTE]
> Les extraits de code précédents sont des exemples, optimisés pour la lisibilité, de manière possible d’encoder la configuration d’un appareil et son état. IoT Hub n’impose pas de schéma spécifique pour les propriétés souhaitées et signalées de jumeau d’appareil dans les jumeaux d’appareil.
> 

Vous pouvez utiliser des jumeaux pour synchroniser des opérations de longue durée, telles que des mises à jour de microprogramme. Pour plus d’informations sur l’utilisation de propriétés pour synchroniser et suivre une opération de longue durée sur des appareils, consultez [Utiliser des propriétés souhaitées pour configurer des appareils](tutorial-device-twins.md).

## <a name="back-end-operations"></a>Opérations principales

Le back-end de solution opère sur le jumeau d’appareil en utilisant les opérations atomiques suivantes, exposées par le biais du protocole HTTPS :

* **Récupérer le jumeau d’appareil par son ID**. Cette opération renvoie le contenu du document du jumeau d’appareil, à savoir les Tags (balises) et les propriétés système souhaitées (Desired) et signalées (Reported).

* **Mettre à jour partiellement le jumeau d’appareil**. Cette opération permet au serveur principal de solution de mettre à jour partiellement les Tags (balises) ou les propriétés souhaitées (Desired) dans un jumeau d’appareil. La mise à jour partielle est exprimée sous la forme d’un document JSON qui ajoute ou met à jour toute propriété. Les propriétés définies sur `null` sont supprimées. L’exemple suivant crée une propriété souhaitée avec la valeur `{"newProperty": "newValue"}`, remplace la valeur existante de `existingProperty` par `"otherNewValue"` et supprime `otherOldProperty`. Aucune autre modification n’est apportée aux autres propriétés souhaitées ou Tags existants :

   ```json
   {
       "properties": {
           "desired": {
               "newProperty": {
                   "nestedProperty": "newValue"
               },
               "existingProperty": "otherNewValue",
               "otherOldProperty": null
           }
       }
   }
   ```

* **Remplacer des propriétés souhaitées**. Cette opération permet au serveur principal de la solution de remplacer complètement toutes les propriétés souhaitées existantes, et de remplacer `properties/desired` par un nouveau document JSON.

* **Remplacer des Tags**. Cette opération permet au serveur principal de la solution de remplacer complètement tous les Tags existants, et de remplacer `tags` par un nouveau document JSON.

* **Recevoir des notifications jumelles**. Cette opération permet au back-end de la solution d’être notifié lorsque le jumeau est modifié. Pour ce faire, votre solution IoT doit créer un itinéraire et définir la source de données équivalente à *twinChangeEvents*. Par défaut, aucune route n’existe, donc aucune notification jumelle n’est envoyée. Si le taux de variation est trop élevé, ou pour d’autres raisons, telles que des défaillances internes, IoT Hub peut envoyer une seule notification qui contient toutes les modifications. Par conséquent, si l’audit et la journalisation fiables de tous les états intermédiaires sont nécessaires pour votre application, vous devez utiliser des messages appareil-à-cloud. Le message de notification jumelle inclut le corps et les propriétés.

  - Propriétés

    | Nom | Valeur |
    | --- | --- |
    $content-type | application/json |
    $iothub-enqueuedtime |  Heure d’envoi de la notification |
    $iothub-message-source | twinChangeEvents |
    $content-encoding | utf-8 |
    deviceId | ID de l’appareil |
    hubName | Nom de l’IoT Hub |
    operationTimestamp | Horodatage [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) de l’opération |
    iothub-message-schema | twinChangeNotification |
    opType | « replaceTwin » ou « updateTwin » |

    Les propriétés système du message ont pour préfixe le symbole `$`.

  - body
        
    Cette section comprend toutes les modifications de double dans un format JSON. Il utilise le même format sous forme de correctif, à la différence près qu’il peut contenir toutes les sections jumelles : balises, properties.reported, properties.desired et qu’il contient les éléments « $metadata ». Par exemple,

    ```json
    {
      "properties": {
          "desired": {
              "$metadata": {
                  "$lastUpdated": "2016-02-30T16:24:48.789Z"
              },
              "$version": 1
          },
          "reported": {
              "$metadata": {
                  "$lastUpdated": "2016-02-30T16:24:48.789Z"
              },
              "$version": 1
          }
      }
    }
    ```

Toutes les opérations précédentes prennent en charge [l’accès concurrentiel optimiste](iot-hub-devguide-device-twins.md#optimistic-concurrency) et nécessitent l’autorisation **ServiceConnect** définie dans [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).

En plus de ces opérations, le back-end de la solution peut :

* Interroger les jumeaux d’appareil à l’aide du [langage de requête IoT Hub](iot-hub-devguide-query-language.md) similaire à SQL.

* Effectuer des opérations sur les grands ensembles de jumeaux d’appareil à l’aide de [travaux](iot-hub-devguide-jobs.md).

## <a name="device-operations"></a>Opérations d’appareil

L’application d’appareil opère sur le jumeau d’appareil en utilisant les opérations atomiques suivantes :

* **Récupérer le jumeau d’appareil**. Cette opération renvoie le contenu du document du jumeau d’appareil, à savoir les propriétés système souhaitées (Desired) et signalées (Reported) pour l’appareil actuellement connecté. (Les balises ne sont pas visibles pour des applications d’appareil.)

* **Mettre à jour partiellement les propriétés signalées (Reported)** . Cette opération permet la mise à jour partielle des propriétés signalées de l’appareil actuellement connecté. Cette opération utilise le même format de mise à jour JSON que le serveur principal de solution utilise pour une mise à jour partielle des propriétés souhaitées.

* **Observer les propriétés souhaitées (Desired)** . L’appareil actuellement connecté peut choisir d’être informé des mises à jour des propriétés souhaitées au moment où elles se produisent. L’appareil reçoit la forme de mise à jour (remplacement partiel ou complet) exécutée par le serveur principal de la solution.

Toutes les opérations précédentes nécessitent l’autorisation **DeviceConnect**, comme défini dans [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).

Les kits [Azure IoT device SDK](iot-hub-devguide-sdks.md) simplifient l’utilisation des opérations précédentes dans un grand nombre de langages et de plateformes. Pour plus d’informations sur les détails des primitives d’IoT Hub concernant la synchronisation des propriétés souhaitées, consultez [Flux de reconnexion d’appareil](iot-hub-devguide-device-twins.md#device-reconnection-flow).

## <a name="tags-and-properties-format"></a>Format des Tags et propriétés

Les Tags (balises) ainsi que les propriétés souhaitées (Desired) et signalées (Reported) sont des objets JSON soumis aux restrictions suivantes :

* **Clés** : Toutes les clés des objets JSON sont encodées en UTF-8, respectent la casse et mesurent jusqu’à 1 Ko. Les caractères autorisés excluent les caractères de contrôle UNICODE (segments C0 et C1), ainsi que `.`, `$` et SP.

* **Valeurs** : Toutes les valeurs figurant dans les objets JSON peuvent être des types JSON suivants : booléen, nombre, chaîne, objet. Les tableaux sont également pris en charge.

    * Les entiers peuvent avoir une valeur minimale de 4503599627370496 et une valeur maximale de 4503599627370495.

    * Les valeurs de chaîne sont encodées au format UTF-8 et peuvent avoir une longueur maximale de 4 Ko.

* **Profondeur** : La profondeur maximale des objets JSON dans les balises, les propriétés souhaitées et les propriétés signalées est de 10. Par exemple, l’objet suivant est valide :

   ```json
   {
       ...
       "tags": {
           "one": {
               "two": {
                   "three": {
                       "four": {
                           "five": {
                               "six": {
                                   "seven": {
                                       "eight": {
                                           "nine": {
                                               "ten": {
                                                   "property": "value"
                                               }
                                           }
                                       }
                                   }
                               }
                           }
                       }
                   }
               }
           }
       },
       ...
   }
   ```

## <a name="device-twin-size"></a>Taille de jumeau d’appareil

IoT Hub applique une limite de taille de 8 Ko à la valeur `tags`, et une limite de taille de 32 Ko aux valeurs `properties/desired` et `properties/reported`. Ces totaux n’incluent pas les éléments en lecture seule, tels que `$etag`, `$version` et `$metadata/$lastUpdated`.

La taille du jumeau est calculée comme suit :

* Pour chaque propriété du document JSON, IoT Hub calcule et ajoute de façon cumulative la longueur de la clé et la valeur de la propriété.

* Les clés de propriété sont considérées comme des chaînes encodées au format UTF8.

* Les valeurs de propriété simples sont considérées comme des chaînes encodées au format UTF8, des valeurs numériques (huit octets) ou des valeurs booléennes (quatre octets).

* La taille des chaînes encodées au format UTF8 est calculée en comptant tous les caractères, à l’exception des caractères de contrôle UNICODE (segments C0 et C1).

* Les valeurs de propriété complexes (objets imbriqués) sont calculées en fonction de la taille agrégée des clés de propriété et des valeurs de propriété qu’elles contiennent.

IoT Hub rejette en générant une erreur toute opération susceptible d’augmenter la taille des documents `tags`, `properties/desired` ou `properties/reported` au-delà de la limite.

## <a name="device-twin-metadata"></a>Métadonnées de jumeau d’appareil

IoT Hub tient à jour l’horodateur de la dernière mise à jour de chaque objet JSON dans les propriétés souhaitées et signalées du jumeau d’appareil. Les horodateurs sont exprimés en UTC et codés au format [ISO8601](https://en.wikipedia.org/wiki/ISO_8601)`YYYY-MM-DDTHH:MM:SS.mmmZ`.

Par exemple :

```json
{
    ...
    "properties": {
        "desired": {
            "telemetryConfig": {
                "sendFrequency": "5m"
            },
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": {
                        "$lastUpdated": "2016-03-30T16:24:48.789Z"
                    },
                    "$lastUpdated": "2016-03-30T16:24:48.789Z"
                },
                "$lastUpdated": "2016-03-30T16:24:48.789Z"
            },
            "$version": 23
        },
        "reported": {
            "telemetryConfig": {
                "sendFrequency": "5m",
                "status": "success"
            },
            "batteryLevel": "55%",
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": {
                        "$lastUpdated": "2016-03-31T16:35:48.789Z"
                    },
                    "status": {
                        "$lastUpdated": "2016-03-31T16:35:48.789Z"
                    },
                    "$lastUpdated": "2016-03-31T16:35:48.789Z"
                },
                "batteryLevel": {
                    "$lastUpdated": "2016-04-01T16:35:48.789Z"
                },
                "$lastUpdated": "2016-04-01T16:24:48.789Z"
            },
            "$version": 123
        }
    }
    ...
}
```

Ces informations sont conservées à chaque niveau (pas uniquement celui des feuilles de la structure JSON) afin de préserver les mises à jour qui suppriment des clés d’objet.

## <a name="optimistic-concurrency"></a>Accès concurrentiel optimiste

Les Tags ainsi que les propriétés souhaitées (Desired) et signalées (Reported) prennent en charge l’accès concurrentiel optimiste.
Les Tags ont un ETag, conforme à la norme [RFC7232](https://tools.ietf.org/html/rfc7232), correspondant à leur représentation JSON. Vous pouvez utiliser les ETags dans des opérations de mise à jour conditionnelle à partir du serveur principal de solution pour assurer la cohérence.

Les propriétés souhaitées et signalées du jumeau d’appareil n’ont pas d’ETag, mais une valeur `$version` dont la nature incrémentielle est garantie. De même qu’un ETag, la version peut être utilisée par la partie effectuant la mise à jour afin d’assurer la cohérence des mises à jour. Par exemple, une application d’appareil pour une propriété signalée ou le serveur principal de solution pour une propriété souhaitée.

Les versions sont également utiles quand un agent observateur (par exemple, l’application d’appareil observant les propriétés souhaitées) doit concilier des concurrences entre les résultats d’une opération de récupération et d’une notification de mise à jour. Pour plus d’informations, consultez la section [Flux de reconnexion d’appareil](iot-hub-devguide-device-twins.md#device-reconnection-flow).

## <a name="device-reconnection-flow"></a>Flux de reconnexion d’appareil

IoT Hub ne conserve pas les notifications de mise à jour de propriétés souhaitées pour les appareils déconnectés. Il en résulte qu’un appareil qui se connecte doit récupérer le document complet des propriétés souhaitées, en plus de s’abonner aux notifications de mise à jour. Étant donné la possibilité de concurrences entre les notifications de mise à jour et la récupération complète, le flux suivant doit être assuré :

1. L’application d’appareil se connecte à un hub IoT.
2. L’application d’appareil s’abonne aux notifications de mise à jour des propriétés souhaitées.
3. L’application d’appareil récupère le document complet pour les propriétés souhaitées.

L’application d’appareil peut ignorer toutes les notifications dont la `$version` a une valeur inférieure ou égale à la version du document complet récupéré. Cette approche est possible, car IoT Hub garantit que les numéros de version sont toujours incrémentiels.

> [!NOTE]
> Cette logique est déjà implémentée dans les kits [Azure IoT device SDK](iot-hub-devguide-sdks.md). Cette description est utile uniquement lorsque l’application d’appareil ne peut utiliser aucun des kits Azure IoT device SDK et que vous devez programmer l’interface MQTT directement.
> 

## <a name="additional-reference-material"></a>Matériel de référence supplémentaire

Les autres rubriques de référence dans le Guide du développeur IoT Hub comprennent :

* L’article [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md) décrit les différents points de terminaison que chaque hub IoT expose pour les opérations d’exécution et de gestion.

* L’article [Quotas et limitation](iot-hub-devguide-quotas-throttling.md) décrit les quotas appliqués au service IoT Hub, et le comportement de limitation auquel s’attendre en cas d’utilisation du service.

* L’article sur les kits [Azure IoT device et service SDK](iot-hub-devguide-sdks.md) liste les kits SDK en différents langages que vous pouvez utiliser pour le développement d’applications d’appareil et de service qui interagissent avec IoT Hub.

* L’article [Langage de requête IoT Hub pour les jumeaux d’appareil, les travaux et le routage des messages](iot-hub-devguide-query-language.md) décrit le langage de requête IoT Hub permettant de récupérer, à partir d’IoT Hub, des informations relatives à vos jumeaux d’appareil et à vos travaux.

* L’article [Prise en charge de MQTT dans IoT Hub](iot-hub-mqtt-support.md) fournit des informations supplémentaires sur la prise en charge du protocole MQTT par IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez ce que sont des jumeaux d’appareil, vous serez peut-être intéressé par les rubriques suivantes du Guide du développeur IoT Hub :

* [Comprendre et utiliser les jumeaux de module dans IoT Hub](iot-hub-devguide-module-twins.md)
* [Appeler une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md)
* [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md)

Pour tenter de mettre en pratique certains des concepts décrits dans cet article, consultez les didacticiels IoT Hub suivants :

* [Guide pratique pour utiliser un jumeau d’appareil](iot-hub-node-node-twin-getstarted.md)
* [Guide pratique pour utiliser des propriétés de jumeau d’appareil](tutorial-device-twins.md)
* [Gestion des appareils avec Azure IoT Tools pour VS Code](iot-hub-device-management-iot-toolkit.md)
