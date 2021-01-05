---
title: 'Azure Cosmos DB : Créer une application de liste de tâches avec Xamarin'
description: Cet article présente un exemple de code Xamarin que vous pouvez utiliser pour vous connecter à Azure Cosmos DB, et pour interroger les données
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/09/2020
ms.author: anfeldma
ms.custom: devx-track-csharp
ms.openlocfilehash: 91e89eaf215468f171974e5f3fd383691fdd6ebe
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096974"
---
# <a name="quickstart-build-a-todo-app-with-xamarin-using-azure-cosmos-db-sql-api-account"></a>Démarrage rapide : Générer une application todo avec Xamarin à l’aide du compte d’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [Kit SDK Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Node.JS](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Rapidement, vous avez la possibilité de créer et d’interroger des documents, des paires clé/valeur, et des bases de données orientées graphe, profitant tous de la distribution à l’échelle mondiale et des capacités de mise à l’échelle horizontale au cœur d’Azure Cosmos DB.

> [!NOTE]
> Des exemples de code pour un exemple d’application Xamarin complète incluant plusieurs offres Azure, y compris CosmosDB, sont disponibles sur GitHub, [ici](https://github.com/xamarinhq/app-geocontacts). Cette application montre l’affichage de contacts géographiquement dispersés et indique comment autoriser ces contacts à mettre à jour leur emplacement.

Ce démarrage rapide explique comment créer, à l’aide du portail Azure, un compte API SQL d’Azure Cosmos DB, une base de données de documents, ainsi qu’un conteneur. Vous allez ensuite générer et déployer une application mobile de liste de tâches qui repose sur [l’API .NET SQL](sql-api-sdk-dotnet.md) et [Xamarin](/xamarin/) en utilisant [Xamarin.Forms](/xamarin/) et le [modèle d’architecture MVVM](/xamarin/xamarin-forms/xaml/xaml-basics/data-bindings-to-mvvm).

:::image type="content" source="./media/create-sql-api-xamarin-dotnet/ios-todo-screen.png" alt-text="Application de liste de tâches Xamarin s’exécutant sur iOS":::

## <a name="prerequisites"></a>Prérequis

Si vous développez sur Windows et que vous n’avez pas encore installé Visual Studio 2019, vous pouvez télécharger et utiliser la version **gratuite** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Veillez à activer **le développement Azure** et le **développement mobile avec les charges de travail .NET** lors de l’installation de Visual Studio.

Si vous utilisez un Mac, vous pouvez télécharger la version **gratuite** de [Visual Studio pour Mac](https://www.visualstudio.com/vs/mac/).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

## <a name="create-a-database-account"></a>Création d’un compte de base de données

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-container"></a>Ajouter un conteneur

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

## <a name="add-sample-data"></a>Ajouter un exemple de données

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Interroger vos données

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Nous allons maintenant cloner l’application API SQL Xamarin à partir de GitHub, vérifier le code, récupérer les clés API et procéder à l’exécution. Vous verrez combien il est facile de travailler par programmation avec des données.

1. Ouvrez une invite de commandes, créez un nouveau dossier nommé git-samples, puis fermez l’invite de commandes.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git comme Git Bash et utilisez la commande `cd` pour accéder au nouveau dossier d’installation pour l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-sql-xamarin-getting-started.git
    ```

4. Dans Visual Studio, ouvrez **C:\git-samples\azure-cosmos-db-sql-xamarin-getting-started\src\ToDoItems.sln**. 

## <a name="obtain-your-api-keys"></a>Récupérer vos clés API

Revenez dans le portail Azure pour récupérer vos informations de clé API et les copier dans l’application.

1. Dans le [portail Azure](https://portal.azure.com/), dans votre compte d’API SQL Azure Cosmos DB, dans le volet de navigation de gauche, cliquez sur **Clés** , puis sur **Clés en lecture-écriture**. Vous utiliserez les boutons de copie sur le côté droit de l’écran pour copier l’URI et la clé primaire dans le fichier APIKeys.cs à l’étape suivante.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/keys.png" alt-text="Afficher et copier une clé d’accès dans le panneau Clés du portail Azure":::

2. Dans Visual Studio, ouvrez **ToDoItems.Core/Helpers/APIKeys.cs**.

3. Dans le portail Azure, à l’aide du bouton Copier, copiez la valeur **URI** et définissez-la comme valeur de la variable `CosmosEndpointUrl` dans APIKeys.cs.

    ```csharp
    //#error Enter the URL of your Azure Cosmos DB endpoint here
            public static readonly string CosmosEndpointUrl = "[URI Copied from Azure Portal]";
    ```

4. Dans le portail Azure, à l’aide du bouton Copier, copiez la valeur **CLÉ PRIMAIRE** et définissez-la comme valeur de `Cosmos Auth Key` dans APIKeys.cs.

    ```csharp
    //#error Enter the read/write authentication key of your Azure Cosmos DB endpoint here
            public static readonly string CosmosAuthKey = "[PRIMARY KEY copied from Azure Portal";
    ```

[!INCLUDE [cosmos-db-auth-key-info](../../includes/cosmos-db-auth-key-info.md)]

## <a name="review-the-code"></a>Vérifier le code

Cette solution montre comment créer une application de liste de tâches à l’aide de l’API SQL Azure Cosmos DB et Xamarin.Forms. L’application comporte deux onglets, le premier onglet inclut un affichage sous forme de liste présentant les éléments de liste de tâches qui ne sont pas encore terminés. Le deuxième onglet affiche les éléments de liste de tâches qui sont déjà terminés. En plus d’avoir un aperçu des éléments de liste de tâches non terminés via le premier onglet, vous pouvez ajouter des éléments de liste de tâches, en modifier et en marquer comme étant terminés.

:::image type="content" source="./media/create-sql-api-xamarin-dotnet/android-todo-screen.png" alt-text="Copier dans les données json et cliquer sur Enregistrer dans l’Explorateur de données du portail Azure":::

Le code de la solution ToDoItems contient :

* **ToDoItems.Core**
   * il s’agit d’un projet .NET Standard contenant un projet Xamarin.Forms et du code de logique d’application partagé qui gère les éléments de liste de tâches dans Azure Cosmos DB.
* **ToDoItems.Android**
  * ce projet contient l’application Android.
* **ToDoItems.iOS**
  * ce projet contient l’application iOS.

Maintenant, examinons rapidement la façon dont l’application communique avec Azure Cosmos DB.

* Le package NuGet [Microsoft.Azure.DocumentDb.Core](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/) doit être ajouté à tous les projets.
* La classe `ToDoItem` dans le dossier **ToDoItems.Core/Models** modélise les documents dans le conteneur **Éléments** créé ci-dessus. Notez que le nom de la propriété est sensible à la casse.
* La classe `CosmosDBService` dans le dossier **ToDoItems.Core/Services** encapsule la communication avec Azure Cosmos DB.
* Dans la classe `CosmosDBService`, une variable de type `DocumentClient` est disponible. `DocumentClient` permet de configurer et d’exécuter des requêtes sur le compte Azure Cosmos DB et est instancié :

    ```csharp
    docClient = new DocumentClient(new Uri(APIKeys.CosmosEndpointUrl), APIKeys.CosmosAuthKey);
    ```

* Lorsque vous interrogez un conteneur pour obtenir des documents, la méthode `DocumentClient.CreateDocumentQuery<T>` est utilisée, comme illustré ici dans la fonction `CosmosDBService.GetToDoItems` :

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=GetToDoItems)] 

    `CreateDocumentQuery<T>` accepte un URI qui pointe vers le conteneur créé lors de l’étape précédente. Vous avez également la possibilité de spécifier des opérateurs LINQ, tels qu’une clause `Where`. Dans ce cas, seuls les éléments de liste de tâches non terminés sont renvoyés.

    La fonction `CreateDocumentQuery<T>` est exécutée de façon synchrone et renvoie un objet `IQueryable<T>`. Toutefois, la méthode `AsDocumentQuery` convertit l’objet `IQueryable<T>` en objet `IDocumentQuery<T>` qui peut être exécuté de façon asynchrone. Cela permet de ne pas bloquer le thread d’UI pour les applications mobiles.

    La fonction `IDocumentQuery<T>.ExecuteNextAsync<T>` récupère la page de résultats d’Azure Cosmos DB, avec `HasMoreResults` pour vérifier si des résultats supplémentaires doivent encore être renvoyés.

> [!TIP]
> Plusieurs fonctions s’exécutant sur des documents et conteneurs Azure Cosmos acceptent un URI en tant que paramètre spécifiant l’adresse du conteneur ou du document. Cet URI est construit à l’aide de la classe `URIFactory`. Tous les URI de bases de données, conteneurs et documents peuvent être créés avec cette classe.

* La fonction `ComsmosDBService.InsertToDoItem` montre comment insérer un nouveau document :

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=InsertToDoItem)] 

    L’URI d’élément est indiqué, ainsi que l’élément à insérer.

* La fonction `CosmosDBService.UpdateToDoItem` montre comment remplacer un document existant par un autre :

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=UpdateToDoItem)] 

    Ici, un nouvel URI est nécessaire pour identifier de façon unique le document à remplacer. Il est obtenu à l’aide de `UriFactory.CreateDocumentUri`, en lui transmettant les noms de base de données et de conteneur, ainsi que l’ID du document.

    `DocumentClient.ReplaceDocumentAsync` remplace le document identifié par l’URI par celui spécifié en tant que paramètre.

* La suppression d’un élément est illustrée par la fonction `CosmosDBService.DeleteToDoItem` :

   [!code-csharp[](~/samples-cosmosdb-xamarin/src/ToDoItems.Core/Services/CosmosDBService.cs?name=DeleteToDoItem)] 

    Notez à nouveau l’URI de document unique qui est créé et transmis à la fonction `DocumentClient.DeleteDocumentAsync`.

## <a name="run-the-app"></a>Exécuter l’application

Vous venez de mettre à jour votre application avec toutes les informations nécessaires pour communiquer avec Azure Cosmos DB.

La procédure suivante montre comment exécuter l’application à l’aide du débogueur Visual Studio pour Mac.

> [!NOTE]
> L’utilisation de l’application en version Android est tout à fait identique ; les différences éventuelles seront signalées dans la procédure ci-dessous. Si vous souhaitez déboguer avec Visual Studio sur Windows, la documentation relative aux listes de tâches est également accessible pour [iOS ici](/xamarin/ios/deploy-test/debugging-in-xamarin-ios?tabs=vswin) et [Android ici](/xamarin/android/deploy-test/debugging/).

1. Commencez par sélectionner la plateforme à cibler en cliquant sur la liste déroulante en surbrillance et en sélectionnant ToDoItems.iOS pour iOS ou ToDoItems.Android pour Android.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ide-select-platform.png" alt-text="Sélection d’une plateforme pour déboguer dans Visual Studio pour Mac":::

2. Pour commencer à déboguer l’application, appuyez sur cmd + Entrée ou cliquez sur le bouton de lecture.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ide-start-debug.png" alt-text="Début du débogage dans Visual Studio pour Mac":::

3. Après le démarrage du simulateur iOS ou de l’émulateur Android, l’application affiche deux onglets en bas de l’écran pour iOS et en haut de l’écran pour Android. Le premier onglet présente les éléments de liste de tâches qui ne sont pas terminés et le deuxième affiche les éléments de liste de tâches qui sont terminés.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/ios-droid-started.png" alt-text="Écran de démarrage de l’application de liste de tâches":::

4. Pour terminer un élément de liste de tâches sur iOS, faites-le glisser à gauche > appuyez sur le bouton **Terminer**. Pour terminer un élément de liste de tâches sur Android, appuyez de façon prolongée sur l’élément > appuyez ensuite sur le bouton Terminer.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-complete.png" alt-text="Terminer un élément de liste de tâches":::

5. Pour modifier un élément de liste de tâches > appuyez sur l’élément > un nouvel écran s’affiche pour que vous puissiez saisir de nouvelles valeurs. En appuyant sur le bouton Enregistrer, les modifications seront conservées dans Azure Cosmos DB.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-edit.png" alt-text="Modifier un élément de liste de tâches":::

6. Pour ajouter un élément de liste de tâches > appuyez sur le bouton **Ajouter** dans le coin supérieur droit de l’écran d’accueil > une nouvelle page vide de modification s’affiche.

    :::image type="content" source="./media/create-sql-api-xamarin-dotnet/simulator-add.png" alt-text="Ajouter un élément de liste de tâches":::

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à créer un compte Azure Cosmos, à créer un conteneur à l’aide de l’Explorateur de données, et à concevoir et déployer une application Xamarin. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos.

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)