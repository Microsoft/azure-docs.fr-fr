---
title: Créer une application .NET Framework/Core Azure Cosmos DB à l’aide de l’API Gremlin
description: Cet article présente un exemple de code .NET Framework/Core que vous pouvez utiliser pour vous connecter au service Azure Cosmos DB et pour l’interroger.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 02/21/2020
ms.author: chrande
ms.custom: devx-track-dotnet
ms.openlocfilehash: 1953f4a21df6f550320592fbe009834a7b573887
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360528"
---
# <a name="quickstart-build-a-net-framework-or-core-application-using-the-azure-cosmos-db-gremlin-api-account"></a>Démarrage rapide : Créer une application .NET Framework ou Core à l’aide du compte d’API Gremlin Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

> [!div class="op_single_selector"]
> * [Console Gremlin](create-graph-gremlin-console.md)
> * [.NET](create-graph-dotnet.md)
> * [Java](create-graph-java.md)
> * [Node.JS](create-graph-nodejs.md)
> * [Python](create-graph-python.md)
> * [PHP](create-graph-php.md)
>  

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Rapidement, vous avez la possibilité de créer et d’interroger des documents, des paires clé/valeur, et des bases de données orientées graphe, profitant tous de la distribution à l’échelle mondiale et des capacités de mise à l’échelle horizontale au cœur d’Azure Cosmos DB. 

Ce guide de démarrage rapide explique comment créer un compte, une base de données et un graphe (conteneur) [API Gremlin](graph-introduction.md) Azure Cosmos DB à l’aide du portail Azure. Par la suite, vous allez créer et exécuter une application console en utilisant le pilote [Gremlin.Net](https://tinkerpop.apache.org/docs/3.2.7/reference/#gremlin-DotNet) open-source.  

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas encore installé Visual Studio 2019, vous pouvez télécharger et utiliser la version **gratuite** [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Veillez à activer **le développement Azure** lors de l’installation de Visual Studio.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-database-account"></a>Création d’un compte de base de données

[!INCLUDE [cosmos-db-create-dbaccount-graph](../../includes/cosmos-db-create-dbaccount-graph.md)]

## <a name="add-a-graph"></a>Ajout d’un graphique

[!INCLUDE [cosmos-db-create-graph](../../includes/cosmos-db-create-graph.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

À présent, nous allons cloner une application API Gremlin à partir de GitHub, configurer la chaîne de connexion et l’exécuter. Vous verrez combien il est facile de travailler par programmation avec des données. 

1. Ouvrez une invite de commandes, créez un nouveau dossier nommé git-samples, puis fermez l’invite de commandes.

    ```bash
    md "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git comme Git Bash et utilisez la commande `cd` pour accéder au nouveau dossier d’installation pour l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-graph-gremlindotnet-getting-started.git
    ```

4. Ouvrez ensuite Visual Studio et le fichier de solution.

5. Restaurez les packages NuGet dans le projet. Sont concernés ici le pilote Gremlin.Net et le package Newtonsoft.Json.


6. Vous pouvez également installer le pilote Gremlin.Net manuellement à l’aide du gestionnaire de package Nuget ou de [l’utilitaire de ligne de commande nuget](/nuget/install-nuget-client-tools) : 

    ```bash
    nuget install Gremlin.Net
    ```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez passer à l’étape [Mise à jour de votre chaîne de connexion](#update-your-connection-string). 

Tous les extraits de code suivants proviennent du fichier Program.cs.

* Définissez vos paramètres de connexion en fonction du compte créé ci-dessus : 

   :::code language="csharp" source="~/azure-cosmosdb-graph-dotnet/GremlinNetSample/Program.cs" id="configureConnectivity":::

* Les commandes Gremlin à exécuter sont listées dans un dictionnaire :

   :::code language="csharp" source="~/azure-cosmosdb-graph-dotnet/GremlinNetSample/Program.cs" id="defineQueries":::

* Créez les objets de connexion `GremlinServer` et `GremlinClient` à l’aide des paramètres fournis ci-dessus :

   :::code language="csharp" source="~/azure-cosmosdb-graph-dotnet/GremlinNetSample/Program.cs" id="defineClientandServerObjects":::

* Exécutez chaque requête Gremlin à l’aide de l’objet `GremlinClient` avec une tâche asynchrone. Vous pouvez lire les requêtes Gremlin à partir du dictionnaire défini à l’étape précédente et les exécuter. Par la suite, vous récupérerez le résultat et lirez les valeurs, qui sont formatées sous forme de dictionnaire, à l’aide de la classe `JsonSerializer` du package Newtonsoft.Json :

   :::code language="csharp" source="~/azure-cosmosdb-graph-dotnet/GremlinNetSample/Program.cs" id="executeQueries":::

## <a name="update-your-connection-string"></a>Mise à jour de votre chaîne de connexion

Maintenant, retournez dans le portail Azure afin d’obtenir les informations de votre chaîne de connexion et de les copier dans l’application.

1. À partir du [Portail Azure](https://portal.azure.com/), accédez à votre compte de base de données de graphiques. Dans l’onglet **Vue d’ensemble** , vous pouvez voir deux points de terminaison : 
 
   **URI du SDK .NET**  : cette valeur est utilisée quand vous vous connectez au compte de graphiques à l’aide de la bibliothèque Microsoft.Azure.Graphs. 

   **Point de terminaison Gremlin**  : cette valeur est utilisée lorsque vous vous connectez au compte de graphiques à l’aide de la bibliothèque Gremlin.Net.

    :::image type="content" source="./media/create-graph-dotnet/endpoint.png" alt-text="Copier le point de terminaison":::

   Pour exécuter cet exemple, copiez la valeur **Point de terminaison Gremlin** , supprimez le numéro de port à la fin, et l’URI devient `https://<your cosmos db account name>.gremlin.cosmosdb.azure.com`. La valeur de point de terminaison doit ressembler à `testgraphacct.gremlin.cosmosdb.azure.com`

1. Ensuite, à partir du portail Azure, copiez la valeur **CLÉ PRIMAIRE** sous l’onglet **Clés**. 

1. Après avoir copié l’URI et la CLÉ PRIMAIRE de votre compte, enregistrez-les dans une nouvelle variable d’environnement sur la machine locale exécutant l’application. Pour définir la variable d’environnement, ouvrez une fenêtre d’invite de commandes, puis exécutez la commande suivante. Veillez à remplacer les valeurs <Your_Azure_Cosmos_account_URI> et <Your_Azure_Cosmos_account_PRIMARY_KEY>.

   ```console
   setx Host "<your Azure Cosmos account name>.gremlin.cosmosdb.azure.com"
   setx PrimaryKey "<Your_Azure_Cosmos_account_PRIMARY_KEY>"
   ```

1. Ouvrez le fichier *Program.cs* et remplacez les variables « database » et « container » par le nom de la base de données et le nom du conteneur (qui est également celui du graphe) créés précédemment.

    `private static string database = "your-database-name";` `private static string container = "your-container-or-graph-name";`

1. Enregistrez le fichier Program.cs. 

Vous venez de mettre à jour votre application avec toutes les informations nécessaires pour communiquer avec Azure Cosmos DB. 

## <a name="run-the-console-app"></a>Exécution de l’application console

Appuyez sur Ctrl + F5 pour exécuter l’application. L’application imprime les commandes de requête Gremlin et les résultats dans la console.

   La fenêtre de console affiche les sommets et les bords ajoutés au graphique. Lorsque le script se termine, appuyez sur ENTRÉE pour fermer la fenêtre de console.

## <a name="browse-using-the-data-explorer"></a>Navigation à l’aide de l’Explorateur de données

Vous pouvez maintenant retourner à l’Explorateur de données dans le Portail Azure pour parcourir et interroger vos nouvelles données graphiques.

1. Dans l’Explorateur de données, la nouvelle base de données apparaît dans le volet Graphique. Développez les nœuds de conteneur et de base de données, puis cliquez sur **Graphique**.

2. Cliquez sur le bouton **Appliquer un filtre** pour utiliser la requête par défaut et afficher tous les vertex dans le graphique. Les données générées par l’exemple d’application s’affichent dans le volet Graphiques.

    Vous pouvez agrandir et réduire le graphique, développer l’espace d’affichage du graphique, ajouter des vertex et déplacer des vertex sur la surface d’affichage.

    :::image type="content" source="./media/create-graph-dotnet/graph-explorer.png" alt-text="Afficher le graphique dans l’Explorateur de données du portail Azure":::

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte Azure Cosmos D, à créer un graphique à l’aide de l’Explorateur de données et à exécuter une application. Vous pouvez maintenant générer des requêtes plus complexes et implémenter une logique de traversée de graphique puissante, à l’aide de Gremlin. 

> [!div class="nextstepaction"]
> [Interroger à l’aide de Gremlin](tutorial-query-graph.md)