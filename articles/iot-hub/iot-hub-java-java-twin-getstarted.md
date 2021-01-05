---
title: Bien démarrer avec les jumeaux d’appareils Azure IoT Hub (Java) | Microsoft Docs
description: Guide d’utilisation des jumeaux d’appareils Azure IoT Hub pour ajouter des balises, puis utiliser une requête IoT Hub. Vous utilisez Azure IoT device SDK pour Java afin d’implémenter l’application pour appareils et Azure IoT service SDK afin d’implémenter une application de service qui ajoute les balises et exécute la requête IoT Hub.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/26/2019
ms.custom: mqtt, devx-track-java
ms.openlocfilehash: 68908b7d62b8211c202262112831d5d2b523f594
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92146957"
---
# <a name="get-started-with-device-twins-java"></a>Bien démarrer avec les jumeaux d’appareils (Java)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

Dans ce tutoriel, vous allez créer deux applications de console Java :

* **add-tags-query**, application principale Java qui ajoute des balises et interroge les jumeaux d’appareils.
* **simulated-device**, application Java pour appareils qui se connecte à votre hub IoT et signale sa connectivité à l’aide d’une propriété signalée.

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

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Créer l’application de service

Dans cette section, vous créez une application Java qui ajoute des métadonnées d’emplacement sous forme de balise au jumeau d’appareil associé à **myDeviceId** dans IoT Hub. L’application interroge IoT Hub pour lister d’abord les appareils situés aux États-Unis, puis les appareils qui signalent une connexion réseau cellulaire.

1. Sur votre ordinateur de développement, créez un dossier vide nommé **iot-java-twin-getstarted**.

2. Dans le dossier **iot-java-twin-getstarted**, créez un projet Maven nommé **add-tags-query** en lançant la commande ci-dessous dans votre invite de commandes :

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=add-tags-query -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

3. Dans votre invite de commandes, accédez au dossier **add-tags-query**.

4. À l’aide d’un éditeur de texte, ouvrez le fichier **pom.xml** situé dans le dossier **add-tags-query** et ajoutez la dépendance suivante au nœud **dependencies**. Cette dépendance vous permet d’utiliser le package **iot-service-client** dans votre application pour communiquer avec votre IoT Hub :

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

5. Ajoutez le nœud **build** suivant après le nœud **dependencies**. Cette configuration ordonne à Maven d’utiliser Java 1.8 pour générer l’application.

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

7. À l’aide d’un éditeur de texte, ouvrez le fichier **add-tags-query\src\main\java\com\mycompany\app\App.java**.

8. Ajoutez les instructions **import** suivantes au fichier :

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.*;
    import com.microsoft.azure.sdk.iot.service.exceptions.IotHubException;

    import java.io.IOException;
    import java.util.HashSet;
    import java.util.Set;
    ```

9. Ajoutez les variables de niveau classe suivantes à la classe **App** . Remplacez `{youriothubconnectionstring}` par la chaîne de connexion de l’IoT Hub que vous avez copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string).

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    public static final String region = "US";
    public static final String plant = "Redmond43";
    ```

10. Mettez à jour la signature de la méthode **main** pour qu’elle inclue la clause `throws` suivante :

    ```java
    public static void main( String[] args ) throws IOException
    ```

11. Remplacez le code dans la méthode **main** par le code suivant pour créer les objets **DeviceTwin** et **DeviceTwinDevice**. L’objet **DeviceTwin** gère la communication avec votre hub IoT. L’objet **DeviceTwinDevice** représente le jumeau de l’appareil avec ses propriétés et ses balises :

    ```java
    // Get the DeviceTwin and DeviceTwinDevice objects
    DeviceTwin twinClient = DeviceTwin.createFromConnectionString(iotHubConnectionString);
    DeviceTwinDevice device = new DeviceTwinDevice(deviceId);
    ```

12. Ajoutez le bloc `try/catch` suivant à la méthode **main** :

    ```java
    try {
      // Code goes here
    } catch (IotHubException e) {
      System.out.println(e.getMessage());
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
    ```

13. Pour mettre à jour les balises **region** et **plant** du jumeau de l’appareil, ajoutez le code suivant dans le bloc `try` :

    ```java
    // Get the device twin from IoT Hub
    System.out.println("Device twin before update:");
    twinClient.getTwin(device);
    System.out.println(device);

    // Update device twin tags if they are different
    // from the existing values
    String currentTags = device.tagsToString();
    if ((!currentTags.contains("region=" + region) && !currentTags.contains("plant=" + plant))) {
      // Create the tags and attach them to the DeviceTwinDevice object
      Set<Pair> tags = new HashSet<Pair>();
      tags.add(new Pair("region", region));
      tags.add(new Pair("plant", plant));
      device.setTags(tags);

      // Update the device twin in IoT Hub
      System.out.println("Updating device twin");
      twinClient.updateTwin(device);
    }

    // Retrieve the device twin with the tag values from IoT Hub
    System.out.println("Device twin after update:");
    twinClient.getTwin(device);
    System.out.println(device);
    ```

14. Pour interroger les jumeaux d’appareils dans IoT Hub, ajoutez le code suivant au bloc `try` après le code que vous avez ajouté à l’étape précédente. Le code exécute deux requêtes. Chaque requête retourne un maximum de 100 appareils.

    ```java
    // Query the device twins in IoT Hub
    System.out.println("Devices in Redmond:");

    // Construct the query
    SqlQuery sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43'", null);

    // Run the query, returning a maximum of 100 devices
    Query twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 100);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }

    System.out.println("Devices in Redmond using a cellular network:");

    // Construct the query
    sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43' AND properties.reported.connectivityType = 'cellular'", null);

    // Run the query, returning a maximum of 100 devices
    twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 3);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }
    ```

15. Enregistrez le fichier **add-tags-query\src\main\java\com\mycompany\app\App.java** et fermez-le.

16. Générez l’application **add-tags-query** et corrigez les erreurs éventuelles. À l’invite de commandes, accédez au dossier **add-tags-query** et exécutez la commande suivante :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-device-app"></a>Créer une application d’appareil

Dans cette section, vous allez créer une application console Java qui définit une valeur de propriété signalée à envoyer à IoT Hub.

1. Dans le dossier **iot-java-twin-getstarted**, créez un projet Maven nommé **simulated-device** en entrant la commande suivante à l’invite de commandes :

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. À l'invite de commandes, accédez au dossier **simulated-device**.

3. À l'aide d'un éditeur de texte, ouvrez le fichier **pom.xml** situé dans le dossier **simulated-device** et ajoutez les dépendances suivantes au nœud **dependencies**. Cette dépendance vous permet d’utiliser le package **iot-device-client** dans votre application pour communiquer avec votre hub IoT.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.17.5</version>
    </dependency>
    ```

    > [!NOTE]
    > Vous pouvez rechercher la dernière version de **iot-device-client** avec la [recherche Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Ajoutez la dépendance suivante au nœud **dependencies**. Cette dépendance configure un NOP pour la façade de journalisation Apache [SLF4J](https://www.slf4j.org/) utilisée par le kit de développement logiciel (SDK) du client d'appareil afin d'implémenter la journalisation. Cette configuration est facultative, mais si vous l’omettez, un avertissement peut s’afficher sur la console lorsque vous lancez l’application. Pour plus d’informations sur la journalisation dans le SDK du client d’appareil, consultez [Journalisation](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md#logging) dans le fichier Readme *Exemples relatifs à Azure IoT device SDK pour Java*.

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

9. Ajoutez les variables de niveau classe suivantes à la classe **App** . Remplacez `{yourdeviceconnectionstring}` par la chaîne de connexion de l’appareil que vous avez copiée dans [Inscrire un nouveau périphérique dans le hub IoT](#register-a-new-device-in-the-iot-hub).

    ```java
    private static String connString = "{yourdeviceconnectionstring}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static String deviceId = "myDeviceId";
    ```

    Cet exemple d’application utilise la variable **protocol** lorsqu’il instancie un objet **DeviceClient**.

10. Ajoutez la méthode suivante à la classe **App** pour imprimer les informations sur les mises à jour de représentations :

    ```java
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
        @Override
        public void execute(IotHubStatusCode status, Object context) {
          System.out.println("IoT Hub responded to device twin operation with status " + status.name());
        }
      }
    ```

11. Remplacez le code de la méthode **main** par le code suivant pour :

    * créer un client d’appareil afin de communiquer avec IoT Hub.

    * créer un objet **Device** afin de stocker les propriétés du jumeau d’appareil.

    ```java
    DeviceClient client = new DeviceClient(connString, protocol);

    // Create a Device object to store the device twin properties
    Device dataCollector = new Device() {
      // Print details when a property value changes
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context) {
        System.out.println(propertyKey + " changed to " + propertyValue);
      }
    };
    ```

12. Ajoutez le code suivant à la méthode **main** pour créer une propriété signalée **connectivityType** et l’envoyer à IoT Hub :

    ```java
    try {
      // Open the DeviceClient and start the device twin services.
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);

      // Create a reported property and send it to your IoT hub.
      dataCollector.setReportedProp(new Property("connectivityType", "cellular"));
      client.sendReportedProperties(dataCollector.getReportedProp());
    }
    catch (Exception e) {
      System.out.println("On exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.closeNow();
      System.out.println("Shutting down...");
    }
    ```

13. Ajoutez le code suivant à la fin de la méthode **main**. Le fait d’attendre la touche **Entrée** laisse le temps à IoT Hub de signaler l’état des opérations du jumeau d’appareil.

    ```java
    System.out.println("Press any key to exit...");

    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();

    dataCollector.clean();
    client.close();
    ```

14. Modifiez la signature de la méthode **main** pour inclure les exceptions suivantes :

     ```java
     public static void main(String[] args) throws URISyntaxException, IOException
     ```

15. Enregistrez et fermez le fichier **simulated-device\src\main\java\com\mycompany\app\App.java**.

16. Générez l’application **simulated-device** et corrigez les erreurs. À l'invite de commandes, accédez au dossier **simulated-device** et exécutez la commande suivante :

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Exécuter les applications

Vous êtes maintenant prêt à exécuter les applications de console.

1. Dans une invite de commandes, exécutez la commande suivante dans le dossier **add-tags-query** pour exécuter de nouveau l’application de service **add-tags-query** :

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Capture d’écran montrant la sortie de la commande pour exécuter l’application de service add-tags-query.](./media/iot-hub-java-java-twin-getstarted/service-app-1.png)

    Vous pouvez constater que les balises **plant** et **region** sont ajoutées au jumeau d’appareil. La première requête retourne votre appareil, la seconde non.

2. Dans une invite de commandes, exécutez la commande suivante dans le dossier **simulated-device** pour ajouter la propriété signalée **connectivityType** au jumeau d’appareil :

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Le client d’appareil ajoute la propriété rapportée connectivityType](./media/iot-hub-java-java-twin-getstarted/device-app-1.png)

3. Dans une invite de commandes, exécutez la commande suivante dans le dossier **add-tags-query** pour exécuter de nouveau l’application de service **add-tags-query** une deuxième fois :

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Application de service Java IoT Hub pour mettre à jour la valeur des balises et exécuter les requêtes des appareils](./media/iot-hub-java-java-twin-getstarted/service-app-2.png)

    Maintenant que votre appareil a envoyé la propriété **connectivityType** à IoT Hub, la deuxième requête retourne votre appareil.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré un nouveau IoT Hub dans le portail Azure, puis créé une identité d’appareil dans le registre des identités de l’IoT Hub. Vous avez ajouté des métadonnées d’appareils sous forme de balises à partir d’une application principale et écrit une application pour appareils afin d’envoyer des informations sur la connectivité des appareils dans le jumeau d’appareil. Vous avez également appris à interroger les informations du jumeau d’appareil avec le langage de requête IoT Hub de type SQL.

Utilisez les ressources suivantes :

* Envoyez des données de télémétrie à partir d’appareils en suivant le didacticiel [Bien démarrer avec IoT Hub](quickstart-send-telemetry-java.md).

* Contrôlez les appareils de façon interactive (par exemple pour mettre en marche un ventilateur à partir d’une application contrôlée par l’utilisateur) en suivant le didacticiel [Utiliser des méthodes directes](quickstart-control-device-java.md).