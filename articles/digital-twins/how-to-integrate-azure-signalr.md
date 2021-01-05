---
title: Intégrer à Azure SignalR Service
titleSuffix: Azure Digital Twins
description: Découvrez comment diffuser en continu la télémétrie Azure Digital Twins vers des clients à l’aide d’Azure SignalR
author: dejimarquis
ms.author: aymarqui
ms.date: 09/02/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 3a11cd9f3208c97748ab16c636aedd9a443c5b9f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93093161"
---
# <a name="integrate-azure-digital-twins-with-azure-signalr-service"></a>Intégrer Azure Digital Twins avec le service Azure SignalR

Dans cet article, vous allez apprendre à intégrer Azure Digital Twins avec [Azure SignalR Service](../azure-signalr/signalr-overview.md).

La solution décrite dans cet article vous permet de transmettre des données de télémétrie de jumeau numérique à des clients connectés, par exemple une page web unique ou une application mobile. Par conséquent, les clients sont mis à jour avec les métriques et l’état en temps réel des appareils IoT, sans qu’il soit nécessaire d’interroger le serveur ou d’envoyer de nouvelles requêtes HTTP pour les mises à jour.

## <a name="prerequisites"></a>Prérequis

Voici les conditions préalables que vous devez remplir avant de continuer :

* Avant d’intégrer votre solution au service Azure SignalR dans cet article, vous devez suivre le [_**Tutoriel Azure Digital Twins : Connecter une solution de bout en bout**_](tutorial-end-to-end.md), car ce guide s’appuie dessus. Ce tutoriel vous guide dans la configuration d’une instance Azure Digital Twins qui fonctionne avec un appareil IoT virtuel pour déclencher des mises à jour de jumeaux numériques. Ce guide permet de connecter ces mises à jour à un exemple d’application web à l’aide du service Azure SignalR.
    - Vous avez besoin du nom de la **rubrique Event Grid** que vous avez créée dans le tutoriel.
* Vous devez avoir [**Node.js**](https://nodejs.org/) installé sur votre machine.

Vous pouvez également vous connecter au [Portail Azure](https://portal.azure.com/) avec votre compte Azure.

## <a name="solution-architecture"></a>Architecture de la solution

Vous allez attacher Azure SignalR Service à Azure Digital Twins via le chemin d’accès ci-dessous. Dans le diagramme, les sections A, B et C sont tirées du diagramme d’architecture du [Didacticiel de bout en bout prérequis](tutorial-end-to-end.md) ; dans ce guide pratique, vous allez lui ajouter la section D.

:::image type="content" source="media/how-to-integrate-azure-signalr/signalr-integration-topology.png" alt-text="Une vue des services Azure dans un scénario de bout en bout. Décrit le flux de données depuis un appareil vers successivement IoT Hub, une fonction Azure (flèche B), une instance Azure Digital Twins (section A), Event Grid et une autre fonction Azure en vue du traitement (flèche C). La section D illustre le flux de données à partir du même Event Grid en flèche C vers une fonction Azure appelée « broadcast ». « broadcast » communique avec une autre fonction Azure nommée « negotiate », et les fonctions « broadcast » et « negotiate » communiquent toutes deux avec les appareils informatiques." lightbox="media/how-to-integrate-azure-signalr/signalr-integration-topology.png":::

## <a name="download-the-sample-applications"></a>Téléchargement des exemples d'applications

Tout d’abord, téléchargez les exemples d’applications requis. Vous aurez besoin des deux éléments suivants :
* [**Exemples Azure Digital Twins de bout en bout**](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) : Cet exemple contient une *AdtSampleApp* contenant deux fonctions Azure pour déplacer des données dans une instance Azure Digital Twins (vous pouvez en savoir plus sur ce scénario dans [*Tutoriel : Connecter une solution de bout en bout*](tutorial-end-to-end.md)). Il contient également un exemple d’application *DeviceSimulator* qui simule un appareil IoT générant une nouvelle valeur de température chaque seconde. 
    - Accédez à l’exemple de lien, puis cliquez sur le bouton *Télécharger le zip* pour télécharger une copie de l’exemple sur votre ordinateur, comme _**Azure_Digital_Twins_end_to_end_samples.zip**_. Décompressez le dossier.
* [**Exemple d’application web d’intégration SignalR**](/samples/azure-samples/digitaltwins-signalr-webapp-sample/digital-twins-samples/) : Il s’agit d’un exemple d’application web React qui consomme des données de télémétrie Azure Digital Twins à partir d’un service Azure SignalR.
    -  Accédez à l’exemple de lien, puis cliquez sur le bouton *Télécharger le zip* pour télécharger une copie de l’exemple sur votre ordinateur, comme _**Azure_Digital_Twins_SignalR_integration_web_app_sample.zip**_. Décompressez le dossier.

[!INCLUDE [Create instance](../azure-signalr/includes/signalr-quickstart-create-instance.md)]

Laissez la fenêtre du navigateur ouverte sur le Portail Azure, car vous l’utiliserez à nouveau dans la section suivante.

## <a name="configure-and-run-the-azure-functions-app"></a>Configurer et exécuter l’application Azure Functions

Dans cette section, vous allez configurer deux fonctions Azure :
* **negotiate** : une fonction de déclencheur HTTP. Elle utilise la liaison d’entrée *SignalRConnectionInfo* pour générer et retourner des informations de connexion valides.
* **broadcast** - Une fonction de déclencheur [Event Grid](../event-grid/overview.md). Elle reçoit les données de télémétrie Azure Digital Twins via Event Grid et utilise la liaison de sortie de l’instance *SignalR* que vous avez créée à l’étape précédente pour diffuser le message à toutes les applications clientes connectées.

Tout d’abord, accédez au navigateur dans lequel le Portail Azure est ouvert, puis procédez comme suit pour obtenir la **chaîne de connexion** pour l’instance SignalR que vous avez configurée. Vous en aurez besoin pour configurer les fonctions.
1. Vérifiez que l’instance de service SignalR que vous avez déployée précédemment a été correctement créée. Pour ce faire, vous pouvez rechercher son nom dans la zone de recherche en haut du portail. Sélectionnez l’instance pour l’ouvrir.

1. Sélectionnez **Clés** dans le menu d’instance pour afficher les chaînes de connexion de l’instance du service SignalR.

1. Sélectionnez l’icône pour copier la chaîne de connexion principale.

    :::image type="content" source="media/how-to-integrate-azure-signalr/signalr-keys.png" alt-text="Capture d’écran du Portail Azure montrant la page Clés pour l’instance SignalR. L’icône « Copier dans le Presse-papiers » en regard de la CHAÎNE DE CONNEXION PRINCIPALE est mise en surbrillance." lightbox="media/how-to-integrate-azure-signalr/signalr-keys.png":::

Ensuite, démarrez Visual Studio (ou un autre éditeur de code de votre choix), puis ouvrez la solution de code dans le dossier *Azure_Digital_Twins_end_to_end_samples > ADTSampleApp*. Puis procédez comme suit pour créer les fonctions :

1. Créez une nouvelle C# appelée **SignalRFunctions.cs** dans le projet *SampleFunctionsApp*.

1. Remplacez le contenu du fichier de classe par le code suivant :

    ```C#
    using System;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Http;
    using Microsoft.Azure.EventGrid.Models;
    using Microsoft.Azure.WebJobs;
    using Microsoft.Azure.WebJobs.Extensions.Http;
    using Microsoft.Azure.WebJobs.Extensions.EventGrid;
    using Microsoft.Azure.WebJobs.Extensions.SignalRService;
    using Microsoft.Extensions.Logging;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using System.Collections.Generic;
    
    namespace SampleFunctionsApp
    {
        public static class SignalRFunctions
        {
            public static double temperature;
    
            [FunctionName("negotiate")]
            public static SignalRConnectionInfo GetSignalRInfo(
                [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequest req,
                [SignalRConnectionInfo(HubName = "dttelemetry")] SignalRConnectionInfo connectionInfo)
            {
                return connectionInfo;
            }
    
            [FunctionName("broadcast")]
            public static Task SendMessage(
                [EventGridTrigger] EventGridEvent eventGridEvent,
                [SignalR(HubName = "dttelemetry")] IAsyncCollector<SignalRMessage> signalRMessages,
                ILogger log)
            {
                JObject eventGridData = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
    
                log.LogInformation($"Event grid message: {eventGridData}");
    
                var patch = (JObject)eventGridData["data"]["patch"][0];
                if (patch["path"].ToString().Contains("/Temperature"))
                {
                    temperature = Math.Round(patch["value"].ToObject<double>(), 2);
                }
    
                var message = new Dictionary<object, object>
                {
                    { "temperatureInFahrenheit", temperature},
                };
        
                return signalRMessages.AddAsync(
                    new SignalRMessage
                    {
                        Target = "newMessage",
                        Arguments = new[] { message }
                    });
            }
        }
    }
    ```

1. Dans la fenêtre *Console du gestionnaire de package* de Visual Studio ou toute fenêtre de commande sur votre machine, dans le dossier *Azure_Digital_Twins_end_to_end_samples\AdtSampleApp\SampleFunctionsApp* , exécutez la commande suivante pour installer le package NuGet `SignalRService` dans le projet :
    ```cmd
    dotnet add package Microsoft.Azure.WebJobs.Extensions.SignalRService --version 1.2.0
    ```

    Cela devrait résoudre tous les problèmes de dépendance dans la classe.

Ensuite, publiez votre fonction dans Azure, en suivant les étapes décrites dans la [section *Publier l’application*](tutorial-end-to-end.md#publish-the-app) du tutoriel *Connecter une solution de bout en bout*. Vous pouvez la publier dans la même application de fonction/App Service que celle utilisée dans le tutoriel de bout en bout prérequis ou en créer une nouvelle, mais vous préférerez peut-être utiliser le même pour réduire la duplication. Par ailleurs, terminez la publication de l’application en procédant comme suit :
1. Collectez l’ **URL de point de terminaison HTTP** de la fonction *negotiate*. Pour ce faire, accédez à la page [Function Apps](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Web%2Fsites/kind/functionapp) du Portail Azure et sélectionnez votre application de fonction dans la liste. Dans le menu de l’application, sélectionnez *Fonctions* et choisissez la fonction *negotiate*.

    :::image type="content" source="media/how-to-integrate-azure-signalr/functions-negotiate.png" alt-text="Vue du Portail Azure de l’application de fonction, avec « Fonctions » mis en surbrillance dans le menu. La liste des fonctions s’affiche sur la page et la fonction « negotiate » est également mise en surbrillance.":::

    Appuyez sur *Obtenir l’URL de la fonction* et copiez la valeur **jusqu’à _/api_ (n’incluez pas la partie _/negotiate?_ à la fin)** . Vous l’utiliserez ultérieurement.

    :::image type="content" source="media/how-to-integrate-azure-signalr/get-function-url.png" alt-text="Vue de la fonction « negotiate » dans le Portail Azure. Le bouton « Obtenir l’URL de la fonction » est mis en surbrillance, avec la partie de l’URL du début jusqu’à « /api »":::

1. Enfin, ajoutez votre **chaîne de connexion** Azure SignalR de plus tôt aux paramètres de l’application de fonction, à l’aide de la commande Azure CLI suivante. La commande peut être exécutée dans [Azure Cloud Shell](https://shell.azure.com) ou localement si Azure CLI est [installée sur votre ordinateur](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) :
 
    ```azurecli-interactive
    az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "AzureSignalRConnectionString=<your-Azure-SignalR-ConnectionString>"
    ```

    La sortie de cette commande imprime tous les paramètres d’application configurés pour votre fonction Azure. Recherchez `AzureSignalRConnectionString` en bas de la liste pour vérifier qu’elle a été ajoutée.

    :::image type="content" source="media/how-to-integrate-azure-signalr/output-app-setting.png" alt-text="Extrait de sortie dans une fenêtre de commande, avec affichage d’un élément de liste appelé « AzureSignalRConnectionString »":::

#### <a name="connect-the-function-to-event-grid"></a>Connecter la fonction à Event Grid

Inscrivez ensuite la fonction Azure *broadcast* dans la **rubrique Event Grid** que vous avez créée lors du [*Tutoriel : Connecter une solution de bout en bout*](tutorial-end-to-end.md) préreq. Cela permet aux données de télémétrie de circuler du jumeau *thermostat67* à la rubrique Event Grid, puis à la fonction, qui peut être diffusée à tous les clients.

Pour ce faire, vous allez créer un **abonnement Event Grid** à partir de votre rubrique Event Grid vers votre fonction Azure *broadcast* en tant que point de terminaison.

Dans le [portail Azure](https://portal.azure.com/), accédez à votre rubrique Event Grid en recherchant son nom dans la barre de recherche supérieure. Sélectionnez *+ Abonnement aux événements*.

:::image type="content" source="media/how-to-integrate-azure-signalr/event-subscription-1b.png" alt-text="Portail Azure : abonnement aux événements Event Grid":::

Dans la page *Créer un abonnement aux événements* , renseignez les champs comme suit (les champs remplis par défaut ne sont pas mentionnés) :
* *DÉTAILS DE L’ABONNEMENT AUX ÉVÉNEMENTS* > **Nom**  : Donnez un nom à votre abonnement aux événements.
* *DÉTAILS DU POINT DE TERMINAISON* > **Type de point de terminaison**  : Sélectionnez *Fonction Azure* dans les options de menu.
* *DÉTAILS DU POINT DE TERMINAISON* > **Point de terminaison**  : Cliquez sur le lien *Sélectionner un point de terminaison*. Une fenêtre *Sélectionner une fonction Azure* s’ouvre :
    - Renseignez vos **Abonnement** , **Groupe de ressources** , **Application de fonction** et **Fonction** ( *broadcast* ). Il est possible que certaines de ces informations soient automatiquement renseignées une fois que vous avez sélectionné l’abonnement.
    - Cliquez sur **Confirmer la sélection**.

:::image type="content" source="media/how-to-integrate-azure-signalr/create-event-subscription.png" alt-text="Vue du Portail Azure de la création d’un abonnement aux événements. Les champs ci-dessus sont renseignés et les boutons « Confirmer la sélection » et « Créer » sont mis en surbrillance.":::

Dans la page *Créer un abonnement aux événements* , cliquez sur **Créer**.

## <a name="configure-and-run-the-web-app"></a>Configurer et exécuter l’application web

Dans cette section, vous verrez le résultat en action. Tout d’abord, vous allez démarrer l’ **exemple d’application d’appareil simulé** qui envoie des données de télémétrie via votre instance Azure Digital Twins. Ensuite, vous allez configurer l’ **exemple d’application web cliente** pour vous connecter au flux Azure SignalR que vous avez configuré. Après cela, vous devriez être en mesure de voir les données mettant à jour l’exemple d’application web en temps réel.

### <a name="run-the-device-simulator"></a>Exécuter le simulateur d’appareil

Au cours du tutoriel de bout en bout prérequis, vous avez [configuré le simulateur d’appareil](tutorial-end-to-end.md#configure-and-run-the-simulation) pour envoyer des données via IoT Hub et à votre instance Azure Digital Twins.

Maintenant, il vous suffit de démarrer le projet de simulateur, situé dans *Azure_Digital_Twins_end_to_end_samples > DeviceSimulator > DeviceSimulator.sln*. Si vous utilisez Visual Studio, vous pouvez ouvrir le projet, puis l’exécuter avec ce bouton dans la barre d’outils :

:::image type="content" source="media/how-to-integrate-azure-signalr/start-button-simulator.png" alt-text="Bouton de démarrage dans Visual Studio (projet DeviceSimulator)":::

Une fenêtre de console s’ouvre, affichant les messages de télémétrie de température simulée. Ils sont envoyés par le biais de votre instance Azure Digital Twins, où ils sont ensuite récupérés par les fonctions Azure et SignalR.

Vous n’avez rien d’autre à faire dans cette console, mais laissez-la s’exécuter pendant que vous effectuez les étapes suivantes.

### <a name="configure-the-sample-client-web-app"></a>Configurer l’exemple d’application web cliente

Ensuite, configurez **l’exemple d’application web d’intégration SignalR** en procédant comme suit :
1. À l’aide de Visual Studio ou de l’éditeur de code de votre choix, ouvrez le dossier décompressé _**Azure_Digital_Twins_SignalR_integration_web_app_sample**_ que vous avez téléchargé dans la section [*Télécharger les exemples d’application*](#download-the-sample-applications).

1. Ouvrez le fichier *src/App.js* et remplacez l’URL dans `HubConnectionBuilder` par l’URL de point de terminaison HTTP de la fonction **negotiate** que vous avez enregistrée plus tôt :

    ```javascript
        const hubConnection = new HubConnectionBuilder()
            .withUrl('<URL>')
            .build();
    ```
1. Dans l’ *invite de commandes développeur* de Visual Studio ou dans toute fenêtre de commande sur votre ordinateur, accédez au dossier *Azure_Digital_Twins_SignalR_integration_web_app_sample\src*. Exécutez la commande suivante pour installer les packages des nœuds dépendants :

    ```cmd
    npm install
    ```

Ensuite, définissez les autorisations dans votre application de fonction dans le Portail Azure :
1. Dans la page [Applications de fonction](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Web%2Fsites/kind/functionapp) du Portail Azure, sélectionnez l’instance de votre application de fonction.
1. Faites défiler le menu de l’instance et sélectionnez *CORS*. Sur la page CORS, ajoutez `http://localhost:3000` comme origine autorisée en l’entrant dans la zone vide. Cochez la case *Activer Access-Control-Allow-Credentials* et appuyez sur *Enregistrer*.

    :::image type="content" source="media/how-to-integrate-azure-signalr/cors-setting-azure-function.png" alt-text="Paramètre CORS dans la fonction Azure":::

### <a name="see-the-results"></a>Afficher les résultats

Pour voir les résultats en action, démarrez l’ **exemple d’application web d’intégration SignalR**. Vous pouvez effectuer cette opération à partir de n’importe quelle fenêtre de console située à l’emplacement *Azure_Digital_Twins_SignalR_integration_web_app_sample\src* en exécutant la commande suivante :

```cmd
npm start
```

Cela ouvre une fenêtre de navigateur exécutant l’exemple d’application, qui affiche une jauge de température visuelle. Une fois que l’application est en cours d’exécution, vous devriez commencer à voir les valeurs de télémétrie de température du simulateur d’appareil qui se propagent via Azure Digital Twins reflétées par l’application web en temps réel.

:::image type="content" source="media/how-to-integrate-azure-signalr/signalr-webapp-output.png" alt-text="Extrait de l’exemple d’application web cliente, présentant une jauge de température visuelle. La température reflétée est 67,52":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources créées dans cet article, effectuez les étapes suivantes pour les supprimer. 

Dans Azure Cloud Shell ou Azure CLI en local, vous pouvez supprimer toutes les ressources Azure d’un groupe de ressources avec la commande [az group delete](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az-group-delete). La suppression du groupe de ressources entraînera également la suppression...
* de l’instance Azure Digital Twins (du tutoriel de bout en bout)
* du IoT Hub et de l’inscription de l’appareil Hub (du tutoriel de bout en bout)
* de la rubrique Event Grid et des abonnements associés
* de l’application Azure Functions, y compris les trois fonctions et les ressources associées, comme le stockage
* de l’instance Azure SignalR

> [!IMPORTANT]
> La suppression d’un groupe de ressources est irréversible. Le groupe de ressources et toutes les ressources qu’il contient sont supprimés définitivement. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. 

```azurecli-interactive
az group delete --name <your-resource-group>
```

Enfin, supprimez les dossiers d’exemples de projets que vous avez téléchargés sur votre ordinateur local ( *Azure_Digital_Twins_end_to_end_samples.zip* et *Azure_Digital_Twins_SignalR_integration_web_app_sample.zip* ).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous configurez des fonctions Azure avec SignalR pour diffuser des événements de télémétrie Azure Digital Twins vers un exemple d’application cliente.

Ensuite, vous en apprendrez plus sur Azure SignalR Service :
* [*Qu’est-ce que le service Azure SignalR ?*](../azure-signalr/signalr-overview.md)

Ou apprenez-en davantage sur l’authentification de service SignalR Azure avec Azure Functions :
* [*Authentification par Azure SignalR Service*](../azure-signalr/signalr-tutorial-authenticate-azure-functions.md)