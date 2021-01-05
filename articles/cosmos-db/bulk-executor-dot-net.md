---
title: Utiliser la bibliothèque .NET Bulk Executor dans Azure Cosmos DB pour les opérations d’importation et de mise à jour en bloc
description: Découvrez comment importer et mettre à jour en bloc les documents dans Azure Cosmos DB à l’aide de la bibliothèque .NET de l’exécuteur en bloc.
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: how-to
ms.date: 03/23/2020
ms.author: ramkris
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 6f6994717ff4c730fb27bd26c40d199fb198e528
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96019954"
---
# <a name="use-the-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db"></a>Utiliser la bibliothèque d’exécuteur en bloc .NET pour effectuer des opérations en bloc dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!NOTE]
> La bibliothèque d’exécuteur en bloc décrite dans cet article est conservée pour les applications qui utilisent le Kit de développement logiciel (SDK) .NET version 2.x. Pour les nouvelles applications, vous pouvez utiliser la **prise en charge de l’exécution en bloc** qui est directement disponible avec le [Kit de développement logiciel (SDK) .NET version 3.x](tutorial-sql-api-dotnet-bulk-import.md) et ne nécessite aucune bibliothèque externe. 

> Si vous utilisez actuellement la bibliothèque d’exécuteur en bloc et envisagez une migration vers la prise en charge de l’exécution en bloc sur le nouveau SDK, suivez les étapes décrites dans le [Guide de migration](how-to-migrate-from-bulk-executor-library.md) pour migrer votre application.

Ce tutoriel fournit des instructions sur l’utilisation de la bibliothèque de l’exécuteur en bloc .NET pour importer et mettre à jour des documents vers un conteneur Azure Cosmos. Pour en savoir plus sur la bibliothèque de l’exécuteur en bloc et sur la façon dont elle vous aide à profiter d’un débit et d’un stockage conséquents, consultez l’article [Vue d’ensemble de la bibliothèque de l’exécuteur en bloc](bulk-executor-overview.md). Dans ce tutoriel, vous voyez un exemple d’application .NET qui importe en bloc des documents, générés de manière aléatoire, dans un conteneur Azure Cosmos. Après l’importation, il illustre comment mettre à jour en bloc les données importées en spécifiant des correctifs comme opérations à effectuer sur des champs de documents spécifiques.

Actuellement, la bibliothèque de l’exécuteur en bloc est prise en charge uniquement par les comptes d’API Gremlin et d’API SQL Azure Cosmos DB. Cet article décrit comment utiliser la bibliothèque .NET de l’exécuteur en bloc avec des comptes d’API SQL. Pour en savoir plus sur l’utilisation de la bibliothèque .NET de l’exécuteur en bloc avec les comptes d’API Gremlin, consultez [Effectuer des opérations en bloc dans l’API Gremlin Azure Cosmos DB](bulk-executor-graph-dotnet.md).

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas encore installé Visual Studio 2019, vous pouvez télécharger et utiliser [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Veillez à activer le « développement Azure » lors de l’installation de Visual Studio.

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

* Vous pouvez [essayer Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure, libre de tous frais et engagements. Vous pouvez également utiliser l’[émulateur Azure Cosmos DB](./local-emulator.md) avec le point de terminaison `https://localhost:8081`. La clé primaire est fournie dans des [requêtes d’authentification](local-emulator.md#authenticate-requests).

* Créez un compte d’API SQL Azure Cosmos DB en suivant les étapes décrites dans la section [Créer un compte de base de données](create-sql-api-dotnet.md#create-account) de l’article de démarrage rapide .NET.

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Nous allons maintenant passer à l’utilisation de code en téléchargeant un exemple d’application .NET à partir de GitHub. Cette application effectue des opérations en bloc sur les données stockées dans votre compte Azure Cosmos. Pour cloner l’application, ouvrez une invite de commandes, accédez au répertoire où vous souhaitez la copier, puis exécutez la commande suivante :

```bash
git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started.git
```

Le dépôt cloné contient deux exemples : « BulkImportSample » et « BulkUpdateSample ». Vous pouvez ouvrir l’un ou l’autre exemple d’application, mettre à jour les chaînes de connexion dans le fichier App.config avec les chaînes de connexion de votre compte Azure Cosmos DB, générer la solution et l’exécuter.

L’application « BulkImportSample » génère des documents aléatoires et les importe en bloc dans votre compte Azure Cosmos. L’application « BulkUpdateSample » met à jour en bloc les documents importés en spécifiant des correctifs comme opérations à effectuer sur des champs de documents spécifiques. Dans les sections suivantes, nous allons examiner le code de chacun de ces exemples d’applications.

## <a name="bulk-import-data-to-an-azure-cosmos-account"></a>Importer des données en bloc dans un compte Azure Cosmos

1. Accédez au dossier « BulkImportSample » et ouvrez le fichier « BulkImportSample.sln ».  

2. Les chaînes de connexion d’Azure Cosmos DB sont récupérées à partir du fichier App.config, comme indiqué dans le code suivant :  

   ```csharp
   private static readonly string EndpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
   private static readonly string AuthorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];
   private static readonly string DatabaseName = ConfigurationManager.AppSettings["DatabaseName"];
   private static readonly string CollectionName = ConfigurationManager.AppSettings["CollectionName"];
   private static readonly int CollectionThroughput = int.Parse(ConfigurationManager.AppSettings["CollectionThroughput"]);
   ```

   L’importateur en bloc crée une base de données et un conteneur avec le nom de la base de données, le nom du conteneur et les valeurs de débit spécifiées dans le fichier App.config.

3. Ensuite, l’objet DocumentClient est initialisé avec le mode de connexion TCP direct :  

   ```csharp
   ConnectionPolicy connectionPolicy = new ConnectionPolicy
   {
      ConnectionMode = ConnectionMode.Direct,
      ConnectionProtocol = Protocol.Tcp
   };
   DocumentClient client = new DocumentClient(new Uri(endpointUrl),authorizationKey,
   connectionPolicy)
   ```

4. L’objet BulkExecutor est initialisé avec une valeur de nouvelle tentative élevée pour la durée d’attente et les requêtes limitées. Ensuite, elles sont définies sur 0 pour passer le contrôle de congestion à BulkExecutor pendant sa durée de vie.  

   ```csharp
   // Set retry options high during initialization (default values).
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 30;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 9;

   IBulkExecutor bulkExecutor = new BulkExecutor(client, dataCollection);
   await bulkExecutor.InitializeAsync();

   // Set retries to 0 to pass complete control to bulk executor.
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 0;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 0;
   ```

5. L’application appelle l’API BulkImportAsync. La bibliothèque .NET fournit deux surcharges de l’API d’importation en bloc : une qui accepte une liste de documents JSON sérialisés et une autre qui accepte une liste de documents POCO désérialisés. Pour en savoir plus sur les définitions de chacune de ces méthodes surchargées, consultez la [documentation de l’API](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkimportasync?view=azure-dotnet&preserve-view=true).

   ```csharp
   BulkImportResponse bulkImportResponse = await bulkExecutor.BulkImportAsync(
     documents: documentsToImportInBatch,
     enableUpsert: true,
     disableAutomaticIdGeneration: true,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```
   **La méthode BulkImportAsync accepte les paramètres suivants :**
   
   |**Paramètre**  |**Description** |
   |---------|---------|
   |enableUpsert    |   Indicateur permettant d’activer les opérations upsert sur les documents. Si un document avec l’ID donné existe déjà, il est mis à jour. Par défaut, cet indicateur a la valeur false.      |
   |disableAutomaticIdGeneration    |    Indicateur permettant de désactiver la génération automatique d’ID. Par défaut, il est défini sur true.     |
   |maxConcurrencyPerPartitionKeyRange    | Degré maximal de concurrence par plage de clés de partition. Si vous lui affectez la valeur null, la bibliothèque utilise la valeur par défaut 20. |
   |maxInMemorySortingBatchSize     |  Nombre maximal de documents qui sont extraits à partir de l’énumérateur de documents, qui est passé à l’appel d’API à chaque étape. Pour la phase de tri en mémoire qui se produit avant l’importation en bloc, si vous affectez à ce paramètre la valeur null, la bibliothèque utilise la valeur minimale par défaut (documents.count, 1000000).       |
   |cancellationToken    |    Jeton d’annulation permettant de quitter normalement l’opération d’importation en bloc.     |

   **Définition de l’objet de réponse d’importation en bloc** Le résultat de l’appel d’API d’importation en bloc contient les attributs suivants :

   |**Paramètre**  |**Description**  |
   |---------|---------|
   |NumberOfDocumentsImported (long)   |  Nombre total de documents qui ont été importés avec succès, sur tous les documents fournis à l’appel d’API d’importation en bloc.       |
   |TotalRequestUnitsConsumed (double)   |   Nombre total d’unités de requête (RU) consommées par l’appel d’API d’importation en bloc.      |
   |TotalTimeTaken (TimeSpan)    |   Temps total nécessaire à l’exécution de l’appel d’API d’importation en bloc.      |
   |BadInputDocuments (List\<object>)   |     Liste de documents au format incorrect qui n’ont pas été importés dans l’appel d’API d’importation en bloc. Corrigez les documents retournés et retentez l’importation. Les documents au format incorrect incluent les documents dont la valeur d’ID n’est pas une chaîne (null ou tout autre type de données est considéré comme non valide).    |

## <a name="bulk-update-data-in-your-azure-cosmos-account"></a>Mettre à jour des données en bloc dans votre compte Azure Cosmos

Vous pouvez mettre à jour des documents existants à l’aide de l’API BulkUpdateAsync. Dans cet exemple, vous allez affecter une nouvelle valeur au champ `Name` et supprimer le champ `Description` des documents existants. Pour connaître l’ensemble complet d’opérations de mise à jour prises en charge, consultez la [documentation de l’API](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet&preserve-view=true).

1. Accédez au dossier « BulkUpdateSample » et ouvrez le fichier « BulkUpdateSample.sln ».  

2. Définissez les éléments de mise à jour ainsi que les opérations de mise à jour de champs correspondantes. Dans cet exemple, vous allez utiliser `SetUpdateOperation` pour mettre à jour le champ `Name` et `UnsetUpdateOperation` pour supprimer le champ `Description` de tous les documents. Vous pouvez également effectuer d’autres opérations comme incrémenter un champ de document d’une valeur spécifique, envoyer des valeurs spécifiques dans un champ de tableau ou supprimer une valeur spécifique d’un champ de tableau. Pour en savoir plus sur les différentes méthodes fournies par l’API de mise à jour en bloc, consultez la [documentation de l’API](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet&preserve-view=true).

   ```csharp
   SetUpdateOperation<string> nameUpdate = new SetUpdateOperation<string>("Name", "UpdatedDoc");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   List<UpdateOperation> updateOperations = new List<UpdateOperation>();
   updateOperations.Add(nameUpdate);
   updateOperations.Add(descriptionUpdate);

   List<UpdateItem> updateItems = new List<UpdateItem>();
   for (int i = 0; i < 10; i++)
   {
    updateItems.Add(new UpdateItem(i.ToString(), i.ToString(), updateOperations));
   }
   ```

3. L’application appelle l’API BulkUpdateAsync. Pour en savoir plus sur la définition de la méthode BulkUpdateAsync, consultez la [documentation de l’API](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.ibulkexecutor.bulkupdateasync?view=azure-dotnet&preserve-view=true).  

   ```csharp
   BulkUpdateResponse bulkUpdateResponse = await bulkExecutor.BulkUpdateAsync(
     updateItems: updateItems,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```  
   **La méthode BulkUpdateAsync accepte les paramètres suivants :**

   |**Paramètre**  |**Description** |
   |---------|---------|
   |maxConcurrencyPerPartitionKeyRange    |   Degré maximal de concurrence par plage de clés de partition. Si vous affectez à ce paramètre la valeur null, la bibliothèque utilise la valeur par défaut (20).   |
   |maxInMemorySortingBatchSize    |    Nombre maximal d’éléments de mise à jour tirés (pull) de l’énumérateur d’éléments de mise à jour transmis à l’appel d’API à chaque étape. Pour la phase de tri en mémoire qui se produit avant la mise à jour en bloc, si vous affectez à ce paramètre la valeur null, la bibliothèque utilise la valeur minimale par défaut (updateItems.count, 1000000).     |
   | cancellationToken|Jeton d’annulation permettant de quitter normalement l’opération de mise à jour en bloc. |

   **Définition de l’objet de réponse de mise à jour en bloc** Le résultat de l’appel d’API de mise à jour en bloc contient les attributs suivants :

   |**Paramètre**  |**Description** |
   |---------|---------|
   |NumberOfDocumentsUpdated (long)    |   Nombre de documents qui ont été mis à jour avec succès, sur tous les documents fournis à l’appel d’API de mise à jour en bloc.      |
   |TotalRequestUnitsConsumed (double)   |    Nombre total d’unités de requête (RU) consommées par l’appel d’API de mise à jour en bloc.    |
   |TotalTimeTaken (TimeSpan)   | Temps total nécessaire à l’exécution de l’appel d’API de mise à jour en bloc. |
    
## <a name="performance-tips"></a>Conseils sur les performances 

Pour bénéficier de meilleures performances lors de l’utilisation de la bibliothèque de l’exécuteur en bloc, considérez les points suivants :

* Pour des performances optimales, exécutez votre application à partir d’une machine virtuelle Azure qui se trouve dans la région d’écriture de votre compte Azure Cosmos.  

* Nous vous recommandons d’instancier un objet `BulkExecutor` unique pour l’ensemble de l’application au sein d’une seule machine virtuelle qui correspond à un conteneur Azure Cosmos spécifique.  

* L’exécution d’une API d’opération en bloc consomme une grande partie des E/S réseau et du processeur de l’ordinateur client (en raison de la génération de plusieurs tâches en interne). Évitez de générer au sein du processus de votre application plusieurs tâches simultanées qui exécutent des appels d’API d’opération en bloc. Si un appel d’API d’opération en bloc en cours d’exécution sur une seule machine virtuelle ne peut pas consommer le débit complet du conteneur (si le débit de votre conteneur est supérieur à 1 million RU/s), il est préférable de créer des machines virtuelles distinctes pour exécuter simultanément les appels d’API d’opérations en bloc.  

* Vérifiez que la méthode `InitializeAsync()` est appelée après l’instanciation d’un objet BulkExecutor pour extraire le mappage de partition du conteneur Cosmos cible.  

* Pour obtenir de meilleures performances, vérifiez que **gcServer** est activé dans le fichier App.Config de votre application.
  ```xml  
  <runtime>
    <gcServer enabled="true" />
  </runtime>
  ```
* La bibliothèque émet des traces qui peuvent être collectées dans un fichier journal ou sur la console. Pour activer ces deux méthodes de collecte, ajoutez le code suivant au fichier App.Config de votre application.

  ```xml
  <system.diagnostics>
    <trace autoflush="false" indentsize="4">
      <listeners>
        <add name="logListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="application.log" />
        <add name="consoleListener" type="System.Diagnostics.ConsoleTraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
  ```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les packages NuGet et les notes de publication, consultez les [détails sur le Kit de développement logiciel (SDK) de l’exécuteur en bloc](sql-api-sdk-bulk-executor-dot-net.md).
