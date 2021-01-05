---
title: Comment utiliser le kit SDK WebJobs
description: Découvrez comment écrire du code pour le Kit de développement logiciel (SDK) WebJobs. Créez des travaux de traitement en arrière-plan basé sur les événements, qui accèdent aux données dans Azure et des services tiers.
author: ggailey777
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: f941c394c3dab0e5e6997898a48a248f6a0cfe42
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352437"
---
# <a name="how-to-use-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>Comment utiliser le Kit de développement logiciel (SDK) Azure WebJobs pour le traitement en arrière-plan basé sur les événements

Cet article fournit de l’aide sur l’utilisation du kit SDK Azure WebJobs. Pour bien démarrer avec les WebJobs, consultez [Prise en main du kit SDK Azure WebJobs pour le traitement en arrière-plan basé sur les événements](webjobs-sdk-get-started.md). 

## <a name="webjobs-sdk-versions"></a>Versions du SDK WebJobs

Voici les différences clés entre la version 3.*x* et la version 2.*x* du kit SDK WebJobs :

* La version 3.*x* prend en charge le .NET Core.
* Dans la version 3.*x*, vous devez installer explicitement l’extension de liaison de stockage nécessaire au kit SDK WebJobs. Dans la version 2.*x*, les liaisons de stockage sont incluses dans le kit SDK.
* Les outils Visual Studio pour les projets .NET Core (3.*x*) diffèrent des outils pour les projets .NET Framework (2.*x*). Pour en savoir plus, consultez [Développer et déployer des WebJobs à l’aide de Visual Studio - Azure App Service](webjobs-dotnet-deploy-vs.md).

Quand cela est possible, des exemples sont fournis pour la version 3.*x* et la version 2.*x*.

> [!NOTE]
> [Azure Functions](../azure-functions/functions-overview.md) repose sur le kit SDK WebJobs, et cet article fournit des liens vers la documentation Azure Functions pour certaines rubriques. Notez les différences suivantes entre Functions et le kit SDK WebJobs :
> * Azure Functions version 2.*x* correspond au kit SDK WebJobs version 3.*x* et Azure Functions 1.*x* correspond au kit SDK WebJobs 2.*x*. Les dépôts de code source utilisent la numérotation du kit SDK WebJobs.
> * Un exemple de code des bibliothèques de classes C# Azure Functions est semblable au code du kit SDK WebJobs, à ceci près que vous n’avez pas besoin d’un attribut `FunctionName` dans un projet du kit SDK WebJobs.
> * Certains types de liaison, par exemple HTTP (Webhooks) et Event Grid (liaison basée sur HTTP), sont pris en charge uniquement dans Functions.
>
> Pour plus d’informations, consultez [Comparer Functions et WebJobs](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md#compare-functions-and-webjobs).

## <a name="webjobs-host"></a>Hôte de WebJobs

L’hôte est un conteneur d’exécution pour les fonctions.  Il écoute les déclencheurs et appelle les fonctions. Dans la version 3.*x*, l’hôte est une implémentation de `IHost`. Dans la version 2.*x*, vous utilisez l’objet `JobHost`. Vous créez une instance d’hôte dans votre code et écrivez du code pour personnaliser son comportement.

Il s’agit d’une différence clé entre l’utilisation directe du kit SDK WebJobs et son utilisation indirecte via Azure Functions. Dans Azure Functions, le service contrôle l’hôte. Vous ne pouvez pas personnaliser cet hôte en écrivant du code. Azure Functions vous permet de personnaliser le comportement de l’hôte via les paramètres du fichier host.json. Ces paramètres sont des chaînes, et non du code, ce qui limite les genres de personnalisation possibles.

### <a name="host-connection-strings"></a>Chaînes de connexion de l’hôte

Le kit SDK WebJobs recherche les chaînes de connexion du Stockage Azure et d’Azure Service Bus dans le fichier local.settings.json quand vous l’exécutez localement, ou dans l’environnement WebJob quand vous l’exécutez dans Azure. Par défaut, un paramètre de chaîne de connexion de stockage nommé `AzureWebJobsStorage` est obligatoire.  

La version 2.*x* du kit SDK vous permet d’utiliser vos propres noms pour ces chaînes de connexion, ou de les stocker ailleurs. Vous pouvez définir des noms dans le code à l’aide de [`JobHostConfiguration`], comme indiqué ici :

```cs
static void Main(string[] args)
{
    var _storageConn = ConfigurationManager
        .ConnectionStrings["MyStorageConnection"].ConnectionString;

    //// Dashboard logging is deprecated; use Application Insights.
    //var _dashboardConn = ConfigurationManager
    //    .ConnectionStrings["MyDashboardConnection"].ConnectionString;

    JobHostConfiguration config = new JobHostConfiguration();
    config.StorageConnectionString = _storageConn;
    //config.DashboardConnectionString = _dashboardConn;
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

> [!NOTE]
> Dans la mesure où la version 3.*x* utilise les API de configuration .NET Core par défaut, aucune API ne permet de changer les noms de chaînes de connexion. Consultez [Développement et déploiement de WebJobs avec Visual Studio](webjobs-dotnet-deploy-vs.md).

### <a name="host-development-settings"></a>Paramètres de développement de l’hôte

Vous pouvez exécuter l’hôte en mode de développement pour améliorer l’efficacité du développement local. Voici certains des paramètres qui changent durant l’exécution en mode de développement :

| Propriété | Paramètre de développement |
| ------------- | ------------- |
| `Tracing.ConsoleLevel` | `TraceLevel.Verbose` pour optimiser la sortie de la journalisation. |
| `Queues.MaxPollingInterval`  | Une valeur faible pour garantir le déclenchement immédiat des méthodes de file d’attente.  |
| `Singleton.ListenerLockPeriod` | 15 secondes pour favoriser le développement itératif rapide. |

Le processus d’activation du mode de développement dépend de la version du kit SDK. 

#### <a name="version-3x"></a>Version 3.*x*

La version 3.*x* utilise les API ASP.NET Core standard. Appelez la méthode [`UseEnvironment`](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.useenvironment) sur l’instance [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Passez une chaîne nommée `development`, comme dans cet exemple :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.UseEnvironment("development");
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

La classe `JobHostConfiguration` dispose d’une méthode `UseDevelopmentSettings` qui active le mode de développement.  L’exemple suivant montre comment utiliser les paramètres de développement. Pour que `config.IsDevelopment` retourne `true` quand il s’exécute localement, définissez une variable d’environnement locale nommée `AzureWebJobsEnv` avec la valeur `Development`.

```cs
static void Main()
{
    config = new JobHostConfiguration();

    if (config.IsDevelopment)
    {
        config.UseDevelopmentSettings();
    }

    var host = new JobHost(config);
    host.RunAndBlock();
}
```

### <a name="managing-concurrent-connections-version-2x"></a><a name="jobhost-servicepointmanager-settings"></a>Gestion des connexions simultanées (version 2.*x*)

Dans la version 3.*x*, la limite du nombre de connexions par défaut est infinie. Si, pour une raison quelconque, vous devez changer cette limite, vous pouvez utiliser la propriété [`MaxConnectionsPerServer`](/dotnet/api/system.net.http.winhttphandler.maxconnectionsperserver) de la classe [`WinHttpHandler`](/dotnet/api/system.net.http.winhttphandler).

Dans la version 2.*x*, vous contrôlez le nombre de connexions simultanées à un hôte à l’aide de l’API [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit#System_Net_ServicePointManager_DefaultConnectionLimit). Dans la version 2.*x*, vous devez augmenter cette valeur par défaut de 2 avant de démarrer votre hôte WebJobs.

Toutes les requêtes HTTP sortantes que vous effectuez à partir d’une fonction à l’aide de `HttpClient` passent par `ServicePointManager`. Une fois que vous avez atteint la valeur définie dans `DefaultConnectionLimit`, `ServicePointManager` commence à mettre les requêtes en file d’attente avant de les envoyer. Supposons que votre limite `DefaultConnectionLimit` est définie sur 2 et que votre code effectue 1 000 requêtes HTTP. Au départ, seulement deux requêtes sont autorisées à transiter jusqu’au système d’exploitation. Les 998 autres sont mises en file d’attente jusqu’à ce que de la place se libère pour elles. Cela signifie que votre `HttpClient` risque d’arriver à expiration, car il donne l’impression d’avoir effectué la requête, à ceci près que le système d’exploitation ne l’a jamais envoyée au serveur de destination. Par conséquent, il se peut que vous observiez un comportement qui semble illogique : votre `HttpClient` local met 10 secondes pour effectuer une requête, mais votre service renvoie chaque requête en 200 ms. 

Pour les applications ASP.NET, la valeur par défaut est `Int32.MaxValue`, une valeur qui a de grandes chances de convenir aux WebJobs exécutés dans un plan App Service De base ou supérieur. Les WebJobs ont généralement besoin du paramètre Always On, qui est uniquement pris en charge par les plans App Service De base et supérieurs.

Si votre tâche web est exécutée dans un plan App Service Gratuit ou Partagé, votre application est limitée par le bac à sable App Service, qui présente actuellement une [limite de connexion de 300](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#per-sandbox-per-appper-site-numerical-limits). Avec une limite de connexion indépendante dans `ServicePointManager`, il existe une plus grande probabilité que le seuil de connexion en bac à sable soit atteint et que le site soit arrêté. Dans ce cas, le fait de définir `DefaultConnectionLimit` sur une valeur inférieure, par exemple 50 ou 100, peut empêcher ce problème tout en assurant un débit suffisant.

Le paramètre doit être configuré avant d’effectuer toute requête HTTP. C’est la raison pour laquelle l’hôte WebJobs ne doit pas ajuster le paramètre automatiquement. Des requêtes HTTP peuvent se produire avant le démarrage de l’hôte, ce qui peut entraîner un comportement inattendu. La meilleure approche consiste à définir la valeur immédiatement dans votre méthode `Main` avant d’initialiser `JobHost`, comme indiqué ici :

```csharp
static void Main(string[] args)
{
    // Set this immediately so that it's used by all requests.
    ServicePointManager.DefaultConnectionLimit = Int32.MaxValue;

    var host = new JobHost();
    host.RunAndBlock();
}
```

## <a name="triggers"></a>Déclencheurs

Les fonctions doivent être des méthodes publiques et doivent avoir un attribut de déclencheur ou l’attribut [`NoAutomaticTrigger`](#manual-triggers).

### <a name="automatic-triggers"></a>Déclencheurs automatiques

Les déclencheurs automatiques appellent une fonction en réponse à un événement. Prenons l’exemple d’une fonction déclenchée par un message ajouté au Stockage File d’attente Azure. Elle répond en lisant un objet blob provenant du Stockage Blob Azure :

```cs
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

L’attribut `QueueTrigger` indique au runtime d’appeler la fonction chaque fois qu’un message de file d’attente apparaît dans la file d’attente `myqueue-items`. L’attribut `Blob` indique au runtime d’utiliser le message de file d’attente pour lire un objet blob dans le conteneur *sample-workitems*. Le nom de l’élément d’objet BLOB dans le conteneur `samples-workitems` est obtenu directement à partir du déclencheur de file d’attente en tant qu’expression de liaison (`{queueTrigger}`).

[!INCLUDE [webjobs-always-on-note](../../includes/webjobs-always-on-note.md)]

### <a name="manual-triggers"></a>Déclencheurs manuels

Pour déclencher une fonction manuellement, utilisez l’attribut `NoAutomaticTrigger`, comme indiqué ici :

```cs
[NoAutomaticTrigger]
public static void CreateQueueMessage(
ILogger logger,
string value,
[Queue("outputqueue")] out string message)
{
    message = value;
    logger.LogInformation("Creating queue message: ", message);
}
```

Le processus de déclenchement manuel de la fonction dépend de la version du kit SDK.

#### <a name="version-3x"></a>Version 3.*x*

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddAzureStorage();
    });
    var host = builder.Build();
    using (host)
    {
        var jobHost = host.Services.GetService(typeof(IJobHost)) as JobHost;
        var inputs = new Dictionary<string, object>
        {
            { "value", "Hello world!" }
        };

        await host.StartAsync();
        await jobHost.CallAsync("CreateQueueMessage", inputs);
        await host.StopAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

```cs
static void Main(string[] args)
{
    JobHost host = new JobHost();
    host.Call(typeof(Program).GetMethod("CreateQueueMessage"), new { value = "Hello world!" });
}
```

## <a name="input-and-output-bindings"></a>Liaisons d’entrée et de sortie

Les liaisons d’entrée fournissent une méthode déclarative pour rendre disponibles les données des services Azure ou tiers pour votre code. Les liaisons de sortie offrent un moyen de mettre à jour des données. L’[article de prise en main](webjobs-sdk-get-started.md) propose un exemple pour chaque type de liaison.

Vous pouvez utiliser une valeur de retour de méthode pour une liaison de sortie en appliquant l’attribut à la valeur de retour de méthode. Consultez l’exemple dans [Utilisation de la valeur de retour Azure Functions](../azure-functions/functions-bindings-return-value.md).

## <a name="binding-types"></a>Types de liaisons

Le processus d’installation et de gestion des types de liaison varie selon que vous utilisez la version 3.*x* ou la version 2.*x* du kit SDK. Vous pouvez trouver le package à installer pour un type de liaison spécifique dans la section « Packages » de l’[article de référence](#binding-reference-information) Azure Functions de ce type de liaison. L’absence de prise en charge par Azure Functions du déclencheur et de la liaison Files (pour le système de fichiers local) constitue une exception.

#### <a name="version-3x"></a>Version 3.*x*

Dans la version 3.*x*, les liaisons de stockage sont incluses dans le package `Microsoft.Azure.WebJobs.Extensions.Storage`. Appelez la méthode d’extension `AddAzureStorage` dans la méthode `ConfigureWebJobs`, comme indiqué ici :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddAzureStorage();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour utiliser d’autres types de liaisons et de déclencheurs, installez le package NuGet qui les contient et appelez la méthode d’extension `Add<binding>` implémentée dans l’extension. Par exemple, si vous souhaitez utiliser une liaison Azure Cosmos DB, installez `Microsoft.Azure.WebJobs.Extensions.CosmosDB` et appelez `AddCosmosDB`, comme suit :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddCosmosDB();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour utiliser le déclencheur de minuteur ou la liaison Files, qui font partie des services principaux, appelez les méthodes d’extension `AddTimers` ou `AddFiles`, respectivement.

#### <a name="version-2x"></a>Version 2.*x*

Ces types de déclencheur et de liaison sont inclus dans la version 2.*x* du package `Microsoft.Azure.WebJobs` :

* Stockage d'objets blob
* Stockage de files d'attente
* Stockage de tables

Pour utiliser d’autres types de liaisons et de déclencheurs, installez le package NuGet qui les contient et appelez une méthode `Use<binding>` sur l’objet `JobHostConfiguration`. Par exemple, si vous souhaitez utiliser un déclencheur Timer, installez `Microsoft.Azure.WebJobs.Extensions` et appelez `UseTimers` dans la méthode `Main`, comme indiqué ici :

```cs
static void Main()
{
    config = new JobHostConfiguration();
    config.UseTimers();
    var host = new JobHost(config);
    host.RunAndBlock();
}
```

Pour utiliser la liaison Files, installez `Microsoft.Azure.WebJobs.Extensions` et appelez `UseFiles`.

### <a name="executioncontext"></a>ExecutionContext

WebJobs vous permet de lier à [`ExecutionContext`]. Avec cette liaison, vous pouvez accéder à [`ExecutionContext`] en tant que paramètre dans votre signature de fonction. Par exemple, le code suivant utilise l’objet de contexte pour accéder à l’ID d’appel, ce que vous pouvez utiliser pour mettre en corrélation tous les journaux d’activité produits par l’appel d’une fonction donnée.  

```cs
public class Functions
{
    public static void ProcessQueueMessage([QueueTrigger("queue")] string message,
        ExecutionContext executionContext,
        ILogger logger)
    {
        logger.LogInformation($"{message}\n{executionContext.InvocationId}");
    }
}
```

Le processus de liaison à [`ExecutionContext`] dépend de la version de votre kit SDK.

#### <a name="version-3x"></a>Version 3.*x*

Appelez la méthode d’extension `AddExecutionContextBinding` dans la méthode `ConfigureWebJobs`, comme indiqué ici :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddExecutionContextBinding();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

Le package `Microsoft.Azure.WebJobs.Extensions` mentionné précédemment fournit également un type de liaison spécial que vous pouvez enregistrer en appelant la méthode `UseCore`. Cette liaison vous permet de définir un paramètre [`ExecutionContext`] dans votre signature de fonction, ce qui est rendu possible comme suit :

```cs
class Program
{
    static void Main()
    {
        config = new JobHostConfiguration();
        config.UseCore();
        var host = new JobHost(config);
        host.RunAndBlock();
    }
}
```

## <a name="binding-configuration"></a>Configuration de liaison

Vous pouvez configurer le comportement de certains déclencheurs et de certaines liaisons. Leur processus de configuration dépend de la version du kit SDK.

* **Version 3.*x*:** Définissez la configuration quand la méthode `Add<Binding>` est appelée dans `ConfigureWebJobs`.
* **Version 2.*x*:** Définissez la configuration en définissant des propriétés dans un objet de configuration que vous passez à `JobHost`.

Ces paramètres spécifiques à la liaison sont équivalents aux paramètres du [fichier projet host.json](../azure-functions/functions-host-json.md) dans Azure Functions.

Vous pouvez configurer les liaisons suivantes :

* [Déclencheur Azure Cosmos DB](#azure-cosmosdb-trigger-configuration-version-3x)
* [Déclencheur Event Hubs](#event-hubs-trigger-configuration-version-3x)
* [Déclencheur de stockage de file d’attente](#queue-storage-trigger-configuration)
* [Liaison SendGrid](#sendgrid-binding-configuration-version-3x)
* [Déclencheur Service Bus](#service-bus-trigger-configuration-version-3x)

### <a name="azure-cosmosdb-trigger-configuration-version-3x"></a>Configuration du déclencheur Azure Cosmos DB (version 3.*x*)

Cet exemple montre comment configurer le déclencheur Azure Cosmos DB :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddCosmosDB(a =>
        {
            a.ConnectionMode = ConnectionMode.Gateway;
            a.Protocol = Protocol.Https;
            a.LeaseOptions.LeasePrefix = "prefix1";

        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour plus d’informations, consultez l’article sur la [liaison Azure Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-output.md#hostjson-settings).

### <a name="event-hubs-trigger-configuration-version-3x"></a>Configuration du déclencheur Event Hubs (version 3.*x*)

Cet exemple montre comment configurer le déclencheur Event Hubs :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddEventHubs(a =>
        {
            a.BatchCheckpointFrequency = 5;
            a.EventProcessorOptions.MaxBatchSize = 256;
            a.EventProcessorOptions.PrefetchCount = 512;
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour plus d’informations, consultez l’article sur la [liaison Event Hubs](../azure-functions/functions-bindings-event-hubs-trigger.md#host-json).

### <a name="queue-storage-trigger-configuration"></a>Configuration du déclencheur Stockage File d’attente

Ces exemples montrent comment configurer le déclencheur Stockage File d’attente :

#### <a name="version-3x"></a>Version 3.*x*

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddAzureStorage(a => {
            a.BatchSize = 8;
            a.NewBatchThreshold = 4;
            a.MaxDequeueCount = 4;
            a.MaxPollingInterval = TimeSpan.FromSeconds(15);
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour plus d’informations, consultez l’article sur la [liaison Stockage File d’attente](../azure-functions/functions-bindings-storage-queue-trigger.md#hostjson-properties).

#### <a name="version-2x"></a>Version 2.*x*

```cs
static void Main(string[] args)
{
    JobHostConfiguration config = new JobHostConfiguration();
    config.Queues.BatchSize = 8;
    config.Queues.NewBatchThreshold = 4;
    config.Queues.MaxDequeueCount = 4;
    config.Queues.MaxPollingInterval = TimeSpan.FromSeconds(15);
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

Pour plus d’informations, consultez les [informations de référence sur host.json v1.x](../azure-functions/functions-host-json-v1.md#queues).

### <a name="sendgrid-binding-configuration-version-3x"></a>Configuration de la liaison SendGrid (version 3.*x*)

Cet exemple montre comment configurer la liaison de sortie SendGrid :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddSendGrid(a =>
        {
            a.FromAddress.Email = "samples@functions.com";
            a.FromAddress.Name = "Azure Functions";
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour plus d’informations, consultez l’article sur la [liaison SendGrid](../azure-functions/functions-bindings-sendgrid.md#hostjson-settings).

### <a name="service-bus-trigger-configuration-version-3x"></a>Configuration du déclencheur Service Bus (version 3.*x*)

Cet exemple montre comment configurer le déclencheur Service Bus :

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddServiceBus(sbOptions =>
        {
            sbOptions.MessageHandlerOptions.AutoComplete = true;
            sbOptions.MessageHandlerOptions.MaxConcurrentCalls = 16;
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Pour plus d’informations, consultez l’article sur la [liaison Service Bus](../azure-functions/functions-bindings-service-bus-output.md#hostjson-settings).

### <a name="configuration-for-other-bindings"></a>Configuration pour les autres liaisons

Certains types de déclencheur et de liaison définissent leurs propres types de configuration personnalisés. Par exemple, le déclencheur Files vous permet de spécifier le chemin racine à superviser, comme dans les exemples suivants :

#### <a name="version-3x"></a>Version 3.*x*

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddFiles(a => a.RootPath = @"c:\data\import");
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

```cs
static void Main()
{
    config = new JobHostConfiguration();
    var filesConfig = new FilesConfiguration
    {
        RootPath = @"c:\data\import"
    };
    config.UseFiles(filesConfig);
    var host = new JobHost(config);
    host.RunAndBlock();
}
```

## <a name="binding-expressions"></a>Expressions de liaison

Dans les paramètres de constructeur d’attribut, vous pouvez utiliser des expressions qui sont remplacées par des valeurs provenant de diverses sources. Par exemple, dans le code suivant, le chemin d’accès pour l’attribut `BlobTrigger` crée une expression nommée `filename`. Lorsqu’elle est utilisée pour la liaison de sortie, l’expression `filename` est remplacée par le nom de l’objet blob déclencheur.

```cs
public static void CreateThumbnail(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    ILogger logger)
{
    logger.Info($"Blob trigger processing: {filename}");
    // ...
}
```

Pour plus d’informations sur les expressions de liaison, consultez [Modèles et expressions de liaison](../azure-functions/functions-bindings-expressions-patterns.md) dans la documentation Azure Functions.

### <a name="custom-binding-expressions"></a>Expressions de liaison personnalisées

Dans certains cas, vous souhaitez spécifier un nom de file d’attente, un nom d’objet blob ou de conteneur, ou encore un nom de table, dans le code plutôt que effectuer un codage de manière irréversible. Par exemple, il se peut que vous souhaitiez spécifier le nom de la file d’attente de l’attribut `QueueTrigger` dans une variable d’environnement ou un fichier de configuration.

Vous pouvez le faire en passant un objet `NameResolver` à l’objet `JobHostConfiguration`. Incluez des espaces réservés dans les paramètres de constructeur d’attribut pour que le code de votre objet `NameResolver` fournisse les valeurs réelles à utiliser à la place de ces espaces réservés. Pour identifier les espaces réservés, placez-les entre des symboles de pourcentage (%), comme indiqué ici :

```cs
public static void WriteLog([QueueTrigger("%logqueue%")] string logMessage)
{
    Console.WriteLine(logMessage);
}
```

Ce code vous permet d’utiliser une file d’attente nommée `logqueuetest` dans l’environnement de test, et une autre nommée `logqueueprod` en production. Au lieu d’un nom de file d’attente codé en dur, vous spécifiez le nom d’une entrée dans la collection `appSettings`.

Un `NameResolver` est utilisé par défaut si vous n’en fournissez pas un personnalisé. L’objet par défaut obtient les valeurs à partir des paramètres d’application ou des variables d’environnement.

Votre classe `NameResolver` obtient le nom de la file d’attente à partir de `appSettings`, comme indiqué ici :

```cs
public class CustomNameResolver : INameResolver
{
    public string Resolve(string name)
    {
        return ConfigurationManager.AppSettings[name].ToString();
    }
}
```

#### <a name="version-3x"></a>Version 3.*x*

Vous configurez le programme de résolution à l’aide de l’injection de dépendances. Ces exemples nécessitent l’instruction `using` suivante :

```cs
using Microsoft.Extensions.DependencyInjection;
```

Vous ajoutez le programme de résolution en appelant la méthode d’extension [`ConfigureServices`] sur [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder), comme dans l’exemple suivant :

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    var resolver = new CustomNameResolver();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureServices(s => s.AddSingleton<INameResolver>(resolver));
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

Passez votre classe `NameResolver` à l’objet `JobHost`, comme indiqué ici :

```cs
 static void Main(string[] args)
{
    JobHostConfiguration config = new JobHostConfiguration();
    config.NameResolver = new CustomNameResolver();
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

Azure Functions implémente `INameResolver` pour obtenir les valeurs à partir des paramètres d’application, comme illustré dans l’exemple. Quand vous utilisez le Kit de développement logiciel (SDK) WebJobs directement, vous pouvez écrire une implémentation personnalisée qui obtient les valeurs de remplacement des espaces réservés à partir de la source de votre choix.

## <a name="binding-at-runtime"></a>Liaison au runtime

Si vous devez effectuer une tâche dans votre fonction avant d’utiliser un attribut de liaison tel que `Queue`, `Blob`, ou `Table`, vous pouvez utiliser l’interface `IBinder`.

L’exemple suivant prend un message en file d’attente d’entrée et crée un message avec le même contenu dans une file d’attente de sortie. Le nom de file d’attente de sortie est défini par le code dans le corps de la fonction.

```cs
public static void CreateQueueMessage(
    [QueueTrigger("inputqueue")] string queueMessage,
    IBinder binder)
{
    string outputQueueName = "outputqueue" + DateTime.Now.Month.ToString();
    QueueAttribute queueAttribute = new QueueAttribute(outputQueueName);
    CloudQueue outputQueue = binder.Bind<CloudQueue>(queueAttribute);
    outputQueue.AddMessageAsync(new CloudQueueMessage(queueMessage));
}
```

Pour plus d’informations, consultez [Liaison au runtime](../azure-functions/functions-dotnet-class-library.md#binding-at-runtime) dans la documentation Azure Functions.

## <a name="binding-reference-information"></a>Informations de référence sur les liaisons

La documentation Azure Functions fournit des informations de référence sur chaque type de liaison. Vous trouverez les informations suivantes dans chaque article de référence sur les liaisons. (Cet exemple est basé sur la file d’attente de Stockage.)

* [Packages](../azure-functions/functions-bindings-storage-queue.md). Package à installer pour inclure la prise en charge de la liaison dans un projet du kit SDK WebJobs.
* [Exemples](../azure-functions/functions-bindings-storage-queue-trigger.md). Exemples de code. L’exemple de bibliothèque de classe C# s’applique au kit SDK WebJobs. Omettez simplement l’attribut `FunctionName`.
* [Attributs](../azure-functions/functions-bindings-storage-queue-trigger.md#attributes-and-annotations). Attributs à utiliser pour le type de liaison.
* [Configuration](../azure-functions/functions-bindings-storage-queue-trigger.md#configuration). Explications relatives aux propriétés d’attribut et aux paramètres de constructeur.
* [Utilisation](../azure-functions/functions-bindings-storage-queue-trigger.md#usage). Types avec lesquels vous pouvez effectuer une liaison et informations sur le fonctionnement de la liaison. Exemple : algorithme d’interrogation, traitement de file d’attente de messages incohérents.
  
Pour obtenir la liste des articles de référence sur les liaisons, consultez « Liaisons prises en charge » dans l’article [Déclencheurs et liaisons](../azure-functions/functions-triggers-bindings.md#supported-bindings) pour Azure Functions. Dans cette liste, les liaisons HTTP, Webhooks et Event Grid sont prises en charge uniquement par Azure Functions, et pas par le kit SDK WebJobs.

## <a name="disable-attribute"></a>Attribut Disable 

L’attribut [`Disable`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/DisableAttribute.cs) vous permet de contrôler si une fonction peut être déclenchée. 

Dans l’exemple suivant, si le paramètre d’application `Disable_TestJob` a la valeur `1` ou `True` (non-respect de la casse), la fonction ne s’exécute pas. Dans ce cas, le runtime crée le message de journal *Function ’Functions.TestJob’ is disabled* (La fonction « Functions.TestJob » est désactivée).

```cs
[Disable("Disable_TestJob")]
public static void TestJob([QueueTrigger("testqueue2")] string message)
{
    Console.WriteLine("Function with Disable attribute executed!");
}
```

Quand vous changez les valeurs des paramètres d’application dans le portail Azure, le WebJob redémarre pour récupérer le nouveau paramètre.

L’attribut peut être déclaré au niveau du paramètre, de la méthode ou de la classe. Le nom du paramètre peut également contenir des expressions de liaison.

## <a name="timeout-attribute"></a>Attribut Timeout

L’attribut [`Timeout`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/TimeoutAttribute.cs) entraîne l’annulation d’une fonction, si son exécution ne s’achève pas dans le délai spécifié. Dans l’exemple suivant, sans l’attribut Timeout, la fonction s’exécute pendant une journée. L’attribut Timeout entraîne l’annulation de la fonction au-delà de 15 secondes.

```cs
[Timeout("00:00:15")]
public static async Task TimeoutJob(
    [QueueTrigger("testqueue2")] string message,
    CancellationToken token,
    TextWriter log)
{
    await log.WriteLineAsync("Job starting");
    await Task.Delay(TimeSpan.FromDays(1), token);
    await log.WriteLineAsync("Job completed");
}
```

Vous pouvez appliquer l’attribut Timeout au niveau de la classe ou de la méthode, et vous pouvez spécifier un délai d’expiration global à l’aide de `JobHostConfiguration.FunctionTimeout`. Les délais d’expiration au niveau de la classe ou de la méthode remplacent les délais d’expiration globaux.

## <a name="singleton-attribute"></a>Attribut Singleton

L’attribut [`Singleton`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/SingletonAttribute.cs) permet de vérifier qu’une seule instance d’une fonction s’exécute, même s’il existe plusieurs instances de l’application web hôte. Pour cela, il utilise le [verrouillage distribué](#viewing-lease-blobs).

Dans l’exemple suivant, une seule instance de la fonction `ProcessImage` s’exécute à un moment donné :

```cs
[Singleton]
public static async Task ProcessImage([BlobTrigger("images")] Stream image)
{
     // Process the image.
}
```

### <a name="singletonmodelistener"></a>SingletonMode.Listener

Certains déclencheurs intègrent la prise en charge de la gestion de l’accès concurrentiel :

* **QueueTrigger**. Définissez `JobHostConfiguration.Queues.BatchSize` sur `1`.
* **ServiceBusTrigger**. Définissez `ServiceBusConfiguration.MessageOptions.MaxConcurrentCalls` sur `1`.
* **FileTrigger**. Définissez `FileProcessor.MaxDegreeOfParallelism` sur `1`.

Vous pouvez utiliser ces paramètres pour vous assurer que votre fonction s’exécute en tant que singleton sur une instance unique. Pour vérifier qu’une seule instance de la fonction s’exécute quand l’application web est étendue à plusieurs instances, appliquez un verrou singleton au niveau de l’écouteur sur la fonction (`[Singleton(Mode = SingletonMode.Listener)]`). Les verrous d’écouteurs sont acquis au démarrage du JobHost. Si trois instances scale-out démarrent en même temps, une seule de ces instances acquiert le verrou et un seul écouteur démarre.

> [!NOTE]
> Pour en savoir plus sur le fonctionnement de SingletonMode.Function, consultez ce [dépôt GitHub](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/SingletonMode.cs).

### <a name="scope-values"></a>Valeurs d’étendue

Vous pouvez spécifier une *expression/valeur d’étendue* sur un singleton. L’expression/La valeur permet de vérifier que toutes les exécutions de la fonction dans une étendue spécifique sont sérialisées. L’implémentation d’un verrouillage plus granulaire peut ainsi permettre un certain niveau de parallélisme pour votre fonction, tout en sérialisant d’autres appels conformément à vos besoins. Par exemple, dans le code suivant, l’expression d’étendue est liée à la valeur `Region` du message entrant. Quand la file d’attente contient trois messages dans les régions Est, Est et Ouest respectivement, les messages pour la région Est sont exécutés en série pendant que le message pour la région Ouest est exécuté en parallèle à ceux de la région Est.

```csharp
[Singleton("{Region}")]
public static async Task ProcessWorkItem([QueueTrigger("workitems")] WorkItem workItem)
{
     // Process the work item.
}

public class WorkItem
{
     public int ID { get; set; }
     public string Region { get; set; }
     public int Category { get; set; }
     public string Description { get; set; }
}
```

### <a name="singletonscopehost"></a>SingletonScope.Host

L’étendue par défaut d’un verrou est `SingletonScope.Function`, ce qui signifie que l’étendue du verrouillage (le chemin du bail des objets blob) est liée au nom de fonction complet. Pour verrouiller des fonctions, spécifiez `SingletonScope.Host` et utilisez un nom d’ID d’étendue identique pour toutes les fonctions qui ne doivent pas être exécutées simultanément. Dans l’exemple suivant, seule une instance de `AddItem` ou `RemoveItem` est exécutée à la fois :

```csharp
[Singleton("ItemsLock", SingletonScope.Host)]
public static void AddItem([QueueTrigger("add-item")] string message)
{
     // Perform the add operation.
}

[Singleton("ItemsLock", SingletonScope.Host)]
public static void RemoveItem([QueueTrigger("remove-item")] string message)
{
     // Perform the remove operation.
}
```

### <a name="viewing-lease-blobs"></a>Affichage des baux d’objets blob

Le Kit de développement logiciel (SDK) WebJobs utilise des [baux d’objets blob Azure](../storage/common/storage-concurrency.md#pessimistic-concurrency-for-blobs) en arrière-plan pour implémenter le verrouillage distribué. Les objets blob de bail utilisés par l’attribut Singleton se trouvent dans le conteneur `azure-webjobs-host` du compte de stockage `AzureWebJobsStorage` dans le chemin « locks ». Par exemple, le chemin d’accès du bail d’objet blob pour le premier exemple `ProcessImage` illustré précédemment peut être `locks/061851c758f04938a4426aa9ab3869c0/WebJobs.Functions.ProcessImage`. Tous les chemins d’accès incluent l’ID JobHost, dans le cas présent 061851c758f04938a4426aa9ab3869c0.

## <a name="async-functions"></a>Fonctions asynchrones

Pour plus d’informations sur le codage de fonctions async, consultez la [documentation Azure Functions](../azure-functions/functions-dotnet-class-library.md#async).

## <a name="cancellation-tokens"></a>Jetons d’annulation

Pour plus d’informations sur la gestion des jetons d’annulation, consultez la documentation Azure Functions sur [les jetons d’annulation et l’arrêt approprié](../azure-functions/functions-dotnet-class-library.md#cancellation-tokens).

## <a name="multiple-instances"></a>Instances multiples

Si votre application web s’exécute sur plusieurs instances, une tâche web continue se lance sur chaque instance, écoutant les déclencheurs et appelant les fonctions. Les différentes liaisons de déclencheur sont conçues pour partager efficacement le travail de façon collaborative entre les instances afin que l’extension à d’autres instances vous permette de gérer une charge plus importante.

Contrairement à certains déclencheurs qui peuvent entraîner un double traitement, les déclencheurs de stockage File d'attente et Blob empêchent automatiquement une fonction de traiter plusieurs fois un objet blob ou un message en file d'attente. Pour plus d'informations, consultez [Conception pour une entrée identique](../azure-functions/functions-idempotent.md) dans la documentation Azure Functions.

Le déclencheur timer garantit automatiquement l’exécution d’une seule instance du minuteur de sorte qu’une instance unique de la fonction soit exécutée à une heure planifiée donnée.

Si vous souhaitez vérifier qu’une seule instance d’une fonction s’exécute même quand il existe plusieurs instances de l’application web hôte, vous pouvez utiliser l’attribut [`Singleton`](#singleton-attribute).

## <a name="filters"></a>Filtres

Les filtres de fonctions (préversion) vous offrent un moyen de personnaliser le pipeline d’exécution de tâches web avec votre propre logique. Ces filtres sont similaires aux [filtres ASP.NET Core](/aspnet/core/mvc/controllers/filters). Vous pouvez les implémenter en tant qu’attributs déclaratifs appliqués à vos fonctions ou classes. Pour plus d’informations, consultez [Function Filters](https://github.com/Azure/azure-webjobs-sdk/wiki/Function-Filters) (Filtres de fonctions).

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

Nous vous recommandons le framework de journalisation développé pour ASP.NET. L’[article de prise en main](webjobs-sdk-get-started.md) montre comment l’utiliser. 

### <a name="log-filtering"></a>Filtrage de journal

Chaque journal créé par une instance `ILogger` présente des attributs `Category` et `Level` associés. [`LogLevel`](/dotnet/api/microsoft.extensions.logging.loglevel) est une énumération, et le code d’entier indique l’importance relative :

|LogLevel    |Code|
|------------|---|
|Trace       | 0 |
|Débogage       | 1 |
|Information | 2 |
|Avertissement     | 3 |
|Error       | 4 |
|Critique    | 5 |
|None        | 6 |

Vous pouvez filtrer indépendamment chaque catégorie en fonction d’un [`LogLevel`](/dotnet/api/microsoft.extensions.logging.loglevel) spécifique. Par exemple, il se peut que vous souhaitiez afficher tous les journaux d’activité pour le traitement de déclencheurs blob, mais uniquement les niveaux `Error` et plus élevés pour tous les autres éléments.

#### <a name="version-3x"></a>Version 3.*x*

La version 3.*x* du kit SDK repose sur le filtrage intégré à .NET Core. La classe `LogCategories` vous permet de définir des catégories pour des fonctions, des déclencheurs ou des utilisateurs spécifiques. Elle définit également des filtres pour des états d’hôte spécifiques, par exemple `Startup` et `Results`. Cela vous permet d’affiner la sortie de la journalisation. Si aucune correspondance n’est trouvée dans les catégories définies, le filtre se replie sur la valeur `Default` pour décider si le message doit être filtré ou non.

`LogCategories` nécessite l’instruction using suivante :

```cs
using Microsoft.Azure.WebJobs.Logging; 
```

L’exemple suivant permet de créer un filtre qui filtre par défaut tous les journaux au niveau `Warning`. Les catégories `Function` et `results` (équivalentes à `Host.Results` dans la version 2.*x*) sont filtrées au niveau `Error`. Le filtre compare la catégorie actuelle à tous les niveaux enregistrés dans l’instance `LogCategories`, et choisit la correspondance la plus longue. Cela signifie que le niveau `Debug` inscrit pour `Host.Triggers` correspond à `Host.Triggers.Queue` ou `Host.Triggers.Blob`. Cela vous permet de contrôler des catégories plus vastes sans avoir à ajouter chacune d’entre elles.

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureLogging(logging =>
            {
                logging.SetMinimumLevel(LogLevel.Warning);
                logging.AddFilter("Function", LogLevel.Error);
                logging.AddFilter(LogCategories.CreateFunctionCategory("MySpecificFunctionName"),
                    LogLevel.Debug);
                logging.AddFilter(LogCategories.Results, LogLevel.Error);
                logging.AddFilter("Host.Triggers", LogLevel.Debug);
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Version 2.*x*

Dans la version 2.*x* du kit SDK, vous utilisez la classe `LogCategoryFilter` pour contrôler le filtrage. `LogCategoryFilter` a une propriété `Default` ayant la valeur initiale `Information`, ce qui signifie que tous les messages des niveaux `Information`, `Warning`, `Error` ou `Critical` sont journalisés, mais que tous les messages des niveaux `Debug` ou `Trace` sont filtrés.

Comme pour `LogCategories` dans la version 3.*x*, la propriété `CategoryLevels` permet de spécifier des niveaux de consignation pour des catégories spécifiques, afin d’affiner la sortie de la journalisation. Si aucune correspondance n’est trouvée dans le dictionnaire `CategoryLevels`, le filtre se replie sur la valeur `Default` pour décider si le message doit être filtré ou non.

L’exemple suivant crée un filtre qui filtre par défaut tous les journaux d’activité au niveau `Warning`. Les catégories `Function` et `Host.Results` sont filtrées au niveau `Error`. L’objet `LogCategoryFilter` compare la catégorie actuelle à toutes les propriétés `CategoryLevels` enregistrées et choisit la correspondance la plus longue. Ainsi, le niveau `Debug` inscrit pour `Host.Triggers` correspond à `Host.Triggers.Queue` ou `Host.Triggers.Blob`. Cela vous permet de contrôler des catégories plus vastes sans avoir à ajouter chacune d’entre elles.

```csharp
var filter = new LogCategoryFilter();
filter.DefaultLevel = LogLevel.Warning;
filter.CategoryLevels[LogCategories.Function] = LogLevel.Error;
filter.CategoryLevels[LogCategories.Results] = LogLevel.Error;
filter.CategoryLevels["Host.Triggers"] = LogLevel.Debug;

config.LoggerFactory = new LoggerFactory()
    .AddApplicationInsights(instrumentationKey, filter.Filter)
    .AddConsole(filter.Filter);
```

### <a name="custom-telemetry-for-application-insights"></a>Télémétrie personnalisée pour Application Insights​

Le processus d’implémentation de la télémétrie personnalisée pour [Application Insights](../azure-monitor/app/app-insights-overview.md) dépend de la version du kit SDK. Pour savoir comment configurer Application Insights, consultez [Ajouter la journalisation Application Insights](webjobs-sdk-get-started.md#add-application-insights-logging).

#### <a name="version-3x"></a>Version 3.*x*

Dans la mesure où la version 3.*x* du kit SDK WebJobs repose sur l’hôte générique .NET Core, aucune fabrique de télémétrie personnalisée n’est fournie. Toutefois, vous pouvez ajouter une télémétrie personnalisée au pipeline à l’aide de l’injection de dépendances. Les exemples de cette section nécessitent les instructions `using` suivantes :

```cs
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.Channel;
```

L’implémentation personnalisée suivante de [`ITelemetryInitializer`] vous permet d’ajouter votre propre [`ITelemetry`](/dotnet/api/microsoft.applicationinsights.channel.itelemetry) à la valeur par défaut de [`TelemetryConfiguration`].

```cs
internal class CustomTelemetryInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        // Do something with telemetry.
    }
}
```

Appelez [`ConfigureServices`] dans le générateur pour ajouter votre [`ITelemetryInitializer`] personnalisé au pipeline.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureLogging((context, b) =>
    {
        // Add logging providers.
        b.AddConsole();

        // If this key exists in any config, use it to enable Application Insights.
        string appInsightsKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
        if (!string.IsNullOrEmpty(appInsightsKey))
        {
            // This uses the options callback to explicitly set the instrumentation key.
            b.AddApplicationInsights(o => o.InstrumentationKey = appInsightsKey);
        }
    });
    builder.ConfigureServices(services =>
        {
            services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();
        });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Lorsque [`TelemetryConfiguration`] est construit, tous les types enregistrés de [`ITelemetryInitializer`] sont inclus. Pour en savoir plus, consultez [API Application Insights pour les événements et les métriques personnalisés](../azure-monitor/app/api-custom-events-metrics.md).

Dans la version 3.*x*, vous n’avez plus besoin de vider [`TelemetryClient`] quand l’hôte s’arrête. Le système d’injection de dépendance de .NET Core supprime automatiquement l’élément `ApplicationInsightsLoggerProvider` enregistré, ce qui vide [`TelemetryClient`].

#### <a name="version-2x"></a>Version 2.*x*

Dans la version 2.*x*, le [`TelemetryClient`] créé de façon interne par le fournisseur Application Insights pour le kit SDK WebJobs utilise [`ServerTelemetryChannel`](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/.publicApi/Microsoft.AI.ServerTelemetryChannel.dll). Quand le point de terminaison d’Application Insights est non disponible ou limite les requêtes entrantes, ce canal [enregistre les requêtes dans le système de fichiers de l’application web et les soumet à nouveau ultérieurement](https://apmtips.com/posts/2015-09-03-more-telemetry-channels/).

[`TelemetryClient`] est créé par une classe qui implémente `ITelemetryClientFactory`. Par défaut, il s’agit de [`DefaultTelemetryClientFactory`](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

Si vous souhaitez modifier une partie du pipeline Application Insights, vous pouvez fournir votre propre classe `ITelemetryClientFactory`, qui sera utilisée par l’hôte pour construire un élément [`TelemetryClient`]. Par exemple, ce code remplace `DefaultTelemetryClientFactory` pour modifier une propriété de `ServerTelemetryChannel` :

```csharp
private class CustomTelemetryClientFactory : DefaultTelemetryClientFactory
{
    public CustomTelemetryClientFactory(string instrumentationKey, Func<string, LogLevel, bool> filter)
        : base(instrumentationKey, new SamplingPercentageEstimatorSettings(), filter)
    {
    }

    protected override ITelemetryChannel CreateTelemetryChannel()
    {
        ServerTelemetryChannel channel = new ServerTelemetryChannel();

        // Change the default from 30 seconds to 15 seconds.
        channel.MaxTelemetryBufferDelay = TimeSpan.FromSeconds(15);

        return channel;
    }
}
```

L’objet `SamplingPercentageEstimatorSettings` configure l’[échantillonnage adaptatif](../azure-monitor/app/sampling.md). Cela signifie que dans certains scénarios impliquant de grands volumes, Applications Insights envoie un sous-ensemble spécifique de données de télémétrie au serveur.

Une fois que vous avez créé la fabrique de télémétrie, vous devez la passer au fournisseur de journalisation Application Insights :

```csharp
var clientFactory = new CustomTelemetryClientFactory(instrumentationKey, filter.Filter);

config.LoggerFactory = new LoggerFactory()
    .AddApplicationInsights(clientFactory);
```

## <a name="next-steps"></a><a id="nextsteps"></a>Étapes suivantes

Cet article vous a permis de découvrir des extraits qui montrent comment prendre en charge les scénarios courants d’utilisation du kit SDK WebJobs. Pour obtenir des exemples complets, consultez [azure-webjobs-sdk-samples](https://github.com/Azure/azure-webjobs-sdk/tree/dev/sample/SampleHost).

[`ExecutionContext`]: https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Core/ExecutionContext.cs
[`TelemetryClient`]: /dotnet/api/microsoft.applicationinsights.telemetryclient
[`ConfigureServices`]: /dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configureservices
[`ITelemetryInitializer`]: /dotnet/api/microsoft.applicationinsights.extensibility.itelemetryinitializer
[`TelemetryConfiguration`]: /dotnet/api/microsoft.applicationinsights.extensibility.telemetryconfiguration
[JobHostConfiguration]: https://github.com/Azure/azure-webjobs-sdk/blob/v2.x/src/Microsoft.Azure.WebJobs.Host/JobHostConfiguration.cs
