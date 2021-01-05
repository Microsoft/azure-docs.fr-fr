---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 3a7bcbba99595468aa69d852493308a5a77851d8
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511263"
---
Ce tutoriel vous montre comment créer un exemple d’application d’appareil IoT Plug-and-Play avec des composants, comment le connecter à votre hub IoT, et comment utiliser l’outil Explorateur Azure IoT pour voir les informations qu’il envoie au hub. L’exemple d’application est écrit en Python et inclus dans le SDK Azure IoT Device pour Python. Un créateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner le code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce tutoriel, vous avez besoin de Python 3.7 sur votre machine de développement. Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [python.org](https://www.python.org/). Vous pouvez vérifier votre version de Python à l’aide de la commande suivante :  

```cmd/sh
python --version
```

Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [python.org](https://www.python.org/).

## <a name="download-the-code"></a>Téléchargement du code

Le package **azure-iot-device** est publié en tant que PIP.

Dans votre environnement Python local, installez le package comme suit :

```cmd/sh
pip install azure-iot-device
```

Si vous avez terminé [Démarrage rapide : Connecter un exemple d’application d’appareil IoT Plug-and-Play exécutant Windows sur Linux à IoT Hub (Python)](../articles/iot-pnp/quickstart-connect-device.md), vous avez déjà cloné le référentiel.

Clonez le référentiel IoT du Kit de développement logiciel (SDK) Python :

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-python
```

## <a name="review-the-code"></a>Vérifier le code

Cet exemple implémente un appareil contrôleur de température IoT Plug-and-Play. Le modèle implémenté par cet exemple utilise [plusieurs composants](../articles/iot-pnp/concepts-components.md). Le [fichier de modèle DTDL (Digital Twins Definition Language) pour l’appareil de contrôle de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) définit la télémétrie, les propriétés et les commandes implémentées par l’appareil.

Le dossier *azure-iot-sdk-python\azure-iot-device\samples\pnp* contient l’exemple de code pour l’appareil IoT Plug-and-Play. Les fichiers de l’exemple de contrôleur de température sont les suivants :

- temp_controller_with_thermostats.py
- pnp_helper.py

Le contrôleur de température a plusieurs composants et un composant par défaut, en fonction du modèle DTDL du contrôleur de température.

Ouvrez le fichier *temp_controller_with_thermostats.py* dans l’éditeur de votre choix. Le code dans ce fichier effectue les opérations suivantes :

1. Importe `pnp_helper.py` pour accéder aux méthodes d’assistance.

1. Définit deux identificateurs de modèle de jumeau numérique (DTMI) qui représentent de manière unique deux interfaces différentes, définies dans le modèle DTDL. Les composants d’un contrôleur de température réel doivent implémenter ces deux interfaces. Ces deux interfaces sont déjà publiées dans un dépôt central. Les DTMI doivent être connus de l’utilisateur et varient en fonction du scénario d’implémentation de l’appareil. Pour l’exemple actuel, ces deux interfaces représentent :

    - un thermostat ;
    - des informations sur l’appareil développées par Azure.

1. Définit le DTMI `model_id` pour l’appareil en cours d’implémentation. Le DTMI est défini par l’utilisateur et doit correspondre au DTMI spécifié dans le fichier du modèle DTDL.

1. Définit les noms donnés aux composants dans le fichier DTDL. Il existe deux thermostats dans le DTDL et un composant d’informations sur l’appareil. Une constante nommée `serial_number` est également définie dans le composant par défaut. Un `serial_number` ne peut pas changer pour un appareil.

1. Définit des implémentations de gestionnaire de commandes. Celles-ci définissent ce que fait l’appareil quand il reçoit des demandes de commande.

1. Définit des fonctions pour créer une réponse de commande. Celles-ci définissent la façon dont l’appareil répond avec des demandes de commande. Vous pouvez créer des fonctions de réponse de commande si une commande doit renvoyer une réponse personnalisée au hub IoT. Si une fonction de réponse pour une commande n’est pas fournie, une réponse générique est envoyée. Dans cet exemple, seule la commande **getMaxMinReport** a une réponse personnalisée.

1. Définit une fonction pour envoyer la télémétrie de cet appareil. Les thermostats et le composant par défaut envoient de la télémétrie. Cette fonction accepte un paramètre de nom de composant facultatif lui permettant d’identifier le composant qui a envoyé la télémétrie.

1. Définit un écouteur pour les demandes de commande.

1. Définit un écouteur pour les mises à jour de propriétés souhaitées.

1. A une fonction `main` qui effectue les opérations suivantes :

    - Utilise le Kit de développement logiciel (SDK) d’appareil pour créer un client d’appareil et se connecter à votre hub IoT. L’appareil envoie le `model_id` pour permettre au hub IoT d’identifier l’appareil en tant qu’appareil IoT Plug-and-Play.

    - Utilise la fonction `create_reported_properties` dans le fichier d’assistance pour créer les propriétés. Transmettez à cette fonction le nom du composant et les propriétés en tant que paires clé-valeur.

    - Met à jour les propriétés lisibles de ses composants en appelant `patch_twin_reported_properties`.

    - Commence à écouter les demandes de commande à l’aide de la fonction `execute_command_listener`. La fonction configure un écouteur pour les demandes de commande provenant du service. Quand vous configurez l’écouteur, vous fournissez un `method_name`, un `user_command_handler` et un `create_user_response_handler` facultatif en tant que paramètres.
        - Le `method_name` définit la demande de commande. Dans cet exemple, le modèle définit les commandes **reboot**, et **getMaxMinReport**.
        - La fonction `user_command_handler` définit ce que l’appareil doit faire quand il reçoit une commande.
        - La fonction `create_user_response_handler` crée une réponse à envoyer à votre hub IoT quand une commande s’exécute correctement. Vous pouvez afficher cette réponse dans le portail. Si cette fonction n’est pas fournie, une réponse générique est envoyée au service.

    - Utilise `execute_property_listener` pour écouter les mises à jour de propriétés.

    - Commence à envoyer la télémétrie à l’aide de `send_telemetry`. L’exemple de code utilise une boucle pour appeler trois fonctions d’envoi de télémétrie. Chacune d’elles est appelée toutes les huit secondes.

    - Désactive l’ensemble des écouteurs et des tâches, puis quitte la boucle quand vous appuyez sur **Q** ou **q**.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/pnp/README.md).

Utilisez la commande suivante pour exécuter cet exemple :

```cmd/sh
python temp_controller_with_thermostats.py
```

L’exemple d’appareil envoie à votre IoT Hub des messages de télémétrie à intervalles de quelques secondes.

Vous voyez la sortie suivante indiquant que l’appareil a commencé à envoyer des données de télémétrie au hub et est prêt à recevoir des commandes et des mises à jour de propriétés.

![Messages de confirmation de l’appareil](media/iot-pnp-multiple-components-python/multiple-component.png)

Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]
