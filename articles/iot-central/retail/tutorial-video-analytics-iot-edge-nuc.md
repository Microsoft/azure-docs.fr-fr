---
title: Tutoriel - Créer une instance d'IoT Edge pour l'analytique vidéo dans Azure IoT Central (Intel NUC)
description: Ce tutoriel explique comment créer une instance d'IoT Edge pour l'analytique vidéo afin de l'utiliser avec le modèle d'application d'analytique vidéo pour la détection d'objet et de mouvement.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/27/2020
ms.openlocfilehash: e2e97b857f648390ec017a529115b23c4f17a68d
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427987"
---
# <a name="tutorial-create-an-iot-edge-instance-for-video-analytics-intel-nuc"></a>Tutoriel : Créer une instance d'IoT Edge pour l'analytique vidéo (Intel NUC)

Azure IoT Edge est un service complètement managé qui permet de bénéficier de l'intelligence du cloud localement en déployant et en exécutant les ressources suivantes :

* Logique personnalisée
* Services Azure
* Intelligence artificielle

Dans IoT Edge, ces services s'exécutent directement sur des appareils IoT multiplateformes, ce qui vous permet d'exécuter votre solution IoT en toute sécurité et à grande échelle, tant dans le cloud que hors connexion.

Ce tutoriel explique comment installer et configurer le runtime IoT Edge sur un appareil Intel NUC.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Mettre à jour et configurer IoT Edge
> * Configurer la passerelle IoT Edge
> * Connecter une caméra locale compatible ONVIF à votre appareil Intel NUC

## <a name="prerequisites"></a>Prérequis

* Avant de commencer, vous devez avoir suivi le tutoriel précédent [Créer une application d’analytique vidéo en direct dans Azure IoT Central (YOLO v3)](./tutorial-video-analytics-create-app-yolo-v3.md) ou [Créer une application d’analytique vidéo dans Azure IoT Central (OpenVINO)&trade;](tutorial-video-analytics-create-app-openvino.md).
* Un appareil, de type Intel NUC par exemple, exécutant Linux, capable d'exécuter des conteneurs Docker et disposant d'une puissance de traitement suffisante pour l'analytique vidéo.
* [Runtime IoT Edge installé](../../iot-edge/how-to-install-iot-edge.md) et en cours d'exécution sur l'appareil.
* Pour pouvoir vous connecter à l'appareil IoT Edge à partir de votre ordinateur Windows, vous devez disposer du [client PuTTY SSH](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) ou d'un utilitaire équivalent.
* Vous devez également disposer d'un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un gratuitement dans la [page d’inscription à Azure](https://aka.ms/createazuresubscription).

## <a name="configure-the-iot-edge-device"></a>Configurer un appareil IoT Edge

Si le runtime IoT Edge n'est pas installé sur votre machine Intel NUC, consultez les instructions de l'article [Installer le runtime Azure IoT Edge sur les systèmes Linux basés sur Debian](../../iot-edge/how-to-install-iot-edge.md).

Pour mettre à jour le runtime IoT Edge :

1. Utilisez l'utilitaire PuTTy pour vous connecter à l'appareil IoT Edge.

1. Exécutez les commandes suivantes pour mettre à jour et vérifier la version du runtime IoT Edge. Au moment de la rédaction de cette documentation, la version est 1.0.9 :

    ```bash
    sudo apt-get update
    sudo apt-get install libiothsm iotedge
    sudo iotedge --version
    ```

1. Exécutez les commandes suivantes pour créer les dossiers utilisés par le déploiement avec les autorisations nécessaires :

    ```bash
    sudo mkdir -p /data/storage
    sudo mkdir -p /data/media
    sudo chmod -R 777 /data
    ```

Pour ajouter le fichier de configuration *state.json* au dossier */data/storage* de votre appareil IoT Edge :

1. Utilisez un éditeur de texte pour ouvrir le fichier *state.json* dans le dossier *lva-configuration* de votre machine locale.

1. Remplacez les espaces réservés `system` et `iotCentral > properties` par des valeurs de chaîne qui décrivent votre appareil de passerelle. Vous pourrez afficher ces valeurs ultérieurement dans le tableau de bord de l'application IoT Central.

1. Remplacez les espaces réservés `iotCentral > appKeys` par les valeurs que vous avez notées dans le fichier *scratchpad.txt* du tutoriel précédent. Enregistrez les modifications.

1. Utilisez l'utilitaire PuTTy `scp` dans une invite de commandes pour copier le fichier *state.json* que vous venez de modifier dans le dossier */data/storage* de votre appareil IoT Edge. Cet exemple utilise `192.168.0.144` comme exemple d'adresse IP, et la remplace par l'adresse IP de votre appareil IoT Edge :

    ```cmd
    scp state.json YourUserName@192.168.0.144:/data/storage/state.json`
    ```

Configurez IoT Edge pour l'inscription et la connexion à votre application IoT Central :

1. Utilisez l'utilitaire PuTTy pour vous connecter à l'appareil IoT Edge.

1. Utilisez un éditeur tel que `nano` pour ouvrir le fichier config.yaml d'IoT Edge.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

    > [!WARNING]
    > Les fichiers YAML ne peuvent pas utiliser de tabulations pour la mise en retrait. Utilisez plutôt deux espaces. Les éléments de niveau supérieur ne peuvent pas comporter d'espace de début.

1. Faites défiler jusqu’à faire apparaître `# Manual provisioning configuration`. Placez les trois lignes suivantes en commentaire comme indiqué dans l’extrait de code suivant :

    ```yaml
    # Manual provisioning configuration
    #provisioning:
    #  source: "manual"
    #  device_connection_string: "temp"
    ```

1. Faites défiler jusqu’à faire apparaître `# DPS symmetric key provisioning configuration`. Décommentez les huit lignes suivantes comme indiqué dans l’extrait de code suivant :

    ```yaml
    # DPS symmetric key provisioning configuration
    provisioning:
      source: "dps"
      global_endpoint: "https://global.azure-devices-provisioning.net"
      scope_id: "{scope_id}"
      attestation:
        method: "symmetric_key"
        registration_id: "{registration_id}"
        symmetric_key: "{symmetric_key}"
    ```

1. Remplacez `{scope_id}` par l'**Étendue d'ID** que vous avez notée dans le fichier *scratchpad.txt* du tutoriel précédent.

1. Remplacez `{registration_id}` par *gateway-001*, l’appareil de passerelle que vous avez créé dans le tutoriel précédent.

1. Remplacez `{symmetric_key}` par la **Clé primaire** de l’appareil **gateway-001** que vous avez notée dans le fichier *scratchpad.txt* du tutoriel précédent.

1. Exécutez la commande suivante pour redémarrer le démon IoT Edge :

    ```bash
    sudo systemctl restart iotedge
    ```

1. Pour vérifier l’état des modules IoT Edge, exécutez la commande suivante :

    ```bash
    iotedge list
    ```

    La sortie de la commande précédente indique que cinq modules sont en cours d'exécution. Vous pouvez également afficher l'état des modules en cours d'exécution dans votre application IoT Central.

    > [!TIP]
    > Vous pouvez réexécuter cette commande pour vérifier l'état. Vous devrez peut-être attendre que tous les modules commencent à s’exécuter.

Si les modules IoT Edge ne démarrent pas correctement, consultez [Résoudre les problèmes liés à votre appareil IoT Edge](../../iot-edge/troubleshoot.md).

## <a name="collect-the-rtsp-stream-from-your-camera"></a>Collecter le flux RTSP de votre caméra

Identifiez les URL de flux RTSP pour les caméras connectées à votre appareil IoT Edge, par exemple :

`rtsp://192.168.1.64:554/Streaming/Channels/101/`

> [!TIP]
> Essayez d'afficher le flux de la caméra sur l'ordinateur IoT Edge à l'aide d'un lecteur multimédia tel que VLC.

## <a name="next-steps"></a>Étapes suivantes

Vous venez de déployer le runtime IoT Edge et les modules d'analytique vidéo en direct sur l'appareil de passerelle Intel NUC.

Pour gérer les caméras, reportez-vous au tutoriel suivant :

> [!div class="nextstepaction"]
> [Surveiller et gérer une application d'analytique vidéo pour la détection d'objet et de mouvement](./tutorial-video-analytics-manage.md)