---
title: API Table Azure Cosmos DB à l’aide du SDK .NET Standard
description: Découvrez comment stocker et interroger les données structurées dans le compte d’API Table Azure Cosmos DB
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 12/03/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: f43a890185fcd69967fd2a035c27e1cf92798f04
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548152"
---
# <a name="get-started-with-azure-cosmos-db-table-api-and-azure-table-storage-using-the-net-sdk"></a>Bien démarrer avec l’API Table d’Azure Cosmos DB et le stockage Table Azure à l’aide du SDK .NET
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Vous pouvez utiliser l’API Table d’Azure Cosmos DB ou le stockage Table Azure pour stocker des données NoSQL structurées dans le cloud, en fournissant un magasin de clés/attributs avec une conception sans schéma. Comme l’API Table d’Azure Cosmos DB et le stockage Table Azure présentent une conception sans schéma, il est facile d’adapter vos données en fonction des besoins de votre application. Vous pouvez utiliser l’API Table d’Azure Cosmos DB et le stockage Table Azure pour stocker des jeux de données flexibles, comme des données utilisateur pour des applications web, des carnets d’adresses, des informations sur les appareils ou d’autres types de métadonnées requis par votre service. 

Ce tutoriel décrit un exemple qui vous montre comment utiliser la [bibliothèque Microsoft Azure Cosmos DB Table pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) dans des scénarios basés sur l’API Table d’Azure Cosmos DB et le Stockage Table Azure. Vous devez utiliser la connexion propre au service Azure. Ces scénarios sont explorés à l’aide d’exemples en C# qui montrent comment créer des tables, ajouter ou mettre à jour des données, interroger des données et supprimer des tables.

## <a name="prerequisites"></a>Prérequis

Vous aurez besoin des éléments suivants pour suivre cet exemple :

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)

* [Bibliothèque Microsoft Azure CosmosDB Table pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table) : cette bibliothèque est actuellement disponible pour l’infrastructure .NET Standard et .NET. 

* [Compte d’API Table Cosmos DB Azure](create-table-dotnet.md#create-a-database-account).

## <a name="create-an-azure-cosmos-db-table-api-account"></a>Créer un compte d’API de table Azure Cosmos DB

[!INCLUDE [cosmos-db-create-dbaccount-table](../../includes/cosmos-db-create-dbaccount-table.md)]

## <a name="create-a-net-console-project"></a>Créer un projet de console .NET

Dans Visual Studio, créez une application console .NET. Les étapes suivantes vous montrent comment créer une application console dans Visual Studio 2019. Vous pouvez utiliser la bibliothèque Azure Cosmos DB Table dans n’importe quel type d’application .NET, y compris un service cloud Azure, une application web, une application de bureau ou une application mobile. Dans ce guide, nous utilisons une application console pour plus de simplicité.

1. Sélectionnez **Fichier** > **Nouveau** > **Projet**.

1. Choisissez **Application console (.NET Core)** , puis sélectionnez **Suivant**.

1. Dans le champ **Nom de projet**, entrez un nom pour votre application, tel que **CosmosTableSamples**. Vous pouvez fournir un autre nom.

1. Sélectionnez **Create** (Créer).

Tous les exemples de code figurant dans cette démonstration peuvent être ajoutés à la méthode Main() dans le fichier **Program.cs** de votre application console.

## <a name="install-the-required-nuget-package"></a>Installer le package NuGet nécessaire

Pour obtenir le package NuGet, effectuez les étapes suivantes :

1. Cliquez avec le bouton droit sur votre projet dans **l’Explorateur de solutions**, puis sélectionnez **Gérer les packages NuGet**.

1. Recherchez en ligne [`Microsoft.Azure.Cosmos.Table`](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), [`Microsoft.Extensions.Configuration`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration), [`Microsoft.Extensions.Configuration.Json`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json), [`Microsoft.Extensions.Configuration.Binder`](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder), puis sélectionnez **Installer** pour installer la bibliothèque Microsoft Azure Cosmos DB Table.

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage

1. À partir du [Portail Azure](https://portal.azure.com/), accédez à votre compte Azure Cosmos ou au compte Stockage Table. 

1. Ouvrez le volet **Chaîne de connexion** ou **Clés d’accès**. Utilisez les boutons de copie sur le côté droit de la fenêtre pour copier la **CHAÎNE DE CONNEXION PRINCIPALE**.

   :::image type="content" source="./media/create-table-dotnet/connection-string.png" alt-text="Consulter et copier la CHAÎNE DE CONNEXION PRINCIPALE dans le volet Chaîne de connexion":::
   
1. Pour configurer votre chaîne de connexion, à partir de Visual Studio, cliquez avec le bouton droit sur votre projet **CosmosTableSamples**.

1. Sélectionnez **Ajouter**, puis **Nouvel élément**. Créez un fichier **Settings.json** avec le type de fichier **config JSON TypeScript**. 

1. Remplacez le code dans le fichier Settings.json par le code suivant et affectez la chaîne de connexion principale :

   ```csharp
   {
   "StorageConnectionString": <Primary connection string of your Azure Cosmos DB account>
   }
   ```

1. Cliquez avec le bouton droit sur votre projet **CosmosTableSamples**. Sélectionnez **Ajouter**, **Nouvel élément** et ajoutez une classe nommée **AppSettings.cs**.

1. Ajoutez le code suivant au fichier AppSettings.cs. Ce fichier lit la chaîne de connexion contenue dans le fichier Settings.json et l’affecte au paramètre de configuration :

  :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/AppSettings.cs":::

## <a name="parse-and-validate-the-connection-details"></a>Analyser et valider les informations de connexion

1. Cliquez avec le bouton droit sur votre projet **CosmosTableSamples**. Sélectionnez **Ajouter**, **Nouvel élément** et ajoutez une classe nommée **Common.cs**. Vous allez écrire le code qui valide les informations de connexion et crée une table au sein de cette classe.

1. Définissez une méthode `CreateStorageAccountFromConnectionString` comme indiqué ci-dessous. Cette méthode analyse les informations de la chaîne de connexion et vérifie que le nom et la clé du compte fournis dans le fichier « Settings.json » sont corrects.

   :::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="createStorageAccount":::

## <a name="create-a-table"></a>Créer une table 

La classe [CloudTableClient](/dotnet/api/microsoft.azure.cosmos.table.cloudtableclient) vous permet de récupérer des tables et entités stockées dans Table Storage. Étant donné qu’il n’y a pas de tables dans le compte d’API Table Cosmos DB, nous allons en créer une en ajoutant la méthode `CreateTableAsync` à la classe **Common.cs** :

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Common.cs" id="CreateTable":::

Si vous recevez une erreur « Exception 503 - Service indisponible », il est possible que les ports nécessaires pour le mode de connectivité soient bloqués par un pare-feu. Pour résoudre ce problème, ouvrez les ports nécessaires ou utilisez la connectivité en mode de passerelle, comme indiqué dans le code suivant :

```csharp
tableClient.TableClientConfiguration.UseRestExecutorForCosmosEndpoint = true;
```

## <a name="define-the-entity"></a>Définir l’entité 

Les entités mappent vers les objets C# en utilisant une classe personnalisée dérivée d’une [TableEntity](/dotnet/api/microsoft.azure.cosmos.table.tableentity). Pour ajouter une entité à une table, créez une classe définissant les propriétés de votre entité.

Cliquez avec le bouton droit sur votre projet **CosmosTableSamples**. Sélectionnez **Ajouter**, **Nouveau dossier** et nommez le dossier **Model** (Modèle). Dans le dossier Model, ajoutez une classe nommée **CustomerEntity.cs** et ajoutez-y le code suivant.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Model/CustomerEntity.cs":::

Ce code définit une classe d’entité où le prénom et le nom de famille du client sont utilisés en tant que clé de ligne et clé de partition, respectivement. Ensemble, les clés de partition et de ligne d’une entité l’identifient de façon unique dans la table. Les requêtes sur des entités ayant la même clé de partition sont plus rapides que celles effectuées sur des entités qui n’ont pas la même, mais l’utilisation de clés de partition différentes améliore la scalabilité des opérations parallèles. Les entités à stocker dans des tables doivent être d’un type pris en charge, par exemple dérivé de la classe [TableEntity](/dotnet/api/microsoft.azure.cosmos.table.tableentity). Les propriétés de l’entité à stocker dans une table doivent être des propriétés publiques du type et prendre en charge la récupération et la définition de valeurs. De plus, le type d’entité doit exposer un constructeur sans paramètre.

## <a name="insert-or-merge-an-entity"></a>Insérer ou fusionner une entité

L’exemple de code suivant crée un objet entité et l’ajoute à la table. La méthode InsertOrMerge dans la classe [TableOperation](/dotnet/api/microsoft.azure.cosmos.table.tableoperation) s’utilise pour insérer ou fusionner une entité. La méthode [CloudTable.ExecuteAsync](/dotnet/api/microsoft.azure.cosmos.table.cloudtable.executeasync?preserve-view=true&view=azure-dotnet) est appelée pour exécuter l’opération. 

Cliquez avec le bouton droit sur votre projet **CosmosTableSamples**. Sélectionnez **Ajouter**, **Nouvel élément** et ajoutez une classe nommée **SamplesUtils.cs**. Cette classe stocke tout le code requis pour effectuer des opérations CRUD sur les entités. 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="InsertItem":::

## <a name="get-an-entity-from-a-partition"></a>Obtenir une entité à partir d’une partition

Vous pouvez obtenir une entité à partir d’une partition en utilisant la méthode Retrieve dans la classe [TableOperation](/dotnet/api/microsoft.azure.cosmos.table.tableoperation). L’exemple de code suivant obtient la clé de ligne de la clé de partition, l’e-mail et le numéro de téléphone d’une entité client. Cet exemple affiche également les unités de requête consommées pour obtenir l’entité. Pour rechercher une entité, ajoutez le code suivant au fichier **SamplesUtils.cs** :

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="QueryData":::

## <a name="delete-an-entity"></a>Suppression d’une entité

Il est facile de supprimer une entité après l’avoir récupérée. Il suffit pour cela d’utiliser la procédure suivie pour la mise à jour d’une entité. Le code suivant extrait et supprime une entité de client. Pour supprimer une entité, ajoutez le code suivant au fichier **SamplesUtils.cs** : 

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/SamplesUtils.cs" id="DeleteItem":::

## <a name="execute-the-crud-operations-on-sample-data"></a>Exécuter les opérations CRUD sur les exemples de données

Après avoir défini les méthodes pour créer une table et ajouter ou fusionner des entités, exécutez-les sur les exemples de données. Pour ce faire, cliquez avec le bouton droit sur votre projet **CosmosTableSamples**. Sélectionnez **Ajouter**, **Nouvel élément** et ajoutez une classe nommée **BasicSamples.cs**, puis ajoutez-y le code suivant. Ce code crée une table et y ajoute des entités.

Si vous ne souhaitez pas supprimer l’entité et la table à la fin du projet, commentez les méthodes `await table.DeleteIfExistsAsync()` et `SamplesUtils.DeleteEntityAsync(table, customer)` dans le code suivant. Il est préférable de commenter ces méthodes et de valider les données avant de supprimer la table.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/BasicSamples.cs":::

Le code précédent crée une table dont le nom est composé du préfixe « demo », suivi du GUID généré. Il ajoute ensuite une entité client avec le nom « Harp » et le prénom « Walter », puis met à jour le numéro de téléphone de cet utilisateur. 

Dans ce tutoriel, vous avez créé le code nécessaire pour effectuer des opérations CRUD de base sur les données stockées dans un compte d’API Table. Vous pouvez aussi effectuer diverses opérations avancées, par exemple, ajouter des données par lot, interroger toutes les données d’une partition, interroger une plage de données d’une partition et lister les tables du compte dont les noms commencent par le préfixe spécifié. Vous pouvez télécharger l’exemple complet à partir du dépôt [azure-cosmos-table-dotnet-core-getting-started](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started) sur GitHub. La classe [AdvancedSamples.cs](https://github.com/Azure-Samples/azure-cosmos-table-dotnet-core-getting-started/blob/main/CosmosTableSamples/AdvancedSamples.cs) permet d’effectuer davantage d’opérations sur les données.  

## <a name="run-the-project"></a>Exécuter le projet

À partir de votre projet **CosmosTableSamples**. Ouvrez la classe nommée **Program.cs** et ajoutez-lui le code suivant pour appeler BasicSamples quand le projet s’exécute.

:::code language="csharp" source="~/azure-cosmosdb-dotnet-table/CosmosTableSamples/Program.cs":::

Maintenant, générez la solution et appuyez sur F5 pour exécuter le projet. Après l’exécution du projet, vous voyez la sortie suivante dans l’invite de commandes :

:::image type="content" source="./media/tutorial-develop-table-standard/output-from-sample.png" alt-text="Sortie de l’invite de commandes":::

Si vous voyez une erreur indiquant qu’un fichier Settings.json n’a pas été trouvé pendant l’exécution du projet, vous pouvez la résoudre en ajoutant l’entrée XML suivante dans les paramètres du projet. Cliquez avec le bouton droit sur CosmosTableSamples, sélectionnez Modifier CosmosTableSamples.csproj et ajoutez l’élément itemGroup suivant : 

```csharp
  <ItemGroup>
    <None Update="Settings.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>
```
Vous pouvez maintenant vous connecter au portail Azure et vérifier que les données existent bien dans la table. 

:::image type="content" source="./media/tutorial-develop-table-standard/results-in-portal.png" alt-text="Résultats dans le portail":::

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant passer au tutoriel suivant pour découvrir comment migrer les données vers un compte d’API Table Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Migrer des données vers l’API Table Azure Cosmos DB](../cosmos-db/table-import.md)