---
ms.openlocfilehash: 304f62def117d1b1a803b2f604151c5e52dbe7be
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97532027"
---
### <a name="examine-and-edit-the-sample-files"></a>Examiner et modifier les exemples de fichiers

Dans le cadre des prérequis, vous avez téléchargé l’exemple de code dans un dossier. Effectuez les étapes suivantes pour examiner et modifier les exemples de fichiers.

1. Dans Visual Studio Code, accédez à *src/edge*. Votre fichier  *.env* et quelques fichiers de modèle de déploiement s’affichent.

    Le modèle de déploiement fait référence au manifeste de déploiement de l’appareil de périphérie. Il inclut des valeurs d’espace réservé. Le fichier  *.env* inclut les valeurs de ces variables.
1. Accédez au dossier *src/cloud-to-device-console-app*. Ce dernier contient votre fichier *appsettings.json* et quelques autres fichiers :

    * ***c2d-console-app.csproj** _ : fichier projet pour Visual Studio Code.
    _ ***operations.json** _ : liste des opérations que vous voulez que le programme exécute.
    _ ***Program.cs** _ : exemple de code de programme. Ce code :

        _ Charge les paramètres de l’application.
        * Invoque les méthodes directes exposées par le module Live Video Analytics sur IoT Edge. Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant ses [méthodes directes](../../../direct-methods.md).
        * S’interrompt pour vous permettre d’examiner la sortie du programme dans la fenêtre **TERMINAL** et d’examiner les événements qui ont été générés par le module dans la fenêtre **SORTIE**.
        * Invoque des méthodes directes pour nettoyer des ressources.
1. Modifiez le fichier *operations.json* :
    * Changez le lien vers la topologie de graphe :

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtension/2.0/topology.json"`

    * Sous `GraphInstanceSet`, modifiez le nom de la topologie de graphe de manière à ce qu’il corresponde à la valeur figurant dans le lien ci-dessus précédent :

      `"topologyName" : "InferencingWithHttpExtension"`

    * Sous `GraphTopologyDelete`, modifiez le nom :

      `"name": "InferencingWithHttpExtension"`

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Générez et déployez le manifeste de déploiement IoT Edge

1. Cliquez avec le bouton droit sur le fichier *src/edge/ deployment.yolov3.template.json*, puis sélectionnez **Générer un manifeste de déploiement IoT Edge**.

    ![Générez un manifeste de déploiement IoT Edge](../../../media/quickstarts/generate-iot-edge-deployment-manifest-yolov3.png)  

    Le fichier de manifeste *deployment.yolov3.amd64.json* est créé dans le dossier *src/edge/config*.
1. Si vous avez suivi le guide de démarrage rapide [Détecter les événements de mouvement et d’émission](../../../detect-motion-emit-events-quickstart.md), ignorez cette étape. 

    Dans le cas contraire, en regard du volet **AZURE IOT HUB** dans l’angle en bas à gauche, sélectionnez l’icône **Autres actions**, puis sélectionnez **Définir la chaîne de connexion IoT Hub**. Vous pouvez copier la chaîne à partir du fichier *appsettings.json*. Ou, pour garantir que vous avez configuré le hub IoT approprié dans Visual Studio Code, utilisez la [commande Sélectionner IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).
    
    ![Définir la chaîne de connexion IoT Hub](../../../media/quickstarts/set-iotconnection-string.png)
1. Cliquez avec le bouton droit sur *src/edge/config/ deployment.yolov3.amd64.json*, puis sélectionnez **Créer un déploiement pour un seul appareil**. 

    ![Créer un déploiement pour un seul appareil](../../../media/quickstarts/create-deployment-single-device.png)
1. Quand vous êtes invité à sélectionner un appareil IoT Hub, sélectionnez **lva-sample-device**.
1. Après environ 30 secondes, dans l’angle en bas à gauche de la fenêtre, actualisez Azure IoT Hub. L’appareil de périphérie affiche maintenant les modules déployés suivants :

    * Le module Live Video Analytics, nommé `lvaEdge`.
    * Le module `rtspsim`, qui simule un serveur RTSP et fait office de source d’un flux vidéo en direct.

        > [!NOTE]
        > Si vous utilisez votre propre périphérique au lieu de celui approvisionné par notre script de configuration, accédez à votre périphérique et exécutez les commandes suivantes avec des **droits d’administrateur** pour extraire et stocker l’exemple de fichier vidéo utilisé pour ce démarrage rapide :  
        
        ```
        mkdir /home/lvaadmin/samples
        mkdir /home/lvaadmin/samples/input    
        curl https://lvamedia.blob.core.windows.net/public/camera-300s.mkv > /home/lvaadmin/samples/input/camera-300s.mkv  
        chown -R lvaadmin /home/lvaadmin/samples/  
        ```
        * Module `yolov3`, qui est le modèle de détection d’objet Yolov3 qui applique la vision par ordinateur aux images, et retourne plusieurs classes de types d’objets
 
      ![Modules qui sont déployés sur l’appareil de périphérie](../../../media/quickstarts/yolov3.png)

### <a name="prepare-to-monitor-events"></a>Préparer la supervision d’événements

1. Dans Visual Studio Code, ouvrez l’onglet **Extensions** (ou appuyez sur Ctrl+Maj+X) et recherchez Azure IoT Hub.
1. Cliquez avec le bouton droit et sélectionnez **Paramètres d’extension**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/extensions-tab.png" alt-text="Paramètres d’extension":::
1. Recherchez et activez « Afficher le message détaillé ».

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/show-verbose-message.png" alt-text="Afficher le message détaillé":::
1. Cliquez avec le bouton droit sur l’appareil Live Video Analytics, puis sélectionnez sur **Démarrer la supervision du point de terminaison d’événement intégré**. Vous devez effectuer cette étape pour superviser les événements IoT Hub dans la fenêtre **SORTIE** de Visual Studio Code. 

   ![Démarrer la surveillance](../../../media/quickstarts/start-monitoring-iothub-events.png) 

### <a name="run-the-sample-program"></a>Exécuter l'exemple de programme

1. Pour démarrer une session de débogage, appuyez sur la touche F5. Des messages s’affichent dans la fenêtre **TERMINAL**.
1. Le code *operations.json* commence par appeler les méthodes directes `GraphTopologyList` et `GraphInstanceList`. Si vous avez nettoyé les ressources après avoir suivi les guides de démarrage rapide précédents, ce processus retourne des listes vides, puis s’interrompt. Pour continuer, sélectionnez la touche Entrée.

   ```
   --------------------------------------------------------------------------
   Executing operation GraphTopologyList
   -----------------------  Request: GraphTopologyList  --------------------------------------------------
   {
   "@apiVersion": "2.0"
   }
   ---------------  Response: GraphTopologyList - Status: 200  ---------------
   {
   "value": []
   }
   --------------------------------------------------------------------------
   Executing operation WaitForInput
   Press Enter to continue
   ```

    La fenêtre **TERMINAL** affiche le jeu d’appels de méthode directe suivant :

     * Un appel à `GraphTopologySet` qui utilise le `topologyUrl` précédent
     * Un appel à `GraphInstanceSet` qui utilise le corps suivant :

         ```
         {
           "@apiVersion": "2.0",
           "name": "Sample-Graph-1",
           "properties": {
             "topologyName": "InferencingWithHttpExtension",
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
