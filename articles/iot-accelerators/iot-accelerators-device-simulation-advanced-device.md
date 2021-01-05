---
title: Créer un modèle d’appareil simulé avancé - Azure | Microsoft Docs
description: Dans ce guide pratique, vous apprendrez à créer un modèle d’appareil avancé à utiliser avec l’accélérateur de solution de simulation d’appareil.
author: troyhopwood
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: troyhop
ms.custom:
- mvc
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: 58891bdb4601744129c97454f43202d3ba07b709
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852555"
---
# <a name="create-an-advanced-device-model"></a>Créer un modèle d’appareil avancé

Ce guide décrit les fichiers JSON et JavaScript qui définissent un modèle d’appareil personnalisé. L’article inclut des exemples de fichiers de définition de modèles d’appareils et vous montre comment les charger sur votre instance de simulation d’appareil. Vous pouvez créer des modèles d’appareils avancés afin de simuler des comportements des appareils plus réalistes pour vos tests.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes de ce guide pratique, vous avez besoin d’une instance déployée de la simulation d’appareil dans votre abonnement Azure.

Si vous n’avez pas encore déployé la simulation d’appareil, consultez [Déploiement de la simulation d’appareil](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) sur GitHub.

### <a name="open-device-simulation"></a>Ouvrir la Simulation d’appareil

Si vous n’avez pas encore déployé la simulation d’appareil, consultez [Déploiement de la simulation d’appareil](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) sur GitHub.

## <a name="device-models"></a>Modèles d’appareils

Chaque appareil simulé est un modèle d’appareil spécifique qui définit le comportement de simulation. Ce comportement inclut la fréquence à laquelle envoyer la télémétrie, les types de messages à envoyer et les méthodes prises en charge.

Vous définissez un modèle d’appareil à l’aide d’un fichier de définition d’appareil JSON et d’un ensemble de fichiers JavaScript. Ces fichiers JavaScript définissent le comportement de simulation, par exemple la télémétrie aléatoire et la logique de la méthode.

Un modèle d’appareil classique comprend :

* un fichier JSON pour chaque modèle d’appareil (par exemple, elevator.json) ;
* un fichier de script de comportement JavaScript pour chaque modèle d’appareil (par exemple, elevator-state.js) ;
* un fichier de script de méthode JavaScript pour chaque méthode d’appareil (par exemple, elevator-go-down.js).

> [!NOTE]
> Tous les modèles d’appareils ne définissent pas des méthodes. Un modèle d’appareil peut donc ne pas avoir de script de méthode. Toutefois, tous les modèles d’appareils doivent avoir un script de comportement.

## <a name="device-definition-file"></a>Fichier de définition d’appareil

Chaque fichier de définition d’appareil contient les détails d’un modèle d’appareil simulé, notamment les informations suivantes :

* Nom du modèle d’appareil : chaîne.
* Protocole : AMQP | MQTT | HTTP.
* État initial de l'appareil.
* Fréquence d’actualisation de l’état de l’appareil.
* Fichiers JavaScript à utiliser pour actualiser l’état de l’appareil.
* Liste de messages de télémétrie à envoyer, chacun avec une fréquence spécifique.
* Schéma des messages de télémétrie utilisé par l’application principale pour analyser les données de télémétrie reçues.
* Liste des méthodes prises en charge et fichier JavaScript à utiliser pour simuler chaque méthode.

### <a name="file-schema"></a>Schéma de fichier

La version du schéma est toujours « 1.0.0 » et est spécifique au format de ce fichier :

```json
"SchemaVersion": "1.0.0"
```

### <a name="device-model-description"></a>Description du modèle d’appareil

Les propriétés suivantes décrivent le modèle d’appareil. Chaque type a un identificateur unique, une version sémantique, un nom et une description :

```json
"Id": "chiller-01",
"Version": "0.0.1",
"Name": "Chiller",
"Description": "Chiller with external temperature and humidity sensors."
```

### <a name="iot-protocol"></a>Protocole IoT

Les appareils IoT peuvent se connecter à l’aide de différents protocoles. La simulation vous permet d’utiliser **AMQP**, **MQTT** ou **HTTP** :

```json
"Protocol": "AMQP"
```

### <a name="simulated-device-state"></a>État de l’appareil simulé

Chaque appareil simulé a un état interne, qui doit être défini. L’état définit également les propriétés pouvant être signalées dans les données de télémétrie. Par exemple, un refroidisseur peut avoir un état initial tel que :

```json
"InitialState": {
    "temperature": 50,
    "humidity": 40
},
```

Un appareil mobile avec plusieurs capteurs peut avoir plus de propriétés, par exemple :

```json
"InitialState": {
    "latitude": 47.445301,
    "longitude": -122.296307,
    "speed": 30.0,
    "speed_unit": "mph",
    "cargotemperature": 38.0,
    "cargotemperature_unit": "F"
}
```

L’état de l’appareil est conservé en mémoire par le service de simulation et fourni comme entrée à la fonction JavaScript. La fonction JavaScript peut décider de ce qui suit :

* Ignorer l’état et générer des données aléatoires.
* Mettre à jour l’état de l’appareil d’une façon réaliste pour un scénario donné.

La fonction qui génère l’état reçoit également comme entrée :

* L’ID de l’appareil.
* Le modèle de l’appareil.
* L’heure actuelle. Cette valeur permet de générer des données différentes pour chaque appareil et chaque heure.

### <a name="generating-telemetry-messages"></a>Génération de messages de télémétrie

Le service de simulation peut envoyer plusieurs types de données de télémétrie pour chaque appareil. En règle générale, les données de télémétrie incluent des données d’état de l’appareil. Par exemple, une salle simulée peut envoyer des informations sur la température et l’humidité toutes les 10 secondes. Notez les espaces réservés dans l’extrait suivant, qui sont automatiquement remplacés par des valeurs d’état de l’appareil :

```json
"Telemetry": [
    {
        "Interval": "00:00:10",
        "MessageTemplate":
            "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"humidity\":\"${humidity}\"}",
        "MessageSchema": {
            "Name": "RoomComfort;v1",
            "Format": "JSON",
            "Fields": {
                "temperature": "double",
                "temperature_unit": "text",
                "humidity": "integer"
            }
        }
    }
],
```

Les espaces réservés utilisent une syntaxe spéciale **${NAME}** , où **NAME** est une clé de l’objet d’état de l’appareil retournée par la fonction **principale** de JavaScript. Les chaînes doivent être mises entre guillemets, mais pas les nombres.

#### <a name="message-schema"></a>Schéma du message

Chaque type de message doit avoir un schéma bien défini. Le schéma du message est également publié sur IoT Hub, si bien que les applications principales peuvent réutiliser les informations afin d’interpréter les données de télémétrie entrantes.

Le schéma prend en charge le format JSON, ce qui permet une analyse, une transformation et une analytique faciles sur plusieurs systèmes et services.

Les champs répertoriés dans le schéma peuvent avoir les types suivants :

* Objet - sérialisé à l’aide de JSON
* Binaire - sérialisé à l’aide de base64
* Texte
* Boolean
* Integer
* Double
* DateTime

### <a name="supported-methods"></a>Méthodes prises en charge

Les appareils simulés peuvent également réagir aux appels de méthode, auquel cas ils exécutent une logique et fournissent une réponse. Comme pour la simulation, la logique de la méthode est stockée dans un fichier JavaScript et peut interagir avec l’état de l’appareil. Par exemple :

```json
"CloudToDeviceMethods": {
    "Start": {
        "Type": "JavaScript",
        "Path": "release-pressure.js"
    }
}
```

## <a name="create-a-device-definition-file"></a>Créer un fichier de définition d’appareil

Dans ce guide pratique, vous allez voir comment créer un modèle d'appareil pour un drone. Le drone volera de manière aléatoire autour d’un ensemble initial de coordonnées en changeant d’emplacement et d’altitude.

Copiez le fichier JSON suivant dans un éditeur de texte et enregistrez-le sous **drone.json**.

### <a name="device-definition-json-example"></a>Exemple JSON de définition d’appareil

```json
{
  "SchemaVersion": "1.0.0",
  "Id": "drone",
  "Version": "0.0.1",
  "Name": "Drone",
  "Description": "Simple drone.",
  "Protocol": "AMQP",
  "Simulation": {
    "InitialState": {
      "velocity": 0.0,
      "velocity_unit": "mm/sec",
      "acceleration": 0.0,
      "acceleration_unit": "mm/sec^2",
      "latitude": 47.476075,
      "longitude": -122.192026,
      "altitude": 0.0
    },
    "Interval": "00:00:05",
    "Scripts": [{
      "Type": "JavaScript",
      "Path": "drone-state.js"
    }]
  },
  "Properties": {
    "Type": "Drone",
    "Firmware": "1.0",
    "Model": "P-96"
  },
  "Tags": {
    "Owner": "Contoso"
  },
  "Telemetry": [{
      "Interval": "00:00:05",
      "MessageTemplate": "{\"velocity\":\"${velocity}\",\"acceleration\":\"${acceleration}\",\"position\":\"${latitude}|${longitude}|${altitude}\"}",
      "MessageSchema": {
        "Name": "drone-event-sensor;v1",
        "Format": "JSON",
        "Fields": {
          "velocity": "double",
          "velocity_unit": "text",
          "acceleration": "double",
          "acceleration_unit": "text",
          "latitude": "double",
          "longitude": "double",
          "altitude": "double"
        }
      }
    }
  ],
    "CloudToDeviceMethods": {
        "RecallDrone": {
            "Type": "JavaScript",
            "Path": "droneRecall-method.js"
        }
    }
}
```

## <a name="behavior-script-files"></a>Fichiers de script de comportement

Le code dans le fichier de script de comportement déplace le drone. Le script modifie l’élévation et l’emplacement du drone en manipulant l’état de la mémoire de l’appareil.

Les fichiers JavaScript doivent avoir une fonction **principale** qui accepte deux paramètres :

* Un objet de **contexte** qui contient trois propriétés :
    * **currentTime** sous forme de chaîne au format **yyyy-MM-dd'T'HH:mm:sszzz**.
    * **deviceId**. Par exemple, **Simulated.Elevator.123**.
    * **deviceModel**. Par exemple, **Elevator**.
* Un objet d’**état**, qui est la valeur retournée par la fonction dans l’appel précédent. Cet état de l’appareil est géré par le service de simulation et utilisé pour générer des messages de télémétrie.

La fonction **principale** retourne le nouvel état de l’appareil. Par exemple :

```JavaScript
function main(context, state) {

    // Use context if the simulation depends on
    // time or device details.
    // Execute some logic, updating 'state'

    return state;
}
```

## <a name="create-a-behavior-script-file"></a>Créer un fichier de script de comportement

Copiez le fichier JavaScript suivant dans un éditeur de texte et enregistrez-le sous **drone-state.js**.

### <a name="device-model-javascript-simulation-example"></a>Exemple de simulation de modèle d’appareil JavaScript

```JavaScript
"use strict";

// Position control
const DefaultLatitude = 47.476075;
const DefaultLongitude = -122.192026;
const DistanceVariation = 10;

// Altitude control
const DefaultAltitude = 0.0;
const AverageAltitude = 499.99;
const AltitudeVariation = 5;

// Velocity control
const AverageVelocity = 60.00;
const MinVelocity = 20.00;
const MaxVelocity = 120.00;
const VelocityVariation = 5;

// Acceleration control
const AverageAcceleration = 2.50;
const MinAcceleration = 0.01;
const MaxAcceleration = 9.99;
const AccelerationVariation = 1;

// Display control for position and other attributes
const GeoSpatialPrecision = 6;
const DecimalPrecision = 2;

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: DefaultLatitude,
    longitude: DefaultLongitude,
    altitude: DefaultAltitude
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Simple formula generating a random value around the average
 * in between min and max
 */
function vary(avg, percentage, min, max) {
    var value = avg * (1 + ((percentage / 100) * (2 * Math.random() - 1)));
    value = Math.max(value, min);
    value = Math.min(value, max);
    return value;
}

/**
 * Entry point function called by the simulation engine.
 * Returns updated simulation state.
 * Device property updates must call updateProperties() to persist.
 *
 * @param context             The context contains current time, device model and id
 * @param previousState       The device state since the last iteration
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global state before generating the new telemetry, so that
    // the telemetry can apply changes using the previous function state.
    restoreSimulation(previousState, previousProperties);

    state.acceleration = vary(AverageAcceleration, AccelerationVariation, MinAcceleration, MaxAcceleration).toFixed(DecimalPrecision);
    state.velocity = vary(AverageVelocity, VelocityVariation, MinVelocity, MaxVelocity).toFixed(DecimalPrecision);

    // Use the last coordinates to calculate the next set with a given variation
    var coords = varylocation(Number(state.latitude), Number(state.longitude), DistanceVariation);
    state.latitude = Number(coords.latitude).toFixed(GeoSpatialPrecision);
    state.longitude = Number(coords.longitude).toFixed(GeoSpatialPrecision);

    // Fluctuate altitude between given variation constant by more or less
    state.altitude = vary(AverageAltitude, AltitudeVariation, AverageAltitude - AltitudeVariation, AverageAltitude + AltitudeVariation).toFixed(DecimalPrecision);

    return state;
}

/**
 * Generate a random geolocation at some distance (in miles)
 * from a given location
 */
function varylocation(latitude, longitude, distance) {
    // Convert to meters, use Earth radius, convert to radians
    var radians = (distance * 1609.344 / 6378137) * (180 / Math.PI);
    return {
        latitude: latitude + radians,
        longitude: longitude + radians / Math.cos(latitude * Math.PI / 180)
    };
}
```

## <a name="create-a-method-script-file"></a>Créer un fichier de script de méthode

Les scripts de méthode sont similaires aux scripts de comportement. Ils définissent le comportement de l’appareil lorsqu’un cloud spécifique à la méthode de l’appareil est appelé.

Le script de rappel du drone définit les coordonnées du drone à un point fixe pour simuler son retour au point de départ.

Copiez le fichier JavaScript suivant dans un éditeur de texte et enregistrez-le sous **droneRecall-method.js**.

### <a name="device-model-javascript-simulation-example"></a>Exemple de simulation de modèle d’appareil JavaScript

```JavaScript
"use strict";

// Default state
var state = {
    velocity: 0.0,
    velocity_unit: "mm/sec",
    acceleration: 0.0,
    acceleration_unit: "mm/sec^2",
    latitude: 0.0,
    longitude: 0.0,
    altitude: 0.0
};

// Default device properties
var properties = {};

/**
 * Restore the global state using data from the previous iteration.
 *
 * @param previousState device state from the previous iteration
 * @param previousProperties device properties from the previous iteration
 */
function restoreSimulation(previousState, previousProperties) {
    // If the previous state is null, force a default state
    if (previousState) {
        state = previousState;
    } else {
        log("Using default state");
    }

    if (previousProperties) {
        properties = previousProperties;
    } else {
        log("Using default properties");
    }
}

/**
 * Entry point function called by the simulation engine.
 *
 * @param context        The context contains current time, device model and id, not used
 * @param previousState  The device state since the last iteration, not used
 * @param previousProperties  The device properties since the last iteration
 */
/*jslint unparam: true*/
function main(context, previousState, previousProperties) {

    // Restore the global device properties and the global state before
    // generating the new telemetry, so that the telemetry can apply changes
    // using the previous function state.
    restoreSimulation(previousState, previousProperties);

    //simulate the behavior of a drone when recalled
  state.latitude = 47.476075;
  state.longitude = -122.192026;
  return state;
}
```

## <a name="debugging-script-files"></a>Débogage des fichiers de script

Alors que vous ne pouvez pas attacher un débogueur à un fichier de comportement en cours d’exécution, il est possible d’écrire des informations dans le journal de service à l’aide de la fonction **journal**. En cas d’erreurs de syntaxe, l’interpréteur échoue et écrit des informations concernant l’exception dans le journal.

Exemple de journalisation :

```JavaScript
function main(context, state) {

    log("This message will appear in the service logs.");

    log(context.deviceId);

    if (typeof(state) !== "undefined" && state !== null) {
        log("Previous value: " + state.temperature);
    }

    // ...

    return updateState;
}
```

## <a name="deploy-an-advanced-device-model"></a>Déployer un modèle d’appareil avancé

Pour déployer votre modèle d’appareil avancé, vous chargez les fichiers sur votre instance de simulation d’appareil :

Sélectionnez **Modèles d’appareils** dans la barre de menus. La page **Modèles d’appareils** répertorie les modèles d’appareils disponibles dans cette instance de simulation d’appareil :

![Modèles d’appareils](media/iot-accelerators-device-simulation-advanced-device/devicemodelnav.png)

Cliquez sur **+ Ajouter des modèles d’appareils** dans l’angle supérieur droit de la page :

![Ajouter un modèle d’appareil](media/iot-accelerators-device-simulation-advanced-device/devicemodels.png)

Cliquez sur **Avancé** pour ouvrir l’onglet Modèle de périphérique avancé :

![Options avancées, onglet](media/iot-accelerators-device-simulation-advanced-device/advancedtab.png)

Cliquez sur **Parcourir** et sélectionnez les fichiers JSON et JavaScript que vous avez créés. Veillez à sélectionner les trois fichiers. S’il manque un fichier, la validation échoue :

![Parcourir les fichiers](media/iot-accelerators-device-simulation-advanced-device/browse.png)

Si vos fichiers passent la validation, cliquez sur **Enregistrer** et votre modèle d’appareil est prêt à être utilisé dans le cadre d’une simulation. Sinon, corrigez les erreurs et rechargez les fichiers :

![Enregistrer](media/iot-accelerators-device-simulation-advanced-device/validated.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide pratique, vous avez appris à créer les fichiers du modèle d’appareil utilisés dans la simulation d’appareils et à établir un modèle d’appareil avancé. Vous voudrez ensuite peut-être explorer comment [Utiliser Time Series Insights pour visualiser les données de télémétrie envoyées par l’accélérateur de solution de simulation d’appareil](./iot-accelerators-device-simulation-time-series-insights.md).