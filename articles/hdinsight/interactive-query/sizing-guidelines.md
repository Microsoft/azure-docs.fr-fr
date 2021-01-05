---
title: Guide de dimensionnement du cluster Interactive Query dans Azure HDInsight
description: Guide de dimensionnement Interactive Query dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: a81eff1dcf48996c319933aa4dd46170043b943b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83663647"
---
# <a name="interactive-query-cluster-sizing-guide-in-azure-hdinsight"></a>Guide de dimensionnement du cluster Interactive Query dans Azure HDInsight

Ce document décrit le dimensionnement du cluster Interactive Query HDInsight (LLAP) pour une charge de travail classique, dans le but d’obtenir des performances raisonnables. Les suggestions fournies dans ce document sont génériques et des charges de travail spécifiques peuvent nécessiter un paramétrage spécifique.

## <a name="default-vm-types-for-interactive-query"></a>Types de machine virtuelle par défaut pour Interactive Query

| Type de nœud | Instance | Taille |
|---|---|---|
| Head | D13 v2 | 8 processeurs virtuels, 56 Go de RAM, disque SSD de 400 Go |
| Worker | D14 v2 | 16 processeurs virtuels, 112 Go de RAM, disque SSD de 800 Go |
| ZooKeeper | A4 v2 | 4 processeurs virtuels, 8 Go de RAM, disque SSD de 40 Go |

## <a name="recommended-configurations"></a>Configurations recommandées

Les valeurs de configuration recommandées sont basées sur le nœud Worker de type D14 v2.

| Clé | Valeur | Description |
|---|---|---|
| yarn.nodemanager.resource.memory-mb | 102 400 (Mo) | Mémoire totale donnée, en Mo, pour tous les conteneurs YARN sur un nœud. |
| yarn.scheduler.maximum-allocation-mb | 102 400 (Mo) | Allocation maximale pour chaque requête de conteneur au gestionnaire des ressources, en Mo. Les requêtes de mémoire de valeur supérieure à celle-ci ne sont pas prises en compte. |
| yarn.scheduler.maximum-allocation-vcores | 12 |Nombre maximal de cœurs de processeur pour chaque requête de conteneur adressée au gestionnaire des ressources. Les requêtes de mémoire de valeur supérieure à celle-ci ne sont pas prises en compte. |
| yarn.scheduler.capacity.root.llap.capacity | 90% | Allocation de la capacité YARN pour la file d’attente LLAP.  |
| hive.server2.tez.sessions.per.default.queue | number_of_worker_nodes |Nombre de sessions pour chaque file d’attente nommée dans hive.server2.tez.default.queues. Ce nombre correspond au nombre de coordinateurs de requêtes (AppMaster Tez). |
| tez.am.resource.memory.mb | 4 096 (Mo) | Quantité de mémoire (en Mo) que doit utiliser le AppMaster Tez. |
| hive.tez.container.size | 4 096 (Mo) | Taille du conteneur Tez spécifiée en Mo. |
| hive.llap.daemon.num.executors | 12 | Nombre d’exécuteurs par démon LLAP. |
| hive.llap.io.threadpool.size | 12 | Taille du pool de threads pour les exécuteurs. |
| hive.llap.daemon.yarn.container.mb | 86 016 (Mo) | Mémoire totale, en Mo, utilisée par les démons LLAP individuels (quantité de mémoire par démon).|
| hive.llap.io.memory.size | 409 600 (Mo) | Taille, en Mo, du cache affecté à chaque démon LLAP, si le cache SSD est activé. |
| hive.auto.convert.join.noconditionaltask.size | 2 048 (Mo) | Taille de la mémoire pour la jointure de mappage, en Mo. |

## <a name="llap-daemon-size-estimations"></a>Estimation des tailles de démons LLAP  

### <a name="yarnnodemanagerresourcememory-mb"></a>yarn.nodemanager.resource.memory-mb

Cette valeur indique la quantité maximale de mémoire utilisée par les conteneurs YARN sur chaque nœud, en Mo. Elle spécifie la quantité de mémoire que YARN peut utiliser sur ce nœud. Elle doit donc être inférieure à la quantité de mémoire totale disponible sur ce nœud.

Définissez cette valeur comme suit = [Mémoire physique totale sur le nœud] – [Mémoire pour le système d’exploitation + autres services].

Il est recommandé de définir cette valeur sur environ 90 % de la mémoire RAM disponible. La valeur recommandée est de **102 400 Mo** pour D14 v2.

### <a name="yarnschedulermaximum-allocation-mb"></a>yarn.scheduler.maximum-allocation-mb

Cette valeur indique l’allocation maximale pour chaque requête de conteneur au gestionnaire des ressources, en Mo. Les requêtes de mémoire de taille supérieure à celle-ci ne sont pas prises en compte. Le gestionnaire des ressources peut uniquement fournir de la mémoire aux conteneurs par incréments de `yarn.scheduler.minimum-allocation-mb` et ne peut pas dépasser la taille spécifiée par `yarn.scheduler.maximum-allocation-mb`. Cette valeur ne doit pas être supérieure à la taille de la mémoire totale donnée du nœud, spécifiée par `yarn.nodemanager.resource.memory-mb`.

Pour les nœuds Worker D14 v2, la valeur recommandée est de **102 400 Mo**.

### <a name="yarnschedulermaximum-allocation-vcores"></a>yarn.scheduler.maximum-allocation-vcores

Cette configuration indique le nombre maximal de cœurs de processeur virtuel pour chaque requête de conteneur adressée au gestionnaire des ressources. Toute requête de valeur supérieure à cette configuration sera ignorée. Cette configuration est une propriété globale du planificateur YARN. Pour le conteneur du démon LLAP, cette valeur peut être définie sur 75 % du nombre total de cœurs virtuels disponibles (vCores). Les 25 % restants doivent être réservés à NodeManager, DataNode et d’autres services s’exécutant sur les nœuds Worker.  

Pour les nœuds Worker D14 v2, il existe 16 vCores, et vous pouvez spécifier 75 % de ces 16 vCores. La valeur recommandée pour le conteneur du démon LLAP est donc **12**.

### <a name="hiveserver2tezsessionsperdefaultqueue"></a>hive.server2.tez.sessions.per.default.queue

Cette valeur de configuration détermine le nombre de sessions Tez qui doivent être lancées en parallèle pour chacune des files d’attente spécifiées par `hive.server2.tez.default.queues`. La valeur correspond au nombre d’AM Tez (coordinateurs de requêtes). Nous vous recommandons d’utiliser le même nombre de nœuds Worker pour avoir un AM Tez par nœud. Le nombre d’AM Tez peut être supérieur au nombre de nœuds du démon LLAP. Leur principale responsabilité consiste à coordonner l’exécution de la requête et à affecter des fragments de plan de requête aux démons LLAP correspondants, à des fins d’exécution. Il est recommandé de conserver le plus grand nombre de nœuds de démon LLAP pour obtenir un débit plus élevé.  

Le cluster HDInsight par défaut a quatre démons LLAP en cours d’exécution sur quatre nœuds Worker ; la valeur recommandée est donc **4**.  

### <a name="tezamresourcememorymb-hivetezcontainersize"></a>tez.am.resource.memory.mb, hive.tez.container.size

`tez.am.resource.memory.mb` définit la taille du AppMaster Tez.  
La valeur recommandée est de **4 096 Mo**.

`hive.tez.container.size` définit la quantité de mémoire donnée pour le conteneur Tez. Cette valeur doit être définie entre la taille minimale du conteneur YARN (`yarn.scheduler.minimum-allocation-mb`) et sa taille maximale (`yarn.scheduler.maximum-allocation-mb`).  
Il est recommandé de définir cette taille sur **4 096 Mo**.  

Une règle générale consiste à conserver une valeur inférieure à la quantité de mémoire par processeur, sur la base d’un processeur par conteneur. Mémoire `Rreserve` pour le nombre d’AM Tez sur un nœud avant l’octroi de mémoire à un démon LLAP. Par exemple, si vous utilisez deux AM Tez (de 4 Go chacun) par nœud, octroyez 82 sur 90 Go au démon LLAP, en réservant les 8 Go restants pour les deux AM Tez.

### <a name="yarnschedulercapacityrootllapcapacity"></a>yarn.scheduler.capacity.root.llap.capacity

Cette valeur indique un pourcentage de capacité octroyé à la file d’attente LLAP. Le cluster HDInsights Interactive Query obtient 90 % de la capacité totale de la file d’attente LLAP, les 10 % restants étant définis sur la file d’attente par défaut pour les autres allocations de conteneur.  
Pour les nœuds Worker D14v2, la valeur recommandée de la file d’attente est de **90**.

### <a name="hivellapdaemonyarncontainermb"></a>hive.llap.daemon.yarn.container.mb

La taille totale de la mémoire pour le démon LLAP dépend des composants suivants :

* Configuration de la taille du conteneur YARN (`yarn.scheduler.maximum-allocation-mb`, `yarn.scheduler.maximum-allocation-mb`, `yarn.nodemanager.resource.memory-mb`)

* Mémoire de tas utilisée par les exécuteurs (Xmx)

    La quantité de mémoire vive disponible une fois la marge retirée.  
    Pour D14 v2, HDI 4.0 : cette valeur est égale à (86 Go -6 Go) soit 80 Go  
    Pour D14 v2, HDI 3.6 : cette valeur est égale à (84 Go - 6 Go) soit 78 Go

* Cache en mémoire hors tas par démon (hive.llap.io.memory.size)

* Marge

    Il s’agit d’une partie de la mémoire hors tas utilisée pour la surcharge des machines virtuelles Java (espace de mémoire, pile de threads, structures de données GC, etc.). Cette partie représente généralement 6 % environ de la taille du tas (Xmx). Par prudence, on peut la calculer comme correspondant à 6 % de la taille de la mémoire totale du démon LLAP. Comme cette opération est possible lorsque le cache SSD est activé, le démon LLAP est autorisé à utiliser tout l’espace en mémoire disponible uniquement pour le tas.  
    Pour D14 v2, la valeur recommandée est : plafond(86 Go x 0,06) ~= **6 Go**.  

Mémoire par démon = [Taille du cache en mémoire] + [Taille du tas] + [Marge].

Elle peut être calculée comme suit :

Mémoire d’AM Tez par nœud = [ (Nombre d’AM Tez/nombre de nœuds de démon LLAP) * Taille d’AM Tez].
Taille du conteneur du démon LLAP = [ 90 % de  mémoire du conteneur max YARN ] – [ Mémoire d’AM Tez par nœud ].

Pour les nœuds Worker D14 v2, HDI 4.0, la valeur recommandée est de (90 - (1/1 * 4 Go)) = **86 Go**.
(Pour HDI 3.6, la valeur recommandée est **84 Go,** car vous devez réserver environ 2 Go pour l’AM Slider.)  

### <a name="hivellapiomemorysize"></a>hive.llap.io.memory.size

Cette configuration correspond à la quantité de mémoire disponible sous la forme de cache pour le démon LLAP. Les démons LLAP peuvent utiliser l’espace SSD comme cache. Définissez le paramètre `hive.llap.io.allocator.mmap` = true pour activer la mise en cache SSD. La solution D14 v2 est fournie avec environ 800 Go de disque SSD et la mise en cache SSD est activée par défaut pour le cluster Interactive Query (LLAP). Elle est configurée pour utiliser 50 % de l’espace SSD pour le cache hors tas.

La valeur recommandée est de **409 600 Mo** pour D14 v2.  

Pour les autres machines virtuelles, sans mise en cache SSD activée, il est préférable de fournir une partie de la mémoire RAM disponible pour la mise en cache LLAP afin d’obtenir de meilleures performances. Ajustez la taille totale de la mémoire pour le démon LLAP, comme suit :  

Mémoire total du démon LLAP = [Taille du cache LLAP] + [Taille du tas] + [Marge].

Il est recommandé d’ajuster la taille du cache et du tas en fonction de la charge de travail.  

### <a name="hivellapdaemonnumexecutors"></a>hive.llap.daemon.num.executors

Cette configuration contrôle le nombre d’exécuteurs qui peuvent lancer des tâches en parallèle pour chaque démon LLAP. Cette valeur est un équilibre entre le nombre de vCores disponibles, la quantité de mémoire donnée par exécuteur et la mémoire totale disponible par démon LLAP. En règle générale, nous souhaitons que cette valeur soit aussi proche que possible du nombre de cœurs.

Pour D14 v2, 16 vCores sont disponibles, mais tous les vCores ne peuvent pas être fournis. Les nœuds Worker exécutent également d’autres services, tels que NodeManager, DataNode, et la fonction de surveillance des métriques, qui nécessitent une partie des vCores disponibles. Cette valeur peut être définie sur un maximum de 75 % du nombre total de vCores disponibles sur ce nœud.

Pour D14 v2, la valeur recommandée est (0,75 x 16) = **12**.

Il est recommandé de réserver environ 6 Go d’espace du tas par exécuteur. Ajustez votre nombre d’exécuteurs en fonction de la taille du démon LLAP disponible et du nombre de vCores utilisables par nœud.  

### <a name="hivellapiothreadpoolsize"></a>hive.llap.io.threadpool.size

Cette valeur spécifie la taille du pool de threads pour les exécuteurs. Étant donné que les exécuteurs sont définis de manière fixe comme spécifié, leur nombre est égal à celui des exécuteurs par démon LLAP.

Il est recommandé de définir cette valeur sur **12** pour D14 v2.

Cette configuration ne peut pas dépasser la valeur `yarn.nodemanager.resource.cpu-vcores`.

### <a name="hiveautoconvertjoinnoconditionaltasksize"></a>hive.auto.convert.join.noconditionaltask.size

Vérifiez que vous avez activé `hive.auto.convert.join.noconditionaltask` pour que ce paramètre prenne effet. Cette configuration permet à l’utilisateur de spécifier la taille des tables qui peuvent tenir dans la mémoire pour effectuer une jointure de mappage. Si la taille totale de n-1 de `tables/partitions` pour la jointure à N voies est inférieure à la valeur configurée, la jointure de mappage est choisie. La taille de la mémoire de l’exécuteur LLAP doit être utilisée pour calculer le seuil de conversion automatique en jointure de mappage.

Il est recommandé de définir cette valeur sur **2 048 Mo** pour D14 v2.

Nous vous recommandons d’ajuster cette valeur selon votre charge de travail. En effet, si vous choisissez une valeur trop faible, il se peut que la conversion automatique ne soit pas utilisée. Par contre, une valeur trop élevée peut entraîner des interruptions de GC, ce qui peut nuire aux performances des requêtes.

## <a name="next-steps"></a>Étapes suivantes

* [Gateway guidelines](gateway-best-practices.md) (Instructions relatives à la passerelle)
* [Demystify Apache Tez Memory Tuning – Step by Step](https://community.cloudera.com/t5/Community-Articles/Demystify-Apache-Tez-Memory-Tuning-Step-by-Step/ta-p/245279) (Rationaliser l’ajustement de la mémoire d’Apache Tez : procédure étape par étape)
* [Map Join Memory Sizing For LLAP](https://community.cloudera.com/t5/Community-Articles/Map-Join-Memory-Sizing-For-LLAP/ta-p/247462) (Dimensionnement de la mémoire pour la jointure de mappage pour LLAP)
* [LLAP – a one-page architecture overview](https://community.cloudera.com/t5/Community-Articles/LLAP-a-one-page-architecture-overview/ta-p/247439) (Présentation de l’architecture LLAP sur une page)
* [Hive LLAP deep dive](https://community.cloudera.com/t5/Community-Articles/Hive-LLAP-deep-dive/ta-p/248893) (Présentation approfondie de Hive LLAP)
