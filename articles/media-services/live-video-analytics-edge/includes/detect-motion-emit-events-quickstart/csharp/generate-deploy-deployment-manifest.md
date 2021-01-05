---
ms.openlocfilehash: 055448eacf1cc12c6d021c6571a0478cb35442ba
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89566899"
---
Le manifeste de déploiement définit les modules qui sont déployés sur un appareil de périphérie. Il définit également les paramètres de configuration de ces modules. 

Effectuez les étapes suivantes pour générer le manifeste à partir du fichier de modèle, puis déployez-le sur l’appareil de périphérie.

1. Ouvrez Visual Studio Code.
1. En regard du volet **AZURE IOT HUB**, sélectionnez l’icône **Autres actions** pour définir la chaîne de connexion IoT Hub. Vous pouvez copier la chaîne à partir du fichier *src/cloud-to-device-console-app/appsettings.json*. 

    ![Définir la chaîne de connexion Azure IoT](../../../media/quickstarts/set-iotconnection-string.png)
1. Cliquez avec le bouton droit sur **src/edge/deployment.template.json**, puis sélectionnez **Générez un manifeste de déploiement IoT Edge**.

    ![Générer le manifeste de déploiement IoT Edge](../../../media/quickstarts/generate-iot-edge-deployment-manifest.png)

    Cette action doit créer un fichier manifeste nommé *deployment.amd64.json* dans le dossier *src/edge/config*.
1. Cliquez avec le bouton droit sur **src/edge/config/deployment.amd64.json**, sélectionnez **Créer un déploiement pour un seul appareil**, puis sélectionnez le nom de votre appareil de périphérie.

    ![Créer un déploiement pour un seul appareil](../../../media/quickstarts/create-deployment-single-device.png)

1. Quand vous êtes invité à sélectionner un appareil IoT Hub, choisissez **lva-sample-device** dans le menu déroulant.
1. Après environ 30 secondes, dans l’angle en bas à gauche de la fenêtre, actualisez Azure IoT Hub. L’appareil de périphérie affiche maintenant les modules déployés suivants :

    * Live Video Analytics sur IoT Edge (nom de module`lvaEdge`)
    * Simulateur RTSP (Real-Time Streaming Protocol) (nom de module `rtspsim`)

Le module de simulateur RTSP simule un flux vidéo en direct à l’aide d’un fichier vidéo qui a été copié sur votre appareil de périphérie quand vous avez exécuté le [script d’installation des ressources Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup). 

> [!NOTE]
> Si vous utilisez votre propre périphérique au lieu de celui approvisionné par notre script de configuration, accédez à votre périphérique et exécutez les commandes suivantes avec des **droits d’administrateur** pour extraire et stocker l’exemple de fichier vidéo utilisé pour ce démarrage rapide :  

```
mkdir /home/lvaadmin/samples      
mkdir /home/lvaadmin/samples/input    
curl https://lvamedia.blob.core.windows.net/public/camera-300s.mkv > /home/lvaadmin/samples/input/camera-300s.mkv  
chown -R lvaadmin /home/lvaadmin/samples/  
```
À ce stade, les modules sont déployés, mais aucun graphe multimédia n’est actif.
