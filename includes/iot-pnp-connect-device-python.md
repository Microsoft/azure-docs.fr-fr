---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 607c83bd97f8e371896a5a8ac0c9aa05ab34fa2c
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511660"
---
Ce guide de démarrage rapide montre comment créer un exemple d’application d’appareil IoT Plug-and-Play, comment le connecter à votre hub IoT et comment utiliser l’outil Explorateur Azure IoT pour afficher la télémétrie qu’il envoie. L’exemple d’application est écrit pour Python et inclus dans le Kit de développement logiciel (SDK) d’appareil Azure IoT Hub pour Python. Un créateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner le code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce guide de démarrage rapide, vous avez besoin de Python 3.7 sur votre ordinateur de développement. Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [python.org](https://www.python.org/). Vous pouvez vérifier votre version de Python à l’aide de la commande suivante :  

```cmd/sh
python --version
```

Dans votre environnement Python local, installez le package comme suit :

```cmd/sh
pip install azure-iot-device
```

Clonez le référentiel IoT du Kit de développement logiciel (SDK) Python et extrayez **master** :

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-python
```

## <a name="run-the-sample-device"></a>Exécuter l’exemple d’appareil

Le dossier *azure-iot-sdk-python\azure-iot-device\samples\pnp* contient l’exemple de code pour l’appareil IoT Plug-and-Play. Ce guide de démarrage rapide utilise le fichier *simple_thermostat.py*. Cet exemple de code implémente un appareil compatible IoT Plug-and-Play et utilise la bibliothèque de client d’appareil Azure IoT Python.

Ouvrez le fichier **simple_thermostat.py** dans un éditeur de texte. Notez comment il :

1. Définit un identificateur de modèle de jumeau d’appareil unique (DTMI) qui représente de façon unique le [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json). Un DTMI doit être connus de l’utilisateur et varie en fonction du scénario d’implémentation de l’appareil. Pour l’exemple actuel, le modèle représente un thermostat possédant une télémétrie, des propriétés et des commandes associées à la surveillance de la température.

1. Dispose de fonctions permettant de définir des implémentations de gestionnaire de commandes. Vous écrivez ces gestionnaires pour définir la manière dont l’appareil répond aux demandes de commande.

1. Dispose d’une fonction pour définir une réponse à la commande. Vous créez des fonctions de réponse à la commande pour renvoyer une réponse à votre hub IoT.

1. Définit une fonction d’écouteur de clavier de saisie pour vous permettre de quitter l’application.

1. Dispose d’une fonction **main**. La fonction **main** :

    1. Utilise le Kit de développement logiciel (SDK) d’appareil pour créer un client d’appareil et se connecter à votre hub IoT.

    1. Met à jour les propriétés. Le modèle que nous utilisons, **Thermostat**, définit `targetTemperature` et `maxTempSinceLastReboot` comme les deux propriétés pour notre thermostat. C’est donc ce que nous allons utiliser. Les propriétés sont mises à jour à l’aide de la méthode `patch_twin_reported_properties` définie sur le `device_client`.

    1. Commence à écouter les demandes de commande à l’aide de la fonction **execute_command_listener**. La fonction configure un « écouteur » pour écouter les commandes provenant du service. Quand vous configurez l’écouteur, vous fournissez un `method_name`, un `user_command_handler` et un `create_user_response_handler`.
        - La fonction `user_command_handler` définit ce que l’appareil doit faire quand il reçoit une commande. Par exemple, si votre alarme vient à être désactivée, l’effet de la réception de cette commande est l’éveil. Imaginez cela comme l’« effet » de la commande appelée.
        - La fonction `create_user_response_handler` crée une réponse à envoyer à votre hub IoT quand une commande s’exécute correctement. Par exemple, si votre alarme vient à être désactivée, vous répondez en appuyant sur la touche répéter. Considérez cela comme la réponse que vous donnez au service. Vous pouvez afficher cette réponse dans le portail.

    1. Commence à envoyer la télémétrie. La valeur **pnp_send_telemetry** est définie dans le fichier pnp_methods.py. L’exemple de code utilise une boucle pour appeler cette fonction toutes les huit secondes.

    1. Désactive l’ensemble des écouteurs et des tâches, puis quitte la boucle quand vous appuyez sur **Q** ou **q**.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/pnp/README.md).

Maintenant que vous avez vu le code, utilisez la commande suivante pour exécuter l’exemple :

```cmd/sh
python simple_thermostat.py
```

Vous voyez la sortie suivante indiquant que l’appareil a commencé à envoyer des données de télémétrie au hub et est prêt à recevoir des commandes et des mises à jour de propriétés :

```cmd/sh
Listening for command requests and property updates
Press Q to quit
Sending telemetry for temperature
Sent message
```

Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]
