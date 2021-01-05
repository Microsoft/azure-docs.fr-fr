---
title: Tutoriel Node.js pour l’API SQL Azure Cosmos DB
description: Didacticiel Node.js qui montre comment se connecter à et interroger Azure Cosmos DB à l’aide de l’API SQL
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: dech
ms.custom: devx-track-js
ms.openlocfilehash: 1115d7bb0d6857aa39f246743df54a6cf3fd3676
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93098655"
---
# <a name="tutorial-build-a-nodejs-console-app-with-the-javascript-sdk-to-manage-azure-cosmos-db-sql-api-data"></a>Tutoriel : Générer une application console Node.js avec le SDK JavaScript pour gérer des données de l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](./create-sql-api-java.md)
> * [Java asynchrone](./create-sql-api-java.md)
> * [Node.JS](sql-api-nodejs-get-started.md)
> 

En tant que développeur, vous pouvez avoir des applications qui utilisent des données de documents NoSQL. Vous pouvez utiliser un compte d’API SQL dans Azure Cosmos DB pour stocker et accéder à ces données de documents. Ce tutoriel vous explique comment générer une application de console Node.js pour créer des ressources Azure Cosmos DB et les interroger.

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Créer et se connecter à un compte Azure Cosmos DB.
> * Configurer votre application.
> * Créer une base de données.
> * Créez un conteneur.
> * Ajouter des éléments au conteneur.
> * Effectuer des opérations de base sur les éléments, le conteneur et la base de données.

## <a name="prerequisites"></a>Prérequis 

Vérifiez que vous disposez des ressources suivantes :

* Un compte Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit des services Azure](https://azure.microsoft.com/pricing/free-trial/)dès aujourd’hui. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js](https://nodejs.org/) version 6.0.0 ou ultérieure.

## <a name="create-azure-cosmos-db-account"></a>Créer un compte Azure Cosmos DB

Commençons par créer un compte Azure Cosmos DB. Si vous avez déjà un compte que vous souhaitez utiliser, vous pouvez passer directement à l’étape [Configurer votre application Node.js](#SetupNode). Si vous utilisez l’émulateur Azure Cosmos DB, suivez les étapes de la section [Émulateur Azure Cosmos DB](local-emulator.md) pour le configurer, puis passez directement à l’étape [Configurer votre application Node.js](#SetupNode). 

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="set-up-your-nodejs-application"></a><a id="SetupNode"></a>Configurer votre application Node.js

Avant de commencer à écrire du code pour générer l’application, vous pouvez générer l’infrastructure de votre application. Suivez les étapes suivantes pour configurer votre application Node.js contenant le code de l'infrastructure :

1. Ouvrez votre terminal préféré.
2. Recherchez le dossier ou le répertoire où vous souhaitez enregistrer votre application Node.js.
3. Créez des fichiers JavaScript vides avec les commandes suivantes :

   * Windows :
     * `fsutil file createnew app.js 0`
     * `fsutil file createnew config.js 0`
     * `md data`
     * `fsutil file createnew data\databaseContext.js 0`

   * Linux/OS X :
     * `touch app.js`
     * `touch config.js`
     * `mkdir data`
     * `touch data/databaseContext.js`

4. Créez et initialisez un fichier `package.json`. Utilisez la commande suivante :
   * ```npm init -y```

5. Installez le module @azure/cosmos via npm. Utilisez la commande suivante :
   * ```npm install @azure/cosmos --save```

## <a name="set-your-apps-configurations"></a><a id="Config"></a>Définir les configurations de votre application

Maintenant que votre application existe, vous devez vous assurer qu’elle peut communiquer avec Azure Cosmos DB. En mettant à jour quelques paramètres de configuration, comme indiqué dans les étapes suivantes, vous pouvez configurer votre application pour communiquer avec Azure Cosmos DB :

1. Ouvrez le fichier *config.js* dans l’éditeur de texte de votre choix.

1. Copiez et collez l’extrait de code ci-dessous dans le fichier *config.js* et attribuez aux propriétés `endpoint` et `key` l’URI de votre point de terminaison Azure Cosmos DB et votre clé primaire. Les noms de base de données et de conteneur sont définis sur **Tâches** et **Éléments**. La clé de partition que vous allez utiliser pour cette application est **/category**.

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/config.js":::

   Vous pouvez trouver les détails sur le point de terminaison et la clé dans le volet **Clés** du [portail Azure](https://portal.azure.com).

   :::image type="content" source="media/sql-api-nodejs-get-started/node-js-tutorial-keys.png" alt-text="Obtenir des clés à partir de la capture d’écran du portail Azure":::

Le SDK JavaScript utilise les termes génériques *conteneur* et *élément*. Un conteneur peut être une collection, un graphique ou une table. Un élément peut être un document, arête/sommet ou ligne, et correspond au contenu à l’intérieur d’un conteneur. Dans l’extrait de code précédent, le code `module.exports = config;` est utilisé pour exporter l’objet de configuration, afin que vous puissiez le référencer dans le fichier *app.js*.

## <a name="create-a-database-and-a-container"></a>Créer une base de données et un conteneur

1. Ouvrez le fichier *databaseContext.js* dans l’éditeur de texte de votre choix.

1. Copiez et collez le code suivant dans le fichier *databaseContext.js*. Ce code définit une fonction qui crée la base de données « Tâches » et le conteneur « Éléments » s’ils n’existent pas déjà dans votre compte Azure Cosmos :

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/data/databaseContext.js" id="createDatabaseAndContainer":::

   Une base de données est le conteneur logique d’éléments partitionnés entre les conteneurs. Vous pouvez créer une base de données à l’aide de la fonction `createIfNotExists` ou create de la classe **Databases**. Un conteneur est composé d’éléments (des documents JSON dans le cas de l’API SQL). Vous pouvez créer un conteneur à l’aide de la fonction `createIfNotExists` ou create de la classe **Containers**. Après avoir créé un conteneur, vous pouvez stocker et interroger les données.

   > [!WARNING]
   > La création d'un conteneur a des conséquences tarifaires. Pour plus d'informations, rendez-vous sur notre [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/).

## <a name="import-the-configuration"></a>Importer la configuration

1. Ouvrez le fichier *app.js* dans l’éditeur de texte de votre choix.

1. Copiez et collez le code ci-dessous pour importer le module `@azure/cosmos`, la configuration et le databaseContext que vous avez définis dans les étapes précédentes. 

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="ImportConfiguration":::

## <a name="connect-to-the-azure-cosmos-account"></a>Se connecter au compte Azure Cosmos

Dans le fichier *app.js* , copiez et collez le code suivant afin d’utiliser le point de terminaison et la clé précédemment enregistrés pour créer un objet CosmosClient.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateClientObjectDatabaseContainer":::

> [!Note]
> Si vous vous connectez à l’ **émulateur Cosmos DB** , désactivez la vérification TLS pour votre processus de nœud :
>   ```javascript
>   process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
>   const client = new CosmosClient({ endpoint, key });
>   ```

Maintenant que vous avez le code permettant d’initialiser le client Azure Cosmos DB, voyons comment utiliser les ressources Azure Cosmos DB.

## <a name="query-items"></a><a id="QueryItem"></a>Éléments de requête

Azure Cosmos DB prend en charge les requêtes enrichies sur les éléments JSON stockés dans chaque conteneur. L’exemple de code suivant montre une requête que vous pouvez exécuter sur les éléments de votre conteneur. Vous pouvez interroger les éléments à l’aide de la fonction de requête de la classe `Items`. Ajoutez le code suivant au fichier *app.js* pour interroger les éléments à partir de votre compte Azure Cosmos :

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="QueryItems":::

## <a name="create-an-item"></a><a id="CreateItem"></a>Créer un élément

Un élément peut être créé à l’aide de la fonction create de la classe `Items`. Lorsque vous utilisez l’API SQL, les éléments sont projetés sous forme de documents, à savoir du contenu JSON (arbitraire) défini par l’utilisateur. Dans ce tutoriel, vous allez créer un élément dans la base de données Tâches.

1. Dans le fichier app.js, définissez la définition de l’élément :

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DefineNewItem":::

1. Ajoutez le code suivant pour créer l’élément précédemment défini :

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateItem":::

## <a name="update-an-item"></a><a id="ReplaceItem"></a>Mettre à jour un élément

Azure Cosmos DB prend en charge le remplacement du contenu d’éléments. Copiez et collez le code suivant dans le fichier *app.js*. Ce code obtient un élément à partir du conteneur et met à jour le champ *isComplete* sur la valeur true.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="UpdateItem":::

## <a name="delete-an-item"></a><a id="DeleteItem"></a>Supprimer un élément

Azure Cosmos DB prend en charge la suppression des éléments JSON. Le code suivant montre comment obtenir un élément par son ID et le supprimer. Copiez et collez le code suivant dans le fichier *app.js*  :

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DeleteItem":::

## <a name="run-your-nodejs-application"></a><a id="Run"></a>Exécuter votre application Node.js

Votre code définitif doit ressembler à ceci :

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js":::

Sur votre terminal, recherchez votre fichier ```app.js``` et exécutez la commande :

```bash 
node app.js
```

La sortie de votre application de prise en main doit s’afficher. La sortie doit correspondre au texte en exemple ci-dessous.

```
Created database:
Tasks

Created container:
Items

Querying container: Items
1 - Pick up apples and strawberries.

Created new item: 3 - Complete Cosmos DB Node.js Quickstart ⚡

Updated item: 3 - Complete Cosmos DB Node.js Quickstart ⚡
Updated isComplete to true

Deleted item with id: 3
```

## <a name="get-the-complete-nodejs-tutorial-solution"></a><a id="GetSolution"></a>Obtenir la solution complète du didacticiel Node.js

Si vous n’avez pas le temps de suivre les étapes de ce didacticiel, ou que vous voulez simplement télécharger le code, vous pouvez l’obtenir à partir de [GitHub](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started ).

Pour exécuter la solution de prise en main qui contient l’ensemble du code de cet article, vous devez disposer des éléments suivants :

* Un [compte Azure Cosmos DB][create-account].
* La solution [Prise en main](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started) disponible sur GitHub.

Installez les dépendances du projet via npm. Utilisez la commande suivante :

* ```npm install``` 

Ensuite, dans le fichier ```config.js```, mettez à jour les valeurs de config.endpoint et de config.key, comme indiqué dans [Étape 3 : Définir les configurations de votre application](#Config).  

Sur votre terminal, recherchez votre fichier ```app.js``` et exécutez la commande :  

```bash  
node app.js 
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’avez plus besoin de ces ressources, vous pouvez supprimer le groupe de ressources, le compte Azure Cosmos DB et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources que vous avez utilisé pour le compte Azure Cosmos DB, sélectionnez **Supprimer** , puis confirmez le nom du groupe de ressources à supprimer.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Surveiller un compte Azure Cosmos DB](./monitor-cosmos-db.md)

[create-account]: create-sql-api-dotnet.md#create-account