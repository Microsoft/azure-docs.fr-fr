---
title: Le Kit de développement logiciel (SDK) d’appareil Azure IoT pour C | Microsoft Docs
description: Prenez en main Azure IoT device SDK pour C et apprenez à créer des applications d’appareil qui communiquent avec un IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 9b870e21ffd5c6a8261b6731b939b5dff558256d
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501190"
---
# <a name="azure-iot-device-sdk-for-c"></a>Azure IoT device SDK pour C

Le **Kit de développement logiciel (SDK) d’appareil Azure IoT** (Azure IoT device SDK) est un ensemble de bibliothèques conçu pour simplifier le processus d’envoi et de réception de messages vers et à partir du service **Azure IoT Hub**. Il existe différentes variantes de ce kit de développement logiciel, chacune concernant une plateforme spécifique, mais cet article met l'accent sur le **kit de développement logiciel Azure IoT device SDK pour C**.

> [!NOTE]
> Le kit SDK Embedded C est une alternative pour les appareils avec contraintes qui prend en charge l’approche BYON (Bring Your Own Network). Les développeurs IoT ont la liberté d’apporter le client MQTT, le protocole TLS et le socket de leur choix pour créer une solution d’appareil. [En savoir plus sur le kit SDK Embedded C](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Le kit de développement logiciel d’appareil Azure IoT pour C est rédigé en code ANSI C (C99) afin d’optimiser sa portabilité. Cette fonctionnalité rend les bibliothèques adaptées pour fonctionner sur plusieurs plateformes et appareils, en particulier quand la réduction de l’encombrement du disque et de l’empreinte mémoire est une priorité.

Le Kit de développement logiciel (SDK) a été testé sur un large éventail de plateformes (consultez le [Catalogue d’appareils certifiés Azure pour l’IoT](https://catalog.azureiotsolutions.com/) pour plus d’informations). Bien que cet article contienne des procédures pas à pas d’exemple de code s’exécutant sur la plateforme Windows, le code décrit dans cet article est identique sur l’ensemble des plateformes prises en charge.

La vidéo suivante présente une vue d’ensemble du Kit de développement logiciel (SDK) Azure IoT pour C :

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-IoT-C-SDK-insights/Player]

Cet article, vous initie à l’architecture du kit de développement logiciel (SDK) d’appareil Azure IoT pour C. Il montre comment initialiser la bibliothèque de périphériques, envoyer des données à IoT Hub et recevoir des messages de sa part. Les informations de cet article sont suffisantes pour commencer à utiliser le Kit de développement logiciel (SDK), mais elles vous fournissent également des indications qui vous permettront d’obtenir des informations supplémentaires sur les bibliothèques.

## <a name="sdk-architecture"></a>Architecture du kit de développement logiciel (SDK)

Vous trouverez [**Azure IoT device SDK pour C**](https://github.com/Azure/azure-iot-sdk-c) dans le référentiel GitHub. Vous pouvez consulter les détails de l’[API dans Référence de l’API C](/azure/iot-hub/iot-c-sdk-ref/).

Vous trouverez la dernière version des bibliothèques dans la branche **maître** du référentiel :

  ![Capture d’écran de la branche maître du référentiel](./media/iot-hub-device-sdk-c-intro/RepoMasterBranch.png)

* L’implémentation de base du Kit de développement logiciel (SDK) se trouve dans le dossier **iothub\_client** qui contient l’implémentation de la couche d’API la plus basse du Kit de développement logiciel (SDK) : la bibliothèque **IoTHubClient**. La bibliothèque **IoTHubClient** contient des API implémentant des messages bruts permettant d’envoyer des messages à IoT Hub et de recevoir des messages de IoT Hub. Lorsque vous utilisez cette bibliothèque, vous êtes responsable de la mise en œuvre de la sérialisation du message, mais d’autres détails concernant la communication avec IoT Hub sont gérés pour vous.

* Le dossier **serializer** contient des fonctions d’assistance et des exemples qui vous montrent comment sérialiser des données avant d’effectuer un envoi à Azure IoT Hub à l’aide de la bibliothèque cliente. L’utilisation du sérialiseur n’est pas obligatoire et est fournie à titre de commodité. Pour utiliser la bibliothèque **serializer**, vous définissez un modèle désignant les données que vous souhaitez envoyer à IoT Hub, et les messages que vous attendez de sa part. Une fois le modèle défini, le Kit de développement logiciel (SDK) vous fournit une surface d’API qui vous permet de travailler facilement avec des messages Appareil vers cloud et Cloud vers appareil, sans se soucier des détails de sérialisation. La bibliothèque dépend d’autres bibliothèques open source qui implémentent le transport à l’aide de protocoles tels que MQTT et AMQP.

* La bibliothèque **IoTHubClient** dépend d’autres bibliothèques open source :

  * La bibliothèque de [l’utilitaire partagé Azure C](https://github.com/Azure/azure-c-shared-utility), qui fournit des fonctionnalités communes pour les tâches de base (comme la manipulation de chaînes ou de listes et les E/S) nécessaires dans plusieurs Kits de développement logiciel (SDK) C liés à Azure.

  * La bibliothèque [Azure uAMQP](https://github.com/Azure/azure-uamqp-c), qui est une implémentation côté client du protocole AMQP optimisée pour les appareils avec contraintes de ressources.

  * La bibliothèque [Azure uMQTT](https://github.com/Azure/azure-umqtt-c) , qui est une bibliothèque à usage général implémentant le protocole MQTT et optimisée pour les appareils avec contraintes de ressources.

L’exemple de code permet de mieux comprendre l’utilisation de ces bibliothèques. Les sections suivantes vous guident à travers plusieurs exemples d’applications inclus dans le kit de développement logiciel (SDK). Cette procédure pas à pas devrait vous donner une idée des différentes fonctionnalités des couches architecturales du kit de développement logiciel (SDK) et vous initier au fonctionnement de l’API.

## <a name="before-you-run-the-samples"></a>Avant d’exécuter les exemples

Avant de pouvoir exécuter les exemples du Kit de développement logiciel (SDK) d’appareil Azure IoT pour C, vous devez [créer une instance du service IoT Hub](iot-hub-create-through-portal.md) dans votre abonnement Azure. Effectuez ensuite les tâches suivantes :

* Préparer votre environnement de développement
* Obtenir les informations d’identification de l’appareil.

### <a name="prepare-your-development-environment"></a>Préparer votre environnement de développement

Les packages sont fournis pour les plates-formes courantes (telles que NuGet pour Windows ou apt_get pour Debian et Ubuntu) et les exemples utilisent ces packages lorsqu’ils sont disponibles. Dans certains cas, vous devez compiler le Kit de développement logiciel (SDK) pour ou sur votre appareil. Si vous avez besoin compiler le Kit de développement logiciel (SDK), consultez [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) dans le référentiel GitHub.

Pour obtenir l’exemple de code d’application, téléchargez une copie du Kit de développement logiciel (SDK) à partir de GitHub. Obtenez votre copie de la source à partir de la branche **maître** du [référentiel GitHub](https://github.com/Azure/azure-iot-sdk-c).


### <a name="obtain-the-device-credentials"></a>Obtenir les informations d’identification de l’appareil

Maintenant que vous avez l’exemple de code source, vous devez à présent obtenir un ensemble d’informations d’identification d’appareils. Pour qu’un appareil puisse accéder à un IoT Hub, vous devez d’abord ajouter l’appareil au registre des identifiés de l’IoT Hub. Lorsque vous ajoutez votre périphérique, vous obtenez un jeu d’informations d’identification dont vous avez besoin pour permettre au périphérique de se connecter au IoT Hub. Les exemples d’application qui figurent dans la section qui suit attendent ces informations d’identification sous la forme de **chaîne de connexion d’appareil**.

Il existe plusieurs outils open source pour vous aider à gérer votre IoT Hub.

* Une application Windows appelée [Explorateur Azure IoT](https://github.com/Azure/azure-iot-explorer).

* Une extension multiplateforme de Visual Studio Code appelée [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

* Une interface CLI Python multiplateforme appelée [extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

Ce didacticiel utilise l’outil graphique *Explorateur d’appareils*. Vous pouvez utiliser *Azure IoT Tools pour VS Code* si vous développez dans VS Code. Vous pouvez également utiliser l’outil d’*extension IoT pour Azure CLI 2.0* si vous préférez utiliser un outil CLI.

L’outil Explorateur d’appareils utilise les bibliothèques de service Azure IoT pour effectuer diverses fonctions sur IoT Hub, notamment ajouter des appareils. Si vous utilisez l'outil Explorateur d'appareils pour ajouter un appareil, vous obtenez une chaîne de connexion pour votre appareil. Vous avez besoin de cette chaîne de connexion pour exécuter les exemples d'applications.

Si vous n’êtes pas familiarisé avec l’outil Explorateur d’appareils, la procédure qui suit explique comment l’utiliser pour ajouter un périphérique et obtenir une chaîne de connexion d’appareil.

1. Pour installer l’outil Explorateur d’appareils, consultez [How to use the Device Explorer for IoT Hub devices](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/) (comment utiliser l’Explorateur d’appareils pour les appareils IoT Hub).

1. Lorsque vous exécutez le programme, vous voyez cette interface :

   ![Capture d’écran de Device Explorer Twin](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinConfigTab.png)

1. Entrez votre **chaîne de connexion IoT Hub** dans le premier champ, puis cliquez sur **Mettre à jour**. Cette étape sert à configurer l’outil pour lui permettre de communiquer avec IoT Hub. 

La **chaîne de connexion** peut être trouvée sous **Service IoT Hub** > **Paramètres** > **Stratégie d’accès partagé** > **iothubowner**.

1. Lorsque la chaîne de connexion IoT Hub est configurée, cliquez sur l’onglet **Gestion** :

   ![Capture d’écran de Device Explorer Twin / Management](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab.png)

C’est depuis cet onglet que vous gérez les appareils inscrits dans votre IoT Hub.

1. Vous créez un périphérique en cliquant sur le bouton **Créer** . Une boîte de dialogue avec un jeu de clés (primaire et secondaire) préalablement remplies s’affiche. Entrez un **ID d’appareil**, puis cliquez sur **Créer**.

   ![Capture d’écran de la création de l’appareil](./media/iot-hub-device-sdk-c-intro/CreateDevice.png)

1. Lorsque l’appareil est créé, la liste des appareils s’actualise avec tous les appareils inscrits, dont celui que vous venez de créer. Si vous cliquez avec le bouton droit sur le nouvel appareil, le menu qui suit s’affiche :

   ![Résultat après clic droit sur Device Explorer Twin](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab_RightClick.png)

1. Si vous choisissez **Copier la chaîne de connexion de l’appareil sélectionné**, la chaîne de connexion en question est copiée dans le Presse-papiers. Conservez une copie de la chaîne de connexion de l’appareil. Vous en avez besoin au moment d’exécuter les exemples d’application décrits dans les sections suivantes.

Lorsque vous avez effectué les opérations ci-dessus, vous êtes prêt à commencer l’exécution du code. En haut du fichier source principal, la plupart des exemples décrits ci-dessous contiennent une constante qui permet d’entrer une chaîne de connexion. Par exemple, la ligne correspondante de l’application **iothub_client\_samples\_iothub_convenience_sample** se présente comme suit.

```c
static const char* connectionString = "[device connection string]";
```

## <a name="use-the-iothubclient-library"></a>Utiliser la bibliothèque IoTHubClient

Dans le dossier **iothub\_client** du référentiel [azure-iot-sdk-c](https://github.com/azure/azure-iot-sdk-c) se trouve un dossier **samples** contenant une application appelée **iothub\_client\_sample\_mqtt**.

La version Windows de l’application **iothub_client\_samples\_iothub_convenience_sample** contient la solution Visual Studio suivante :

  ![Explorateur de solutions Visual Studio](./media/iot-hub-device-sdk-c-intro/iothub-client-sample-mqtt.png)

> [!NOTE]
> Si Visual Studio vous demande de recibler le projet vers la dernière version, acceptez l’invite.

Cette solution inclut un seul projet : Cette solution installe quatre packages NuGet :

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.umqtt

Quand vous travaillez avec le Kit de développement logiciel (SDK), vous devez toujours utiliser le package **Microsoft.Azure.C.SharedUtility** . Cet exemple utilise le protocole MQTT. Par conséquent vous devez inclure les packages **Microsoft.Azure.umqtt** et **Microsoft.Azure.IoTHub.MqttTransport** (il existe des packages équivalents pour AMQP et HTTPS). Comme l’exemple utilise la bibliothèque **IoTHubClient**, vous devez également inclure le package **Microsoft.Azure.IoTHub.IoTHubClient** dans votre solution.

L’implémentation de l’exemple d’application est disponible dans le fichier source **iothub_client\_samples\_iothub_convenience_sample**.

Les étapes suivantes utilisent cet exemple d’application pour vous montrer les éléments requis pour utiliser la bibliothèque **IoTHubClient**.

### <a name="initialize-the-library"></a>Initialiser la bibliothèque

> [!NOTE]
> Avant d’utiliser les bibliothèques, vous devrez peut-être procéder à une initialisation spécifique à la plateforme. Par exemple, si vous prévoyez d’utiliser AMQP sur Linux, vous devez initialiser la bibliothèque OpenSSL. Les exemples du [référentiel GitHub](https://github.com/Azure/azure-iot-sdk-c) appellent la fonction de l’utilitaire **platform\_init** quand le client démarre et appelle la fonction **platform\_deinit** avant de se fermer. Ces fonctions sont déclarées dans le fichier d’en-tête platform.h. Examinez les définitions de ces fonctions pour votre plateforme cible dans le [référentiel](https://github.com/Azure/azure-iot-sdk-c) pour déterminer si vous devez inclure du code d’initialisation spécifique à la plateforme dans votre client.

Pour commencer à travailler avec les bibliothèques, attribuez d’abord un pointeur client IoT Hub :

```c
if ((iotHubClientHandle = 
  IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
{
    (void)printf("ERROR: iotHubClientHandle is NULL!\r\n");
}
else
{
    ...
```

Transférez une copie de la chaîne de connexion d’appareil (obtenue dans l’outil Explorateur d’appareils) vers cette fonction. Vous désignez également le protocole de communication à utiliser. Cet exemple utilise MQTT, mais AMQP et HTTPS sont également possibles.

Lorsque vous disposez d’un pointeur **IOTHUB\_CLIENT\_HANDLE** valide, vous pouvez commencer à appeler des API pour envoyer des messages vers IoT Hub et en recevoir de sa part.

### <a name="send-messages"></a>Envoyer des messages

L’exemple d’application configure une boucle pour envoyer des messages à votre IoT Hub. L'extrait de code suivant :

- Crée un message.
- Ajoute une propriété au message.
- Envoie un message.

Tout d’abord, créez un message :

```c
size_t iterator = 0;
do
{
    if (iterator < MESSAGE_COUNT)
    {
        sprintf_s(msgText, sizeof(msgText), "{\"deviceId\":\"myFirstDevice\",\"windSpeed\":%.2f}", avgWindSpeed + (rand() % 4 + 2));
        if ((messages[iterator].messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText))) == NULL)
        {
            (void)printf("ERROR: iotHubMessageHandle is NULL!\r\n");
        }
        else
        {
            messages[iterator].messageTrackingId = iterator;
            MAP_HANDLE propMap = IoTHubMessage_Properties(messages[iterator].messageHandle);
            (void)sprintf_s(propText, sizeof(propText), "PropMsg_%zu", iterator);
            if (Map_AddOrUpdate(propMap, "PropName", propText) != MAP_OK)
            {
                (void)printf("ERROR: Map_AddOrUpdate Failed!\r\n");
            }

            if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messages[iterator].messageHandle, SendConfirmationCallback, &messages[iterator]) != IOTHUB_CLIENT_OK)
            {
                (void)printf("ERROR: IoTHubClient_LL_SendEventAsync..........FAILED!\r\n");
            }
            else
            {
                (void)printf("IoTHubClient_LL_SendEventAsync accepted message [%d] for transmission to IoT Hub.\r\n", (int)iterator);
            }
        }
    }
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1);

    iterator++;
} while (g_continueRunning);
```

À chaque fois que vous envoyez un message, vous spécifiez une référence à une fonction de rappel invoquée lors de l’envoi des données. Dans cet exemple, la fonction de rappel est appelée **SendConfirmationCallback**. L’extrait de code suivant illustre cette fonction de rappel :

```c
static void SendConfirmationCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    EVENT_INSTANCE* eventInstance = (EVENT_INSTANCE*)userContextCallback;
    (void)printf("Confirmation[%d] received for message tracking id = %zu with result = %s\r\n", callbackCounter, eventInstance->messageTrackingId, MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
    /* Some device specific action code goes here... */
    callbackCounter++;
    IoTHubMessage_Destroy(eventInstance->messageHandle);
}
```

Notez l’appel de la fonction **IoTHubMessage\_Destroy** quand vous en avez terminé avec le message. Cette fonction libère les ressources affectées au moment de la création du message.

### <a name="receive-messages"></a>Recevoir des messages

La réception d’un message est une opération asynchrone. Tout d’abord, vous enregistrez le rappel à appeler lorsque le périphérique reçoit un message :

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext) != IOTHUB_CLIENT_OK)
{
    (void)printf("ERROR: IoTHubClient_LL_SetMessageCallback..........FAILED!\r\n");
}
else
{
    (void)printf("IoTHubClient_LL_SetMessageCallback...successful.\r\n");
    ...
```

Le dernier paramètre est un pointeur vide vers ce que vous souhaitez. Dans l’exemple, il s’agit d’un pointeur vers un entier, mais il peut s’agir d’un pointeur vers une structure de données plus complexe. Ce paramètre permet à la fonction de rappel de fonctionner sur l’état partagé avec l’appelant de cette fonction.

Lorsque le périphérique reçoit un message, la fonction de rappel enregistrée est appelée. Cette fonction de rappel récupère :

* L’ID de message et l’ID de corrélation du message.
* Le contenu du message.
* Toutes les propriétés personnalisées du message.

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    int* counter = (int*)userContextCallback;
    const char* buffer;
    size_t size;
    MAP_HANDLE mapProperties;
    const char* messageId;
    const char* correlationId;

    // Message properties
    if ((messageId = IoTHubMessage_GetMessageId(message)) == NULL)
    {
        messageId = "<null>";
    }

    if ((correlationId = IoTHubMessage_GetCorrelationId(message)) == NULL)
    {
        correlationId = "<null>";
    }

    // Message content
    if (IoTHubMessage_GetByteArray(message, (const unsigned char**)&buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        (void)printf("unable to retrieve the message data\r\n");
    }
    else
    {
        (void)printf("Received Message [%d]\r\n Message ID: %s\r\n Correlation ID: %s\r\n Data: <<<%.*s>>> & Size=%d\r\n", *counter, messageId, correlationId, (int)size, buffer, (int)size);
        // If we receive the work 'quit' then we stop running
        if (size == (strlen("quit") * sizeof(char)) && memcmp(buffer, "quit", size) == 0)
        {
            g_continueRunning = false;
        }
    }

    // Retrieve properties from the message
    mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                size_t index;

                printf(" Message Properties:\r\n");
                for (index = 0; index < propertyCount; index++)
                {
                    (void)printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                (void)printf("\r\n");
            }
        }
    }

    /* Some device specific action code goes here... */
    (*counter)++;
    return IOTHUBMESSAGE_ACCEPTED;
}
```

Utilisez la fonction **IoTHubMessage\_GetByteArray** pour récupérer le message, qui, dans cet exemple, est une chaîne.

### <a name="uninitialize-the-library"></a>Désinitialiser la bibliothèque

Quand vous avez terminé d’envoyer des événements et de recevoir des messages, vous pouvez annuler l’initialisation de la bibliothèque IoT. Pour ce faire, lancez l'appel de fonction suivant :

```c
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

Cet appel permet de libérer les ressources allouées précédemment par la fonction **IoTHubClient\_CreateFromConnectionString**.

Comme vous pouvez le voir, il est facile d’envoyer et de recevoir des messages avec la bibliothèque **IoTHubClient**. La bibliothèque traite les détails de la communication avec IoT Hub, notamment le protocole à utiliser (du point de vue du développeur, il s’agit d’une option de configuration simple).

La bibliothèque **IoTHubClient** offre également un contrôle précis de la manière de sérialiser les données que votre appareil envoie à IoT Hub. Dans certains cas, ce niveau de contrôle est un avantage, mais dans d’autres cas, il s’agit d’un détail d’implémentation qui ne vous concerne pas. Si tel est le cas, vous pouvez envisager d’utiliser la bibliothèque **serializer**, qui est décrite dans la prochaine section.

## <a name="use-the-serializer-library"></a>Utiliser la bibliothèque du sérialiseur

Sur le plan conceptuel, la bibliothèque **serializer** se trouve au-dessus de la bibliothèque **IoTHubClient** dans le Kit de développement logiciel (SDK). Elle utilise la bibliothèque **IoTHubClient** pour la communication sous-jacente avec IoT Hub, mais ajoute des fonctions de modélisation qui soulagent le développeur de la charge que représente le traitement de la sérialisation de message. Le fonctionnement de la bibliothèque est mieux illustré par un exemple.

Dans le dossier **serializer** du référentiel [azure-iot-sdk-c](https://github.com/Azure/azure-iot-sdk-c) se trouve un dossier **samples** (exemples) contenant une application appelée **simplesample\_mqtt**. La version Windows de cet exemple inclut la solution Visual Studio suivante :

  ![Exemple de solution Visual Studio pour mqtt](./media/iot-hub-device-sdk-c-intro/simplesample_mqtt.png)

> [!NOTE]
> Si Visual Studio vous demande de recibler le projet vers la dernière version, acceptez l’invite.

Comme l’exemple précédent, celui-ci contient plusieurs packages NuGet :

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.IoTHub.Serializer
* Microsoft.Azure.umqtt

Vous avez vu la plupart de ces packages dans l’exemple précédent, mais **Microsoft.Azure.IoTHub.Serializer** est nouveau. Ce package est obligatoire lorsque vous utilisez la bibliothèque **serializer**.

L’implémentation de l’exemple d’application est disponible dans le fichier **iothub_client\_samples\_iothub_convenience_sample**.

Les sections suivantes vous guident à travers les éléments clés de cet exemple.

### <a name="initialize-the-library"></a>Initialiser la bibliothèque

Pour commencer à travailler avec la bibliothèque **serializer**, appelez les API d’initialisation :

```c
if (serializer_init(NULL) != SERIALIZER_OK)
{
    (void)printf("Failed on serializer_init\r\n");
}
else
{
    IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle = IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol);
    srand((unsigned int)time(NULL));
    int avgWindSpeed = 10;

    if (iotHubClientHandle == NULL)
    {
        (void)printf("Failed on IoTHubClient_LL_Create\r\n");
    }
    else
    {
        ContosoAnemometer* myWeather = CREATE_MODEL_INSTANCE(WeatherStation, ContosoAnemometer);
        if (myWeather == NULL)
        {
            (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
        }
        else
        {
...
```

L’appel à la fonction **serializer\_init** est un appel unique qui initialise la bibliothèque sous-jacente. Vous pouvez ensuite appeler la fonction **IoTHubClient\_LL\_CreateFromConnectionString**, qui est la même API que celle de l’exemple **IoTHubClient**. Cet appel définit la chaîne de connexion de l’appareil (c’est également avec cet appel que vous choisissez le protocole à utiliser). Cet exemple utilise MQTT comme protocole de transport, mais pourrait utiliser AMQP ou HTTPS.

Enfin, appelez la fonction **CREATE\_MODEL\_INSTANCE**. **WeatherStation** est l’espace de noms du modèle et **ContosoAnemometer**, celui du modèle lui-même. Une fois que l’instance de modèle est créée, vous pouvez l’utiliser pour commencer à envoyer et recevoir des messages. Cependant, il est important de comprendre ce qu’est un modèle.

### <a name="define-the-model"></a>Définir le modèle

Un modèle de la bibliothèque **serializer** définit les messages que votre appareil peut envoyer à IoT Hub et les messages, appelés *actions* dans le langage de la modélisation, qu’il peut recevoir. Vous définissez un modèle à l’aide d’un ensemble de macros C comme dans l’exemple d’application **iothub_client\_samples\_iothub_convenience_sample** :

```c
BEGIN_NAMESPACE(WeatherStation);

DECLARE_MODEL(ContosoAnemometer,
WITH_DATA(ascii_char_ptr, DeviceId),
WITH_DATA(int, WindSpeed),
WITH_ACTION(TurnFanOn),
WITH_ACTION(TurnFanOff),
WITH_ACTION(SetAirResistance, int, Position)
);

END_NAMESPACE(WeatherStation);
```

Les macros **BEGIN\_NAMESPACE** et **END\_NAMESPACE** se servent toutes les deux de l’espace de noms du modèle comme argument. En principe, tous les éléments compris entre ces deux macros sont la définition du ou des modèles et structures de données que les modèles utilisent.

Dans cet exemple, il existe un seul modèle appelé **ContosoAnemometer**. Ce modèle définit deux éléments de données que votre appareil peut envoyer à IoT Hub : **DeviceId** et **WindSpeed**. Il définit également trois actions (messages) que votre appareil peut recevoir : **TurnFanOn**, **TurnFanOff** et **SetAirResistance**. Chaque élément de données possède un type et chaque action a un nom (et éventuellement, un ensemble de paramètres).

Les données et les actions définis dans le modèle définissent une surface API que vous pouvez utiliser pour envoyer des messages à IoT Hub et répondre aux messages envoyés à l’appareil. Un exemple permet de mieux comprendre l’utilisation de ce modèle.

### <a name="send-messages"></a>Envoyer des messages

Ce modèle définit les données que vous pouvez envoyer à IoT Hub. Dans cet exemple, cela correspond à l’un des deux éléments de données définis à l’aide de la macro **WITH_DATA**. Plusieurs étapes sont nécessaires pour envoyer les valeurs **DeviceId** et **WindSpeed** à un IoT Hub. La première consiste à définir les données que vous souhaitez envoyer :

```c
myWeather->DeviceId = "myFirstDevice";
myWeather->WindSpeed = avgWindSpeed + (rand() % 4 + 2);
```

Le modèle que vous avez défini précédemment vous permet de définir les valeurs en définissant des membres d’un **struct**. Ensuite, sérialisez le message que vous voulez envoyer :

```c
unsigned char* destination;
size_t destinationSize;
if (SERIALIZE(&destination, &destinationSize, myWeather->DeviceId, myWeather->WindSpeed) != CODEFIRST_OK)
{
    (void)printf("Failed to serialize\r\n");
}
else
{
    sendMessage(iotHubClientHandle, destination, destinationSize);
    free(destination);
}
```

Ce code sérialise l’appareil-à-cloud vers une mémoire tampon (référencée par **destination**). Le code appelle ensuite la fonction **sendMessage** pour envoyer le message à IoT Hub :

```c
static void sendMessage(IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
{
    static unsigned int messageTrackingId;
    IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
    if (messageHandle == NULL)
    {
        printf("unable to create a new IoTHubMessage\r\n");
    }
    else
    {
        if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId) != IOTHUB_CLIENT_OK)
        {
            printf("failed to hand over the message to IoTHubClient");
        }
        else
        {
            printf("IoTHubClient accepted the message for delivery\r\n");
        }
        IoTHubMessage_Destroy(messageHandle);
    }
    messageTrackingId++;
}
```

L’avant-dernier paramètre de **IoTHubClient\_LL\_SendEventAsync** est une référence à une fonction de rappel appelée lorsque l’envoi des données a abouti. Voici la fonction de rappel dans l’exemple :

```c
void sendCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    unsigned int messageTrackingId = (unsigned int)(uintptr_t)userContextCallback;

    (void)printf("Message Id: %u Received.\r\n", messageTrackingId);

    (void)printf("Result Call Back Called! Result is: %s \r\n", MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
}
```

Le deuxième paramètre est un pointeur vers le contexte utilisateur, le même pointeur a été transmis à **IoTHubClient\_LL\_SendEventAsync**. Dans ce cas, le contexte est un simple compteur, mais il peut s’agir de tout élément que vous choisissez.

C’est tout ce qu’il y a à savoir concernant l’envoi de messages d’appareil vers cloud. Le seul sujet qu’il nous reste à aborder est le mode de réception des messages.

### <a name="receive-messages"></a>Recevoir des messages

La réception d’un message fonctionne de la même façon que les messages de la bibliothèque **IoTHubClient** . Tout d’abord, vous enregistrez une fonction de rappel de message :

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, 
  IoTHubMessage, myWeather) != IOTHUB_CLIENT_OK)
{
    printf("unable to IoTHubClient_SetMessageCallback\r\n");
}
else
{
...
```

Vous écrivez ensuite la fonction de rappel invoquée à la réception d’un message :

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    IOTHUBMESSAGE_DISPOSITION_RESULT result;
    const unsigned char* buffer;
    size_t size;
    if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        printf("unable to IoTHubMessage_GetByteArray\r\n");
        result = IOTHUBMESSAGE_ABANDONED;
    }
    else
    {
        /*buffer is not zero terminated*/
        char* temp = malloc(size + 1);
        if (temp == NULL)
        {
            printf("failed to malloc\r\n");
            result = IOTHUBMESSAGE_ABANDONED;
        }
        else
        {
            (void)memcpy(temp, buffer, size);
            temp[size] = '\0';
            EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
            result =
                (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
                (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
                IOTHUBMESSAGE_REJECTED;
            free(temp);
        }
    }
    return result;
}
```

Ce code est réutilisable. Le code est le même quelle que soit la solution. Cette fonction reçoit le message et prend en charge son acheminement vers la fonction appropriée via l’appel **d’EXECUTE\_COMMAND**. La fonction appelée à ce stade dépend de la définition des actions dans votre modèle.

Quand vous définissez une action dans votre modèle, vous devez mettre en œuvre une fonction qui est appelée au moment où l’appareil reçoit le message correspondant. Par exemple, si votre modèle définit cette action :

```c
WITH_ACTION(SetAirResistance, int, Position)
```

Définissez une fonction avec cette signature :

```c
EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
{
    (void)device;
    (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
    return EXECUTE_COMMAND_SUCCESS;
}
```

Notez que le nom de la fonction correspond au nom de l’action dans le modèle et que les paramètres de la fonction correspondent aux paramètres spécifiés pour l’action. Le premier paramètre est toujours requis et contient un pointeur vers l’instance de votre modèle.

Lorsque le périphérique reçoit un message qui correspond à cette signature, la fonction associée est appelée. Ainsi, hormis l’inclusion du code réutilisable d’ **IoTHubMessage**, recevoir des messages revient à définir une fonction simple pour chaque action définie dans votre modèle.

### <a name="uninitialize-the-library"></a>Désinitialiser la bibliothèque

Lorsque vous avez terminé d’envoyer les données et de recevoir des messages, vous pouvez annuler l’initialisation de la bibliothèque IoT :

```c
...
        DESTROY_MODEL_INSTANCE(myWeather);
    }
    IoTHubClient_LL_Destroy(iotHubClientHandle);
}
serializer_deinit();
```

Chacune de ces trois fonctions s’aligne sur les trois fonctions d’initialisation décrites précédemment. L’appel de ces API vous permet de libérer les ressources affectées au préalable.

## <a name="next-steps"></a>Étapes suivantes

Cet article a abordé les principes de base de l’utilisation des bibliothèques dans le **Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C**. Il vous a fourni suffisamment d’informations pour comprendre ce qui est inclus dans le Kit de développement logiciel (SDK), son architecture et la manière d’utiliser les exemples Windows. Le prochain article poursuit la description du kit de développement logiciel en approfondissant les explications relatives à [la bibliothèque IoTHubClient](iot-hub-device-sdk-c-iothubclient.md).

Pour en savoir plus sur le développement pour IoT Hub, consultez les [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md).

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)
