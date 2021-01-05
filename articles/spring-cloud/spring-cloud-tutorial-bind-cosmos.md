---
title: Lier une base de données Azure Cosmos DB à votre application Azure Spring Cloud
description: Découvrez comment lier une base de données Azure Cosmos DB à votre application Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/06/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 20926988d554d562f36587734bd99c34b4731e69
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92088718"
---
# <a name="bind-an-azure-cosmos-db-database-to-your-azure-spring-cloud-application"></a>Lier une base de données Azure Cosmos DB à votre application Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java

Au lieu de configurer manuellement vos applications Spring Boot, vous pouvez lier automatiquement certains services Azure à vos applications à l’aide d’Azure Spring Cloud. Cet article explique comment lier votre application à une base de données Azure Cosmos DB.

Configuration requise :

* Une instance Azure Spring Cloud déployée. Pour bien démarrer, suivez notre [guide de démarrage rapide sur le déploiement d’une application via Azure CLI](spring-cloud-quickstart.md).
* Un compte Azure Cosmos DB avec un niveau d’autorisation minimal de contributeur

## <a name="bind-azure-cosmos-db"></a>Lier Azure Cosmos DB

Azure Cosmos DB comprend cinq types d’API qui prennent en charge les liaisons. La procédure suivante montre comment les utiliser :

1. Crée une base de données Azure Cosmos DB. Pour obtenir de l’aide, reportez-vous au guide de démarrage rapide sur la [création d’une base de données](../cosmos-db/create-cosmosdb-resources-portal.md). 

1. Prenez note du nom de votre base de données. Dans cette procédure, le nom de la base de données est **testdb**.

1. Ajoutez l’une des dépendances suivantes dans le fichier pom.xml de votre application Azure Spring Cloud. Choisissez la dépendance qui convient à votre type d’API.

    * Type d’API : Core (SQL)

      ```xml
      <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-cosmosdb-spring-boot-starter</artifactId>
          <version>2.1.6</version>
      </dependency>
      ```

    * Type d’API : MongoDB

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-mongodb</artifactId>
      </dependency>
      ```

    * Type d’API : Cassandra

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-cassandra</artifactId>
      </dependency>
      ```

    * Type d’API : Gremlin (graphique)

      ```xml
      <dependency>
          <groupId>com.microsoft.spring.data.gremlin</groupId>
          <artifactId>spring-data-gremlin</artifactId>
          <version>2.1.7</version>
      </dependency>
      ```

    * Type d’API : table Azure

      ```xml
      <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-storage-spring-boot-starter</artifactId>
          <version>2.0.5</version>
      </dependency>
      ```

1. Utilisez `az spring-cloud app update` pour mettre à jour le déploiement actuel, ou utilisez `az spring-cloud app deployment create` pour créer un déploiement. Ces commandes mettent à jour ou créent l’application avec la nouvelle dépendance.

1. Accédez à la page du service Azure Spring Cloud dans le portail Azure. Accédez au **Tableau de bord de l’application** et sélectionnez l’application à lier à Azure Cosmos DB. Il s’agit de l’application que vous avez mise à jour ou déployée à l’étape précédente.

1. Sélectionnez **Liaison de service**, puis sélectionnez **Créer une liaison de service**. Pour remplir le formulaire, sélectionnez :
   * Le **type de liaison** **Azure Cosmos DB**
   * Le type d’API
   * Le nom de votre base de données
   * Un compte Azure Cosmos DB

    > [!NOTE]
    > Si vous utilisez Cassandra, utilisez un espace de clés pour le nom de la base de données.

1. Redémarrez l’application en sélectionnant **Redémarrer** dans la page de l’application.

1. Pour vérifier que le service est lié correctement, sélectionnez le nom de la liaison et examinez les détails. Le champ `property` doit être similaire à l’exemple suivant :

    ```
    azure.cosmosdb.uri=https://<some account>.documents.azure.com:443
    azure.cosmosdb.key=abc******
    azure.cosmosdb.database=testdb
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez vu comment lier votre application Azure Spring Cloud à une base de données Azure Cosmos DB. Pour en savoir plus sur la liaison de services à votre application, consultez [Lier à un cache Azure Cache pour Redis](spring-cloud-tutorial-bind-redis.md).