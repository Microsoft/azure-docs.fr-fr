---
title: Meilleures pratiques de développement d’applications – Azure Database pour MySQL
description: Découvrez les meilleures pratiques pour créer une application avec Azure Database pour MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: afe14bc03f0d12e56e1512aeb788a77c64151b58
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547246"
---
# <a name="best-practices-for-building-an-application-with-azure-database-for-mysql"></a>Meilleures pratiques pour créer une application avec Azure Database pour MySQL 

Voici quelques-unes des meilleures pratiques pour vous aider à créer une application cloud à l’aide d’Azure Database pour MySQL. Ces meilleures pratiques peuvent réduire le temps de développement de votre application. 

## <a name="configuration-of-application-and-database-resources"></a>Configuration des ressources d’application et de base de données

### <a name="keep-the-application-and-database-in-the-same-region"></a>Conserver l’application et la base de données dans la même région
Assurez-vous que toutes vos dépendances se trouvent dans la même région lors du déploiement de votre application dans Azure. La répartition des instances sur différentes régions ou zones de disponibilité crée une latence réseau, ce qui peut avoir un impact sur les performances globales de votre application. 

### <a name="keep-your-mysql-server-secure"></a>Sécuriser votre serveur MySQL
Votre serveur MySQL doit être configuré pour être [sécurisé](./concepts-security.md) et non accessible au public. Utilisez l’une des options suivantes pour sécuriser votre serveur : 
- [Règles de pare-feu](./concepts-firewall-rules.md)
- [Réseaux virtuels](./concepts-data-access-and-security-vnet.md) 
- [Azure Private Link](./concepts-data-access-security-private-link.md)

Pour des raisons de sécurité, vous devez toujours vous connecter à votre serveur MySQL via SSL et configurer votre serveur MySQL et votre application pour utiliser TLS 1.2. Voir [Configurer le protocole SSL](./concepts-ssl-connection-security.md). 

### <a name="tune-your-server-parameters"></a>Régler les paramètres de votre serveur
Pour les charges de travail intensives, le réglage des paramètres du serveur, `tmp_table_size` et `max_heap_table_size`, peut aider à optimiser les performances. Pour calculer les valeurs requises pour ces variables, examinez les valeurs totales de la mémoire par connexion et de la mémoire de base. La somme des paramètres de mémoire par connexion, à l’exclusion de `tmp_table_size`, combinée à la mémoire de base, représente la mémoire totale du serveur.

Pour calculer la plus grande taille possible de `tmp_table_size` et `max_heap_table_size`, utilisez la formule suivante :

```(total memory - (base memory + (sum of per-connection memory * # of connections)) / # of connections```

>[!NOTE]
> Mémoire totale indique la quantité totale de mémoire dont dispose le serveur sur les vCores provisionnés.  Par exemple, dans un serveur Azure Database pour MySQL avec deux vCores universel, la mémoire totale sera de 5 Go * 2. Vous trouverez plus de détails sur la mémoire pour chaque niveau de service dans la documentation relative aux [niveaux tarifaires](./concepts-pricing-tiers.md).
>
> La mémoire de base indique les variables de mémoire, comme `query_cache_size` et `innodb_buffer_pool_size`, que MySQL initialise et alloue au démarrage du serveur. La mémoire par connexion, comme `sort_buffer_size` et `join_buffer_size`, correspond à la mémoire qui est allouée uniquement lorsqu’une requête en a besoin.

### <a name="create-non-admin-users"></a>Créer des utilisateurs non-administrateurs 
[Créez des utilisateurs non-administrateurs](./howto-create-users.md) pour chaque base de données. En règle générale, les noms des utilisateurs sont identifiés comme les noms des bases de données.

### <a name="reset-your-password"></a>Réinitialiser votre mot de passe
Vous pouvez [réinitialiser votre mot de passe](./howto-create-manage-server-portal.md#update-admin-password) pour votre serveur MySQL à l’aide du portail Azure. 

La réinitialisation du mot de passe de votre serveur pour une base de données de production peut rendre votre application inactive. Il est recommandé de réinitialiser le mot de passe pour toute charge de travail de production aux heures creuses afin de minimiser l’impact sur les utilisateurs de votre application.

## <a name="performance-and-resiliency"></a>Performance et résilience 
Voici quelques outils et pratiques que vous pouvez utiliser pour vous aider à déboguer les problèmes de performance de votre application.

### <a name="enable-slow-query-logs-to-identify-performance-issues"></a>Activer les journaux de requêtes lentes pour identifier les problèmes de performance
Vous pouvez activer les [journaux de requêtes lentes](./concepts-server-logs.md) et les [journaux d’audit](./concepts-audit-logs.md) sur votre serveur. L’analyse de journaux de requêtes lentes peut aider à identifier les goulots d’étranglement en matière de performances, afin de les résoudre. 

Les journaux d’audit sont également disponibles par le biais des journaux Diagnostics Azure dans les journaux Azure Monitor, Azure Event Hubs et les comptes de stockage. Consultez [Guide pratique pour résoudre les problèmes de performances des requêtes](./howto-troubleshoot-query-performance.md).

### <a name="use-connection-pooling"></a>Utiliser le regroupement de connexions
La gestion des connexions de base de données peut avoir un impact significatif sur les performances de l’application dans son ensemble. Pour optimiser les performances, vous devez réduire le nombre de fois où les connexions sont établies et le temps nécessaire à l’établissement des connexions dans les chemins de code clés. Utilisez le [regroupement de connexions](./concepts-connectivity.md#access-databases-by-using-connection-pooling-recommended) pour vous connecter à Azure Database pour MySQL afin d’améliorer la résilience et les performances. 

Vous pouvez utiliser le regroupement de connexions [ProxySQL](https://proxysql.com/) pour gérer efficacement les connexions. L’utilisation d’une fonction de regroupement de connexions peut réduire le nombre de connexions inactives et la réutilisation des connexions existantes permettra d’éviter tout problème. Pour plus d’informations, consultez [Configuration de ProxySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/connecting-efficiently-to-azure-database-for-mysql-with-proxysql/ba-p/1279842). 

### <a name="retry-logic-to-handle-transient-errors"></a>Logique de nouvelle tentative pour traiter les erreurs temporaires
Votre application peut rencontrer des [erreurs temporaires](./concepts-connectivity.md#handling-transient-errors) où les connexions à la base de données sont interrompues ou perdues par intermittence. Dans ce cas, le serveur est opérationnel après une ou deux nouvelles tentatives en 5 à 10 secondes. 

Nous vous recommandons de patienter 5 secondes avant votre première nouvelle tentative. Ensuite, attendez progressivement plus longtemps entre chaque tentative, jusqu’à un délai de 60 secondes. Limitez le nombre maximal de nouvelles tentatives au-delà duquel votre application considère que l’opération a échoué afin que vous puissiez ensuite poursuivre votre enquête. Pour plus d’informations, consultez [Comment résoudre les erreurs de connexion](./howto-troubleshoot-common-connection-issues.md). 

### <a name="enable-read-replication-to-mitigate-failovers"></a>Activer le réplica en lecture pour atténuer les basculements
Vous pouvez utiliser la [réplication des données entrantes](./howto-data-in-replication.md) pour les scénarios de basculement. Lors de l'utilisation de réplicas de lecture, aucun basculement automatique n'a lieu entre les serveurs source et réplica. 

Vous remarquerez un décalage entre le serveur source et le réplica, car la réplication est asynchrone. Le décalage du réseau dépend d'un grand nombre de facteurs, comme la taille de la charge de travail exécutée sur le serveur source et la latence entre les centres de données. Dans la plupart des cas, le décalage du réplica va de quelques secondes à quelques minutes.

## <a name="database-deployment"></a>Déploiement de base de données 

### <a name="configure-an-azure-database-for-mysql-task-in-your-cicd-deployment-pipeline"></a>Configurer une tâche Azure Database pour MySQL dans votre pipeline de déploiement CI/CD
Parfois, vous devez déployer des modifications dans votre base de données. Dans ce cas, vous pouvez utiliser l’intégration continue (CI) et la livraison continue (CD) via [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) et utiliser une tâche pour [votre serveur MySQL](/azure/devops/pipelines/tasks/deploy/azure-mysql-deployment?view=azure-devops&preserve-view=true) afin de mettre à jour la base de données en exécutant un script personnalisé.

### <a name="use-an-effective-process-for-manual-database-deployment"></a>Utiliser un processus efficace pour le déploiement manuel d’une base de données 
Lors du déploiement manuel d’une base de données, suivez ces étapes pour minimiser le temps d’arrêt ou réduire le risque d’échec du déploiement : 

1. Créez une copie de la base de données de production sur une nouvelle base de données en utilisant [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) ou [MySQL Workbench](https://dev.mysql.com/doc/workbench/en/wb-admin-export-import-management.html). 
2. Mettez à jour la nouvelle base de données avec vos nouvelles modifications de schéma ou les mises à jour nécessaires pour votre base de données. 
3. Placez la base de données de production en lecture seule. Vous ne devez pas avoir d’opérations d’écriture sur la base de données de production tant que le déploiement n’est pas terminé. 
4. Testez votre application avec la base de données que vous venez de mettre à jour à l’étape 1.
5. Déployez les modifications apportées à votre application et assurez-vous que l’application utilise désormais la nouvelle base de données contenant les dernières mises à jour. 
6. Conservez l’ancienne base de données de production afin de pouvoir restaurer les modifications. Vous pouvez alors décider de supprimer l’ancienne base de données de production ou de l’exporter sur le Stockage Azure si nécessaire. 

>[!NOTE]
>Si l’application est semblable à une application d’e-commerce et que vous ne pouvez pas la mettre en lecture seule, déployez les modifications directement sur la base de données de production après avoir effectué une sauvegarde. Ces changements doivent avoir lieu pendant les heures creuses avec un faible trafic vers l’application afin de minimiser l’impact, car certains utilisateurs peuvent rencontrer un échec de requête. 
>
>Assurez-vous que votre code d’application traite également toutes les requêtes ayant échoué.

### <a name="use-mysql-native-metrics-to-see-if-your-workload-is-exceeding-in-memory-temporary-table-sizes"></a>Utiliser les métriques natives de MySQL pour déterminer si votre charge de travail dépasse la taille des tables temporaires en mémoire
Avec une charge de travail nécessitant beaucoup de lectures, les requêtes qui s’exécutent sur votre serveur MySQL peuvent dépasser la taille des tables temporaires en mémoire. Une charge de travail intensive en lecture peut amener votre serveur à passer à l’écriture de tables temporaires sur le disque, ce qui perturbe le niveau de performance de votre application. Pour déterminer si votre serveur écrit sur le disque en cas de dépassement de la taille des tables temporaires, examinez les métriques suivantes :

```
show global status like 'created_tmp_disk_tables';
show global status like 'created_tmp_tables';
```
La métrique `created_tmp_disk_tables` indique le nombre de tables qui ont été créées sur le disque. La métrique `created_tmp_table` indique le nombre de tables temporaires qui doivent être formées en mémoire, en fonction de votre charge de travail. Pour déterminer si l’exécution d’une requête spécifique utilisera des tables temporaires, exécutez l’instruction [EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain.html) sur la requête. Les détails de la colonne `extra` indiquent `Using temporary` si la requête s’exécute à l’aide de tables temporaires.

Pour calculer le pourcentage de votre charge de travail avec des requêtes débordant sur des disques, utilisez vos valeurs de métriques dans la formule ci-dessous :

```(created_tmp_disk_tables / (created_tmp_disk_tables + created_tmp_tables)) * 100```

Dans l’idéal, ce pourcentage doit être inférieur à 25 %. Si vous constatez que le pourcentage est supérieur ou égal à 25 %, nous vous suggérons de modifier deux paramètres de serveur, tmp_table_size et max_heap_table_size.

## <a name="database-schema-and-queries"></a>Schéma de la base de données et requêtes

Voici quelques astuces à garder à l’esprit lorsque vous créez votre schéma de base de données et vos requêtes.

### <a name="use-the-right-datatype-for-your-table-columns"></a>Utiliser le type de données approprié pour les colonnes de votre table
L’utilisation des types de données appropriés en fonction du type de données que vous souhaitez stocker peut optimiser le stockage et réduire les erreurs qui peuvent se produire en raison de types de données incorrects.

### <a name="use-indexes"></a>Utiliser des index
Pour éviter les requêtes lentes, vous pouvez utiliser des index. Les index permettent de trouver rapidement des lignes avec des colonnes spécifiques. Consultez [Comment utiliser des index dans MySQL](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html).

### <a name="use-explain-for-your-select-queries"></a>Utiliser l’instruction EXPLAIN sur vos requêtes SELECT
Utilisez l’instruction `EXPLAIN` pour obtenir des informations sur ce que MySQL fait pour exécuter votre requête. Cela peut vous aider à détecter les goulots d’étranglement ou les problèmes liés à votre requête. Voir [Guide pratique pour utiliser l’instruction EXPLAIN visant à profiler les performances](./howto-troubleshoot-query-performance.md).