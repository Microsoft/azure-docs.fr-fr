---
title: Déployer des modules à l’échelle à l’aide de Visual Studio Code – Azure IoT Edge
description: Utilisez l’extension IoT pour Visual Studio Code afin de créer des déploiements automatiques pour les groupes d’appareils IoT Edge.
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 1/8/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 7f6e90edc0503326dc9dbb06abfcf59fa2d51e1e
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043814"
---
# <a name="deploy-iot-edge-modules-at-scale-using-visual-studio-code"></a>Déployer des modules IoT Edge à l’échelle à l’aide de Visual Studio Code

Vous pouvez créer un **déploiement automatique IoT Edge** à l’aide de Visual Studio Code pour gérer les déploiements en cours de plusieurs appareils à la fois. Les déploiements automatiques IoT Edge font partie de la fonctionnalité [Gestion automatique des appareils](../iot-hub/iot-hub-automatic-device-management.md) d’IoT Hub. Les déploiements sont des processus dynamiques qui vous permettent de déployer plusieurs modules sur plusieurs appareils. Vous pouvez également suivre l’état et l’intégrité des modules et apporter des modifications le cas échéant.

Pour plus d’informations, consultez [Comprendre les déploiements automatiques IoT Edge pour un seul ou de nombreux appareils](module-deployment-monitoring.md).

Dans cet article, vous configurez Visual Studio Code et l’extension IoT. Vous apprendrez ensuite à déployer des modules sur un ensemble d’appareils IoT Edge.

## <a name="prerequisites"></a>Prérequis

* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) dans votre abonnement Azure.
* Un ou plusieurs appareils IoT Edge.

  Si vous n’avez aucun appareil IoT Edge configuré, vous pouvez en créer un sur une machine virtuelle Azure. Suivez les étapes décrites dans l’un des articles de démarrage rapide pour [Créer un appareil Linux virtuel](quickstart-linux.md) ou [Créer un appareil Windows virtuel](quickstart.md).

* [Visual Studio Code](https://code.visualstudio.com/).
* [Outils Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools#overview) pour Visual Studio Code.

## <a name="sign-in-to-access-your-iot-hub"></a>Se connecter pour accéder à votre hub ioT

Vous pouvez utiliser les extensions Azure IoT pour Visual Studio Code afin d’effectuer des opérations avec votre Hub. Pour que ces opérations fonctionnent, vous devez vous connecter à votre compte Azure, puis sélectionner le hub IoT sur lequel vous travaillez.

1. Dans Visual Studio Code, ouvrez la vue **Explorateur**.

1. Au bas de l’Explorateur, développez la section **Azure IoT Hub**.

1. Cliquez sur **...** dans l’en-tête de section **Azure IoT Hub**. Si vous ne voyez pas les points de suspension, pointez sur l’en-tête.

1. Choisissez **Sélectionner un hub IoT**.

1. Si vous n’êtes pas connecté à votre compte Azure, suivez les invites à cette fin.

1. Sélectionnez votre abonnement Azure.

1. Sélectionnez votre hub IoT.

## <a name="configure-a-deployment-manifest"></a>Configurer un manifeste de déploiement

Le manifeste de déploiement est un document JSON qui décrit les modules à déployer. Il décrit également le flux des données entre les modules et les propriétés souhaitées des jumeaux de module. Pour plus d’informations, consultez [Déployer des modules et établir des routes dans IoT Edge](module-composition.md).

Pour déployer des modules à l’aide de Visual Studio Code, enregistrez localement le manifeste de déploiement dans un fichier .JSON. Vous devrez fournir son emplacement au moment d’exécuter la commande permettant d’appliquer la configuration à votre appareil.

Par exemple, voici un manifeste de déploiement de base comportant un seul module :

>[!NOTE]
>Cet exemple de manifeste de déploiement utilise la version de schéma 1.1 pour l’agent et le hub IoT Edge. La version de schéma 1.1 a été publiée avec IoT Edge version 1.0.10 ; elle fournit des fonctionnalités telles que l’ordre de démarrage des modules et la hiérarchisation des routes.

```json
{
  "content": {
    "modulesContent": {
      "$edgeAgent": {
        "properties.desired": {
          "schemaVersion": "1.1",
          "runtime": {
            "type": "docker",
            "settings": {
              "minDockerVersion": "v1.25",
              "loggingOptions": "",
              "registryCredentials": {}
            }
          },
          "systemModules": {
            "edgeAgent": {
              "type": "docker",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                "createOptions": "{}"
              }
            },
            "edgeHub": {
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
              }
            }
          },
          "modules": {
            "SimulatedTemperatureSensor": {
              "version": "1.1",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                "createOptions": "{}"
              }
            }
          }
        }
      },
      "$edgeHub": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "routes": {
            "upstream": "FROM /messages/* INTO $upstream"
          },
          "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
          }
        }
      },
      "SimulatedTemperatureSensor": {
        "properties.desired": {
          "SendData": true,
          "SendInterval": 5
        }
      }
    }
  }
}
```

Si vous avez besoin de déterminer les appareils IoT Edge que vous pouvez actuellement configurer, exécutez la commande **IoT Edge: Get Device Info**.

## <a name="identify-devices-with-target-conditions"></a>Identifier les appareils dotés des conditions cibles

Pour identifier les appareils IoT Edge qui doivent recevoir le déploiement, vous devez spécifier une condition cible. Une condition cible est remplie lorsque les critères spécifiés correspondent à un deviceId, une valeur d’étiquette ou une valeur de propriété rapportée.

Vous configurez des étiquettes dans le jumeau d’appareil. Voici un exemple de jumeau d’appareil doté d’étiquettes :

```json
"tags":{
  "location":{
    "building": "20",
    "floor": "2"
  },
  "roomtype": "conference",
  "environment": "prod"
}
```

Cet appareil recevra un déploiement si la condition cible du déploiement contient une expression correspondant à la valeur de l’une des étiquettes, par exemple `tag.location.building = '20'`.

Si vous souhaitez cibler un appareil spécifique indépendamment de ses étiquettes ou autres valeurs, spécifiez simplement le `deviceId` pour la condition cible.

Voici d’autres exemples :

* deviceId =’linuxprod1’
* deviceId = 'linuxprod1' OR deviceId = 'linuxprod2' OR deviceId = 'linuxprod3'
* tags.environment =’prod’
* tags.environment = 'prod' AND tags.location = 'westus2'
* tags.environment = 'prod' OR tags.location = 'westus2'
* tags.operator = 'John' AND tags.environment = 'prod' AND NOT deviceId = 'linuxprod1'

Pour plus d’informations, consultez [Condition cible](module-deployment-monitoring.md#target-condition). Pour plus d’informations sur les étiquettes et les jumeaux d’appareils, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).

### <a name="edit-the-device-twin"></a>Modifier le jumeau d’appareil

Vous pouvez modifier le jumeau d’appareil dans Visual Studio Code pour configurer les étiquettes. Dans le menu **Affichage**, sélectionnez **Palette de commandes** et exécutez la commande **IOT Edge: Edit Device Twin**. Sélectionnez votre appareil IoT Edge et le jumeau d’appareil s’affiche.

Dans cet exemple, aucune étiquette n’a été définie. Remplacez la section vide actuelle `"tags": {}` par votre propre définition d’étiquettes.

```json
{
    "deviceId": "myEdgeDevice",
    "etag": "AAAAAAAAAAE=",
    "deviceEtag": "NTgwMDg5MDAz",
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00Z",
    "connectionState": "Disconnected",
    "lastActivityTime": "0001-01-01T00:00:00Z",
    "cloudToDeviceMessageCount": 0,
    "authenticationType": "sas",
    "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
    },
    "version": 2,
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2019-12-29T00:58:49.9315265Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2019-12-29T00:58:49.9315265Z"
            },
            "$version": 1
        }
    },
    "capabilities": {
        "iotEdge": true
    },
    "deviceScope": "ms-azure-iot-edge://myEdgeDevice-637131779299315265",
    "tags": {}
}
```

Une fois que vous avez enregistré le fichier local, exécutez la commande **IoT Edge: Update Device Twin**.

## <a name="create-deployment-at-scale"></a>Créer un déploiement à l’échelle

Une fois que vous avez configuré le manifeste de déploiement et configuré des étiquettes dans le jumeau d’appareil, vous êtes prêt à effectuer le déploiement.

1. Dans le menu **Affichage**, sélectionnez **Palette de commandes**, puis la commande **Azure IoT Edge: Create Deployment at Scale**.

1. Recherchez le fichier JSON de manifeste de déploiement à utiliser, puis cliquez sur **Select Edge Deployment Manifest** (Sélectionner un manifeste de déploiement Edge).

1. Fournissez les valeurs comme demandé, en commençant par l’**ID de déploiement**.

   ![Spécifier un ID de déploiement](./media/how-to-deploy-monitor-vscode/create-deployment-at-scale.png)

   Spécifier les valeurs de ces paramètres :

  | Paramètre | Description |
  | --- | --- |
  | ID de déploiement | Nom du déploiement à créer dans le hub IoT. Donnez à votre déploiement un nom unique comportant au plus 128 lettres minuscules. Évitez les espaces et les caractères non valides suivants : `& ^ [ ] { } \ | " < > /`. |
  | Condition cible | Entrez une condition cible pour déterminer quels sont les appareils ciblés par ce déploiement.  La condition est basée sur les balises de jumeau d’appareil ou sur les propriétés signalées du jumeau d’appareil et doit correspondre au format de l’expression. Par exemple, `tags.environment='test' and properties.reported.devicemodel='4000x'`.  |
  | Priorité |  Entier positif. Si deux ou plusieurs déploiements sont ciblés sur le même appareil, le déploiement ayant la valeur numérique la plus élevée pour Priority s’applique. |

  Une fois la priorité spécifiée, le terminal doit afficher une sortie similaire à la description suivante :

   ```cmd
   [Edge] Start deployment with deployment id [{specified-value}] and target condition [{specified-value}]
   [Edge] Deployment with deployment id [{specified-value}] succeeded.
   ```

## <a name="monitoring-and-modifying-deployments"></a>Surveillance et modification de déploiements

Utilisez le [Portail Azure](how-to-monitor-iot-edge-deployments.md#monitor-a-deployment-in-the-azure-portal) ou [Azure CLI](how-to-monitor-iot-edge-deployments.md#monitor-a-deployment-with-azure-cli) pour surveiller, modifier et supprimer des déploiements. Les deux fournissent des mesures sur vos déploiements.

## <a name="next-steps"></a>Étapes suivantes

Découvrez le [déploiement des modules sur des appareils IoT Edge](module-deployment-monitoring.md).