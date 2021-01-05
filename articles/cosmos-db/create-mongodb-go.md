---
title: Connecter une application Go à l’API Azure Cosmos DB pour MongoDB
description: Ce guide de démarrage rapide montre comment connecter une application Go existante à l’API Azure Cosmos DB pour MongoDB.
author: abhirockzz
ms.author: abhishgu
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: go
ms.topic: quickstart
ms.date: 04/24/2020
ms.openlocfilehash: 3f6ff0f81c1979297ed0144eca336cb252c2e787
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574358"
---
# <a name="quickstart-connect-a-go-application-to-azure-cosmos-dbs-api-for-mongodb"></a>Démarrage rapide : Connecter une application Go à l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.JS](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale. Dans ce guide de démarrage rapide, vous créez et gérez un compte Azure Cosmos DB en utilisant Azure Cloud Shell, vous clonez un exemple d’application modèle GitHub existant, puis vous le configurez pour qu'il fonctionne avec Azure Cosmos DB. 

L’exemple d’application est un outil de gestion `todo` avec ligne de commande écrit en Go. L’API Azure Cosmos DB pour MongoDB est [compatible avec le protocole filaire MongoDB](./mongodb-introduction.md#wire-protocol-compatibility), permettant à n’importe quel pilote client MongoDB de s’y connecter. Comme cette application utilise le [pilote Go pour MongoDB](https://github.com/mongodb/mongo-go-driver), elle sait que les données sont stockées dans une base de données Azure Cosmos DB.

## <a name="prerequisites"></a>Prérequis
- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free). Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure. Vous pouvez aussi utiliser l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) avec la chaîne de connexion `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Go](https://golang.org/) installé sur votre ordinateur et une expérience de travail avec Go.
- [Git](https://git-scm.com/downloads).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Exécutez les commandes suivantes pour cloner l’exemple de référentiel.

1. Ouvrez une invite de commandes, créez un nouveau dossier nommé `git-samples`, puis fermez l’invite de commandes.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git comme Git Bash et utilisez la commande `cd` pour accéder au nouveau dossier d’installation pour l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur. 

    ```bash
    git clone https://github.com/Azure-Samples/cosmosdb-go-mongodb-quickstart
    ```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment l’application fonctionne, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez directement passer à la section [Exécution de l’application](#run-the-application). La disposition de l’application est la suivante :

```bash
.
├── go.mod
├── go.sum
└── todo.go
```

Tous les extraits suivants proviennent du fichier `todo.go`.

### <a name="connecting-the-go-app-to-azure-cosmos-db"></a>Connecter l’application Go à Azure Cosmos DB

[`clientOptions`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions) encapsule la chaîne de connexion pour Azure Cosmos DB, qui est transmise à l’aide d’une variable d’environnement (voir la section suivante pour plus détails). La connexion est initialisée à l’aide de l’élément [`mongo.NewClient`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#NewClient) vers lequel l’instance `clientOptions` est passée. La [fonction `Ping`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Client.Ping) est appelée pour confirmer la réussite de la connectivité (stratégie de basculement rapide)

```go
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
    defer cancel()

    clientOptions := options.Client().ApplyURI(mongoDBConnectionString).SetDirect(true)
    
    c, err := mongo.NewClient(clientOptions)
    err = c.Connect(ctx)
    if err != nil {
        log.Fatalf("unable to initialize connection %v", err)
    }

    err = c.Ping(ctx, nil)
    if err != nil {
        log.Fatalf("unable to connect %v", err)
    }
```

> [!NOTE] 
> L’utilisation de la configuration [`SetDirect(true)`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo/options?tab=doc#ClientOptions.SetDirect) est importante car sans elle vous obtenez l’erreur de connectivité suivante : `unable to connect connection(cdb-ms-prod-<azure-region>-cm1.documents.azure.com:10255[-4]) connection is closed`
>

### <a name="create-a-todo-item"></a>Créer un élément `todo`

Pour créer un élément `todo`, nous obtenons un handle vers un élément [`mongo.Collection`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection) et appelons la fonction [`InsertOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.InsertOne). 

```go
func create(desc string) {
    c := connect()
    ctx := context.Background()
    defer c.Disconnect(ctx)

    todoCollection := c.Database(database).Collection(collection)
    r, err := todoCollection.InsertOne(ctx, Todo{Description: desc, Status: statusPending})
    if err != nil {
        log.Fatalf("failed to add todo %v", err)
    }
```

Nous passons un struct `Todo` contenant la description et l’état (initialement défini sur `pending`)

```go
type Todo struct {
    ID          primitive.ObjectID `bson:"_id,omitempty"`
    Description string             `bson:"description"`
    Status      string             `bson:"status"`
}
```
### <a name="list-todo-items"></a>Répertorier les éléments `todo`

Nous pouvons répertorier les éléments TODO selon des critères. Un élément [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D) est créé pour encapsuler les critères de filtre

```go
func list(status string) {
    .....
    var filter interface{}
    switch status {
    case listAllCriteria:
        filter = bson.D{}
    case statusCompleted:
        filter = bson.D{{statusAttribute, statusCompleted}}
    case statusPending:
        filter = bson.D{{statusAttribute, statusPending}}
    default:
        log.Fatal("invalid criteria for listing todo(s)")
    }
```

[`Find`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.Find) permet de rechercher des documents en fonction du filtre, et le résultat est converti en une tranche de `Todo`

```go
    todoCollection := c.Database(database).Collection(collection)
    rs, err := todoCollection.Find(ctx, filter)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
    var todos []Todo
    err = rs.All(ctx, &todos)
    if err != nil {
        log.Fatalf("failed to list todo(s) %v", err)
    }
```

Enfin, les informations sont affichées sous forme de tableau

```go
    todoTable := [][]string{}

    for _, todo := range todos {
        s, _ := todo.ID.MarshalJSON()
        todoTable = append(todoTable, []string{string(s), todo.Description, todo.Status})
    }

    table := tablewriter.NewWriter(os.Stdout)
    table.SetHeader([]string{"ID", "Description", "Status"})

    for _, v := range todoTable {
        table.Append(v)
    }
    table.Render()
```

### <a name="update-a-todo-item"></a>Mettre à jour un élément `todo`

Un élément `todo` peut être mis à jour en fonction de sa valeur `_id`. Un filtre [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D) est créé en fonction de l’élément `_id`, et un autre filtre est créé pour les informations mises à jour, soit un nouvel état (`completed` ou `pending`) dans ce cas. Enfin, la fonction [`UpdateOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.UpdateOne) est appelée avec le filtre et le document mis à jour

```go
func update(todoid, newStatus string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
    filter := bson.D{{"_id", oid}}
    update := bson.D{{"$set", bson.D{{statusAttribute, newStatus}}}}
    _, err = todoCollection.UpdateOne(ctx, filter, update)
    if err != nil {
        log.Fatalf("failed to update todo %v", err)
    }
```

### <a name="delete-a-todo"></a>Supprimer un élément `todo`

Un élément `todo` est supprimé en fonction de sa valeur `_id`, puis il est encapsulé sous la forme d’une instance [`bson.D`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/bson?tab=doc#D). [`DeleteOne`](https://pkg.go.dev/go.mongodb.org/mongo-driver@v1.3.2/mongo?tab=doc#Collection.DeleteOne) est appelé pour supprimer le document.

```go
func delete(todoid string) {
....
    todoCollection := c.Database(database).Collection(collection)
    oid, err := primitive.ObjectIDFromHex(todoid)
    if err != nil {
        log.Fatalf("invalid todo ID %v", err)
    }
    filter := bson.D{{"_id", oid}}
    _, err = todoCollection.DeleteOne(ctx, filter)
    if err != nil {
        log.Fatalf("failed to delete todo %v", err)
    }
}
```

## <a name="build-the-application"></a>Créer l’application

Accédez au répertoire dans lequel vous avez cloné l’application, puis générez l’application (en utilisant `go build`).

```bash
cd monogdb-go-quickstart
go build -o todo
```

Pour confirmer que l’application a été correctement générée.

```bash
./todo --help
```

## <a name="setup-azure-cosmos-db"></a>Configurer Azure Cosmos DB

### <a name="sign-in-to-azure"></a>Connexion à Azure

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, vous devez exécuter Azure CLI version 2.0 ou une version ultérieure pour poursuivre la procédure décrite dans cet article. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]. 

Si vous utilisez une interface de ligne de commande Azure installée, connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/reference-index#az-login) et suivez les instructions à l’écran. Vous pouvez ignorer cette étape si vous utilisez Azure Cloud Shell.

```azurecli
az login 
``` 
   
### <a name="add-the-azure-cosmos-db-module"></a>Ajouter le module Azure Cosmos DB

Si vous utilisez une interface de ligne de commande Azure installée, vérifiez si le composant `cosmosdb` est déjà installé en exécutant la commande `az`. Si `cosmosdb` figure dans la liste des commandes de base, passez à la commande suivante. Vous pouvez ignorer cette étape si vous utilisez Azure Cloud Shell.

Si `cosmosdb` n’est pas dans la liste des commandes de base, réinstallez [Azure CLI](/cli/azure/install-azure-cli).

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources](../azure-resource-manager/management/overview.md) avec la commande [az group create](/cli/azure/group#az-group-create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure comme les applications web, les bases de données et les comptes de stockage sont déployées et gérées. 

L’exemple suivant crée un groupe de ressources dans la région Europe Ouest. Choisissez un nom unique pour le groupe de ressources.

Si vous utilisez Azure Cloud Shell, sélectionnez **Essayer**, suivez les invites à l’écran pour vous connecter, puis copiez la commande dans l’invite de commandes.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

### <a name="create-an-azure-cosmos-db-account"></a>Création d’un compte Azure Cosmos DB

Créez un compte Cosmos à l’aide de la commande [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create).

Dans la commande suivante, indiquez le nom unique de votre compte Cosmos là où se trouve l’espace réservé `<cosmosdb-name>`. Ce nom unique sera utilisé en tant que point de terminaison Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`). Pour cette raison, le nom doit être unique sur l’ensemble des comptes Cosmos dans Azure. 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

Le paramètre `--kind MongoDB` prend en charge les connexions clientes MongoDB.

Une fois le compte Azure Cosmos DB créé, Azure CLI affiche des informations similaires à celles de l’exemple suivant. 

> [!NOTE]
> Cet exemple utilise JSON comme format de sortie de l’interface de ligne de commande Azure, qui constitue le format par défaut. Pour utiliser un autre format de sortie, consultez [Formats de sortie pour les commandes Azure CLI](/cli/azure/format-output-azure-cli).

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

### <a name="retrieve-the-database-key"></a>Récupérer la clé de la base de données

Pour vous connecter à une base de données Cosmos, vous avez besoin de la clé de la base de données. Utilisez la commande [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az-cosmosdb-keys-list) pour récupérer la clé primaire.

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

Azure CLI génère des informations semblables à ce qui suit. 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

## <a name="configure-the-application"></a>Configuration de l'application 

<a name="devconfig"></a>
### <a name="export-the-connection-string-mongodb-database-and-collection-names-as-environment-variables"></a>Exportez la chaîne de connexion, la base de données MongoDB et les noms de collection en tant que variables d’environnement. 

```bash
export MONGODB_CONNECTION_STRING="mongodb://<COSMOSDB_ACCOUNT_NAME>:<COSMOSDB_PASSWORD>@<COSMOSDB_ACCOUNT_NAME>.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&maxIdleTimeMS=120000&appName=@<COSMOSDB_ACCOUNT_NAME>@"
```

> [!NOTE] 
> L’option `ssl=true` est importante, en raison des exigences de Cosmos DB. Pour plus d’informations, consultez [Exigences relatives à la chaîne de connexion](connect-mongodb-account.md#connection-string-requirements).
>

Pour la variable d’environnement `MONGODB_CONNECTION_STRING`, remplacez les espaces réservés pour `<COSMOSDB_ACCOUNT_NAME>` et `<COSMOSDB_PASSWORD>`

1. `<COSMOSDB_ACCOUNT_NAME>`: nom du compte Azure Cosmos DB que vous avez créé
2. `<COSMOSDB_PASSWORD>`: clé de base de données extraite à l’étape précédente

```bash
export MONGODB_DATABASE=todo-db
export MONGODB_COLLECTION=todos
```

Vous pouvez choisir vos valeurs préférées pour `MONGODB_DATABASE` et `MONGODB_COLLECTION` ou les conserver en l’état.

## <a name="run-the-application"></a>Exécution de l'application

Pour créer un élément `todo`

```bash
./todo --create "Create an Azure Cosmos DB database account"
```

En cas de réussite, vous devriez voir une sortie avec l’élément MongoDB `_id` du document nouvellement créé :

```bash
added todo ObjectID("5e9fd6befd2f076d1f03bd8a")
```

Créer un autre élément `todo`

```bash
./todo --create "Get the MongoDB connection string using the Azure CLI"
```

Répertorier tous les éléments `todo`

```bash
./todo --list all
```

Vous devriez voir les éléments que vous venez d’ajouter dans un format tabulaire

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | pending   |
|                            | database account               |           |
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

Pour mettre à jour l’état d’un élément `todo` (par exemple, le définir à l’état `completed`), utilisez l’ID `todo`

```bash
./todo --update 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Répertorier uniquement les éléments `todo`terminés

```bash
./todo --list completed
```

Vous devriez voir celui que vous venez de mettre à jour

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6b1bcd2fa6bd267d4c4" | Create an Azure Cosmos DB      | completed |
|                            | database account               |           |
+----------------------------+--------------------------------+-----------+
```

### <a name="view-data-in-data-explorer"></a>Afficher les données dans l’Explorateur de données

Les données stockées dans Azure Cosmos DB peuvent être affichées et interrogées dans le portail Azure.

Pour afficher, interroger et manipuler les données utilisateur créées à l’étape précédente, connectez-vous au [portail Azure](https://portal.azure.com) dans votre navigateur web.

Dans la zone de recherche en haut de la page, entrez **Azure Cosmos DB**. Lorsque le panneau de votre compte Cosmos s’ouvre, sélectionnez votre compte Cosmos. Dans le volet de navigation gauche, sélectionnez **Explorateur de données**. Développez votre collection dans le volet Collections, pour pouvoir afficher les documents de la collection, interroger les données et même créer et exécuter des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur. 

:::image type="content" source="./media/create-mongodb-go/go-cosmos-db-data-explorer.png" alt-text="Explorateur de données montrant le document créé":::


Supprimer un élément `todo` à l’aide de son ID

```bash
./todo --delete 5e9fd6b1bcd2fa6bd267d4c4,completed
```

Répertorier les éléments `todo` à confirmer

```bash
./todo --list all
```

L’élément `todo` que vous venez de supprimer ne devrait pas apparaître

```bash
+----------------------------+--------------------------------+-----------+
|             ID             |          DESCRIPTION           |  STATUS   |
+----------------------------+--------------------------------+-----------+
| "5e9fd6befd2f076d1f03bd8a" | Get the MongoDB connection     | pending   |
|                            | string using the Azure CLI     |           |
+----------------------------+--------------------------------+-----------+
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte d’API Azure Cosmos DB MongoDB à l’aide d’Azure Cloud Shell et à créer puis exécuter une application de ligne de commande Go pour gérer des éléments `todo`. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)