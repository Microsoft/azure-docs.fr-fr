---
title: Exemples du cache Azure pour Redis
description: 'Découvrez comment utiliser Azure Cache for Redis avec ces exemples de code : connexion à un cache, lecture et écriture de données dans un cache, fournisseurs Azure Cache for Redis ASP.NET.'
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-dotnet
ms.topic: sample
ms.date: 01/23/2017
ms.openlocfilehash: 553850173f463a05b13768eb3b9e17703bfa2886
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212283"
---
# <a name="azure-cache-for-redis-samples"></a>Exemples du cache Azure pour Redis
Cette rubrique fournit une liste d'exemples relatifs au cache Azure pour Redis, qui couvrent les scénarios de connexion au cache, de lecture et écriture des données vers et depuis le cache, ainsi que l'utilisation des fournisseurs ASP.NET de cache Azure pour Redis. Certains de ces exemples sont des projets téléchargeables. D'autres fournissent des instructions étape par étape et incluent des extraits de code, mais ne sont pas liés à un projet téléchargeable.

## <a name="hello-world-samples"></a>Exemples Hello world
Les exemples de cette section montrent les principes fondamentaux de la connexion à une instance du cache Azure pour Redis et ceux de la lecture et de l'écriture de données dans le cache à l'aide d'une variété de langages et de clients Redis.

L’exemple [Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) montre comment effectuer différentes opérations de cache à l’aide du client .NET [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis).

Cet exemple montre comment :

* Utiliser les différentes options de connexion
* Lire et écrire des objets dans le cache à l’aide des opérations synchrones et asynchrones
* Utiliser les commandes Redis MGET/MSET pour retourner les valeurs de clés spécifiées
* Effectuer des opérations Redis transactionnelles
* Utiliser des listes Redis et des ensembles triés
* Stocker des objets .NET à l’aide des sérialiseurs JsonConvert
* Utiliser les jeux Redis pour implémenter le balisage
* Travailler avec le Cluster Redis

Pour plus d’informations, consultez la documentation [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) sur GitHub. Pour plus de scénarios d’utilisation, consultez les tests unitaires [StackExchange.Redis.Tests](https://github.com/StackExchange/StackExchange.Redis/tree/master/tests).

L’exemple [How to use Azure Cache for Redis with Python](cache-python-get-started.md) montre comment utiliser le cache Azure pour Redis avec Python et le client [redis-py](https://github.com/andymccurdy/redis-py).

L’exemple [Work with .NET objects in the cache](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache) montre comment sérialiser des objets .NET afin de pouvoir les lire et les écrire depuis une instance du cache Azure pour Redis. 

## <a name="use-azure-cache-for-redis-as-a-scale-out-backplane-for-aspnet-signalr"></a>Utiliser le cache Azure pour Redis comme un fond de panier de scale-out pour ASP.NET SignalR
L’exemple [Use Azure Cache for Redis as a Scale out Backplane for ASP.NET SignalR](https://github.com/rustd/RedisSamples/tree/master/RedisAsSignalRBackplane) montre comment utiliser le cache Azure pour Redis comme fond de panier SignalR. Pour plus d’informations sur le fond de panier, consultez [SignalR Scaleout with Redis](https://www.asp.net/signalr/overview/performance/scaleout-with-redis).

## <a name="azure-cache-for-redis-customer-query-sample"></a>Cache Azure pour Redis - exemple de requête client
Cet exemple compare les performances entre l'accès aux données à partir d'un cache et l'accès aux données à partir d'un espace de persistance. Cet exemple comporte deux projets.

* [Démonstration de l’amélioration des performances par la mise en cache des données avec le cache Azure pour Redis](https://github.com/rustd/RedisSamples/tree/master/RedisCacheCustomerQuerySample)
* [Amorcer la base de données et le cache pour la démonstration](https://github.com/rustd/RedisSamples/tree/master/SeedCacheForCustomerQuerySample)

## <a name="aspnet-session-state-and-output-caching"></a>État de session ASP.NET et mise en cache de la sortie
L’exemple [Use Azure Cache for Redis to store ASP.NET SessionState and OutputCache](https://github.com/rustd/RedisSamples/tree/master/SessionState_OutputCaching) montre comment utiliser le cache Azure pour Redis afin de stocker les valeurs de session et de cache de sortie d’ASP.NET avec les fournisseurs SessionState et OutputCache de Redis.

## <a name="manage-azure-cache-for-redis-with-maml"></a>Gérer le cache Azure pour Redis avec MAML
L’exemple [Manage Azure Cache for Redis using Azure Management Libraries](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML) montre comment utiliser les bibliothèques de gestion Azure pour gérer (créer, mettre à jour et supprimer) votre cache. 

## <a name="custom-monitoring-sample"></a>Exemple de surveillance personnalisée
L’exemple [Access Azure Cache for Redis Monitoring data](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) montre comment accéder aux données de surveillance de votre cache Azure pour Redis en dehors du portail Azure.

## <a name="a-twitter-style-clone-written-using-php-and-redis"></a>Un clone de style Twitter écrit avec PHP et Redis
L’exemple [Retwis](https://github.com/SyntaxC4-MSFT/retwis) est Redis Hello World. Il s’agit d’un clone minimal de réseau social de style Twitter écrit avec Redis et PHP, à l’aide du client [Predis](https://github.com/nrk/predis) . Le code source est conçu pour être très simple et en même temps pour présenter les différentes structures de données Redis.

## <a name="bandwidth-monitor"></a>Surveiller la bande passante
L’exemple [Bandwidth monitor](https://github.com/JonCole/SampleCode/tree/master/BandWidthMonitor) vous permet de surveiller la bande passante utilisée sur le client. Pour mesurer la bande passante, exécutez l’exemple sur l’ordinateur client du cache, effectuez des appels au cache et observez la bande passante signalée par l’exemple de surveillance de la bande passante.
