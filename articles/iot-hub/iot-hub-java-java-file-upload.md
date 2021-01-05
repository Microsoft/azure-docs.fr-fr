---
title: Charger des fichiers sur Azure IoT Hub à partir d’appareils en Java | Microsoft Docs
description: Guide pratique pour charger des fichiers sur le cloud à partir d’un appareil avec Azure IoT device SDK pour Java. Les fichiers téléchargés sont stockés dans un conteneur d’objets blob de stockage Azure.
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
ms.openlocfilehash: 2a02dc94c0dc0939e6c1eef624fc6fe65fb98ac6
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147018"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-java"></a>Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Java)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Ce tutoriel s’appuie sur le code du tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-java-java-c2d.md) pour montrer comment utiliser les [fonctions de chargement de fichiers d’IoT Hub](iot-hub-devguide-file-upload.md) afin de charger un fichier sur le [Stockage Blob Azure](../storage/index.yml). Ce didacticiel explique les procédures suivantes :

* Fournissez en toute sécurité à un appareil un URI d’objet blob Azure pour le chargement d’un fichier.

* Utilisez les notifications de chargement de fichier IoT Hub pour déclencher le traitement du fichier dans votre serveur principal d’application.

Le guide de démarrage rapide [Envoyer des données de télémétrie à partir d’un appareil à un hub IoT](quickstart-send-telemetry-java.md) et le tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-java-java-c2d.md) illustrent les fonctionnalités de messages appareil-à-cloud et cloud-à-appareil de base offertes par IoT Hub. Le tutoriel [Configurer le routage des messages avec IoT Hub](tutorial-routing.md) décrit un moyen de stocker de façon fiable les messages appareil-à-cloud dans le stockage d’objets blob Azure. Toutefois, dans certains scénarios, vous ne pouvez pas facilement mapper les données que vos appareils envoient dans des messages appareil-à-cloud relativement petits et acceptés par IoT Hub. Par exemple :

* Fichiers volumineux qui contiennent des images
* Vidéos
* Données de vibration échantillonnées à une fréquence élevée
* Un certain type de données prétraitées.

Ces fichiers sont généralement traités par lot dans le cloud à l’aide d’outils tels que [Azure Data Factory](../data-factory/introduction.md) ou de la pile [Hadoop](../hdinsight/index.yml). Lorsque vous avez besoin de charger des fichiers à partir d’un appareil, vous pouvez quand même exploiter la sécurité et la fiabilité d’IoT Hub.

À la fin de ce didacticiel, vous exécuterez deux applications de console Java :

* **simulated-device**, version modifiée de l’application créée dans le cadre du tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub]. Cette application charge un fichier de stockage à l’aide d’un URI SAP fourni par votre IoT Hub.

* **read-file-upload-notification**, qui reçoit les notifications de chargement de fichiers envoyées par le hub IoT.

> [!NOTE]
> IoT Hub prend en charge de nombreuses plateformes d’appareils et de nombreux langages (notamment C, .NET et JavaScript) par le biais des kits Azure IoT device SDK. Pour obtenir des instructions pas à pas expliquant comment connecter votre appareil à Azure IoT Hub, voir le [Centre de développement Azure IoT](https://azure.microsoft.com/develop/iot).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Prérequis

* [Java SE Development Kit 8](/java/azure/jdk/?view=azure-java-stable). Veillez à sélectionner **Java 8** sous **Prise en charge à long terme** pour accéder aux téléchargements du kit JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Charger un fichier à partir d’une application d’appareil

Dans cette section, vous allez modifier l’application pour appareils que vous avez créée dans [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-java-java-c2d.md) pour charger un fichier sur IoT Hub.

1. Copiez un fichier image dans le dossier `simulated-device` et renommez-le `myimage.png`.

2. Ouvrez le fichier `simulated-device\src\main\java\com\mycompany\app\App.java` dans un éditeur de texte.

3. Ajoutez la déclaration de variable à la classe **App** :

    ```java
    private static String fileName = "myimage.png";
    ```

4. Pour traiter les messages de rappel d’état du chargement de fichiers, ajoutez la classe imbriquée suivante à la classe **App** :

    ```java
    // Define a callback method to print status codes from IoT Hub.
    protected static class FileUploadStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to file upload for " + fileName
            + " operation with status " + status.name());
      }
    }
    ```

5. Pour charger des images sur IoT Hub, ajoutez la méthode suivante à la classe **App** :

    ```java
    // Use IoT Hub to upload a file asynchronously to Azure blob storage.
    private static void uploadFile(String fullFileName) throws FileNotFoundException, IOException
    {
      File file = new File(fullFileName);
      InputStream inputStream = new FileInputStream(file);
      long streamLength = file.length();

      client.uploadToBlobAsync(fileName, inputStream, streamLength, new FileUploadStatusCallBack(), null);
    }
    ```

6. Modifiez la méthode **main** de façon à appeler la méthode **uploadFile**, comme dans l’extrait de code suivant :

    ```java
    client.open();

    try
    {
      // Get the filename and start the upload.
      String fullFileName = System.getProperty("user.dir") + File.separator + fileName;
      uploadFile(fullFileName);
      System.out.println("File upload started with success");
    }
    catch (Exception e)
    {
      System.out.println("Exception uploading file: " + e.getCause() + " \nERROR: " + e.getMessage());
    }

    MessageSender sender = new MessageSender();
    ```

7. Utilisez la commande suivante pour générer l’application **simulated-device** et vérifiez l’absence d’erreurs :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour recevoir les messages de notification de chargement de fichiers depuis l’IoT Hub que vous avez créé dans [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-java.md). Pour recevoir les messages de notification de chargement de fichiers, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recevoir une notification de téléchargement de fichier

Dans cette section, vous allez créer une application console Java qui reçoit des messages de notification de chargement de fichiers envoyés par IoT Hub.

1. Créez un projet Maven nommé **read-file-upload-notification** en lançant la commande ci-dessous dans votre invite de commandes. Il s’agit d’une commande unique et longue :

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-file-upload-notification -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. Dans votre invite de commandes, accédez au nouveau dossier `read-file-upload-notification`.

3. Dans un éditeur de texte, ouvrez le fichier `pom.xml` dans le dossier `read-file-upload-notification` et ajoutez la dépendance suivante au nœud **dependencies**. L’ajout de la dépendance vous permet d’utiliser le package **iothub-java-service-client** dans votre application pour communiquer avec votre service d’IoT Hub :

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Vous pouvez rechercher la dernière version de **iot-service-client** avec la [recherche Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Enregistrez et fermez le fichier `pom.xml`.

5. Ouvrez le fichier `read-file-upload-notification\src\main\java\com\mycompany\app\App.java` dans un éditeur de texte.

6. Ajoutez les instructions **import** suivantes au fichier :

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    ```

7. Ajoutez les variables de niveau classe suivantes à la classe **App** . Remplacez la valeur de l’espace réservé `{Your IoT Hub connection string}` par la chaîne de connexion de l’IoT Hub que vous avez précédemment copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string) :

    ```java
    private static final String connectionString = "{Your IoT Hub connection string}";
    private static final IotHubServiceClientProtocol protocol = IotHubServiceClientProtocol.AMQPS;
    private static FileUploadNotificationReceiver fileUploadNotificationReceiver = null;
    ```

8. Pour imprimer des informations sur le chargement de fichiers dans la console, ajoutez la classe imbriquée suivante à la classe **App** :

    ```java
    // Create a thread to receive file upload notifications.
    private static class ShowFileUploadNotifications implements Runnable {
      public void run() {
        try {
          while (true) {
            System.out.println("Receive file upload notifications...");
            FileUploadNotification fileUploadNotification = fileUploadNotificationReceiver.receive();
            if (fileUploadNotification != null) {
              System.out.println("File Upload notification received");
              System.out.println("Device Id : " + fileUploadNotification.getDeviceId());
              System.out.println("Blob Uri: " + fileUploadNotification.getBlobUri());
              System.out.println("Blob Name: " + fileUploadNotification.getBlobName());
              System.out.println("Last Updated : " + fileUploadNotification.getLastUpdatedTimeDate());
              System.out.println("Blob Size (Bytes): " + fileUploadNotification.getBlobSizeInBytes());
              System.out.println("Enqueued Time: " + fileUploadNotification.getEnqueuedTimeUtcDate());
            }
          }
        } catch (Exception ex) {
          System.out.println("Exception reading reported properties: " + ex.getMessage());
        }
      }
    }
    ```

9. Pour démarrer le thread qui écoute les notifications de chargement de fichiers, ajoutez le code suivant à la méthode **main** :

    ```java
    public static void main(String[] args) throws IOException, URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();

        // Get a file upload notification receiver from the ServiceClient.
        fileUploadNotificationReceiver = serviceClient.getFileUploadNotificationReceiver();
        fileUploadNotificationReceiver.open();

        // Start the thread to receive file upload notifications.
        ShowFileUploadNotifications showFileUploadNotifications = new ShowFileUploadNotifications();
        ExecutorService executor = Executors.newFixedThreadPool(1);
        executor.execute(showFileUploadNotifications);

        System.out.println("Press ENTER to exit.");
        System.in.read();
        executor.shutdownNow();
        System.out.println("Shutting down sample...");
        fileUploadNotificationReceiver.close();
        serviceClient.close();
      }
    }
    ```

10. Enregistrez et fermez le fichier `read-file-upload-notification\src\main\java\com\mycompany\app\App.java`.

11. Utilisez la commande suivante pour générer l’application **read-file-upload-notification** et vérifiez l’absence d’erreurs :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

Dans une invite de commandes, exécutez la commande suivante dans le dossier `read-file-upload-notification` :

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

Dans une invite de commandes, exécutez la commande suivante dans le dossier `simulated-device` :

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

La capture d’écran suivante montre la sortie de l’application **simulated-device** :

![Sortie de l’application simulated-device](media/iot-hub-java-java-upload/simulated-device.png)

La capture d’écran suivante montre la sortie de l’application **read-file-upload-notification** :

![Sortie de l’application read-file-upload-notification](media/iot-hub-java-java-upload/read-file-upload-notification.png)

Vous pouvez utiliser le portail pour afficher le fichier chargé dans le conteneur de stockage que vous avez configuré :

![Fichier chargé](media/iot-hub-java-java-upload/uploaded-file.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser les fonctionnalités de téléchargement de fichier d’IoT Hub pour simplifier les chargements de fichiers à partir d’appareils. Vous pouvez continuer à explorer les scénarios et fonctionnalités d’IoT Hub avec les articles suivants :

* [Créer un IoT Hub par programmation](iot-hub-rm-template-powershell.md)

* [Présentation du SDK C](iot-hub-device-sdk-c-intro.md)

* [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Simulation d’un appareil avec IoT Edge](../iot-edge/quickstart-linux.md)