---
title: 'Tutoriel : Développer un module Node.js pour Linus - Azure IoT Edge | Microsoft Docs'
description: Ce didacticiel explique comment créer un module IoT Edge avec du code Node.js et le déployer sur un appareil Edge
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc, devx-track-python, devx-track-js
ms.openlocfilehash: 5c8918995675cae8e70ca9fc1efb0cf4c7cb233b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94959405"
---
# <a name="tutorial-develop-and-deploy-a-nodejs-iot-edge-module-for-linux-devices"></a>Tutoriel : Développer et déployer un module IoT Edge Node.js pour des appareils Linux

Utilisez Visual Studio Code pour écrire du code Node.js et le déployer sur un appareil Linux exécutant Azure IoT Edge.

Vous pouvez utiliser des modules IoT Edge pour déployer du code qui implémente votre logique métier directement sur vos appareils IoT Edge. Ce tutoriel vous guide dans la création et le déploiement d’un module IoT Edge qui filtre des données de capteur. Vous utiliserez l’appareil IoT Edge que vous avez créé dans les guides de démarrage rapide. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Utiliser Visual Studio Code pour créer un module IoT Edge Node.js
> * Utiliser le Visual Studio Code et Docker pour créer une image docker et la publier dans votre registre
> * Déployer le module sur votre appareil IoT Edge
> * Afficher les données générées

Le module IoT Edge que vous créez dans ce tutoriel filtre les données de température générées par votre appareil. Il envoie uniquement des messages en amont lorsque la température dépasse un seuil spécifié. Ce type d’analyse à la périphérie est utile pour réduire la quantité de données communiquées et stockées dans le cloud.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Ce tutoriel montre comment développer un module dans **Node.js** à l’aide de **Visual Studio Code** et comment le déployer dans un **appareil Linux**. IoT Edge ne prend pas en charge les modules Node.js pour appareils Windows.

Utilisez le tableau suivant pour comprendre les options dont vous disposez pour développer et déployer des modules Node.js :

| Node.js | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Utilisez VS Code pour modules Node.js sur Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Utilisez VS Code pour modules Node.js sur Linux AMD32](./media/tutorial-c-module/green-check.png) |  |

Avant de commencer ce tutoriel, vous devez avoir effectué celui qui précède pour configurer votre environnement de développement pour le développement de conteneur Linux : [Développer des modules IoT Edge pour les appareils Linux](tutorial-develop-for-linux.md). Si vous avez suivi l’un ou l’autre de ces tutoriels, la configuration requise doit être la suivante :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Linux exécutant Azure IoT Edge](quickstart-linux.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio Code](https://code.visualstudio.com/) configuré avec [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [Docker CE](https://docs.docker.com/install/) configuré pour exécuter des conteneurs Linux.

Pour développer un module IoT Edge avec Node.js, installez les configurations requises supplémentaires suivantes sur votre machine de développement :

* [Node.js et npm](https://nodejs.org). Le paquet npm est distribué avec Node.js, ce qui signifie que lorsque vous téléchargez Node.js, npm est automatiquement installé sur votre ordinateur.

## <a name="create-a-module-project"></a>Créer un projet de module

Les étapes suivantes vous montrent comment créer un module IoT Edge Node.js à l’aide de Visual Studio Code et des outils Azure IoT.

### <a name="create-a-new-project"></a>Création d'un projet

Utilisez **npm** pour créer un modèle de solution Node.js sur lequel vous pouvez compiler.

1. Dans Visual Studio Code, sélectionnez **Affichage** > **Terminal intégré** pour ouvrir le terminal intégré Visual Studio Code.

2. Dans le terminal intégré, entrez la commande suivante pour installer **yeoman** et le générateur du module Azure IoT Edge Node.js :

    ```cmd/sh
    npm install -g yo generator-azure-iot-edge-module
    ```

3. Sélectionnez **Affichage** > **Palette de commandes** pour ouvrir la palette de commandes VS Code.

4. Dans la palette de commandes, tapez et exécutez la commande **Azure: Sign in** et suivez les instructions pour vous connecter à votre compte Azure. Si vous êtes déjà connecté, vous pouvez ignorer cette étape.

5. Dans la palette de commandes, saisissez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Suivez les invites de la palette de commandes pour créer votre solution.

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution ou acceptez le nom par défaut (**EdgeSolution**). |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Module Node.js**. |
   | Provide a module name (Nommer le module) | Nommez votre module **NodeModule**. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. L’image conteneur est préremplie avec le nom que vous avez indiqué à la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. <br><br>Le dépôt d’images final ressemble à \<registry name\>.azurecr.io/nodemodule. |

   ![Fourniture du référentiel d’images Docker](./media/tutorial-node-module/repository.png)

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le fichier d’environnement stocke les informations d’identification de votre référentiel de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge.

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Dans l’Explorateur VS Code, ouvrez le fichier **.env**.
2. Mettre à jour les champs avec les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez ce fichier.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Actuellement, Visual Studio Code peut développer des modules Node.js pour Linux AMD64 et Linux ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car le conteneur est généré et s’exécute différemment pour chaque type d’architecture. Linux AMD64 est la valeur par défaut.

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre.

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**.

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

Chaque modèle est fourni avec un exemple de code, qui utilise des simulations de données de capteur du module **SimulatedTemperatureSensor** et les achemine vers IoT Hub. Dans cette section, ajoutez du code pour que NodeModule analyse les messages avant leur envoi.

1. Dans l’Explorateur VS Code, ouvrez **modules** > **NodeModule** > **app.js**.

2. Ajoutez une variable de seuil de température sous les modules de nœud requis. Le seuil de température définit la valeur que la température mesurée doit dépasser pour que les données soient envoyées à IoT Hub.

    ```javascript
    var temperatureThreshold = 25;
    ```

3. Remplacez l’intégralité de la fonction `PipeMessage` par la fonction `FilterMessage`.

    ```javascript
    // This function filters out messages that report temperatures below the temperature threshold.
    // It also adds the MessageType property to the message with the value set to Alert.
    function filterMessage(client, inputName, msg) {
        client.complete(msg, printResultFor('Receiving message'));
        if (inputName === 'input1') {
            var message = msg.getBytes().toString('utf8');
            var messageBody = JSON.parse(message);
            if (messageBody && messageBody.machine && messageBody.machine.temperature && messageBody.machine.temperature > temperatureThreshold) {
                console.log(`Machine temperature ${messageBody.machine.temperature} exceeds threshold ${temperatureThreshold}`);
                var outputMsg = new Message(message);
                outputMsg.properties.add('MessageType', 'Alert');
                client.sendOutputEvent('output1', outputMsg, printResultFor('Sending received message'));
            }
        }
    }

    ```

4. Remplacez le nom de fonction `pipeMessage` avec `filterMessage` dans la fonction `client.on()`.

    ```javascript
    client.on('inputMessage', function (inputName, msg) {
        filterMessage(client, inputName, msg);
        });
    ```

5. Copiez l’extrait de code suivant dans le fonction de rappel `client.open()`, une fois `client.on()` à l’intérieur de l’instruction `else`. Cette fonction sera appelée après la mise à jour des propriétés souhaitées.

    ```javascript
    client.getTwin(function (err, twin) {
        if (err) {
            console.error('Error getting twin: ' + err.message);
        } else {
            twin.on('properties.desired', function(delta) {
                if (delta.TemperatureThreshold) {
                    temperatureThreshold = delta.TemperatureThreshold;
                }
            });
        }
    });
    ```

6. Enregistrez le fichier app.js.

7. Dans l’Explorateur VS Code, ouvrez le fichier **deployment.template.json** dans votre espace de travail de solution IoT Edge.

8. Ajoutez le jumeau de module NodeModule au manifeste de déploiement. Insérez le contenu JSON suivant en bas de la section `moduleContent`, après le jumeau de module `$edgeHub` :

   ```json
     "NodeModule": {
         "properties.desired":{
             "TemperatureThreshold":25
         }
     }
   ```

   ![Ajouter un jumeau de module au modèle de déploiement](./media/tutorial-node-module/module-twin.png)

9. Enregistrez le fichier deployment.template.json.

## <a name="build-and-push-your-module"></a>Générer et envoyer (push) votre module

Dans la section précédente, vous avez créé une solution IoT Edge et ajouté du code au module Node pour filtrer les messages quand la température de machine signalée se situe dans les limites acceptables. Vous devez maintenant générer la solution comme image de conteneur et l’envoyer à votre registre de conteneurs.

1. Ouvrez le terminal intégré VS Code en sélectionnant **Affichage** > **Terminal**.

2. Connectez-vous à Docker en entrant la commande suivante dans le terminal. Connectez-vous avec le nom d’utilisateur, le mot de passe et le serveur de connexion de votre registre de conteneurs Azure. Vous pouvez récupérer ces valeurs dans la section **Clés d’accès** de votre registre dans le portail Azure.

   ```bash
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Il se peut que vous receviez un avertissement de sécurité recommandant d’utiliser `--password-stdin`. Bien qu’il s’agisse de la bonne pratique recommandée pour les scénarios de production, elle n’est pas pertinente pour ce tutoriel. Pour plus d’informations, consultez les informations de référence sur [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

3. Dans l’Explorateur VS Code, cliquez avec le bouton droit sur le fichier **deployment.template.json** et sélectionnez **Build and Push IoT Edge solution** (Générer et envoyer (push) la solution IoT Edge).

   La commande de génération et d’envoi (push) déclenche trois opérations. Tout d’abord, elle crée un dossier dans la solution appelé **config** contenant les manifestes de déploiement en entier. Il est généré à partir des informations dans le modèle de déploiement et d’autres fichiers de solution. Ensuite, elle exécute `docker build` pour générer l’image de conteneur basée sur le fichier docker correspondant à votre architecture cible. Puis, elle exécute `docker push` pour envoyer (push) le dépôt d’images vers votre registre de conteneurs.

   Ce processus peut prendre plusieurs minutes la première fois, mais il est plus rapide la prochaine fois que vous exécutez les commandes.

## <a name="deploy-modules-to-device"></a>Déployer des modules sur un appareil

Utilisez l’Explorateur de Visual Studio Code et l’extension Azure IoT Tools pour déployer le projet de module sur votre appareil IoT Edge. Vous disposez déjà d’un manifeste de déploiement préparé pour votre scénario, à savoir le fichier **deployment.amd64.json** figurant dans le dossier config. Il vous suffit alors de sélectionner l’appareil qui recevra le déploiement.

Vérifiez que votre appareil IoT Edge est opérationnel.

1. Dans l’Explorateur Visual Studio Code, sous la section **Azure IoT Hub**, développez **Appareils** pour voir votre liste d’appareils IoT.

2. Cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Create Deployment for Single Device** (Créer un déploiement pour un seul appareil).

3. Sélectionnez le fichier **deployment.amd64.json** dans le dossier **config**, puis cliquez sur **Sélectionner un manifeste de déploiement Edge**. N’utilisez pas le fichier deployment.template.json.

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous devez voir le nouveau module **NodeModule** en cours d’exécution avec le module **SimulatedTemperatureSensor** ainsi que **$edgeAgent** et **$edgeHub**.

    Le démarrage des modules peut prendre plusieurs minutes. Le runtime IoT Edge doit recevoir son nouveau manifeste de déploiement, extraire les images de module à partir du runtime du conteneur, puis démarrer chaque nouveau module.

## <a name="view-the-generated-data"></a>Afficher les données générées

Une fois que vous appliquez le manifeste de déploiement à votre appareil IoT Edge, le runtime IoT Edge sur l’appareil collecte les nouvelles informations de déploiement et commence à s’exécuter sur celui-ci. Tous les modules en cours d’exécution sur l’appareil qui ne sont pas inclus dans le manifeste de déploiement sont arrêtés. Tous les modules manquant de l’appareil sont démarrés.

Vous pouvez afficher l’état de votre appareil IoT Edge dans la section **Appareils Azure IoT Hub** de l’explorateur de Visual Studio Code. Développez les détails de votre appareil pour afficher la liste des modules déployés et en cours d’exécution.

1. Dans l’explorateur Visual Studio Code, cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

2. Affichez les messages reçus dans votre hub IoT. L’arrivée des messages peut prendre un certain temps. L’appareil IoT Edge doit recevoir son nouveau déploiement et démarrer tous les modules. Ensuite, les modifications que nous avons apportées au code du module Node n’enverront pas de message avant que la température de la machine n’atteigne 25 degrés. Le type de message **Alerte** est également ajouté à chaque message qui atteint ce seuil de température.

## <a name="edit-the-module-twin"></a>Modifier le jumeau de module

Nous avons utilisé le jumeau du module Node dans le manifeste de déploiement pour définir le seuil de température à 25 degrés. Vous pouvez utiliser le jumeau de module pour changer cette fonctionnalité sans devoir mettre à jour le code du module.

1. Dans Visual Studio Code, développez les détails de votre appareil IoT Edge pour afficher les modules Cours d’exécution.

2. Faites un clic droit sur **Module Node** et sélectionnez **Modifier le jumeau de module**.

3. Recherchez **TemperatureThreshold** dans les propriétés souhaitées. Définissez sa valeur sur une nouvelle température supérieure de 5 à 10 degrés à la dernière température indiquée.

4. Enregistrez le jumeau de module.

5. Cliquez avec le bouton droit sur le volet d’édition du jumeau de module et sélectionnez **Mettre à jour le jumeau de module**.

6. Supervisez les messages appareil-à-cloud entrants. Vous devriez voir les messages s’arrêter jusqu’à ce que le nouveau seuil de température soit atteint.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez créées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un module IoT Edge qui contient le code pour filtrer les données brutes générées par votre appareil IoT Edge.

Vous pouvez passer aux tutoriels suivants afin de découvrir comment Azure IoT Edge peut vous aider à déployer des services cloud Azure pour traiter et analyser des données en périphérie.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Service Custom Vision](tutorial-deploy-custom-vision.md)