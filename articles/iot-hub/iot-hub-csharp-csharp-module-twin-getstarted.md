---
title: Bien démarrer avec l’identité de module et le jumeau de module IoT Hub (.NET)
description: Découvrez comment créer une identité de module et comment mettre à jour un jumeau de module à l’aide des SDK IoT pour .NET.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/07/2019
ms.author: menchi
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: daba400b9daadf464c4c125ad266745237e71367
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142502"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-net"></a>Bien démarrer avec l’identité de module et le jumeau de module IoT Hub (.NET)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Les identités de module et les jumeaux de module](iot-hub-devguide-module-twins.md) sont similaires aux identités d’appareil et aux jumeaux d’appareil Azure IoT Hub, mais offrent un plus grand niveau de détail. Contrairement à l’identité d’appareil et au jumeau d’appareil Azure IoT Hub qui permettent à l’application back-end de configurer un appareil et d’obtenir une visibilité sur les conditions de l’appareil, une identité de module et un jumeau de module fournissent ces capacités pour les composants individuels d’un appareil. Sur les appareils compatibles qui intègrent plusieurs composants, par exemple des appareils basés sur un système d’exploitation ou des appareils avec un microprogramme, les identités de module et les jumeaux de module permettent d’isoler la configuration et les conditions de chacun de ces composants.

À la fin de ce didacticiel, vous disposerez de deux applications de console .NET :

* **CreateIdentities**. Cette application crée une identité d’appareil, une identité de module et une clé de sécurité associée pour connecter votre appareil et vos clients de module.

* **UpdateModuleTwinReportedProperties**. Cette application envoie à votre hub IoT les propriétés signalées et à jour pour le jumeau de module.

> [!NOTE]
> Pour plus d’informations sur les SDK Azure IoT que vous pouvez utiliser pour générer les deux applications qui s’exécutent sur les appareils et sur le serveur de solution principal, consultez l’article [SDK Azure IoT](iot-hub-devguide-sdks.md).

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

## <a name="create-a-hub"></a>Créer un hub

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-module-twin-shared-access-policy-text](../../includes/iot-hub-howto-module-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

[!INCLUDE [iot-hub-get-started-create-module-identity-csharp](../../includes/iot-hub-get-started-create-module-identity-csharp.md)]

## <a name="update-the-module-twin-using-net-device-sdk"></a>Mettre à jour le jumeau de module à l’aide du SDK d’appareil .NET

Dans cette section, vous allez créer sur votre appareil simulé une application de console .NET qui met à jour les propriétés déclarées du jumeau de module.

Avant de commencer, obtenez la chaîne de connexion de votre module. Connectez-vous au [portail Azure](https://portal.azure.com/). Accédez à votre hub et sélectionnez **Appareils IoT**. Recherchez **myFirstDevice**. Sélectionnez **myFirstDevice** pour l’ouvrir, puis **myFirstModule** pour l’ouvrir. Dans **Détails sur l’identité de module**, copiez la **chaîne de connexion (clé primaire)** si nécessaire dans la procédure suivante.

   ![Détails du module du Portail Azure](./media/iot-hub-csharp-csharp-module-twin-getstarted/module-identity-detail.png)

1. Dans Visual Studio, ajoutez un nouveau projet à votre solution en sélectionnant **Fichier** > **Nouveau** > **Projet**. Dans Créer un projet, sélectionnez **Application console (.NET Framework)** , puis **Suivant**.

1. Nommez le projet *UpdateModuleTwinReportedProperties*. Pour **Solution**, sélectionnez **Ajouter à la solution**. Assurez-vous que la version du .NET Framework est définie sur 4.6.1 ou supérieur.

    ![Créer un projet Visual Studio](./media/iot-hub-csharp-csharp-module-twin-getstarted/configure-update-twins-csharp1.png)

1. Sélectionnez **Créer** pour créer le projet.

1. Dans Visual Studio, ouvrez **Outils** > **Gestionnaire de package NuGet** > **Gérer les packages NuGet pour la solution**. Sélectionnez l’onglet **Parcourir**.

1. Recherchez et sélectionnez **Microsoft.Azure.Devices.Client**, puis sélectionnez **Installer**.

    ![Capture d’écran montrant l’option « Microsoft.Azure.Devices.client » sélectionnée et le bouton « Installer » en surbrillance.](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-client-sdk.png)

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur de l’espace réservé par la chaîne de connexion du module.

    ```csharp
    private const string ModuleConnectionString = "<Your module connection string>";
    private static ModuleClient Client = null;
    static void ConnectionStatusChangeHandler(ConnectionStatus status, 
      ConnectionStatusChangeReason reason)
    {
        Console.WriteLine("Connection Status Changed to {0}; the reason is {1}", 
          status, reason);
    }
    ```

1. Ajoutez la méthode suivante **OnDesiredPropertyChanged** à la classe **Program** :

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
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

1. Ajoutez les lignes suivantes à la méthode **Main** :

    ```csharp
    static void Main(string[] args)
    {
        Microsoft.Azure.Devices.Client.TransportType transport = 
          Microsoft.Azure.Devices.Client.TransportType.Amqp;

        try
        {
            Client = 
              ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
            Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
            Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();

            Console.WriteLine("Retrieving twin");
            var twinTask = Client.GetTwinAsync();
            twinTask.Wait();
            var twin = twinTask.Result;
            Console.WriteLine(JsonConvert.SerializeObject(twin.Properties)); 

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
        Console.ReadLine();
        Client.CloseAsync().Wait();
    }
    ```

    Cet exemple de code montre comment récupérer le jumeau de module et mettre à jour les propriétés déclarées avec le protocole AMQP. Dans la préversion publique, AMQP est pris en charge uniquement pour les opérations de jumeau de module.

1. Si vous le souhaitez, vous pouvez ajouter ces instructions à la méthode **Main** pour envoyer un événement à IoT Hub à partir de votre module. Placez ces lignes sous le bloc `try catch`.

    ```csharp
    Byte[] bytes = new Byte[2];
    bytes[0] = 0;
    bytes[1] = 1;
    var sendEventsTask = Client.SendEventAsync(new Message(bytes));
    sendEventsTask.Wait();
    Console.WriteLine("Event sent to IoT Hub.");
    ```

## <a name="run-the-apps"></a>Exécuter les applications

Vous pouvez maintenant exécuter les applications.

1. Dans Visual Studio, dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Définir les projets de démarrage**.

1. Sous **Propriétés communes**, sélectionnez **Projet de démarrage**.

1. Sélectionnez **Plusieurs projets de démarrage**, puis choisissez l’action **Démarrer** pour les applications et **OK** pour accepter vos modifications.

1. Appuyez sur **F5** pour démarrer les applications.

## <a name="next-steps"></a>Étapes suivantes

Pour continuer la prise en main de IoT Hub et explorer les autres scénarios IoT, consultez les articles suivants :

* [Prise en main de la gestion d’appareils](iot-hub-node-node-device-management-get-started.md)

* [Bien démarrer avec IoT Edge](../iot-edge/quickstart-linux.md)