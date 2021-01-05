---
title: Identité de module et jumeau de module Azure IoT Hub (portail et .NET)
description: Découvrez comment créer une identité de module et comment mettre à jour un jumeau de module à l’aide du portail et de .NET.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: 0d16d7a6dbf903ae790015c12e3d34e4a2a553d3
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92139350"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-the-portal-and-net-device"></a>Prise en main de l’identité de module et du jumeau de module IoT Hub à l’aide du portail et d’un appareil .NET

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Les identités de module et les jumeaux de module](iot-hub-devguide-module-twins.md) sont similaires aux identités d’appareil et aux jumeaux d’appareil Azure IoT Hub, mais offrent un plus grand niveau de détail. Contrairement à l’identité d’appareil et au jumeau d’appareil Azure IoT Hub qui permettent à l’application back-end de configurer un appareil et d’obtenir une visibilité sur les conditions de l’appareil, une identité de module et un jumeau de module fournissent ces capacités pour les composants individuels d’un appareil. Sur les appareils compatibles qui intègrent plusieurs composants, par exemple des appareils basés sur un système d’exploitation ou des appareils avec un microprogramme, les identités de module et les jumeaux de module permettent d’isoler la configuration et les conditions de chacun de ces composants.
>

Ce tutoriel vous apprendra à effectuer les opérations suivantes :

* Créer une identité de module dans le portail.

* Utiliser un SDK d’appareil .NET pour mettre à jour le jumeau de module à partir de votre appareil.

> [!NOTE]
> Pour plus d’informations sur les SDK Azure IoT que vous pouvez utiliser pour générer les deux applications à exécuter sur des appareils et sur le back-end de la solution, consultez [Kits SDK Azure IoT](iot-hub-devguide-sdks.md).
>

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

## <a name="create-a-hub"></a>Créer un hub

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-hub"></a>Inscrire un nouvel appareil dans le hub

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-module-identity-in-the-portal"></a>Créer une identité de module dans le portail

Vous pouvez créer jusqu’à 20 identités de module dans une même identité d’appareil. Pour ajouter une identité, effectuez les étapes suivantes :

1. Pour l’appareil que vous avez créé dans la section précédente, choisissez **Ajouter l’identité de module** pour créer votre première identité de module.

1. Entrez le nom *myFirstModule*. Enregistrez votre identité de module.

    ![Ajouter une identité de module](./media/iot-hub-portal-csharp-module-twin-getstarted/add-module-identity.png)

    Votre nouvelle identité de module s’affiche en bas de l’écran. Sélectionnez-la pour voir les détails de l’identité de module.

    ![Voir les détails de l’identité de module](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

Enregistrez la **chaîne de connexion (clé primaire)** . Vous l’utiliserez dans la section suivante pour configurer votre module sur l’appareil.

## <a name="update-the-module-twin-using-net-device-sdk"></a>Mettre à jour le jumeau de module à l’aide du SDK d’appareil .NET

Vous venez de créer votre identité de module dans votre IoT Hub. Vous allez maintenant tenter d’établir une communication entre votre appareil simulé et le cloud. Une fois que vous avez créé une identité de module, un jumeau de module est implicitement créé dans IoT Hub. Dans cette section, vous allez créer sur votre appareil simulé une application de console .NET qui met à jour les propriétés déclarées du jumeau de module.

### <a name="create-a-visual-studio-project"></a>Créer un projet Visual Studio

Pour créer une application qui met à jour les propriétés signalées du jumeau de module, effectuez les étapes suivantes :

1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**, choisissez **Application console (.NET Framework)** , pus sélectionnez **Suivant**.

1. Dans **Configurer votre nouveau projet**, entrez *UpdateModuleTwinReportedProperties* comme **Nom du projet**. Sélectionnez **Créer** pour continuer.

    ![Configurer votre projet Visual Studio](./media/iot-hub-portal-csharp-module-twin-getstarted/configure-twins-project.png)

### <a name="install-the-latest-azure-iot-hub-net-device-sdk"></a>Installer le dernier SDK d’appareil .NET Azure IoT Hub

L’identité de module et le jumeau de module sont disponibles en préversion publique. Ils sont uniquement disponibles dans les kits SDK d’appareil IoT Hub en préversion. Pour installer le SDK, effectuez les étapes suivantes :

1. Dans Visual Studio, ouvrez **Outils** > **Gestionnaire de package NuGet** > **Gérer les packages NuGet pour la solution**.

1. Sélectionnez **Parcourir**, puis **Inclure la préversion**. Recherchez *Microsoft.Azure.Devices.Client*. Sélectionnez la version la plus récente et installez-la.

    ![Installer la préversion du SDK du service .NET Azure IoT Hub](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-sdk.png)

    Vous avez maintenant accès à toutes les fonctionnalités de module.

### <a name="get-your-module-connection-string"></a>Obtenir votre chaîne de connexion du module

Vous avez besoin de la chaîne de connexion du module pour votre application console. Procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Accédez à votre hub IoT, puis sélectionnez **Appareils IoT**. Ouvrez **myFirstDevice**. Vous voyez alors que **myFirstModule** a bien été créé.

1. Sélectionnez **myFirstModule** sous **Identités de module**. Dans **Détails sur l’identité de module**, copiez la **Chaîne de connexion (clé primaire)** .

    ![Détails du module du Portail Azure](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

### <a name="create-updatemoduletwinreportedproperties-console-app"></a>Créer l’application de console UpdateModuleTwinReportedProperties

Pour créer votre application, effectuez les étapes suivantes :

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

  ```csharp
  using Microsoft.Azure.Devices.Client;
  using Microsoft.Azure.Devices.Shared;
  using Newtonsoft.Json;
  ```

2. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur de l’espace réservé par la chaîne de connexion du module.

  ```csharp
  private const string ModuleConnectionString = "<Your module connection string>";
  private static ModuleClient Client = null;
  ```

3. Ajoutez la méthode suivante **OnDesiredPropertyChanged** à la classe **Program** :

  ```csharp
  private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
      {
          Console.WriteLine("desired property change:");
          Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
          Console.WriteLine("Sending current time as reported property");
          TwinCollection reportedProperties = new TwinCollection
          {
              ["DateTimeLastDesiredPropertyChangeReceived"] = DateTime.Now
          };
  
          await Client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
      }
  ```

4. Enfin, remplacez la méthode **Main** par le code suivant :

  ```csharp
  static void Main(string[] args)
  {
      Microsoft.Azure.Devices.Client.TransportType transport = Microsoft.Azure.Devices.Client.TransportType.Amqp;
  
      try
      {
          Client = ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
          Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
          Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
  
          Console.WriteLine("Retrieving twin");
          var twinTask = Client.GetTwinAsync();
          twinTask.Wait();
          var twin = twinTask.Result;
          Console.WriteLine(JsonConvert.SerializeObject(twin));
  
          Console.WriteLine("Sending app start time as reported property");
          TwinCollection reportedProperties = new TwinCollection();
          reportedProperties["DateTimeLastAppLaunch"] = DateTime.Now;
  
          Client.UpdateReportedPropertiesAsync(reportedProperties);
      }
      catch (AggregateException ex)
      {
          Console.WriteLine("Error in sample: {0}", ex);
      }
  
      Console.WriteLine("Waiting for Events.  Press enter to exit...");
      Console.ReadKey();
      Client.CloseAsync().Wait();
  }
  
  private static void ConnectionStatusChangeHandler(ConnectionStatus status, ConnectionStatusChangeReason reason)
  {
      Console.WriteLine($"Status {status} changed: {reason}");
  }
  ```
  
  Vous pouvez générer et exécuter cette application à l’aide de la touche **F5**.

Cet exemple de code montre comment récupérer le jumeau de module et mettre à jour les propriétés déclarées avec le protocole AMQP. Dans la préversion publique, AMQP est pris en charge uniquement pour les opérations de jumeau de module.

## <a name="next-steps"></a>Étapes suivantes

Pour continuer la prise en main de IoT Hub et explorer les autres scénarios IoT, consultez les articles suivants :

* [Bien démarrer avec le jumeau de module et l’identité de module IoT Hub à l’aide d’une sauvegarde .NET et d’un appareil .NET](iot-hub-csharp-csharp-module-twin-getstarted.md)

* [Bien démarrer avec IoT Edge](../iot-edge/quickstart-linux.md)