---
title: Tutoriel - Déployer un classifieur Custom Vision sur un appareil à l’aide d’Azure IoT Edge
description: Dans ce tutoriel, apprenez à faire en sorte qu'un modèle de vision par ordinateur s'exécute en tant que conteneur à l'aide du service Vision personnalisée et d'IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 564f19a1be5b3ce8a5267a07bd4f1f6de80fecf1
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621278"
---
# <a name="tutorial-perform-image-classification-at-the-edge-with-custom-vision-service"></a>Tutoriel : Effectuer la classification d'images en périphérie avec le service Vision personnalisée

Grâce à Azure IoT Edge, votre solution IoT peut gagner en efficacité, via le déplacement des charges de travail en dehors du cloud et vers la périphérie. Cette fonctionnalité s’avère particulièrement utile pour les services qui traitent de nombreuses données, comme les modèles de vision par ordinateur. [Custom Vision](../cognitive-services/custom-vision-service/overview.md) vous permet de créer des classifieurs d’images personnalisés et de les déployer en tant que conteneurs sur des appareils. En fonctionnant de concert, ces deux services vous permettent de rechercher des insights à partir de flux vidéo ou d’images sans avoir à transférer toutes les données hors site au préalable. Custom Vision fournit un classifieur qui compare une image par rapport à un modèle entraîné pour générer des insights.

Par exemple, Custom Vision sur un appareil IoT Edge peut déterminer si le trafic sur une autoroute est plus ou moins dense que la normale, ou si un parking dispose de places libres dans une ligne. Ces insights peuvent être partagés avec un autre service pour entreprendre des actions.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer un classifieur d’images avec Custom Vision.
> * Développer un module IoT Edge qui interroge le serveur web Custom Vision sur votre appareil.
> * Envoyer les résultats du classifieur d’images à IoT Hub.

<center>

![Diagramme - Tutoriel : Structurer, mettre en lots et déployer un classifieur](./media/tutorial-deploy-custom-vision/custom-vision-architecture.png)
</center>

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

>[!TIP]
>Ce tutoriel est une version simplifiée de l’exemple de projet [Custom Vision and Azure IoT Edge on a Raspberry Pi 3](https://github.com/Azure-Samples/Custom-vision-service-iot-edge-raspberry-pi). Ce tutoriel a été conçu pour s’exécuter sur une machine virtuelle du cloud. Il utilise des images statiques pour entraîner et tester le classifieur d’images, ce qui est utile pour une personne qui commence tout juste à évaluer Custom Vision sur IoT Edge. L’exemple de projet utilise du matériel physique et configure un flux de caméra en direct pour entraîner et tester le classifieur d’images, ce qui est utile pour une personne qui souhaite essayer un scénario plus détaillé et plus concret.

Avant de commencer ce tutoriel, vous devez avoir effectué celui qui précède afin de configurer votre environnement pour le développement de conteneur Linux : [Développer des modules IoT Edge pour les appareils Linux](tutorial-develop-for-linux.md). En suivant ce tutoriel, les conditions préalables suivantes doivent être remplies :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Linux exécutant Azure IoT Edge](quickstart-linux.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio Code](https://code.visualstudio.com/) configuré avec [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [Docker CE](https://docs.docker.com/install/) configuré pour exécuter des conteneurs Linux.

Pour développer un module IoT Edge avec le service Custom Vision, installez les conditions préalables supplémentaires suivantes sur votre machine de développement :

* [Python](https://www.python.org/downloads/)
* [Git](https://git-scm.com/downloads)
* [Extension Python pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

## <a name="build-an-image-classifier-with-custom-vision"></a>Créer un classifieur d’images avec Custom Vision

Pour créer un classifieur d’images, vous devez créer un projet Custom Vision et fournir des images d’entraînement. Pour plus d’informations sur la procédure à suivre dans cette section, consultez [Comment créer un classifieur avec Custom Vision](../cognitive-services/custom-vision-service/getting-started-build-a-classifier.md).

Une fois votre classifieur d’images créé et entraîné, vous pouvez l’exporter en tant que conteneur Docker et le déployer vers un appareil IoT Edge.

### <a name="create-a-new-project"></a>Création d'un projet

1. Dans votre navigateur Web, accédez à la [page web Custom Vision](https://customvision.ai/).

2. Sélectionnez **Se connecter** et connectez-vous avec le même compte que celui utilisé pour accéder aux ressources Azure.

3. Sélectionnez **Nouveau projet**.

4. Créez votre projet avec les valeurs suivantes :

   | Champ | Valeur |
   | ----- | ----- |
   | Nom | Donnez un nom à votre projet, par exemple **EdgeTreeClassifier**. |
   | Description | Description facultative du projet. |
   | Ressource | Sélectionnez l’un de vos groupes de ressources Azure contenant une ressource de service Custom Vision ou **créez-en un** si vous n’en avez pas encore ajouté un. |
   | Types de projet | **Classification** |
   | Classification Types (Types de classification) | **Multiclass (single tag per image)** (Multiclasse (une balise par image)) |
   | Domaines | **General (compact)** (Général (compact)) |
   | Fonctionnalités d’exportation | **Plateformes de base (Tensorflow, CoreML, ONNX...)** |

5. Sélectionnez **Créer un projet**.

### <a name="upload-images-and-train-your-classifier"></a>Charger des images et entraîner votre classifieur

La création d’un classifieur d’images nécessite un ensemble d’images d’entraînement, ainsi que des images de test.

1. Clonez ou téléchargez des exemples d’images à partir du référentiel [Cognitive-CustomVision-Windows](https://github.com/Microsoft/Cognitive-CustomVision-Windows) sur votre machine de développement locale.

   ```cmd/sh
   git clone https://github.com/Microsoft/Cognitive-CustomVision-Windows.git
   ```

2. Revenez à votre projet Custom Vision et sélectionnez **Ajouter des images**.

3. Accédez au référentiel git que vous avez cloné en local et accédez au premier dossier d’images, **Cognitive-CustomVision-Windows / Samples / Images / Hemlock**. Sélectionnez les 10 images dans le dossier, puis cliquez sur **Ouvrir**.

4. Ajoutez la balise **hemlock** à ce groupe d’images, et appuyez sur **Entrée** pour appliquer la balise.

5. Sélectionnez **Upload 10 files** (Charger 10 fichiers).

   ![Charger des fichiers avec balise hemlock vers le service Vision personnalisée](./media/tutorial-deploy-custom-vision/upload-hemlock.png)

6. Une fois les images chargées, sélectionnez **Terminé**.

7. Sélectionnez une nouvelle fois **Ajouter des images**.

8. Accédez au deuxième dossier d’images **Cognitive-CustomVision-Windows / Samples / Images / Japanese Cherry**. Sélectionnez les 10 images dans le dossier, puis cliquez sur **Ouvrir**.

9. Ajoutez la balise **japanese cherry** à ce groupe d’images, et appuyez sur **Entrée** pour appliquer la balise.

10. Sélectionnez **Upload 10 files** (Charger 10 fichiers). Une fois les images chargées, sélectionnez **Terminé**.

11. Lorsque les deux ensembles d’images sont balisés et chargés, sélectionnez **Train** (Entraîner) pour former le classifieur.

### <a name="export-your-classifier"></a>Exporter votre classifieur

1. Après l’entraînement de votre classifieur, sélectionnez **Exporter** sur la page Performances du classifieur.

   ![Exporter votre classifieur d'images formé](./media/tutorial-deploy-custom-vision/export.png)

2. Sélectionnez **DockerFile** pour la plateforme. 

3. Sélectionnez **Linux** pour la version.  

4. Sélectionnez **Exporter**. 

   ![Exporter en tant que fichier DockerFile avec des conteneurs Linux](./media/tutorial-deploy-custom-vision/export-2.png)

5. Une fois l’export terminé, sélectionnez **Télécharger** et enregistrez le package .zip en local sur votre ordinateur. Extrayez tous les fichiers du package. Vous allez utiliser ces fichiers pour créer un module IoT Edge qui contient le serveur de classification d’images. 

Une fois cette étape atteinte, vous avez terminé la création et l’entraînement de votre projet Custom Vision. Vous allez utiliser les fichiers exportés dans la section suivante, mais vous en avez terminé avec la page web Custom Vision. 

## <a name="create-an-iot-edge-solution"></a>Créer une solution IoT Edge

Vous disposez maintenant des fichiers pour une version de conteneur de votre classifieur d’images sur votre machine de développement. Dans cette section, vous configurez le conteneur de classifieur d’images pour qu’il s’exécute en tant que module IoT Edge. Vous créez également un deuxième module qui est déployé en même temps que le classifieur d’images. Le deuxième module publie des requêtes sur le classifieur et envoie les résultats sous la forme de messages à IoT Hub. 

### <a name="create-a-new-solution"></a>Créer une solution

Une solution est une méthode logique de développement et d’organisation de plusieurs modules pour un déploiement IoT Edge unique. Une solution contient le code correspondant à un ou plusieurs modules, ainsi que le manifeste de déploiement qui déclare de quelle façon les configurer sur un appareil IoT Edge. 

1. Dans Visual Studio Code, sélectionnez **Affichage** > **Palette de commandes** pour ouvrir la palette de commandes VS Code. 

1. Dans la palette de commandes, entrez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Dans la palette de commandes, spécifiez les informations suivantes pour créer votre solution : 

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution, par exemple **CustomVisionSolution**, ou acceptez le nom par défaut. |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Module Python**. |
   | Provide a module name (Nommer le module) | Nommez votre module **classifier**.<br><br>Il est important que ce nom de module soit en minuscules. En effet, IoT Edge respecte la casse lorsque vous faites référence aux modules, et cette solution utilise une bibliothèque qui met en forme toutes les requêtes en minuscules. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. Votre image conteneur est préremplie à partir de la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure.<br><br>La chaîne finale ressemble à **\<registry name\>.azurecr.io/classifier**. |
 
   ![Fourniture du référentiel d’images Docker](./media/tutorial-deploy-custom-vision/repository.png)

La fenêtre Visual Studio Code charge l’espace de travail de votre solution IoT Edge.

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le fichier d’environnement stocke les informations d’identification de votre registre de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge.

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Dans l’Explorateur VS Code, ouvrez le fichier .env.
2. Mettre à jour les champs avec les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez ce fichier.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Actuellement, Visual Studio Code peut développer des modules pour Linux AMD64 et Linux ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car le conteneur est généré et s’exécute différemment pour chaque type d’architecture. La valeur par défaut est Linux AMD64, et nous l’utiliserons pour ce tutoriel. 

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre. 

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**. 

### <a name="add-your-image-classifier"></a>Ajouter votre classifieur d’images

Le modèle de module Python dans Visual Studio Code contient des exemples de code que vous pouvez exécuter pour tester IoT Edge. Vous n’allez pas utiliser ce code dans ce scénario. Vous allez plutôt suivre la procédure décrite dans cette section pour remplacer l’exemple de code par le conteneur de classifieur d’images que vous avez exporté précédemment. 

1. Dans votre explorateur de fichiers, accédez au package Custom Vision que vous avez téléchargé et extrait. Copiez tout le contenu du package extrait. Vous devez avoir deux dossiers **app** et **azureml**, et deux fichiers **Dockerfile** et **README**. 

2. Dans votre explorateur de fichiers, accédez au répertoire où vous avez demandé à ce que Visual Studio Code crée votre solution IoT Edge. 

3. Ouvrez le dossier du module classifier. Si vous avez utilisé les suggestions de noms dans la section précédente, la structure de dossiers ressemble à **CustomVisionSolution / modules / classifier**. 

4. Collez les fichiers dans le dossier **classifier**. 

5. Revenez à la fenêtre Visual Studio Code. L’espace de travail de votre solution doit désormais afficher les fichiers de classifieur d’images dans le dossier du module. 

   ![Espace de travail de la solution avec des fichiers de classifieur d’images](./media/tutorial-deploy-custom-vision/workspace.png)

6. Ouvrez le fichier **module.json** dans le dossier du classifieur. 

7. Mettez à jour le paramètre **platforms** pour qu’il pointe vers le nouveau fichier Dockerfile que vous avez ajouté, et supprimez toutes les options en plus de AMD64, qui est la seule architecture que nous utilisons pour ce tutoriel. 

   ```json
   "platforms": {
       "amd64": "./Dockerfile"
   }
   ```

8. Enregistrez vos modifications. 

### <a name="create-a-simulated-camera-module"></a>Créer un module de caméra simulée

Dans un déploiement Custom Vision réel, vous auriez une caméra fournissant des flux vidéo ou des images en direct. Pour ce scénario, vous simulez la caméra en créant un module qui envoie une image de test au classifieur d’images. 

#### <a name="add-and-configure-a-new-module"></a>Ajouter et configurer un nouveau module

Dans cette section, vous ajoutez un nouveau module au même dossier CustomVisionSolution et fournissez du code pour créer la caméra simulée. 

1. Dans la même fenêtre de Visual Studio Code, utilisez la palette de commandes pour exécuter **Azure IoT Edge : Ajouter un module IoT Edge**. Dans la palette de commandes, spécifiez les informations suivantes pour votre nouveau module : 

   | Prompt | Valeur | 
   | ------ | ----- |
   | Select deployment template file (Sélectionner le fichier de modèle de déploiement) | Sélectionnez le fichier **deployment.template.json** dans le dossier CustomVisionSolution. |
   | Select module template (Sélectionner un modèle de module) | Sélectionnez **Module Python** |
   | Provide a module name (Nommer le module) | Nommez votre module **cameraCapture** |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Remplacez **localhost:5000** par la valeur **Serveur de connexion** de votre registre de conteneurs Azure.<br><br>La chaîne finale ressemble à **\<registryname\>.azurecr.io/cameracapture**. |

   La fenêtre VS Code charge votre nouveau module dans l’espace de travail de la solution et met à jour le fichier deployment.template.json. Vous devez maintenant voir deux dossiers de module : classifier et cameraCapture. 

2. Ouvrez le fichier **main.py** dans le dossier **modules** / **cameraCapture**. 

3. Remplacez l’intégralité du fichier par le code suivant. Cet exemple de code envoie des requêtes POST au service de traitement d’images s’exécutant dans le module classifier. Nous proposons ce conteneur de modules avec un exemple d’image à utiliser dans les requêtes. Il empaquette ensuite la réponse sous la forme d’un message IoT Hub et l’envoie à une file d’attente de sortie.  

    ```python
    # Copyright (c) Microsoft. All rights reserved.
    # Licensed under the MIT license. See LICENSE file in the project root for
    # full license information.

    import time
    import sys
    import os
    import requests
    import json
    from azure.iot.device import IoTHubModuleClient, Message

    # global counters
    SENT_IMAGES = 0

    # global client
    CLIENT = None

    # Send a message to IoT Hub
    # Route output1 to $upstream in deployment.template.json
    def send_to_hub(strMessage):
        message = Message(bytearray(strMessage, 'utf8'))
        CLIENT.send_message_to_output(message, "output1")
        global SENT_IMAGES
        SENT_IMAGES += 1
        print( "Total images sent: {}".format(SENT_IMAGES) )

    # Send an image to the image classifying server
    # Return the JSON response from the server with the prediction result
    def sendFrameForProcessing(imagePath, imageProcessingEndpoint):
        headers = {'Content-Type': 'application/octet-stream'}

        with open(imagePath, mode="rb") as test_image:
            try:
                response = requests.post(imageProcessingEndpoint, headers = headers, data = test_image)
                print("Response from classification service: (" + str(response.status_code) + ") " + json.dumps(response.json()) + "\n")
            except Exception as e:
                print(e)
                print("No response from classification service")
                return None

        return json.dumps(response.json())

    def main(imagePath, imageProcessingEndpoint):
        try:
            print ( "Simulated camera module for Azure IoT Edge. Press Ctrl-C to exit." )

            try:
                global CLIENT
                CLIENT = IoTHubModuleClient.create_from_edge_environment()
            except Exception as iothub_error:
                print ( "Unexpected error {} from IoTHub".format(iothub_error) )
                return

            print ( "The sample is now sending images for processing and will indefinitely.")

            while True:
                classification = sendFrameForProcessing(imagePath, imageProcessingEndpoint)
                if classification:
                    send_to_hub(classification)
                time.sleep(10)

        except KeyboardInterrupt:
            print ( "IoT Edge module sample stopped" )

    if __name__ == '__main__':
        try:
            # Retrieve the image location and image classifying server endpoint from container environment
            IMAGE_PATH = os.getenv('IMAGE_PATH', "")
            IMAGE_PROCESSING_ENDPOINT = os.getenv('IMAGE_PROCESSING_ENDPOINT', "")
        except ValueError as error:
            print ( error )
            sys.exit(1)

        if ((IMAGE_PATH and IMAGE_PROCESSING_ENDPOINT) != ""):
            main(IMAGE_PATH, IMAGE_PROCESSING_ENDPOINT)
        else: 
            print ( "Error: Image path or image-processing endpoint missing" )
    ```

4. Enregistrez le fichier **main.py**. 

5. Ouvrez le fichier **requirements.txt**. 

6. Ajoutez une nouvelle ligne pour qu’une bibliothèque l’inclue dans le conteneur.

   ```Text
   requests
   ```

7. Enregistrez le fichier **requirements.txt** .


#### <a name="add-a-test-image-to-the-container"></a>Ajouter une image de test au conteneur

Au lieu d’utiliser une caméra réelle pour fournir un flux d’images pour ce scénario, nous allons utiliser une seule image de test. Une image de test est incluse dans le référentiel GitHub que vous avez téléchargé pour les images d’entraînement, plus haut dans ce didacticiel. 

1. Accédez à l’image de test, située dans **Cognitive-CustomVision-Windows** / **Samples** / **Images** / **Test**. 

2. Copiez **test_image.jpg** 

3. Accédez au répertoire de votre solution IoT Edge et collez l’image de test dans le dossier **modules** / **cameraCapture**. L’image doit être dans le même dossier que le fichier main.py que vous avez modifié dans la section précédente. 

4. Dans Visual Studio Code, ouvrez le fichier **Dockerfile.amd64** pour le module cameraCapture.

5. Après la ligne qui établit le répertoire de travail, `WORKDIR /app`, ajoutez la ligne de code suivante :

   ```Dockerfile
   ADD ./test_image.jpg .
   ```

6. Enregistrez le fichier Dockerfile.

### <a name="prepare-a-deployment-manifest"></a>Préparer un manifeste de déploiement

Jusqu’à présent dans ce didacticiel, vous avez entraîné un modèle Custom Vision pour classer des images d’arbres et empaqueté ce modèle sous la forme d’un module IoT Edge. Vous avez ensuite créé un deuxième module qui peut interroger le serveur de classification d’images et renvoyer ses résultats à IoT Hub. Vous êtes maintenant prêt à créer le manifeste de déploiement qui indiquera à un appareil IoT Edge comment démarrer et exécuter ces deux modules ensemble. 

L’extension IoT Edge pour Visual Studio Code fournit un modèle dans chaque solution IoT Edge pour vous aider à créer un manifeste de déploiement. 

1. Ouvrez le fichier **deployment.template.json** dans le dossier de la solution. 

2. Localisez la section **modules**, qui doit contenir trois modules : les deux que vous avez créés (classifier et cameraCapture) et un troisième qui est inclus par défaut, SimulatedTemperatureSensor. 

3. Supprimez le module **SimulatedTemperatureSensor** avec tous ses paramètres. Ce module est inclus pour fournir des exemples de données pour les scénarios de test, mais nous n’en avons pas besoin pour ce déploiement. 

4. Si vous avez nommé le module de classification d’images autrement que **classifier**, vérifiez le nom maintenant et assurez-vous qu’il est écrit en minuscules. Le module cameraCapture appelle le module classifier à l’aide d’une bibliothèque de requêtes qui met en forme toutes les requêtes en minuscules, et IoT Edge respecte la casse. 

5. Mettez à jour le paramètre **createOptions** pour le module cameraCapture avec le code JSON suivant. Ces informations créent des variables d’environnement dans le conteneur de modules, qui sont récupérées dans le processus main.py. En incluant ces informations dans le manifeste de déploiement, vous pouvez modifier l’image ou le point de terminaison sans avoir à recréer l’image de module. 

    ```json
    "createOptions": "{\"Env\":[\"IMAGE_PATH=test_image.jpg\",\"IMAGE_PROCESSING_ENDPOINT=http://classifier/image\"]}"
    ```

    Si vous avez nommé votre module Custom Vision autrement que *classifier*, mettez à jour la valeur de point de terminaison de traitement d’images pour que les valeurs correspondent. 

6. En bas du fichier, mettez à jour le paramètre **itinéraires** du module $edgeHub. Vous souhaitez acheminer les résultats de prédiction de cameraCapture vers IoT Hub.

    ```json
        "routes": {
          "cameraCaptureToIoTHub": "FROM /messages/modules/cameraCapture/outputs/* INTO $upstream"
        },
    ```

    Si vous avez nommé votre deuxième module autrement que *cameraCapture*, mettez à jour la valeur d’itinéraire pour que les valeurs correspondent. 

7. Enregistrez le fichier **deployment.template.json**.

## <a name="build-and-push-your-iot-edge-solution"></a>Créer et envoyer (push) votre solution IoT Edge

Avec les modules créés et le modèle de manifeste de déploiement configuré, vous êtes prêt à créer les images conteneurs et à les envoyer (push) à votre registre de conteneurs.

Une fois les images dans votre registre, vous pouvez déployer la solution sur un appareil IoT Edge. Vous pouvez définir des modules sur un appareil via le hub IoT, mais vous pouvez également accéder à vos hub IoT et appareils via Visual Studio Code. Dans cette section, vous configurez l’accès à votre hub IoT, puis utilisez VS Code pour déployer la solution sur votre appareil IoT Edge.

Tout d’abord, créez et envoyez (push) votre solution à votre registre de conteneurs.

1. Ouvrez le terminal intégré VS Code en sélectionnant **Affichage** > **Terminal**.

2. Connectez-vous à Docker en entrant la commande suivante dans le terminal. Connectez-vous avec le nom d’utilisateur, le mot de passe et le serveur de connexion de votre registre de conteneurs Azure. Vous pouvez récupérer ces valeurs dans la section **Clés d’accès** de votre registre dans le portail Azure.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Il se peut que vous receviez un avertissement de sécurité recommandant d’utiliser `--password-stdin`. Bien qu’il s’agisse de la bonne pratique recommandée pour les scénarios de production, elle n’est pas pertinente pour ce tutoriel. Pour plus d’informations, consultez les informations de référence sur [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. Dans l’Explorateur VS Code, cliquez avec le bouton droit sur le fichier **deployment.template.json** et sélectionnez **Build and Push IoT Edge solution** (Générer et envoyer (push) la solution IoT Edge).

   La commande de génération et d’envoi (push) déclenche trois opérations. Tout d’abord, elle crée un dossier dans la solution appelé **config** contenant le manifeste de déploiement en entier. Il est généré à partir des informations dans le modèle de déploiement et d’autres fichiers de solution. Ensuite, elle exécute `docker build` pour générer l’image de conteneur basée sur le fichier docker correspondant à votre architecture cible. Puis, elle exécute `docker push` pour envoyer (push) le dépôt d’images vers votre registre de conteneurs.

   Ce processus peut prendre plusieurs minutes la première fois, mais il est plus rapide la prochaine fois que vous exécutez les commandes.

## <a name="deploy-modules-to-device"></a>Déployer des modules sur un appareil

Utilisez l’Explorateur de Visual Studio Code et l’extension Azure IoT Tools pour déployer le projet de module sur votre appareil IoT Edge. Vous disposez déjà d’un manifeste de déploiement préparé pour votre scénario, à savoir le fichier **deployment.amd64.json** figurant dans le dossier config. Il vous suffit alors de sélectionner l’appareil qui recevra le déploiement.

Vérifiez que votre appareil IoT Edge est opérationnel.

1. Dans l’Explorateur Visual Studio Code, sous la section **Azure IoT Hub**, développez **Appareils** pour voir votre liste d’appareils IoT.

2. Cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Create Deployment for Single Device** (Créer un déploiement pour un seul appareil).

3. Sélectionnez le fichier **deployment.amd64.json** dans le dossier **config**, puis cliquez sur **Sélectionner un manifeste de déploiement Edge**. N’utilisez pas le fichier deployment.template.json.

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous voyez normalement les nouveaux modules **classifier** et **cameraCapture** en cours d’exécution avec **$edgeAgent** et **$edgeHub**.  

Vous pouvez également vérifier que tous les modules sont opérationnels sur votre appareil. Sur votre appareil IoT Edge, exécutez la commande suivante pour afficher l’état des modules.

   ```bash
   iotedge list
   ```

Le démarrage des modules peut prendre plusieurs minutes. Le runtime IoT Edge doit recevoir son nouveau manifeste de déploiement, extraire les images de module à partir du runtime du conteneur, puis démarrer chaque nouveau module.

## <a name="view-classification-results"></a>Afficher les résultats de la classification

Il existe deux façons d’afficher les résultats de vos modules, soit sur l’appareil lui-même avec la création et l’envoi de messages, soit depuis Visual Studio Code avec la réception des messages sur l’IoT Hub. 

Sur votre appareil, affichez les journaux d’activité du module cameraCapture pour voir les messages envoyés et leur confirmation de réception par IoT Hub. 

   ```bash
   iotedge logs cameraCapture
   ```

Dans Visual Studio Code, cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**. 

> [!NOTE]
> Au début, vous pouvez constater des erreurs de connexion dans la sortie du module cameraCapture. Elles sont dues au délai entre le déploiement et le démarrage des modules.
>
> Le module cameraCapture tente automatiquement de se reconnecter jusqu'à ce qu'il y parvienne. Une fois la connexion établie, vous verrez les messages de classification d'image attendus décrits ci-dessous.

Les résultats du module Custom Vision, qui sont envoyés sous la forme de messages depuis le module cameraCapture, incluent la probabilité que l’image représente une cigüe ou un cerisier. Dans la mesure où l’image représente une cigüe, la probabilité doit être égale à 1,0.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test. 

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez utilisées dans cet article pour éviter les frais. 

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez entraîné un modèle Custom Vision et l’avez déployé en tant que module sur un appareil IoT Edge. Vous avez ensuite créé un module qui peut interroger le service de classification d’images et renvoyer ses résultats à IoT Hub.

Passez aux tutoriels suivants pour en savoir plus sur les autres façons dont Azure IoT Edge peut vous aider à transformer des données en informations métier « at the edge » (« en périphérie»).

> [!div class="nextstepaction"]
> [Stocker des données en périphérie avec les bases de données SQL Server](tutorial-store-data-sql-server.md)