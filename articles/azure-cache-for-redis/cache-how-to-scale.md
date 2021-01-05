---
title: Mise à l’échelle du cache Azure pour Redis
description: Découvrez comment mettre à l’échelle vos instances Azure Cache pour Redis à l’aide du portail Azure et d’outils tels qu’Azure PowerShell et Azure CLI.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 04/11/2017
ms.openlocfilehash: 332233873bfbcb2ae77f5a70b4aaa5a6102cecec
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537845"
---
# <a name="how-to-scale-azure-cache-for-redis"></a>Mise à l’échelle du cache Azure pour Redis
Le cache Azure pour Redis offre différents types de caches, permettant de choisir parmi plusieurs tailles et fonctionnalités de cache en toute flexibilité. Après la création d’un cache, vous pouvez mettre à l’échelle la taille et le niveau de tarification du cache si les exigences de votre application changent. Cet article montre comment mettre à l’échelle votre cache à l’aide du portail Azure et d’outils tels qu’Azure PowerShell et Azure CLI.

## <a name="when-to-scale"></a>Quand mettre à l’échelle ?
Les fonctionnalités de [surveillance](cache-how-to-monitor.md) du cache Azure pour Redis permettent de surveiller l’intégrité et les performances de votre cache et de déterminer la nécessité de le mettre à l’échelle. 

Vous pouvez surveiller les mesures suivantes pour déterminer si vous avez besoin d’effectuer une mise à l’échelle.

* Charge du serveur Redis
* Utilisation de la mémoire
* Bande passante réseau
* Utilisation du processeur

Si vous déterminez que votre cache ne répond plus aux besoins de votre application, vous pouvez passer à un niveau de tarification de cache plus important ou plus réduit, adapté à votre application. Pour plus d’informations sur la détermination des niveaux de tarification de cache à utiliser, consultez [Choix du niveau approprié](cache-overview.md#choosing-the-right-tier).

## <a name="scale-a-cache"></a>Mise à l’échelle d’un cache
Pour mettre à l’échelle votre cache, [accédez au cache](cache-configure.md#configure-azure-cache-for-redis-settings) dans le [portail Azure](https://portal.azure.com), puis cliquez sur **Mettre à l’échelle** dans le **menu Ressources** .

![Scale](./media/cache-how-to-scale/redis-cache-scale-menu.png)

Sélectionnez le niveau de tarification souhaité dans le panneau **Sélectionner un niveau tarifaire** , puis cliquez sur **Sélectionner** .

![Niveau tarifaire][redis-cache-pricing-tier-blade]


Vous pouvez choisir un niveau tarifaire différent avec les restrictions suivantes :

* Vous ne pouvez pas passer d’un niveau de tarification supérieur à un niveau de tarification inférieur.
  * Vous ne pouvez pas passer d’un cache **Premium** à un cache **Standard** ou **De base** .
  * Vous ne pouvez pas passer d’un cache **Standard** à un cache **De base** .
* Vous pouvez passer d’un cache **De base** à un cache **Standard** , mais vous ne pouvez pas modifier la taille en même temps. Si vous avez besoin d'une taille différente, vous pouvez effectuer ultérieurement une opération de mise à l'échelle vers la taille voulue.
* Vous ne pouvez pas passer directement d’un cache **De base** à un cache **Premium** . Vous devez commencer par passer du niveau **De base** au niveau **Standard** en une opération de mise à l’échelle, puis du niveau **Standard** au niveau **Premium** en une deuxième opération.
* Vous ne pouvez pas mettre à l’échelle depuis une taille supérieure vers la taille **C0 (250 Mo)** .
 
Lorsqu’un cache est mis à l’échelle vers un nouveau niveau tarifaire, le statut **Mise à l’échelle** s’affiche sur le panneau **Cache Azure pour Redis** .

![Mise à l'échelle][redis-cache-scaling]

Une fois la mise à l’échelle terminée, le statut passe de **Mise à l’échelle** à **En cours d’exécution** .

## <a name="how-to-automate-a-scaling-operation"></a>Automatisation d’une opération de mise à l’échelle
En plus de la mise à l’échelle de vos instances de cache dans le portail Azure, vous pouvez effectuer une mise à l’échelle en utilisant les applets de commande PowerShell, l’interface de ligne de commande Azure et les Bibliothèques de gestion Microsoft Azure (MAML). 

* [Mise à l’échelle à l’aide de PowerShell](#scale-using-powershell)
* [Mise à l’échelle à l’aide de l’interface de ligne de commande Azure](#scale-using-azure-cli)
* [Mise à l’échelle à l’aide de MAML](#scale-using-maml)

### <a name="scale-using-powershell"></a>Mise à l’échelle à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez mettre à l’échelle vos instances de cache Azure pour Redis avec PowerShell à l’aide de la cmdlet [Set-AzRedisCache](/powershell/module/az.rediscache/set-azrediscache) lorsque les propriétés `Size`, `Sku` ou `ShardCount` sont modifiées. L’exemple suivant montre comment mettre à l’échelle un cache nommé `myCache` vers un cache de 2,5 Go. 

```powershell
   Set-AzRedisCache -ResourceGroupName myGroup -Name myCache -Size 2.5GB
```

Pour plus d’informations sur la mise à l’échelle avec PowerShell, consultez [Pour mettre à l’échelle un cache Azure pour Redis à l’aide de PowerShell](cache-how-to-manage-redis-cache-powershell.md#scale).

### <a name="scale-using-azure-cli"></a>Mise à l’échelle à l’aide de l’interface de ligne de commande Azure
Pour mettre à l’échelle vos instances de cache Azure pour Redis à l’aide de l’interface de ligne de commande Azure, appelez la commande `azure rediscache set` et transmettez les modifications de configuration souhaitées qui incluent une nouvelle taille, la référence (SKU) ou la taille de cluster, en fonction de l’opération de mise à l’échelle souhaitée.

Pour plus d’informations sur la mise à l’échelle avec l’interface de ligne de commande Azure, consultez [Modification des paramètres d’un cache Azure pour Redis existant](cache-manage-cli.md#scale).

### <a name="scale-using-maml"></a>Mise à l’échelle à l’aide de MAML
Pour mettre à l’échelle vos instances du cache Azure pour Redis à l’aide des [Bibliothèques de gestion Microsoft Azure (MAML)](https://azure.microsoft.com/updates/management-libraries-for-net-release-announcement/), appelez la méthode `IRedisOperations.CreateOrUpdate` et transmettez la nouvelle taille de `RedisProperties.SKU.Capacity`.

```csharp
    static void Main(string[] args)
    {
        // For instructions on getting the access token, see
        // https://azure.microsoft.com/documentation/articles/cache-configure/#access-keys
        string token = GetAuthorizationHeader();

        TokenCloudCredentials creds = new TokenCloudCredentials(subscriptionId,token);

        RedisManagementClient client = new RedisManagementClient(creds);
        var redisProperties = new RedisProperties();

        // To scale, set a new size for the redisSKUCapacity parameter.
        redisProperties.Sku = new Sku(redisSKUName,redisSKUFamily,redisSKUCapacity);
        redisProperties.RedisVersion = redisVersion;
        var redisParams = new RedisCreateOrUpdateParameters(redisProperties, redisCacheRegion);
        client.Redis.CreateOrUpdate(resourceGroupName,cacheName, redisParams);
    }
```

Pour plus d’informations, consultez l’exemple [Gestion du cache Azure pour Redis en utilisant les bibliothèques de gestion Microsoft Azure](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML) .

## <a name="scaling-faq"></a>FAQ sur la mise à l’échelle
La liste suivante présente différentes réponses aux questions les plus fréquemment posées sur la mise à l’échelle du cache Azure pour Redis.

* [Puis-je mettre à l’échelle vers, depuis ou au sein d’un cache Premium ?](#can-i-scale-to-from-or-within-a-premium-cache)
* [Après la mise à l’échelle, dois-je modifier le nom ou les clés d’accès de mon cache ?](#after-scaling-do-i-have-to-change-my-cache-name-or-access-keys)
* [Comment fonctionne la mise à l’échelle ?](#how-does-scaling-work)
* [Vais-je perdre mes données de cache durant la mise à l’échelle ?](#will-i-lose-data-from-my-cache-during-scaling)
* [Les paramètres personnalisés de mes bases de données sont-ils affectés au cours de la mise à l’échelle ?](#is-my-custom-databases-setting-affected-during-scaling)
* [Mon cache reste-t-il disponible durant la mise à l’échelle ?](#will-my-cache-be-available-during-scaling)
* Maintenant que j’ai configuré la géoréplication, pourquoi ne puis-je pas mettre à l’échelle mon cache ou modifier les partitions d’un cluster ?
* [Quelles sont les opérations qui ne sont pas prises en charge ?](#operations-that-are-not-supported)
* [Quelle est la durée d’une mise à l’échelle ?](#how-long-does-scaling-take)
* [Comment savoir quand la mise à l’échelle est terminée ?](#how-can-i-tell-when-scaling-is-complete)

### <a name="can-i-scale-to-from-or-within-a-premium-cache"></a>Puis-je mettre à l’échelle vers, depuis ou au sein d’un cache Premium ?
* Vous ne pouvez pas passer d’un niveau tarifaire de cache **Premium** à un niveau tarifaire **De base** ou **Standard** .
* Vous pouvez passer d’un niveau tarifaire de cache **Premium** à un autre.
* Vous ne pouvez pas passer directement d’un cache **De base** à un cache **Premium** . Vous devez commencer par passer du niveau **De base** au niveau **Standard** en une opération de mise à l’échelle, puis du niveau **Standard** au niveau **Premium** en une deuxième opération.
* Si vous avez activé le clustering lorsque vous avez créé votre cache **Premium** , vous pouvez [changer la taille du cluster](cache-how-to-premium-clustering.md#cluster-size). Si votre cache a été créé sans que le clustering soit activé, vous pourrez configurer le clustering par la suite.
  
  Pour plus d’informations, consultez la page [Comment configurer le clustering Redis pour un cache Azure pour Redis Premium](cache-how-to-premium-clustering.md).

### <a name="after-scaling-do-i-have-to-change-my-cache-name-or-access-keys"></a>Après la mise à l’échelle, dois-je modifier le nom ou les clés d’accès de mon cache ?
Non, le nom et les clés d’accès de votre cache n’ont pas à être modifiés durant une opération de mise à l’échelle.

### <a name="how-does-scaling-work"></a>Comment fonctionne la mise à l’échelle ?
* Lorsqu’un cache **De base** est mis à l’échelle vers une taille différente, il est arrêté et un nouveau cache est approvisionné avec la nouvelle taille définie. Pendant ce temps, le cache n’est pas disponible et toutes les données qu’il contenait sont perdues.
* Lorsqu’un cache **De base** est mis à l’échelle vers un cache **Standard** , un cache de réplica est approvisionné. Les données sont ensuite copiées du cache principal vers le cache de réplica. Le cache reste disponible pendant le processus de mise à l'échelle.
* Quand un cache **Standard** est mis à l’échelle vers une taille différente ou vers un cache **Premium** , l’un des réplicas est arrêté et réapprovisionné avec la nouvelle taille tandis que les données sont transférées. L’autre réplica effectue ensuite un basculement avant d’être réapprovisionné, selon le même processus, durant un basculement de l’un des nœuds du cache.

### <a name="will-i-lose-data-from-my-cache-during-scaling"></a>Vais-je perdre mes données de cache durant la mise à l’échelle ?
* Lors de la mise à l’échelle d’un cache **De base** vers une nouvelle taille, toutes les données sont perdues et le cache n’est plus disponible pendant l’exécution de la mise à l’échelle.
* Lorsqu’un cache **De base** est mis à l’échelle vers un cache **Standard** , les données qui se trouvent dans le cache sont généralement conservées.
* Lors de la mise à l’échelle d’un cache **Standard** vers une taille ou un niveau supérieur, ou d’un cache **Premium** vers un niveau supérieur, toutes les données sont généralement conservées. Lors de la mise à l’échelle d’un cache **Standard** ou **Premium** vers une plus petite taille, il est possible que des données soient perdues si la quantité de données placées dans le cache est supérieure à la nouvelle taille du cache mis à l’échelle. En cas de pertes de données lors de la descente en puissante, les clés sont supprimées à l'aide de la stratégie d’éviction [allkeys-lru](https://redis.io/topics/lru-cache) . 

### <a name="is-my-custom-databases-setting-affected-during-scaling"></a>Les paramètres personnalisés de mes bases de données sont-ils affectés au cours de la mise à l’échelle ?
Si vous avez configuré une valeur personnalisée pour le paramètre `databases` lors de la création du cache, n’oubliez pas que certains niveaux de tarification appliquent des [limites des bases de données](cache-configure.md#databases) différentes. Voici quelques considérations relatives à la mise à l’échelle dans le cadre de ce scénario :

* Lors d’une mise à l’échelle vers un niveau de tarification associé à une limite `databases` plus faible que le niveau de tarification actuel :
  * Si vous utilisez le nombre par défaut de `databases`, c’est-à-dire 16 pour tous les niveaux de tarification, vous ne perdrez aucune donnée.
  * Si vous utilisez un nombre de `databases` personnalisé qui se situe dans les limites du niveau de tarification vers lequel vous effectuez la mise à l’échelle, ce paramètre `databases` sera conservé et vous ne perdrez aucune donnée.
  * Si vous utilisez un nombre de `databases` personnalisé qui dépasse les limites du nouveau niveau de tarification, le paramètre `databases` sera réduit aux limites du nouveau niveau et toutes les données contenues dans les bases de données supprimées seront perdues.
* Lors d’une mise à l’échelle vers un niveau de tarification associé à une limite `databases` identique ou supérieure à celle du niveau actuel, le paramètre `databases` est conservé et vous ne perdez aucune donnée.

Si les caches standard et premium ont un contrat SLA garantissant une disponibilité à 99,9 %, il n’existe pas de contrat SLA contre la perte de données.

### <a name="will-my-cache-be-available-during-scaling"></a>Mon cache reste-t-il disponible durant la mise à l’échelle ?
* Les caches **Standard** et **Premium** restent disponibles pendant l’opération de mise à l’échelle. Toutefois, des spots de connexion peuvent se produire lors de la mise à l’échelle des caches standard vers des caches premium, ainsi que des caches de base vers des caches standard. Ces spots de connexion sont supposés être petits, et les clients Redis doivent être en mesure de rétablir leur connexion instantanément.
* Les caches **de base** sont hors connexion pendant les opérations de mise à l’échelle vers une taille différente. Les caches de base restent disponibles lors de la mise à l’échelle du niveau **de base** vers le niveau **standard** , mais peuvent rencontrer un petit spot de connexion. Si un spot de connexion se produit, les clients Redis doivent être en mesure de rétablir leur connexion instantanément.


### <a name="scaling-limitations-with-geo-replication"></a>Limitations de la mise à l’échelle avec la géoréplication

Si vous ajoutez un lien de géoréplication entre deux caches, vous pourrez plus lancer une opération de mise à l’échelle ni modifier le nombre de partitions d’un cluster. Il vous faudra supprimer le lien du cache pour pouvoir émettre ces commandes. Pour plus d’informations, consultez la page [Configurer la géoréplication](cache-how-to-geo-replication.md).


### <a name="operations-that-are-not-supported"></a>Quelles sont les opérations qui ne sont pas prises en charge ?
* Vous ne pouvez pas passer d’un niveau de tarification supérieur à un niveau de tarification inférieur.
  * Vous ne pouvez pas passer d’un cache **Premium** à un cache **Standard** ou **De base** .
  * Vous ne pouvez pas passer d’un cache **Standard** à un cache **De base** .
* Vous pouvez passer d’un cache **De base** à un cache **Standard** , mais vous ne pouvez pas modifier la taille en même temps. Si vous avez besoin d'une taille différente, vous pouvez effectuer ultérieurement une opération de mise à l'échelle vers la taille voulue.
* Vous ne pouvez pas passer directement d’un cache **De base** à un cache **Premium** . Vous devez passer du niveau **de base** au niveau **standard** en une opération de mise à l’échelle, puis du niveau **standard** au niveau **premium** en une deuxième opération.
* Vous ne pouvez pas mettre à l’échelle depuis une taille supérieure vers la taille **C0 (250 Mo)** .

En cas d’échec d’une opération de mise à l’échelle, le service essaie d’annuler l’opération et le cache reviendra à sa taille d’origine.


### <a name="how-long-does-scaling-take"></a>Quelle est la durée d’une mise à l’échelle ?
Le temps de mise à l’échelle dépend de la quantité de données qui se trouvent dans le cache, les grandes quantités de données prenant plus de temps à traiter. La mise à l’échelle prend environ 20 minutes. Pour les caches en cluster, la mise à l’échelle prend environ 20 minutes par partition.

### <a name="how-can-i-tell-when-scaling-is-complete"></a>Comment savoir quand la mise à l’échelle est terminée ?
Le déroulement de l’opération de mise à l’échelle est affiché dans le portail Azure. Une fois la mise à l’échelle terminée, le statut passe à **En cours d’exécution** .

<!-- IMAGES -->

[redis-cache-pricing-tier-blade]: ./media/cache-how-to-scale/redis-cache-pricing-tier-blade.png

[redis-cache-scaling]: ./media/cache-how-to-scale/redis-cache-scaling.png