---
title: Présentation des jumeaux de module Azure IoT Hub | Microsoft Docs
description: Guide du développeur - Utiliser des jumeaux de module pour synchroniser les données d’état et de configuration entre IoT Hub et vos appareils
author: nehsin
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/29/2020
ms.author: nehsin
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 6e728eaf8335a102e38a3b4b07ab5e504d452294
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91996472"
---
# <a name="understand-and-use-module-twins-in-iot-hub"></a>Comprendre et utiliser les jumeaux de module dans IoT Hub

Cet article suppose que vous avez déjà lu l’article [Comprendre et utiliser les jumeaux d’appareil IoT Hub](iot-hub-devguide-device-twins.md). Dans IoT Hub, sous chaque identité d’appareil, vous pouvez créer jusqu’à 50 identités de module. Chaque identité de module génère implicitement un jumeau de module. À l’image des jumeaux d’appareil, les jumeaux de module sont des documents JSON qui stockent des informations sur l’état des modules (métadonnées, configurations et conditions). Azure IoT Hub conserve un jumeau de module pour chaque module que vous y connectez. 

Côté appareil, les kits IoT Hub device SDK vous permettent de créer des modules dont chacun ouvre une connexion indépendante à IoT Hub. Cette fonctionnalité vous permet d’utiliser des espaces de noms distincts pour les différents composants de votre appareil. Par exemple, vous disposez d’un distributeur automatique dotés de trois capteurs différents. Chaque capteur est contrôlé par différents services de votre entreprise. Vous pouvez créer un module pour chaque capteur. De cette manière, chaque service ne peut envoyer des travaux ou des méthodes directes qu’au capteur qu’il contrôle, ce qui évite les conflits et les erreurs de l’utilisateur.

 L’identité de module et le jumeau de module fournissent les mêmes fonctionnalités que l’identité d’appareil et le jumeau d’appareil, mais avec une plus grande précision. Ainsi, les appareils compatibles, tels que les appareils basés sur le système d’exploitation ou les appareils avec microprogramme gérant plusieurs composants, peuvent isoler la configuration et les conditions pour chacun de ces composants. L’identité de module et les jumeaux de module vous permettent de gérer les problèmes de façon séparée quand vous utilisez des appareils IoT qui comportent des composants logiciels modulaires. Nous œuvrons pour que toutes les fonctionnalités de jumeau d’appareil soient prises en charge au niveau des jumeaux de module au plus tard quand ces derniers seront en disponibilité générale. 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Cet article aborde les points suivants :

* La structure du jumeau de module : *tags* (balises), propriétés *desired* (souhaitées) et *reported* (signalées).
* Les opérations que les modules et backends peuvent effectuer sur des jumeaux de modules.

Pour des conseils sur l’utilisation des propriétés rapportées, des messages appareil-à-cloud ou du chargement de fichiers, consultez [Recommandations sur les communications appareil-à-cloud](iot-hub-devguide-d2c-guidance.md).

Pour des conseils sur l’utilisation des propriétés souhaitées, des méthodes directes ou des messages cloud-à-appareil, consultez [Recommandations sur les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

## <a name="module-twins"></a>Jumeaux de module

Les jumeaux de module stockent des informations relatives aux modules, dont l’utilité est la suivante :

* Elles permettent aux modules sur l’appareil et IoT Hub de synchroniser les conditions et la configuration des modules.

* Ils permettent à un back-end de solution d’interroger et de cibler des opérations de longue durée.

Le cycle de vie d’un jumeau de module est lié à [l’identité de module](iot-hub-devguide-identity-registry.md) correspondante. Des jumeaux de module sont implicitement créés et supprimés au moment de la création ou de la suppression d’une identité de module dans IoT Hub.

Un jumeau de module est un document JSON incluant les éléments suivants :

* **Tags** (balises). Une section du document JSON accessible en lecture et en écriture par le serveur principal de solution. Les balises ne sont pas visibles pour les modules sur l’appareil. Les balises sont définies à des fins d’interrogation.

* **Propriétés souhaitées (Desired)** . Utilisées en même temps que les propriétés signalées pour synchroniser une configuration ou une condition de module. Le backend de solution peut définir les propriétés souhaitées, et l’application de module peut les lire. L’application de module peut également recevoir des notifications sur les changements des propriétés souhaitées.

* **Propriétés signalées (Reported)** . Utilisées en même temps que les propriétés souhaitées pour synchroniser une configuration ou une condition de module. L’application de module peut définir les propriétés signalées, et le backend de solution peut les lire et les interroger.

* **Propriétés d’identité des modules**. La racine du document JSON du jumeau de module contient les propriétés en lecture seule de l’identité de module correspondante stockées dans le [registre des identités](iot-hub-devguide-identity-registry.md).

![Représentation architecturale du jumeau d’appareil](./media/iot-hub-devguide-device-twins/module-twin.jpg)

L’exemple suivant montre un document JSON de jumeau de module :

```json
{
    "deviceId": "devA",
    "moduleId": "moduleA",
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

L’objet racine contient les propriétés d’identité des modules et des objets conteneur pour `tags` et les propriétés `reported` et `desired`. Le conteneur `properties` contient des éléments en lecture seule (`$metadata`, `$etag` et `$version`) décrits dans les sections [Métadonnées de jumeau de module](iot-hub-devguide-module-twins.md#module-twin-metadata) et [Accès concurrentiel optimiste](iot-hub-devguide-device-twins.md#optimistic-concurrency).

### <a name="reported-property-example"></a>Exemple de propriété signalée (Reported)

Dans l’exemple précédent, le jumeau de module contient une propriété `batteryLevel` qui est signalée par l’application de module. Cette propriété permet d’interroger des modules et d’agir sur ceux-ci en fonction du dernier niveau signalé de charge de la batterie. D’autres exemples incluent une application de module signalant des capacités de module ou des options de connectivité.

> [!NOTE]
> Les propriétés signalées simplifient les scénarios où le serveur principal de solution s’intéresse à la dernière valeur connue d’une propriété. Si le back-end de la solution doit traiter une télémétrie de module se présentant sous la forme de séquences d’événements horodatés, telles que des séries chronologiques, utilisez des [messages appareil-à-cloud](iot-hub-devguide-messages-d2c.md).

### <a name="desired-property-example"></a>Exemple de propriété souhaitée

Dans l’exemple précédent, les propriétés souhaitées et signalées du jumeau de module `telemetryConfig` sont utilisées par le serveur principal d’application et l’application de module pour synchroniser la configuration de la télémétrie pour ce module. Par exemple :

1. Le serveur principal de solution définit la propriété souhaitée avec la valeur de configuration souhaitée. Voici la partie du document contenant la propriété souhaitée (Desired) définie :

    ```json
    ...
    "desired": {
        "telemetryConfig": {
            "sendFrequency": "5m"
        },
        ...
    },
    ...
    ```

2. L’application de module est informée de la modification, immédiatement si elle est connectée ou à la première reconnexion. L’application de module signale ensuite la configuration mise à jour (ou une condition d’erreur à l’aide de la propriété `status`). Voici la partie contenant les propriétés signalées :

    ```json
    "reported": {
        "telemetryConfig": {
            "sendFrequency": "5m",
            "status": "success"
        }
        ...
    }
    ```

3. Le back-end de solution peut suivre les résultats de l’opération de configuration sur de nombreux modules en [interrogeant](iot-hub-devguide-query-language.md) les jumeaux de module.

> [!NOTE]
> Les extraits de code précédents sont des exemples, optimisés pour la lisibilité, de manière possible d’encoder la configuration d’un module et son état. IoT Hub n’impose pas de schéma spécifique pour les propriétés souhaitées et signalées de jumeau de module dans les jumeaux de module.
> 
> 

## <a name="back-end-operations"></a>Opérations principales
Le backend de solution opère sur le jumeau de module en utilisant les opérations atomiques suivantes, exposées par le biais du protocole HTTPS :

* **Récupérer le jumeau de module par ID**. Cette opération retourne le contenu du document du jumeau de module, à savoir les Tags (balises) et les propriétés système souhaitées (Desired) et signalées (Reported).

* **Mettre à jour partiellement le jumeau de module**. Cette opération permet au backend de solution de mettre à jour partiellement les Tags (balises) ou les propriétés souhaitées (Desired) dans un jumeau de module. La mise à jour partielle est exprimée sous la forme d’un document JSON qui ajoute ou met à jour toute propriété. Les propriétés définies sur `null` sont supprimées. L’exemple suivant crée une propriété souhaitée avec la valeur `{"newProperty": "newValue"}`, remplace la valeur existante de `existingProperty` par `"otherNewValue"` et supprime `otherOldProperty`. Aucune autre modification n’est apportée aux autres propriétés souhaitées ou Tags existants :

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

* **Recevoir des notifications jumelles**. Cette opération permet au back-end de la solution d’être notifié lorsque le jumeau est modifié. Pour ce faire, votre solution IoT doit créer un itinéraire et définir la source de données équivalente à *twinChangeEvents*. Par défaut, aucune notification jumelle n’est envoyée. Autrement dit, aucun itinéraire n’existe préalablement. Si le taux de variation est trop élevé, ou pour d’autres raisons, telles que des défaillances internes, IoT Hub peut envoyer une seule notification qui contient toutes les modifications. Par conséquent, si l’audit et la journalisation fiables de tous les états intermédiaires sont nécessaires pour votre application, vous devez utiliser des messages appareil-à-cloud. Le message de notification jumelle inclut le corps et les propriétés.

  - Propriétés

    | Nom | Valeur |
    | --- | --- |
    $content-type | application/json |
    $iothub-enqueuedtime |  Heure d’envoi de la notification |
    $iothub-message-source | twinChangeEvents |
    $content-encoding | utf-8 |
    deviceId | ID de l’appareil |
    moduleId | ID du module |
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

Toutes les opérations précédentes prennent en charge [l’accès concurrentiel optimiste](iot-hub-devguide-device-twins.md#optimistic-concurrency) et nécessitent l’autorisation **ServiceConnect** définie dans l’article [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).

En plus de ces opérations, le back-end de solution peut interroger les jumeaux de module à l’aide du [langage de requête d’IoT Hub](iot-hub-devguide-query-language.md) de type SQL.

## <a name="module-operations"></a>Opérations sur les modules

L’application de module opère sur le jumeau de module en utilisant les opérations atomiques suivantes :

* **Récupérer le jumeau de module**. Cette opération retourne le contenu du document du jumeau de module, à savoir les Tags (Balises) et les propriétés système souhaitées (Desired) et signalées (Reported), pour le module connecté.

* **Mettre à jour partiellement les propriétés signalées (Reported)** . Cette opération permet la mise à jour partielle des propriétés signalées du module connecté. Cette opération utilise le même format de mise à jour JSON que le serveur principal de solution utilise pour une mise à jour partielle des propriétés souhaitées.

* **Observer les propriétés souhaitées (Desired)** . Le module connecté peut choisir d’être informé des mises à jour des propriétés souhaitées au moment où elles se produisent. Le module reçoit la forme de mise à jour (remplacement partiel ou complet) exécutée par le backend de la solution.

Toutes les opérations précédentes nécessitent l’autorisation **ModuleConnect**, comme défini dans l’article [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).

Les kits [Azure IoT device SDK](iot-hub-devguide-sdks.md) simplifient l’utilisation des opérations précédentes dans un grand nombre de langages et de plateformes.

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

## <a name="module-twin-size"></a>Taille de jumeau de module

IoT Hub applique une limite de taille de 8 Ko à la valeur `tags`, et une limite de taille de 32 Ko aux valeurs `properties/desired` et `properties/reported`. Ces totaux n’incluent pas les éléments en lecture seule, tels que `$etag`, `$version` et `$metadata/$lastUpdated`.

La taille du jumeau est calculée comme suit :

* Pour chaque propriété du document JSON, IoT Hub calcule et ajoute de façon cumulative la longueur de la clé et la valeur de la propriété.

* Les clés de propriété sont considérées comme des chaînes encodées au format UTF8.

* Les valeurs de propriété simples sont considérées comme des chaînes encodées au format UTF8, des valeurs numériques (huit octets) ou des valeurs booléennes (quatre octets).

* La taille des chaînes encodées au format UTF8 est calculée en comptant tous les caractères, à l’exception des caractères de contrôle UNICODE (segments C0 et C1).

* Les valeurs de propriété complexes (objets imbriqués) sont calculées en fonction de la taille agrégée des clés de propriété et des valeurs de propriété qu’elles contiennent.

IoT Hub rejette en générant une erreur toute opération susceptible d’augmenter la taille de ces documents au-delà de la limite.

## <a name="module-twin-metadata"></a>Métadonnées de jumeau de module

IoT Hub tient à jour l’horodateur de la dernière mise à jour de chaque objet JSON dans les propriétés souhaitées et signalées du jumeau de module. Les horodateurs sont exprimés en UTC et codés au format [ISO8601](https://en.wikipedia.org/wiki/ISO_8601)`YYYY-MM-DDTHH:MM:SS.mmmZ`.
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
                    "sendFrequency": "5m",
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

Les propriétés souhaitées et signalées du jumeau de module n’ont pas d’ETag, mais une valeur `$version` dont la nature incrémentielle est garantie. De même qu’un ETag, la version peut être utilisée par la partie effectuant la mise à jour afin d’assurer la cohérence des mises à jour. Par exemple, une application de module pour une propriété signalée ou le backend de solution pour une propriété souhaitée.

Les versions sont également utiles quand un agent observateur (par exemple, l’application de module observant les propriétés souhaitées) doit concilier des concurrences entre les résultats d’une opération de récupération et d’une notification de mise à jour. Pour plus d’informations, consultez [Flux de reconnexion d’appareil](iot-hub-devguide-device-twins.md#device-reconnection-flow). 

## <a name="next-steps"></a>Étapes suivantes

Pour tenter de mettre en pratique certains des concepts décrits dans cet article, consultez les didacticiels IoT Hub suivants :

* [Bien démarrer avec le jumeau de module et l’identité de module IoT Hub en utilisant un back-end .NET et un appareil .NET](iot-hub-csharp-csharp-module-twin-getstarted.md)
