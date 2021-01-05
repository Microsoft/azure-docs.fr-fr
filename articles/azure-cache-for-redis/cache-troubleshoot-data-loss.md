---
title: Résoudre les problèmes de perte de données dans Azure Cache pour Redis
description: Découvrez comment résoudre les problèmes de perte de données avec Azure Cache for Redis, tels que la perte partielle de clés, l’expiration des clés ou la perte complète des clés.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/17/2019
ms.openlocfilehash: 6db036752bab7b84b72a37b148eaec7aa5765ef3
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538593"
---
# <a name="troubleshoot-data-loss-in-azure-cache-for-redis"></a>Résoudre les problèmes de perte de données dans Azure Cache pour Redis

Cet article explique comment diagnostiquer les pertes de données réelles ou perçues qui peuvent se produire dans Azure Cache pour Redis.

> [!NOTE]
> Dans ce guide, plusieurs procédures de résolution de problèmes comprennent des instructions pour exécuter des commandes Redis et surveiller diverses mesures de performances. Pour plus d’informations et d’instructions, consultez les articles dans la section [Informations supplémentaires](#additional-information) .
>

## <a name="partial-loss-of-keys"></a>Perte partielle des clés

Azure Cache pour Redis ne supprime pas de façon aléatoire les clés lorsqu’elles sont stockées en mémoire. Toutefois, il les supprime en réponse à des stratégies d’expiration ou d’éviction et à la suite de commandes de suppression de clé explicites. Les clés qui ont été écrites sur le nœud principal d’une instance Azure Cache pour Redis Premium ou Standard peuvent ne pas être immédiatement disponibles sur un réplica. Les données sont répliquées du nœud principal vers le réplica de manière asynchrone et non bloquante.

Si vous constatez que des clés ont disparu de votre cache, passez en revue les causes possibles suivantes :

| Cause | Description |
|---|---|
| [Expiration des clés](#key-expiration) | Les clés sont supprimées en raison des délais d’attente définis pour elles. |
| [Éviction des clés](#key-eviction) | Les clés sont supprimées pour cause de sollicitation de la mémoire. |
| [Suppression des clés](#key-deletion) | Les clés sont supprimées par des commandes de suppression explicites. |
| [Réplication asynchrone](#async-replication) | Les clés ne sont pas disponibles sur un réplica en raison de délais affectant la réplication des données. |

### <a name="key-expiration"></a>Expiration des clés

Le cache Azure Cache pour Redis supprime automatiquement une clé si un délai d’attente est affecté à la clé et que la période est passée. Pour en savoir plus sur l’expiration des clés Redis, consultez la documentation relative à la commande [EXPIRE](https://redis.io/commands/expire). Vous pouvez également définir des valeurs de délai d’expiration à l’aide des commandes [SET](https://redis.io/commands/set), [SETEX](https://redis.io/commands/setex) et [GETSET](https://redis.io/commands/getset), et d’autres commandes **\*STORE** .

Vous pouvez utiliser la commande [INFO](https://redis.io/commands/info) pour obtenir des statistiques sur le nombre de clés arrivées à expiration. La section `Stats` montre le nombre total de clés arrivées à expiration. La section `Keyspace` fournit des informations supplémentaires sur le nombre de clés avec des délais d’expiration, et la valeur moyenne du délai d’expiration.

```
# Stats

expired_keys:46583

# Keyspace

db0:keys=3450,expires=2,avg_ttl=91861015336
```

Vous pouvez également examiner les indicateurs de performance de diagnostic de votre cache pour voir s’il existe une corrélation entre le moment où la clé a disparu et un pic d’expiration des clés. Consultez l’annexe de la rubrique relative au [débogage des échecs de l’espace de clés Redis](https://gist.github.com/JonCole/4a249477142be839b904f7426ccccf82#appendix) pour en savoir plus sur les notifications relatives à l’espace de clés ou à **MONITOR** pour corriger ces erreurs.

### <a name="key-eviction"></a>Éviction des clés

Azure Cache pour Redis a besoin d’espace mémoire pour stocker les données. Il vide donc des clés pour libérer de la mémoire, si nécessaire. Lorsque les valeurs **used_memory** ou **used_memory_rss** dans la commande [INFO](https://redis.io/commands/info) s’approchent du paramètre **maxmemory** configuré, Azure Cache pour Redis commence à supprimer des clés de la mémoire en fonction de la [stratégie de cache](https://redis.io/topics/lru-cache).

Utilisez la commande [INFO](https://redis.io/commands/info) pour surveiller le nombre de clés supprimées :

```
# Stats

evicted_keys:13224
```

Vous pouvez également examiner les indicateurs de performance de diagnostic de votre cache pour voir s’il existe une corrélation entre le moment où la clé a disparu et un pic d’éviction des clés. Consultez l’annexe de la rubrique relative au [débogage des échecs de l’espace de clés Redis](https://gist.github.com/JonCole/4a249477142be839b904f7426ccccf82#appendix) pour en savoir plus sur les notifications relatives à l’espace de clés ou à **MONITOR** pour corriger ces erreurs.

### <a name="key-deletion"></a>Suppression des clés

Les clients Redis peuvent émettre la commande [DEL](https://redis.io/commands/del) ou [HDEL](https://redis.io/commands/hdel) pour supprimer explicitement des clés d’Azure Cache pour Redis. Utilisez la commande [INFO](https://redis.io/commands/info) pour effectuer le suivi du nombre d’opérations de suppression. Si des commandes **DEL** ou **HDEL** ont été appelées, elles sont répertoriées dans la section `Commandstats`.

```
# Commandstats

cmdstat_del:calls=2,usec=90,usec_per_call=45.00

cmdstat_hdel:calls=1,usec=47,usec_per_call=47.00
```

### <a name="async-replication"></a>Réplication asynchrone

Toute instance Azure Cache pour Redis de niveau Standard ou Premium est configurée avec un nœud principal et au moins un réplica. Les données sont copiées de façon asynchrone du nœud principal vers un réplica à l’aide d’un processus en arrière-plan. Le site web [redis.io](https://redis.io/topics/replication) décrit le fonctionnement général de la réplication des données dans Redis. Dans les scénarios où les clients écrivent fréquemment dans Redis, vous pouvez subir une perte partielle de données, car l’instantanéité de cette réplication n’est pas garantie. Par exemple, si le nœud principal subit une défaillance *après* qu’un client a écrit une clé sur celui-ci, mais *avant* que le processus en arrière-plan ait eu la l’occasion d’envoyer cette clé au réplica, cette clé est perdue quand le réplica prend le relais en tant que nouveau nœud principal.

## <a name="major-or-complete-loss-of-keys"></a>Perte majeure ou totale des clés

Si vous constatez que tout ou partie des clés ont disparu de votre cache, passez en revue les causes possibles suivantes :

| Cause | Description |
|---|---|
| [Vidage des clés](#key-flushing) | Les clés ont été vidées manuellement. |
| [Sélection incorrecte de la base de données](#incorrect-database-selection) | Le logiciel Azure Cache pour Redis est configuré pour utiliser une base de données autre que celle par défaut. |
| [Échec de l’instance de Redis](#redis-instance-failure) | Le serveur Redis n’est pas disponible. |

### <a name="key-flushing"></a>Vidage des clés

Les clients peuvent appeler la commande [FLUSHDB](https://redis.io/commands/flushdb) pour supprimer toutes les clés d’une base de données *unique* ou [FLUSHALL](https://redis.io/commands/flushall) pour supprimer toutes les clés de *toutes* les bases de données dans un cache Redis. Utilisez la commande [INFO](https://redis.io/commands/info) pour déterminer si les clés ont été vidées. La section `Commandstats` indique si la commande **FLUSH** a été appelée :

```
# Commandstats

cmdstat_flushall:calls=2,usec=112,usec_per_call=56.00

cmdstat_flushdb:calls=1,usec=110,usec_per_call=52.00
```

### <a name="incorrect-database-selection"></a>Sélection incorrecte de la base de données

Azure Cache pour Redis utilise la base de données **db0** par défaut. Si vous basculez vers une autre base de données (par exemple, **db1** ) et essayez de lire des clés à partir de celle-ci, Azure Cache pour Redis ne les trouve pas à cet emplacement. Chaque base de données est une unité logiquement distincte et contient un jeu de données différent. Utilisez la commande [SELECT](https://redis.io/commands/select) pour utiliser d’autres bases de données disponibles et rechercher des clés dans chacune d’elles.

### <a name="redis-instance-failure"></a>Échec de l’instance de Redis

Redis est un magasin de données en mémoire. Les données sont conservées sur les machines physiques ou virtuelles qui hébergent le cache Redis. Une instance d’Azure Cache pour Redis de niveau De base ne s’exécute que sur une seule machine virtuelle. Si cette machine virtuelle tombe en panne, toutes les données que vous avez stockées dans le cache sont perdues. 

Les caches de niveau Standard ou Premium utilisent deux machines virtuelles dans une configuration répliquée, offrant ainsi plus de résilience contre la perte de données. Quand le nœud principal d’un tel cache échoue, le nœud de réplica prend le relais pour traiter automatiquement les données. Ces machines virtuelles se trouvent sur des domaines d’erreur et de mise à jour distincts, afin de réduire le risque qu’elles ne soient pas disponibles en même temps. Toutefois, en cas de défaillance majeure du centre de données, il se peut que les machines virtuelles tombent en panne simultanément. Dans ces rares cas, vos données sont perdues.

Songez à utiliser la [persistance des données Redis](https://redis.io/topics/persistence) et la [géoréplication](./cache-how-to-geo-replication.md) pour renforcer la protection de vos données contre ces défaillances d’infrastructure.

## <a name="additional-information"></a>Informations supplémentaires

- [Résoudre les problèmes côté serveur liés à Azure Cache pour Redis](cache-troubleshoot-server.md)
- [Choix du niveau approprié](cache-overview.md#choosing-the-right-tier)
- [Surveillance du cache Azure pour Redis](cache-how-to-monitor.md)
- [Comment exécuter des commandes Redis ?](cache-development-faq.md#how-can-i-run-redis-commands)