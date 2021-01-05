---
ms.openlocfilehash: e46a56742ab8b98c53c1cd05e840e8ad4b8a73da
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88682117"
---
À l’étape [Générer et déployer le manifeste de déploiement IoT Edge](../../../detect-motion-emit-events-quickstart.md#generate-and-deploy-the-deployment-manifest), dans Visual Studio Code, développez le nœud **lva-sample-device** sous **AZURE IOT HUB** (dans la section en bas à gauche). Vous devez voir les modules suivants déployés :

* Le module Live Video Analytics, nommé `lvaEdge`
* Le module `rtspsim`, qui simule un serveur RTSP faisant office de source d’un flux vidéo en direct

  ![Modules](../../../media/quickstarts/lva-sample-device-node.png)

> [!NOTE]
> Si vous utilisez votre propre périphérique au lieu de celui approvisionné par notre script de configuration, accédez à votre périphérique et exécutez les commandes suivantes avec des **droits d'administrateur** pour extraire et stocker l'exemple de fichier vidéo utilisé avec ce guide de démarrage rapide :  

```
mkdir /home/lvaadmin/samples
mkdir /home/lvaadmin/samples/input    
curl https://lvamedia.blob.core.windows.net/public/camera-300s.mkv > /home/lvaadmin/samples/input/camera-300s.mkv  
chown -R lvaadmin /home/lvaadmin/samples/  
```
