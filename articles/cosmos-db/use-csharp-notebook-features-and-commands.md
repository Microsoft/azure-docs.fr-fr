---
title: Utiliser les fonctionnalités et commandes de notebook intégrées dans les notebooks C# Azure Cosmos DB (préversion)
description: Découvrez comment utiliser les fonctionnalités et commandes intégrées pour effectuer des opérations courantes à l’aide des notebooks C# intégrés d’Azure Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/19/2020
ms.author: dech
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 67fe71d8e2d6ab239989cb30e9bf5a1b4d731037
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340474"
---
# <a name="use-built-in-notebook-commands-and-features-in-azure-cosmos-db-c-notebooks-preview"></a>Utiliser les fonctionnalités et commandes de notebook intégrées dans les notebooks C# Azure Cosmos DB (préversion)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les notebooks Jupyter intégrés dans Azure Cosmos DB vous permettent d’analyser et de visualiser vos données via le portail Azure. Cet article explique comment utiliser les fonctionnalités et commandes de notebook intégrées pour effectuer des opérations courantes dans des notebooks C#.

## <a name="install-a-new-nuget-package"></a>Installer un nouveau package NuGet
Dès lors que vous avez activé la prise en charge des notebooks pour vos comptes Azure Cosmos, vous pouvez ouvrir un nouveau notebook et installer un package.

Dans une nouvelle cellule de code, insérez et exécutez le code suivant, en remplaçant ``PackageToBeInstalled`` par le package NuGet souhaité, et ``optionalVersion`` par une version spécifique du package, le cas échéant. 

```csharp
#r "nuget: PackageToBeInstalled, optionalVersion"
```

Vous pouvez installer plusieurs packages NuGet dans la même cellule. Ces packages pourront être utilisés à partir de n’importe quel notebook dans l’espace de travail du compte Azure Cosmos. 

Actuellement, l’espace de travail des notebooks C# ne prend pas en charge la résolution récursive des packages NuGet. Si un package NuGet a des dépendances sur d’autres packages NuGet qui ne sont pas actuellement installés, vous devez les référencer explicitement avec le package parent.

> [!TIP]
> Si votre notebook nécessite un package personnalisé, nous vous recommandons d’y ajouter une cellule pour installer le package et d’en faire la première cellule. Cela réduit le risque de conflits avec d’autres packages qu’Azure Cosmos DB charge par défaut. Il est également facile de réinstaller les packages si vous [réinitialiser l’espace de travail](#reset-notebooks-workspace), ce qui supprime tous les packages. 

## <a name="use-the-built-in-azure-cosmos-db-net-sdk"></a>Utiliser le Kit de développement logiciel (SDK) .NET Azure Cosmos DB intégré
La version 3 du [Kit de développement logiciel (SDK) .NET Azure Cosmos DB pour l’API SQL](https://github.com/Azure/azure-cosmos-dotnet-v3) est installée et incluse dans l’environnement du notebook pour le compte Azure Cosmos.

Créez une instance de ``CosmosClient`` pour exécuter n’importe quelle opération du Kit de développement logiciel (SDK). 

Par exemple :

```csharp
// Include usings
using System;
using Microsoft.Azure.Cosmos; //namespace for Azure Cosmos DB .NET V3 SDK
using System.Collections;

// Initialize a new instance of CosmosClient using the built-in account endpoint and key parameters
CosmosClient cosmosClient = new CosmosClient(Cosmos.Endpoint, Cosmos.Key);

// Create a new database and container with 400 RU/s
Database database = await cosmosClient.CreateDatabaseIfNotExistsAsync("DatabaseName");
Container container = await database.CreateContainerIfNotExistsAsync("ContainerName", "/partitionKey", 400);
```
Pour plus d’informations, consultez les [exemples du Kit de développement logiciel (SDK) .NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage). 

> [!IMPORTANT]
> Le Kit de développement logiciel (SDK) .NET Azure Cosmos DB intégré est pris en charge uniquement avec les comptes d’API SQL (Core). Pour les autres API, vous devez [installer le pilote .NET](#install-a-new-nuget-package) correspondant à l’API. 

## <a name="set-custom-options-using-cosmosclientoptions"></a>Définir des options personnalisées à l’aide de ``CosmosClientOptions``
Pour plus de flexibilité, vous pouvez définir la propriété ``CosmosClientOptions`` personnalisée et la transmettre dans votre instance ``CosmosClient``. Vous pouvez utiliser cette propriété pour :

- Définir un nom d’application dans le suffixe user-agent pour l’inclure dans chaque requête.
- Définir la région par défaut à utiliser lors de l’exécution d’opérations sur le service.
- Définir une stratégie de nouvelles tentatives personnalisée pour gérer les requêtes à débit limité.

Consultez l’[article de référence sur l’API CosmosClientOptions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions) pour connaître tous les paramètres pris en charge. Voici un exemple qui montre comment définir la propriété `cosmosClientOptions` :

```csharp
using Microsoft.Azure.Cosmos;

// Configure CosmosClientOptions
var cosmosClientOptions = new CosmosClientOptions
{
    ApplicationName = "ContosoNotebookAppName",
    ApplicationRegion = Regions.RegionName, // By default, this is the region you first created your account in
    MaxRetryAttemptsOnRateLimitedRequests = 9, // By default, this value is 9
};

var client = new CosmosClient(Cosmos.Endpoint, Cosmos.Key, cosmosClientOptions);
```

## <a name="access-the-account-endpoint-and-primary-key-variables"></a>Accéder aux variables de point de terminaison et de clé primaire du compte
Vous pouvez accéder au point de terminaison intégré et à la clé du compte Azure Cosmos dans lequel se trouve votre notebook.

```csharp
var key = Cosmos.Key;
var endpoint = Cosmos.Endpoint;
```

> [!IMPORTANT]
> Les variables ``Cosmos.Key`` et ``Cosmos.Endpoint`` s’appliquent uniquement à l’API SQL. Pour les autres API, recherchez le point de terminaison et la clé dans le panneau **Chaînes de connexion** ou **Clés** de votre compte Azure Cosmos.  

## <a name="print-console-output-in-c-code"></a>Imprimer la sortie de la console dans du code C#
Dans votre code C#, vous pouvez utiliser la syntaxe Display.AsMarkdown() avec une [interpolation de chaîne](/dotnet/csharp/language-reference/tokens/interpolated) pour imprimer la sortie de la console qui s’affiche lorsque vous exécutez la cellule. 

Par exemple : 

```csharp
// Print text in the output
Display.AsMarkdown($"Hello world!");

// Print a variable in the output
for (int i = 0; i < 5; i++) {
    Display.AsMarkdown($"Printing out variable: {i}");
}
```
> [!IMPORTANT]
> La syntaxe Console.WriteLine() n’est actuellement pas prise en charge dans les notebooks C#. Utilisez Display.AsMarkdown pour imprimer la sortie de la console à partir de votre programme. 

## <a name="use-built-in-nteract-data-explorer"></a>Utiliser l’Explorateur de données nteract intégré
Vous pouvez utiliser [l’Explorateur de données nteract](https://blog.nteract.io/designing-the-nteract-data-explorer-f4476d53f897) intégré pour filtrer et visualiser une collection d’éléments. Dans une cellule, placez la variable que vous souhaitez visualiser sur la dernière ligne, qui est automatiquement affichée dans nteract lorsque vous exécutez la cellule.

Par exemple, dans ’exemple *GetingStarted_Csharp.ipynb* , nous pouvons imprimer la variable avec notre résultat, le ``telemetryEvents``. Pour obtenir l’exemple complet, consultez le [notebook GettingStarted_Csharp.ipynb](https://github.com/Azure-Samples/cosmos-notebooks/blob/master/CSharp_quickstarts/GettingStarted_CSharp.ipynb). 

:::image type="content" source="media/use-notebook-features-and-commands/csharp-query-cell.png" alt-text="Cellule de requête Csharp":::

:::image type="content" source="media/use-notebook-features-and-commands/csharp-nteract-built-in-chart.png" alt-text="Explorateur de données nteract":::

## <a name="use-built-in-dictionary-viewer"></a>Utiliser la visionneuse de dictionnaire intégrée
Vous pouvez utiliser la visionneuse de dictionnaire intégrée pour afficher une variable. Dans une cellule, placez la variable que vous souhaitez visualiser sur la dernière ligne, qui sera automatiquement affichée lorsque la cellule sera exécutée.

:::image type="content" source="media/use-notebook-features-and-commands/csharp-built-in-dictionary-viewer.png" alt-text="Visionneuse de dictionnaire intégrée":::

## <a name="upload-json-items-to-a-container"></a>Charger des éléments JSON dans un conteneur
Vous pouvez utiliser la commande magic ``%%upload`` pour télécharger des données d’un fichier JSON vers un conteneur Azure Cosmos spécifié. Utilisez la commande suivante pour télécharger les éléments :

```csharp
%%upload --databaseName {database_id} --containerName {container_id} --url {url_location_of_file}
```

- Remplacez ``{database_id}`` et ``{container_id}`` par le nom de la base de données et du conteneur dans votre compte Azure Cosmos. 
- Remplacez ``{url_location_of_file}`` par l’emplacement de votre fichier JSON. Le fichier doit être un tableau d’objets JSON valides et doit être accessible via l’Internet public.

Par exemple :

```csharp
%%upload --database databaseName --container containerName --url 
https://contoso.com/path/to/data.json
```
```
Documents successfully uploaded to ContainerName
Total number of documents imported : 2654
Total time taken : 00:00:38.1228087 hours
Total RUs consumed : 25022.58
```

Avec les statistiques de sortie, vous pouvez calculer les RU/s effectives utilisées pour charger les éléments. Par exemple, si 25 000 unités de requête ont été consommées sur 38 secondes, les RU/s effectives sont de 25 000 RU/38 secondes, soit 658 RU/s.

## <a name="run-another-notebook-in-current-notebook"></a>Exécuter un autre notebook dans le notebook actuel 
Vous pouvez utiliser la commande magique ``%%run`` pour exécuter un autre notebook dans votre espace de travail à partir de votre notebook actuel. Utilisez la syntaxe suivante :

```csharp
%%run ./path/to/{notebookName}.ipynb
```
Remplacez ``{notebookName}`` par le nom du notebook que vous voulez exécuter. Le notebook doit figurer dans votre espace de travail « Mes notebooks » actuel. 

## <a name="reset-notebooks-workspace"></a>Réinitialiser l’espace de travail des notebooks
Pour rétablir les paramètres par défaut de l’espace de travail des notebooks, sélectionnez **Réinitialiser l’espace de travail** dans la barre de commandes. Cette opération supprime tous les packages personnalisés installés et redémarre le serveur Jupyter. Vos notebooks, fichiers et ressources Azure Cosmos ne seront pas affectés.  

:::image type="content" source="media/use-notebook-features-and-commands/reset-workspace.png" alt-text="Réinitialiser l’espace de travail des notebooks":::

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les avantages de [notebooks Jupyter d’Azure Cosmos DB](cosmosdb-jupyter-notebooks.md)
- Apprenez-en davantage sur le [Kit de développement logiciel (SDK) .NET Azure Cosmos DB pour l’API SQL](https://github.com/Azure/azure-cosmos-dotnet-v3).
