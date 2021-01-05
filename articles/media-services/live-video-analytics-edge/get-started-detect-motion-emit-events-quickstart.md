---
title: Bien démarrer avec Live Video Analytics sur IoT Edge - Azure
description: Ce guide de démarrage rapide montre comment commencer avec Live Video Analytics sur IoT Edge. Découvrez comment détecter tout mouvement dans un flux vidéo en direct.
ms.topic: quickstart
ms.date: 04/27/2020
ms.openlocfilehash: db48183e918fcb1096734f912f02c091ddac1d74
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511873"
---
# <a name="quickstart-get-started---live-video-analytics-on-iot-edge"></a>Démarrage rapide : Bien démarrer - Live Video Analytics sur IoT Edge

Ce guide de démarrage rapide vous guide tout au long des étapes de prise en main de Live Video Analytics sur IoT Edge. Il utilise une machine virtuelle Azure comme appareil IoT Edge. Il utilise également un flux vidéo en direct simulé. 

Une fois la procédure de configuration terminée, vous pourrez exécuter un flux vidéo en direct simulé par le biais d’un graphe multimédia qui détecte et signale tout mouvement dans ce flux. Le diagramme suivant représente ce graphe multimédia.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Live Video Analytics basé sur la détection de mouvement":::

Vous pouvez consulter la vidéo suivante qui contient les étapes détaillées pour commencer à utiliser Live Video Analytics sur IoT Edge :

<iframe src="https://www.microsoft.com/en-us/videoplayer/embed/RE4Hcax" width="640" height="320" allowFullScreen="true" frameBorder="0"></iframe>

## <a name="prerequisites"></a>Prérequis

* Un compte Azure disposant d’un abonnement actif. Si vous n’en avez pas déjà un, [créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Visual Studio Code](https://code.visualstudio.com/) sur votre machine de développement. Vérifiez que vous disposez de l’[extension Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Vérifiez que le réseau auquel votre machine de développement est connectée autorise le protocole AMQP (Advanced Message Queueing Protocol) sur le port 5671. Cette configuration permet à Azure IoT Tools de communiquer avec Azure IoT Hub.

> [!TIP]
> Vous serez peut-être invité à installer Docker pendant l’installation de l’extension Azure IoT Tools. N’hésitez pas à ignorer cette invite.

## <a name="set-up-azure-resources"></a>Configurer les ressources Azure

Ce tutoriel nécessite les ressources Azure suivantes :

* IoT Hub
* Compte de stockage
* Compte Azure Media Services
* Machine virtuelle Linux dans Azure, avec le [runtime IoT Edge](../../iot-edge/how-to-install-iot-edge.md) installé

Pour ce guide de démarrage rapide, nous vous recommandons d’utiliser le [script d’installation des ressources Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) pour déployer les ressources nécessaires dans votre abonnement Azure. Pour ce faire, procédez comme suit :

1. Accédez à [ Azure Cloud Shell](https://shell.azure.com).
1. Si vous utilisez Cloud Shell pour la première fois, vous serez invité à sélectionner un abonnement pour créer un compte de stockage et un partage Microsoft Azure Files. Sélectionnez **Créer un stockage** pour créer un compte de stockage pour vos informations de session Cloud Shell. Ce compte de stockage est distinct du compte que le script créera pour être utilisé avec votre compte Azure Media Services.
1. Dans le menu déroulant situé sur le côté gauche de la fenêtre Cloud Shell, sélectionnez **Bash** comme environnement.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/env-selector.png" alt-text="Sélecteur d’environnement":::
1. Exécutez la commande suivante :

    ```
    bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
    ```
    
Si le script se termine correctement, vous devez voir toutes les ressources nécessaires dans votre abonnement. Dans la sortie du script, un tableau de ressources indique le nom du hub IoT. Recherchez le type de ressource `Microsoft.Devices/IotHubs`, puis notez le nom. Vous en aurez besoin à la prochaine étape. 

Le script génère également quelques fichiers de configuration dans le répertoire *~/clouddrive/lva-sample/* . Vous en aurez besoin ultérieurement dans le guide de démarrage rapide.

## <a name="deploy-modules-on-your-edge-device"></a>Déployer des modules sur votre périphérique

Exécutez la commande suivante à partir de Cloud Shell.

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content ~/clouddrive/lva-sample/edge-deployment/deployment.amd64.json
```

Cette commande déploie les modules suivants sur l’appareil de périphérie, qui est la machine virtuelle Linux dans le cas présent.

* Live Video Analytics sur IoT Edge (nom de module`lvaEdge`)
* Simulateur RTSP (Real-Time Streaming Protocol) (nom de module `rtspsim`)

Le module de simulateur RTSP simule un flux vidéo en direct à l’aide d’un fichier vidéo qui a été copié sur votre appareil de périphérie quand vous avez exécuté le [script d’installation des ressources Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup). 

Les modules sont maintenant déployés, mais aucun graphe multimédia n’est actif.

## <a name="configure-the-azure-iot-tools-extension"></a>Configurer l’extension Azure IoT Tools

Suivez ces instructions pour vous connecter à votre hub IoT à l’aide de l’extension Azure IoT Tools.

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou appuyez sur Ctrl+Maj+X) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Paramètres d’extension":::
1. Recherchez et activez « Afficher le message détaillé ».

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Afficher le message détaillé":::
1. Sélectionnez **Affichage** > **Explorateur**. Vous pouvez aussi sélectionner Ctrl+Maj+E.
1. Dans l’angle en bas à gauche de l’onglet **Explorateur**, sélectionnez **Azure IoT Hub**.
1. Sélectionnez l’icône **Plus d’options** pour voir le menu contextuel. Sélectionnez ensuite **Définir la chaîne de connexion IoT Hub**.
1. Quand une zone d’entrée s’affiche, entrez votre chaîne de connexion IoT Hub. Dans Cloud Shell, vous pouvez obtenir la chaîne de connexion dans *~/clouddrive/lva-sample/appsettings.json*.

Si la connexion aboutit, la liste des appareils de périphérie s’affiche. Vous devez normalement voir au moins un appareil nommé **lva-sample-device**. Vous pouvez désormais gérer vos appareils IoT Edge et interagir avec Azure IoT Hub par le biais du menu contextuel. Pour afficher les modules déployés sur l’appareil de périphérie, sous **lva-sample-device**, développez le nœud **Modules**.

![nœud lva-sample-device](./media/quickstarts/lva-sample-device-node.png)

> [!TIP]
> Si vous avez [déployé manuellement Live Video Analytics sur IoT Edge](deploy-iot-edge-device.md) sur un appareil de périphérie (tel qu’un appareil ARM64), vous verrez que le module apparaît sous cet appareil, sous Azure IoT Hub. Vous pouvez sélectionner ce module et suivre le reste des étapes ci-dessous.

## <a name="use-direct-method-calls"></a>Utiliser des appels de méthode directe

Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant des méthodes directes. Pour plus d’informations, consultez [Méthodes directes pour Live Video Analytics sur IoT Edge](direct-methods.md). 

### <a name="invoke-graphtopologylist"></a>Invoquer GraphTopologyList

Pour énumérer toutes les [topologies de graphes](media-graph-concept.md#media-graph-topologies-and-instances) du module :

1. Dans Visual Studio Code, cliquez avec le bouton droit sur le module **lvaEdge**, puis sélectionnez **Invoquer la méthode directe du module**.
1. Dans la zone qui s’affiche, entrez *GraphTopologyList*.
1. Copiez la charge utile JSON suivante, puis collez-la dans la zone. Sélectionnez ensuite la touche Entrée.

    ```
    {
        "@apiVersion" : "2.0"
    }
    ```

    Au bout de quelques secondes, la fenêtre **SORTIE** affiche la réponse suivante.

    ```
    [DirectMethod] Invoking Direct Method [GraphTopologyList] to [lva-sample-device/lvaEdge] ...
    [DirectMethod] Response from [lva-sample-device/lvaEdge]:
    {
      "status": 200,
      "payload": {
        "value": []
      }
    }
    ```
    
    Cette réponse est attendue car aucune topologie de graphe n’a été créée.
    

### <a name="invoke-graphtopologyset"></a>Invoquer GraphTopologySet

En suivant les étapes permettant d’invoquer `GraphTopologyList`, vous pouvez invoquer `GraphTopologySet` pour définir une [topologie de graphe](media-graph-concept.md#media-graph-topologies-and-instances). Utilisez le code JSON suivant comme charge utile.

```
{
    "@apiVersion": "2.0",
    "name": "MotionDetection",
    "properties": {
        "description": "Analyzing live video to detect motion and emit events",
        "parameters": [
            {
                "name": "rtspUserName",
                "type": "String",
                "description": "rtsp source user name.",
                "default": "dummyUserName"
            },
            {
                "name": "rtspPassword",
                "type": "String",
                "description": "rtsp source password.",
                "default": "dummyPassword"
            },
            {
                "name": "rtspUrl",
                "type": "String",
                "description": "rtsp Url"
            }
        ],
        "sources": [
            {
                "@type": "#Microsoft.Media.MediaGraphRtspSource",
                "name": "rtspSource",
                "endpoint": {
                    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                    "url": "${rtspUrl}",
                    "credentials": {
                        "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                        "username": "${rtspUserName}",
                        "password": "${rtspPassword}"
                    }
                }
            }
        ],
        "processors": [
            {
                "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
                "name": "motionDetection",
                "sensitivity": "medium",
                "inputs": [
                    {
                        "nodeName": "rtspSource"
                    }
                ]
            }
        ],
        "sinks": [
            {
                "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
                "name": "hubSink",
                "hubOutputName": "inferenceOutput",
                "inputs": [
                    {
                        "nodeName": "motionDetection"
                    }
                ]
            }
        ]
    }
}

```

Cette charge utile JSON crée une topologie de graphe qui définit trois paramètres. Deux de ces paramètres ont des valeurs par défaut. La topologie possède un nœud source (source RTSP), un nœud processeur (processeur de détection de mouvement) et un nœud récepteur (récepteur IoT Hub).

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE**.

```
[DirectMethod] Invoking Direct Method [GraphTopologySet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt": "2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspUserName",
          "type": "String",
          "description": "rtsp source user name.",
          "default": "dummyUserName"
        },
        {
          "name": "rtspPassword",
          "type": "String",
          "description": "rtsp source password.",
          "default": "dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "String",
          "description": "rtsp Url"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username": "${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
          "sensitivity": "medium",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "hubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

L’état retourné est 201. Cet état indique qu’une topologie a été créée. 

Essayez les étapes suivantes :

1. Invoquez à nouveau `GraphTopologySet`. Le code d’état retourné est 200. Ce code indique qu’une topologie existante a été correctement mise à jour.
1. Invoquez de nouveau `GraphTopologySet`, mais changez la chaîne de description. Le code d’état retourné est 200 et la description est mise à jour avec la nouvelle valeur.
1. Invoquez `GraphTopologyList` comme indiqué dans la section précédente. Vous pouvez maintenant voir la topologie `MotionDetection` dans la charge utile retournée.

### <a name="invoke-graphtopologyget"></a>Invoquer GraphTopologyGet

Invoquez `GraphTopologyGet` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "MotionDetection"
}
```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE** :

```
[DirectMethod] Invoking Direct Method [GraphTopologyGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt": "2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspUserName",
          "type": "String",
          "description": "rtsp source user name.",
          "default": "dummyUserName"
        },
        {
          "name": "rtspPassword",
          "type": "String",
          "description": "rtsp source password.",
          "default": "dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "String",
          "description": "rtsp Url"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username": "${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
          "sensitivity": "medium",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "hubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

Dans la charge utile de réponse, notez les informations suivantes :

* Le code d’état est 200, ce qui indique la réussite.
* La charge utile comprend l’horodatage `created` et l’horodatage `lastModified`.

### <a name="invoke-graphinstanceset"></a>Invoquer GraphInstanceSet

Créez une instance de graphe qui fait référence à la topologie de graphe précédente. Les instances de graphe vous permettent d’analyser des flux vidéo en direct à partir de plusieurs caméras à l’aide de la même topologie de graphe. Pour plus d’informations, consultez [Topologies et instances de graphe multimédia](media-graph-concept.md#media-graph-topologies-and-instances).

Invoquez la méthode directe `GraphInstanceSet` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1",
    "properties" : {
        "topologyName" : "MotionDetection",
        "description" : "Sample graph description",
        "parameters" : [
            { "name" : "rtspUrl", "value" : "rtsp://rtspsim:554/media/camera-300s.mkv" }
        ]
    }
}
```

Notez que cette charge utile :

* Spécifie le nom de la topologie (`MotionDetection`) pour laquelle l’instance doit être créée.
* Contient une valeur de paramètre pour `rtspUrl`, qui n’avait pas de valeur par défaut dans la charge utile de la topologie de graphe.

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE** :

```
[DirectMethod] Invoking Direct Method [GraphInstanceSet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Inactive",
      "description": "Sample graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

Dans la charge utile de réponse, notez les points suivants :

* Le code d’état est 201, ce qui indique qu’une nouvelle instance a été créée.
* L’état est `Inactive`, ce qui indique que l’instance de graphe a été créée, mais pas activée. Pour plus d’informations, consultez [États du graphe multimédia](media-graph-concept.md).

Essayez les étapes suivantes :

1. Invoquez à nouveau `GraphInstanceSet` à l’aide de la même charge utile. Notez que le code d’état retourné est 200.
1. Invoquez à nouveau `GraphInstanceSet`, mais utilisez une description différente. Notez la description mise à jour dans la charge utile de réponse, indiquant que l’instance de graphe a été correctement mise à jour.
1. Invoquez `GraphInstanceSet`, mais remplacez le nom par `Sample-Graph-2`. Dans la charge utile de réponse, notez l’instance de graphe nouvellement créée (autrement dit, le code d’état 201).

### <a name="invoke-graphinstanceactivate"></a>Invoquer GraphInstanceActivate

Activez maintenant l’instance de graphe pour démarrer le flux vidéo en direct par le biais du module. Invoquez la méthode directe `GraphInstanceActivate` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE**.

```
[DirectMethod] Invoking Direct Method [GraphInstanceActivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Le code d’état 200 indique que l’instance de graphe a été correctement activée.

### <a name="invoke-graphinstanceget"></a>Invoquer GraphInstanceGet

Invoquez maintenant la méthode directe `GraphInstanceGet` à l’aide de la charge utile suivante.

```
 {
     "@apiVersion" : "2.0",
     "name" : "Sample-Graph-1"
 }
 ```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE**.

```
[DirectMethod] Invoking Direct Method [GraphInstanceGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Active",
      "description": "graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

Dans la charge utile de réponse, notez les informations suivantes :

* Le code d’état est 200, ce qui indique la réussite.
* L’état est `Active`, ce qui indique que l’instance de graphe est maintenant active.

## <a name="observe-results"></a>Observer les résultats

L’instance de graphe que nous avons créée et activée utilise le nœud processeur de détection de mouvement pour détecter les mouvements dans le flux vidéo en direct entrant. Elle envoie des événements au nœud récepteur IoT Hub. Ces événements sont relayés vers IoT Edge Hub. 

Pour observer les résultats, effectuez les étapes suivantes.

1. Dans Visual Studio Code, ouvrez le volet **Explorateur**. Dans l’angle en bas à gauche, recherchez **Azure IoT Hub**.
2. Développez le nœud **Appareils**.
3. Cliquez avec le bouton droit sur **lva-sample-device**, puis sélectionnez **Démarrer la supervision de l’événement intégré**.

    ![Démarrer la surveillance des événements IoT Hub](./media/quickstarts/start-monitoring-iothub-events.png)
    
La fenêtre **SORTIE** affiche le message suivant :

```
[IoTHubMonitor] [7:44:33 AM] Message received from [lva-sample-device/lvaEdge]:
{
    "body": {
    "timestamp": 143005362606360,
    "inferences": [
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.828452,
            "t": 0.455224,
            "w": 0.1,
            "h": 0.088889
            }
        }
        },
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.661088,
            "t": 0.597015,
            "w": 0.0625,
            "h": 0.051852
            }
        }
        }
    ]
    },
    "applicationProperties": {
    "topic": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/microsoft.media/mediaservices/{amsAccountName}",
    "subject": "/graphInstances/Sample-Graph-1/processors/motionDetection",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-05-19T07:45:34.404Z",
    "dataVersion": "1.0"
    }
}
```

Notez les informations suivantes :

* Le message contient une section `body` et une section `applicationProperties`. Pour plus d’informations, consultez [Créer et lire des messages IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).
* Dans `applicationProperties`, `subject` fait référence au nœud présent dans le `MediaGraph` à partir duquel le message a été généré. Dans ce cas, le message provient du processeur de détection de mouvement.
* Dans `applicationProperties`, `eventType` indique que cet événement est un événement d’analytique.
* La valeur `eventTime` correspond à l’heure à laquelle l’événement s’est produit.
* La section `body` contient des données relatives à l’événement d’analytique. Dans ce cas, l’événement est un événement d’inférence. Par conséquent, le corps contient des données `timestamp` et `inferences`.
* La section `inferences` indique que le `type` est `motion`. Elle fournit des données supplémentaires sur l’événement `motion`.

Si vous laissez le graphe multimédia s’exécuter pendant un moment, le message suivant s’affiche dans la fenêtre **SORTIE**.

```
[IoTHubMonitor] [7:47:45 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1588948185746703 1 IN IP4 172.xx.xx.xx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-05-19T07:47:45.747Z"
  }
}
```

Dans ce message, notez les informations suivantes :

* Dans `applicationProperties`, `subject` indique que le message a été généré à partir du nœud source RTSP dans le graphe multimédia.
* Dans `applicationProperties`, `eventType` indique que cet événement est de diagnostic.
* `body` contient des données relatives à l’événement de diagnostic. Dans ce cas, le message contient le corps car l’événement est `MediaSessionEstablished`.

## <a name="invoke-additional-direct-methods-to-clean-up"></a>Invoquer des méthodes directes supplémentaires pour nettoyer

Invoquez des méthodes directes pour tout d’abord désactiver l’instance de graphe, puis la supprimer.

### <a name="invoke-graphinstancedeactivate"></a>Invoquer GraphInstanceDeactivate

Invoquez la méthode directe `GraphInstanceDeactivate` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE** :

```
[DirectMethod] Invoking Direct Method [GraphInstanceDeactivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Le code d’état 200 indique que l’instance de graphe a été correctement désactivée.

Ensuite, essayez d’invoquer `GraphInstanceGet` comme indiqué précédemment dans cet article. Observez la valeur `state`.

### <a name="invoke-graphinstancedelete"></a>Invoquer GraphInstanceDelete

Invoquez la méthode directe `GraphInstanceDelete` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE** :

```
[DirectMethod] Invoking Direct Method [GraphInstanceDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Le code d’état 200 indique que l’instance de graphe a été correctement supprimée.

### <a name="invoke-graphtopologydelete"></a>Invoquer GraphTopologyDelete

Invoquez la méthode directe `GraphTopologyDelete` à l’aide de la charge utile suivante.

```
{
    "@apiVersion" : "2.0",
    "name" : "MotionDetection"
}
```

Au bout de quelques secondes, la réponse suivante s’affiche dans la fenêtre **SORTIE**.

```
[DirectMethod] Invoking Direct Method [GraphTopologyDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Le code d’état 200 indique que la topologie de graphe a été correctement supprimée.

Essayez les étapes suivantes :

1. Invoquez `GraphTopologyList` et observez que le module ne contient pas de topologies de graphe.
1. Invoquez `GraphInstanceList` à l’aide de la même charge utile que `GraphTopologyList`. Observez qu’aucune instance de graphe n’est énumérée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez les ressources que vous avez créées dans ce guide de démarrage rapide.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [enregistrer une vidéo à l’aide de Live Video Analytics sur IoT Edge](continuous-video-recording-tutorial.md).
* Découvrez-en plus sur les [messages de diagnostic](monitoring-logging.md).