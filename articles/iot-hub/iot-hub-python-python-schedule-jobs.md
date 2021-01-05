---
title: Planifier des travaux avec Azure IoT Hub (Python) | Microsoft Docs
description: Procédure de planification d’une tâche Azure IoT Hub pour appeler une méthode directe sur plusieurs appareils. Vous pouvez utiliser les SDK Azure IoT pour Python afin d’implémenter les applications d’appareil simulé et une application de service pour exécuter le travail.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/17/2020
ms.author: robinsh
ms.custom: devx-track-python
ms.openlocfilehash: 733e3be21a1a1305b5c7947de1ae54ddce5e0d2f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87876680"
---
# <a name="schedule-and-broadcast-jobs-python"></a>Planifier et diffuser des travaux (Python)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Azure IoT Hub est un service entièrement géré qui permet à une application principale de créer et de suivre des travaux qui planifient et mettent à jour des millions d’appareils.  Les travaux peuvent être utilisés pour les actions suivantes :

* Mettre à jour les propriétés souhaitées
* Mettre à jour les balises
* Appeler des méthodes directes

Sur le plan conceptuel, un travail encapsule l’une de ces actions et suit la progression de l’exécution par rapport à un ensemble d’appareils, défini par une requête de représentation d’appareil.  Par exemple, à l’aide d’un travail, une application principale peut appeler une méthode de redémarrage sur 10 000 appareils, spécifiée par une requête de représentation d’appareil et planifiée dans l’avenir.  Cette application peut ensuite suivre la progression à mesure que chacun de ces appareils reçoit et exécute la méthode de redémarrage.

Pour en savoir plus sur chacune de ces fonctionnalités, consultez les articles suivants :

* Jumeau d’appareil et propriétés : [Bien démarrer avec des jumeaux d’appareils](iot-hub-python-twin-getstarted.md) et [Tutoriel : Guide pratique pour utiliser des propriétés de jumeau d’appareil](tutorial-device-twins.md)

* Méthodes directes : [Guide du développeur IoT Hub - Méthodes directes](iot-hub-devguide-direct-methods.md) et [Tutoriel : Méthodes directes](quickstart-control-device-python.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Ce didacticiel vous explique les procédures suivantes :

* Créer une application d’appareil simulé Python disposant d’une méthode directe, qui active **lockDoor**, qui peut être appelé par le backend de solution.

* Créer une application console Python qui appelle la méthode directe **lockDoor** sur l’application d’appareil simulé à l’aide d’un travail et met à jour les propriétés souhaitées à l’aide d’un travail d’appareil.

À la fin de ce didacticiel, vous disposerez de deux applications Python :

**simDevice.py**, qui se connecte à votre hub IoT avec l’identité de l’appareil et reçoit une méthode directe **lockDoor**

**scheduleJobService.py**, qui appelle une méthode directe sur l’application d’appareil simulé et met à jour les propriétés souhaitées du jumeau d’appareil à l’aide d’un travail

> [!NOTE]
> Le **SDK Azure IoT pour Python** ne prend pas directement en charge la fonctionnalité de **travaux**. Au lieu de cela, ce didacticiel offre une solution alternative utilisant des minuteries et des threads asynchrones. Pour obtenir des mises à jour supplémentaires, consultez la liste des fonctionnalités du **SDK de client de service** dans la page du [SDK Azure IoT pour Python](https://github.com/Azure/azure-iot-sdk-python).
>

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Création d’une application de périphérique simulé

Dans cette section, vous allez créer une application console Python qui répond à une méthode directe appelée par le cloud, ce qui déclenche une méthode **lockDoor** simulée.

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-device** :

    ```cmd/sh
    pip install azure-iot-device
    ```

2. À l’aide d’un éditeur de texte, créez un fichier **simDevice.py** dans votre répertoire de travail.

3. Ajoutez les instructions et variables `import` ci-dessous au début du fichier **simDevice.py**. Remplacez `deviceConnectionString` par la chaîne de connexion de l’appareil que vous avez créé plus haut :

    ```python
    import threading
    import time
    from azure.iot.device import IoTHubDeviceClient, MethodResponse

    CONNECTION_STRING = "{deviceConnectionString}"
    ```

4. Ajoutez le rappel de fonction suivant pour gérer la méthode **lockDoor** :

    ```python
    def lockdoor_listener(client):
        while True:
            # Receive the direct method request
            method_request = client.receive_method_request("lockDoor")  # blocking call
            print( "Locking Door!" )

            resp_status = 200
            resp_payload = {"Response": "lockDoor called successfully"}
            method_response = MethodResponse(method_request.request_id, resp_status, resp_payload)
            client.send_method_response(method_response)
    ```

5. Ajoutez un autre rappel de fonction pour gérer les mises à jour de jumeaux d’appareils :

    ```python
    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print ("")
            print ("Twin desired properties patch received:")
            print (patch)
    ```

6. Ajoutez le code suivant pour inscrire le gestionnaire de la méthode **lockDoor**. Ajoutez également la routine `main` :

    ```python
    def iothub_jobs_sample_run():
        try:
            client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)

            print( "Beginning to listen for 'lockDoor' direct method invocations...")
            lockdoor_listener_thread = threading.Thread(target=lockdoor_listener, args=(client,))
            lockdoor_listener_thread.daemon = True
            lockdoor_listener_thread.start()

            # Begin listening for updates to the Twin desired properties
            print ( "Beginning to listen for updates to Twin desired properties...")
            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()
            
            while True:
                time.sleep(1000)

        except KeyboardInterrupt:
            print ( "IoTHubDeviceClient sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python jobs sample..." )
        print ( "IoTHubDeviceClient waiting for commands, press Ctrl-C to exit" )

        iothub_jobs_sample_run()
    ```

7. Enregistrez et fermez le fichier **simDevice.py**.

> [!NOTE]
> Pour simplifier les choses, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, une interruption exponentielle), comme indiqué dans l’article [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous allez créer un service principal qui appelle une méthode directe sur un appareil et met à jour le jumeau de celui-ci. Pour appeler une méthode directe sur un appareil, le service a besoin de l’autorisation de **connexion de service**. Le service a également besoin des autorisations de **lecture du registre** et d’**écriture du registre** pour pouvoir lire et écrire dans le registre des identités. Aucune stratégie d’accès partagé par défaut ne contient que ces autorisations. Vous devez donc en créer une.

Pour créer une stratégie d’accès partagé qui accorde des autorisations de **connexion de service**, de **lecture du registre** et d’**écriture du registre**, et obtenir une chaîne de connexion pour cette stratégie, procédez comme suit :

1. Ouvrez votre IoT Hub dans le [portail Azure](https://portal.azure.com). La méthode la plus simple pour accéder à votre IoT Hub consiste à sélectionner **Groupes de ressources**, le groupe de ressources dans lequel se trouve votre IoT Hub, puis votre IoT Hub dans la liste des ressources.

2. Dans le volet de gauche de votre IoT Hub, sélectionnez **Stratégies d’accès partagé**.

3. Dans le menu supérieur au-dessus de la liste des stratégies, sélectionnez **Ajouter**.

4. Dans le volet **Ajouter une stratégie d’accès partagé**, entrez un nom descriptif pour votre stratégie, par exemple : *serviceAndRegistryReadWrite*. Sous **Autorisations**, sélectionnez **Connexion au service** et **Écriture du registre** (l’option **Lecture du registre** est automatiquement sélectionnée lorsque vous sélectionnez l’option **Écriture du registre**). Sélectionnez ensuite **Créer**.

    ![Montrer comment ajouter une nouvelle stratégie d'accès partagé](./media/iot-hub-python-python-schedule-jobs/add-policy.png)

5. Dans le volet **Stratégies d'accès partagé**, sélectionnez votre nouvelle stratégie dans la liste des stratégies.

6. Sous **Clés d’accès partagé**, sélectionnez l’icône de copie pour la **chaîne de connexion -- clé primaire** et enregistrez la valeur.

    ![Montrer comment récupérer la chaîne de connexion](./media/iot-hub-python-python-schedule-jobs/get-connection-string.png)

Pour plus d’informations sur les autorisations et les stratégies d’accès partagé IoT Hub, consultez [Contrôle d’accès et autorisations](./iot-hub-devguide-security.md#access-control-and-permissions).

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>Planifier des travaux pour appeler une méthode directe et mettre à jour les propriétés d’une représentation d’appareil

Dans cette section, vous créez une application console Python qui lance **lockDoor** à distance sur un appareil à l’aide d’une méthode directe et également met à jour les propriétés souhaitées du jumeau d’appareil.

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-hub** :

    ```cmd/sh
    pip install azure-iot-hub
    ```

2. À l’aide d’un éditeur de texte, créez un fichier **scheduleJobService.py** dans votre répertoire de travail.

3. Ajoutez les instructions et variables `import` ci-dessous au début du fichier **scheduleJobService.py**. Remplacez la valeur de l’espace réservé `{IoTHubConnectionString}` par la chaîne de connexion du hub IoT que vous avez copiée à l’étape [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string). Remplacez l’espace réservé `{deviceId}` par l’ID d’appareil que vous avez inscrit à l’étape [ Inscrire un nouvel appareil dans le hub IoT](#register-a-new-device-in-the-iot-hub) :

    ```python
    import sys
    import time
    import threading
    import uuid

    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties, CloudToDeviceMethod, CloudToDeviceMethodResult, QuerySpecification, QueryResult

    CONNECTION_STRING = "{IoTHubConnectionString}"
    DEVICE_ID = "{deviceId}"

    METHOD_NAME = "lockDoor"
    METHOD_PAYLOAD = "{\"lockTime\":\"10m\"}"
    UPDATE_PATCH = {"building":43,"floor":3}
    TIMEOUT = 60
    WAIT_COUNT = 5
    ```

4. Ajoutez la fonction suivante servant à rechercher des appareils :

    ```python
    def query_condition(iothub_registry_manager, device_id):

        query_spec = QuerySpecification(query="SELECT * FROM devices WHERE deviceId = '{}'".format(device_id))
        query_result = iothub_registry_manager.query_iot_hub(query_spec, None, 1)

        return len(query_result.items)
    ```

5. Ajoutez les méthodes suivantes pour exécuter les travaux qui appellent la méthode directe et le jumeau d’appareil :

    ```python
    def device_method_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job: " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)


        if query_condition(iothub_registry_manager, device_id):
            deviceMethod = CloudToDeviceMethod(method_name=METHOD_NAME, payload=METHOD_PAYLOAD)

            response = iothub_registry_manager.invoke_device_method(DEVICE_ID, deviceMethod)

            print ( "" )
            print ( "Direct method " + METHOD_NAME + " called." )

    def device_twin_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        if query_condition(iothub_registry_manager, device_id):

            twin = iothub_registry_manager.get_twin(DEVICE_ID)
            twin_patch = Twin(properties= TwinProperties(desired=UPDATE_PATCH))
            twin = iothub_registry_manager.update_twin(DEVICE_ID, twin_patch, twin.etag)

            print ( "" )
            print ( "Device twin updated." )
    ```

6. Ajoutez le code suivant pour planifier les travaux et mettre à jour leur statut. Ajoutez également la routine `main` :

    ```python
    def iothub_jobs_sample_run():
        try:
            method_thr_id = uuid.uuid4()
            method_thr = threading.Thread(target=device_method_job, args=(method_thr_id, DEVICE_ID, 20, TIMEOUT), kwargs={})
            method_thr.start()

            print ( "" )
            print ( "Direct method called with Job Id: " + str(method_thr_id) )

            twin_thr_id = uuid.uuid4()
            twin_thr = threading.Thread(target=device_twin_job, args=(twin_thr_id, DEVICE_ID, 10, TIMEOUT), kwargs={})
            twin_thr.start()

            print ( "" )
            print ( "Device twin called with Job Id: " + str(twin_thr_id) )

            while True:
                print ( "" )

                if method_thr.is_alive():
                    print ( "...job " + str(method_thr_id) + " still running." )
                else:
                    print ( "...job " + str(method_thr_id) + " complete." )

                if twin_thr.is_alive():
                    print ( "...job " + str(twin_thr_id) + " still running." )
                else:
                    print ( "...job " + str(twin_thr_id) + " complete." )

                print ( "Job status posted, press Ctrl-C to exit" )

                status_counter = 0
                while status_counter <= WAIT_COUNT:
                    time.sleep(1)
                    status_counter += 1

        except Exception as ex:
            print ( "" )
            print ( "Unexpected error {0}" % ex )
            return
        except KeyboardInterrupt:
            print ( "" )
            print ( "IoTHubService sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub jobs Python sample..." )
        print ( "    Connection string = {0}".format(CONNECTION_STRING) )
        print ( "    Device ID         = {0}".format(DEVICE_ID) )

        iothub_jobs_sample_run()
    ```

7. Enregistrez et fermez le fichier **scheduleJobService.py**.

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

1. À l’invite de commandes dans votre répertoire de travail, exécutez la commande suivante pour commencer à écouter la méthode directe de redémarrage :

    ```cmd/sh
    python simDevice.py
    ```

2. À une autre invite de commandes dans votre répertoire de travail, exécutez la commande suivante pour déclencher les travaux afin de verrouiller la porte et de mettre à jour le jumeau :
  
    ```cmd/sh
    python scheduleJobService.py
    ```

3. Vous voyez les réponses de l’appareil à la méthode directe et la mise à jour des jumeaux d’appareils dans la console.

    ![Exemple 1 de travail IoT Hub -- Sortie d’appareil](./media/iot-hub-python-python-schedule-jobs/sample1-deviceoutput.png)

    ![Exemple 2 de travail IoT Hub -- Sortie d’appareil](./media/iot-hub-python-python-schedule-jobs/sample2-deviceoutput.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé un travail pour planifier une méthode directe sur un appareil et la mise à jour des propriétés de représentation de l’appareil.

Pour poursuivre votre découverte d’IoT Hub et des modèles de gestion d’appareils, comme la mise à jour du microprogramme à distance, consultez [Mettre à jour un microprogramme](tutorial-firmware-update.md).
