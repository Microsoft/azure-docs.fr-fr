---
title: Gérer les points de terminaison et les itinéraires (API et CLI)
titleSuffix: Azure Digital Twins
description: Découvrez comment configurer et gérer les points de terminaison et les itinéraires d’événements pour les données d’Azure Digital Twins.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 11/18/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 7016abc9d52aa12b497d29f605fe351ee3f6a2dd
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96519106"
---
# <a name="manage-endpoints-and-routes-in-azure-digital-twins-apis-and-cli"></a>Gérer les points de terminaison et les itinéraires dans Azure Digital Twins (API et CLI)

[!INCLUDE [digital-twins-route-selector.md](../../includes/digital-twins-route-selector.md)]

Dans Azure Digital Twins, vous pouvez acheminer les [notifications d’événements](how-to-interpret-event-data.md) vers des services en aval ou des ressources de calcul connectées. Pour ce faire, vous devez d’abord configurer des **points de terminaison** qui peuvent recevoir les événements. Vous pouvez ensuite créer [**des itinéraires d’événements**](concepts-route-events.md) qui spécifient quels événements générés par Azure Digital Twins sont remis aux points de terminaison.

Cet article vous guide tout au long du processus de création de points de terminaison et de routes à l’aide des [API de routes d’événements](/rest/api/digital-twins/dataplane/eventroutes), du [kit SDK .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) et de l’[interface CLI Azure Digital Twins](how-to-use-cli.md).

Vous pouvez également gérer les points de terminaison et les routes à l’aide du [portail Azure](https://portal.azure.com). Pour obtenir une version de cet article qui utilise le portail à la place, consultez le [Tutoriel  *: Gérer les points de terminaison et les itinéraires (portail)*](how-to-manage-routes-portal.md).

## <a name="prerequisites"></a>Prérequis

* Vous avez besoin d’un **compte Azure** (vous pouvez en définir un gratuitement [ici](https://azure.microsoft.com/free/?WT.mc_id=A261C142F))
* Vous aurez besoin d’une **instance Azure Digital Twins** dans votre abonnement Azure. Si vous n’avez pas d’instance, vous pouvez en créer une en suivant les étapes décrites du [*Tutoriel : Configurer une instance et l’authentification*](how-to-set-up-instance-cli.md). Utilisez les valeurs suivantes du programme d’installation pour les utiliser plus loin dans cet article :
    - Nom de l’instance
    - Resource group
    
## <a name="create-an-endpoint-for-azure-digital-twins"></a>Créer un point de terminaison pour Azure Digital Twins

Voici les types de points de terminaison pris en charge que vous pouvez créer pour votre instance :
* [Event Grid](../event-grid/overview.md)
* [Hubs d'événements](../event-hubs/event-hubs-about.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Pour plus d’informations sur les différents points de terminaison, consultez [*Choisir entre les différents services de messagerie Azure*](../event-grid/compare-messaging-services.md).

Pour lier un point de terminaison à Azure Digital Twins, l’Event Hub, une rubrique Event Grid ou le Service Bus que vous utilisez pour le point de terminaison doivent pré-exister. 

### <a name="create-an-event-grid-endpoint"></a>Créer un point de terminaison Event Grid

L’exemple suivant montre comment créer un point de terminaison de type rubrique Event Grid à l’aide d’Azure CLI : Vous pouvez utiliser [Azure Cloud Shell](https://shell.azure.com)ou [installer l’interface CLI localement](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).

Premièrement, créez une rubrique Event Grid. Vous pouvez utiliser la commande suivante ou afficher les étapes plus en détail en visitant [la section *Créer une rubrique personnalisée*](../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic) du démarrage rapide *Événements personnalisés* Event Grid.

```azurecli-interactive
az eventgrid topic create -g <your-resource-group-name> --name <your-topic-name> -l <region>
```

> [!TIP]
> Pour obtenir la liste des noms de régions Azure qui peuvent être transmis à des commandes dans Azure CLI, exécutez la commande suivante :
> ```azurecli-interactive
> az account list-locations -o table
> ```

Une fois que vous avez créé la rubrique, vous pouvez la lier à Azure Digital Twins avec la [commande d’interface de ligne de commande Azure Digital Twins](how-to-use-cli.md) suivante :

```azurecli-interactive
az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> -n <your-Azure-Digital-Twins-instance-name>
```

La rubrique Event Grid est ensuite disponible en tant que point de terminaison dans Azure Digital Twins, sous le nom spécifié avec l’argument `--endpoint-name`. Vous utiliserez généralement ce nom en tant que cible d’un **itinéraire d’événement** que vous allez créer plus tard [dans cet article](#create-an-event-route) à l’aide de l’API de service Azure Digital Twins.

### <a name="create-an-event-hubs-or-service-bus-endpoint"></a>Créer un point de terminaison Event Hubs ou Service Bus

Le processus de création de Event Hubs ou de points de terminaison de Service Bus est similaire au processus de Event Grid présenté ci-dessus.

Tout d’abord, créez vos ressources que vous utiliserez comme point de terminaison. Voici ce qui est requis :
* Service Bus : _Espace de noms Service Bus_, _Rubrique Service Bus_,  _Règle d'autorisation_
* Hubs d'événements : _Espace de noms Event Hubs_, _hub d'événements_, _Règle d'autorisation_

Utilisez ensuite les commandes suivantes pour créer les points de terminaison dans Azure Digital Twins : 

* Ajouter un point de terminaison de rubrique Service Bus (nécessite une ressource Service Bus créée au préalable)
```azurecli-interactive 
az dt endpoint create servicebus --endpoint-name <Service-Bus-endpoint-name> --servicebus-resource-group <Service-Bus-resource-group-name> --servicebus-namespace <Service-Bus-namespace> --servicebus-topic <Service-Bus-topic-name> --servicebus-policy <Service-Bus-topic-policy> -n <your-Azure-Digital-Twins-instance-name>
```

* Ajouter un point de terminaison Event Hub (nécessite une ressource Event Hubs créée au préalable)
```azurecli-interactive
az dt endpoint create eventhub --endpoint-name <Event-Hub-endpoint-name> --eventhub-resource-group <Event-Hub-resource-group> --eventhub-namespace <Event-Hub-namespace> --eventhub <Event-Hub-name> --eventhub-policy <Event-Hub-policy> -n <your-Azure-Digital-Twins-instance-name>
```

### <a name="create-an-endpoint-with-dead-lettering"></a>Créer un point de terminaison avec mise en file d’attente de lettres mortes

Lorsqu’un point de terminaison ne peut pas remettre un événement dans un laps de temps donné ou après avoir essayé de remettre l’événement un certain nombre de fois, il peut envoyer l’événement non remis à un compte de stockage. Ce processus est appelé **mise en file d’attente de lettres mortes**.

Pour en savoir plus sur la mise en file d’attente de lettres mortes, consultez [*Concepts : Routes d’événements*](concepts-route-events.md#dead-letter-events). Pour obtenir des instructions sur la configuration d’un point de terminaison avec une mise en file d’attente de lettres mortes, suivez le reste de cette section.

#### <a name="set-up-storage-resources"></a>Configurer des ressources de stockage

Avant de définir l’emplacement des messages non distribués, vous devez disposer d’un [compte de stockage](../storage/common/storage-account-create.md?tabs=azure-portal) avec un [conteneur](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) configurés dans votre compte Azure. 

Vous devrez fournir l’URL de ce conteneur au moment de créer le point de terminaison ultérieurement. L’emplacement de la file d’attente de lettres mortes est fourni au point de terminaison en tant qu’URL de conteneur avec un [jeton SAS](../storage/common/storage-sas-overview.md). Ce jeton a besoin de l’autorisation `write` pour le conteneur de destination dans le compte de stockage. L’URL complète est au format suivant : `https://<storageAccountname>.blob.core.windows.net/<containerName>?<SASToken>`.

Suivez les étapes ci-dessous pour configurer ces ressources de stockage dans votre compte Azure, afin de préparer la configuration de la connexion du point de terminaison dans la section suivante.

1. Suivez les étapes décrites dans [*Créer un compte de stockage*](../storage/common/storage-account-create.md?tabs=azure-portal) pour créer un **compte de stockage** dans votre abonnement Azure. Prenez note du nom du compte de stockage ; vous en aurez besoin plus tard.
2. Suivez les étapes décrites dans [*Créer un conteneur*](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) pour créer un **conteneur** dans le nouveau compte de stockage. Notez le nom de conteneur ; vous en aurez besoin plus tard.
3. Ensuite, créez un **jeton SAS** pour votre compte de stockage que le point de terminaison peut utiliser pour y accéder. Commencez par accéder à votre compte de stockage dans le [portail Azure](https://ms.portal.azure.com/#home) (vous pouvez le rechercher par son nom dans la barre de recherche du portail).
4. Dans la page du compte de stockage, choisissez le lien _Signature d’accès partagé_ dans la barre de navigation de gauche afin de lancer la configuration du jeton SAP.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png" alt-text="Page du compte de stockage dans le portail Azure" lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-1.png":::

1. Dans la page *Signature d’accès partagé*, sous *Services autorisés* et *Types de ressources autorisés*, sélectionnez les paramètres de votre choix. Vous devez sélectionner au moins une case dans chaque catégorie. Sous *Autorisations acceptées*, choisissez **Écriture** (vous pouvez également sélectionner d’autres autorisations si vous le souhaitez).
1. Définissez les valeurs de votre choix pour les autres paramètres.
1. Lorsque vous avez terminé, sélectionnez le bouton _Générer la chaîne de connexion et SAP_ pour générer le jeton SAP. 

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png" alt-text="Page du compte de stockage dans le portail Azure, présentant toutes les sélections de paramètres pour générer un jeton SAP et mise en surbrillance du bouton Générer la chaîne de connexion et SAP" lightbox="./media/how-to-manage-routes-apis-cli/generate-sas-token-2.png"::: 

1. Cela a pour effet de générer plusieurs valeurs de chaîne de connexion et SAP au bas de la même page, sous les sélections de paramètres. Faites défiler l’écran pour afficher les valeurs et utilisez l’icône de *copie dans le presse-papiers* pour copier la valeur du **jeton SAP**. Enregistrez-la en vue d’une utilisation ultérieure.

    :::image type="content" source="./media/how-to-manage-routes-apis-cli/copy-sas-token.png" alt-text="Copiez le jeton SAP à utiliser dans le secret des messages non distribués." lightbox="./media/how-to-manage-routes-apis-cli/copy-sas-token.png":::
    
#### <a name="configure-the-endpoint"></a>Configurer le point de terminaison

Pour créer un point de terminaison pour lequel la file d’attente de lettres mortes est activée, vous devez créer le point de terminaison à l’aide des API Azure Resource Manager. 

1. Tout d’abord, référez-vous à la documentation sur l’[API Azure Resource Manager](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) pour configurer une demande de création d’un point de terminaison et renseignez les paramètres de demande requis. 

1. Ensuite, ajoutez un champ `deadLetterSecret` à l’objet Properties dans le **corps** de la demande. Définissez cette valeur en fonction du modèle ci-dessous, qui génère une URL à partir du nom du compte de stockage, du nom du conteneur et de la valeur du jeton SAP que vous avez collectés dans la [section précédente](#set-up-storage-resources).
      
    ```json
    {
      "properties": {
        "endpointType": "EventGrid",
        "TopicEndpoint": "https://contosoGrid.westus2-1.eventgrid.azure.net/api/events",
        "accessKey1": "xxxxxxxxxxx",
        "accessKey2": "xxxxxxxxxxx",
        "deadLetterSecret":"https://<storageAccountname>.blob.core.windows.net/<containerName>?<SASToken>"
      }
    }
    ```
1. Envoyez la demande de création du point de terminaison.

Pour plus d’informations sur la structuration de cette demande, consultez la documentation sur l’API REST Azure Digital Twins : [Points de terminaison : DigitalTwinsEndpoint CreateOrUpdate](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate).

### <a name="message-storage-schema"></a>Schéma de stockage des messages

Une fois que le point de terminaison avec la file d’attente de lettres mortes est configuré, les messages associés sont stockés au format suivant dans votre compte de stockage :

`{container}/{endpointName}/{year}/{month}/{day}/{hour}/{eventId}.json`

Les messages en file d’attente de lettres mortes correspondent au schéma de l’événement d’origine qui a été conçu pour être remis à votre point de terminaison d’origine.

Voici un exemple de message mis en file d’attente de lettres mortes pour une [notification de création de jumeau](how-to-interpret-event-data.md#digital-twin-life-cycle-notifications) :

```json
{
  "specversion": "1.0",
  "id": "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "<yourInstance>.api.<yourregion>.da.azuredigitaltwins-test.net",
  "data": {
    "$dtId": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "$etag": "W/\"xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "TwinData": "some sample",
    "$metadata": {
      "$model": "dtmi:test:deadlettermodel;1",
      "room": {
        "lastUpdateTime": "2020-10-14T01:11:49.3576659Z"
      }
    }
  },
  "subject": "<yourInstance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "time": "2020-10-14T01:11:49.3667224Z",
  "datacontenttype": "application/json",
  "traceparent": "00-889a9094ba22b9419dd9d8b3bfe1a301-f6564945cb20e94a-01"
}
```

## <a name="create-an-event-route"></a>Création d’un itinéraire d’événements

Pour envoyer concrètement des données d’Azure Digital Twins à un point de terminaison, vous devez définir un **itinéraire d’événement**. Les **API EventRoutes** d’Azure Digital Twins permettent aux développeurs de lier le flux d’événements au sein du système et aux services en aval. Pour en savoir plus sur les itinéraires d’événements, consultez [*Concepts : routage des événements Azure Digital Twins*](concepts-route-events.md).

Les exemples fournis dans cet article utilisent le [Kit de développement logiciel (SDK) C#](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true).

**Condition préalable** : Vous devez créer des points de terminaison comme décrit précédemment dans cet article avant de pouvoir passer à la création d’un itinéraire. Une fois que vos points de terminaison sont configurés, vous pouvez passer à la création d’un itinéraire d’événements.

> [!NOTE]
> Si vous avez récemment déployé vos points de terminaison, vérifiez que le déploiement est terminé **avant** de tenter de les utiliser pour un nouvel itinéraire d’événements. Si le déploiement de l’itinéraire échoue parce que les points de terminaison ne sont pas prêts, patientez quelques minutes, puis réessayez.
>
> Si vous créez un script pour ce flux, vous pouvez prendre cela en compte en intégrant un temps d’attente de 2-3 minutes pour permettre au service de point de terminaison d’achever le déploiement avant de passer à la configuration de l’itinéraire.

### <a name="creation-code-with-apis-and-the-c-sdk"></a>Code de création avec des API et le kit de développement logiciel (SDK) C#

Les itinéraires d’événements sont définis à l’aide d’[API de plan de données](how-to-use-apis-sdks.md#overview-data-plane-apis). 

Une définition d’itinéraire peut contenir les éléments suivants :
* nom de l’itinéraire que vous souhaitez utiliser ;
* nom du point de terminaison que vous souhaitez utiliser ;
* filtre définissant les événements envoyés au point de terminaison. 

À défaut de nom d’itinéraire, aucun message n’est acheminé en dehors d’Azure Digital Twins. S’il existe un nom d’itinéraire et que le filtre est `true`, tous les messages sont acheminés vers le point de terminaison. Si un nom d’itinéraire et un filtre différent sont ajoutés, les messages sont filtrés en fonction du filtre.

Un itinéraire doit permettre la sélection de plusieurs notifications et types d’événements. 

`CreateOrReplaceEventRouteAsync` est l’appel du kit de développement logiciel (SDK) utilisé pour ajouter un itinéraire d’événement. Voici un exemple de son utilisation :

```csharp
string eventFilter = "$eventType = 'DigitalTwinTelemetryMessages' or $eventType = 'DigitalTwinLifecycleNotification'";
var er = new DigitalTwinsEventRoute("<your-endpointName>", eventFilter);
await client.CreateOrReplaceEventRouteAsync("routeName", er);
```
    
> [!TIP]
> Toutes les fonctions du Kit de développement logiciel (SDK) sont disponibles en versions synchrone et asynchrone.

### <a name="event-route-sample-code"></a>Exemple de code d’itinéraire d’événement

L’exemple de méthode suivant montre comment créer, lister et supprimer une route d’événement :
```csharp
private async static Task CreateEventRoute(DigitalTwinsClient client, String routeName, DigitalTwinsEventRoute er)
{
  try
  {
    Console.WriteLine("Create a route: testRoute1");
            
    // Make a filter that passes everything
    er.Filter = "true";
    await client.CreateOrReplaceEventRouteAsync(routeName, er);
    Console.WriteLine("Create route succeeded. Now listing routes:");
    Pageable<DigitalTwinsEventRoute> result = client.GetEventRoutes();
    foreach (DigitalTwinsEventRoute r in result)
    {
        Console.WriteLine($"Route {r.Id} to endpoint {r.EndpointName} with filter {r.Filter} ");
    }
    Console.WriteLine("Deleting routes:");
    foreach (DigitalTwinsEventRoute r in result)
    {
        Console.WriteLine($"Deleting route {r.Id}:");
        client.DeleteEventRoute(r.Id);
    }
  }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"*** Error in event route processing ({e.ErrorCode}):\n${e.Message}");
    }
  }
```

## <a name="filter-events"></a>Filtrer les événements

Sans filtrage, les points de terminaison reçoivent un grand nombre d’événements d’Azure Digital Twins :
* télémétrie déclenchée par des [jumeaux numériques](concepts-twins-graph.md) à l’aide de l’API de service d’Azure Digital Twins ;
* notifications de changement de propriété de jumeau, déclenchées par des modifications des propriétés d’un jumeau dans l’instance Azure Digital Twins ;
* événements de cycle de vie déclenchés lors de la création ou de la suppression de jumeaux ou de relations ;

Vous pouvez limiter les événements envoyés en ajoutant un **filtre** à un point de terminaison sur votre itinéraire d’événement.

Pour ajouter un filtre, vous pouvez envoyer une demande PUT à *https://{Your-azure-digital-twins-hostname}/eventRoutes/{event-route-name}?api-version=2020-10-31* avec le corps suivant :

```json  
{
    "endpointName": "<endpoint-name>",
    "filter": "<filter-text>"
}
``` 
Voici les filtres d’itinéraire pris en charge. Utilisez les détails de la colonne *Filtrer le schéma du texte* pour remplacer l’espace réservé `<filter-text>` dans le corps de la requête ci-dessus.

[!INCLUDE [digital-twins-route-filters](../../includes/digital-twins-route-filters.md)]

## <a name="manage-endpoints-and-routes-with-cli"></a>Gérer les points de terminaison et les itinéraires avec l’interface de ligne de commande

Vous pouvez également gérer les points de terminaison et les itinéraires à l’aide de l’interface de commande d’Azure Digital Twins. Pour plus d’informations sur l’utilisation de l’interface CLI et sur les commandes disponibles, consultez le [*Tutoriel : utiliser l’interface CLI Azure Digital Twins*](how-to-use-cli.md).

[!INCLUDE [digital-twins-route-metrics](../../includes/digital-twins-route-metrics.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les différents types de messages d’événements que vous pouvez recevoir :
* [*Guide pratique pour interpréter des données d’événement*](how-to-interpret-event-data.md)
