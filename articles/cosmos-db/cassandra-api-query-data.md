---
title: 'Tutoriel : Interroger les données à partir d’un compte d’API Cassandra dans Azure Cosmos DB'
description: Ce tutoriel montre comment interroger les données utilisateur d’un compte d’API Cassandra Azure Cosmos DB à l’aide d’une application Java.
ms.service: cosmos-db
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: 693984ceaef1a71ebfbe4122c18a45a563c9c101
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100427"
---
# <a name="tutorial-query-data-from-a-cassandra-api-account-in-azure-cosmos-db"></a>Tutoriel : Interroger les données à partir d’un compte d’API Cassandra dans Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

En tant que développeur, vous pouvez avoir des applications qui utilisent des paires clé/valeur. Vous pouvez utiliser un compte d’API Cassandra dans Azure Cosmos DB pour stocker et interroger des données de clé/valeur. Ce tutoriel montre comment interroger les données utilisateur d’un compte d’API Cassandra dans Azure Cosmos DB à l’aide d’une application Java. L’application Java utilise le [pilote Java](https://github.com/datastax/java-driver) et interroge des données utilisateur, comme l’ID de l’utilisateur, son nom et sa localité. 

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Interroger les données d’une table Cassandra
> * Exécuter l’application

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Cet article appartient à un tutoriel en plusieurs parties. Avant de commencer, veillez à effectuer les étapes précédentes pour créer un compte, un espace de clés et une table d’API Cassandra, puis pour [charger des exemples de données dans la table](cassandra-api-load-data.md). 

## <a name="query-data"></a>Interroger des données

Utilisez les étapes suivantes pour interroger des données de votre table d’API Cassandra :

1. Ouvrez le fichier `UserRepository.java` qui se trouve dans le dossier `src\main\java\com\azure\cosmosdb\cassandra`. Ajoutez-y le bloc de code suivant. Ce code fournit trois méthodes : 

   * Pour interroger tous les utilisateurs de la base de données
   * Pour interroger un utilisateur spécifique, filtré par ID d’utilisateur
   * Pour supprimer une table

   ```java
   /**
   * Select all rows from user table
   */
   public void selectAllUsers() {

     final String query = "SELECT * FROM uprofile.user";
     List<Row> rows = session.execute(query).all();

     for (Row row : rows) {
        LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
     }
   }

   /**
   * Select a row from user table
   *
   * @param id user_id
   */
   public void selectUser(int id) {
      final String query = "SELECT * FROM uprofile.user where user_id = 3";
      Row row = session.execute(query).one();

      LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
   }

   /**
   * Delete user table.
   */
   public void deleteTable() {
     final String query = "DROP TABLE IF EXISTS uprofile.user";
     session.execute(query);
   }
   ```

2. Ouvrez le fichier `UserProfile.java` qui se trouve dans le dossier `src\main\java\com\azure\cosmosdb\cassandra`. Cette classe contient la méthode principale qui appelle les méthodes d’insertion des données createKeyspace et createTable que vous avez définies précédemment. Ajoutez maintenant le code suivant pour interroger tous les utilisateurs ou un utilisateur spécifique :

   ```java
   LOGGER.info("Select all users");
   repository.selectAllUsers();

   LOGGER.info("Select a user by id (3)");
   repository.selectUser(3);

   LOGGER.info("Delete the users profile table");
   repository.deleteTable();
   ```

## <a name="run-the-java-app"></a>Exécuter l’application Java
1. Ouvrez une invite de commandes ou une fenêtre de terminal. Collez le bloc de code suivant. 

   Ce code change le répertoire (cd) pour le chemin du dossier où vous avez créé le projet. Ensuite, il exécute la commande `mvn clean install` pour générer le fichier `cosmosdb-cassandra-examples.jar` dans le dossier cible. Enfin, il exécute l’application Java.

   ```bash
   cd "cassandra-demo"
   
   mvn clean install
   
   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
   ```

2. Maintenant, dans le Portail Azure, ouvrez l’ **Explorateur de données** et confirmez la suppression de la table utilisateur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources, le compte Azure Cosmos et toutes les ressources associées. Pour cela, sélectionnez le groupe de ressources de la machine virtuelle, sélectionnez **Supprimer** , puis confirmez le nom du groupe de ressources à supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à interroger les données d’un compte d’API Cassandra dans Azure Cosmos DB. Vous pouvez maintenant passer à l’article suivant :

> [!div class="nextstepaction"]
> [Migrer des données vers un compte d’API Cassandra](cassandra-import-data.md)


