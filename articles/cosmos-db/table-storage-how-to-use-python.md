---
title: Utiliser l’API Table Azure Cosmos DB et le Stockage Table Azure avec Python
description: Stockez des données structurées dans le cloud à l’aide du Stockage Table Azure ou de l’API Table Azure Cosmos DB avec Python.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: python
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.custom: devx-track-python
ms.openlocfilehash: 341615bf8fe231eafa606411948ad014399b8261
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93079544"
---
# <a name="get-started-with-azure-table-storage-and-the-azure-cosmos-db-table-api-using-python"></a>Prise en main du stockage de tables Azure et de l’API Table d’Azure Cosmos DB avec Python
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Le Stockage Table Azure et Azure Cosmos DB sont des services qui stockent des données NoSQL structurées dans le cloud en fournissant une conception sans schéma à un magasin de clés/attributs. Comme le stockage de tables et Azure Cosmos DB sont sans schéma, il est aisé d’adapter vos données en fonction des besoins de votre application. L’accès au stockage de tables et aux données de l’API de table est rapide et économique pour de nombreux types d’applications et généralement moins coûteux que le SQL traditionnel pour des volumes de données similaires.

Vous pouvez utiliser le Stockage Table ou Azure Cosmos DB pour stocker des jeux de données flexibles comme des données utilisateur pour des applications web, des carnets d’adresses, des informations sur les appareils ou d’autres types de métadonnées requis par votre service. Vous pouvez stocker un nombre quelconque d'entités dans une table, et un compte de stockage peut contenir un nombre quelconque de tables, jusqu'à la limite de capacité du compte de stockage.

### <a name="about-this-sample"></a>À propos de cet exemple

Cet exemple montre comment utiliser le [Kit de développement logiciel (SDK) de table Azure Cosmos DB pour Python](https://pypi.python.org/pypi/azure-cosmosdb-table/) dans les scénarios de stockage de tables Azure courants. Le nom du Kit de développement logiciel (SDK) indique qu’il doit être utilisé avec Azure Cosmos DB, mais il fonctionne avec Azure Cosmos DB et avec le stockage de tables Azure, chaque service possédant un point de terminaison unique. Ces scénarios sont explorés à l’aide d’exemples Python qui montrent comment :

* créer et supprimer des tables ;
* Insérer et interroger des entités
* Modifier des entités

Pendant que vous étudiez les scénarios de cet exemple, vous pouvez vous référer à la [Référence du kit de développement logiciel (SDK) Azure Cosmos DB pour l’API Python](/python/api/overview/azure/cosmosdb?preserve-view=true&view=azure-python).

## <a name="prerequisites"></a>Conditions préalables requises

Vous aurez besoin des éléments suivants pour suivre cet exemple :

* [Python](https://www.python.org/downloads/) 2.7, 3.3, 3.4, 3.5 ou 3.6
* [Kit de développement logiciel (SDK) du service Table d’Azure Cosmos DB pour Python](https://pypi.python.org/pypi/azure-cosmosdb-table/). Ce kit de développement logiciel (SDK) se connecte à la fois avec l’API Table Azure Cosmos DB et le Stockage Table Azure.
* [Compte de stockage Azure](../storage/common/storage-account-create.md) ou [Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)

## <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Créer un compte de stockage Azure**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Créer un compte d’API Table Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="install-the-azure-cosmos-db-table-sdk-for-python"></a>Installer le Kit de développement logiciel (SDK) de table Azure Cosmos DB pour Python

Après avoir créé un compte de stockage, l’étape suivante consiste à installer le [Kit de développement logiciel (SDK) de table Microsoft Azure Cosmos DB pour Python](https://pypi.python.org/pypi/azure-cosmosdb-table/). Pour plus de détails sur l’installation du SDK, consultez le fichier [README.rst](https://github.com/Azure/azure-cosmosdb-python/blob/master/azure-cosmosdb-table/README.rst) dans le dépôt SDK Table Cosmos DB pour Python sur GitHub.

## <a name="import-the-tableservice-and-entity-classes"></a>Importer les classes TableService et Entity

Pour travailler avec les entités dans le service de Table Azure dans Python, vous utilisez les classes [TableService][py_TableService] et [Entity][py_Entity]. Ajoutez ce code vers le haut de votre fichier Python pour importer les deux :

```python
from azure.cosmosdb.table.tableservice import TableService
from azure.cosmosdb.table.models import Entity
```

## <a name="connect-to-azure-table-service"></a>Se connecter au service de Table Azure

Pour vous connecter au service de Table de stockage Azure, créez un objet [TableService][py_TableService], puis transmettez votre clé de compte et votre nom de compte de stockage. Remplacez `myaccount` et `mykey` par le nom et la clé de votre compte.

```python
table_service = TableService(account_name='myaccount', account_key='mykey')
```

## <a name="connect-to-azure-cosmos-db"></a>Se connecter à Azure Cosmos DB

Pour vous connecter à Azure Cosmos DB, copiez votre chaîne de connexion principale à partir du portail Azure, puis créez un objet [TableService][py_TableService] en utilisant votre chaîne de connexion copiée :

```python
table_service = TableService(connection_string='DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;TableEndpoint=myendpoint;')
```

## <a name="create-a-table"></a>Créer une table

Appelez [create_table][py_create_table] pour créer la table.

```python
table_service.create_table('tasktable')
```

## <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Pour ajouter une entité, vous créez d’abord un objet qui représente votre entité, puis passez l’objet à la [méthode TableService.insert_entity][py_TableService]. L’objet d’entité peut être un dictionnaire ou un objet de type [Entité][py_Entity], et définit les noms et valeurs de votre entité. Chaque entité doit inclure les propriétés [PartitionKey et RowKey](#partitionkey-and-rowkey), en plus de toutes les autres propriétés que vous définissez pour l’entité.

Cet exemple crée un objet dictionnaire représentant une entité, puis le passe à la méthode [insert_entity][py_insert_entity] pour l’ajouter à la table :

```python
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001',
        'description': 'Take out the trash', 'priority': 200}
table_service.insert_entity('tasktable', task)
```

Cet exemple crée un objet [Entité][py_Entity], puis le passe à la méthode [insert_entity][py_insert_entity] pour l’ajouter à la table :

```python
task = Entity()
task.PartitionKey = 'tasksSeattle'
task.RowKey = '002'
task.description = 'Wash the car'
task.priority = 100
table_service.insert_entity('tasktable', task)
```

### <a name="partitionkey-and-rowkey"></a>PartitionKey et RowKey

Vous devez spécifier les propriétés **PartitionKey** et **RowKey** pour chaque entité. Il s’agit des identificateurs uniques de vos entités, car ils forment ensemble la clé primaire d’une entité. Vous pouvez interroger à l’aide de ces valeurs beaucoup plus vite que vous pouvez interroger d’autres propriétés d’entité, car seules ces propriétés sont indexées.

Le service de table utilise **PartitionKey** pour répartir intelligemment les entités de table entre nœuds de stockage. Les entités partageant la même clé **PartitionKey** sont stockées sur le même nœud. **RowKey** est l'identifiant unique de l'entité dans sa partition.

## <a name="update-an-entity"></a>Mise à jour d'une entité

Pour mettre à jour toutes les valeurs de propriété d’une entité, appelez la méthode [update_entity][py_update_entity]. Cet exemple montre comment remplacer d'une entité existante par une version mise à jour :

```python
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001',
        'description': 'Take out the garbage', 'priority': 250}
table_service.update_entity('tasktable', task)
```

Si l'entité à remplacer n'existe pas encore, l'opération de mise à jour échoue. Si vous voulez stocker une entité, qu’elle existe déjà ou non, utilisez [insert_or_replace_entity][py_insert_or_replace_entity]. Dans l'exemple suivant, le premier appel remplace l'entité existante. Le deuxième appel insère une nouvelle entité, car il n’existe aucune entité ayant les clés PartitionKey et RowKey spécifiées.

```python
# Replace the entity created earlier
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001',
        'description': 'Take out the garbage again', 'priority': 250}
table_service.insert_or_replace_entity('tasktable', task)

# Insert a new entity
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '003',
        'description': 'Buy detergent', 'priority': 300}
table_service.insert_or_replace_entity('tasktable', task)
```

> [!TIP]
> La méthode [update_entity][py_update_entity] remplace toutes les propriétés et valeurs d’une entité existante, ce qui vous permet également de supprimer les propriétés d’une entité existante. Vous pouvez utiliser la méthode [merge_entity][py_merge_entity] pour mettre à jour une entité existante avec des valeurs de propriétés nouvelles ou modifiées sans remplacement complet de l’entité.

## <a name="modify-multiple-entities"></a>Modifier plusieurs entités

Pour garantir le traitement atomique d’une demande par le service de table, vous pouvez envoyer plusieurs opérations dans un lot. Tout d’abord, utilisez la classe [TableBatch][py_TableBatch] pour ajouter plusieurs opérations à un seul lot. Ensuite, appelez [TableService][py_TableService].[commit_batch][py_commit_batch] pour envoyer les opérations dans une opération atomique. Toutes les entités à modifier dans le lot doivent être dans la même partition.

Cet exemple permet d'ajouter deux entités dans un lot :

```python
from azure.cosmosdb.table.tablebatch import TableBatch
batch = TableBatch()
task004 = {'PartitionKey': 'tasksSeattle', 'RowKey': '004',
           'description': 'Go grocery shopping', 'priority': 400}
task005 = {'PartitionKey': 'tasksSeattle', 'RowKey': '005',
           'description': 'Clean the bathroom', 'priority': 100}
batch.insert_entity(task004)
batch.insert_entity(task005)
table_service.commit_batch('tasktable', batch)
```

Les lots peuvent également être utilisés avec la syntaxe du gestionnaire de contexte :

```python
task006 = {'PartitionKey': 'tasksSeattle', 'RowKey': '006',
           'description': 'Go grocery shopping', 'priority': 400}
task007 = {'PartitionKey': 'tasksSeattle', 'RowKey': '007',
           'description': 'Clean the bathroom', 'priority': 100}

with table_service.batch('tasktable') as batch:
    batch.insert_entity(task006)
    batch.insert_entity(task007)
```

## <a name="query-for-an-entity"></a>Interrogation d’une entité

Pour rechercher une entité dans une table, passez ses paramètres PartitionKey et RowKey à la méthode [TableService][py_TableService].[get_entity][py_get_entity].

```python
task = table_service.get_entity('tasktable', 'tasksSeattle', '001')
print(task.description)
print(task.priority)
```

## <a name="query-a-set-of-entities"></a>Interrogation d’un ensemble d’entités

Vous pouvez interroger un jeu d’entités en fournissant une chaîne de filtre avec le paramètre **filter**. Cet exemple recherche toutes les tâches dans Seattle pour appliquer un filtre sur PartitionKey :

```python
tasks = table_service.query_entities(
    'tasktable', filter="PartitionKey eq 'tasksSeattle'")
for task in tasks:
    print(task.description)
    print(task.priority)
```

## <a name="query-a-subset-of-entity-properties"></a>Interrogation d'un sous-ensemble de propriétés d'entité

Vous pouvez également restreindre les propriétés renvoyées pour chaque entité dans une requête. Cette technique, nommée *projection* , réduit la consommation de bande passante et peut améliorer les performances des requêtes, notamment pour les entités volumineuses ou les jeux de résultats. Utilisez le paramètre **select** et transmettez le nom des propriétés à renvoyer au client.

La requête contenue dans le code suivant ne renvoie que la description des entités de la table.

> [!NOTE]
> L’extrait suivant ne fonctionne qu’avec Stockage Azure. L’émulateur de stockage ne le prend pas en charge.

```python
tasks = table_service.query_entities(
    'tasktable', filter="PartitionKey eq 'tasksSeattle'", select='description')
for task in tasks:
    print(task.description)
```

## <a name="delete-an-entity"></a>Suppression d’une entité

Supprimez une entité en passant des propriétés **PartitionKey** et **RowKey** à la méthode [delete_entity][py_delete_entity].

```python
table_service.delete_entity('tasktable', 'tasksSeattle', '001')
```

## <a name="delete-a-table"></a>Suppression d’une table

Si vous n’avez plus besoin d’une table ou de l’une des entités qui s’y trouvent, appelez la méthode [delete_table][py_delete_table] pour supprimer définitivement la table de Stockage Azure.

```python
table_service.delete_table('tasktable')
```

## <a name="next-steps"></a>Étapes suivantes

* [FAQ - Développer avec l’API Table](./faq.md)
* [Référence du kit de développement logiciel (SDK) Azure Cosmos DB pour l’API Python](/python/api/overview/azure/cosmosdb?preserve-view=true&view=azure-python)
* [Centre de développement Python](https://azure.microsoft.com/develop/python/)
* [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) : une application gratuite et multiplateforme de Microsoft qui vous permet d’exploiter visuellement les données de stockage Azure sur Windows, macOS et Linux.
* [Utilisation de Python dans Visual Studio (Windows)](/visualstudio/python/overview-of-python-tools-for-visual-studio)



[py_commit_batch]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_create_table]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_delete_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_get_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_insert_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_insert_or_replace_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_Entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.models.entity?preserve-view=true&view=azure-python
[py_merge_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_update_entity]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_delete_table]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_TableService]: /python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?preserve-view=true&view=azure-python
[py_TableBatch]: https://docs.microsoft.com/python/api/azure-cosmosdb-table/azure.cosmosdb.table.tableservice.tableservice?view=azure-python&preserve-view=true