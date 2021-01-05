---
title: Fonctionnalités HTTP dans Durable Functions - Azure Functions
description: Découvrez les fonctionnalités HTTP intégrées dans l’extension Durable Functions pour Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: azfuncdf
ms.openlocfilehash: 64d40de50f21811a56318971de1836abc8fbf8c9
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027259"
---
# <a name="http-features"></a>Fonctionnalités HTTP

Durable Functions propose plusieurs fonctionnalités pour faciliter l’incorporation d’orchestrations et d’entités durables dans les workflows HTTP. Cet article décrit en détail certaines de ces fonctionnalités.

## <a name="exposing-http-apis"></a>Exposition des API HTTP

Les orchestrations et les entités peuvent être appelées et gérées à l’aide de requêtes HTTP. L’extension Durable Functions expose les API HTTP intégrées. Elle fournit également des API pour interagir avec les orchestrations et les entités au sein des fonctions déclenchées via HTTP.

### <a name="built-in-http-apis"></a>API HTTP intégrées

L’extension Durable Functions ajoute automatiquement un ensemble d’API HTTP à l’hôte Azure Functions. Ces API vous permettent d’interagir avec les orchestrations et les entités, ainsi que de les gérer, sans écrire de code.

Les API HTTP intégrées suivantes sont prises en charge.

* [Démarrer une nouvelle orchestration](durable-functions-http-api.md#start-orchestration)
* [Interroger l’instance d’orchestration](durable-functions-http-api.md#get-instance-status)
* [Mettre fin à l’instance d’orchestration](durable-functions-http-api.md#terminate-instance)
* [Envoyer un événement externe à une orchestration](durable-functions-http-api.md#raise-event)
* [Vider l’historique d’orchestration](durable-functions-http-api.md#purge-single-instance-history)
* [Envoyer un événement d’opération à une entité](durable-functions-http-api.md#signal-entity)
* [Obtenir l’état d’une entité](durable-functions-http-api.md#get-entity)
* [Interroger la liste des entités](durable-functions-http-api.md#list-entities)

Consultez l’[article sur les API HTTP](durable-functions-http-api.md) pour obtenir une description complète de toutes les API HTTP intégrées exposées par l’extension Durable Functions.

### <a name="http-api-url-discovery"></a>Découverte de l’URL de l’API HTTP

La [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) expose les API qui peuvent générer des charges utiles de réponse HTTP appropriées. Par exemple, elle peut créer une réponse contenant des liens vers des API de gestion pour une instance d’orchestration spécifique. Les exemples suivants présentent une fonction de déclencheur HTTP qui montre comment utiliser cette API pour une nouvelle instance d’orchestration :

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpStart/index.js)]

**function.json**

[!code-json[Main](~/samples-durable-functions/samples/javascript/HttpStart/function.json)]

# <a name="python"></a>[Python](#tab/python)

**__init__.py**

```python
import logging
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    function_name = req.route_params['functionName']
    event_data = req.get_body()

    instance_id = await client.start_new(function_name, instance_id, event_data)
    
    logging.info(f"Started orchestration with ID = '{instance_id}'.")
    return client.create_check_status_response(req, instance_id)
```

**function.json**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "route": "orchestrators/{functionName}",
      "methods": [
        "post",
        "get"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "orchestrationClient",
      "direction": "in"
    }
  ]
}
```

---

Le démarrage d’une fonction d’orchestrateur à l’aide des fonctions de déclencheur HTTP présentées précédemment peut être effectué à l’aide d’un client HTTP quelconque. La commande curl suivante démarre une fonction d’orchestrateur nommée `DoWork` :

```bash
curl -X POST https://localhost:7071/orchestrators/DoWork -H "Content-Length: 0" -i
```

Voici un exemple de réponse pour une orchestration dont l’ID est `abc123`. Certains détails ont été supprimés pour plus de clarté.

```http
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX
Retry-After: 10

{
    "id": "abc123",
    "purgeHistoryDeleteUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/raiseEvent/{eventName}?code=XXX",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/terminate?reason={text}&code=XXX"
}
```

Dans l’exemple précédent, chacun des champs se terminant par `Uri` correspond à une API HTTP intégrée. Vous pouvez utiliser ces API pour gérer l’instance d’orchestration cible.

> [!NOTE]
> Le format des URL de webhook dépend de la version de l’hôte Azure Functions que vous exécutez. L’exemple précédent a trait à l’hôte Azure Functions 2.0.

Pour obtenir une description de toutes les API HTTP intégrées, consultez la [Référence sur l’API HTTP](durable-functions-http-api.md).

### <a name="async-operation-tracking"></a>Suivi de l’opération asynchrone

La réponse HTTP mentionnée précédemment est conçue pour faciliter l’implémentation d’API asynchrones HTTP longues avec Durable Functions. Ce modèle est parfois appelé *modèle d’interrogation du consommateur*. Le flux client/serveur fonctionne comme suit :

1. Le client émet une requête HTTP pour démarrer un processus à long terme, tel qu’une fonction d’orchestrateur.
1. Le déclencheur HTTP cible retourne une réponse HTTP 202 avec un en-tête Location dont la valeur est « statusQueryGetUri ».
1. Le client interroge l’URL figurant dans l’en-tête Location. Le client continue à voir des réponses HTTP 202 avec un en-tête Location.
1. Lorsque l’exécution de l’instance se termine ou échoue, le point de terminaison dans l’en-tête Location retourne HTTP 200.

Ce protocole permet de coordonner les processus de longue durée avec des clients ou services externes qui peuvent interroger un point de terminaison HTTP et suivre l’en-tête Location. Les implémentations cliente et serveur de ce modèle sont intégrées dans les API HTTP de Durable Functions.

> [!NOTE]
> Par défaut, toutes les actions basées sur HTTP fournies par [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) prennent en charge le modèle d’opération asynchrone standard. Cette fonctionnalité permet d’incorporer une fonction durable de longue durée dans un flux de travail Logic Apps. Pour plus d’informations sur la prise en charge des modèles HTTP asynchrones par Logic Apps, voir la [documentation sur les actions de flux de travail et les déclencheurs Azure Logic Apps](../../logic-apps/logic-apps-workflow-actions-triggers.md).

> [!NOTE]
> Il est possible d’interagir avec des orchestrations à partir de n’importe quel type de fonction, pas seulement des fonctions déclenchées par HTTP.

Pour plus d’informations sur la façon de gérer les orchestrations et les entités à l’aide d’API clientes, voir l’article [Gestion d’instance](durable-functions-instance-management.md).

## <a name="consuming-http-apis"></a>Utilisation des API HTTP

Comme décrit dans [Contraintes de code des fonctions d’orchestrateur](durable-functions-code-constraints.md), les fonctions d’orchestrateur ne peuvent pas effectuer d’E/S directement. Au lieu de cela, elles appellent généralement des [fonctions d’activité](durable-functions-types-features-overview.md#activity-functions) qui effectuent les opérations d’E/S.

Depuis Durable Functions 2.0, les orchestrations sont en mesure de consommer en mode natif les API HTTP à l’aide d’une [liaison de déclencheur d’orchestration](durable-functions-bindings.md#orchestration-trigger).

L’exemple de code suivant montre une fonction d’orchestrateur C# qui exécute une requête HTTP sortante :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("CheckSiteAvailable")]
public static async Task CheckSiteAvailable(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    Uri url = context.GetInput<Uri>();

    // Makes an HTTP GET request to the specified endpoint
    DurableHttpResponse response = 
        await context.CallHttpAsync(HttpMethod.Get, url);

    if (response.StatusCode >= 400)
    {
        // handling of error codes goes here
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context){
    const url = context.df.getInput();
    const response = yield context.df.callHttp("GET", url)

    if (response.statusCode >= 400) {
        // handling of error codes goes here
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    url = context.get_input()
    response = yield context.call_http('GET', url)
    
    if response["statusCode"] >= 400:
        # handling of error codes goes here

main = df.Orchestrator.create(orchestrator_function)
```

---

L’action « call HTTP » vous permet d’effectuer les actions suivantes dans vos fonctions d’orchestrateur :

* Appeler les API HTTP directement à partir des fonctions d’orchestration, avec certaines limitations mentionnées ci-après.
* Prendre en charge automatiquement les modèles d’interrogation d’état HTTP 202 côté client.
* Utiliser les [identités managées Azure](../../active-directory/managed-identities-azure-resources/overview.md) pour effectuer des appels HTTP autorisés à d’autres points de terminaison Azure.

La capacité à consommer les API HTTP directement à partir des fonctions d’orchestrateur est conçue à titre de commodité pour un certain ensemble de scénarios courants. Vous pouvez implémenter toutes ces fonctionnalités vous-même à l’aide de fonctions d’activité. Dans de nombreux cas, les fonctions d’activité peuvent vous offrir plus de souplesse.

### <a name="http-202-handling"></a>Gestion des cas HTTP 202

L’API « call HTTP » peut implémenter automatiquement le côté client du modèle d’interrogation de consommateur. Si une API appelée renvoie une réponse HTTP 202 avec un en-tête Location, la fonction d’orchestrateur interroge automatiquement la ressource Location jusqu’à recevoir une réponse autre que 202. Cette réponse est celle renvoyée au code de la fonction d’orchestrateur.

> [!NOTE]
> Les fonctions d’orchestrateur prennent également en charge en mode natif le modèle d’interrogation de consommateur côté serveur, comme décrit dans [Suivi de l’opération asynchrone](#async-operation-tracking). Cette prise en charge signifie que les orchestrations au sein d’une application de fonction peuvent facilement coordonner les fonctions d’orchestrateur dans d’autres applications de fonction. Cela est similaire au concept de [sous-orchestration](durable-functions-sub-orchestrations.md), mais avec la prise en charge des communications entre applications. Cette prise en charge s’avère particulièrement utile pour le développement d’applications de type microservice.

### <a name="managed-identities"></a>Identités managées

Durable Functions prend en charge en mode natif l’appel d’API qui acceptent des jetons Azure Active Directory pour l’autorisation. Cette prise en charge utilise des [identités managées Azure](../../active-directory/managed-identities-azure-resources/overview.md) pour acquérir ces jetons.

Le code suivant est un exemple de fonction d’orchestrateur .NET. La fonction qui effectue des appels authentifiés pour redémarrer une machine virtuelle à l’aide de l’[API REST Machines virtuelles](/rest/api/compute/virtualmachines) d’Azure Resource Manager.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RestartVm")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string subscriptionId = "mySubId";
    string resourceGroup = "myRG";
    string vmName = "myVM";
    string apiVersion = "2019-03-01";
    
    // Automatically fetches an Azure AD token for resource = https://management.core.windows.net/.default
    // and attaches it to the outgoing Azure Resource Manager API call.
    var restartRequest = new DurableHttpRequest(
        HttpMethod.Post, 
        new Uri($"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/virtualMachines/{vmName}/restart?api-version={apiVersion}"),
        tokenSource: new ManagedIdentityTokenSource("https://management.core.windows.net/.default"));
    DurableHttpResponse restartResponse = await context.CallHttpAsync(restartRequest);
    if (restartResponse.StatusCode != HttpStatusCode.OK)
    {
        throw new ArgumentException($"Failed to restart VM: {restartResponse.StatusCode}: {restartResponse.Content}");
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const subscriptionId = "mySubId";
    const resourceGroup = "myRG";
    const vmName = "myVM";
    const apiVersion = "2019-03-01";
    const tokenSource = new df.ManagedIdentityTokenSource("https://management.core.windows.net/.default");

    // get a list of the Azure subscriptions that I have access to
    const restartResponse = yield context.df.callHttp(
        "POST",
        `https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.Compute/virtualMachines/${vmName}/restart?api-version=${apiVersion}`,
        undefined, // no request content
        undefined, // no request headers (besides auth which is handled by the token source)
        tokenSource);

    return restartResponse;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    subscription_id = "mySubId"
    resource_group = "myRg"
    vm_name = "myVM"
    api_version = "2019-03-01"
    token_source = df.ManagedIdentityTokenSource("https://management.core.windows.net/.default")

    # get a list of the Azure subscriptions that I have access to
    restart_response = yield context.call_http("POST", 
        f"https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}/restart?api-version={api_version}",
        None,
        None,
        token_source)
    return restart_response

main = df.Orchestrator.create(orchestrator_function)
```

---

Dans l’exemple précédent, le paramètre `tokenSource` est configuré pour acquérir des jetons Azure AD pour [Azure Resource Manager](../../azure-resource-manager/management/overview.md). Les jetons sont identifiés par l’URI de ressource `https://management.core.windows.net/.default`. Cet exemple part du principe que l’application de fonction actuelle s’exécute localement ou a été déployée en tant qu’application de fonction avec une identité managée. L’identité locale ou l’identité managée est supposée avoir l’autorisation de gérer des machines virtuelles dans le groupe de ressources spécifié `myRG`.

Lors de l’exécution, la source de jeton configurée retourne automatiquement un jeton d’accès OAuth 2.0. La source ajoute ensuite le jeton en tant que jeton du porteur à l’en-tête Authorization de la demande sortante. Ce modèle constitue une amélioration par rapport à l’ajout manuel d’en-têtes d’autorisation aux requêtes HTTP pour les raisons suivantes :

* L’actualisation des jetons est gérée automatiquement. Vous n’avez pas à vous soucier des jetons expirés.
* Les jetons ne sont jamais stockés dans l’état d’orchestration durable.
* Vous n’êtes pas obligé d’écrire du code pour gérer l’acquisition des jetons.

Vous trouverez un exemple plus complet dans l’[exemple « RestartVMs » C# précompilé](https://github.com/Azure/azure-functions-durable-extension/blob/dev/samples/precompiled/RestartVMs.cs).

Les identités managées ne sont pas limitées à la gestion des ressources Azure. Vous pouvez utiliser des identités managées pour accéder à n’importe quelle API acceptant des jetons du porteur Azure AD, notamment à des services Azure de Microsoft et à des applications web de partenaires. Une application web d’un partenaire peut même être une autre application de fonction. Pour obtenir la liste des services Azure de Microsoft qui prennent en charge l’authentification avec Azure AD, voir [Services Azure qui prennent en charge l’authentification Azure AD](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

### <a name="limitations"></a>Limites

La prise en charge intégrée de l’appel des API HTTP est une fonctionnalité de commodité. Elle ne convient pas pour tous les scénarios.

Les requêtes HTTP envoyées par les fonctions d’orchestrateur et leurs réponses sont sérialisées et conservées en tant que messages en file d’attente. Ce comportement de mise en file d’attente garantit [la fiabilité et la sécurité des appels HTTP pour la relecture d’orchestration](durable-functions-orchestrations.md#reliability). Toutefois, le comportement de mise en file d’attente présente également des limitations :

* Chaque requête HTTP implique une latence supplémentaire par rapport à un client HTTP natif.
* Les messages de demande ou de réponse volumineux qui ne peuvent pas tenir dans un message de file d’attente peuvent nuire considérablement aux performances de l’orchestration. La surcharge liée au déchargement des charges utiles des messages peut entraîner une détérioration des performances.
* Les charges utiles de streaming, segmentées et binaires ne sont pas prises en charge.
* La possibilité de personnaliser le comportement du client HTTP est limitée.

Si l’une de ces limitations risque d’avoir un impact sur votre cas d’utilisation, envisagez plutôt d’utiliser des fonctions d’activité et des bibliothèques clientes HTTP spécifiques d’un langage pour effectuer des appels HTTP sortants.

> [!NOTE]
> Si vous êtes un développeur .NET, vous vous demandez peut-être pourquoi cette fonctionnalité utilise les types **DurableHttpRequest** et **DurableHttpResponse** au lieu des types **HttpRequestMessage** et **HttpResponseMessage**.
>
> Ce choix de conception est intentionnel. La principale raison est que les types personnalisés permettent de s’assurer que les utilisateurs ne font pas de suppositions incorrectes concernant les comportements pris en charge du client HTTP interne. Les types spécifiques de Durable Functions permettent également de simplifier la conception des API. Ils peuvent également rendre plus facilement disponibles des fonctionnalités spéciales telles que l’[intégration d’identité managée](#managed-identities) et le [modèle d’interrogation de consommateur](#http-202-handling). 

### <a name="extensibility-net-only"></a>Extensibilité (.NET uniquement)

La personnalisation du comportement du client HTTP interne de l’orchestration est possible par [injection de dépendances .NET Azure Functions](../functions-dotnet-dependency-injection.md). Cette possibilité peut s’avérer utile pour effectuer des changements de comportement minimes. Elle peut également s’avérer utile pour le test unitaire du client HTTP en injectant des objets fictifs.

L’exemple suivant illustre l’utilisation de l’injection de dépendances pour désactiver la validation de certificat TLS/SSL pour des fonctions d’orchestrateur qui appellent des points de terminaison HTTP externes.

```csharp
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        // Register own factory
        builder.Services.AddSingleton<
            IDurableHttpMessageHandlerFactory,
            MyDurableHttpMessageHandlerFactory>();
    }
}

public class MyDurableHttpMessageHandlerFactory : IDurableHttpMessageHandlerFactory
{
    public HttpMessageHandler CreateHttpMessageHandler()
    {
        // Disable TLS/SSL certificate validation (not recommended in production!)
        return new HttpClientHandler
        {
            ServerCertificateCustomValidationCallback =
                HttpClientHandler.DangerousAcceptAnyServerCertificateValidator,
        };
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les entités durables](durable-functions-entities.md)
