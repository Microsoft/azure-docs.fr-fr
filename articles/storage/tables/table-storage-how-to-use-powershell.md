---
title: Effectuer des opérations sur Stockage Table Azure avec PowerShell | Microsoft Docs
description: Découvrir comment exécuter des tâches courantes telles que la création, l’interrogation et la suppression de données depuis un compte de stockage de table Azure à l’aide de PowerShell.
author: roygara
ms.service: storage
ms.topic: article
ms.date: 04/05/2019
ms.author: rogarana
ms.subservice: tables
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9a7502d48095fe18b983c1971d5145f51e766c95
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93306906"
---
# <a name="perform-azure-table-storage-operations-with-azure-powershell"></a>Effectuer des opérations sur Stockage Table Azure avec Azure PowerShell 
[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-langsoon-tip-include.md)]

Le stockage Table Azure est une banque de données NoSQL que vous pouvez utiliser pour stocker et interroger de grands ensembles de données non relationnelles et structurées. Les principaux composants du service sont les tables, les entités et les propriétés. une table est une collection d’entités. Une entité est un ensemble de propriétés. Chaque entité peut avoir jusqu’à 252 propriétés, qui sont toutes des paires nom-valeur. Cet article suppose que vous êtes déjà familiarisé avec les concepts du service Stockage Table Azure. Pour plus d’informations, consultez [Présentation du modèle de données du service de Table](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) et [Prise en main d’Azure Table Storage à l’aide de .NET](../../cosmos-db/tutorial-develop-table-dotnet.md).

Cet article sur les procédures décrit les opérations courantes liées à Stockage Table Azure. Vous allez apprendre à effectuer les actions suivantes : 

> [!div class="checklist"]
> * Créer une table
> * Récupération d’une table
> * Ajouter des entités de table
> * Interrogation d’une table
> * Suppression d’entités de table
> * Suppression d’une table

Cet article sur les procédures vous montre comment créer un nouveau compte de stockage Azure dans un nouveau groupe de ressources afin que vous puissiez le supprimer facilement quand vous avez terminé. Si vous préférez, vous pouvez utiliser un compte de stockage existant à la place.

Les exemples nécessitent les modules Az PowerShell `Az.Storage (1.1.0 or greater)` et `Az.Resources (1.2.0 or greater)`. Dans une fenêtre PowerShell, exécutez `Get-Module -ListAvailable Az*` pour trouver la version. Si aucune information n’est affichée ou que vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Pour utiliser cette fonctionnalité Azure à partir de PowerShell, vous devez installer le module `Az`. La version actuelle de `AzTable` n’est pas compatible avec l’ancien module AzureRM.
> Si nécessaire, suivez les [dernières instructions d’installation du module Az](/powershell/azure/install-az-ps).

Après avoir installé ou mis à jour Azure PowerShell, vous devez installer le module **AzTable** , qui contient les commandes permettant de gérer les entités. Pour installer ce module, exécutez PowerShell en tant qu’administrateur et utilisez la commande **Install-Module**.

> [!IMPORTANT]
> Pour des raisons de compatibilité de nom de module, nous continuons de publier ce même module sous l’ancien nom `AzureRmStorageTables` dans PowerShell Gallery. Ce document fait uniquement référence au nouveau nom.

```powershell
Install-Module AzTable
```

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la commande `Add-AzAccount` et suivez les instructions à l’écran.

```powershell
Add-AzAccount
```

## <a name="retrieve-list-of-locations"></a>Récupérer la liste des régions

Si vous ne savez pas quelle localisation utiliser, affichez la liste des localisations disponibles. Dans la liste, trouvez celle que vous souhaitez utiliser. Ces exemples utilisent **eastus**. Stockez cette valeur dans la variable **location** pour une utilisation ultérieure.

```powershell
Get-AzLocation | select Location
$location = "eastus"
```

## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Stockez le nom du groupe de ressources dans une variable pour une utilisation ultérieure. Dans cet exemple, un groupe de ressources nommé *pshtablesrg* est créé dans la région *eastus*.

```powershell
$resourceGroup = "pshtablesrg"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>Créer un compte de stockage

Créez un compte de stockage universel standard avec un stockage localement redondant (LRS) à l’aide de [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount). Veillez à spécifier un nom de compte de stockage unique. Ensuite, obtenez le contexte qui représente le compte de stockage. Si un compte de stockage est utilisé, vous pouvez référencer le contexte pour vous éviter d’avoir à entrer maintes fois vos informations d’identification.

```powershell
$storageAccountName = "pshtablestorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind Storage

$ctx = $storageAccount.Context
```

## <a name="create-a-new-table"></a>Créer une table

Pour créer une table, utilisez l’applet de commande [New-AzStorageTable](/powershell/module/az.storage/New-AzStorageTable). Dans cet exemple, la table est appelée `pshtesttable`.

```powershell
$tableName = "pshtesttable"
New-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="retrieve-a-list-of-tables-in-the-storage-account"></a>Récupérer une liste de tables dans le compte de stockage

Récupérez la liste de tables dans le compte de stockage à l’aide de [Get-AzStorageTable](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
Get-AzStorageTable –Context $ctx | select Name
```

## <a name="retrieve-a-reference-to-a-specific-table"></a>Récupérer une référence à une table spécifique

Pour effectuer des opérations sur une table, vous avez besoin d’une référence à cette table. Obtenez une référence à l’aide de [Get-AzStorageTable](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
$storageTable = Get-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="reference-cloudtable-property-of-a-specific-table"></a>Référencer la propriété CloudTable d’une table spécifique

> [!IMPORTANT]
> L’utilisation de CloudTable est obligatoire quand vous utilisez le module PowerShell **AzTable**. Appelez la commande **Get-AzStorageTable** pour obtenir la référence à cet objet. Cette commande crée aussi la table si elle n’existe pas déjà.

Pour effectuer des opérations sur une table à l’aide de **AzTable** , vous avez besoin d’une référence à la propriété CloudTable d’une table spécifique.

```powershell
$cloudTable = (Get-AzStorageTable –Name $tableName –Context $ctx).CloudTable
```

[!INCLUDE [storage-table-entities-powershell-include](../../../includes/storage-table-entities-powershell-include.md)]

## <a name="delete-a-table"></a>Suppression d’une table

Pour supprimer une table, utilisez [Remove-AzStorageTable](/powershell/module/az.storage/Remove-AzStorageTable). Cette applet de commande supprime la table, données comprises.

```powershell
Remove-AzStorageTable –Name $tableName –Context $ctx

# Retrieve the list of tables to verify the table has been removed.
Get-AzStorageTable –Context $Ctx | select Name
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez créé un groupe de ressources et un compte de stockage au début de cet article sur les procédures, vous pouvez supprimer toutes les ressources que vous avez créées dans cet exercice en supprimant le groupe de ressources. Cette commande supprime toutes les ressources contenues dans le groupe de ressources, ainsi que le groupe lui-même.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Cet article sur les procédures vous a présenté des opérations Stockage Table Azure courantes réalisables avec PowerShell. Vous avez notamment appris à effectuer les tâches suivantes : 

> [!div class="checklist"]
> * Créer une table
> * Récupération d’une table
> * Ajouter des entités de table
> * Interrogation d’une table
> * Suppression d’entités de table
> * Suppression d’une table

Pour plus d’informations, consultez les articles suivants :

* [Applets de commande PowerShell - Stockage](/powershell/module/az.storage#storage)

* [Utilisation de Tables Azure à partir de PowerShell – Module PS AzureRmStorageTable/AzTable v2.0](https://paulomarquesc.github.io/working-with-azure-storage-tables-from-powershell)

* [Microsoft Azure Storage Explorer](../../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.