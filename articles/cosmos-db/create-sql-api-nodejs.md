---
title: 'Démarrage rapide : Utiliser Node.js pour interroger les données d’un compte d’API SQL Azure Cosmos DB'
description: Comment utiliser Node.js pour créer une application qui se connecte à un compte d’API SQL Azure Cosmos DB pour interroger des données.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: anfeldma
ms.custom: devx-track-js
ms.openlocfilehash: b4ed27e1515e898a71fb503bb0f260c608ef9f6f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93090203"
---
# <a name="quickstart-use-nodejs-to-connect-and-query-data-from-azure-cosmos-db-sql-api-account"></a>Démarrage rapide : Utiliser Node.js pour se connecter à un compte d’API SQL Azure Cosmos DB et interroger ses données
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> - [.NET V3](create-sql-api-dotnet.md)
> - [.NET V4](create-sql-api-dotnet-V4.md)
> - [Kit SDK Java v4](create-sql-api-java.md)
> * [Spring Data v3](create-sql-api-spring-data.md)
> - [Node.JS](create-sql-api-nodejs.md)
> - [Python](create-sql-api-python.md)
> - [Xamarin](create-sql-api-xamarin-dotnet.md)

Dans ce guide de démarrage rapide, vous allez créer et gérer un compte d’API SQL Azure Cosmos DB à partir du portail Azure et à l’aide d’une application Node.js clonée à partir de GitHub. Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale.

## <a name="walkthrough-video"></a>Vidéo de la procédure pas à pas

Regardez cette vidéo pour connaître la procédure pas à pas complète correspondant au contenu de cet article.

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Quickstart-Use-Nodejs-to-connect-and-query-data-from-Azure-Cosmos-DB-SQL-API-account/player]

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure. Vous pouvez aussi utiliser l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) avec l’URI `https://localhost:8081` et la clé `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`.
- [Node.js 6.0.0 ou ultérieur](https://nodejs.org/).
- [Git](https://www.git-scm.com/downloads).

## <a name="create-an-azure-cosmos-account"></a>Créer un compte Azure Cosmos

Pour ce guide de démarrage rapide, vous pouvez utiliser l’option [Essayer Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) pour créer un compte Azure Cosmos.

1. Accédez à la page [Essayer Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/).

1. Choisissez le compte d’API **SQL** et sélectionnez **Créer**. Connectez-vous en utilisant votre compte Microsoft.

1. Votre compte Azure Cosmos doit être prêt dès que vous êtes connecté. Sélectionnez **Ouvrir dans le portail Azure** pour ouvrir le compte que vous venez de créer.

L’option « Essayer Azure Cosmos DB gratuitement » ne nécessite pas d’abonnement Azure et vous permet de disposer d’un compte Azure Cosmos pour une période de 30 jours. Si vous souhaitez utiliser le compte Azure Cosmos plus longtemps, [créez le compte](create-cosmosdb-resources-portal.md#create-an-azure-cosmos-db-account) dans votre abonnement Azure.

## <a name="add-a-container"></a>Ajouter un conteneur

Vous pouvez désormais utiliser l’outil Explorateur de données dans le portail Azure pour créer une base de données et un conteneur.

1. Sélectionnez **Explorateur de données** > **Nouveau conteneur**.

   La zone **Ajouter un conteneur** s’affiche à l’extrême droite : il peut donc être nécessaire de faire défiler à droite pour l’afficher.

   :::image type="content" source="./media/create-sql-api-nodejs/azure-cosmosdb-data-explorer.png" alt-text="Explorateur de données du portail Azure, volet Ajouter un conteneur":::

2. Dans la page **Ajouter un conteneur** , entrez les paramètres du nouveau conteneur.

   | Paramètre           | Valeur suggérée | Description                                                                                                                                                                                                                                                                                                                                                                           |
   | ----------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | **ID de base de données**   | Tâches           | Entrez _Tasks_ comme nom pour la nouvelle base de données. Les noms de base de données doivent comporter entre 1 et 255 caractères, et ne peuvent pas contenir les caractères `/, \\, #, ?`, ni un espace de fin. Cochez l’option **Provisionner le débit de base de données** qui vous permet de partager le débit provisionné pour la base de données entre tous les conteneurs au sein de la base de données. Cette option permet également de réduire les coûts. |
   | **Débit**    | 400             | Laissez le débit sur 400 unités de requête par seconde (RU/s). Si vous souhaitez réduire la latence, vous pourrez augmenter le débit par la suite.                                                                                                                                                                                                                                                    |
   | **ID de conteneur**  | Éléments           | Entrez _Éléments_ comme nom de votre nouveau conteneur. Les ID de conteneur sont soumis aux mêmes exigences en termes de caractères que les noms de base de données.                                                                                                                                                                                                                                                               |
   | **Clé de partition** | /category       | L’exemple décrit dans cet article utilise _/category_ comme clé de partition.                                                                                                                                                                                                                                                                                                           |

   Outre les paramètres précédents, vous pouvez ajouter des **clés uniques** pour le conteneur. Laissez le champ vide dans cet exemple. Les clés uniques permettent aux développeurs d’ajouter une couche d’intégrité des données à la base de données. En créant une stratégie de clé unique durant la création d’un conteneur, vous garantissez l’unicité d’une ou de plusieurs valeurs par clé de partition. Pour en savoir plus, référez-vous à l’article [Clés uniques dans Azure Cosmos DB](unique-keys.md).

   Sélectionnez **OK**. L’Explorateur de données affiche la nouvelle base de données et le nouveau conteneur.

## <a name="add-sample-data"></a>Ajouter un exemple de données

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>Interroger vos données

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

À présent, nous allons cloner une application Node.js à partir de GitHub, configurer la chaîne de connexion et l’exécuter.

1. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

   ```bash
   git clone https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started.git
   ```

## <a name="review-the-code"></a>Vérifier le code

Cette étape est facultative. Pour savoir comment les ressources de base de données Azure Cosmos sont créées dans le code, vous pouvez examiner les extraits de code suivants. Sinon, vous pouvez passer à l’étape [Mise à jour de votre chaîne de connexion](#update-your-connection-string).

Si vous connaissez déjà la version précédente du SDK JavaScript SQL, vous avez peut-être l’habitude de voir les termes _collection_ et _document_. Étant donné qu’Azure Cosmos DB prend en charge [plusieurs modèles d’API](introduction.md), les [versions 2.0 et ultérieures du SDK JavaScript](https://www.npmjs.com/package/@azure/cosmos) utilisent les termes génériques _conteneur_ , qui peut être une collection, un graphe ou une table, et _élément_ pour décrire le contenu du conteneur.

Le SDK JavaScript Cosmos DB est nommé « @azure/cosmos ». Il peut être installé à partir de npm...

```bash
npm install @azure/cosmos
```

Tous les extraits de code suivants proviennent du fichier _app.js_.

- Le `CosmosClient` est importé à partir du package npm `@azure/cosmos`.

  ```javascript
  const CosmosClient = require("@azure/cosmos").CosmosClient;
  ```

- Un nouvel objet `CosmosClient` est initialisé.

  ```javascript
  const client = new CosmosClient({ endpoint, key });
  ```

- Sélectionnez la base de données « Tasks ».

  ```javascript
  const database = client.database(databaseId);
  ```

- Sélectionnez le conteneur ou la collection « Items ».

  ```javascript
  const container = database.container(containerId);
  ```

- Sélectionnez tous les éléments dans le conteneur « Items ».

  ```javascript
  // query to return all items
  const querySpec = {
    query: "SELECT * from c"
  };

  const { resources: items } = await container.items
    .query(querySpec)
    .fetchAll();
  ```

- Créer un élément

  ```javascript
  const { resource: createdItem } = await container.items.create(newItem);
  ```

- Mettre à jour un élément

  ```javascript
  const { id, category } = createdItem;

  createdItem.isComplete = true;
  const { resource: updatedItem } = await container
    .item(id, category)
    .replace(createdItem);
  ```

- Supprimer un élément

  ```javascript
  const { resource: result } = await container.item(id, category).delete();
  ```

> [!NOTE]
> Dans les deux méthodes « Mettre à jour » et « Supprimer », l’élément doit être sélectionné dans la base de données en appelant `container.item()`. Les deux paramètres transmis sont l’ID de l’élément et la clé de partition de l’élément. Dans ce cas, la clé de partition est la valeur du champ « category ».

## <a name="update-your-connection-string"></a>Mise à jour de votre chaîne de connexion

Revenez maintenant au portail Azure pour afficher les détails de la chaîne de connexion de votre compte Azure Cosmos. Copiez la chaîne de connexion dans l’application afin que celle-ci puisse se connecter à votre base de données.

1. Dans le [portail Azure](https://portal.azure.com/), dans votre compte Azure Cosmos DB, sélectionnez **Clés** dans le volet de navigation de gauche, puis sélectionnez **Clés en lecture-écriture**. Utilisez les boutons de copie sur le côté droit de l’écran pour copier l’URI et la clé primaire dans le fichier _app.js_ à l’étape suivante.

   :::image type="content" source="./media/create-sql-api-dotnet/keys.png" alt-text="Afficher et copier une clé d’accès dans le panneau Clés du portail Azure":::

2. Ouvrez le fichier _config.js_.

3. Copiez votre valeur d’URI à partir du portail (à l’aide du bouton Copier) et définissez-la comme valeur de la clé du point de terminaison dans le fichier _config.js_.

   `endpoint: "<Your Azure Cosmos account URI>"`

4. Ensuite, copiez la valeur de votre CLÉ PRIMAIRE à partir du portail et définissez-la comme valeur pour `config.key` dans _config.js_. Vous venez de mettre à jour votre application avec toutes les informations nécessaires pour communiquer avec Azure Cosmos DB.

   `key: "<Your Azure Cosmos account key>"`

## <a name="run-the-app"></a>Exécuter l’application

1. Exécutez `npm install` sur un terminal pour installer le package npm « @azure/cosmos ».

2. Exécutez `node app.js` sur un terminal pour démarrer votre application Node.

3. Les deux éléments que vous avez créés précédemment dans le cadre de ce guide de démarrage rapide sont listés. Un autre élément est créé. L’indicateur « isComplete » de cet élément devient « true », puis l’élément est supprimé.

Vous pouvez continuer à effectuer des essais avec cet exemple d’application ou revenir à l’Explorateur de données et modifier et utiliser vos données.

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte Azure Cosmos DB, à créer un conteneur avec l’Explorateur de données et à exécuter une application Node.js. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importer des données dans Azure Cosmos DB](import-data.md)
