---
title: Intégrer à Azure Maps
titleSuffix: Azure Digital Twins
description: Découvrez comment créer une fonction Azure capable d’utiliser le graphique de jumeaux et les notifications Azure Digital Twins pour mettre à jour une carte d’intérieur Azure Maps.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 6/3/2020
ms.topic: how-to
ms.service: digital-twins
ms.reviewer: baanders
ms.openlocfilehash: 3e5eb49a91e2c8bbd73f5dd37ed90f10b406fa3d
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92496037"
---
# <a name="use-azure-digital-twins-to-update-an-azure-maps-indoor-map"></a>Utiliser Azure Digital Twins pour mettre à jour un plan intérieur Azure Maps

Cet article décrit les étapes nécessaires à l’utilisation des données Azure Digital Twins pour mettre à jour les informations affichées sur un *plan intérieur* à l’aide d’ [Azure Maps](../azure-maps/about-azure-maps.md). Azure Digital Twins stocke un graphique de vos relations d’appareils IoT et achemine les données de télémétrie vers différents points de terminaison, ce qui en fait le service parfait pour mettre à jour les superpositions d’informations sur les cartes.

Cette procédure couvre les sujets suivants :

1. Configuration de votre instance Azure Digital Twins pour envoyer des événements de mise à jour de jumeaux à une fonction dans [Azure Functions](../azure-functions/functions-overview.md).
2. Création d’une fonction Azure pour mettre à jour un ensemble d’états des fonctionnalités de plans intérieurs Azure Maps.
3. Procédure de stockage de vos ID de cartes et d’ensemble d’états des fonctionnalités dans le graphique Azure Digital Twins.

### <a name="prerequisites"></a>Prérequis

* Suivez le [*Didacticiel Azure Digital Twins : Connecter une solution de bout en bout*](./tutorial-end-to-end.md).
    * Vous allez étendre ce jumeaux avec un point de terminaison et un itinéraire supplémentaires. Vous allez également ajouter une autre fonction à votre application de fonction à partir de ce didacticiel. 
* Suivez le [*Didacticiel Azure Maps : Utiliser Azure Maps Creator pour créer des cartes d’intérieur*](../azure-maps/tutorial-creator-indoor-maps.md) afin de créer une carte d’intérieur Azure Maps avec un *ensemble d’états des fonctionnalités* .
    * Les [ensembles d’états des fonctionnalités](../azure-maps/creator-indoor-maps.md#feature-statesets) sont une collections de propriétés dynamiques (états) affectées à des fonctionnalités de jeu de données, telles que des salles ou des équipements. Dans le didacticiel Azure Maps ci-dessus, l’ensemble d’états des fonctionnalités stocke l’état de la salle que vous allez afficher sur une carte.
    * Vous avez besoin de l’ *ID de votre ensemble des états* des fonctionnalités et de l’ *ID d’abonnement* Azure Maps.

### <a name="topology"></a>Topologie

L’image ci-dessous illustre où les éléments d’intégration des plans intérieurs de ce didacticiel s’intègrent dans un scénario Azure Digital Twins de bout en bout plus grand.

:::image type="content" source="media/how-to-integrate-maps/maps-integration-topology.png" alt-text="Vue des services Azure dans un scénario de bout en bout, mise en surbrillance de l’élément d’intégration de plans intérieurs" lightbox="media/how-to-integrate-maps/maps-integration-topology.png":::

## <a name="create-a-function-to-update-a-map-when-twins-update"></a>Créer une fonction pour mettre une carte à jour en cas de mise à jour des jumeaux

Tout d’abord, vous allez créer un itinéraire dans Azure Digital Twins pour transférer tous les événements de mise à jour des jumeaux dans une rubrique Event Grid. Ensuite, vous allez utiliser une fonction Azure pour lire ces messages de mise à jour et mettre à jour un ensemble d’états des fonctionnalités dans Azure Maps. 

## <a name="create-a-route-and-filter-to-twin-update-notifications"></a>Créer un itinéraire et un filtre pour les notifications de mise à jour des jumeaux

Les instances Azure Digital Twins peuvent émettre des événements de mise à jour des jumeaux chaque fois que l’état d’un jumeau est mis à jour. Le [*Didacticiel Azure Digital Twins : Connecter une solution de bout en bout*](./tutorial-end-to-end.md) référencé ci-dessus présente un scénario dans lequel un thermomètre est utilisé pour mettre à jour un attribut de température associé au jumeau d’une pièce. Vous allez étendre cette solution en vous abonnant aux notifications de mise à jour pour les jumeaux et en utilisant ces informations pour mettre à jour vos cartes.

Ce modèle lit directement à partir du jumeau de la pièce, plutôt que de l’appareil IoT, ce qui vous donne la possibilité de modifier la source de données sous-jacente pour la température sans devoir mettre à jour votre logique de mappage. Par exemple, vous pouvez ajouter plusieurs thermomètres ou définir cette pièce de manière à partager un thermomètre avec une autre pièce, tout cela sans devoir mettre à jour votre logique de mappage.

1. Créez une rubrique Event Grid, qui recevra les événements de votre instance Azure Digital Twins.
    ```azurecli-interactive
    az eventgrid topic create -g <your-resource-group-name> --name <your-topic-name> -l <region>
    ```

2. Créez un point de terminaison pour lier votre rubrique Event Grid à Azure Digital Twins.
    ```azurecli-interactive
    az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
    ```

3. Créez un itinéraire dans Azure Digital Twins pour envoyer des événements de mise à jour de jumeaux à votre point de terminaison.

    >[!NOTE]
    >Il existe actuellement un **problème connu** dans Cloud Shell affectant les groupes de commandes suivants : `az dt route`, `az dt model`, `az dt twin`.
    >
    >Pour le résoudre, exécutez `az login` dans Cloud Shell avant d’exécuter la commande, ou utilisez la [CLI locale](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) au lieu de Cloud Shell. Pour plus d’informations, consultez [*résolution des problèmes : Problèmes connus dans Azure Digital Twins*](troubleshoot-known-issues.md#400-client-error-bad-request-in-cloud-shell).

    ```azurecli-interactive
    az dt route create -n <your-Azure-Digital-Twins-instance-name> --endpoint-name <Event-Grid-endpoint-name> --route-name <my_route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
    ```

## <a name="create-an-azure-function-to-update-maps"></a>Créer une fonction Azure pour mettre les cartes à jour

Vous allez créer une fonction déclenchée par Event Grid à l’intérieur de votre application de fonction à partir du didacticiel de bout en bout ( [*Didacticiel : Connecter une solution de bout en bout*](./tutorial-end-to-end.md)). Cette fonction va décompresser ces notifications et envoyer des mises à jour à un ensemble d’états des fonctionnalités Azure Maps pour mettre à jour la température d’une pièce. 

Consultez le document suivant pour obtenir des informations de référence : [*Déclencheur Azure Event Grid pour Azure Functions*](../azure-functions/functions-bindings-event-grid-trigger.md).

Remplacez le code de fonction par le code suivant. Vous appliquez ainsi un filtre pour n’obtenir que les mises à jour apportées aux jumeaux de l’espace, vous lisez la température mise à jour et envoyez ces informations à Azure Maps.

```C#
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Threading.Tasks;
using System.Net.Http;

namespace SampleFunctionsApp
{
    public static class ProcessDTUpdatetoMaps
    {   //Read maps credentials from application settings on function startup
        private static string statesetID = Environment.GetEnvironmentVariable("statesetID");
        private static string subscriptionKey = Environment.GetEnvironmentVariable("subscription-key");
        private static HttpClient httpClient = new HttpClient();

        [FunctionName("ProcessDTUpdatetoMaps")]
        public static async Task Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            JObject message = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
            log.LogInformation("Reading event from twinID:" + eventGridEvent.Subject.ToString() + ": " +
                eventGridEvent.EventType.ToString() + ": " + message["data"]);

            //Parse updates to "space" twins
            if (message["data"]["modelId"].ToString() == "dtmi:contosocom:DigitalTwins:Space;1")
            {   //Set the ID of the room to be updated in your map. 
                //Replace this line with your logic for retrieving featureID. 
                string featureID = "UNIT103";

                //Iterate through the properties that have changed
                foreach (var operation in message["data"]["patch"])
                {
                    if (operation["op"].ToString() == "replace" && operation["path"].ToString() == "/Temperature")
                    {   //Update the maps feature stateset
                        var postcontent = new JObject(new JProperty("States", new JArray(
                            new JObject(new JProperty("keyName", "temperature"),
                                 new JProperty("value", operation["value"].ToString()),
                                 new JProperty("eventTimestamp", DateTime.Now.ToString("s"))))));

                        var response = await httpClient.PostAsync(
                            $"https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetID}&featureID={featureID}&subscription-key={subscriptionKey}",
                            new StringContent(postcontent.ToString()));

                        log.LogInformation(await response.Content.ReadAsStringAsync());
                    }
                }
            }
        }
    }
}
```

Vous allez devoir définir deux variables d’environnement dans votre application de fonction. L’une est votre [clé d’abonnement principal Azure Maps](../azure-maps/quick-demo-map-app.md#get-the-primary-key-for-your-account) et l’autre est votre [ID d’ensemble d’états Azure Maps](../azure-maps/tutorial-creator-indoor-maps.md#create-a-feature-stateset).

```azurecli-interactive
az functionapp config appsettings set --settings "subscription-key=<your-Azure-Maps-primary-subscription-key> -g <your-resource-group> -n <your-App-Service-(function-app)-name>"
az functionapp config appsettings set --settings "statesetID=<your-Azure-Maps-stateset-ID> -g <your-resource-group> -n <your-App-Service-(function-app)-name>
```

### <a name="view-live-updates-on-your-map"></a>Afficher les mises à jour en direct sur votre carte

Pour afficher la température mise à jour en direct, procédez comme suit :

1. Commencez à envoyer des données IoT simulées en exécutant le projet **DeviceSimulator** à partir du [*Didacticiel Azure Digital Twins : Connecter une solution de bout en bout*](tutorial-end-to-end.md). Les instructions correspondantes se trouvent dans la section [*Configurer et exécuter la simulation*](././tutorial-end-to-end.md#configure-and-run-the-simulation).
2. Le [module **Plans intérieurs Azure**](../azure-maps/how-to-use-indoor-module.md) vous permet d’afficher des plans intérieurs créés dans Azure Maps Creator.
    1. Copiez le code HTML de la section [*Exemple : Utiliser le module Indoor Maps*](../azure-maps/how-to-use-indoor-module.md#example-use-the-indoor-maps-module) du [*Didacticiel : Utiliser le module Indoor Maps d’Azure Maps*](../azure-maps/how-to-use-indoor-module.md) des cartes d’intérieur dans un fichier local.
    1. Remplacez le *tilesetId* et *statesetID* dans le fichier HTML local par vos valeurs.
    1. Ouvrez ce fichier dans votre navigateur.

Les deux exemples envoient la température dans une plage compatible. Vous devez donc voir la couleur de la mise à jour de la salle 121 sur la carte toutes les 30 secondes environ.

:::image type="content" source="media/how-to-integrate-maps/maps-temperature-update.png" alt-text="Vue des services Azure dans un scénario de bout en bout, mise en surbrillance de l’élément d’intégration de plans intérieurs":::

## <a name="store-your-maps-information-in-azure-digital-twins"></a>Stocker les informations de vos cartes dans Azure Digital Twins

Maintenant que vous disposez d’une solution codée en dur pour mettre à jour les informations de vos cartes, vous pouvez utiliser le graphique Azure Digital Twins pour stocker toutes les informations nécessaires à la mise à jour de votre plan intérieur. Cela inclut l’ID d’ensemble d’états, l’ID d’abonnement des cartes et l’ID de fonctionnalité de chaque mappage et emplacement respectivement. 

Une solution pour cet exemple spécifique impliquerait la mise à jour de chaque espace de niveau supérieur pour avoir un attribut d’ID d’ensemble d’états et d’ID d’abonnement de cartes, ainsi que la mise à jour de chaque salle pour avoir un ID de fonctionnalité. Vous devez définir ces valeurs une fois lors de l’initialisation du graphique de jumeaux, puis interroger ces valeurs pour chaque événement de mise à jour de jumeau.

Selon la configuration de votre topologie, vous pourrez stocker ces trois attributs à différents niveaux en corrélation avec le niveau de précision de votre carte.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la gestion, la mise à niveau et la récupération d’informations à partir du graphique de jumeaux, consultez les références suivantes :

* [*Guide pratique : Gérer des jumeaux numériques*](./how-to-manage-twin.md)
* [*Guide pratique : Interroger le graphe de jumeaux*](./how-to-query-graph.md)