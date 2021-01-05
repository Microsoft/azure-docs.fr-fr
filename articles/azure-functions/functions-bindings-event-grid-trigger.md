---
title: Déclencheur Azure Event Grid pour Azure Functions
description: Apprenez à exécuter du code lors de la distribution des événements Event Grid dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: devx-track-csharp, fasttrack-edit, devx-track-python, devx-track-azurecli
ms.openlocfilehash: 0e2e09bc72991330ccdec7a35400460cbeba26fc
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327030"
---
# <a name="azure-event-grid-trigger-for-azure-functions"></a>Déclencheur Azure Event Grid pour Azure Functions

Utilisez le déclencheur de fonction pour répondre à un événement envoyé à une rubrique Event Grid.

Pour plus d'informations sur les détails d'installation et de configuration, consultez la [vue d'ensemble](./functions-bindings-event-grid.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

Pour obtenir un exemple de déclencheur HTTP, consultez [Recevoir des événements sur un point de terminaison HTTP](../event-grid/receive-events.md).

### <a name="c-2x-and-higher"></a>C# (2.x et ultérieur)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) liée à `EventGridEvent` :

```cs
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTest")]
        public static void EventGridTest([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

Pour plus d'informations, consultez Packages, [Attributs](#attributes-and-annotations), [Configuration](#configuration) et [Utilisation](#usage).

### <a name="version-1x"></a>Version 1.x

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) Functions 1.x liée à `JObject` :

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTriggerCSharp")]
        public static void Run([EventGridTrigger]JObject eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
        }
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison.

Voici les données de liaison dans le fichier *function.json* :

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

### <a name="version-2x-and-higher"></a>Versions 2.x et ultérieures

Voici un exemple qui crée une liaison avec `EventGridEvent` :

```csharp
#r "Microsoft.Azure.EventGrid"
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(EventGridEvent eventGridEvent, ILogger log)
{
    log.LogInformation(eventGridEvent.Data.ToString());
}
```

Pour plus d'informations, consultez Packages, [Attributs](#attributes-and-annotations), [Configuration](#configuration) et [Utilisation](#usage).

### <a name="version-1x"></a>Version 1.x

Voici le code de script C# Functions 1.x qui lie à `JObject` :

```cs
#r "Newtonsoft.Json"

using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static void Run(JObject eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.ToString(Formatting.Indented));
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison.

Voici les données de liaison dans le fichier *function.json* :

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, eventGridEvent) {
    context.log("JavaScript Event Grid function processed a request.");
    context.log("Subject: " + eventGridEvent.subject);
    context.log("Time: " + eventGridEvent.eventTime);
    context.log("Data: " + JSON.stringify(eventGridEvent.data));
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui utilise la liaison.

Voici les données de liaison dans le fichier *function.json* :

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "event",
      "direction": "in"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

Voici le code Python :

```python
import json
import logging

import azure.functions as func

def main(event: func.EventGridEvent):

    result = json.dumps({
        'id': event.id,
        'data': event.get_json(),
        'topic': event.topic,
        'subject': event.subject,
        'event_type': event.event_type,
    })

    logging.info('Python EventGrid trigger processed an event: %s', result)
```

# <a name="java"></a>[Java](#tab/java)

Cette section contient les exemples suivants :

* [Déclencheur Event Grid, paramètre String](#event-grid-trigger-string-parameter)
* [Déclencheur Event Grid, paramètre POJO](#event-grid-trigger-pojo-parameter)

Les exemples suivants illustrent la liaison de déclencheur en [Java](functions-reference-java.md). Ils utilisent la liaison et affichent un événement, en recevant d'abord l'événement en tant que `String` puis en tant que POJO.

### <a name="event-grid-trigger-string-parameter"></a>Déclencheur Event Grid, paramètre String

```java
  @FunctionName("eventGridMonitorString")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    String content, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: " + content);      
  }
```

### <a name="event-grid-trigger-pojo-parameter"></a>Déclencheur Event Grid, paramètre POJO

Cet exemple utilise le POJO suivant, représentant les propriétés de niveau supérieur d’un événement Event Grid :

```java
import java.util.Date;
import java.util.Map;

public class EventSchema {

  public String topic;
  public String subject;
  public String eventType;
  public Date eventTime;
  public String id;
  public String dataVersion;
  public String metadataVersion;
  public Map<String, Object> data;

}
```

À l’arrivée, la charge utile JSON de l’événement est désérialisée dans le POJO ```EventSchema``` pour une utilisation par la fonction. Ce processus permet à la fonction d'accéder aux propriétés de l'événement selon une approche orientée objet.

```java
  @FunctionName("eventGridMonitor")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    EventSchema event, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: ");
      context.getLogger().info("Subject: " + event.subject);
      context.getLogger().info("Time: " + event.eventTime); // automatically converted to Date by the runtime
      context.getLogger().info("Id: " + event.id);
      context.getLogger().info("Data: " + event.data);
  }
```

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `EventGridTrigger` sur les paramètres dont la valeur proviendrait d’EventGrid. Les paramètres ayant ces annotations entraînent l’exécution de la fonction quand un événement se produit.  Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [EventGridTrigger](https://github.com/Azure/azure-functions-eventgrid-extension/blob/master/src/EventGridExtension/TriggerBinding/EventGridTriggerAttribute.cs).

Voici un attribut `EventGridTrigger` dans une signature de méthode :

```csharp
[FunctionName("EventGridTest")]
public static void EventGridTest([EventGridTrigger] JObject eventGridEvent, ILogger log)
{
    ...
}
```

Vous trouverez un exemple complet sur la page Exemple C#.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

L’annotation [EventGridTrigger](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/annotation/EventGridTrigger.java) vous permet de configurer de façon déclarative une liaison Event Grid en fournissant des valeurs de configuration. Pour plus d’informations, consultez les sections [Exemple](#example) et [Configuration](#configuration).

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json*. Il n’y a aucun paramètre de constructeur ni aucune propriété à définir dans l’attribut `EventGridTrigger`.

|Propriété function.json |Description|
|---------|---------|
| **type** | Obligatoire : doit être défini sur `eventGridTrigger`. |
| **direction** | Obligatoire : doit être défini sur `in`. |
| **name** | Obligatoire : nom de variable utilisé dans le code de fonction pour le paramètre qui reçoit les données de l’événement. |

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Dans Azure Functions 1.x et ultérieur, vous pouvez utiliser les types de paramètres suivants pour le déclencheur Event Grid :

* `JObject`
* `string`

Dans Azure Functions 2.x et ultérieur, vous pouvez également utiliser le type de paramètre suivant pour le déclencheur Event Grid :

* `Microsoft.Azure.EventGrid.Models.EventGridEvent`- Définit les propriétés pour les champs communs à tous les types d’événements.

> [!NOTE]
> Dans Functions v1, si vous essayez de lier à `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent`, le compilateur affiche un message « déprécié » et vous conseille d’utiliser `Microsoft.Azure.EventGrid.Models.EventGridEvent` à la place. Pour utiliser le type le plus récent, référencez le package NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) et qualifiez entièrement le nom de type `EventGridEvent` en le faisant précéder de `Microsoft.Azure.EventGrid.Models`.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Dans Azure Functions 1.x et ultérieur, vous pouvez utiliser les types de paramètres suivants pour le déclencheur Event Grid :

* `JObject`
* `string`

Dans Azure Functions 2.x et ultérieur, vous pouvez également utiliser le type de paramètre suivant pour le déclencheur Event Grid :

* `Microsoft.Azure.EventGrid.Models.EventGridEvent`- Définit les propriétés pour les champs communs à tous les types d’événements.

> [!NOTE]
> Dans Functions v1, si vous essayez de lier à `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent`, le compilateur affiche un message « déprécié » et vous conseille d’utiliser `Microsoft.Azure.EventGrid.Models.EventGridEvent` à la place. Pour utiliser le type le plus récent, référencez le package NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) et qualifiez entièrement le nom de type `EventGridEvent` en le faisant précéder de `Microsoft.Azure.EventGrid.Models`. Pour plus d’informations sur la façon de référencer des packages NuGet dans une fonction de script C#, consultez [Utiliser des packages NuGet](functions-reference-csharp.md#using-nuget-packages).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’instance Event Grid est disponible via le paramètre configuré dans la propriété `name` du fichier *function.json*.

# <a name="python"></a>[Python](#tab/python)

L’instance Event Grid est disponible via le paramètre configuré dans la propriété `name` du fichier *function.json*, typée en `func.EventGridEvent`.

# <a name="java"></a>[Java](#tab/java)

L’instance d’événement Event Grid est disponible via le paramètre associé à l’attribut `EventGridTrigger`, typé en `EventSchema`. Pour plus d’informations, consultez l’[exemple](#example).

---

## <a name="event-schema"></a>Schéma d’événement

Les données d’un événement Event Grid sont réceptionnées sous la forme d’un objet JSON dans le corps d’une requête HTTP, comme dans l’exemple suivant :

```json
[{
  "topic": "/subscriptions/{subscriptionid}/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstore",
  "subject": "/blobServices/default/containers/{containername}/blobs/blobname.jpg",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2018-01-23T17:02:19.6069787Z",
  "id": "{guid}",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "{guid}",
    "requestId": "{guid}",
    "eTag": "0x8D562831044DDD0",
    "contentType": "application/octet-stream",
    "contentLength": 2248,
    "blobType": "BlockBlob",
    "url": "https://egblobstore.blob.core.windows.net/{containername}/blobname.jpg",
    "sequencer": "000000000000272D000000000003D60F",
    "storageDiagnostics": {
      "batchId": "{guid}"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

Cet exemple est un tableau comportant un seul élément. Event Grid envoie toujours un tableau et peut y inclure plusieurs événements. Le runtime appelle la fonction une fois par élément du tableau.

Les propriétés de niveau supérieur présentes dans les données JSON de l’événement sont les mêmes quel que soit le type d’événement, tandis que la valeur de la propriété `data` est propre à chaque type d’événement. L’exemple correspond à un événement de Stockage Blob.

Vous trouverez des explications sur les propriétés communes et propres aux événements dans la section [Propriétés des événements](../event-grid/event-schema.md#event-properties) de la documentation Event Grid.

Le type `EventGridEvent` définit uniquement les propriétés de niveau supérieur ; la propriété `Data` est un `JObject`.

## <a name="create-a-subscription"></a>Création d’un abonnement

Pour pouvoir recevoir des requêtes HTTP Event Grid, créez un abonnement Event Grid spécifiant l’URL de point de terminaison qui appelle la fonction.

### <a name="azure-portal"></a>Portail Azure

Pour les fonctions développées dans le portail Azure avec le déclencheur Event Grid, sélectionnez **Intégration**, puis choisissez **Event Grid Trigger** et sélectionnez **Créer un abonnement Event Grid**.

:::image type="content" source="media/functions-bindings-event-grid/portal-sub-create.png" alt-text="Connectez un nouvel abonnement aux événements à déclencher dans le portail.":::

Ce lien ouvre la page **Créer un abonnement aux événements** dans le portail avec le point de terminaison du déclencheur actuel déjà défini.

:::image type="content" source="media/functions-bindings-event-grid/endpoint-url.png" alt-text="Créer un abonnement aux événements avec le point de terminaison de fonction déjà défini" :::

Pour plus d’informations sur la création d’abonnements à l’aide du Portail Azure, consultez la section [Créer un événement personnalisé - Portail Azure](../event-grid/custom-event-quickstart-portal.md) dans la documentation Event Grid.

### <a name="azure-cli"></a>Azure CLI

Pour créer un abonnement avec [Azure CLI](/cli/azure/get-started-with-azure-cli), utilisez la commande [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create).

La commande a besoin de l’URL de point de terminaison qui appelle la fonction. L’exemple suivant illustre le modèle d’URL spécifique à la version :

#### <a name="version-2x-and-higher-runtime"></a>Runtime de la version 2.x (et ultérieure)

```http
https://{functionappname}.azurewebsites.net/runtime/webhooks/eventgrid?functionName={functionname}&code={systemkey}
```

#### <a name="version-1x-runtime"></a>Runtime de la version 1.x

```http
https://{functionappname}.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName={functionname}&code={systemkey}
```

La clé système est une clé d’autorisation qui doit être incluse dans l’URL de point de terminaison d’un déclencheur Event Grid. La section suivante explique comment l’obtenir.

Voici un exemple d’abonnement à un compte de Stockage Blob (avec un espace réservé pour la clé système) :

#### <a name="version-2x-and-higher-runtime"></a>Runtime de la version 2.x (et ultérieure)

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
    --provider-namespace Microsoft.Storage --resource-type storageAccounts \
    --resource-name myblobstorage12345 --name myFuncSub \
    --included-event-types Microsoft.Storage.BlobCreated \
    --subject-begins-with /blobServices/default/containers/images/blobs/ \
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/runtime/webhooks/eventgrid?functionName=imageresizefunc&code=<key>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup ^
    --provider-namespace Microsoft.Storage --resource-type storageAccounts ^
    --resource-name myblobstorage12345 --name myFuncSub ^
    --included-event-types Microsoft.Storage.BlobCreated ^
    --subject-begins-with /blobServices/default/containers/images/blobs/ ^
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/runtime/webhooks/eventgrid?functionName=imageresizefunc&code=<key>
```

---

#### <a name="version-1x-runtime"></a>Runtime de la version 1.x

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
    --provider-namespace Microsoft.Storage --resource-type storageAccounts \
    --resource-name myblobstorage12345 --name myFuncSub \
    --included-event-types Microsoft.Storage.BlobCreated \
    --subject-begins-with /blobServices/default/containers/images/blobs/ \
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=<key>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup ^
    --provider-namespace Microsoft.Storage --resource-type storageAccounts ^
    --resource-name myblobstorage12345 --name myFuncSub ^
    --included-event-types Microsoft.Storage.BlobCreated ^
    --subject-begins-with /blobServices/default/containers/images/blobs/ ^
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=<key>
```

---

Pour plus d’informations sur la création d’un abonnement, consultez la section [Guide de démarrage rapide sur le Stockage Blob](../storage/blobs/storage-blob-event-quickstart.md#subscribe-to-your-storage-account) ou les autres guides de démarrage rapide Event Grid.

### <a name="get-the-system-key"></a>Obtenir la clé système

Vous pouvez obtenir la clé système à l’aide de l’API suivante (HTTP GET) :

#### <a name="version-2x-and-higher-runtime"></a>Runtime de la version 2.x (et ultérieure)

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgrid_extension?code={masterkey}
```

#### <a name="version-1x-runtime"></a>Runtime de la version 1.x

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgridextensionconfig_extension?code={masterkey}
```

Il s’agit d’une API d’administration, qui a donc besoin de la [clé principale](functions-bindings-http-webhook-trigger.md#authorization-keys) de votre application de fonction. Ne confondez pas la clé système (permettant d’appeler une fonction de déclenchement Event Grid) avec la clé principale (servant à effectuer des tâches d’administration sur l’application de fonction). Veillez à utiliser la clé système pour vous abonner à une rubrique Event Grid.

Voici un exemple de réponse fournissant la clé système :

```
{
  "name": "eventgridextensionconfig_extension",
  "value": "{the system key for the function}",
  "links": [
    {
      "rel": "self",
      "href": "{the URL for the function, without the system key}"
    }
  ]
}
```

Vous pouvez obtenir la clé principale pour votre application de fonction à partir de l’onglet **Paramètres de l’application de fonction** dans le portail.

> [!IMPORTANT]
> La clé principale fournit un accès administrateur à votre application de fonction. Ne partagez pas cette clé avec des tiers et ne la distribuez pas dans les applications clientes natives.

Pour plus d’informations, consultez la section [Clés d’autorisation](functions-bindings-http-webhook-trigger.md#authorization-keys) dans l’article de référence sur les déclencheurs HTTP.

Vous pouvez également envoyer une requête HTTP PUT pour spécifier la valeur de la clé vous-même.

## <a name="local-testing-with-viewer-web-app"></a>Tests locaux avec une application web de visionneuse

Pour qu’un déclencheur Event Grid soit testé en local, les requêtes HTTP Event Grid doivent partir de leur origine dans le cloud et arriver sur l’ordinateur local. Vous pouvez pour cela capturer les requêtes en ligne et les renvoyer manuellement sur votre ordinateur local :

1. [Créez une application web de visionneuse](#create-a-viewer-web-app) qui capture les messages d’événement.
1. [Créez un abonnement Event Grid](#create-an-event-grid-subscription) qui envoie les événements à l’application de visionneuse.
1. [Générez une requête](#generate-a-request) et copiez le corps de la requête à partir de l’application de visionneuse.
1. [Publiez (POST) manuellement la requête](#manually-post-the-request) sur l’URL localhost de votre fonction de déclenchement Event Grid.

À l’issue des tests, vous pourrez utiliser le même abonnement en production en mettant à jour le point de terminaison. Utilisez la commande Azure CLI [az eventgrid event-subscription update](/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-update).

### <a name="create-a-viewer-web-app"></a>Créer une application web de visionneuse

Pour simplifier la capture des messages d’événement, vous pouvez déployer une [application web prédéfinie](https://github.com/Azure-Samples/azure-event-grid-viewer) qui affiche les messages d’événement. La solution déployée comprend un plan App Service, une offre App Service Web Apps et du code source en provenance de GitHub.

Sélectionnez **Déployer sur Azure** pour déployer la solution sur votre abonnement. Dans le portail Azure, indiquez des valeurs pour les paramètres.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png" alt="Button to Deploy to Azure." /></a>

Le déploiement peut prendre quelques minutes. Une fois le déploiement réussi, affichez votre application web pour vérifier qu’elle s’exécute. Dans un navigateur web, accédez à : `https://<your-site-name>.azurewebsites.net`

Vous voyez le site, mais aucun événement n’est encore posté sur celui-ci.

![Afficher le nouveau site](media/functions-bindings-event-grid/view-site.png)

### <a name="create-an-event-grid-subscription"></a>Créer un abonnement Event Grid

Créez un abonnement Event Grid du type que vous souhaitez tester et attribuez-lui l’URL de votre application web comme point de terminaison pour la notification d’événement. Le point de terminaison de votre application web doit inclure le suffixe `/api/updates/`. L’URL complète est donc `https://<your-site-name>.azurewebsites.net/api/updates`

Pour plus d’informations sur la création d’abonnements à l’aide du portail Azure, consultez la section [Créer un événement personnalisé - Portail Azure](../event-grid/custom-event-quickstart-portal.md) dans la documentation Event Grid.

### <a name="generate-a-request"></a>Générer une requête

Déclenchez un événement qui génèrera du trafic HTTP sur le point de terminaison de votre application web.  Par exemple, si vous avez créé un abonnement au Stockage Blob, chargez ou supprimez un objet blob. Lorsqu’une requête s’affiche dans votre application web, copiez le corps de la requête.

La requête de validation d’abonnement sera reçue la première ; ignorez toutes les requêtes de validation et copiez la requête d’événement.

![Copier le corps de la requête à partir de l’application web](media/functions-bindings-event-grid/view-results.png)

### <a name="manually-post-the-request"></a>Publier (POST) manuellement la requête

Exécutez votre fonction Event Grid en local.

Utilisez un outil comme [Postman](https://www.getpostman.com/) ou [curl](https://curl.haxx.se/docs/httpscripting.html) pour créer une requête HTTP POST :

* Définissez un en-tête `Content-Type: application/json`.
* Définissez un en-tête `aeg-event-type: Notification`.
* Collez les données RequestBin dans le corps de la requête.
* Publiez (POST) sur l’URL de votre fonction de déclenchement Event Grid.
  * Pour 2.x et ultérieur, utilisez le modèle suivant :

    ```
    http://localhost:7071/runtime/webhooks/eventgrid?functionName={FUNCTION_NAME}
    ```

  * Pour 1.x, utilisez :

    ```
    http://localhost:7071/admin/extensions/EventGridExtensionConfig?functionName={FUNCTION_NAME}
    ```

Le paramètre `functionName` doit être le nom spécifié dans l’attribut `FunctionName`.

Les captures d’écran suivantes montrent les en-têtes et le corps de la requête dans Postman :

![En-têtes dans Postman](media/functions-bindings-event-grid/postman2.png)

![Corps de la requête dans Postman](media/functions-bindings-event-grid/postman.png)

La fonction de déclenchement Event Grid s’exécute et affiche des journaux d’activité de ce type :

![Exemple de journaux d’activité d’une fonction de déclenchement Event Grid](media/functions-bindings-event-grid/eg-output.png)

## <a name="next-steps"></a>Étapes suivantes

* [Distribuer un événement Event Grid](./functions-bindings-event-grid-output.md)
