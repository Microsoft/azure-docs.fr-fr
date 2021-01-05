---
title: Analyser la vidéo en direct à l’aide de l’extension IA OpenVINO™ Model Server d’Intel
description: Dans ce tutoriel, vous allez utiliser un serveur de modèles IA fourni par Intel pour analyser le flux vidéo en direct provenant d’une caméra IP (simulée).
ms.topic: tutorial
ms.date: 09/08/2020
titleSuffix: Azure
ms.openlocfilehash: 82906111e64bd278d4371d1c3497fefc4510bbbd
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401205"
---
# <a name="tutorial-analyze-live-video-by-using-openvino-model-server--ai-extension-from-intel"></a>Tutoriel : Analyser la vidéo en direct à l’aide de l’extension IA OpenVINO™ Model Server d’Intel 

Ce tutoriel montre comment utiliser l’extension IA OpenVINO™ Model Server d’Intel pour analyser un flux vidéo en direct provenant d’une caméra IP (simulée). Vous verrez comment ce serveur d’inférence vous donne accès à des modèles pour la détection d’objets (une personne, un véhicule ou un vélo) et à un modèle pour la classification des véhicules. Un sous-ensemble d’images du flux vidéo en direct est envoyé à ce serveur d’inférence, et les résultats sont envoyés au hub IoT Edge.

Ce tutoriel utilise une machine virtuelle Azure comme appareil IoT Edge, ainsi qu’un flux vidéo en direct simulé. Il est basé sur l’exemple de code écrit en C# et s’appuie sur le guide de démarrage rapide [Détecter les événements de mouvement et d’émission](detect-motion-emit-events-quickstart.md).

> [!NOTE]
> Ce didacticiel nécessite l’utilisation d’un ordinateur x86-64 comme appareil de périphérie.

## <a name="prerequisites"></a>Prérequis

* Un compte Azure incluant un abonnement actif. Si vous n’en avez pas déjà un, [créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Visual Studio Code](https://code.visualstudio.com/), avec les extensions suivantes :
    * [Outils IoT Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [Kit SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Si vous n’avez pas suivi le guide de démarrage rapide [Détecter les événements de mouvement et d’émission](detect-motion-emit-events-quickstart.md), veillez à effectuez les étapes de [configuration des ressources Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources).

> [!TIP]
> Lors de l’installation d’Azure IoT Tools, vous pouvez être invité à installer Docker. Vous pouvez ignorer cette invite.

## <a name="review-the-sample-video"></a>Réviser l’exemple de vidéo

Quand vous configurez les ressources Azure, une courte vidéo d’un parking est copiée dans Azure sur la machine virtuelle Linux que vous utilisez comme appareil IoT Edge. Ce guide de démarrage rapide utilise le fichier vidéo pour simuler un flux en direct.

Ouvrez une application comme le [lecteur multimédia VLC](https://www.videolan.org/vlc/). Sélectionnez Ctrl+N, puis collez un lien vers [la vidéo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) pour démarrer la lecture. Vous voyez la séquence vidéo des véhicules sur le parking, la plupart d’entre eux garés et l’un d’eux en déplacement.

Dans ce guide de démarrage rapide, vous allez utiliser Live Video Analytics sur IoT Edge ainsi que l’extension IA OpenVINO™ Model Server d’Intel pour détecter des objets tels que des véhicules ou pour les classifier. Vous publierez les événements d’inférence associés sur un hub IoT Edge.

## <a name="overview"></a>Vue d’ensemble

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/http-extension-with-vino.svg" alt-text="Vue d'ensemble":::

Ce diagramme montre comment les signaux circulent dans ce guide de démarrage rapide. Un [module de périphérie](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simule une caméra IP hébergeant un serveur RTSP (Real-Time Streaming Protocol). Un nœud [source RTSP](media-graph-concept.md#rtsp-source) extrait le flux vidéo provenant de ce serveur et envoie des images vidéo au nœud [processeur d’extension HTTP](media-graph-concept.md#http-extension-processor). 

Le nœud d’extension HTTP joue le rôle d’un proxy. Il échantillonne les images vidéo entrantes définies par votre champ `samplingOptions` et convertit également les images vidéo en type d’image spécifié. Ensuite, il relaie l’image sur REST vers un autre module de périphérie qui exécute des modèles IA derrière un point de terminaison HTTP. Dans cet exemple, ce module de périphérie est l’extension IA OpenVINO™ Model Server d’Intel. Le nœud processeur d’extension HTTP collecte les résultats de la détection et publie les événements sur le nœud [récepteur IoT Hub](media-graph-concept.md#iot-hub-message-sink). Le nœud envoie ensuite ces événements à [IoT Edge Hub](../../iot-edge/iot-edge-glossary.md#iot-edge-hub).

Ce didacticiel présente les procédures suivantes :

1. Créer et déployer le graphe multimédia, et le modifier.
1. interpréter les résultats ;
1. Supprimer des ressources.

## <a name="about-openvino-model-server--ai-extension-from-intel"></a>À propos de l’extension IA OpenVINO™ Model Server d’Intel

La distribution Intel® du [kit de ressources OpenVINO™](https://software.intel.com/content/www/us/en/develop/tools/openvino-toolkit.html) (inférence visuelle ouverte et optimisation de réseau neuronal) est un kit logiciel gratuit qui permet aux développeurs et aux scientifiques des données d’accélérer les charges de travail de vision par ordinateur, de rationaliser l’inférence et les déploiements de deep learning, et de bénéficier d’exécutions simples et hétérogènes sur toutes les plateformes Intel®, de la périphérie au cloud. Elle comprend le kit de ressources de déploiement deep learning d’Intel® avec optimiseur de modèle et moteur d’inférence, ainsi que le référentiel [Open Model Zoo](https://github.com/openvinotoolkit/open_model_zoo) qui comprend plus de 40 modèles pré-entraînés et optimisés.

Pour créer des solutions d’analytique vidéo en direct complexes et à hautes performances, le module Live Video Analytics sur IoT Edge doit être associé à un moteur d’inférence puissant capable de tirer parti de l’échelle à la périphérie. Dans ce tutoriel, les requêtes d’inférence sont envoyées à l’[extension IA OpenVINO™ Model Server d’Intel](https://aka.ms/lva-intel-ovms), un module Edge conçu pour fonctionner avec Live Video Analytics sur IoT Edge. Ce module de serveur d’inférence contient OpenVINO™ Model Server (OVMS), un serveur d’inférence tirant parti du kit de ressources OpenVINO™, hautement optimisé pour les charges de travail de vision par ordinateur et développé pour les architectures Intel®. Une extension a été ajoutée à OVMS pour faciliter l’échange de trames vidéo et de résultats d’inférence entre le serveur inférence et le module Live Video Analytics sur IoT Edge, ce qui vous permet d’exécuter n’importe quel modèle de boîte à outils OpenVINO™ pris en charge (vous pouvez personnaliser le module de serveur d’inférence en modifiant le [code](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_wrapper)). Vous pouvez de plus choisir parmi la large gamme de mécanismes d’accélération fournis par le matériel Intel®. notamment les processeurs (Atom, Core, Xeon), FPGA et VPU.

Dans la version initiale de ce serveur d’inférence, vous avez accès aux [modèles](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_models) suivants :

- Détection de véhicule (URL d’inférence : http://{nom_module}:4000/vehicleDetection)
- Détection de personne/véhicule/vélo (URL d’inférence : http://{nom_module}:4000/personVehicleBikeDetection)
- Classification de véhicule (URL d’inférence : http://{nom_module}:4000/vehicleClassification)
- Détection de visage (URL d’inférence : http://{nom_module}:4000/faceDetection)

> [!NOTE]
> En téléchargeant et en utilisant le module Edge d’extension IA pour OpenVINO™ Model Server d’Intel et le logiciel inclus, vous acceptez les termes et conditions du [contrat de licence](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel s’engage à respecter les droits de l’homme et à éviter toute complicité en matière d’abus des droits de l’homme. Consultez les [principes généraux d’Intel en matière de droits de l’homme](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Les produits et logiciels Intel sont destinés uniquement à être utilisés dans des applications qui n’entraînent pas ou ne contribuent pas à une violation d’un droit humain reconnu au niveau international.

## <a name="create-and-deploy-the-media-graph"></a>Créer et déployer le graphe multimédia

### <a name="examine-and-edit-the-sample-files"></a>Examiner et modifier les exemples de fichiers

Dans le cadre des prérequis, vous avez téléchargé l’exemple de code dans un dossier. Effectuez les étapes suivantes pour examiner et modifier les exemples de fichiers.

1. Dans Visual Studio Code, accédez à *src/edge*. Votre fichier  *.env* et quelques fichiers de modèle de déploiement s’affichent.

    Le modèle de déploiement fait référence au manifeste de déploiement de l’appareil de périphérie. Il inclut des valeurs d’espace réservé. Le fichier  *.env* inclut les valeurs de ces variables.

1. Accédez au dossier *src/cloud-to-device-console-app*. Ce dernier contient votre fichier *appsettings.json* et quelques autres fichiers :

    * ***c2d-console-app.csproj** _ : fichier projet pour Visual Studio Code.
    _ ***operations.json** _ : liste des opérations que vous voulez que le programme exécute.
    _ ***Program.cs** _ : exemple de code de programme. Ce code :

        _ Charge les paramètres de l’application.
        * Invoque les méthodes directes exposées par le module Live Video Analytics sur IoT Edge. Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant ses [méthodes directes](direct-methods.md).
        * S’interrompt pour vous permettre d’examiner la sortie du programme dans la fenêtre **TERMINAL** et d’examiner les événements qui ont été générés par le module dans la fenêtre **SORTIE**.
        * Invoque des méthodes directes pour nettoyer des ressources.


1. Modifiez le fichier *operations.json* :
    * Changez le lien vers la topologie de graphe :

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json"`

    * Sous `GraphInstanceSet`, modifiez le nom de la topologie de graphe de manière à ce qu’il corresponde à la valeur figurant dans le lien ci-dessus précédent :

      `"topologyName" : "InferencingWithOpenVINO"`

    * Sous `GraphTopologyDelete`, modifiez le nom :

      `"name": "InferencingWithOpenVINO"`

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Générez et déployez le manifeste de déploiement IoT Edge

1. Cliquez avec le bouton droit sur le fichier *src/edge/deployment.openvino.template.json*, puis sélectionnez **Générer un manifeste de déploiement IoT Edge**.

    ![Générez un manifeste de déploiement IoT Edge](./media/use-intel-openvino-tutorial/generate-deployment-manifest.png)  

    Le fichier de manifeste *deployment.yolov3.amd64.json* est créé dans le dossier *src/edge/config*.

1. Si vous avez suivi le guide de démarrage rapide [Détecter les événements de mouvement et d’émission](detect-motion-emit-events-quickstart.md), ignorez cette étape. 

    Dans le cas contraire, en regard du volet **AZURE IOT HUB** dans l’angle en bas à gauche, sélectionnez l’icône **Autres actions**, puis sélectionnez **Définir la chaîne de connexion IoT Hub**. Vous pouvez copier la chaîne à partir du fichier *appsettings.json*. Ou, pour garantir que vous avez configuré le hub IoT approprié dans Visual Studio Code, utilisez la [commande Sélectionner IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).
    
    ![Définir la chaîne de connexion IoT Hub](./media/quickstarts/set-iotconnection-string.png)

1. Cliquez avec le bouton droit sur *src/edge/config/deployment.openvino.amd64.json*, puis sélectionnez **Créer un déploiement pour un seul appareil**. 

    ![Créer un déploiement pour un seul appareil](./media/use-intel-openvino-tutorial/deploy-manifest.png)

1. Quand vous êtes invité à sélectionner un appareil IoT Hub, sélectionnez **lva-sample-device**.
1. Après environ 30 secondes, dans l’angle en bas à gauche de la fenêtre, actualisez Azure IoT Hub. L’appareil de périphérie affiche maintenant les modules déployés suivants :

    * Le module Live Video Analytics, nommé **lvaEdge**
    * Le module **rtspsim**, qui simule un serveur RTSP et fait office de source d’un flux vidéo en direct
    * Le module **openvino**, qui est le module d’extension IA pour OpenVINO™ Model Server d’Intel 

### <a name="prepare-to-monitor-events"></a>Préparer la supervision d’événements

Cliquez avec le bouton droit sur l’appareil Live Video Analytics, puis sélectionnez sur **Démarrer la supervision du point de terminaison d’événement intégré**. Vous devez effectuer cette étape pour superviser les événements IoT Hub dans la fenêtre **SORTIE** de Visual Studio Code. 

![Démarrer la surveillance](./media/quickstarts/start-monitoring-iothub-events.png) 

### <a name="run-the-sample-program-to-detect-vehicles"></a>Exécuter l’exemple de programme pour détecter les véhicules
Si vous ouvrez la [topologie de graphe](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) de ce tutoriel dans un navigateur, vous constaterez que la valeur de `inferencingUrl` a été définie sur `http://openvino:4000/vehicleDetection`, ce qui signifie que le serveur d’inférence retournera les résultats après la détection des véhicules, le cas échéant, dans la vidéo en direct.

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou appuyez sur Ctrl+Maj+X) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Paramètres d’extension":::
1. Recherchez et activez « Afficher le message détaillé ».

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Afficher le message détaillé":::
1. Pour démarrer une session de débogage, appuyez sur la touche F5. Des messages s’affichent dans la fenêtre **TERMINAL**.
1. Le code *operations.json* commence par appeler les méthodes directes `GraphTopologyList` et `GraphInstanceList`. Si vous avez nettoyé les ressources après avoir suivi les guides de démarrage rapide précédents, ce processus retourne des listes vides, puis s’interrompt. Pour continuer, sélectionnez la touche Entrée.

    La fenêtre **TERMINAL** affiche le jeu d’appels de méthode directe suivant :

     * Un appel à `GraphTopologySet` qui utilise le `topologyUrl` précédent
     * Un appel à `GraphInstanceSet` qui utilise le corps suivant :

         ```
         {
           "@apiVersion": "1.0",
           "name": "Sample-Graph-1",
           "properties": {
             "topologyName": "InferencingWithOpenVINO",
             "description": "Sample graph description",
             "parameters": [
               {
                 "name": "rtspUrl",
                 "value": "rtsp://rtspsim:554/media/lots_015.mkv"
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

     * Un appel à `GraphInstanceActivate` qui démarre l’instance de graphe et le flux vidéo
     * Un deuxième appel à `GraphInstanceList` qui indique que l’instance de graphe est dans l’état En cours d’exécution
1. La sortie affichée dans la fenêtre **TERMINAL** fait une pause au niveau d’une invite `Press Enter to continue`. Ne sélectionnez pas encore Entrée. Faites défiler vers le haut pour voir les charges utiles de réponse JSON pour les méthodes directes que vous avez invoquées.
1. Basculez vers la fenêtre **SORTIE** de Visual Studio Code. Les messages indiquant que le module Live Video Analytics sur IoT Edge effectue des envois à IoT Hub s’affichent. La section suivante de ce guide de démarrage rapide décrit ces messages.
1. Le graphe multimédia continue à s’exécuter et à afficher les résultats. Le simulateur RTSP continue de boucler la vidéo source. Pour arrêter le graphe multimédia, retournez dans la fenêtre **TERMINAL**, puis sélectionnez Entrée. 

    La série d’appels suivante nettoie les ressources :
      * Un appel à `GraphInstanceDeactivate` désactive l’instance de graphe.
      * Un appel à `GraphInstanceDelete` supprime l’instance.
      * Un appel à `GraphTopologyDelete` supprime la topologie.
      * Un dernier appel à `GraphTopologyList` indique que la liste est vide.

## <a name="interpret-results"></a>Interpréter les résultats

Quand vous exécutez le graphe multimédia, les résultats du nœud processeur d’extension HTTP passe par le nœud récepteur IoT Hub au hub IoT. Les messages qui s’affichent dans la fenêtre **SORTIE** contiennent une section `body` et une section `applicationProperties`. Pour plus d’informations, consultez [Créer et lire des messages IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

Dans les messages suivants, le module Live Video Analytics définit les propriétés de l’application et le contenu du corps. 

### <a name="mediasessionestablished-event"></a>Événement MediaSessionEstablished

Quand un graphe multimédia est instancié, le nœud source RTSP tente de se connecter au serveur RTSP qui s’exécute sur le conteneur rtspsim-live555. Si la connexion réussit, l’événement suivant est affiché. Le type de l’événement est `Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished`.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [lvaedgesample/lvaEdge]:
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/sources/rtspSource",
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-07-24T16:42:18.1280000Z"
  }
}
```

Dans ce message, notez les informations suivantes :

* Le message est un événement de diagnostic. `MediaSessionEstablished` indique que le nœud source RTSP (l’objet) s’est connecté au simulateur RTSP et a commencé à recevoir un flux en direct (simulé).
* Dans `applicationProperties`, `subject` indique que le message a été généré à partir du nœud source RTSP dans le graphe multimédia.
* Dans `applicationProperties`, `eventType` indique que cet événement est un événement de diagnostic.
* `eventTime` indique l’heure à laquelle l’événement s’est produit.
* `body` contient des données relatives à l’événement de diagnostic. Dans ce cas, les données comprennent les détails du [protocole SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Événement d'inférence

Le nœud processeur d’extension HTTP reçoit les résultats d’inférence à partir du module d’extension IA pour OpenVINO™ Model Server. Il émet ensuite les résultats par le biais du nœud récepteur IoT Hub sous la forme d’événements d’inférence. 

Dans ces événements, le type est défini sur `entity` pour indiquer qu’il s’agit d’une entité, comme une voiture ou un camion. La valeur `eventTime` est l’heure UTC à laquelle l’objet a été détecté. 

Dans l’exemple suivant, deux véhicules ont été détectés, avec une valeur de confiance supérieure à 0,9.

```
[IoTHubMonitor] [9:43:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.9951713681221008
          },
          "box": {
            "l": 0.042635321617126465,
            "t": 0.4004564881324768,
            "w": 0.10961548984050751,
            "h": 0.07942074537277222
          }
        }
      },
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.928486168384552
          },
          "box": {
            "l": 0.2506900727748871,
            "t": 0.07512682676315308,
            "w": 0.05470699071884155,
            "h": 0.07408371567726135
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:43:18.1280000Z"
  }
}
```

Dans le message, notez les informations suivantes :

* Dans `applicationProperties`, `subject` fait référence au nœud présent dans la topologie de graphe à partir duquel le message a été généré. 
* Dans `applicationProperties`, `eventType` indique que cet événement est un événement d’analytique.
* La valeur `eventTime` correspond à l’heure à laquelle l’événement s’est produit.
* La section `body` contient des données relatives à l’événement d’analytique. Dans ce cas, l’événement est un événement d’inférence. Par conséquent, le corps contient des données `inferences`.
* La section `inferences` indique que le `type` est `entity`. Cette section inclut des données supplémentaires sur l’entité.

## <a name="run-the-sample-program-to-detect-persons-or-vehicles-or-bikes"></a>Exécuter l’exemple de programme pour détecter les personnes, les véhicules ou les vélos
Pour utiliser un modèle différent, vous devez modifier la topologie de graphe, ainsi que le fichier `operations.json`.

Copiez la [topologie de graphe](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) dans un fichier local, par exemple `C:\TEMP\topology.json`. Ouvrez cette copie, puis affectez `http://openvino:4000/personVehicleBikeDetection` comme valeur de `inferencingUrl`.

Ensuite, dans Visual Studio Code, accédez au dossier *src/cloud-to-device-console-app* et ouvrez le fichier `operations.json`. Modifiez la ligne indiquant `topologyUrl` comme suit :

```
      "topologyFile" : "C:\\TEMP\\topology.json" 
```
Vous pouvez maintenant répéter les étapes ci-dessus pour réexécuter l’exemple de programme avec la nouvelle topologie. Les résultats de l’inférence seront similaires (dans le schéma) à ceux du modèle de détection de véhicule, mais avec `subtype` défini sur `personVehicleBikeDetection`.

## <a name="run-the-sample-program-to-classify-vehicles"></a>Exécuter l’exemple de programme pour classifier les véhicules
Dans Visual Studio Code, ouvrez la copie locale du fichier `topology.json` de l’étape précédente, puis remplacez la valeur de `inferencingUrl` par `http://openvino:4000/vehicleClassification`. Si vous avez exécuté l’exemple précédent pour détecter des personnes, des véhicules ou des vélos, vous n’avez pas besoin de remodifier le fichier `operations.json`.

Vous pouvez maintenant répéter les étapes ci-dessus pour réexécuter l’exemple de programme avec la nouvelle topologie. Voici un exemple de résultat de la classification.

```
[IoTHubMonitor] [9:44:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "classification",
        "subtype": "color",
        "classification": {
          "tag": {
            "value": "black",
            "confidence": 0.9179772138595581
          }
        }
      },
      {
        "type": "classification",
        "subtype": "type",
        "classification": {
          "tag": {
            "value": "truck",
            "confidence": 1
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:44:18.1280000Z"
  }
}
```

## <a name="run-the-sample-program-to-detect-faces"></a>Exécuter l’exemple de programme pour détecter les visages
Dans Visual Studio Code, ouvrez la copie locale du fichier `topology.json` de l’étape précédente, puis remplacez la valeur de `inferencingUrl` par `http://openvino:4000/faceDetection`. Si vous avez exécuté l’exemple précédent pour détecter des personnes, des véhicules ou des vélos, vous n’avez pas besoin de remodifier le fichier `operations.json`.

Vous pouvez maintenant répéter les étapes ci-dessus pour réexécuter l’exemple de programme avec la nouvelle topologie. Vous trouverez ci-dessous un exemple de résultat de détection (Remarque : La vidéo du parking utilisée plus haut ne contient pas de visages détectables ; vous devez utiliser une autre vidéo pour tester ce modèle).

```
[IoTHubMonitor] [9:54:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "faceDetection",
        "entity": {
          "tag": {
            "value": "face",
            "confidence": 0.9997053742408752
          },
          "box": {
            "l": 0.2559490501880646,
            "t": 0.03403960168361664,
            "w": 0.17685115337371826,
            "h": 0.45835764706134796
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:54:18.1280000Z"
  }
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’essayer d’autres guides de démarrage rapide ou tutoriels, conservez les ressources que vous avez créées. Dans le cas contraire, accédez au portail Azure, puis à vos groupes de ressources. Sélectionnez le groupe de ressources où vous avez exécuté ce tutoriel, puis supprimez toutes les ressources.

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les défis supplémentaires pour les utilisateurs expérimentés :

* Utilisez une [caméra IP](https://en.wikipedia.org/wiki/IP_camera) qui prend en charge RTSP au lieu d’utiliser le simulateur RTSP. Vous pouvez rechercher les caméras IP qui prennent RTSP en charge dans la page des produits [conformes ONVIF](https://www.onvif.org/conformant-products/). Recherchez les appareils conformes aux profils G, S ou T.
* Utilisez un appareil Linux AMD64 ou x64 plutôt qu’une machine virtuelle Linux Azure. Cet appareil doit se trouver dans le même réseau que la caméra IP. Vous pouvez suivre les instructions mentionnées dans [Installer le runtime Azure IoT Edge sur Linux](../../iot-edge/how-to-install-iot-edge.md). Inscrivez ensuite l’appareil auprès d’Azure IoT Hub en suivant les instructions mentionnées dans [Déployer votre premier module IoT Edge sur un appareil virtuel Linux](../../iot-edge/quickstart-linux.md).