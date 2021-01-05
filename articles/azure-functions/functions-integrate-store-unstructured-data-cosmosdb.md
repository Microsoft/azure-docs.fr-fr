---
title: Stocker des données non structurées à l’aide d’Azure Cosmos DB et d’Azure Functions
description: Stocker des données non structurées à l’aide d’Azure Functions et de Cosmos DB
ms.topic: quickstart
ms.date: 10/01/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 5bc3895cb219338acde492b871dce806db70622b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91661157"
---
# <a name="store-unstructured-data-using-azure-functions-and-azure-cosmos-db"></a>Stocker des données non structurées à l’aide d’Azure Cosmos DB et d’Azure Functions

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) est un excellent moyen de stocker des données JSON et non structurées. Combiné à Azure Functions, Cosmos DB rend le stockage de données simple et rapide avec beaucoup moins de code que pour stocker des données dans une base de données relationnelle.

> [!NOTE]
> À l’heure actuelle, le déclencheur Azure Cosmos DB, les liaisons d’entrée et les liaisons de sortie fonctionnent uniquement avec les comptes d’API SQL et d’API Graph.

Dans Azure Functions, les liaisons d’entrée et de sortie fournissent une méthode déclarative pour se connecter à des données de service externe à partir de votre fonction. Dans cet article, découvrez comment mettre à jour une fonction existante pour ajouter une liaison de sortie qui stocke des données non structurées dans un document Azure Cosmos DB.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

[!INCLUDE [Previous quickstart note](../../includes/functions-quickstart-previous-topics.md)]

## <a name="create-an-azure-cosmos-db-account"></a>Création d’un compte Azure Cosmos DB

Vous devez disposer d’un compte Azure Cosmos DB qui utilise l’API SQL avant de créer la liaison de sortie.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-an-output-binding"></a>Ajouter une liaison de sortie

1. Dans le portail Azure, accédez à l’application de fonction que vous avez créée précédemment et sélectionnez-la.

1. Sélectionnez **Fonctions**, puis sélectionnez la fonction HttpTrigger.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-select-http-function.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::

1. Sélectionnez **Intégration** et **+ Ajouter une sortie**.

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-add-output-binding.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::

1. Utilisez les paramètres **Créer une sortie** comme spécifié dans le tableau :

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-configure-cosmosdb-binding.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::

    | Paramètre      | Valeur suggérée  | Description                                |
    | ------------ | ---------------- | ------------------------------------------ |
    | **Type de liaison** | Azure Cosmos DB | Nom du type de liaison à sélectionner pour créer la liaison de sortie à Azure Cosmos DB. |
    | **Nom du paramètre de document** | taskDocument | Nom qui fait référence à l’objet Cosmos DB dans le code. |
    | **Nom de la base de données** | taskDatabase | Nom de la base de données pour l’enregistrement des documents. |
    | **Nom de la collection** | taskCollection | Nom de la collection de bases de données. |
    | **Si la valeur est true, crée la collection et la base de données Cosmos DB** | Oui | La collection n’existe pas, vous devez la créer. |
    | **Connexion au compte Cosmos DB** | Nouveau paramètre | Sélectionnez **Nouveau**, puis choisissez **Compte Azure Cosmos DB**, le **Compte de base de données** que vous avez créé précédemment, puis sélectionnez **OK**. Crée un paramètre d’application pour votre connexion de compte. Ce paramètre est utilisé par la liaison pour se connecter à la base de données. |

1. Sélectionnez **OK** pour créer la liaison.

## <a name="update-the-function-code"></a>Mettre à jour le code de fonction

Remplacez le code de fonction existant par le code suivant, dans le langage de votre choix :

# <a name="c"></a>[C#](#tab/csharp)

Remplacez la fonction C# existante par le code suivant :

```csharp
#r "Newtonsoft.Json"

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

public static IActionResult Run(HttpRequest req, out object taskDocument, ILogger log)
{
    string name = req.Query["name"];
    string task = req.Query["task"];
    string duedate = req.Query["duedate"];

    // We need both name and task parameters.
    if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(task))
    {
        taskDocument = new
        {
            name,
            duedate,
            task
        };

        return (ActionResult)new OkResult();
    }
    else
    {
        taskDocument = null;
        return (ActionResult)new BadRequestResult();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Remplacez la fonction JavaScript existante par le code suivant :

```js
module.exports = async function (context, req) {

    // We need both name and task parameters.
    if (req.query.name && req.query.task) {

        // Set the output binding data from the query object.
        context.bindings.taskDocument = req.query;

        // Success.
        context.res = {
            status: 200
        };
    }
    else {
        context.res = {
            status: 400,
            body: "The query options 'name' and 'task' are required."
        };
    }
};
```
---

Cet exemple de code lit les chaînes de requête HTTP et les assigne aux champs de l’objet `taskDocument`. La liaison `taskDocument` envoie les données d’objet de ce paramètre de liaison pour les stocker dans la base de données du document lié. La base de données est créée la première fois que la fonction s’exécute.

## <a name="test-the-function-and-database"></a>Tester la fonction et la base de données

1. Sélectionnez **Test/exécution**. Sous **Requête**, sélectionnez **+ Ajouter un paramètre** et ajoutez les paramètres suivants à la chaîne de requête :

    + `name`
    + `task`
    + `duedate`

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::


1. Sélectionnez **Exécuter** et vérifiez que l’état 200 est renvoyé.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function-output.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::


1. Dans le portail Azure, recherchez et sélectionnez **Azure Cosmos DB**.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-search-cosmos-db.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::

1. Sélectionnez votre compte Azure Cosmos DB, puis sélectionnez **Explorateur de données**.

1. Développez les nœuds **TaskCollection**, sélectionnez le nouveau document et confirmez que le document contient des valeurs de votre chaîne de requête, ainsi que des métadonnées supplémentaires.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-data-explorer-check-document.png" alt-text="Dans le portail Azure, sélectionnez votre fonction HTTP." border="true":::

Vous êtes parvenu à ajouter une liaison à votre déclencheur HTTP pour stocker des données non structurées dans une base de données Azure Cosmos DB.

[!INCLUDE [Clean-up section](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la liaison vers une base de données Cosmos DB, consultez la page [Liaisons Cosmos DB Azure Functions](functions-bindings-cosmosdb.md).

[!INCLUDE [functions-quickstart-next-steps](../../includes/functions-quickstart-next-steps-2.md)]
