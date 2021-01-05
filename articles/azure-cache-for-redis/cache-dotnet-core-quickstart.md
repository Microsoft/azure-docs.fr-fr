---
title: 'Démarrage rapide : Utiliser Azure Cache pour Redis dans .NET Core'
description: Dans ce guide de démarrage rapide, apprenez à accéder au cache Azure pour Redis dans vos applications .NET Core
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.devlang: dotnet
ms.custom: devx-track-csharp, mvc
ms.topic: quickstart
ms.date: 06/18/2020
ms.openlocfilehash: 945d4a3d2bba84bf8f5973fd8dec092c66794c11
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004294"
---
# <a name="quickstart-use-azure-cache-for-redis-in-net-core"></a>Démarrage rapide : Utiliser Azure Cache pour Redis dans .NET Core

Dans ce guide de démarrage rapide, vous allez incorporer le cache Azure pour Redis dans une application .NET Core pour avoir accès à un cache sécurisé et dédié accessible à partir de n’importe quelle application dans Azure. Vous utiliserez spécifiquement le client [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) avec du code C# dans une application console .NET Core.

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- [Kit de développement logiciel (SDK) .NET Core](https://dotnet.microsoft.com/download)

## <a name="create-a-cache"></a>Création d'un cache
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-access-keys](../../includes/redis-cache-access-keys.md)]

Prenez note du **NOM D’HÔTE** et de la clé d’accès **PRINCIPALE**. Vous utiliserez ces valeurs ultérieurement pour construire le secret *CacheConnection*.



## <a name="create-a-console-app"></a>Créer une application console

Ouvrez une nouvelle fenêtre de commande et exécutez la commande suivante pour créer une nouvelle application console .NET Core :

```
dotnet new console -o Redistest
```

Dans votre fenêtre de commande, remplacez par le nouveau répertoire de projet *Redistest*.



## <a name="add-secret-manager-to-the-project"></a>Ajouter Secret Manager au projet

Dans cette section, vous allez ajouter l’outil [Secret Manager](/aspnet/core/security/app-secrets) à votre projet. L’outil Secret Manager stocke les données sensibles pour les travaux de développement à l’extérieur de l’arborescence de votre projet. Cette approche empêche le partage accidentel des secrets d’une application au sein du code source.

Ouvrez votre fichier *Redistest.csproj*. Ajoutez un élément `DotNetCliToolReference` afin d’inclure *Microsoft.Extensions.SecretManager.Tools*. Ajoutez également un élément `UserSecretsId`, comme indiqué ci-dessous, puis enregistrez le fichier.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp2.0</TargetFramework>
        <UserSecretsId>Redistest</UserSecretsId>
    </PropertyGroup>
    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.0" />
    </ItemGroup>
</Project>
```

Exécutez la commande suivante pour ajouter le package *Microsoft.Extensions.Configuration.UserSecrets* au projet :

```
dotnet add package Microsoft.Extensions.Configuration.UserSecrets
```

Exécutez la commande suivante pour restaurer vos packages :

```
dotnet restore
```

Dans la fenêtre de commande, exécutez la commande suivante pour stocker un nouveau secret nommé *CacheConnection*, après avoir remplacé les espaces réservés (y compris les crochets pointus) pour le nom de cache et la clé d’accès principale :

```
dotnet user-secrets set CacheConnection "<cache name>.redis.cache.windows.net,abortConnect=false,ssl=true,password=<primary-access-key>"
```

Ajoutez l’instruction `using` suivante à *Program.cs* :

```csharp
using Microsoft.Extensions.Configuration;
```

Ajoutez les membres suivants à la classe `Program` dans *Program.cs*. Ce code initialise une configuration permettant d’accéder au secret de l’utilisateur pour la chaîne de connexion du cache Azure pour Redis.

```csharp
private static IConfigurationRoot Configuration { get; set; }
const string SecretName = "CacheConnection";

private static void InitializeConfiguration()
{
    var builder = new ConfigurationBuilder()
        .AddUserSecrets<Program>();

    Configuration = builder.Build();
}
```

## <a name="configure-the-cache-client"></a>Configuration du client de cache

Dans cette section, vous allez configurer l’application console pour utiliser le client [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) pour .NET.

Dans la fenêtre de commande, exécutez la commande suivante dans le répertoire du projet *Redistest* :

```
dotnet add package StackExchange.Redis
```

Une fois l’installation terminée, le client de cache *StackExchange.Redis* est disponible pour être utilisé avec votre projet.


## <a name="connect-to-the-cache"></a>Connexion au cache

Ajoutez l’instruction `using` suivante à *Program.cs* :

```csharp
using StackExchange.Redis;
```

La connexion au cache Azure pour Redis est gérée par la classe `ConnectionMultiplexer`. Cette classe doit être partagée et réutilisée dans votre application cliente. Ne créez pas une nouvelle connexion pour chaque opération. 

Dans *Program.cs*, ajoutez les membres suivants à la classe `Program` de votre application console :

```csharp
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = Configuration[SecretName];
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

Cette approche pour partager une instance `ConnectionMultiplexer` dans votre application utilise une propriété statique qui renvoie une instance connectée. Ce code fournit une méthode thread-safe permettant d’initialiser une seule instance `ConnectionMultiplexer` connectée. `abortConnect` a la valeur false, ce qui signifie que l’appel réussit même si aucune connexion au cache Azure pour Redis n’est établie. Une fonctionnalité clé de `ConnectionMultiplexer` est qu’il restaure automatiquement la connectivité au cache une fois que le problème réseau ou d’autres causes sont résolus.

La valeur du secret *CacheConnection* est accessible à l’aide du fournisseur de configuration Secret Manager et utilisée en tant que paramètre du mot de passe.

## <a name="executing-cache-commands"></a>Exécution des commandes de cache

Dans *Program.cs*, ajoutez le code suivant pour la procédure `Main` de la classe `Program` pour votre application console :

```csharp
static void Main(string[] args)
{
    InitializeConfiguration();

    // Connection refers to a property that returns a ConnectionMultiplexer
    // as shown in the previous example.
    IDatabase cache = lazyConnection.Value.GetDatabase();

    // Perform cache operations using the cache object...

    // Simple PING command
    string cacheCommand = "PING";
    Console.WriteLine("\nCache command  : " + cacheCommand);
    Console.WriteLine("Cache response : " + cache.Execute(cacheCommand).ToString());

    // Simple get and put of integral data types into the cache
    cacheCommand = "GET Message";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
    Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());

    cacheCommand = "SET Message \"Hello! The cache is working from a .NET Core console app!\"";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringSet()");
    Console.WriteLine("Cache response : " + cache.StringSet("Message", "Hello! The cache is working from a .NET Core console app!").ToString());

    // Demonstrate "SET Message" executed as expected...
    cacheCommand = "GET Message";
    Console.WriteLine("\nCache command  : " + cacheCommand + " or StringGet()");
    Console.WriteLine("Cache response : " + cache.StringGet("Message").ToString());

    // Get the client list, useful to see if connection list is growing...
    cacheCommand = "CLIENT LIST";
    Console.WriteLine("\nCache command  : " + cacheCommand);
    Console.WriteLine("Cache response : \n" + cache.Execute("CLIENT", "LIST").ToString().Replace("id=", "id="));

    lazyConnection.Value.Dispose();
}
```

Enregistrez *Program.cs*.

Le cache Azure pour Redis dispose d’un nombre configurable de bases de données (16 par défaut) pouvant être utilisées pour séparer de manière logique les données dans un cache Azure pour Redis. Le code se connecte à la base de données par défaut, DB 0. Pour plus d’informations, consultez les sections [What are Redis databases?](cache-development-faq.md#what-are-redis-databases) (Que sont les bases de données Redis ?) et [Configuration du serveur Redis par défaut](cache-configure.md#default-redis-server-configuration).

Les éléments de cache peuvent être stockés et extraits en utilisant les méthodes `StringSet` et `StringGet`.

Redis stocke la plupart des données sous la forme de chaînes Redis, mais ces chaînes peuvent contenir de nombreux types de données, notamment des données binaires sérialisées, qui peuvent être utilisées lors du stockage d'objets .NET dans le cache.

Exécutez la commande suivante dans la fenêtre de commande pour générer l’application :

```
dotnet build
```

Exécutez ensuite l’application avec la commande suivante :

```
dotnet run
```

Dans l’exemple ci-dessous, vous pouvez voir que la clé `Message` présentait auparavant une valeur mise en cache, qui avait été définie à l’aide de la console Redis du portail Azure. L’application a mis à jour cette valeur mise en cache. Elle a également exécuté les commandes `PING` et `CLIENT LIST`.

![Application console partielle](./media/cache-dotnet-core-quickstart/cache-console-app-partial.png)


## <a name="work-with-net-objects-in-the-cache"></a>Utilisation des objets .NET dans le cache

Le cache Azure pour Redis peut mettre en cache des objets .NET et des types de données primitifs, mais avant qu’un objet .NET puisse être mis en cache, il doit être sérialisé. La sérialisation d’objet .NET échoit au développeur d’applications, qui a toute latitude pour choisir le sérialiseur.

Une méthode simple pour sérialiser des objets consiste à utiliser les méthodes de sérialisation `JsonConvert` dans [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) et à sérialiser vers et à partir de JSON. Dans cette section, vous allez ajouter un objet .NET dans le cache.

Exécutez la commande suivante pour ajouter le package *Newtonsoft.json* à l’application :

```
dotnet add package Newtonsoft.json
```

Ajoutez l’instruction `using` suivante au début de *Program.cs* :

```csharp
using Newtonsoft.Json;
```

Ajoutez la définition de classe `Employee` suivante à *Program.cs* :

```csharp
class Employee
{
    public string Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }

    public Employee(string EmployeeId, string Name, int Age)
    {
        this.Id = EmployeeId;
        this.Name = Name;
        this.Age = Age;
    }
}
```

Au bas de la procédure `Main()` dans *Program.cs* et avant l’appel à `Dispose()`, ajoutez les lignes de code suivantes à mettre en cache et récupérez un objet .NET sérialisé :

```csharp
    // Store .NET object to cache
    Employee e007 = new Employee("007", "Davide Columbo", 100);
    Console.WriteLine("Cache response from storing Employee .NET object : " + 
    cache.StringSet("e007", JsonConvert.SerializeObject(e007)));

    // Retrieve .NET object from cache
    Employee e007FromCache = JsonConvert.DeserializeObject<Employee>(cache.StringGet("e007"));
    Console.WriteLine("Deserialized Employee .NET object :\n");
    Console.WriteLine("\tEmployee.Name : " + e007FromCache.Name);
    Console.WriteLine("\tEmployee.Id   : " + e007FromCache.Id);
    Console.WriteLine("\tEmployee.Age  : " + e007FromCache.Age + "\n");
```

Enregistrez *Program.cs* et régénérez l’application avec la commande suivante :

```
dotnet build
```

Exécutez l’application avec la commande suivante pour tester la sérialisation d’objets .NET :

```
dotnet run
```

![Application console terminée](./media/cache-dotnet-core-quickstart/cache-console-app-complete.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’exécuter le didacticiel suivant, vous pouvez conserver les ressources créées dans le cadre de ce guide de démarrage rapide afin de les réutiliser.

Sinon, si l’exemple d’application de démarrage rapide était votre dernière opération, vous pouvez supprimer les ressources Azure créées dans ce démarrage rapide afin d’éviter tout frais. 

> [!IMPORTANT]
> La suppression d’un groupe de ressources est définitive ; le groupe de ressources et l’ensemble des ressources qu’il contient sont supprimés de manière permanente. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. Si vous avez créé les ressources pour l’hébergement de cet exemple dans un groupe de ressources existant contenant des ressources que vous souhaitez conserver, vous pouvez supprimer chaque ressource individuellement à partir de son panneau respectif, au lieu de supprimer l’intégralité du groupe de ressources.
>

Connectez-vous au [Portail Azure](https://portal.azure.com) et cliquez sur **Groupes de ressources**.

Dans la zone de texte **Filtrer par nom.** , saisissez le nom de votre groupe de ressources. Les instructions de cet article ont utilisé un groupe de ressources nommé *TestResources*. Sur votre groupe de ressources dans la liste des résultats, cliquez sur **...** , puis sur **Supprimer le groupe de ressources**.

![DELETE](./media/cache-dotnet-core-quickstart/cache-delete-resource-group.png)

Il vous sera demandé de confirmer la suppression du groupe de ressources. Saisissez le nom de votre groupe de ressources pour confirmer, puis cliquez sur **Supprimer**.

Après quelques instants, le groupe de ressources et toutes les ressources qu’il contient sont supprimés.



<a name="next-steps"></a>

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser le cache Azure pour Redis à partir d’une application .NET Core. Passez au guide de démarrage rapide suivant pour utiliser le cache Azure pour Redis avec une application web ASP.NET.

> [!div class="nextstepaction"]
> [Créer une application web ASP.NET qui utilise un cache Azure pour Redis.](./cache-web-app-howto.md)

Vous souhaitez optimiser et réduire vos coûts de cloud ?

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts avec Cost Management](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)