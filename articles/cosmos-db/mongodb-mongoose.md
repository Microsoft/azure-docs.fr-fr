---
title: Connecter une application Mongoose Node.js à Azure Cosmos DB
description: Découvrez comment utiliser l’infrastructure Mongoose pour stocker et gérer des données dans Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 03/20/2020
author: timsander1
ms.author: tisande
ms.custom: seodec18, devx-track-js
ms.openlocfilehash: 8958699ae279d2613f8dbadca802ee2137407e75
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442393"
---
# <a name="connect-a-nodejs-mongoose-application-to-azure-cosmos-db"></a>Connecter une application Mongoose Node.js à Azure Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Ce tutoriel montre comment utiliser le [framework Mongoose](https://mongoosejs.com/) lors du stockage de données dans Cosmos DB. Dans cette procédure pas à pas, nous utilisons l’API Azure Cosmos DB pour MongoDB. Pour ceux d’entre vous qui ne connaissent pas Mongoose, il s’agit d’une infrastructure de modélisation d’objet pour MongoDB dans Node.js qui offre une solution simple basée sur un schéma pour modéliser vos données d’application.

Cosmos DB est le service de base de données multi-modèle de Microsoft, distribué à l’échelle mondiale. Vous avez la possibilité de créer et d’interroger rapidement des documents, des paires clé/valeur et des bases de données de graphe, profitant tous de la distribution à l’échelle mondiale et des capacités de mise à l’échelle horizontale au cœur de Cosmos DB.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

[Node.js](https://nodejs.org/) version v0.10.29 ou supérieure.

## <a name="create-a-cosmos-account"></a>Créer un compte Cosmos

Passons à la création d’un compte Cosmos. Si vous avez déjà un compte que vous souhaitez utiliser, vous pouvez passer directement à l’étape Configurer votre application Node.js. Si vous utilisez l’émulateur Azure Cosmos DB, suivez les étapes de la section [Émulateur Azure Cosmos DB](local-emulator.md) pour le configurer, puis passez directement à l’étape Configurer votre application Node.js.

[!INCLUDE [cosmos-db-create-dbaccount-mongodb](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

### <a name="create-a-database"></a>Création d'une base de données 
Dans cette application, nous allons aborder deux façons de créer des collections dans Azure Cosmos DB : 
- **Stockage de chaque modèle objet dans une collection distincte**  : Nous vous recommandons de [créer d’une base de données avec un débit dédié](set-throughput.md#set-throughput-on-a-database). L’utilisation de ce modèle de capacité vous offre une meilleure rentabilité.

    :::image type="content" source="./media/mongodb-mongoose/db-level-throughput.png" alt-text="Didacticiel Node.js : capture d’écran du Portail Azure, montrant comment créer une base de données dans Azure Data Explorer pour un compte Azure Cosmos DB, à utiliser avec le module Node Mongoose":::

- **Stockage de tous les modèles objet dans une collection Cosmos DB unique**  : Si vous préférez stocker tous les modèles dans une seule collection, vous pouvez simplement créer une base de données sans sélectionner l’option Approvisionner le débit. L’utilisation de ce modèle de capacité crée chaque collection avec sa propre capacité de débit pour chaque modèle objet.

Après avoir créé la base de données, vous utiliserez son nom dans la variable d’environnement `COSMOSDB_DBNAME` ci-dessous.

## <a name="set-up-your-nodejs-application"></a>Configurer votre application Node.js

>[!Note]
> Si vous souhaitez simplement parcourir l’exemple de code sans configurer l’application elle-même, clonez l’[exemple](https://github.com/Azure-Samples/Mongoose_CosmosDB) utilisé pour ce didacticiel et créez votre application Mongoose Node.js dans Azure Cosmos DB.

1. Pour créer une application Node.js dans le dossier de votre choix, exécutez la commande suivante dans une invite de commandes Node.

    ```npm init```

    Répondez aux questions et votre projet est prêt à être utilisé.

2. Ajoutez un nouveau fichier dans le dossier et nommez-le ```index.js```.
3. Installez les packages nécessaires à l’aide de l’une des options ```npm install``` :
   * Mongoose : ```npm install mongoose@5 --save```

     > [!Note]
     > La connexion d’exemple Mongoose ci-dessous repose sur Mongoose 5+, ce qui a changé depuis les versions antérieures.
    
   * Dotenv (si vous voulez charger vos secrets à partir d’un fichier .env) : ```npm install dotenv --save```

     >[!Note]
     > L’indicateur ```--save``` ajoute la dépendance au fichier package.json.

4. Importez les dépendances dans votre fichier index.js.

    ```JavaScript
   var mongoose = require('mongoose');
   var env = require('dotenv').config();   //Use the .env file to load the variables
    ```

5. Ajoutez la chaîne de connexion Cosmos DB et le nom Cosmos DB au fichier ```.env```. Remplacez les espaces réservés {cosmos-account-name} et {dbname} par le nom de votre compte Cosmos et le nom de la base de données, sans les symboles de l’accolade.

    ```JavaScript
   # You can get the following connection details from the Azure portal. You can find the details on the Connection string pane of your Azure Cosmos account.

   COSMOSDB_USER = "<Azure Cosmos account's user name, usually the database account name>"
   COSMOSDB_PASSWORD = "<Azure Cosmos account password, this is one of the keys specified in your account>"
   COSMOSDB_DBNAME = "<Azure Cosmos database name>"
   COSMOSDB_HOST= "<Azure Cosmos Host name>"
   COSMOSDB_PORT=10255
    ```

6. Connectez-vous à Cosmos DB à l’aide du framework Mongoose en ajoutant le code suivant à la fin du fichier index.js.
    ```JavaScript
   mongoose.connect("mongodb://"+process.env.COSMOSDB_HOST+":"+process.env.COSMOSDB_PORT+"/"+process.env.COSMOSDB_DBNAME+"?ssl=true&replicaSet=globaldb", {
      auth: {
        user: process.env.COSMOSDB_USER,
        password: process.env.COSMOSDB_PASSWORD
      },
    useNewUrlParser: true,
    useUnifiedTopology: true,
    retryWrites: false
    })
    .then(() => console.log('Connection to CosmosDB successful'))
    .catch((err) => console.error(err));
    ```
    >[!Note]
    > Ici, les variables d’environnement sont chargées à l’aide de process.env.{nomVariable} en utilisant le package npm « dotenv ».

    Une fois connecté à Azure Cosmos DB, vous pouvez commencer à configurer des modèles d’objet dans Mongoose.

## <a name="best-practices-for-using-mongoose-with-cosmos-db"></a>Meilleures pratiques pour l’utilisation de Mongoose avec Cosmos DB

Pour chaque modèle que vous créez, Mongoose crée une collection. Le meilleur moyen est d’utiliser l’option [Débit au niveau de la base de données](set-throughput.md#set-throughput-on-a-database), dont il a été question précédemment. Pour utiliser une collection unique, vous devez utiliser des [discriminateurs](https://mongoosejs.com/docs/discriminators.html) Mongoose. Il s’agit d’un mécanisme d’héritage de schéma. Les discriminateurs vous permettent d’avoir plusieurs modèles avec des schémas superposés sur la même collection MongoDB sous-jacente.

Vous pouvez stocker les différents modèles de données dans la même collection, puis utiliser une clause de filtre au moment de la requête pour extraire uniquement les données nécessaires. Examinons chacun de ces modèles.

### <a name="one-collection-per-object-model"></a>Une collection par modèle d’objet

Cette section explique comment y parvenir avec l’API Azure Cosmos DB pour MongoDB. Cette méthode est recommandée, car elle vous permet de maîtriser les coûts et la capacité. Par conséquent, la quantité d’unités de requête sur la base de données ne dépend pas du nombre de modèles objet. Il s’agit du modèle de fonctionnement par défaut pour Mongoose, vous devez donc déjà le connaître.

1. Ouvrez de nouveau votre ```index.js```.

1. Créez la définition de schéma pour « Family ».

    ```JavaScript
    const Family = mongoose.model('Family', new mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
            gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }));
    ```

1. Créez un objet pour « Family ».

    ```JavaScript
    const family = new Family({
        lastName: "Volum",
        parents: [
            { firstName: "Thomas" },
            { firstName: "Mary Kay" }
        ],
        children: [
            { firstName: "Ryan", gender: "male", grade: 8 },
            { firstName: "Patrick", gender: "male", grade: 7 }
        ],
        pets: [
            { givenName: "Buddy" }
        ],
        address: { country: "USA", state: "WA", city: "Seattle" }
    });
    ```

1. Pour finir, nous allons enregistrer l’objet dans Cosmos DB. Cette opération crée une collection en arrière-plan.

    ```JavaScript
    family.save((err, saveFamily) => {
        console.log(JSON.stringify(saveFamily));
    });
    ```

1. Nous allons ensuite créer un autre schéma et un objet. Nous allons le consacrer aux destinations de vacances (« Vacation Destinations ») susceptibles d’intéresser les familles.
   1. Comme la dernière fois, nous allons créer le schéma.
      ```JavaScript
      const VacationDestinations = mongoose.model('VacationDestinations', new mongoose.Schema({
       name: String,
       country: String
      }));
      ```

   1. Créez un exemple d’objet (vous pouvez en ajouter plusieurs à ce schéma) et enregistrez-le.
      ```JavaScript
      const vacaySpot = new VacationDestinations({
       name: "Honolulu",
       country: "USA"
      });

      vacaySpot.save((err, saveVacay) => {
       console.log(JSON.stringify(saveVacay));
      });
      ```

1. Dans le portail Azure, vous remarquez que deux collections ont été créées dans Cosmos DB.

   :::image type="content" source="./media/mongodb-mongoose/mongo-mutliple-collections.png" alt-text="Tutoriel Node.js – Capture d’écran du portail Azure montrant un compte Azure Cosmos DB avec plusieurs noms de collections en surbrillance – Base de données de nœud":::

1. Pour finir, nous allons lire les données à partir de Cosmos DB. Étant donné que nous utilisons le modèle de fonctionnement par défaut de Mongoose, la lecture est identique à n’importe quelle autre lecture avec Mongoose.

    ```JavaScript
    Family.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family: " + JSON.stringify(fam)));
    });
    ```

### <a name="using-mongoose-discriminators-to-store-data-in-a-single-collection"></a>Utiliser les discriminateurs Mongoose pour stocker des données dans une seule collection

Dans cette méthode, nous utilisons les [discriminateurs Mongoose](https://mongoosejs.com/docs/discriminators.html) pour optimiser les coûts de chaque collection. Les discriminateurs vous permettent de définir une « clé » de différenciation destinée à stocker, à différencier et à filtrer les différents modèles d’objet.

Nous créons ici un modèle d’objet de base, définissons une clé de différenciation et ajoutons « Family » et « VacationDestinations » comme extension au modèle de base.

1. Commençons par définir la configuration de base et la clé du discriminateur.

    ```JavaScript
    const baseConfig = {
        discriminatorKey: "_type", //If you've got a lot of different data types, you could also consider setting up a secondary index here.
        collection: "alldata"   //Name of the Common Collection
    };
    ```

1. Nous allons ensuite définir le modèle d’objet commun

    ```JavaScript
    const commonModel = mongoose.model('Common', new mongoose.Schema({}, baseConfig));
    ```

1. Nous définissons maintenant le modèle « Family ». Notez que nous utilisons ```commonModel.discriminator``` au lieu de ```mongoose.model```. Nous ajoutons également la configuration de base au schéma Mongoose. L’élément discriminatorKey est donc ```FamilyType```.

    ```JavaScript
    const Family_common = commonModel.discriminator('FamilyType', new     mongoose.Schema({
        lastName: String,
        parents: [{
            familyName: String,
            firstName: String,
            gender: String
        }],
        children: [{
            familyName: String,
            firstName: String,
           gender: String,
            grade: Number
        }],
        pets:[{
            givenName: String
        }],
        address: {
            country: String,
            state: String,
            city: String
        }
    }, baseConfig));
    ```

1. De la même manière, nous allons ajouter un autre schéma, cette fois-ci pour « VacationDestinations ». Ici, l’élément discriminatorKey est ```VacationDestinationsType```.

    ```JavaScript
    const Vacation_common = commonModel.discriminator('VacationDestinationsType', new mongoose.Schema({
        name: String,
        country: String
    }, baseConfig));
    ```

1. Pour finir, nous allons créer des objets pour le modèle, puis l’enregistrer.
   1. Commençons par ajouter un ou plusieurs objets au modèle « Family ».
      ```JavaScript
      const family_common = new Family_common({
       lastName: "Volum",
       parents: [
           { firstName: "Thomas" },
           { firstName: "Mary Kay" }
       ],
       children: [
           { firstName: "Ryan", gender: "male", grade: 8 },
           { firstName: "Patrick", gender: "male", grade: 7 }
       ],
       pets: [
           { givenName: "Buddy" }
       ],
       address: { country: "USA", state: "WA", city: "Seattle" }
      });

      family_common.save((err, saveFamily) => {
       console.log("Saved: " + JSON.stringify(saveFamily));
      });
      ```

   1. Nous ajoutons ensuite un ou plusieurs objets au modèle « VacationDestinations », avant de l’enregistrer.
      ```JavaScript
      const vacay_common = new Vacation_common({
       name: "Honolulu",
       country: "USA"
      });

      vacay_common.save((err, saveVacay) => {
       console.log("Saved: " + JSON.stringify(saveVacay));
      });
      ```

1. De retour dans le portail Azure, vous remarquez qu’il n’existe qu’une seule collection appelée ```alldata``` contenant les données de « Family » et de « VacationDestinations ».

   :::image type="content" source="./media/mongodb-mongoose/mongo-collections-alldata.png" alt-text="Tutoriel Node.js – Capture d’écran du portail Azure montrant un compte Azure Cosmos DB avec le nom de la collection en surbrillance – Base de données de nœud":::

1. Vous remarquez également que chaque objet possède un autre attribut nommé ```__type```, qui vous permet de différencier les deux modèles d’objet.

1. Pour finir, nous allons lire les données stockées dans Azure Cosmos DB. Mongoose s’occupe de filtrer les données en fonction du modèle. Vous n’avez donc rien d’autre à faire lors de la lecture des données. Il vous suffit de spécifier votre modèle (dans cet exemple, ```Family_common```) et Mongoose se charge d’appliquer un filtrage en fonction de l’élément « DiscriminatorKey ».

    ```JavaScript
    Family_common.find({ 'children.gender' : "male"}, function(err, foundFamily){
        foundFamily.forEach(fam => console.log("Found Family (using discriminator): " + JSON.stringify(fam)));
    });
    ```

Comme vous pouvez le constater, il est très simple d’utiliser des discriminateurs Mongoose. Si votre application utilise le framework Mongoose, ce didacticiel vous permet de préparer votre application à l’aide de l’API Azure Cosmos DB pour MongoDB sans nécessiter trop de modifications.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser Studio 3T](mongodb-mongochef.md) avec l’API Azure Cosmos DB pour MongoDB.
- Découvrez comment [utiliser Robo 3T](mongodb-robomongo.md) avec l’API Azure Cosmos DB pour MongoDB.
- Explorez les [exemples](mongodb-samples.md) MongoDB avec l’API Azure Cosmos DB pour MongoDB.

[dbleveltp]: ./media/mongodb-mongoose/db-level-throughput.png
