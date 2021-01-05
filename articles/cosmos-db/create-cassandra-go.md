---
title: Générer une application Go avec l’API Cassandra Azure Cosmos DB à l’aide du client gocql
description: Ce guide de démarrage rapide montre comment utiliser un client Go pour interagir avec l’API Cassandra Azure Cosmos DB
ms.service: cosmos-db
author: abhirockzz
ms.author: abhishgu
ms.subservice: cosmosdb-cassandra
ms.devlang: go
ms.topic: quickstart
ms.date: 07/14/2020
ms.openlocfilehash: 595ec1aaa4aedc3916d1b4d46986dcabae887aaf
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93076399"
---
# <a name="quickstart-build-a-go-app-with-the-gocql-client-to-manage-azure-cosmos-db-cassandra-api-data"></a>Démarrage rapide : Générer une application Go avec le client `gocql` pour gérer les données de l’API Cassandra Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.JS](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
> * [Golang](create-cassandra-go.md)
>

Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale. Dans ce guide de démarrage rapide, vous allez commencer par créer un compte pour l’API Cassandra Azure Cosmos DB. Vous allez ensuite exécuter une application Go pour créer un espace de clés et une table Cassandra ainsi que pour exécuter quelques opérations. Cette application Go utilise [gocql](https://github.com/gocql/gocql), qui est un client Cassandra pour le langage Go. 

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=cassandrago-docs-abhishgu). Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/?WT.mc_id=cassandrago-docs-abhishgu) sans abonnement Azure.
- [Go](https://golang.org/) installé sur votre ordinateur et une expérience de travail avec Go.
- [Git](https://git-scm.com/downloads).

## <a name="create-a-database-account"></a>Création d’un compte de base de données

Pour pouvoir créer une base de données, vous devez créer un compte Cassandra avec Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Commencez par cloner l’application à partir de GitHub.

1. Ouvrez une invite de commandes, puis créez un dossier nommé `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git, comme git bash. Utilisez la commande `cd` pour accéder au nouveau dossier et installer l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-go-getting-started.git
    ```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Si vous souhaitez savoir comment le code crée les ressources de base de données, vous pouvez consulter les extraits de code suivants. Sinon, vous pouvez passer à la section [Exécuter l’application](#run-the-application)

La fonction `GetSession` (qui fait partie de `utils\utils.go`) retourne un [`*gocql.Session`](https://godoc.org/github.com/gocql/gocql#Session) qui permet d’exécuter des opérations de cluster telles que l’insertion, la recherche, etc.

```go
func GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword string) *gocql.Session {
    clusterConfig := gocql.NewCluster(cosmosCassandraContactPoint)
    port, err := strconv.Atoi(cosmosCassandraPort)
    
    clusterConfig.Authenticator = gocql.PasswordAuthenticator{Username: cosmosCassandraUser, Password: cosmosCassandraPassword}
    clusterConfig.Port = port
    clusterConfig.SslOpts = &gocql.SslOptions{Config: &tls.Config{MinVersion: tls.VersionTLS12}}
    clusterConfig.ProtoVersion = 4
    
    session, err := clusterConfig.CreateSession()
    ...
    return session
}
```

L’hôte Cassandra pour Azure Cosmos DB est passé à la fonction [`gocql.NewCluster`](https://godoc.org/github.com/gocql/gocql#NewCluster) afin d’obtenir un struct [`*gocql.ClusterConfig`](https://godoc.org/github.com/gocql/gocql#ClusterConfig) qui est ensuite configuré pour utiliser le nom d’utilisateur, le mot de passe, le port et la version TLS appropriée ([Exigence de sécurité de chiffrement HTTPS/SSL/TLS](./database-security.md?WT.mc_id=cassandrago-docs-abhishgu#how-does-azure-cosmos-db-secure-my-database))

La fonction `GetSession` est ensuite appelée à partir de la fonction `main` (`main.go`).

```go
func main() {
    session := utils.GetSession(cosmosCassandraContactPoint, cosmosCassandraPort, cosmosCassandraUser, cosmosCassandraPassword)
    defer session.Close()
    ...
}
```

Les informations de connectivité et les informations d’identification sont acceptées sous la forme de variables d’environnement (résolues dans la méthode `init`)

```go
func init() {
    cosmosCassandraContactPoint = os.Getenv("COSMOSDB_CASSANDRA_CONTACT_POINT")
    cosmosCassandraPort = os.Getenv("COSMOSDB_CASSANDRA_PORT")
    cosmosCassandraUser = os.Getenv("COSMOSDB_CASSANDRA_USER")
    cosmosCassandraPassword = os.Getenv("COSMOSDB_CASSANDRA_PASSWORD")

    if cosmosCassandraContactPoint == "" || cosmosCassandraUser == "" || cosmosCassandraPassword == "" {
        log.Fatal("missing mandatory environment variables")
    }
}
```

Cela permet ensuite d’exécuter diverses opérations (qui font partie de `operations\setup.go`) sur Azure Cosmos DB en commençant par la création de `keyspace` et `table`.

Comme son nom l’indique, la fonction `DropKeySpaceIfExists` supprime `keyspace` seulement s’il existe.

```go
const dropKeyspace = "DROP KEYSPACE IF EXISTS %s"

func DropKeySpaceIfExists(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(dropKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to drop keyspace", err)
    }
    log.Println("Keyspace dropped")
}
```

La fonction `CreateKeySpace` sert à créer le `keyspace` (`user_profile`)

```go
const createKeyspace = "CREATE KEYSPACE %s WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"

func CreateKeySpace(keyspace string, session *gocql.Session) {
    err := utils.ExecuteQuery(fmt.Sprintf(createKeyspace, keyspace), session)
    if err != nil {
        log.Fatal("Failed to create keyspace", err)
    }
    log.Println("Keyspace created")
}
```

Cela donne lieu ensuite à la création de table (`user`), qui est effectuée par la fonction `CreateUserTable`

```go
const createTable = "CREATE TABLE %s.%s (user_id int PRIMARY KEY, user_name text, user_bcity text)"

func CreateUserTable(keyspace, table string, session *gocql.Session) {
    err := session.Query(fmt.Sprintf(createTable, keyspace, table)).Exec()
    if err != nil {
        log.Fatal("failed to create table ", err)
    }
    log.Println("Table created")
}
```

Une fois que l’espace de clés et la table sont créés, nous appelons les opérations CRUD (qui font partie de `operations\crud.go`). 

`InsertUser` est utilisé pour créer un `User`. Il définit les informations utilisateur (ID, nom et ville) en tant qu’arguments de requête à l’aide de [`Bind`](https://godoc.org/github.com/gocql/gocql#Query.Bind)

```go
const createQuery = "INSERT INTO %s.%s (user_id, user_name , user_bcity) VALUES (?,?,?)"

func InsertUser(keyspace, table string, session *gocql.Session, user model.User) {
    err := session.Query(fmt.Sprintf(createQuery, keyspace, table)).Bind(user.ID, user.Name, user.City).Exec()
    if err != nil {
        log.Fatal("Failed to create user", err)
    }
    log.Println("User created")
}
```

`FindUser` est utilisé pour rechercher un utilisateur (`model\user.go`) à l’aide d’un identifiant utilisateur spécifique alors que [`Scan`](https://godoc.org/github.com/gocql/gocql#Iter.Scan) lie les attributs utilisateur (retournés par Cassandra) à des variables individuelles (`userid`, `name`, `city`). Il ne s’agit là que d’une des façons dont vous pouvez utiliser le résultat obtenu en tant que résultat de la requête de recherche

```go
const selectQuery = "SELECT * FROM %s.%s where user_id = ?"

func FindUser(keyspace, table string, id int, session *gocql.Session) model.User {
    var userid int
    var name, city string
    err := session.Query(fmt.Sprintf(selectQuery, keyspace, table)).Bind(id).Scan(&userid, &name, &city)

    if err != nil {
        if err == gocql.ErrNotFound {
            log.Printf("User with id %v does not exist\n", id)
        } else {
            log.Printf("Failed to find user with id %v - %v\n", id, err)
        }
    }
    return model.User{ID: userid, Name: name, City: city}
}
```

`FindAllUsers` est utilisé pour extraire tous les utilisateurs. [`SliceMap`](https://godoc.org/github.com/gocql/gocql#Iter.SliceMap) est utilisé en tant que raccourci pour obtenir toutes les informations de l’utilisateur sous la forme d’un segment de `map`. Imaginez chaque `map` sous forme de paires clé-valeur, où le nom de colonne (par exemple `user_id`) représente la clé avec sa valeur respective.

```go
const findAllUsersQuery = "SELECT * FROM %s.%s"

func FindAllUsers(keyspace, table string, session *gocql.Session) []model.User {
    var users []model.User
    results, _ := session.Query(fmt.Sprintf(findAllUsersQuery, keyspace, table)).Iter().SliceMap()

    for _, u := range results {
        users = append(users, mapToUser(u))
    }
    return users
}
```

Chaque `map` d’informations utilisateur est converti en `User` à l’aide de la fonction `mapToUser`, qui extrait simplement la valeur de sa colonne respective et l’utilise pour créer une instance du struct `User`

```go
func mapToUser(m map[string]interface{}) model.User {
    id, _ := m["user_id"].(int)
    name, _ := m["user_name"].(string)
    city, _ := m["user_bcity"].(string)

    return model.User{ID: id, Name: name, City: city}
}
```

## <a name="run-the-application"></a>Exécution de l'application

Comme indiqué, l’application accepte les informations de connectivité et les informations d’identification sous la forme de variables d’environnement. 

1. Dans votre compte Azure Cosmos DB, sur le [portail Azure](https://portal.azure.com/), sélectionnez **Chaîne de connexion**. 

    :::image type="content" source="./media/create-cassandra-go/copy-username-connection-string-azure-portal.png" alt-text="Visualiser et copier les détails de la page de chaîne de connexion dans le portail Azure":::

Copiez les valeurs des attributs suivants (`CONTACT POINT`, `PORT`, `USERNAME` et `PRIMARY PASSWORD`), puis affectez-leur les variables d’environnement respectives

```shell
set COSMOSDB_CASSANDRA_CONTACT_POINT=<value for "CONTACT POINT">
set COSMOSDB_CASSANDRA_PORT=<value for "PORT">
set COSMOSDB_CASSANDRA_USER=<value for "USERNAME">
set COSMOSDB_CASSANDRA_PASSWORD=<value for "PRIMARY PASSWORD">
```

Dans la fenêtre de terminal, accédez au dossier approprié. Par exemple :

```shell
cd "C:\git-samples\azure-cosmosdb-cassandra-go-getting-started"
```

2. Dans le terminal, exécutez la commande suivante pour démarrer l’application.

```shell
go run main.go
```

3. La fenêtre de terminal affiche des notifications pour les différentes opérations, notamment la configuration de l’espace de clés et de la table, la création des utilisateurs, etc.

4. Dans le portail Azure, ouvrez **l’Explorateur de données** pour interroger, modifier et utiliser ces nouvelles données. 

    :::image type="content" source="./media/create-cassandra-go/view-data-explorer-go-app.png" alt-text="Afficher les données dans l’Explorateur de données – Azure Cosmos DB":::

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte Azure Cosmos DB avec l’API Cassandra. De plus, vous avez appris à exécuter une application Go, qui crée une base de données et un conteneur Cassandra. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importer des données Cassandra dans Azure Cosmos DB](cassandra-import-data.md)