---
title: 'Tutoriel : Développer un module C pour Windows - Azure IoT Edge | Microsoft Docs'
description: Ce tutoriel explique comment créer un module IoT Edge avec du code C, et comment le déployer sur un appareil Windows exécutant IoT Edge.
services: iot-edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/28/2019
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: d9cffcadcb95b6c8c61205d458610f402fa7286d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964590"
---
# <a name="tutorial-develop-a-c-iot-edge-module-for-windows-devices"></a>Tutoriel : Développer un module IoT Edge en C pour les appareils Windows

Utilisez Visual Studio pour écrire du code C et pour le déployer sur un appareil Windows exécutant Azure IoT Edge.

Vous pouvez utiliser des modules Azure IoT Edge pour déployer un code qui implémente votre logique métier directement sur vos appareils IoT Edge. Ce tutoriel vous guide dans la création et le déploiement d’un module IoT Edge qui filtre des données de capteur. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Utiliser Visual Studio pour créer un module IoT Edge basé sur le SDK C.
> * Utiliser Visual Studio et Docker pour créer une image Docker et la publier sur votre registre
> * Déployer le module sur votre appareil IoT Edge
> * Afficher les données générées

Le module IoT Edge que vous créez dans ce didacticiel filtre les données de température générées par votre appareil. Il envoie uniquement des messages en amont lorsque la température dépasse un seuil spécifié. Ce type d’analyse à la périphérie est utile pour réduire la quantité de données communiquées et stockées dans le cloud.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Ce tutoriel montre comment développer un module en **C** à l’aide de **Visual Studio 2019** et comment le déployer sur un **appareil Windows**. Si vous développez des modules pour les appareils Linux, consultez [Développer un module IoT Edge en C pour les appareils Linux](tutorial-c-module.md).

Utilisez le tableau suivant afin de comprendre les options dont vous disposez pour développer et déployer des modules en C sur des appareils Windows :

| C | Visual Studio Code | Visual Studio 2017/2019 |
| -- | ------------------ | ------------------ |
| **Windows AMD64** |  | ![Développer des modules en C pour WinAMD64 dans Visual Studio](./media/tutorial-c-module/green-check.png) |

Avant de commencer ce tutoriel, vous devez avoir suivi celui qui précède afin de configurer votre environnement pour le développement de conteneur Windows : [Développer des modules IoT Edge pour les appareils Windows](tutorial-develop-for-windows.md) Ce tutoriel vous permet d’obtenir les prérequis suivants :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.
* Un [appareil Windows exécutant Azure IoT Edge](quickstart.md).
* Un registre de conteneurs tel qu’[Azure Container Registry](../container-registry/index.yml).
* [Visual Studio 2019](/visualstudio/install/install-visual-studio) configuré avec l’extension [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16iotedgetools).
* [Docker Desktop](https://docs.docker.com/docker-for-windows/install/), configuré pour exécuter des conteneurs Windows.
* Installez le SDK C Azure IoT pour Windows x64 par le biais de vcpkg :

   ```powershell
   git clone https://github.com/Microsoft/vcpkg
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg install azure-iot-sdk-c:x64-windows
   .\vcpkg --triplet x64-windows integrate install
   ```

> [!TIP]
> Si vous utilisez Visual Studio 2017 (version 15.7 ou supérieure), téléchargez et installez [Azure IoT Edge Tools](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vsiotedgetools) pour VS 2017 à partir de la place de marché Visual Studio

## <a name="create-a-module-project"></a>Créer un projet de module

Les étapes suivantes montrent comment créer un projet de module IoT Edge basé sur le SDK C à l’aide de Visual Studio et de l’extension Azure IoT Edge Tools. Une fois le modèle de projet créé, ajoutez le nouveau code afin que le module filtre les messages sur la base des propriétés signalées.

### <a name="create-a-new-project"></a>Création d'un projet

Créez un modèle de solution C que vous pouvez personnaliser avec votre propre code.

1. Lancez Visual Studio 2019, puis sélectionnez **Créer un projet**.

2. Recherchez **IoT Edge**, puis choisissez le projet **Azure IoT Edge (Windows amd64)** . Cliquez sur **Suivant**.

   ![Créer un projet Azure IoT Edge](./media/tutorial-c-module-windows/new-project.png)

3. Renommez le projet et la solution à l’aide d’un nom descriptif comme **CTutorialApp**. Cliquez sur **Créer** pour créer le projet.

   ![Configurer un nouveau projet Azure IoT Edge](./media/tutorial-c-module-windows/configure-project.png)

4. Configurez votre projet avec les valeurs suivantes :

   | Champ | Valeur |
   | ----- | ----- |
   | Sélectionner un modèle | Sélectionnez **Module C**. |
   | Nom du projet de module | Nommez votre module **CModule**. |
   | Dépôt d’images Docker | Un référentiel d’images comprend le nom de votre registre de conteneurs et celui de votre image conteneur. Votre image conteneur est préremplie avec le nom du projet de module. Remplacez **localhost:5000** par la valeur de **Serveur de connexion** provenant de votre registre de conteneurs Azure. Vous pouvez récupérer le serveur de connexion à partir de la page Vue d’ensemble de votre registre de conteneurs dans le portail Azure. <br><br> Le référentiel d’images final ressemble à \<registry name\>.azurecr.io/javamodule. |

   ![Configurer votre projet pour l’appareil cible, le type de module et le registre de conteneurs](./media/tutorial-c-module-windows/add-application-and-module.png)

5. Sélectionnez **Ajouter** pour créer le projet.

### <a name="add-your-registry-credentials"></a>Ajouter les informations d’identification de votre registre

Le manifeste de déploiement partage les informations d’identification de votre registre de conteneurs avec le runtime IoT Edge. Le runtime a besoin de ces informations d’identification pour extraire vos images privées sur l’appareil IoT Edge. Utilisez les informations d’identification de la section **Clés d’accès** dans votre registre de conteneurs Azure.

1. Dans l’Explorateur de solutions Visual Studio, ouvrez le fichier **deployment.template.json**.

2. Recherchez la propriété **registryCredentials** dans les propriétés $edgeAgent souhaitées. Votre adresse de registre doit être automatiquement remplie à partir des informations que vous avez fournies au moment de la création du projet. Les champs du nom d’utilisateur et du mot de passe doivent contenir des noms de variables. Par exemple :

   ```json
   "registryCredentials": {
     "<registry name>": {
       "username": "$CONTAINER_REGISTRY_USERNAME_<registry name>",
       "password": "$CONTAINER_REGISTRY_PASSWORD_<registry name>",
       "address": "<registry name>.azurecr.io"
     }
   }
   ```

3. Ouvrez le fichier **.env** dans votre solution de module. (Dans la mesure où il est masqué par défaut dans l’Explorateur de solutions, vous devrez peut-être sélectionner le bouton **Afficher tous les fichiers** pour l’afficher.) Le fichier .env doit contenir les mêmes variables de nom d’utilisateur et de mot de passe que celles que vous avez vues dans le fichier deployment.template.json.

4. Ajoutez les valeurs de **Nom d’utilisateur** et **Mot de passe** provenant de votre registre de conteneurs Azure.

5. Enregistrez les modifications apportées au fichier .env.

### <a name="update-the-module-with-custom-code"></a>Mettre à jour le module avec du code personnalisé

Le code du module par défaut reçoit des messages dans une file d’attente d’entrée, puis les transmet à une file d’attente de sortie. Nous allons ajouter du code supplémentaire afin que le module traite les messages en périphérie avant leur transfert vers IoT Hub. Mettez à jour le module afin qu’il analyse les données de température dans chaque message et n’envoie le message à IoT Hub que si la température dépasse un certain seuil.

1. Dans ce scénario, les données du capteur sont fournies au format JSON. Pour filtrer les messages au format JSON, importez une bibliothèque JSON pour C. Ce didacticiel utilise Parson.

   1. Téléchargez le [référentiel GitHub Parson](https://github.com/kgabis/parson). Copiez les fichiers **parson.c** et **parson.h** dans le projet **CModule**.

   2. Dans Visual Studio, ouvrez le fichier **CMakeLists.txt** à partir du dossier de projet CModule. Au début du fichier, importez les fichiers Parson sous la forme d’une bibliothèque appelée **my_parson**.

      ```txt
      add_library(my_parson
          parson.c
          parson.h
      )
      ```

   3. Ajoutez `my_parson` à la liste des bibliothèques de la section **target_link_libraries** du fichier CMakeLists.txt.

   4. Enregistrez le fichier **CMakeLists.txt**.

   5. Ouvrez **CModule** > **main.c**. Au bas de la liste des instructions include, ajoutez une nouvelle instruction afin d’inclure `parson.h` pour la prise en charge de JSON :

      ```c
      #include "parson.h"
      ```

2. Dans le fichier **main.c** , ajoutez une variable globale appelée `temperatureThreshold` en regard de la variable messagesReceivedByInput1Queue. Cette variable définit la valeur que la température mesurée doit dépasser pour que les données soient envoyées à IoT Hub.

    ```c
    static double temperatureThreshold = 25;
    ```

3. Recherchez la fonction `CreateMessageInstance` dans le fichier main.c. Remplacez l’instruction if-else interne par le code suivant qui ajoute quelques lignes de fonctionnalités :

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

4. Recherchez la fonction `InputQueue1Callback` et remplacez tout le code de la fonction par le code suivant : Cette fonction implémente le filtre de messagerie réel. Lorsqu’un message est reçu, il vérifie si la température signalée dépasse le seuil défini. Si c’est le cas, il transfère le message vers sa file d’attente de sortie. Si ce n’est pas le cas, il ignore le message.

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

        // If temperature exceeds threshold, send to output1
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
        // If message does not exceed threshold, do not forward
        else
        {
            printf("Not sending message (%zu) to the next stage in pipeline.\r\n", messagesReceivedByInput1Queue);
            result = IOTHUBMESSAGE_ACCEPTED;
        }

        messagesReceivedByInput1Queue++;
        return result;
    }
    ```

5. Ajoutez une fonction `moduleTwinCallback`. Cette méthode reçoit des mises à jour sur les propriétés souhaitées à partir du double de module et met à jour la variable **temperatureThreshold** en conséquence. Tous les modules ont leur propre double, ce qui vous permet de configurer le code exécuté à l’intérieur d’un module directement à partir du cloud.

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

6. Recherchez la fonction `SetupCallbacksForModule`. Remplacez la fonction par le code suivant, qui ajoute une instruction **if else** pour vérifier si le jumeau de module a été mis à jour.

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

7. Enregistrez le fichier main.c.

8. Ouvrez le fichier **deployment.template.json**.

9. Ajoutez le jumeau de module CModule au manifeste de déploiement. Insérez le contenu JSON suivant en bas de la section `moduleContent`, après le jumeau de module `$edgeHub` :

   ```json
   "CModule": {
       "properties.desired":{
           "TemperatureThreshold":25
       }
   }
   ```

   ![Ajout du jumeau de module CModule au modèle de déploiement](./media/tutorial-c-module-windows/module-twin.png)

10. Enregistrez le fichier **deployment.template.json**.

## <a name="build-and-push-your-module"></a>Générer et envoyer (push) votre module

Dans la section précédente, vous avez créé une solution IoT Edge et ajouté du code dans le module **CModule** pour filtrer les messages qui signalent une température inférieure au seuil acceptable. Vous devez maintenant générer la solution comme image de conteneur et l’envoyer à votre registre de conteneurs.

### <a name="sign-in-to-docker"></a>Se connecter à Docker

Fournissez les informations d’identification du registre de conteneurs à Docker sur votre machine de développement afin qu’il puisse envoyer (push) votre image conteneur à stocker dans le registre.

1. Ouvrez PowerShell ou une invite de commandes.

2. Connectez-vous à Docker avec les informations d’identification du registre de conteneurs Azure que vous avez enregistré après avoir créé le registre.

   ```cmd
   docker login -u <ACR username> -p <ACR password> <ACR login server>
   ```

   Il se peut que vous receviez un avertissement de sécurité recommandant d’utiliser `--password-stdin`. Bien qu’il s’agisse de la bonne pratique recommandée pour les scénarios de production, elle n’est pas pertinente pour ce tutoriel. Pour plus d’informations, consultez les informations de référence sur [docker login](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin).

### <a name="build-and-push"></a>Générer et envoyer (push)

Votre machine de développement a désormais accès à votre registre de conteneurs ainsi que vos appareils IoT Edge. Il est temps de transformer le code du projet en image conteneur.

1. Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur le nom du projet que vous voulez générer. Le nom par défaut est **AzureIotEdgeApp1**. Pour ce tutoriel, le nom **CTutorialApp** a été choisi. Puisque vous créez un module Windows, l’extension doit être **Windows.Amd64**.

2. Sélectionnez **Générer et envoyer (push) les modules IoT Edge**.

   La commande de génération et d’envoi (push) déclenche trois opérations. Tout d’abord, elle crée un nouveau dossier dans la solution appelé **config** contenant l’intégralité du manifeste de déploiement. Celui-ci est généré à partir des informations du modèle de déploiement et d’autres fichiers de solution. Ensuite, elle exécute `docker build` pour générer l’image de conteneur basée sur le fichier docker correspondant à votre architecture cible. Puis, elle exécute `docker push` pour envoyer (push) le dépôt d’images vers votre registre de conteneurs.

   Ce processus peut prendre plusieurs minutes la première fois, mais il est plus rapide la prochaine fois que vous exécutez les commandes.

## <a name="deploy-modules-to-device"></a>Déployer des modules sur un appareil

Utilisez Visual Studio Cloud Explorer et l’extension Azure IoT Edge Tools pour déployer le projet de module sur votre appareil IoT Edge. Vous disposez déjà d’un manifeste de déploiement préparé pour votre scénario, à savoir le fichier **deployment.windows-amd64.json** figurant dans le dossier config. Il vous suffit alors de sélectionner l’appareil qui recevra le déploiement.


Vérifiez que votre appareil IoT Edge est opérationnel.

1. Dans Visual Studio Cloud Explorer, développez les ressources pour voir la liste des appareils IoT.

2. Cliquez avec le bouton droit sur le nom de l’appareil IoT Edge sur lequel effectuer le déploiement.

3. Sélectionnez **Créer un déploiement**.

4. Dans l’Explorateur de fichiers, sélectionnez le fichier **deployment.windows-amd64** dans le dossier config de votre solution.

5. Actualisez Cloud Explorer pour voir les modules déployés listés sous votre appareil.

## <a name="view-generated-data"></a>Afficher les données générées

Une fois que vous appliquez le manifeste de déploiement à votre appareil IoT Edge, le runtime IoT Edge sur l’appareil collecte les nouvelles informations de déploiement et commence à s’exécuter sur celui-ci. Tous les modules en cours d’exécution sur l’appareil qui ne sont pas inclus dans le manifeste de déploiement sont arrêtés. Tous les modules manquant de l’appareil sont démarrés.

Vous pouvez utiliser l’extension IoT Edge Tools pour voir les messages à mesure qu’ils arrivent dans votre hub IoT.

1. Dans Visual Studio Cloud Explorer, sélectionnez le nom de votre appareil IoT Edge.

2. Dans la liste **Actions**, sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

3. Affichez les messages reçus dans votre hub IoT. Les messages peuvent mettre un certain temps à arriver, car l’appareil IoT Edge doit recevoir son nouveau déploiement et démarrer tous les modules. Ensuite, les modifications que nous avons apportées au code du module CModule attendent que la température de la machine atteigne 25 degrés avant d’envoyer les messages. Le type de message **Alerte** est également ajouté à chaque message qui atteint ce seuil de température.

   ![Afficher les messages reçus sur IoT Hub](./media/tutorial-c-module-windows/view-d2c-message.png)

## <a name="edit-the-module-twin"></a>Modifier le jumeau de module

Nous avons utilisé le jumeau de module de CModule pour définir le seuil de température sur 25 degrés. Vous pouvez utiliser le jumeau de module pour changer cette fonctionnalité sans devoir mettre à jour le code du module.

1. Dans Visual Studio, ouvrez le fichier **deployment.windows-amd64.json** (et non le fichier deployment.template. Si vous ne voyez pas le manifeste de déploiement dans le fichier config dans l’Explorateur de solutions, sélectionnez l’icône **Afficher tous les fichiers** dans la barre d’outils de l’explorateur).

2. Recherchez le jumeau de module de CModule et remplacez la valeur du paramètre **temperatureThreshold** par une température supérieure de 5 à 10 degrés par rapport à la dernière température signalée.

3. Enregistrez le fichier **deployment.windows-amd64.json**.

4. Effectuez de nouveau les étapes de déploiement pour appliquer le manifeste de déploiement mis à jour à votre appareil.

5. Supervisez les messages appareil-à-cloud entrants. Vous devriez voir les messages s’arrêter jusqu’à ce que le nouveau seuil de température soit atteint.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez utilisées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé un module IoT Edge contenant le code pour filtrer les données brutes générées par votre appareil IoT Edge.

Vous pouvez passer aux tutoriels suivants afin de découvrir comment Azure IoT Edge peut vous aider à déployer des services cloud Azure pour traiter et analyser des données en périphérie.

> [!div class="nextstepaction"]
> [Functions](tutorial-deploy-function.md)
> [Stream Analytics](tutorial-deploy-stream-analytics.md)
> [Machine Learning](tutorial-deploy-machine-learning.md)
> [Service Custom Vision](tutorial-deploy-custom-vision.md)