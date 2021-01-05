---
title: Démarrage rapide - Utiliser Spring Data Azure Cosmos DB v3 pour créer une base de données de documents avec Azure Cosmos DB
description: Ce guide de démarrage rapide présente un exemple de code Spring Data Azure Cosmos DB v3 que vous pouvez utiliser pour vous connecter à l’API SQL Azure Cosmos DB et l’interroger.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: quickstart
ms.date: 10/06/2020
ms.author: anfeldma
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: f31eb0fa6dbb881f7a09b21b9dd4842fdfd291f5
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93090288"
---
# <a name="quickstart-build-a-spring-data-azure-cosmos-db-v3-app-to-manage-azure-cosmos-db-sql-api-data"></a>Démarrage rapide : Créer une application Spring Data Azure Cosmos DB v3 pour gérer les données de l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET V3](create-sql-api-dotnet.md)
> * [.NET V4](create-sql-api-dotnet-V4.md)
> * [Kit SDK Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> * [Node.JS](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)

Dans ce guide de démarrage rapide, vous allez créer et gérer un compte d’API SQL Azure Cosmos DB à partir du portail Azure et avec une application Spring Data Azure Cosmos DB v3 clonée à partir de GitHub. Tout d’abord, vous créez un compte d’API SQL Azure Cosmos DB à l’aide du portail Azure, puis vous créez une application Spring Boot à l’aide du connecteur Spring Data Azure Cosmos DB v3 et vous ajoutez des ressources à votre compte Cosmos DB en utilisant l’application Spring Boot. Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale.

> [!IMPORTANT]  
> Ces notes de publication sont destinées à la version 3 de Spring Data Azure Cosmos DB. Les [notes de publication de la version 2 sont accessibles ici](sql-api-sdk-java-spring-v2.md). 
>
> Spring Data Azure Cosmos DB prend uniquement en charge l’API SQL.
>
> Pour plus d’informations concernant Spring Data sur d’autres API d’Azure Cosmos DB, consultez les articles suivants :
> * [Spring Data pour Apache Cassandra avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
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
git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started.git
```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez directement passer à la section [Exécution de l’application](#run-the-app). 

### <a name="application-configuration-file"></a>Fichier de configuration de l’application

Ici, nous démontrons comment Spring Boot et Spring Data améliorent l’expérience utilisateur : le processus d’établissement d’un client Cosmos et de connexion à des ressources Cosmos relève désormais d’une configuration plutôt que de code. Au démarrage de l’application, Spring Boot gère tout ce qui est réutilisable à l’aide des paramètres définis dans **application.properties**  :

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

Une fois que vous avez créé un compte, une base de données et un conteneur Azure Cosmos DB, il vous suffit de renseigner le fichier de configuration pour que Spring Boot/Spring Data effectuent automatiquement les opérations suivantes : (1) créer une instance `CosmosClient` du SDK Java sous-jacent avec l’URI et la clé, puis (2) établir une connexion à la base de données et au conteneur. Vous êtes fin prêt : **pas d’autre code de gestion des ressources !**

### <a name="java-source"></a>Source Java

La valeur ajoutée par Spring Data vient aussi de son interface simple, propre, normalisée et indépendante de la plateforme utilisable sur des magasins de données. En s’appuyant sur l’exemple Spring Data GitHub lié ci-dessus, voici des exemples de requête CRUD pour la manipulation des documents Azure Cosmos DB avec Spring Data Azure Cosmos DB.

* Création et mise à jour d’éléments à l’aide de la méthode `save`.

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Create)]
   
* Les lectures de points à l’aide de la méthode de requête dérivée définie dans le dépôt. `findByIdAndLastName` effectue des lectures de points pour `UserRepository`. Les champs mentionnés dans le nom de la méthode amènent Spring Data à exécuter une lecture de point définie par les champs `id` et `lastName` :

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Read)]

* Suppressions d’éléments à l’aide de `deleteAll` :

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Delete)]

* Requête dérivée basée sur le nom de la méthode du dépôt. Spring Data implémente la méthode `UserRepository` `findByFirstName` en tant que requête SQL du SDK Java sur le champ `firstName` (cette requête n’a pas pu être implémentée en tant que lecture de point) :

    [!code-java[](~/spring-data-azure-cosmos-db-sql-tutorial/azure-spring-data-cosmos-java-getting-started/src/main/java/com/azure/spring/data/cosmostutorial/SampleApplication.java?name=Query)]

## <a name="run-the-app"></a>Exécuter l’application

Maintenant, retournez dans le portail Azure afin d’obtenir les informations de votre chaîne de connexion et lancer l’application avec vos informations sur le point de terminaison. Cette opération permet à votre application de communiquer avec votre base de données hébergée.

1. Dans la fenêtre de terminal git, ouvrez `cd` le dossier d’exemple de code.

    ```bash
    cd azure-spring-data-cosmos-java-sql-api-getting-started/azure-spring-data-cosmos-java-getting-started/
    ```

2. Dans la fenêtre de terminal git, utilisez la commande suivante pour installer les packages Spring Data Azure Cosmos DB nécessaires.

    ```bash
    mvn clean package
    ```

3. Dans la fenêtre de terminal git, utilisez la commande suivante pour démarrer l’application Spring Data Azure Cosmos DB :

    ```bash
    mvn spring-boot:run
    ```
    
4. L’application charge **application.properties** et connecte les ressources dans votre compte Azure Cosmos DB.
5. L’application effectue des opérations CRUD de points décrites ci-dessus.
6. L’application exécute une requête dérivée.
7. L’application ne supprime pas vos ressources. Revenez au portail pour [nettoyer les ressources](#clean-up-resources) dans votre compte pour éviter des frais.

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte d’API SQL Azure Cosmos DB, à créer une base de données de documents et un conteneur avec l’Explorateur de données et à exécuter une application Spring Data pour effectuer les mêmes opérations programmatiquement. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)