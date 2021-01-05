---
title: Supervision - Azure Database pour MySQL - Serveur flexible
description: Cet article décrit les métriques de supervision et d’alerte disponibles pour un serveur flexible Azure Database pour MySQL, à savoir des statistiques sur le processeur, le stockage et la connexion.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: 5fe1d5a5a472b47abd364a89d1a65f1249c67c0d
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538661"
---
# <a name="monitor-azure-database-for-mysql-flexible-servers-with-built-in-metrics"></a>Superviser les serveurs flexibles Azure Database pour MySQL à l’aide des métriques intégrées

> [!IMPORTANT] 
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Le serveur flexible Azure Database pour MySQL propose une supervision des serveurs par le biais d’Azure Monitor. Les métriques sont des valeurs numériques qui décrivent certains aspects des ressources de votre serveur à un moment donné. La supervision des ressources de votre serveur facilite la résolution des problèmes liés à votre charge de travail et son optimisation en vous permettant de superviser ce qui vous importe le plus. Superviser les métriques adéquates vous permet de maintenir les performances, la fiabilité et la disponibilité de votre serveur et de vos applications.

Dans cet article, vous allez découvrir les différentes métriques disponibles pour votre serveur flexible, sources d’insights sur le comportement de celui-ci.

## <a name="available-metrics"></a>Métriques disponibles

Le serveur flexible Azure Database pour MySQL propose diverses métriques pour mieux comprendre comment votre charge de travail s’exécute . Sur la base de ces données, vous pouvez appréhender l’impact sur votre serveur et vos applications. Par exemple, sur un serveur flexible, vous pouvez superviser le **pourcentage de processeur hôte** , les **connexions actives** , le **pourcentage d’E/S** et le **pourcentage de mémoire hôte** pour identifier les éventuels impacts sur les performances. À partir de là, vous pouvez éventuellement optimiser votre charge de travail, la mettre à l’échelle verticalement en modifiant les niveaux de calcul ou la mettre à l’échelle horizontalement à l’aide d’un réplica en lecture.

Toutes les métriques Azure présentent une fréquence d’une minute et chaque métrique fournit 30 jours d’historique. Vous pouvez configurer des alertes basées sur les métriques. Pour obtenir des instructions pas à pas, consultez [Guide pratique pour configurer des alertes](./how-to-alert-on-metric.md). Les autres tâches incluent la configuration d’actions automatisées, l’exécution d’analyses avancées et l’archivage de l’historique. Pour plus d’informations, consultez [Vue d’ensemble des mesures dans Microsoft Azure](../../azure-monitor/platform/data-platform.md).

### <a name="list-of-metrics"></a>Liste des métriques
Ces métriques sont disponibles pour Azure Database pour MySQL :

|Nom d’affichage de la métrique|Métrique|Unité|Description|
|---|---|---|---|
|Pourcentage de processeur hôte|cpu_percent|Pourcentage|Pourcentage d’utilisation du processeur sur le serveur, comprenant l’utilisation du processeur par la charge de travail du client et les processus Azure MySQL|
|Réseau entrant hôte |network_bytes_ingress|Octets|Trafic réseau entrant sur le serveur, comprenant le trafic en provenance de la base de données du client et des fonctionnalités Azure MySQL comme la réplication, la supervision, les journaux, etc.|
|Réseau sortant hôte|network_bytes_egress|Octets|Trafic réseau sortant sur le serveur, comprenant le trafic en provenance de la base de données du client et des fonctionnalités Azure MySQL comme la réplication, la supervision, les journaux, etc.|
|Décalage de la réplication|replication_lag|Secondes|Temps écoulé depuis la dernière transaction réexécutée. Cette métrique est disponible pour les serveurs réplicas uniquement.|
|Connexions actives|active_connection|Count|Nombre de connexions actives sur le serveur.|
|Stockage de sauvegarde utilisé|backup_storage_used|Octets|Quantité de stockage de sauvegarde utilisée.|
|Pourcentage d’E/S|io_consumption_percent|Pourcentage|Pourcentage d’E/S en cours d’utilisation.|
|Pourcentage de mémoire hôte|memory_percent|Pourcentage|Pourcentage de mémoire en cours d’utilisation sur le serveur, comprenant l’utilisation de la mémoire par la charge de travail du client et les processus Azure MySQL|
|Limite de stockage|storage_limit|Octets|Stockage maximal pour ce serveur.|
|Pourcentage de stockage|storage_percent|Pourcentage|Pourcentage de stockage utilisé par rapport à la limite maximale du serveur.|
|Stockage utilisé|storage_used|Octets|Quantité de stockage en cours d’utilisation. Le stockage utilisé par le service peut inclure les fichiers de base de données, les journaux d’activité des transactions et les journaux d’activité du serveur.|
|Nombre total de connexions|total_connections|Nombre|Nombre total de connexions au serveur|
|Connexions abandonnées|aborted_connections|Nombre|Nombre de tentatives de connexion au serveur MySQL qui ont échoué, par exemple en raison d’informations d’identification incorrectes.|
|Requêtes|queries|Nombre|Nombre de requêtes par seconde|

## <a name="next-steps"></a>Étapes suivantes
- Consultez le [guide pratique pour configurer des alertes](./how-to-alert-on-metric.md) pour savoir comment créer une alerte sur une métrique.
- Découvrez-en plus sur la [mise à l’échelle des IOPS](./concepts/../concepts-compute-storage.md#iops) pour améliorer les performances.