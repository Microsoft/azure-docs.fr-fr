---
title: Haute disponibilité pour Azure Cache pour Redis
description: En savoir plus sur les fonctionnalités et les options de haute disponibilité d’Azure Cache pour Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 10/28/2020
ms.author: yegu
ms.openlocfilehash: e44aed1415f85bf4ea597eac6720207301946b97
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93076909"
---
# <a name="high-availability-for-azure-cache-for-redis"></a>Haute disponibilité pour Azure Cache pour Redis

Azure Cache pour Redis offre une haute disponibilité intégrée. L’objectif de son architecture de haute disponibilité est de garantir le fonctionnement de votre instance Redis gérée, même lorsque ses machines virtuelles sous-jacentes sont concernées par des interruptions planifiées ou non planifiées. Elle offre des taux de pourcentage bien supérieurs à ce qui est réalisable en hébergeant Redis sur une seule machine virtuelle.

Azure Cache pour Redis implémente la haute disponibilité en utilisant plusieurs machines virtuelles, appelées *nœuds* , pour un cache. Il configure ces nœuds de manière à ce que la réplication des données et le basculement se fassent de manière coordonnée. Il orchestre également des opérations de maintenance telles que la mise à jour corrective du logiciel Redis. Diverses options de haute disponibilité sont disponibles dans les niveaux Standard, Premium et Entreprise :

| Option | Description | Disponibilité | standard | Premium | Enterprise |
| ------------------- | ------- | ------- | :------: | :---: | :---: |
| [Réplication standard](#standard-replication)| Configuration répliquée à deux nœuds dans un centre de donnée ou une zone de disponibilité, avec basculement automatique | 99,9 % |✔|✔|-|
| [Cluster Entreprise](#enterprise-cluster) | Instances de cache liées dans deux régions, avec basculement automatique | 99,9 % |-|-|✔|
| [Redondance de zone](#zone-redundancy) | Configuration répliquée à plusieurs nœuds dans les zones de disponibilité, avec basculement automatique | 99,95 % (réplication standard), 99,99 % (cluster Entreprise) |-|✔|✔|
| [Géoréplication](#geo-replication) | Instances de cache liées dans deux régions, avec basculement contrôlé par l’utilisateur | 99,9 % (pour une seule région) |-|✔|-|

## <a name="standard-replication"></a>Réplication standard

Une instance Azure Cache pour Redis dans le niveau Standard ou Premium s’exécute par défaut sur une paire de serveurs Redis. Les deux serveurs sont hébergés sur des machines virtuelles dédiées. Le logiciel Redis open source permet à un seul serveur de traiter les demandes d’écriture de données. Ce serveur est le nœud *principal* , tandis que l’autre est son *réplica*. Après avoir configuré les nœuds de serveur, Azure Cache pour Redis leur attribue des rôles principaux et de réplica. Le nœud principal est généralement chargé de traiter les demandes d’écriture et de lecture des clients Redis. Lors d’une opération d’écriture, il valide une nouvelle clé et une mise à jour de clé dans sa mémoire interne et répond immédiatement au client. Il transfère l’opération au réplica de manière asynchrone.

:::image type="content" source="media/cache-high-availability/replication.png" alt-text="Configuration de la réplication des données":::
   
>[!NOTE]
>Normalement, un client Redis communique avec le nœud principal dans un cache Redis pour toutes les demandes de lecture et d’écriture. Certains clients Redis peuvent être configurés pour lire à partir du nœud de réplica.
>
>

Si le nœud principal d’un cache Redis n’est pas disponible, le réplica se promouvra automatiquement pour devenir le nouveau nœud principal. Ce processus s’appelle le *basculement*. Le réplica attendra suffisamment de temps avant de prendre le relais au cas où le nœud principal se rétablirait rapidement. En cas de basculement, Azure Cache pour Redis approvisionne une nouvelle machine virtuelle et la joint au cache en tant que nœud de réplica. Le réplica effectue une synchronisation complète des données avec le nœud principal afin de disposer d’une autre copie des données du cache.

Un nœud principal peut être mis hors service dans le cadre d’une activité de maintenance planifiée, telle que la mise à jour du système d’exploitation ou du logiciel Redis. Il peut également cesser de fonctionner en raison d’événements imprévus tels que des défaillances du matériel, du logiciel ou du réseau sous-jacent. L’article [Basculement et mise à jour corrective pour Azure Cache pour Redis](cache-failover.md) fournit une explication détaillée sur les types de basculements de Redis. Une instance Azure Cache pour Redis connaîtra de nombreux basculements pendant sa durée de vie. L’architecture de haute disponibilité est conçue pour rendre ces modifications à l’intérieur d’un cache aussi transparente que possible pour ses clients.

>[!NOTE]
>Les éléments suivants sont disponibles en préversion.
>
>

En outre, Azure Cache pour Redis autorise des nœuds de réplica supplémentaires dans le niveau Premium. Un [cache à réplicas multiples](cache-how-to-multi-replicas.md) peut être configuré avec un maximum de trois nœuds de réplica. Le fait de disposer d’un plus grand nombre de réplicas améliore généralement la résilience en raison des nœuds supplémentaires qui viennent en renfort du nœud principal. Même avec davantage de réplicas, une instance Azure Cache pour Redis peut toujours subir les conséquences d’une panne du centre de données ou de la zone de disponibilité. Vous pouvez augmenter la disponibilité du cache en utilisant des réplicas multiples conjointement avec la [redondance de zone](#zone-redundancy).

## <a name="enterprise-cluster"></a>Cluster Entreprise

>[!NOTE]
>Cette option est disponible en préversion.
>
>

Un cache dans l’un ou l’autre des niveaux Entreprise s’exécute sur un cluster Redis Entreprise. Il nécessite un nombre impair de nœuds de serveur à tout moment pour former un quorum. Par défaut, il est composé de trois nœuds, chacun hébergé sur une machine virtuelle dédiée. Un cache Entreprise a deux *nœuds de données* de taille identique et un *nœud de quorum* plus petit. Un cache Enterprise Flash comporte trois nœuds de données de même taille. Le cluster Entreprise divise les données Redis en partitions internes. Chaque partition possède un *principal* et au moins un *réplica*. Chaque nœud de données contient une ou plusieurs partitions. Le cluster Entreprise s’assure que le principal et les réplicas de n’importe quelle partition ne sont jamais colocalisés sur le même nœud de données. Les partitions répliquent les données de manière asynchrone des principaux vers leurs réplicas correspondants.

Lorsqu’un nœud de données devient indisponible ou qu’un fractionnement du réseau se produit, un basculement similaire à celui décrit dans [Réplication standard](#standard-replication) a lieu. Le cluster Entreprise utilise un modèle basé sur le quorum pour déterminer les nœuds survivants qui feront partie d’un nouveau quorum. Il promeut également des partitions de réplica au sein de ces nœuds vers les principaux, le cas échéant.

## <a name="zone-redundancy"></a>Redondance de zone

>[!NOTE]
>Cette option est disponible en préversion.
>
>

Azure Cache pour Redis prend en charge les configurations redondantes interzones dans les niveaux Premium et Entreprise. Un [cache redondant interzone](cache-how-to-zone-redundancy.md) peut placer ses nœuds dans différentes [Zones de disponibilité Azure](../availability-zones/az-overview.md) au sein de la même région. Ainsi, les pannes du centre de donnée ou de la zone de disponibilité ne constituent plus un point de défaillance unique et la disponibilité globale de votre cache en est accrue.

Le diagramme suivant illustre la configuration redondante interzone :

:::image type="content" source="media/cache-high-availability/zone-redundancy.png" alt-text="Configuration de la redondance de zone":::
   
Azure Cache pour Redis distribue les nœuds dans un cache redondant interzone de manière alternée sur les zones de disponibilité que vous avez sélectionnées. Il détermine également quel nœud servira de nœud principal au départ.

Un cache redondant interzone assure le basculement automatique. Lorsque le nœud principal actuel n’est pas disponible, l’un des réplicas prend le relais. Le temps de réponse du cache de votre application peut être plus long si le nouveau nœud principal se trouve dans une autre zone de disponibilité. Les zones de disponibilité se trouvent à des endroits différents. Passer d’une zone de disponibilité à une autre modifie la distance physique entre les emplacements où votre application et votre cache sont hébergés. Cette modification a un impact sur les latences réseau aller-retour entre votre application et le cache. La latence supplémentaire devrait se situer dans une fourchette acceptable pour la plupart des applications. Nous vous recommandons de tester votre application pour vous assurer qu’elle peut fonctionner correctement avec un cache redondant interzone.

## <a name="geo-replication"></a>Géoréplication

La géoréplication est conçue principalement pour la récupération d’urgence. Elle vous donne la possibilité de configurer une instance Azure Cache pour Redis dans une autre région Azure afin de sauvegarder votre cache principal. L’article [Configurer la géoréplication pour Azure Cache pour Redis](cache-how-to-geo-replication.md) fournit une explication détaillée sur le fonctionnement de la géoréplication.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la configuration des options de haute disponibilité d’Azure Cache pour Redis.

* [Niveaux de service Premium Azure Cache pour Redis](cache-overview.md#service-tiers)
* [Ajouter des réplicas à Azure Cache pour Redis](cache-how-to-multi-replicas.md)
* [Activer la redondance de zone pour Azure Cache pour Redis](cache-how-to-zone-redundancy.md)
* [Configurer la géoréplication pour Azure Cache pour Redis](cache-how-to-geo-replication.md)