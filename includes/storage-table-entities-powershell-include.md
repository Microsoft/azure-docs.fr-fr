---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/27/2019
ms.author: tamram
ms.openlocfilehash: 9a60c624b181a1efd2f6deebd349daa82214a8a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176738"
---
<!--created by Robin Shahan to go in the articles for table storage w/powershell.
    There is one for Azure Table Storage and one for Azure Cosmos DB Table API -->

## <a name="managing-table-entities"></a>Gestion des entités de table

Maintenant que vous avez une table, intéressons-nous à la manière de gérer les entités, ou lignes, dans cette table. 

Ces entités peuvent avoir jusqu’à 255 propriétés, dont trois propriétés système : **PartitionKey**, **RowKey** et **Timestamp**. Vous êtes chargé d’insérer et de mettre à jour les valeurs de **PartitionKey** et **RowKey**. Le serveur gère la valeur de **Timestamp**, qui ne peut pas être modifiée. Les propriétés **PartitionKey** et **RowKey** identifient de manière unique chaque entité d’une table.

* **PartitionKey** : détermine la partition dans laquelle l’entité est stockée.
* **RowKey** : identifie de façon unique l’entité dans la partition.

Vous pouvez définir au maximum 252 propriétés personnalisées par entité. 

### <a name="add-table-entities"></a>Ajouter des entités de table

Ajouter des entités à une table à l’aide de **Add-AzTableRow**. Ces exemples utilisent des clés de partition avec des valeurs `partition1` et `partition2`, ainsi que des clés de ligne équivalant à des abréviations d’état. Les propriétés de chaque entité sont `username` et `userid`. 

```powershell
$partitionKey1 = "partition1"
$partitionKey2 = "partition2"

# add four rows 
Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("CA") -property @{"username"="Chris";"userid"=1}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("NM") -property @{"username"="Jessie";"userid"=2}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey1 `
    -rowKey ("WA") -property @{"username"="Christine";"userid"=3}

Add-AzTableRow `
    -table $cloudTable `
    -partitionKey $partitionKey2 `
    -rowKey ("TX") -property @{"username"="Steven";"userid"=4}
```

### <a name="query-the-table-entities"></a>Interroger les entités de table

Vous pouvez interroger les entités dans une table avec la commande **Get-AzTableRow**.

> [!NOTE]
> Les cmdlets **Get-AzureStorageTableRowAll**, **Get-AzureStorageTableRowByPartitionKey**, **Get-AzureStorageTableRowByColumnName**, et **Get-AzureStorageTableRowByCustomFilter** sont déconseillées et seront supprimées dans une mise à jour ultérieure.

#### <a name="retrieve-all-entities"></a>Récupérer toutes les entités

```powershell
Get-AzTableRow -table $cloudTable | ft
```

Cette commande génère des résultats similaires à la table suivante :

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |
| 2 | Jessie | partition2 | NM |
| 4 | Steven | partition2 | TX |

#### <a name="retrieve-entities-for-a-specific-partition"></a>Récupérer les entités d’une partition spécifique

```powershell
Get-AzTableRow -table $cloudTable -partitionKey $partitionKey1 | ft
```

Les résultats ressemblent à la table suivante :

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |

#### <a name="retrieve-entities-for-a-specific-value-in-a-specific-column"></a>Récupérer les entités d’une valeur spécifique dans une colonne spécifique

```powershell
Get-AzTableRow -table $cloudTable `
    -columnName "username" `
    -value "Chris" `
    -operator Equal
```

Cette requête récupère un seul enregistrement.

|field|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

#### <a name="retrieve-entities-using-a-custom-filter"></a>Récupérer des entités à l’aide d’un filtre personnalisé 

```powershell
Get-AzTableRow `
    -table $cloudTable `
    -customFilter "(userid eq 1)"
```

Cette requête récupère un seul enregistrement.

|field|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

### <a name="updating-entities"></a>Mise à jour des entités 

Trois étapes sont nécessaires pour mettre à jour des entités. La première consiste à récupérer l’entité à modifier. La deuxième à apporter la modification. La troisième à valider la modification à l’aide de la commande **Update-AzTableRow**.

Mettez à jour l’entité dont le nom d’utilisateur est « Jessie » pour que le nom d’utilisateur devienne « Jessie2 ». Cet exemple montre également une autre façon de créer un filtre personnalisé à l’aide de types .NET.

```powershell
# Create a filter and get the entity to be updated.
[string]$filter = `
    [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
    [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie")
$user = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter

# Change the entity.
$user.username = "Jessie2"

# To commit the change, pipe the updated record into the update cmdlet.
$user | Update-AzTableRow -table $cloudTable

# To see the new record, query the table.
Get-AzTableRow -table $cloudTable `
    -customFilter "(username eq 'Jessie2')"
```

Les résultats montrent l’enregistrement Jessie2.

|field|value|
|----|----|
| userid | 2 |
| username | Jessie2 |
| PartitionKey | partition2 |
| RowKey      | NM |

### <a name="deleting-table-entities"></a>Suppression d’entités de table

Vous pouvez supprimer une entité ou toutes les entités dans une table.

#### <a name="deleting-one-entity"></a>Suppression d’une seule entité

Pour supprimer une seule entité, obtenez une référence à cette entité et redirigez-la vers **Remove-AzTableRow**.

```powershell
# Set filter.
[string]$filter = `
  [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("username",`
  [Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Jessie2")

# Retrieve entity to be deleted, then pipe it into the remove cmdlet.
$userToDelete = Get-AzTableRow `
    -table $cloudTable `
    -customFilter $filter
$userToDelete | Remove-AzTableRow -table $cloudTable

# Retrieve entities from table and see that Jessie2 has been deleted.
Get-AzTableRow -table $cloudTable | ft
```

#### <a name="delete-all-entities-in-the-table"></a>Supprimer toutes les entités d’une table

Pour supprimer toutes les entités d’une table, récupérez-les, puis redirigez les résultats vers l’applet de commande remove. 

```powershell
# Get all rows and pipe the result into the remove cmdlet.
Get-AzTableRow `
    -table $cloudTable | Remove-AzTableRow -table $cloudTable 

# List entities in the table (there won't be any).
Get-AzTableRow -table $cloudTable | ft
```
