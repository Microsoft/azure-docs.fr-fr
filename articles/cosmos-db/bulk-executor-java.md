---
title: Utiliser la bibliothèque Java de l’exécuteur en bloc dans Azure Cosmos DB pour effectuer des importations et mises à jour en bloc
description: Découvrez comment importer et mettre à jour en bloc des documents dans Azure Cosmos DB à l’aide de la bibliothèque Java de l’exécuteur en bloc.
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 08/26/2020
ms.author: ramkris
ms.reviewer: sngun
ms.custom: devx-track-java
ms.openlocfilehash: 89d21e4464cb3c7578b68d68009065ab7848ed19
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092532"
---
# <a name="use-bulk-executor-java-library-to-perform-bulk-operations-on-azure-cosmos-db-data"></a>Utiliser la bibliothèque Java de l’exécuteur en bloc pour effectuer des opérations en bloc sur les données Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ce tutoriel fournit des instructions sur l’utilisation de la bibliothèque Java BulkExecutor d’Azure Cosmos DB pour importer et mettre à jour des documents Azure Cosmos DB. Pour en savoir plus sur la bibliothèque de l’exécuteur en bloc et sur la façon dont elle vous aide à profiter d’un débit et d’un stockage conséquents, consultez l’article [Vue d’ensemble de la bibliothèque BulkExecutor](bulk-executor-overview.md). Dans ce tutoriel, vous allez créer une application Java qui génère des documents aléatoires, qui sont ensuite importés en bloc dans un conteneur Azure Cosmos. Après l’importation, vous mettrez à jour en bloc certaines propriétés d’un document. 

Actuellement, la bibliothèque de l’exécuteur en bloc est prise en charge uniquement par les comptes d’API Gremlin et d’API SQL Azure Cosmos DB. Cet article décrit comment utiliser la bibliothèque Java de l’exécuteur en bloc avec des comptes d’API SQL. Pour en savoir plus sur l’utilisation de la bibliothèque .NET de l’exécuteur en bloc avec l’API Gremlin, consultez [Effectuer des opérations en bloc dans l’API Gremlin Azure Cosmos DB](bulk-executor-graph-dotnet.md). La bibliothèque d’exécuteur en bloc décrite est disponible uniquement pour le [kit de développement logiciel (SDK) Azure Cosmos DB de synchronisation Java v2](sql-api-sdk-java.md) et est la solution actuellement recommandée pour la prise en charge en bloc Java. Elle n’est actuellement pas disponible pour les versions 3.x, 4.x ou d’autres versions ultérieures du kit de développement logiciel (SDK).

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.  

* Vous pouvez [essayer Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure, libre de tous frais et engagements. Vous pouvez également utiliser l'[émulateur Azure Cosmos DB](./local-emulator.md) avec le point de terminaison `https://localhost:8081`. La clé primaire est fournie dans des [requêtes d’authentification](local-emulator.md#authenticate-requests).  

* [Java Development Kit (JDK) 1.7+](/java/azure/jdk/?view=azure-java-stable&preserve-view=true)  
  - Sur Ubuntu, exécutez `apt-get install default-jdk` pour installer le JDK.  

  - Veillez à définir la variable d’environnement JAVA_HOME pour qu’elle pointe vers le dossier dans lequel le JDK est installé.

* [Téléchargement](https://maven.apache.org/download.cgi) et [installation](https://maven.apache.org/install.html) d’une archive binaire [Maven](https://maven.apache.org/)  
  
  - Sur Ubuntu, vous pouvez exécuter `apt-get install maven` pour installer Maven.

* Créez un compte d’API SQL Azure Cosmos DB en suivant les étapes décrites dans la section [Créer un compte de base de données](create-sql-api-java.md#create-a-database-account) de l’article de démarrage rapide Java.

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Nous allons maintenant passer à l’utilisation de code en téléchargeant un exemple d’application Java à partir de GitHub. Cette application effectue des opérations en bloc sur des données Azure Cosmos DB. Pour cloner l’application, ouvrez une invite de commandes, accédez au répertoire où vous souhaitez la copier, puis exécutez la commande suivante :

```bash
 git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-java-getting-started.git 
```

Le référentiel cloné contient deux exemples, « bulkimport » et « bulkupdate », relatifs au dossier « \azure-cosmosdb-bulkexecutor-java-getting-started\samples\bulkexecutor-sample\src\main\java\com\microsoft\azure\cosmosdb\bulkexecutor ». L’application « bulkimport » génère des documents aléatoires et les importe dans Azure Cosmos DB. L’application « bulkupdate » met à jour certains documents dans Azure Cosmos DB. Dans les sections suivantes, nous allons examiner le code de chacun de ces exemples d’applications. 

## <a name="bulk-import-data-to-azure-cosmos-db"></a>Importer en bloc des données dans Azure Cosmos DB

1. Les chaînes de connexion d’Azure Cosmos DB sont lues en tant qu’arguments, et affectées à des variables définies dans le fichier CmdLineConfiguration.java.  

2. Ensuite, l’objet DocumentClient est initialisé à l’aide des instructions suivantes :  

   ```java
   ConnectionPolicy connectionPolicy = new ConnectionPolicy();
   connectionPolicy.setMaxPoolSize(1000);
   DocumentClient client = new DocumentClient(
      HOST,
      MASTER_KEY, 
      connectionPolicy,
      ConsistencyLevel.Session)
   ```

3. L’objet DocumentBulkExecutor est initialisé avec des valeurs de nouvelle tentative élevées pour la durée d’attente et les requêtes limitées. Ensuite, elles sont définies sur 0 pour passer le contrôle de congestion à DocumentBulkExecutor pendant sa durée de vie.  

   ```java
   // Set client's retry options high for initialization
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(30);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(9);

   // Builder pattern
   Builder bulkExecutorBuilder = DocumentBulkExecutor.builder().from(
     client,
     DATABASE_NAME,
     COLLECTION_NAME,
     collection.getPartitionKey(),
     offerThroughput) // throughput you want to allocate for bulk import out of the container's total throughput

   // Instantiate DocumentBulkExecutor
   DocumentBulkExecutor bulkExecutor = bulkExecutorBuilder.build()

   // Set retries to 0 to pass complete control to bulk executor
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(0);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(0);
   ```

4. Appelez l’API importAll qui génère des documents aléatoires à importer en bloc dans un conteneur Azure Cosmos. Vous pouvez configurer les configurations de la ligne de commande dans le fichier CmdLineConfiguration.java.

   ```java
   BulkImportResponse bulkImportResponse = bulkExecutor.importAll(documents, false, true, null);
   ```
   L’API d’importation en bloc accepte une collection de documents JSON sérialisés, et sa syntaxe est la suivante (pour plus d’informations, consultez la [documentation de l’API](/java/api/com.microsoft.azure.documentdb.bulkexecutor)) :

   ```java
   public BulkImportResponse importAll(
        Collection<String> documents,
        boolean isUpsert,
        boolean disableAutomaticIdGeneration,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;   
   ```

   La méthode importAll accepte les paramètres suivants :
 
   |**Paramètre**  |**Description**  |
   |---------|---------|
   |IsUpsert    |   Indicateur permettant d’activer les opérations d’upsert des documents. Si un document avec l’ID donné existe déjà, il est mis à jour.  |
   |disableAutomaticIdGeneration     |   Indicateur permettant de désactiver la génération automatique d’ID. Par défaut, il est défini sur true.   |
   |maxConcurrencyPerPartitionRange    |  Degré maximal de concurrence par plage de clés de partition. La valeur par défaut est 20.  |

   **Définition de l’objet de réponse d’importation en bloc** Le résultat de l’appel d’API d’importation en bloc contient les méthodes get suivantes :

   |**Paramètre**  |**Description**  |
   |---------|---------|
   |int getNumberOfDocumentsImported()  |   Nombre total de documents qui ont été importés avec succès, sur tous les documents fournis à l’appel d’API d’importation en bloc.      |
   |double getTotalRequestUnitsConsumed()   |  Nombre total d’unités de requête (RU) consommées par l’appel d’API d’importation en bloc.       |
   |Duration getTotalTimeTaken()   |    Temps total nécessaire à l’exécution de l’appel d’API d’importation en bloc.     |
   |List\<Exception> getErrors() |  Obtient la liste des erreurs si certains documents du lot fourni à l’appel d’API d’importation en bloc n’ont pas pu être insérés.       |
   |List\<Object> getBadInputDocuments()  |    Liste de documents au format incorrect qui n’ont pas été importés dans l’appel d’API d’importation en bloc. L’utilisateur doit corriger les documents retournés et retenter l’importation. Les documents au format incorrect incluent les documents dont la valeur d’ID n’est pas une chaîne (null ou tout autre type de données est considéré comme non valide).     |

5. Une fois l’application d’importation en bloc prête, générez l’outil en ligne de commande à partir de la source à l’aide de la commande « mvn clean package ». Cette commande génère un fichier jar dans le dossier cible :  

   ```bash
   mvn clean package
   ```

6. Une fois les dépendances cibles générées, vous pouvez appeler l’application d’importation en bloc à l’aide de la commande suivante :  

   ```bash
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint *<Fill in your Azure Cosmos DB's endpoint>*  -masterKey *<Fill in your Azure Cosmos DB's primary key>* -databaseId bulkImportDb -collectionId bulkImportColl -operation import -shouldCreateCollection -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

   L’importateur en bloc crée une base de données et une collection avec le nom de la base de données, le nom de la collection et les valeurs de débit spécifiées dans le fichier App.config. 

## <a name="bulk-update-data-in-azure-cosmos-db"></a>Mettre à jour des données en bloc dans Azure Cosmos DB

Vous pouvez mettre à jour des documents existants à l’aide de l’API BulkUpdateAsync. Dans cet exemple, vous allez affecter une nouvelle valeur au champ Name et supprimer le champ Description des documents existants. Pour connaître l’ensemble complet d’opérations de mise à jour de champs prises en charge, consultez la [documentation de l’API](/java/api/com.microsoft.azure.documentdb.bulkexecutor). 

1. Définissez les éléments de mise à jour ainsi que les opérations de mise à jour de champs correspondantes. Dans cet exemple, nous allons utiliser SetUpdateOperation pour mettre à jour le champ Name et UnsetUpdateOperation pour supprimer le champ Description de tous les documents. Vous pouvez également effectuer d’autres opérations comme incrémenter un champ de document d’une valeur spécifique, envoyer des valeurs spécifiques dans un champ de tableau ou supprimer une valeur spécifique d’un champ de tableau. Pour en savoir plus sur les différentes méthodes fournies par l’API de mise à jour en bloc, consultez la [documentation de l’API](/java/api/com.microsoft.azure.documentdb.bulkexecutor).  

   ```java
   SetUpdateOperation<String> nameUpdate = new SetUpdateOperation<>("Name","UpdatedDocValue");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   ArrayList<UpdateOperationBase> updateOperations = new ArrayList<>();
   updateOperations.add(nameUpdate);
   updateOperations.add(descriptionUpdate);

   List<UpdateItem> updateItems = new ArrayList<>(cfg.getNumberOfDocumentsForEachCheckpoint());
   IntStream.range(0, cfg.getNumberOfDocumentsForEachCheckpoint()).mapToObj(j -> {                        
    return new UpdateItem(Long.toString(prefix + j), Long.toString(prefix + j), updateOperations);
    }).collect(Collectors.toCollection(() -> updateItems));
   ```

2. Appelez l’API updateAll qui génère des documents aléatoires à importer en bloc dans un conteneur Azure Cosmos. Vous pouvez configurer les configurations de la ligne de commande à passer dans le fichier CmdLineConfiguration.java.

   ```java
   BulkUpdateResponse bulkUpdateResponse = bulkExecutor.updateAll(updateItems, null)
   ```

   L’API de mise à jour en bloc accepte une collection d’éléments à mettre à jour. Chaque élément de mise à jour spécifie la liste des opérations de mise à jour de champs à effectuer sur un document identifié par un ID et une valeur de clé de partition. Pour plus d’informations, consultez la [documentation de l’API](/java/api/com.microsoft.azure.documentdb.bulkexecutor) :

   ```java
   public BulkUpdateResponse updateAll(
        Collection<UpdateItem> updateItems,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;
   ```

   La méthode updateAll accepte les paramètres suivants :

   |**Paramètre** |**Description** |
   |---------|---------|
   |maxConcurrencyPerPartitionRange   |  Degré maximal de concurrence par plage de clés de partition. La valeur par défaut est 20.  |
 
   **Définition de l’objet de réponse d’importation en bloc** Le résultat de l’appel d’API d’importation en bloc contient les méthodes get suivantes :

   |**Paramètre** |**Description**  |
   |---------|---------|
   |int getNumberOfDocumentsUpdated()  |   Nombre total de documents qui ont été mis à jour avec succès, sur tous les documents fournis à l’appel d’API de mise à jour en bloc.      |
   |double getTotalRequestUnitsConsumed() |  Nombre total d’unités de requête (RU) consommées par l’appel d’API de mise à jour en bloc.       |
   |Duration getTotalTimeTaken()  |   Temps total nécessaire à l’exécution de l’appel d’API de mise à jour en bloc.      |
   |List\<Exception> getErrors()   |       Obtient la liste des problèmes opérationnels ou de réseau liés à l’opération de mise à jour.      |
   |List\<BulkUpdateFailure> getFailedUpdates()   |       Obtient la liste des mises à jour qui n’ont pas pu être effectuées, ainsi que les exceptions spécifiques menant aux échecs.|

3. Une fois l’application de mise à jour en bloc prête, générez l’outil en ligne de commande à partir de la source à l’aide de la commande « mvn clean package ». Cette commande génère un fichier jar dans le dossier cible :  

   ```bash
   mvn clean package
   ```

4. Une fois les dépendances cibles générées, vous pouvez appeler l’application de mise à jour en bloc à l’aide de la commande suivante :

   ```bash
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint **<Fill in your Azure Cosmos DB's endpoint>* -masterKey **<Fill in your Azure Cosmos DB's primary key>* -databaseId bulkUpdateDb -collectionId bulkUpdateColl -operation update -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

## <a name="performance-tips"></a>Conseils sur les performances 

Pour bénéficier de meilleures performances lors de l’utilisation de la bibliothèque de l’exécuteur en bloc, considérez les points suivants :

* Pour de meilleures performances, exécutez votre application à partir d’une machine virtuelle Azure qui se trouve dans la région d’écriture du compte Cosmos DB.  
* Pour atteindre un débit plus élevé :  

   * Affectez à la taille du tas de la machine virtuelle Java une valeur suffisamment élevée pour éviter tout problème de mémoire lors du traitement d’un grand nombre de documents. Suggestion de taille de tas : max(3GB, 3 * sizeof(tous les documents transmis à l’API d’importation en bloc dans un lot)).  
   * Il y a un temps de prétraitement, grâce auquel vous obtiendrez un débit supérieur lors de l’exécution d’opérations en bloc avec un grand nombre de documents. Si vous souhaitez importer 10 000 000 documents, il est préférable d’exécuter une importation en bloc 10 fois sur 10 lots de documents en contenant chacun 1 000 000, plutôt que d’exécuter une importation en bloc 100 fois sur 100 lots de documents en contenant chacun 100 000.  

* Nous vous recommandons d’instancier un objet DocumentBulkExecutor unique pour l’ensemble de l’application au sein d’une seule machine virtuelle qui correspond à un conteneur Azure Cosmos spécifique.  

* L’exécution d’une API d’opération en bloc consomme une grande partie des E/S réseau et du processeur de l’ordinateur client. Cela est dû à la génération automatique de plusieurs tâches en interne. Évitez de générer plusieurs tâches simultanées dans votre processus d’application, exécutant chacune des appels d’API d’opérations en bloc. Si un appel d’API d’opération en bloc en cours d’exécution sur une seule machine virtuelle ne peut pas consommer le débit complet de votre conteneur (si le débit de votre conteneur est supérieur à 1 million RU/s), il est préférable de créer des machines virtuelles distinctes pour exécuter simultanément les appels d’API d’opérations en bloc.

    
## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur les packages maven et les notes de publication de la bibliothèque Java de l’exécuteur en bloc, consultez les [détails sur le SDK BulkExecutor](sql-api-sdk-bulk-executor-java.md).