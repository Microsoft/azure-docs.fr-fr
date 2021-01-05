---
title: Planifier des travaux avec Azure IoT Hub (.NET/.NET) | Microsoft Docs
description: Procédure de planification d’une tâche Azure IoT Hub pour appeler une méthode directe sur plusieurs appareils. Vous utilisez le Kit de développement logiciel Azure IoT device SDK pour .NET afin d’implémenter les applications d’appareil simulé et une application de service pour exécuter le travail.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 0391afe5562238f488fde7e50d914c46a5c32944
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142456"
---
# <a name="schedule-and-broadcast-jobs-net"></a>Planifier et diffuser des travaux (.NET)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Utilisez Azure IoT Hub pour planifier et suivre des travaux qui mettent à jour des millions d’appareils. Utilisez les travaux pour :

* Mettre à jour les propriétés souhaitées

* Mettre à jour les balises

* Appeler des méthodes directes

Un travail encapsule l’une de ces actions et suit l’exécution sur un ensemble d’appareils, défini par une requête de jumeau d’appareil. Par exemple, une application principale peut utiliser un travail pour appeler une méthode directe sur 10 000 appareils et les redémarrer. Vous spécifiez l’ensemble des appareils avec une requête de jumeau d’appareil et planifiez le travail à exécuter ultérieurement. Le travail suit la progression à mesure que chacun de ces appareils reçoit et exécute la méthode directe de redémarrage.

Pour plus d’informations sur chacune de ces fonctionnalités, consultez les pages :

* Jumeau d’appareil et propriétés : [Bien démarrer avec des jumeaux d’appareils](iot-hub-csharp-csharp-twin-getstarted.md) et [Tutoriel : Guide pratique pour utiliser des propriétés de jumeau d’appareil](tutorial-device-twins.md)

* Méthodes directes : [Guide du développeur IoT Hub - méthodes directes](iot-hub-devguide-direct-methods.md) et [Didacticiel : Utiliser des méthodes directes](quickstart-control-device-dotnet.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Ce didacticiel vous explique les procédures suivantes :

* Création d’une application pour appareils implémentant une méthode directe nommée **LockDoor**, qui peut être appelée par l’application back-end.

* Création d’une application back-end qui crée un travail pour appeler la méthode directe **LockDoor** sur plusieurs appareils. Un autre travail envoie les mises à jour de propriétés souhaitées à plusieurs appareils.

À la fin de ce didacticiel, vous disposerez de deux applications console .NET (C#) :

* **SimulateDeviceMethods**. Cette application se connecte à votre hub IoT et implémente la méthode directe **LockDoor**.

* **ScheduleJob**. Cette application utilise des travaux pour appeler la méthode directe **LockDoor** et mettre à jour les propriétés souhaitées du jumeau d’appareil sur plusieurs appareils.

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Création d’une application de périphérique simulé

Dans cette section, vous allez créer une application console .NET qui répond à une méthode directe appelée par le serveur principal de la solution.

1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**, puis choisissez le modèle de projet **Application console (.NET Framework)** . Sélectionnez **Suivant** pour continuer.

1. Dans **Configurer votre nouveau projet**, nommez le projet *SimulateDeviceMethods*, puis sélectionnez **Créer**.

    ![Configurer votre projet SimulateDeviceMethods](./media/iot-hub-csharp-csharp-schedule-jobs/configure-device-app.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **SimulateDeviceMethods**, puis sélectionnez **Gérer les packages NuGet**.

1. Dans **Gestionnaire de package NuGet**, sélectionnez **Parcourir**, puis recherchez et choisissez **Microsoft.Azure.Devices.Client**. Sélectionnez **Installer**.

    ![Application cliente dans la fenêtre du gestionnaire de package NuGet](./media/iot-hub-csharp-csharp-schedule-jobs/device-app-nuget.png)

    Cette étape télécharge, installe et ajoute une référence au package NuGet [Azure IoT device SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) et à ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur d’espace réservé par la chaîne de connexion de l’appareil que vous avez notée dans la section précédente :

    ```csharp
    static string DeviceConnectionString = "<yourDeviceConnectionString>";
    static DeviceClient Client = null;
    ```

1. Ajoutez le code suivant pour implémenter la méthode directe sur l’appareil :

    ```csharp
    static Task<MethodResponse> LockDoor(MethodRequest methodRequest, object userContext)
    {
        Console.WriteLine();
        Console.WriteLine("Locking Door!");
        Console.WriteLine("\nReturning response for method {0}", methodRequest.Name);

        string result = "'Door was locked.'";
        return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
    }
    ```

1. Ajoutez la méthode suivante pour implémenter l’écouteur des jumeaux d’appareils sur l’appareil :

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
    {
        Console.WriteLine("Desired property change:");
        Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
    }
    ```

1. Enfin, ajoutez le code suivant à la méthode **Main** pour ouvrir la connexion à votre IoT Hub et initialiser l’écouteur de la méthode :

    ```csharp
    try
    {
        Console.WriteLine("Connecting to hub");
        Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
          TransportType.Mqtt);

        Client.SetMethodHandlerAsync("LockDoor", LockDoor, null);
        Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null);

        Console.WriteLine("Waiting for direct method call and device twin update\n Press enter to exit.");
        Console.ReadLine();

        Console.WriteLine("Exiting...");

        Client.SetMethodHandlerAsync("LockDoor", null, null);
        Client.CloseAsync().Wait();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Enregistrez votre travail et générez votre solution.

> [!NOTE]
> Pour simplifier les choses, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, une nouvelle tentative de connexion), comme suggéré dans [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-sending-device-twin-updates"></a>Planifier des travaux pour appeler une méthode directe et envoyer des mises à jour de jumeaux d’appareils

Dans cette section, vous allez créer une application console .NET (en C#) qui utilise des travaux pour appeler la méthode directe **LockDoor** et envoyer les mises à jour des propriétés souhaitées à plusieurs appareils.

1. Dans Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet**. Sous **Créer un projet**, choisissez **Application console (.NET Framework)** , puis cliquez sur **Suivant**.

1. Dans **Configurer votre nouveau projet**, nommez le projet *ScheduleJob*. Pour **Solution**, choisissez **Ajouter à la solution**, puis sélectionnez **Créer**.

    ![Nommez et configurez votre projet ScheduleJob](./media/iot-hub-csharp-csharp-schedule-jobs/config-schedule-job-app.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ScheduleJob**, puis sélectionnez **Gérer les packages NuGet**.

1. Dans le **Gestionnaire de package NuGet**, sélectionnez **Parcourir**, recherchez et choisissez **Microsoft.Azure.Devices**, puis sélectionnez **Installer**.

   Cette étape lance le téléchargement et l’installation et ajoute une référence au [package Azure IoT Service SDK NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices/) et ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices;
    using Microsoft.Azure.Devices.Shared;
    ```

1. Ajoutez l’instruction `using` suivante si elle ne figure pas déjà dans les instructions par défaut.

    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez les espaces réservés par la chaîne de connexion de l’IoT Hub que vous avez précédemment copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string), puis nommez votre appareil.

    ```csharp
    static JobClient jobClient;
    static string connString = "<yourIotHubConnectionString>";
    static string deviceId = "<yourDeviceId>";
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp
    public static async Task MonitorJob(string jobId)
    {
        JobResponse result;
        do
        {
            result = await jobClient.GetJobAsync(jobId);
            Console.WriteLine("Job Status : " + result.Status.ToString());
            Thread.Sleep(2000);
        } while ((result.Status != JobStatus.Completed) && 
          (result.Status != JobStatus.Failed));
    }
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp
    public static async Task StartMethodJob(string jobId)
    {
        CloudToDeviceMethod directMethod = 
          new CloudToDeviceMethod("LockDoor", TimeSpan.FromSeconds(5), 
          TimeSpan.FromSeconds(5));

        JobResponse result = await jobClient.ScheduleDeviceMethodAsync(jobId,
            $"DeviceId IN ['{deviceId}']",
            directMethod,
            DateTime.UtcNow,
            (long)TimeSpan.FromMinutes(2).TotalSeconds);

        Console.WriteLine("Started Method Job");
    }
    ```

1. Ajoutez une autre méthode à la classe **Program** :

    ```csharp
    public static async Task StartTwinUpdateJob(string jobId)
    {
        Twin twin = new Twin(deviceId);
        twin.Tags = new TwinCollection();
        twin.Tags["Building"] = "43";
        twin.Tags["Floor"] = "3";
        twin.ETag = "*";

        twin.Properties.Desired["LocationUpdate"] = DateTime.UtcNow;

        JobResponse createJobResponse = jobClient.ScheduleTwinUpdateAsync(
            jobId,
            $"DeviceId IN ['{deviceId}']", 
            twin, 
            DateTime.UtcNow, 
            (long)TimeSpan.FromMinutes(2).TotalSeconds).Result;

        Console.WriteLine("Started Twin Update Job");
    }
    ```

    > [!NOTE]
    > Pour plus d’informations sur la syntaxe de requête, consultez l’article [Langage de requête IoT Hub](./iot-hub-devguide-query-language.md).
    >

1. Enfin, ajoutez les lignes suivantes à la méthode **Main** :

    ```csharp
    Console.WriteLine("Press ENTER to start running jobs.");
    Console.ReadLine();

    jobClient = JobClient.CreateFromConnectionString(connString);

    string methodJobId = Guid.NewGuid().ToString();

    StartMethodJob(methodJobId);
    MonitorJob(methodJobId).Wait();
    Console.WriteLine("Press ENTER to run the next job.");
    Console.ReadLine();

    string twinUpdateJobId = Guid.NewGuid().ToString();

    StartTwinUpdateJob(twinUpdateJobId);
    MonitorJob(twinUpdateJobId).Wait();
    Console.WriteLine("Press ENTER to exit.");
    Console.ReadLine();
    ```

1. Enregistrez votre travail et générez votre solution.

## <a name="run-the-apps"></a>Exécuter les applications

Vous êtes maintenant prêt à exécuter les applications.

1. Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Définir les projets de démarrage**.

1. Sélectionnez **Propriétés communes** > **Projet de démarrage**, puis sélectionnez **Plusieurs projets de démarrage**.

1. Assurez-vous que `SimulateDeviceMethods` figure en haut de la liste, suivi de `ScheduleJob`. Définissez leurs deux actions sur **Démarrer**, puis sélectionnez **OK**.

1. Exécutez les projets en cliquant sur **Démarrer** ou accédez au menu **Déboguer** et cliquez sur **Démarrer le débogage**.

   La sortie de l’appareil et des applications principales s’affiche.

    ![Exécuter les applications pour planifier des tâches](./media/iot-hub-csharp-csharp-schedule-jobs/schedule-jobs-console-results.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé un travail pour planifier une méthode directe sur un appareil et la mise à jour des propriétés de représentation de l’appareil.

* Pour approfondir la prise en main d’IoT Hub et des modèles de gestion d’appareils, comme la mise à jour du microprogramme à distance, consultez [Didacticiel : Mettre à jour un microprogramme](tutorial-firmware-update.md).

* Pour en savoir plus sur le déploiement de l’intelligence artificielle sur des appareils de périphérie avec Azure IoT Edge, consultez [Bien démarrer avec IoT Edge](../iot-edge/quickstart-linux.md).