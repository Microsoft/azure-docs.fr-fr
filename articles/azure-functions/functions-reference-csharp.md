---
title: Informations de référence pour les développeurs de scripts C# Azure Functions
description: Découvrez comment développer sur Azure Functions à l’aide de scripts C#.
author: craigshoemaker
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 12/12/2017
ms.author: cshoe
ms.openlocfilehash: 48614640660da6d85face5ea416d267fa9f59515
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92164837"
---
# <a name="azure-functions-c-script-csx-developer-reference"></a>Informations de référence pour les développeurs de scripts C# (.csx) Azure Functions

<!-- When updating this article, make corresponding changes to any duplicate content in functions-dotnet-class-library.md -->

Cet article est une introduction au développement d’Azure Functions à l’aide de scripts C# ( *.csx* ).

Azure Functions prend en charge le langage de programmation C#, mais également le langage de script C#. Pour plus d’informations sur l’[utilisation de C# dans un projet de bibliothèque de classes Visual Studio](functions-develop-vs.md), consultez les [informations de référence pour les développeurs C#](functions-dotnet-class-library.md).

Cet article suppose que vous avez déjà lu le [Guide de développement Azure Functions](functions-reference.md).

## <a name="how-csx-works"></a>Fonctionnement de .csx

L’expérience de scripts C# pour Azure Functions repose sur le [Kit SDK Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki/Introduction). Les données circulent dans votre fonction C# via des arguments de méthode. Les noms d’argument sont spécifiés dans un fichier `function.json`, et il existe des noms prédéfinis pour accéder à des éléments tels que l’enregistreur de fonctions et les jetons d’annulation.

Le format *.csx* vous permet d’écrire de façon moins « réutilisable » et de vous concentrer uniquement sur l’écriture d’une fonction C#. Au lieu de tout encapsuler dans un espace de noms et une classe, définissez simplement une méthode `Run`. Ajoutez les éventuels références d’assemblys et espaces de noms au début du fichier, comme d’habitude.

Les fichiers *.csx* d’une application de fonction sont compilés quand une instance est initialisée. Cette étape de compilation signifie que des opérations telles qu’un démarrage à froid peuvent être plus longues à effectuer pour les fonctions de script C# par rapport aux bibliothèques de classes C#. C’est également la raison pour laquelle les fonctions de script C# sont modifiables dans le portail Azure, alors que les bibliothèques de classes C# ne le sont pas.

## <a name="folder-structure"></a>Structure de dossiers

La structure de dossiers pour un projet de script C# est similaire à la structure ci-après :

```
FunctionsProject
 | - MyFirstFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - MySecondFunction
 | | - run.csx
 | | - function.json
 | | - function.proj
 | - host.json
 | - extensions.csproj
 | - bin
```

Il existe un fichier [host.json](functions-host-json.md) partagé que vous pouvez utiliser pour configurer l’application de fonction. Chaque fonction a son propre fichier de code (.csx) et un fichier de configuration de liaison (function.json).

Les extensions de liaison nécessaires dans la [version 2.x et ultérieure](functions-versions.md) du runtime Functions sont définies dans le fichier `extensions.csproj`, les fichiers de bibliothèque proprement dits se trouvant dans le dossier `bin`. Quand vous développez localement, vous devez [inscrire les extensions de liaison](./functions-bindings-register.md#extension-bundles). Quand vous développez des fonctions dans le portail Azure, cet enregistrement est effectué pour vous.

## <a name="binding-to-arguments"></a>Liaison aux arguments

Les données d’entrée ou de sortie sont liées à un paramètre de fonction de script C# via la propriété `name` du fichier de configuration *function.json* . L’exemple suivant montre un fichier *function.json* et un fichier *run.csx* pour une fonction déclenchée par une file d’attente. Le paramètre qui reçoit les données du message de file d’attente s’appelle `myQueueItem`, car il s’agit de la valeur de la propriété `name`.

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}");
}
```

L’instruction `#r` est expliquée [plus loin dans cet article](#referencing-external-assemblies).

## <a name="supported-types-for-bindings"></a>Types pris en charge pour les liaisons

Chaque liaison possède ses propres types pris en charge. Par exemple, un déclencheur d’objet blob peut être utilisé avec un paramètre de chaîne, un paramètre OCT, un paramètre `CloudBlockBlob` ou l’un des autres types pris en charge. L’[article sur les références de liaison pour les liaisons d’objets blob](functions-bindings-storage-blob-trigger.md#usage) répertorie tous les types de paramètre pris en charge pour les déclencheurs d’objet blob. Pour plus d’informations, consultez [Déclencheurs et liaisons](functions-triggers-bindings.md), ainsi que les [documents sur les références de liaison pour chaque type de liaison](functions-triggers-bindings.md#next-steps).

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

## <a name="referencing-custom-classes"></a>Référencement de classes personnalisées

Si vous souhaitez utiliser une classe d’objet CLR traditionnel (OCT) personnalisée, vous pouvez inclure la définition de classe dans le même fichier ou la placer dans un fichier distinct.

L’exemple de fichier *run.csx* suivant inclut une définition de classe OCT.

```csharp
public static void Run(string myBlob, out MyClass myQueueItem)
{
    log.Verbose($"C# Blob trigger function processed: {myBlob}");
    myQueueItem = new MyClass() { Id = "myid" };
}

public class MyClass
{
    public string Id { get; set; }
}
```

Des méthodes getter et setter doivent être définies pour chaque propriété d’une classe OCT.

## <a name="reusing-csx-code"></a>Réutilisation du code .csx

Vous pouvez utiliser des classes et des méthodes définies dans d’autres fichiers *.csx* au sein de votre fichier *run.csx* . Pour ce faire, utilisez les directives `#load` dans votre fichier *run.csx* . Dans l’exemple suivant, une routine de journalisation nommée `MyLogger` est partagée dans *myLogger.csx* et chargés dans *run.csx* à l’aide de la directive`#load` :

Exemple *run.csx* :

```csharp
#load "mylogger.csx"

using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Log by run.csx: {DateTime.Now}");
    MyLogger(log, $"Log by MyLogger: {DateTime.Now}");
}
```

Exemple *mylogger.csx* :

```csharp
public static void MyLogger(ILogger log, string logtext)
{
    log.LogInformation(logtext);
}
```

L’utilisation d’un fichier *.csx* partagé est un cas de figure courant quand vous souhaitez typer fortement les données transmises entre les fonctions à l’aide d’un objet OCT. Dans l’exemple simplifié suivant, un déclencheur HTTP et un déclencheur de file d’attente partagent un objet OCT nommé `Order` pour typer fortement les données de commande :

Exemple avec *run.csx* pour un déclencheur HTTP :

```cs
#load "..\shared\order.csx"

using System.Net;
using Microsoft.Extensions.Logging;

public static async Task<HttpResponseMessage> Run(Order req, IAsyncCollector<Order> outputQueueItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function received an order.");
    log.LogInformation(req.ToString());
    log.LogInformation("Submitting to processing queue.");

    if (req.orderId == null)
    {
        return new HttpResponseMessage(HttpStatusCode.BadRequest);
    }
    else
    {
        await outputQueueItem.AddAsync(req);
        return new HttpResponseMessage(HttpStatusCode.OK);
    }
}
```

Exemple avec *run.csx* pour un déclencheur de file d’attente :

```cs
#load "..\shared\order.csx"

using System;
using Microsoft.Extensions.Logging;

public static void Run(Order myQueueItem, out Order outputQueueItem, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed order...");
    log.LogInformation(myQueueItem.ToString());

    outputQueueItem = myQueueItem;
}
```

Exemple avec *order.csx*  :

```cs
public class Order
{
    public string orderId {get; set; }
    public string custName {get; set;}
    public string custAddress {get; set;}
    public string custEmail {get; set;}
    public string cartId {get; set; }

    public override String ToString()
    {
        return "\n{\n\torderId : " + orderId +
                  "\n\tcustName : " + custName +
                  "\n\tcustAddress : " + custAddress +
                  "\n\tcustEmail : " + custEmail +
                  "\n\tcartId : " + cartId + "\n}";
    }
}
```

Vous pouvez utiliser un chemin d’accès relatif avec la directive `#load` :

* `#load "mylogger.csx"` charge un fichier situé dans le dossier de la fonction.
* `#load "loadedfiles\mylogger.csx"` charge un fichier situé dans un dossier du dossier de la fonction.
* `#load "..\shared\mylogger.csx"` charge un fichier situé dans un dossier situé au même niveau que le dossier de la fonction, c’est-à-dire directement sous *wwwroot* .

La directive `#load` ne fonctionne qu’avec des fichiers *.csx* , et non avec des fichiers *.cs* .

## <a name="binding-to-method-return-value"></a>Liaison à une valeur renvoyée par la méthode

Vous pouvez utiliser une valeur renvoyée par la méthode pour une liaison de sortie, en utilisant le nom `$return` dans *function.json* . Pour obtenir des exemples, consultez [Déclencheurs et liaisons](./functions-bindings-return-value.md).

Utilisez la valeur de retour seulement si une exécution réussie de la fonction aboutit toujours à une valeur de retour à passer à la liaison de sortie. Sinon, utilisez `ICollector` ou `IAsyncCollector`, comme illustré dans la section suivante.

## <a name="writing-multiple-output-values"></a>Écrire plusieurs valeurs de sortie

Pour écrire plusieurs valeurs dans une liaison de sortie, ou si un appel de fonction réussi n’aboutit pas nécessairement à quelque chose à passer à la liaison de sortie, utilisez les types [`ICollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) ou [`IAsyncCollector`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs). Ces types sont des collections en écriture seule, écrites dans la liaison de sortie à la fin de la méthode.

Cet exemple écrit plusieurs messages de file d’attente dans la même file d’attente à l’aide de `ICollector` :

```csharp
public static void Run(ICollector<string> myQueue, ILogger log)
{
    myQueue.Add("Hello");
    myQueue.Add("World!");
}
```

## <a name="logging"></a>Journalisation

Pour consigner la sortie dans vos journaux d’activité de streaming en C#, ajoutez un argument de type [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger). Nous vous recommandons de le nommer `log`. Évitez d’utiliser `Console.Write` dans Azure Functions.

```csharp
public static void Run(string myBlob, ILogger log)
{
    log.LogInformation($"C# Blob trigger function processed: {myBlob}");
}
```

> [!NOTE]
> Pour plus d’informations sur un framework de journalisation plus récent que vous pouvez utiliser au lieu de `TraceWriter`, consultez la documentation [ILogger](functions-dotnet-class-library.md#ilogger) dans le Guide du développeur de la bibliothèque de classes .NET.

### <a name="custom-metrics-logging"></a>Journalisation des métriques personnalisées

Vous pouvez utiliser la méthode d’extension `LogMetric` sur `ILogger` pour créer des métriques personnalisées dans Application Insights. Voici un exemple d’appel de méthode :

```csharp
logger.LogMetric("TestMetric", 1234);
```

Ce code est une alternative à l’appel de `TrackMetric` à l’aide de l’API Application Insights pour .NET.

## <a name="async"></a>Async

Pour rendre une fonction [asynchrone](/dotnet/csharp/programming-guide/concepts/async/), utilisez le mot clé `async` et retournez un objet `Task`.

```csharp
public async static Task ProcessQueueMessageAsync(
        string blobName,
        Stream blobInput,
        Stream blobOutput)
{
    await blobInput.CopyToAsync(blobOutput, 4096);
}
```

Vous ne pouvez pas utiliser de paramètres `out` dans des fonctions asynchrones. Pour les liaisons de sortie, utilisez la [valeur de retour de fonction](#binding-to-method-return-value) ou un [objet collecteur](#writing-multiple-output-values) à la place.

## <a name="cancellation-tokens"></a>Jetons d’annulation

Une fonction peut accepter un paramètre [CancellationToken](/dotnet/api/system.threading.cancellationtoken) qui permet au système d’exploitation de notifier votre code quand la fonction est sur le point de se terminer. Vous pouvez utiliser cette notification pour vous assurer que la fonction ne s’arrête pas de manière inattendue et laisse les données dans un état incohérent.

L’exemple suivant montre comment vérifier l’arrêt imminent d’une fonction.

```csharp
using System;
using System.IO;
using System.Threading;

public static void Run(
    string inputText,
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
```

## <a name="importing-namespaces"></a>Importation des espaces de noms

Si vous avez besoin d’importer des espaces de noms, vous pouvez le faire comme vous en avez l’habitude à l’aide de la clause `using` .

```csharp
using System.Net;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

Les espaces de noms suivants sont automatiquement importés et sont donc facultatifs :

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`

## <a name="referencing-external-assemblies"></a>Référencement des assemblys externes

Pour les assemblys de framework, ajoutez des références à l’aide de la directive `#r "AssemblyName"` .

```csharp
#r "System.Web.Http"

using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
```

Les assemblys suivants sont ajoutés automatiquement par l’environnement hébergeant Azure Functions :

* `mscorlib`
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`

Les assemblys suivants peuvent être référencés par nom simple (par exemple, `#r "AssemblyName"`) :

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNet.WebHooks.Common`
* `Microsoft.Azure.NotificationHubs`

## <a name="referencing-custom-assemblies"></a>Référencer des assemblys personnalisés

Pour référencer un assembly personnalisé, vous pouvez utiliser soit un assembly *partagé* , soit un assembly  *privé* :

* Les assemblys partagés sont communs à toutes les fonctions d’une application de fonction. Pour référencer un assembly personnalisé, chargez-le dans un dossier nommé `bin` dans le [dossier racine de votre application de fonction](functions-reference.md#folder-structure) (wwwroot).

* Les assemblys privés font partie du contexte d’une fonction donnée et sont compatibles avec le chargement indépendant de versions différentes. Les assemblys privés doivent être chargés dans un dossier `bin` du répertoire de la fonction. Référencez les assemblys avec le nom de fichier, par exemple `#r "MyAssembly.dll"`.

Pour plus d’informations sur le téléchargement de fichiers vers votre conteneur de fonctions, consultez la section sur la [gestion des packages](#using-nuget-packages).

### <a name="watched-directories"></a>Répertoires surveillés

Le répertoire qui contient le fichier de script de la fonction est automatiquement surveillé pour détecter les modifications apportées aux assemblys. Pour surveiller les modifications des assemblys dans d’autres répertoires, ajoutez-les à la liste `watchDirectories` dans [host.json](functions-host-json.md).

## <a name="using-nuget-packages"></a>Utiliser des packages NuGet
Pour utiliser des packages NuGet dans une fonction C# 2.x et ultérieur, chargez un fichier *function.proj* dans le dossier de la fonction du système de fichiers de l’application de fonction. Voici un exemple de fichier *function.proj* qui ajoute une référence à *Microsoft.ProjectOxford.Face* version *1.1.0*  :

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="Microsoft.ProjectOxford.Face" Version="1.1.0" />
    </ItemGroup>
</Project>
```

Pour utiliser un flux NuGet personnalisé, spécifiez-le dans un fichier *Nuget.Config* à la racine de l’application de fonction. Pour plus d’informations, consultez la page [Configurer le comportement de NuGet](/nuget/consume-packages/configuring-nuget-behavior).

> [!NOTE]
> Dans les fonctions C# 1.0, les packages NuGet sont référencés avec un fichier *project.json* au lieu d’un fichier *function.proj* .

Pour les fonctions 1.x, utilisez un fichier *project.json* à la place. Voici un exemple de fichier *project.json*  :

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

### <a name="using-a-functionproj-file"></a>Utilisation d’un fichier function.proj

1. Ouvrez la fonction sur le Portail Azure. L’onglet journaux d’activité affiche la sortie d’installation du package.
2. Pour charger un fichier *function.json* , utilisez l’une des méthodes décrites dans la section [ Guide pratique pour mettre à jour les fichiers d’application de fonction](functions-reference.md#fileupdate) de la rubrique Informations de référence pour les développeurs sur Azure Functions.
3. Une fois le fichier *function.proj* chargé, une sortie semblable à l’exemple ci-après s’affiche dans le journal de diffusion en continu de votre fonction :

```
2018-12-14T22:00:48.658 [Information] Restoring packages.
2018-12-14T22:00:48.681 [Information] Starting packages restore
2018-12-14T22:00:57.064 [Information] Restoring packages for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj...
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\function.proj...
2018-12-14T22:01:00.844 [Information] Installing Newtonsoft.Json 10.0.2.
2018-12-14T22:01:01.041 [Information] Installing Microsoft.ProjectOxford.Common.DotNetStandard 1.0.0.
2018-12-14T22:01:01.140 [Information] Installing Microsoft.ProjectOxford.Face.DotNetStandard 1.0.0.
2018-12-14T22:01:09.799 [Information] Restore completed in 5.79 sec for D:\local\Temp\9e814101-fe35-42aa-ada5-f8435253eb83\function.proj.
2018-12-14T22:01:10.905 [Information] Packages restored.
```

## <a name="environment-variables"></a>Variables d'environnement

Pour obtenir une variable d’environnement ou une valeur de paramètre d’application, utilisez `System.Environment.GetEnvironmentVariable`, comme illustré dans l’exemple de code suivant :

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    log.LogInformation(GetEnvironmentVariable("AzureWebJobsStorage"));
    log.LogInformation(GetEnvironmentVariable("WEBSITE_SITE_NAME"));
}

public static string GetEnvironmentVariable(string name)
{
    return name + ": " +
        System.Environment.GetEnvironmentVariable(name, EnvironmentVariableTarget.Process);
}
```

<a name="imperative-bindings"></a>

## <a name="binding-at-runtime"></a>Liaison au runtime

Avec C# et d’autres langages .NET, vous pouvez utiliser un schéma de liaison [impératif](https://en.wikipedia.org/wiki/Imperative_programming), par opposition aux liaisons [*déclaratives*](https://en.wikipedia.org/wiki/Declarative_programming) dans *function.json* . La liaison impérative est utile lorsque les paramètres de liaison doivent être calculés au moment du runtime plutôt que lors de la conception. Avec ce modèle, vous pouvez effectuer une liaison à la volée avec une liaison d’entrée et de sortie prise en charge dans le code de votre fonction.

Définissez une liaison impérative comme suit :

- **N’incluez pas** d’entrée dans *function.json* pour les liaisons impératives souhaitées.
- Transmettez un paramètre d’entrée [`Binder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Host/Bindings/Runtime/Binder.cs) ou [`IBinder binder`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IBinder.cs).
- Utilisez le modèle en C# suivant pour effectuer la liaison de données.

```cs
using (var output = await binder.BindAsync<T>(new BindingTypeAttribute(...)))
{
    ...
}
```

`BindingTypeAttribute` est l’attribut .NET qui définit votre liaison et `T` est le type d’entrée ou de sortie pris en charge par ce type de liaison. `T` ne peut pas être un type de paramètre `out` (comme `out JObject`). Par exemple, la liaison de sortie de la table Mobile Apps prend en charge [six types de sortie](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs#L17-L22), mais vous pouvez utiliser uniquement [ICollector\<T>](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/ICollector.cs) ou [`IAsyncCollector<T>`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/IAsyncCollector.cs) pour `T`.

### <a name="single-attribute-example"></a>Exemple d’attribut unique

L’exemple de code suivant crée une [liaison de sortie d’objet blob de stockage](functions-bindings-storage-blob-output.md) avec un chemin d’objet blob défini au moment de l’exécution, puis écrit une chaîne vers l’objet blob.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    using (var writer = await binder.BindAsync<TextWriter>(new BlobAttribute("samples-output/path")))
    {
        writer.Write("Hello World!!");
    }
}
```

[BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs) définit la liaison d’entrée ou de sortie de [l’objet blob de stockage](functions-bindings-storage-blob.md), et [TextWriter](/dotnet/api/system.io.textwriter) est un type de liaison de sortie pris en charge.

### <a name="multiple-attribute-example"></a>Exemple d’attributs multiples

L’exemple précédent obtient le paramètre d’application pour la chaîne de connexion du compte de stockage principal de l’application de fonction (à savoir `AzureWebJobsStorage`). Vous pouvez spécifier un paramètre d’application personnalisé à utiliser pour le compte de stockage en ajoutant l’attribut [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) et en transmettant le tableau d’attributs dans `BindAsync<T>()`. Utilisez un paramètre `Binder`, et non `IBinder`.  Par exemple :

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host.Bindings.Runtime;

public static async Task Run(string input, Binder binder)
{
    var attributes = new Attribute[]
    {
        new BlobAttribute("samples-output/path"),
        new StorageAccountAttribute("MyStorageAccount")
    };

    using (var writer = await binder.BindAsync<TextWriter>(attributes))
    {
        writer.Write("Hello World!");
    }
}
```

Le tableau suivant répertorie les attributs .NET pour chaque type de liaison et les packages dans lesquels ils sont définis.

> [!div class="mx-codeBreakAll"]
> | Liaison | Attribut | Ajouter la référence |
> |------|------|------|
> | Cosmos DB | [`Microsoft.Azure.WebJobs.DocumentDBAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.CosmosDB"` |
> | Event Hubs | [`Microsoft.Azure.WebJobs.ServiceBus.EventHubAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/v2.x/src/Microsoft.Azure.WebJobs.ServiceBus/EventHubs/EventHubAttribute.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.Jobs.ServiceBus"` |
> | Mobile Apps | [`Microsoft.Azure.WebJobs.MobileTableAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.MobileApps"` |
> | Notification Hubs | [`Microsoft.Azure.WebJobs.NotificationHubAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.NotificationHubs"` |
> | Service Bus | [`Microsoft.Azure.WebJobs.ServiceBusAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAttributeTests.cs), [`Microsoft.Azure.WebJobs.ServiceBusAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/b798412ad74ba97cf2d85487ae8479f277bdd85c/test/Microsoft.Azure.WebJobs.ServiceBus.UnitTests/ServiceBusAccountTests.cs) | `#r "Microsoft.Azure.WebJobs.ServiceBus"` |
> | File d’attente de stockage | [`Microsoft.Azure.WebJobs.QueueAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Objet blob de stockage | [`Microsoft.Azure.WebJobs.BlobAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Table de stockage | [`Microsoft.Azure.WebJobs.TableAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs), [`Microsoft.Azure.WebJobs.StorageAccountAttribute`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs) | |
> | Twilio | [`Microsoft.Azure.WebJobs.TwilioSmsAttribute`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs) | `#r "Microsoft.Azure.WebJobs.Extensions.Twilio"` |

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons](functions-triggers-bindings.md)

> [!div class="nextstepaction"]
> [En savoir plus sur les meilleures pratiques pour Azure Functions](functions-best-practices.md)
