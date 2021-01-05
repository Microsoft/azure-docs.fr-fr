---
title: 'Tutoriel : Créer une application console .NET pour gérer des données dans le compte de l’API SQL Azure Cosmos DB'
description: 'Tutoriel : Découvrez comment créer des ressources d’API SQL Azure Cosmos DB à l’aide d’une application console C#.'
author: kirankumarkolli
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: kirankk
ms.custom: devx-track-csharp
ms.openlocfilehash: 49fa928285b29eaff806b009cf327e84e17491c6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93098720"
---
# <a name="tutorial-build-a-net-console-app-to-manage-data-in-azure-cosmos-db-sql-api-account"></a>Tutoriel : Créer une application console .NET pour gérer des données dans le compte de l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](./create-sql-api-java.md)
> * [Java asynchrone](./create-sql-api-java.md)
> * [Node.JS](sql-api-nodejs-get-started.md)
>

Bienvenue dans le tutoriel de prise en main de l’API SQL Azure Cosmos DB. À la fin de ce didacticiel, vous disposerez d’une application de console qui crée et interroge des ressources Azure Cosmos DB.

Ce tutoriel utilise la version 3.0 ou ultérieure du kit [SDK .NET Azure Cosmos DB](https://www.nuget.org/packages/Microsoft.Azure.Cosmos). Vous pouvez utiliser [.NET Framework ou .NET Core](https://dotnet.microsoft.com/download).

Ce didacticiel contient les sections suivantes :

> [!div class="checklist"]
>
> * Création et connexion à un compte Azure Cosmos
> * Configuration de votre projet dans Visual Studio
> * Création d’une base de données et d’un conteneur
> * Ajout d’éléments au conteneur
> * Interrogation du conteneur
> * Exécution d’opérations de création, lecture, mise à jour et suppression (CRUD) sur l’élément
> * Suppression de la base de données

Vous n’avez pas le temps ? Ne vous inquiétez pas ! La solution complète est disponible sur [GitHub](https://github.com/Azure-Samples/cosmos-dotnet-getting-started). Pour obtenir des instructions rapides, allez à la section [Obtenir la solution complète du tutoriel](#GetSolution).

Commençons dès maintenant !

## <a name="prerequisites"></a>Prérequis

* Un compte Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [compte gratuit](https://azure.microsoft.com/free/).

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [!INCLUDE [cosmos-db-emulator-vs](../../includes/cosmos-db-emulator-vs.md)]

## <a name="step-1-create-an-azure-cosmos-db-account"></a>Étape 1 : Création d’un compte Azure Cosmos DB

Commençons par créer un compte Azure Cosmos DB. Si vous disposez déjà d’un compte et que vous voulez l’utiliser, ignorez cette section. Pour utiliser l’émulateur Azure Cosmos DB, suivez les étapes de la section [Émulateur Azure Cosmos DB](local-emulator.md) pour le configurer. Passez ensuite à l’[Étape 2 : Configurer votre projet Visual Studio](#SetupVS).

[!INCLUDE [create-dbaccount-preview](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="step-2-set-up-your-visual-studio-project"></a><a id="SetupVS"></a>Étape 2 : Configurer votre projet Visual Studio

1. Ouvrez Visual Studio et sélectionnez **Créer un projet**.
1. Dans **Créer un projet** , choisissez **Application console (.NET Framework)** pour C#, puis sélectionnez **Suivant**.
1. Nommez votre projet *CosmosGettingStartedTutorial* , puis sélectionnez **Créer**.

    :::image type="content" source="./media/sql-api-get-started/configure-cosmos-getting-started-2019.png" alt-text="Configurer votre projet":::

1. Dans l’ **Explorateur de solutions** , cliquez avec le bouton droit sur votre nouvelle application console, qui se trouve sous votre solution Visual Studio, puis sélectionnez **Gérer les packages NuGet**.
1. Dans le **Gestionnaire de package NuGet** , sélectionnez **Parcourir** , puis recherchez *Microsoft.Azure.Cosmos*. Choisissez **Microsoft.Azure.Cosmos** et sélectionnez **Installer**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-manage-nuget-2019.png" alt-text="Installer NuGet pour le SDK client Azure Cosmos DB":::

   L’ID de package de la bibliothèque cliente d’API SQL Azure Cosmos DB est [Microsoft Azure Cosmos DB Client Library](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/).

Parfait ! L’installation étant terminée, nous pouvons passer à l’écriture du code. Pour le projet terminé de ce tutoriel, consultez [Développement d’une application console .NET à l’aide d’Azure Cosmos DB](https://github.com/Azure-Samples/cosmos-dotnet-getting-started).

## <a name="step-3-connect-to-an-azure-cosmos-db-account"></a><a id="Connect"></a>Étape 3 : Se connecter à un compte Azure Cosmos DB

1. Remplacez les références au début de votre application C# dans le fichier *Program.cs* par ces références :

   ```csharp
   using System;
   using System.Threading.Tasks;
   using System.Configuration;
   using System.Collections.Generic;
   using System.Net;
   using Microsoft.Azure.Cosmos;
   ```

1. Ajoutez ces constantes et variables dans votre classe `Program`.

    ```csharp
    public class Program
    {
        // ADD THIS PART TO YOUR CODE

        // The Azure Cosmos DB endpoint for running this sample.
        private static readonly string EndpointUri = "<your endpoint here>";
        // The primary key for the Azure Cosmos account.
        private static readonly string PrimaryKey = "<your primary key>";

        // The Cosmos client instance
        private CosmosClient cosmosClient;

        // The database we will create
        private Database database;

        // The container we will create.
        private Container container;

        // The name of the database and container we will create
        private string databaseId = "FamilyDatabase";
        private string containerId = "FamilyContainer";
    }
    ```

   > [!NOTE]
   > Si la version précédente du SDK .NET vous êtes familière, vous connaissez peut-être les termes *collection* et *document*. Comme Azure Cosmos DB prend en charge plusieurs modèles d’API, la version 3.0 du SDK .NET utilise les termes génériques *conteneur* et *élément*. Un *conteneur* peut être une collection, un graphe ou une table. Un *élément* peut être un document, une arête/un sommet ou une ligne, et il s’agit du contenu situé à l’intérieur d’un conteneur. Pour plus d’informations, consultez [Utiliser des bases de données, des conteneurs et des éléments dans Azure Cosmos DB](account-databases-containers-items.md).

1. Ouvrez le [portail Azure](https://portal.azure.com). Recherchez votre compte Azure Cosmos DB, puis sélectionnez **Clés**.

   :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-portal-keys.png" alt-text="Obtenir des clés Azure Cosmos DB à partir du portail Azure":::

1. Dans *Program.cs* , remplacez `<your endpoint URL>` par la valeur de l’ **URI**. Remplacez `<your primary key>` par la valeur de **CLÉ PRIMAIRE**.

1. Sous la méthode **Main** , ajoutez une nouvelle tâche asynchrone appelée **GetStartedDemoAsync** , qui instancie le nouveau `CosmosClient`.

    ```csharp
    public static async Task Main(string[] args)
    {
    }

    // ADD THIS PART TO YOUR CODE
    /*
        Entry point to call methods that operate on Azure Cosmos DB resources in this sample
    */
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
    }
    ```

    Nous utilisons **GetStartedDemoAsync** comme point d’entrée appelant des méthodes qui opèrent sur des ressources Azure Cosmos DB.

1. Ajoutez le code suivant pour exécuter la tâche asynchrone **GetStartedDemoAsync** à partir de la méthode **Main**. La méthode **Main** intercepte les exceptions et les consigne dans la console.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=Main)]

1. Sélectionnez F5 pour exécuter votre application.

    La console affiche le message : **End of demo, press any key to exit.** Ce message confirme que votre application a établi une connexion avec Azure Cosmos DB. Vous pouvez alors fermer la fenêtre de console.

Félicitations ! Vous avez établi une connexion avec un compte Azure Cosmos DB.

## <a name="step-4-create-a-database"></a>Étape 4 : Création d'une base de données

Une base de données est le conteneur logique d’éléments partitionnés entre les conteneurs. La méthode `CreateDatabaseIfNotExistsAsync` ou `CreateDatabaseAsync` de la classe [CosmosClient](/dotnet/api/microsoft.azure.cosmos.cosmosclient) peut créer une base de données.

1. Copiez et collez la méthode `CreateDatabaseAsync` en dessous de votre méthode `GetStartedDemoAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateDatabaseAsync&highlight=7)]

    `CreateDatabaseAsync` crée une base de données avec l’ID `FamilyDatabase` s’il n’existe pas déjà, sinon avec l’ID spécifié dans le champ `databaseId`.

1. Copiez et collez le code ci-dessous où vous instanciez le CosmosClient pour qu’il appelle la méthode **CreateDatabaseAsync** que vous venez d’ajouter.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);

        //ADD THIS PART TO YOUR CODE
        await this.CreateDatabaseAsync();
    }
    ```

    Votre *Program.cs* doit maintenant se présenter comme ceci, avec vos point de terminaison et clé primaire renseignés.

    ```csharp
    using System;
    using System.Threading.Tasks;
    using System.Configuration;
    using System.Collections.Generic;
    using System.Net;
    using Microsoft.Azure.Cosmos;

    namespace CosmosGettingStartedTutorial
    {
        class Program
        {
            // The Azure Cosmos DB endpoint for running this sample.
            private static readonly string EndpointUri = "<your endpoint here>";
            // The primary key for the Azure Cosmos account.
            private static readonly string PrimaryKey = "<your primary key>";

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
                    Console.WriteLine("Beginning operations...");
                    Program p = new Program();
                    await p.GetStartedDemoAsync();
                }
                catch (CosmosException de)
                {
                    Exception baseException = de.GetBaseException();
                    Console.WriteLine("{0} error occurred: {1}\n", de.StatusCode, de);
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: {0}\n", e);
                }
                finally
                {
                    Console.WriteLine("End of demo, press any key to exit.");
                    Console.ReadKey();
                }
            }

            /// <summary>
            /// Entry point to call methods that operate on Azure Cosmos DB resources in this sample
            /// </summary>
            public async Task GetStartedDemoAsync()
            {
                // Create a new instance of the Cosmos Client
                this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
                await this.CreateDatabaseAsync();
            }

            /// <summary>
            /// Create the database if it does not exist
            /// </summary>
            private async Task CreateDatabaseAsync()
            {
                // Create a new database
                this.database = await this.cosmosClient.CreateDatabaseIfNotExistsAsync(databaseId);
                Console.WriteLine("Created Database: {0}\n", this.database.Id);
            }
        }
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

   > [!NOTE]
   > Si vous recevez une erreur « Exception 503 - Service indisponible », il est possible que les [ports](sql-sdk-connection-modes.md#service-port-ranges) nécessaires pour le mode de connectivité directe soient bloqués par un pare-feu. Pour résoudre ce problème, ouvrez les ports nécessaires ou utilisez la connectivité en mode de passerelle, comme indiqué dans le code suivant :
   ```csharp
     // Create a new instance of the Cosmos Client in Gateway mode
     this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey, new CosmosClientOptions()
            {
                ConnectionMode = ConnectionMode.Gateway
            });
   ```

Félicitations ! Vous avez réussi à créer une base de données Azure Cosmos.  

## <a name="step-5-create-a-container"></a><a id="CreateColl"></a>Étape 5 : Créez un conteneur.

> [!WARNING]
> La méthode `CreateContainerIfNotExistsAsync` crée un conteneur, ce qui a des répercussions sur les tarifs. Pour plus d'informations, visitez notre [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).
>
>

Vous pouvez créer un conteneur en utilisant la méthode [**CreateContainerIfNotExistsAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync?view=azure-dotnet&preserve-view=true#Microsoft_Azure_Cosmos_Database_CreateContainerIfNotExistsAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) ou [**CreateContainerAsync**](/dotnet/api/microsoft.azure.cosmos.database.createcontainerasync?view=azure-dotnet&preserve-view=true#Microsoft_Azure_Cosmos_Database_CreateContainerAsync_Microsoft_Azure_Cosmos_ContainerProperties_System_Nullable_System_Int32__Microsoft_Azure_Cosmos_RequestOptions_System_Threading_CancellationToken_) dans la classe `CosmosDatabase`. Un conteneur est composé d’éléments (documents JSON dans le cas de l’API SQL) et d’une logique d’application côté serveur en JavaScript, par exemple des procédures stockées, des fonctions définies par l’utilisateur et des déclencheurs.

1. Copiez et collez la méthode `CreateContainerAsync` en dessous de votre méthode `CreateDatabaseAsync`. `CreateContainerAsync` crée un conteneur avec l’ID `FamilyContainer` s’il n’existe pas déjà, avec l’ID spécifié dans le champ `containerId` partitionné par la propriété `LastName`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=CreateContainerAsync&highlight=9)]

1. Copiez et collez le code ci-dessous, dans lequel vous avez instancié le CosmosClient pour qu’il appelle la méthode **CreateContainer** que vous venez d’ajouter.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();

        //ADD THIS PART TO YOUR CODE
        await this.CreateContainerAsync();
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à créer un conteneur Azure Cosmos.  

## <a name="step-6-add-items-to-the-container"></a><a id="CreateDoc"></a>Étape 6 : Ajouter des éléments au conteneur

La méthode [**CreateItemAsync**](/dotnet/api/microsoft.azure.cosmos.container.createitemasync?view=azure-dotnet&preserve-view=true#Microsoft_Azure_Cosmos_Container_CreateItemAsync__1___0_System_Nullable_Microsoft_Azure_Cosmos_PartitionKey__Microsoft_Azure_Cosmos_ItemRequestOptions_System_Threading_CancellationToken_) de la classe `CosmosContainer` peut créer un élément. Quand vous utilisez l’API SQL, les éléments sont projetés sous forme de documents, à savoir du contenu JSON arbitraire défini par l’utilisateur. Vous pouvez maintenant insérer un élément dans votre conteneur Azure Cosmos.

Tout d’abord, nous allons créer une classe `Family` qui représente les objets stockés dans Azure Cosmos DB pour cet exemple. Nous allons aussi créer les sous-classes `Parent`, `Child`, `Pet`, `Address` qui sont utilisées dans `Family`. L’élément doit avoir une propriété `Id` sérialisée comme `id` dans le JSON.

1. Sélectionnez Ctrl+Maj+A pour ouvrir **Ajouter un nouvel élément**. Ajoutez une nouvelle classe `Family.cs` à votre projet.

    :::image type="content" source="./media/sql-api-get-started/cosmos-getting-started-add-family-class-2019.png" alt-text="Capture d’écran de l’ajout d’une nouvelle classe Family.cs dans le projet":::

1. Copiez et collez la classe `Family`, `Parent`, `Child``Pet` et `Address` dans `Family.cs`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Family.cs)]


1. De retour dans *Program.cs* , ajoutez la méthode `AddItemsToContainerAsync` après votre méthode `CreateContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=AddItemsToContainerAsync)]


    Le code procède à une vérification pour s’assurer qu’il n’existe pas d’élément avec le même ID. Nous allons insérer deux éléments, un pour la *famille Andersen* et l’autre pour la *famille Wakefield*.

1. Ajoutez un appel à `AddItemsToContainerAsync` dans la méthode `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.AddItemsToContainerAsync();
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à créer deux éléments Azure Cosmos.  

## <a name="step-7-query-azure-cosmos-db-resources"></a><a id="Query"></a>Étape 7 : Interroger les ressources Azure Cosmos DB

Azure Cosmos DB prend en charge les requêtes enrichies sur les documents JSON stockés dans chaque conteneur. Pour plus d’informations, consultez [Bien démarrer avec les requêtes SQL](./sql-query-getting-started.md). L’exemple de code suivant montre comment exécuter une requête sur les éléments que nous avons insérés à l’étape précédente.

1. Copiez et collez la méthode `QueryItemsAsync` après votre méthode `AddItemsToContainerAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=QueryItemsAsync&highlight=10-11,17-18)]

1. Ajoutez un appel à ``QueryItemsAsync`` dans la méthode ``GetStartedDemoAsync``.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();

        //ADD THIS PART TO YOUR CODE
        await this.QueryItemsAsync();
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à interroger un conteneur Azure Cosmos.

## <a name="step-8-replace-a-json-item"></a><a id="ReplaceItem"></a>Étape 8 : Remplacer un élément JSON

À présent, nous allons mettre à jour un élément dans Azure Cosmos DB. Nous allons modifier la propriété `IsRegistered` de `Family` et le niveau scolaire (`Grade`) de l’un des enfants.

1. Copiez et collez la méthode `ReplaceFamilyItemAsync` après votre méthode `QueryItemsAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=ReplaceFamilyItemAsync&highlight=15)]

1. Ajoutez un appel à `ReplaceFamilyItemAsync` dans la méthode `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();

        //ADD THIS PART TO YOUR CODE
        await this.ReplaceFamilyItemAsync();
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à remplacer un élément Azure Cosmos.

## <a name="step-9-delete-item"></a><a id="DeleteDocument"></a>Étape 9 : Delete item (Supprimer un élément)

À présent, nous allons supprimer un élément dans Azure Cosmos DB.

1. Copiez et collez la méthode `DeleteFamilyItemAsync` après votre méthode `ReplaceFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteFamilyItemAsync&highlight=10)]

1. Ajoutez un appel à `DeleteFamilyItemAsync` dans la méthode `GetStartedDemoAsync`.

    ```csharp
    public async Task GetStartedDemoAsync()
    {
        // Create a new instance of the Cosmos Client
        this.cosmosClient = new CosmosClient(EndpointUri, PrimaryKey);
        await this.CreateDatabaseAsync();
        await this.CreateContainerAsync();
        await this.AddItemsToContainerAsync();
        await this.QueryItemsAsync();
        await this.ReplaceFamilyItemAsync();

        //ADD THIS PART TO YOUR CODE
        await this.DeleteFamilyItemAsync();
    }
    ```

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à supprimer un élément Azure Cosmos.

## <a name="step-10-delete-the-database"></a><a id="DeleteDatabase"></a>Étape 10 : Supprimer la base de données

Maintenant, nous allons supprimer notre base de données. Supprimer la base de données créée revient à supprimer la base de données et toutes les ressources enfants. Les ressources incluent des conteneurs, des éléments ainsi que des procédures stockées, des fonctions définies par l’utilisateur et des déclencheurs. Nous supprimons aussi l’instance `CosmosClient`.

1. Copiez et collez la méthode `DeleteDatabaseAndCleanupAsync` après votre méthode `DeleteFamilyItemAsync`.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=DeleteDatabaseAndCleanupAsync)]

1. Ajoutez un appel à ``DeleteDatabaseAndCleanupAsync`` dans la méthode ``GetStartedDemoAsync``.

    [!code-csharp[](~/cosmos-dotnet-getting-started/CosmosGettingStartedTutorial/Program.cs?name=GetStartedDemoAsync&highlight=14)]

1. Sélectionnez F5 pour exécuter votre application.

Félicitations ! Vous avez réussi à supprimer une base de données Azure Cosmos.

## <a name="step-11-run-your-c-console-application-all-together"></a><a id="Run"></a>Étape 11 : Exécuter votre application de console C#

Sélectionnez F5 dans Visual Studio pour générer et exécuter l’application en mode débogage.

La sortie de votre application entière doit s’afficher dans une fenêtre de console. La sortie affiche les résultats des requêtes que nous avons ajoutées. Elle doit correspondre à l’exemple de texte ci-dessous.

```cmd
Beginning operations...

Created Database: FamilyDatabase

Created Container: FamilyContainer

Created item in database with id: Andersen.1 Operation consumed 11.43 RUs.

Created item in database with id: Wakefield.7 Operation consumed 14.29 RUs.

Running query: SELECT * FROM c WHERE c.LastName = 'Andersen'

        Read {"id":"Andersen.1","LastName":"Andersen","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":false}

Updated Family [Wakefield,Wakefield.7].
        Body is now: {"id":"Wakefield.7","LastName":"Wakefield","Parents":[{"FamilyName":"Wakefield","FirstName":"Robin"},{"FamilyName":"Miller","FirstName":"Ben"}],"Children":[{"FamilyName":"Merriam","FirstName":"Jesse","Gender":"female","Grade":6,"Pets":[{"GivenName":"Goofy"},{"GivenName":"Shadow"}]},{"FamilyName":"Miller","FirstName":"Lisa","Gender":"female","Grade":1,"Pets":null}],"Address":{"State":"NY","County":"Manhattan","City":"NY"},"IsRegistered":true}

Deleted Family [Wakefield,Wakefield.7]

Deleted Database: FamilyDatabase

End of demo, press any key to exit.
```

Félicitations ! Vous avez terminé le didacticiel et vous disposez d’une application de console C# qui fonctionne !

## <a name="get-the-complete-tutorial-solution"></a><a id="GetSolution"></a> Obtenir la solution complète du didacticiel

Si vous n’avez pas eu le temps de terminer les étapes de ce tutoriel ou que vous voulez simplement télécharger les exemples de code, vous pouvez la télécharger.

Pour générer la solution `GetStarted`, vous avez besoin des prérequis suivants :

* Un compte Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [compte gratuit](https://azure.microsoft.com/free/).
* Un [compte Azure Cosmos DB][cosmos-db-create-account].
* La solution [GetStarted](https://github.com/Azure-Samples/cosmos-dotnet-getting-started) disponible sur GitHub.

Pour restaurer les références au kit SDK .NET Azure Cosmos DB dans Visual Studio, cliquez avec le bouton droit sur la solution dans l’ **Explorateur de solutions** , puis sélectionnez **Restaurer des packages NuGet**. Ensuite, dans le fichier *App.config* , mettez à jour les valeurs `EndPointUri` et `PrimaryKey` comme décrit à l’ [Étape 3 : Se connecter à un compte Azure Cosmos DB](#Connect).

Voilà, générez l’élément et c’est parti !

## <a name="next-steps"></a>Étapes suivantes

* Vous voulez un didacticiel ASP.NET MVC plus complexe ? Voir le [tutoriel : Développer une application web ASP.NET Core MVC avec Azure Cosmos DB à l’aide du kit SDK .NET](sql-api-dotnet-application.md).
* Vous voulez tester la mise à l’échelle et les performances avec Azure Cosmos DB ? Consultez [Test des performances et de la mise à l’échelle avec Azure Cosmos DB](performance-testing.md).
* Pour savoir comment superviser les demandes, l’utilisation et le stockage Azure Cosmos DB, consultez [Superviser les métriques de performances et de stockage dans Azure Cosmos DB](./monitor-cosmos-db.md).
* Pour exécuter des requêtes sur notre exemple de jeu de données, consultez [Query Playground](https://www.documentdb.com/sql/demo).
* Pour en savoir plus sur Azure Cosmos DB, consultez la page [Bienvenue dans Azure Cosmos DB](./introduction.md).

[cosmos-db-create-account]: create-sql-api-java.md#create-a-database-account