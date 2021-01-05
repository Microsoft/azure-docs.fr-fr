---
title: Déployer des modules et des routes avec des manifestes de déploiement - Azure IoT Edge
description: Découvrez comment un manifeste de déploiement déclare les modules à déployer, comment les déployer et comment créer des itinéraires de messages entre eux.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 10/08/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 3f6c12b892e01aafd5beecdff14751481cf7fc96
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963395"
---
# <a name="learn-how-to-deploy-modules-and-establish-routes-in-iot-edge"></a>Découvrez comment déployer des modules et établir des itinéraires dans IoT Edge.

Chaque appareil IoT Edge exécute au moins deux modules : $edgeAgent et $edgeHub, qui font partie du runtime IoT Edge. Un appareil IoT Edge peut exécuter des modules supplémentaires pour autant de processus que nécessaire. Utilisez un manifeste de déploiement pour indiquer à un appareil les modules à installer et comment les configurer pour qu'ils fonctionnent ensemble.

Le *manifeste de déploiement* est un document JSON qui décrit :

* Le jumeau de module de l’**agent IoT Edge** comprend trois composants :
  * L’image conteneur pour chaque module qui s’exécute sur l’appareil.
  * Les informations d’identification pour accéder aux registres de conteneurs privés contenant des images de module.
  * Des instructions sur la manière de créer et de gérer chaque module.
* le jumeau de module du **hub IoT Edge**, qui inclut le mode de flux des messages entre les modules et finalement vers IoT Hub ;
* Les propriétés souhaitées de tout jumeau de module supplémentaire (facultatif).

Tous les appareils IoT Edge doivent être configurés avec un manifeste de déploiement. Un runtime IoT Edge nouvellement installé renvoie un code d’erreur tant qu’il n’est pas configuré avec un manifeste valide.

Dans les didacticiels Azure IoT Edge, vous générez un manifeste de déploiement via un Assistant dans le portail Azure IoT Edge. Vous pouvez également appliquer un manifeste de déploiement par programmation à l’aide du Kit de développement logiciel (SDK) REST ou IoT Hub Service. Pour plus d’informations, consultez [Comprendre les déploiements IoT Edge](module-deployment-monitoring.md).

## <a name="create-a-deployment-manifest"></a>Créer un manifeste de déploiement

À un niveau élevé, un manifeste de déploiement est une liste des jumeaux de module configurés avec leurs propriétés souhaitées. Un manifeste de déploiement indique à un appareil IoT Edge (ou à un groupe d’appareils) les modules à installer et comment les configurer. Les manifestes de déploiement incluent les *propriétés souhaitées* pour chaque jumeau de module. Les appareils IoT Edge rapportent les *propriétés signalées* pour chaque module.

Deux modules sont nécessaires dans chaque manifeste de déploiement : `$edgeAgent` et `$edgeHub`. Ces modules font partie du runtime IoT Edge qui gère l’appareil IoT Edge et les modules en cours d’exécution sur ce dernier. Pour plus d’informations sur ces modules, consultez [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md).

Outre les deux modules de runtime, vous pouvez ajouter jusqu'à 50 modules de votre choix à exécuter sur un appareil IoT Edge.

Un manifeste de déploiement ne contenant que le runtime IoT Edge (edgeAgent et edgeHub) est valide.

Les manifestes de déploiement suivent la structure suivante :

```json
{
  "modulesContent": {
    "$edgeAgent": { // required
      "properties.desired": {
        // desired properties of the IoT Edge agent
        // includes the image URIs of all deployed modules
        // includes container registry credentials
      }
    },
    "$edgeHub": { //required
      "properties.desired": {
        // desired properties of the IoT Edge hub
        // includes the routing information between modules, and to IoT Hub
      }
    },
    "module1": {  // optional
      "properties.desired": {
        // desired properties of module1
      }
    },
    "module2": {  // optional
      "properties.desired": {
        // desired properties of module2
      }
    }
  }
}
```

## <a name="configure-modules"></a>Configurer des modules

Définir comment le runtime IoT Edge installe les modules dans votre déploiement. L’agent IoT Edge est le composant de runtime qui gère l’installation, les mises à jour et les rapports d’état d’un appareil IoT Edge. Pour cette raison, le jumeau du module $edgeAgent contient les informations de configuration et de gestion de tous les modules. Ces informations incluent les paramètres de configuration pour l’agent IoT Edge proprement dit.

Les propriétés $edgeAgent suivent cette structure :

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.1",
        "runtime": {
          "settings":{
            "registryCredentials":{
              // give the IoT Edge agent access to container images that aren't public
            }
          }
        },
        "systemModules": {
          "edgeAgent": {
            // configuration and management details
          },
          "edgeHub": {
            // configuration and management details
          }
        },
        "modules": {
          "module1": {
            // configuration and management details
          },
          "module2": {
            // configuration and management details
          }
        }
      }
    },
    "$edgeHub": { ... },
    "module1": { ... },
    "module2": { ... }
  }
}
```

La version 1.1 du schéma de l’agent IoT Edge a été publiée avec la version 1.0.10 de IoT Edge ; elle permet l’ordonnancement du démarrage des modules. Nous vous recommandons d’utiliser la version 1.1 du schéma pour les déploiements de IoT Edge exécutant la version 1.0.10 ou une version plus récente.

### <a name="module-configuration-and-management"></a>Configuration et gestion de module

La liste des propriétés souhaitées de l’agent IoT Edge vous permet de définir les modules qui sont déployés sur un appareil IoT Edge et la façon dont ils doivent être configurés et gérés.

Pour obtenir une liste complète des propriétés souhaitées qui peuvent ou doivent être incluses, consultez [Propriétés de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md).

Par exemple :

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.1",
        "runtime": { ... },
        "systemModules": {
          "edgeAgent": { ... },
          "edgeHub": { ... }
        },
        "modules": {
          "module1": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "startupOrder": 2,
            "settings": {
              "image": "myacr.azurecr.io/module1:latest",
              "createOptions": "{}"
            }
          },
          "module2": { ... }
        }
      }
    },
    "$edgeHub": { ... },
    "module1": { ... },
    "module2": { ... }
  }
}
```

Chaque module a une propriété **settings** (paramètres) qui contient le module **image**, une adresse pour l’image conteneur dans un registre de conteneurs, ainsi que **createOptions** pour configurer l’image au démarrage. Pour en savoir plus, consultez [Guide pratique pour configurer les options de création de conteneur pour les modules IoT Edge](how-to-use-create-options.md).

Le module edgeHub et les modules personnalisés ont également trois propriétés qui indiquent à l’agent IoT Edge comment les gérer :

* **État** : Indique si le module doit être en cours d’exécution ou arrêté lors du premier déploiement. Obligatoire.
* **RestartPolicy**: Indique si l’agent IoT Edge doit redémarrer le module si ce dernier s’arrête et, le cas échéant, quand le redémarrage doit se produire. Obligatoire.
* **StartupOrder** : *Introduit dans IoT Edge version 1.0.10.* L’ordre dans lequel l’agent IoT Edge doit démarrer les modules lors du premier déploiement. L’ordre est déclaré en entrant des nombres entiers. La valeur 0 désigne le module qui sera démarré en premier. Les nombres suivants désignent les modules qui seront démarrés après lui. Le module edgeAgent n’a pas de valeur de démarrage, car il démarre toujours en premier. Optionnel.

  L’agent de IoT Edge initialise les modules selon l’ordre de la valeur de démarrage, mais n’attend pas que chaque module finisse de démarrer avant de passer à l’étape suivante.

  L’ordre de démarrage est utile si certains modules dépendent d’autres. Par exemple, vous pouvez souhaiter que le module edgeHub démarre en premier afin qu’il soit prêt à acheminer les messages lorsque d’autres modules démarrent. Nous vous recommandons également de démarrer un module de stockage avant les modules qui lui envoient des données. Toutefois, vous devez toujours concevoir vos modules pour gérer les défaillances d’autres modules. Il s’agit de la nature des conteneurs qu’ils peuvent arrêter et redémarrer à tout moment, et autant de fois que nécessaire.

## <a name="declare-routes"></a>Déclarer des itinéraires

Le hub IoT Edge gère la communication entre les modules, IoT Hub et tous les appareils de nœuds terminaux. Par conséquent, le jumeau de module $edgeHub contient une propriété souhaitée appelée *itinéraires* qui indique la façon dont les messages sont transmis au sein d’un déploiement. Vous pouvez avoir plusieurs itinéraires dans le même déploiement.

Les itinéraires sont déclarés dans les propriétés souhaitées **$edgeHub** avec la syntaxe suivante :

```json
{
  "modulesContent": {
    "$edgeAgent": { ... },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.1",
        "routes": {
          "route1": "FROM <source> WHERE <condition> INTO <sink>",
          "route2": {
            "route": "FROM <source> WHERE <condition> INTO <sink>",
            "priority": 0,
            "timeToLiveSecs": 86400
          }
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 10
        }
      }
    },
    "module1": { ... },
    "module2": { ... }
  }
}
```

La version 1.1 du schéma du hub IoT Edge a été publiée avec la version 1.0.10 de IoT Edge. Elle permet de hiérarchiser les itinéraires et de leur définir une durée de vie. Nous vous recommandons d’utiliser la version 1.1 du schéma pour les déploiements de IoT Edge exécutant la version 1.0.10 ou une version plus récente.

Chaque itinéraire a besoin d’une *source* d’où proviennent les messages et d’un *récepteur* vers lequel les messages sont envoyés. La *condition* est un élément facultatif que vous pouvez utiliser pour filtrer les messages.

Vous pouvez affecter des *priorités* aux itinéraires dont vous voulez traiter les messages en premier. Cette fonctionnalité est utile dans les scénarios où la connexion en amont est faible ou limitée tandis que vous avez des données critiques qui doivent être prioritaires sur les messages de télémétrie standards.

### <a name="source"></a>Source

La source spécifie d'où proviennent les messages. IoT Edge peut acheminer les messages d’appareils de modules ou appareils de nœud terminal.

À l’aide des kits de développement logiciel (SDK) IoT, les modules peuvent déclarer des files d’attente de sortie spécifiques pour leurs messages à l’aide de la classe ModuleClient. Les files d’attente de sortie ne sont pas nécessaires, mais utiles pour gérer plusieurs itinéraires. Les appareils de nœud terminal peuvent utiliser la classe DeviceClient des kits de développement logiciel (SDK) IoT pour envoyer des messages aux appareils de passerelle IoT Edge de la même manière qu'ils enverraient des messages à IoT Hub. Pour plus d'informations, consultez [Comprendre et utiliser les kits de développement logiciel (SDK) Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md).

La propriété source peut être l’une des valeurs suivantes :

| Source | Description |
| ------ | ----------- |
| `/*` | Tous les messages appareil-à-cloud ou les notifications de changement de jumeau à partir de n’importe quel module ou appareil de nœud terminal |
| `/twinChangeNotifications` | Tout changement de jumeau (propriétés signalées) en provenance de n’importe quel module ou appareil de nœud terminal |
| `/messages/*` | Tout message appareil-à-cloud envoyé par un module via une sortie ou non, pour par un appareil de nœud terminal |
| `/messages/modules/*` | Tout message appareil-à-cloud envoyé par un module via une sortie ou aucune |
| `/messages/modules/<moduleId>/*` | Tout message appareil-à-cloud envoyé par un module spécifique, via une sortie ou non |
| `/messages/modules/<moduleId>/outputs/*` | Tout message appareil-à-cloud envoyé par un module spécifique, via une sortie |
| `/messages/modules/<moduleId>/outputs/<output>` | Tout message appareil-à-cloud envoyé par un module spécifique via une sortie spécifique |

### <a name="condition"></a>Condition

La condition est facultative dans une déclaration d’itinéraire. Si vous souhaitez transmettre tous les messages de la source au récepteur, il suffit d’omettre la clause **WHERE** entièrement. Vous pouvez également utiliser le [langage de requête IoT Hub](../iot-hub/iot-hub-devguide-routing-query-syntax.md) pour filtrer certains messages ou types de messages qui répondent à la condition. Les itinéraires IoT Edge ne prennent pas en charge les messages de filtrage basés sur les propriétés ou balises de jumeaux.

Les messages qui transitent entre les modules dans IoT Edge sont mis en forme de la même manière que les messages qui passent entre les appareils et Azure IoT Hub. Tous les messages sont au format JSON et ont les paramètres **systemProperties**, **appProperties** et **corps**.

Vous pouvez générer des requêtes autour de chacun des trois paramètres avec la syntaxe suivante :

* Propriétés du système : `$<propertyName>` ou `{$<propertyName>}`
* Propriétés de l’application : `<propertyName>`
* Propriétés du corps : `$body.<propertyName>`

Pour obtenir des exemples sur la façon de créer des requêtes pour les propriétés de message, consultez [Expressions de requête des itinéraires des messages appareil-à-cloud](../iot-hub/iot-hub-devguide-routing-query-syntax.md).

Par exemple, sur IoT Edge, si vous voulez filtrer des messages reçus sur un appareil de passerelle à partir d’un appareil de nœud terminal. Les messages provenant des modules incluent une propriété système appelée **connectionModuleId**. Par conséquent, si vous souhaitez router des messages à partir d’appareils de nœud terminal directement vers IoT Hub, utilisez l’itinéraire suivant pour exclure les messages de modules :

```query
FROM /messages/* WHERE NOT IS_DEFINED($connectionModuleId) INTO $upstream
```

### <a name="sink"></a>Récepteur

Le récepteur définit où les messages sont envoyés. Seuls les modules et IoT Hub peuvent recevoir des messages. Les messages ne peut pas être acheminés vers d’autres appareils. Il n’existe aucune option de caractère générique dans la propriété de récepteur.

La propriété de récepteur peut être l’une des valeurs suivantes :

| Récepteur | Description |
| ---- | ----------- |
| `$upstream` | Envoyer le message à IoT Hub |
| `BrokeredEndpoint("/modules/<moduleId>/inputs/<input>")` | Envoyer le message à une entrée spécifique d’un module spécifique |

IoT Edge offre la garantie « Au moins une remise ». Le hub IoT Edge stocke les messages localement dans le cas où un itinéraire ne peut pas remettre le message à son récepteur. C’est le cas, par exemple, si le hub IoT Edge ne peut pas se connecter à IoT Hub ou que le module cible n’est pas connecté.

Le hub IoT Edge stocke les messages jusqu’à l’heure spécifiée dans la propriété `storeAndForwardConfiguration.timeToLiveSecs` des [propriétés souhaitées du hub IoT Edge](module-edgeagent-edgehub.md).

### <a name="priority-and-time-to-live"></a>Priorité et durée de vie

Les itinéraires peuvent être déclarés : soit en utilisant simplement une chaîne pour définir un itinéraire, soit en tant qu’objet qui prend une chaîne de routage, un nombre entier pour définir sa priorité et un nombre entier pour définir sa durée de vie.

Option 1 :

   ```json
   "route1": "FROM <source> WHERE <condition> INTO <sink>",
   ```

Option 2, introduite dans la version 1.0.10 de IoT Edge avec la version 1.1 du schéma du hub IoT Edge :

   ```json
   "route2": {
     "route": "FROM <source> WHERE <condition> INTO <sink>",
     "priority": 0,
     "timeToLiveSecs": 86400
   }
   ```

Les valeurs de **priorité** peuvent aller de 0 à 9, mais aussi être inclusives, où 0 est la priorité la plus élevée. Les messages sont mis en file d’attente en fonction de leurs points de terminaison. Tous les messages de priorité 0 ciblant un point de terminaison spécifique sont traités avant le traitement de tous les messages de priorité 1 ciblant le même point de terminaison, et en aval. Si plusieurs itinéraires pour le même point de terminaison ont le même niveau de priorité, leurs messages sont traités selon le principe du « premier arrivé, premier servi ». Si aucune priorité n’est spécifiée, l’itinéraire est affecté à la priorité la plus basse.

La propriété **timeToLiveSecs** hérite de sa valeur à partir de la propriété **storeAndForwardConfiguration** du hub IoT Edge, sauf en cas de définition explicite. La valeur peut être un entier positif.

Pour en savoir plus sur la gestion des files d’attente de priorité, consultez le référentiel GitHub [Priorité et durée de vie d’itinéraire](https://github.com/Azure/iotedge/blob/master/doc/Route_priority_and_TTL.md).

## <a name="define-or-update-desired-properties"></a>Définir ou mettre à jour les propriétés souhaitées

Le manifeste de déploiement spécifie les propriétés souhaitées pour chaque module déployé sur l’appareil IoT Edge. Les propriétés souhaitées dans le manifeste de déploiement remplacent les propriétés souhaitées actuellement spécifiées dans le jumeau de module.

Si vous ne spécifiez pas de propriétés souhaitées d’un jumeau de module dans le manifeste de déploiement, IoT Hub ne modifiera d’aucune façon le jumeau de module. Au lieu de cela, vous pouvez définir programmatiquement les propriétés souhaitées.

Les mêmes mécanismes que ceux qui vous permettent de modifier des jumeaux d’appareils sont utilisés pour modifier des jumeaux de modules. Pour plus d’informations, consultez le [guide du développeur des jumeaux de module](../iot-hub/iot-hub-devguide-module-twins.md).

## <a name="deployment-manifest-example"></a>Exemple de manifeste de déploiement

L’exemple suivant montre à quoi peut ressembler un document de manifeste de déploiement valide.

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.1",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {
              "ContosoRegistry": {
                "username": "myacr",
                "password": "<password>",
                "address": "myacr.azurecr.io"
              }
            }
          }
        },
        "systemModules": {
          "edgeAgent": {
            "type": "docker",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
              "createOptions": ""
            }
          },
          "edgeHub": {
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "startupOrder": 0,
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
              "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
            }
          }
        },
        "modules": {
          "SimulatedTemperatureSensor": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "startupOrder": 2,
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
              "createOptions": "{}"
            }
          },
          "filtermodule": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "startupOrder": 1,
            "env": {
              "tempLimit": {"value": "100"}
            },
            "settings": {
              "image": "myacr.azurecr.io/filtermodule:latest",
              "createOptions": "{}"
            }
          }
        }
      }
    },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.1",
        "routes": {
          "sensorToFilter": {
            "route": "FROM /messages/modules/SimulatedTemperatureSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/filtermodule/inputs/input1\")",
            "priority": 0,
            "timeToLiveSecs": 1800
          },
          "filterToIoTHub": {
            "route": "FROM /messages/modules/filtermodule/outputs/output1 INTO $upstream",
            "priority": 1,
            "timeToLiveSecs": 1800
          }
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 100
        }
      }
    }
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir la liste complète des propriétés qui peuvent ou doivent être incluses dans $edgeAgent et $edgeHub, consultez [Propriétés de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md).

* Maintenant que vous savez comment les modules IoT Edge sont utilisés, vous pouvez lire [Comprendre les exigences et les outils de développement de modules IoT Edge](module-development.md).
