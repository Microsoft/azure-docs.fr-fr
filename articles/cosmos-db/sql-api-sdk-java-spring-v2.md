---
title: Spring Data Azure Cosmos DB v2 pour les notes de publication et les ressources de l’API SQL
description: Découvrez Spring Data Azure Cosmos DB v2 pour l’API SQL, notamment les dates de mise en production, les dates de mise hors service et les modifications apportées entre chaque version du Kit de développement logiciel (SDK) Java asynchrone SQL Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: d5c85095e767c0e92c22410054ac4f8fc5267660
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097139"
---
# <a name="spring-data-azure-cosmos-db-v2-for-core-sql-api-release-notes-and-resources"></a>Spring Data Azure Cosmos DB v2 pour l’API Core (SQL) : Notes de publication et ressources
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

 Spring Data Azure Cosmos DB v2 pour Core (SQL) permet aux développeurs d’utiliser Azure Cosmos DB dans des applications Spring. Spring Data Azure Cosmos DB expose l’interface de données Spring pour manipuler les bases de données et les collections, travailler avec des documents et émettre des requêtes. Les API synchrones et asynchrones (réactives) sont prises en charge dans le même artefact Maven. 

[Spring Framework](https://spring.io/projects/spring-framework) est un modèle de programmation et de configuration qui rationalise le développement d’applications Java. Spring rationalise la « plomberie » des applications en injectant des dépendances. De nombreux développeurs apprécient Spring, car il rend la création et le test d’applications plus simples. [Spring Boot](https://spring.io/projects/spring-boot) étend cette prise en charge de la plomberie en vue du développement d’applications web et de microservices. [Spring Data](https://spring.io/projects/spring-data) est un modèle de programmation permettant l’accès à des magasins de données tels qu’Azure Cosmos DB à partir du contexte d’une application Spring ou Spring Boot. 

Vous pouvez utiliser Spring Data Azure Cosmos DB dans vos applications [Azure Spring Cloud](https://azure.microsoft.com/services/spring-cloud/).

> [!IMPORTANT]  
> Ces notes de publication sont destinées à la version 2 de Spring Data Azure Cosmos DB. Les [notes de publication de la version 3 sont accessibles ici](sql-api-sdk-java-spring-v3.md). 
>
> Spring Data Azure Cosmos DB prend uniquement en charge l’API SQL.
>
> Pour plus d’informations concernant Spring Data sur d’autres API d’Azure Cosmos DB, consultez les articles suivants :
> * [Spring Data pour Apache Cassandra avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-apache-cassandra-with-cosmos-db)
> * [Spring Data MongoDB avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-mongodb-with-cosmos-db)
> * [Spring Data Gremlin avec Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-data-gremlin-java-app-with-cosmos-db)
>
> Vous souhaitez vous familiariser rapidement ?
> 1. Installez le [runtime Java minimal pris en charge, JDK 8](/java/azure/jdk/?view=azure-java-stable&preserve-view=true), pour pouvoir utiliser le Kit de développement logiciel (SDK).
> 2. Créez une application Spring Data Azure Cosmos DB à l’aide du [starter](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db). C’est facile.
> 3. Utilisez le [guide du développeur Spring Data Azure Cosmos DB](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) qui vous guide pas à pas à travers les requêtes Azure Cosmos DB de base.
>
> Vous pouvez créer rapidement des applications Spring Boot Starter grâce à [Spring Initializr](https://start.spring.io/) !
>

## <a name="resources"></a>Ressources

| Ressource | Lien |
|---|---|
| **Téléchargement du Kit de développement logiciel (SDK)** | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/spring-data-cosmosdb) |
|**Documentation de l’API** | [Documentation de référence Spring Data Azure Cosmos DB]() |
|**Contribution au Kit de développement logiciel (SDK)** | [Référentiel Spring Data Azure Cosmos DB sur GitHub](https://github.com/microsoft/spring-data-cosmosdb) | 
|**Spring Boot Starter**| [Bibliothèque de client Spring Boot Starter Azure Cosmos DB pour Java](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Exemple d’application Spring TODO avec Azure Cosmos DB**| [Expérience Java de bout en bout dans App Service Linux (partie 2)](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2) |
|**Guide du développeur** | [Guide du développeur Spring Data Azure Cosmos DB](/azure/developer/java/spring-framework/how-to-guides-spring-data-cosmosdb) | 
|**Utilisation de Starter** | [Comment utiliser Spring Boot Starter avec l’API SQL Azure Cosmos DB](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db) <br> [Référentiel GitHub pour Azure Cosmos DB Spring Boot Starter](https://github.com/MicrosoftDocs/azure-dev-docs/blob/master/articles/java/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db.md) |
|**Exemple avec Azure App Service** | [Guide pratique pour utiliser Spring et Cosmos DB avec App Service sur Linux](/azure/developer/java/spring-framework/configure-spring-app-with-cosmos-db-on-app-service-linux) <br> [Exemple d’application TODO](https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git) |

## <a name="release-history"></a>Historique des mises en production

### <a name="230-may-21-2020"></a>2.3.0 (21 mai 2020)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour Spring Boot vers la version 2.3.0.


### <a name="225-may-19-2020"></a>2.2.5 (19 mai 2020)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour Azure Cosmos DB vers la version 3.7.3.
#### <a name="key-bug-fixes"></a>Correctifs des bogues clés
* Contient des correctifs de fuites de mémoire et des mises à niveau de version Netty à partir du Kit de développement logiciel (SDK) Azure Cosmos DB v3.7.3.

### <a name="224-april-6-2020"></a>2.2.4 (6 avril 2020)
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige l’indicateur `allowTelemetry` pour le prendre en compte à partir de `CosmosDbConfig`.
* Corrige la propriété `TTL` sur le conteneur.

### <a name="223-february-25-2020"></a>2.2.3 (25 février 2020)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute une nouvelle `findAll` par API de clé de partition.
* Met à jour Azure Cosmos DB vers la version 3.7.0.
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige `collectionName` -> `containerName`.
* Corrige `entityClass` et `domainClass` -> `domainType`.
* Corrige « Retour de la collection d’entités enregistrée par référentiel au lieu d’entités d’entrée ».

### <a name="2110-february-25-2020"></a>2.1.10 (25 février 2020)
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Les backports corrigent « Retour de la collection d’entités enregistrée par référentiel au lieu d’entités d’entrée ».

### <a name="222-january-15-2020"></a>2.2.2 (15 janvier 2020)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour Azure Cosmos DB vers la version 3.6.0.
#### <a name="key-bug-fixes"></a>Principaux bogues résolus

### <a name="221-december-31-2019"></a>2.2.1 (31 décembre 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour le Kit de développement logiciel (SDK) Azure Cosmos DB vers la version 3.5.0.
* Ajoute un champ d’annotation pour activer ou désactiver la création automatique de la collection.
* Améliore la gestion des exceptions. Expose `CosmosClientException` via `CosmosDBAccessException`.
* Expose `requestCharge` et `activityId` via `ResponseDiagnostics`.
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* La mise à jour du Kit de développement logiciel (SDK) 3.5.0 corrige « Exception lorsque l’en-tête de réponse HTTP Cosmos DB dépasse 8 192 octets » et « Échec de ConsistencyPolicy.defaultConsistencyLevel() sur l’obsolescence limitée et le préfixe cohérent ».
* Corrige le comportement de la méthode `findById`. Auparavant, cette méthode retournait une réponse vide si l’entité n’étais pas trouvée au lieu de lever une exception.
* Corrige un bogue dans lequel le tri n’était pas appliqué à la page suivante quand `CosmosPageRequest` était utilisé.

### <a name="219-december-26-2019"></a>2.1.9 (26 décembre 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute un champ d’annotation pour activer ou désactiver la création automatique de la collection.
#### <a name="key-bug-fixes"></a>Correctifs des bogues clés
*  Corrige le comportement de la méthode `findById`. Auparavant, cette méthode retournait une réponse vide si l’entité n’étais pas trouvée au lieu de lever une exception.

### <a name="220-october-21-2019"></a>2.2.0 (21 octobre 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Prise en charge complète du référentiel Cosmos réactif.
* Prise en charge de la chaîne de diagnostics des requêtes et des métriques de requêtes Azure Cosmos DB.
* Met à jour le Kit de développement logiciel (SDK) Azure Cosmos DB vers la version 3.3.1.
* Met à niveau Spring Framework vers la version 5.2.0.RELEASE.
* Met à niveau Spring Data Commons vers la version 2.2.0.RELEASE.
* Ajoute les API `findByIdAndPartitionKey` et `deleteByIdAndPartitionKey`.
* Supprime la dépendance d’azure-documentdb.
* Renomme DocumentDB en Azure Cosmos DB.
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige « Le tri lève une exception quand pageSize est inférieur au nombre total d’éléments dans le référentiel ».

### <a name="218-october-18-2019"></a>2.1.8 (18 octobre 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Déprécie les API DocumentDB.
* Ajoute les API `findByIdAndPartitionKey` et `deleteByIdAndPartitionKey`.
* Ajoute un verrouillage optimiste basé sur `_etag`.
* Active l’expression SpEL pour le nom de la collection de documents.
* Ajoute des améliorations à `ObjectMapper`.

### <a name="217-october-18-2019"></a>2.1.7 (18 octobre 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute la dépendance au Kit de développement logiciel (SDK) Azure Cosmos DB version 3.
* Ajoute un référentiel Cosmos réactif.
* Met à jour l’implémentation de `DocumentDbTemplate` pour utiliser le Kit de développement logiciel (SDK) Azure Cosmos DB version 3.
* Ajoute d’autres modifications de configuration pour la prise en charge des référentiels Cosmos réactifs.

### <a name="212-march-19-2019"></a>2.1.2 (19 mars 2019)
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Supprime la dépendance à `applicationInsights` pour :
    * Risque potentiel des dépendances polluantes.
    * Incompatibilité Java 11.
    * Éviter un impact potentiel sur les performances du processeur et/ou de la mémoire.

### <a name="207-march-20-2019"></a>2.0.7 (20 mars 2019)
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Le backport supprime la dépendance à `applicationInsights` pour :
    * Risque potentiel des dépendances polluantes.
    * Incompatibilité Java 11.
    * Éviter un impact potentiel sur les performances du processeur et/ou de la mémoire.

### <a name="211-march-7-2019"></a>2.1.1 (7 mars 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour la version principale vers 2.1.1.

### <a name="206-march-7-2019"></a>2.0.6 (7 mars 2019)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ignorer toutes les exceptions de la télémétrie.

### <a name="210-december-17-2018"></a>2.1.0 (17 décembre 2018)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Met à jour la version vers 2.1.0 pour résoudre le problème.

### <a name="205-september-13-2018"></a>2.0.5 (13 septembre 2018)
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute les mots clés `exists` et `startsWith`.
* Met à jour le fichier Lisez-moi.
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige « Impossible d’appeler autohref directement pour l’entité ».
* Corrige « findAll échoue si la collection n’est pas créée ».

### <a name="204-prerelease-august-23-2018"></a>2.0.4 (prépublication) [23 août 2018]
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Renomme le package documentdb en cosmosdb.
* Ajoute une nouvelle fonctionnalité de requête avec la méthode des mots clés. Seize mots clés de l’API SQL sont désormais pris en charge.
* Ajoute une nouvelle fonctionnalité de requête avec la pagination et le tri.
* Simplifie la configuration de spring-data-cosmosdb.
* Ajoute les API `deleteCollection` et `deleteAll`.

#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Correction des bogues et atténuation des défauts.

## <a name="faq"></a>Questions fréquentes (FAQ)
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Étapes suivantes
Apprenez-en davantage sur [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Apprenez-en davantage sur [Spring Framework](https://spring.io/projects/spring-framework).

Apprenez-en davantage sur [Spring Boot](https://spring.io/projects/spring-boot).

Apprenez-en davantage sur [Spring Data](https://spring.io/projects/spring-data).