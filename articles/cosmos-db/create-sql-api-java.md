---
title: Démarrage rapide – Utiliser Java pour créer une base de données de documents à l’aide d’Azure Cosmos DB
description: Ce guide de démarrage rapide présente un exemple de code Java que vous pouvez utiliser pour vous connecter à l’API SQL d’Azure Cosmos DB et pour l’interroger.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: anfeldma
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 4b62b591c408f663fd28d5077af924f785ee66c8
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93090407"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-sql-api-data"></a>Démarrage rapide : Créer une application Java pour gérer les données de l’API SQL d’Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [Kit SDK Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Node.JS](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Dans ce guide de démarrage rapide, vous allez créer et gérer un compte d’API SQL Azure Cosmos DB à partir du portail Azure et avec une application Java clonée à partir de GitHub. Vous allez d’abord créer un compte d’API SQL Azure Cosmos DB à l’aide du portail Azure, puis une application Java en utilisant le SDK Java SQL. Vous ajouterez ensuite des ressources à votre compte Cosmos DB à l’aide de l’application Java. Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale.

> [!IMPORTANT]  
> Ce guide de démarrage rapide s’applique uniquement au SDK Java v4 Azure Cosmos DB. Pour plus d’informations, consultez les [Notes de publication](sql-api-sdk-java-v4.md) du SDK Java v4 Azure Cosmos DB, le [Référentiel Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), les [conseils en matière de performances](performance-tips-java-sdk-v4-sql.md) du SDK Java v4 Azure Cosmos DB et le [guide de dépannage](troubleshoot-java-sdk-v4-sql.md) du SDK Java v4 Azure Cosmos DB. Si vous utilisez actuellement une version antérieure à v4, consultez le guide [Migrer votre application pour utiliser le SDK Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) pour obtenir de l’aide sur la mise à niveau vers v4.
>

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure. Vous pouvez aussi utiliser l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) avec l’URI `https://localhost:8081` et la clé `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`.
- [Kit de développement Java (JDK) 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Faites pointer la variable d’environnement `JAVA_HOME` vers le dossier dans lequel le JDK est installé.
- [Archive binaire Maven](https://maven.apache.org/download.cgi). Sur Ubuntu, exécutez `apt-get install maven` pour installer Maven.
- [Git](https://www.git-scm.com/downloads). Sur Ubuntu, exécutez `sudo apt-get install git` pour installer Git.

## <a name="introductory-notes"></a>Remarques d’introduction

*Structure d’un compte Cosmos DB.* Quels que soient l’API et le langage de programmation, un *compte* Cosmos DB contient zéro, une ou plusieurs *bases de données* , une *base de données* contient zéro, un ou plusieurs *conteneurs* et un *conteneur* contient zéro, un ou plusieurs éléments, comme illustré dans le schéma ci-dessous :

:::image type="content" source="./media/account-databases-containers-items/cosmos-entities.png" alt-text="Entités du compte Azure Cosmos" border="false":::

Pour en savoir plus sur les bases de données, les conteneurs et les éléments, consultez [cette page](account-databases-containers-items.md). Certaines propriétés importantes sont définies au niveau du conteneur, notamment le *débit provisionné* et la *clé de partition*. 

Le débit provisionné est mesuré en unités de requête ( *RU* ), pour lesquelles un prix est fixé. Il s’agit d’un facteur déterminant du coût d’exploitation du compte. Vous pouvez choisir un débit provisionné par conteneur ou par base de données. Toutefois, il est généralement préférable de spécifier le débit au niveau du conteneur. Pour en savoir plus sur le provisionnement du débit consultez [cette page](set-throughput.md).

À mesure que des éléments sont insérés dans un conteneur Cosmos DB, la base de données croît horizontalement : les capacités de stockage et de calcul augmentent pour assurer le traitement des requêtes. Les capacités de stockage et de calcul sont ajoutées par unités séparées appelées *partitions*. Vous devez choisir dans vos documents un champ qui sera la clé de partition qui mappe chaque document à une partition. Les partitions sont gérées de la façon suivante : à chaque partition est attribuée une section relativement équivalente de la plage de valeurs de clé de partition. Il est donc recommandé de choisir une clé de partition relativement aléatoire ou distribuée uniformément. Dans le cas contraire, un plus grand nombre de requêtes sera adressé à certaines partitions ( *partitions à chaud* ) et un nombre de requêtes sensiblement inférieur sera adressé à d’autres partitions ( *partitions à froid* ). Cette situation doit être évitée. Pour en savoir plus sur le partitionnement, consultez [cette page](partitioning-overview.md).

## <a name="create-a-database-account"></a>Création d’un compte de base de données

Pour être en mesure de créer une base de données de documents, vous devez avoir préalablement créé un compte d’API SQL avec Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-container"></a>Ajouter un conteneur

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

<a id="add-sample-data"></a>
## <a name="add-sample-data"></a>Ajouter un exemple de données

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Interroger vos données

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

À présent, travaillons sur le code. Nous allons cloner une application API SQL à partir de GitHub, configurer la chaîne de connexion et l’exécuter. Vous verrez combien il est facile de travailler par programmation avec des données. 

Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

```bash
git clone https://github.com/Azure-Samples/azure-cosmos-java-getting-started.git
```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez directement passer à la section [Exécution de l’application](#run-the-app). 


# <a name="sync-api"></a>[API synchrone](#tab/sync)

### <a name="managing-database-resources-using-the-synchronous-sync-api"></a>Gestion des ressources de base de données à l’aide de l’API synchrone (sync)

* Initialisation de `CosmosClient`. Le `CosmosClient` fournit une représentation logique côté client du service de base de données Azure Cosmos. Ce client est utilisé pour configurer et exécuter des requêtes auprès du service.
    
    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateSyncClient)]

* Création de `CosmosDatabase`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateDatabaseIfNotExists)]

* Création de `CosmosContainer`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateContainerIfNotExists)]

* Création d’éléments avec la méthode `createItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateItem)]
   
* Les lectures de points sont effectuées avec la méthode `readItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=ReadItem)]

* Les requêtes SQL sur JSON sont effectuées avec la méthode `queryItems`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=QueryItems)]

# <a name="async-api"></a>[API asynchrone](#tab/async)

### <a name="managing-database-resources-using-the-asynchronous-async-api"></a>Gestion des ressources de base de données avec l’API asynchrone (async)

* Les appels de l’API async sont retournés immédiatement, sans attendre une réponse du serveur. À la lumière de cela, les extraits de code suivants illustrent des modèles de conception adaptés pour accomplir toutes les tâches de gestion précédentes à l’aide de l’API async.

* Initialisation de `CosmosAsyncClient`. Le `CosmosAsyncClient` fournit une représentation logique côté client du service de base de données Azure Cosmos. Ce client est utilisé pour configurer et exécuter des requêtes asynchrones auprès du service.
    
    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=CreateAsyncClient)]

* Création de `CosmosAsyncDatabase`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateDatabaseIfNotExists)]

* Création de `CosmosAsyncContainer`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/sync/SyncMain.java?name=CreateContainerIfNotExists)]

* Comme pour l’API sync, les éléments sont créés avec la méthode `createItem`. Cet exemple montre comment effectuer efficacement de nombreuses requêtes `createItem` asynchrones en s’abonnant à un flux réactif qui envoie les requêtes et affiche les notifications. Dans la mesure où cet exemple simple s’exécute jusqu’à la fin avant de se terminer, les instances `CountDownLatch` servent à garantir que le programme ne se termine pas durant la création d’un élément. **La bonne pratique en matière de programmation asynchrone est de ne pas bloquer les appels asynchrones. En réalité, les requêtes sont générées à partir d’une boucle main() qui s’exécute indéfiniment, ce qui évite d’avoir à verrouiller les appels asynchrones.**

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=CreateItem)]
   
* Comme avec l’API sync, les lectures de points sont effectuées avec la méthode `readItem`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=ReadItem)]

* De même, les requêtes SQL sur JSON sont effectuées avec la méthode `queryItems`.

    [!code-java[](~/azure-cosmosdb-java-v4-getting-started/src/main/java/com/azure/cosmos/sample/async/AsyncMain.java?name=QueryItems)]

---

## <a name="run-the-app"></a>Exécuter l’application

Maintenant, retournez dans le portail Azure afin d’obtenir les informations de votre chaîne de connexion et lancer l’application avec vos informations sur le point de terminaison. Cette opération permet à votre application de communiquer avec votre base de données hébergée.

1. Dans la fenêtre de terminal git, ouvrez `cd` le dossier d’exemple de code.

    ```bash
    cd azure-cosmos-java-getting-started
    ```

2. Dans la fenêtre de terminal git, utilisez la commande suivante pour installer les packages Java requis.

    ```bash
    mvn package
    ```

3. Dans la fenêtre de terminal git, utilisez la commande suivante pour démarrer l’application Java (remplacez SYNCASYNCMODE par `sync` ou `async` en fonction de l’exemple de code que vous voulez exécuter, remplacez YOUR_COSMOS_DB_HOSTNAME par la valeur d’URI entre guillemets du portail et remplacez YOUR_COSMOS_DB_MASTER_KEY par la clé primaire entre guillemets du portail)

    ```bash
    mvn exec:java@SYNCASYNCMODE -DACCOUNT_HOST=YOUR_COSMOS_DB_HOSTNAME -DACCOUNT_KEY=YOUR_COSMOS_DB_MASTER_KEY

    ```

    La fenêtre de terminal affiche une notification vous informant que la base de données FamilyDB a été créée. 
    
4. L’application crée une base de données avec le nom `AzureSampleFamilyDB`
5. L’application crée un conteneur avec le nom `FamilyContainer`
6. L’application effectue des lectures de points en utilisant les ID d’objet et la valeur de la clé de partition (lastName dans notre exemple). 
7. L’application interroge les éléments pour récupérer toutes les familles dont le nom de famille est dans (« Andersen », « Wakefield », « Johnson »)

7. L’application ne supprime pas les ressources créées. Revenez sur le portail pour [nettoyez les ressources](#clean-up-resources).  depuis votre compte afin d’éviter les frais.

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte d’API SQL Azure Cosmos DB, à créer une base de données de documents et un conteneur avec l’Explorateur de données et à exécuter une application Java pour effectuer les mêmes opérations programmatiquement. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)
