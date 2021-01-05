---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 49b18b83c778a990398c4443d508743566ecfb20
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511269"
---
Ce tutoriel montre comment créer un exemple d’application d’appareil IoT Plug-and-Play incluant plusieurs composants, comment le connecter à votre hub IoT et comment utiliser Azure CLI pour afficher les données de télémétrie qu’il envoie. L’exemple d’application est écrit en Java et est inclus dans l’Azure IoT device SDK pour Java. Un créateur de solutions peut utiliser Azure CLI pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner du code d’appareil.

Ce tutoriel vous montre comment créer un exemple d'application d'appareil IoT Plug-and-Play avec des composants, comment le connecter à votre hub IoT, et comment utiliser l'outil Explorateur Azure IoT pour voir les informations qu'il envoie au hub. L’exemple d’application est écrit en Java et est inclus dans l’Azure IoT device SDK pour Java. Un créateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner du code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour effectuer ce tutoriel sur Windows, vous devez installer les logiciels suivants sur votre environnement Windows local :

* Java SE Development Kit 8. Dans [Prise en charge à long terme de Java pour Azure et Azure Stack](/java/azure/jdk/?preserve-view=true&view=azure-java-stable), sous **Prise en charge à long terme**, sélectionnez **Java 8**.
* [Apache Maven 3](https://maven.apache.org/download.cgi).

## <a name="download-the-code"></a>Téléchargement du code

Si vous avez terminé [Démarrage rapide : Connecter un exemple d'application d'appareil IoT Plug-and-Play s'exécutant sous Windows à IoT Hub (Java)](../articles/iot-pnp/quickstart-connect-device.md), vous avez déjà cloné le référentiel.

Ouvrez une invite de commandes dans le répertoire de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub de [kits de développement logiciel (SDK) et de bibliothèques Java Azure IoT](https://github.com/Azure/azure-iot-sdk-java) dans cet emplacement :

```cmd
git clone https://github.com/Azure/azure-iot-sdk-java.git
```

Attendez-vous à ce que cette opération prenne plusieurs minutes.

## <a name="build-the-code"></a>Générer le code

Sous Windows, accédez au dossier racine du dépôt du Kit de développement logiciel (SDK) Java cloné. Exécutez la commande suivante pour générer les dépendances :

```cmd/sh
mvn install -T 2C -DskipTests
```

## <a name="run-the-device-sample"></a>Exécuter l’exemple d’appareil

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour exécuter l'exemple d'application, accédez au dossier *\device\iot-device-samples\pnp-device-sample\temperature-controller-device-sample* et exécutez la commande suivante :

```cmd/sh
mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
```

L’appareil, désormais prêt à recevoir des commandes et des mises à jour de propriétés, a commencé à envoyer des données de télémétrie au hub. Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Vérifier le code

Cet exemple implémente un appareil contrôleur de température IoT Plug-and-Play. Le modèle implémenté par cet exemple utilise [plusieurs composants](../articles/iot-pnp/concepts-components.md). Le [fichier de modèle DTDL (Digital Twins Definition Language) pour l’appareil de contrôle de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) définit la télémétrie, les propriétés et les commandes implémentées par l’appareil.

Le code d’appareil utilise la classe `DeviceClient` standard pour se connecter à votre hub IoT. L’appareil envoie l’ID de modèle du modèle DTDL qu’il implémente dans la demande de connexion. Un appareil qui envoie un ID de modèle est un appareil IoT Plug-and-Play :

```java
private static void initializeDeviceClient() throws URISyntaxException, IOException {
    ClientOptions options = new ClientOptions();
    options.setModelId(MODEL_ID);
    deviceClient = new DeviceClient(deviceConnectionString, protocol, options);

    deviceClient.registerConnectionStatusChangeCallback((status, statusChangeReason, throwable, callbackContext) -> {
        log.debug("Connection status change registered: status={}, reason={}", status, statusChangeReason);

        if (throwable != null) {
            log.debug("The connection status change was caused by the following Throwable: {}", throwable.getMessage());
            throwable.printStackTrace();
        }
    }, deviceClient);

    deviceClient.open();
}
```

L’ID de modèle est stocké dans le code, comme indiqué dans l’extrait de code suivant :

```java
private static final String MODEL_ID = "dtmi:com:example:Thermostat;1";
```

Une fois que l’appareil se connecte à votre hub IoT, le code inscrit les gestionnaires de commandes.

```java
deviceClient.subscribeToDeviceMethod(new MethodCallback(), null, new MethodIotHubEventCallback(), null);
```

Il existe des gestionnaires distincts pour les mises à jour des propriétés souhaitées sur les deux composants de thermostat :

```java
deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new GenericPropertyUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback = Stream.of(
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_1, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_1)),
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_2, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_2))
).collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, AbstractMap.SimpleEntry::getValue));

deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
```

L’exemple de code envoie des données de télémétrie depuis chaque composant de thermostat :

```java
sendTemperatureReading(THERMOSTAT_1);
sendTemperatureReading(THERMOSTAT_2);
```

La méthode `sendTemperatureReading` utilise la classe `PnpHhelper` pour créer des messages pour chaque composant :

```java
Message message = PnpHelper.createIotHubMessageUtf8(telemetryName, currentTemperature, componentName);
```

La classe `PnpHelper` contient d’autres exemples de méthodes que vous pouvez utiliser avec un modèle à plusieurs composants.

Utilisez l’outil Explorateur Azure IoT pour visualiser les données de télémétrie et les propriétés des deux composants de thermostat :

:::image type="content" source="media/iot-pnp-multiple-components-java/multiple-component.png" alt-text="Appareil à plusieurs composants dans l’Explorateur Azure IoT":::

Vous pouvez également utiliser l'outil Explorateur Azure IoT pour appeler des commandes dans l'un ou l'autre des deux composants de thermostat, ou dans le composant par défaut.

[!INCLUDE [iot-pnp-clean-resources.md](iot-pnp-clean-resources.md)]
