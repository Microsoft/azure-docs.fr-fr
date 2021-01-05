---
title: Utiliser le Stockage Table et l’API Table d’Azure Cosmos DB avec Ruby
description: Stockez des données structurées dans le cloud à l’aide du stockage de tables Azure ou de l’API Table d’Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: ruby
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.openlocfilehash: 2d0c8433fff58854cb77a4e806058eae1937e71b
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101117"
---
# <a name="how-to-use-azure-table-storage-and-the-azure-cosmos-db-table-api-with-ruby"></a>Comment utiliser le Stockage Table et l’API Table d’Azure Cosmos DB avec Ruby
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Cet article explique comment créer des tables, stocker vos données et effectuer des opérations CRUD sur celles-ci. Choisissez le service de Table Azure ou l’API Table Azure Cosmos DB. Les exemples décrits dans cet article sont écrits en Ruby et utilisent la [Bibliothèque de client du service de Table du Stockage Azure pour Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table). Les scénarios traités incluent la création d’une table, la suppression d’une table, l’insertion d’entités et l’interrogation d’entités à partir de la table.

## <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Créer un compte de stockage Azure**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Création d’un compte Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="add-access-to-azure-storage-or-azure-cosmos-db"></a>Ajouter un accès au stockage Azure ou à Azure Cosmos DB

Pour utiliser le Stockage Azure ou Azure Cosmos DB, vous devez télécharger et utiliser le package Azure Ruby, qui comprend un ensemble de bibliothèques permettant de communiquer avec les services REST de Table.

### <a name="use-rubygems-to-obtain-the-package"></a>Utilisation de RubyGems pour obtenir le package

1. Ouvrez une interface de ligne de commande, telle que **PowerShell** (Windows), **Terminal** (Mac) ou **Bash** (Unix).
2. Tapez **gem install azure-storage-table** dans la fenêtre de commande pour installer gem et les dépendances.

### <a name="import-the-package"></a>Importation du package

À l’aide de votre éditeur de texte, ajoutez la commande suivante au début du fichier Ruby où vous comptez utiliser Azure Storage :

```ruby
require "azure/storage/table"
```

## <a name="add-your-connection-string"></a>Ajouter votre chaîne de connexion

Vous pouvez vous connecter au compte de stockage Azure ou au compte de l’API Table Azure Cosmos DB. Obtenez la chaîne de connexion en fonction du type de compte que vous utilisez.

### <a name="add-an-azure-storage-connection"></a>Ajout d’une connexion au stockage Azure

Le module de Stockage Azure lit les variables d’environnement **AZURE_STORAGE_ACCOUNT** et **AZURE_STORAGE_ACCESS_KEY** pour obtenir les informations nécessaires à la connexion à votre compte de Stockage Azure. Si ces variables d’environnement ne sont pas définies, vous devrez spécifier les informations de compte avant d’utiliser **Azure::Storage::Table::TableService** grâce au code suivant :

```ruby
Azure.config.storage_account_name = "<your Azure Storage account>"
Azure.config.storage_access_key = "<your Azure Storage access key>"
```

Pour obtenir ces valeurs à partir d’un compte de stockage classique ou Resource Manager sur le portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez au compte de Stockage que vous souhaitez utiliser.
3. Dans le panneau Paramètres à droite, cliquez sur **Clés d’accès**.
4. Dans le panneau Clés d’accès qui apparaît, la clé d’accès 1 et la clé d’accès 2 sont affichées. Vous pouvez utiliser les deux.
5. Cliquez sur l'icône de copie pour copier la clé dans le Presse-papiers.

### <a name="add-an-azure-cosmos-db-connection"></a>Ajouter une connexion à Azure Cosmos DB

Pour vous connecter à Azure Cosmos DB, copiez votre chaîne de connexion principale à partir du portail Azure, puis utilisez-la pour créer un objet **Client**. Vous pouvez transmettre l’objet **Client** lorsque vous créez un objet **TableService**  :

```ruby
common_client = Azure::Storage::Common::Client.create(storage_account_name:'myaccount', storage_access_key:'mykey', storage_table_host:'mycosmosdb_endpoint')
table_client = Azure::Storage::Table::TableService.new(client: common_client)
```

## <a name="create-a-table"></a>Créer une table

L’objet **Azure::Storage::Table::TableService** permet d’utiliser des tables et des entités. Pour créer une table, utilisez la méthode **create_table()** . L’exemple suivant crée une table ou imprime l’erreur le cas échéant.

```ruby
azure_table_service = Azure::Storage::Table::TableService.new
begin
    azure_table_service.create_table("testtable")
rescue
    puts $!
end
```

## <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Pour ajouter une entité, créez tout d'abord un objet de hachage qui définit les propriétés de votre entité. Notez que pour chaque entité, vous devez spécifier les clés **PartitionKey** et **RowKey**. Il s’agit d’identificateurs uniques de vos entités, dont les valeurs peuvent être interrogées bien plus rapidement que d’autres propriétés. Azure Storage utilise **PartitionKey** pour distribuer automatiquement les entités de la table sur plusieurs nœuds de stockage. Les entités partageant la même clé **PartitionKey** sont stockées sur le même nœud. **RowKey** est l'identifiant unique de l'entité dans sa partition.

```ruby
entity = { "content" => "test entity",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.insert_entity("testtable", entity)
```

## <a name="update-an-entity"></a>Mise à jour d'une entité

Plusieurs méthodes permettent de mettre à jour une entité existante :

* **update_entity() :** met à jour une entité existante en la remplaçant.
* **merge_entity() :** met à jour une entité existante en fusionnant les nouvelles valeurs des propriétés avec l’entité existante.
* **insert_or_merge_entity() :** met à jour une entité existante en la remplaçant. En l'absence d'entité, une nouvelle entité est insérée.
* **insert_or_replace_entity() :** met à jour une entité existante en fusionnant les nouvelles valeurs des propriétés avec l’entité existante. En l'absence d'entité, une nouvelle entité est insérée.

L’exemple suivant illustre la mise à jour d’une entité avec **update_entity()**  :

```ruby
entity = { "content" => "test entity with updated content",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.update_entity("testtable", entity)
```

Avec **update_entity()** et **merge_entity()** , si l’entité que vous mettez à jour n’existe pas, l’opération échoue. Si vous voulez stocker une entité, qu’elle existe déjà ou non, utilisez plutôt **insert_or_replace_entity()** ou **insert_or_merge_entity()** .

## <a name="work-with-groups-of-entities"></a>Utilisation des groupes d'entités

Il est parfois intéressant de soumettre un lot d'opérations simultanément pour assurer un traitement atomique par le serveur. Pour cela, vous devez d’abord créer un objet **Batch** , puis utiliser la méthode **execute_batch()** sur **TableService**. L'exemple ci-dessous présente l'envoi de deux entités avec RowKey 2 et 3 dans un lot. Notez qu'il ne s'applique qu'aux entités possédant le même élément PartitionKey.

```ruby
azure_table_service = Azure::TableService.new
batch = Azure::Storage::Table::Batch.new("testtable",
    "test-partition-key") do
    insert "2", { "content" => "new content 2" }
    insert "3", { "content" => "new content 3" }
end
results = azure_table_service.execute_batch(batch)
```

## <a name="query-for-an-entity"></a>Interrogation d’une entité

Pour interroger une entité dans une table, utilisez la méthode **get_entity()** en transmettant le nom de la table, **PartitionKey** et **RowKey**.

```ruby
result = azure_table_service.get_entity("testtable", "test-partition-key",
    "1")
```

## <a name="query-a-set-of-entities"></a>Interrogation d’un ensemble d’entités

Pour interroger un ensemble d’entités dans une table, créez un objet de hachage de requête et utilisez la méthode **query_entities()** . L'exemple ci-dessous présente l'obtention de toutes les identités avec le même élément **PartitionKey** :

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'" }
result, token = azure_table_service.query_entities("testtable", query)
```

> [!NOTE]
> Si le jeu de résultats est trop grand pour être renvoyé par une seule requête, un jeton de liaison est renvoyé. Vous pouvez l’utiliser pour récupérer les pages suivantes.


## <a name="query-a-subset-of-entity-properties"></a>Interrogation d'un sous-ensemble de propriétés d'entité

Vous pouvez utiliser une requête de table pour extraire uniquement quelques propriétés d’une entité. Cette technique, nommée « projection », réduit la consommation de bande passante et peut améliorer les performances des requêtes, notamment pour les entités volumineuses. Utilisez la clause select et transmettez le nom des propriétés à soumettre au client.

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'",
    :select => ["content"] }
result, token = azure_table_service.query_entities("testtable", query)
```

## <a name="delete-an-entity"></a>Suppression d’une entité

Pour supprimer une entité, utilisez la méthode **delete_entity()** . Transmettez le nom de la table qui contient l’entité, ainsi que les éléments PartitionKey et RowKey de l’entité.

```ruby
azure_table_service.delete_entity("testtable", "test-partition-key", "1")
```

## <a name="delete-a-table"></a>Suppression d’une table

Pour supprimer une table, utilisez la méthode **delete_table()** et transmettez le nom de la table que vous souhaitez supprimer.

```ruby
azure_table_service.delete_table("testtable")
```

## <a name="next-steps"></a>Étapes suivantes

* [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
* [Centre de développement Ruby](https://azure.microsoft.com/develop/ruby/)
* [Bibliothèque de client du service de Table du Stockage Microsoft Azure pour Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table)