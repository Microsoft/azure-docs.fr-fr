---
title: Fichier Include
description: Fichier Include
services: iot-hub
author: chrissie926
ms.service: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: menchi
ms.custom: include file
ms.openlocfilehash: a5c1ddd085ae65b9920d73f50f993f4646785a69
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68883831"
---
## <a name="create-a-module-identity"></a>Création d’un module d’identité

Dans cette section, vous allez créer une application console .NET qui crée une identité d’appareil et une identité de module dans le registre d’identités de votre Hub. Un appareil ou un module ne peut pas se connecter à Hub, à moins de posséder une entrée dans le registre des identités. Pour plus d’informations, consultez la [section Registre des identités du Guide du développeur IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

En exécutant cette application console, une clé et un ID uniques sont générés pour chaque appareil et module. Votre appareil et le module utilisent ces valeurs pour s’identifier lorsqu’ils envoient des messages d’appareil-à-cloud à IoT Hub. Les ID sont sensibles à la casse.

1. Ouvrez Visual Studio et sélectionnez **Créer un projet**.

1. Dans **Créer un projet**, sélectionnez **Application console (.NET Framework)** .

1. Sélectionnez **Suivant** pour ouvir **Configurer votre nouveau projet**. Nommez le projet *CreateIdentity* et nommez la solution *IoTHubGetStarted*. Assurez-vous que la version du .NET Framework est définie sur 4.6.1 ou supérieur.

    ![Entrez un nom et un framework pour votre solution Visual Studio](./media/iot-hub-get-started-create-module-identity-csharp/configure-createidentities-project.png)

1. Dans Visual Studio, ouvrez **Outils** > **Gestionnaire de package NuGet** > **Gérer les packages NuGet pour la solution**. Sélectionnez l’onglet **Parcourir**.

1. Recherchez **Microsoft.Azure.Devices**. Sélectionnez-le, puis sélectionnez **Installer**.

    ![Installer la version actuelle du kit SDK du service .NET Azure IoT Hub](./media/iot-hub-get-started-create-module-identity-csharp/install-service-sdk.png)

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Common.Exceptions;
   ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur d’espace réservé par la chaîne de connexion IoT Hub pour le hub créé dans la section précédente.

   ```csharp
   const string connectionString = "<replace_with_iothub_connection_string>";
   const string deviceID = "myFirstDevice";
   const string moduleID = "myFirstModule";
   ```

1. Ajoutez le code suivant à la classe **Main**.

   ```csharp
   static void Main(string[] args)
   {
       AddDeviceAsync().Wait();
       AddModuleAsync().Wait();
   }
   ```

1. Ajoutez les méthodes suivantes à la classe **Program** :

    ```csharp
    private static async Task AddDeviceAsync()
    {
       RegistryManager registryManager = 
         RegistryManager.CreateFromConnectionString(connectionString);
       Device device;

       try
       {
           device = await registryManager.AddDeviceAsync(new Device(deviceID));
       }
       catch (DeviceAlreadyExistsException)
        {
            device = await registryManager.GetDeviceAsync(deviceID);
        }

        Console.WriteLine("Generated device key: {0}", 
          device.Authentication.SymmetricKey.PrimaryKey);
    }

    private static async Task AddModuleAsync()
    {
        RegistryManager registryManager = 
          RegistryManager.CreateFromConnectionString(connectionString);
        Module module;

        try
        {
            module = 
              await registryManager.AddModuleAsync(new Module(deviceID, moduleID));
        }
        catch (ModuleAlreadyExistsException)
        {
            module = await registryManager.GetModuleAsync(deviceID, moduleID);
        }

        Console.WriteLine("Generated module key: {0}", module.Authentication.SymmetricKey.PrimaryKey);
    }
    ```

    La méthode `AddDeviceAsync` crée une identité d’appareil avec l’ID **myFirstDevice**. Si cet ID d’appareil existe déjà dans le registre d’identité, le code récupère simplement les informations d’appareil existantes. L’application affiche ensuite la clé primaire pour cette identité. Vous utilisez cette clé dans l’application de périphérique simulé pour vous connecter à votre Hub.

    La méthode `AddModuleAsync` crée une identité de module avec l’ID **myFirstModule** sous l’appareil **myFirstDevice**. Si cet ID de module existe déjà dans le registre d’identité, le code récupère simplement les informations existantes du module. L’application affiche ensuite la clé primaire pour cette identité. Vous utilisez cette clé dans l’application de module simulé pour vous connecter à votre Hub.

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

1. Exécutez cette application et notez la clé de l’appareil et la clé du module.

> [!NOTE]
> Le registre des identités IoT Hub stocke uniquement les identités des appareils et des modules pour permettre un accès sécurisé à Hub. Le registre des identités stocke les ID et les clés d’appareil à utiliser en tant qu’informations d’identification de sécurité. Il stocke également un indicateur activé/désactivé pour chaque appareil pouvant être utilisé pour désactiver l’accès de cet appareil. Si votre application a besoin de stocker d’autres métadonnées spécifiques aux appareils, elle doit utiliser un magasin spécifique aux applications. Il n’y a aucun indicateur d’activation/désactivation pour les identités de module. Pour plus d’informations, reportez-vous au [Guide du développeur IoT Hub](../articles/iot-hub/iot-hub-devguide-identity-registry.md).
