---
title: Migrer vers Azure Cache pour Redis
description: Découvrez comment migrer votre cache existant vers Azure Cache pour Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: yegu
ms.openlocfilehash: 5de4e1b465cfc3ced59f8fe34a7f397324b4a225
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537624"
---
# <a name="migrate-to-azure-cache-for-redis"></a>Migrer vers Azure Cache pour Redis
Cet article décrit un certain nombre d’approches pour migrer un cache existant exécuté localement ou dans un autre service cloud vers Azure Cache pour Redis.

## <a name="migration-scenarios"></a>Scénarios de migration
Redis open source peut s’exécuter dans de nombreux environnements de calcul. Voici quelques exemples communs :

- **Local** : les caches Redis s’exécutant dans des centres de données privés.
- **Machines virtuelles basées sur le cloud** : les caches Redis qui s’exécutent sur des machines virtuelles Azure, AWS EC2, etc.
- **Services d’hébergement** : les services Redis gérés tels qu’AWS ElastiCache.
- **Différentes régions** : les caches Redis situés dans une autre région Azure.

Si vous disposez d’un tel cache, vous pourrez peut-être le déplacer vers Azure Cache pour Redis avec un temps d’interruption ou d’arrêt minime.

## <a name="migration-options"></a>Options de migration

Il existe différentes façons de passer d’un cache à un autre. Selon l’emplacement de votre cache et la manière dont votre application interagit avec, une méthode sera plus utile que les autres. Certaines stratégies de migration fréquemment utilisées sont détaillées ci-dessous.

   | Option       | Avantages | Inconvénients |
   | ------------ | ---------- | ------------- |
   | Créer un cache | Plus simple à implémenter. | Vous devez remplir de nouveau le cache, ce qui peut ne pas fonctionner avec de nombreuses applications. |
   | Exporter et importer des données via un fichier RDB | Compatible avec tout cache Redis en général. | Certaines données peuvent être perdues si elles sont écrites dans le cache existant après la génération du fichier RDB. | 
   | Double-écriture de données dans deux caches | Aucune perte de données, ni aucun temps d’arrêt. Opérations ininterrompues du cache existant. Test plus facile du nouveau cache. | Nécessite deux caches pendant une période prolongée. | 
   | Migrer des données par programme | Contrôle total sur la façon dont les données sont déplacées. | Requiert du code personnalisé. | 

### <a name="create-a-new-azure-cache-for-redis"></a>Créer une instance Azure Cache pour Redis

Cette approche n’est techniquement pas une migration. Si la perte de données n’est pas un problème, le moyen le plus simple de passer à Azure Cache pour Redis consiste à créer une instance de cache et à y connecter votre application. Par exemple, si vous utilisez Redis comme cache de recherche d’enregistrements de base de données, vous pouvez facilement reconstruire le cache à partir de zéro.

Les étapes générales pour implémenter cette option sont les suivantes :

1. Créez une instance Azure Cache pour Redis.

2. Mettez à jour votre application pour utiliser la nouvelle instance.

3. Supprimez l’ancienne instance de Redis.

### <a name="export-data-to-an-rdb-file-and-import-it-into-azure-cache-for-redis"></a>Exporter des données vers un fichier RDB et les importer dans Azure Cache pour Redis

Redis open source définit un mécanisme standard pour la capture d’un instantané du jeu de données en mémoire d’un cache et son enregistrement dans un fichier. Ce fichier, appelé RDB, peut être lu par un autre cache Redis. [Le niveau Premium d’Azure Cache pour Redis](cache-overview.md#service-tiers) prend en charge l’importation de données dans une instance de cache par le biais de fichiers RDB. Vous pouvez utiliser un fichier RDB pour transférer des données à partir d’un cache existant vers Azure Cache pour Redis.

> [!IMPORTANT]
> Le format de fichier RDB peut changer entre les versions de Redis et risque de ne pas assurer la compatibilité descendante. La version de Redis du cache à partir duquel vous exportez doit être inférieure ou égale à la version fournie par Azure Cache pour Redis.
>

Les étapes générales pour implémenter cette option sont les suivantes :

1. Créez une instance Azure Cache pour Redis dans le niveau Premium avec la même taille (ou une taille plus grande) que le cache existant.

2. Enregistrez un instantané du cache Redis existant. Vous pouvez [configurer des instructions Redis pour enregistrer régulièrement des instantanés](https://redis.io/topics/persistence) ou exécuter le processus manuellement à l’aide des commandes [SAVE](https://redis.io/commands/save) et [BGSAVE](https://redis.io/commands/bgsave). Par défaut, le fichier RDB est nommé « dump.rdb » et se trouve dans le chemin d’accès spécifié dans le fichier de configuration *redis.conf* .

    > [!NOTE]
    > Si vous migrez des données dans Azure Cache pour Redis, consultez [ces instructions sur la façon d’exporter un fichier RDB](cache-how-to-import-export-data.md) ou utilisez [l’applet de commande Export de PowerShell](/powershell/module/azurerm.rediscache/export-azurermrediscache?view=azurermps-6.13.0&viewFallbackFrom=azurermps-6.4.0) à la place.
    >

3. Copiez le fichier RDB dans un compte de stockage Azure dans la région où se trouve votre nouveau cache. Pour ce faire, vous pouvez utiliser AzCopy.

4. Importez le fichier RDB dans le nouveau cache à l’aide de ces [instructions d’importation](cache-how-to-import-export-data.md) ou de l’[applet de commande Import de PowerShell](/powershell/module/azurerm.rediscache/import-azurermrediscache?view=azurermps-6.13.0&viewFallbackFrom=azurermps-6.4.0).

5. Mettez à jour votre application pour utiliser la nouvelle instance de cache.

### <a name="write-to-two-redis-caches-simultaneously-during-migration-period"></a>Écrire simultanément dans deux caches Redis au cours de la période de migration

Au lieu de déplacer les données directement entre les caches, vous pouvez utiliser votre application pour écrire des données dans un cache existant et un nouveau cache que vous configurez. L’application continuera à lire les données du cache existant. Lorsque le nouveau cache disposera des données nécessaires, vous basculerez l’application vers ce cache et mettrez l’ancien hors service. Supposons, par exemple, que vous utilisez Redis comme magasin de sessions et que les sessions d’application sont valides pendant sept jours. Après avoir écrit dans les deux caches pendant une semaine, vous avez la certitude que le nouveau cache contient toutes les informations de session non expirées. Vous pouvez, en toute sécurité, l’utiliser à partir de ce point, sans vous soucier des pertes de données.

Les étapes générales pour implémenter cette option sont les suivantes :

1. Créez une instance Azure Cache pour Redis dans le niveau Premium avec la même taille (ou une taille plus grande) que le cache existant.

2. Modifiez le code de l’application pour écrire à la fois dans la nouvelle instance et celle d’origine.

3. Continuez la lecture des données à partir de l’instance d’origine jusqu’à ce que la nouvelle instance soit suffisamment remplie avec des données.

4. Mettez à jour le code de l’application pour qu’elle lise et écrive à partir de la nouvelle instance uniquement.

5. Supprimez l'instance d'origine.

### <a name="migrate-programmatically"></a>Migrer par programme

Vous pouvez créer un processus de migration personnalisé en lisant par programmation les données d’un cache existant et en les écrivant dans Azure Cache pour Redis. Cet [outil open source](https://github.com/deepakverma/redis-copy) peut être utilisé pour copier des données d’une instance Azure Cache pour Redis vers une autre. Cet outil est utile pour déplacer des données entre des instances de cache dans différentes régions Azure Cache. Une [version compilée](https://github.com/deepakverma/redis-copy/releases/download/alpha/Release.zip) est également disponible. Vous pourriez également trouver le code source utile pour l’écriture de votre propre outil de migration.

> [!NOTE]
> Cet outil n’est pas officiellement pris en charge par Microsoft. 
>

Les étapes générales pour implémenter cette option sont les suivantes :

1. Créez une machine virtuelle dans la région où se trouve le cache existant. Si votre jeu de données est volumineux, choisissez une machine virtuelle relativement puissante pour réduire le temps de copie.

2. Créez une instance Azure Cache pour Redis.

3. Videz les données du nouveau cache pour vous assurer qu’il est vide. Cette étape est requise, car l’outil de copie lui-même ne remplace aucune clé existante dans le cache cible.

    > [!IMPORTANT]
    > Veillez à ne pas vider le cache source.
    >

4. Utilisez une application telle que l’outil open source ci-dessus pour automatiser la copie des données du cache source vers le cache cible. N’oubliez pas que le processus de copie peut prendre un certain temps en fonction de la taille de votre jeu de données.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

* [Niveaux de service Azure Cache pour Redis](cache-overview.md#service-tiers)
* [Importer des données](cache-how-to-import-export-data.md#import)