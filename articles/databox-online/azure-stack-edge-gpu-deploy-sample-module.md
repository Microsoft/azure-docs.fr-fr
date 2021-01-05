---
title: Déployer un module GPU sur votre appareil Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment activer le calcul et y préparer votre appareil Azure Stack Edge Pro par le biais de l’interface utilisateur locale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/31/2020
ms.author: alkohli
ms.openlocfilehash: 1f16ef0ede25f17acb915a7812ae5b15b45f78a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90899721"
---
# <a name="deploy-a-gpu-enabled-iot-module-on-azure-stack-edge-pro-gpu-device"></a>Déployer un module IoT compatible GPU sur un appareil Azure Stack Edge Pro avec GPU

Cet article explique comment déployer un module IoT Edge compatible GPU sur votre appareil Azure Stack Edge Pro avec GPU. 

Dans cet article, vous apprendrez comment :
  - Préparer Azure Stack Edge Pro pour exécuter un module GPU.
  - Télécharger et installer un exemple de code à partir d’un dépôt Git.
  - Générer la solution et un manifeste de déploiement.
  - Déployer la solution sur un appareil Azure Stack Edge Pro.
  - surveiller la sortie du module.


## <a name="about-sample-module"></a>À propos de l’exemple de module

L’exemple de module GPU mentionné dans cet article comprend un exemple de code d’évaluation PyTorch et TensorFlow pour comparer l’UC au GPU.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que vous disposez  des éléments suivants :

- Vous avez accès à un appareil Azure Stack Edge Pro à 1 nœud compatible avec GPU. Cet appareil est activé avec une ressource dans Azure. Consultez [Activer l’appareil](azure-stack-edge-gpu-deploy-activate.md).
- Vous avez configuré le calcul sur cet appareil. Suivez les étapes décrites dans [Tutoriel : Configurer le calcul sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md).
- Un ACR (Azure Container Registry). Accédez au panneau **Clés d’accès** et notez le serveur de connexion, le nom d’utilisateur et le mot de passe de l’ACR. Pour plus d’informations, accédez à [Démarrage rapide : Créer un registre de conteneurs privé à l’aide du portail Azure](../container-registry/container-registry-get-started-portal.md#create-a-container-registry).
- Les ressources de développement suivantes sur un client Windows :
    - [Azure CLI 2.0 ou version ultérieure](https://aka.ms/installazurecliwindows)
    - [Docker CE](https://store.docker.com/editions/community/docker-ce-desktop-windows). Vous devrez peut-être créer un compte pour télécharger et installer le logiciel.
    - [Visual Studio Code](https://code.visualstudio.com/)  
    - [Extension Azure IoT Edge pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).    
    - [Extension Python pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)    
    - [Python 3](https://www.python.org/)    
    - PIP pour l’installation des packages Python (en général inclus avec votre installation de Python)

## <a name="get-the-sample-code"></a>Obtenir l’exemple de code

1. Accédez au [Azure intelligent Edge Patterns dans les exemples Azure](https://github.com/azure-samples/azure-intelligent-edge-patterns). Clonez ou téléchargez le fichier zip pour le code. 

    ![Télécharger le fichier ZIP](media/azure-stack-edge-gpu-deploy-sample-module/download-zip-file-1.png)

    Extrayez les fichiers du zip. Vous pouvez également cloner les exemples.

    ```json
    git clone https://github.com/Azure-Samples/azure-intelligent-edge-patterns.git
    ```

## <a name="build-and-deploy-module"></a>Créer et déployer un module

1. Ouvrez le dossier **GpuReferenceModules** dans Visual Studio Code.

    ![Ouvrir GPUReferenceModules dans VS Code](media/azure-stack-edge-gpu-deploy-sample-module/open-folder-gpu-sample-1.png)

2. Ouvrez le fichier *deployment.template.json* et identifiez les paramètres qu’il référence pour le registre de conteneurs. Dans le fichier suivant, les paramètres CONTAINER_REGISTRY_USERNAME, CONTAINER_REGISTRY_PASSWORD et CONTAINER_REGISTRY_NAME sont utilisés.

    ```json
        {
      "$schema-template": "2.0.0",
      "modulesContent": {
        "$edgeAgent": {
          "properties.desired": {
            "schemaVersion": "1.0",
            "runtime": {
              "type": "docker",
              "settings": {
                "minDockerVersion": "v1.25",
                "loggingOptions": "",
                "registryCredentials": {
                  "${CONTAINER_REGISTRY_NAME}":{
                  "username": "$CONTAINER_REGISTRY_USERNAME",
                  "password": "$CONTAINER_REGISTRY_PASSWORD",
                  "address": "${CONTAINER_REGISTRY_NAME}.azurecr.io"
                  }
                }
              }
            },
    ```
3. Créez un fichier. Renseignez les valeurs de vos paramètres de registre de conteneurs (utilisez celles identifiées à l’étape précédente) comme suit : 

    ```json
    CONTAINER_REGISTRY_NAME=<YourContainerRegistryName>
    CONTAINER_REGISTRY_USERNAME=<YourContainerRegistryUserName>
    CONTAINER_REGISTRY_PASSWORD=<YourContainerRegistryPassword>
    ```
    Un exemple de fichier *.env* figure ci-dessous :
    
    ![Créer et enregistrer le fichier. env](media/azure-stack-edge-gpu-deploy-sample-module/create-save-env-file-1.png)

4. Enregistrez le fichier au format *.env* dans le dossier **SampleSolution**.

5. Pour vous connecter à Docker, entrez la commande suivante dans le terminal intégré Visual Studio Code. 

    ```json
    docker login -u <CONTAINER_REGISTRY_USERNAME> -p <CONTAINER_REGISTRY_PASSWORD> <CONTAINER_REGISTRY_NAME>
    ```
    Accédez à la section **Clés d’accès** de votre registre de conteneurs dans le portail Azure. Copiez et utilisez le nom, le mot de passe et le serveur de connexion du registre.

    ![Clés d’accès dans votre registre de conteneurs](media/azure-stack-edge-gpu-deploy-sample-module/container-registry-access-keys-1.png)

    Une fois les informations d’identification fournies, la connexion s’effectue correctement.

    ![Connexion réussie](media/azure-stack-edge-gpu-deploy-sample-module/successful-sign-in-1.png)

6. Envoyez (push) votre image à votre registre de conteneurs Azure. Dans l’Explorateur VS Code, sélectionnez le fichier **deployment.template.json** et cliquez dessus avec le bouton droit, puis sélectionnez **Générer et envoyer (push) la solution IoT Edge**. 

    ![Générer et envoyer (push) la solution IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/build-push-iot-edge-solution-1.png)   

    Si les extensions Python et Python ne sont pas installées, elles sont installées lorsque vous générez et envoyez (push) la solution. Toutefois, cela entraîne des temps de génération plus longs. 

    Une fois cette étape terminée, vous voyez le module dans votre registre de conteneurs.

    ![Module dans le registre de conteneurs](media/azure-stack-edge-gpu-deploy-sample-module/module-container-registry-1.png)    


7. Pour créer un manifeste de déploiement, cliquez avec le bouton droit sur le fichier **deployment.template.json**, puis sélectionnez **Générer un manifeste de déploiement IoT Edge**. 

    ![Générer un manifeste de déploiement IoT Edge](media/azure-stack-edge-gpu-deploy-sample-module/generate-iot-edge-deployment-manifest-1.png)  

    La notification vous informe du chemin d’accès dans lequel le manifeste de déploiement a été généré. Le manifeste est le fichier `deployment.amd64.json` généré dans le dossier **config** . 

8. Sélectionnez le fichier **Deployment. amd64. JSON** dans le dossier **config**, puis choisissez **Créer un déploiement pour un seul appareil**. N’utilisez pas le **fichier deployment.template.json**. 

    ![Créer un déploiement pour un seul appareil](media/azure-stack-edge-gpu-deploy-sample-module/create-deployment-single-device-1.png)  

    Dans la fenêtre **Sortie**, vous devriez voir un message indiquant que le déploiement a réussi.

    ![Déploiement réussi dans la sortie](media/azure-stack-edge-gpu-deploy-sample-module/deployment-succeeded-output-1.png) 

## <a name="monitor-the-module"></a>Surveiller le module  

1. Dans la palette de commandes VS Code, exécutez **Azure IoT Hub: Select IoT Hub** (Azure IoT Hub : Sélectionner l’IoT Hub).

2. Choisissez l’abonnement et l’IoT Hub qui contiennent l’appareil IoT Edge que vous souhaitez configurer. Dans ce cas, sélectionnez l’abonnement utilisé pour déployer l’appareil Azure Stack Edge Pro, puis sélectionnez l’appareil IoT Edge créé pour votre appareil Azure Stack Edge Pro. Cela se produit lorsque vous avez configuré calcul via le portail Azure dans les étapes précédentes.

3. Dans l’Explorateur VS Code, développez la section Appareils Azure IoT Hub. Sous **Appareils**, vous devez voir l’appareil IoT Edge correspondant à votre appareil Azure Stack Edge Pro. 

    1. Sélectionnez cet appareil, cliquez dessus avec le bouton droit, puis sélectionnez **Démarrer la supervision du point de terminaison d’événement intégré**.
  
        ![Démarrer la surveillance](media/azure-stack-edge-gpu-deploy-sample-module/monitor-builtin-event-endpoint-1.png)  

    2. Accédez à **Appareils > Modules**. Vous devriez voir votre **module GPU** en cours d’exécution.

        ![Module dans IoT Hub](media/azure-stack-edge-gpu-deploy-sample-module/module-iot-hub-1.png)  

    3. Le terminal VS Code doit également afficher les événements IoT Hub en tant que sortie de supervision pour votre appareil Azure Stack Edge Pro.

        ![Sortie de surveillance](media/azure-stack-edge-gpu-deploy-sample-module/monitor-events-output-1.png) 

        Vous pouvez voir que le temps nécessaire pour exécuter le même ensemble d’opérations (5 000 itérations de transformation de forme) par GPU est sensiblement inférieur dont a besoin une UC.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la façon de [configurer le GPU pour utiliser un module](azure-stack-edge-j-series-configure-gpu-modules.md).
