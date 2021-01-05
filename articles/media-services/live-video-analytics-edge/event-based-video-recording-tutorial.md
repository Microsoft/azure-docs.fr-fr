---
title: 'Tutoriel : Enregistrement vidéo basé sur les événements et lecture vidéo dans le cloud - Azure'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure Live Video Analytics sur Azure IoT Edge pour effectuer un enregistrement vidéo basé sur les événements dans le cloud et le lire depuis le cloud.
ms.topic: tutorial
ms.date: 05/27/2020
ms.openlocfilehash: 8f3ecdf7e4260d700f31663852abbb39474cd474
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401665"
---
# <a name="tutorial-event-based-video-recording-to-the-cloud-and-playback-from-the-cloud"></a>Tutoriel : Enregistrement vidéo basé sur les événements et lecture vidéo dans le cloud

Dans ce tutoriel, vous allez apprendre à utiliser Azure Live Video Analytics sur Azure IoT Edge pour enregistrer certaines parties d’une source vidéo live dans Azure Media Services, dans le cloud. Ici, nous désignerons ce cas d’usage par [enregistrement vidéo basé sur les événements](event-based-video-recording-concept.md) (EVR, Event-based Video Recording). Pour enregistrer certaines parties d’une vidéo live, vous allez utiliser un modèle d’intelligence artificielle de détection d’objets afin de rechercher des objets dans la vidéo et d’enregistrer des clips vidéo uniquement lorsqu’un certain type d’objet est détecté. Vous allez également découvrir comment lire les clips vidéo enregistrés à l’aide de Media Services. Cette possibilité s’avère utile pour un grand nombre de scénarios dans lesquels il est nécessaire de conserver une archive des clips vidéo d’intérêt. 

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Configurer les ressources appropriées.
> * Examiner le code qui effectue l’enregistrement EVR.
> * Exécuter l’exemple de code.
> * Examiner les résultats et regarder la vidéo.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée  

Lisez ces articles avant de commencer :

* [Présentation de Live Video Analytics sur IoT Edge](overview.md)
* [Terminologie de Live Video Analytics sur IoT Edge](terminology.md)
* [Concepts relatifs aux graphes multimédia](media-graph-concept.md) 
* [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)
* [Tutoriel : Développement d’un module IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Modification de deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)
* Section expliquant [comment déclarer des routes dans le manifeste de déploiement IoT Edge](../../iot-edge/module-composition.md#declare-routes)

## <a name="prerequisites"></a>Prérequis

Les prérequis pour ce tutoriel sont les suivants :

* [Visual Studio Code](https://code.visualstudio.com/) sur votre machine de développement avec les extensions [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) et [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

    > [!TIP]
    > Vous serez peut-être invité à installer Docker. Ignorez cette invite.
* Le [SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.201-windows-x64-installer) sur votre machine de développement.
* Avoir lu [Script d’installation des ressources Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) et [Configurer l’environnement](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/detect-motion-emit-events-quickstart?branch=release-preview-media-services-lva#set-up-the-environment)

À l’issue de ces étapes, vous disposerez des ressources Azure appropriées déployées dans votre abonnement Azure :

* Azure IoT Hub
* Compte de Stockage Azure
* Compte Azure Media Services
* Machine virtuelle Linux dans Azure, avec le [runtime IoT Edge](../../iot-edge/how-to-install-iot-edge.md) installé

## <a name="concepts"></a>Concepts

Un enregistrement vidéo basé sur les événements est un enregistrement vidéo qui a été déclenché par un événement. Cet événement peut être généré à partir des éléments suivants :
- Traitement du signal vidéo lui-même, par exemple, lors de la détection d’un objet qui se déplace dans la vidéo.
- Source indépendante, par exemple, ouverture d’une porte. 

Vous pouvez également déclencher l’enregistrement uniquement lorsqu’un service d’inférence détecte qu’un événement particulier s’est produit. Dans ce tutoriel, vous allez utiliser une vidéo de véhicules qui roulent sur une autoroute et enregistrer des clips vidéo chaque fois qu’un camion est détecté.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/overview.svg" alt-text="Graphe multimédia":::

Le diagramme est une représentation graphique d’un [graphe multimédia](media-graph-concept.md) et des modules supplémentaires qui accomplissent le scénario souhaité. Quatre modules IoT Edge sont impliqués :

* Le module Live Video Analytics sur IoT Edge.
* Un module de périphérie exécutant un modèle d’intelligence artificielle derrière un point de terminaison HTTP. Ce module d’intelligence artificielle utilise le modèle [YOLOv3](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis/yolov3-onnx), qui peut détecter de nombreux types d’objets.
* Un module personnalisé pour compter et filtrer les objets, appelé compteur d’objets dans le diagramme. Vous allez créer un compteur d’objets, puis le déployer au cours de ce tutoriel.
* Un [module de simulateur RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) pour simuler une caméra RTSP.
    
Comme le montre le diagramme, vous allez utiliser un nœud de [source RTSP](media-graph-concept.md#rtsp-source) dans le graphe multimédia afin de capturer la simulation de vidéo en direct du trafic sur l’autoroute et envoyer cette vidéo vers deux chemins :

* Le premier chemin mène à un nœud d’extension HTTP. Le nœud échantillonne les trames vidéo en une valeur que vous définissez à l’aide du champ `samplingOptions`, puis relaie les trames, en tant qu’images, vers le module d’IA YOLOv3, qui est un détecteur d’objets. Le nœud reçoit les résultats, qui correspondent aux objets (véhicules dans le trafic) détectés par le modèle. Le nœud d’extension HTTP publie ensuite les résultats par le biais du nœud récepteur de messages IoT Hub dans le hub IoT Edge.
* Le module objectCounter est configuré pour recevoir des messages du hub IoT Edge, notamment les résultats de la détection des objets (véhicules dans le trafic). Le module vérifie ces messages et recherche des objets d’un certain type, qui ont été configurés par le biais d’un paramètre. Quand un tel objet est trouvé, ce module envoie un message au hub IoT Edge. Les messages « object found » (objet trouvé) sont ensuite routés vers le nœud source IoT Hub du graphe multimédia. Lors de la réception d’un tel message, le nœud source IoT Hub dans le graphe multimédia déclenche le nœud [processeur de porte de signal](media-graph-concept.md#signal-gate-processor). Le nœud processeur de porte de signal est alors ouvert pour une durée configurée. Pendant cette période, la vidéo passe par la porte pour aller au nœud récepteur d’actifs multimédias. Cette partie du stream en direct est ensuite enregistrée par le biais du nœud [asset sink](media-graph-concept.md#asset-sink) (récepteur d’actifs multimédias) dans un [actif multimédia](terminology.md#asset) de votre compte Azure Media Services.

## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement

Avant de commencer, vérifiez que vous avez bien exécuté le troisième point de la section [Prérequis](#prerequisites). Une fois le script d’installation des ressources terminé, sélectionnez les accolades pour exposer la structure du dossier. Vous verrez quelques fichiers créés sous le répertoire ~/clouddrive/lva-sample.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstarts/clouddrive.png" alt-text="Paramètres de l’application":::

Les fichiers qui nous intéressent dans le cadre de ce tutoriel sont les suivants :

* **~/clouddrive/lva-sample/edge-deployment/.env** : Contient les propriétés que Visual Studio Code utilise pour déployer des modules sur un périphérique.
* **~/clouddrive/lva-sample/appsettings.json** : Utilisé par Visual Studio Code pour exécuter l’exemple de code.

Vous aurez besoin de ces fichiers pour ces étapes.

1. Clonez le dépôt à partir du lien GitHub https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp.
1. Démarrez Visual Studio Code et ouvrez le dossier dans lequel vous avez téléchargé le dépôt.
1. Dans Visual Studio Code, accédez au dossier src/cloud-to-device-console-app et créez un fichier nommé **appsettings.json**. Ce fichier contient les paramètres nécessaires à l’exécution du programme.
1. Copiez le contenu du fichier ~/clouddrive/lva-sample/appsettings.json. Le texte doit ressembler à l’exemple suivant :

    ```
    {  
        "IoThubConnectionString" : "HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX",  
        "deviceId" : "lva-sample-device",  
        "moduleId" : "lvaEdge"  
    }
    ```

    La chaîne de connexion IoT Hub vous permet d’utiliser Visual Studio Code pour envoyer des commandes aux modules de périphérie par le biais d’Azure IoT Hub.
    
1. Ensuite, accédez au dossier src/edge et créez un fichier nommé **.env**.
1. Copiez le contenu du fichier .env « ~/clouddrive/lva-sample/edge-deployment/ ». Le texte doit ressembler à l’exemple suivant :

    ```
    SUBSCRIPTION_ID="<Subscription ID>"  
    RESOURCE_GROUP="<Resource Group>"  
    AMS_ACCOUNT="<AMS Account ID>"  
    IOTHUB_CONNECTION_STRING="HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx"  
    AAD_TENANT_ID="<AAD Tenant ID>"  
    AAD_SERVICE_PRINCIPAL_ID="<AAD SERVICE_PRINCIPAL ID>"  
    AAD_SERVICE_PRINCIPAL_SECRET="<AAD SERVICE_PRINCIPAL ID>"  
    INPUT_VIDEO_FOLDER_ON_DEVICE="/home/lvaadmin/samples/input"  
    OUTPUT_VIDEO_FOLDER_ON_DEVICE="/home/lvaadmin/samples/output"  
    APPDATA_FOLDER_ON_DEVICE="/var/local/mediaservices"
    CONTAINER_REGISTRY_USERNAME_myacr="<your container registry username>"  
    CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry username>"      
    ```

## <a name="examine-the-template-file"></a>Étudier le fichier de modèle 

À l’étape précédente, vous avez démarré Visual Studio Code et ouvert le dossier qui contient l’exemple de code.

Dans Visual Studio Code, accédez à src/edge. Vous allez y voir le fichier .env que vous avez créé et quelques fichiers de modèle de déploiement. Ce modèle définit les modules de périphérie que vous allez déployer sur le périphérique (la machine virtuelle Linux Azure). Le fichier .env contient des valeurs pour les variables utilisées dans ces modèles, telles que les informations d’identification Media Services.

Ouvrez src/edge/deployment.objectCounter.template.json. Il existe quatre entrées sous la section **modules** qui correspondent aux éléments listés dans la section Concepts précédente :

* **lvaEdge** : Il s’agit du module Live Video Analytics sur IoT Edge.
* **yolov3** : Il s’agit du module d’intelligence artificielle créé à l’aide du modèle YOLO v3.
* **rtspsim** : Il s’agit du simulateur RTSP.
* **objectCounter** : Il s’agit du module qui recherche des objets spécifiques dans les résultats de yolov3.

Pour le module objectCounter, consultez la chaîne (${MODULES.objectCounter}) utilisée pour la valeur « image ». Celle-ci se base sur le [tutoriel](../../iot-edge/tutorial-develop-for-linux.md) sur le développement d’un module IoT Edge. Visual Studio Code reconnaît automatiquement que le code du module objectCounter se trouve sous src/edge/modules/objectCounter. 

Lisez [cette section](../../iot-edge/module-composition.md#declare-routes) sur la manière de déclarer des routes dans le manifeste de déploiement IoT Edge. Examinez ensuite les routes dans le fichier JSON du modèle. Remarque :

* LVAToObjectCounter est utilisé pour envoyer certains événements à un point de terminaison du module objectCounter.
* ObjectCounterToLVA est utilisé pour envoyer un événement déclencheur à un point de terminaison spécifique (qui doit être le nœud source IoT Hub) du module lvaEdge.
* objectCounterToIoTHub est utilisé comme outil de débogage permettant de voir la sortie d’objectCounter lorsque vous exécutez ce tutoriel.

> [!NOTE]
> Vérifiez les propriétés souhaitées pour le module objectCounter, qui sont configurées pour rechercher les objets portant l’étiquette « truck » (camion), avec un niveau de confiance d’au moins 50 %.

## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Générez et déployez le manifeste de déploiement IoT Edge 

Le manifeste de déploiement définit les modules qui sont déployés sur un périphérique et les paramètres de configuration de ces modules. Effectuez les étapes suivantes pour générer un manifeste à partir du fichier de modèle, puis déployez-le sur le périphérique.

À l’aide de Visual Studio Code, suivez [ces instructions](../../iot-edge/tutorial-develop-for-linux.md#build-and-push-your-solution) pour vous connecter à Docker. Sélectionnez ensuite **Créer et envoyer (push) la solution IoT Edge**. Utilisez src/edge/deployment.objectCounter.template.json pour cette étape.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/build-push.png" alt-text="Créer et envoyer (push) la solution IoT Edge":::

Cette action permet de créer le module objectCounter pour le comptage des objets et d’envoyer (push) l’image à Azure Container Registry.

* Vérifiez que les variables d’environnement CONTAINER_REGISTRY_USERNAME_myacr et CONTAINER_REGISTRY_PASSWORD_myacr sont définies dans le fichier .env.

Cette étape permet de créer le manifeste de déploiement IoT Edge dans src/edge/config/deployment.objectCounter.amd64.json. Cliquez avec le bouton droit sur ce fichier, puis sélectionnez **Créer un déploiement pour un seul appareil**.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstarts/create-deployment-single-device.png" alt-text="Créer un déploiement pour un seul appareil":::

S’il s’agit de votre premier tutoriel avec Live Video Analytics sur IoT Edge, Visual Studio Code vous invite à entrer la chaîne de connexion IoT Hub. Vous pouvez copier la chaîne à partir du fichier appsettings.json.

Visual Studio Code vous demande ensuite de sélectionner un appareil IoT Hub. Sélectionnez votre appareil IoT Edge, à savoir lva-sample-device.

À ce stade, le déploiement des modules de périphérie sur votre appareil IoT Edge a démarré.
Après environ 30 secondes, actualisez Azure IoT Hub dans la section inférieure gauche de Visual Studio Code. Vous devez y voir quatre modules déployés nommés lvaEdge, rtspsim, yolov3 et objectCounter.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/iot-hub.png" alt-text="Quatre modules déployés":::

## <a name="prepare-for-monitoring-events"></a>Préparez-vous à des événements de supervision

Pour voir les événements du module objectCounter et du module Live Video Analytics sur IoT Edge, effectuez les étapes suivantes :

1. Ouvrez le volet Explorateur dans Visual Studio Code, puis recherchez **Azure IoT Hub** dans l’angle inférieur gauche.
1. Développez le nœud **Appareils**.
1. Cliquez avec le bouton droit sur le fichier lva-sample-device, puis sélectionnez **Démarrer la supervision du point de terminaison d’événement intégré**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/start-monitoring-iothub-events.png" alt-text="Démarrer la supervision du point de terminaison d’événement intégré":::
    
## <a name="run-the-program"></a>Exécuter le programme

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou appuyez sur Ctrl+Maj+X) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Paramètres d’extension":::
1. Recherchez et activez « Afficher le message détaillé ».

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Afficher le message détaillé":::
1. <!--In Visual Studio Code, go-->Accédez à src/cloud-to-device-console-app/operations.json.
1. Sous le nœud **GraphTopologySet**, modifiez les points suivants :

    `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json"`
    
1. Ensuite, sous les nœuds **GraphInstanceSet** et **GraphTopologyDelete**, modifiez :

    `"topologyName" : "EVRtoAssetsOnObjDetect"`
1. Démarrez une session de débogage en appuyant sur F5. Quelques messages s’affichent dans la fenêtre **TERMINAL**.
1. Le fichier operations.json commence par appeler GraphTopologyList et GraphInstanceList. Si vous avez nettoyé des ressources à l’issue de démarrages rapides ou de tutoriels, cette action renvoie des listes vides et marque des pauses pour vous permettre de sélectionner **Entrée**, comme illustré ci-dessous :

    ```
    --------------------------------------------------------------------------
    Executing operation GraphTopologyList
    -----------------------  Request: GraphTopologyList  --------------------------------------------------
    {
      "@apiVersion": "1.0"
    }
    ---------------  Response: GraphTopologyList - Status: 200  ---------------
    {
      "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    Press Enter to continue
    ```
1. Une fois que vous avez sélectionné **Entrée** dans la fenêtre **TERMINAL**, l’ensemble suivant d’appels de méthode directe est effectué :
   * Un appel à GraphTopologySet à l’aide de la valeur topologyUrl précédente
   * Un appel à GraphInstanceSet à l’aide du corps suivant
     
        ```
        {
          "@apiVersion": "1.0",
          "name": "Sample-Graph-1",
          "properties": {
            "topologyName": "EVRtoAssetsOnObjDetect",
            "description": "Sample graph description",
            "parameters": [
              {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
              },
              {
                "name": "rtspUserName",
                "value": "testuser"
              },
              {
                "name": "rtspPassword",
                "value": "testpassword"
              }
            ]
          }
        }
        ```
    
   * Un appel à GraphInstanceActivate pour démarrer l’instance de graphe et démarrer le flux vidéo
   * Un deuxième appel à GraphInstanceList pour indiquer que l’instance de graphe est en cours d’exécution
     
1. La sortie de la fenêtre **TERMINAL** s’interrompt maintenant à l’invite **Appuyez sur Entrée pour continuer**. Ne sélectionnez pas **Entrée** pour l’instant. Faites défiler vers le haut pour voir les charges utiles de réponse JSON des méthodes directes que vous avez appelées.
1. Si vous basculez maintenant vers la fenêtre **SORTIE** de Visual Studio Code, vous allez voir les messages envoyés à IoT Hub par le module Live Video Analytics sur IoT Edge.

   Ces messages sont abordés dans la section qui suit.
1. L’instance de graphe continue à s’exécuter et à enregistrer la vidéo. Le simulateur RTSP continue de boucler la vidéo source. Passez en revue les messages comme expliqué dans la section suivante. Ensuite, pour arrêter l’instance, revenez à la fenêtre **TERMINAL** et sélectionnez **Entrée**. La série d’appels suivante permet de nettoyer les ressources en utilisant :

   * Un appel à GraphInstanceDeactivate pour désactiver l’instance de graphe.
   * Un appel à GraphInstanceDelete pour supprimer l’instance.
   * Un appel à GraphTopologyDelete pour supprimer la topologie.
   * Un dernier appel à GraphTopologyList pour indiquer que la liste est maintenant vide.

## <a name="interpret-the-results"></a>Interpréter les résultats 

Quand vous exécutez le graphe multimédia, le module Live Video Analytics sur IoT Edge envoie certains événements opérationnels et de diagnostic au hub IoT Edge. Ces événements sont les messages que vous voyez dans la fenêtre **SORTIE** de Visual Studio Code. Ils contiennent une section body et une section applicationProperties. Pour comprendre ce que ces sections représentent, consultez [Créer et lire des messages IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

Dans les messages suivants, les propriétés de l’application et le contenu du corps sont définis par le module Live Video Analytics.

## <a name="diagnostics-events"></a>Événements de diagnostic

### <a name="mediasessionestablished-event"></a>Événement MediaSessionEstablished 

Lorsqu’un graphe multimédia est instancié, le nœud source RTSP tente de se connecter au serveur RTSP exécuté dans le conteneur du simulateur RTSP. En cas de réussite, il imprime cet événement. Le type d’événement est Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished.

```
[IoTHubMonitor] [5:53:17 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 XXX.XX.XX.XX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-05-17T17:53:16.981Z",
    "dataVersion": "1.0"
  }
}
```


* Le message est un événement de diagnostic (MediaSessionEstablished). Il indique que le nœud source RTSP (l’objet) a établi une connexion au simulateur RTSP et a commencé à recevoir un flux en direct (simulé).
* La section subject (l’objet) dans applicationProperties fait référence au nœud dans la topologie du graphe à partir duquel le message a été généré. Dans ce cas, le message provient du nœud source RTSP.
* La section eventType dans applicationProperties indique qu’il s’agit d’un événement de diagnostic.
* La section eventTime indique l’heure à laquelle l’événement s’est produit. Il s’agit de l’heure à laquelle la vidéo du trafic (fichier MKV) a commencé à arriver dans le module sous forme de stream en direct.
* La section body contient des données relatives à l’événement de diagnostic, qui, dans ce cas, correspondent aux détails [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).


## <a name="operational-events"></a>Événements opérationnels

Une fois que le graphe multimédia se sera exécuté pendant un certain temps, vous obtiendrez un événement à partir du module objectCounter. 

```
[IoTHubMonitor] [5:53:44 PM] Message received from [lva-sample-device/objectCounter]:
{
  "body": {
    "count": 2
  },
  "applicationProperties": {
    "eventTime": "2020-05-17T17:53:44.062Z"
  }
}
```

La section applicationProperties contient l’heure de l’événement. Il s’agit de l’heure à laquelle le module objectCounter a constaté que les résultats du module yolov3 contenaient des objets d’intérêt (camions).

Vous pouvez constater que de plus en plus de ces événements apparaissent à mesure que d’autres camions sont détectés dans la vidéo.

### <a name="recordingstarted-event"></a>Événement RecordingStarted

Presque immédiatement après l’envoi de l’événement par le compteur d’objets, vous verrez un événement de type Microsoft.Media.Graph.Operational.RecordingStarted :

```
[IoTHubMonitor] [5:53:46 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation": "sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": " 2020-05-17T17:53:46.132Z",
    "dataVersion": "1.0"
  }
}
```

La section subject dans applicationProperties fait référence au nœud récepteur d’actifs multimédias du graphe qui a généré ce message. La section body contient des informations sur l’emplacement de sortie. Dans ce cas, il s’agit du nom de la ressource Azure Media Services dans laquelle la vidéo est enregistrée. Notez cette valeur.

### <a name="recordingavailable-event"></a>Événement RecordingAvailable

Une fois que le nœud récepteur d’actifs multimédias a chargé la vidéo dans l’actif multimédia, il émet cet événement de type Microsoft.Media.Graph.Operational.RecordingAvailable :

```
[IoTHubMonitor] [5:54:15 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation": "sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingAvailable",
    "eventTime": "2020-05-17T17:54:15.808Z",
    "dataVersion": "1.0"
  }
}
```

Cet événement indique que suffisamment de données ont été écrites dans l’actif multimédia pour que les lecteurs ou clients démarrent la lecture de la vidéo. La section subject dans applicationProperties fait référence au nœud AssetSink du graphe qui a généré ce message. La section body contient des informations sur l’emplacement de sortie. Dans ce cas, il s’agit du nom de la ressource Azure Media Services dans laquelle la vidéo est enregistrée.

### <a name="recordingstopped-event"></a>Événement RecordingStopped

Si vous examinez les paramètres d’activation (maximumActivationTime) pour le nœud processeur de porte de signal dans la [topologie](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json), vous verrez que la porte est configurée pour se fermer après 30 secondes d’envoi de vidéo. Environ 30 secondes après l’événement RecordingStarted, vous devriez voir un événement de type Microsoft.Media.Graph.Operational.RecordingStopped. Cet événement indique que le nœud récepteur d’actifs multimédias a cessé d’enregistrer la vidéo sur l’actif multimédia.

```
[IoTHubMonitor] [5:54:15 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation": "sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStopped",
    "eventTime": "2020-05-17T17:54:15.040Z",
    "dataVersion": "1.0"
  }
}
```

Cet événement indique que l’enregistrement s’est arrêté. La section subject dans applicationProperties fait référence au nœud AssetSink du graphe qui a généré ce message. La section body contient des informations sur l’emplacement de sortie. Dans ce cas, il s’agit du nom de la ressource Azure Media Services dans laquelle la vidéo est enregistrée.

## <a name="media-services-asset"></a>Actif multimédia Media Services  

Vous pouvez examiner l’actif multimédia Media Services qui a été créé par le graphe en vous connectant au portail Azure et en visionnant la vidéo.

1. Ouvrez votre navigateur web pour accéder au [portail Azure](https://portal.azure.com/). Entrez vos informations d’identification pour vous connecter au portail. Il s’ouvre par défaut sur le tableau de bord des services.
1. Localisez votre compte Media Services parmi les ressources que vous avez dans votre abonnement. Ouvrez le volet du compte.
1. Sélectionnez **Ressources** dans la liste **Media Services**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/continuous-video-recording-tutorial/assets.png" alt-text="Enregistrement vidéo en continu":::
1. Vous trouverez un élément multimédia listé portant le nom sampleAssetFromEVR-LVAEdge-{DateHeure}. Il s’agit du nom fourni dans la propriété outputLocation de l’événement RecordingStarted. Dans la topologie, assetNamePattern détermine la façon dont ce nom a été généré.
1. Sélectionnez l’élément multimédia.
1. Dans la page de détails de l’élément multimédia, sélectionnez **Créer** sous la zone de texte **URL de streaming**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/continuous-video-recording-tutorial/new-asset.png" alt-text="Nouvel actif":::
1. Dans l’Assistant qui s’ouvre, acceptez les options par défaut et sélectionnez **Ajouter**. Pour plus d’informations, consultez [Lecture vidéo](video-playback-concept.md).

    > [!TIP]
    > Vérifiez que votre [point de terminaison de streaming est en cours d’exécution](../latest/streaming-endpoint-concept.md).
1. Le lecteur doit charger la vidéo. Sélectionnez **Lire** pour la visionner.

> [!NOTE]
> Étant donné que la source de la vidéo était un conteneur simulant le flux d’une caméra, les horodatages de la vidéo sont liés au moment où vous avez activé l’instance de graphe et où vous l’avez désactivée. Si vous utilisez les commandes de lecture intégrées au tutoriel [Lecture d’enregistrements sur plusieurs jours](playback-multi-day-recordings-tutorial.md), vous pouvez voir l’horodatage de la vidéo affichée à l’écran.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’essayer les autres tutoriels, tenez-vous en aux ressources créées. Sinon, accédez au portail Azure et à vos groupes de ressources, sélectionnez le groupe de ressources sous lequel vous avez exécuté ce tutoriel, puis supprimez le groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

* Utilisez une [caméra IP](https://en.wikipedia.org/wiki/IP_camera) avec prise en charge de RTSP au lieu d’utiliser le simulateur RTSP. Vous pouvez consulter les caméras IP prenant RTSP en charge dans la [page de produits conformes ONVIF](https://www.onvif.org/conformant-products/) en recherchant les appareils conformes aux profils G, S ou T.
* Utilisez un périphérique Linux AMD64 ou x64 (au lieu d’utiliser une machine virtuelle Linux Azure). Cet appareil doit se trouver dans le même réseau que la caméra IP. Suivez les instructions données dans [Installer le runtime Azure IoT Edge sur Linux](../../iot-edge/how-to-install-iot-edge.md). Suivez ensuite les instructions données dans le démarrage rapide [Déployer votre premier module IoT Edge sur un appareil Linux virtuel](../../iot-edge/quickstart-linux.md) pour inscrire l’appareil auprès d’Azure IoT Hub.