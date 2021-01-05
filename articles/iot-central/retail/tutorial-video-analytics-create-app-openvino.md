---
title: Tutoriel - Créer une application d’analytique vidéo pour la détection d’objets et de mouvements dans Azure IoT Central (OpenVINO)
description: Ce tutoriel montre comment créer une application d’analytique vidéo dans IoT Central. Vous pouvez la créer, la personnaliser et la connecter à d’autres services Azure. Ce tutoriel utilise le kit de ressources Intel OpenVINO&trade; pour la détection d’objets en temps réel.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
author: KishorIoT
ms.author: nandab
ms.date: 10/06/2020
ms.openlocfilehash: fbe1e84525eed47127a08abc9fb7ec5d1144d02f
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763607"
---
# <a name="tutorial-create-a-video-analytics---object-and-motion-detection-application-in-azure-iot-central-openvinotrade"></a>Tutoriel : Créer une application d’analytique vidéo pour la détection d’objets et de mouvements dans Azure IoT Central (OpenVINO&trade;)

En tant que créateur de solutions, apprenez à créer une application d’analytique vidéo avec le modèle d’application d’*analytique vidéo pour la détection d’objets et de mouvements* IoT Central, les appareils Azure IoT Edge, Azure Media Services et le module OpenVINO&trade; optimisé par le matériel d’Intel pour la détection d’objets et de mouvements. La solution utilise un magasin de vente au détail pour montrer comment répondre aux besoins courants des entreprises afin de surveiller les caméras de sécurité. La solution utilise la détection automatique d’objets dans un flux vidéo pour identifier et localiser rapidement les événements intéressants.

> [!TIP]
> Pour utiliser YOLO v3 au lieu d’OpenVINO&trade; pour la détection d’objets et de mouvements, consultez [Tutoriel : Créer une application d’analytique vidéo pour la détection d’objets et de mouvements dans Azure IoT Central (YOLO v3)](tutorial-video-analytics-create-app-yolo-v3.md).

[!INCLUDE [iot-central-video-analytics-part1](../../../includes/iot-central-video-analytics-part1.md)]

- [Scratchpad.txt](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt) : dans ce fichier, vous pouvez enregistrer les différentes options de configuration que vous devrez utiliser tout au long de ces tutoriels.
- [deployment.openvino.amd64.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/deployment.openvino.amd64.json)
- [LvaEdgeGatewayDcm.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/LvaEdgeGatewayDcm.json)
- [state.json](https://raw.githubusercontent.com/Azure/live-video-analytics/master/ref-apps/lva-edge-iot-central-gateway/setup/state.json) : téléchargez ce fichier uniquement si vous envisagez d’utiliser l’appareil Intel NUC dans le deuxième tutoriel.

[!INCLUDE [iot-central-video-analytics-part2](../../../includes/iot-central-video-analytics-part2.md)]

## <a name="edit-the-deployment-manifest"></a>Modifier le manifeste de déploiement

Vous déployez un module IoT Edge à l’aide d’un manifeste de déploiement. Dans IoT Central, vous pouvez importer le manifeste en tant que modèle d’appareil, puis laisser IoT Central gérer le déploiement du module.

Pour préparer le manifeste de déploiement :

1. À l’aide d’un éditeur de texte, ouvrez le fichier *deployment.openvino.amd64.json* que vous avez enregistré dans le dossier *lva-configuration*.

1. Recherchez les paramètres `LvaEdgeGatewayModule` et vérifiez que le nom de l’image est identique à celui indiqué dans l’extrait de code suivant :

    ```json
    "LvaEdgeGatewayModule": {
        "settings": {
            "image": "mcr.microsoft.com/lva-utilities/lva-edge-iotc-gateway:1.0-amd64",
    ```

1. Ajoutez le nom de votre compte Media Services au nœud `env` de la section `LvaEdgeGatewayModule`. Vous avez noté le nom du compte Media Services dans le fichier *scratchpad.txt* :

    ```json
    "env": {
        "lvaEdgeModuleId": {
            "value": "lvaEdge"
        },
        "amsAccountName": {
            "value": "<YOUR_AZURE_MEDIA_SERVICES_ACCOUNT_NAME>"
        }
    }
    ```

1. Le modèle n’exposant pas ces propriétés dans IoT Central, vous devez ajouter les valeurs de configuration Media Services au manifeste de déploiement. Recherchez le `lvaEdge`module et remplacez les espaces réservés par les valeurs que vous avez notées dans le fichier *scratchpad.txt* lors de la création de votre compte Media Services.

    `azureMediaServicesArmId` correspond à l’**ID de ressource** que vous avez noté dans le fichier *scratchpad.txt* lors de la création du compte Media Services.

    Le tableau suivant indique les valeurs de la section **Connect to Media Services API (JSON)** dans le fichier *scratchpad.txt* que vous devez utiliser dans le manifeste de déploiement :

    | Manifeste de déploiement       | Scratchpad  |
    | ------------------------- | ----------- |
    | aadTenantId               | AadTenantId |
    | aadServicePrincipalAppId  | AadClientId |
    | aadServicePrincipalSecret | AadSecret   |

    > [!CAUTION]
    > Aidez-vous du tableau précédent pour veiller à ajouter les valeurs correctes dans le manifeste de déploiement. Sinon, l’appareil ne fonctionnera pas.

    ```json
    {
        "lvaEdge":{
        "properties.desired": {
            "applicationDataDirectory": "/var/lib/azuremediaservices",
            "azureMediaServicesArmId": "[Resource ID from scratchpad]",
            "aadTenantId": "[AADTenantID from scratchpad]",
            "aadServicePrincipalAppId": "[AadClientId from scratchpad]",
            "aadServicePrincipalSecret": "[AadSecret from scratchpad]",
            "aadEndpoint": "https://login.microsoftonline.com",
            "aadResourceId": "https://management.core.windows.net/",
            "armEndpoint": "https://management.azure.com/",
            "diagnosticsEventsOutputName": "AmsDiagnostics",
            "operationalMetricsOutputName": "AmsOperational",
            "logCategories": "Application,Event",
            "AllowUnsecuredEndpoints": "true",
            "TelemetryOptOut": false
            }
        }
    }
    ```

1. Enregistrez les modifications.

Ce tutoriel configure votre solution de sorte qu’elle utilise le module OpenVINO&trade; pour la détection d’objets et de mouvements. L’extrait de code suivant montre la configuration du module :

```json
"OpenVINOModelServerEdgeAIExtensionModule": {
    "settings": {
        "image": "marketplace.azurecr.io/intel_corporation/open_vino",
        "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"4000/tcp\":[{\"HostPort\":\"4000\"}]}},\"Cmd\":[\"/ams_wrapper/start_ams.py\",\"--ams_port=4000\",\"--ovms_port=9000\"]}"
    },
    "type": "docker",
    "version": "1.0",
    "status": "running",
    "restartPolicy": "always"
}
```

[!INCLUDE [iot-central-video-analytics-part3](../../../includes/iot-central-video-analytics-part3.md)]

### <a name="replace-the-manifest"></a>Remplacer le manifeste

Dans la page **LVA Edge Gateway v2**, sélectionnez **+ Remplacer le manifeste**.

:::image type="content" source="./media/tutorial-video-analytics-create-app-openvino/replace-manifest.png" alt-text="Remplacer le manifeste":::

Accédez au dossier *lva-configuration* et sélectionnez le fichier manifeste *deployment.openvino.amd64.json* modifié précédemment. Sélectionnez **Télécharger**. Une fois la validation terminée, sélectionnez **Remplacer**.

[!INCLUDE [iot-central-video-analytics-part4](../../../includes/iot-central-video-analytics-part4.md)]
