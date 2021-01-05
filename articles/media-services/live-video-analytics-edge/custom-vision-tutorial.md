---
title: Analyser des vidéos en direct avec Live Video Analytics sur IoT Edge et Azure Custom Vision
description: Découvrez comment utiliser Azure Custom Vision pour créer un modèle conteneurisé capable de détecter un camion jouet et d’utiliser la fonctionnalité d’extensibilité de l’IA d’Azure Live Video Analytics sur Azure IoT Edge pour déployer le modèle en périphérie et détecter des jouets à partir d’un flux vidéo en direct.
ms.topic: tutorial
ms.date: 09/08/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: 614c4e401579eda68d8030dc2d2a42b2c4736031
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401693"
---
# <a name="tutorial-analyze-live-video-with-live-video-analytics-on-iot-edge-and-azure-custom-vision"></a>Tutoriel : Analyser des vidéos en direct avec Live Video Analytics sur IoT Edge et Azure Custom Vision

Dans ce tutoriel, vous allez voir comment utiliser Azure [Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) pour créer un modèle conteneurisé capable de détecter un camion jouet et d’utiliser la [fonctionnalité d’extensibilité de l’IA](analyze-live-video-concept.md#analyzing-video-using-a-custom-vision-model) d’Azure Live Video Analytics sur Azure IoT Edge pour déployer le modèle en périphérie et détecter des jouets à partir d’un flux vidéo en direct.

Nous allons vous montrer comment tirer parti de la puissance de Custom Vision en vue de créer et d’entraîner un modèle de vision par ordinateur en chargeant et en étiquetant quelques images. Vous n’avez pas besoin d’avoir des connaissances en science des données, machine learning ou intelligence artificielle. Vous découvrirez également les capacités de Live Video Analytics qui permettent de déployer facilement un modèle personnalisé sous la forme d’un conteneur à la périphérie, et d’analyser un flux vidéo en direct simulé.

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/custom-vision-tutorial/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/custom-vision-tutorial/python/header.md)]
::: zone-end

Ce didacticiel explique les procédures suivantes :

> [!div class="checklist"]
> * Configurer les ressources appropriées.
> * Créer un modèle Custom Vision dans le cloud pour détecter les camions jouets, et le déployer en périphérie
> * Créer et déployer un graphe multimédia avec une extension HTTP sur un modèle Custom Vision
> * Exécuter l’exemple de code.
> * Examinez et interprétez les résultats.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée  

Lisez les articles suivants avant de commencer :

* [Présentation de Live Video Analytics sur IoT Edge](overview.md)
* [Présentation d’Azure Custom Vision](../../cognitive-services/custom-vision-service/overview.md)
* [Terminologie de Live Video Analytics sur IoT Edge](terminology.md)
* [Concepts relatifs aux graphes multimédia](media-graph-concept.md)
* [Live Video Analytics sans enregistrement de vidéo](analyze-live-video-concept.md)
* [Exécuter Live Video Analytics avec votre propre modèle](use-your-model-quickstart.md)
* [Tutoriel : Développement d’un module IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Modification de deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)

## <a name="prerequisites"></a>Prérequis


::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/python/prerequisites.md)]
::: zone-end
## <a name="review-the-sample-video"></a>Réviser l’exemple de vidéo


Ce tutoriel utilise un fichier de [vidéo d’inférence du jouet](https://lvamedia.blob.core.windows.net/public/t2.mkv) pour simuler un stream en direct. Vous pouvez regarder la vidéo grâce à une application telle que [VLC Media Player](https://www.videolan.org/vlc/). Sélectionnez **Ctrl+N**, puis collez un lien vers la [vidéo d’inférence du jouet](https://lvamedia.blob.core.windows.net/public/t2.mkv) pour démarrer la lecture. Lorsque vous regarderez la vidéo, vous verrez qu’un camion jouet apparaît au bout de 36 secondes. Le modèle personnalisé a été entraîné à détecter ce jouet spécifique. Dans ce tutoriel, vous allez utiliser Live Video Analytics sur IoT Edge afin de détecter ce type de jouets et de publier des événements d’inférence associés sur le hub IoT Edge.

## <a name="overview"></a>Vue d’ensemble

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/topology-custom-vision.svg" alt-text="Diagramme montrant une vue d’ensemble de Custom Vision.":::

Ce diagramme montre comment les signaux circulent dans ce tutoriel. Un [module de périphérie](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simule une caméra IP hébergeant un serveur RTSP (Real-Time Streaming Protocol). Un nœud [source RTSP](media-graph-concept.md#rtsp-source) extrait le flux vidéo provenant de ce serveur et envoie des images vidéo au nœud [processeur d’extension HTTP](media-graph-concept.md#http-extension-processor).

Le nœud d’extension HTTP joue le rôle d’un proxy.  Il échantillonne les images vidéo entrantes définies par vous à l’aide du champ `samplingOptions` et convertit également les images vidéo vers le type d’image spécifié. Ensuite, il relaie l’image sur REST vers un autre module de périphérie qui exécute un modèle IA derrière un point de terminaison HTTP. Dans cet exemple, ce module Edge est le modèle de détecteur de camion jouet créé à l’aide de Custom Vision. Le nœud processeur d’extension HTTP collecte les résultats de la détection et publie les événements sur le nœud [récepteur Azure IoT Hub](media-graph-concept.md#iot-hub-message-sink). Le nœud envoie ensuite ces événements au [hub IoT Edge](../../iot-edge/iot-edge-glossary.md#iot-edge-hub).

## <a name="build-and-deploy-a-custom-vision-toy-detection-model"></a>Créer et déployer un modèle de détection de jouet avec Custom Vision 

Comme le nom Custom Vision le suggère, vous pouvez utiliser ce module afin de créer votre propre détecteur ou classifieur personnalisé d’objets dans le cloud. Il fournit une interface simple, facile à utiliser et intuitive pour créer des modèles Custom Vision qui peuvent être déployés dans le cloud ou en périphérie grâce à des conteneurs.

Pour créer un détecteur de camion jouet, suivez les étapes décrites dans [Démarrage rapide : Créer un détecteur d’objets avec le site web Custom Vision](../../cognitive-services/custom-vision-service/get-started-build-detector.md).

Informations complémentaires :
 
* Pour ce tutoriel, n’utilisez pas les exemples d’images fournis dans la [section Prérequis](../../cognitive-services/custom-vision-service/get-started-build-detector.md#prerequisites) du guide de démarrage rapide. Pour créer le modèle Custom Vision de détection des jouets, nous allons utiliser un autre ensemble d’images. Utilisez [ces images](https://lvamedia.blob.core.windows.net/public/ToyCarTrainingImages.zip) quand vous êtes invité à [choisir vos images d’entraînement](../../cognitive-services/custom-vision-service/get-started-build-detector.md#choose-training-images) dans le guide de démarrage rapide.
* Dans la section Étiquetage des images du guide de démarrage rapide, veillez à étiqueter le camion jouet vu dans l’image avec l’étiquette « delivery truck » (camion de livraison).

Une fois que vous avez terminé, vous pouvez exporter le modèle vers un conteneur Docker à l’aide du bouton **Exporter** sous l’onglet **Performances**. Vérifiez que vous avez choisi Linux comme type de plateforme de conteneur. Il s’agit de la plateforme sur laquelle le conteneur s’exécutera. L’ordinateur sur lequel vous téléchargez le conteneur peut fonctionner sous Windows ou sous Linux. Les instructions qui suivent se basent sur le fichier conteneur téléchargé sur un ordinateur Windows.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/docker-file.png" alt-text="Capture d’écran montrant le fichier Dockerfile sélectionné.":::
 
1. Vous devez trouver un fichier zip nommé `<projectname>.DockerFile.Linux.zip` sur votre ordinateur local. 
1. Vérifiez que Docker est installé. Si ce n’est pas le cas, installez [Docker](https://docs.docker.com/get-docker/) sur votre poste de travail Windows.
1. Décompressez le fichier téléchargé à l’emplacement de votre choix. Utilisez la ligne de commande pour accéder au répertoire du dossier décompressé.
    
    Exécutez les commandes suivantes :
    
    1. `docker build -t cvtruck` 
    
        Cette commande télécharge de nombreux packages, génère l’image Docker, et l’étiquette ainsi : `cvtruck:latest`.
    
        > [!NOTE]
        > Si l’opération réussit, vous devez voir les messages suivants : `Successfully built <docker image id>` et `Successfully tagged cvtruck:latest`. Si la commande de génération échoue, réessayez. Il arrive que les packages de dépendances ne puissent pas être téléchargés la première fois.
    1. `docker  image ls`

        Cette commande vérifie si la nouvelle image se trouve dans votre registre local.
    1. `docker run -p 127.0.0.1:80:80 -d cvtruck`
    
        Cette commande doit publier le port exposé de Docker (80) sur le port de votre ordinateur local (80).
    1. `docker container ls`
    
        Cette commande vérifie les mappages de port, et elle vérifie si le conteneur Docker s’exécute correctement sur votre ordinateur. La sortie suivante doit s’afficher :

        ```
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
        8b7505398367        cvtruck             "/bin/sh -c 'python …"   13 hours ago        Up 25 seconds       127.0.0.1:80->80/tcp   practical_cohen
        ```
      1. `curl -X POST http://127.0.0.1:80/image -F imageData=@<path to any image file that has the toy delivery truck in it>`
            
            Cette commande teste le conteneur sur l’ordinateur local. Si l’image montre le même camion de livraison que celui utilisé pour entraîner le modèle, la sortie ressemblera à l’exemple suivant. Celle-ci indique que le camion de livraison a été détecté avec une probabilité de 90,12 %.
    
            ```
            {"created":"2020-03-20T07:10:47.827673","id":"","iteration":"","predictions":[{"boundingBox":{"height":0.66167289,"left":-0.03923762,"top":0.12781593,"width":0.70003178},"probability":0.90128148,"tagId":0,"tagName":"delivery truck"},{"boundingBox":{"height":0.63733053,"left":0.25220079,"top":0.0876643,"width":0.53331227},"probability":0.59745145,"tagId":0,"tagName":"delivery truck"}],"project":""}
            ```

<!--## Create and deploy a media graph with http extension to custom vision model-->

## <a name="examine-the-sample-files"></a>Examiner les exemples de fichiers


::: zone pivot="programming-language-csharp"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/csharp/examine-sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/python/examine-sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-deployment-manifest"></a>Générer et déployer le manifeste de déploiement

1. Dans Visual Studio Code, accédez à src/cloud-to-device-console-app/operations.json.

1. Sous `GraphTopologySet`, vérifiez que ce qui suit est vrai :<br/>`"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtension/topology.json"`
1. Sous `GraphInstanceSet`, vérifiez que ce qui suit est vrai :
    1. `"topologyName" : "InferencingWithHttpExtension"`
    1. Ajoutez ce qui suit en haut du tableau des paramètres : `{"name": "inferencingUrl","value": "http://cv:80/image"},`
    1. Remplacez la valeur du paramètre `rtspUrl` par la valeur `"rtsp://rtspsim:554/media/t2.mkv"`.
1. Sous `GraphTopologyDelete`, vérifiez que `"name": "InferencingWithHttpExtension"` est vrai.
1. Cliquez avec le bouton droit sur le fichier deployment.customvision.template.json du dossier src/edge/, puis sélectionnez **Générer un manifeste de déploiement IoT Edge**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-template-json.png" alt-text="Capture d’écran montrant l’option Générer un manifeste de déploiement IoT Edge":::
  
    Cette action doit créer un fichier manifeste nommé deployment.customvision.amd64.json sous src/edge/config.
1. Ouvrez le fichier deployment.customvision.template.json sous src/edge/, puis recherchez le bloc JSON `registryCredentials`. Dans ce bloc, vous trouverez l’adresse de votre registre de conteneurs Azure, ainsi que son nom d’utilisateur et son mot de passe.
1. Envoyez (push) le conteneur Custom Vision local dans votre instance Azure Container Registry en effectuant les étapes suivantes sur la ligne de commande :

    1. Connectez-vous au registre en exécutant la commande suivante :
    
        `docker login <address>`
    
        Entrez le nom d’utilisateur et le mot de passe lorsqu’ils vous sont demandés pour l’authentification.
        
        > [!NOTE]
        > Le mot de passe ne s’affiche pas sur la ligne de commande.
    1. Étiquetez votre image à l’aide de cette commande : <br/>`docker tag cvtruck   <address>/cvtruck`.
    1. Envoyez (push) votre image à l’aide de cette commande : <br/>`docker push <address>/cvtruck`.

        Si l’envoi réussit, `Pushed` doit s’afficher sur la ligne de commande avec l’algorithme SHA pour l’image.
    1. Vous pouvez également vérifier que l’envoi a réussi en accédant à votre instance Azure Container Registry sur le portail Azure. À cet endroit, le nom du dépôt et l’étiquette s’afficheront.
1. Définissez la chaîne de connexion IoT Hub en sélectionnant l’icône **Autres actions** en regard du volet **AZURE IOT HUB** dans le coin inférieur gauche. Vous pouvez copier la chaîne à partir du fichier appsettings.json. (Pour être sûr que le bon hub IoT est configuré dans Visual Studio Code, vous pouvez également utiliser la [commande Sélectionner un hub IoT](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub))

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/connection-string.png" alt-text="Capture d’écran montrant la chaîne de connexion Définir un hub IoT.":::
1. Ensuite, sous src/edge/config/, cliquez avec le bouton droit sur deployment.customvision.amd64.json, puis sélectionnez **Créer un déploiement pour un seul appareil**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-amd64-json.png" alt-text="Capture d’écran montrant l’option Créer un déploiement pour un seul appareil":::.
1. Vous êtes ensuite invité à sélectionner un appareil IoT Hub. Sélectionnez **lva-sample-device** dans la liste déroulante.
1. Après environ 30 secondes, actualisez Azure IoT Hub dans la section en bas à gauche. Les modules suivants doivent être déployés sur l’appareil de périphérie :

    * Le module Live Video Analytics sur IoT Edge nommé `lvaEdge`
    * Un module nommé `rtspsim`, qui simule un serveur RTSP, agissant comme la source d’un flux vidéo en direct
    * Un module nommé `cv` qui, comme son nom l’indique, est le modèle de détection de camion jouet Custom Vision qui applique une vision personnalisée aux images et retourne plusieurs types d’étiquettes. (Notre modèle a été entraîné à l’aide d’une seule étiquette : « delivery truck » (camion de livraison).)

## <a name="prepare-for-monitoring-events"></a>Préparez-vous à des événements de supervision

Cliquez avec le bouton droit sur l’appareil Live Video Analytics, puis sélectionnez **Démarrer la supervision du point de terminaison d’événement intégré**. Vous devez effectuer cette étape pour superviser les événements IoT Hub dans la fenêtre **SORTIE** de Visual Studio Code.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/start-monitoring.png" alt-text="Capture d’écran de l’option Démarrer la supervision du point de terminaison d’événement intégré":::

## <a name="run-the-sample-program"></a>Exécuter l'exemple de programme

Si vous ouvrez la topologie du graphe de ce tutoriel dans un navigateur, vous verrez que la valeur de `inferencingUrl` a été définie sur `http://cv:80/image`. Ce paramètre signifie que le serveur d’inférence retourne les résultats après avoir détecté des camions jouets dans la vidéo en direct.

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou sélectionnez **Ctrl+Maj+X**) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Capture d’écran montrant les paramètres d’extension":::
1. Recherchez et activez **Afficher le message détaillé**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Capture d’écran montrant l’option Afficher le message détaillé.":::
1. Pour démarrer une session de débogage, sélectionnez la touche **F5**. Des messages s’affichent dans la fenêtre **TERMINAL**.
1. Le code operations.json commence par appeler les méthodes directes `GraphTopologyList` et `GraphInstanceList`. Si vous avez nettoyé les ressources après avoir suivi les guides de démarrage rapide précédents, ce processus retourne des listes vides, puis s’interrompt. Pour continuer, sélectionnez la touche **Entrée**.
    
   La fenêtre **TERMINAL** affiche le jeu d’appels de méthode directe suivant :
    
   * Un appel à `GraphTopologySet` qui utilise le `topologyUrl` précédent.
   * Un appel à `GraphInstanceSet` qui utilise le corps suivant :
        
   ```
        {
          "@apiVersion": "1.0",
          "name": "Sample-Graph-1",
          "properties": {
            "topologyName": "CustomVisionWithHttpExtension",
            "description": "Sample graph description",
            "parameters": [
              { 
                "name": "inferencingUrl",
                "value": "http://cv:80/image"
              },
              {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/t2.mkv"
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
    
   * Un appel à `GraphInstanceActivate` qui démarre l’instance de graphe et le flux vidéo.
   * Un deuxième appel à `GraphInstanceList` qui indique que l’instance de graphe est dans l’état En cours d’exécution.
    
1. La sortie de la fenêtre **TERMINAL** s’interrompt avec l’invite **Appuyez sur Entrée pour continuer**. Ne sélectionnez pas encore **Entrée**. Faites défiler vers le haut pour voir les charges utiles de réponse JSON pour les méthodes directes que vous avez invoquées.
1. Basculez vers la fenêtre **SORTIE** de Visual Studio Code. Les messages indiquant que le module Live Video Analytics sur IoT Edge effectue des envois à IoT Hub s’affichent. La section suivante de ce tutoriel décrit ces messages.
1. Le graphe multimédia continue à s’exécuter et à afficher les résultats. Le simulateur RTSP continue de boucler la vidéo source. Pour arrêter le graphe multimédia, retournez dans la fenêtre **TERMINAL**, puis sélectionnez **Entrée**.
La série d’appels suivante nettoie les ressources :
    
   * Un appel à `GraphInstanceDeactivate` désactive l’instance de graphe.
   * Un appel à `GraphInstanceDelete` supprime l’instance.
   * Un appel à `GraphTopologyDelete` supprime la topologie.
   * Un dernier appel à `GraphTopologyList` indique que la liste est vide.
    
## <a name="interpret-the-results"></a>Interpréter les résultats

Quand vous exécutez le graphe multimédia, les résultats du nœud processeur d’extension HTTP passe par le nœud récepteur IoT Hub au hub IoT. Les messages qui s’affichent dans la fenêtre **SORTIE** contiennent une section Body et une section `applicationProperties`. Pour plus d’informations, consultez [Créer et lire des messages IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

Dans les messages suivants, le module Live Video Analytics définit les propriétés de l’application et le contenu du corps.

### <a name="mediasessionestablished-event"></a>Événement MediaSessionEstablished

Quand un graphe multimédia est instancié, le nœud source RTSP tente de se connecter au serveur RTSP qui s’exécute sur le conteneur rtspsim-live555. Si la connexion réussit, l’événement suivant est affiché. Le type de l’événement est `Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished`.

```
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1599412849822466 1 IN IP4 172.18.0.3\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/t2.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-78.357\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/t2.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=42C01F;sprop-parameter-sets=Z0LAH9kAUAW6EAAAAwAQAAADAwDxgySA,aMuBcsg=\r\na=control:track1\r\nm=audio 0 RTP/AVP 97\r\nc=IN IP4 0.0.0.0\r\nb=AS:96\r\na=rtpmap:97 MPEG4-GENERIC/44100/2\r\na=fmtp:97 streamtype=5;profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3;config=121056E500\r\na=control:track2\r\n"
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lvaavi_0827/providers/microsoft.media/mediaservices/lvasampleulf2rv2x5msy2",
    "subject": "/graphInstances/ GRAPHINSTANCENAMEHERE /sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-09-06T17:20:49.823Z",
    "dataVersion": "1.0"
  }
```

Dans ce message, notez les informations suivantes :

* Le message est un événement de diagnostic. `MediaSessionEstablished` indique que le nœud source RTSP (l’objet) s’est connecté au simulateur RTSP et a commencé à recevoir un flux en direct (simulé).
* Dans `applicationProperties`, `subject` indique que le message a été généré à partir du nœud source RTSP dans le graphe multimédia.
* Dans `applicationProperties`, le type d’événement indique que cet événement est un événement de diagnostic.
* L’heure de l’événement indique l’heure à laquelle l’événement s’est produit.
* Le corps contient des données relatives à l’événement de diagnostic. Dans ce cas, les données comprennent les détails du [protocole SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Événement d'inférence

Le nœud de processeur d’extension HTTP reçoit les résultats d’inférence du conteneur Custom Vision et les émet via le nœud du récepteur IoT Hub en tant qu’événements d’inférence.

```
{
  "body": {
    "created": "2020-05-18T01:08:34.483Z",
    "id": "",
    "iteration": "",
    "predictions": [
      {
        "boundingBox": {
          "height": 0.06219418,
          "left": 0.14977954,
          "top": 0.65847444,
          "width": 0.01879117
        },
        "probability": 0.69806677,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.1148454,
          "left": 0.77430711,
          "top": 0.54488315,
          "width": 0.11315252
        },
        "probability": 0.29394844,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.03187029,
          "left": 0.62644471,
          "top": 0.59640052,
          "width": 0.01582896
        },
        "probability": 0.14435098,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.11421723,
          "left": 0.72737214,
          "top": 0.55907888,
          "width": 0.12627538
        },
        "probability": 0.1141128,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.04717135,
          "left": 0.66516746,
          "top": 0.34617995,
          "width": 0.03802613
        },
        "probability": 0.10461298,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.20650843,
          "left": 0.56349564,
          "top": 0.51482571,
          "width": 0.35045707
        },
        "probability": 0.10056595,
        "tagId": 0,
        "tagName": "delivery truck"
      }
    ],
    "project": ""
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lsravi/providers/microsoft.media/mediaservices/lvasamplerc3he6a7uhire",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/httpExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-05-18T01:08:34.038Z"
  }
}
```

Notez les informations suivantes dans les messages précédents :

* Dans référence `applicationProperties`, l’objet référence le nœud présent dans le graphe multimédia à partir duquel le message a été généré. Dans ce cas, le message provient du processeur d’extension HTTP.
* Dans `applicationProperties`, le type d’événement indique qu’il s’agit d’un événement d’inférence d’analytique.
* L’heure de l’événement indique l’heure à laquelle l’événement s’est produit.
* Le corps contient des données relatives à l’événement d’analytique. Dans ce cas, l’événement est un événement d’inférence. Le corps contient donc un tableau d’inférences appelées « prédictions ».
* La section « predictions » contient la liste des prédictions dans lesquelles un camion jouet (étiquette = « delivery truck ») a été trouvé. Comme vous le savez, « delivery truck » (camion de livraison) est l’étiquette personnalisée que vous avez fournie à votre modèle entraîné personnalisé pour le camion jouet. Le modèle inférence et identifie le camion jouet dans la vidéo d’entrée avec différents scores de probabilité.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’essayer les autres tutoriels ou guides de démarrage rapide, conservez les ressources que vous avez créées. Dans le cas contraire, accédez au portail Azure et à vos groupes de ressources, sélectionnez le groupe de ressources sous lequel vous avez exécuté ce tutoriel, puis supprimez le groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les défis supplémentaires pour les utilisateurs expérimentés :

* Utilisez une [caméra IP](https://en.wikipedia.org/wiki/IP_camera) qui prend en charge RTSP au lieu d’utiliser le simulateur RTSP. Vous pouvez rechercher les caméras IP qui prennent RTSP en charge dans la page des produits [conformes ONVIF](https://www.onvif.org/conformant-products/). Recherchez les appareils conformes aux profils G, S ou T.
* Utilisez un appareil Linux AMD64 ou x64 plutôt qu’une machine virtuelle Linux Azure. Cet appareil doit se trouver dans le même réseau que la caméra IP. Vous pouvez suivre les instructions mentionnées dans [Installer le runtime Azure IoT Edge sur Linux](../../iot-edge/how-to-install-iot-edge.md).

Inscrivez ensuite l’appareil auprès d’Azure IoT Hub en suivant les instructions mentionnées dans [Déployer votre premier module IoT Edge sur un appareil virtuel Linux](../../iot-edge/quickstart-linux.md).