---
title: Générer une application web en utilisant l’API d’Azure Cosmos DB pour MongoDB et le kit de développement logiciel (SDK) .NET
description: Cet article présente un exemple de code .NET que vous pouvez utiliser pour vous connecter à l’API d’Azure Cosmos DB pour MongoDB et pour l’interroger.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/15/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 5d3c73bdc107ce1971e2958073ad5c100c6bf6bd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349195"
---
# <a name="quickstart-build-a-net-web-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Démarrage rapide : Générer une application web .NET à l’aide de l’API d’Azure Cosmos DB pour MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.JS](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Azure Cosmos DB est la base de données NoSQL rapide de Microsoft avec des API ouvertes pour toute échelle. Rapidement, vous avez la possibilité de créer et d’interroger des documents, des paires clé/valeur et des bases de données orientées graphe, profitant tous de la distribution à l’échelle mondiale et des capacités de mise à l’échelle horizontale au cœur de Cosmos DB. 

Ce démarrage rapide montre comment créer un compte Cosmos avec l’[API d’Azure Cosmos DB pour MongoDB](mongodb-introduction.md). Vous allez ensuite créer et déployer une application web de liste des tâches à l’aide du [pilote .NET MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/).

## <a name="prerequisites-to-run-the-sample-app"></a>Configuration requise pour exécuter l’exemple d’application

Pour exécuter l’exemple, vous devez disposer de [Visual Studio](https://www.visualstudio.com/downloads/) et d’un compte Azure Cosmos DB valide.

Si vous ne disposez pas de Visual Studio, téléchargez [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) avec la charge de travail **Développement ASP.NET et web** installée au moment de l’installation.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>Création d’un compte de base de données

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

L’exemple décrit dans cet article est compatible avec la version 2.6.1 du pilote MongoDB.Driver.

## <a name="clone-the-sample-app"></a>Clonage de l’exemple d’application

Commencez par télécharger l’exemple d’application sur GitHub. 

1. Ouvrez une invite de commandes, créez un nouveau dossier nommé git-samples, puis fermez l’invite de commandes.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git comme Git Bash et utilisez la commande `cd` pour accéder au nouveau dossier d’installation pour l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
    ```

Si vous ne souhaitez pas utiliser git, vous pouvez également [télécharger le projet en tant que fichier .zip](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip).

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez passer à l’étape [Mise à jour de votre chaîne de connexion](#update-your-connection-string). 

Tous les extraits de code suivants proviennent du fichier Dal.cs dans le répertoire DAL.

* Initialisez l’objet client.

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* Récupérez la base de données et la collection.

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* Récupérez tous les documents.

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

Créez une tâche et insérer-la dans la collection.

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   De même, vous pouvez mettre à jour et supprimer des documents à l’aide des méthodes [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) et [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html). 

## <a name="update-your-connection-string"></a>Mise à jour de votre chaîne de connexion

Maintenant, retournez dans le portail Azure afin d’obtenir les informations de votre chaîne de connexion et de les copier dans l’application.

1. Dans le [portail Azure](https://portal.azure.com/), dans votre compte Cosmos, dans le volet de navigation de gauche, cliquez sur **Chaîne de connexion**, puis sur **Clés en lecture-écriture**. Vous utiliserez le bouton Copier sur le côté droit de l’écran pour copier le nom d’utilisateur, le mot de passe et l’hôte dans le fichier Dal.cs à l’étape suivante.

2. Ouvrez le fichier **Dal.cs** dans le répertoire **DAL**. 

3. Copiez votre valeur **nom d’utilisateur** à partir du portail (à l’aide du bouton Copier) et définissez-la comme valeur de **nom d’utilisateur** dans le fichier **Dal.cs**. 

4. Puis, copiez votre valeur **hôte** à partir du portail et définissez-la comme valeur **hôte** dans votre fichier **Dal.cs**. 

5. Puis, copiez votre valeur **mot de passe** à partir du portail et définissez-la comme valeur **mot de passe** dans votre fichier **Dal.cs**. 

Vous venez de mettre à jour votre application avec toutes les informations nécessaires pour communiquer avec Cosmos DB. 
    
## <a name="run-the-web-app"></a>Exécuter l’application web

1. Dans Visual Studio, cliquez avec le bouton droit sur le nom du projet dans l’**Explorateur de solutions**, puis cliquez sur **Gérer les packages NuGet**. 

2. Dans la zone **Parcourir** de NuGet, tapez *MongoDB.Driver*.

3. À partir des résultats, installez la bibliothèque **MongoDB.Driver**. Cette opération permet d’installer le package MondoDB.Driver ainsi que toutes les dépendances.

4. Appuyez sur Ctrl + F5 pour exécuter l’application. Votre application s’affiche dans votre navigateur. 

5. Cliquez sur **Créer** dans le navigateur et créez quelques tâches dans votre application de liste des tâches.

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte Cosmos, à créer une collection et à exécuter une application console. Vous pouvez maintenant importer des données supplémentaires dans votre base de données Cosmos. 

> [!div class="nextstepaction"]
> [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)