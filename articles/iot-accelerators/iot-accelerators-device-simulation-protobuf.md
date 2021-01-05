---
title: Utiliser Protocol Buffers avec Device Simulation - Azure| Microsoft Docs
description: Dans ce guide pratique, vous allez apprendre à utiliser Protocol Buffers pour sérialiser des données de télémétrie envoyées depuis l'accélérateur de solution Device Simulation.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc, amqp, devx-track-csharp
ms.date: 11/06/2018
ms.author: dobett
ms.openlocfilehash: 77a795b8f704084b612a2dcbd364b8d5be0a3333
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852147"
---
# <a name="serialize-telemetry-using-protocol-buffers"></a>Sérialiser des données de télémétrie à l'aide de Protocol Buffers

Protocol Buffers (Protobuf) est un format de sérialisation binaire pour données structurées. La conception de Protobuf avait pour objectif la simplicité et la performance avec en tête d'être plus léger et plus rapide que le XML.

Device Simulation prend en charge la version **proto3** du langage Protocol Buffers.

Protobuf ayant besoin d'un code compilé pour sérialiser les données, vous devez créer une version personnalisée de Device Simulation.

Les étapes décrites dans ce guide pratique montrent comment :

1. Préparer un environnement de développement
1. Spécifier l'utilisation du format Protobuf dans un modèle d'appareil
1. Définir votre format Protobuf
1. Générer des classes Protobuf
1. Tester les topologies localement

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans ce guide pratique, vous devez disposer des éléments suivants :

* Visual Studio Code. Vous pouvez télécharger [Visual Studio Code pour Mac, Linux ou Windows](https://code.visualstudio.com/download).
* .NET Core. Vous pouvez télécharger [.NET Core pour Mac, Linux ou Windows](https://www.microsoft.com/net/download).
* Postman. Vous pouvez télécharger [Postman pour Mac, Windows ou Linux](https://www.getpostman.com/apps).
* Un [hub IoT déployé dans votre abonnement Azure](../iot-hub/iot-hub-create-through-portal.md). Vous avez besoin de la chaîne de connexion du hub IoT pour effectuer les étapes décrites dans ce guide. Vous pouvez obtenir cette chaîne de connexion sur le portail Azure.
* Une [base de données Cosmos DB déployée dans le cadre de votre abonnement Azure](../cosmos-db/create-sql-api-dotnet.md#create-account) qui utilise l'API SQL et qui est configurée avec une [cohérence forte](../cosmos-db/how-to-manage-database-account.md). Vous avez besoin de la chaîne de connexion de la base de données Cosmos DB pour effectuer les étapes décrites dans ce guide. Vous pouvez obtenir cette chaîne de connexion sur le portail Azure.
* Un [compte de stockage Azure déployé dans le cadre de votre abonnement Azure](../storage/common/storage-account-create.md). La chaîne de connexion Compte de stockage est requise pour suivre les étapes décrites dans ce guide. Vous pouvez obtenir cette chaîne de connexion sur le portail Azure.

## <a name="prepare-your-development-environment"></a>Préparer votre environnement de développement

Effectuez les tâches suivantes pour préparer votre environnement de développement :

* Téléchargez la source du microservice de simulation d’appareil.
* Téléchargez la source du microservice d’adaptateur de stockage.
* Exécutez le microservice d’adaptateur de stockage localement.

Les instructions figurant dans cet article supposent que vous utilisez Windows. Si vous utilisez un autre système d’exploitation, vous devrez changer certains chemins de fichiers et certaines commandes en fonction de votre environnement.

### <a name="download-the-microservices"></a>Télécharger les microservices

Téléchargez les [Microservices de surveillance à distance](https://github.com/Azure/remote-monitoring-services-dotnet/archive/master.zip) à partir de GitHub et décompressez-les à l'emplacement approprié sur votre ordinateur local. Ce référentiel inclut le microservice d'adaptateur de stockage dont vous avez besoin pour suivre les étapes de ce guide pratique.

Téléchargez et décompressez le [microservice de simulation d’appareil](https://github.com/Azure/device-simulation-dotnet/archive/master.zip) à partir de GitHub à un emplacement approprié sur votre machine locale.

### <a name="run-the-storage-adapter-microservice"></a>Exécuter le microservice d’adaptateur de stockage

Dans Visual Studio Code, ouvrez le dossier **remote-monitoring-services-dotnet-master\storage-adapter**. Cliquez sur les boutons **Restaurer** pour corriger les dépendances non résolues.

Ouvrez le fichier **.vscode/launch.json** et assignez la chaîne de connexion de la base de données Cosmos DB à la variable d'environnement **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

> [!NOTE]
> Pour un fonctionnement optimal, une instance de la base de données Cosmos DB est également requise dans Azure lorsque vous exécutez le microservice localement sur votre ordinateur.

Pour exécuter le microservice d'adaptateur de stockage localement, cliquez sur **Déboguer \> Démarrer le débogage**.

Dans Visual Studio Code, la fenêtre **Terminal** affiche la sortie du microservice en cours d'exécution, avec une URL pour la vérification d'intégrité du service web : <http://127.0.0.1:9022/v1/status>. Lorsque vous accédez à cette adresse, vous devez voir l’état « OK: Alive and well ».

Laissez le microservice d'adaptateur de stockage s'exécuter dans cette instance de Visual Studio Code pendant que vous effectuez les étapes suivantes.

## <a name="define-your-device-model"></a>Définir votre modèle d'appareil

Ouvrez le dossier **device-simulation-dotnet-master** que vous avez téléchargé à partir de GitHub dans une nouvelle instance de Visual Studio Code. Cliquez sur les boutons **Restaurer** pour corriger les éventuelles dépendances non résolues.

Dans ce guide pratique, vous allez créer un nouveau modèle d'appareil pour un traqueur de ressources :

1. Créez un nouveau fichier de modèle d'appareil appelé **assettracker-01.json** dans le dossier **Services\data\devicemodels**.

1. Définissez les fonctionnalités de l'appareil dans le fichier de modèle d'appareil **assettracker-01.json**. La section des données de télémétrie d'un modèle d'appareil Protobuf doit :

   * Inclure le nom de la classe Protobuf que vous générez pour votre appareil. La section suivante explique comment générer cette classe.
   * Spécifiez Protobuf comme format de message.

     ```json
     {
     "SchemaVersion": "1.0.0",
     "Id": "assettracker-01",
     "Version": "0.0.1",
     "Name": "Asset Tracker",
     "Description": "An asset tracker with location, temperature, and humidity",
     "Protocol": "AMQP",
     "Simulation": {
       "InitialState": {
         "online": true,
         "latitude": 47.445301,
         "longitude": -122.296307,
         "temperature": 38.0,
         "humidity": 62.0
       },
       "Interval": "00:01:00",
       "Scripts": [
         {
           "Type": "javascript",
           "Path": "assettracker-01-state.js"
         }
       ]
     },
     "Properties": {
       "Type": "AssetTracker",
       "Location": "Field",
       "Latitude": 47.445301,
       "Longitude": -122.296307
     },
     "Telemetry": [
       {
         "Interval": "00:00:10",
         "MessageTemplate": "{\"latitude\":${latitude},\"longitude\":${longitude},\"temperature\":${temperature},\"humidity\":${humidity}}",
         "MessageSchema": {
           "Name": "assettracker-sensors;v1",
           "ClassName": "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf.AssetTracker",
           "Format": "Protobuf",
           "Fields": {
             "latitude": "double",
             "longitude": "double",
             "temperature": "double",
             "humidity": "double"
           }
         }
       }
     ]
     }
     ```

### <a name="create-device-behaviors-script"></a>Créer le script de comportement de l'appareil

Rédigez le script définissant le comportement de votre appareil. Pour plus d'informations, voir [Créer un appareil simulé avancé](iot-accelerators-device-simulation-advanced-device.md).

## <a name="define-your-protobuf-format"></a>Définir votre format Protobuf

Une fois que vous disposez d'un modèle d'appareil et que vous avez déterminé votre format de message, vous pouvez créer un fichier **proto**. Dans le fichier **proto**, vous allez ajouter :

* Un `csharp_namespace` correspondant à la propriété **ClassName** de votre modèle d'appareil
* Un message pour chacune des structures de données à sérialiser
* Un nom et un type pour chacun des champs du message

1. Créez un nouveau fichier appelé **assettracker.proto** dans le dossier **Services\Models\Protobuf\proto**.

1. Dans le fichier **proto**, définissez la syntaxe, l'espace de noms et un schéma de message comme suit :

    ```proto
    syntax = "proto3";

    option csharp_namespace = "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf";

    message AssetTracker {
      double latitude=1;
      double longitude=2;
      double temperature=5;
      double humidity=6;
    }
    ```

Les marqueurs `=1`, `=2` présents sur chaque élément spécifient une balise unique que le champ utilise dans le codage binaire. Les nombres compris entre 1 et 15 nécessitent le codage d'un octet de moins par rapport aux nombres plus élevés.

## <a name="generate-the-protobuf-class"></a>Générer la classe Protobuf

Lorsque vous disposez d'un fichier **proto**, l'étape suivante consiste à générer les classes requises pour lire et écrire des messages. Cette étape requiert l'utilisation du compilateur Protobuf **Protoc**.

1. [Télécharger le compilateur Protobuf à partir de GitHub](https://github.com/protocolbuffers/protobuf/releases/download/v3.4.0/protoc-3.4.0-win32.zip)

1. Exécutez le compilateur, en spécifiant le répertoire source, le répertoire de destination et le nom de votre fichier **proto**. Par exemple :

    ```cmd
    protoc -I c:\temp\device-simulation-dotnet-master\Services\Models\Protobuf\proto --csharp_out=C:\temp\device-simulation-dotnet-master\Services\Models\Protobuf assettracker.proto
    ```

    Cette commande génère un fichier **Assettracker.cs** dans le dossier **Services\Models\Protobuf**.

## <a name="test-protobuf-locally"></a>Tester Protobuf localement

Dans cette section, vous allez tester le traqueur de ressources que vous avez localement créé aux sections précédentes.

### <a name="run-the-device-simulation-microservice"></a>Exécuter le microservice de simulation d’appareil

Ouvrez le fichier **.vscode/launch.json** et assignez votre :

* chaîne de connexion IoT Hub à la variable d'environnement **PCS\_IOTHUB\_CONNSTRING** ;
* chaîne de connexion Compte de stockage à la variable d'environnement **PCS\_AZURE\_STORAGE\_ACCOUNT** ;
* chaîne de connexion Cosmos DB à la variable d'environnement **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Ouvrez le fichier **WebService/Properties/launchSettings.json** et assignez votre :

* chaîne de connexion IoT Hub à la variable d'environnement **PCS\_IOTHUB\_CONNSTRING** ;
* chaîne de connexion Compte de stockage à la variable d'environnement **PCS\_AZURE\_STORAGE\_ACCOUNT** ;
* chaîne de connexion Cosmos DB à la variable d'environnement **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Ouvrez le fichier **WebService\appsettings.ini** et modifiez les paramètres comme suit :

#### <a name="configure-the-solution-to-include-your-new-device-model-files"></a>Configurer la solution pour inclure les fichiers de votre nouveau modèle d'appareil

Par défaut, les fichiers JSON et JS de votre nouveau modèle d'appareil ne sont pas copiés dans la solution créée. Vous devez les y inclure explicitement.

Pour chacun des fichiers à inclure, ajoutez une entrée dans le fichier **services\services.csproj**. Par exemple :

```xml
<None Update="data\devicemodels\assettracker-01.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
<None Update="data\devicemodels\scripts\assettracker-01-state.js">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
```

Pour exécuter le microservice localement, cliquez sur **Déboguer \> Démarrer le débogage**.

La fenêtre **Terminal** dans Visual Studio Code affiche la sortie du microservice en cours d’exécution.

Laissez le microservice de simulation d’appareil s’exécuter dans cette instance de Visual Studio Code pendant que vous effectuez les étapes suivantes.

### <a name="set-up-a-monitor-for-device-events"></a>Configurer un moniteur des événements des appareils

Dans cette section, vous utilisez l’interface Azure CLI pour configurer un moniteur d’événements qui affiche les données de télémétrie envoyées par les appareils connectés à votre hub IoT.

Le script suivant suppose que le nom de votre hub IoT est **device-simulation-test**.

```azurecli-interactive
# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Monitor telemetry sent to your hub
az iot hub monitor-events --hub-name device-simulation-test
```

Laissez le moniteur d’événements s’exécuter pendant que vous testez les appareils simulés.

### <a name="create-a-simulation-with-the-asset-tracker-device-type"></a>Créer une simulation avec le type d'appareil Traqueur de ressources

Dans cette section, vous allez utiliser l'outil Postman pour demander au microservice de simulation d'appareil de démarrer une simulation en utilisant le type d'appareil Traqueur de ressources. Postman est un outil qui permet d’envoyer des demandes REST à un service web.

Pour configurer Postman :

1. Ouvrez Postman sur votre machine locale.

1. Cliquez sur **File \> Import** (Fichier > Importer). Cliquez ensuite sur **Choose Files** (Choisir des fichiers).

1. Sélectionnez **Azure IoT Device Simulation solution accelerator.postman\_collection** et **Azure IoT Device Simulation solution accelerator.postman\_environment**, puis cliquez sur **Open** (Ouvrir).

1. Développez **Azure IoT Device Simulation solution accelerator** (Accélérateur de solution Simulation d'appareil Azure IoT) pour afficher les demandes que vous pouvez envoyer.

1. Cliquez sur **No Environment** (Aucun environnement) et sélectionnez **Azure IoT Device Simulation solution accelerator** (Accélérateur de solution Simulation d'appareil Azure IoT).

Vous avez maintenant chargé une collection et un environnement dans votre espace de travail Postman que vous pouvez utiliser pour interagir avec le microservice de simulation d’appareil.

Pour configurer et démarrer la simulation :

1. Dans la collection Postman, sélectionnez **Create asset tracker simulation** (Créer une simulation Traqueur de ressources) et cliquez sur **Send** (Envoyer). Cette demande crée quatre instances du type d'appareil Traqueur de ressources simulé.

1. Dans la fenêtre Azure CLI, la sortie du moniteur d'événements affiche les données de télémétrie provenant des appareils simulés.

Pour arrêter la simulation, sélectionnez la demande **Stop simulation** (Arrêter la simulation) dans Postman, puis cliquez sur **Send** (Envoyer).

### <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez arrêter les deux microservices exécutés localement dans leurs instances de Visual Studio Code (**Déboguer \> Arrêter le débogage**).

Si vous n’avez plus besoin des instances IoT Hub et Cosmos DB, supprimez-les de votre abonnement Azure pour éviter toute facturation inutile.

## <a name="iot-hub-support"></a>Prise en charge d'IoT Hub

Rares sont les fonctionnalités IoT Hub qui prennent nativement en charge Protobuf ou d'autres formats binaires. Par exemple, il est impossible de procéder à un routage basé sur la charge utile des messages, car IoT Hub ne sera pas en mesure de traiter cette charge utile. Toutefois, vous pouvez procéder à un routage basé sur les en-têtes des messages.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à personnaliser la simulation d’appareil afin d’utiliser Protobuf pour l’envoi de données de télémétrie, l’étape suivante consiste à visiter le dépôt GitHub pour en savoir plus sur la [simulation d’appareil](https://github.com/Azure/device-simulation-dotnet).
