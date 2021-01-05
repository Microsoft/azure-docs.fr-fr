---
title: Planification des travaux avec Azure IoT Hub (Java) | Microsoft Docs
description: Procédure de planification d’une tâche Azure IoT Hub pour appeler une méthode directe et définir une propriété de votre choix sur plusieurs appareils. Vous utilisez Azure IoT device SDK pour Java afin d’implémenter les applications d’appareil simulé, et Azure IoT service SDK pour Java afin d’implémenter une application de service qui exécute la tâche.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/16/2019
ms.custom: mqtt, devx-track-java
ms.openlocfilehash: 36092e0bb7d0249837d2eaa7898e9231ecb5f5a9
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147005"
---
# <a name="schedule-and-broadcast-jobs-java"></a>Planifier et diffuser des travaux (Java)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Utilisez Azure IoT Hub pour planifier et suivre des travaux qui mettent à jour des millions d’appareils. Utilisez les travaux pour :

* Mettre à jour les propriétés souhaitées
* Mettre à jour les balises
* Appeler des méthodes directes

Un travail encapsule l’une de ces actions et suit l’exécution sur un ensemble d’appareils. Une requête de jumeau d’appareil définit l’ensemble d’appareils sur lesquels le travail s’exécute. Par exemple, une application principale peut utiliser un travail pour appeler une méthode directe sur 10 000 appareils et les redémarrer. Vous spécifiez l’ensemble des appareils avec une requête de jumeau d’appareil et planifiez le travail à exécuter ultérieurement. Le travail suit la progression à mesure que chacun de ces appareils reçoit et exécute la méthode directe de redémarrage.

Pour plus d’informations sur chacune de ces fonctionnalités, consultez les pages :

* Jumeau d’appareil et propriétés : [Prise en main des représentations d’appareils](iot-hub-java-java-twin-getstarted.md)

* Méthodes directes : [Guide du développeur IoT Hub - méthodes directes](iot-hub-devguide-direct-methods.md) et [Didacticiel : Utiliser des méthodes directes](quickstart-control-device-java.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Ce didacticiel vous explique les procédures suivantes :

* Créer une application pour périphérique implémentant une méthode directe nommée **lockDoor**. L’application pour périphérique reçoit également les modifications de propriétés souhaitées envoyées par l’application principale.

* Créer une application principale qui crée un travail pour appeler la méthode directe **lockDoor** sur plusieurs appareils. Un autre travail envoie les mises à jour de propriétés souhaitées à plusieurs appareils.

À la fin de ce didacticiel, vous disposerez d’une application pour périphérique de console Java et d’une application principale de console Java :

**simulated-device** se connecte à IoT Hub et implémente la méthode directe **lockDoor** et gère les modifications de propriétés souhaitées.

**schedule-jobs** utilise des travaux pour appeler la méthode directe **lockDoor** et mettre à jour les propriétés souhaitées du jumeau de plusieurs appareils.

> [!NOTE]
> L’article relatif aux [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md) fournit des informations sur les Kits de développement logiciel (SDK) Azure que l’on peut utiliser pour générer des applications pour périphérique et des applications principales.

## <a name="prerequisites"></a>Prérequis

* [Java SE Development Kit 8](/java/azure/jdk/?view=azure-java-stable). Veillez à sélectionner **Java 8** sous **Prise en charge à long terme** pour accéder aux téléchargements du kit JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

Vous pouvez également utiliser l’outil [Extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension) pour ajouter un appareil à votre IoT Hub.

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="create-the-service-app"></a>Créer l’application de service

Dans cette section, vous créez une application console Java qui utilise des travaux pour :

* Appeler la méthode directe **lockDoor** sur plusieurs appareils.

* Envoyer les propriétés souhaitées à plusieurs appareils.

Pour créer l’application :

1. Sur votre ordinateur de développement, créez un dossier vide nommé **iot-java-schedule-jobs**.

2. Dans le dossier **iot-java-schedule-jobs**, créez un projet Maven nommé **schedule-jobs** en entrant la commande ci-dessous à l'invite de commandes. Il s’agit d’une commande unique et longue :

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=schedule-jobs -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

3. À l'invite de commandes, accédez au dossier **schedule-jobs**.

4. À l'aide d'un éditeur de texte, ouvrez le fichier **pom.xml** situé dans le dossier **schedule-jobs** et ajoutez la dépendance suivante au nœud **dependencies**. Cette dépendance vous permet d’utiliser le package **iot-service-client** dans votre application pour communiquer avec votre IoT Hub :

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.17.1</version>
      <type>jar</type>
    </dependency>
    ```

    > [!NOTE]
    > Vous pouvez rechercher la dernière version de **iot-service-client** avec la [recherche Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

5. Ajoutez le nœud **build** suivant après le nœud **dependencies**. Cette configuration ordonne à Maven d’utiliser Java 1.8 pour générer l’application :

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Enregistrez et fermez le fichier **pom.xml**.

7. À l'aide d'un éditeur de texte, ouvrez le fichier **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

8. Ajoutez les instructions **import** suivantes au fichier :

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceTwinDevice;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Pair;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Query;
    import com.microsoft.azure.sdk.iot.service.devicetwin.SqlQuery;
    import com.microsoft.azure.sdk.iot.service.jobs.JobClient;
    import com.microsoft.azure.sdk.iot.service.jobs.JobResult;
    import com.microsoft.azure.sdk.iot.service.jobs.JobStatus;

    import java.util.Date;
    import java.time.Instant;
    import java.util.HashSet;
    import java.util.Set;
    import java.util.UUID;
    ```

9. Ajoutez les variables de niveau classe suivantes à la classe **App** . Remplacez `{youriothubconnectionstring}` par la chaîne de connexion de votre IoT Hub que vous avez précédemment copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string) :

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    // How long the job is permitted to run without
    // completing its work on the set of devices
    private static final long maxExecutionTimeInSeconds = 30;
    ```

10. Ajoutez la méthode suivante à la classe **App** pour planifier un travail et mettre à jour les propriétés **Construction** et **Plancher** souhaitées dans le jumeau d’appareil :

    ```java
    private static JobResult scheduleJobSetDesiredProperties(JobClient jobClient, String jobId) {
      DeviceTwinDevice twin = new DeviceTwinDevice(deviceId);
      Set<Pair> desiredProperties = new HashSet<Pair>();
      desiredProperties.add(new Pair("Building", 43));
      desiredProperties.add(new Pair("Floor", 3));
      twin.setDesiredProperties(desiredProperties);
      // Optimistic concurrency control
      twin.setETag("*");

      // Schedule the update twin job to run now
      // against a single device
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleUpdateTwin(jobId, 
          "deviceId='" + deviceId + "'",
          twin,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling desired properties job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    }
    ```

11. Pour planifier un travail afin d’appeler la méthode **lockDoor**, ajoutez la méthode suivante à la classe **App** :

    ```java
    private static JobResult scheduleJobCallDirectMethod(JobClient jobClient, String jobId) {
      // Schedule a job now to call the lockDoor direct method
      // against a single device. Response and connection
      // timeouts are set to 5 seconds.
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleDeviceMethod(jobId,
          "deviceId='" + deviceId + "'",
          "lockDoor",
          5L, 5L, null,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling direct method job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    };
    ```

12. Pour surveiller un travail, ajoutez la méthode suivante à la classe **App** :

    ```java
    private static void monitorJob(JobClient jobClient, String jobId) {
      try {
        JobResult jobResult = jobClient.getJob(jobId);
        if(jobResult == null)
        {
          System.out.println("No JobResult for: " + jobId);
          return;
        }
        // Check the job result until it's completed
        while(jobResult.getJobStatus() != JobStatus.completed)
        {
          Thread.sleep(100);
          jobResult = jobClient.getJob(jobId);
          System.out.println("Status " + jobResult.getJobStatus() + " for job " + jobId);
        }
        System.out.println("Final status " + jobResult.getJobStatus() + " for job " + jobId);
      } catch (Exception e) {
        System.out.println("Exception monitoring job: " + jobId);
        System.out.println(e.getMessage());
        return;
      }
    }
    ```

13. Pour obtenir les détails des travaux que vous avez exécutés, ajoutez la méthode suivante :

    ```java
    private static void queryDeviceJobs(JobClient jobClient, String start) throws Exception {
      System.out.println("\nQuery device jobs since " + start);

      // Create a jobs query using the time the jobs started
      Query deviceJobQuery = jobClient
          .queryDeviceJob(SqlQuery.createSqlQuery("*", SqlQuery.FromType.JOBS, "devices.jobs.startTimeUtc > '" + start + "'", null).getQuery());

      // Iterate over the list of jobs and print the details
      while (jobClient.hasNextJob(deviceJobQuery)) {
        System.out.println(jobClient.getNextJob(deviceJobQuery));
      }
    }
    ```

14. Mettez à jour la signature de la méthode **main** pour qu’elle inclue la clause `throws` suivante :

    ```java
    public static void main( String[] args ) throws Exception
    ```

15. Pour exécuter et surveiller les deux travaux de manière séquentielle, remplacez le code de la méthode **main** par le code suivant :

    ```java
    // Record the start time
    String start = Instant.now().toString();

    // Create JobClient
    JobClient jobClient = JobClient.createFromConnectionString(iotHubConnectionString);
    System.out.println("JobClient created with success");

    // Schedule twin job desired properties
    // Maximum concurrent jobs is 1 for Free and S1 tiers
    String desiredPropertiesJobId = "DPCMD" + UUID.randomUUID();
    scheduleJobSetDesiredProperties(jobClient, desiredPropertiesJobId);
    monitorJob(jobClient, desiredPropertiesJobId);

    // Schedule twin job direct method
    String directMethodJobId = "DMCMD" + UUID.randomUUID();
    scheduleJobCallDirectMethod(jobClient, directMethodJobId);
    monitorJob(jobClient, directMethodJobId);

    // Run a query to show the job detail
    queryDeviceJobs(jobClient, start);

    System.out.println("Shutting down schedule-jobs app");
    ```

16. Enregistrez et fermez le fichier **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

17. Générez l’application **schedule-jobs** et corrigez les erreurs éventuelles. À l'invite de commandes, accédez au dossier **schedule-jobs** et exécutez la commande suivante :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-device-app"></a>Créer une application d’appareil

Dans cette section, vous créez une application de console Java qui gère les propriétés souhaitées envoyées à partir de IoT Hub et implémente l’appel de méthode directe.

1. Dans le dossier **iot-java-schedule-jobs**, créez un projet Maven nommé **simulated-device** en entrant la commande suivante à l'invite de commandes. Il s’agit d’une commande unique et longue :

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

2. À l'invite de commandes, accédez au dossier **simulated-device**.

3. À l'aide d'un éditeur de texte, ouvrez le fichier **pom.xml** situé dans le dossier **simulated-device** et ajoutez les dépendances suivantes au nœud **dependencies**. Cette dépendance vous permet d’utiliser le package **iot-device-client** dans votre application pour communiquer avec votre IoT Hub :

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.17.5</version>
    </dependency>
    ```

    > [!NOTE]
    > Vous pouvez rechercher la dernière version de **iot-device-client** avec la [recherche Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Ajoutez la dépendance suivante au nœud **dependencies**. Cette dépendance configure un NOP pour la façade de journalisation Apache [SLF4J](https://www.slf4j.org/) utilisée par le kit de développement logiciel (SDK) du client d'appareil afin d'implémenter la journalisation. Cette configuration est facultative, mais si vous l'omettez, un avertissement peut s'afficher sur la console lorsque vous lancez l'application. Pour plus d'informations sur la journalisation dans le SDK du client d'appareil, consultez [Journalisation](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md#logging) dans le fichier Readme *Exemples relatifs à Azure IoT device SDK pour Java*.

    ```xml
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-nop</artifactId>
      <version>1.7.28</version>
    </dependency>
    ```

5. Ajoutez le nœud **build** suivant après le nœud **dependencies**. Cette configuration ordonne à Maven d’utiliser Java 1.8 pour générer l’application :

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Enregistrez et fermez le fichier **pom.xml**.

7. À l'aide d'un éditeur de texte, ouvrez le fichier **simulated-device\src\main\java\com\mycompany\app\App.java**.

8. Ajoutez les instructions **import** suivantes au fichier :

    ```java
    import com.microsoft.azure.sdk.iot.device.*;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.*;

    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.Scanner;
    ```

9. Ajoutez les variables de niveau classe suivantes à la classe **App** . Remplacez `{yourdeviceconnectionstring}` par la chaîne de connexion de l'appareil que vous avez précédemment copiée à la section [Inscrire un nouveau périphérique dans le hub IoT](#register-a-new-device-in-the-iot-hub) :

    ```java
    private static String connString = "{yourdeviceconnectionstring}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static final int METHOD_SUCCESS = 200;
    private static final int METHOD_NOT_DEFINED = 404;
    ```

    Cet exemple d’application utilise la variable **protocol** lorsqu’il instancie un objet **DeviceClient**.

10. Pour imprimer des informations du jumeau d’appareil dans la console, ajoutez la classe imbriquée suivante à la classe **App** :

    ```java
    // Handler for device twin operation notifications from IoT Hub
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to device twin operation with status " + status.name());
      }
    }
    ```

11. Pour imprimer des notifications relatives à la méthode directe dans la console, ajoutez la classe imbriquée suivante à la classe **App** :

    ```java
    // Handler for direct method notifications from IoT Hub
    protected static class DirectMethodStatusCallback implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to direct method operation with status " + status.name());
      }
    }
    ```

12. Pour gérer les appels de méthode directe à partir de IoT Hub, ajoutez la classe imbriquée suivante à la classe **App** :

    ```java
    // Handler for direct method calls from IoT Hub
    protected static class DirectMethodCallback
        implements DeviceMethodCallback {
      @Override
      public DeviceMethodData call(String methodName, Object methodData, Object context) {
        DeviceMethodData deviceMethodData;
        switch (methodName) {
          case "lockDoor": {
            System.out.println("Executing direct method: " + methodName);
            deviceMethodData = new DeviceMethodData(METHOD_SUCCESS, "Executed direct method " + methodName);
            break;
          }
          default: {
            deviceMethodData = new DeviceMethodData(METHOD_NOT_DEFINED, "Not defined direct method " + methodName);
          }
        }
        // Notify IoT Hub of result
        return deviceMethodData;
      }
    }
    ```

13. Mettez à jour la signature de la méthode **main** pour qu’elle inclue la clause `throws` suivante :

    ```java
    public static void main( String[] args ) throws IOException, URISyntaxException
    ```

14. Remplacez le code de la méthode **main** par le code suivant pour :
    * créer un client d’appareil afin de communiquer avec IoT Hub.
    * créer un objet **Device** afin de stocker les propriétés du jumeau d’appareil.

    ```java
    // Create a device client
    DeviceClient client = new DeviceClient(connString, protocol);

    // An object to manage device twin desired and reported properties
    Device dataCollector = new Device() {
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context)
      {
        System.out.println("Received desired property change: " + propertyKey + " " + propertyValue);
      }
    };
    ```

15. Pour démarrer les services du client de l’appareil, ajoutez le code suivant à la méthode **principale** :

    ```java
    try {
      // Open the DeviceClient
      // Start the device twin services
      // Subscribe to direct method calls
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);
      client.subscribeToDeviceMethod(new DirectMethodCallback(), null, new DirectMethodStatusCallback(), null);
    } catch (Exception e) {
      System.out.println("Exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.closeNow();
      System.out.println("Shutting down...");
    }
    ```

16. Pour attendre que l’utilisateur appuie sur la touche **Entrée** avant l’arrêt, ajoutez le code suivant à la fin de la méthode **principal** :

    ```java
    // Close the app
    System.out.println("Press any key to exit...");
    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();
    dataCollector.clean();
    client.closeNow();
    scanner.close();
    ```

17. Enregistrez et fermez le fichier **simulated-device\src\main\java\com\mycompany\app\App.java**.

18. Générez l’application **simulated-device** et corrigez les erreurs. À l'invite de commandes, accédez au dossier **simulated-device** et exécutez la commande suivante :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Exécuter les applications

Vous êtes maintenant prêt à exécuter les applications de console.

1. À l'invite de commande, dans le dossier **simulated-device**, exécutez la commande suivante afin de commencer à écouter les applications de l'appareil pour accéder aux modifications de propriété et aux appels de méthode directe souhaités :

   ```cmd/sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![Le client de périphérique démarre](./media/iot-hub-java-java-schedule-jobs/device-app-1.png)

2. À l’invite de commande du dossier `schedule-jobs`, exécutez la commande suivante pour faire fonctionner l’application de service **schedule-jobs** lors de l’exécution de deux travaux. Le premier définit les valeurs de propriété de votre choix et le second appelle la méthode directe :

   ```cmd\sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![L’application de service Java IoT Hub crée deux travaux](./media/iot-hub-java-java-schedule-jobs/service-app-1.png)

3. L’application pour périphérique gère la modification de propriété souhaitée et l’appel de méthode directe :

   ![Le client d’appareil répond aux modifications](./media/iot-hub-java-java-schedule-jobs/device-app-2.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé un travail pour planifier une méthode directe sur un appareil et la mise à jour des propriétés de représentation de l’appareil.

Utilisez les ressources suivantes :

* Envoyez des données de télémétrie à partir d’appareils en suivant le didacticiel [Bien démarrer avec IoT Hub](quickstart-send-telemetry-java.md).

* Contrôlez les appareils de façon interactive (par exemple pour mettre en marche un ventilateur à partir d’une application contrôlée par l’utilisateur) en suivant le tutoriel [Utiliser des méthodes directes](quickstart-control-device-java.md).