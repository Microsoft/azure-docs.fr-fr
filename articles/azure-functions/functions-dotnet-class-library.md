---
title: Informations de référence pour les développeurs C# sur Azure Functions
description: Découvrez comment développer sur Azure Functions à l’aide de C#.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 07/24/2020
ms.openlocfilehash: 9e11d013b6e7473f290ba1ccb54857034491d116
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672663"
---
# <a name="azure-functions-c-developer-reference"></a>Informations de référence pour les développeurs C# sur Azure Functions

<!-- When updating this article, make corresponding changes to any duplicate content in functions-reference-csharp.md -->

Cet article est une introduction au développement d’Azure Functions à l’aide de C# dans des bibliothèques de classes .NET.

En tant que développeur C#, vous pouvez également être intéressé par l’un des articles suivants :

| Prise en main | Concepts| Apprentissage guidé/exemples |
| -- | -- | -- | 
| <ul><li>[Utilisation de Visual Studio](functions-create-your-first-function-visual-studio.md)</li><li>[Utilisation de Visual Studio Code](create-first-function-vs-code-csharp.md)</li><li>[Utilisation d’outils en ligne de commande](create-first-function-cli-csharp.md)</li></ul> | <ul><li>[Options d’hébergement](functions-scale.md)</li><li>[Considérations relatives aux &nbsp;performances](functions-best-practices.md)</li><li>[Développement Visual Studio](functions-develop-vs.md)</li><li>[Injection de dépendances](functions-dotnet-dependency-injection.md)</li></ul> | <ul><li>[Créer des applications serverless](/learn/paths/create-serverless-applications/)</li><li>[Exemples C#](/samples/browse/?products=azure-functions&languages=csharp)</li></ul> |

Azure Functions prend en charge le langage de programmation C#, mais également le langage de script C#. Pour plus d’informations sur l’[utilisation de C# dans le portail Azure](functions-create-function-app-portal.md), consultez [Informations de référence pour les développeurs de scripts C# (.csx)](functions-reference-csharp.md).

## <a name="supported-versions"></a>Versions prises en charge

Les versions du runtime Functions fonctionnent avec des versions spécifiques de .NET. Le tableau suivant indique le niveau le plus élevé de .NET Core et .NET Framework qui peuvent être utilisés avec une version spécifique d’Azure Functions dans votre projet. 

| Version du runtime Functions | Version maximale de .NET |
| ---- | ---- |
| Functions 3.x | .NET Core 3.1 |
| Functions 2.x | .NET Core 2.2 |
| Functions 1.x | .NET Framework 4.7 |

Pour en savoir plus, voir [Versions du runtime Azure Functions](functions-versions.md)

## <a name="functions-class-library-project"></a>Projet de bibliothèque de classes Azure Functions

Dans Visual Studio, le modèle de projet **Azure Functions** crée un projet de bibliothèque de classes C# qui contient les fichiers suivants :

* [host.json](functions-host-json.md) : stocke les paramètres de configuration qui affectent toutes les fonctions dans le projet au cours d’une exécution locale ou dans Azure.
* [local.settings.json](functions-run-local.md#local-settings-file) : stocke les paramètres de l’application et les chaînes de connexion utilisés au cours d’une exécution locale. Ce fichier contient des secrets et n’est pas publié dans votre application de fonction dans Azure. À la place, [ajoutez des paramètres d’application à votre Function App](functions-develop-vs.md#function-app-settings).

Lorsque vous créez le projet, une structure de dossiers qui ressemble à l’exemple suivant est générée dans le répertoire de sortie de build :

```
<framework.version>
 | - bin
 | - MyFirstFunction
 | | - function.json
 | - MySecondFunction
 | | - function.json
 | - host.json
```

Ce répertoire correspond à ce qui est déployé dans votre application de fonction dans Azure. Les extensions de liaison requises dans la [version 2.x](functions-versions.md) du runtime Azure Functions sont [ajoutées au projet en tant que packages NuGet](./functions-bindings-register.md#vs).

> [!IMPORTANT]
> Le processus de génération crée un fichier *function.json* pour chaque fonction. Ce fichier *function.json* n’est pas destiné à être directement modifié. Vous ne pouvez pas modifier la configuration des liaisons ni désactiver la fonction en modifiant ce fichier. Pour savoir comment désactiver une fonction, consultez le [Guide pratique pour désactiver des fonctions](disable-function.md).


## <a name="methods-recognized-as-functions"></a>Méthodes reconnues en tant que fonctions

Dans une bibliothèque de classes, une fonction est une méthode statique avec un attribut `FunctionName` et un attribut de déclencheur, comme illustré dans l’exemple suivant :

```csharp
public static class SimpleExample
{
    [FunctionName("QueueTrigger")]
    public static void Run(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
} 
```

L’attribut `FunctionName` marque une méthode comme point d’entrée de la fonction. Le nom doit être unique au sein d’un projet, commencer par une lettre et ne contenir que des lettres, des chiffres, `_` et `-`, jusqu’à 127 caractères. Les modèles de projets créent souvent une méthode nommée `Run`, mais le nom de la méthode peut être n’importe quel nom de méthode C# valide.

L’attribut de déclencheur spécifie le type de déclencheur et lie les données d’entrée à un paramètre de méthode. L’exemple de fonction est déclenché par un message de file d’attente, qui est lui-même transmis à la méthode dans le paramètre `myQueueItem`.

## <a name="method-signature-parameters"></a>Paramètres de signature de méthode

La signature de méthode peut contenir des paramètres autres que ceux utilisés avec l’attribut de déclencheur. Voici quelques-uns des paramètres supplémentaires que vous pouvez inclure :

* des [liaisons d’entrée et de sortie](functions-triggers-bindings.md) marquées comme telles à l’aide d’attributs ;  
* un paramètre `ILogger` ou `TraceWriter` ([version 1.x uniquement](functions-versions.md#creating-1x-apps)) pour la [journalisation](#logging).
* un paramètre `CancellationToken` pour [l’arrêt approprié](#cancellation-tokens) ;
* des paramètres [d’expressions de liaison](./functions-bindings-expressions-patterns.md) pour obtenir des métadonnées de déclencheur.

L’ordre des paramètres dans la signature de fonction n’a pas d’importance. Par exemple, vous pouvez placer les paramètres de déclencheur avant ou après les autres liaisons, de même que vous pouvez placer le paramètre de l’enregistreur d’événements avant ou après les paramètres de liaison ou de déclencheur.

### <a name="output-binding-example"></a>Exemple de liaison de sortie

L’exemple suivant modifie l’exemple précédent en ajoutant une liaison de file d’attente de sortie. La fonction écrit le message de file d’attente qui déclenche la fonction vers un nouveau message de file d’attente dans une autre file d’attente.

```csharp
public static class SimpleExampleWithOutput
{
    [FunctionName("CopyQueueMessage")]
    public static void Run(
        [QueueTrigger("myqueue-items-source")] string myQueueItem, 
        [Queue("myqueue-items-destination")] out string myQueueItemCopy,
        ILogger log)
    {
        log.LogInformation($"CopyQueueMessage function processed: {myQueueItem}");
        myQueueItemCopy = myQueueItem;
    }
}
```

Les articles de référence sur les liaisons ([Liaisons de stockage File d’attente Azure pour Azure Functions](functions-bindings-storage-queue.md), par exemple) décrivent les types de paramètre que vous pouvez utiliser avec les attributs de liaison de déclencheur, d’entrée ou de sortie.

### <a name="binding-expressions-example"></a>Exemple d’expression de liaison

Le code suivant permet d’obtenir le nom de la file d’attente à surveiller à partir d’un paramètre d’application, et de récupérer l’heure de création du message de file d’attente dans le paramètre `insertionTime`.

```csharp
public static class BindingExpressionsExample
{
    [FunctionName("LogQueueMessage")]
    public static void Run(
        [QueueTrigger("%queueappsetting%")] string myQueueItem,
        DateTimeOffset insertionTime,
        ILogger log)
    {
        log.LogInformation($"Message content: {myQueueItem}");
        log.LogInformation($"Created at: {insertionTime}");
    }
}
```

## <a name="autogenerated-functionjson"></a>Fichier function.json généré automatiquement

Le processus de build crée un fichier *function.json* dans un dossier de fonction du dossier de build. Comme indiqué précédemment, ce fichier n’est pas destiné à être modifié directement. Vous ne pouvez pas modifier la configuration des liaisons ni désactiver la fonction en modifiant ce fichier. 

L’objectif de ce fichier est de fournir au contrôleur de mise à l’échelle les informations à utiliser pour les [ décisions de mise à l’échelle affectant le plan Consommation](functions-scale.md#how-the-consumption-and-premium-plans-work). C’est pourquoi le fichier ne contient pas de liaisons d’entrée ou de sortie, mais uniquement des informations de déclencheur.

Le fichier *function.json* généré inclut une propriété `configurationSource` qui indique au runtime d’utiliser les attributs .NET pour les liaisons, au lieu de la configuration *function.json*. Voici un exemple :

```json
{
  "generatedBy": "Microsoft.NET.Sdk.Functions-1.0.0.0",
  "configurationSource": "attributes",
  "bindings": [
    {
      "type": "queueTrigger",
      "queueName": "%input-queue-name%",
      "name": "myQueueItem"
    }
  ],
  "disabled": false,
  "scriptFile": "..\\bin\\FunctionApp1.dll",
  "entryPoint": "FunctionApp1.QueueTrigger.Run"
}
```

## <a name="microsoftnetsdkfunctions"></a>Microsoft.NET.Sdk.Functions

La génération du fichier *function.json* est effectuée par le package NuGet [Microsoft\.NET\.Sdk\.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions). 

Le même package est utilisé pour les versions 1.x et 2.x du runtime Functions. Le framework cible est ce qui différencie un projet 1.x d’un projet 2.x. Voici les parties correspondantes des fichiers *.csproj*, qui montrent les frameworks cibles et le même package `Sdk` :

# <a name="v2x"></a>[v2.x+](#tab/v2)

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.1</TargetFramework>
  <AzureFunctionsVersion>v2</AzureFunctionsVersion>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```

# <a name="v1x"></a>[v1.x](#tab/v1)

```xml
<PropertyGroup>
  <TargetFramework>net461</TargetFramework>
</PropertyGroup>
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="1.0.8" />
</ItemGroup>
```
---


Parmi les dépendances de package `Sdk` figurent les déclencheurs et les liaisons. Un projet 1.x fait référence à des déclencheurs et des liaisons 1.x parce que ceux-ci ciblent le .NET Framework, alors que les déclencheurs et liaisons 2.x ciblent .NET Core.

Le package `Sdk` dépend également de [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json) et indirectement de [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage). Ces dépendances garantissent que votre projet utilise des versions de package compatibles avec la version du runtime Functions qui est ciblée par le projet. Par exemple, `Newtonsoft.Json` a la version 11 pour .NET Framework 4.6.1, mais le runtime Functions qui cible .NET Framework 4.6.1 est compatible uniquement avec `Newtonsoft.Json` 9.0.1. Par conséquent, le code de fonction de votre projet doit également utiliser `Newtonsoft.Json` 9.0.1.

Le code source de `Microsoft.NET.Sdk.Functions` est disponible dans le dépôt GitHub [azure\-functions\-vs\-build\-sdk](https://github.com/Azure/azure-functions-vs-build-sdk).

## <a name="runtime-version"></a>Version du runtime

Visual Studio utilise les outils [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools) pour exécuter les projets Functions. Ils constituent l’interface de ligne de commande du runtime Functions.

L’installation des outils Core Tools à l’aide de npm n’affecte pas la version de Core Tools utilisée par Visual Studio. Pour le runtime Functions version 1.x, Visual Studio stocke les versions de Core Tools dans *%USERPROFILE%\AppData\Local\Azure.Functions.Cli* et utilise la dernière version stockée. Pour Functions 2.x, les outils Core Tools sont inclus dans l’extension **Azure Functions and Web Jobs Tools**. Pour les versions 1.x et 2.x, vous pouvez voir quelle version est utilisée dans la sortie de console lorsque vous exécutez un projet Functions :

```terminal
[3/1/2018 9:59:53 AM] Starting Host (HostId=contoso2-1518597420, Version=2.0.11353.0, ProcessId=22020, Debug=False, Attempt=0, FunctionsExtensionVersion=)
```

## <a name="readytorun"></a>ReadyToRun

Vous pouvez compiler votre application de fonction en tant que [binaires ReadyToRun](/dotnet/core/whats-new/dotnet-core-3-0#readytorun-images). ReadyToRun est une forme de compilation à l’avance qui peut améliorer les performances de démarrage pour réduire l’impact de [ démarrage à froid](functions-scale.md#cold-start) lors de l’exécution dans un [plan de consommation.](functions-scale.md#consumption-plan)

ReadyToRun est disponible dans .NET 3.0 et requiert la[version 3.0 d’Azure Functions Runtime](functions-versions.md).

Pour compiler votre projet en tant que ReadyToRun, mettez à jour votre fichier projet en ajoutant les éléments `<PublishReadyToRun>` et `<RuntimeIdentifier>`. Voici la configuration de la publication dans une application de fonction Windows 32 bits.

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.1</TargetFramework>
  <AzureFunctionsVersion>v3</AzureFunctionsVersion>
  <PublishReadyToRun>true</PublishReadyToRun>
  <RuntimeIdentifier>win-x86</RuntimeIdentifier>
</PropertyGroup>
```

> [!IMPORTANT]
> ReadyToRun ne prend actuellement pas en charge la compilation croisée. Vous devez générer votre application sur la même plateforme que la cible de déploiement. Faites également attention au « nombre de bits » configuré dans votre application de fonction. Par exemple, si votre application de fonction dans Azure est Windows 64 bits, vous devez compiler votre application sur Windows avec `win-x64` comme [identificateur d’exécution](/dotnet/core/rid-catalog).

Vous pouvez également générer votre application avec ReadyToRun à partir de la ligne de commande. Pour plus d'informations, consultez `-p:PublishReadyToRun=true`l'option dans [`dotnet publish`](/dotnet/core/tools/dotnet-publish).

## <a name="supported-types-for-bindings"></a>Types pris en charge pour les liaisons

Chaque liaison possède ses propres types pris en charge. Par exemple, un attribut de déclencheur d’objet blob peut être appliqué à un paramètre de chaîne, à un paramètre OCT, à un paramètre `CloudBlockBlob` ou à l’un des autres types pris en charge. L’[article sur les références de liaison pour les liaisons d’objets blob](functions-bindings-storage-blob-trigger.md#usage) répertorie tous les types de paramètre pris en charge. Pour plus d’informations, consultez [Déclencheurs et liaisons](functions-triggers-bindings.md), ainsi que les [documents sur les références de liaison pour chaque type de liaison](functions-triggers-bindings.md#next-steps).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="binding-to-method-return-value"></a>Liaison à une valeur renvoyée par la méthode

Vous pouvez utiliser une valeur de retour de méthode pour une liaison de sortie en appliquant l’attribut à la valeur de retour de méthode. Pour obtenir des exemples, consultez [Déclencheurs et liaisons](./functions-bindings-return-value.md). 

Utilisez la valeur de retour seulement si une exécution réussie de la fonction aboutit toujours à une valeur de retour à passer à la liaison de sortie. Sinon, utilisez `ICollector` ou `IAsyncCollector`, comme illustré dans la section suivante.

## <a name="writing-multiple-output-values"></a>Écrire plusieurs valeurs de sortie

Pour écrire plusieurs valeurs dans une liaison de sortie, ou si un appel de fonction réussi n’aboutit pas nécessairement à quelque chose à passer à la liaison de sortie, utilisez les types [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) ou [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs). Ces types sont des collections en écriture seule, écrites dans la liaison de sortie à la fin de la méthode.

Cet exemple écrit plusieurs messages de file d’attente dans la même file d’attente à l’aide de `ICollector` :

```csharp
public static class ICollectorExample
{
    [FunctionName("CopyQueueMessageICollector")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-3")] string myQueueItem,
        [Queue("myqueue-items-destination")] ICollector<string> myDestinationQueue,
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
        myDestinationQueue.Add($"Copy 1: {myQueueItem}");
        myDestinationQueue.Add($"Copy 2: {myQueueItem}");
    }
}
```

## <a name="async"></a>Async

Pour rendre une fonction [asynchrone](/dotnet/csharp/programming-guide/concepts/async/), utilisez le mot clé `async` et retournez un objet `Task`.

```csharp
public static class AsyncExample
{
    [FunctionName("BlobCopy")]
    public static async Task RunAsync(
        [BlobTrigger("sample-images/{blobName}")] Stream blobInput,
        [Blob("sample-images-copies/{blobName}", FileAccess.Write)] Stream blobOutput,
        CancellationToken token,
        ILogger log)
    {
        log.LogInformation($"BlobCopy function processed.");
        await blobInput.CopyToAsync(blobOutput, 4096, token);
    }
}
```

Vous ne pouvez pas utiliser de paramètres `out` dans des fonctions asynchrones. Pour les liaisons de sortie, utilisez la [valeur de retour de fonction](#binding-to-method-return-value) ou un [objet collecteur](#writing-multiple-output-values) à la place.

## <a name="cancellation-tokens"></a>Jetons d’annulation

Une fonction peut accepter un paramètre [CancellationToken](/dotnet/api/system.threading.cancellationtoken) qui permet au système d’exploitation de notifier votre code quand la fonction est sur le point de se terminer. Vous pouvez utiliser cette notification pour vous assurer que la fonction ne s’arrête pas de manière inattendue et laisse les données dans un état incohérent.

L’exemple suivant montre comment vérifier l’arrêt imminent d’une fonction.

```csharp
public static class CancellationTokenExample
{
    public static void Run(
        [QueueTrigger("inputqueue")] string inputText,
        TextWriter logger,
        CancellationToken token)
    {
        for (int i = 0; i < 100; i++)
        {
            if (token.IsCancellationRequested)
            {
                logger.WriteLine("Function was cancelled at iteration {0}", i);
                break;
            }
            Thread.Sleep(5000);
            logger.WriteLine("Normal processing for queue message={0}", inputText);
        }
    }
}
```

## <a name="logging"></a>Journalisation

Dans votre code de fonction, vous pouvez écrire la sortie dans des journaux qui apparaissent sous forme de traces dans Application Insights. La méthode recommandée pour écrire dans les journaux consiste à inclure un paramètre de type [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger), généralement nommé `log`. La version 1. x du runtime Functions a utilisé `TraceWriter`, qui écrit également dans Application Insights, mais ne prend pas en charge la journalisation structurée. N’utilisez pas `Console.Write` pour écrire vos journaux, car ces données ne sont pas capturées par Application Insights. 

### <a name="ilogger"></a>ILogger

Dans votre définition de fonction, incluez un paramètre [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger) qui prend en charge la [journalisation structurée](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).

Avec un objet `ILogger`, vous appelez des [méthodes d’extension `Log<level>` sur ILogger](/dotnet/api/microsoft.extensions.logging.loggerextensions#methods) pour créer des journaux. Le code suivant écrit des journaux d’activité `Information` ayant la catégorie `Function.<YOUR_FUNCTION_NAME>.User.` :

```cs
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger logger)
{
    logger.LogInformation("Request for item with key={itemKey}.", id);
```

Pour en savoir plus sur la façon dont Functions implémente `ILogger`, consultez [Collecte de données de télémétrie](functions-monitoring.md#collecting-telemetry-data). Les catégories dotées du préfixe `Function` supposent que vous utilisez une instance `ILogger`. Si vous choisissez d’utiliser à la place un `ILogger<T>`, le nom de la catégorie peut être basé sur `T`.  

### <a name="structured-logging"></a>Journalisation structurée

C’est l’ordre des espaces réservés, pas leurs noms, qui détermine les paramètres utilisés dans le message du journal. Supposons que vous disposiez du code suivant :

```csharp
string partitionKey = "partitionKey";
string rowKey = "rowKey";
logger.LogInformation("partitionKey={partitionKey}, rowKey={rowKey}", partitionKey, rowKey);
```

Si vous conservez la même chaîne de message et que vous inversez l’ordre des paramètres, les valeurs dans le texte du message résultant ne sont pas situées à la bonne place.

Ce mode de gestion des espaces réservés vous permet d’effectuer une journalisation structurée. Application Insights stocke les paires nom de paramètre/valeur et la chaîne de message. Ainsi, les arguments du message deviennent des champs pouvant faire l’objet de requêtes.

Si votre appel de méthode de l’enregistreur d’événements ressemble à l’exemple précédent, vous pouvez interroger le champ `customDimensions.prop__rowKey`. L’ajout du préfixe `prop__` évite toute collision entre les champs ajoutés par le runtime et les champs ajoutés par votre fonction de code.

Vous pouvez également exécuter une requête portant sur la chaîne de message d’origine en référençant le champ `customDimensions.prop__{OriginalFormat}`.  

Voici un exemple de représentation JSON des données `customDimensions` :

```json
{
  "customDimensions": {
    "prop__{OriginalFormat}":"C# Queue trigger function processed: {message}",
    "Category":"Function",
    "LogLevel":"Information",
    "prop__message":"c9519cbf-b1e6-4b9b-bf24-cb7d10b1bb89"
  }
}
```

## <a name="log-custom-telemetry-in-c-functions"></a>Journaliser des données de télémétrie personnalisées dans les fonctions C#

Il existe une version du kit de développement logiciel (SDK) Application Insights spécifique à Functions et que vous pouvez utiliser pour envoyer des données de télémétrie personnalisées de vos fonctions vers Application Insights : [Microsoft.Azure.WebJobs.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights). Utilisez la commande suivante depuis l’invite de commandes pour installer ce package :

# <a name="command"></a>[Commande](#tab/cmd)

```cmd
dotnet add package Microsoft.Azure.WebJobs.Logging.ApplicationInsights --version <VERSION>
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version <VERSION>
```

---

Dans cette commande, remplacez `<VERSION>` par une version de ce package qui prend en charge la version installée de [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs/). 

L’exemple suivant en C# utilise l’[API de télémétrie personnalisée](../azure-monitor/app/api-custom-events-metrics.md). L’exemple concerne une bibliothèque de classes .NET, mais le code Application Insights est le même pour le script C#.

# <a name="v2x"></a>[v2.x+](#tab/v2)

La version 2.x et les versions ultérieures du runtime utilisent les nouvelles fonctionnalités d’Application Insights pour mettre automatiquement en corrélation les données de télémétrie avec l’opération en cours. Il est inutile de définir manuellement les champs de l’opération `Id`, `ParentId` ou `Name`.

```cs
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using System.Linq;

namespace functionapp0915
{
    public class HttpTrigger2
    {
        private readonly TelemetryClient telemetryClient;

        /// Using dependency injection will guarantee that you use the same configuration for telemetry collected automatically and manually.
        public HttpTrigger2(TelemetryConfiguration telemetryConfiguration)
        {
            this.telemetryClient = new TelemetryClient(telemetryConfiguration);
        }

        [FunctionName("HttpTrigger2")]
        public Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = null)]
            HttpRequest req, ExecutionContext context, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            DateTime start = DateTime.UtcNow;

            // Parse query parameter
            string name = req.Query
                .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
                .Value;

            // Write an event to the customEvents table.
            var evt = new EventTelemetry("Function called");
            evt.Context.User.Id = name;
            this.telemetryClient.TrackEvent(evt);

            // Generate a custom metric, in this case let's use ContentLength.
            this.telemetryClient.GetMetric("contentLength").TrackValue(req.ContentLength);

            // Log a custom dependency in the dependencies table.
            var dependency = new DependencyTelemetry
            {
                Name = "GET api/planets/1/",
                Target = "swapi.co",
                Data = "https://swapi.co/api/planets/1/",
                Timestamp = start,
                Duration = DateTime.UtcNow - start,
                Success = true
            };
            dependency.Context.User.Id = name;
            this.telemetryClient.TrackDependency(dependency);

            return Task.FromResult<IActionResult>(new OkResult());
        }
    }
}
```

Dans cet exemple, les données de métriques personnalisées sont agrégées par l’hôte avant d’être envoyées à la table customMetrics. Pour plus d’informations, consultez la documentation [GetMetric](../azure-monitor/app/api-custom-events-metrics.md#getmetric) dans Application Insights. 

Lors d’une exécution locale, vous devez ajouter le paramètre `APPINSIGHTS_INSTRUMENTATIONKEY` avec la clé Application Insights, au fichier [local.settings.json](functions-run-local.md#local-settings-file).


# <a name="v1x"></a>[v1.x](#tab/v1)

```cs
using System;
using System.Net;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Azure.WebJobs;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Linq;

namespace functionapp0915
{
    public static class HttpTrigger2
    {
        private static string key = TelemetryConfiguration.Active.InstrumentationKey = 
            System.Environment.GetEnvironmentVariable(
                "APPINSIGHTS_INSTRUMENTATIONKEY", EnvironmentVariableTarget.Process);

        private static TelemetryClient telemetryClient = 
            new TelemetryClient() { InstrumentationKey = key };

        [FunctionName("HttpTrigger2")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
            HttpRequestMessage req, ExecutionContext context, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            DateTime start = DateTime.UtcNow;

            // Parse query parameter
            string name = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
                .Value;

            // Get request body
            dynamic data = await req.Content.ReadAsAsync<object>();

            // Set name to query string or body data
            name = name ?? data?.name;
         
            // Track an Event
            var evt = new EventTelemetry("Function called");
            UpdateTelemetryContext(evt.Context, context, name);
            telemetryClient.TrackEvent(evt);
            
            // Track a Metric
            var metric = new MetricTelemetry("Test Metric", DateTime.Now.Millisecond);
            UpdateTelemetryContext(metric.Context, context, name);
            telemetryClient.TrackMetric(metric);
            
            // Track a Dependency
            var dependency = new DependencyTelemetry
            {
                Name = "GET api/planets/1/",
                Target = "swapi.co",
                Data = "https://swapi.co/api/planets/1/",
                Timestamp = start,
                Duration = DateTime.UtcNow - start,
                Success = true
            };
            UpdateTelemetryContext(dependency.Context, context, name);
            telemetryClient.TrackDependency(dependency);
        }
        
        // Correlate all telemetry with the current Function invocation
        private static void UpdateTelemetryContext(TelemetryContext context, ExecutionContext functionContext, string userName)
        {
            context.Operation.Id = functionContext.InvocationId.ToString();
            context.Operation.ParentId = functionContext.InvocationId.ToString();
            context.Operation.Name = functionContext.FunctionName;
            context.User.Id = userName;
        }
    }    
}
```
---

N’appelez pas `TrackRequest` ou `StartOperation<RequestTelemetry>`, afin d’éviter les doublons de requêtes pour un appel de fonction.  Le runtime d’Azure Functions effectue automatiquement le suivi des demandes.

Ne définissez pas `telemetryClient.Context.Operation.Id`. Ce paramètre global provoque une erreur de corrélation lorsqu’un grand nombre de fonctions s’exécutent en même temps. Au lieu de cela, créez une nouvelle instance de télémétrie (`DependencyTelemetry`, `EventTelemetry`) et modifiez sa propriété `Context`. Passez ensuite l’instance de télémétrie à la méthode `Track` correspondante sur `TelemetryClient` (`TrackDependency()`, `TrackEvent()`, `TrackMetric()`). Cette méthode permet de s'assurer que les données de télémétrie comportent les bonnes informations de corrélation pour l’appel de fonction actuel.


## <a name="environment-variables"></a>Variables d'environnement

Pour obtenir une variable d’environnement ou une valeur de paramètre d’application, utilisez `System.Environment.GetEnvironmentVariable`, comme illustré dans l’exemple de code suivant :

```csharp
public static class EnvironmentVariablesExample
{
    [FunctionName("GetEnvironmentVariables")]
    public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
    {
        log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
        log.LogInformation(GetEnvironmentVariable("AzureWebJobsStorage"));
        log.LogInformation(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
    }

    private static string GetEnvironmentVariable(string name)
    {
        return name + ": " +
            System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
    }
}
```

Les paramètres de l’application peuvent être lus à partir de variables d’environnement lors du développement localement et de l’exécution dans Azure. Lors du développement localement, les paramètres de l’application proviennent de la collection `Values` dans le fichier *local.settings.json*. Dans les deux environnements, local et Azure, `GetEnvironmentVariable("<app setting name>")` récupère la valeur du paramètre d’application nommé. Par exemple, lorsque vous exécutez localement, « My Site Name » est retourné si votre fichier *local.settings.json* contient `{ "Values": { "WEBSITE_SITE_NAME": "My Site Name" } }`.

La propriété [System.Configuration.ConfigurationManager.AppSettings](/dotnet/api/system.configuration.configurationmanager.appsettings) est une autre API permettant d’obtenir des valeurs de paramètre d’application, mais nous vous recommandons d’utiliser `GetEnvironmentVariable` comme indiqué ici.

## <a name="binding-at-runtime"></a>Liaison au runtime

Avec C# et d’autres langages .NET, vous pouvez utiliser un schéma de liaison [impératif](https://en.wikipedia.org/wiki/Imperative_programming), par opposition aux liaisons [*déclaratives*](https://en.wikipedia.org/wiki/Declarative_programming) dans les attributs. La liaison impérative est utile lorsque les paramètres de liaison doivent être calculés au moment du runtime plutôt que lors de la conception. Avec ce modèle, vous pouvez effectuer une liaison à la volée avec une liaison d’entrée et de sortie prise en charge dans le code de votre fonction.

Définissez une liaison impérative comme suit :

- **N’incluez pas** d’attribut dans la signature de fonction pour les liaisons impératives souhaitées.
- Transmettez un paramètre d’entrée [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) ou [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs).
- Utilisez le modèle en C# suivant pour effectuer la liaison de données.

  ```cs
  using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
  {
      ...
  }
  ```

  `BindingTypeAttribute` est l’attribut .NET qui définit votre liaison et `T` est le type d’entrée ou de sortie pris en charge par ce type de liaison. `T` ne peut pas être un type de paramètre `out` (comme `out JObject`). Par exemple, la liaison de sortie de la table Mobile Apps prend en charge [six types de sortie](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22), mais vous pouvez utiliser uniquement [ICollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) ou [IAsyncCollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) avec des liaisons impératives.

### <a name="single-attribute-example"></a>Exemple d’attribut unique

L’exemple de code suivant crée une [liaison de sortie d’objet blob de stockage](functions-bindings-storage-blob-output.md) avec un chemin d’objet blob défini au moment de l’exécution, puis écrit une chaîne vers l’objet blob.

```cs
public static class IBinderExample
{
    [FunctionName("CreateBlobUsingBinder")]
    public static void Run(
        [QueueTrigger("myqueue-items-source-4")] string myQueueItem,
        IBinder binder,
        ILogger log)
    {
        log.LogInformation($"CreateBlobUsingBinder function processed: {myQueueItem}");
        using (var writer = binder.Bind<TextWriter>(new BlobAttribute(
                    $"samples-output/{myQueueItem}", FileAccess.Write)))
        {
            writer.Write("Hello World!");
        };
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs) définit la liaison d’entrée ou de sortie de [l’objet blob de stockage](functions-bindings-storage-blob.md), et [TextWriter](/dotnet/api/system.io.textwriter) est un type de liaison de sortie pris en charge.

### <a name="multiple-attribute-example"></a>Exemple d’attributs multiples

L’exemple précédent obtient le paramètre d’application pour la chaîne de connexion du compte de stockage principal de l’application de fonction (à savoir `AzureWebJobsStorage`). Vous pouvez spécifier un paramètre d’application personnalisé à utiliser pour le compte de stockage en ajoutant l’attribut [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) et en transmettant le tableau d’attributs dans `BindAsync<T>()`. Utilisez un paramètre `Binder`, et non `IBinder`.  Par exemple :

```cs
public static class IBinderExampleMultipleAttributes
{
    [FunctionName("CreateBlobInDifferentStorageAccount")]
    public async static Task RunAsync(
            [QueueTrigger("myqueue-items-source-binder2")] string myQueueItem,
            Binder binder,
            ILogger log)
    {
        log.LogInformation($"CreateBlobInDifferentStorageAccount function processed: {myQueueItem}");
        var attributes = new Attribute[]
        {
        new BlobAttribute($"samples-output/{myQueueItem}", FileAccess.Write),
        new StorageAccountAttribute("MyStorageAccount")
        };
        using (var writer = await binder.BindAsync<TextWriter>(attributes))
        {
            await writer.WriteAsync("Hello World!!");
        }
    }
}
```

## <a name="triggers-and-bindings"></a>Déclencheurs et liaisons 

[!INCLUDE [Supported triggers and bindings](../../includes/functions-bindings.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [En savoir plus sur les meilleures pratiques pour Azure Functions](functions-best-practices.md)
