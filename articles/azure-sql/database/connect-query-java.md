---
title: Utiliser Java et JDBC avec Azure SQL Database
description: Découvrez comment utiliser Java et JDBC avec une base de données Azure SQL Database.
services: sql-database
author: jdubois
ms.author: judubois
ms.service: sql-database
ms.subservice: development
ms.topic: quickstart
ms.devlang: java
ms.date: 06/26/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: badf6b8887c356c2a7fc7308f6aa15f551e4bb67
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95972672"
---
# <a name="use-java-and-jdbc-with--azure-sql-database"></a>Utiliser Java et JDBC avec Azure SQL Database

Cette rubrique illustre la création d’un exemple d’application qui utilise Java et [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity) pour stocker et récupérer des informations dans [Azure SQL Database](/azure/sql-database/).

JDBC est l’API Java standard pour se connecter à des bases de données relationnelles classiques.

## <a name="prerequisites"></a>Prérequis

- Un compte Azure. Si vous n’en avez pas, inscrivez-vous pour un [essai gratuit](https://azure.microsoft.com/free/).
- [Azure Cloud Shell](../../cloud-shell/quickstart.md) ou [Azure CLI](/cli/azure/install-azure-cli). Nous vous recommandons d’utiliser Azure Cloud Shell. Vous serez ainsi connecté automatiquement et vous aurez accès à tous les outils dont vous avez besoin.
- Un [Java Development Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) pris en charge, version 8 (inclus dans Azure Cloud Shell).
- L’outil de build [Apache Maven](https://maven.apache.org/).

## <a name="prepare-the-working-environment"></a>Préparer l’environnement de travail

Nous allons utiliser des variables d’environnement pour limiter les erreurs de saisie et pour faciliter la personnalisation de la configuration suivante en fonction de vos besoins spécifiques.

Configurez ces variables d’environnement à l’aide des commandes suivantes :

```bash
AZ_RESOURCE_GROUP=database-workshop
AZ_DATABASE_NAME=<YOUR_DATABASE_NAME>
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_SQL_SERVER_USERNAME=demo
AZ_SQL_SERVER_PASSWORD=<YOUR_AZURE_SQL_PASSWORD>
AZ_LOCAL_IP_ADDRESS=<YOUR_LOCAL_IP_ADDRESS>
```

Remplacez les espaces réservés par les valeurs suivantes, qui sont utilisées dans cet article :

- `<YOUR_DATABASE_NAME>`: Nom de votre serveur Azure SQL Database. Il doit être unique dans tout Azure.
- `<YOUR_AZURE_REGION>`: région Azure que vous allez utiliser. Vous pouvez utiliser `eastus` par défaut, mais nous vous recommandons de configurer une région plus proche de l’endroit où vous vivez. Vous pouvez obtenir la liste complète des régions disponibles en entrant `az account list-locations`.
- `<AZ_SQL_SERVER_PASSWORD>`: Mot de passe de votre serveur Azure SQL Database. Ce mot de passe doit comporter au minimum huit caractères. Il doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres (0-9) et caractères non alphanumériques (comme !, $, #, %, etc.).
- `<YOUR_LOCAL_IP_ADDRESS>`: adresse IP de votre ordinateur local, à partir duquel vous exécuterez l’application Java. Pour ce faire, vous pouvez faire pointer votre navigateur vers [whatismyip.akamai.com](http://whatismyip.akamai.com/).

Ensuite, créez un groupe de ressources à l’aide de la commande suivante :

```azurecli
az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
  	| jq
```

> [!NOTE]
> Nous utilisons l’utilitaire `jq` pour afficher les données JSON et les rendre plus lisibles. Cet utilitaire est installé par défaut sur [Azure Cloud Shell](https://shell.azure.com/). Si vous ne souhaitez pas utiliser cet utilitaire, vous pouvez supprimer la partie `| jq` de toutes les commandes que nous allons utiliser sans risque.

## <a name="create-an-azure-sql-database-instance"></a>Créer une instance Azure SQL Database

La première chose que nous allons créer est un serveur Azure SQL Database managé.

> [!NOTE]
> Vous pouvez consulter des informations plus détaillées sur la création de serveurs Azure SQL Database dans [Démarrage rapide : Créer une base de données unique Azure SQL Database](./single-database-create-quickstart.md).

Dans [Azure Cloud Shell](https://shell.azure.com/), exécutez la commande suivante :

```azurecli
az sql server create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME \
    --location $AZ_LOCATION \
    --admin-user $AZ_SQL_SERVER_USERNAME \
    --admin-password $AZ_SQL_SERVER_PASSWORD \
  	| jq
```

Cette commande crée un serveur Azure SQL Database.

### <a name="configure-a-firewall-rule-for-your-azure-sql-database-server"></a>Configurer une règle de pare-feu pour votre serveur Azure SQL Database

Les instances Azure SQL Database sont sécurisées par défaut. Elles ont un pare-feu qui n’autorise aucune connexion entrante. Pour pouvoir utiliser notre base de données, vous devez ajouter une règle de pare-feu qui permet à l’adresse IP locale d’accéder au serveur de base de données.

Comme vous avez configuré votre adresse IP locale au début de cet article, vous pouvez ouvrir le pare-feu du serveur en exécutant la commande suivante :

```azurecli
az sql server firewall-rule create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME-database-allow-local-ip \
    --server $AZ_DATABASE_NAME \
    --start-ip-address $AZ_LOCAL_IP_ADDRESS \
    --end-ip-address $AZ_LOCAL_IP_ADDRESS \
  	| jq
```

### <a name="configure-a-azure-sql-database"></a>Configurer une base de donnéesSQL Azure

Le serveur Azure SQL Database que vous avez créé précédemment est vide. Il ne contient aucune base de données que vous pouvez utiliser avec l’application Java. Créez une base de données nommée `demo` en exécutant la commande suivante :

```azurecli
az sql db create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name demo \
    --server $AZ_DATABASE_NAME \
  	| jq
```

### <a name="create-a-new-java-project"></a>Créer un projet Java

À l’aide de votre IDE favori, créez un projet Java, puis ajoutez un fichier `pom.xml` dans son répertoire racine :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.microsoft.sqlserver</groupId>
            <artifactId>mssql-jdbc</artifactId>
            <version>7.4.1.jre8</version>
        </dependency>
    </dependencies>
</project>
```

Il s’agit d’un fichier [Apache Maven](https://maven.apache.org/) qui configure le projet pour qu’il utilise :

- Java 8
- Pilote SQL Server récent pour Java

### <a name="prepare-a-configuration-file-to-connect-to-azure-sql-database"></a>Préparer un fichier de configuration pour se connecter à Azure SQL Database

Créez un fichier *src/main/resources/application.properties* et ajoutez :

```properties
url=jdbc:sqlserver://$AZ_DATABASE_NAME.database.windows.net:1433;database=demo;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
user=demo@$AZ_DATABASE_NAME
password=$AZ_SQL_SERVER_PASSWORD
```

- Remplacez les deux variables `$AZ_DATABASE_NAME` par la valeur que vous avez configurée au début de cet article.
- Remplacez la variable `$AZ_SQL_SERVER_PASSWORD` par la valeur que vous avez configurée au début de cet article.

### <a name="create-an-sql-file-to-generate-the-database-schema"></a>Créer un fichier SQL pour générer le schéma de base de données

Nous allons utiliser un fichier *src/main/resources/`schema.sql`* afin de créer un schéma de base de données. Créez ce fichier avec le contenu suivant :

```sql
DROP TABLE IF EXISTS todo;
CREATE TABLE todo (id INT PRIMARY KEY, description VARCHAR(255), details VARCHAR(4096), done BIT);
```

## <a name="code-the-application"></a>Coder l’application

### <a name="connect-to-the-database"></a>Se connecter à la base de données

Ensuite, ajoutez le code Java qui utilisera JDBC pour stocker et récupérer des données à partir de votre base de données Azure SQL Database.

Créez un fichier *src/main/java/DemoApplication.java*, qui contient :

```java
package com.example.demo;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class DemoApplication {

    private static final Logger log;

    static {
        System.setProperty("java.util.logging.SimpleFormatter.format", "[%4$-7s] %5$s %n");
        log =Logger.getLogger(DemoApplication.class.getName());
    }

    public static void main(String[] args) throws Exception {
        log.info("Loading application properties");
        Properties properties = new Properties();
        properties.load(DemoApplication.class.getClassLoader().getResourceAsStream("application.properties"));

        log.info("Connecting to the database");
        Connection connection = DriverManager.getConnection(properties.getProperty("url"), properties);
        log.info("Database connection test: " + connection.getCatalog());

        log.info("Create database schema");
        Scanner scanner = new Scanner(DemoApplication.class.getClassLoader().getResourceAsStream("schema.sql"));
        Statement statement = connection.createStatement();
        while (scanner.hasNextLine()) {
            statement.execute(scanner.nextLine());
        }

        /*
        Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
        insertData(todo, connection);
        todo = readData(connection);
        todo.setDetails("congratulations, you have updated data!");
        updateData(todo, connection);
        deleteData(todo, connection);
        */

        log.info("Closing database connection");
        connection.close();
    }
}
```

Ce code Java utilisera les fichiers *application.properties* et *schema.sql* que nous avons créés précédemment pour établir la connexion à la base de données SQL Server et créer un schéma qui stockera nos données.

Dans ce fichier, vous pouvez voir que nous avons commenté des méthodes pour insérer, lire, mettre à jour et supprimer des données : nous allons coder ces méthodes dans le reste de cet article, et vous pourrez en supprimer les marques de commentaire les unes après les autres.

> [!NOTE]
> Les informations d’identification de base de données sont stockées dans les propriétés *user* et *password* du fichier *application.properties*. Ces informations d’identification sont utilisées lors de l’exécution de `DriverManager.getConnection(properties.getProperty("url"), properties);`, car le fichier de propriétés est passé comme argument.

Vous pouvez maintenant exécuter cette classe main avec votre outil favori :

- Avec votre IDE, vous devez être en mesure de cliquer avec le bouton droit sur la classe *DemoApplication* et de l’exécuter.
- À l’aide de Maven, vous pouvez utiliser l’application en exécutant : `mvn exec:java -Dexec.mainClass="com.example.demo.DemoApplication"`.

L’application doit se connecter à Azure SQL Database, créer un schéma de base de données, puis fermer la connexion, comme vous le verrez dans les journaux de la console :

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Closing database connection 
```

### <a name="create-a-domain-class"></a>Créer une classe de domaine

Créez une classe Java `Todo` parallèlement à la classe `DemoApplication`, puis ajoutez le code suivant :

```java
package com.example.demo;

public class Todo {

    private Long id;
    private String description;
    private String details;
    private boolean done;

    public Todo() {
    }

    public Todo(Long id, String description, String details, boolean done) {
        this.id = id;
        this.description = description;
        this.details = details;
        this.done = done;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDetails() {
        return details;
    }

    public void setDetails(String details) {
        this.details = details;
    }

    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }

    @Override
    public String toString() {
        return "Todo{" +
                "id=" + id +
                ", description='" + description + '\'' +
                ", details='" + details + '\'' +
                ", done=" + done +
                '}';
    }
}
```

Cette classe est un modèle de domaine mappé sur la table `todo` que vous avez créée lors de l’exécution du script *schema.sql*.

### <a name="insert-data-into-azure-sql-database"></a>Insérer des données dans Azure SQL Database

Dans le fichier *src/main/java/DemoApplication.java*, après la méthode main, ajoutez la méthode suivante pour insérer des données dans la base de données :

```java
private static void insertData(Todo todo, Connection connection) throws SQLException {
    log.info("Insert data");
    PreparedStatement insertStatement = connection
            .prepareStatement("INSERT INTO todo (id, description, details, done) VALUES (?, ?, ?, ?);");

    insertStatement.setLong(1, todo.getId());
    insertStatement.setString(2, todo.getDescription());
    insertStatement.setString(3, todo.getDetails());
    insertStatement.setBoolean(4, todo.isDone());
    insertStatement.executeUpdate();
}
```

Vous pouvez maintenant supprimer les marques de commentaire des deux lignes suivantes dans la méthode `main` :

```java
Todo todo = new Todo(1L, "configuration", "congratulations, you have set up JDBC correctly!", true);
insertData(todo, connection);
```

L’exécution de la classe main doit maintenant produire la sortie suivante :

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Closing database connection
```

### <a name="reading-data-from-azure-sql-database"></a>Lecture de données à partir d’Azure SQL Database

Nous allons lire les données précédemment insérées pour valider le bon fonctionnement de notre code.

Dans le fichier *src/main/java/DemoApplication.java*, après la méthode `insertData`, ajoutez la méthode suivante pour lire des données de la base de données :

```java
private static Todo readData(Connection connection) throws SQLException {
    log.info("Read data");
    PreparedStatement readStatement = connection.prepareStatement("SELECT * FROM todo;");
    ResultSet resultSet = readStatement.executeQuery();
    if (!resultSet.next()) {
        log.info("There is no data in the database!");
        return null;
    }
    Todo todo = new Todo();
    todo.setId(resultSet.getLong("id"));
    todo.setDescription(resultSet.getString("description"));
    todo.setDetails(resultSet.getString("details"));
    todo.setDone(resultSet.getBoolean("done"));
    log.info("Data read from the database: " + todo.toString());
    return todo;
}
```

Vous pouvez maintenant supprimer les marques de commentaire de la ligne suivante dans la méthode `main` :

```java
todo = readData(connection);
```

L’exécution de la classe main doit maintenant produire la sortie suivante :

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="updating-data-in-azure-sql-database"></a>Mise à jour de données dans Azure SQL Database

Nous allons mettre à jour les données que nous avons insérées précédemment.

Toujours dans le fichier *src/main/java/DemoApplication.java*, après la méthode `readData`, ajoutez la méthode suivante pour mettre à jour des données dans la base de données :

```java
private static void updateData(Todo todo, Connection connection) throws SQLException {
    log.info("Update data");
    PreparedStatement updateStatement = connection
            .prepareStatement("UPDATE todo SET description = ?, details = ?, done = ? WHERE id = ?;");

    updateStatement.setString(1, todo.getDescription());
    updateStatement.setString(2, todo.getDetails());
    updateStatement.setBoolean(3, todo.isDone());
    updateStatement.setLong(4, todo.getId());
    updateStatement.executeUpdate();
    readData(connection);
}
```

Vous pouvez maintenant supprimer les marques de commentaire des deux lignes suivantes dans la méthode `main` :

```java
todo.setDetails("congratulations, you have updated data!");
updateData(todo, connection);
```

L’exécution de la classe main doit maintenant produire la sortie suivante :

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Closing database connection 
```

### <a name="deleting-data-in-azure-sql-database"></a>Suppression de données dans Azure SQL Database

Enfin, nous allons supprimer les données que nous avons insérées précédemment.

Toujours dans le fichier *src/main/java/DemoApplication.java*, après la méthode `updateData`, ajoutez la méthode suivante pour supprimer des données dans la base de données :

```java
private static void deleteData(Todo todo, Connection connection) throws SQLException {
    log.info("Delete data");
    PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM todo WHERE id = ?;");
    deleteStatement.setLong(1, todo.getId());
    deleteStatement.executeUpdate();
    readData(connection);
}
```

Vous pouvez maintenant supprimer les marques de commentaire de la ligne suivante dans la méthode `main` :

```java
deleteData(todo, connection);
```

L’exécution de la classe main doit maintenant produire la sortie suivante :

```
[INFO   ] Loading application properties 
[INFO   ] Connecting to the database 
[INFO   ] Database connection test: demo 
[INFO   ] Create database schema 
[INFO   ] Insert data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have set up JDBC correctly!', done=true} 
[INFO   ] Update data 
[INFO   ] Read data 
[INFO   ] Data read from the database: Todo{id=1, description='configuration', details='congratulations, you have updated data!', done=true} 
[INFO   ] Delete data 
[INFO   ] Read data 
[INFO   ] There is no data in the database! 
[INFO   ] Closing database connection 
```

## <a name="conclusion-and-resources-clean-up"></a>Conclusion et nettoyage des ressources

Félicitations ! Vous avez créé une application Java qui utilise JDBC pour stocker et récupérer des données dans Azure SQL Database.

Pour nettoyer toutes les ressources utilisées dans le cadre de ce guide de démarrage rapide, supprimez le groupe de ressources à l’aide de la commande suivante :

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Étapes suivantes

- [Concevoir votre première base de données dans Azure SQL Database](design-first-database-tutorial.md)  
- [Pilote JDBC Microsoft pour SQL Server](https://github.com/microsoft/mssql-jdbc)  
- [Report issues/ask questions](https://github.com/microsoft/mssql-jdbc/issues) (Signaler des problèmes/poser des questions)
