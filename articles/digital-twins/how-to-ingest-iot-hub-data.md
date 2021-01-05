---
title: Ingérer la télémétrie depuis IoT Hub
titleSuffix: Azure Digital Twins
description: Découvrez comment ingérer les messages de télémétrie des appareils provenant d’IoT Hub.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 9/15/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: d2606f793c7ab2e3ac29b1eb869e60a2c8e634ad
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145920"
---
# <a name="ingest-iot-hub-telemetry-into-azure-digital-twins"></a>Ingérer des données de télémétrie IoT Hub dans Azure Digital Twins

Azure Digital Twins se base sur les données des appareils IoT et d’autres sources. Pour les données d’appareil, il est courant d’utiliser la source [IoT Hub](../iot-hub/about-iot-hub.md) dans Azure Digital Twins.

Le processus d’ingestion des données dans Azure Digital Twins consiste à configurer une ressource de calcul externe, par exemple une [fonction Azure](../azure-functions/functions-overview.md), qui reçoit les données et utilise les [API DigitalTwins](/rest/api/digital-twins/dataplane/twins) pour définir des propriétés ou déclencher des événements de télémétrie en conséquence pour les [jumeaux numériques](concepts-twins-graph.md). 

Ce document explique le processus d’écriture d’une fonction Azure pouvant ingérer des données de télémétrie à partir d’IoT Hub.

## <a name="prerequisites"></a>Prérequis

Avant de poursuivre avec cet exemple, vous devez mettre en place les ressources suivantes en guise de conditions préalables :
* **Un hub IoT**. Pour obtenir des instructions, consultez la section *Créer un hub IoT* de [ce démarrage rapide d’IoT Hub](../iot-hub/quickstart-send-telemetry-cli.md).
* **Une fonction Azure** avec les autorisations appropriées pour appeler votre instance de jumeau numérique. Pour obtenir des instructions, consultez [*Procédure : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md). 
* **Une instance Azure Digital Twins** qui recevra la télémétrie de votre appareil. Pour obtenir des instructions, consultez [*Procédure : Configurer une instance Azure Digital Twins et l’authentification*](./how-to-set-up-instance-portal.md).

### <a name="example-telemetry-scenario"></a>Exemple de scénario de télémétrie

Cet article explique comment envoyer des messages entre IoT Hub et Azure Digital Twins à l’aide d’une fonction Azure. Pour envoyer des messages, il existe de nombreuses configurations et stratégies possibles. Cependant, l’exemple de cet article contient les éléments suivants :
* Un thermomètre situé dans IoT Hub, ayant un ID d’appareil connu
* Un jumeau numérique représentant l’appareil, avec un ID correspondant

> [!NOTE]
> Cet exemple utilise une correspondance d’ID simple entre l’ID de l’appareil et celui d’un jumeau numérique. Toutefois, il est possible de fournir des mappages plus sophistiqués entre l’appareil et son jumeau (par exemple, avec une table de mappage).

Chaque fois qu’un événement de télémétrie de température est envoyé par le thermomètre, une fonction Azure traite la télémétrie, et la propriété *temperature* du jumeau numérique doit être mise à jour. Ce scénario est présenté dans un diagramme ci-dessous :

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Diagramme montrant un organigramme. Dans le graphique, un appareil IoT Hub envoie des données de télémétrie de température via IoT Hub à une fonction Azure, qui met à jour une propriété de température pour un jumeau dans Azure Digital Twins." border="false":::

## <a name="add-a-model-and-twin"></a>Ajouter un modèle et un jumeau

Vous pouvez ajouter/télécharger un modèle à l’aide de la commande CLI ci-dessous, puis créer un double à l’aide de ce modèle qui sera mis à jour avec les informations d’IoT Hub.

Le modèle se présente ainsi :
```JSON
{
  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",
  "@type": "Interface",
  "@context": "dtmi:dtdl:context;2",
  "contents": [
    {
      "@type": "Property",
      "name": "Temperature",
      "schema": "double"
    }
  ]
}
```

Pour **télécharger ce modèle sur votre instance de jumeaux** , ouvrez Azure CLI et exécutez la commande suivante :

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' -n {digital_twins_instance_name}
```

Vous devez ensuite **créer un jumeau à l’aide de ce modèle**. Utilisez la commande suivante pour créer un jumeau et définissez 0,0 comme valeur initiale de température.

```azurecli-interactive
az dt twin create --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{"Temperature": 0.0,}' --dt-name {digital_twins_instance_name}
```

La sortie d’une commande de création de jumeau réussie doit se présenter comme suit :
```json
{
  "$dtId": "thermostat67",
  "$etag": "W/\"0000000-9735-4f41-98d5-90d68e673e15\"",
  "$metadata": {
    "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
    "Temperature": {
      "ackCode": 200,
      "ackDescription": "Auto-Sync",
      "ackVersion": 1,
      "desiredValue": 0.0,
      "desiredVersion": 1
    }
  },
  "Temperature": 0.0
}
```

## <a name="create-an-azure-function"></a>Création d’une fonction Azure

Cette section utilise les mêmes étapes de démarrage Visual Studio et le même squelette de fonction Azure que ceux de [*Procédure : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md). Le squelette gère l’authentification et crée un client de service, prêt à traiter les données et à appeler les API Azure Digital Twins. 

Dans les étapes qui suivent, vous y ajouterez du code pour le traitement des événements de télémétrie IoT provenant d’IoT Hub.  

### <a name="add-telemetry-processing"></a>Ajouter le traitement des données de télémétrie
    
Les événements de télémétrie sont fournis sous la forme de messages provenant de l’appareil. Lorsque vous ajoutez du code de traitement des données de télémétrie, la première étape consiste à extraire la partie utile de ce message d’appareil à partir de l’événement Event Grid. 

Les différents types d’appareils peuvent structurer leurs messages différemment. Le code de **cette étape dépend donc de l’appareil connecté.** 

Le code suivant illustre un exemple d’appareil simple qui envoie des données de télémétrie au format JSON. Cet exemple est entièrement abordé dans le [Tutoriel  *: Connecter une solution de bout en bout*](./tutorial-end-to-end.md). Le code suivant trouve l’ID de l’appareil qui a envoyé le message, ainsi que la valeur de température.

```csharp
JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
var temperature = deviceMessage["body"]["Temperature"];
```

L’exemple de code suivant prend la valeur d’ID et de température et les utilise pour « corriger » (mettre à jour) ce jumeau.

```csharp
//Update twin using device temperature
var updateTwinData = new JsonPatchDocument();
updateTwinData.AppendReplace("/Temperature", temperature.Value<double>());
await client.UpdateDigitalTwinAsync(deviceId, updateTwinData);
...
```

### <a name="update-your-azure-function-code"></a>Mettre à jour le code de votre fonction Azure

Maintenant que vous comprenez le code des exemples précédents, ouvrez votre fonction Azure à partir de la section [*Prérequis*](#prerequisites) dans Visual Studio. (Si vous n’avez pas de fonction Azure, visitez le lien dans les prérequis pour en créer une maintenant).

Remplacez le code de votre fonction Azure par cet exemple de code.

```csharp
using System;
using System.Net.Http;
using Azure.Core.Pipeline;
using Azure.DigitalTwins.Core;
using Azure.DigitalTwins.Core.Serialization;
using Azure.Identity;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace IotHubtoTwins
{
    public class IoTHubtoTwins
    {
        private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static readonly HttpClient httpClient = new HttpClient();

        [FunctionName("IoTHubtoTwins")]
        public async void Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
        {
            if (adtInstanceUrl == null) log.LogError("Application setting \"ADT_SERVICE_URL\" not set");

            try
            {
                //Authenticate with Digital Twins
                ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
                DigitalTwinsClient client = new DigitalTwinsClient(
                    new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions 
                    { Transport = new HttpClientTransport(httpClient) });
                log.LogInformation($"ADT service client connection created.");
            
                if (eventGridEvent != null && eventGridEvent.Data != null)
                {
                    log.LogInformation(eventGridEvent.Data.ToString());

                    // Reading deviceId and temperature for IoT Hub JSON
                    JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
                    string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
                    var temperature = deviceMessage["body"]["Temperature"];
                    
                    log.LogInformation($"Device:{deviceId} Temperature is:{temperature}");

                    //Update twin using device temperature
                    var updateTwinData = new JsonPatchDocument();
                    updateTwinData.AppendReplace("/Temperature", temperature.Value<double>());
                    await client.UpdateDigitalTwinAsync(deviceId, updateTwinData);
                }
            }
            catch (Exception e)
            {
                log.LogError($"Error in ingest function: {e.Message}");
            }
        }
    }
}
```
Enregistrez votre code de fonction et publiez l’application de fonction dans Azure. Pour ce faire, reportez-vous à la section [*Publier l’application de fonction*](./how-to-create-azure-function.md#publish-the-function-app-to-azure) du [*Guide pratique : Configurer une fonction Azure pour le traitement des données*](how-to-create-azure-function.md).

Après une publication réussie, vous verrez la sortie dans la fenêtre de commande de Visual Studio, comme indiqué ci-dessous :

```cmd
1>------ Build started: Project: adtIngestFunctionSample, Configuration: Release Any CPU ------
1>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\bin\Release\netcoreapp3.1\bin\adtIngestFunctionSample.dll
2>------ Publish started: Project: adtIngestFunctionSample, Configuration: Release Any CPU ------
2>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\bin\Release\netcoreapp3.1\bin\adtIngestFunctionSample.dll
2>adtIngestFunctionSample -> C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\obj\Release\netcoreapp3.1\PubTmp\Out\
2>Publishing C:\Users\source\repos\Others\adtIngestFunctionSample\adtIngestFunctionSample\obj\Release\netcoreapp3.1\PubTmp\adtIngestFunctionSample - 20200911112545669.zip to https://adtingestfunctionsample20200818134346.scm.azurewebsites.net/api/zipdeploy...
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
========== Publish: 1 succeeded, 0 failed, 0 skipped ==========
```
Vous pouvez également vérifier l’état du processus de publication dans le [Portail Azure](https://portal.azure.com/). Recherchez votre _groupe de ressources_ et accédez à _Journal d’activité_ , puis recherchez _Obtenir le profil de publication de l’application web_ dans la liste et vérifiez que l’état indique une réussite.

:::image type="content" source="media/how-to-ingest-iot-hub-data/azure-function-publish-activity-log.png" alt-text="Capture d’écran du Portail Azure montrant l’état du processus de publication.":::

## <a name="connect-your-function-to-iot-hub"></a>Connecter votre fonction à IoT Hub

Configurez une destination d’événement pour les données de concentrateur.
Dans le [Portail Azure](https://portal.azure.com/), accédez à l’instance IoT Hub que vous avez créée dans la section [*Prérequis*](#prerequisites). Sous **Événements** , créez un abonnement pour votre fonction Azure.

:::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Capture d’écran du Portail Azure qui montre l’ajout d’un abonnement aux événements.":::

Dans la page **Créer un abonnement aux événements** , renseignez les champs de la façon suivante :
  1. Sous **Nom** , nommez l’abonnement que vous souhaitez.
  2. Sous **Schéma d’événement** , choisissez _Schéma Event Grid_.
  3. Sous **Types d’événements** , choisissez la case à cocher _Télémétrie d’appareil_ et décochez les autres types d’événements.
  4. Sous **Type de point de terminaison** , sélectionnez _Fonction Azure_.
  5. Sous **Point de terminaison** , sélectionnez le lien _Sélectionner un point de terminaison_ pour créer un point de terminaison.
    
:::image type="content" source="media/how-to-ingest-iot-hub-data/create-event-subscription.png" alt-text="Capture d’écran du Portail Azure pour créer les détails de l’abonnement aux événements":::

Dans la page _Sélectionner une fonction Azure_ qui s’ouvre, vérifiez les détails ci-dessous.
 1. **Abonnement** : Votre abonnement Azure
 2. **Groupe de ressources**  : Votre groupe de ressources
 3. **Application de fonction**  : Nom de votre application de fonction
 4. **Emplacement**  : _Production_
 5. **Fonction**  : Sélectionnez votre fonction Azure dans la liste déroulante.

Enregistrez vos détails en sélectionnant le bouton _Confirmer la sélection_.            
      
:::image type="content" source="media/how-to-ingest-iot-hub-data/select-azure-function.png" alt-text="Capture d’écran du Portail Azure pour sélectionner la fonction Azure":::

Sélectionnez le bouton _Créer_ pour créer un abonnement aux événements.

## <a name="send-simulated-iot-data"></a>Envoyer des données IoT simulées

Pour tester votre nouvelle fonction d’entrée, utilisez le simulateur d’appareil à partir du [*Tutoriel : Connecter une solution de bout en bout*](./tutorial-end-to-end.md). Ce tutoriel est piloté par un exemple de projet écrit en C#. L’exemple de code se trouve ici : [Exemples Azure Digital Twins de bout en bout](/samples/azure-samples/digital-twins-samples/digital-twins-samples). Vous utiliserez le projet **DeviceSimulator** dans ce référentiel.

Dans ce tutoriel intégral, vous allez effectuer les étapes suivantes :
1. [*Inscrire l’appareil simulé auprès d’IoT Hub*](./tutorial-end-to-end.md#register-the-simulated-device-with-iot-hub)
2. [*Configurer et démarrer la simulation*](./tutorial-end-to-end.md#configure-and-run-the-simulation)

## <a name="validate-your-results"></a>Valider vos résultats

Lors de l’exécution du simulateur d’appareil ci-dessus, la valeur de température de votre jumeau numérique sera modifiée. Dans Azure CLI, exécutez la commande suivante pour afficher la valeur de la température.

```azurecli-interactive
az dt twin query -q "select * from digitaltwins" -n {digital_twins_instance_name}
```

Votre sortie doit contenir une valeur de température comme ceci :

```json
{
  "result": [
    {
      "$dtId": "thermostat67",
      "$etag": "W/\"0000000-1e83-4f7f-b448-524371f64691\"",
      "$metadata": {
        "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
        "Temperature": {
          "ackCode": 200,
          "ackDescription": "Auto-Sync",
          "ackVersion": 1,
          "desiredValue": 69.75806974934324,
          "desiredVersion": 1
        }
      },
      "Temperature": 69.75806974934324
    }
  ]
}
```

Pour voir la modification de la valeur, exécutez à plusieurs reprises la commande de requête ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’entrée et la sortie de données avec Azure Digital Twins :
* [*Concepts : Intégration à d’autres services*](concepts-integration.md)