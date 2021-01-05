---
title: Migrer des données existantes vers un compte API Table dans Azure Cosmos DB
description: Apprenez à migrer ou importer des données locales ou cloud vers un compte d’API Table Azure dans Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 12/07/2017
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: e876ca028532bb3721146e90a91d68c4c12bf79f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096075"
---
# <a name="migrate-your-data-to-azure-cosmos-db-table-api-account"></a>Migrer vos données vers un compte d’API Table Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Ce didacticiel fournit des instructions sur l’importation de données à utiliser avec [l’API Table](table-introduction.md) Azure Cosmos DB. Si vous avez des données stockées dans le stockage Table Azure, vous pouvez utiliser l’outil de migration de données ou AzCopy pour importer vos données dans l’API Table Azure Cosmos DB. Si vous avez des données stockées dans un compte d’API Table Azure Cosmos DB (préversion), vous devez utiliser l’outil de migration de données pour la migration. 

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Importation de données avec l’outil de migration de données
> * Importation de données avec AzCopy
> * Migration depuis une API Table (préversion) vers une autre 

## <a name="prerequisites"></a>Prérequis

* **Augmenter le débit :** la durée de la migration de vos données dépend du débit que vous définissez pour un conteneur spécifique ou un ensemble de conteneurs. Veillez à augmenter le débit pour les migrations de données plus importantes. Une fois que vous avez effectué la migration, diminuez le débit pour réduire les coûts. Pour plus d’informations sur l’augmentation du débit dans le portail Azure, consultez les niveaux de performances et niveaux tarifaires dans Azure Cosmos DB.

* **Créer des ressources Azure Cosmos DB :** avant de commencer à migrer des données, créez au préalable toutes vos tables à partir du portail Azure. Si vous migrez vers un compte Azure Cosmos DB qui a un débit de niveau base de données, assurez-vous de fournir une clé de partition lors de la création de tables Azure Cosmos DB.

## <a name="data-migration-tool"></a>Outil de migration de données

L’outil de migration des données de données Azure Cosmos DB en ligne de commande (dt.exe) peut être utilisé pour importer vos données de stockage de table Azure existante vers un compte GA d’API Table, ou migrer les données depuis un compte d’API Table (préversion) vers un compte GA d’API Table. Actuellement, les autres sources ne sont pas prises en charge. L’outil de migration des données basé sur une interface utilisateur (dtui.exe) n’est pas pris en charge pour les comptes API Table. 

Pour migrer des données de table, effectuez les tâches suivantes :

1. Téléchargez l’outil de migration à partir de [GitHub](https://github.com/azure/azure-documentdb-datamigrationtool).
2. Exécutez `dt.exe` en utilisant les arguments de ligne de commande de votre scénario. `dt.exe` prend une commande au format suivant :

   ```bash
    dt.exe [/<option>:<value>] /s:<source-name> [/s.<source-option>:<value>] /t:<target-name> [/t.<target-option>:<value>] 
   ```

Les options prises en charge pour cette commande sont les suivantes :

* **/ErrorLog :** facultatif. Nom du fichier CSV pour rediriger les échecs de transfert de données
* **/OverwriteErrorLog :** facultatif. Remplacer le fichier journal des erreurs
* **/ProgressUpdateInterval :** facultatif. La valeur par défaut est 00:00:01. Intervalle de temps pour l’actualisation de la progression du transfert des données à l’écran
* **/ErrorDetails :** facultatif. La valeur par défaut est None (Aucun). Spécifie que des informations détaillées sur l’erreur doivent être affichées pour les erreurs suivantes : None (Aucun), Critical (Critique), All (Tout)
* **/EnableCosmosTableLog :** facultatif. Dirigez le journal vers un compte de table Cosmos. Si cette option est définie, la chaîne de connexion du compte de destination est définie par défaut, sauf si /CosmosTableLogConnectionString est également fournie. Cela est utile si plusieurs instances de DT sont exécutées simultanément.
* **/CosmosTableLogConnectionString :** facultatif. Chaîne de connexion (ConnectionString) pour diriger le journal vers un compte de table Cosmos distante.

### <a name="command-line-source-settings"></a>Paramètres de la source de ligne de commande

Utilisez les options de source suivantes lorsque vous définissez le Stockage Table Azure ou la préversion de l’API Table en tant que source de la migration.

* **/s:AzureTable :** Lit des données à partir du Stockage Table Azure
* **/s.ConnectionString :** chaîne de connexion pour le point de terminaison de table. Elle peut être récupérée à partir du portail Azure
* **/s.LocationMode :** facultatif. La valeur par défaut est PrimaryOnly. Spécifie le mode d’emplacement à utiliser lors de la connexion au Stockage Table Azure : PrimaryOnly, PrimaryThenSecondary, SecondaryOnly, SecondaryThenPrimary
* **/s.Table :** Nom de la table Azure
* **/s.InternalFields :** définir sur All (Tout) pour la migration de table, car RowKey et PartitionKey sont obligatoires pour l’importation.
* **/s.Filter :** facultatif. Chaîne de filtrage à appliquer
* **/s.Projection :** facultatif. Liste des colonnes à sélectionner

Pour récupérer la chaîne de connexion source lors de l’importation à partir du Stockage Table Azure, ouvrez le portail Azure, puis cliquez sur **Comptes de stockage** > **Compte** > **Clés d’accès** , puis utilisez le bouton Copier pour copier la **chaîne de connexion**.

:::image type="content" source="./media/table-import/storage-table-access-key.png" alt-text="Capture d’écran affichant les options Comptes de stockage > Compte > Clés d’accès avec la mise en évidence du bouton Copier.":::

Pour récupérer la chaîne de connexion source lors de l’importation à partir du compte d’API Table Azure Cosmos DB (préversion), ouvrez le portail Azure, puis cliquez sur **Azure Cosmos DB** > **Chaîne de connexion** > **Clés d’accès** , puis utilisez le bouton Copier pour copier la **chaîne de connexion**.

:::image type="content" source="./media/table-import/cosmos-connection-string.png" alt-text="Capture d’écran des options sources HBase":::

[Exemple de commande du Stockage Table Azure](#azure-table-storage)

[Exemple de commande d’API Table Azure Cosmos DB (préversion)](#table-api-preview)

### <a name="command-line-target-settings"></a>Paramètres de la cible de la ligne de commande

Utilisez les options de cible suivantes lorsque vous définissez l’API Table Azure Cosmos DB en tant que cible de la migration.

* **/t:TableAPIBulk :** charge des données par lots dans une table CosmosDB Azure
* **/t.ConnectionString :** chaîne de connexion pour le point de terminaison de table
* **/t.TableName :** Spécifie le nom de la table dans laquelle écrire
* **/t.Overwrite :** facultatif. La valeur par défaut est false. Spécifie si les valeurs existantes doivent être remplacées
* **/t.MaxInputBufferSize :** facultatif. La valeur par défaut est 1 Go. Estimation approximative des octets d’entrée à mettre en mémoire tampon avant de vider les données dans le récepteur
* **/t.Throughput :** facultatif. Si aucune valeur n’est spécifiée, le service est utilisé par défaut. Spécifie le débit à configurer pour la table
* **/t.MaxBatchSize :** facultatif. La valeur par défaut est 2 Mo. Spécifier la taille du lot, en octets

<a id="azure-table-storage"></a>
### <a name="sample-command-source-is-azure-table-storage"></a>Exemple de commande : la source correspond au Stockage Table Azure

Voici un exemple de ligne de commande indiquant comment effectuer l’importation entre le Stockage Table Azure et l’API Table :

```bash
dt /s:AzureTable /s.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Table storage account name>;AccountKey=<Account Key>;EndpointSuffix=core.windows.net /s.Table:<Table name> /t:TableAPIBulk /t.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Cosmos DB account name>;AccountKey=<Azure Cosmos DB account key>;TableEndpoint=https://<Account name>.table.cosmosdb.azure.com:443 /t.TableName:<Table name> /t.Overwrite
```

<a id="table-api-preview"></a>
### <a name="sample-command-source-is-azure-cosmos-db-table-api-preview"></a>Exemple de commande : la source correspond à l'API Table Azure Cosmos DB (préversion)

Voici un exemple de ligne de commande permettant d’effectuer l’importation entre l’API Table (préversion) et la version GA de l’API Table :

```bash
dt /s:AzureTable /s.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Table API preview account name>;AccountKey=<Table API preview account key>;TableEndpoint=https://<Account Name>.documents.azure.com; /s.Table:<Table name> /t:TableAPIBulk /t.ConnectionString:DefaultEndpointsProtocol=https;AccountName=<Azure Cosmos DB account name>;AccountKey=<Azure Cosmos DB account key>;TableEndpoint=https://<Account name>.table.cosmosdb.azure.com:443 /t.TableName:<Table name> /t.Overwrite
```

## <a name="migrate-data-by-using-azcopy"></a>Migrer des données à l’aide d’AzCopy

L’utilitaire de ligne de commande AzCopy est l’autre option qui permet de migrer les données à partir du Stockage Table Azure vers l’API Table Azure Cosmos DB. Pour utiliser AzCopy, vous devez d’abord exporter vos données comme décrit dans [Exporter des données à partir du stockage Table](/previous-versions/azure/storage/storage-use-azcopy#export-data-from-table-storage), puis importer les données vers Azure Cosmos DB comme indiqué dans [l’API Table Azure Cosmos DB](/previous-versions/azure/storage/storage-use-azcopy#import-data-into-table-storage).

Lorsque vous importez les données dans Azure Cosmos DB, consultez l’exemple suivant. Notez que la valeur /Dest utilise « cosmosdb », et non « core ».

Exemple de commande d’importation :

```bash
AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.cosmosdb.windows.net/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace
```

## <a name="migrate-from-table-api-preview-to-table-api"></a>Migration depuis une API Table (préversion) vers une autre

> [!WARNING]
> Si vous voulez immédiatement profiter des avantages des tables généralement disponibles, migrez vos tables en préversion existantes comme indiqué dans cette section. Sinon, nous effectuerons des migrations automatiques pour les clients en préversion existants au cours des semaines à venir. Toutefois, notez que les tables en préversion migrées automatiquement sont associées à des restrictions, alors que les nouvelles tables créées ne le sont pas.

L’API Table est généralement disponible (GA). Il existe des différences entre les préversions et les versions GA des tables, que ce soit dans le code exécuté dans le cloud ou dans le code exécuté sur le client. Pour cette raison, nous vous invitons à éviter de mélanger un client SDK en préversion avec un compte d’API Table GA, et vice versa. L’API Table permet de créer une préversion pour les clients qui veulent continuer à utiliser les tables existantes, mais qui, dans un environnement de production, doivent migrer des données depuis la préversion vers l’environnement GA, ou attendre une migration automatique. Si vous attendez la migration automatique, vous serez informé des restrictions associées aux tables migrées. Après la migration, vous ne pourrez pas créer sans restriction des tables sur votre compte existant (seules les tables migrées seront associées à des restrictions).

Pour effectuer la migration à partir de l’API Table (préversion) vers l’API Table généralement disponible, procédez comme suit :

1. Créez un compte Azure Cosmos DB et définissez son type d’API vers la table Azure, comme indiqué dans la section [Créer un compte de base de données](create-table-dotnet.md#create-a-database-account).

2. Modifiez les clients pour utiliser une version GA des [Kits de développement logiciel (SDK) d’API Table](table-sdk-dotnet.md).

3. Migrez les données client à partir des tables en préversion vers les tables GA à l’aide de l’outil de migration de données. Les instructions sur l’utilisation de l’outil de migration de données à cet effet sont décrites dans la section [Outil de migration de données](#data-migration-tool). 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Importer des données avec l’outil de migration de données
> * Importer des données avec AzCopy
> * Migration depuis une API Table (préversion) vers une autre

Vous pouvez maintenant passer au didacticiel suivant et découvrir comment interroger les données à l’aide de l’API Table Azure Cosmos DB. 

> [!div class="nextstepaction"]
>[Comment interroger les données ?](../cosmos-db/tutorial-query-table.md)