---
title: Gérer les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide de PowerShell
description: Gérez les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide de PowerShell.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: seodec18
ms.openlocfilehash: 2d99a760e3bf44febcee05bf9827479616bf7bd6
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93333424"
---
# <a name="manage-azure-cosmos-db-core-sql-api-resources-using-powershell"></a>Gérer les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide de PowerShell
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le guide suivant décrit comment utiliser PowerShell pour générer des scripts et automatiser la gestion des ressources de l’API Azure Cosmos DB Core (SQL), notamment les comptes, les bases de données, les conteneurs et les débits Cosmos. Pour obtenir des applets de commande PowerShell pour d’autres API, consultez les pages [Exemples Azure PowerShell pour Azure Cosmos DB – API Cassandra](powershell-samples-cassandra.md), [Exemples Azure PowerShell destinés à l’API Azure Cosmos DB pour MongoDB](powershell-samples-mongodb.md), [Exemples Azure PowerShell pour Azure Cosmos DB – API Gremlin](powershell-samples-gremlin.md), [Exemples Azure PowerShell pour l’API Table Azure Cosmos DB](powershell-samples-table.md)

> [!NOTE]
> Les exemples de cet article utilisent les applets de commande de gestion [Az.CosmosDB](/powershell/module/az.cosmosdb). Consultez la page de référence de l’API [Az.CosmosDB](/powershell/module/az.cosmosdb) pour prendre connaissance des dernières modifications.

Pour une gestion multiplateforme d’Azure Cosmos DB, vous pouvez utiliser les applets de commande `Az` et `Az.CosmosDB` avec [PowerShell multiplateforme](/powershell/scripting/install/installing-powershell) ainsi que l’interface [Azure CLI](manage-with-cli.md), l’[API REST][rp-rest-api] ou le [portail Azure](create-sql-api-dotnet.md#create-account).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="getting-started"></a>Mise en route

Suivez les instructions indiquées dans [Guide pratique pour installer et configurer Azure PowerShell][powershell-install-configure] pour installer votre compte Azure dans PowerShell et vous y connecter.

> [!IMPORTANT]
> Les ressources Azure Cosmos DB ne peuvent pas être renommées, car cela enfreint les règles de traitement des URI de ressources par Azure Resource Manager.

## <a name="azure-cosmos-accounts"></a>Comptes Azure Cosmos

Les sections suivantes montrent comment gérer le compte Azure Cosmos, et notamment :

* [Créer un compte Azure Cosmos](#create-account)
* [Mettre à jour un compte Azure Cosmos](#update-account)
* [Lister tous les comptes Azure Cosmos d’un abonnement](#list-accounts)
* [Obtenir un compte Azure Cosmos](#get-account)
* [Supprimer un compte Azure Cosmos](#delete-account)
* [Mettre à jour les balises pour un compte Azure Cosmos](#update-tags)
* [Répertorier les clés pour un compte Azure Cosmos](#list-keys)
* [Régénérer les clés pour un compte Azure Cosmos](#regenerate-keys)
* [Répertorier les chaînes de connexion d’un compte Azure Cosmos](#list-connection-strings)
* [Modifier la priorité de basculement pour un compte Azure Cosmos](#modify-failover-priority)
* [Déclencher un basculement manuel pour un compte Azure Cosmos](#trigger-manual-failover)
* [Lister les verrous de ressources sur un compte Azure Cosmos DB](#list-account-locks)

### <a name="create-an-azure-cosmos-account"></a><a id="create-account"></a> Créer un compte Azure Cosmos

Cette commande crée un compte de base de données Azure Cosmos DB avec [plusieurs régions][distribute-data-globally], un [basculement automatique](how-to-manage-database-account.md#automatic-failover) et une [stratégie de cohérence](consistency-levels.md) d’obsolescence limitée.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "BoundedStaleness"
$maxStalenessInterval = 300
$maxStalenessPrefix = 100000
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$true `
    -DefaultConsistencyLevel $consistencyLevel `
    -MaxStalenessIntervalInSeconds $maxStalenessInterval `
    -MaxStalenessPrefix $maxStalenessPrefix
```

* `$resourceGroupName` Groupe de ressources Azure dans lequel déployer le compte Cosmos. Il doit déjà exister.
* `$locations` Les régions du compte de base de données, la région avec `FailoverPriority 0` est la région d’écriture.
* `$accountName` Nom du compte Azure Cosmos. Il doit être unique, en minuscules, comporter uniquement des caractères alphanumériques et « - », et sa longueur doit être comprise entre 3 et 31 caractères.
* `$apiKind` Type de compte Cosmos à créer. Pour plus d’informations, consultez [API dans Cosmos DB](introduction.md#simplified-application-development).
* `$consistencyPolicy`, `$maxStalenessInterval` et `$maxStalenessPrefix` Niveau de cohérence par défaut et paramètres du compte Azure Cosmos. Pour plus d’informations, consultez [Niveaux de cohérence dans Azure Cosmos DB](consistency-levels.md).

Les comptes Azure Cosmos peuvent être configurés avec un pare-feu IP, des points de terminaison de service Réseau virtuel et des points de terminaison privés. Pour plus d’informations sur la configuration du pare-feu IP pour Azure Cosmos DB, consultez [Configurer le pare-feu IP](how-to-configure-firewall.md). Pour savoir comment activer des points de terminaison de service pour Azure Cosmos DB, consultez [Configurer l’accès à partir de réseaux virtuels](how-to-configure-vnet-service-endpoint.md). Pour savoir comment activer des points de terminaison privés pour Azure Cosmos DB, consultez [Configurer l’accès à partir points de terminaison privés](how-to-configure-private-endpoints.md).

### <a name="list-all-azure-cosmos-accounts-in-a-resource-group"></a><a id="list-accounts"></a> Lister tous les comptes Azure Cosmos d’un groupe de ressources

Cette commande liste tous les comptes Azure Cosmos d’un groupe de ressources.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName
```

### <a name="get-the-properties-of-an-azure-cosmos-account"></a><a id="get-account"></a> Obtenir les propriétés d’un compte Azure Cosmos

Cette commande vous permet d’obtenir les propriétés d’un compte Azure Cosmos existant.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName
```

### <a name="update-an-azure-cosmos-account"></a><a id="update-account"></a> Mettre à jour un compte Azure Cosmos

Cette commande vous permet de mettre à jour les propriétés de votre compte de base de données Azure Cosmos DB. Les propriétés pouvant être mises à jour sont les suivantes :

* Ajout ou suppression de régions
* Modification de la stratégie de cohérence par défaut
* Modification du filtre de plage IP
* Modification des configurations de Réseau virtuel
* Activer les écritures multirégions

> [!NOTE]
> Vous ne pouvez pas ajouter ou supprimer simultanément des régions (`locations`) et changer d’autres propriétés pour un compte Azure Cosmos. La modification des régions est une opération qui doit être effectuée séparément de toute autre modification apportée au compte.
> [!NOTE]
> Cette commande vous permet d’ajouter ou de supprimer des régions, mais ne vous permet pas de modifier des priorités de basculement ni de déclencher un basculement manuel. Consultez [Modifier la priorité de basculement](#modify-failover-priority) et [Déclencher un basculement manuel](#trigger-manual-failover).

```azurepowershell-interactive
# Create account with two regions
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$apiKind = "Sql"
$consistencyLevel = "Session"
$enableAutomaticFailover = $true
$locations = @()
$locations += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locations += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0

# Create the Cosmos DB account
New-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -LocationObject $locations `
    -Name $accountName `
    -ApiKind $apiKind `
    -EnableAutomaticFailover:$enableAutomaticFailover `
    -DefaultConsistencyLevel $consistencyLevel

# Add a region to the account
$locationObject2 = @()
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "West US" -FailoverPriority 1 -IsZoneRedundant 0
$locationObject2 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 2 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject2

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
Write-Host "When region was added, press any key to continue."
$HOST.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown") | OUT-NULL
$HOST.UI.RawUI.Flushinputbuffer()

# Remove West US region from the account
$locationObject3 = @()
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "East US" -FailoverPriority 0 -IsZoneRedundant 0
$locationObject3 += New-AzCosmosDBLocationObject -LocationName "South Central US" -FailoverPriority 1 -IsZoneRedundant 0

Update-AzCosmosDBAccountRegion `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -LocationObject $locationObject3

Write-Host "Update-AzCosmosDBAccountRegion returns before the region update is complete."
Write-Host "Check account in Azure portal or using Get-AzCosmosDBAccount for region status."
```

### <a name="enable-multiple-write-regions-for-an-azure-cosmos-account"></a><a id="multi-region-writes"></a> Activer plusieurs régions d’écriture pour un compte Azure Cosmos

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $false
$enableMultiMaster = $true

# First disable automatic failover - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover

# Now enable multi-region writes
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster
```

### <a name="delete-an-azure-cosmos-account"></a><a id="delete-account"></a> Supprimer un compte Azure Cosmos

Cette commande permet de supprimer un compte Azure Cosmos existant.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Remove-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -PassThru:$true
```

### <a name="update-tags-of-an-azure-cosmos-account"></a><a id="update-tags"></a> Mettre à jour les balises pour un compte Azure Cosmos

Cette commande permet de définir les [balises de ressources Azure][azure-resource-tags] pour un compte Azure Cosmos. Les balises peuvent être définies aussi bien au moment de la création du compte à l’aide de `New-AzCosmosDBAccount` que de la mise à jour du compte à l’aide de `Update-AzCosmosDBAccount`.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$tags = @{dept = "Finance"; environment = "Production";}

Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Tag $tags
```

### <a name="list-account-keys"></a><a id="list-keys"></a> Liste des clés de comptes

Quand vous créez un compte Azure Cosmos, le service génère deux clés d’accès principales qui peuvent être utilisées pour l’authentification au moment d’accéder au compte Azure Cosmos. Des clés en lecture seule pour l’authentification des opérations en lecture seule sont aussi générées.
En fournissant deux clés d’accès, Azure Cosmos DB vous permet de procéder à la regénération et à la rotation d’une seule clé à la fois sans imposer d’interruption à votre compte Azure Cosmos DB.
Les comptes Cosmos DB ont deux clés en lecture et écriture (primaire et secondaire) et deux clés en lecture seule (primaire et secondaire).

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "Keys"
```

### <a name="list-connection-strings"></a><a id="list-connection-strings"></a> Répertorier les chaînes de connexion

La commande suivante permet de récupérer des chaînes de connexion pour connecter des applications au compte Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -Type "ConnectionStrings"
```

### <a name="regenerate-account-keys"></a><a id="regenerate-keys"></a> Régénérer des clés de compte

Les clés d’accès à un compte Azure Cosmos doivent être régulièrement regénérées pour préserver la sécurité des connexions. Des clés d’accès primaire et secondaire sont affectées au compte. Les clients peuvent ainsi conserver un accès pendant la regénération d’une clé à la fois.
Il existe quatre types de clés pour un compte Azure Cosmos (Primary, Secondary, PrimaryReadonly et SecondaryReadonly)

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup" # Resource Group must already exist
$accountName = "mycosmosaccount" # Must be all lower case
$keyKind = "primary" # Other key kinds: secondary, primaryReadOnly, secondaryReadOnly

New-AzCosmosDBAccountKey `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -KeyKind $keyKind
```

### <a name="enable-automatic-failover"></a><a id="enable-automatic-failover"></a> Activer le basculement automatique

La commande suivante permet de définir le basculement automatique d’un compte Cosmos DB vers sa région secondaire si la région primaire devient indisponible.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$enableAutomaticFailover = $true
$enableMultiMaster = $false

# First disable multi-region writes - cannot have both automatic
# failover and multi-region writes on an account
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableMultipleWriteLocations:$enableMultiMaster

# Now enable automatic failover
Update-AzCosmosDBAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -EnableAutomaticFailover:$enableAutomaticFailover
```

### <a name="modify-failover-priority"></a><a id="modify-failover-priority"></a> Modifier la priorité de basculement

Pour les comptes configurés avec le basculement automatique, vous pouvez modifier l’ordre dans lequel Cosmos promeut les réplicas secondaires en réplicas principaux en cas d’indisponibilité de ce dernier.

Dans l’exemple ci-dessous, partez du principe que la priorité de basculement actuelle est `West US 2 = 0`, `East US 2 = 1`, `South Central US = 2`. La commande la fait passer à `West US 2 = 0`, `South Central US = 1`, `East US 2 = 2`.

> [!CAUTION]
> Le changement d’endroit pour `failoverPriority=0` déclenche le basculement manuel d’un compte Azure Cosmos. Tout autre changement de priorité ne déclenche pas de basculement.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("West US 2", "South Central US", "East US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="trigger-manual-failover"></a><a id="trigger-manual-failover"></a> Déclencher un basculement manuel

Pour les comptes configurés avec le basculement manuel, vous pouvez faire basculer n’importe quel réplica secondaire et le promouvoir en réplica principal en affectant `failoverPriority=0`. Cette opération permet de lancer un test de reprise d’activité de façon à tester la planification de reprise d’activité.

Pour l’exemple ci-dessous, supposons que le compte a actuellement la priorité de basculement `West US 2 = 0` et `East US 2 = 1`, et inversons les régions.

> [!CAUTION]
> Le changement de `locationName` pour `failoverPriority=0` déclenche un basculement manuel pour un compte Azure Cosmos. Tout autre changement de priorité ne déclenche pas de basculement.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$locations = @("East US 2", "West US 2") # Regions ordered by UPDATED failover priority

Update-AzCosmosDBAccountFailoverPriority `
    -ResourceGroupName $resourceGroupName `
    -Name $accountName `
    -FailoverPolicy $locations
```

### <a name="list-resource-locks-on-an-azure-cosmos-db-account"></a><a id="list-account-locks"></a> Lister les verrous de ressources sur un compte Azure Cosmos DB

Des verrous de ressources peuvent être placés sur les ressources Azure Cosmos DB, y compris les bases de données et les collections. L’exemple ci-dessous montre comment lister tous les verrous de ressources Azure sur un compte Azure Cosmos DB.

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceTypeAccount = "Microsoft.DocumentDB/databaseAccounts"
$accountName = "mycosmosaccount"

Get-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceTypeAccount `
    -ResourceName $accountName
```

## <a name="azure-cosmos-db-database"></a>Base de données Azure Cosmos DB

Les sections suivantes montrent comment gérer la base de données Azure Cosmos DB, et notamment de :

* [Créer une base de données Azure Cosmos DB](#create-db)
* [Créer une base de données Azure Cosmos DB avec débit partagé](#create-db-ru)
* [Obtenir le débit d’une base de données Azure Cosmos DB](#get-db-ru)
* [Migrer le débit de base de données vers la mise à l’échelle automatique](#migrate-db-ru)
* [Lister toutes les bases de données Azure Cosmos DB d’un compte](#list-db)
* [Obtenir une base de données Azure Cosmos DB unique](#get-db)
* [Supprimer une base de données Azure Cosmos DB](#delete-db)
* [Créer un verrou de ressource sur une base de données Azure Cosmos DB pour empêcher sa suppression](#create-db-lock)
* [Supprimer un verrou de ressource sur une base de données Azure Cosmos DB](#remove-db-lock)

### <a name="create-an-azure-cosmos-db-database"></a><a id="create-db"></a>Créer une base de données Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-an-azure-cosmos-db-database-with-shared-throughput"></a><a id="create-db-ru"></a>Créer une base de données Azure Cosmos DB avec débit partagé

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$databaseRUs = 400

New-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -Throughput $databaseRUs
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-database"></a><a id="get-db-ru"></a>Obtenir le débit d’une base de données Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabaseThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

## <a name="migrate-database-throughput-to-autoscale"></a><a id="migrate-db-ru"></a>Migrer le débit de base de données vers la mise à l’échelle automatique

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Invoke-AzCosmosDBSqlDatabaseThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName `
    -ThroughputType Autoscale
```

### <a name="get-all-azure-cosmos-db-databases-in-an-account"></a><a id="list-db"></a>Obtenir toutes les bases de données Azure Cosmos DB d’un compte

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName
```

### <a name="get-a-single-azure-cosmos-db-database"></a><a id="get-db"></a>Obtenir une base de données Azure Cosmos DB unique

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="delete-an-azure-cosmos-db-database"></a><a id="delete-db"></a>Supprimer une base de données Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Remove-AzCosmosDBSqlDatabase `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -Name $databaseName
```

### <a name="create-a-resource-lock-on-an-azure-cosmos-db-database-to-prevent-delete"></a><a id="create-db-lock"></a>Créer un verrou de ressource sur une base de données Azure Cosmos DB pour empêcher sa suppression

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-database"></a><a id="remove-db-lock"></a>Supprimer un verrou de ressource sur une base de données Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$resourceName = "$accountName/$databaseName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="azure-cosmos-db-container"></a>Conteneur Azure Cosmos DB

Les sections suivantes montrent comment gérer le conteneur Azure Cosmos DB, et notamment de :

* [Créer un conteneur Azure Cosmos DB](#create-container)
* [Créer un conteneur Azure Cosmos DB avec une mise à l’échelle automatique](#create-container-autoscale)
* [Créer un conteneur Azure Cosmos DB avec une grande clé de partition](#create-container-big-pk)
* [Obtenir le débit d’un conteneur Azure Cosmos DB](#get-container-ru)
* [Migrer le débit de conteneur vers la mise à l’échelle automatique](#migrate-container-ru)
* [Créer un conteneur Azure Cosmos DB avec une indexation personnalisée](#create-container-custom-index)
* [Créer un conteneur Azure Cosmos DB avec l’indexation désactivée](#create-container-no-index)
* [Créer un conteneur Azure Cosmos DB avec une clé unique et une durée de vie](#create-container-unique-key-ttl)
* [Créer un conteneur Azure Cosmos DB avec la résolution des conflits](#create-container-lww)
* [Lister tous les conteneurs Azure Cosmos DB d’une base de données](#list-containers)
* [Obtenir un conteneur Azure Cosmos DB unique d’une base de données](#get-container)
* [Supprimer un conteneur Azure Cosmos DB](#delete-container)
* [Créer un verrou de ressource sur un conteneur Azure Cosmos DB pour empêcher sa suppression](#create-container-lock)
* [Supprimer un verrou de ressource sur un conteneur Azure Cosmos DB](#remove-container-lock)

### <a name="create-an-azure-cosmos-db-container"></a><a id="create-container"></a>Créer un conteneur Azure Cosmos DB

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and throughput at 400 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$throughput = 400 #minimum = 400

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -Throughput $throughput
```

### <a name="create-an-azure-cosmos-db-container-with-autoscale"></a><a id="create-container-autoscale"></a>Créer un conteneur Azure Cosmos DB avec une mise à l’échelle automatique

```azurepowershell-interactive
# Create an Azure Cosmos DB container with default indexes and autoscale throughput at 4000 RU
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$autoscaleMaxThroughput = 4000 #minimum = 4000

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -AutoscaleMaxThroughput $autoscaleMaxThroughput
```

### <a name="create-an-azure-cosmos-db-container-with-a-large-partition-key-size"></a><a id="create-container-big-pk"></a>Créer un conteneur Azure Cosmos DB avec une clé de partition de grande taille

```azurepowershell-interactive
# Create an Azure Cosmos DB container with a large partition key value (version = 2)
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -PartitionKeyVersion 2
```

### <a name="get-the-throughput-of-an-azure-cosmos-db-container"></a><a id="get-container-ru"></a>Obtenir le débit d’un conteneur Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainerThroughput `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="migrate-container-throughput-to-autoscale"></a><a id="migrate-container-ru"></a>Migrer le débit de conteneur vers la mise à l’échelle automatique

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Invoke-AzCosmosDBSqlContainerThroughputMigration `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -ThroughputType Autoscale
```

### <a name="create-an-azure-cosmos-db-container-with-custom-index-policy"></a><a id="create-container-custom-index"></a>Créer un conteneur Azure Cosmos DB avec une stratégie d’indexation personnalisée

```azurepowershell-interactive
# Create a container with a custom indexing policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$indexPathIncluded = "/*"
$indexPathExcluded = "/myExcludedPath/*"

$includedPathIndex = New-AzCosmosDBSqlIncludedPathIndex -DataType String -Kind Range
$includedPath = New-AzCosmosDBSqlIncludedPath -Path $indexPathIncluded -Index $includedPathIndex

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IncludedPath $includedPath `
    -ExcludedPath $indexPathExcluded `
    -IndexingMode Consistent `
    -Automatic $true

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-indexing-turned-off"></a><a id="create-container-no-index"></a>Créer un conteneur Azure Cosmos DB avec l’indexation désactivée

```azurepowershell-interactive
# Create an Azure Cosmos DB container with no indexing
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"

$indexingPolicy = New-AzCosmosDBSqlIndexingPolicy `
    -IndexingMode None

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -IndexingPolicy $indexingPolicy
```

### <a name="create-an-azure-cosmos-db-container-with-unique-key-policy-and-ttl"></a><a id="create-container-unique-key-ttl"></a>Créer un conteneur Azure Cosmos DB avec une stratégie de clé unique et une durée de vie

```azurepowershell-interactive
# Create a container with a unique key policy and TTL of one day
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$uniqueKeyPath = "/myUniqueKeyPath"
$ttlInSeconds = 86400 # Set this to -1 (or don't use it at all) to never expire

$uniqueKey = New-AzCosmosDBSqlUniqueKey `
    -Path $uniqueKeyPath

$uniqueKeyPolicy = New-AzCosmosDBSqlUniqueKeyPolicy `
    -UniqueKey $uniqueKey

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -UniqueKeyPolicy $uniqueKeyPolicy `
    -TtlInSeconds $ttlInSeconds
```

### <a name="create-an-azure-cosmos-db-container-with-conflict-resolution"></a><a id="create-container-lww"></a>Créer un conteneur Azure Cosmos DB avec la résolution des conflits

Pour écrire tous les conflits dans ConflictsFeed et les gérer séparément, passez `-Type "Custom" -Path ""`.

```azurepowershell-interactive
# Create container with last-writer-wins conflict resolution policy
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionPath = "/myResolutionPath"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type LastWriterWins `
    -Path $conflictResolutionPath

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```

Pour créer une stratégie de résolution des conflits afin d’utiliser une procédure stockée, appelez `New-AzCosmosDBSqlConflictResolutionPolicy` et passez les paramètres `-Type` et `-ConflictResolutionProcedure`.

```azurepowershell-interactive
# Create container with custom conflict resolution policy using a stored procedure
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$partitionKeyPath = "/myPartitionKey"
$conflictResolutionSprocName = "mysproc"

$conflictResolutionSproc = "/dbs/$databaseName/colls/$containerName/sprocs/$conflictResolutionSprocName"

$conflictResolutionPolicy = New-AzCosmosDBSqlConflictResolutionPolicy `
    -Type Custom `
    -ConflictResolutionProcedure $conflictResolutionSproc

New-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName `
    -PartitionKeyKind Hash `
    -PartitionKeyPath $partitionKeyPath `
    -ConflictResolutionPolicy $conflictResolutionPolicy
```


### <a name="list-all-azure-cosmos-db-containers-in-a-database"></a><a id="list-containers"></a>Lister tous les conteneurs Azure Cosmos DB d’une base de données

```azurepowershell-interactive
# List all Azure Cosmos DB containers in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName
```

### <a name="get-a-single-azure-cosmos-db-container-in-a-database"></a><a id="get-container"></a>Obtenir un conteneur Azure Cosmos DB unique d’une base de données

```azurepowershell-interactive
# Get a single Azure Cosmos DB container in a database
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Get-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```

### <a name="delete-an-azure-cosmos-db-container"></a><a id="delete-container"></a>Supprimer un conteneur Azure Cosmos DB

```azurepowershell-interactive
# Delete an Azure Cosmos DB container
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"

Remove-AzCosmosDBSqlContainer `
    -ResourceGroupName $resourceGroupName `
    -AccountName $accountName `
    -DatabaseName $databaseName `
    -Name $containerName
```
### <a name="create-a-resource-lock-on-an-azure-cosmos-db-container-to-prevent-delete"></a><a id="create-container-lock"></a>Créer un verrou de ressource sur un conteneur Azure Cosmos DB pour empêcher sa suppression

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"
$lockLevel = "CanNotDelete"

New-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName `
    -LockLevel $lockLevel
```

### <a name="remove-a-resource-lock-on-an-azure-cosmos-db-container"></a><a id="remove-container-lock"></a>Supprimer un verrou de ressource sur un conteneur Azure Cosmos DB

```azurepowershell-interactive
$resourceGroupName = "myResourceGroup"
$resourceType = "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers"
$accountName = "mycosmosaccount"
$databaseName = "myDatabase"
$containerName = "myContainer"
$resourceName = "$accountName/$databaseName/$containerName"
$lockName = "myResourceLock"

Remove-AzResourceLock `
    -ResourceGroupName $resourceGroupName `
    -ResourceType $resourceType `
    -ResourceName $resourceName `
    -LockName $lockName
```

## <a name="next-steps"></a>Étapes suivantes

* [Tous les exemples PowerShell](powershell-samples.md)
* [Comment gérer un compte Azure Cosmos](how-to-manage-database-account.md)
* [Créer un conteneur Azure Cosmos DB](how-to-create-container.md)
* [Configurer la durée de vie dans Azure Cosmos DB](how-to-time-to-live.md)

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[powershell-install-configure]: /powershell/azure/
[scaling-globally]: distribute-data-globally.md#EnableGlobalDistribution
[distribute-data-globally]: distribute-data-globally.md
[azure-resource-groups]: ../azure-resource-manager/management/overview.md#resource-groups
[azure-resource-tags]: ../azure-resource-manager/management/tag-resources.md
[rp-rest-api]: /rest/api/cosmos-db-resource-provider/