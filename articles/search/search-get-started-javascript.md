---
title: 'Démarrage rapide : Créer un index de recherche en JavaScript'
titleSuffix: Azure Cognitive Search
description: Dans ce guide de démarrage rapide JavaScript, découvrez comment créer un index, charger des données et exécuter des requêtes sur Recherche cognitive Azure en utilisant JavaScript.
author: dereklegenzoff
manager: luisca
ms.author: delegenz
ms.devlang: javascript
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 10/26/2020
ms.custom: devx-track-js
ms.openlocfilehash: 69cef4c17fd17219f0221c509775ad3fd6a80a27
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498980"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-using-the-javascript-sdk"></a>Démarrage rapide : Créer un index Recherche cognitive Azure en utilisant le SDK JavaScript
> [!div class="op_single_selector"]
> * [JavaScript](search-get-started-javascript.md)
> * [C#](search-get-started-dotnet.md)
> * [Portail](search-get-started-portal.md)
> * [PowerShell](./search-get-started-powershell.md)
> * [Python](search-get-started-python.md)
> * [REST](search-get-started-rest.md)


Utilisez le [SDK JavaScript/TypeScript pour Recherche cognitive Azure](/javascript/api/overview/azure/search-documents-readme) pour créer une application Node.js en JavaScript qui crée, charge et interroge un index de recherche.

Cet article explique comment créer l’application pas à pas. Vous pouvez aussi [télécharger le code source et les données](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11), puis exécuter l’application à partir de la ligne de commande.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez disposer des outils et services suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

+ Service Recherche cognitive Azure. [Créez un service](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Vous pouvez utiliser un service gratuit pour ce guide de démarrage rapide. 

+ [Node.js](https://nodejs.org) et [NPM](https://www.npmjs.com)

+ [Visual Studio Code](https://code.visualstudio.com) ou un autre IDE


## <a name="set-up-your-project"></a>Configuration de votre projet

Commencez par obtenir le point de terminaison et la clé pour votre service de recherche. Créez ensuite un projet avec NPM comme indiqué ci-dessous.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Copier une clé et un point de terminaison

Les appels au service nécessitent un point de terminaison d’URL et une clé d’accès pour chaque requête. Dans un premier temps, recherchez la clé et l’URL de l’API à ajouter à votre projet. Vous allez spécifier les deux valeurs lors de la création du client dans une étape ultérieure.

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

2. Dans **Paramètres** > **Clés**, procurez-vous une clé d’administration pour disposer de droits complets sur le service, indispensables pour créer ou supprimer des objets. Il existe deux clés, primaire et secondaire, interchangeables. Vous pouvez utiliser celle de votre choix.

   ![Obtenir un point de terminaison et une clé d’accès HTTP](media/search-get-started-rest/get-url-key.png "Obtenir un point de terminaison et une clé d’accès HTTP")

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

### <a name="create-a-new-npm-project"></a>Créer un projet NPM

Commencez par ouvrir VS Code et son [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal) ou un autre terminal, comme l’invite de commandes Node.js.

1. Créez un répertoire de développement, que vous appellerez `quickstart` :

    ```cmd
    mkdir quickstart
    cd quickstart
    ```

2. Initialisez un projet vide avec NPM en exécutant 

    ```cmd
    npm init
    ```
     Acceptez les valeurs par défaut, sauf pour la licence, que vous devez définir sur « MIT ». 

3. Installez `@azure/search-documents`, le [SDK JavaScript/TypeScript pour Recherche cognitive Azure](/javascript/api/overview/azure/search-documents-readme).

    ```cmd
    npm install @azure/search-documents
    ```

4. Installez `dotenv`, qui est utilisé pour importer les variables d’environnement, comme le nom de notre service et notre clé d’API.
    ```cmd
    npm install dotenv
    ```

5. Vérifiez que vous avez configuré les projets et leurs dépendances en regardant si votre fichier **package.json** se présente comme le JSON suivant :

    ```json
    {
      "name": "quickstart",
      "version": "1.0.0",
      "description": "Azure Cognitive Search Quickstart",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [
        "Azure",
        "Search"
      ],
      "author": "Your Name",
      "license": "MIT",
      "dependencies": {
        "@azure/search-documents": "^11.0.3",
        "dotenv": "^8.2.0"
      }
    }
    ```

6. Créez un fichier **.env** pour stocker les paramètres de votre service de recherche :

    ```
    SEARCH_API_KEY=<search-admin-key>
    SEARCH_API_ENDPOINT=https://<search-service-name>.search.windows.net
    ```

Remplacez la valeur `<search-service-name>` par le nom de votre service de recherche. Remplacez `<search-admin-key>` par la valeurs de clé que vous avez enregistrée précédemment. 

### <a name="create-indexjs-file"></a>Créer un fichier index.js

Nous créons ensuite un fichier **index.js**, qui est le fichier principal qui va héberger notre code.

Dans le haut de ce fichier, nous importons la bibliothèque `@azure/search-documents` :

```javascript
const { SearchIndexClient, SearchClient, AzureKeyCredential, odata } = require("@azure/search-documents");
```

Ensuite, nous devons indiquer au package `dotenv` de lire les paramètres du fichier **.env** comme suit :

```javascript
// Load the .env file if it exists
require("dotenv").config();

// Getting endpoint and apiKey from .env file
const endpoint = process.env.SEARCH_API_ENDPOINT || "";
const apiKey = process.env.SEARCH_API_KEY || "";
```

Avec nos importations et nos variables d’environnement en place, nous sommes prêts à définir la fonction main.

La plupart des fonctionnalités du SDK sont asynchrones, c’est pourquoi notre fonction main sera `async`. Nous incluons également un `main().catch()` sous la fonction principale pour intercepter et journaliser les erreurs rencontrées :

```javascript
async function main() {
    console.log(`Running Azure Cognitive Search Javascript quickstart...`);
    if (!endpoint || !apiKey) {
        console.log("Make sure to set valid values for endpoint and apiKey with proper authorization.");
        return;
    }

    // remaining quickstart code will go here
}

main().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

Une fois cela en place, nous sommes prêts à créer un index.

## <a name="1---create-index"></a>1 - Créer un index 

Créez un fichier **hotels_quickstart_index.json**.  Ce fichier définit le fonctionnement de la Recherche cognitive Azure avec les documents que vous allez charger à l’étape suivante. Chaque champ est identifié par un `name` et a un `type` spécifié. Chaque champ contient également une série d’attributs d’index qui spécifient si la Recherche cognitive Azure peut effectuer des opérations de recherche, de tri, de filtrage et de définition de facettes sur le champ. La plupart des champs sont des types de données simples, mais certains, comme `AddressType`, sont des types complexes qui vous permettent de créer des structures de données riches dans votre index.  Vous pouvez en savoir plus sur les [types de données pris en charge](/rest/api/searchservice/supported-data-types) et les [attributs d’index](./search-what-is-an-index.md#index-attributes). 

Ajoutez l’élément suivant à **hotels_quickstart_index.json** ou [téléchargez le fichier](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels_quickstart_index.json). 

```json
{
    "name": "hotels-quickstart",
    "fields": [
        {
            "name": "HotelId",
            "type": "Edm.String",
            "key": true,
            "filterable": true
        },
        {
            "name": "HotelName",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": true,
            "facetable": false
        },
        {
            "name": "Description",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "en.lucene"
        },
        {
            "name": "Description_fr",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "fr.lucene"
        },
        {
            "name": "Category",
            "type": "Edm.String",
            "searchable": true,
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Tags",
            "type": "Collection(Edm.String)",
            "searchable": true,
            "filterable": true,
            "sortable": false,
            "facetable": true
        },
        {
            "name": "ParkingIncluded",
            "type": "Edm.Boolean",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "LastRenovationDate",
            "type": "Edm.DateTimeOffset",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Rating",
            "type": "Edm.Double",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Address",
            "type": "Edm.ComplexType",
            "fields": [
                {
                    "name": "StreetAddress",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true
                },
                {
                    "name": "City",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "StateProvince",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "PostalCode",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "Country",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                }
            ]
        }
    ],
    "suggesters": [
        {
            "name": "sg",
            "searchMode": "analyzingInfixMatching",
            "sourceFields": [
                "HotelName"
            ]
        }
    ]
}
```

Une fois notre définition d’index en place, nous voulons importer **hotels_quickstart_index.json** dans le haut de **index.js** pour que la fonction main puisse accéder à la définition de l’index.

```javascript
const indexDefinition = require('./hotels_quickstart_index.json');
```

Dans la fonction main, nous créons ensuite un `SearchIndexClient`, qui est utilisé pour créer et gérer des index pour Recherche cognitive Azure. 

```javascript
const indexClient = new SearchIndexClient(endpoint, new AzureKeyCredential(apiKey));
```

Ensuite, nous voulons supprimer l’index s’il existe déjà. Il s’agit d’une pratique courante pour le code de test/démonstration.

Pour ce faire, nous définissons une fonction simple qui tente de supprimer l’index.

```javascript
async function deleteIndexIfExists(indexClient, indexName) {
    try {
        await indexClient.deleteIndex(indexName);
        console.log('Deleting index...');
    } catch {
        console.log('Index does not exist yet.');
    }
}
```

Pour exécuter la fonction, nous extrayons le nom de l’index de la définition de l’index et nous passons `indexName` avec `indexClient` à la fonction `deleteIndexIfExists()`.

```javascript
const indexName = indexDefinition["name"];

console.log('Checking if index exists...');
await deleteIndexIfExists(indexClient, indexName);
```

Après cela, nous sommes prêts à créer l’index avec la méthode `createIndex()`.

```javascript
console.log('Creating index...');
let index = await indexClient.createIndex(indexDefinition);

console.log(`Index named ${index.name} has been created.`);
```

### <a name="run-the-sample"></a>Exécution de l'exemple

À ce stade, vous êtes prêt à exécuter l’exemple. Utilisez une fenêtre de terminal pour exécuter la commande suivante :

```cmd
node index.js
```

Si vous avez [téléchargé le code source](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) et que vous n’avez pas encore installé les packages requis, exécutez d’abord `npm install`.

Vous devez voir une série de messages décrivant les actions effectuées par le programme. 

Ouvrez la **Vue d’ensemble** de votre service de recherche dans le Portail Azure. Sélectionnez l’onglet **Index**. Un résultat tel que celui-ci doit s’afficher :

:::image type="content" source="media/search-get-started-javascript/create-index-no-data.png" alt-text="Capture d’écran du portail Azure, de la vue d’ensemble du service Recherche, de l’onglet Index" border="false":::

À l’étape suivante, vous ajouterez des données à l’index. 

## <a name="2---load-documents"></a>2 – Charger des documents 


Dans la Recherche cognitive Azure, les documents sont des structures de données qui sont à la fois des entrées pour l’indexation et des sorties de requêtes. Vous pouvez envoyer des données de ce type dans l’index ou utiliser un [indexeur](search-indexer-overview.md). Dans le cas présent, nous envoyons les documents à l’index programmatiquement.

Les entrées de documents peuvent être des lignes dans une base de données, des objets blob dans le Stockage Blob ou, comme dans cet exemple, des documents JSON sur le disque. Vous pouvez télécharger [hotels.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels.json) ou créer votre propre fichier **hotels.json** avec le contenu suivant :

```json
{
    "value": [
        {
            "HotelId": "1",
            "HotelName": "Secret Point Motel",
            "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
            "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
            "Category": "Boutique",
            "Tags": ["pool", "air conditioning", "concierge"],
            "ParkingIncluded": false,
            "LastRenovationDate": "1970-01-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "677 5th Ave",
                "City": "New York",
                "StateProvince": "NY",
                "PostalCode": "10022"
            }
        },
        {
            "HotelId": "2",
            "HotelName": "Twin Dome Motel",
            "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Boutique",
            "Tags": ["pool", "free wifi", "concierge"],
            "ParkingIncluded": "false",
            "LastRenovationDate": "1979-02-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "140 University Town Center Dr",
                "City": "Sarasota",
                "StateProvince": "FL",
                "PostalCode": "34243"
            }
        },
        {
            "HotelId": "3",
            "HotelName": "Triple Landscape Hotel",
            "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Resort and Spa",
            "Tags": ["air conditioning", "bar", "continental breakfast"],
            "ParkingIncluded": "true",
            "LastRenovationDate": "2015-09-20T00:00:00Z",
            "Rating": 4.8,
            "Address": {
                "StreetAddress": "3393 Peachtree Rd",
                "City": "Atlanta",
                "StateProvince": "GA",
                "PostalCode": "30326"
            }
        },
        {
            "HotelId": "4",
            "HotelName": "Sublime Cliff Hotel",
            "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
            "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
            "Category": "Boutique",
            "Tags": ["concierge", "view", "24-hour front desk service"],
            "ParkingIncluded": true,
            "LastRenovationDate": "1960-02-06T00:00:00Z",
            "Rating": 4.6,
            "Address": {
                "StreetAddress": "7400 San Pedro Ave",
                "City": "San Antonio",
                "StateProvince": "TX",
                "PostalCode": "78216"
            }
        }
    ]
}
```

À l’instar de ce que nous avons fait avec indexDefinition, nous devons aussi importer `hotels.json` dans le haut de **index.js** pour que les données soient accessibles dans notre fonction main.

```javascript
const hotelData = require('./hotels.json');
```


Pour indexer les données dans l’index de recherche, nous devons maintenant créer un `SearchClient`. Alors que le `SearchIndexClient` est utilisé pour créer et gérer un index, le `SearchClient` est utilisé pour charger des documents et interroger l’index.

Il existe deux façons de créer un objet `SearchClient`. La première option consiste à créer un `SearchClient` à partir de zéro :

```javascript
 const searchClient = new SearchClient(endpoint, indexName, new AzureKeyCredential(apiKey));
```

Vous pouvez également utiliser la méthode `getSearchClient()` du `SearchIndexClient` pour créer le `SearchClient` :

```javascript
const searchClient = indexClient.getSearchClient(indexName);
```

Maintenant que le client est défini, chargez les documents dans l’index de recherche. Dans le cas présent, nous utilisons la méthode `mergeOrUploadDocuments()`, qui charge les documents ou les fusionne avec un document existant si un document ayant la même clé existe déjà.

```javascript
console.log('Uploading documents...');
let indexDocumentsResult = await searchClient.mergeOrUploadDocuments(hotelData['value']);

console.log(`Index operations succeeded: ${JSON.stringify(indexDocumentsResult.results[0].succeeded)}`);
```

Exécutez à nouveau le programme avec `node index.js`. Vous devriez voir un ensemble de messages légèrement différents de ceux que vous avez vus à l’étape 1. Cette fois, l’index *existe* et vous voyez normalement un message à propos de sa suppression avant que l’application ne crée l’index et n’y envoie des données. 

Avant d’exécuter les requêtes à l’étape suivante, définissez une fonction pour que le programme attende une seconde. Faites-le seulement à des fins de test ou de démonstration pour garantir que l’indexation se termine et que les documents sont disponibles dans l’index pour nos requêtes.

```javascript
function sleep(ms) {
    var d = new Date();
    var d2 = null;
    do {
        d2 = new Date();
    } while (d2 - d < ms);
}
```

Pour que le programme attende une seconde, appelez la fonction `sleep` comme ci-dessous :

```javascript
sleep(1000);
```

## <a name="3---search-an-index"></a>3 – Rechercher dans un index

Avec un index créé et des documents chargés, vous êtes prêt à envoyer des requêtes à l’index. Dans cette section, nous allons envoyer cinq requêtes différentes à l’index de recherche afin d’illustrer les différentes fonctionnalités de requête disponibles.

Les requêtes sont écrites dans une fonction `sendQueries()` que nous allons appeler dans la fonction main comme suit :

```javascript
await sendQueries(searchClient);
```

Les requêtes sont envoyées en utilisant la méthode `search()` de `searchClient`. Le premier paramètre est le texte recherché et le deuxième paramètre contient les options de recherche.

La première requête recherche `*`, ce qui équivaut à effectuer une recherche dans tout, et sélectionne trois des champs de l’index. C’est une bonne pratique que d’appliquer un `select` seulement aux champs dont vous avez besoin, car l’extraction de données inutiles peut ajouter de la latence à vos requêtes.

Les `searchOptions` de cette requête ont également `includeTotalCount` défini sur `true`, ce qui retourne le nombre de résultats correspondants trouvés.

```javascript
async function sendQueries(searchClient) {
    console.log('Query #1 - search everything:');
    let searchOptions = {
        includeTotalCount: true,
        select: ["HotelId", "HotelName", "Rating"]
    };

    let searchResults = await searchClient.search("*", searchOptions);
    for await (const result of searchResults.results) {
        console.log(`${JSON.stringify(result.document)}`);
    }
    console.log(`Result count: ${searchResults.count}`);

    // remaining queries go here
}
```

Les requêtes restantes décrites ci-dessous doivent également être ajoutées à la fonction `sendQueries()`. Elles sont séparées ici seulement pour des raisons de lisibilité.

Dans la requête suivante, nous spécifions le terme de recherche `"wifi"` et nous incluons aussi un filtre pour retourner seulement les résultats où l’état est égal à `'FL'`. Les résultats sont également classés selon la valeur du champ `Rating` de l’hôtel.

```javascript
console.log('Query #2 - Search with filter, orderBy, and select:');
let state = 'FL';
searchOptions = {
    filter: odata`Address/StateProvince eq ${state}`,
    orderBy: ["Rating desc"],
    select: ["HotelId", "HotelName", "Rating"]
};

searchResults = await searchClient.search("wifi", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

Ensuite, la recherche est limitée à un seul champ pouvant faire l’objet d’une recherche en utilisant le paramètre `searchFields`. Il s’agit d’une option intéressante pour améliorer l’efficacité de votre requête si vous savez que vous êtes intéressé seulement par les correspondances dans certains champs. 

```javascript
console.log('Query #3 - Limit searchFields:');
searchOptions = {
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("sublime cliff", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
console.log();
```

Une autre option courante à inclure dans une requête est `facets`. Les facettes vous permettent de créer des filtres sur votre interface utilisateur pour permettre aux utilisateurs de connaître facilement les valeurs sur lesquelles ils peuvent filtrer.

```javascript
console.log('Query #4 - Use facets:');
searchOptions = {
    facets: ["Category"],
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("*", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

La dernière requête utilise la méthode `getDocument()` du `searchClient`. Ceci vous permet de récupérer efficacement un document en utilisant sa clé. 

```javascript
console.log('Query #5 - Lookup document:');
let documentResult = await searchClient.getDocument(key='3')
console.log(`HotelId: ${documentResult.HotelId}; HotelName: ${documentResult.HotelName}`)
```

### <a name="run-the-sample"></a>Exécution de l'exemple

Exécutez le programme avec `node index.js`. À présent, en plus des étapes précédentes, les requêtes sont envoyées et les résultats sont écrits dans la console.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide JavaScript, vous avez effectué une série de tâches pour créer un index, le charger avec des documents et exécuter des requêtes. 

Si vous connaissez déjà la Recherche cognitive Azure, vous pouvez utiliser cet exemple comme tremplin pour tester des suggesteurs (requêtes prédictives ou d’autocomplétion), des filtres et la navigation par facettes. Si vous débutez avec la Recherche cognitive Azure, nous vous recommandons de suivre d’autres tutoriels pour mieux comprendre ce que vous pouvez créer. Consultez les autres ressources disponibles dans notre [page de documentation](https://azure.microsoft.com/documentation/services/search/) . 

> [!div class="nextstepaction"]
> [Créer un front-end React pour Recherche cognitive Azure](https://github.com/dereklegenzoff/azure-search-react-template)