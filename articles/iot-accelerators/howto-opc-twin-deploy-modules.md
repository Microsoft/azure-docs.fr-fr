---
title: Déployer le module de représentations de OPC à partir de zéro | Microsoft Docs
description: Cet article explique comment déployer OPC Twin à partir de zéro avec le panneau IoT Edge du Portail Azure, ainsi qu’avec l’interface CLI AZ.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: conceptual
ms.service: industrial-iot
ms.custom: devx-track-azurecli
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 1c12b1c7fd393227cb22d011f8b88f914cfded59
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445491"
---
# <a name="deploy-opc-twin-module-and-dependencies-from-scratch"></a>Déployer le module de représentations de OPC et les dépendances à partir de zéro

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Le module de représentations de OPC s’exécute sur IoT Edge et fournit plusieurs services edge au jumeau d’appareil et aux services de registre. 

Il existe plusieurs options pour déployer des modules dans votre passerelle [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/).

- [Déploiement à partir du panneau IoT Edge du portail Azure](../iot-edge/how-to-deploy-modules-portal.md)
- [Déploiement à l’aide de l’interface CLI AZ](../iot-edge/how-to-deploy-cli-at-scale.md)

> [!NOTE]
> Pour obtenir des informations détaillées et des instructions sur le déploiement, consultez le [dépôt](https://github.com/Azure/azure-iiot-components) GitHub.

## <a name="deployment-manifest"></a>Manifeste de déploiement

Tous les modules sont déployés à l’aide d’un manifeste de déploiement.  Un exemple de manifeste pour déployer l’[Éditeur OPC](https://github.com/Azure/iot-edge-opc-publisher) et [OPC Twin](https://github.com/Azure/azure-iiot-opc-twin-module) est indiqué ci-dessous.

```json
{
  "content": {
    "modulesContent": {
      "$edgeAgent": {
        "properties.desired": {
          "schemaVersion": "1.0",
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
                "createOptions": ""
              }
            },
            "edgeHub": {
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}], \"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
              }
            }
          },
          "modules": {
            "opctwin": {
              "version": "1.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-twin:latest",
                "createOptions": "{\"NetworkingConfig\": {\"EndpointsConfig\": {\"host\": {}}}, \"HostConfig\": {\"NetworkMode\": \"host\" }}"
              }
            },
            "opcpublisher": {
              "version": "2.0",
              "type": "docker",
              "status": "running",
              "restartPolicy": "always",
              "settings": {
                "image": "mcr.microsoft.com/iotedge/opc-publisher:latest",
                "createOptions": "{\"Hostname\":\"publisher\",\"Cmd\":[\"publisher\",\"--pf=./pn.json\",\"--di=60\",\"--tm\",\"--aa\",\"--si=0\",\"--ms=0\"],\"ExposedPorts\":{\"62222/tcp\":{}},\"NetworkingConfig\":{\"EndpointsConfig\":{\"host\":{}}},\"HostConfig\":{\"NetworkMode\":\"host\",\"PortBindings\":{\"62222/tcp\":[{\"HostPort\":\"62222\"}]}}}"
              }
            }
          }
        }
      },
      "$edgeHub": {
        "properties.desired": {
          "schemaVersion": "1.0",
          "routes": {
            "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
            "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
          },
          "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
          }
        }
      }
    }
  }
}
```

## <a name="deploying-from-azure-portal"></a>Déploiement à partir du portail Azure

Le moyen le plus facile de déployer les modules dans un appareil de passerelle IoT Edge est via le portail Azure.  

### <a name="prerequisites"></a>Prérequis

1. Déployez les [dépendances](howto-opc-twin-deploy-dependencies.md) d’OPC Twin et obtenez le fichier `.env`. Notez le `hub name` déployé de la variable `PCS_IOTHUBREACT_HUB_NAME` dans le fichier `.env` obtenu.

2. Inscrivez et démarrez une passerelle IoT Edge [Linux](../iot-edge/how-to-install-iot-edge.md) ou [Windows](../iot-edge/how-to-install-iot-edge.md) et notez son `device id`.

### <a name="deploy-to-an-edge-device"></a>Déployer sur un appareil Edge

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez à votre IoT Hub.

2. Dans le menu de gauche, sélectionnez **IoT Edge**.

3. Cliquez sur l’ID de l’appareil cible dans la liste des appareils.

4. Sélectionnez **Définir modules**.

5. Dans la section **Modules de déploiement** de la page, sélectionnez **Ajouter** puis **Module IoT Edge**.

6. Dans la boîte de dialogue **Module IoT Edge personnalisé**, utilisez `opctwin` comme nom du module, puis spécifiez l’*URI d’image* du conteneur comme

   ```bash
   mcr.microsoft.com/iotedge/opc-twin:latest
   ```

   Pour *Options de création de conteneur* utilisez le JSON suivant :

   ```json
   {"NetworkingConfig": {"EndpointsConfig": {"host": {}}}, "HostConfig": {"NetworkMode": "host" }}
   ```

   Renseignez les champs facultatifs si nécessaire. Pour plus d’informations sur les options de création de conteneur, la stratégie de redémarrage et l’état souhaité, consultez [Propriétés souhaitées pour EdgeAgent](../iot-edge/module-edgeagent-edgehub.md#edgeagent-desired-properties). Pour plus d’informations sur le jumeau de module, consultez [Définir ou mettre à jour les propriétés souhaitées](../iot-edge/module-composition.md#define-or-update-desired-properties).

7. Sélectionnez **Enregistrer** et répétez l’étape **5**.  

8. Dans la boîte de dialogue Module IoT Edge personnalisé, utilisez `opcpublisher` comme nom du module, et l’*URI d’image* du conteneur comme 

   ```bash
   mcr.microsoft.com/iotedge/opc-publisher:latest
   ```

   Pour *Options de création de conteneur* utilisez le JSON suivant :

   ```json
   {"Hostname":"publisher","Cmd":["publisher","--pf=./pn.json","--di=60","--tm","--aa","--si=0","--ms=0"],"ExposedPorts":{"62222/tcp":{}},"HostConfig":{"PortBindings":{"62222/tcp":[{"HostPort":"62222"}] }}}
   ```

9. Sélectionnez **Enregistrer** puis **Suivant** pour passer à la section des itinéraires.

10. Dans l’onglet des itinéraires, copiez les valeurs suivantes 

    ```json
    {
      "routes": {
        "opctwinToIoTHub": "FROM /messages/modules/opctwin/* INTO $upstream",
        "opcpublisherToIoTHub": "FROM /messages/modules/opcpublisher/* INTO $upstream"
      }
    }
    ```

    et sélectionnez **Suivant**

11. Passez en revue les informations et le manifeste de votre déploiement.  Il doit ressembler au manifeste de déploiement ci-dessus.  Sélectionnez **Envoyer**.

12. Une fois les modules déployés sur votre appareil, vous pouvez les voir tous dans la page **Détails de l’appareil** du portail. Cette page affiche le nom de chaque module déployé, ainsi que des informations utiles telles que le code de sortie et l’état du déploiement.

## <a name="deploying-using-azure-cli"></a>Déploiement à l’aide d’Azure CLI

### <a name="prerequisites"></a>Prérequis

1. Installez la dernière version de l’[interface de ligne de commande AZ](/cli/azure/?view=azure-cli-latest) en suivant [ce lien](/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="quickstart"></a>Démarrage rapide

1. Enregistrez le manifeste de déploiement ci-dessus dans un fichier `deployment.json`.  

2. Pour appliquer la configuration à un appareil IoT Edge, utilisez la commande suivante :

   ```azurecli
   az iot edge set-modules --device-id [device id] --hub-name [hub name] --content ./deployment.json
   ```

   Le paramètre `device id` respecte la casse. Le paramètre content pointe vers le fichier manifeste de déploiement que vous avez enregistré. 
    ![az IoT Edge set-modules output](/azure/iot-edge/media/how-to-deploy-cli/set-modules.png)

3. Une fois les modules déployés sur votre appareil, vous pouvez les voir tous à l’aide de la commande suivante :

   ```azurecli
   az iot hub module-identity list --device-id [device id] --hub-name [hub name]
   ```

   Le paramètre device-id respecte la casse. ![az iot hub module-identity list output](/azure/iot-edge/media/how-to-deploy-cli/list-modules.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment déployer OPC Twin à partir de zéro, voici l’étape suivante suggérée :

> [!div class="nextstepaction"]
> [Déployer OPC Twin sur un projet existant](howto-opc-twin-deploy-existing.md)