---
title: Importer des données en bloc dans un compte d’API SQL Azure Cosmos DB à l’aide du kit SDK .NET
description: Découvrez comment importer ou ingérer des données dans Azure Cosmos DB en générant une application console .NET qui optimise le débit provisionné (RU/s) nécessaire pour importer des données
author: ealsur
ms.author: maquaran
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: tutorial
ms.date: 09/21/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 6cf0e77657175449b126eeca02a12c164478e568
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548067"
---
# <a name="bulk-import-data-to-azure-cosmos-db-sql-api-account-by-using-the-net-sdk"></a>Importer des données en bloc dans un compte d’API SQL Azure Cosmos DB à l’aide du kit SDK .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ce tutoriel vous montre comment créer une application console .NET qui optimise le débit approvisionné (RU/s) requis pour importer des données dans Azure Cosmos DB. Dans cet article, vous allez lire des données à partir d’un exemple de source de données et les importer dans un conteneur Azure Cosmos.
Ce tutoriel utilise la [Version 3.0+](https://www.nuget.org/packages/Microsoft.Azure.Cosmos) du kit de développement logiciel (SDK) .NET Azure Cosmos DB, qui peut être ciblé sur le .NET Framework ou .NET Core.

Ce didacticiel contient les sections suivantes :

> [!div class="checklist"]
> * Création d’un compte Azure Cosmos
> * Configuration de votre projet
> * Connexion à un compte Azure Cosmos avec prise en charge de l'exécution en bloc activée
> * Effectuer une importation de données via des opérations de création simultanées

## <a name="prerequisites"></a>Prérequis

Avant de suivre les instructions de cet article, vérifiez que vous disposez bien des ressources suivantes :

* Un compte Azure actif. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Kit de développement logiciel (SDK) .NET Core 3](https://dotnet.microsoft.com/download/dotnet-core) Pour connaître la version disponible dans votre environnement, exécutez `dotnet --version`.

## <a name="step-1-create-an-azure-cosmos-db-account"></a>Étape 1 : Création d’un compte Azure Cosmos DB

[Créez un compte d'API SQL Azure Cosmos DB](create-cosmosdb-resources-portal.md) à partir du portail Azure ou à l'aide de l'[émulateur Azure Cosmos DB](local-emulator.md).

## <a name="step-2-set-up-your-net-project"></a>Étape 2 : Configurer votre projet .NET

Ouvrez l’invite de commandes Windows ou une fenêtre de terminal à partir de votre ordinateur local. Vous allez exécuter toutes les commandes des sections suivantes à partir de l’invite de commandes ou du terminal. Exécutez la commande dotnet new suivante pour créer une application nommée *bulk-import-demo*. Le paramètre `--langVersion` définit la propriété *LangVersion* dans le fichier projet créé.

   ```bash
   dotnet new console –langVersion:8 -n bulk-import-demo
   ```

Déplacez vos répertoires vers le dossier d’application nouvellement créé. Vous pouvez générer l’application avec :

   ```bash
   cd bulk-import-demo
   dotnet build
   ```

La sortie attendue de la build doit ressembler à ceci :

   ```bash
   Restore completed in 100.37 ms for C:\Users\user1\Downloads\CosmosDB_Samples\bulk-import-demo\bulk-import-demo.csproj.
     bulk -> C:\Users\user1\Downloads\CosmosDB_Samples\bulk-import-demo \bin\Debug\netcoreapp2.2\bulk-import-demo.dll

   Build succeeded.
       0 Warning(s)
       0 Error(s)

   Time Elapsed 00:00:34.17
   ```

## <a name="step-3-add-the-azure-cosmos-db-package"></a>Étape 3 : Ajouter le package Azure Cosmos DB

Tout en restant dans le répertoire de l’application, installez la bibliothèque de client Azure Cosmos DB pour .NET Core à l’aide de la commande dotnet add Package.

   ```bash
   dotnet add package Microsoft.Azure.Cosmos
   ```

## <a name="step-4-get-your-azure-cosmos-account-credentials"></a>Étape 4 : Obtenir les informations d'identification de votre compte Azure Cosmos

L’exemple d’application doit s’authentifier auprès de votre compte Azure Cosmos. Pour vous authentifier, vous devez transmettre les informations d’identification du compte Azure Cosmos à l’application. Pour obtenir les informations d’identification de votre compte Azure Cosmos, procédez comme suit :

1.  Connectez-vous au [portail Azure](https://portal.azure.com/).
1.  Accédez à votre compte Azure Cosmos.
1.  Ouvrez le volet **Clés** et copiez l’**URI** et la **CLÉ PRIMAIRE** de votre compte.

Si vous utilisez l’émulateur Azure Cosmos DB, obtenez les [informations d’identification à partir de cet article](local-emulator.md#authenticate-requests).

## <a name="step-5-initialize-the-cosmosclient-object-with-bulk-execution-support"></a>Étape 5 : Initialiser l’objet CosmosClient avec prise en charge de l’exécution en bloc

Ouvrez le fichier `Program.cs` généré dans un éditeur de code. Vous allez créer une nouvelle instance CosmosClient avec l’exécution en bloc activée et l’utiliser pour effectuer des opérations sur Azure Cosmos DB. 

Commencez par remplacer la méthode `Main` par défaut et définir les variables globales. Ces variables globales incluront le point de terminaison, les clés d’autorisation, le nom de la base de données, le conteneur que vous créez ainsi que le nombre d’éléments que vous insérez en bloc. Veillez à remplacer les valeurs correspondant à endpointURL et aux clés d’autorisation en fonction de votre environnement. 


   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Diagnostics;
   using System.IO;
   using System.Text.Json;
   using System.Threading.Tasks;
   using Microsoft.Azure.Cosmos;

   public class Program
   {
        private const string EndpointUrl = "https://<your-account>.documents.azure.com:443/";
        private const string AuthorizationKey = "<your-account-key>";
        private const string DatabaseName = "bulk-tutorial";
        private const string ContainerName = "items";
        private const int ItemsToInsert = 300000;

        static async Task Main(string[] args)
        {

        }
   }
   ```

Dans la méthode `Main`, ajoutez le code suivant pour initialiser l’objet CosmosClient :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=CreateClient)]

Une fois l’exécution en bloc activée, CosmosClient regroupe en interne les opérations simultanées en appels de service unique. Ainsi, il optimise l’utilisation du débit en distribuant les appels de service entre les partitions, puis en attribuant les résultats individuels aux appelants d’origine.

Vous pouvez ensuite créer un conteneur pour stocker tous les éléments.  Définissez `/pk` en tant que clé de partition, 50 000 RU/s en tant que débit approvisionné, et une stratégie d’indexation personnalisée qui exclut tous les champs afin d'optimiser le débit d’écriture. Ajoutez le code suivant après l’instruction d’initialisation CosmosClient :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Initialize)]

## <a name="step-6-populate-a-list-of-concurrent-tasks"></a>Étape 6 : Remplir une liste de tâches simultanées

Pour bénéficier de la prise en charge de l’exécution en bloc, créez une liste de tâches asynchrones en fonction de la source de données et des opérations que vous souhaitez effectuer, puis utilisez `Task.WhenAll` pour les exécuter simultanément.
Commencez par utiliser des données « fictives » pour générer une liste d’éléments à partir du modèle de données. Dans une application réelle, les éléments proviennent de la source de données souhaitée.

Tout d’abord, ajoutez le package fictif à la solution à l’aide de la commande dotnet add package.

   ```bash
   dotnet add package Bogus
   ```

Définissez les éléments que vous souhaitez enregistrer. Vous devez définir la classe `Item` dans le fichier `Program.cs` :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Model)]

Ensuite, créez une fonction d’assistance au sein la classe `Program`. Cette fonction d’assistance obtient le nombre d’éléments que vous avez définis à des fins d'insertion et génère des données aléatoires :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Bogus)]

Lisez les éléments et sérialisez-les dans des instances de flux à l’aide de la classe `System.Text.Json`. Compte tenu de la nature des données générées automatiquement, vous sérialisez les données sous forme de flux. Vous pouvez aussi utiliser directement l’instance d’élément, mais en les convertissant en flux, tirez pleinement parti des performances des API de flux CosmosClient. En règle générale, vous pouvez utiliser les données directement, sous réserve de connaître la clé de partition. 


Pour convertir les données en instances de flux, dans la méthode `Main`, ajoutez le code suivant juste après avoir créé le conteneur :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=Operations)]

Utilisez ensuite les flux de données pour créer des tâches simultanées et remplissez la liste des tâches afin d'insérer les éléments dans le conteneur. Pour effectuer cette opération, ajoutez le code suivant à la classe `Program` :

[!code-csharp[Main](~/cosmos-dotnet-bulk-import/src/Program.cs?name=ConcurrentTasks)]

Toutes ces opérations de point simultanées sont exécutées ensemble (en bloc), comme décrit dans la section Introduction.

## <a name="step-7-run-the-sample"></a>Étape 7 : Exécution de l'exemple

Pour exécuter l’exemple, vous pouvez tout simplement utiliser la commande `dotnet` :

   ```bash
   dotnet run
   ```

## <a name="get-the-complete-sample"></a>Obtenir l'exemple complet

Si vous n’avez pas le temps de suivre les étapes de ce didacticiel, ou que vous voulez simplement télécharger les exemples de code, vous pouvez les obtenir à partir de [GitHub](https://github.com/Azure-Samples/cosmos-dotnet-bulk-import-throughput-optimizer).

Après avoir cloné le projet, assurez-vous de mettre à jour les informations d’identification souhaitées dans [Program.cs](https://github.com/Azure-Samples/cosmos-dotnet-bulk-import-throughput-optimizer/blob/main/src/Program.cs#L25).

Pour exécuter l'exemple, vous utilisez le répertoire du référentiel et `dotnet` :

   ```bash
   cd cosmos-dotnet-bulk-import-throughput-optimizer
   dotnet run
   ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez effectué les tâches suivantes :

> [!div class="checklist"]
> * Création d’un compte Azure Cosmos
> * Configuration de votre projet
> * Connexion à un compte Azure Cosmos avec prise en charge de l'exécution en bloc activée
> * Effectuer une importation de données via des opérations de création simultanées

Vous pouvez maintenant passer au tutoriel suivant :

> [!div class="nextstepaction"]
>[Interroger Azure Cosmos DB à l’aide de l’API SQL](tutorial-query-sql-api.md)