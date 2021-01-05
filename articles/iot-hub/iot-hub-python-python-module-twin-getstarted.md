---
title: Identité de module et jumeau de module Azure IoT Hub (Python)
description: Découvrez comment créer une identité de module et mettre à jour un jumeau de module à l’aide des SDK IoT pour Python.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: menchi
ms.custom: devx-track-python
ms.openlocfilehash: 665281adc892e6b3655c0b1d0533cb3148e62940
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92139401"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-python"></a>Bien démarrer avec l’identité de module et le jumeau de module IoT Hub (Python)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Les identités de module et les jumeaux de module](iot-hub-devguide-module-twins.md) sont similaires aux identités d’appareil et aux jumeaux d’appareil Azure IoT Hub, mais offrent un plus grand niveau de détail. Contrairement aux identités d’appareil et aux jumeaux d’appareil Azure IoT Hub qui permettent à l’application back-end de configurer un appareil et d’obtenir une visibilité sur les conditions de l’appareil, les identités de module et les jumeaux de module fournissent ces fonctionnalités pour les composants individuels d’un appareil. Sur les appareils compatibles qui intègrent plusieurs composants, par exemple des appareils basés sur un système d’exploitation ou des appareils avec un microprogramme, ils permettent d’isoler la configuration et les conditions de chacun de ces composants.
>

À la fin de ce tutoriel, vous disposerez de trois applications Python :

* **CreateModule**, qui crée une identité d’appareil, une identité de module et des clés de sécurité associées pour connecter votre appareil et vos clients de module.

* **UpdateModuleTwinDesiredProperties**, qui envoie à votre hub IoT les propriétés souhaitées de votre jumeau de module mises à jour.

* **ReceiveModuleTwinDesiredPropertiesPatch**, qui reçoit le correctif des propriétés souhaitées du jumeau de module sur votre appareil.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous allez créer un service back-end qui ajoute un appareil dans le registre des identités, puis ajoute un module à cet appareil. Ce service nécessite l’autorisation **Écriture du registre** (qui comprend également **Lecture du registre**). Vous créerez aussi un service qui ajoute les propriétés souhaitées au jumeau de module pour le module nouvellement créé. Ce service a besoin de l’autorisation **Connexion de service**. Bien qu’il existe des stratégies d’accès partagé par défaut qui accordent ces autorisations individuellement, dans cette section, vous allez créer une stratégie d’accès partagé personnalisée qui contient ces deux autorisations.

[!INCLUDE [iot-hub-include-find-service-regrw-connection-string](../../includes/iot-hub-include-find-service-regrw-connection-string.md)]

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>Créer une identité d’appareil et une identité de module dans IoT Hub

Dans cette section, vous allez créer une application de service Python qui crée une identité d’appareil et une identité de module dans le registre des identités de votre hub IoT. Un appareil ou un module ne peut se connecter à un hub IoT que s’il a une entrée dans le registre des identités. Pour plus d’informations, consultez [Comprendre le registre des identités dans votre IoT Hub](iot-hub-devguide-identity-registry.md). En exécutant cette application console, une clé et un ID uniques sont générés pour chaque appareil et module. Votre appareil et le module utilisent ces valeurs pour s’identifier lorsqu’ils envoient des messages d’appareil-à-cloud à IoT Hub. Les ID sont sensibles à la casse.

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-hub** :

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **msrest** : Vous avez besoin de ce package pour intercepter les exceptions **HTTPOperationError**.

    ```cmd/sh
    pip install msrest
    ```

1. À l’aide d’un éditeur de texte, créez un fichier nommé **CreateModule.py** dans votre répertoire de travail.

1. Ajoutez le code suivant à votre fichier Python. Remplacez *YourIoTHubConnectionString* par la chaîne de connexion que vous avez copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from msrest.exceptions import HttpOperationError
    from azure.iot.hub import IoTHubRegistryManager

    CONNECTION_STRING = "YourIotHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        try:
            # CreateDevice - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            device_state = "enabled"
            new_device = iothub_registry_manager.create_device_with_sas(
                DEVICE_ID, primary_key, secondary_key, device_state
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the device already exists.
                new_device = iothub_registry_manager.get_device(DEVICE_ID)
            else:
                raise

        print("device <" + DEVICE_ID +
              "> has primary key = " + new_device.authentication.symmetric_key.primary_key)

        try:
            # CreateModule - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            managed_by = ""
            new_module = iothub_registry_manager.create_module_with_sas(
                DEVICE_ID, MODULE_ID, managed_by, primary_key, secondary_key
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the module already exists.
                new_module = iothub_registry_manager.get_module(DEVICE_ID, MODULE_ID)
            else:
                raise

        print("device/module <" + DEVICE_ID + "/" + MODULE_ID +
              "> has primary key = " + new_module.authentication.symmetric_key.primary_key)

    except Exception as ex:
        print("Unexpected error {0}".format(ex))
    except KeyboardInterrupt:
        print("IoTHubRegistryManager sample stopped")
    ```

1. À l’invite de commandes, exécutez la commande suivante :

    ```cmd/sh
    python CreateModule.py
    ```

Cette application crée une identité d’appareil avec l’ID **myFirstDevice** et une identité de module avec l’ID **myFirstModule** sous l’appareil **myFirstDevice**. (Si l’ID de module ou d’appareil existe déjà dans le registre des identités, le code récupère simplement les informations existantes du module ou de l’appareil). L’application affiche l’ID et la clé primaire pour chaque identité.

> [!NOTE]
> Le registre des identités IoT Hub stocke uniquement les identités des appareils et des modules pour permettre un accès sécurisé à IoT Hub. Le registre des identités stocke les ID et les clés d’appareil à utiliser en tant qu’informations d’identification de sécurité. Il stocke également un indicateur activé/désactivé pour chaque appareil pouvant être utilisé pour désactiver l’accès de cet appareil. Si votre application a besoin de stocker d’autres métadonnées spécifiques aux appareils, elle doit utiliser un magasin spécifique aux applications. Il n’y a aucun indicateur d’activation/désactivation pour les identités de module. Pour plus d’informations, consultez [Comprendre le registre des identités dans votre IoT Hub](iot-hub-devguide-identity-registry.md).
>

## <a name="update-the-module-twin-using-python-service-sdk"></a>Mettre à jour le jumeau de module à l’aide du SDK de service Python

Dans cette section, vous allez créer une application de service Python qui met à jour les propriétés souhaitées du jumeau de module.

1. À l’invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-hub**. Vous pouvez ignorer cette étape si vous avez installé le package **azure-iot-hub** dans la section précédente.

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. À l’aide d’un éditeur de texte, créez un fichier nommé **UpdateModuleTwinDesiredProperties.py** dans votre répertoire de travail.

1. Ajoutez le code suivant à votre fichier Python. Remplacez *YourIoTHubConnectionString* par la chaîne de connexion que vous avez copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties

    CONNECTION_STRING = "YourIoTHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        module_twin = iothub_registry_manager.get_module_twin(DEVICE_ID, MODULE_ID)
        print ( "" )
        print ( "Module twin properties before update    :" )
        print ( "{0}".format(module_twin.properties) )

        # Update twin
        twin_patch = Twin()
        twin_patch.properties = TwinProperties(desired={"telemetryInterval": 122})
        updated_module_twin = iothub_registry_manager.update_module_twin(
            DEVICE_ID, MODULE_ID, twin_patch, module_twin.etag
        )
        print ( "" )
        print ( "Module twin properties after update     :" )
        print ( "{0}".format(updated_module_twin.properties) )

    except Exception as ex:
        print ( "Unexpected error {0}".format(ex) )
    except KeyboardInterrupt:
        print ( "IoTHubRegistryManager sample stopped" )
    ```

## <a name="get-updates-on-the-device-side"></a>Recevoir les mises à jour du côté appareil

Dans cette section, vous allez créer une application Python pour obtenir la mise à jour des propriétés souhaitées du jumeau de module sur votre appareil.

1. Obtenez la chaîne de connexion de votre module. Dans le [portail Azure](https://portal.azure.com/), accédez à votre hub IoT et sélectionnez **Appareils IoT** dans le volet gauche. Sélectionnez **myFirstDevice** dans la liste des appareils et ouvrez-le. Sous **Identités de module**, sélectionnez **myFirstModule**. Copiez la chaîne de connexion du module. Vous en aurez besoin lors d’une étape ultérieure.

   ![Détails du module du Portail Azure](./media/iot-hub-python-python-module-twin-getstarted/module-detail.png)

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-device** :

    ```cmd/sh
    pip install azure-iot-device
    ```

1. À l’aide d’un éditeur de texte, créez un fichier nommé **ReceiveModuleTwinDesiredPropertiesPatch.py** dans votre répertoire de travail.

1. Ajoutez le code suivant à votre fichier Python. Remplacez *YourModuleConnectionString* par la chaîne de connexion de module que vous avez copiée à l’étape 1.

    ```python
    import time
    import threading
    from azure.iot.device import IoTHubModuleClient

    CONNECTION_STRING = "YourModuleConnectionString"


    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print("")
            print("Twin desired properties patch received:")
            print(patch)

    def iothub_client_sample_run():
        try:
            module_client = IoTHubModuleClient.create_from_connection_string(CONNECTION_STRING)

            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(module_client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()

            while True:
                time.sleep(1000000)

        except KeyboardInterrupt:
            print("IoTHubModuleClient sample stopped")


    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python sample..." )
        print ( "IoTHubModuleClient waiting for commands, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

## <a name="run-the-apps"></a>Exécuter les applications

Dans cette section, vous allez exécuter l’application d’appareil **ReceiveModuleTwinDesiredPropertiesPatch**, puis exécuter l’application de service **UpdateModuleTwinDesiredProperties** pour mettre à jour les propriétés souhaitées de votre module.

1. Ouvrez une invite de commandes et exécutez l’application d’appareil :

    ```cmd/sh
    python ReceiveModuleTwinDesiredPropertiesPatch.py
    ```

   ![Sortie initiale de l’application d’appareil](./media/iot-hub-python-python-module-twin-getstarted/device-1.png)

1. Ouvrez une invite de commandes distincte et exécutez l’application de service :

    ```cmd/sh
    python UpdateModuleTwinDesiredProperties.py
    ```

    Notez que la propriété souhaitée **TelemetryInterval** apparaît dans le jumeau de module mis à jour dans la sortie de votre application de service :

   ![Sortie de l’application de service](./media/iot-hub-python-python-module-twin-getstarted/service.png)

    La même propriété apparaît dans le correctif des propriétés souhaitées reçu dans la sortie de votre application d’appareil :

   ![La sortie de l’application d’appareil affiche le correctif des propriétés souhaitées](./media/iot-hub-python-python-module-twin-getstarted/device-2.png)

## <a name="next-steps"></a>Étapes suivantes

Pour continuer la prise en main de IoT Hub et explorer les autres scénarios IoT, consultez les articles suivants :

* [Prise en main de la gestion d’appareils](iot-hub-node-node-device-management-get-started.md)

* [Bien démarrer avec IoT Edge](../iot-edge/quickstart-linux.md)