---
title: Charger les fichiers d’un appareil sur Azure IoT Hub avec Python | Microsoft Docs
description: Guide pratique pour charger les fichiers d’un appareil sur le cloud avec Azure IoT device SDK pour Python. Les fichiers téléchargés sont stockés dans un conteneur d’objets blob de stockage Azure.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/31/2020
ms.author: robinsh
ms.custom: mqtt, devx-track-python
ms.openlocfilehash: 1112df382fb10051ac06c4461a9e4328e1663388
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92139321"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-python"></a>Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Python)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Cet article explique comment utiliser les [fonctionnalités de chargement d’IoT Hub](iot-hub-devguide-file-upload.md) pour charger un fichier sur le [Stockage Blob Azure](../storage/index.yml). Ce didacticiel explique les procédures suivantes :

* Fournir en toute sécurité un conteneur de stockage pour le chargement d’un fichier

* Utiliser le client Python pour charger un fichier par le biais de votre hub IoT

Le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-python.md) présente les fonctionnalités de messages appareil-à-cloud de base d’IoT Hub. Toutefois, dans certains scénarios, vous ne pouvez pas facilement mapper les données que vos appareils envoient dans des messages appareil-à-cloud relativement petits et acceptés par IoT Hub. Lorsque vous avez besoin de charger des fichiers à partir d’un appareil, vous pouvez quand même exploiter la sécurité et la fiabilité d’IoT Hub.

À la fin de ce didacticiel, vous allez exécuter l’application console Python :

* **FileUpload.py**, qui charge un fichier sur le stockage à l’aide du kit Python Device SDK.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-hub-include-python-v2-async-installation-notes](../../includes/iot-hub-include-python-v2-async-installation-notes.md)]

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Charger un fichier à partir d’une application d’appareil

Dans cette section, créez l’application d’appareil pour charger un fichier sur IoT Hub.

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package **azure-iot-device**. Vous utilisez ce package pour coordonner le chargement du fichier avec votre hub IoT.

    ```cmd/sh
    pip install azure-iot-device
    ```

1. À partir de votre invite de commandes, exécutez la commande suivante pour installer le package [**azure.storage.blob**](https://pypi.org/project/azure-storage-blob/). Vous utilisez ce package pour effectuer le chargement du fichier.

    ```cmd/sh
    pip install azure.storage.blob
    ```

1. Créez un fichier de test que vous téléchargerez dans le stockage Blob.

1. À l’aide d’un éditeur de texte, créez un fichier **FileUpload.py** dans votre dossier de travail.

1. Ajoutez les variables et instructions `import` ci-dessous au début du fichier **FileUpload.py**.

    ```python
    import os
    import asyncio
    from azure.iot.device.aio import IoTHubDeviceClient
    from azure.core.exceptions import AzureError
    from azure.storage.blob import BlobClient

    CONNECTION_STRING = "[Device Connection String]"
    PATH_TO_FILE = r"[Full path to local file]"
    ```

1. Dans votre fichier, remplacez `[Device Connection String]` par la chaîne de connexion de l’appareil de votre hub IoT. Remplacez `[Full path to local file]` par le chemin d’accès du fichier test que vous avez créé ou de tout fichier figurant sur votre appareil, que vous souhaitez télécharger.

1. Créez une fonction pour charger le fichier dans le stockage Blob :

    ```python
    async def store_blob(blob_info, file_name):
        try:
            sas_url = "https://{}/{}/{}{}".format(
                blob_info["hostName"],
                blob_info["containerName"],
                blob_info["blobName"],
                blob_info["sasToken"]
            )

            print("\nUploading file: {} to Azure Storage as blob: {} in container {}\n".format(file_name, blob_info["blobName"], blob_info["containerName"]))

            # Upload the specified file
            with BlobClient.from_blob_url(sas_url) as blob_client:
                with open(file_name, "rb") as f:
                    result = blob_client.upload_blob(f, overwrite=True)
                    return (True, result)

        except FileNotFoundError as ex:
            # catch file not found and add an HTTP status code to return in notification to IoT Hub
            ex.status_code = 404
            return (False, ex)

        except AzureError as ex:
            # catch Azure errors that might result from the upload operation
            return (False, ex)
    ```

    Cette fonction analyse la structure *blob_info* qui lui est transmise pour créer une URL qu’il utilise pour initialiser un [azure.storage.blob.BlobClient](/python/api/azure-storage-blob/azure.storage.blob.blobclient?view=azure-python). Ensuite, elle charge votre fichier dans le stockage Blob Azure à l’aide de ce client.

1. Pour connecter le client et charger le fichier, ajoutez le code suivant :

    ```python
    async def main():
        try:
            print ( "IoT Hub file upload sample, press Ctrl-C to exit" )

            conn_str = CONNECTION_STRING
            file_name = PATH_TO_FILE
            blob_name = os.path.basename(file_name)

            device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)

            # Connect the client
            await device_client.connect()

            # Get the storage info for the blob
            storage_info = await device_client.get_storage_info_for_blob(blob_name)

            # Upload to blob
            success, result = await store_blob(storage_info, file_name)

            if success == True:
                print("Upload succeeded. Result is: \n") 
                print(result)
                print()

                await device_client.notify_blob_upload_status(
                    storage_info["correlationId"], True, 200, "OK: {}".format(file_name)
                )

            else :
                # If the upload was not successful, the result is the exception object
                print("Upload failed. Exception is: \n") 
                print(result)
                print()

                await device_client.notify_blob_upload_status(
                    storage_info["correlationId"], False, result.status_code, str(result)
                )

        except Exception as ex:
            print("\nException:")
            print(ex)

        except KeyboardInterrupt:
            print ( "\nIoTHubDeviceClient sample stopped" )

        finally:
            # Finally, disconnect the client
            await device_client.disconnect()


    if __name__ == "__main__":
        asyncio.run(main())
        #loop = asyncio.get_event_loop()
        #loop.run_until_complete(main())
        #loop.close()
    ```

    Celui-ci crée un **IoTHubDeviceClient** asynchrone et utilise les API suivantes pour gérer le chargement de fichier avec votre hub IoT :

    * **get_storage_info_for_blob** obtient des informations de votre hub IoT concernant le compte de stockage lié que vous avez créé précédemment. Ces informations incluent le nom d’hôte, le nom du conteneur, le nom de l’objet Blob et un jeton SAP. Les informations de stockage sont transmises à la fonction **store_blob** (créée à l’étape précédente), ce qui permet au **BlobClient** dans cette fonction de s’authentifier auprès du stockage Azure. La méthode **get_storage_info_for_blob** retourne également un correlation_id qui est utilisé dans la méthode **notify_blob_upload_status**. Le correlation_id est le moyen qu’IoT Hub utilise pour marquer l’objet Blob sur lequel vous travaillez.

    * **notify_blob_upload_status** notifie IoT Hub de l’état de votre opération de stockage Blob. Vous lui transmettez le correlation_id obtenu par la méthode **get_storage_info_for_blob**. IoT Hub utilise celui-ci pour notifier à un service qui pourrait écouter dans l’attente d’une notification sur l’état de la tâche de chargement de fichier.

1. Enregistrez et fermez le fichier **UploadFile.py**.

## <a name="run-the-application"></a>Exécution de l'application

Vous êtes maintenant prêt à exécuter l’application.

1. À partir d’une invite de commandes dans votre dossier de travail, exécutez la commande suivante :

    ```cmd/sh
    python FileUpload.py
    ```

2. La capture d’écran ci-dessous montre la sortie de l’application **FileUpload** :

    ![Sortie de l’application simulated-device](./media/iot-hub-python-python-file-upload/run-device-app.png)

3. Vous pouvez utiliser le portail pour afficher le fichier chargé dans le conteneur de stockage que vous avez configuré :

    ![Fichier chargé](./media/iot-hub-python-python-file-upload/view-blob.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser les fonctionnalités de téléchargement de fichier d’IoT Hub pour simplifier les chargements de fichiers à partir d’appareils. Vous pouvez continuer à explorer les scénarios et fonctionnalités d’IoT Hub avec les articles suivants :

* [Créer un IoT Hub par programmation](iot-hub-rm-template-powershell.md)

* [Présentation du SDK C](iot-hub-device-sdk-c-intro.md)

* [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md)

Suivez les liens suivants pour en savoir plus sur Stockage Blob Azure :

* [Documentation sur Stockage Blob Azure](../storage/blobs/index.yml)

* [Documentation sur Stockage Blob Azure pour l’API Python](/python/api/overview/azure/storage-blob-readme?view=azure-python)