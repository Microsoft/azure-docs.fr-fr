---
title: 'Didacticiel : Développer un module C pour Linux – Azure IoT Edge | Microsoft Docs'
description: Ce didacticiel explique comment créer un module IoT Edge avec du code C, et le déployer sur un appareil Linux exécutant IoT Edge
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/30/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 470f82026cc27431555336570ef6f41063442c1e
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964539"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-linux-devices"></a>Tutoriel : Développer un module IoT Edge en C pour appareils Linux

Utilisez Visual Studio Code pour écrire du code C et le déployer sur un appareil Linux exécutant Azure IoT Edge.

Vous pouvez utiliser des modules IoT Edge pour déployer du code qui implémente votre logique métier directement sur vos appareils IoT Edge. Ce tutoriel vous guide dans la création et le déploiement d’un module IoT Edge qui filtre des données de capteur. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Utiliser Visual Studio Code pour créer un module IoT Edge en C
> * Utiliser Visual Studio Code et Docker pour créer une image Docker et la publier dans un registre de conteneurs
> * Déployer le module sur votre appareil IoT Edge
> * Afficher les données générées

Le module IoT Edge que vous créez dans ce tutoriel filtre les données de température générées par votre appareil. Il envoie uniquement des messages en amont lorsque la température dépasse un seuil spécifié. Ce type d’analyse à la périphérie est utile pour réduire la quantité de données communiquées et stockées dans le cloud.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Ce didacticiel montre comment développer un module en **C** à l’aide de **Visual Studio Code**, et comment le déployer sur un **appareil Linux**. Si vous développez des modules pour appareils Windows, voir [Développer un module IoT Edge en C pour appareils Windows](tutorial-c-module-windows.md).

Le tableau suivant présente les options dont vous disposez pour développer et déployer des modules C pour Linux :

| C | Visual Studio Code | Visual Studio |
| - | ------------------ | ------------- |
| **Linux AMD64** | ![Utiliser VS Code pour les modules C sur Linux AMD64](./media/tutorial-c-module/green-check.png) | ![Utiliser Visual Studio pour les modules C sur Linux AMD64](./media/tutorial-c-module/green-check.png) |
| **Linux ARM32** | ![Utiliser VS Code pour les modules C sur Linux AMD32](./media/tutorial-c-module/green-check.png) | ![Utiliser Visual Studio pour les modules C sur Linux ARM32](./media/tutorial-c-module/green-check.png) |

Avant de commencer ce tutoriel, vous devez avoir effectué celui qui précède pour configurer votre environnement de développement pour le développement de conteneur Linux : [Développer des modules IoT Edge pour les appareils Linux](tutorial-develop-for-linux.md). En suivant ce tutoriel, les conditions préalables suivantes doivent être remplies :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Linux exécutant Azure IoT Edge](quickstart-linux.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio Code](https://code.visualstudio.com/) configuré avec [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* [Docker CE](https://docs.docker.com/install/) configuré pour exécuter des conteneurs Linux.

Pour développer un module IoT Edge en C, installez les configurations requises supplémentaires suivantes sur votre machine de développement :

* [Extension C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) pour Visual Studio Code.

L’installation du kit de développement logiciel (SDK) C Azure IoT n’est pas nécessaire pour ce didacticiel, mais elle peut fournir des fonctionnalités utiles, telles qu’IntelliSense et la lecture de définitions de programme. Pour plus d’informations sur l’installation, consultez les [kits de développement logiciel (SDK) et bibliothèques C Azure IoT](https://github.com/Azure/azure-iot-sdk-c).

## <a name="create-a-module-project"></a>Créer un projet de module

Les étapes suivantes permettent de créer un projet de module IoT Edge en C à l’aide de Visual Studio Code et de l’extension Azure IoT Tools. Une fois le modèle de projet créé, ajoutez le nouveau code afin que le module filtre les messages sur la base des propriétés signalées.

### <a name="create-a-new-project"></a>Création d'un projet

Créez un modèle de solution C que vous pouvez personnaliser avec votre propre code.

1. Sélectionnez **Affichage** > **Palette de commandes** pour ouvrir la palette de commandes VS Code.

2. Dans la palette de commandes, tapez et exécutez la commande **Azure: Sign in** et suivez les instructions pour vous connecter à votre compte Azure. Si vous êtes déjà connecté, vous pouvez ignorer cette étape.

3. Dans la palette de commandes, saisissez et exécutez la commande **Azure IoT Edge: New IoT Edge solution**. Suivez les invites de la palette de commandes pour créer votre solution.

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner le dossier | Choisissez l’emplacement sur votre machine de développement pour que VS Code crée les fichiers de la solution. |
   | Provide a solution name (Nommer la solution) | Entrez un nom descriptif pour votre solution ou acceptez le nom par défaut (**EdgeSolution**). |
   | Select module template (Sélectionner un modèle de module) | Choisissez **Module C**. |
   | Provide a module name (Nommer le module) | Nommez votre module **CModule**. |
   | Provide Docker image repository for the module (Indiquer le référentiel d’images Docker pour le module) | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. L’image conteneur est préremplie avec le nom que vous avez indiqué à la dernière étape. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. <br><br> Le référentiel d’images final ressemble à \<registry name\>.azurecr.io/javamodule. |

   ![Fourniture du référentiel d’images Docker](./media/tutorial-c-module/repository.png)

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le fichier d’environnement stocke les informations d’identification de votre registre de conteneurs et les partage avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge.

L’extension IoT Edge tente d’extraire d’Azure vos informations d’identification de registre de conteneurs et de les insérer dans le fichier d’environnement. Vérifiez si vos informations d’identification sont déjà incluses. Si ce n’est pas le cas, veuillez les ajouter maintenant :

1. Dans l’Explorateur VS Code, ouvrez le fichier .env.
2. Mettre à jour les champs avec les valeurs de **nom d’utilisateur** et de **mot de passe** que vous avez copiées à partir de votre registre de conteneurs Azure.
3. Enregistrez ce fichier.

### <a name="select-your-target-architecture"></a>Sélectionner votre architecture cible

Visual Studio Code peut développer des modules C pour les appareils Linux AMD64 et Linux ARM32v7. Vous devez sélectionner l’architecture que vous ciblez avec chaque solution, car le conteneur est généré et s’exécute différemment pour chaque type d’architecture. Linux AMD64 est la valeur par défaut.

1. Ouvrez la palette de commandes et recherchez **Azure IoT Edge: Set Default Target Platform for Edge Solution** (Azure IoT Edge : définir la plateforme cible par défaut pour la solution Edge), ou sélectionnez l’icône de raccourci dans la barre latérale en bas de la fenêtre.

2. Dans la palette de commandes, sélectionnez l’architecture cible dans la liste des options. Pour ce tutoriel, comme nous utilisons une machine virtuelle Ubuntu en tant qu’appareil IoT Edge, nous allons conserver la valeur par défaut **amd64**.

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

Le code du module par défaut reçoit des messages dans une file d’attente d’entrée, puis les transmet à une file d’attente de sortie. Nous allons ajouter du code supplémentaire afin que le module traite les messages en périphérie avant leur transfert vers IoT Hub. Mettez à jour le module afin qu’il analyse les données de température dans chaque message et n’envoie le message à IoT Hub que si la température dépasse un certain seuil.

1. Dans ce scénario, les données du capteur sont fournies au format JSON. Pour filtrer les messages au format JSON, importez une bibliothèque JSON pour C. Ce didacticiel utilise Parson.

   1. Téléchargez le [référentiel GitHub Parson](https://github.com/kgabis/parson). Copiez les fichiers **parson.c** et **parson.h** dans le dossier **CModule**.

   2. Ouvrez **modules** > **CModule** > **CMakeLists.txt**. Au début du fichier, importez les fichiers Parson sous la forme d’une bibliothèque appelée **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```

   3. Ajoutez `my_parson` à la liste des bibliothèques de la fonction **target_link_libraries** de CMakeLists.txt.

   4. Enregistrez le fichier **CMakeLists.txt**.

   5. Ouvrez **modules** > **CModule** > **main.c**. Au bas de la liste des instructions include, ajoutez une nouvelle instruction afin d’inclure `parson.h` pour la prise en charge de JSON :

      ```c
      #include "parson.h"
      ```

1. Dans le fichier **main.c**, ajoutez une variable globale appelée `temperatureThreshold` après la section include. Cette variable définit la valeur que la température mesurée doit dépasser pour que les données soient envoyées à IoT Hub.

    ```c
    static double temperatureThreshold = 25;
    ```

1. Recherchez la fonction `CreateMessageInstance` dans le fichier main.c. Remplacez l’instruction if-else interne par le code suivant qui ajoute quelques lignes de fonctionnalités :

   ```c
       if ((messageInstance->messageHandle = IoTHubMessage_Clone(message)) == NULL)
       {
           free(messageInstance);
           messageInstance = NULL;
       }
       else
       {
           messageInstance->messageTrackingId = messagesReceivedByInput1Queue;
           MAP_HANDLE propMap = IoTHubMessage_Properties(messageInstance->messageHandle);
           if (Map_AddOrUpdate(propMap, "MessageType", "Alert") != MAP_OK)
           {
              printf("ERROR: Map_AddOrUpdate Failed!\r\n");
           }
       }
   ```

   Les nouvelles lignes de code de l’instruction else ajoutent une propriété au message qui lui attribue une étiquette d’alerte. Ce code attribue une étiquette d’alerte à tous les messages, car nous allons ajouter une fonctionnalité qui n’envoie des messages à IoT Hub que si ceux-ci signalent des températures élevées.

1. Remplacez la totalité de la fonction `InputQueue1Callback` par le code suivant. Cette fonction implémente le filtre de messagerie réel. Lorsqu’un message est reçu, il vérifie si la température signalée dépasse le seuil défini. Si c’est le cas, il transfère le message vers sa file d’attente de sortie. Si ce n’est pas le cas, il ignore le message.

    ```c
    static unsigned char *bytearray_to_str(const unsigned char *buffer, size_t len)
    {
        unsigned char *ret = (unsigned char *)malloc(len + 1);
        memcpy(ret, buffer, len);
        ret[len] = '\0';
        return ret;
    }

    static IOTHUBMESSAGE_DISPOSITION_RESULT InputQueue1Callback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
    {
        IOTHUBMESSAGE_DISPOSITION_RESULT result;
        IOTHUB_CLIENT_RESULT clientResult;
        IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle = (IOTHUB_MODULE_CLIENT_LL_HANDLE)userContextCallback;

        unsigned const char* messageBody;
        size_t contentSize;

        if (IoTHubMessage_GetByteArray(message, &messageBody, &contentSize) == IOTHUB_MESSAGE_OK)
        {
            messageBody = bytearray_to_str(messageBody, contentSize);
        } else
        {
            messageBody = "<null>";
        }

        printf("Received Message [%zu]\r\n Data: [%s]\r\n",
                messagesReceivedByInput1Queue, messageBody);

        // Check if the message reports temperatures higher than the threshold
        JSON_Value *root_value = json_parse_string(messageBody);
        JSON_Object *root_object = json_value_get_object(root_value);
        double temperature;
        if (json_object_dotget_value(root_object, "machine.temperature") != NULL && (temperature = json_object_dotget_number(root_object, "machine.temperature")) > temperatureThreshold)
        {
            printf("Machine temperature %f exceeds threshold %f\r\n", temperature, temperatureThreshold);
            // This message should be sent to next stop in the pipeline, namely "output1".  What happens at "outpu1" is determined
            // by the configuration of the Edge routing table setup.
            MESSAGE_INSTANCE *messageInstance = CreateMessageInstance(message);
            if (NULL == messageInstance)
            {
                result = IOTHUBMESSAGE_ABANDONED;
            }
            else
            {
                printf("Sending message (%zu) to the next stage in pipeline\n", messagesReceivedByInput1Queue);

                clientResult = IoTHubModuleClient_LL_SendEventToOutputAsync(iotHubModuleClientHandle, messageInstance->messageHandle, "output1", SendConfirmationCallback, (void *)messageInstance);
                if (clientResult != IOTHUB_CLIENT_OK)
                {
                    IoTHubMessage_Destroy(messageInstance->messageHandle);
                    free(messageInstance);
                    printf("IoTHubModuleClient_LL_SendEventToOutputAsync failed on sending msg#=%zu, err=%d\n", messagesReceivedByInput1Queue, clientResult);
                    result = IOTHUBMESSAGE_ABANDONED;
                }
                else
                {
                    result = IOTHUBMESSAGE_ACCEPTED;
                }
            }
        }
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

1. Ajoutez une fonction `moduleTwinCallback`. Cette méthode reçoit des mises à jour sur les propriétés souhaitées à partir du double de module et met à jour la variable **temperatureThreshold** en conséquence. Tous les modules ont leur propre double, ce qui vous permet de configurer le code exécuté à l’intérieur d’un module directement à partir du cloud.

    ```c
    static void moduleTwinCallback(DEVICE_TWIN_UPDATE_STATE update_state, const unsigned char* payLoad, size_t size, void* userContextCallback)
    {
        printf("\r\nTwin callback called with (state=%s, size=%zu):\r\n%s\r\n",
            MU_ENUM_TO_STRING(DEVICE_TWIN_UPDATE_STATE, update_state), size, payLoad);
        JSON_Value *root_value = json_parse_string(payLoad);
        JSON_Object *root_object = json_value_get_object(root_value);
        if (json_object_dotget_value(root_object, "desired.TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_dotget_number(root_object, "desired.TemperatureThreshold");
        }
        if (json_object_get_value(root_object, "TemperatureThreshold") != NULL) {
            temperatureThreshold = json_object_get_number(root_object, "TemperatureThreshold");
        }
    }
    ```

1. Recherchez la fonction `SetupCallbacksForModule`. Remplacez la fonction par le code suivant, qui ajoute une instruction **if else** pour vérifier si le jumeau de module a été mis à jour.

   ```c
   static int SetupCallbacksForModule(IOTHUB_MODULE_CLIENT_LL_HANDLE iotHubModuleClientHandle)
   {
       int ret;

       if (IoTHubModuleClient_LL_SetInputMessageCallback(iotHubModuleClientHandle, "input1", InputQueue1Callback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetInputMessageCallback(\"input1\")..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else if (IoTHubModuleClient_LL_SetModuleTwinCallback(iotHubModuleClientHandle, moduleTwinCallback, (void*)iotHubModuleClientHandle) != IOTHUB_CLIENT_OK)
       {
           printf("ERROR: IoTHubModuleClient_LL_SetModuleTwinCallback(default)..........FAILED!\r\n");
           ret = MU_FAILURE;
       }
       else
       {
           ret = 0;
       }

       return ret;
   }
   ```

1. Enregistrez le fichier main.c.

1. Dans l’Explorateur VS Code, ouvrez le fichier **deployment.template.json** dans votre espace de travail de solution IoT Edge.

1. Ajoutez le jumeau de module CModule au manifeste de déploiement. Insérez le contenu JSON suivant en bas de la section `moduleContent`, après le jumeau de module `$edgeHub` :

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Ajout du jumeau de module CModule au modèle de déploiement](./media/tutorial-c-module/module-twin.png)

1. Enregistrez le fichier **deployment.template.json**.

## <a name="build-and-push-your-module"></a>Générer et envoyer (push) votre module

Dans la section précédente, vous avez créé une solution IoT Edge et ajouté du code à CModule pour filtrer les messages quand la température de machine signalée se situe dans les limites acceptables. Vous devez maintenant générer la solution comme image de conteneur et l’envoyer à votre registre de conteneurs.

1. Ouvrez le terminal VS Code en sélectionnant **Affichage** > **Terminal**.

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

4. Développez la section **Modules** sous votre appareil pour voir la liste des modules déployés et en cours d’exécution. Cliquez sur le bouton Actualiser. Vous devez voir le nouveau module **CModule** en cours d’exécution avec le module **SimulatedTemperatureSensor** ainsi que **$edgeAgent** et **$edgeHub**.

    Le démarrage des modules peut prendre plusieurs minutes. Le runtime IoT Edge doit recevoir son nouveau manifeste de déploiement, extraire les images de module à partir du runtime du conteneur, puis démarrer chaque nouveau module.

## <a name="view-generated-data"></a>Afficher les données générées

Une fois que vous appliquez le manifeste de déploiement à votre appareil IoT Edge, le runtime IoT Edge sur l’appareil collecte les nouvelles informations de déploiement et commence à s’exécuter sur celui-ci. Tous les modules en cours d’exécution sur l’appareil qui ne sont pas inclus dans le manifeste de déploiement sont arrêtés. Tous les modules manquant de l’appareil sont démarrés.

1. Dans l’explorateur Visual Studio Code, cliquez avec le bouton droit sur le nom de votre appareil IoT Edge, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

2. Affichez les messages reçus dans votre hub IoT. Les messages peuvent mettre un certain temps à arriver, car l’appareil IoT Edge doit recevoir son nouveau déploiement et démarrer tous les modules. Ensuite, les modifications que nous avons apportées au code du module CModule attendent que la température de la machine atteigne 25 degrés avant d’envoyer les messages. Le type de message **Alerte** est également ajouté à chaque message qui atteint ce seuil de température.

   ![Afficher les messages reçus sur IoT Hub](./media/tutorial-c-module/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Modifier le jumeau de module

Nous avons utilisé le module CModule dans le manifeste de déploiement pour définir le seuil de température sur 25 degrés. Vous pouvez utiliser le jumeau de module pour changer cette fonctionnalité sans devoir mettre à jour le code du module.

1. Dans Visual Studio Code, développez les détails de votre appareil IoT Edge pour afficher les modules Cours d’exécution.

2. Cliquez avec le bouton droit sur **CModule**, puis sélectionnez **Modifier le jumeau de module**.

3. Recherchez **TemperatureThreshold** dans les propriétés souhaitées. Définissez sa valeur sur une nouvelle température supérieure de 5 à 10 degrés à la dernière température indiquée.

4. Enregistrez le jumeau de module.

5. Cliquez avec le bouton droit sur le volet d’édition du jumeau de module et sélectionnez **Mettre à jour le jumeau de module**.

6. Supervisez les messages appareil-à-cloud entrants. Vous devriez voir les messages s’arrêter jusqu’à ce que le nouveau seuil de température soit atteint.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez utilisées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un module IoT Edge qui contient le code pour filtrer les données brutes générées par votre appareil IoT Edge.

Vous pouvez passer aux tutoriels suivants afin de découvrir comment Azure IoT Edge peut vous aider à déployer des services cloud Azure pour traiter et analyser des données en périphérie.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Service Custom Vision](tutorial-deploy-custom-vision.md)