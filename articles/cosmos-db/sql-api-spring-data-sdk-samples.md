---
title: 'API SQL Azure Cosmos DB : Exemples Spring Data v3'
description: Recherchez des exemples Spring Data v3 sur GitHub pour les tâches courantes utilisant l’API SQL Azure Cosmos DB, y compris les opérations CRUD.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 09/23/2020
ms.custom: devx-track-java
ms.author: anfeldma
ms.openlocfilehash: 378bb891c8539a6cf3d61f6511a0f58377d2bfd2
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93091138"
---
# <a name="azure-cosmos-db-sql-api-spring-data-azure-cosmos-db-v3-examples"></a>API SQL Azure Cosmos DB : Exemples Spring Data Azure Cosmos DB v3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Exemples du kit SDK .NET V2](sql-api-dotnet-samples.md)
> * [Exemples du kit SDK .NET V3](sql-api-dotnet-v3sdk-samples.md)
> * [Exemples du kit SDK Java V4](sql-api-java-sdk-samples.md)
> * [Exemples du kit SDK Spring Data V3](sql-api-spring-data-sdk-samples.md)
> * [Exemples Node.js](sql-api-nodejs-samples.md)
> * [Exemples Python](sql-api-python-samples.md)
> * [Galerie d’exemples de code Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

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

> [!IMPORTANT]  
>[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
>  
>- Vous pouvez [activer les avantages d’abonnement Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) : Votre abonnement Visual Studio vous donne droit chaque mois à des crédits dont vous pouvez vous servir pour les services Azure payants.
>
>[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]
>

Le référentiel GitHub [azure-spring-data-cosmos-java-sql-api-samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples) contient la dernière version des exemples d’applications qui exécutent des opérations CRUD, ainsi que d’autres opérations courantes sur les ressources Azure Cosmos DB. Cet article fournit :

* Des liens vers les tâches dans chacun des exemples de fichiers de projet Spring Data Azure Cosmos DB. 
* Des liens vers le contenu de référence d’API connexe.

**Composants requis**

Pour exécuter cet exemple d’application, vous avez besoin de ce qui suit :

* Kit de développement Java 8
* Spring Data Azure Cosmos DB v3

Vous pouvez également utiliser Maven pour obtenir les tout derniers fichiers binaires Spring Data Azure Cosmos DB v3 et les utiliser dans votre projet. Maven ajoute automatiquement toutes les dépendances nécessaires. Sinon, vous pouvez directement télécharger les dépendances répertoriées dans le fichier **pom.xml** et les ajouter à votre chemin d’accès de build.

```bash
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-spring-data-cosmos</artifactId>
    <version>LATEST</version>
</dependency>
```

**Exécution des exemples d'applications**

Clonez l’exemple de référentiel :
```bash
$ git clone https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples

$ cd azure-spring-data-cosmos-java-sql-api-samples
```

Vous pouvez exécuter les exemples à l’aide d’un environnement de développement intégré (Eclipse, IntelliJ ou VSCODE), ou depuis la ligne de commande en utilisant Maven.

Dans **application.properties** , ces variables d’environnement doivent être définies

```xml
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

afin d’accorder aux exemples un accès en lecture/écriture à votre compte, vos bases de données et vos conteneurs.

Votre IDE peut offrir la possibilité d’exécuter l’exemple de code Spring Data. Dans le cas contraire, vous pouvez utiliser la commande de terminal suivante pour exécuter l’exemple :

```bash
mvn spring-boot:run
```

## <a name="document-crud-examples"></a>Exemples de CRUD de document
Le fichier [samples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) montre comment effectuer les tâches suivantes. Pour en savoir plus sur les documents Azure Cosmos avant d’exécuter les exemples suivants, consultez l’article conceptuel [Utiliser des bases de données, des conteneurs et des éléments](account-databases-containers-items.md).

| Tâche | Informations de référence sur l'API |
| --- | --- |
| [Créer un document](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L46-L47) | CosmosRepository.save |
| [Lire un document par identifiant](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L56-L58) | CosmosRepository.derivedQueryMethod |
| [Enregistrer tous les documents](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L39-L41) | CosmosRepository.deleteAll |

## <a name="derived-query-method-examples"></a>Exemples de méthodes de requête dérivées
Le fichier [exemples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) montre comment effectuer les tâches suivantes. Pour en savoir plus sur les requêtes Azure Cosmos DB avant d’exécuter les exemples suivants, il peut s’avérer utile de lire l’article sur les [méthodes de requête dérivées de Baeldung dans Spring](https://www.baeldung.com/spring-data-derived-queries).

| [Rechercher des documents](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java#L73-L77) | CosmosRepository.derivedQueryMethod |

## <a name="custom-query-examples"></a>Exemples de requêtes personnalisées
Le fichier [exemples](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/SampleApplication.java) indique comment effectuer les tâches suivantes à l’aide de la syntaxe de requête SQL. Pour en savoir plus sur les informations de référence sur les requêtes SQL dans Azure Cosmos DB avant d’exécuter les exemples suivants, consultez [Exemples de requêtes SQL pour Azure Cosmos DB](./sql-query-getting-started.md). 


| Tâche | Informations de référence sur l'API |
| --- | --- |
| [Rechercher tous les documents](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L20-L22) | Annotation @Query |
| [Demander une égalité à l’aide de ==](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L24-L26) | Annotation @Query |
| [Demander une inégalité à l’aide des opérateurs != et NOT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L28-L38) | Annotation @Query |
| [Interroger à l’aide d’opérateurs de plage comme &gt;, &lt;&gt;, =, &lt; =](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L40-L42) | Annotation @Query |
| [Interroger en utilisant des opérateurs de plage sur des chaînes](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L44-L46) | Annotation @Query |
| [Soumettre des requêtes avec la clause ORDER BY](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L48-L50) | Annotation @Query |
| [Requête avec DISTINCT](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L52-L54) | Annotation @Query |
| [Soumettre des requêtes avec des fonctions d’agrégation](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L56-L62) | Annotation @Query |
| [Utiliser des sous-documents](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L64-L66) | Annotation @Query |
| [Interroger avec des jointures à l’intérieur d’un document](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L68-L85) | Annotation @Query |
| [Effectuer des requêtes à l’aide d’opérateurs de chaîne, d’opérateurs mathématiques et d’opérateurs de tableau](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/springexamples/quickstart/sync/UserRepository.java#L87-L97) | Annotation @Query |