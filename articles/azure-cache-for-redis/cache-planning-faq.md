---
title: FAQ sur la planification d’Azure Cache pour Redis
description: Découvrez les réponses aux questions courantes qui vous aideront à planifier des solutions pour Azure Cache pour Redis
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 398b567e173691068e6dd932e075d6598dcfe601
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537403"
---
# <a name="azure-cache-for-redis-planning-faqs"></a>FAQ sur la planification d’Azure Cache pour Redis

Cet article fournit des réponses aux questions courantes sur la planification de solutions pour Azure Cache pour Redis.

## <a name="common-questions-and-answers"></a>Questions et réponses courantes
Cette section aborde les FAQ suivantes :

* [Performances du Cache Azure pour Redis](#azure-cache-for-redis-performance)
* [Dans quelle région dois-je placer mon cache ?](#in-what-region-should-i-locate-my-cache)
* [Où résident mes données mises en cache ?](#where-do-my-cached-data-reside)
* [Comment suis-je facturé pour le Cache Azure pour Redis ?](#how-am-i-billed-for-azure-cache-for-redis)
* [Puis-je utiliser Azure Cache pour Redis avec le cloud Azure Government, Azure China 21Vianet ou Microsoft Azure Allemagne ?](#can-i-use-azure-cache-for-redis-with-azure-government-cloud-azure-china-21vianet-cloud-or-microsoft-azure-germany)

### <a name="azure-cache-for-redis-performance"></a>Performances du Cache Azure pour Redis
Le tableau suivant présente les valeurs maximales de bande passante observées lors de tests exécutés sur différentes tailles de caches Standard et Premium à l’aide de `redis-benchmark.exe` à partir d’une machine virtuelle IaaS sur le point de terminaison du Cache Azure pour Redis. Pour le débit TLS, redis-benchmark est utilisé avec le stunnel pour se connecter au point de terminaison d’Azure Cache pour Redis.

>[!NOTE] 
>Ces valeurs ne sont pas garanties et il n’y a pas de contrat SLA pour ces chiffres, mais ils sont à peu près normaux. Vous devez tester la charge de votre application pour déterminer la taille de cache adaptée.
>Ces chiffres peuvent changer au fur et à mesure que nous publions des résultats plus récents.
>

De ce tableau, nous pouvons tirer les conclusions suivantes :

* Le débit des caches de taille identique est plus élevé dans le niveau Premium que dans le niveau Standard. Par exemple, avec un cache de 6 Go, le débit de P1 s’élève à 180 000 demandes par seconde par rapport à 100 000 demandes par seconde pour C3.
* Avec le clustering Redis, le débit augmente de façon linéaire à mesure que vous augmentez le nombre de partitions (nœuds) dans le cluster. Par exemple, si vous créez un cluster P4 de 10 partitions, alors le débit disponible s’élève à 400 000 *10 = 4 millions de demandes par seconde.
* Pour les tailles de clé supérieures, le débit du niveau Premium est plus élevé que celui du niveau Standard.

| Niveau tarifaire | Taille | Cœurs d’unité centrale | Bande passante disponible | Taille de la valeur 1 Ko | Taille de la valeur 1 Ko |
| --- | --- | --- | --- | --- | --- |
| **Tailles de cache Standard** | | |**Mégabits par seconde (Mbit/s) / mégaoctets par seconde (Mo/s)** |**Demandes par seconde (RPS) hors SSL** |**Demandes par seconde (RPS) SSL** |
| C0 | 250 Mo | Partagé | 100 / 12,5  |  15,000 |   7 500 |
| C1 |   1 Go | 1      | 500 / 62,5  |  38 000 |  20 720 |
| C2 | 2,5 Go | 2      | 500 / 62,5  |  41 000 |  37 000 |
| C3 |   6 Go | 4      | 1 000 / 125  | 100 000 |  90 000 |
| C4 |  13 Go | 2      | 500 / 62,5  |  60 000 |  55 000 |
| C5 |  26 Go | 4      | 1 000 / 125 | 102 000 |  93 000 |
| C6 |  53 Go | 8      | 2 000 / 250 | 126 000 | 120 000 |
| **Tailles de cache Premium** | |**Cœurs de processeur par partition** | **Mégabits par seconde (Mbit/s) / mégaoctets par seconde (Mo/s)** |**Demandes par seconde (RPS) hors SSL, par partition** |**Demandes par seconde (RPS) SSL, par partition** |
| P1 |   6 Go |  2 | 1 500 / 187,5 | 180 000 | 172 000 |
| P2 |  13 Go |  4 | 3 000 / 375   | 350 000 | 341 000 |
| P3 |  26 Go |  4 | 3 000 / 375   | 350 000 | 341 000 |
| P4 |  53 Go |  8 | 6 000 / 750   | 400 000 | 373 000 |
| P5 | 120 Go | 20 | 6 000 / 750   | 400 000 | 373 000 |

Pour obtenir des instructions sur la configuration du stunnel ou le téléchargement des outils Redis comme `redis-benchmark.exe`, consultez [Comment exécuter des commandes Redis ?](cache-development-faq.md#how-can-i-run-redis-commands)

### <a name="in-what-region-should-i-locate-my-cache"></a>Dans quelle région dois-je placer mon cache ?
Pour optimiser les performances et la latence, placez votre Cache Azure pour Redis dans la même région que celle de votre application de client de cache.

### <a name="where-do-my-cached-data-reside"></a>Où résident mes données mises en cache ?
Le Cache Azure pour Redis stocke vos données d’application dans la mémoire RAM de la ou des machines virtuelles, selon le niveau, qui hébergent votre cache. Vos données résident strictement dans la région Azure que vous avez sélectionnée par défaut. Vos données peuvent quitter une région dans deux cas :
* Quand vous activez la persistance sur le cache, le Cache Azure pour Redis sauvegarde vos données dans un compte Stockage Azure à vous. Si le compte de stockage que vous indiquez se trouve dans une autre région, une copie de vos données y parvient.
* Si vous configurez la géoréplication et que votre cache secondaire se trouve dans une autre région, ce qui est normalement le cas, vos données sont répliquées dans cette région.

Vous devez configurer explicitement le Cache Azure pour Redis pour qu’il utilise ces fonctionnalités. Vous disposez également d’un contrôle total sur la région dans laquelle se trouve le compte de stockage ou le cache secondaire.

### <a name="how-am-i-billed-for-azure-cache-for-redis"></a>Comment suis-je facturé pour le Cache Azure pour Redis ?
La tarification du Cache Azure pour Redis est disponible [ici](https://azure.microsoft.com/pricing/details/cache/). La page de tarification répertorie le tarif horaire et le tarif mensuel. Les caches sont facturés à la minute, de la création du cache jusqu’à sa suppression. Aucune option ne vous permet d’arrêter ou de suspendre la facturation d'un cache.

### <a name="can-i-use-azure-cache-for-redis-with-azure-government-cloud-azure-china-21vianet-cloud-or-microsoft-azure-germany"></a>Puis-je utiliser Azure Cache pour Redis avec le cloud Azure Government, Azure China 21Vianet ou Microsoft Azure Allemagne ?
Oui, le Cache Azure pour Redis est disponible dans le cloud Azure Government, le cloud Azure Chine 21Vianet et Microsoft Azure Allemagne. Les URL d’accès et de gestion du Cache Azure pour Redis diffèrent de celles du cloud public Azure.

| Cloud   | Suffixe DNS pour Redis            |
|---------|---------------------------------|
| Public  | *.redis.cache.windows.net       |
| Gouvernement des États-Unis  | *.redis.cache.usgovcloudapi.net |
| Allemagne | *.redis.cache.cloudapi.de       |
| Chine   | *.redis.cache.chinacloudapi.cn  |

Pour plus d’informations sur les éléments à prendre en compte lors de l’utilisation du Cache Azure pour Redis avec d’autres clouds, consultez les liens suivants.

- [Bases de données Azure Government - Cache Azure pour Redis](../azure-government/compare-azure-government-global-azure.md)
- [Cloud Azure Chine 21Vianet - Cache Azure pour Redis](https://www.azure.cn/home/features/redis-cache/)
- [Microsoft Azure Germany](https://azure.microsoft.com/overview/clouds/germany/)

Pour plus d’informations sur l’utilisation du Cache Azure pour Redis avec PowerShell dans le cloud Azure Government, le cloud Azure Chine 21Vianet et Microsoft Azure Allemagne, consultez [Comment se connecter aux autres clouds - Cache Azure pour Redis PowerShell](cache-how-to-manage-redis-cache-powershell.md#how-to-connect-to-other-clouds).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [FAQ sur Azure Cache pour Redis](cache-faq.md).