---
title: 'Démarrage rapide : Connecter une application MongoDB Node.js à Azure Cosmos DB'
description: Ce démarrage rapide montre comment connecter une application MongoDB écrite en Node.js à Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 05/21/2019
ms.custom: seo-javascript-september2019, seo-javascript-october2019, devx-track-js, devx-track-azurecli
ms.openlocfilehash: cfeb19fa36bf65021fadc7ac3bc60a9f3a620602
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349076"
---
# <a name="quickstart-migrate-an-existing-mongodb-nodejs-web-app-to-azure-cosmos-db"></a>Démarrage rapide : Migrer une application web Node.js MongoDB existante vers Azure Cosmos DB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.JS](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

Dans ce guide de démarrage rapide, vous allez créer et gérer un compte d’API Azure Cosmos DB pour Mongo DB à l’aide d’Azure Cloud Shell et avec une application MEAN (MongoDB, Express, Angular et Node.js) clonée à partir de GitHub. Azure Cosmos DB est un service de base de données multimodèle qui vous permet de créer et d’interroger rapidement des bases de données de documents, de tables, de paires clé/valeur et de graphes avec des capacités de distribution mondiale et de mise à l’échelle horizontale.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] Vous pouvez également [essayez Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans disposer d’un abonnement Azure. Vous pouvez aussi utiliser l’[émulateur Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) avec la chaîne de connexion `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.

- [Node.js](https://nodejs.org/) et une bonne connaissance de Node.js.

- [Git](https://git-scm.com/downloads).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.


## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Exécutez les commandes suivantes pour cloner l’exemple de référentiel. Cet exemple de référentiel contient l’application [MEAN.js](https://meanjs.org/) par défaut.

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
    git clone https://github.com/prashanthmadi/mean
    ```

## <a name="run-the-application"></a>Exécution de l'application

Cette application MongoDB écrite en Node.js se connecte à votre base de données Azure Cosmos DB, qui prend en charge le client MongoDB. En d’autres termes, l’application sait que les données sont stockées dans une base de données Azure Cosmos DB.

Installez les packages requis et démarrez l’application.

```bash
cd mean
npm install
npm start
```
L’application tente de se connecter à une source MongoDB et échoue. Continuez et quittez l’application lorsque la sortie renvoie le message « [MongoError: connect ECONNREFUSED 127.0.0.1:27017] ».

## <a name="sign-in-to-azure"></a>Connexion à Azure

Si vous utilisez une interface de ligne de commande Azure installée, connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/reference-index#az-login) et suivez les instructions à l’écran. Vous pouvez ignorer cette étape si vous utilisez Azure Cloud Shell.

```azurecli
az login 
``` 
   
## <a name="add-the-azure-cosmos-db-module"></a>Ajouter le module Azure Cosmos DB

Si vous utilisez une interface de ligne de commande Azure installée, vérifiez si le composant `cosmosdb` est déjà installé en exécutant la commande `az`. Si `cosmosdb` figure dans la liste des commandes de base, passez à la commande suivante. Vous pouvez ignorer cette étape si vous utilisez Azure Cloud Shell.

Si `cosmosdb` n’est pas dans la liste des commandes de base, réinstallez [Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources](../azure-resource-manager/management/overview.md) avec la commande [az group create](/cli/azure/group#az-group-create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure comme les applications web, les bases de données et les comptes de stockage sont déployées et gérées. 

L’exemple suivant crée un groupe de ressources dans la région Europe Ouest. Choisissez un nom unique pour le groupe de ressources.

Si vous utilisez Azure Cloud Shell, sélectionnez **Essayer**, suivez les invites à l’écran pour vous connecter, puis copiez la commande dans l’invite de commandes.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

## <a name="create-an-azure-cosmos-db-account"></a>Création d’un compte Azure Cosmos DB

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

## <a name="connect-your-nodejs-application-to-the-database"></a>Connecter votre application Node.js à la base de données

Pendant cette étape, vous connectez votre exemple d’application MEAN.js au compte de base de données Azure Cosmos DB que vous venez de créer. 

<a name="devconfig"></a>
## <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configurer la chaîne de connexion dans votre application Node.js

Dans votre référentiel MEAN.js, ouvrez `config/env/local-development.js`.

Remplacez le contenu de ce fichier par le code suivant. Veillez également à remplacer les deux espaces réservés `<cosmosdb-name>` par le nom de votre compte Cosmos.

```javascript
'use strict';

module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean-dev?ssl=true&sslverifycertificate=false'
  }
};
```

## <a name="retrieve-the-key"></a>Récupérer la clé

Pour vous connecter à une base de données Cosmos, vous avez besoin de la clé de la base de données. Utilisez la commande [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az-cosmosdb-keys-list) pour récupérer la clé primaire.

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

Azure CLI génère des informations semblables à ce qui suit. 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

Copiez la valeur de `primaryMasterKey`. Collez celle-ci sur `<primary_master_key>` dans `local-development.js`.

Enregistrez vos modifications.

### <a name="run-the-application-again"></a>Exécutez de nouveau l'application.

Exécutez de nouveau `npm start`. 

```bash
npm start
```

Un message de console doit maintenant vous indiquer que l’environnement de développement est fonctionnel et en cours d’exécution. 

Dans un navigateur, accédez à `http://localhost:3000`. Sélectionnez **S’inscrire** dans le menu supérieur et essayez de créer deux utilisateurs fictifs. 

L’exemple d’application MEAN.js stocke les données utilisateur dans la base de données. Si vous réussissez et si MEAN.js se connecte automatiquement à l’utilisateur créé, cela signifie que votre connexion à Azure Cosmos DB fonctionne. 

:::image type="content" source="./media/create-mongodb-nodejs/mongodb-connect-success.png" alt-text="MEAN.js se connecte correctement à MongoDB":::

## <a name="view-data-in-data-explorer"></a>Afficher les données dans l’Explorateur de données

Les données stockées dans une base de données Cosmos peuvent être affichées et interrogées dans le Portail Azure.

Pour afficher, interroger et manipuler les données utilisateur créées à l’étape précédente, connectez-vous au [portail Azure](https://portal.azure.com) dans votre navigateur web.

Dans la zone de recherche en haut de la page, entrez **Azure Cosmos DB**. Lorsque le panneau de votre compte Cosmos s’ouvre, sélectionnez votre compte Cosmos. Dans le volet de navigation gauche, sélectionnez **Explorateur de données**. Développez votre collection dans le volet Collections, pour pouvoir afficher les documents de la collection, interroger les données et même créer et exécuter des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur. 

:::image type="content" source="./media/create-mongodb-nodejs/cosmosdb-connect-mongodb-data-explorer.png" alt-text="Explorateur de données sur le portail Azure":::


## <a name="deploy-the-nodejs-application-to-azure"></a>Déployer l’application Node.js dans Azure

Dans cette étape, vous allez déployer votre application Node.js dans Cosmos DB.

Vous avez peut-être remarqué que le fichier de configuration que vous avez modifié précédemment est destiné à l’environnement de développement (`/config/env/local-development.js`). Lorsque vous déployez votre application dans App Service, elle s’exécute dans l’environnement de production par défaut. Maintenant, vous devez apporter la même modification au fichier de configuration correspondant.

Dans votre référentiel MEAN.js, ouvrez `config/env/production.js`.

Dans l’objet `db`, remplacez la valeur de `uri` comme indiqué dans l’exemple suivant. Veillez à remplacer les espaces réservés comme précédemment.

```javascript
'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean?ssl=true&sslverifycertificate=false',
```

> [!NOTE] 
> L’option `ssl=true` est importante, en raison des exigences de Cosmos DB. Pour plus d’informations, consultez [Exigences relatives à la chaîne de connexion](connect-mongodb-account.md#connection-string-requirements).
>
>

Dans le terminal, validez toutes vos modifications dans Git. Vous pouvez copier les deux commandes pour les exécuter ensemble.

```bash
git add .
git commit -m "configured MongoDB connection string"
```
## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer un compte d’API Azure Cosmos DB MongoDB à l’aide d’Azure Cloud Shell et à créer et exécuter une application MEAN.js pour ajouter des utilisateurs au compte. Vous pouvez maintenant importer des données supplémentaires dans votre compte Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
