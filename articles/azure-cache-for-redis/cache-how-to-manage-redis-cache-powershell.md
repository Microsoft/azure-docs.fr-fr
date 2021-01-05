---
title: Gérer le Cache Azure pour Redis avec Azure PowerShell
description: Découvrez comment effectuer des tâches administratives pour le Cache Azure pour Redis à l'aide d'Azure PowerShell.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 07/13/2017
ms.author: yegu
ms.openlocfilehash: 88e433dbfa87d8ea483789b1fd838c62a6a481c0
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536757"
---
# <a name="manage-azure-cache-for-redis-with-azure-powershell"></a>Gérer le Cache Azure pour Redis avec Azure PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](cache-how-to-manage-redis-cache-powershell.md)
> * [Azure CLI](cache-manage-cli.md)
> 
> 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Cette rubrique décrit comment effectuer des tâches courantes telles que la création, la mise à jour et la mise à l’échelle de vos instances du Cache Azure pour Redis, comment régénérer les clés d'accès et comment afficher des informations sur vos caches. Pour obtenir une liste complète des cmdlets PowerShell du Cache Redis Azure, consultez [Cmdlets du Cache Azure pour Redis](/powershell/module/az.rediscache).

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

Pour en savoir plus sur le modèle de déploiement Classic, consultez [Déploiement Azure Resource Manager et déploiement Classic : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/management/deployment-models.md).

## <a name="prerequisites"></a>Conditions préalables requises
Si vous avez déjà installé Azure PowerShell, vous devez disposer d’Azure PowerShell version 1.0.0 ou ultérieure. Vous pouvez vérifier la version d’Azure PowerShell que vous avez installée à l’aide de cette commande à l’invite de commandes Azure PowerShell.

```azurepowershell
    Get-Module Az | format-table version
```

Tout d’abord, vous devez vous connecter à Azure avec cette commande.

```azurepowershell
    Connect-AzAccount
```

Spécifiez l'adresse de messagerie électronique et le mot de passe de votre compte Azure dans la boîte de dialogue de connexion à Microsoft Azure.

Ensuite, si vous avez plusieurs abonnements, vous devez sélectionner l’abonnement Azure à utiliser. Pour afficher une liste de vos abonnements en cours, exécutez la commande suivante.

```azurepowershell
    Get-AzSubscription | sort SubscriptionName | Select SubscriptionName
```

Pour spécifier l’abonnement, exécutez la commande suivante. Dans l’exemple suivant, le nom de l’abonnement est `ContosoSubscription`.

```azurepowershell
    Select-AzSubscription -SubscriptionName ContosoSubscription
```

Avant de pouvoir utiliser Windows PowerShell avec Azure Resource Manager, vous devez disposer des composants suivants :

* Windows PowerShell, version 3.0 ou 4.0. Pour trouver la version de Windows PowerShell, tapez : `$PSVersionTable` et vérifiez que la valeur de `PSVersion` est 3.0 ou 4.0. Pour installer une version compatible, consultez [Windows Management Framework 3.0](https://www.microsoft.com/download/details.aspx?id=34595).

Pour accéder à l’aide détaillée d’un applet de commande présenté dans ce didacticiel, utilisez l’applet de commande Get-Help.

```azurepowershell
    Get-Help <cmdlet-name> -Detailed
```

Par exemple, pour obtenir de l’aide sur l’applet de commande `New-AzRedisCache` , tapez :

```azurepowershell
    Get-Help New-AzRedisCache -Detailed
```

### <a name="how-to-connect-to-other-clouds"></a>Guide pratique pour se connecter à d’autres clouds
Par défaut, l’environnement Azure est `AzureCloud`, qui représente l’instance globale du cloud Azure. Pour vous connecter à une autre instance, utilisez la commande `Connect-AzAccount` avec le commutateur de ligne de commande `-Environment` ou -`EnvironmentName` accompagné de l’environnement ou du nom d’environnement désiré.

Pour afficher la liste des environnements disponibles, exécutez l’applet de commande `Get-AzEnvironment` .

### <a name="to-connect-to-the-azure-government-cloud"></a>Pour vous connecter au cloud Azure Government
Pour vous connecter au cloud Azure Government, utilisez une des commandes suivantes.

```azurepowershell
    Connect-AzAccount -EnvironmentName AzureUSGovernment
```

or

```azurepowershell
    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureUSGovernment)
```

Pour créer un cache dans le cloud Azure Government, utilisez un des emplacements suivants.

* USGov Virginia
* USGov Iowa

Pour plus d’informations sur le cloud Azure Government, voir [Microsoft Azure Government](https://azure.microsoft.com/features/gov/) et [Guide du développeur Microsoft Azure Government](../azure-government/documentation-government-developer-guide.md).

### <a name="to-connect-to-the-azure-china-cloud"></a>Pour vous connecter au cloud Azure de Chine
Pour vous connecter au cloud Azure de Chine, utilisez une des commandes suivantes.

```azurepowershell
    Connect-AzAccount -EnvironmentName AzureChinaCloud
```

or

```azurepowershell
    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureChinaCloud)
```

Pour créer un cache dans le cloud Azure de Chine, utilisez un des emplacements suivants.

* Chine orientale
* Chine du Nord

Pour plus d’informations sur le cloud Azure de Chine, consultez [AzureChinaCloud pour Azure géré par 21Vianet en Chine](https://www.windowsazure.cn/).

### <a name="to-connect-to-microsoft-azure-germany"></a>Pour se connecter à Microsoft Azure Allemagne
Pour vous connecter à Microsoft Azure Allemagne, utilisez une des commandes suivantes.

```azurepowershell
    Connect-AzAccount -EnvironmentName AzureGermanCloud
```

or

```azurepowershell
    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureGermanCloud)
```

Pour créer un cache dans Microsoft Azure Allemagne, utilisez un des emplacements suivants.

* Centre de l’Allemagne
* Nord-Est de l’Allemagne

Pour plus d’informations sur Microsoft Azure Allemagne, consultez [Microsoft Azure Allemagne](https://azure.microsoft.com/overview/clouds/germany/).

### <a name="properties-used-for-azure-cache-for-redis-powershell"></a>Propriétés utilisées pour le Cache Azure pour Redis avec PowerShell
Le tableau suivant contient les propriétés et les descriptions pour les paramètres fréquemment utilisés lors de la création et de la gestion de vos instances du Cache Azure pour Redis avec Azure PowerShell.

| Paramètre | Description | Default |
| --- | --- | --- |
| Name |Nom du cache | |
| Location |Emplacement du cache | |
| ResourceGroupName |Nom du groupe de ressources dans lequel créer le cache | |
| Size |Taille du cache. Les valeurs valides sont : P1, P2, P3, P4, C0, C1, C2, C3, C4, C5, C6, 250 Mo, 1 Go, 2,5 Go, 6 Go, 13 Go, 26 Go, 53 Go |1 Go |
| ShardCount |Le nombre de partitions à créer lors de la création d'un cache premium avec le clustering activé. Les valeurs valides sont : 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 | |
| SKU |Spécifie la référence du cache. Les valeurs valides sont : De base, Standard, Premium |standard |
| RedisConfiguration |Spécifie les paramètres de configuration de Redis. Pour plus d’informations sur chaque paramètre, consultez le tableau [Propriétés RedisConfiguration](#redisconfiguration-properties) suivant. | |
| EnableNonSslPort |Indique si le port non SSL est activé. |False |
| MaxMemoryPolicy |Ce paramètre est obsolète. Utilisez RedisConfiguration à la place. | |
| StaticIP |Lorsque vous hébergez votre cache dans un réseau virtuel, spécifie une adresse IP unique dans le sous-réseau pour le cache. Si elle est omise, une adresse IP est choisie pour vous dans le sous-réseau. | |
| Subnet |Lorsque vous hébergez votre cache dans un réseau virtuel, spécifie le nom du sous-réseau dans lequel déployer le cache. | |
| VirtualNetwork |Lorsque vous hébergez votre cache dans un réseau virtuel, spécifie l’ID de ressource du réseau virtuel dans lequel déployer le cache. | |
| KeyType |Spécifie la clé d'accès à régénérer lors du renouvellement des clés d'accès. Les valeurs valides sont : Primaire, Secondaire | |

### <a name="redisconfiguration-properties"></a>Propriétés RedisConfiguration
| Propriété | Description | Niveaux de tarification |
| --- | --- | --- |
| rdb-backup-enabled |Indique si [la persistance des données Redis](cache-how-to-premium-persistence.md) est activée |Premium uniquement |
| rdb-storage-connection-string |La chaîne de connexion au compte de stockage pour [la persistance des données Redis](cache-how-to-premium-persistence.md) |Premium uniquement |
| rdb-backup-frequency |La fréquence de sauvegarde pour [la persistance des données Redis](cache-how-to-premium-persistence.md) |Premium uniquement |
| maxmemory-reserved |Configure la [mémoire réservée](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) aux processus sans mise en cache |Standard et Premium |
| maxmemory-policy |Configure la [stratégie d’éviction](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) pour le cache |Tous les niveaux de tarification |
| notify-keyspace-events |Configure les [notifications d’espace de clés](cache-configure.md#keyspace-notifications-advanced-settings) |Standard et Premium |
| hash-max-ziplist-entries |Configure [l’optimisation de la mémoire](https://redis.io/topics/memory-optimization) pour les petites quantités de types de données agrégées |Standard et Premium |
| hash-max-ziplist-value |Configure [l’optimisation de la mémoire](https://redis.io/topics/memory-optimization) pour les petites quantités de types de données agrégées |Standard et Premium |
| set-max-intset-entries |Configure [l’optimisation de la mémoire](https://redis.io/topics/memory-optimization) pour les petites quantités de types de données agrégées |Standard et Premium |
| zset-max-ziplist-entries |Configure [l’optimisation de la mémoire](https://redis.io/topics/memory-optimization) pour les petites quantités de types de données agrégées |Standard et Premium |
| zset-max-ziplist-value |Configure [l’optimisation de la mémoire](https://redis.io/topics/memory-optimization) pour les petites quantités de types de données agrégées |Standard et Premium |
| databases |Configure le nombre de bases de données. Cette propriété ne peut être configurée qu’au moment de la création du cache. |Standard et Premium |

## <a name="to-create-an-azure-cache-for-redis"></a>Créer un Cache Azure pour Redis
Les nouvelles instances du Cache Azure pour Redis sont créées à l’aide de la cmdlet [New-AzRedisCache](/powershell/module/az.rediscache/new-azrediscache) .

> [!IMPORTANT]
> La première fois que vous créez un Cache Azure pour Redis dans un abonnement à l’aide du portail Azure, le portail inscrit l’espace de noms `Microsoft.Cache` pour cet abonnement. Si vous tentez de créer le premier Cache Azure pour Redis dans un abonnement à l’aide de PowerShell, vous devez d’abord inscrire cet espace de noms à l’aide de la commande suivante. Dans le cas contraire, les applets de commande comme `New-AzRedisCache` et `Get-AzRedisCache` échoueront.
> 
> `Register-AzResourceProvider -ProviderNamespace "Microsoft.Cache"`
> 
> 

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `New-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help New-AzRedisCache -detailed

    NAME
        New-AzRedisCache

    SYNOPSIS
        Creates a new Azure Cache for Redis.


    SYNTAX
        New-AzRedisCache -Name <String> -ResourceGroupName <String> -Location <String> [-RedisVersion <String>]
        [-Size <String>] [-Sku <String>] [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort
        <Boolean>] [-ShardCount <Integer>] [-VirtualNetwork <String>] [-Subnet <String>] [-StaticIP <String>]
        [<CommonParameters>]


    DESCRIPTION
        The New-AzRedisCache cmdlet creates a new Azure Cache for Redis.


    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to create.

        -ResourceGroupName <String>
            Name of resource group in which to create the Azure Cache for Redis.

        -Location <String>
            Location in which to create the Azure Cache for Redis.

        -RedisVersion <String>
            RedisVersion is deprecated and will be removed in future release.

        -Size <String>
            Size of the Azure Cache for Redis. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of Azure Cache for Redis. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value, databases.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Cache for Redis. If no value is provided, the default value is false and the
            non-SSL port will be disabled. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        -VirtualNetwork <String>
            The exact ARM resource ID of the virtual network to deploy the Azure Cache for Redis in. Example format: /subscriptions/{
            subid}/resourceGroups/{resourceGroupName}/providers/Microsoft.ClassicNetwork/VirtualNetworks/{vnetName}

        -Subnet <String>
            Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network.

        -StaticIP <String>
            Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

Pour créer un cache avec les paramètres par défaut, exécutez la commande suivante.

```azurepowershell
    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US"
```

`ResourceGroupName`, `Name`, et `Location` sont des paramètres obligatoires, mais les autres sont facultatifs et disposent de valeurs par défaut. L’exécution de la commande précédente crée une instance du Cache Azure pour Redis avec référence standard avec le nom, l’emplacement et le groupe de ressources spécifiés, dont la taille est 1 Go avec le port non SSL désactivé.

Pour créer un cache premium, spécifiez la taille de P1 (de 6 Go à 60 Go), P2 (de 13 Go à 130 Go), P3 (de 26 Go à 260 Go) ou P4 (de 53 Go à 530 Go). Pour activer le clustering, spécifiez un nombre de partitions à l'aide du paramètre `ShardCount`. L'exemple suivant permet de créer un cache premium P1 avec 3 partitions. La taille d’un cache premium P1 est de 6 Go. Puisque nous avons spécifié trois partitions, la taille totale est de 18 Go (3 x 6 Go).

```azurepowershell
    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P1 -ShardCount 3
```

Pour spécifier des valeurs pour le paramètre `RedisConfiguration`, entourez les valeurs dans `{}` en tant que paire clé/valeur telle que `@{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}`. L'exemple suivant permet de créer un cache standard de 1 Go avec la stratégie maxmemory `allkeys-random` et les notifications de keyspace configurées avec `KEA`. Pour plus d’informations, voir [Notifications de keyspace (paramètres avancés)](cache-configure.md#keyspace-notifications-advanced-settings) et [Stratégies de mémoire](cache-configure.md#memory-policies).

```azurepowershell
    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}
```

<a name="databases"></a>

## <a name="to-configure-the-databases-setting-during-cache-creation"></a>Pour configurer les paramètres des bases de données lors de la création du cache
Le paramètre `databases` ne peut être configuré qu’au moment de la création du cache. L’exemple suivant crée un cache premium P3 (26 Go) avec 48 bases de données à l’aide de l’applet de commande [New-AzRedisCache](/powershell/module/az.rediscache/New-azRedisCache) .

```azurepowershell
    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P3 -RedisConfiguration @{"databases" = "48"}
```

Pour plus d’informations sur la propriété `databases` , consultez la section [Configuration du serveur du Cache Azure pour Redis par défaut](cache-configure.md#default-redis-server-configuration). Pour plus d’informations sur la création d’un cache à l’aide de la cmdlet [New-AzRedisCache](/powershell/module/az.rediscache/new-azrediscache), voir la section précédente, Créer un Cache Azure pour Redis.

## <a name="to-update-an-azure-cache-for-redis"></a>Mettre à jour un cache Azure pour Redis
Les instances du Cache Azure pour Redis sont mises à jour à l’aide de la cmdlet [New-AzRedisCache](/powershell/module/az.rediscache/Set-azRedisCache).

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Set-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Set-AzRedisCache -detailed

    NAME
        Set-AzRedisCache

    SYNOPSIS
        Set Azure Cache for Redis updatable parameters.

    SYNTAX
        Set-AzRedisCache -Name <String> -ResourceGroupName <String> [-Size <String>] [-Sku <String>]
        [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort <Boolean>] [-ShardCount
        <Integer>] [<CommonParameters>]

    DESCRIPTION
        The Set-AzRedisCache cmdlet sets Azure Cache for Redis parameters.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to update.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -Size <String>
            Size of the Azure Cache for Redis. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of Azure Cache for Redis. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Cache for Redis. The default value is null and no change will be made to the
            currently configured value. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

L'applet de commande `Set-AzRedisCache` peut être utilisée pour mettre à jour des propriétés telles que les valeurs `Size`, `Sku`, `EnableNonSslPort` et `RedisConfiguration`. 

La commande suivante met à jour le paramètre maxmemory-policy du Cache Azure pour Redis appelé myCache.

```azurepowershell
    Set-AzRedisCache -ResourceGroupName "myGroup" -Name "myCache" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random"}
```

<a name="scale"></a>

## <a name="to-scale-an-azure-cache-for-redis"></a>Mettre à l'échelle un Cache Azure pour Redis
`Set-AzRedisCache` peut être utilisé pour mettre à l’échelle une instance du Cache Azure pour Redis lorsque les propriétés `Size`, `Sku` ou `ShardCount` sont modifiées. 

> [!NOTE]
> La mise à l’échelle d’un cache à l’aide de PowerShell est soumise aux mêmes limites et recommandations que la mise à l’échelle d’un cache à l’aide du portail Azure. Vous pouvez choisir un niveau tarifaire différent avec les restrictions suivantes.
> 
> * Vous ne pouvez pas passer d’un niveau de tarification supérieur à un niveau de tarification inférieur.
> * Vous ne pouvez pas passer d’un cache **Premium** à un cache **Standard** ou **De base** .
> * Vous ne pouvez pas passer d’un cache **Standard** à un cache **De base** .
> * Vous pouvez passer d’un cache **De base** à un cache **Standard** , mais vous ne pouvez pas modifier la taille en même temps. Si vous avez besoin d'une taille différente, vous pouvez effectuer ultérieurement une opération de mise à l'échelle vers la taille voulue.
> * Vous ne pouvez pas passer directement d’un cache **De base** à un cache **Premium** . Vous devez passer du niveau **De base** au niveau **Standard** en une opération de mise à l’échelle, puis du niveau **Standard** au niveau **Premium** en une deuxième opération.
> * Vous ne pouvez pas mettre à l’échelle depuis une taille supérieure vers la taille **C0 (250 Mo)** .
> 
> Pour plus d’informations, voir [Mise à l’échelle du Cache Azure pour Redis](cache-how-to-scale.md).
> 
> 

L’exemple suivant montre comment mettre à l’échelle un cache nommé `myCache` vers un cache de 2,5 Go. Notez que cette commande fonctionne pour un cache De base ou un cache Standard.

```azurepowershell
    Set-AzRedisCache -ResourceGroupName myGroup -Name myCache -Size 2.5GB
```

Une fois cette commande émise, l’état du cache est retourné (ceci est similaire à l’appel `Get-AzRedisCache`). Notez que `ProvisioningState` est `Scaling`.

```azurepowershell
    PS C:\> Set-AzRedisCache -Name myCache -ResourceGroupName myGroup -Size 2.5GB


    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/mygroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Scaling
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 150], [notify-keyspace-events, KEA],
                         [maxmemory-delta, 150]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : mygroup
    PrimaryKey         : ....
    SecondaryKey       : ....
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :
```

Quand l’opération de mise à l’échelle est terminée, `ProvisioningState` passe à `Succeeded`. Si vous devez effectuer une opération de mise à l'échelle associée, comme mettre à l’échelle un cache De base vers un cache Standard, puis changer la taille, vous devez patienter jusqu’à ce que l’opération précédente soit terminée. Dans le cas contraire, vous recevrez une erreur similaire à la suivante.

```azurepowershell
    Set-AzRedisCache : Conflict: The resource '...' is not in a stable state, and is currently unable to accept the update request.
```

## <a name="to-get-information-about-an-azure-cache-for-redis"></a>Obtenir des informations sur un Cache Azure pour Redis
Vous pouvez récupérer des informations sur un cache à l’aide de l’applet de commande [Get-AzRedisCache](/powershell/module/az.rediscache/get-azrediscache).

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Get-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Get-AzRedisCache -detailed

    NAME
        Get-AzRedisCache

    SYNOPSIS
        Gets details about a single cache or all caches in the specified resource group or all caches in the current
        subscription.

    SYNTAX
        Get-AzRedisCache [-Name <String>] [-ResourceGroupName <String>] [<CommonParameters>]

    DESCRIPTION
        The Get-AzRedisCache cmdlet gets the details about a cache or caches depending on input parameters. If both
        ResourceGroupName and Name parameters are provided then Get-AzRedisCache will return details about the
        specific cache name provided.

        If only ResourceGroupName is provided than it will return details about all caches in the specified resource group.

        If no parameters are given than it will return details about all caches the current subscription.

    PARAMETERS
        -Name <String>
            The name of the cache. When this parameter is provided along with ResourceGroupName, Get-AzRedisCache
            returns the details for the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache or caches. If ResourceGroupName is provided with Name
            then Get-AzRedisCache returns the details of the cache specified by Name. If only the ResourceGroup
            parameter is provided, then details for all caches in the resource group are returned.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

Pour retourner des informations sur tous les caches de l’abonnement actuel, exécutez `Get-AzRedisCache` sans aucun paramètre.

```azurepowershell
    Get-AzRedisCache
```

Pour retourner des informations sur tous les caches d’un groupe de ressources spécifique, exécutez `Get-AzRedisCache` avec le paramètre `ResourceGroupName`.

```azurepowershell
    Get-AzRedisCache -ResourceGroupName myGroup
```

Pour retourner des informations sur un cache spécifique, exécutez `Get-AzRedisCache` avec le paramètre `Name` contenant le nom du cache et le paramètre `ResourceGroupName` avec le groupe de ressources contenant ce cache.

```azurepowershell
    PS C:\> Get-AzRedisCache -Name myCache -ResourceGroupName myGroup

    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/myGroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Succeeded
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 62], [notify-keyspace-events, KEA],
                         [maxclients, 1000]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : myGroup
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :
```

## <a name="to-retrieve-the-access-keys-for-an-azure-cache-for-redis"></a>Récupérer les clés d’accès pour une instance du Cache Azure pour Redis
Pour récupérer les clés d’accès de votre cache, vous pouvez utiliser l’applet de commande [Get-AzRedisCacheKey](/powershell/module/az.rediscache/Get-azRedisCacheKey) .

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Get-AzRedisCacheKey`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Get-AzRedisCacheKey -detailed

    NAME
        Get-AzRedisCacheKey

    SYNOPSIS
        Gets the accesskeys for the specified Azure Cache for Redis.


    SYNTAX
        Get-AzRedisCacheKey -Name <String> -ResourceGroupName <String> [<CommonParameters>]

    DESCRIPTION
        The Get-AzRedisCacheKey cmdlet gets the access keys for the specified cache.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

Pour récupérer les clés de votre cache, appelez l’applet de commande `Get-AzRedisCacheKey` , et passez le nom de votre cache et le nom du groupe de ressources contenant le cache.

```azurepowershell
    PS C:\> Get-AzRedisCacheKey -Name myCache -ResourceGroupName myGroup

    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : ABhfB757JgjIgt785JgKH9865eifmekfnn649303JKL=
```

## <a name="to-regenerate-access-keys-for-your-azure-cache-for-redis"></a>Régénérer les clés d’accès pour votre instance du Cache Azure pour Redis
Pour régénérer les clés d’accès de votre cache, vous pouvez utiliser l’applet de commande [New-AzRedisCacheKey](/powershell/module/az.rediscache/New-azRedisCacheKey).

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `New-AzRedisCacheKey`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help New-AzRedisCacheKey -detailed

    NAME
        New-AzRedisCacheKey

    SYNOPSIS
        Regenerates the access key of an Azure Cache for Redis.

    SYNTAX
        New-AzRedisCacheKey -Name <String> -ResourceGroupName <String> -KeyType <String> [-Force] [<CommonParameters>]

    DESCRIPTION
        The New-AzRedisCacheKey cmdlet regenerate the access key of an Azure Cache for Redis.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -KeyType <String>
            Specifies whether to regenerate the primary or secondary access key. Possible values are Primary or Secondary.

        -Force
            When the Force parameter is provided, the specified access key is regenerated without any confirmation prompts.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

Pour régénérer la clé principale ou secondaire de votre cache, appelez l’applet de commande `New-AzRedisCacheKey` et passez le nom et le groupe de ressources, et spécifiez `Primary` ou `Secondary` pour le paramètre `KeyType`. Dans l’exemple suivant, la clé d’accès secondaire d’un cache est régénérée.

```azurepowershell
    PS C:\> New-AzRedisCacheKey -Name myCache -ResourceGroupName myGroup -KeyType Secondary

    Confirm
    Are you sure you want to regenerate Secondary key for Azure Cache for Redis 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : c53hj3kh4jhHjPJk8l0jji785JgKH9865eifmekfnn6=
```

## <a name="to-delete-an-azure-cache-for-redis"></a>Supprimer un Cache Azure pour Redis
Pour supprimer un Cache Azure pour Redis, utilisez la cmdlet [Remove-AzRedisCache](/powershell/module/az.rediscache/remove-azrediscache) .

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Remove-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Remove-AzRedisCache -detailed

    NAME
        Remove-AzRedisCache

    SYNOPSIS
        Remove Azure Cache for Redis if exists.

    SYNTAX
        Remove-AzRedisCache -Name <String> -ResourceGroupName <String> [-Force] [-PassThru] [<CommonParameters>

    DESCRIPTION
        The Remove-AzRedisCache cmdlet removes an Azure Cache for Redis if it exists.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to remove.

        -ResourceGroupName <String>
            Name of the resource group of the cache to remove.

        -Force
            When the Force parameter is provided, the cache is removed without any confirmation prompts.

        -PassThru
            By default Remove-AzRedisCache removes the cache and does not return any value. If the PassThru par
            is provided then Remove-AzRedisCache returns a boolean value indicating the success of the operatio

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```

Dans l’exemple suivant, le cache nommé `myCache` est supprimé.

```azurepowershell
    PS C:\> Remove-AzRedisCache -Name myCache -ResourceGroupName myGroup

    Confirm
    Are you sure you want to remove Azure Cache for Redis 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```


## <a name="to-import-an-azure-cache-for-redis"></a>Importer un Cache Azure pour Redis
Vous pouvez importer des données dans une instance du Cache Azure pour Redis à l’aide de la cmdlet `Import-AzRedisCache` .

> [!IMPORTANT]
> L’importation/exportation est uniquement disponible pour les caches de [niveau Premium](cache-overview.md#service-tiers). Pour plus d’informations sur l’importation/exportation, voir [Importer et exporter des données dans le Cache Azure pour Redis](cache-how-to-import-export-data.md).
> 
> 

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Import-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Import-AzRedisCache -detailed

    NAME
        Import-AzRedisCache

    SYNOPSIS
        Import data from blobs to Azure Cache for Redis.


    SYNTAX
        Import-AzRedisCache -Name <String> -ResourceGroupName <String> -Files <String[]> [-Format <String>] [-Force]
        [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Import-AzRedisCache cmdlet imports data from the specified blobs into Azure Cache for Redis.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Files <String[]>
            SAS urls of blobs whose content should be imported into the cache.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -Force
            When the Force parameter is provided, import will be performed without any confirmation prompts.

        -PassThru
            By default Import-AzRedisCache imports data in cache and does not return any value. If the PassThru
            parameter is provided then Import-AzRedisCache returns a boolean value indicating the success of the
            operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```


La commande suivante importe des données à partir de l’objet blob spécifié par l’URI SAP dans le Cache Azure pour Redis.

```azurepowershell
    PS C:\>Import-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Files @("https://mystorageaccount.blob.core.windows.net/mycontainername/blobname?sv=2015-04-05&sr=b&sig=caIwutG2uDa0NZ8mjdNJdgOY8%2F8mhwRuGNdICU%2B0pI4%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwd") -Force
```

## <a name="to-export-an-azure-cache-for-redis"></a>Exporter un Cache Azure pour Redis
Vous pouvez exporter des données depuis une instance du Cache Azure pour Redis à l’aide de la cmdlet `Export-AzRedisCache`.

> [!IMPORTANT]
> L’importation/exportation est uniquement disponible pour les caches de [niveau Premium](cache-overview.md#service-tiers). Pour plus d’informations sur l’importation/exportation, voir [Importer et exporter des données dans le Cache Azure pour Redis](cache-how-to-import-export-data.md).
> 
> 

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Export-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Export-AzRedisCache -detailed

    NAME
        Export-AzRedisCache

    SYNOPSIS
        Exports data from Azure Cache for Redis to a specified container.


    SYNTAX
        Export-AzRedisCache -Name <String> -ResourceGroupName <String> -Prefix <String> -Container <String> [-Format
        <String>] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Export-AzRedisCache cmdlet exports data from Azure Cache for Redis to a specified container.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Prefix <String>
            Prefix to use for blob names.

        -Container <String>
            SAS url of container where data should be exported.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -PassThru
            By default Export-AzRedisCache does not return any value. If the PassThru parameter is provided
            then Export-AzRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```


La commande suivante exporte les données à partir d’une instance du Cache Azure pour Redis vers le conteneur spécifié par l’URI SAP.

```azurepowershell
    PS C:\>Export-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Prefix "blobprefix"
    -Container "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2015-04-05&sr=c&sig=HezZtBZ3DURmEGDduauE7
    pvETY4kqlPI8JCNa8ATmaw%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwdl"
```

## <a name="to-reboot-an-azure-cache-for-redis"></a>Redémarrer un Cache Azure pour Redis
Vous pouvez redémarrer votre Cache Azure pour Redis à l’aide de la cmdlet `Reset-AzRedisCache`.

> [!IMPORTANT]
> Le redémarrage est uniquement disponible pour les caches de [niveau Premium](cache-overview.md#service-tiers). Pour plus d’informations sur le redémarrage de votre cache, voir [Administration du cache - Redémarrage](cache-administration.md#reboot).
> 
> 

Pour afficher la liste des paramètres disponibles et leurs descriptions pour `Reset-AzRedisCache`, exécutez la commande suivante.

```azurepowershell
    PS C:\> Get-Help Reset-AzRedisCache -detailed

    NAME
        Reset-AzRedisCache

    SYNOPSIS
        Reboot specified node(s) of an Azure Cache for Redis instance.


    SYNTAX
        Reset-AzRedisCache -Name <String> -ResourceGroupName <String> -RebootType <String> [-ShardId <Integer>]
        [-Force] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Reset-AzRedisCache cmdlet reboots the specified node(s) of an Azure Cache for Redis instance.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -RebootType <String>
            Which node to reboot. Possible values are "PrimaryNode", "SecondaryNode", "AllNodes".

        -ShardId <Integer>
            Which shard to reboot when rebooting a premium cache with clustering enabled.

        -Force
            When the Force parameter is provided, reset will be performed without any confirmation prompts.

        -PassThru
            By default Reset-AzRedisCache does not return any value. If the PassThru parameter is provided
            then Reset-AzRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
```


La commande suivante redémarre les deux nœuds du cache spécifié.

```azurepowershell
    PS C:\>Reset-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -RebootType "AllNodes"
    -Force
```


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur l’utilisation de Windows PowerShell avec Azure, reportez-vous aux ressources suivantes :

* [Documentation relative à la cmdlet du Cache Azure pour Redis sur MSDN](/powershell/module/az.rediscache)
* [Applets de commande Azure Resource Manager](/powershell/module/) : découvrez comment utiliser les applets de commande dans le module Azure Resource Manager.
* [Utilisation de groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/templates/deploy-portal.md): découvrez comment créer et gérer des groupes de ressources dans le portail Azure.
* [Blog Azure](https://azure.microsoft.com/blog/): découvrez les nouvelles fonctionnalités d'Azure.
* [Blog Windows PowerShell](https://devblogs.microsoft.com/powershell/): découvrez les nouvelles fonctionnalités de Windows PowerShell.
* [Blog « Hey, Scripting Guy! »](https://devblogs.microsoft.com/scripting/tag/hey-scripting-guy/): bénéficiez des conseils et astuces de la communauté Windows PowerShell.