---
title: 'Tutoriel : Application Java pour charger des exemples de données dans une table d’API Cassandra dans Azure Cosmos DB'
description: Ce tutoriel montre comment charger des exemples de données utilisateur dans une table d’API Cassandra dans Azure Cosmos DB avec une application Java.
author: kanshiG
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: ad3dfc8c86d22e58377ebfbdb94b2beb004ded56
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092464"
---
# <a name="tutorial-load-sample-data-into-a-cassandra-api-table-in-azure-cosmos-db"></a>Tutoriel : Charger des exemples de données dans une table d’API Cassandra dans Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

En tant que développeur, vous pouvez avoir des applications qui utilisent des paires clé/valeur. Vous pouvez utiliser un compte d’API Cassandra dans Azure Cosmos DB pour stocker et gérer des données de clé/valeur. Ce tutoriel montre comment charger des exemples de données utilisateur dans une table du compte d’API Cassandra dans Azure Cosmos DB avec une application Java. L’application Java utilise le [pilote Java](https://github.com/datastax/java-driver) et charge des données utilisateur, comme l’ID de l’utilisateur, son nom et sa localité. 

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Charger des données dans une table Cassandra
> * Exécuter l’application

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Cet article appartient à un tutoriel en plusieurs parties. Avant de commencer avec ce document, veillez à [créer le compte, l’espace de clés et la table de l’API Cassandra](create-cassandra-api-account-java.md).   

## <a name="load-data-into-the-table"></a>Charger des données dans la table

Utilisez les étapes suivantes pour charger des données dans votre table de l’API Cassandra :

1. Ouvrez le fichier « UserRepository.java » sous le dossier « src\main\java\com\azure\cosmosdb\cassandra » et ajoutez le code pour insérer les champs user_id, user_name et user_bcity dans la table :

   ```java
   /**
   * Insert a row into user table
   *
   * @param id   user_id
   * @param name user_name
   * @param city user_bcity
   */
   public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
   }

   /**
   * Create a PrepareStatement to insert a row to user table
   *
   * @return PreparedStatement
   */
   public PreparedStatement prepareInsertStatement() {
      final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
   return session.prepare(insertStatement);
   }
   ```
 
2. Ouvrez le fichier « UserProfile.java » sous le dossier « src\main\java\com\azure\cosmosdb\cassandra ». Cette classe contient la méthode principale qui appelle les méthodes createKeyspace et createTable que vous avez définies précédemment. Ajoutez maintenant le code suivant pour insérer des exemples de données dans la table de l’API Cassandra.

   ```java
   //Insert rows into user table
   PreparedStatement preparedStatement = repository.prepareInsertStatement();
     repository.insertUser(preparedStatement, 1, "JohnH", "Seattle");
     repository.insertUser(preparedStatement, 2, "EricK", "Spokane");
     repository.insertUser(preparedStatement, 3, "MatthewP", "Tacoma");
     repository.insertUser(preparedStatement, 4, "DavidA", "Renton");
     repository.insertUser(preparedStatement, 5, "PeterS", "Everett");
   ```

## <a name="run-the-app"></a>Exécuter l’application

Ouvrez une invite de commandes ou une fenêtre de terminal, puis changez le chemin du dossier pour celui dans lequel vous avez créé le projet. Exécutez la commande « mvn clean install » pour générer le fichier cosmosdb-cassandra-examples.jar dans le dossier cible et exécuter l’application. 

```bash
cd "cassandra-demo"

mvn clean install

java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
```

Vous pouvez maintenant ouvrir l’Explorateur de données dans le portail Azure pour vérifier que les informations utilisateur sont ajoutées à la table.
    
## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment charger des exemples de données dans un compte d’API Cassandra dans Azure Cosmos DB. Vous pouvez maintenant passer à l’article suivant :

> [!div class="nextstepaction"]
> [Interroger des données à partir du compte de l’API Cassandra](cassandra-api-query-data.md)
