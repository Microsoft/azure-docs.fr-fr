---
title: Créer un point de terminaison RESTful
description: Ce tutoriel explique comment créer un point de terminaison RESTful pour des fournisseurs personnalisés. Il décrit en détail comment gérer les requêtes et les réponses des méthodes HTTP RESTful prises en charge.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: d7f6c51211ce0572797ade659b9316003502da1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75648726"
---
# <a name="author-a-restful-endpoint-for-custom-providers"></a>Créer un point de terminaison RESTful pour des fournisseurs personnalisés

Un fournisseur personnalisé implique qu’un contrat soit passé entre Azure et un point de terminaison. Les fournisseurs personnalisés vous permettent de personnaliser des workflows dans Azure. Ce tutoriel montre comment créer un point de terminaison RESTful de fournisseur personnalisé. Si vous ne savez pas encore ce qu’est un fournisseur personnalisé Azure, consultez la [présentation des fournisseurs de ressources personnalisés](overview.md).

> [!NOTE]
> Ce tutoriel repose sur le tutoriel intitulé [Configurer des fonctions Azure pour des fournisseurs personnalisés Azure](./tutorial-custom-providers-function-setup.md). Certaines des étapes de ce tutoriel ne fonctionnent que si une application de fonction Azure a été configurée pour utiliser des fournisseurs personnalisés.

## <a name="work-with-custom-actions-and-custom-resources"></a>Utiliser des actions et des ressources personnalisées

Dans ce tutoriel, vous allez mettre à jour l’application de fonction afin qu’elle fonctionne comme un point de terminaison RESTful pour le fournisseur personnalisé. Dans Azure, les ressources et les actions sont modélisées selon la spécification RESTful de base :

- **PUT** : Créer une nouvelle ressource
- **GET (instance)**  : Récupère une ressource existante
- **DELETE** : Supprime une ressource existante
- **POST** : Déclenche une action
- **GET (collection)**  : Liste toutes les ressources existantes

 Dans ce tutoriel, vous allez utiliser le stockage Table Azure. Vous pouvez toutefois utiliser n’importe quel service de base de données ou de stockage.

## <a name="partition-custom-resources-in-storage"></a>Partitionner des ressources personnalisées dans le stockage

Dans la mesure où vous créez un service RESTful, vous avez besoin de stocker les ressources créées. Pour le stockage Table Azure, vous devez générer des clés de partition et de ligne pour vos données. Dans le cas de fournisseurs personnalisés, les données doivent être partitionnées par le fournisseur personnalisé. Quand une requête entrante est envoyée au fournisseur personnalisé, celui-ci ajoute l’en-tête `x-ms-customproviders-requestpath` aux requêtes sortantes à destination du point de terminaison.

L’exemple suivant montre l’en-tête `x-ms-customproviders-requestpath` d’une ressource personnalisée :

```
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{myResourceType}/{myResourceName}
```

En fonction de l’en-tête `x-ms-customproviders-requestpath` de l’exemple, vous pouvez créer les paramètres *partitionKey* et *rowKey* pour votre stockage, comme indiqué dans le tableau suivant :

Paramètre | Modèle | Description
---|---|---
*partitionKey* | `{subscriptionId}:{resourceGroupName}:{resourceProviderName}` | Le paramètre *partitionKey* spécifie la façon dont les données sont partitionnées. En général, les données doivent être partitionnées par l’instance de fournisseur personnalisé.
*rowKey* | `{myResourceType}:{myResourceName}` | Le paramètre *rowKey* spécifie l’identificateur des données. En général, l’identificateur correspond au nom de la ressource.

Vous devez également créer une classe pour modéliser votre ressource personnalisée. Dans ce tutoriel, vous allez ajouter la classe **CustomResource** suivante à votre application de fonction :

```csharp
// Custom Resource Table Entity
public class CustomResource : TableEntity
{
    public string Data { get; set; }
}
```
**CustomResource** est une classe générique simple qui accepte tout type de données d’entrée. Elle est basée sur **TableEntity**, qui est utilisé pour stocker des données. La classe **CustomResource** hérite de **TableEntity** les deux propriétés suivantes : **partitionKey** et **rowKey**.

## <a name="support-custom-provider-restful-methods"></a>Prendre en charge les méthodes RESTful de fournisseur personnalisé

> [!NOTE]
> Si vous ne copiez pas le code directement à partir de ce tutoriel, la réponse doit être constituée de code JSON valide qui définit l’en-tête `Content-Type` sur `application/json`.

Maintenant que vous avez configuré le partitionnement des données, vous pouvez créer les méthodes CRUD et les méthodes de déclenchement de base pour les ressources et les actions personnalisées. Étant donné que les fournisseurs personnalisés servent de proxys, le point de terminaison RESTful doit modéliser et gérer la requête et la réponse. Les extraits de code suivants montrent comment gérer les opérations RESTful de base.

### <a name="trigger-a-custom-action"></a>Déclencher une action personnalisée

Pour les fournisseurs personnalisés, une action personnalisée est déclenchée par le biais de requêtes POST. Une action personnalisée peut éventuellement accepter un corps de demande qui contient un ensemble de paramètres d’entrée. L’action retourne ensuite une réponse signalant le résultat de l’action et si celle-ci a réussi ou échoué.

Ajoutez la méthode **TriggerCustomAction** suivante à votre application de fonction :

```csharp
/// <summary>
/// Triggers a custom action with some side effects.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <returns>The HTTP response result of the custom action.</returns>
public static async Task<HttpResponseMessage> TriggerCustomAction(HttpRequestMessage requestMessage)
{
    var myCustomActionRequest = await requestMessage.Content.ReadAsStringAsync();

    var actionResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    actionResponse.Content = myCustomActionRequest != string.Empty ? 
        new StringContent(JObject.Parse(myCustomActionRequest).ToString(), System.Text.Encoding.UTF8, "application/json") :
        null;
    return actionResponse;
}
```

La méthode **TriggerCustomAction** accepte une requête entrante et renvoie simplement la réponse avec un code d’état.

### <a name="create-a-custom-resource"></a>Créer une ressource personnalisée

Pour les fournisseurs personnalisés, une ressource personnalisée est créée au moyen de requêtes PUT. Le fournisseur personnalisé accepte un corps de requête JSON qui contient un ensemble de propriétés pour la ressource personnalisée. Dans Azure, les ressources suivent un modèle RESTful. Vous pouvez utiliser la même URL de requête pour créer, récupérer ou supprimer une ressource.

Ajoutez la méthode **CreateCustomResource** suivante pour créer des ressources :

```csharp
/// <summary>
/// Creates a custom resource and saves it to table storage.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="azureResourceId">The parsed Azure resource ID.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the created custom resource.</returns>
public static async Task<HttpResponseMessage> CreateCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, ResourceId azureResourceId, string partitionKey, string rowKey)
{
    // Adds the Azure top-level properties.
    var myCustomResource = JObject.Parse(await requestMessage.Content.ReadAsStringAsync());
    myCustomResource["name"] = azureResourceId.Name;
    myCustomResource["type"] = azureResourceId.FullResourceType;
    myCustomResource["id"] = azureResourceId.Id;

    // Save the resource into storage.
    var insertOperation = TableOperation.InsertOrReplace(
        new CustomResource
        {
            PartitionKey = partitionKey,
            RowKey = rowKey,
            Data = myCustomResource.ToString(),
        });
    await tableStorage.ExecuteAsync(insertOperation);

    var createResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    createResponse.Content = new StringContent(myCustomResource.ToString(), System.Text.Encoding.UTF8, "application/json");
    return createResponse;
}
```

La méthode **CreateCustomResource** met à jour la requête entrante en y ajoutant les champs propres à Azure : **id**, **name** et **type**. Ces champs correspondent à des propriétés de niveau supérieur utilisées par les services dans Azure. Elles permettent au fournisseur personnalisé d’interagir avec d’autres services comme Azure Policy, les modèles Azure Resource Manager et les journaux d’activité Azure.

Propriété | Exemple | Description
---|---|---
**name** | {myCustomResourceName} | Nom de la ressource personnalisée
**type** | Microsoft.CustomProviders/resourceProviders/{resourceTypeName} | Espace de noms resource-type
**id** | /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/<br>providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/<br>{resourceTypeName}/{myCustomResourceName} | ID de la ressource

En plus de l’ajout des propriétés, vous avez également enregistré le document JSON dans le stockage Table Azure.

### <a name="retrieve-a-custom-resource"></a>Récupérer une ressource personnalisée

Pour les fournisseurs personnalisés, une ressource personnalisée est récupérée au moyen de requêtes GET. Le fournisseur personnalisé *n’accepte pas* un corps de requête JSON. Pour les requêtes GET, le point de terminaison utilise l’en-tête `x-ms-customproviders-requestpath` afin de retourner la ressource déjà créée.

Ajoutez la méthode **RetrieveCustomResource** suivante pour récupérer les ressources existantes :

```csharp
/// <summary>
/// Retrieves a custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the existing custom resource.</returns>
public static async Task<HttpResponseMessage> RetrieveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    var retrieveResponse = requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NotFound);

    retrieveResponse.Content = existingCustomResource != null ?
            new StringContent(existingCustomResource.Data, System.Text.Encoding.UTF8, "application/json"):
            null;
    return retrieveResponse;
}
```

Dans Azure, les ressources suivent un modèle RESTful. L’URL de la requête qui crée une ressource retourne également la ressource si une requête GET est exécutée.

### <a name="remove-a-custom-resource"></a>Supprimer une ressource personnalisée

Pour les fournisseurs personnalisés, une ressource personnalisée est supprimée au moyen de requêtes DELETE. Le fournisseur personnalisé *n’accepte pas* un corps de requête JSON. Pour les requêtes DELETE, le point de terminaison utilise l’en-tête `x-ms-customproviders-requestpath` afin de supprimer la ressource déjà créée.

Ajoutez la méthode **RemoveCustomResource** suivante pour supprimer les ressources existantes :

```csharp
/// <summary>
/// Removes an existing custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure storage account table.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the result of the deletion.</returns>
public static async Task<HttpResponseMessage> RemoveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    if (existingCustomResource != null) {
        var deleteOperation = TableOperation.Delete(existingCustomResource);
        await tableStorage.ExecuteAsync(deleteOperation);
    }

    return requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NoContent);
}
```

Dans Azure, les ressources suivent un modèle RESTful. L’URL de la requête qui crée une ressource supprime également la ressource si une requête DELETE est exécutée.

### <a name="list-all-custom-resources"></a>Lister toutes les ressources personnalisées

Pour les fournisseurs personnalisés, vous pouvez énumérer la liste des ressources personnalisées existantes au moyen d’une collection de requêtes GET. Le fournisseur personnalisé *n’accepte pas* un corps de requête JSON. Pour une collection de requêtes GET, le point de terminaison utilise l’en-tête `x-ms-customproviders-requestpath` pour énumérer les ressources déjà créées.

Ajoutez la méthode **EnumerateAllCustomResources** suivante pour énumérer les ressources existantes :

```csharp
/// <summary>
/// Enumerates all the stored custom resources for a given type.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="resourceType">The resource type of the enumeration.</param>
/// <returns>The HTTP response containing a list of resources stored under 'value'.</returns>
public static async Task<HttpResponseMessage> EnumerateAllCustomResources(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string resourceType)
{
    // Generate upper bound of the query.
    var rowKeyUpperBound = new StringBuilder(resourceType);
    rowKeyUpperBound[rowKeyUpperBound.Length - 1]++;

    // Create the enumeration query.
    var enumerationQuery = new TableQuery<CustomResource>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, partitionKey),
            TableOperators.And,
            TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, resourceType),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, rowKeyUpperBound.ToString()))));
    
    var customResources = (await tableStorage.ExecuteQuerySegmentedAsync(enumerationQuery, null))
        .ToList().Select(customResource => JToken.Parse(customResource.Data));

    var enumerationResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    enumerationResponse.Content = new StringContent(new JObject(new JProperty("value", customResources)).ToString(), System.Text.Encoding.UTF8, "application/json");
    return enumerationResponse;
}
```

> [!NOTE]
> Les clés de ligne QueryComparisons.GreaterThan et QueryComparisons.LessThan correspondent à la syntaxe du stockage Table Azure qui permet d’exécuter une requête « startswith » pour des chaînes.

Pour lister toutes les ressources existantes, générez une requête de stockage Table Azure qui vérifie que les ressources existent sous la partition de votre fournisseur personnalisé. La requête vérifie ensuite que la clé de ligne commence par la même valeur `{myResourceType}`.

## <a name="integrate-restful-operations"></a>Intégrer des opérations RESTful

Une fois toutes les méthodes RESTful ajoutées à l’application de fonction, mettez à jour la méthode **Run** principale pour appeler les fonctions et gérer les différentes requêtes REST :

```csharp
/// <summary>
/// Entry point for the Azure function app webhook that acts as the service behind a custom provider.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="log">The logger.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <returns>The HTTP response for the custom Azure API.</returns>
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log, CloudTable tableStorage)
{
    // Get the unique Azure request path from request headers.
    var requestPath = req.Headers.GetValues("x-ms-customproviders-requestpath").FirstOrDefault();

    if (requestPath == null)
    {
        var missingHeaderResponse = req.CreateResponse(HttpStatusCode.BadRequest);
        missingHeaderResponse.Content = new StringContent(
            new JObject(new JProperty("error", "missing 'x-ms-customproviders-requestpath' header")).ToString(),
            System.Text.Encoding.UTF8, 
            "application/json");
    }

    log.LogInformation($"The Custom Provider Function received a request '{req.Method}' for resource '{requestPath}'.");

    // Determines if it is a collection level call or action.
    var isResourceRequest = requestPath.Split('/').Length % 2 == 1;
    var azureResourceId = isResourceRequest ? 
        ResourceId.FromString(requestPath) :
        ResourceId.FromString($"{requestPath}/");

    // Create the Partition Key and Row Key
    var partitionKey = $"{azureResourceId.SubscriptionId}:{azureResourceId.ResourceGroupName}:{azureResourceId.Parent.Name}";
    var rowKey = $"{azureResourceId.FullResourceType.Replace('/', ':')}:{azureResourceId.Name}";

    switch (req.Method)
    {
        // Action request for a custom action.
        case HttpMethod m when m == HttpMethod.Post && !isResourceRequest:
            return await TriggerCustomAction(
                requestMessage: req);

        // Enumerate request for all custom resources.
        case HttpMethod m when m == HttpMethod.Get && !isResourceRequest:
            return await EnumerateAllCustomResources(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                resourceType: rowKey);

        // Retrieve request for a custom resource.
        case HttpMethod m when m == HttpMethod.Get && isResourceRequest:
            return await RetrieveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Create request for a custom resource.
        case HttpMethod m when m == HttpMethod.Put && isResourceRequest:
            return await CreateCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                azureResourceId: azureResourceId,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Remove request for a custom resource.
        case HttpMethod m when m == HttpMethod.Delete && isResourceRequest:
            return await RemoveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Invalid request received.
        default:
            return req.CreateResponse(HttpStatusCode.BadRequest);
    }
}
```

La méthode **Run** mise à jour comprend désormais la liaison d’entrée *tableStorage* que vous avez ajoutée pour le stockage Table Azure. La première partie de la méthode lit l’en-tête `x-ms-customproviders-requestpath` et utilise la bibliothèque `Microsoft.Azure.Management.ResourceManager.Fluent` pour analyser la valeur comme un ID de ressource. L’en-tête `x-ms-customproviders-requestpath` est envoyé par le fournisseur personnalisé et spécifie le chemin de la requête entrante.

À l’aide de l’ID de ressource analysé, vous pouvez générer les valeurs **partitionKey** et **rowKey** pour les données à rechercher ou pour le stockage des ressources personnalisées.

Après avoir ajouté les méthodes et les classes, vous devez mettre à jour les méthodes **using** pour l’application de fonction. Ajoutez le code suivant au début du fichier C# :

```csharp
#r "Newtonsoft.Json"
#r "Microsoft.WindowsAzure.Storage"
#r "../bin/Microsoft.Azure.Management.ResourceManager.Fluent.dll"

using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Configuration;
using System.Text;
using System.Threading;
using System.Globalization;
using System.Collections.Generic;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.WindowsAzure.Storage.Table;
using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

Si vous avez perdu le fil à un moment ou à un autre de ce tutoriel, vous trouverez l’exemple de code complet dans la [référence sur le point de terminaison RESTful C# pour les fournisseurs personnalisés](./reference-custom-providers-csharp-endpoint.md). Une fois que vous avez terminé l’application de fonction, enregistrez son URL. Celle-ci pourra être utilisée pour déclencher l’application de fonction dans les prochains tutoriels.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un point de terminaison RESTful qui fonctionne avec le point de terminaison d’un fournisseur personnalisé Azure. Pour apprendre à créer un fournisseur personnalisé, lisez l’article [Tutoriel : Création d’un fournisseur personnalisé](./tutorial-custom-providers-create.md).
