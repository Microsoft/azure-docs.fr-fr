---
title: 'Tutoriel : Générer une application Java pour créer un compte d’API Cassandra Azure Cosmos DB'
description: Ce didacticiel montre comment créer un compte d’API Cassandra, et ajouter une base de données (également appelée « espace de clés ») et une table à ce compte avec une application Java.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 12/06/2018
ms.custom: seodec18, devx-track-java
ms.openlocfilehash: fe452f61d17f0b2014957e3b458ef1ad1b3c539d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357625"
---
# <a name="tutorial-create-a-cassandra-api-account-in-azure-cosmos-db-by-using-a-java-application-to-store-keyvalue-data"></a>Tutoriel : Créer un compte d’API Cassandra dans Azure Cosmos DB avec une application Java pour stocker des données de clé/valeur
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

En tant que développeur, vous pouvez avoir des applications qui utilisent des paires clé/valeur. Vous pouvez utiliser un compte d’API Cassandra dans Azure Cosmos DB pour stocker les données de clé/valeur. Ce tutoriel explique comment utiliser une application Java pour créer un compte d’API Cassandra dans Azure Cosmos DB et ajouter une base de données (également appelée « espace de clés ») et une table. L’application Java utilise le [pilote Java](https://github.com/datastax/java-driver) pour créer une base de données utilisateur qui contient des détails comme l’ID d’utilisateur, le nom d’utilisateur et la localité de l’utilisateur.  

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Créer un compte de base de données Cassandra
> * Obtenir la chaîne de connexion du compte
> * Créer un projet Maven et des dépendances
> * Ajouter une base de données et une table
> * Exécuter l’application

## <a name="prerequisites"></a>Prérequis 

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer. 

* Obtenez la version la plus récente du [JDK (Java Development Kit)](/java/azure/jdk/?view=azure-java-stable&preserve-view=true). 

* [Téléchargez](https://maven.apache.org/download.cgi) et [installez](https://maven.apache.org/install.html) l’archive binaire [Maven](https://maven.apache.org/). 
  - Sur Ubuntu, vous pouvez exécuter `apt-get install maven` pour installer Maven. 

## <a name="create-a-database-account"></a>Création d’un compte de base de données 

1. Connectez-vous au [portail Azure](https://portal.azure.com/). 

2. Sélectionnez **Créer une ressource** > **Bases de données** > **Azure Cosmos DB**. 

3. Dans le volet **Nouveau compte**, entrez les paramètres du nouveau compte Azure Cosmos. 

   |Paramètre   |Valeur suggérée  |Description  |
   |---------|---------|---------|
   |id   |   Entrer un nom unique    | Entrez un nom unique pour identifier ce compte Azure Cosmos. <br/><br/>Comme cassandra.cosmosdb.azure.com est ajouté à l’ID que vous fournissez pour créer votre point de contact, utilisez un ID unique, mais identifiable.         |
   |API    |  Cassandra   |  L’API détermine le type de compte à créer. <br/> Sélectionnez **Cassandra**, car dans cet article, vous allez créer une base de données avec des colonnes larges, qui peut être interrogée avec la syntaxe de langage de requête Cassandra.  |
   |Abonnement    |  Votre abonnement        |  Sélectionnez l’abonnement Azure que vous voulez utiliser pour ce compte Azure Cosmos.        |
   |Groupe de ressources   | Entrez un nom    |  Sélectionnez **Créer nouveau**, puis entrez le nom du nouveau groupe de ressources pour votre compte. Pour plus de simplicité, vous pouvez utiliser le même nom que votre ID.    |
   |Emplacement    |  Sélectionner la région la plus proche de vos utilisateurs    |  Sélectionnez l’emplacement géographique dans lequel héberger votre compte Azure Cosmos. Utilisez l’emplacement le plus proche de vos utilisateurs pour leur donner l’accès le plus rapide possible aux données.    |

   :::image type="content" source="./media/create-cassandra-api-account-java/create-account.png" alt-text="Créer un compte avec le portail":::

4. Sélectionnez **Create** (Créer). <br/>La création du compte prend quelques minutes. Une fois que la ressource est créée, vous pouvez voir la notification **Déploiement réussi** dans la partie de droite du portail.

## <a name="get-the-connection-details-of-your-account"></a>Obtenir les détails de connexion de votre compte  

Obtenez les informations de la chaîne de connexion auprès du Portail Azure, puis copiez-les dans le fichier de configuration Java. La chaîne de connexion permet à votre application de communiquer avec votre base de données hébergée. 

1. Dans le [portail Azure](https://portal.azure.com/), accédez à votre compte Azure Cosmos. 

2. Ouvrez le volet **Chaîne de connexion**.  

3. Copiez les valeurs de **POINT DE CONTACT**, **PORT**, **NOM D’UTILISATEUR** et **MOT DE PASSE PRINCIPAL**, que vous utiliserez dans les étapes suivantes.

## <a name="create-the-project-and-the-dependencies"></a>Créer le projet et les dépendances 

L’exemple de projet Java que vous utilisez dans cet article est hébergé dans GitHub. Vous pouvez exécuter les étapes de ce document ou télécharger l’exemple à partir du référentiel [azure-cosmos-db-cassandra-java-getting-started](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started). 

Après avoir téléchargé les fichiers, mettez à jour les informations de la chaîne de connexion dans le fichier `java-examples\src\main\resources\config.properties` et exécutez-le.  

```java
cassandra_host=<FILLME_with_CONTACT POINT> 
cassandra_port = 10350 
cassandra_username=<FILLME_with_USERNAME> 
cassandra_password=<FILLME_with_PRIMARY PASSWORD> 
```

Suivez la procédure ci-après pour créer l’exemple depuis le début : 

1. À partir du terminal ou d’une invite de commandes, créez un projet Maven nommé Cassandra-demo. 

   ```bash
   mvn archetype:generate -DgroupId=com.azure.cosmosdb.cassandra -DartifactId=cassandra-demo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false 
   ```
 
2. Recherchez le dossier `cassandra-demo`. Avec un éditeur de texte, ouvrez le fichier `pom.xml` qui a été généré. 

   Ajoutez les dépendances de Cassandra et générez les plug-ins nécessaires pour votre projet, comme indiqué dans le fichier [pom.xml](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/pom.xml).  

3. Sous le dossier `cassandra-demo\src\main`, créez un dossier nommé `resources`.  Sous le dossier des ressources, ajoutez les fichiers config.properties et log4j.properties :

   - Le fichier [config.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/resources/config.properties) stocke les valeurs du point de terminaison et de la clé de connexion du compte de l’API Cassandra. 
   
   - Le fichier [log4j.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/resources/log4j.properties) définit le niveau de journalisation nécessaire pour l’interaction avec l’API Cassandra.  

4. Accédez au dossier `src/main/java/com/azure/cosmosdb/cassandra/`. Dans le dossier cassandra, créez un autre dossier nommé `utils`. Le nouveau dossier stocke les classes utilitaires nécessaires pour se connecter au compte de l’API Cassandra. 

   Ajoutez la classe [CassandraUtils](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java) pour créer le cluster, et pour ouvrir et fermer les sessions Cassandra. Le cluster se connecte au compte de l’API Cassandra dans Azure Cosmos DB et retourne une session permettant l’accès. Utilisez la classe [Configurations](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/util/Configurations.java) pour lire les informations de la chaîne de connexion dans le fichier config.properties. 

5. L’exemple Java crée une base de données avec les informations utilisateur, comme le nom d’utilisateur, l’ID d’utilisateur et la localité de l’utilisateur. Vous devez définir des méthodes get et set pour accéder aux détails de l’utilisateur dans la fonction main.
 
   Créez une classe [User.java](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/main/src/main/java/com/azure/cosmosdb/cassandra/examples/UserProfile.java) sous le dossier `src/main/java/com/azure/cosmosdb/cassandra/` avec des méthodes get et set. 

## <a name="add-a-database-and-a-table"></a>Ajouter une base de données et une table  

Cette section décrit comment ajouter une base de données (espace de clés) et une table avec CQL.

1. Sous le dossier `src\main\java\com\azure\cosmosdb\cassandra`, créez un dossier nommé `repository`. 

2. Créez la classe Java `UserRepository` et ajoutez-y le code suivant : 

   ```java
   package com.azure.cosmosdb.cassandra.repository; 
   import java.util.List; 
   import com.datastax.driver.core.BoundStatement; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Row; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Create a Cassandra session 
    */ 
   public class UserRepository { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserRepository.class); 
       private Session session; 
       public UserRepository(Session session) { 
           this.session = session; 
       } 
   
       /** 
       * Create keyspace uprofile in cassandra DB 
        */ 
   
       public void createKeyspace() { 
            final String query = "CREATE KEYSPACE IF NOT EXISTS uprofile WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"; 
           session.execute(query); 
           LOGGER.info("Created keyspace 'uprofile'"); 
       } 
   
       /** 
        * Create user table in cassandra DB 
        */ 
   
       public void createTable() { 
           final String query = "CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)"; 
           session.execute(query); 
           LOGGER.info("Created table 'user'"); 
       } 
   } 
   ```

3. Recherchez le dossier `src\main\java\com\azure\cosmosdb\cassandra` et créez un sous-dossier nommé `examples`.

4. Créez la classe Java `UserProfile`. Cette classe contient la méthode principale qui appelle les méthodes createKeyspace et createTable que vous avez définies précédemment : 

   ```java
   package com.azure.cosmosdb.cassandra.examples; 
   import java.io.IOException; 
   import com.azure.cosmosdb.cassandra.repository.UserRepository; 
   import com.azure.cosmosdb.cassandra.util.CassandraUtils; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Example class which will demonstrate following operations on Cassandra Database on CosmosDB 
    * - Create Keyspace 
    * - Create Table 
    * - Insert Rows 
    * - Select all data from a table 
    * - Select a row from a table 
    */ 
   
   public class UserProfile { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserProfile.class); 
       public static void main(String[] s) throws Exception { 
           CassandraUtils utils = new CassandraUtils(); 
           Session cassandraSession = utils.getSession(); 
   
           try { 
               UserRepository repository = new UserRepository(cassandraSession); 
               //Create keyspace in cassandra database 
               repository.createKeyspace(); 
               //Create table in cassandra database 
               repository.createTable(); 
   
           } finally { 
               utils.close(); 
               LOGGER.info("Please delete your table after verifying the presence of the data in portal or from CQL"); 
           } 
       } 
   } 
   ```
 
## <a name="run-the-app"></a>Exécuter l’application 

1. Ouvrez une invite de commandes ou une fenêtre de terminal. Collez le bloc de code suivant. 

   Ce code change le répertoire (cd) pour le chemin du dossier où vous avez créé le projet. Ensuite, il exécute la commande `mvn clean install` pour générer le fichier `cosmosdb-cassandra-examples.jar` dans le dossier cible. Enfin, il exécute l’application Java.

   ```bash
   cd cassandra-demo

   mvn clean install 

   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile 
   ```

   La fenêtre de terminal affiche des notifications indiquant que l’espace de clé et la table sont créés. 
   
2. Maintenant, dans le portail Azure, ouvrez **l’Explorateur de données** pour vérifier que l’espace de clés et la table ont été créés.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment créer un compte d’API Cassandra dans Azure Cosmos DB, ainsi qu’une base de données et une table à l’aide d’une application Java. Vous pouvez maintenant passer à l’article suivant :

> [!div class="nextstepaction"]
> [Charger des exemples de données dans la table de l’API Cassandra](cassandra-api-load-data.md).
