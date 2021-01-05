---
title: 'Tutoriel : Modifier les modules d’analytique vidéo en direct Azure IoT Edge'
description: Ce tutoriel explique comment modifier et créer les modules de passerelle d’analytique vidéo en direct qu’utilise le modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: c2f6f386f4a8ea062980c0efc97d0cfb4f37f4f2
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92124892"
---
# <a name="tutorial-modify-and-build-the-live-video-analytics-gateway-modules"></a>Tutoriel : Modifier et créer des modules de passerelle d’analytique vidéo en direct

Ce tutoriel vous montre comment modifier le code de module IoT Edge pour les modules d’analytique vidéo en direct.

Les tutoriels précédents utilisent des images prédéfinies des modules.

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes de ce tutoriel, vous devez disposer des éléments suivants :

* [Node.js](https://nodejs.org/en/download/) v10 ou version ultérieure ;
* [Visual Studio Code](https://code.visualstudio.com/Download) avec l’extension [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin) installée ;
* moteur [Docker](https://www.docker.com/products/docker-desktop) ;
* [Azure Container Registry](../../container-registry/index.yml) pour héberger vos versions des modules.
* compte [Azure Media Services](../../media-services/index.yml). Si vous avez suivi les tutoriels précédents, vous pouvez réutiliser celui que vous avez créé précédemment.

## <a name="clone-the-repository"></a>Cloner le référentiel

Si vous n’avez pas encore cloné le référentiel, utilisez la commande suivante pour le cloner dans un emplacement approprié sur votre ordinateur local :

```cmd
git clone https://github.com/Azure/live-video-analytics
```

Ouvrez le dossier du référentiel *live-video-analytics* local avec VS Code.

## <a name="edit-the-deploymentamd64json-file"></a>Modifier le fichier deployment.amd64.json

1. Si ce n’est déjà fait, créez un dossier nommé *ref-apps/lva-edge-iot-central-gateway/storage* dans la copie locale du référentiel **lva-gateway**. Git ignore ce dossier pour vous empêcher d’archiver accidentellement des informations confidentielles.
1. Copiez le fichier *deployment.amd64.json* du dossier *setup* vers le dossier *storage*.
1. Dans VS Code, ouvrez le fichier*storage/deployment.amd64.json*.
1. Modifiez la section `registryCredentials` pour ajouter vos informations d’identification Azure Container Registry.
1. Modifiez la section du module `LvaEdgeGatewayModule` pour ajouter le nom de votre image et le nom de votre compte AMS dans `env:amsAccountName:value`.
1. Modifiez la section du module `lvaYolov3` et ajoutez le nom de votre image.
1. Modifiez la section du module `lvaEdge` et ajoutez le nom de votre image.
1. Pour plus d’informations sur la façon d’effectuer la configuration, consultez [Créer une application d’analytique vidéo dans Azure IoT Central](tutorial-video-analytics-create-app-yolo-v3.md).

## <a name="build-the-code"></a>Générer le code

1. Avant d’essayer de générer le code pour la première fois, utilisez le terminal VS Code pour exécuter la commande `npm install`. Cette commande installe les packages requis et exécute les scripts de configuration.

    > [!TIP]
    > Si vous extrayez une version plus récente du référentiel GitHub, supprimez le dossier *node_modules*, puis réexécutez `npm install`.

1. Modifiez le fichier *./setup/imageConfig.json* pour mettre à jour l’image nommée en fonction du nom de votre registre de conteneurs :

    ```json
    {
        "arch": "amd64",
        "imageName": "[Server].azurecr.io/lva-edge-gateway"
    }
    ```

1. Utilisez le terminal VS Code pour exécuter la commande `docker login [your server].azurecr.io`. Utilisez les informations d’identification que vous avez fournies dans le manifeste de déploiement pour les modules.

1. Utilisez le terminal VS Code pour exécuter la commande **npm version patch**. Ce script de génération déploie les images dans votre registre de conteneurs. La sortie de la fenêtre de terminal VS Code vous indique si la génération a réussi.

1. La version de l’image **LvaEdgeGatewayModule** s’incrémente chaque fois que la génération est terminée. Vous devez utiliser cette version dans le fichier manifeste de déploiement.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous connaissez le modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement et les modules IoT Edge d’analytique vidéo en direct, l’étape suivante suggérée consiste à en apprendre davantage sur ce qui suit :

> [!div class="nextstepaction"]
> [Création de solutions de vente au détail avec Azure IoT Central](overview-iot-central-retail.md)