---
title: Résoudre les problèmes côté serveur liés à Azure Cache pour Redis
description: Découvrir comment résoudre les problèmes courants côté serveur liés à Azure Cache pour Redis, comme la sollicitation de la mémoire, l’UC élevée, les commandes qui prennent du temps ou les limites de bande passante.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/18/2019
ms.openlocfilehash: 12d78846f5892e71388de6e6e76b868f9b14d4de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88008914"
---
# <a name="troubleshoot-azure-cache-for-redis-server-side-issues"></a>Résoudre les problèmes côté serveur liés à Azure Cache pour Redis

Cette section traite de la résolution des problèmes qui se produisent à la suite d’une condition sur Azure Cache pour Redis ou sur la ou les machines virtuelles qui l’hébergent.

- [Sollicitation de la mémoire sur le serveur Redis](#memory-pressure-on-redis-server)
- [Utilisation élevée du processeur ou charge importante du serveur](#high-cpu-usage-or-server-load)
- [Commandes de longue durée](#long-running-commands)
- [Limitation de la bande passante côté serveur](#server-side-bandwidth-limitation)

> [!NOTE]
> Dans ce guide, plusieurs procédures de résolution de problèmes comprennent des instructions pour exécuter des commandes Redis et surveiller diverses mesures de performances. Pour plus d’informations et d’instructions, consultez les articles dans la section [Informations supplémentaires](#additional-information) .
>

## <a name="memory-pressure-on-redis-server"></a>Sollicitation de la mémoire sur le serveur Redis

La saturation de la mémoire sur le serveur génère toutes sortes de problèmes de performance susceptibles de retarder le traitement des demandes. Lorsque la mémoire est trop sollicitée, le système peut paginer les données vers le disque. Cette _pagination_ ralentit considérablement le système. Plusieurs causes peuvent être à l’origine de cette saturation de la mémoire :

- Le cache contient beaucoup de données et il est sur le point d’atteindre sa capacité maximale.
- Redis constate une importante fragmentation de mémoire. Cette fragmentation est plus souvent provoquée par le stockage d’objets volumineux, car Redis est optimisé pour les petits objets.

Redis expose deux statistiques via la commande [INFO](https://redis.io/commands/info) qui peuvent vous aider à identifier ce problème : « used_memory » et « used_memory_rss ». Vous pouvez [afficher ces métriques](cache-how-to-monitor.md#view-metrics-with-azure-monitor) à l’aide du portail.

Vous pouvez apporter plusieurs modifications pour maintenir une utilisation correcte de la mémoire :

- [Configurez une stratégie de mémoire](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) et définissez des délais d’expiration sur vos clés. Cette stratégie peut ne pas suffire si vous constatez une fragmentation.
- [Configurez une valeur maxmemory-reserved](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) suffisamment grande pour compenser la fragmentation de la mémoire.
- Divisez vos objets volumineux mis en cache en objets plus petits.
- [Créez des alertes](cache-how-to-monitor.md#alerts) pour des métriques comme la mémoire utilisée, afin d’être averti le plus tôt possible des impacts potentiels.
- [Effectuez une mise à l’échelle](cache-how-to-scale.md) pour une plus grande taille de cache et une plus grande capacité de mémoire.

## <a name="high-cpu-usage-or-server-load"></a>Utilisation élevée du processeur ou charge importante du serveur

Une charge serveur ou une utilisation du processeur élevées signifie que le serveur ne peut pas traiter les requêtes en temps voulu. Le serveur peut être lent et ne pas être capable de répondre aux requêtes en temps voulu.

[Supervisez les métriques](cache-how-to-monitor.md#view-metrics-with-azure-monitor) comme l’utilisation du processeur ou la charge serveur. Regardez les pics d’utilisation du processeur qui correspondent à des délais d’expiration.

Plusieurs modifications sont possibles pour réduire la charge serveur :

- Recherchez la cause des pics d’utilisation du processeur, comme l’exécution de [commandes de longue durée](#long-running-commands) ou les défauts de page dus à une trop grande sollicitation de la mémoire.
- [Créez des alertes](cache-how-to-monitor.md#alerts) pour des métriques comme l’utilisation du processus ou la charge serveur, afin d’être averti le plus tôt possible des impacts potentiels.
- [Effectuez une mise à l’échelle](cache-how-to-scale.md) pour une plus grande taille de cache et une plus grande capacité de processeur.

## <a name="long-running-commands"></a>Commandes de longue durée

Certaines commandes Redis sont plus coûteuses à exécuter que d’autres. La [documentation concernant les commandes Redis](https://redis.io/commands) montre la complexité temporelle de chaque commande. Étant donné que le traitement des commandes Redis ne fait appel qu’à un seul thread, une commande qui prend du temps à s’exécuter bloque toutes les autres qui suivent. Vous devez passer en revue les commandes que vous émettez à votre serveur Redis pour comprendre leur impact sur le niveau de performance. Par exemple, les utilisateurs se servent souvent de la commande [KEYS](https://redis.io/commands/keys) sans savoir qu’il s’agit d’une opération O(N). Vous pouvez éviter la commande KEYS en utilisant [SCAN](https://redis.io/commands/scan) pour réduire les pics d’utilisation du processeur.

Avec la commande [SLOWLOG](https://redis.io/commands/slowlog), vous pouvez mesurer les commandes consommant beaucoup de ressources qui sont actuellement exécutées sur le serveur.

## <a name="server-side-bandwidth-limitation"></a>Limitation de la bande passante côté serveur

La capacité de la bande passante réseau varie selon la taille du cache. Si le serveur dépasse la bande passante disponible, les données ne sont pas envoyées au client aussi rapidement. Les requêtes du client risquent d’expirer, car le serveur ne peut pas envoyer (push) les données au client suffisamment rapidement.

Vous pouvez vous servir des métriques « Lecture du cache » et « Écriture dans le cache » pour voir la quantité de bande passante qui est utilisée côté serveur. Vous pouvez [afficher ces métriques](cache-how-to-monitor.md#view-metrics-with-azure-monitor) dans le portail.

Pour réduire une utilisation de la bande passante réseau proche de la capacité maximale :

- Modifiez le comportement d’appel du client afin de réduire la demande réseau.
- [Créez des alertes](cache-how-to-monitor.md#alerts) pour des métriques comme la lecture du cache ou l’écriture dans le cache, afin d’être averti le plus tôt possible des impacts potentiels.
- [Effectuez une mise à l’échelle](cache-how-to-scale.md) pour une plus grande taille de cache et une plus grande capacité de bande passante réseau.

## <a name="additional-information"></a>Informations supplémentaires

- [Résoudre les problèmes côté client liés à Azure Cache pour Redis](cache-troubleshoot-client.md)
- [Choix du niveau approprié](cache-overview.md#choosing-the-right-tier)
- [Comment puis-je évaluer et tester les performances de mon cache ?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
- [Surveillance du cache Azure pour Redis](cache-how-to-monitor.md)
- [Comment exécuter des commandes Redis ?](cache-development-faq.md#how-can-i-run-redis-commands)
