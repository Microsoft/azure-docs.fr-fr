---
title: Déclencheur HTTP Azure Functions
description: Apprenez à appeler une fonction Azure via HTTP.
author: craigshoemaker
ms.topic: reference
ms.date: 02/21/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 6466647056535635b67cd53012d051f11e9b484c
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91323309"
---
# <a name="azure-functions-http-trigger"></a>Déclencheur HTTP Azure Functions

Le déclencheur HTTP vous permet d’appeler une fonction avec une requête HTTP. Vous pouvez utiliser un déclencheur HTTP pour générer des API serverless et répondre aux webhooks.

La valeur de retour par défaut pour une fonction déclenchée par HTTP est :

- `HTTP 204 No Content` avec un corps vide dans Functions 2.x et versions ultérieures
- `HTTP 200 OK` avec un corps vide dans Functions 1.x

Pour modifier la réponse HTTP, configurez une [liaison de sortie](./functions-bindings-http-webhook-output.md).

Pour plus d'informations sur les liaisons HTTP, consultez la [présentation](./functions-bindings-http-webhook.md) et les [informations de référence sur les liaisons de sortie](./functions-bindings-http-webhook-output.md).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui recherche un paramètre `name` dans la chaîne de requête ou dans le corps de la requête HTTP. Notez que la valeur renvoyée est utilisée pour la liaison de sortie, mais qu’aucun attribut de valeur renvoyée n’est requis.

```cs
[FunctionName("HttpTriggerCSharp")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]
    HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. Cette fonction recherche un paramètre `name` dans la chaîne de requête ou dans le corps de la requête HTTP.

Voici le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            "authLevel": "function",
            "name": "req",
            "type": "httpTrigger",
            "direction": "in",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "name": "$return",
            "type": "http",
            "direction": "out"
        }
    ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code de script C# qui lie à `HttpRequest` :

```cs
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string name = req.Query["name"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    name = name ?? data?.name;

    return name != null
        ? (ActionResult)new OkObjectResult($"Hello, {name}")
        : new BadRequestObjectResult("Please pass a name on the query string or in the request body");
}
```

Vous pouvez lier à un objet personnalisé au lieu de `HttpRequest`. Cet objet est créé à partir du corps de la requête et analysé en tant que JSON. De même, un type peut être passé à la liaison de sortie de réponse HTTP, et être retourné en tant que corps de la réponse, avec un code d'état `200`.

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static string Run(Person person, ILogger log)
{   
    return person.Name != null
        ? (ActionResult)new OkObjectResult($"Hello, {person.Name}")
        : new BadRequestObjectResult("Please pass an instance of Person.");
}

public class Person {
     public string Name {get; set;}
}
```

# <a name="java"></a>[Java](#tab/java)

* [Lire un paramètre de la chaîne de requête](#read-parameter-from-the-query-string)
* [Lire le corps d’une demande POST](#read-body-from-a-post-request)
* [Lire un paramètre d’une route](#read-parameter-from-a-route)
* [Lire le corps POJO d’une demande POST](#read-pojo-body-from-a-post-request)

Les exemples suivants illustrent la liaison du déclencheur HTTP.

#### <a name="read-parameter-from-the-query-string"></a>Lire un paramètre de la chaîne de requête

Cet exemple lit un paramètre nommé `id` dans la chaîne de requête et l’utilise pour générer un document JSON retourné au client, avec le type de contenu `application/json`.

```java
@FunctionName("TriggerStringGet")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<String>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("GET parameters are: " + request.getQueryParameters());

    // Get named parameter
    String id = request.getQueryParameters().getOrDefault("id", "");

    // Convert and display
    if (id.isEmpty()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String name = "fake_name";
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-body-from-a-post-request"></a>Lire le corps d’une demande POST

Cet exemple lit le corps d’une demande POST, par exemple une `String`, et l’utilise pour générer un document JSON retourné au client, avec le type de contenu `application/json`.

```java
    @FunctionName("TriggerStringPost")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", 
              methods = {HttpMethod.POST}, 
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        
        // Item list
        context.getLogger().info("Request body is: " + request.getBody().orElse(""));

        // Check request body
        if (!request.getBody().isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        } 
        else {
            // return JSON from to the client
            // Generate document
            final String body = request.getBody().get();
            final String jsonDocument = "{\"id\":\"123456\", " + 
                                         "\"description\": \"" + body + "\"}";
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(jsonDocument)
                          .build();
        }
    }
```

#### <a name="read-parameter-from-a-route"></a>Lire un paramètre d’une route

Cet exemple lit un paramètre obligatoire, nommé `id`, et un paramètre facultatif `name` dans le chemin de route, et les utilise pour générer un document JSON retourné au client, avec le type de contenu `application/json`. T

```java
@FunctionName("TriggerStringRoute")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.GET}, 
            authLevel = AuthorizationLevel.ANONYMOUS,
            route = "trigger/{id}/{name=EMPTY}") // name is optional and defaults to EMPTY
        HttpRequestMessage<Optional<String>> request,
        @BindingName("id") String id,
        @BindingName("name") String name,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Route parameters are: " + id);

    // Convert and display
    if (id == null) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " + 
                                        "\"description\": \"" + name + "\"}";
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(jsonDocument)
                        .build();
    }
}
```

#### <a name="read-pojo-body-from-a-post-request"></a>Lire le corps POJO d’une demande POST

Voici le code de la classe `ToDoItem`, référencée dans cet exemple :

```java

public class ToDoItem {

  private String id;
  private String description;  

  public ToDoItem(String id, String description) {
    this.id = id;
    this.description = description;
  }

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }
  
  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}

```

Cet exemple lit le corps d’une demande POST. Le corps de la demande est automatiquement désérialisé dans un objet `ToDoItem` et est retourné au client, avec le type de contenu `application/json`. Le paramètre `ToDoItem` est sérialisé par le runtime Functions, car il est attribué à la propriété `body` de la classe `HttpMessageResponse.Builder`.

```java
@FunctionName("TriggerPojoPost")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", 
            methods = {HttpMethod.POST}, 
            authLevel = AuthorizationLevel.ANONYMOUS)
        HttpRequestMessage<Optional<ToDoItem>> request,
        final ExecutionContext context) {
    
    // Item list
    context.getLogger().info("Request body is: " + request.getBody().orElse(null));

    // Check request body
    if (!request.getBody().isPresent()) {
        return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                        .body("Document not found.")
                        .build();
    } 
    else {
        // return JSON from to the client
        // Generate document
        final ToDoItem body = request.getBody().get();
        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(body)
                        .build();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. Cette fonction recherche un paramètre `name` dans la chaîne de requête ou dans le corps de la requête HTTP.

Voici le fichier *function.json* :

```json
{
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function(context, req) {
    context.log('Node.js HTTP trigger function processed a request. RequestUri=%s', req.originalUrl);

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction PowerShell](functions-reference-node.md). Cette fonction recherche un paramètre `name` dans la chaîne de requête ou dans le corps de la requête HTTP.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$name = $Request.Query.Name
if (-not $name) {
    $name = $Request.Body.Name
}

$body = "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."

if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}

# Associate values to output bindings by calling 'Push-OutputBinding'.
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body       = $body
})
```


# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre une liaison de déclencheur dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui utilise la liaison. Cette fonction recherche un paramètre `name` dans la chaîne de requête ou dans le corps de la requête HTTP.

Voici le fichier *function.json* :

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,    
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req"
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Python :

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md) et Java, l’attribut `HttpTrigger` est disponible pour configurer la fonction.

Vous pouvez définir le niveau d’autorisation et des méthodes HTTP autorisées dans les paramètres de constructeur d’attribut, le type webhook et un modèle de routage. Pour plus d'informations sur ces paramètres, consultez [configuration](#configuration).

# <a name="c"></a>[C#](#tab/csharp)

Cet exemple montre comment utiliser l’attribut [HttpTrigger](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions.Http/HttpTriggerAttribute.cs).

```csharp
[FunctionName("HttpTriggerCSharp")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequest req)
{
    ...
}
```

Pour obtenir un exemple complet, consultez l’[exemple de déclencheur](#example).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="java"></a>[Java](#tab/java)

Cet exemple montre comment utiliser l’attribut [HttpTrigger](https://github.com/Azure/azure-functions-java-library/blob/dev/src/main/java/com/microsoft/azure/functions/annotation/HttpTrigger.java).

```java
@FunctionName("HttpTriggerJava")
public HttpResponseMessage<String> HttpTrigger(
        @HttpTrigger(name = "req",
                     methods = {"get"},
                     authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<String> request,
        final ExecutionContext context) {

    ...
}
```

Pour obtenir un exemple complet, consultez l’[exemple de déclencheur](#example).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les attributs ne sont pas pris en charge par PowerShell.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `HttpTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
| **type** | n/a| Obligatoire : doit être défini sur `httpTrigger`. |
| **direction** | n/a| Obligatoire : doit être défini sur `in`. |
| **name** | n/a| Obligatoire : nom de variable utilisé dans le code de la fonction pour la requête ou le corps de la requête. |
| <a name="http-auth"></a>**authLevel** |  **AuthLevel** |Détermine, le cas échéant, les clés qui doivent être présentes dans la requête pour appeler la fonction. Le niveau d’autorisation peut être l’une des valeurs suivantes : <ul><li><code>anonymous</code>&mdash;Aucune clé API n’est obligatoire.</li><li><code>function</code>&mdash;Une clé API spécifique à la fonction est obligatoire. Il s’agit de la valeur par défaut si aucune valeur n’est fournie.</li><li><code>admin</code>&mdash;La clé principale est obligatoire.</li></ul> Pour plus d’informations, consultez la section sur les [clés d’autorisation](#authorization-keys). |
| **methods** |**Méthodes** | Tableau des méthodes HTTP auxquelles la fonction répond. À défaut de spécification, la fonction répond à toutes les méthodes HTTP. Consultez [Personnaliser le point de terminaison HTTP](#customize-the-http-endpoint). |
| **route** | **Itinéraire** | Définit le modèle de routage, en contrôlant les URL de requête auxquelles votre fonction répond. La valeur par défaut est `<functionname>`. Pour plus d’informations, consultez [Personnaliser le point de terminaison HTTP](#customize-the-http-endpoint). |
| **webHookType** | **WebHookType** | _Prise en charge uniquement pour le runtime version 1.x._<br/><br/>Configure le déclencheur HTTP pour qu’il agisse en tant que récepteur de [Webhook](https://en.wikipedia.org/wiki/Webhook) pour le fournisseur spécifié. Ne définissez pas la propriété `methods` si vous définissez cette propriété. Le type de webhook peut être l’une des valeurs suivantes :<ul><li><code>genericJson</code>&mdash;Point de terminaison webhook à usage général sans logique pour un fournisseur spécifique. Ce paramètre limite les requêtes à celles utilisant HTTP POST et le type de contenu `application/json`.</li><li><code>github</code>&mdash;La fonction répond aux [Webhooks GitHub](https://developer.github.com/webhooks/). N’utilisez pas la propriété _authLevel_ avec des webhooks GitHub. Pour plus d’informations, consultez la section sur les webhooks GitHub plus loin dans cet article.</li><li><code>slack</code>&mdash;La fonction répond aux [Webhooks Slack](https://api.slack.com/outgoing-webhooks). N’utilisez pas la propriété _authLevel_ avec des webhooks Slack. Pour plus d’informations, consultez la section sur les webhooks Slack plus loin dans cet article.</li></ul>|

## <a name="payload"></a>Payload

Le type d’entrée du déclencheur est déclaré comme étant `HttpRequest` ou un type personnalisé. Si vous choisissez `HttpRequest`, vous obtenez un accès complet à l’objet de la requête. Pour un type personnalisé, le runtime tente d’analyser le corps de la requête JSON pour définir les propriétés de l’objet.

## <a name="customize-the-http-endpoint"></a>Personnaliser le point de terminaison HTTP

Par défaut, quand vous créez une fonction pour un déclencheur HTTP, la fonction est adressable avec une route de la forme :

```http
http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>
```

Vous pouvez personnaliser cet itinéraire en utilisant la propriété facultative `route` sur la liaison d’entrée du déclencheur HTTP. Par exemple, le fichier *function.json* suivant définit une propriété `route` pour un déclencheur HTTP :

```json
{
    "bindings": [
    {
        "type": "httpTrigger",
        "name": "req",
        "direction": "in",
        "methods": [ "get" ],
        "route": "products/{category:alpha}/{id:int?}"
    },
    {
        "type": "http",
        "name": "res",
        "direction": "out"
    }
    ]
}
```

Avec cette configuration, la fonction est désormais adressable avec l’itinéraire suivant au lieu de l’itinéraire d’origine.

```
http://<APP_NAME>.azurewebsites.net/api/products/electronics/357
```

Cette configuration permet au code de la fonction de prendre en charge deux paramètres dans l'adresse, _category_ et _id_.

# <a name="c"></a>[C#](#tab/csharp)

Vous pouvez utiliser les [contraintes d’itinéraire des API Web](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints) de votre choix avec vos paramètres. Le code de la fonction C# suivant utilise les deux paramètres.

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Vous pouvez utiliser les [contraintes d’itinéraire des API Web](https://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#constraints) de votre choix avec vos paramètres. Le code de la fonction C# suivant utilise les deux paramètres.

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;

public static IActionResult Run(HttpRequest req, string category, int? id, ILogger log)
{
    var message = String.Format($"Category: {category}, ID: {id}");
    return (ActionResult)new OkObjectResult(message);
}
```

# <a name="java"></a>[Java](#tab/java)

Le contexte d’exécution de la fonction correspond à des propriétés déclarées dans l’attribut `HttpTrigger`. L’attribut vous permet de définir des paramètres d’acheminement, des niveaux d’autorisation, des verbes HTTP et l’instance de requête entrante.

Les paramètres d’acheminement sont définis par l’intermédiaire de l’attribut `HttpTrigger`.

```java
package com.function;

import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class HttpTriggerJava {
    public HttpResponseMessage<String> HttpTrigger(
            @HttpTrigger(name = "req",
                         methods = {"get"},
                         authLevel = AuthorizationLevel.FUNCTION,
                         route = "products/{category:alpha}/{id:int}") HttpRequestMessage<String> request,
            @BindingName("category") String category,
            @BindingName("id") int id,
            final ExecutionContext context) {

        String message = String.format("Category  %s, ID: %d", category, id);
        return request.createResponseBuilder(HttpStatus.OK).body(message).build();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Dans Node, le runtime Functions fournit le corps de la demande à partir de l’objet `context`. Pour plus d’informations, consultez l’[exemple de déclencheur JavaScript](#example).

L’exemple suivant vous indique comment lire les paramètres de routage à partir de `context.bindingData`.

```javascript
module.exports = function (context, req) {

    var category = context.bindingData.category;
    var id = context.bindingData.id;
    var message = `Category: ${category}, ID: ${id}`;

    context.res = {
        body: message;
    }

    context.done();
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les paramètres de routage déclarés dans le fichier *function.json* sont accessibles en tant que propriété de l’objet `$Request.Params`.

```powershell
$Category = $Request.Params.category
$Id = $Request.Params.id

$Message = "Category:" + $Category + ", ID: " + $Id

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body = $Message
})
```

# <a name="python"></a>[Python](#tab/python)

Le contexte d’exécution de la fonction est exposé par le biais d’un paramètre déclaré comme en tant que `func.HttpRequest`. Cette instance permet à une fonction d’accéder à des paramètres d’acheminement des données, des valeurs de chaîne de requête et des méthodes qui vous permettent de retourner des réponses HTTP.

Une fois définis, les paramètres d’acheminement sont disponibles pour la fonction en appelant la méthode `route_params`.

```python
import logging

import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    category = req.route_params.get('category')
    id = req.route_params.get('id')
    message = f"Category: {category}, ID: {id}"

    return func.HttpResponse(message)
```

---

Par défaut, tous les itinéraires de fonction sont préfixés par *api*. Vous pouvez également personnaliser ou supprimer le préfixe avec la propriété `extensions.http.routePrefix` dans votre fichier [host.json](functions-host-json.md). L’exemple suivant supprime le préfixe d’itinéraire *api* en sélectionnant une chaîne vide pour le préfixe dans le fichier *host.json*.

```json
{
    "extensions": {
        "http": {
            "routePrefix": ""
        }
    }
}
```

## <a name="using-route-parameters"></a>Utilisation des paramètres de routage

Les paramètres de routage définissant le modèle `route` d’une fonction sont disponibles pour chaque liaison. Par exemple, si vous avez un itinéraire défini en tant que `"route": "products/{id}"`, une liaison de stockage de table peut utiliser la valeur du paramètre `{id}` dans la configuration de liaison.

La configuration suivante montre comment le paramètre `{id}` est passé au `rowKey` de la liaison.

```json
{
    "type": "table",
    "direction": "in",
    "name": "product",
    "partitionKey": "products",
    "tableName": "products",
    "rowKey": "{id}"
}
```

## <a name="working-with-client-identities"></a>Utilisation d’identités de clients

S votre application de fonction utilise [Authentification d’App Service/Autorisation](../app-service/overview-authentication-authorization.md), vous pouvez afficher des informations sur les clients authentifiés à partir de votre code. Ces informations sont disponibles en tant qu’[en-têtes de demande injectées par la plateforme](../app-service/app-service-authentication-how-to.md#access-user-claims).

Vous pouvez également lire ces informations à partir de la liaison de données. Cette fonctionnalité est uniquement disponible pour le runtime Functions dans les versions 2.x et ultérieures. Elle n’est actuellement également disponible que pour les langages .NET.

# <a name="c"></a>[C#](#tab/csharp)

Les informations relatives aux clients authentifiés sont disponibles en tant que [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). Le ClaimsPrincipal est disponible dans le cadre du contexte de requête, comme le montre l’exemple suivant :

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Le ClaimsPrincipal peut aussi simplement être inclus comme paramètre supplémentaire dans la signature de fonction :

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les informations relatives aux clients authentifiés sont disponibles en tant que [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal). Le ClaimsPrincipal est disponible dans le cadre du contexte de requête, comme le montre l’exemple suivant :

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public static IActionResult Run(HttpRequest req, ILogger log)
{
    ClaimsPrincipal identities = req.HttpContext.User;
    // ...
    return new OkObjectResult();
}
```

Le ClaimsPrincipal peut aussi simplement être inclus comme paramètre supplémentaire dans la signature de fonction :

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Newtonsoft.Json.Linq;

public static void Run(JObject input, ClaimsPrincipal principal, ILogger log)
{
    // ...
    return;
}
```

# <a name="java"></a>[Java](#tab/java)

L’utilisateur authentifié est disponible par l’intermédiaire d’[en-têtes HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’utilisateur authentifié est disponible par l’intermédiaire d’[en-têtes HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L’utilisateur authentifié est disponible par l’intermédiaire d’[en-têtes HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).

# <a name="python"></a>[Python](#tab/python)

L’utilisateur authentifié est disponible par l’intermédiaire d’[en-têtes HTTP](../app-service/app-service-authentication-how-to.md#access-user-claims).


---

## <a name="function-access-keys"></a><a name="authorization-keys"></a>Clé d’accès aux fonctions

[!INCLUDE [functions-authorization-keys](../../includes/functions-authorization-keys.md)]

## <a name="obtaining-keys"></a>Obtention de clés

Les clés sont stockées dans votre Function App dans Azure, et chiffrées au repos. Pour voir vos clés, en créer de nouvelles ou affecter de nouvelles valeurs aux clés, accédez à une de vos fonctions déclenchées par HTTP dans le [portail Azure](https://portal.azure.com), puis sélectionnez **Gérer**.

![Gérez les clés de fonction dans le portail.](./media/functions-bindings-http-webhook/manage-function-keys.png)

Vous pouvez obtenir des clés de fonction programmatiquement en utilisant des [API Gestion des clés](https://github.com/Azure/azure-functions-host/wiki/Key-management-API).

## <a name="api-key-authorization"></a>Autorisation de clé API

La plupart des modèles de déclencheur HTTP nécessitent une clé d’API dans la requête. Ainsi, votre requête HTTP doit se présenter comme l’URL suivante :

```http
https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>
```

La clé peut être incluse dans une variable de chaîne de requête nommée `code`, comme ci-dessus. Elle peut également être incluse dans un en-tête HTTP `x-functions-key`. La valeur de la clé peut être toute clé de fonction définie pour la fonction, ou toute clé d’hôte.

Vous pouvez autoriser les requêtes anonymes, qui ne nécessitent pas de clés. Vous pouvez également exiger que la clé principale soit utilisée. Pour modifier le niveau d’autorisation par défaut, utilisez la propriété `authLevel` dans le JSON de liaison. Pour plus d’informations, consultez [Déclencheur - configuration](#configuration).

> [!NOTE]
> Lors de l’exécution de fonctions localement, l’autorisation est désactivée, quel que soit le paramètre de niveau d’autorisation spécifié. Après la publication sur Azure, le paramètre `authLevel` de votre déclencheur est appliqué. Les clés sont quand même exigées lors d’une exécution [locale dans un conteneur](functions-create-function-linux-custom-image.md#build-the-container-image-and-test-locally).


## <a name="secure-an-http-endpoint-in-production"></a>Sécuriser un point de terminaison HTTP en production

Pour sécuriser complètement vos points de terminaison de fonction en production, vous devez envisager d’implémenter une des options suivantes de sécurité au niveau de l’application de fonction. Quand vous utilisez l’une de ces méthodes de sécurité au niveau de l’application de fonction, vous devez définir le niveau d’autorisation de la fonction déclenchée par HTTP sur `anonymous`.

[!INCLUDE [functions-enable-auth](../../includes/functions-enable-auth.md)]

#### <a name="deploy-your-function-app-in-isolation"></a>Déployer votre application de fonction en isolation

[!INCLUDE [functions-deploy-isolation](../../includes/functions-deploy-isolation.md)]

## <a name="webhooks"></a>webhooks

> [!NOTE]
> Le mode Webhook est disponible seulement pour la version 1.x du runtime Functions. Cette modification a été apportée afin d’améliorer les performances des déclencheurs HTTP dans les versions 2.x et ultérieures.

Dans la version 1.x, les modèles de webhook offrent une validation supplémentaire pour les charges utiles de webhook. Dans les versions 2.x et ultérieures, le déclencheur HTTP de base continue néanmoins de fonctionner et constitue l’approche recommandée pour les webhooks. 

### <a name="github-webhooks"></a>Webhooks GitHub

Pour répondre aux webhooks GitHub, commencez par créer votre fonction avec un déclencheur HTTP, puis définissez la propriété **webHookType** sur `github`. Ensuite, copiez son URL et sa clé API dans la page **Ajouter un Webhook** de votre dépôt GitHub. 

![Capture d’écran montrant comment ajouter un webhook pour votre fonction.](./media/functions-bindings-http-webhook/github-add-webhook.png)

### <a name="slack-webhooks"></a>Webhooks Slack

La webhook de Slack génère un jeton à votre place au lieu de vous laisser le spécifier. Vous devez donc configurer une clé spécifique de la fonction avec le jeton reçu de Slack. Consultez [Clés d’autorisation](#authorization-keys).

## <a name="webhooks-and-keys"></a>Webhooks et clés

Une autorisation de webhook est gérée par le composant récepteur de webhook, qui fait partie du déclencheur HTTP, et le mécanisme varie en fonction du type de webhook. Chaque mécanisme s’appuie sur une clé. Par défaut, la clé de fonction nommée « default » est utilisée. Pour utiliser une autre clé, configurez le fournisseur de webhook pour envoyer le nom de clé avec la requête de l’une des manières suivantes :

* **Chaîne de requête** : Le fournisseur transmet le nom de la clé au paramètre de chaîne de requête `clientid` (par exemple, `https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?clientid=<KEY_NAME>`).
* **En-tête de requête** : Le fournisseur transmet le nom de la clé à l’en-tête `x-functions-clientid`.

## <a name="content-types"></a>Types de contenu

La transmission de données binaires et de formulaire à une fonction non C# requiert l’utilisation de l’en-tête de contenu-type approprié. Les types de contenu pris en charge incluent `octet-stream` pour les données binaires et les types [en plusieurs parties](https://www.iana.org/assignments/media-types/media-types.xhtml#multipart).

### <a name="known-issues"></a>Problèmes connus

Dans les fonctions non C#, les requêtes envoyées avec le type de contenu `image/jpeg` entraînent la transmission d’une valeur `string` à la fonction. Dans ce type de cas, vous pouvez convertir manuellement la valeur `string` en un tableau d’octets pour accéder aux données binaires brutes.

## <a name="limits"></a>limites

La longueur de la requête HTTP est limitée à 100 Mo (104 857 600 octets) et la longueur de l’URL à 4 Ko (4 096 octets). Ces limites sont spécifiées par l’élément `httpRuntime` du [fichier Web.config](https://github.com/Azure/azure-functions-host/blob/v3.x/src/WebJobs.Script.WebHost/web.config) du runtime.

Si une fonction utilisant le déclencheur HTTP ne se termine pas au bout de 230 secondes, [Azure Load Balancer](../app-service/faq-availability-performance-application-issues.md#why-does-my-request-time-out-after-230-seconds) arrive à expiration et retourne une erreur HTTP 502. La fonction continuera à s’exécuter, mais ne pourra pas renvoyer de réponse HTTP. Pour les fonctions à exécution longues, nous vous recommandons de suivre des modèles asynchrones et de retourner un emplacement où vous pouvez effectuer un test ping de l’état de la requête. Pour plus d’informations sur la durée d’exécution d’une fonction, consultez [Scale and hosting - Consumption plan](functions-scale.md#timeout) (Mise à l’échelle et hébergement – Plan de consommation).


## <a name="next-steps"></a>Étapes suivantes

- [Retourner une réponse HTTP à partir d'une fonction](./functions-bindings-http-webhook-output.md)
