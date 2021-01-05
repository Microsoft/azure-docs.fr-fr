---
title: Azure IoT device SDK pour C - IoTHubClient | Microsoft Docs
description: Guide d’utilisation de la bibliothèque IoTHubClient dans Azure IoT device SDK pour C et de création d’applications d’appareil qui communiquent avec un IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 08/29/2017
ms.author: robinsh
ms.custom: amqp
ms.openlocfilehash: f33521dd9110d7ba6ee84650345b38c8c6a4950b
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92149140"
---
# <a name="azure-iot-device-sdk-for-c--more-about-iothubclient"></a>Kit de développement logiciel d’appareil Azure IoT pour C : en savoir plus sur IoTHubClient

[Kit de développement logiciel d’appareil Azure IoT pour C](iot-hub-device-sdk-c-intro.md) est le premier article de cette série présentant le **Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C**. Cet article explique qu’il existe deux couches architecturales dans le Kit de développement logiciel (SDK). À la base se trouve la bibliothèque **IoTHubClient** qui gère directement la communication avec IoT Hub. Il existe également la bibliothèque **sérialiseur** , qui s’ajoute à cette dernière pour fournir des services de sérialisation. Dans cet article, vous trouverez des informations supplémentaires sur la bibliothèque **IoTHubClient**.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

L’article précédent explique comment utiliser la bibliothèque **IoTHubClient** pour envoyer des événements à IoT Hub et recevoir des messages. Cet article approfondit le sujet en expliquant comment gérer de manière plus précise le *moment* de l’envoi ou de la réception des données, en vous présentant les **API de niveau inférieur**. Nous expliquerons également comment associer des propriétés à des événements (et les récupérer à partir de messages) en utilisant des fonctions de gestion de propriété de la bibliothèque **IoTHubClient** . Enfin, nous fournirons des explications supplémentaires sur les différentes façons de gérer les messages reçus d’IoT Hub.

Cet article conclut en abordant des sujets divers, notamment les informations d’identification des appareils et la manière de modifier le comportement **d’IoTHubClient** par le biais des options de configuration.

Nous allons utiliser les exemples du Kit de développement logiciel (SDK) **IoTHubClient** pour illustrer ces rubriques. Si vous souhaitez assurer le suivi, consultez les applications **iothub\_client\_sample\_http** et **iothub\_client\_sample\_amqp** incluses dans le Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C. Les détails décrits dans les sections suivantes sont illustrés dans ces exemples.

Vous trouverez [**Azure IoT device SDK pour C**](https://github.com/Azure/azure-iot-sdk-c) dans le référentiel GitHub. Vous pouvez consulter les détails de l’[API dans Référence de l’API C](/azure/iot-hub/iot-c-sdk-ref/).

## <a name="the-lower-level-apis"></a>API de niveau inférieur

L’article précédent traitait du fonctionnement de base **d’IotHubClient** dans le contexte de l’application **iothub\_client\_sample\_amqp**. Par exemple, il expliquait comment initialiser la bibliothèque à l’aide de ce code.

```C
IOTHUB_CLIENT_HANDLE iotHubClientHandle;
iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);
```

Il décrivait également la façon d’envoyer des événements à l’aide de cet appel de fonction.

```C
IoTHubClient_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message);
```

Cet article expliquait également comment recevoir des messages en enregistrant une fonction de rappel.

```C
int receiveContext = 0;
IoTHubClient_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext);
```

L’article a également montré comment libérer les ressources à l’aide du code suivant.

```C
IoTHubClient_Destroy(iotHubClientHandle);
```

Il existe des fonctions d’accompagnement pour chacune de ces API :

* IoTHubClient\_LL\_CreateFromConnectionString
* IoTHubClient\_LL\_SendEventAsync
* IoTHubClient\_LL\_SetMessageCallback
* IoTHubClient\_LL\_Destroy

Ces fonctions incluent toutes **LL** dans le nom d’API. Outre la partie **LL** du nom, les paramètres de chacune de ces fonctions sont identiques aux éléments non-LL équivalents. Toutefois, le comportement de ces fonctions diffère sur un point important.

Lorsque vous appelez **IoTHubClient\_CreateFromConnectionString**, les bibliothèques sous-jacentes créent un thread qui s’exécute en arrière-plan. Ce thread envoie les événements vers IoT Hub et reçoit les messages émanant d’IoT Hub. Aucun thread de ce type n’est créé lorsque vous travaillez avec des API **LL**. La création du thread d’arrière-plan est pratique pour le développeur. Vous n’avez pas à vous soucier d’envoyer des événements et de recevoir des messages d’IoT Hub. Ces opérations s’exécutent automatiquement en arrière-plan. En revanche, l’API **LL** vous offre un contrôle explicite sur la communication avec IoT Hub, si nécessaire.

Pour mieux comprendre ce concept, examinons un exemple :

Lorsque vous appelez **IoTHubClient\_SendEventAsync**, vous mettez en fait un événement en mémoire tampon. Le thread d’arrière-plan créé lorsque vous appelez **IoTHubClient\_CreateFromConnectionString** surveille en continu ce tampon et envoie toutes les données qu’il contient à IoT Hub. Cela se produit en arrière-plan, pendant que le thread principal exécute une autre tâche.

De même, quand vous enregistrez une fonction de rappel pour les messages à l’aide **d’IoTHubClient\_SetMessageCallback**, vous indiquez au Kit de développement logiciel (SDK) que le thread d’arrière-plan doit appeler la fonction de rappel quand un message est reçu, indépendamment du thread principal.

Les API **LL** ne créent pas un thread d’arrière-plan. Au lieu de cela, une nouvelle API doit être appelée explicitement pour envoyer et recevoir des données de la part d’IoT Hub. Cette opération est illustrée dans l’exemple suivant.

L’application **iothub\_client\_sample\_http** incluse dans le Kit de développement logiciel (SDK) présente les API de niveau inférieur. Dans cet exemple, nous envoyons des événements à IoT Hub avec un code similaire à celui qui suit :

```C
EVENT_INSTANCE message;
sprintf_s(msgText, sizeof(msgText), "Message_%d_From_IoTHubClient_LL_Over_HTTP", i);
message.messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText));

IoTHubClient_LL_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message)
```

Les trois premières lignes créent le message et la dernière ligne envoie l’événement. Cependant, comme indiqué précédemment l’envoi de l’événement indique simplement que les données sont placées dans une mémoire tampon. Lorsque nous appelons **IoTHubClient\_LL\_SendEventAsync**, rien n’est transmis sur le réseau. Pour entrer réellement les données sur IoT Hub, vous devez appeler **IoTHubClient\_LL\_DoWork**, comme dans cet exemple :

```C
while (1)
{
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(100);
}
```

Ce code (tiré de l’application **iothub\_client\_sample\_http**) appelle à plusieurs reprises **IoTHubClient\_LL\_DoWork**. Chaque fois que la fonction **IoTHubClient\_LL\_DoWork** est appelée, elle envoie des événements de la mémoire tampon vers IoT Hub et récupère ensuite un message en file d’attente envoyé à l’appareil. Dans ce dernier cas, si nous avons inscrit une fonction de rappel pour les messages, le rappel est invoqué (en supposant que des messages sont en file d’attente). Cette fonction de rappel a été enregistrée avec le code qui suit :

```C
IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext)
```

La raison pour laquelle la fonction **IoTHubClient\_LL\_DoWork** est souvent appelée dans une boucle est qu’à chaque appel, elle envoie *certains* événements mis en mémoire tampon à IoT Hub et récupère le message en file d’attente *suivant* destiné à l’appareil. Chaque appel ne garantit pas que tous les événements en mémoire tampon sont envoyés ou que tous les messages en file d’attente sont récupérés. Si vous souhaitez envoyer tous les événements en mémoire tampon, puis continuer avec un autre traitement, vous pouvez remplacer cette boucle par un code similaire à celui qui suit :

```C
IOTHUB_CLIENT_STATUS status;

while ((IoTHubClient_LL_GetSendStatus(iotHubClientHandle, &status) == IOTHUB_CLIENT_OK) && (status == IOTHUB_CLIENT_SEND_STATUS_BUSY))
{
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(100);
}
```

Ce code appelle **IoTHubClient\_LL\_DoWork** jusqu’à ce que tous les événements placés en mémoire tampon aient été envoyés à IoT Hub. Notez que cela ne signifie pas non plus que tous les messages en file d’attente ont été reçus. Cela est en partie dû au fait que le contrôle de « tous » les messages n’est pas une action déterminante. Que se passe-t-il si vous récupérez « tous » les messages, mais qu’un autre est envoyé à l’appareil immédiatement après ? Une bonne façon de traiter cette possibilité consiste à programmer un délai d’attente. Par exemple, la fonction de rappel de message peut réinitialiser une minuterie à chaque fois qu’elle est appelée. Vous pouvez ensuite rédiger un programme permettant de poursuivre le traitement si, par exemple, aucun message n’a été reçu au cours des *X* dernières secondes.

Lorsque vous avez terminé de réceptionner les événements et les messages, assurez-vous d’appeler la fonction correspondante pour nettoyer les ressources.

```C
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

En fait, il existe un seul ensemble d’API pour envoyer et recevoir des données avec un thread d’arrière-plan et un autre ensemble d’API qui fait la même chose sans thread d’arrière-plan. Un grand nombre de développeurs préfèrent les API non LL, mais les API de niveau inférieur sont utiles lorsque le développeur souhaite obtenir un contrôle explicite sur les transmissions réseau. Par exemple, certains appareils recueillent des données au fil du temps et n’enregistrent que les événements à des intervalles spécifiés (par exemple, une fois par heure ou une fois par jour). Les API de niveau inférieur vous donnent la possibilité de contrôler explicitement l’envoi et la réception depuis IoT Hub. D’autres préfèrent la simplicité qu’offrent les API de niveau inférieur. Tout se produit sur le thread principal plutôt que sur les travaux qui s’exécutent en arrière-plan.

Quel que soit le modèle que vous choisissez, assurez-vous d’être cohérent avec les API que vous utilisez. Si vous commencez par appeler **IoTHubClient\_LL\_CreateFromConnectionString**, veillez à utiliser uniquement les API de niveau inférieur correspondantes pour les travaux ultérieurs :

* IoTHubClient\_LL\_SendEventAsync
* IoTHubClient\_LL\_SetMessageCallback
* IoTHubClient\_LL\_Destroy
* IoTHubClient\_LL\_DoWork

L’inverse est également vrai. Si vous démarrez avec **IoTHubClient\_CreateFromConnectionString**, utilisez les API non LL pour tout traitement supplémentaire.

Dans le Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C, consultez l’application **iothub\_client\_sample\_http** pour obtenir un exemple complet des API de niveau inférieur. L’application **iothub\_client\_sample\_amqp** peut être référencée pour un exemple complet des API non LL.

## <a name="property-handling"></a>Gestion des propriétés

Jusqu’à présent, lorsque nous avons décrit l’envoi de données, nous avons fait référence au corps du message. Considérez par exemple le code suivant :

```C
EVENT_INSTANCE message;
sprintf_s(msgText, sizeof(msgText), "Hello World");
message.messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText));
IoTHubClient_LL_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message)
```

Cet exemple envoie un message à IoT Hub avec le texte « Hello World ». Cependant, IoT Hub permet également de joindre des propriétés à chaque message. Les propriétés sont des paires nom/valeur pouvant être jointes au message. Par exemple, nous pouvons modifier le code précédent pour attacher une propriété au message :

```C
MAP_HANDLE propMap = IoTHubMessage_Properties(message.messageHandle);
sprintf_s(propText, sizeof(propText), "%d", i);
Map_AddOrUpdate(propMap, "SequenceNumber", propText);
```

Nous allons commencer en appelant **IoTHubMessage\_Properties** et en lui transmettant le pointeur de notre message. Nous obtenons une référence **MAP\_HANDLE** qui nous permet de commencer à ajouter des propriétés. Cette opération s’effectue en appelant **Map\_AddOrUpdate** qui accepte une référence MAP\_HANDLE, le nom de propriété et la valeur de propriété. Avec cette API, nous pouvons ajouter autant de propriétés que nous le souhaitons.

Lorsque l’événement est lu à partir **d’Event Hubs**, le récepteur peut énumérer les propriétés et récupérer les valeurs correspondantes. Par exemple, dans .NET, cette opération s’effectue grâce à un accès à la [collection de propriétés sur l’objet EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata).

Dans l’exemple précédent, nous allons rattacher des propriétés à un événement que nous envoyons à IoT Hub. Les propriétés peuvent également être jointes à des messages provenant d’IoT Hub. Si vous souhaitez récupérer les propriétés d’un message, nous pouvons utiliser un code du type qui suit dans notre fonction de rappel de message :

```C
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    . . .

    // Retrieve properties from the message
    MAP_HANDLE mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                printf("Message Properties:\r\n");
                for (size_t index = 0; index < propertyCount; index++)
                {
                    printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                printf("\r\n");
            }
        }
    }

    . . .
}
```

L’appel à **IoTHubMessage\_Properties** renvoie la référence **MAP\_HANDLE**. Cette référence est ensuite transmise à **Map\_GetInternals** pour obtenir une référence à un tableau de paires nom/valeur (ainsi que le nombre des propriétés). À ce stade, il est très simple d’énumérer les propriétés pour obtenir les valeurs souhaitées.

Vous n’êtes pas obligé d’utiliser des propriétés dans votre application. Mais si vous devez les définir sur les événements ou les récupérer à partir de messages, la bibliothèque **IoTHubClient** facilite cette opération.

## <a name="message-handling"></a>Gestion des messages

Comme expliqué précédemment, lorsque les messages arrivent d’IoT Hub, la bibliothèque **IoTHubClient** répond en appelant une fonction de rappel enregistrée. Un des paramètres de retour de cette fonction mérite cependant quelques explications supplémentaires. Voici un extrait de la fonction de rappel de l’exemple d’application **iothub\_client\_sample\_http** :

```C
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    . . .
    return IOTHUBMESSAGE_ACCEPTED;
}
```

Notez que le type de retour est **IOTHUBMESSAGE\_DISPOSITION\_RESULT** et, dans ce cas particulier, c’est le message **IOTHUBMESSAGE\_ACCEPTED** qui est renvoyé. Il existe d’autres valeurs que nous pouvons renvoyer à partir de cette fonction afin de modifier la réaction de la bibliothèque **IoTHubClient** au rappel de message. Les options proposées sont les suivantes.

* **IOTHUBMESSAGE\_ACCEPTED** : le message a été traité avec succès. La bibliothèque **IoTHubClient** ne rappellera pas la fonction de rappel avec le même message.

* **IOTHUBMESSAGE\_REJECTED** : le message n’a pas été traité, et ne devrait pas l’être à l’avenir. La bibliothèque **IoTHubClient** n’appellera pas la fonction de rappel avec le même message.

* **IOTHUBMESSAGE\_ABANDONED** : le message n’a pas été traité correctement, mais la bibliothèque **IoTHubClient** doit de nouveau invoquer la fonction de rappel, en utilisant le même message.

Pour les deux premiers codes de retour, la bibliothèque **IoTHubClient** envoie à IoT Hub un message indiquant que le message doit être supprimé de la file d’attente de l’appareil et non remis à nouveau. La conséquence est identique (le message est supprimé de la file d’attente de l’appareil), mais le fait que le message ait été accepté ou rejeté reste dans les enregistrements.  L’enregistrement de cette différence est utile aux expéditeurs du message qui peuvent écouter le retour et découvrir si un appareil a accepté ou rejeté un message particulier.

Dans ce dernier cas, un message est également envoyé à IoT Hub, mais il indique que le message doit être à nouveau remis. En général, si vous rencontrez une erreur mais que vous souhaitez essayer de traiter de nouveau le message, vous devez abandonner le message. En revanche, le rejet d’un message se justifie lorsque vous rencontrez une erreur irrécupérable (ou si vous décidez simplement que vous ne voulez pas traiter le message).

Dans tous les cas, vous devez connaître les différents codes de retour de façon à choisir le comportement que vous souhaitez obtenir de la bibliothèque **IoTHubClient** .

## <a name="alternate-device-credentials"></a>Autres informations d’identification d’appareil

Comme nous l’avons expliqué plus haut, la première chose à faire lorsqu’on travaille avec la bibliothèque **IoTHubClient** consiste à obtenir une référence **IOTHUB\_CLIENT\_HANDLE** avec un appel tel que le suivant :

```C
IOTHUB_CLIENT_HANDLE iotHubClientHandle;
iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);
```

Les arguments de **IoTHubClient\_CreateFromConnectionString** correspondent à la chaîne de connexion de l’appareil et un paramètre indique le protocole que nous allons utiliser pour communiquer avec IoT Hub. Le format de la chaîne de connexion de l’appareil se présente comme suit :

```C
HostName=IOTHUBNAME.IOTHUBSUFFIX;DeviceId=DEVICEID;SharedAccessKey=SHAREDACCESSKEY
```

Cette chaîne contient quatre éléments d’information : le nom du hub IoT Hub, le suffixe IoT Hub, l’ID de l’appareil et clé d’accès partagé. Vous obtenez le nom de domaine complet (FQDN) d’un IoT Hub lorsque vous créez une instance d’IoT Hub dans le portail Azure. Vous obtenez ainsi le nom d’IoT Hub (la première partie du nom de domaine complet) et le suffixe d’IoT Hub (le reste du nom de domaine complet). L’ID d’appareil et la clé d’accès partagé vous sont fournis lorsque vous inscrivez votre appareil avec IoT Hub (comme le décrit [l’article précédent](iot-hub-device-sdk-c-intro.md)).

**IoTHubClient\_CreateFromConnectionString** vous propose une façon d’initialiser la bibliothèque. Vous pouvez également créer un élément **IOTHUB\_CLIENT\_HANDLE** à l’aide de ces paramètres individuels plutôt que de la chaîne de connexion de l’appareil. Vous pouvez y parvenir avec le code suivant :

```C
IOTHUB_CLIENT_CONFIG iotHubClientConfig;
iotHubClientConfig.iotHubName = "";
iotHubClientConfig.deviceId = "";
iotHubClientConfig.deviceKey = "";
iotHubClientConfig.iotHubSuffix = "";
iotHubClientConfig.protocol = HTTP_Protocol;
IOTHUB_CLIENT_HANDLE iotHubClientHandle = IoTHubClient_LL_Create(&iotHubClientConfig);
```

Il effectue la même opération que **IoTHubClient\_CreateFromConnectionString**.

Il semble évident que vous préférerez utiliser **IoTHubClient\_CreateFromConnectionString** plutôt que la méthode d’initialisation plus détaillée. Cependant, gardez à l’esprit que lorsque vous enregistrez un appareil dans IoT Hub, vous obtenez un ID d’appareil et une clé d’appareil (pas une chaîne de connexion). L’outil du Kit de développement logiciel (SDK) *Explorateur d’appareils* présenté dans [l’article précédent](iot-hub-device-sdk-c-intro.md) utilise des bibliothèques du **Kit de développement logiciel (SDK) de service Azure IoT** pour créer la chaîne de connexion à l’appareil à partir de l’ID d’appareil, de la clé d’appareil et du nom d’hôte IoT Hub. Ainsi, il peut être préférable d’appeler **IoTHubClient\_LL\_Create**, car cela vous évite de générer une chaîne de connexion. Utilisez la méthode adéquate.

## <a name="configuration-options"></a>Options de configuration

Jusqu’ici, tout ce qui est décrit dans la façon dont la bibliothèque **IoTHubClient** fonctionne reflète son comportement par défaut. Cependant, vous pouvez définir quelques options que vous pouvez définir pour modifier le fonctionnement de la bibliothèque. Vous pouvez procéder en exploitant l’API **IoTHubClient\_LL\_SetOption**. Examinez cet exemple :

```C
unsigned int timeout = 30000;
IoTHubClient_LL_SetOption(iotHubClientHandle, "timeout", &timeout);
```

Quelques options sont couramment utilisées :

* **SetBatching** (bool) : si cette option a la valeur **true**, les données transmises à IoT Hub sont envoyées par lots. Si elle a la valeur **false**, les messages sont envoyés individuellement. La valeur par défaut est **false**. Le traitement par lot via AMQP/WS-AMQP, ainsi que l’ajout de propriétés système sur les messages D2C, est pris en charge.

* **Timeout** (unsigned int) : cette valeur est exprimée en millisecondes. Si l’envoi d’une requête HTTPS ou la réception d’une réponse prend davantage de temps, la connexion expire.

L’option de traitement par lot est importante. Par défaut, la bibliothèque intègre des événements individuellement (un événement unique correspond à ce que vous transmettez à **IoTHubClient\_LL\_SendEventAsync**). Mais si l’option de traitement par lot est définie sur **true**, la bibliothèque collecte autant d’événements que possible (jusqu’à atteindre la taille de message maximale acceptée par IoT Hub).  Le lot d’événements est envoyé à IoT Hub par le biais d’un seul appel HTTPS (les événements individuels sont regroupés dans un tableau JSON). L’activation de l’option de traitement par lot permet d’obtenir des gains de performance, car vous réduisez le nombre d’allers-retours sur le réseau. Elle réduit également considérablement la bande passante, car vous envoyez un seul ensemble d’en-têtes HTTPS avec un lot d’événements, plutôt qu’un ensemble d’en-têtes pour chaque événement. À moins d’avoir une raison particulière de faire autrement, vous devrez activer le traitement par lot.

## <a name="next-steps"></a>Étapes suivantes

Cet article décrit en détail le comportement de la bibliothèque **IoTHubClient** se trouvant dans le **Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C**. Ces informations doivent vous permettre de bien comprendre les fonctionnalités de la bibliothèque **IoTHubClient**. Le deuxième article de cette série est [Kit de développement logiciel (SDK) d’appareil Azure IoT (Azure IoT device SDK) pour C - Sérialiseur](iot-hub-device-sdk-c-serializer.md), qui fournit des détails similaires sur la bibliothèque de **sérialiseur**.

Pour en savoir plus sur le développement pour IoT Hub, consultez les [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md).

Pour explorer davantage les capacités d’IoT Hub, consultez l’article [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md).