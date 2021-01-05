---
title: Écrire des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB
description: Découvrez comment écrire des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: 18cedad34a6ca7d9a0ba18cd01c082f2878380a8
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339816"
---
# <a name="how-to-write-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>Comment écrire des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB fournit une exécution transactionnelle avec langage intégré de JavaScript qui vous permet d’écrire des **procédures stockées** , des **déclencheurs** et des **fonctions définies par l’utilisateur**. Lorsque vous utilisez l’API SQL dans Azure Cosmos DB, vous pouvez définir les procédures stockées, les déclencheurs et les fonctions définies par l’utilisateur dans le langage JavaScript. Vous pouvez écrire votre logique dans JavaScript et l’exécuter dans le moteur de base de données. Vous pouvez créer et exécuter des déclencheurs, des procédures stockées et des fonctions définies par l’utilisateur à l’aide du [Portail Azure](https://portal.azure.com/), de [l’API de requête avec langage intégré JavaScript dans Azure Cosmos DB](javascript-query-api.md) et des [Kits de développement logiciel (SDK) clients de l’API SQL Cosmos DB](sql-api-dotnet-samples.md). 

Pour appeler une procédure stockée, un déclencheur, une fonction définie par l’utilisateur, vous devez les inscrire. Pour plus d’informations, consultez [How to register and use stored procedures, triggers, and user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Comment inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB).

> [!NOTE]
> Pour les conteneurs partitionnés, lorsque vous exécutez une procédure stockée, vous devez fournir une valeur de clé de partition dans les options de requête. Les procédures stockées se limitent toujours à une clé de partition. Les éléments qui ont une valeur de clé de partition différente ne seront pas visibles dans la procédure stockée. Cela s’applique également aux déclencheurs.
> [!Tip]
> Cosmos prend en charge le déploiement de conteneurs avec des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur. Pour plus d’informations, consultez [Créer un conteneur Azure Cosmos DB avec des fonctionnalités côté serveur.](./manage-with-templates.md#create-sproc)

## <a name="how-to-write-stored-procedures"></a><a id="stored-procedures"></a>Comment écrire des procédures stockées

Les procédures stockées sont écrites à l’aide de JavaScript ; elles peuvent créer, mettre à jour, lire, interroger et supprimer des éléments à l’intérieur d’un conteneur Azure Cosmos. Les procédures stockées sont enregistrées par collection, et elles peuvent s’appliquer à tout document ou pièce jointe figurant dans cette collection.

Voici une simple procédure stockée qui renvoie une réponse « Hello World ».

```javascript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

L’objet de contexte donne accès à toutes les opérations pouvant être effectuées dans Azure Cosmos DB, ainsi que l’accès aux objets de requête et de réponse. En l’occurrence, vous utilisez l’objet de réponse pour définir le corps de la réponse renvoyée au client.

Une fois écrite, la procédure stockée doit être inscrite auprès d’une collection. Pour plus d’informations, consultez l’article [How to use stored procedures in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures) (Comment utiliser des procédures stockées dans Azure Cosmos DB).

### <a name="create-an-item-using-stored-procedure"></a><a id="create-an-item"></a>Créer un élément à l’aide de la procédure stockée

Quand vous créez un élément à l’aide d’une procédure stockée, il est inséré dans le conteneur Azure Cosmos et un ID pour le nouvel élément est retourné. La création d’un élément est une opération asynchrone, et varie selon les fonctions de rappel JavaScript. La fonction de rappel présente deux paramètres : un pour l’objet d’erreur en cas d’échec de l’opération et un autre pour une valeur renvoyée, ici l’objet créé. À l’intérieur du rappel, vous pouvez gérer l’exception ou générer une erreur. Si aucun rappel n’est fourni et qu’une erreur se produit, le runtime d’Azure Cosmos DB génère une erreur.

La procédure stockée inclut également un paramètre pour définir la description ; il s’agit d’une valeur booléenne. Lorsque le paramètre est défini sur true et si la description est manquante, la procédure stockée génère une exception. Sinon, le reste de la procédure stockée continue de s’exécuter.

L’exemple de procédure stockée suivant prend un nouvel élément Azure Cosmos en tant qu’entrée, l’insère dans le conteneur Azure Cosmos et retourne le nombre des éléments insérés. Dans cet exemple, nous utilisons l’exemple ToDoList de l’article [Démarrage rapide : Développer une application web .NET avec Azure Cosmos DB à l’aide de l’API SQL et du portail Azure](create-sql-api-dotnet.md)

```javascript
function createToDoItems(items) {
    var collection = getContext().getCollection();
    var collectionLink = collection.getSelfLink();
    var count = 0;

    if (!items) throw new Error("The array is undefined or null.");

    var numItems = items.length;

    if (numItems == 0) {
        getContext().getResponse().setBody(0);
        return;
    }

    tryCreate(items[count], callback);

    function tryCreate(item, callback) {
        var options = { disableAutomaticIdGeneration: false };

        var isAccepted = collection.createDocument(collectionLink, item, options, callback);

        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    function callback(err, item, options) {
        if (err) throw err;
        count++;
        if (count >= numItems) {
            getContext().getResponse().setBody(count);
        } else {
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="arrays-as-input-parameters-for-stored-procedures"></a>Tableaux en tant que paramètres d’entrée pour les procédures stockées

Quand vous définissez une procédure stockée dans le Portail Azure, les paramètres d’entrée sont toujours envoyés sous forme de chaîne à la procédure stockée. Même si vous passez un tableau de chaînes sous forme d’entrée, le tableau est converti en chaîne et envoyé à la procédure stockée. Pour contourner cela, vous pouvez définir une fonction à l’intérieur de votre procédure stockée pour que la chaîne soit analysée en tant que tableau. Le code suivant montre comment analyser un paramètre d’entrée de chaîne sous forme de tableau :

```javascript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

### <a name="transactions-within-stored-procedures"></a><a id="transactions"></a>Transactions dans des procédures stockées

Vous pouvez implémenter des transactions sur des éléments dans un conteneur à l’aide d’une procédure stockée. L’exemple suivant utilise des transactions au sein d’une application de jeu de football fantastique pour échanger des joueurs entre deux équipes dans une seule opération. La procédure stockée essaie de lire les deux éléments Azure Cosmos, qui correspondent chacun aux ID de joueurs transmis en tant qu’arguments. Si deux joueurs sont trouvés, la procédure stockée met à jour les éléments en intervertissant leurs équipes. Si des erreurs se produisent, elle génère une exception JavaScript qui annule implicitement la transaction.

```javascript
// JavaScript source code
function tradePlayers(playerId1, playerId2) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();

    var player1Document, player2Document;

    // query for players
    var filterQuery =
    {
        'query' : 'SELECT * FROM Players p where p.id = @playerId1',
        'parameters' : [{'name':'@playerId1', 'value':playerId1}] 
    };

    var accept = container.queryDocuments(container.getSelfLink(), filterQuery, {},
        function (err, items, responseOptions) {
            if (err) throw new Error("Error" + err.message);

            if (items.length != 1) throw "Unable to find both names";
            player1Item = items[0];

            var filterQuery2 =
            {
                'query' : 'SELECT * FROM Players p where p.id = @playerId2',
                'parameters' : [{'name':'@playerId2', 'value':playerId2}]
            };
            var accept2 = container.queryDocuments(container.getSelfLink(), filterQuery2, {},
                function (err2, items2, responseOptions2) {
                    if (err2) throw new Error("Error" + err2.message);
                    if (items2.length != 1) throw "Unable to find both names";
                    player2Item = items2[0];
                    swapTeams(player1Item, player2Item);
                    return;
                });
            if (!accept2) throw "Unable to read player details, abort ";
        });

    if (!accept) throw "Unable to read player details, abort ";

    // swap the two players’ teams
    function swapTeams(player1, player2) {
        var player2NewTeam = player1.team;
        player1.team = player2.team;
        player2.team = player2NewTeam;

        var accept = container.replaceDocument(player1._self, player1,
            function (err, itemReplaced) {
                if (err) throw "Unable to update player 1, abort ";

                var accept2 = container.replaceDocument(player2._self, player2,
                    function (err2, itemReplaced2) {
                        if (err) throw "Unable to update player 2, abort"
                    });

                if (!accept2) throw "Unable to update player 2, abort";
            });

        if (!accept) throw "Unable to update player 1, abort";
    }
}
```

### <a name="bounded-execution-within-stored-procedures"></a><a id="bounded-execution"></a>Exécution liée dans des procédures stockées

Voici un exemple de procédure stockée qui importe en bloc des éléments dans un conteneur Azure Cosmos. La procédure stockée gère l’exécution liée en vérifiant la valeur de retour booléenne à partir de `createDocument`, puis utilise le nombre d’éléments insérés dans chaque appel de la procédure stockée pour effectuer le suivi de la progression et la reprendre d’un lot à un autre.

```javascript
function bulkImport(items) {
    var container = getContext().getCollection();
    var containerLink = container.getSelfLink();

    // The count of imported items, also used as current item index.
    var count = 0;

    // Validate input.
    if (!items) throw new Error("The array is undefined or null.");

    var itemsLength = items.length;
    if (itemsLength == 0) {
        getContext().getResponse().setBody(0);
    }

    // Call the create API to create an item.
    tryCreate(items[count], callback);

    // Note that there are 2 exit conditions:
    // 1) The createDocument request was not accepted.
    //    In this case the callback will not be called, we just call setBody and we are done.
    // 2) The callback was called items.length times.
    //    In this case all items were created and we don’t need to call tryCreate anymore. Just call setBody and we are done.
    function tryCreate(item, callback) {
        var isAccepted = container.createDocument(containerLink, item, callback);

        // If the request was accepted, callback will be called.
        // Otherwise report current count back to the client,
        // which will call the script again with remaining set of items.
        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    // This is called when container.createDocument is done in order to process the result.
    function callback(err, item, options) {
        if (err) throw err;

        // One more item has been inserted, increment the count.
        count++;

        if (count >= itemsLength) {
            // If we created all items, we are done. Just set the response.
            getContext().getResponse().setBody(count);
        } else {
            // Create next document.
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="async-await-with-stored-procedures"></a><a id="async-promises"></a>Attente asynchrone avec des procédures stockées

Voici un exemple de procédure stockée qui utilise Async-await avec des promesses à l’aide d’une fonction d’assistance. La procédure stockée interroge un élément et le remplace.

```javascript
function async_sample() {
    const ERROR_CODE = {
        NotAccepted: 429
    };

    const asyncHelper = {
        queryDocuments(sqlQuery, options) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.queryDocuments(__.getSelfLink(), sqlQuery, options, (err, feed, options) => {
                    if (err) reject(err);
                    resolve({ feed, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        },

        replaceDocument(doc) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.replaceDocument(doc._self, doc, (err, result, options) => {
                    if (err) reject(err);
                    resolve({ result, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        }
    };

    async function main() {
        let continuation;
        do {
            let { feed, options } = await asyncHelper.queryDocuments("SELECT * from c", { continuation });

            for (let doc of feed) {
                doc.newProp = 1;
                await asyncHelper.replaceDocument(doc);
            }

            continuation = options.continuation;
        } while (continuation);
    }

    main().catch(err => getContext().abort(err));
}
```

## <a name="how-to-write-triggers"></a><a id="triggers"></a>Comment écrire des déclencheurs

Azure Cosmos DB prend en charge les prédéclencheurs et les post-déclencheurs. Les prédéclencheurs sont exécutés avant de modifier un élément de la base de données, et les post-déclencheurs sont exécutés après la modification d’un élément de la base de données.

### <a name="pre-triggers"></a><a id="pre-triggers"></a>Prédéclencheurs

Voici un exemple de la façon dont un prédéclencheur est utilisé pour valider les propriétés d’un élément Azure Cosmos en cours de création. Dans cet exemple, nous utilisons l’exemple ToDoList de l’article [Démarrage rapide : Développer une application web .NET avec Azure Cosmos DB à l’aide de l’API SQL et du portail Azure](create-sql-api-dotnet.md) pour ajouter une propriété timestamp à un élément nouvellement ajouté qui n’en contient pas.

```javascript
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```

Les pré-déclencheurs ne peuvent pas avoir de paramètres en entrée. L’objet de requête dans le déclencheur est utilisé pour manipuler le message de requête associé à l’opération. Dans l’exemple précédent, le prédéclencheur est exécuté lors de la création d’un élément Azure Cosmos, et le corps du message de requête contient l’élément qui doit être créé au format JSON.

Lorsque les déclencheurs sont inscrits, vous pouvez spécifier les opérations avec lesquelles ils peuvent s’exécuter. Ce déclencheur doit être créé avec une valeur `TriggerOperation` de `TriggerOperation.Create`, ce qui signifie que l’utilisation du déclencheur dans une opération de remplacement comme indiqué dans le code suivant n’est pas autorisée.

Pour des exemples d’inscription et d’appel d’un prédéclencheur, consultez les articles dédiés aux [prédéclencheurs](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) et aux [post-déclencheurs](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

### <a name="post-triggers"></a><a id="post-triggers"></a>Post-déclencheurs

L’exemple suivant illustre un post-déclencheur. Ce déclencheur interroge l’élément de métadonnées et le met à jour avec des informations relatives à l’élément qui vient d’être créé.


```javascript
function updateMetadata() {
var context = getContext();
var container = context.getCollection();
var response = context.getResponse();

// item that was created
var createdItem = response.getBody();

// query for metadata document
var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
    updateMetadataCallback);
if(!accept) throw "Unable to update metadata, abort";

function updateMetadataCallback(err, items, responseOptions) {
    if(err) throw new Error("Error" + err.message);
        if(items.length != 1) throw 'Unable to find metadata document';

        var metadataItem = items[0];

        // update metadata
        metadataItem.createdItems += 1;
        metadataItem.createdNames += " " + createdItem.id;
        var accept = container.replaceDocument(metadataItem._self,
            metadataItem, function(err, itemReplaced) {
                    if(err) throw "Unable to update metadata, abort";
            });
        if(!accept) throw "Unable to update metadata, abort";
        return;
}
```

Un élément important à noter est l’exécution transactionnelle des déclencheurs dans Azure Cosmos DB. Le post-déclencheur s’exécute dans le cadre de la même transaction pour l’élément sous-jacent lui-même. Une exception pendant l’exécution du post-déclencheur fait échouer toute la transaction. Tout ce qui est déjà validé est annulé et une exception est retournée.

Pour des exemples d’inscription et d’appel d’un prédéclencheur, consultez les articles dédiés aux [prédéclencheurs](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) et aux [post-déclencheurs](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

## <a name="how-to-write-user-defined-functions"></a><a id="udfs"></a>Comment écrire des fonctions définies par l’utilisateur

L’exemple suivant crée une fonction définie par l’utilisateur pour calculer les impôts sur le revenu en fonction de diverses tranches de revenu. Cette fonction définie par l’utilisateur peut ensuite être utilisée dans une requête. Dans le cadre de cet exemple, supposons qu’il existe un conteneur appelé « Revenus » avec des propriétés comme suit :

```json
{
   "name": "Satya Nadella",
   "country": "USA",
   "income": 70000
}
```

Ce qui suit correspond à une définition de fonction pour calculer les impôts sur le revenu en fonction de diverses tranches de revenu :

```javascript
function tax(income) {

        if(income == undefined)
            throw 'no input';

        if (income < 1000)
            return income * 0.1;
        else if (income < 10000)
            return income * 0.2;
        else
            return income * 0.4;
    }
```

Pour des exemples d’inscription et d’utilisation d’une fonction définie par l’utilisateur, consultez l’article [How to register and use stored procedures, triggers, and user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#udfs) (Comment inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB).

## <a name="logging"></a>Journalisation 

Lorsque vous utilisez une procédure stockée, des déclencheurs ou des fonctions définies par l’utilisateur, vous pouvez journaliser les étapes à l’aide de la commande `console.log()`. Cette commande concentre une chaîne à des fins de débogage lorsque `EnableScriptLogging` a la valeur true, comme indiqué dans l’exemple suivant :

```javascript
var response = await client.ExecuteStoredProcedureAsync(
document.SelfLink,
new RequestOptions { EnableScriptLogging = true } );
Console.WriteLine(response.ScriptLog);
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus de concepts et comment écrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB :

* [Comment inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md)

* [How to write stored procedures and triggers in Azure Cosmos DB by using the JavaScript query API](how-to-write-javascript-query-api.md) (Comment écrire des procédures stockées et des déclencheurs à l’aide de l’API de requête JavaScript dans Azure Cosmos DB)

* [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions](stored-procedures-triggers-udfs.md) (Utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur Azure Cosmos DB)

* [Working with JavaScript language-integrated query API with Azure Cosmos DB](javascript-query-api.md) (Utiliser l’API de requête avec langage intégré JavaScript dans Azure Cosmos DB)