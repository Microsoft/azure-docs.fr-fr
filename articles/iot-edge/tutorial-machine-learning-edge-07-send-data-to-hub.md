---
title: 'Tutoriel : Envoyer des données d’appareil via une passerelle transparente - Machine Learning sur Azure IoT Edge'
description: Ce tutoriel montre comment utiliser votre machine de développement en tant qu’appareil IoT Edge simulé pour envoyer des données au hub IoT en passant par un appareil configuré en tant que passerelle transparente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 6/30/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: devx-track-csharp
ms.openlocfilehash: 50df3424892594a6817d481aa4a3d540a342854f
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932317"
---
# <a name="tutorial-send-data-via-transparent-gateway"></a>Tutoriel : Envoyer des données via une passerelle transparente

Dans cet article, nous utilisons une fois de plus la machine virtuelle de développement comme appareil simulé. Toutefois, au lieu d’envoyer des données directement au hub IoT, l’appareil les envoie à l’appareil IoT Edge configuré en tant que passerelle transparente.

Nous surveillons le fonctionnement de l’appareil IoT Edge pendant que l’appareil simulé envoie des données. Une fois que l’appareil a terminé son exécution, nous examinons les données dans notre compte de stockage pour valider que tout a fonctionné comme prévu.

Cette étape est généralement effectuée par un développeur d’appareil.

Dans cette section du tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer et exécuter un appareil de nœud terminal.
> * Vérifier que les données générées sont stockées dans votre stockage Blob Azure.
> * Vérifier que le modèle Machine Learning a classifié les données de l’appareil.

## <a name="prerequisites"></a>Prérequis

Cet article fait partie d’une série décrivant l’utilisation d’Azure Machine Learning sur IoT Edge. Chaque article de la série s’appuie sur le travail réalisé dans le cadre de l’article précédent. Si vous êtes tombé directement sur cet article, reportez-vous au [premier article](tutorial-machine-learning-edge-01-intro.md) de la série.

## <a name="review-device-harness"></a>Passer en revue le projet Device Harness

Réutilisez le [projet DeviceHarness](tutorial-machine-learning-edge-03-generate-data.md) pour simuler l’appareil en aval (ou nœud terminal). La connexion à la passerelle transparente nécessite deux éléments supplémentaires :

* Inscrivez le certificat pour que l’appareil IoT en aval approuve l’autorité de certification utilisée par le runtime IoT Edge. Dans notre cas, l’appareil en aval est la machine virtuelle de développement.
* Ajoutez le nom de domaine complet de la passerelle Edge à la chaîne de connexion de l’appareil.

Examinez le code pour voir comment ces deux éléments sont implémentés.

1. Sur votre machine de développement, ouvrez Visual Studio Code.

1. Utilisez **Fichier** > **Ouvrir le dossier...** pour ouvrir C:\\source\\IoTEdgeAndMlSample\\DeviceHarness.

1. Examinez la méthode InstallCertificate() dans le fichier Program.cs.

1. Notez que si le code recherche le chemin d’accès du certificat, il appelle la méthode CertificateManager.InstallCACert pour installer le certificat sur la machine.

1. Examinez à présent la méthode GetIotHubDevice sur la classe TurbofanDevice.

1. Lorsque l’utilisateur spécifie le nom de domaine complet de la passerelle à l’aide de l’option « -g », cette valeur est transmise à cette méthode en tant que variable `gatewayFqdn`, qui est ajoutée à la chaîne de connexion de l’appareil.

   ```csharp
   connectionString = $"{connectionString};GatewayHostName={gatewayFqdn.ToLower()}";
   ```

## <a name="build-and-run-leaf-device"></a>Créer et exécuter un appareil de nœud terminal

1. Avec le projet DeviceHarness toujours ouvert dans Visual Studio Code, générez le projet. Dans le menu **Terminal**, sélectionnez **Exécuter la tâche de génération** et sélectionnez **Génération**.

1. Recherchez le nom de domaine complet de votre passerelle Edge en accédant à votre appareil IoT Edge (machine virtuelle Linux) dans le portail Azure et en copiant la valeur de **Nom DNS** à partir de la page de présentation.

1. Démarrez votre appareil IoT (machine virtuelle Linux) s’il n’est pas déjà en cours d’exécution.

1. Ouvrez le terminal Visual Studio Code. Dans le menu **Terminal**, sélectionnez **Nouveau terminal** et exécutez la commande suivante en remplaçant `<edge_device_fqdn>` par le nom DNS que vous avez copié à partir de l’appareil IoT Edge (machine virtuelle Linux) :

   ```cmd
   dotnet run -- --gateway-host-name "<edge_device_fqdn>" --certificate C:\edgecertificates\certs\azure-iot-test-only.root.ca.cert.pem --max-devices 1
   ```

1. L’application tente d’installer le certificat sur votre machine de développement. Lorsque cela arrive, acceptez l’avertissement de sécurité.

1. Lorsque vous êtes invité à entrer la chaîne de connexion au hub IoT, cliquez sur les points de suspension ( **...** ) sur le volet Appareils Azure IoT Hub, puis sélectionnez **Copier une chaîne de connexion IoT Hub**. Collez la valeur dans le terminal.

1. Une sortie semblable à celle-ci s’affiche :

   ```output
   Found existing device: Client_001
   Using device connection string: HostName=<your hub>.azure-devices.net;DeviceId=Client_001;SharedAccessKey=xxxxxxx; GatewayHostName=iotedge-xxxxxx.<region>.cloudapp.azure.com
   Device: 1 Message count: 50
   Device: 1 Message count: 100
   Device: 1 Message count: 150
   Device: 1 Message count: 200
   Device: 1 Message count: 250
   ```

   Notez l’ajout de « GatewayHostName » à la chaîne de connexion de l’appareil, ce qui fait que l’appareil communique via IoT Hub via la passerelle transparente IoT Edge.

## <a name="check-output"></a>Vérifier la sortie

### <a name="iot-edge-device-output"></a>Sortie de l’appareil IoT Edge

La sortie du module avroFileWriter peut être facilement observée en examinant l’appareil IoT Edge.

1. Ouvrez une session SSH sur votre machine virtuelle IoT Edge.

1. Recherchez les fichiers écrits sur le disque.

   ```bash
   find /data/avrofiles -type f
   ```

1. La sortie de la commande doit se présenter comme suit :

   ```output
   /data/avrofiles/2019/4/18/22/10.avro
   ```

   Vous pouvez avoir plusieurs fichiers en fonction du minutage de l’exécution.

1. Prenez note des timestamps. Le module avroFileWriter charge les fichiers dans le cloud lorsque la dernière modification a été apportée il y a plus de 10 minutes (consultez MODIFIED\_FICHIER\_TIMEOUT dans uploader.py dans le module avroFileWriter).

1. Une fois que les 10 minutes se sont écoulées, le module doit charger les fichiers. Si le chargement est réussi, le module supprime les fichiers du disque.

### <a name="azure-storage"></a>Stockage Azure

Nous pouvons observer les résultats de notre appareil de nœud terminal qui envoie des données en examinant les comptes de stockage dans lesquels les données doivent être routées.

1. Sur la machine de développement, ouvrez Visual Studio Code.

1. Dans le panneau « Stockage Azure » de la fenêtre de l’explorateur, naviguez dans l’arborescence pour trouver votre compte de stockage.

1. Développez le nœud **Conteneurs d’objets blob**.

1. À partir du travail que nous avons effectué dans la partie précédente du tutoriel, nous nous attendons à ce que le conteneur **ruldata** contienne des messages avec une durée de vie restante. Développez le nœud **ruldata**.

1. Vous voyez alors un ou plusieurs fichiers blob nommés comme suit : `<IoT Hub Name>/<partition>/<year>/<month>/<day>/<hour>/<minute>`.

1. Cliquez avec le bouton droit sur l’un des fichiers et choisissez **Download Blob (Télécharger le blob)** pour enregistrer le fichier sur votre machine de développement.

1. Ensuite, développez le nœud **uploadturbofanfiles**. Dans l’article précédent, nous avons défini cet emplacement comme cible pour les fichiers chargés par le module avroFileWriter.

1. Cliquez avec le bouton droit sur les fichiers et choisissez **Download Blob (Télécharger le blob)** pour l’enregistrer sur votre machine de développement.

### <a name="read-avro-file-contents"></a>Lire le contenu des fichiers AVRO

Nous avons inclus un utilitaire en ligne de commande simple pour lire un fichier Avro et retourner une chaîne JSON des messages dans le fichier. Dans cette section, nous allons l’installer et l’exécuter.

1. Ouvrez un terminal dans Visual Studio Code (**Terminal** > **Nouveau terminal**).

1. Installez hubavroreader :

   ```cmd
   pip install c:\source\IoTEdgeAndMlSample\HubAvroReader
   ```

1. Utilisez hubavroreader pour lire le fichier Avro que vous avez téléchargé à partir de **ruldata**.

   ```cmd
   hubavroreader <avro file with ath> | more
   ```

1. Notez que le corps du message se présente comme nous l’attendons avec l’ID d’appareil et la durée de vie restante prédite.

   ```json
   {
       "Body": {
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "CycleTime": 1.0,
           "PredictedRul": 170.1723693909444
       },
       "EnqueuedTimeUtc": "<time>",
       "Properties": {
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "CreationTimeUtc": "01/01/0001 00:00:00",
           "EnqueuedTimeUtc": "01/01/0001 00:00:00"
       },
       "SystemProperties": {
           "connectionAuthMethod": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
           "connectionDeviceGenerationId": "636857841798304970",
           "connectionDeviceId": "aaTurbofanEdgeDevice",
           "connectionModuleId": "turbofanRouter",
           "contentEncoding": "utf-8",
           "contentType": "application/json",
           "correlationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "enqueuedTime": "<time>",
           "iotHubName": "mledgeiotwalkthroughhub"
       }
   }
   ```

1. Exécutez la même commande, en transmettant le fichier Avro que vous avez téléchargé à partir de **uploadturbofanfiles**.

1. Comme prévu, ces messages contiennent toutes les données de capteur et les paramètres opérationnels du message d’origine. Ces données peuvent servir à améliorer le modèle de durée de vie restante sur notre périphérique en mode Edge.

   ```json
   {
       "Body": {
           "CycleTime": 1.0,
           "OperationalSetting1": -0.0005000000237487257,
           "OperationalSetting2": 0.00039999998989515007,
           "OperationalSetting3": 100.0,
           "PredictedRul": 170.17236328125,
           "Sensor1": 518.6699829101562,
           "Sensor10": 1.2999999523162842,
           "Sensor11": 47.29999923706055,
           "Sensor12": 522.3099975585938,
           "Sensor13": 2388.010009765625,
           "Sensor14": 8145.31982421875,
           "Sensor15": 8.424599647521973,
           "Sensor16": 0.029999999329447746,
           "Sensor17": 391.0,
           "Sensor18": 2388.0,
           "Sensor19": 100.0,
           "Sensor2": 642.3599853515625,
           "Sensor20": 39.11000061035156,
           "Sensor21": 23.353700637817383,
           "Sensor3": 1583.22998046875,
           "Sensor4": 1396.8399658203125,
           "Sensor5": 14.619999885559082,
           "Sensor6": 21.610000610351562,
           "Sensor7": 553.969970703125,
           "Sensor8": 2387.9599609375,
           "Sensor9": 9062.169921875
       },
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "70df0c98-0958-4c8f-a422-77c2a599594f",
           "CreationTimeUtc": "0001-01-01T00:00:00+00:00",
           "EnqueuedTimeUtc": "<time>"
   }
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’explorer les ressources utilisées par ce tutoriel de bout en bout, patientez jusqu’à ce que vous ayez terminé de nettoyer les ressources que vous avez créées. Dans le cas contraire, procédez comme suit pour les supprimer :

1. Supprimez le ou les groupes de ressources créés pour contenir la machine virtuelle Dev, la machine virtuelle IoT Edge, IoT Hub, le compte de stockage, le service de l’espace de travail Machine Learning (et les ressources créées : registre de conteneurs, insights d’application, coffre de clés, compte de stockage).

1. Supprimez le projet Machine Learning dans [Azure Notebooks](https://notebooks.azure.com).

1. Si vous avez cloné le référentiel localement, fermez toutes les fenêtres PowerShell ou VS Code faisant référence au référentiel local, puis supprimez le répertoire du référentiel.

1. Si vous avez créé des certificats localement, supprimez le dossier c:\\edgeCertificates.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons utilisé notre machine virtuelle de développement pour simuler un appareil de nœud terminal envoyant des données de capteur et des données opérationnelles vers notre appareil IoT Edge. Nous avons validé que les modules sur l’appareil ont routé, classé, rendu persistantes et chargé les données en examinant le fonctionnement en temps réel de l’appareil de périphérie et en examinant les fichiers chargés sur le compte de stockage.

Pour continuer votre apprentissage des fonctionnalités IoT Edge, suivez maintenant ce tutoriel :

> [!div class="nextstepaction"]
> [Créer une hiérarchie d’appareils IoT Edge (préversion)](tutorial-nested-iot-edge.md?view=iotedge-2020-11&preserve-view=true)