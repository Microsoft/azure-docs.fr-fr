---
title: Utiliser le Stockage Table Azure et l’API Table Azure Cosmos DB avec C++
description: Stockez des données structurées dans le cloud à l’aide du Stockage Table Azure ou de l’API Table Azure Cosmos DB avec C++.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: cpp
ms.topic: sample
ms.date: 10/07/2019
author: sakash279
ms.author: akshanka
ms.openlocfilehash: 79b58b76954cf15289e85dbf763b7a399897635d
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489877"
---
# <a name="how-to-use-azure-table-storage-and-azure-cosmos-db-table-api-with-c"></a>Procédure d’utilisation du Stockage Table Azure et de l’API de Table Azure Cosmos DB avec C++
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Ce guide décrit des scénarios courants en utilisant le service de stockage Table Azure ou l’API Table Azure Cosmos DB. Les exemples ont été écrits en C++ et utilisent la [bibliothèque cliente Azure Storage pour C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md). Cet article aborde les scénarios suivants :

* Création et suppression d’une table
* Utilisation d’entités de table

> [!NOTE]
> Ce guide cible la bibliothèque cliente Azure Storage pour C++ version 1.0.0 et les versions ultérieures. La version recommandée est la bibliothèque cliente de stockage version 2.2.0, disponible par le biais de [NuGet](https://www.nuget.org/packages/wastorage) ou [GitHub](https://github.com/Azure/azure-storage-cpp/).
>

## <a name="create-accounts"></a>Création de comptes

### <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>Créer un compte de stockage Azure

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

### <a name="create-an-azure-cosmos-db-table-api-account"></a>Créer un compte d’API de table Azure Cosmos DB

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-c-application"></a>Création d’une application C++

Dans ce guide, vous utilisez les fonctionnalités de stockage d’une application C++. Pour ce faire, installez la bibliothèque cliente Stockage Azure pour C++.

Pour installer la bibliothèque cliente Stockage Azure pour C++, procédez comme suit :

* **Linux :** Suivez les instructions disponibles dans la page [Bibliothèque de client du stockage Azure pour C++ README : Prise en main de la page Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux).
* **Windows :** Sous Windows, utilisez [vcpkg](https://github.com/microsoft/vcpkg) en tant que manager de dépendances. Suivez le [démarrage rapide](https://github.com/microsoft/vcpkg#quick-start) pour initialiser vcpkg. Puis, pour installer la bibliothèque, utilisez la commande suivante :

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Un guide sur la façon de générer le code source et de l’exporter vers Nuget est disponible dans le fichier [LISEZ-MOI](https://github.com/Azure/azure-storage-cpp#download--install).

### <a name="configure-access-to-the-table-client-library"></a>Configurer l’accès à la bibliothèque de client de Table

Pour utiliser des API de stockage Azure afin d’accéder aux tables, ajoutez les instructions `include` suivantes au début du fichier C+ :

```cpp
#include <was/storage_account.h>
#include <was/table.h>
```

Un client de stockage Azure ou un client Cosmos DB utilise une chaîne de connexion pour stocker des points de terminaison et des informations d’identification permettant l’accès aux services de gestion des données. Quand vous exécutez une application cliente, vous devez fournir la chaîne de connexion de stockage ou la chaîne de connexion Azure Cosmos DB dans le format approprié.

### <a name="set-up-an-azure-storage-connection-string"></a>Configurer une chaîne de connexion au stockage Azure

Cet exemple montre comment déclarer un champ statique pour qu’il contienne la chaîne de connexion de stockage Azure :  

```cpp
// Define the Storage connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_storage_account>;AccountKey=<your_storage_account_key>"));
```

Utilisez le nom de votre compte de stockage pour `<your_storage_account>`. Pour <your_storage_account_key>, utilisez la clé d’accès associée au compte de stockage listé dans le [portail Azure](https://portal.azure.com). Pour plus d’informations sur les comptes de stockage et les clés d’accès, consultez [Créer un compte de stockage](../storage/common/storage-account-create.md).

### <a name="set-up-an-azure-cosmos-db-connection-string"></a>Configurer une chaîne de connexion Azure Cosmos DB

Cet exemple montre comment déclarer un champ statique pour qu’il contienne la chaîne de connexion Azure Cosmos DB :

```cpp
// Define the Azure Cosmos DB connection string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=<your_cosmos_db_account>;AccountKey=<your_cosmos_db_account_key>;TableEndpoint=<your_cosmos_db_endpoint>"));
```

Utilisez le nom de votre compte Azure Cosmos DB pour `<your_cosmos_db_account>`. Entrez votre clé primaire pour `<your_cosmos_db_account_key>`. Entrez le point de terminaison listé dans le [portail Azure](https://portal.azure.com) pour `<your_cosmos_db_endpoint>`.

Pour tester votre application sur votre ordinateur Windows local, vous pouvez utiliser l’émulateur de stockage Azure installé avec le [Kit de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/). L’émulateur de stockage est un utilitaire qui simule les services Azure d’objet blob, de file d’attente et de table disponibles sur votre ordinateur de développement local. L’exemple suivant montre comment déclarer un champ statique pour qu’il contienne une chaîne de connexion vers votre émulateur de stockage local :  

```cpp
// Define the connection string with Azure Storage Emulator.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

Pour démarrer l’émulateur de stockage Azure, à partir de votre Bureau Windows, sélectionnez le bouton **Démarrer** ou la touche Windows. Entrez et exécutez *Émulateur de stockage Microsoft Azure*. Pour plus d’informations, consultez [Utilisation de l’émulateur de stockage Azure pour le développement et le test](../storage/common/storage-use-emulator.md).

### <a name="retrieve-your-connection-string"></a>Récupération de votre chaîne de connexion

Vous pouvez utiliser la classe `cloud_storage_account` pour représenter les informations de votre compte de stockage. Pour extraire les informations de votre compte de stockage de la chaîne de connexion de stockage, utilisez la méthode `parse`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

Ensuite, récupérez une référence à une classe `cloud_table_client`. Cette classe vous permet d’obtenir des objets de référence pour les tables et entités stockées dans le service de stockage de Table. Le code suivant crée un objet `cloud_table_client` en utilisant l’objet de compte de stockage que vous avez récupéré :  

```cpp
// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();
```

## <a name="create-and-add-entities-to-a-table"></a>Créer et ajouter une entité à une table

### <a name="create-a-table"></a>Créer une table

Un objet `cloud_table_client`vous permet d’obtenir les objets de référence pour les tables et entités. Le code suivant crée un objet `cloud_table_client` et l’utilise pour créer une table.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);  

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();  
```

### <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Pour ajouter une entité à une table, créez un objet `table_entity` et transmettez-le à `table_operation::insert_entity`. Le code suivant utilise le prénom du client en tant que clé de ligne et son nom de famille en tant que clé de partition. Ensemble, les clés de partition et de ligne d’une entité identifient l’entité de façon unique dans la table. Les requêtes d'entités dont les clés de partition sont identiques sont plus rapides que celles d'entités dont les clés de partition sont différentes. L’utilisation de différentes clés de partition permet une plus grande scalabilité des opérations parallèles. Pour plus d’informations, consultez [Liste de contrôle des performances et de l’extensibilité de Microsoft Azure Storage](../storage/blobs/storage-performance-checklist.md).

Le code suivant crée une instance de la classe `table_entity` avec des données clientes à stocker. Le code appelle ensuite `table_operation::insert_entity` pour créer un objet `table_operation` afin d’insérer une entité dans une table et d’y associer la nouvelle entité de table. Enfin, le code appelle la méthode `execute` sur l’objet `cloud_table`. Le nouvel objet `table_operation` envoie une demande au service de Table pour insérer la nouvelle entité de client dans la table `people`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Retrieve a reference to a table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table if it doesn't exist.
table.create_if_not_exists();

// Create a new customer entity.
azure::storage::table_entity customer1(U("Harp"), U("Walter"));

azure::storage::table_entity::properties_type& properties = customer1.properties();
properties.reserve(2);
properties[U("Email")] = azure::storage::entity_property(U("Walter@contoso.com"));

properties[U("Phone")] = azure::storage::entity_property(U("425-555-0101"));

// Create the table operation that inserts the customer entity.
azure::storage::table_operation insert_operation = azure::storage::table_operation::insert_entity(customer1);

// Execute the insert operation.
azure::storage::table_result insert_result = table.execute(insert_operation);
```

### <a name="insert-a-batch-of-entities"></a>Insertion d'un lot d'entités

Vous pouvez insérer un lot d’entités dans le service de Table en une seule opération d’écriture. Le code suivant crée un objet `table_batch_operation`, puis y ajoute trois opérations d’insertion. Chaque opération d’insertion est ajoutée en créant un objet d’entité, en définissant ses valeurs, puis en appelant la méthode `insert` sur l’objet `table_batch_operation` pour associer la nouvelle entité à une nouvelle opération d’insertion. Ensuite, le code appelle `cloud_table.execute` pour exécuter l’opération.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define a batch operation.
azure::storage::table_batch_operation batch_operation;

// Create a customer entity and add it to the table.
azure::storage::table_entity customer1(U("Smith"), U("Jeff"));

azure::storage::table_entity::properties_type& properties1 = customer1.properties();
properties1.reserve(2);
properties1[U("Email")] = azure::storage::entity_property(U("Jeff@contoso.com"));
properties1[U("Phone")] = azure::storage::entity_property(U("425-555-0104"));

// Create another customer entity and add it to the table.
azure::storage::table_entity customer2(U("Smith"), U("Ben"));

azure::storage::table_entity::properties_type& properties2 = customer2.properties();
properties2.reserve(2);
properties2[U("Email")] = azure::storage::entity_property(U("Ben@contoso.com"));
properties2[U("Phone")] = azure::storage::entity_property(U("425-555-0102"));

// Create a third customer entity to add to the table.
azure::storage::table_entity customer3(U("Smith"), U("Denise"));

azure::storage::table_entity::properties_type& properties3 = customer3.properties();
properties3.reserve(2);
properties3[U("Email")] = azure::storage::entity_property(U("Denise@contoso.com"));
properties3[U("Phone")] = azure::storage::entity_property(U("425-555-0103"));

// Add customer entities to the batch insert operation.
batch_operation.insert_or_replace_entity(customer1);
batch_operation.insert_or_replace_entity(customer2);
batch_operation.insert_or_replace_entity(customer3);

// Execute the batch operation.
std::vector<azure::storage::table_result> results = table.execute_batch(batch_operation);
```

Quelques remarques sur les opérations par lots :

* Vous pouvez effectuer jusqu’à 100 opérations `insert`, `delete`, `merge`, `replace`, `insert-or-merge`et `insert-or-replace` dans n’importe quelle combinaison en un seul lot.  
* Une opération par lot peut comporter une opération d’extraction, s’il s’agit de la seule opération du lot.  
* Toutes les entités d'une opération par lot doivent avoir la même clé de partition.  
* Une opération par lot est limitée à une charge utile de données de 4 Mo.  

## <a name="query-and-modify-entities"></a>Interroger et modifier les entités

### <a name="retrieve-all-entities-in-a-partition"></a>Extraction de toutes les entités d'une partition

Pour exécuter une requête de table portant sur toutes les entités d’une partition, utilisez un objet `table_query`. L’exemple de code suivant indique un filtre pour les entités où `Smith` est la clé de partition. Il imprime les champs de chaque entité dans les résultats de requête vers la console.  

> [!NOTE]
> Ces méthodes ne sont pas actuellement prises en charge pour C++ dans Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Construct the query operation for all customer entities where PartitionKey="Smith".
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::generate_filter_condition(U("PartitionKey"), azure::storage::query_comparison_operator::equal, U("Smith")));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Print the fields for each customer.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

La requête de cet exemple retourne toutes les entités qui correspondent aux critères de filtre. Si vous avez des tables volumineuses et que vous devez télécharger les entités de table souvent, nous vous recommandons de plutôt stocker vos données dans des objets blob de stockage Azure.

### <a name="retrieve-a-range-of-entities-in-a-partition"></a>Extraction d’un ensemble d’entités dans une partition

Si vous ne voulez pas interroger toutes les entités d’une partition, vous pouvez définir une plage. Combinez le filtre de clé de partition avec un filtre de clé de ligne. L’exemple de code suivant utilise deux filtres pour obtenir toutes les entités dans la partition `Smith` où la clé de ligne (prénom) commence par une lettre située avant la lettre `E` dans l’ordre alphabétique, puis imprime les résultats de la requête.  

> [!NOTE]
> Ces méthodes ne sont pas actuellement prises en charge pour C++ dans Azure Cosmos DB.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create the table query.
azure::storage::table_query query;

query.set_filter_string(azure::storage::table_query::combine_filter_conditions(
    azure::storage::table_query::generate_filter_condition(U("PartitionKey"),
    azure::storage::query_comparison_operator::equal, U("Smith")),
    azure::storage::query_logical_operator::op_and,
    azure::storage::table_query::generate_filter_condition(U("RowKey"), azure::storage::query_comparison_operator::less_than, U("E"))));

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Loop through the results, displaying information about the entity.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    const azure::storage::table_entity::properties_type& properties = it->properties();

    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
        << U(", Property1: ") << properties.at(U("Email")).string_value()
        << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
}  
```

### <a name="retrieve-a-single-entity"></a>Extraction d'une seule entité

Vous pouvez écrire une requête pour extraire une seule entité. Le code suivant utilise `table_operation::retrieve_entity` pour spécifier le client `Jeff Smith`. Cette méthode retourne une seule entité, au lieu d’une collection. De plus, la valeur retournée est dans `table_result`. La méthode la plus rapide pour extraire une seule entité dans le service de Table consiste à spécifier une clé de partition et une clé de ligne.  

```cpp
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Output the entity.
azure::storage::table_entity entity = retrieve_result.entity();
const azure::storage::table_entity::properties_type& properties = entity.properties();

std::wcout << U("PartitionKey: ") << entity.partition_key() << U(", RowKey: ") << entity.row_key()
    << U(", Property1: ") << properties.at(U("Email")).string_value()
    << U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
```

### <a name="replace-an-entity"></a>Remplacement d’une entité

Pour remplacer une entité, récupérez-la dans le service de Table, modifiez l’objet d’entité, puis enregistrez les modifications dans le service de Table. Le code suivant modifie le numéro de téléphone et l’adresse de messagerie électronique d’un client existant. Au lieu d’appeler `table_operation::insert_entity`, ce code utilise `table_operation::replace_entity`. Cette approche entraîne le remplacement complet de l’entité sur le serveur, sauf si cette dernière a changé depuis sa récupération. En cas de changement, l’opération échoue. Cet échec empêche votre application de remplacer une modification apportée entre la récupération et la mise à jour par un autre composant. Pour gérer correctement cet échec, vous devez récupérer de nouveau l’entité, apporter vos modifications, si elles sont toujours valides, puis effectuer une autre opération `table_operation::replace_entity`.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Replace an entity.
azure::storage::table_entity entity_to_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_replace = entity_to_replace.properties();
properties_to_replace.reserve(2);

// Specify a new phone number.
properties_to_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0106"));

// Specify a new email address.
properties_to_replace[U("Email")] = azure::storage::entity_property(U("JeffS@contoso.com"));

// Create an operation to replace the entity.
azure::storage::table_operation replace_operation = azure::storage::table_operation::replace_entity(entity_to_replace);

// Submit the operation to the Table service.
azure::storage::table_result replace_result = table.execute(replace_operation);
```

### <a name="insert-or-replace-an-entity"></a>Insérer ou remplacer une entité

Les opérations `table_operation::replace_entity` échouent si l’entité est changée depuis sa récupération à partir du serveur. De plus, vous devez d’abord récupérer l’entité du serveur pour que l’opération `table_operation::replace_entity` réussisse. Parfois, vous ne savez pas si l’entité existe sur le serveur. Les valeurs actuelles qui y sont stockées ne sont pas pertinentes, car votre mise à jour doit les remplacer toutes. Pour obtenir ce résultat, utilisez une opération `table_operation::insert_or_replace_entity`. Cette opération insère l’entité si elle n’existe pas. L’opération remplace l’entité si elle existe. Dans l’exemple de code suivant, l’entité de client pour `Jeff Smith` est toujours récupérée, mais elle est ensuite enregistrée sur le serveur à l’aide de `table_operation::insert_or_replace_entity`. Les mises à jour apportées à l’entité entre les opérations de récupération et de mise à jour sont remplacées.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Insert or replace an entity.
azure::storage::table_entity entity_to_insert_or_replace(U("Smith"), U("Jeff"));
azure::storage::table_entity::properties_type& properties_to_insert_or_replace = entity_to_insert_or_replace.properties();

properties_to_insert_or_replace.reserve(2);

// Specify a phone number.
properties_to_insert_or_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0107"));

// Specify an email address.
properties_to_insert_or_replace[U("Email")] = azure::storage::entity_property(U("Jeffsm@contoso.com"));

// Create an operation to insert or replace the entity.
azure::storage::table_operation insert_or_replace_operation = azure::storage::table_operation::insert_or_replace_entity(entity_to_insert_or_replace);

// Submit the operation to the Table service.
azure::storage::table_result insert_or_replace_result = table.execute(insert_or_replace_operation);
```

### <a name="query-a-subset-of-entity-properties"></a>Interrogation d'un sous-ensemble de propriétés d'entité

Vous pouvez utiliser une requête de table pour extraire uniquement quelques propriétés d’une entité. La requête contenue dans le code suivant utilise la méthode `table_query::set_select_columns` pour retourner uniquement les adresses e-mail des entités dans la table.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Define the query, and select only the Email property.
azure::storage::table_query query;
std::vector<utility::string_t> columns;

columns.push_back(U("Email"));
query.set_select_columns(columns);

// Execute the query.
azure::storage::table_query_iterator it = table.execute_query(query);

// Display the results.
azure::storage::table_query_iterator end_of_results;
for (; it != end_of_results; ++it)
{
    std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key();

    const azure::storage::table_entity::properties_type& properties = it->properties();
    for (auto prop_it = properties.begin(); prop_it != properties.end(); ++prop_it)
    {
        std::wcout << ", " << prop_it->first << ": " << prop_it->second.str();
    }

    std::wcout << std::endl;
}
```

> [!NOTE]
> L’interrogation d’un petit nombre de propriétés d’une entité est une opération plus efficace que l’extraction de toutes les propriétés.
>

## <a name="delete-content"></a>Supprimer du contenu

### <a name="delete-an-entity"></a>Suppression d’une entité

Vous pouvez supprimer une entité après l’avoir récupérée. Après avoir récupéré une entité, appelez `table_operation::delete_entity` avec l’entité à supprimer. Appelez ensuite la méthode `cloud_table.execute`. Le code suivant récupère et supprime une entité dont la clé de partition est `Smith` et la clé de ligne `Jeff`.

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

// Create an operation to delete the entity.
azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

// Submit the delete operation to the Table service.
azure::storage::table_result delete_result = table.execute(delete_operation);  
```

### <a name="delete-a-table"></a>Suppression d’une table

Pour finir, l’exemple de code suivant supprime une table d’un compte de stockage. Une table supprimée ne peut pas être recréée pendant un certain temps.  

```cpp
// Retrieve the storage account from the connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the table client.
azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

// Create a cloud table object for the table.
azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

// Delete the table if it exists
if (table.delete_table_if_exists())
{
    std::cout << "Table deleted!";
}
else
{
    std::cout << "Table didn't exist";
}
```

## <a name="troubleshooting"></a>Dépannage

Pour Visual Studio Community Edition, si votre projet obtient des erreurs de build en raison des fichiers include *storage_account.h* et *table.h* , supprimez le commutateur du compilateur **/permissive-**  :

1. Dans **Explorateur de solutions** , cliquez avec le bouton de droite sur votre projet et sélectionnez **Propriétés**.
1. Dans la boîte de dialogue **Pages de propriétés** , développez **Propriétés de configuration** , développez **C/C++** , puis sélectionnez **Langage**.
1. Définissez **Mode de conformité** sur **Non**.

## <a name="next-steps"></a>Étapes suivantes

[Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.

Suivez ces liens pour en savoir plus sur le stockage Azure et l’API de Table dans Azure Cosmos DB :

* [Introduction à l’API de Table](table-introduction.md)
* [Listage des ressources Azure Storage en C++](../storage/common/storage-c-plus-plus-enumeration.md)
* [Référence de la bibliothèque cliente de stockage pour C++](https://azure.github.io/azure-storage-cpp)
* [Documentation d’Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
