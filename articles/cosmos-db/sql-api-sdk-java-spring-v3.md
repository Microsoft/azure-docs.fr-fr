---
title: Spring Data Azure Cosmos DB v3 pour les notes de publication et les ressources de l’API SQL
description: Découvrez Spring Data Azure Cosmos DB v3 pour l’API SQL, notamment les dates de mise en production, les dates de mise hors service et les modifications apportées entre chaque version du Kit de développement logiciel (SDK) Java asynchrone SQL Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/18/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 64054a2bb5c1f7e17eef87c3babb28137b6c912a
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097122"
---
# <a name="spring-data-azure-cosmos-db-v3-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v3 pour l’API Core (SQL) : Notes de publication et ressources
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](sql-api-sdk-dotnet-standard.md)
> * [SDK .NET v2](sql-api-sdk-dotnet.md)
> * [SDK .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK .NET Change Feed v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.JS](sql-api-sdk-node.md)
> * [SDK Java v4](sql-api-sdk-java-v4.md)
> * [SDK Java Async v2](sql-api-sdk-async-java.md)
> * [SDK Java Sync v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark Connector](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [API REST Resource Provider](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Exécuteur en bloc – .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Exécuteur en bloc – Java](sql-api-sdk-bulk-executor-java.md)

Spring Data Azure Cosmos DB v3 pour Core (SQL) permet aux développeurs d’utiliser Azure Cosmos DB dans des applications Spring. Spring Data Azure Cosmos DB expose l’interface de données Spring pour manipuler les bases de données et les collections, travailler avec des documents et émettre des requêtes. Les API synchrones et asynchrones (réactives) sont prises en charge dans le même artefact Maven. 

Spring Data Azure Cosmos DB est dépendant de l’infrastructure Spring Data. L’équipe du Kit de développement logiciel (SDK) Azure Cosmos DB publie des artefacts Maven pour les versions 2.2 et 2.3 de Spring Data.

[Spring Framework](https://spring.io/projects/spring-framework) est un modèle de programmation et de configuration qui rationalise le développement d’applications Java. Spring rationalise la « plomberie » des applications en injectant des dépendances. De nombreux développeurs apprécient Spring, car il rend la création et le test d’applications plus simples. [Spring Boot](https://spring.io/projects/spring-boot) étend cette prise en charge de la plomberie en vue du développement d’applications web et de microservices. [Spring Data](https://spring.io/projects/spring-data) est un modèle de programmation et une infrastructure permettant l’accès à des magasins de données tels qu’Azure Cosmos DB à partir du contexte d’une application Spring ou Spring Boot. 

Vous pouvez utiliser Spring Data Azure Cosmos DB dans vos applications [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

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

## <a name="start-here"></a>Démarrer ici

# <a name="explore"></a>[Explorer](#tab/explore)

<img src="media/sql-api-sdk-java-spring-v3/up-arrow.png" alt="explore the tabs above" width="80"/>

#### <a name="these-tabs-contain-basic-spring-data-azure-cosmos-db-samples"></a>Ces onglets contiennent des exemples Spring Data Azure Cosmos DB de base.

# <a name="pomxml"></a>[pom.xml](#tab/pom)

### <a name="configure-dependencies"></a>Configurer des dépendances

  ```xml
  <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-spring-data-cosmos</artifactId>
      <version>latest</version>
  </dependency>
  ```

# <a name="connect"></a>[Connexion](#tab/connect)

### <a name="connect"></a>Se connecter

Spécifiez les détails du compte et du conteneur Azure Cosmos DB. Spring Data Azure Cosmos DB crée automatiquement le client et se connecte au conteneur.

[application.properties](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started/blob/main/azure-spring-data-cosmos-java-getting-started/src/main/resources/application.properties) :
```
cosmos.uri=${ACCOUNT_HOST}
cosmos.key=${ACCOUNT_KEY}
cosmos.secondaryKey=${SECONDARY_ACCOUNT_KEY}

dynamic.collection.name=spel-property-collection
# Populate query metrics
cosmos.queryMetricsEnabled=true
```

# <a name="doc-ops"></a>[Ops doc](#tab/docs)

### <a name="document-operations"></a>Opérations de document

---

## <a name="resources"></a>Ressources

* **Contribution au Kit de développement logiciel (SDK)**  : [Référentiel Spring Data Azure Cosmos DB sur GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-spring-data-cosmos)

* **Tutoriel**  : [Didacticiel Spring Data Azure Cosmos DB sur GitHub](https://github.com/Azure-Samples/azure-spring-data-cosmos-java-sql-api-getting-started) 

[!INCLUDE[Release notes](~/azure-sdk-for-java-cosmos-db/sdk/cosmos/azure-spring-data-cosmos/CHANGELOG.md)]

## <a name="faq"></a>Questions fréquentes (FAQ)

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Apprenez-en davantage sur [Spring Framework](https://spring.io/projects/spring-framework).

Apprenez-en davantage sur [Spring Boot](https://spring.io/projects/spring-boot).

Apprenez-en davantage sur [Spring Data](https://spring.io/projects/spring-data).