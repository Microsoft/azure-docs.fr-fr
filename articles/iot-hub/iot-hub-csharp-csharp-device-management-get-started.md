---
title: Bien démarrer avec la gestion des appareils Azure IoT Hub (.NET/.NET) | Microsoft Docs
description: Guide d’utilisation de la gestion des appareils Azure IoT Hub pour lancer un redémarrage d’appareil à distance. Vous utilisez Azure IoT device SDK pour .NET afin d’implémenter une application d’appareil simulé qui inclut une méthode directe et Azure IoT service SDK pour .NET afin d’implémenter une application de service qui appelle la méthode directe.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 5760c574e64a3b3b4a1df12092cff44374790a90
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89018397"
---
# <a name="get-started-with-device-management-net"></a>Prise en main de la gestion d’appareils (.NET)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

Ce didacticiel vous explique les procédures suivantes :

* Utiliser le portail Azure pour créer un IoT Hub et une identité d’appareil dans celui-ci.

* Créer une application d’appareil simulé disposant d’une méthode directe permettant le redémarrage de cet appareil. Les méthodes directes sont appelées à partir du cloud.

* Créer une application console .NET qui appelle une méthode directe de redémarrage sur l’application d’appareil simulé via votre IoT Hub.

À la fin de ce didacticiel, vous disposerez de deux applications de console .NET :

* **SimulateManagedDevice**. Cette application se connecte à votre hub IoT avec l’identité d’appareil créée précédemment, reçoit une méthode directe de redémarrage, simule un redémarrage physique et indique l’heure du dernier redémarrage.

* **TriggerReboot**. Cette application appelle une méthode directe sur l’application d’appareil simulé et affiche la réponse et les propriétés signalées mises à jour.

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Déclencher un redémarrage à distance sur l’appareil à l’aide d’une méthode directe

Dans cette section, vous créez une application console .NET (à l’aide de C#) qui lance un redémarrage à distance sur un appareil avec une méthode directe. L’application utilise des requêtes du jumeau d’appareil pour déterminer l’heure du dernier redémarrage de cet appareil.

1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**.

1. Sous **Créer un projet**, recherchez et sélectionnez le modèle de projet **Application console (.NET Framework)** , puis sélectionnez **suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, nommez le projet *TriggerReboot*, puis sélectionnez .NET Framework version 4.5.1 ou ultérieure. Sélectionnez **Create** (Créer).

    ![Nouveau projet Visual C# Bureau classique Windows](./media/iot-hub-csharp-csharp-device-management-get-started/create-trigger-reboot-configure.png)

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **TriggerReboot**, puis sélectionnez **Gérer les packages NuGet**.

1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **Microsoft.Azure.Devices**. Sélectionnez **Installer** pour installer le package **Microsoft.Azure.Devices**.

    ![Fenêtre du gestionnaire de package NuGet](./media/iot-hub-csharp-csharp-device-management-get-started/create-trigger-reboot-nuget-devices.png)

   Cette étape lance le téléchargement et l’installation et ajoute une référence au [package Azure IoT Service SDK NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices/) et ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Shared;
   ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur de l’espace réservé `{iot hub connection string}` par la chaîne de connexion IoT Hub que vous avez copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

   ```csharp
   static RegistryManager registryManager;
   static string connString = "{iot hub connection string}";
   static ServiceClient client;
   static string targetDevice = "myDeviceId";
   ```

1. Ajoutez la méthode suivante à la classe **Program**.  Ce code obtient la représentation d’appareil pour le redémarrage de l’appareil et renvoie les propriétés signalées.

   ```csharp
   public static async Task QueryTwinRebootReported()
   {
       Twin twin = await registryManager.GetTwinAsync(targetDevice);
       Console.WriteLine(twin.Properties.Reported.ToJson());
   }
   ```

1. Ajoutez la méthode suivante à la classe **Program**.  Ce code lance un redémarrage à distance sur l’appareil à l’aide d’une méthode directe.

   ```csharp
   public static async Task StartReboot()
   {
       client = ServiceClient.CreateFromConnectionString(connString);
       CloudToDeviceMethod method = new CloudToDeviceMethod("reboot");
       method.ResponseTimeout = TimeSpan.FromSeconds(30);

       CloudToDeviceMethodResult result = await 
         client.InvokeDeviceMethodAsync(targetDevice, method);

       Console.WriteLine("Invoked firmware update on device.");
   }
   ```

1. Enfin, ajoutez les lignes suivantes à la méthode **Main** :

   ```csharp
   registryManager = RegistryManager.CreateFromConnectionString(connString);
   StartReboot().Wait();
   QueryTwinRebootReported().Wait();
   Console.WriteLine("Press ENTER to exit.");
   Console.ReadLine();
   ```

1. Sélectionnez **Générer** > **Générer la solution**.

> [!NOTE]
> Ce didacticiel n'effectue qu’une seule requête pour les propriétés indiquées de l’appareil. Dans le code de production, nous recommandons d’effectuer une interrogation pour détecter les modifications apportées aux propriétés indiquées.

## <a name="create-a-simulated-device-app"></a>Création d’une application de périphérique simulé

Dans cette section, vous allez :

* Créer une application console .NET qui répond à une méthode directe appelée par le cloud.

* Déclencher un redémarrage d’appareil simulé.

* Utiliser les propriétés signalées pour activer les requêtes sur le jumeau d’appareil afin d’identifier les appareils et l’heure de leur dernier redémarrage.

Pour créer l’application d’appareil simulé, procédez comme suit :

1. Dans Visual Studio, dans la solution TriggerReboot que vous avez déjà créée, sélectionnez **Fichier** > **Nouveau** > **Projet**. Sous **Créer un projet**, recherchez et sélectionnez le modèle de projet **Application console (.NET Framework)** , puis sélectionnez **suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, nommez le projet *SimulateManagedDevice*. Sous **Solution**, sélectionnez **Ajouter à la solution**. Sélectionnez **Create** (Créer).

    ![Nommer et ajouter votre projet à la solution](./media/iot-hub-csharp-csharp-device-management-get-started/configure-device-app.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nouveau projet **SimulateManagedDevice**, puis sélectionnez **Gérer les packages NuGet**.

1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **Microsoft.Azure.Devices.Client**. Sélectionnez **Installer**.

    ![Application cliente dans la fenêtre du gestionnaire de package NuGet](./media/iot-hub-csharp-csharp-device-management-get-started/create-device-nuget-devices-client.png)

   Cette étape télécharge, installe et ajoute une référence au package NuGet [Azure IoT device SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) et à ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur de l’espace réservé `{device connection string}` par la chaîne de connexion de l’appareil que vous avez précédemment notée dans [Inscrire un nouvel appareil dans le hub IoT](#register-a-new-device-in-the-iot-hub).

    ```csharp
    static string DeviceConnectionString = "{device connection string}";
    static DeviceClient Client = null;
    ```

1. Ajoutez le code suivant pour implémenter la méthode directe sur l’appareil :

   ```csharp
   static Task<MethodResponse> onReboot(MethodRequest methodRequest, object userContext)
   {
       // In a production device, you would trigger a reboot 
       //   scheduled to start after this method returns.
       // For this sample, we simulate the reboot by writing to the console
       //   and updating the reported properties.
       try
       {
           Console.WriteLine("Rebooting!");

           // Update device twin with reboot time. 
           TwinCollection reportedProperties, reboot, lastReboot;
           lastReboot = new TwinCollection();
           reboot = new TwinCollection();
           reportedProperties = new TwinCollection();
           lastReboot["lastReboot"] = DateTime.Now;
           reboot["reboot"] = lastReboot;
           reportedProperties["iothubDM"] = reboot;
           Client.UpdateReportedPropertiesAsync(reportedProperties).Wait();
       }
       catch (Exception ex)
       {
           Console.WriteLine();
           Console.WriteLine("Error in sample: {0}", ex.Message);
       }

       string result = @"{""result"":""Reboot started.""}";
       return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
   }
   ```

1. Enfin, ajoutez le code suivant à la méthode **Main** pour ouvrir la connexion à votre IoT Hub et initialiser l’écouteur de la méthode :

   ```csharp
   try
   {
       Console.WriteLine("Connecting to hub");
       Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
         TransportType.Mqtt);

       // setup callback for "reboot" method
       Client.SetMethodHandlerAsync("reboot", onReboot, null).Wait();
       Console.WriteLine("Waiting for reboot method\n Press enter to exit.");
       Console.ReadLine();

       Console.WriteLine("Exiting...");

       // as a good practice, remove the "reboot" handler
       Client.SetMethodHandlerAsync("reboot", null, null).Wait();
       Client.CloseAsync().Wait();
   }
   catch (Exception ex)
   {
       Console.WriteLine();
       Console.WriteLine("Error in sample: {0}", ex.Message);
   }
   ```

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Définir les projets de démarrage**.

1. Sous **Propriétés communes** > **Projet de démarrage**, sélectionnez **Projet de démarrage unique**, puis sélectionnez le projet **SimulateManagedDevice**. Sélectionnez **OK** pour enregistrer vos modifications.

1. Sélectionnez **Générer** > **Générer la solution**.

> [!NOTE]
> Pour simplifier les choses, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, un backoff exponentiel) comme indiqué dans [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).

## <a name="run-the-apps"></a>Exécuter les applications

Vous êtes maintenant prêt à exécuter les applications.

1. Pour exécuter l’application d’appareil .NET **SimulateManagedDevice**, dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **SimulateManagedDevice**, sélectionnez **Déboguer**, puis sélectionnez **Démarrer une nouvelle instance**. L’application doit commencer à écouter les appels de méthode provenant de votre hub IoT.

1. Une fois que l’appareil est connecté et en attente d’appels de méthode, cliquez avec le bouton droit sur le projet **TriggerReboot**, sélectionnez **Déboguer**, puis sélectionnez **Démarrer une nouvelle instance**.

   « Rebooting! » (Redémarrage en cours !) doit s’afficher dans la console **SimulatedManagedDevice** et les propriétés indiquées de l’appareil, qui incluent l’heure du dernier redémarrage, écrit dans la console **TriggerReboot**.

    ![Exécution d’application de service et d’appareil](./media/iot-hub-csharp-csharp-device-management-get-started/combinedrun.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
