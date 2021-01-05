---
title: Créer une application web Flask en Python à l’aide de l’API d’Azure Cosmos DB pour MongoDB
description: Cet article présente un exemple de code Python Flask que vous pouvez utiliser pour vous connecter à l’API d’Azure Cosmos DB pour MongoDB et pour l’interroger.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: python
ms.topic: quickstart
ms.date: 12/26/2018
ms.custom: devx-track-python
ms.openlocfilehash: 3d066005ac18bf42f913885558bf6f0ee5fc50c4
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349161"
---
# <a name="quickstart-build-a-python-app-using-azure-cosmos-dbs-api-for-mongodb"></a>Démarrage rapide : Générer une application Python en utilisant l’API d’Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.JS](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Dans ce guide de démarrage rapide, vous allez utiliser un compte d’API Azure Cosmos DB pour Mongo DB ou l’émulateur Azure Cosmos DB pour exécuter une application web To-Do Python Flask clonée à partir de GitHub. Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure. Vous pouvez aussi utiliser l’[émulateur Azure Cosmos DB](local-emulator.md). 
- [Python 3.6+](https://www.python.org/downloads/)
- [Visual Studio Code](https://code.visualstudio.com/Download) avec l’[extension Python](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python).

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

À présent, nous allons cloner une application Flask-MongoDB à partir de GitHub, configurer la chaîne de connexion, puis l’exécuter. Vous pouvez constater à quel point il est facile de travailler par programmation avec des données.

1. Ouvrez une invite de commandes, créez un nouveau dossier nommé git-samples, puis fermez l’invite de commandes.

    ```bash
    md "C:\git-samples"
    ```

2. Ouvrez une fenêtre de terminal git comme Git Bash et utilisez la commande `cd` pour accéder au nouveau dossier d’installation pour l’exemple d’application.

    ```bash
    cd "C:\git-samples"
    ```

3. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample.git
    ```
3. Exécutez la commande suivante pour installer les modules Python.

    ```bash 
    pip install -r .\requirements.txt
    ```
4. Ouvrez le dossier dans Visual Studio Code.

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez directement passer à la section [Exécution de l’application web](#run-the-web-app). 

Les extraits de code suivants proviennent tous du fichier *app.py* et utilisent la chaîne de connexion pour l’émulateur Azure Cosmos DB local. Le mot de passe doit être fractionné comme indiqué ci-dessous afin de prendre en charge les barres obliques, qui sinon ne pourraient pas être analysées.

* Initialisez le client MongoDB, récupérez la base de données et authentifiez-vous.

    ```python
    client = MongoClient("mongodb://127.0.0.1:10250/?ssl=true") #host uri
    db = client.test    #Select the database
    db.authenticate(name="localhost",password='C2y6yDjf5' + r'/R' + '+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw' + r'/Jw==')
    ```

* Récupérez la collection, ou créez-la si elle n’existe pas.

    ```python
    todos = db.todo #Select the collection
    ```

* Créer l’application

    ```Python
    app = Flask(__name__)
    title = "TODO with Flask"
    heading = "ToDo Reminder"
    ```
    
## <a name="run-the-web-app"></a>Exécuter l’application web

1. Vérifiez que l’émulateur Azure Cosmos DB est en cours d’exécution.

2. Ouvrez une fenêtre de terminal et accédez avec la commande `cd` au répertoire dans lequel l’application est enregistrée.

3. Définissez ensuite la variable d'environnement de l'application Flask avec `set FLASK_APP=app.py`, `$env:FLASK_APP = app.py` pour les éditeurs PowerShell, ou `export FLASK_APP=app.py` si vous utilisez un Mac. 

4. Exécutez l’application avec `flask run` et accédez à *http:\//127.0.0.1:5000/* .

5. Ajoutez et supprimez des tâches, et observez l’ajout ou la modification correspondante dans la collection.

## <a name="create-a-database-account"></a>Création d’un compte de base de données

Si vous souhaitez tester le code par rapport à un compte Azure Cosmos DB actif, accédez au portail Azure pour créer un compte.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="update-your-connection-string"></a>Mise à jour de votre chaîne de connexion

Pour tester le code par rapport au compte de Azure Cosmos DB actif, récupérez vos informations de chaîne de connexion. Ensuite, copiez-les dans l’application.

1. Dans le portail Azure, dans votre compte Azure Cosmos DB, dans le volet de navigation de gauche, sélectionnez **Chaîne de connexion**, puis **Clés en lecture-écriture**. Vous utiliserez les boutons de copie sur le côté droit de l’écran pour copier le nom d’utilisateur, la chaîne de connexion et le mot de passe. 

2. Ouvrez le fichier *app.py* dans le répertoire racine.

3. Copiez votre valeur de **nom d’utilisateur** à partir du portail (à l’aide du bouton Copier) et définissez-la comme valeur de **name** dans le fichier *app.py*.

4. Ensuite, copiez votre valeur de **chaîne de connexion** à partir du portail et affectez-la comme valeur de **MongoClient** dans votre fichier *app.py*.

5. Pour finir, copiez votre valeur de **mot de passe** à partir du portail et affectez-la comme valeur de **password** dans votre fichier *app.py*.

Vous venez de mettre à jour votre application avec toutes les informations nécessaires pour communiquer avec Azure Cosmos DB. Vous pouvez l’exécuter de la même manière qu’avant.

## <a name="deploy-to-azure"></a>Déployer dans Azure

Pour déployer cette application, vous pouvez créer une application web dans Azure et activer le déploiement continu avec une duplication (fork) de ce référentiel GitHub. Suivez ce [didacticiel](../app-service/deploy-continuous-deployment.md) pour configurer le déploiement continu avec GitHub dans Azure.

Lors du déploiement sur Azure, vous devez supprimer vos clés d’application et vérifier que la section ci-dessous n’est pas commentée :

```python
    client = MongoClient(os.getenv("MONGOURL"))
    db = client.test    #Select the database
    db.authenticate(name=os.getenv("MONGO_USERNAME"),password=os.getenv("MONGO_PASSWORD"))
```

Vous devez ensuite ajouter vos valeurs MONGOURL, MONGO_PASSWORD et MONGO_USERNAME aux paramètres d’application. Vous pouvez suivre ce [didacticiel](../app-service/configure-common.md#configure-app-settings) pour en savoir plus sur les paramètres d’application dans Azure Web Apps.

Si vous ne souhaitez pas créer de duplication (fork) de ce dépôt, vous pouvez également sélectionner le bouton **Déployer sur Azure** ci-dessous. Vous devez ensuite aller dans Azure et configurer les paramètres d’application avec vos informations de compte Azure Cosmos DB.

<a href="https://deploy.azure.com/?repository=https://github.com/heatherbshapiro/To-Do-List---Flask-MongoDB-Example" target="_blank">
<img src="https://azuredeploy.net/deploybutton.png" alt="Click to Deploy to Azure">
</a>

> [!NOTE]
> Si vous envisagez de stocker votre code dans GitHub ou d’autres options de contrôle de code source, n’oubliez pas de supprimer vos chaînes de connexion du code. Elles peuvent être définies avec des paramètres d’application pour l’application web à la place.

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte d’API Azure Cosmos DB pour Mongo DB et à utiliser l’émulateur Azure Cosmos DB pour exécuter une application web To-Do Python Flask clonée à partir de GitHub. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB. 

> [!div class="nextstepaction"]
> [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)