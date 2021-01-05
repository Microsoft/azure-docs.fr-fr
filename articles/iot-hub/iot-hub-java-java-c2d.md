---
title: Messages cloud-à-appareil avec Azure IoT Hub (Java) | Microsoft Docs
description: Envoi de messages cloud-à-appareil vers un appareil depuis un Azure IoT Hub à l’aide des kits de développement logiciel Azure IoT pour Java. Vous modifiez une application d’appareil simulé pour recevoir des messages cloud-à-appareil et modifiez une application principale pour envoyer des messages cloud-à-appareil.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 06/28/2017
ms.custom:
- amqp
- mqtt
- devx-track-java
ms.openlocfilehash: 763b9e05adc07c02265dbb511c073b42df44ea95
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92146856"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-java"></a>Envoi de messages cloud à appareil avec IoT Hub (Java)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

Azure IoT Hub est un service entièrement géré qui permet d’autoriser des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils et un serveur principal de solution. Le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-java.md) explique comment créer un hub IoT, y provisionner une identité d’appareil et coder une application d’appareil simulé qui envoie des messages appareil-à-cloud.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Ce tutoriel est basé sur [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-java.md). Il vous montre comment effectuer les étapes suivantes :

* À partir du serveur principal de votre application, envoyez des messages cloud-à-appareil vers un appareil unique via IoT Hub.

* Recevez des messages cloud-à-appareil sur un appareil.

* À partir du back-end de votre solution, demandez l’accusé de réception (*commentaires*) pour les messages envoyés à un appareil depuis IoT Hub.

Vous trouverez des informations supplémentaires sur les messages cloud-à-appareil dans le [Guide du développeur IoT Hub](iot-hub-devguide-messaging.md).

À la fin de ce didacticiel, vous exécutez deux applications de console Java :

* **simulated-device**, une version modifiée de l’application créée dans [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-java.md), qui se connecte à votre hub IoT et reçoit des messages cloud-à-appareil.

* **send-c2d-messages**, qui envoie un message cloud-à-appareil à l’application d’appareil simulé via IoT Hub, puis reçoit son accusé de réception.

> [!NOTE]
> IoT Hub offre la prise en charge de Kits de développement logiciel (SDK) pour plusieurs plateformes d’appareils et plusieurs langages (notamment C, Java, Python et Javascript) par le biais des Kits Azure IoT device SDK. Pour obtenir des instructions détaillées sur la façon de connecter votre appareil au code de ce didacticiel, et à Azure IoT Hub de manière générale, consultez le [Centre de développement Azure IoT](https://azure.microsoft.com/develop/iot).

## <a name="prerequisites"></a>Prérequis

* Une version opérationnelle complète du guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil vers un hub IoT](quickstart-send-telemetry-java.md) ou du didacticiel [Configurer le routage des messages avec IoT Hub](tutorial-routing.md).

* [Java SE Development Kit 8](/java/azure/jdk/?view=azure-java-stable). Veillez à sélectionner **Java 8** sous **Prise en charge à long terme** pour accéder aux téléchargements du kit JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d'informations sur les différentes façons de contourner ce problème, consultez [Se connecter à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recevoir des messages dans l’application d’appareil simulé

Dans cette section, vous modifiez l’application d’appareil simulé créée dans [Envoyer des données de télémétrie d’un appareil à un IoT Hub](quickstart-send-telemetry-java.md) pour recevoir des messages cloud-à-appareil en provenance du hub IoT.

1. À l’aide d’un éditeur de texte, ouvrez le fichier simulated-device\src\main\java\com\mycompany\app\App.java.

2. Ajoutez la classe **MessageCallback** suivante comme classe imbriquée à l'intérieur de la classe **App**. La méthode **execute** est appelée lorsque l’appareil reçoit un message d’IoT Hub. Dans cet exemple, l’appareil notifie toujours l’IoT Hub qu’il a terminé le message :

    ```java
    private static class AppMessageCallback implements MessageCallback {
      public IotHubMessageResult execute(Message msg, Object context) {
        System.out.println("Received message from hub: "
          + new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET));

        return IotHubMessageResult.COMPLETE;
      }
    }
    ```

3. Modifiez la méthode **main** pour créer une instance **AppMessageCallback** et appelez la méthode **setMessageCallback** avant l’ouverture du client comme suit :

    ```java
    client = new DeviceClient(connString, protocol);

    MessageCallback callback = new AppMessageCallback();
    client.setMessageCallback(callback, null);
    client.open();
    ```

4. Pour générer l’application **simulated-device** à l’aide de Maven, exécutez la commande suivante à l’invite de commandes dans le dossier simulated-device :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

La méthode `execute` dans la classe `AppMessageCallback` retourne `IotHubMessageResult.COMPLETE`. Cela notifie IoT Hub que le message a été traité avec succès et peut être supprimé en toute sécurité de la file d’attente de l’appareil. L’appareil doit retourner cette valeur quand son traitement se termine correctement, quel que soit le protocole utilisé.

Avec AMQP et HTTPS, mais pas MQTT, l’appareil peut également :

* abandonner un message - IoT Hub conserve alors le message dans la file d’attente de l’appareil pour un traitement ultérieur ;
* rejeter un message, ce qui le supprime définitivement de la file d’attente de l’appareil.

S’il se produit un événement qui empêche l’appareil de traiter, d’abandonner ou de rejeter le message, IoT Hub le met à nouveau en file d’attente après un délai d’attente déterminé. C’est la raison pour laquelle la logique de traitement des messages de l’application pour périphérique doit être *idempotente* pour qu’un message identique reçu plusieurs fois produise le même résultat.

Pour plus d'informations sur la façon dont IoT Hub traite les messages cloud-à-appareil, y compris sur le cycle de vie des messages cloud-à-appareil, consultez [Envoyer des messages cloud-à-appareil à partir d'un hub IoT](iot-hub-devguide-messages-c2d.md).

> [!NOTE]
> Si vous utilisez HTTPS plutôt que MQTT ou AMQP comme moyen de transport, l'instance **DeviceClient** ne vérifie pas très souvent les messages provenant d'IoT Hub (au minimum toutes les 25 minutes). Pour plus d'informations sur les différences de prise en charge entre MQTT, AMQP et HTTPS, consultez [Conseils sur les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md) et [Choisir un protocole de communication](iot-hub-devguide-protocols.md).

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour envoyer des messages cloud-à-appareil via l’IoT Hub que vous avez créé dans [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-java.md). Pour envoyer des messages cloud-à-appareil, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envoi d’un message cloud vers appareil

Dans cette section, vous créez une application de console Java qui envoie des messages cloud-à-appareil à l’application de l’appareil simulé. Vous avez besoin de l’ID de l’appareil que vous avez ajouté dans le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-java.md). Vous avez également besoin de la chaîne de connexion de l’IoT Hub que vous avez copiée précédemment dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string).

1. Créez un projet Maven nommé **send-c2d-messages** à l’aide de la commande ci-dessous, à l’invite de commandes. Il s’agit d’une commande unique et longue :

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=send-c2d-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. À l’invite de commandes, accédez au nouveau dossier send-c2d-messages.

3. Dans un éditeur de texte, ouvrez le fichier pom.xml dans le dossier send-c2d-messages et ajoutez la dépendance suivante au nœud **dependencies** . L’ajout de la dépendance vous permet d’utiliser le package **iothub-java-service-client** dans votre application pour communiquer avec votre service d’IoT Hub :

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Vous pouvez rechercher la dernière version de **iot-service-client** avec la [recherche Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Enregistrez et fermez le fichier pom.xml.

5. À l’aide d’un éditeur de texte, ouvrez le fichier send-c2d-messages\src\main\java\com\mycompany\app\App.java.

6. Ajoutez les instructions **import** suivantes au fichier :

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. Ajoutez les variables de niveau classe suivantes à la classe **App** en remplaçant **{yourhubconnectionstring}** et **{yourdeviceid}** par les valeurs que vous avez notées précédemment :

    ```java
    private static final String connectionString = "{yourhubconnectionstring}";
    private static final String deviceId = "{yourdeviceid}";
    private static final IotHubServiceClientProtocol protocol =    
        IotHubServiceClientProtocol.AMQPS;
    ```

8. Remplacez la méthode **main** par le code suivant. Ce code établit la connexion à votre IoT Hub, envoie un message à votre appareil, puis attend un accusé de réception indiquant que l’appareil a reçu le message et l’a traité :

    ```java
    public static void main(String[] args) throws IOException,
        URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(
        connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();
        FeedbackReceiver feedbackReceiver = serviceClient
          .getFeedbackReceiver();
        if (feedbackReceiver != null) feedbackReceiver.open();

        Message messageToSend = new Message("Cloud to device message.");
        messageToSend.setDeliveryAcknowledgement(DeliveryAcknowledgement.Full);

        serviceClient.send(deviceId, messageToSend);
        System.out.println("Message sent to device");

        FeedbackBatch feedbackBatch = feedbackReceiver.receive(10000);
        if (feedbackBatch != null) {
          System.out.println("Message feedback received, feedback time: "
            + feedbackBatch.getEnqueuedTimeUtc().toString());
        }

        if (feedbackReceiver != null) feedbackReceiver.close();
        serviceClient.close();
      }
    }
    ```

    > [!NOTE]
    > Par souci de simplicité, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, une interruption exponentielle), comme indiqué dans l’article [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).

9. Pour générer l’application **simulated-device** à l’aide de Maven, exécutez la commande suivante à l’invite de commandes dans le dossier simulated-device :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

1. À l’invite de commandes du dossier simulated-device, exécutez la commande suivante pour commencer à envoyer la télémétrie à votre IoT Hub et écouter les messages cloud-à-appareil envoyés depuis ce dernier :

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Exécution de l’application de périphérique simulé](./media/iot-hub-java-java-c2d/receivec2d.png)

2. À l’invite de commandes dans le dossier send-c2d-messages, exécutez la commande suivante pour envoyer un message cloud-à-appareil, puis attendez de recevoir l’accusé de réception :

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Exécutez la commande pour envoyer le message cloud-à-appareil](media/iot-hub-java-java-c2d/sendc2d.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à envoyer et recevoir des messages cloud-à-appareil.

Pour voir des exemples de solutions de bout en bout qui utilisent IoT Hub, consultez [Accélérateurs de solution Azure IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Pour en savoir plus sur le développement de solutions avec IoT Hub, consultez le [Guide du développeur d’IoT Hub](iot-hub-devguide.md).