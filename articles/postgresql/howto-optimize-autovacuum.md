---
title: Optimiser le nettoyage automatique - Azure Database pour PostgreSQL – Serveur unique
description: Cet article décrit comment optimiser le nettoyage automatique sur un serveur Azure Database pour PostgreSQL – Serveur unique
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: how-to
ms.date: 07/09/2020
ms.openlocfilehash: a94afc1ab970c2cd3f509c86efba4e455d46fd13
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012559"
---
# <a name="optimize-autovacuum-on-an-azure-database-for-postgresql---single-server"></a>Optimiser le nettoyage automatique sur un serveur Azure Database pour PostgreSQL – Serveur unique

Cet article explique comment optimiser le nettoyage automatique sur un serveur Azure Database pour PostgreSQL.

## <a name="overview-of-autovacuum"></a>Vue d’ensemble du nettoyage automatique

PostgreSQL utilise le contrôle d’accès concurrentiel multiversion (MVCC) pour bénéficier d’un meilleur accès simultané à la base de données. Chaque mise à jour entraîne une insertion et une suppression, et chaque suppression entraîne le marquage adouci des lignes à supprimer. Le marquage adouci identifie les tuples morts qui seront vidés ultérieurement. Pour exécuter ces tâches, PostgreSQL exécute une tâche de nettoyage.

Vous pouvez déclencher un travail de nettoyage manuellement ou automatiquement. Il y a davantage de tuples morts quand la base de données subit une quantité importante d’opérations de mise à jour ou de suppression. Il y a moins de tuples morts lorsque la base de données est inactive. Lorsque la charge de la base de données est importante, vous devez effectuer des nettoyages plus fréquemment. L’exécution *manuelle* de ces tâches de nettoyage n’est donc pas pratique.

Vous pouvez configurer le nettoyage automatique et l’optimiser grâce à un réglage. Les valeurs par défaut de PostgreSQL permettent au produit de fonctionner sur tous les types d’appareils. Ces appareils comprennent notamment les appareils Raspberry Pi. Les valeurs de configuration idéales dépendent des éléments suivants :

- Quantité totale des ressources disponibles, par exemple, taille de stockage et référence SKU
- Utilisation des ressources
- Caractéristiques des différents objets

## <a name="autovacuum-benefits"></a>Avantages du nettoyage automatique

Si vous n’effectuez pas de nettoyage régulièrement, les tuples morts qui s’accumulent peuvent avoir les conséquences suivantes :

- Ballonnement des données, par exemple, des tables et des bases de données plus volumineuses
- Accroissement de la taille des index non optimaux
- Augmentation des E/S

## <a name="monitor-bloat-with-autovacuum-queries"></a>Superviser le ballonnement avec des requêtes de nettoyage automatique

L’exemple de requête suivant est conçu pour identifier le nombre de tuples vivants et morts dans une table nommée XYZ :

```sql
SELECT relname, n_dead_tup, n_live_tup, (n_dead_tup/ n_live_tup) AS DeadTuplesRatio, last_vacuum, last_autovacuum FROM pg_catalog.pg_stat_all_tables WHERE relname = 'XYZ' order by n_dead_tup DESC;
```

## <a name="autovacuum-configurations"></a>Configurations de nettoyage automatique

Les paramètres de configuration qui contrôlent le nettoyage automatique sont axés sur deux questions clés :

- Quand doit-il commencer ?
- Quelle quantité doit-il nettoyer une fois commencé ?

Voici des paramètres de configuration du nettoyage automatique que vous pouvez mettre à jour en fonction des réponses aux questions précédentes, ainsi que quelques conseils.

Paramètre|Description|Valeur par défaut
---|---|---
autovacuum_vacuum_threshold|Spécifie le nombre minimal de tuples mis à jour ou supprimés pour déclencher une opération de nettoyage dans une table. La valeur par défaut est de 50 tuples. Définissez ce paramètre uniquement dans le fichier postgresql.conf ou sur la ligne de commande du serveur. Pour substituer ce paramètre au niveau de chaque table, changez les paramètres de stockage de la table.|50
autovacuum_vacuum_scale_factor|Spécifie une fraction de la taille de la table à ajouter à autovacuum_vacuum_threshold lorsque vous décidez qu’il faut déclencher une opération de nettoyage. La valeur par défaut est de 0,2 (c’est-à-dire, 20 % de la taille de la table). Définissez ce paramètre uniquement dans le fichier postgresql.conf ou sur la ligne de commande du serveur. Pour substituer ce paramètre au niveau de chaque table, changez les paramètres de stockage de la table.|0.2
autovacuum_vacuum_cost_limit|Spécifie la valeur limite du coût qui doit être utilisée dans les opérations de nettoyage automatique. Si vous spécifiez -1 (la valeur par défaut), la valeur régulière de vacuum_cost_limit est utilisée. S’il existe plusieurs workers, la valeur est répartie proportionnellement entre les workers de nettoyage automatique actuellement exécutés. La somme des limites pour chaque worker ne dépasse pas la valeur de cette variable. Définissez ce paramètre uniquement dans le fichier postgresql.conf ou sur la ligne de commande du serveur. Pour substituer ce paramètre au niveau de chaque table, changez les paramètres de stockage de la table.|-1
autovacuum_vacuum_cost_delay|Spécifie la valeur de délai du coût qui doit être utilisée dans les opérations de nettoyage automatique. Si vous spécifiez -1, la valeur régulière de vacuum_cost_delay est utilisée. La valeur par défaut est de 20 millisecondes. Définissez ce paramètre uniquement dans le fichier postgresql.conf ou sur la ligne de commande du serveur. Pour substituer ce paramètre au niveau de chaque table, changez les paramètres de stockage de la table.|20 ms
autovacuum_naptime | Spécifie le délai minimal entre les exécutions de nettoyage automatique sur une base de données spécifique. À chaque cycle, le démon examine la base de données et émet des commandes de nettoyage et d’analyse en fonction des besoins pour les tables de cette base de données. Le délai est exprimé en secondes. Définissez ce paramètre uniquement dans le fichier postgresql.conf ou sur la ligne de commande du serveur.| 15 s
autovacuum_max_workers | Spécifie le nombre maximal de processus de nettoyage automatique (autres que le lanceur de nettoyage automatique) qui peuvent s’exécuter à tout moment. La valeur par défaut est trois. Définissez ce paramètre uniquement au démarrage du serveur.|3

Pour substituer ces paramètres au niveau de chaque table, changez les paramètres de stockage de la table.

## <a name="autovacuum-cost"></a>Coût du nettoyage automatique

Voici les « coûts » qu’entraîne l’exécution d’une opération de nettoyage automatique :

- Les pages de données sur lesquelles le nettoyage est exécuté sont verrouillées.
- Des ressources de calcul et de mémoire sont utilisées lors de l’exécution d’une tâche de nettoyage.

Par conséquent, veillez à ne pas exécuter les tâches de nettoyage trop fréquemment ou trop rarement. Une tâche de nettoyage doit être adaptée à la charge de travail. Testez tous les changements des paramètres de nettoyage automatique, en raison des inconvénients liés à chacun d’eux.

## <a name="autovacuum-start-trigger"></a>Déclencheur de démarrage du nettoyage automatique

Le nettoyage automatique est déclenché quand le nombre de tuples morts dépasse autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * reltuples. Ici, reltuples est une constante.

Le nettoyage automatique doit s’adapter à la charge de la base de données. Sinon, vous pouvez manquer de stockage et constater un ralentissement général des requêtes. Amortie au fil du temps, la vitesse d’une opération de nettoyage des tuples morts doit être égale à la vitesse de création des tuples morts.

Les bases de données contenant de nombreuses mises à jour et suppressions ont plus de tuples morts et ont besoin de davantage d’espace. En général, avec les bases de données soumises à de nombreuses mises à jour et suppressions, il est préférable d’affecter des valeurs basses à autovacuum_vacuum_scale_factor et à autovacuum_vacuum_threshold. Les valeurs basses empêchent l’accumulation prolongée de tuples morts. Vous pouvez utiliser des valeurs plus élevées pour ces deux paramètres avec les bases de données plus petites, car la nécessité d’un nettoyage est moins urgente. Les nettoyages fréquents représentent un coût en calcul et en mémoire.

Le facteur d’échelle par défaut de 20 % fonctionne bien pour les tables ayant un faible pourcentage de tuples morts. En revanche, il ne fonctionne pas bien sur les tables ayant un pourcentage élevé de tuples morts. Par exemple, dans une table de 20 Go, ce facteur d’échelle se traduit par 4 Go de tuples morts. Dans une table de 1 To, il se traduit par 200 Go de tuples morts.

Avec PostgreSQL, vous pouvez définir ces paramètres au niveau de l’instance ou de la table. Aujourd’hui, vous pouvez définir ces paramètres au niveau de la table uniquement dans Azure Database pour PostgreSQL.

## <a name="estimate-the-cost-of-autovacuum"></a>Estimer le coût du nettoyage automatique

L’exécution du nettoyage automatique est « coûteuse », et il existe des paramètres pour contrôler l’exécution de telles opérations. Les paramètres suivants aident à estimer le coût d’exécution du nettoyage :

- vacuum_cost_page_hit = 1
- vacuum_cost_page_miss = 10
- vacuum_cost_page_dirty = 20

Le processus de nettoyage lit les pages physiques et vérifie la présence de tuples morts. Chaque page de shared_buffers a un coût de 1 (vacuum_cost_page_hit). Toutes les autres pages sont considérées comme ayant un coût de 20 (vacuum_cost_page_dirty) s’il existe des tuples morts, ou de 10 (vacuum_cost_page_miss) s’il n’existe aucun tuple mort. L’opération de nettoyage s’arrête quand le processus dépasse la valeur de autovacuum_vacuum_cost_limit.

Une fois la limite atteinte, le processus bascule en veille pendant la durée spécifiée par le paramètre autovacuum_vacuum_cost_delay, avant d’être redémarré. Si la limite n’est pas atteinte, le nettoyage automatique démarre après la valeur spécifiée par le paramètre autovacuum_nap_time.

En résumé, les paramètres autovacuum_vacuum_cost_delay et autovacuum_vacuum_cost_limit contrôlent la quantité de nettoyage de données autorisée par unité de temps. Notez que les valeurs par défaut sont trop faibles pour la plupart des niveaux tarifaires. Les valeurs optimales pour ces paramètres dépendent du niveau de tarification, et doivent être configurées en conséquence.

Le paramètre autovacuum_max_workers détermine le nombre maximal de processus de nettoyage automatique pouvant être exécutés simultanément.

Avec PostgreSQL, vous pouvez définir ces paramètres au niveau de l’instance ou de la table. Aujourd’hui, vous pouvez définir ces paramètres au niveau de la table uniquement dans Azure Database pour PostgreSQL.

## <a name="optimize-autovacuum-per-table"></a>Optimiser le nettoyage automatique pour chaque table

Vous pouvez configurer tous les paramètres de configuration précédente pour chaque table. Voici un exemple :

```sql
ALTER TABLE t SET (autovacuum_vacuum_threshold = 1000);
ALTER TABLE t SET (autovacuum_vacuum_scale_factor = 0.1);
ALTER TABLE t SET (autovacuum_vacuum_cost_limit = 1000);
ALTER TABLE t SET (autovacuum_vacuum_cost_delay = 10);
```

Le nettoyage automatique est un processus synchrone exécuté par table. Plus une table contient un pourcentage élevé de tuples morts, plus le « coût » du nettoyage automatique est élevé. Vous pouvez fractionner en plusieurs tables les tables qui ont un taux élevé de mises à jour et de suppressions. Le fractionnement des tables permet d’exécuter parallèlement plusieurs nettoyages automatiques et de réduire le « coût » lié au nettoyage automatique d’une table. Vous pouvez également augmenter le nombre de workers parallèles de nettoyage automatique afin de garantir une planification libérale des workers.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation et le paramétrage du nettoyage automatique, consultez la documentation PostgreSQL suivante :

- [Chapter 18, Server configuration](https://www.postgresql.org/docs/9.5/static/runtime-config-autovacuum.html)
- [Chapter 24, Routine database maintenance tasks](https://www.postgresql.org/docs/9.6/static/routine-vacuuming.html)
