---
title: Démarrage rapide – Générer une application console .NET pour gérer les ressources de l’API SQL Azure Cosmos DB
description: Découvrez comment générer une application console .NET pour gérer les ressources du compte de l’API SQL Azure Cosmos DB dans ce guide de démarrage rapide.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/21/2020
ms.custom: devx-track-dotnet
ms.openlocfilehash: aceb26604d67f42cdbbe1395e3a4b08675d70ea1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93078524"
---
# <a name="quickstart-build-a-net-console-app-to-manage-azure-cosmos-db-sql-api-resources"></a>Démarrage rapide : Générer une application console .NET pour gérer les ressources de l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [Kit SDK Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Node.JS](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Commencez à utiliser la bibliothèque de client de l’API SQL Azure Cosmos DB pour .NET. Suivez les étapes décrites dans ce document pour installer le package .NET, générer une application et essayer l’exemple de code pour les opérations CRUD de base sur les données stockées dans Azure Cosmos DB. 

Azure Cosmos DB est la base de données NoSQL rapide de Microsoft avec des API ouvertes pour toute échelle. Vous pouvez utiliser Azure Cosmos DB pour créer et interroger rapidement des bases de données de clés/valeurs, de documents et de graphiques. Utilisez la bibliothèque de client de l’API SQL Azure Cosmos DB pour .NET afin d’effectuer les opérations suivantes :

* Créer une base de données et un conteneur Azure Cosmos
* Ajouter des exemples de données au conteneur
* Interroger les données 
* Supprimer la base de données

[Documentation de référence sur l’API](/dotnet/api/microsoft.azure.cosmos?view=azure-dotnet&preserve-view=true) | [Code source de la bibliothèque](https://github.com/Azure/azure-cosmos-dotnet-v3) | [Package (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Cosmos)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure – [Vous pouvez en créer un gratuitement](https://azure.microsoft.com/free/) ou [essayer Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure, sans frais et sans engagement. 
* Kit de développement logiciel (SDK) [.NET Core 2.1 ou version ultérieure](https://dotnet.microsoft.com/download/dotnet-core/2.1).

## <a name="setting-up"></a>Configuration

Cette section vous guide dans la création d’un compte Azure Cosmos et la configuration d’un projet qui utilise la bibliothèque de client de l’API SQL Azure Cosmos DB pour .NET afin de gérer les ressources. L’exemple de code décrit dans cet article crée une base de données `FamilyDatabase` et des membres de la famille (chacun d’eux est un élément) au sein de cette base de données. Chaque membre de la famille possède des propriétés telles que `Id, FamilyName, FirstName, LastName, Parents, Children, Address,`. La propriété `LastName` est utilisée en tant que clé de partition pour le conteneur. 

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a>Créer un compte Azure Cosmos

Si vous utilisez l’option [Essayer gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) pour créer un compte Azure Cosmos, vous devez créer un compte Azure Cosmos DB de type **API SQL**. Un compte de test Azure Cosmos DB est déjà créé pour vous. Vous n’avez pas besoin de créer le compte explicitement. Vous pouvez donc ignorer cette section et passer à la section suivante.

Si vous disposez de votre propre abonnement Azure ou si vous avez créé un abonnement gratuitement, vous devez créer un compte Azure Cosmos de manière explicite. Le code suivant crée un compte Azure Cosmos avec une cohérence de session. Le compte est répliqué dans `South Central US` et `North Central US`.  

Vous pouvez utiliser Azure Cloud Shell pour créer le compte Azure Cosmos. Azure Cloud Shell est un interpréteur de commandes interactif, authentifié et accessible par navigateur qui permet de gérer les ressources Azure. Il vous donne la possibilité de choisir l'expérience d'interpréteur de commandes la plus adaptée à votre façon de travailler, qu'il s'agisse de Bash ou de PowerShell. Pour ce guide de démarrage rapide, choisissez le mode **Bash**. Azure Cloud Shell nécessite également un compte de stockage. Vous pouvez en créer un quand vous y êtes invité.

Sélectionnez le bouton **Essayer** en regard du code suivant, choisissez le mode **Bash** , sélectionnez **Créer un compte de stockage** , puis connectez-vous à Cloud Shell. Ensuite, copiez et collez le code suivant dans Azure Cloud Shell, puis exécutez-le. Le nom du compte Azure Cosmos doit être globalement unique. Veillez à mettre à jour la valeur `mysqlapicosmosdb` avant d’exécuter la commande.

```azurecli-interactive

# Set variables for the new SQL API account, database, and container
resourceGroupName='myResourceGroup'
location='southcentralus'

# The Azure Cosmos account name must be globally unique, make sure to update the `mysqlapicosmosdb` value before you run the command
accountName='mysqlapicosmosdb'

# Create a resource group
az group create \
    --name $resourceGroupName \
    --location $location

# Create a SQL API Cosmos DB account with session consistency and multi-region writes enabled
az cosmosdb create \
    --resource-group $resourceGroupName \
    --name $accountName \
    --kind GlobalDocumentDB \
    --locations regionName="South Central US" failoverPriority=0 --locations regionName="North Central US" failoverPriority=1 \
    --default-consistency-level "Session" \
    --enable-multiple-write-locations true

```

La création du compte Azure Cosmos prend un certain temps. Une fois l’opération réussie, la sortie de confirmation s’affiche. Une fois la commande correctement exécutée, connectez-vous au [portail Azure](https://portal.azure.com/) et vérifiez que le compte Azure Cosmos portant le nom spécifié existe. Vous pouvez fermer la fenêtre Azure Cloud Shell une fois la ressource créée. 

### <a name="create-a-new-net-app"></a><a id="create-dotnet-core-app"></a>Créer une application .NET

Créez une application .NET dans votre éditeur ou IDE favori. Ouvrez l’invite de commandes Windows ou une fenêtre de terminal à partir de votre ordinateur local. Vous allez exécuter toutes les commandes des sections suivantes à partir de l’invite de commandes ou du terminal.  Exécutez la commande dotnet new suivante pour créer une application nommée `todo`. Le paramètre --langVersion définit la propriété LangVersion dans le fichier projet créé.

```console
dotnet new console --langVersion 7.1 -n todo
```

Déplacez vos répertoires vers le dossier d’application nouvellement créé. Vous pouvez générer l’application avec :

```console
cd todo
dotnet build
```

La sortie attendue de la build doit ressembler à ceci :

```console
  Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\todo\todo.csproj.
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.dll
  todo -> C:\Users\user1\Downloads\CosmosDB_Samples\todo\bin\Debug\netcoreapp2.2\todo.Views.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:34.17
```

### <a name="install-the-azure-cosmos-db-package"></a><a id="install-package"></a>Installer le package Azure Cosmos DB

Tout en restant dans le répertoire de l’application, installez la bibliothèque de client Azure Cosmos DB pour .NET Core à l’aide de la commande dotnet add Package.

```console
dotnet add package Microsoft.Azure.Cosmos
```

### <a name="copy-your-azure-cosmos-account-credentials-from-the-azure-portal"></a>Copiez les informations d’identification de votre compte Azure Cosmos à partir du portail Azure

L’exemple d’application doit s’authentifier auprès de votre compte Azure Cosmos. Pour vous authentifier, vous devez transmettre les informations d’identification du compte Azure Cosmos à l’application. Pour obtenir les informations d’identification de votre compte Azure Cosmos, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Accédez à votre compte Azure Cosmos.

1. Ouvrez le volet **Clés** et copiez l’ **URI** et la **CLÉ PRIMAIRE** de votre compte. Vous allez ajouter les valeur d’URI et de clés à une variable d’environnement à l’étape suivante.

### <a name="set-the-environment-variables"></a>Définir les variables d’environnement

Après avoir copié l’ **URI** et la **CLÉ PRIMAIRE** de votre compte, enregistrez-les dans une nouvelle variable d’environnement sur la machine locale exécutant l’application. Pour définir la variable d’environnement, ouvrez une fenêtre de console, puis exécutez la commande suivante. Veillez à remplacer les valeurs `<Your_Azure_Cosmos_account_URI>` et `<Your_Azure_Cosmos_account_PRIMARY_KEY>`.

**Windows**

```console
setx EndpointUrl "<Your_Azure_Cosmos_account_URI>"
setx PrimaryKey "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**Linux**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

**macOS**

```bash
export EndpointUrl = "<Your_Azure_Cosmos_account_URI>"
export PrimaryKey = "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
```

 ## <a name="object-model"></a><a id="object-model"></a>Modèle objet

Avant de commencer à générer l’application, examinons la hiérarchie des ressources dans Azure Cosmos DB et le modèle objet utilisé pour créer ces ressources et y accéder. Azure Cosmos DB crée des ressources dans l’ordre suivant :

* Compte Azure Cosmos 
* Bases de données 
* Containers 
* Éléments

Pour en savoir plus sur la hiérarchie des différentes entités, voir [Utilisation de bases de données, conteneurs et éléments dans Azure Cosmos DB](account-databases-containers-items.md). Vous allez utiliser les classes .NET suivantes pour interagir avec ces ressources :

* [CosmosClient](/dotnet/api/microsoft.azure.cosmos.cosmosclient?preserve-view=true&view=azure-dotnet) : cette classe fournit une représentation logique côté client pour le service Azure Cosmos DB. Ce client est utilisé pour configurer et exécuter des requêtes sur le service.

* [CreateDatabaseIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync?view=azure-dotnet&preserve-view=true) : cette méthode obtient ou, si nécessaire, crée une ressource de base de données en tant qu’opération asynchrone. 

* [CreateContainerIfNotExistsAsync](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet&preserve-view=true) : cette méthode obtient ou, si nécessaire, crée un conteneur en tant qu’opération asynchrone. Vous pouvez vérifier le code d’état de la réponse pour déterminer si le conteneur vient d’être créé (201) ou si un conteneur existant a été retourné (200). 
* [CreateItemAsync](/dotnet/api/microsoft.azure.cosmos.container.createitemasync?view=azure-dotnet&preserve-view=true) : cette méthode crée un élément dans le conteneur. 

* [UpsertItemAsync](/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync?view=azure-dotnet&preserve-view=true) : cette méthode crée un élément dans le conteneur s’il n’existe pas déjà ou remplace l’élément s’il existe déjà. 

* [GetItemQueryIterator](/dotnet/api/microsoft.azure.cosmos.container.GetItemQueryIterator?view=azure-dotnet&preserve-view=true) : cette méthode crée une requête pour les éléments sous un conteneur dans une base de données Azure Cosmos à l’aide d’une instruction SQL avec des valeurs paramétrables. 

* [DeleteAsync](/dotnet/api/microsoft.azure.cosmos.database.deleteasync?view=azure-dotnet&preserve-view=true) : supprime la base de données spécifiée de votre compte Azure Cosmos. Le méthode `DeleteAsync` supprime uniquement les base de données. La suppression de l’instance `Cosmosclient` doit se produire séparément (ce qu’elle fait dans la méthode DeleteDatabaseAndCleanupAsync. 

 ## <a name="code-examples"></a><a id="code-examples"></a>Exemples de codes

L’exemple de code décrit dans cet article crée une base de données de famille dans Azure Cosmos DB. La base de données de famille contient des détails sur la famille, tels que le nom, l’adresse, l’emplacement, ainsi que les parents, enfants et animaux associés. Avant de renseigner les données dans votre compte Azure Cosmos, définissez les propriétés d’un élément de famille. Créez une classe nommée `Family.cs` au niveau racine de votre exemple d’application, puis ajoutez-y le code suivant :

```csharp
using Newtonsoft.Json;

namespace todo
{
    public class Family
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }
        public string LastName { get; set; }
        public Parent[] Parents { get; set; }
        public Child[] Children { get; set; }
        public Address Address { get; set; }
        public bool IsRegistered { get; set; }
        // The ToString() method is used to format the output, it's used for demo purpose only. It's not required by Azure Cosmos DB
        public override string ToString()
        {
            return JsonConvert.SerializeObject(this);
        }
    }

    public class Parent
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
    }

    public class Child
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
        public string Gender { get; set; }
        public int Grade { get; set; }
        public Pet[] Pets { get; set; }
    }

    public class Pet
    {
        public string GivenName { get; set; }
    }

    public class Address
    {
        public string State { get; set; }
        public string County { get; set; }
        public string City { get; set; }
    }
}
```

### <a name="add-the-using-directives--define-the-client-object"></a>Ajoutez les directives d’utilisation et définir l’objet client

Dans le répertoire du projet, ouvrez le fichier `Program.cs` dans votre éditeur et ajoutez les directives d’utilisation suivantes en haut de votre application :

```csharp

using System;
using System.Threading.Tasks;
using System.Configuration;
using System.Collections.Generic;
using System.Net;
using Microsoft.Azure.Cosmos;
```

Pour le fichier **Program.cs** , ajoutez du code pour lire les variables d’environnement que vous avez définies à l’étape précédente. Définissez les valeurs `CosmosClient`, `Database` et les objets `Container`. Ensuite, ajoutez du code à la méthode principale qui appelle la méthode `GetStartedDemoAsync` dans laquelle vous gérez les ressources de compte Azure Cosmos. 

```csharp
namespace todo
{
public class Program
{

    /// The Azure Cosmos DB endpoint for running this GetStarted sample.
    private string EndpointUrl = Environment.GetEnvironmentVariable("EndpointUrl");

    /// The primary key for the Azure DocumentDB account.
    private string PrimaryKey = Environment.GetEnvironmentVariable("PrimaryKey");

    // The Cosmos client instance
    private CosmosClient cosmosClient;

    // The database we will create
    private Database database;

    // The container we will create.
    private Container container;

    // The name of the database and container we will create
    private string databaseId = "FamilyDatabase";
    private string containerId = "FamilyContainer";

    public static async Task Main(string[] args)
    {
       try
       {
           Console.WriteLine("Beginning operations...\n");
           Program p = new Program();
           await p.GetStartedDemoAsync();

        }
        catch (CosmosException de)
        {
           Exception baseException = de.GetBaseException();
           Console.WriteLine("{0} error occurred: {1}", de.StatusCode, de);
        }
        catch (Exception e)
        {
            Console.WriteLine("Error: {0}", e);
        }
        finally
        {
            Console.WriteLine("End of demo, press any key to exit.");
            Console.ReadKey();
        }
    }
}
}
```

### <a name="create-a-database"></a>Création d'une base de données 

Définissez la méthode `CreateDatabaseAsync` dans la classe `program.cs`. Cette méthode crée le `FamilyDatabase` s’il n’existe pas.

```csharp
private async Task CreateDatabaseAsync()
{
    // Create a new database
    this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
    Console.WriteLine("Created Database: {0}\n", this.database.Id);
}
```

### <a name="create-a-container"></a>Créez un conteneur.

Définissez la méthode `CreateContainerAsync` dans la classe `program.cs`. Cette méthode crée le `FamilyContainer` s’il n’existe pas. 

```csharp
/// Create the container if it does not exist. 
/// Specifiy "/LastName" as the partition key since we're storing family information, to ensure good distribution of requests and storage.
private async Task CreateContainerAsync()
{
    // Create a new container
    this.container = await this.database.CreateContainerIfNotExistsAsync(containerId, "/LastName");
    Console.WriteLine("Created Container: {0}\n", this.container.Id);
}
```

### <a name="create-an-item"></a>Créer un élément

Créez un élément de famille en ajoutant la méthode `AddItemsToContainerAsync` avec le code suivant. Vous pouvez utiliser les méthodes `UpsertItemAsync` ou `CreateItemAsync` pour créer un élément :

```csharp
private async Task AddItemsToContainerAsync()
{
    // Create a family object for the Andersen family
    Family andersenFamily = new Family
    {
        Id = "Andersen.1",
        LastName = "Andersen",
        Parents = new Parent[]
        {
           new Parent { FirstName = "Thomas" },
           new Parent { FirstName = "Mary Kay" }
        },
        Children = new Child[]
        {
           new Child
            {
                FirstName = "Henriette Thaulow",
                Gender = "female",
                Grade = 5,
                Pets = new Pet[]
                {
                    new Pet { GivenName = "Fluffy" }
                }
            }
        },
        Address = new Address { State = "WA", County = "King", City = "Seattle" },
        IsRegistered = false
    };

    try
    {
        // Create an item in the container representing the Andersen family. Note we provide the value of the partition key for this item, which is "Andersen".
        ItemResponse<Family> andersenFamilyResponse = await this.container.CreateItemAsync<Family>(andersenFamily, new PartitionKey(andersenFamily.LastName));
        // Note that after creating the item, we can access the body of the item with the Resource property of the ItemResponse. We can also access the RequestCharge property to see the amount of RUs consumed on this request.
        Console.WriteLine("Created item in database with id: {0} Operation consumed {1} RUs.\n", andersenFamilyResponse.Resource.Id, andersenFamilyResponse.RequestCharge);
    }
    catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.Conflict)
    {
        Console.WriteLine("Item in database with id: {0} already exists\n", andersenFamily.Id);                
    }
}

```

### <a name="query-the-items"></a>Interroger les éléments

Après avoir inséré un élément, vous pouvez exécuter une requête pour obtenir les détails de la famille « Anderson ». Le code suivant montre comment procéder directement à l’aide de la requête SQL. La requête SQL pour obtenir les détails de la famille « Anderson » est la suivante : `SELECT * FROM c WHERE c.LastName = 'Andersen'`. Définissez la méthode `QueryItemsAsync` dans la classe `program.cs` et ajoutez-y le code suivant :


```csharp
private async Task QueryItemsAsync()
{
    var sqlQueryText = "SELECT * FROM c WHERE c.LastName = 'Andersen'";

    Console.WriteLine("Running query: {0}\n", sqlQueryText);

    QueryDefinition queryDefinition = new QueryDefinition(sqlQueryText);
    FeedIterator<Family> queryResultSetIterator = this.container.GetItemQueryIterator<Family>(queryDefinition);

    List<Family> families = new List<Family>();

    while (queryResultSetIterator.HasMoreResults)
    {
        FeedResponse<Family> currentResultSet = await queryResultSetIterator.ReadNextAsync();
        foreach (Family family in currentResultSet)
        {
            families.Add(family);
            Console.WriteLine("\tRead {0}\n", family);
        }
    }
}

```

### <a name="delete-the-database"></a>Supprimer la base de données 

Enfin, vous pouvez supprimer la base de données en ajoutant la méthode `DeleteDatabaseAndCleanupAsync` avec le code suivant :

```csharp
private async Task DeleteDatabaseAndCleanupAsync()
{
   DatabaseResponse databaseResourceResponse = await this.database.DeleteAsync();
   // Also valid: await this.cosmosClient.Databases["FamilyDatabase"].DeleteAsync();

   Console.WriteLine("Deleted Database: {0}\n", this.databaseId);

   //Dispose of CosmosClient
    this.cosmosClient.Dispose();
}
```

### <a name="execute-the-crud-operations"></a>Exécuter les opérations CRUD

Après avoir défini toutes les méthodes requises, exécutez-les avec la méthode `GetStartedDemoAsync`. La méthode `DeleteDatabaseAndCleanupAsync` est commentée dans ce code parce que vous ne verrez aucune ressource si elle est exécutée. Vous pouvez supprimer les marques de commentaire après avoir vérifié que vos ressources Azure Cosmos DB ont été créées dans le portail Azure. 

```csharp
public async Task GetStartedDemoAsync()
{
    // Create a new instance of the Cosmos Client
    this.cosmosClient = new CosmosClient(EndpointUrl, PrimaryKey);
    await this.CreateDatabaseAsync();
    await this.CreateContainerAsync();
    await this.AddItemsToContainerAsync();
    await this.QueryItemsAsync();
}
```

Après avoir ajouté toutes les méthodes requises, enregistrez le fichier `Program.cs`. 

## <a name="run-the-code"></a>Exécuter le code

Ensuite, générez et exécutez l’application pour créer les ressources Azure Cosmos DB. Veillez à ouvrir une nouvelle fenêtre d’invite de commandes, et n’utilisez pas la même instance que celle que vous avez utilisée pour définir les variables d’environnement. En effet, les variables d’environnement ne sont pas définies dans la fenêtre ouverte active. Vous devrez ouvrir une nouvelle invite de commandes pour afficher les mises à jour. 

```console
dotnet build
```

```console
dotnet run
```

La sortie suivante est générée lorsque vous exécutez l’application. Vous pouvez également vous connecter au portail Azure et vérifier que les ressources ont été créées :

```console
Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.62 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

End of demo, press any key to exit.
```

Vous pouvez vérifier que les données ont été créées en vous connectant au portail Azure et en affichant les éléments requis dans votre compte Azure Cosmos. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du compte Azure Cosmos, vous pouvez utiliser Azure CLI ou Azure PowerShell pour le supprimer ainsi que le groupe de ressources correspondant. La commande suivante montre comment supprimer le groupe de ressources à l’aide d’Azure CLI :

```azurecli
az group delete -g "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte Azure Cosmos, ainsi qu’une base de données et un conteneur à l’aide d’une application .NET Core. Vous pouvez désormais importer des données supplémentaires dans votre compte Azure Cosmos en procédant de la manière décrite dans l’article suivant. 

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)