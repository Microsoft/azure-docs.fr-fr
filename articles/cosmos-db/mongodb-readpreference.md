---
title: Utiliser la préférence de lecture MongoDB avec l’API Azure Cosmos DB pour MongoDB
description: Découvrir comment utiliser la préférence de lecture MongoDB avec l’API Azure Cosmos DB pour MongoDB
author: sivethe
ms.author: sivethe
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 02/26/2019
ms.custom: devx-track-js
ms.openlocfilehash: 3c78ad6605e927015d35df12cadf0347dd0337cf
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349042"
---
# <a name="how-to-globally-distribute-reads-using-azure-cosmos-dbs-api-for-mongodb"></a>Comment distribuer à l’échelle mondiale des lectures par l’intermédiaire de l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Cet article explique comment distribuer à l’échelle mondiale des opérations de lecture à l’aide des paramètres de [préférence de lecture MongoDB](https://docs.mongodb.com/manual/core/read-preference/) avec l’API Azure Cosmos DB pour MongoDB.

## <a name="prerequisites"></a>Prérequis 
Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 
[!INCLUDE [cosmos-db-emulator-mongodb](../../includes/cosmos-db-emulator-mongodb.md)]

Consultez ce [Guide de démarrage rapide](tutorial-global-distribution-mongodb.md) afin d’obtenir des instructions sur l’utilisation du portail Azure pour configurer un compte Cosmos avec la distribution mondiale et s’y connecter.

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Ouvrez une fenêtre de terminal git, comme git bash, et accédez à un répertoire de travail à l’aide de la commande `cd`.  

Exécutez les commandes suivantes pour cloner l’exemple de référentiel. Selon la plateforme qui vous intéresse, utilisez l’un des dépôts suivants :

1. [Exemple d’application .NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference)
2. [Exemple d’application NodeJS]( https://github.com/Azure-Samples/azure-cosmos-db-mongodb-node-geo-readpreference)
3. [Exemple d’application Mongoose](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-mongoose-geo-readpreference)
4. [Exemple d’application Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)
5. [Exemple d’application SpringBoot](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-spring)


```bash
git clone <sample repo url>
```

## <a name="run-the-application"></a>Exécution de l'application

En fonction de la plateforme utilisée, installez les packages nécessaires et démarrez l’application. Pour installer les dépendances, suivez les instructions du fichier LISEZMOI inclus dans le dépôt d’exemple d’application. Par exemple, dans l’exemple d’application NodeJS, utilisez les commandes suivantes pour installer les packages nécessaires et démarrer l’application.

```bash
cd mean
npm install
node index.js
```
L’application tente de se connecter à une source MongoDB, et échoue car la chaîne de connexion n’est pas valide. Suivez les étapes décrites dans le fichier LISEZMOI pour mettre à jour la chaîne de connexion `url`. Mettez aussi à jour `readFromRegion` avec une région de lecture dans votre compte Cosmos. Les instructions ci-dessous sont tirées de l’exemple NodeJS :

```
* Next, substitute the `url`, `readFromRegion` in App.Config with your Cosmos account's values. 
```

Une fois ces étapes terminées, l’exemple d’application s’exécute et génère la sortie suivante :

```
connected!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromSecondaryfunc query completed!
```

## <a name="read-using-read-preference-mode"></a>Lire à l’aide du mode de préférence de lecture

Le protocole MongoDB fournit les modes de préférence de lecture suivants utilisables par les clients :

1. PRIMARY
2. PRIMARY_PREFERRED
3. SECONDARY
4. SECONDARY_PREFERRED
5. NEAREST

Pour plus d’informations sur le comportement de chacun de ces modes de préférence de lecture, consultez la documentation détaillée sur le [comportement de préférence de lecture MongoDB](https://docs.mongodb.com/manual/core/read-preference-mechanics/#replica-set-read-preference-behavior). Dans Cosmos DB, primary mappe à la région d’écriture et secondary mappe à la région de lecture.

Sur la base des scénarios courants, nous recommandons d’utiliser les paramètres suivants :

1. Si des **lectures à faible latence** sont nécessaires, utilisez le mode de préférence de lecture **NEAREST**. Ce paramètre dirige les opérations de lecture vers la région disponible la plus proche. Notez que si la région la plus proche est la région d’écriture, ces opérations sont dirigées vers cette région.
2. Si une **haute disponibilité et une géoréplication des lectures** sont nécessaires (la latence n’est pas une contrainte), utilisez le mode de préférence de lecture **PRIMARY PREFERRED** ou **SECONDARY PREFERRED**. Ce paramètre dirige les opérations de lecture vers une région de lecture ou d’écriture disponible. Si la région n’est pas disponible, les demandes sont dirigées vers la région disponible suivante, conformément au comportement de la préférence de lecture.

L’extrait de code suivant tiré de l’exemple d’application montre comment configurer la préférence de lecture NEAREST dans NodeJS :

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.NEAREST});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromNearestfunc query completed!");
  });
```

De même, l’extrait de code ci-dessous montre comment configurer la préférence de lecture SECONDARY_PREFERRED dans NodeJS :

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.SECONDARY_PREFERRED});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromSecondaryPreferredfunc query completed!");
  });
```

La préférence de lecture peut également être définie en passant `readPreference` comme paramètre dans les options d’URI de chaîne de connexion :

```javascript
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

// Connection URL
const url = 'mongodb://localhost:27017?ssl=true&replicaSet=globaldb&readPreference=nearest';

// Database Name
const dbName = 'myproject';

// Use connect method to connect to the Server
MongoClient.connect(url, function(err, client) {
  console.log("Connected correctly to server");

  const db = client.db(dbName);

  client.close();
});
```

Pour d’autres plateformes telles que [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) et [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference), consultez les dépôts des exemples d’application correspondants.

## <a name="read-using-tags"></a>Lire à l’aide de balises

Outre le mode de préférence de lecture, le protocole MongoDB autorise l’utilisation de balises pour diriger les opérations de lecture. Dans l’API Azure Cosmos DB pour MongoDB, la balise `region` est incluse par défaut dans la réponse `isMaster` :

```json
"tags": {
         "region": "West US"
      }
```

Ainsi, MongoClient peut utiliser la balise `region` en plus du nom de la région pour diriger les opérations de lecture vers des régions spécifiques. Pour les comptes Cosmos, les noms des régions figurent dans le portail Azure, à gauche sous **Paramètres -> Répliquer les données globalement**. Ce paramètre est utile pour bénéficier d’une **isolation de lecture**, cas dans lesquels une application cliente souhaite diriger les opérations de lecture uniquement vers une région spécifique. Ce paramètre est idéal pour les scénarios de type analytique/non-production, qui s’exécutent en arrière-plan et ne sont pas des services essentiels pour la production.

L’extrait de code suivant tiré de l’exemple d’application montre comment configurer la préférence de lecture avec des balises dans NodeJS :

```javascript
 var query = {};
  var readcoll = client.db('regionDB').collection('regionTest',{readPreference: new ReadPreference(ReadPreference.SECONDARY_PREFERRED, {"region": "West US"})});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromRegionfunc query completed!");
  });
```

Pour d’autres plateformes telles que [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) et [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference), consultez les dépôts des exemples d’application correspondants.

Dans cet article, vous avez découvert comment distribuer à l’échelle mondiale des opérations de lecture à l’aide de la préférence de lecture avec l’API Azure Cosmos DB pour MongoDB.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne prévoyez pas de continuer à utiliser cette application, supprimez toutes les ressources créées par cet article dans le portail Azure en effectuant les étapes suivantes :

1. Dans le menu de gauche du portail Azure, cliquez sur **Groupes de ressources**, puis sur le nom de la ressource que vous avez créée. 
2. Sur la page de votre groupe de ressources, cliquez sur **Supprimer**, tapez le nom de la ressource à supprimer dans la zone de texte, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

* [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
* [Configurer une base de données distribuée à l’échelle mondiale avec l’API Azure Cosmos DB pour MongoDB](tutorial-global-distribution-mongodb.md)
* [Développer localement avec l’émulateur Azure Cosmos DB](local-emulator.md)