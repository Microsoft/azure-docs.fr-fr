---
title: Tutoriel - Tutoriel de module Java personnalisé à l’aide d’Azure IoT Edge
description: Ce tutoriel explique comment créer un module IoT Edge avec un code Java et le déployer sur un appareil Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom:
- mvc
- mqtt
- devx-track-java
ms.openlocfilehash: 51b7f6e814a9fad286a934466daeb1ffced225c1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968058"
---
# <a name="tutorial-develop-a-java-iot-edge-module-for-linux-devices"></a>Tutoriel : Développer des modules Java IoT Edge pour les appareils Linux

Vous pouvez utiliser des modules Azure IoT Edge pour déployer un code qui implémente votre logique métier directement sur vos appareils IoT Edge. Ce tutoriel vous guide dans la création et le déploiement d’un module IoT Edge qui filtre des données de capteur. Vous allez utiliser l’appareil IoT Edge simulé que vous avez créé dans le guide de démarrage rapide Déployer Azure IoT Edge sur un appareil simulé dans [Linux](quickstart-linux.md). Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Utilisez Visual Studio Code pour créer un module Java IoT Edge basé sur le package de modèle maven Azure IoT Edge et le kit de développement logiciel d’appareil Java Azure IoT.
> * Utiliser Visual Studio Code et Docker pour créer une image Docker et la publier dans votre registre
> * Déployer le module sur votre appareil IoT Edge
> * Afficher les données générées

Le module IoT Edge que vous créez dans ce didacticiel filtre les données de température générées par votre appareil. Il envoie uniquement des messages en amont lorsque la température dépasse un seuil spécifié. Ce type d’analyse à la périphérie est utile pour réduire la quantité de données communiquées et stockées dans le cloud.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Ce tutoriel montre comment développer un module dans **Java** à l’aide de **Visual Studio Code** et comment le déployer dans un **appareil Linux**. IoT Edge ne prend pas en charge les modules Java pour appareils Windows.

Utilisez le tableau suivant pour comprendre les options dont vous disposez pour développer et déployer des modules Java :

| Java | Visual Studio Code | Visual Studio 2017/2019 |
| - | ------------------ | ------------------ |
| **Linux AMD64** | ![Utilisez VS Code pour modules Java sur Linux AMD64](./media/tutorial-c-module/green-check.png) |  |
| **Linux ARM32** | ![Utilisez VS Code pour modules Java sur Linux AMD32](./media/tutorial-c-module/green-check.png) |  |

Avant de commencer ce tutoriel, vous devez avoir effectué celui qui précède pour configurer votre environnement de développement pour le développement de conteneur Linux : [Développer des modules IoT Edge pour les appareils Linux](tutorial-develop-for-linux.md). Si vous avez suivi l’un ou l’autre de ces tutoriels, la configuration requise doit être la suivante :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Linux exécutant Azure IoT Edge](quickstart-linux.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio Code](https://code.visualstudio.com/) configuré avec [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [Docker CE](https://docs.docker.com/install/) configuré pour exécuter des conteneurs Linux.

Pour développer un module IoT Edge avec Java, installez les configurations requises supplémentaires suivantes sur votre machine de développement : 

* [Pack d’extension Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) pour Visual Studio Code.
* [Java SE Development Kit 10](/azure/developer/java/fundamentals/java-jdk-long-term-support), et [définissez la variable d’environnement`JAVA_HOME`](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)pour pointer vers votre installation JDK.
* [Maven](https://maven.apache.org/)

   >[!TIP]
   >Les processus d’installation de Java et Maven ajoutent des variables d’environnement à votre système. Une fois l’installation terminée, redémarrez tout terminal Visual Studio Code ouvert, PowerShell ou les instances d’invite de commandes. Cette étape garantissent que ces utilitaires pourront reconnaître les commandes Java et Maven.

## <a name="create-a-module-project"></a>Créer un projet de module

Les étapes suivantes créent un projet de module IoT Edge basé sur le package de modèle maven Azure IoT Edge et le SDK d’appareil Java Azure IoT. Vous créez le projet à l’aide de Visual Studio Code et des outils Azure IoT.

### <a name="create-a-new-project"></a>Création d'un projet

Créez un modèle de solution Java que vous pouvez personnaliser avec votre propre code.

1. Dans Visual Studio Code, sélectionnez **Affichage** > **Palette de commandes** pour ouvrir la palette de commandes VS Code.

2. Dans la palette de commandes, entrez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Suivez les invites de la palette de commandes pour créer votre solution.

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution ou acceptez le nom par défaut (**EdgeSolution**). |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Module Java**. |
   | Provide a module name (Nommer le module) | Nommez votre module **JavaModule**. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. L’image conteneur est préremplie avec le nom que vous avez indiqué à la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. <br><br>Le dépôt d’images final ressemble à \<registry name\>.azurecr.io/javamodule. |
   | Provide value for groupId (Fournir une valeur pour l’ID de groupe) | Entrez une valeur pour l’ID de groupe ou acceptez la valeur par défaut (**com.edgemodule**). |

   ![Fourniture du référentiel d’images Docker](./media/tutorial-java-module/repository.png)

Si c’est la première fois que vous créez un module Java, le téléchargement des packages maven peut prendre plusieurs minutes. Quand la solution est prête, la fenêtre VS Code charge l’espace de travail de votre solution IoT Edge. L’espace de travail de la solution contient cinq composants de niveau supérieur :

* Le dossier **modules** contient le code Java de votre module et les fichiers Docker pour générer votre module en tant qu’image conteneur.
* Le fichier **\.env** stocke les informations d’identification de votre registre de conteneurs.
* Le fichier **deployment.template.json** contient les informations utilisées par le runtime IoT Edge pour déployer des modules sur un appareil.
* Les conteneurs de fichiers **deployment.debug.template.json** contiennent la version debug des modules.
* Dans le cadre de ce didacticiel, vous ne modifierez pas le dossier **\.vscode** ni le fichier **\.gitignore**.

Si vous n’avez pas spécifié de registre de conteneurs lors de la création de votre solution, mais accepté la valeur localhost:5000 par défaut, vous n’aurez pas de fichier \.env.

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le fichier d’environnement stocke les informations d’identification de votre registre de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge.

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Dans l’Explorateur VS Code, ouvrez le fichier .env.
2. Mettre à jour les champs avec les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez ce fichier.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Actuellement, Visual Studio Code peut développer des modules Java pour Linux AMD64 et Linux ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car le conteneur est généré et s’exécute différemment pour chaque type d’architecture. Linux AMD64 est la valeur par défaut.

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre.

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**.

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

1. Dans l’Explorateur VS Code, ouvrez **modules** > **JavaModule** > **src** > **main** > **java** > **com** > **edgemodule** > **App.java**.

2. Ajoutez le code suivant au début du fichier pour importer de nouvelles classes référencées.

    ```java
    import java.io.StringReader;
    import java.util.concurrent.atomic.AtomicLong;
    import java.util.HashMap;
    import java.util.Map;

    import javax.json.Json;
    import javax.json.JsonObject;
    import javax.json.JsonReader;

    import com.microsoft.azure.sdk.iot.device.DeviceTwin.Pair;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.Property;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.TwinPropertyCallBack;
    ```

3. Ajoutez la définition suivante dans la classe **App**. Cette variable définit un seuil de température. La température de machine mesurée ne sera pas signalée à IoT Hub tant qu’elle ne dépassera pas cette valeur.

    ```java
    private static final String TEMP_THRESHOLD = "TemperatureThreshold";
    private static AtomicLong tempThreshold = new AtomicLong(25);
    ```

4. Remplacez la méthode d’exécution de **MessageCallbackMqtt** par le code suivant. Cette méthode est appelée chaque fois que le module reçoit un message MQTT d’IoT Edge Hub. Il filtre les messages qui signalent des températures situées sous le seuil de température défini via le double de module.

    ```java
    protected static class MessageCallbackMqtt implements MessageCallback {
        private int counter = 0;
        @Override
        public IotHubMessageResult execute(Message msg, Object context) {
            this.counter += 1;

            String msgString = new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET);
            System.out.println(
                   String.format("Received message %d: %s",
                            this.counter, msgString));
            if (context instanceof ModuleClient) {
                try (JsonReader jsonReader = Json.createReader(new StringReader(msgString))) {
                    final JsonObject msgObject = jsonReader.readObject();
                    double temperature = msgObject.getJsonObject("machine").getJsonNumber("temperature").doubleValue();
                    long threshold = App.tempThreshold.get();
                    if (temperature >= threshold) {
                        ModuleClient client = (ModuleClient) context;
                        System.out.println(
                            String.format("Temperature above threshold %d. Sending message: %s",
                            threshold, msgString));
                        client.sendEventAsync(msg, eventCallback, msg, App.OUTPUT_NAME);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            return IotHubMessageResult.COMPLETE;
        }
    }
    ```

5. Ajoutez les deux classes internes statiques suivantes dans la classe **App**. Ces classes mettent à jour la variable tempThreshold quand la propriété souhaitée du jumeau de module change. Tous les modules ont leur propre module jumeau, ce qui vous permet de configurer le code exécuté à l’intérieur d’un module directement à partir du cloud.

    ```java
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
        @Override
        public void execute(IotHubStatusCode status, Object context) {
            System.out.println("IoT Hub responded to device twin operation with status " + status.name());
        }
    }

    protected static class OnProperty implements TwinPropertyCallBack {
        @Override
        public void TwinPropertyCallBack(Property property, Object context) {
            if (!property.getIsReported()) {
                if (property.getKey().equals(App.TEMP_THRESHOLD)) {
                    try {
                        long threshold = Math.round((double) property.getValue());
                        App.tempThreshold.set(threshold);
                    } catch (Exception e) {
                        System.out.println("Faile to set TemperatureThread with exception");
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    ```

6. Ajoutez les lignes suivantes à la méthode **main** après **client.open()** pour vous abonner aux mises à jour du jumeau de module.

    ```java
    client.startTwin(new DeviceTwinStatusCallBack(), null, new OnProperty(), null);
    Map<Property, Pair<TwinPropertyCallBack, Object>> onDesiredPropertyChange = new HashMap<Property, Pair<TwinPropertyCallBack, Object>>() {
        {
            put(new Property(App.TEMP_THRESHOLD, null), new Pair<TwinPropertyCallBack, Object>(new OnProperty(), null));
        }
    };
    client.subscribeToTwinDesiredProperties(onDesiredPropertyChange);
    client.getTwin();
    ```

7. Enregistrez le fichier App.java.

8. Dans l’Explorateur VS Code, ouvrez le fichier **deployment.template.json** dans votre espace de travail de solution IoT Edge.

9. Ajoutez le module jumeau **JavaModule** au manifeste de déploiement. Insérez le contenu JSON suivant en bas de la section **moduleContent**, après le jumeau de module **$edgeHub** :

   ```json
     "JavaModule": {
         "properties.desired":{
             "TemperatureThreshold":25
         }
     }
   ```

   ![Ajouter un jumeau de module au modèle de déploiement](./media/tutorial-java-module/module-twin.png)

10. Enregistrez le fichier deployment.template.json.

## <a name="build-and-push-your-module"></a>Générer et envoyer (push) votre module

Dans la section précédente, vous avez créé une solution IoT Edge et ajouté du code dans le module **JavaModule** pour filtrer les messages où la température de la machine signalée est inférieure à la limite acceptable. Maintenant, générez la solution en tant qu’image conteneur et envoyez-la à votre registre de conteneurs.

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

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous devez voir le nouveau module **JavaModule** en cours d’exécution avec le module **SimulatedTemperatureSensor** ainsi que **$edgeAgent** et **$edgeHub**.  

    Le démarrage des modules peut prendre plusieurs minutes. Le runtime IoT Edge doit recevoir son nouveau manifeste de déploiement, extraire les images de module à partir du runtime du conteneur, puis démarrer chaque nouveau module.

## <a name="view-the-generated-data"></a>Afficher les données générées

Une fois que vous appliquez le manifeste de déploiement à votre appareil IoT Edge, le runtime IoT Edge sur l’appareil collecte les nouvelles informations de déploiement et commence à s’exécuter sur celui-ci. Tous les modules en cours d’exécution sur l’appareil qui ne sont pas inclus dans le manifeste de déploiement sont arrêtés. Tous les modules manquant de l’appareil sont démarrés.

1. Dans l’explorateur Visual Studio Code, cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

2. Affichez les messages reçus dans votre hub IoT. L’arrivée des messages peut prendre un certain temps. L’appareil IoT Edge doit recevoir son nouveau déploiement et démarrer tous les modules. Ensuite, les modifications que nous avons apportées au code du module Java n’enverront pas de message avant que la température de la machine n’atteigne 25 degrés. Le type de message **Alerte** est également ajouté à chaque message qui atteint ce seuil de température.

## <a name="edit-the-module-twin"></a>Modifier le jumeau de module

Nous avons utilisé le jumeau du module Java dans le manifeste de déploiement pour définir le seuil de température à 25 degrés. Vous pouvez utiliser le jumeau de module pour changer cette fonctionnalité sans devoir mettre à jour le code du module.

1. Dans Visual Studio Code, développez les détails de votre appareil IoT Edge pour afficher les modules Cours d’exécution.

2. Faites un clic droit sur **Module Java** et sélectionnez **Modifier le jumeau de module**.

3. Recherchez **TemperatureThreshold** dans les propriétés souhaitées. Définissez sa valeur sur une nouvelle température supérieure de 5 à 10 degrés à la dernière température indiquée.

4. Enregistrez le jumeau de module.

5. Cliquez avec le bouton droit sur le volet d’édition du jumeau de module et sélectionnez **Mettre à jour le jumeau de module**.

6. Supervisez les messages appareil-à-cloud entrants. Vous devriez voir les messages s’arrêter jusqu’à ce que le nouveau seuil de température soit atteint.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez créées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un module IoT Edge qui filtre les données brutes générées par votre appareil IoT Edge.

Passez aux tutoriels suivants afin de découvrir comment Azure IoT Edge vous aide à déployer des services cloud Azure pour traiter et analyser des données en périphérie.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Service Custom Vision](tutorial-deploy-custom-vision.md)