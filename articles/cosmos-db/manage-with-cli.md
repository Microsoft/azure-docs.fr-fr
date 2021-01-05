---
title: Gérer les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide d’Azure CLI
description: Gérez les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide d’Azure CLI.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.openlocfilehash: b13f5bfffced9afd80663d606e30e028e52643ac
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563833"
---
# <a name="manage-azure-cosmos-core-sql-api-resources-using-azure-cli"></a>Gérer les ressources de l’API Azure Cosmos DB Core (SQL) à l’aide d’Azure CLI
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le guide suivant décrit les commandes courantes permettant d’automatiser la gestion de vos comptes, bases de données et conteneurs Azure Cosmos DB à l’aide de l’interface Azure CLI. Des pages d’informations de référence pour toutes les commandes Azure Cosmos DB CLI sont disponibles dans les [Informations de référence sur Azure CLI](/cli/azure/cosmosdb). Vous trouverez également des exemples dans [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md), notamment sur la création et la gestion de comptes, bases de données et conteneurs Cosmos DB pour les API MongoDB, Gremlin, Cassandra et Table.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.12.1 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

Pour obtenir des exemples d’Azure CLI pour d’autres API, consultez [Exemples de CLI pour Cassandra](cli-samples-cassandra.md), [Exemples de CLI pour l’API MongoDB](cli-samples-mongodb.md), [Exemples de CLI pour Gremlin](cli-samples-gremlin.md), [Exemples de CLI pour table](cli-samples-table.md)

> [!IMPORTANT]
> Les ressources Azure Cosmos DB ne peuvent pas être renommées, car cela enfreint les règles de traitement des URI de ressources par Azure Resource Manager.

## <a name="azure-cosmos-accounts"></a>Comptes Azure Cosmos

Les sections suivantes montrent comment gérer le compte Azure Cosmos, et notamment :

* [Créer un compte Azure Cosmos](#create-an-azure-cosmos-db-account)
* [Ajouter ou supprimer des régions](#add-or-remove-regions)
* [Activer les écritures multirégions](#enable-multiple-write-regions)
* [Définir la priorité de basculement régionale](#set-failover-priority)
* [Activer le basculement automatique](#enable-automatic-failover)
* [Déclencher un basculement manuel](#trigger-manual-failover)
* [Répertorier les clés de compte](#list-account-keys)
* [Répertorier les clés de compte en lecture seule](#list-read-only-account-keys)
* [Répertorier les chaînes de connexion](#list-connection-strings)
* [Régénérer la clé de compte](#regenerate-account-key)

### <a name="create-an-azure-cosmos-db-account"></a>Création d’un compte Azure Cosmos DB

Créez un compte Azure Cosmos DB avec prise en charge de l'API SQL et de la cohérence de session dans les régions USA Ouest 2 et USA Est 2 :

> [!IMPORTANT]
> Le nom du compte Azure Cosmos doit être en minuscules et comporter moins de 44 caractères.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount' #needs to be lower case and less than 44 characters

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --default-consistency-level Session \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False
```

### <a name="add-or-remove-regions"></a>Ajouter ou supprimer des régions

Créez un compte Azure Cosmos avec deux régions, ajoutez une région et supprimez-en une.

> [!NOTE]
> Vous ne pouvez pas ajouter ou supprimer simultanément des régions `locations` et modifier d’autres propriétés d’un compte Azure Cosmos. La modification des régions doit être effectuée en tant qu’opération distincte de toute autre modification apportée à la ressource de compte.
> [!NOTE]
> Cette commande vous permet d’ajouter ou de supprimer des régions, mais ne vous permet pas de modifier des priorités de basculement ni de déclencher un basculement manuel. Consultez [Définir la priorité de basculement](#set-failover-priority) et [Déclencher un basculement manuel](#trigger-manual-failover).

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Create an account with 2 regions
az cosmosdb create --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False

# Add a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False \
    --locations regionName="South Central US" failoverPriority=2 isZoneRedundant=False

# Remove a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False
```

### <a name="enable-multiple-write-regions"></a>Activer plusieurs régions d'écriture

Activer les écritures multirégions pour un compte Cosmos

```azurecli-interactive
# Update an Azure Cosmos account from single write region to multiple write regions
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-multiple-write-locations true
```

### <a name="set-failover-priority"></a>Définir la priorité de basculement

Définir la priorité de basculement d'un compte Azure Cosmos configuré pour le basculement automatique

```azurecli-interactive
# Assume region order is initially 'West US 2'=0 'East US 2'=1 'South Central US'=2 for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Make South Central US the next region to fail over to instead of East US 2
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'West US 2=0' 'South Central US=1' 'East US 2=2'
```

### <a name="enable-automatic-failover"></a>Activer le basculement automatique

```azurecli-interactive
# Enable automatic failover on an existing account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-automatic-failover true
```

### <a name="trigger-manual-failover"></a>Déclencher un basculement manuel

> [!CAUTION]
> La modification de la région avec Priority = 0 déclenchera un basculement manuel pour un compte Azure Cosmos. Tout autre changement de priorité ne déclenche pas de basculement.

```azurecli-interactive
# Assume region order is initially 'West US 2=0' 'East US 2=1' 'South Central US=2' for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Trigger a manual failover to promote East US 2 as new write region
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'East US 2=0' 'South Central US=1' 'West US 2=2'
```

### <a name="list-all-account-keys"></a><a id="list-account-keys"></a> Répertorier toutes les clés de compte

Obtenez toutes les clés d'un compte Cosmos.

```azurecli-interactive
# List all account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
   -n $accountName \
   -g $resourceGroupName
```

### <a name="list-read-only-account-keys"></a>Répertorier les clés de compte en lecture seule

Obtenez toutes les clés en lecture seule d'un compte Cosmos.

```azurecli-interactive
# List read-only account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type read-only-keys
```

### <a name="list-connection-strings"></a>Répertorier les chaînes de connexion

Obtenez les chaînes de connexion d'un compte Cosmos.

```azurecli-interactive
# List connection strings
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type connection-strings
```

### <a name="regenerate-account-key"></a>Régénérer la clé de compte

Régénérez une nouvelle clé pour un compte Cosmos.

```azurecli-interactive
# Regenerate secondary account keys
# key-kind values: primary, primaryReadonly, secondary, secondaryReadonly
az cosmosdb keys regenerate \
    -n $accountName \
    -g $resourceGroupName \
    --key-kind secondary
```

## <a name="azure-cosmos-db-database"></a>Base de données Azure Cosmos DB

Les sections suivantes montrent comment gérer la base de données Azure Cosmos DB, et notamment de :

* [Créer une base de données](#create-a-database)
* [Créer une base de données avec débit partagé](#create-a-database-with-shared-throughput)
* [Migrer le débit d’une base de données vers la mise à l’échelle automatique](#migrate-a-database-to-autoscale-throughput)
* [Modifier le débit de la base de données](#change-database-throughput)
* [Empêcher la suppression d’une base de données](#prevent-a-database-from-being-deleted)

### <a name="create-a-database"></a>Création d'une base de données

Créez une base de données Cosmos DB.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName
```

### <a name="create-a-database-with-shared-throughput"></a>Créer une base de données avec débit partagé

Créez une base de données Cosmos avec débit partagé.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
throughput=400

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $throughput
```

### <a name="migrate-a-database-to-autoscale-throughput"></a>Migrer le débit d’une base de données vers la mise à l’échelle automatique

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

# Migrate to autoscale throughput
az cosmosdb sql database throughput migrate \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    -t 'autoscale'

# Read the new autoscale max throughput
az cosmosdb sql database throughput show \
    -g $resourceGroupName \
    -a $accountName \
    -n $databaseName \
    --query resource.autoscaleSettings.maxThroughput \
    -o tsv
```

### <a name="change-database-throughput"></a>Modifier le débit de la base de données

Augmentez le débit d'une base de données Cosmos de 1 000 RU/s.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql database throughput show \
    -g $resourceGroupName -a $accountName -n $databaseName \
    --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql database throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $newRU
```

### <a name="prevent-a-database-from-being-deleted"></a>Empêcher la suppression d’une base de données

Placez un verrou de suppression de ressource Azure sur une base de données pour empêcher sa suppression. Cette fonctionnalité nécessite de verrouiller le compte Cosmos afin qu’il ne puisse pas être modifié par les Kits de développement logiciel (SDK) de plan de données. Pour plus d’informations, consultez [Prévention des modifications par les Kits de développement logiciel (SDK)](role-based-access-control.md#prevent-sdk-changes). Les verrous de ressources Azure peuvent également empêcher la modification d’une ressource en spécifiant un type de verrou `ReadOnly`. Pour une base de données Cosmos, ils peuvent être utilisés pour empêcher la modification du débit.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
databaseLockName="$databaseName-Lock"

# Create a delete lock on database
az lock create --name $databaseLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/sqlDatabases \
    --lock-type $lockType \
    --parent $databaseParent \
    --resource $databaseName

# Delete lock on database
lockid=$(az lock show --name $databaseLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/sqlDatabases \
        --resource $databaseName \
        --parent $databaseParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="azure-cosmos-db-container"></a>Conteneur Azure Cosmos DB

Les sections suivantes montrent comment gérer le conteneur Azure Cosmos DB, et notamment de :

* [Créer un conteneur](#create-a-container)
* [Créer un conteneur avec mise à l’échelle automatique](#create-a-container-with-autoscale)
* [Créer un conteneur avec durée de vie (TTL) activée](#create-a-container-with-ttl)
* [Créer un conteneur avec stratégie d’index personnalisée](#create-a-container-with-a-custom-index-policy)
* [Modifier le débit d’un conteneur](#change-container-throughput)
* [Migrer le débit d’un conteneur vers la mise à l’échelle automatique](#migrate-a-container-to-autoscale-throughput)
* [Empêcher la suppression d’un conteneur](#prevent-a-container-from-being-deleted)

### <a name="create-a-container"></a>Créez un conteneur.

Créez un conteneur Cosmos avec la stratégie d'index, la clé de partition et le débit de 400 RU/s par défaut.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput
```

### <a name="create-a-container-with-autoscale"></a>Créer un conteneur avec mise à l’échelle automatique

Créez un conteneur Cosmos avec la stratégie d’index, la clé de partition et le débit avec mise à l’échelle automatique de 4000 RU/s par défaut.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
maxThroughput=4000

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --max-throughput $maxThroughput
```

### <a name="create-a-container-with-ttl"></a>Créer un conteneur avec durée de vie (TTL)

Créez un conteneur Cosmos avec durée de vie (TTL) activée.

```azurecli-interactive
# Create an Azure Cosmos container with TTL of one day
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

az cosmosdb sql container update \
    -g $resourceGroupName \
    -a $accountName \
    -d $databaseName \
    -n $containerName \
    --ttl=86400
```

### <a name="create-a-container-with-a-custom-index-policy"></a>Créer un conteneur avec stratégie d'index personnalisée

Créez un conteneur Cosmos avec stratégie d'index personnalisée, index spatial, index composite, clé de partition et débit de 400 RU/s.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

# Define the index policy for the container, include spatial and composite indexes
idxpolicy=$(cat << EOF
{
    "indexingMode": "consistent",
    "includedPaths": [
        {"path": "/*"}
    ],
    "excludedPaths": [
        { "path": "/headquarters/employees/?"}
    ],
    "spatialIndexes": [
        {"path": "/*", "types": ["Point"]}
    ],
    "compositeIndexes":[
        [
            { "path":"/name", "order":"ascending" },
            { "path":"/age", "order":"descending" }
        ]
    ]
}
EOF
)
# Persist index policy to json file
echo "$idxpolicy" > "idxpolicy-$uniqueId.json"


az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput \
    --idx @idxpolicy-$uniqueId.json

# Clean up temporary index policy file
rm -f "idxpolicy-$uniqueId.json"
```

### <a name="change-container-throughput"></a>Modifier le débit d’un conteneur

Augmentez le débit d'un conteneur Cosmos de 1 000 RU/s.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql container throughput show \
    -g $resourceGroupName -a $accountName -d $databaseName \
    -n $containerName --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql container throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    --throughput $newRU
```

### <a name="migrate-a-container-to-autoscale-throughput"></a>Migrer le débit d’un conteneur vers la mise à l’échelle automatique

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

# Migrate to autoscale throughput
az cosmosdb sql container throughput migrate \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    -t 'autoscale'

# Read the new autoscale max throughput
az cosmosdb sql container throughput show \
    -g $resourceGroupName \
    -a $accountName \
    -d $databaseName \
    -n $containerName \
    --query resource.autoscaleSettings.maxThroughput \
    -o tsv
```

### <a name="prevent-a-container-from-being-deleted"></a>Empêcher la suppression d’un conteneur

Placez un verrou de suppression de ressource Azure sur un conteneur pour empêcher sa suppression. Cette fonctionnalité nécessite de verrouiller le compte Cosmos afin qu’il ne puisse pas être modifié par les Kits de développement logiciel (SDK) de plan de données. Pour plus d’informations, consultez [Prévention des modifications par les Kits de développement logiciel (SDK)](role-based-access-control.md#prevent-sdk-changes). Les verrous de ressources Azure peuvent également empêcher la modification d’une ressource en spécifiant un type de verrou `ReadOnly`. Pour un conteneur Cosmos, ils peuvent être utilisés pour empêcher la modification du débit ou de toute autre propriété.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
containerParent="databaseAccounts/$accountName/sqlDatabases/$databaseName"
containerLockName="$containerName-Lock"

# Create a delete lock on container
az lock create --name $containerLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/containers \
    --lock-type $lockType \
    --parent $containerParent \
    --resource $containerName

# Delete lock on container
lockid=$(az lock show --name $containerLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/containers \
        --resource-name $containerName \
        --parent $containerParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez :

* [Installation de l’interface de ligne de commande Azure](/cli/azure/install-azure-cli)
* [Référence de ligne de Commande](/cli/azure/cosmosdb)
* [Autres exemples Azure CLI pour Azure Cosmos DB](cli-samples.md)