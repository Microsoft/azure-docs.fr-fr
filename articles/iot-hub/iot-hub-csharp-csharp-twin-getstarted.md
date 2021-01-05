---
title: Bien démarrer avec les jumeaux d’appareil Azure IoT Hub (.NET/.NET) | Microsoft Docs
description: Guide d’utilisation des jumeaux d’appareils Azure IoT Hub pour ajouter des balises, puis utiliser une requête IoT Hub. Vous utilisez le kit Azure IoT device SDK pour .NET pour implémenter l’application d’appareil simulé et le kit Azure IoT service SDK pour .NET pour implémenter une application de service qui ajoute des balises et exécute la requête IoT Hub.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/26/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 267a69486dc91ef95c0de704346eeb1d1780ef48
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89013756"
---
# <a name="get-started-with-device-twins-net"></a>Prise en main des représentations d’appareils (.NET)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

Dans ce tutoriel, vous allez créer les applications console .NET suivantes :

* **AddTagsAndQuery**. Cette application back-end ajoute des balises et interroge les jumeaux d’appareil.

* **ReportConnectivity**. Cette application d’appareil simule un appareil qui se connecte à votre hub IoT avec l’identité d’appareil créée précédemment et signale son état de connectivité.

> [!NOTE]
> L’article relatif aux [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md) fournit des informations sur les Kits de développement logiciel (SDK) Azure que l’on peut utiliser pour générer des applications pour périphérique et des applications principales.
>

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Créer l’application de service

Dans cette section, vous créez une application console .NET utilisant C# qui ajoute des métadonnées d’emplacement au jumeau d’appareil associé à **myDeviceId**. L’application console interroge ensuite les jumeaux d’appareil stockés dans le hub IoT en sélectionnant les appareils situés aux États-Unis, puis ceux qui signalent une connexion mobile.

1. Dans Visual Studio, sélectionnez **Créer un nouveau projet**. Dans **Créer un nouveau projet**, sélectionnez **Application console (.NET Framework)** , puis **Suivant**.

1. Dans **Configurer votre nouveau projet**, nommez le projet **AddTagsAndQuery**.

    ![Configurer votre projet AddTagsAndQuery](./media/iot-hub-csharp-csharp-twin-getstarted/config-addtagsandquery-app.png)

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **AddTagsAndQuery**, puis sélectionnez **Gérer les packages NuGet**.

1. Sélectionnez **Parcourir**, puis recherchez et sélectionnez **Microsoft.Azure.Devices**. Sélectionnez **Installer**.

    ![Fenêtre du gestionnaire de package NuGet](./media/iot-hub-csharp-csharp-twin-getstarted/nuget-package-addtagsandquery-app.png)

   Cette étape lance le téléchargement et l’installation et ajoute une référence au [package Azure IoT Service SDK NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices/) et ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez `{iot hub connection string}` par la chaîne de connexion IoT Hub que vous avez copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

    ```csharp  
    static RegistryManager registryManager;
    static string connectionString = "{iot hub connection string}";
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp  
    public static async Task AddTagsAndQuery()
    {
        var twin = await registryManager.GetTwinAsync("myDeviceId");
        var patch =
            @"{
                tags: {
                    location: {
                        region: 'US',
                        plant: 'Redmond43'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
        var twinsInRedmond43 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43: {0}", 
          string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));

        query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
        var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43 using cellular network: {0}", 
          string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
    }
    ```

    La classe **RegistryManager** expose toutes les méthodes requises pour interagir avec des jumeaux d’appareil à partir du service. Le code précédent initialise l’objet **registryManager**, récupère le jumeau d’appareil de **myDeviceId**, puis met à jour ses balises avec les informations d’emplacement souhaitées.

    Après la mise à jour, il exécute deux requêtes : la première sélectionne uniquement les jumeaux d’appareils situés dans l’usine **Redmond43** et la seconde affine la requête pour sélectionner uniquement les appareils qui sont également connectés via un réseau cellulaire.

    Quand il crée l’objet **query**, le code précédent spécifie un nombre maximal de documents retournés. L’objet **query** contient une propriété booléenne **HasMoreResults** permettant d’appeler les méthodes **GetNextAsTwinAsync** plusieurs fois afin de récupérer tous les résultats. Une méthode appelée **GetNextAsJson** est disponible pour les résultats qui ne sont pas des jumeaux d’appareil, par exemple, les résultats de requêtes d’agrégation.

1. Enfin, ajoutez les lignes suivantes à la méthode **Main** :

    ```csharp  
    registryManager = RegistryManager.CreateFromConnectionString(connectionString);
    AddTagsAndQuery().Wait();
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. Exécutez cette application en cliquant avec le bouton droit sur le projet **AddTagsAndQuery** et en sélectionnant **Déboguer**, puis **Démarrer une nouvelle instance**. Vous devriez voir un appareil dans les résultats de la requête demandant tous les appareils situés à **Redmond43**, et aucun pour la requête limitant les résultats aux appareils utilisant un réseau cellulaire.

    ![Résultats de requête dans la fenêtre](./media/iot-hub-csharp-csharp-twin-getstarted/addtagapp.png)

Dans la section suivante, vous allez créer une application d’appareil qui transmet les informations de connectivité et modifie le résultat de la requête de la section précédente.

## <a name="create-the-device-app"></a>Créer l’application pour appareil

Dans cette section, vous allez créer une application console .NET qui se connecte à votre hub en tant que **myDeviceId**, puis met à jour ses propriétés signalées afin qu’elles contiennent les informations indiquant qu’elle est connectée par le biais d’un réseau mobile.

1. Dans Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet**. Dans **Créer un projet**, choisissez **Application console (.NET Framework)** , puis sélectionnez **Suivant**.

1. Dans **Configurer votre nouveau projet**, nommez le projet **ReportConnectivity**. Pour **Solution**, choisissez **Ajouter à la solution**, puis sélectionnez **Créer**.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ReportConnectivity**, puis sélectionnez **Gérer les packages NuGet**.

1. Sélectionnez **Parcourir**, puis recherchez et choisissez **Microsoft.Azure.Devices.Client**. Sélectionnez **Installer**.

   Cette étape télécharge, installe et ajoute une référence au package NuGet [Azure IoT device SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) et à ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez `{device connection string}` par la chaîne de connexion de l’appareil que vous avez notée dans [Inscrire un nouvel appareil dans le hub IoT](#register-a-new-device-in-the-iot-hub).

    ```csharp  
    static string DeviceConnectionString = "HostName=<yourIotHubName>.azure-devices.net;DeviceId=<yourIotDeviceName>;SharedAccessKey=<yourIotDeviceAccessKey>";
    static DeviceClient Client = null;
    ```

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp
    public static async void InitClient()
    {
        try
        {
            Console.WriteLine("Connecting to hub");
            Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
              TransportType.Mqtt);
            Console.WriteLine("Retrieving twin");
            await Client.GetTwinAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

    L’objet **Client** expose toutes les méthodes requises pour interagir avec des jumeaux d’appareil à partir de l’appareil. Le code présenté ci-dessus initialise l’objet **Client**, puis récupère le jumeau d’appareil pour **myDeviceId**.

1. Ajoutez la méthode suivante à la classe **Program** :

    ```csharp  
    public static async void ReportConnectivity()
    {
        try
        {
            Console.WriteLine("Sending connectivity data as reported property");

            TwinCollection reportedProperties, connectivity;
            reportedProperties = new TwinCollection();
            connectivity = new TwinCollection();
            connectivity["type"] = "cellular";
            reportedProperties["connectivity"] = connectivity;
            await Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

   Le code ci-dessus met à jour la propriété signalée de **myDeviceId** avec les informations de connectivité.

1. Enfin, ajoutez les lignes suivantes à la méthode **Main** :

    ```csharp
    try
    {
        InitClient();
        ReportConnectivity();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Définir les projets de démarrage**.

1. Dans **Propriétés communes** > **Projet de démarrage**, sélectionnez **Plusieurs projets de démarrage**. Pour **ReportConnectivity**,sélectionnez **Démarrer** comme **Action**. Sélectionnez **OK** pour enregistrer vos modifications.  

1. Exécutez cette application en cliquant avec le bouton droit sur le projet **ReportConnectivity** et en sélectionnant **Déboguer**, puis **Démarrer une nouvelle instance**. Vous devez voir l’application obtenir les informations relatives au jumeau puis envoyer la connectivité sous la forme d’une ***propriété signalée***.

    ![Exécuter l’application pour appareil pour signaler la connectivité](./media/iot-hub-csharp-csharp-twin-getstarted/rundeviceapp.png)

   Une fois que l’appareil a signalé ses informations de connectivité, il doit apparaître dans les deux requêtes.

1. Cliquez avec le bouton droit sur le projet **AddTagsAndQuery**, puis sélectionnez **Déboguer** > **Démarrer une nouvelle instance** pour réexécuter les requêtes. Cette fois, **myDeviceId** doit apparaître dans les résultats des deux requêtes.

    ![Connectivité des appareils signalée avec succès](./media/iot-hub-csharp-csharp-twin-getstarted/tagappsuccess.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré un nouveau IoT Hub dans le portail Azure, puis créé une identité d’appareil dans le registre des identités de l’IoT Hub. Vous avez ajouté des métadonnées d’appareil en tant que balises à partir d’une application principale et écrit une application pour appareil simulée pour signaler des informations de connectivité d’appareil dans le jumeau d’appareil. Vous avez également appris à interroger ces informations à l’aide du langage de requête IoT Hub de type SQL.

Vous pouvez en apprendre davantage à partir des ressources suivantes :

* Pour savoir comment envoyer des données de télémétrie à partir d’appareils, consultez le tutoriel [Envoyer des données de télémétrie à partir d’un appareil vers un hub IoT](quickstart-send-telemetry-dotnet.md).

* Pour savoir comment configurer des appareils à l’aide des propriétés souhaitées de jumeau d’appareil, consultez le tutoriel [Utiliser des propriétés souhaitées pour configurer des appareils](tutorial-device-twins.md).

* Pour savoir comment contrôler des appareils de façon interactive, par exemple mettre en marche un ventilateur à partir d’une application contrôlée par l’utilisateur, consultez le tutoriel [Utiliser des méthodes directes](quickstart-control-device-dotnet.md).
