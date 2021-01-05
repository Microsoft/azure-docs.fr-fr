---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: d803796792a3933cdf915214f15c56d146ecb49d
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511266"
---
Ce tutoriel vous montre comment créer un exemple d'application d'appareil IoT Plug-and-Play avec des composants, comment le connecter à votre hub IoT, et comment utiliser l'outil Explorateur Azure IoT pour voir les informations qu'il envoie au hub. L’exemple d’application est écrit pour Node.js et est inclus dans le kit Azure IoT Hub Device SDK pour Node.js. Un créateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner le code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce tutoriel, vous avez besoin de Node.js sur votre machine de développement. Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [nodejs.org](https://nodejs.org).

Vous pouvez vérifier la version actuelle de Node.js sur votre machine de développement à l’aide de la commande suivante :

```cmd/sh
node --version
```

## <a name="download-the-code"></a>Téléchargement du code

Si vous avez terminé [Démarrage rapide : Connecter un exemple d'application d'appareil IoT Plug-and-Play s'exécutant sous Windows à IoT Hub (Node.js)](../articles/iot-pnp/quickstart-connect-device.md), vous avez déjà cloné le référentiel.

Ouvrez une invite de commandes dans le répertoire de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [SDK Microsoft Azure IoT pour for Node.js](https://github.com/Azure/azure-iot-sdk-node) à cet emplacement :

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="install-required-libraries"></a>Installer les bibliothèques nécessaires

Vous utilisez le kit de développement logiciel (SDK) de l’appareil pour créer l’exemple de code inclus. L’application que vous créez simule un appareil Plug-and-Play avec plusieurs composants qui se connecte à un hub IoT. L’application envoie les données de télémétrie et les propriétés et reçoit des commandes.

1. Dans une fenêtre de terminal locale, accédez au dossier de votre dépôt cloné, puis au dossier */azure-iot-sdk-node/device/samples/pnp*. Exécutez ensuite la commande suivante pour installer les bibliothèques requises :

```cmd/sh
npm install
```

Ceci va installer les fichiers npm appropriés nécessaires pour exécuter les exemples dans le dossier.

## <a name="review-the-code"></a>Vérifier le code

Accédez au dossier *Azure-IOT-SDK-node\device\samples\pnp*.

Le dossier *Azure-IOT-SDK-node\device\samples\pnp* contient l’exemple de code pour l’appareil contrôleur de température IoT Plug-and-Play.

Le code figurant dans le fichier *pnpTemperatureController.js* implémente un appareil contrôleur de température IoT Plug-and-Play. Le modèle implémenté par cet exemple utilise [plusieurs composants](../articles/iot-pnp/concepts-components.md). Le [fichier de modèle DTDL (Digital Twins Definition Language) pour l’appareil de contrôle de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) définit la télémétrie, les propriétés et les commandes implémentées par l’appareil.

Ouvrez le fichier *pnpTemperatureController.js* dans un éditeur de code de votre choix. L’exemple de code montre comment :

- Définir le `modelId` qui est le DTMI pour l’appareil à implémenter. Ce DTMI est défini par l’utilisateur et doit correspondre au DTMI du [modèle DTDL du contrôleur de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

- Implémenter les composants définis dans le modèle DTDL du contrôleur de température. Les composants d’un contrôleur de température réel doivent implémenter ces deux interfaces. Ces deux interfaces sont déjà publiées dans un dépôt central. Dans cet exemple, les deux interfaces sont :

  - Thermostat
  - Informations sur l’appareil développées par Azure

- Définit les noms des composants. Cet exemple a deux thermostats et un composant d’informations sur l’appareil.

- Définit un nom de commande. Il s’agit des commandes auxquelles l’appareil répond.

- Définit la constante `serialNumber`. `serialNumber` est fixe pour un appareil donné.

- Définit les gestionnaires de commandes.

- Définit les fonctions pour envoyer des réponses aux commandes.

- Définit des fonctions helper pour journaliser les demandes de commandes.

- Définit une fonction helper pour créer les propriétés.

- Définit un écouteur pour les mises à jour des propriétés.

- Définit une fonction pour envoyer de la télémétrie depuis cet appareil. Les thermostats et le composant par défaut envoient de la télémétrie. Cette fonction reçoit le nom du composant comme paramètre.

- Définissez une fonction `main` qui :

  - Utilise le SDK d’appareil pour créer un client d’appareil et se connecter à votre hub IoT. L’appareil fournit le `modelId` pour que IoT Hub puisse identifier l’appareil en tant qu’appareil IoT Plug-and-Play.

  - Commence à écouter les demandes de commande en utilisant la fonction `onDeviceMethod`. La fonction configure un écouteur pour les demandes de commande provenant du service :

    - Le DTDL de l’appareil définit les commandes `reboot` et `getMaxMinReport`.
    - La fonction `commandHandler` définit la façon dont l’appareil répond à une commande.

  - Commence à envoyer de la télémétrie en utilisant `setInterval` et `sendTelemetry`.

  - Utilise la fonction `helperCreateReportedPropertiesPatch` pour créer les propriétés et les `updateComponentReportedProperties` pour mettre à jour les propriétés.

  - Utilise `desiredPropertyPatchListener` pour être à l’écoute des mises à jour de propriétés.

  - Désactive tous les écouteurs et toutes les tâches, puis quitte la boucle quand vous appuyez sur **Q** ou **q**.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/pnp/readme.md).

Maintenant que vous avez vu le code, utilisez la commande suivante pour exécuter l’exemple :

```cmd\sh
node pnpTemperatureController.js
```

Vous voyez la sortie suivante, qui indique que l’appareil a commencé à envoyer des données de télémétrie au hub et est maintenant prêt à recevoir des commandes et des mises à jour de propriétés.

![Messages de confirmation de l’appareil](media/iot-pnp-multiple-components-node/multiple-component.png)

Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

[!INCLUDE [iot-pnp-clean-resources.md](iot-pnp-clean-resources.md)]
