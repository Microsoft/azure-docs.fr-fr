---
title: Utilisation d’Apache Ambari dans Azure HDInsight
description: Présentation de l’utilisation d’Apache Ambari dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/05/2020
ms.openlocfilehash: cfb83c4ec9972fda3813d414583bc73edeef3229
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285969"
---
# <a name="apache-ambari-usage-in-azure-hdinsight"></a>Utilisation d’Apache Ambari dans Azure HDInsight

HDInsight utilise Apache Ambari pour le déploiement et la gestion de clusters. Des agents Ambari s’exécutent sur chaque nœud (nœud principal, nœud Worker, zookeeper et nœud de périphérie s’il existe). Le serveur Ambari s’exécute uniquement sur le nœud principal (hn0 ou hn1). Une seule instance du serveur Ambari doit être exécutée à la fois. Ceci est contrôlé par le contrôleur de basculement HDInsight. Quand l’un des nœuds principaux est en cours de redémarrage ou de maintenance, les autres nœuds principaux deviennent actifs et le serveur Ambari sur le deuxième nœud principal démarre.

Toutes les configurations de cluster doivent être effectuées par le biais de l’[interface utilisateur Ambari](./hdinsight-hadoop-manage-ambari.md) ; toute modification locale est remplacée quand le nœud est redémarré.

## <a name="failover-controller-services"></a>Services du contrôleur de basculement

Le contrôleur de basculement HDInsight est également chargé de mettre à jour l’adresse IP de l’hôte nœud principal, qui pointe vers le nœud principal actif actuel. Tous les agents Ambari sont configurés pour signaler leur état et leur pulsation à l’hôte nœud principal. Le contrôleur de basculement est un ensemble de services s’exécutant sur chaque nœud du cluster. S’ils ne sont pas en cours d’exécution, le basculement du nœud principal risque de ne pas fonctionner correctement et vous recevrez une erreur HTTP 502 quand vous tenterez d’accéder au serveur Ambari.

Pour vérifier quel nœud principal est actif, une approche consiste à utiliser le protocole SSH pour contacter l’un des nœuds du cluster, puis à exécuter `ping headnodehost` et à comparer l’adresse IP avec celle des deux nœuds principaux.

Si les services du contrôleur de basculement ne sont pas en cours d’exécution, le basculement du nœud principal peut ne pas se produire correctement, et le serveur Ambari risque de ne pas s’exécuter. Pour vérifier si les services du contrôleur de basculement sont en cours d’exécution, exécutez :

```bash
ps -ef | grep failover
```

## <a name="logs"></a>Journaux d’activité

Sur le nœud principal actif, vous pouvez vérifier les journaux du serveur Ambari aux emplacements suivants :

```
/var/log/ambari-server/ambari-server.log
/var/log/ambari-server/ambari-server-check-database.log
```

Sur n’importe quel nœud du cluster, vous pouvez vérifier les journaux de l’agent Ambari à l’emplacement suivant :

```bash
/var/log/ambari-agent/ambari-agent.log
```

## <a name="service-start-sequences"></a>Séquences de démarrage des services

Voici la séquence de démarrage des services au moment du démarrage :

1. Hdinsight-agent démarre les services du contrôleur de basculement.
1. Les services du contrôleur de basculement démarrent l’agent Ambari sur chaque nœud et le serveur Ambari sur le nœud principal actif.

## <a name="ambari-database"></a>Base de données Ambari

HDInsight crée une base de données dans SQL Database en coulisses afin de servir de base de données pour le serveur Ambari. Le [niveau de service par défaut est S0](../azure-sql/database/elastic-pool-scale.md).

Pour tout cluster avec un nombre de nœuds Worker supérieur à 16 lors de la création du cluster, S2 est le niveau de service de la base de données.

## <a name="takeaway-points"></a>Remarques importantes

Ne jamais démarrer/arrêter manuellement les services ambari-server ou ambari-agent, sauf si vous essayez de redémarrer le service pour contourner un problème. Pour forcer un basculement, vous pouvez redémarrer le nœud principal actif.

Ne jamais modifier manuellement les fichiers de configuration sur un nœud de cluster. Laissez plutôt l’interface utilisateur d’Ambari le faire pour vous.

## <a name="next-steps"></a>Étapes suivantes

* [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md)
* [Gérer des clusters HDInsight à l’aide de l’API REST d’Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]