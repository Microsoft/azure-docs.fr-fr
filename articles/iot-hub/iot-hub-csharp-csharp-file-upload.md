---
title: Charger des fichiers sur Azure IoT Hub à partir d’appareils avec .NET | Microsoft Docs
description: Comment charger des fichiers sur le cloud à partir d’un appareil avec Azure IoT device SDK pour .NET. Les fichiers téléchargés sont stockés dans un conteneur d’objets blob de stockage Azure.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 07/04/2017
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 8d45ad630d09a4909cf00b830df139057cc0fcaf
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142291"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-net"></a>Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (.NET)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Ce tutoriel s’appuie sur le code du tutoriel [Envoyer des messages cloud-à -appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md) pour vous montrer comment utiliser les fonctions de chargement de fichier d’IoT Hub. Cette rubrique vous explique les procédures suivantes :

* Fournissez en toute sécurité à un appareil un URI d’objet blob Azure pour le chargement d’un fichier.

* Utilisez les notifications de chargement de fichier IoT Hub pour déclencher le traitement du fichier dans votre serveur principal d’application.

Le guide de démarrage rapide [Envoyer des données de télémétrie à partir d’un appareil à un hub IoT](quickstart-send-telemetry-dotnet.md) et le tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md) illustrent les fonctionnalités de messages appareil-à-cloud et cloud-à-appareil de base offertes par IoT Hub. Le tutoriel [Configurer le routage des messages avec IoT Hub](tutorial-routing.md) décrit un moyen de stocker de façon fiable les messages appareil-à-cloud dans le stockage d’objets blob Microsoft Azure. Toutefois, dans certains scénarios, vous ne pouvez pas facilement mapper les données que vos appareils envoient dans des messages appareil-à-cloud relativement petits et acceptés par IoT Hub. Par exemple :

* Fichiers volumineux qui contiennent des images

* Vidéos

* Données de vibration échantillonnées à une fréquence élevée

* Un certain type de données prétraitées

Ces fichiers sont généralement traités par lot dans le cloud à l’aide d’outils tels que [Azure Data Factory](../data-factory/introduction.md) ou de la pile [Hadoop](../hdinsight/index.yml). Si vous avez besoin de charger des fichiers à partir d’un appareil, vous pouvez toujours exploiter la sécurité et la fiabilité d’IoT Hub.

À la fin de ce didacticiel, vous exécutez deux applications console .NET :

* **SimulatedDevice**. Cette application charge un fichier de stockage à l’aide d’un URI SAP fourni par votre IoT Hub. Il s’agit d’une version modifiée de l’application créée dans le cadre du tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md).

* **ReadFileUploadNotification**. Cette application reçoit les notifications de chargement de fichier à partir de votre hub IoT.

> [!NOTE]
> IoT Hub offre la prise en charge de nombreux langages (notamment C, Java, Python et Javascript) et plateformes d’appareils par le biais des Kits Azure IoT device SDK. Pour obtenir des instructions pas à pas expliquant comment connecter votre appareil à Azure IoT Hub, voir le [Centre de développement Azure IoT](https://azure.microsoft.com/develop/iot).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Prérequis

* Visual Studio

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Charger un fichier à partir d’une application d’appareil

Dans cette section, vous allez modifier l’application d’appareil créée dans [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md) pour recevoir des messages cloud-à-appareil en provenance du hub IoT.

1. Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur le projet **SimulatedDevice**, puis sélectionnez **Ajouter** > **Élément existant**. Recherchez un fichier image et ajoutez-le à votre projet. Ce didacticiel suppose que l’image a pour nom `image.jpg`.

1. Cliquez avec le bouton droit sur l’image, puis sélectionnez **Propriétés**. Assurez-vous que l’option **Copier dans le répertoire de sortie** est définie sur **Toujours copier**.

    ![Indiquer où mettre à jour la propriété de l’image pour Copier dans le répertoire de sortie](./media/iot-hub-csharp-csharp-file-upload/image-properties.png)

1. Au début du fichier **Program.cs** , ajoutez les instructions suivantes :

    ```csharp
    using System.IO;
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp
    private static async void SendToBlobAsync()
    {
        string fileName = "image.jpg";
        Console.WriteLine("Uploading file: {0}", fileName);
        var watch = System.Diagnostics.Stopwatch.StartNew();

        using (var sourceData = new FileStream(@"image.jpg", FileMode.Open))
        {
            await deviceClient.UploadToBlobAsync(fileName, sourceData);
        }

        watch.Stop();
        Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
    }
    ```

    La méthode `UploadToBlobAsync` utilise le nom de fichier et la source de flux du fichier à télécharger et gère le téléchargement vers le stockage. L’application console affiche le temps nécessaire pour télécharger le fichier.

1. Ajoutez la ligne suivante à la méthode **Main** juste avant `Console.ReadLine()` :

    ```csharp
    SendToBlobAsync();
    ```

> [!NOTE]
> Par souci de simplicité, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, un backoff exponentiel) comme indiqué dans [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour recevoir les messages de notification de chargement de fichiers à partir du hub IoT que vous avez créé dans [Envoyer des données de télémétrie à partir d’un appareil vers un hub IoT](quickstart-send-telemetry-dotnet.md). Pour recevoir les messages de notification de chargement de fichiers, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recevoir une notification de téléchargement de fichier

Dans cette section, vous allez écrire une application console .NET qui reçoit des messages de notification de téléchargement de fichier à partir d’IoT Hub.

1. Dans la solution Visual Studio actuelle, sélectionnez **Fichier** > **Nouveau** > **Projet**. Sous **Créer un projet**, sélectionnez **Application console (.NET Framework)** , puis cliquez sur **Suivant**.

1. Nommez le projet *ReadFileUploadNotification*. Sous **Solution**, sélectionnez **Ajouter à la solution**. Sélectionnez **Créer** pour créer le projet.

    ![Configurer le projet ReadFileUploadNotification dans Visual Studio](./media/iot-hub-csharp-csharp-file-upload/read-file-upload-project-configure.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ReadFileUploadNotification**, puis sélectionnez **Gérer les packages NuGet**.

1. Dans le **Gestionnaire de package NuGet**, sélectionnez **Parcourir**. Recherchez et sélectionnez **Microsoft.Azure.Devices**, puis sélectionnez **Installer**.

    Cette opération a pour effet de télécharger, d’installer et d’ajouter une référence au [package NuGet de l’Azure IoT service SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices/) dans le projet **ReadFileUploadNotification**.

1. Au début du fichier **Program.cs** de ce projet, ajoutez l’instruction suivante :

    ```csharp
    using Microsoft.Azure.Devices;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur de l’espace réservé `{iot hub connection string}` par la chaîne de connexion de l’IoT Hub que vous avez précédemment copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string) :

    ```csharp
    static ServiceClient serviceClient;
    static string connectionString = "{iot hub connection string}";
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp
    private async static void ReceiveFileUploadNotificationAsync()
    {
        var notificationReceiver = serviceClient.GetFileNotificationReceiver();

        Console.WriteLine("\nReceiving file upload notification from service");
        while (true)
        {
            var fileUploadNotification = await notificationReceiver.ReceiveAsync();
            if (fileUploadNotification == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received file upload notification: {0}", 
              string.Join(", ", fileUploadNotification.BlobName));
            Console.ResetColor();

            await notificationReceiver.CompleteAsync(fileUploadNotification);
        }
    }
    ```

    Notez que ce modèle de réception est le même que celui utilisé pour recevoir des messages cloud-à-appareil à partir de l’application de l’appareil.

1. Enfin, ajoutez les lignes suivantes à la méthode **Main** :

    ```csharp
    Console.WriteLine("Receive file upload notifications\n");
    serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
    ReceiveFileUploadNotificationAsync();
    Console.WriteLine("Press Enter to exit\n");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la solution, puis sélectionnez **Définir les projets de démarrage**.

1. Sous **Propriétés communes** > **Projet de démarrage**, sélectionnez **Plusieurs projets de démarrage**, puis sélectionnez l’action **Démarrer** pour **ReadFileUploadNotification** et **SimulatedDevice**. Sélectionnez **OK** pour enregistrer vos modifications.

1. Appuyez sur **F5**. Les deux applications doivent démarrer. Vous devriez voir le téléchargement terminé dans une application console et le message de notification de téléchargement reçus par l’autre application console. Vous pouvez utiliser le [portail Azure](https://portal.azure.com/) ou l’Explorateur de serveurs Visual Studio pour vérifier la présence du fichier chargé dans votre compte Azure Storage.

    ![Capture d’écran montrant l’écran de sortie](./media/iot-hub-csharp-csharp-file-upload/run-apps1.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser les fonctionnalités de téléchargement de fichier d’IoT Hub pour simplifier les chargements de fichiers à partir d’appareils. Vous pouvez continuer à explorer les scénarios et fonctionnalités d’IoT Hub avec les articles suivants :

* [Créer un IoT Hub par programmation](iot-hub-rm-template-powershell.md)

* [Présentation du SDK C](iot-hub-device-sdk-c-intro.md)

* [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)