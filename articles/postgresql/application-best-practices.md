---
title: Bonnes pratiques de développement d’applications – Azure Database pour PostgreSQL
description: Découvrez les bonnes pratiques pour créer une application avec Azure Database pour PostgreSQL.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: conceptual
ms.date: 12/10/2020
ms.openlocfilehash: 6463f30bc79d937bd5a51a5c8c78fbdd72954b1e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97364529"
---
# <a name="best-practices-for-building-an-application-with-azure-database-for-postgresql"></a>Bonnes pratiques pour créer une application avec Azure Database pour PostgreSQL

Voici quelques-unes des bonnes pratiques pour vous aider à créer une application cloud à l’aide d’Azure Database pour PostgreSQL. Ces meilleures pratiques peuvent réduire le temps de développement de votre application.

## <a name="configuration-of-application-and-database-resources"></a>Configuration des ressources d’application et de base de données

### <a name="keep-the-application-and-database-in-the-same-region"></a>Conserver l’application et la base de données dans la même région
Assurez-vous que toutes vos dépendances se trouvent dans la même région lors du déploiement de votre application dans Azure. La répartition des instances sur différentes régions ou zones de disponibilité crée une latence réseau, ce qui peut avoir un impact sur les performances globales de votre application.

### <a name="keep-your-postgresql-server-secure"></a>Sécuriser votre serveur PostgreSQL
Votre serveur PostgreSQL doit être configuré pour être [sécurisé](./concepts-security.md) et non accessible au public. Utilisez l’une des options suivantes pour sécuriser votre serveur :
- [Règles de pare-feu](./concepts-firewall-rules.md)
- [Réseaux virtuels](./concepts-data-access-and-security-vnet.md)
- [Azure Private Link](./concepts-data-access-and-security-private-link.md)

Pour des raisons de sécurité, vous devez toujours vous connecter à votre serveur PostgreSQL via SSL et configurer votre serveur PostgreSQL et votre application pour utiliser TLS 1.2. Voir [Configurer le protocole SSL](./concepts-ssl-connection-security.md).

### <a name="tune-your-server-parameters"></a>Régler les paramètres de votre serveur
Pour les charges de travail intensives, le réglage des paramètres du serveur, `tmp_table_size` et `max_heap_table_size`, peut aider à optimiser les performances. Pour calculer les valeurs requises pour ces variables, examinez les valeurs totales de la mémoire par connexion et de la mémoire de base. La somme des paramètres de mémoire par connexion, à l’exclusion de `tmp_table_size`, combinée à la mémoire de base, représente la mémoire totale du serveur.

### <a name="use-environment-variables-for-connection-information"></a>Utiliser des variables d’environnement pour les informations de connexion
N’enregistrez pas vos informations d’identification de base de données dans le code de votre application. Selon l’application front-end, suivez les instructions pour configurer les variables d’environnement. Pour l’utilisation d’un service d’application, consultez [Configuration des paramètres d’application](../app-service/configure-common.md#configure-app-settings) et pour un service Azure Kubernetes, consultez l’article sur l’[utilisation des secrets Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/).

## <a name="performance-and-resiliency"></a>Performance et résilience
Voici quelques outils et pratiques que vous pouvez utiliser pour vous aider à déboguer les problèmes de performance de votre application.

### <a name="use-connection-pooling"></a>Utiliser le regroupement de connexions
Avec le regroupement de connexions, un ensemble fixe de connexions est établi au moment du démarrage et conservé. Cela permet également de réduire la fragmentation de la mémoire sur le serveur, qui est provoquée par les nouvelles connexions dynamiques établies sur le serveur de base de données. Le regroupement de connexions peut être configuré côté application si le framework d’application ou le pilote de base de données le prend en charge. S’il n’est pas pris en charge, l’autre option recommandée consiste à utiliser un service de regroupement de connexions proxy comme [PgBouncer](https://pgbouncer.github.io/) ou [Pgpool](https://pgpool.net/mediawiki/index.php/Main_Page) qui s’exécute en dehors de l’application et se connecte au serveur de base de données. PgBouncer et Pgpool sont des outils basés sur la communauté qui fonctionnent avec Azure Database pour PostgreSQL.

### <a name="retry-logic-to-handle-transient-errors"></a>Logique de nouvelle tentative pour traiter les erreurs temporaires
Votre application peut rencontrer des erreurs temporaires où les connexions à la base de données sont interrompues ou perdues par intermittence. Dans ce cas, le serveur est opérationnel après une ou deux nouvelles tentatives en 5 à 10 secondes. Nous vous recommandons de patienter 5 secondes avant votre première nouvelle tentative. Ensuite, attendez progressivement plus longtemps entre chaque tentative, jusqu’à un délai de 60 secondes. Limitez le nombre maximal de nouvelles tentatives au-delà duquel votre application considère que l’opération a échoué afin que vous puissiez ensuite poursuivre votre enquête. Pour plus d’informations, consultez [Comment résoudre les erreurs de connexion](./concepts-connectivity.md).

### <a name="enable-read-replication-to-mitigate-failovers"></a>Activer le réplica en lecture pour atténuer les basculements
Vous pouvez utiliser la [réplication des données entrantes](./concepts-read-replicas.md) pour les scénarios de basculement. Lors de l’utilisation de réplicas de lecture, aucun basculement automatique n’a lieu entre les serveurs source et réplica. Vous remarquerez un décalage entre le serveur source et le réplica, car la réplication est asynchrone. Le décalage du réseau dépend d'un grand nombre de facteurs, comme la taille de la charge de travail exécutée sur le serveur source et la latence entre les centres de données. Dans la plupart des cas, le décalage du réplica va de quelques secondes à quelques minutes.


## <a name="database-deployment"></a>Déploiement de base de données

### <a name="configure-cicd-deployment-pipeline"></a>Configurer un pipeline de déploiement CI/CD
Parfois, vous devez déployer des modifications dans votre base de données. Dans ce cas, vous pouvez utiliser l’intégration continue (CI) via des [actions GitHub](https://github.com/Azure/postgresql/blob/master/README.md) pour votre serveur PostgreSQL afin de mettre à jour la base de données en exécutant un script personnalisé.

### <a name="define-manual-database-deployment-process"></a>Définir le processus de déploiement manuel d’une base de données
Lors du déploiement manuel d’une base de données, suivez ces étapes pour minimiser le temps d’arrêt ou réduire le risque d’échec du déploiement :

- Créez une copie d’une base de données de production sur une nouvelle base de données en utilisant pg_dump.
- Mettez à jour la nouvelle base de données avec vos nouvelles modifications de schéma ou les mises à jour nécessaires pour votre base de données.
- Placez la base de données de production en lecture seule. Vous ne devez pas avoir d’opérations d’écriture sur la base de données de production tant que le déploiement n’est pas terminé.
- Testez votre application avec la base de données que vous venez de mettre à jour à l’étape 1.
- Déployez les modifications apportées à votre application et assurez-vous que l’application utilise désormais la nouvelle base de données contenant les dernières mises à jour.
- Conservez l’ancienne base de données de production afin de pouvoir restaurer les modifications. Vous pouvez alors décider de supprimer l’ancienne base de données de production ou de l’exporter sur le Stockage Azure si nécessaire.

>  [!NOTE]
> Si l’application est semblable à une application d’e-commerce et que vous ne pouvez pas la mettre en lecture seule, déployez les modifications directement sur la base de données de production après avoir effectué une sauvegarde. Ces changements doivent avoir lieu pendant les heures creuses avec un faible trafic vers l’application afin de minimiser l’impact, car certains utilisateurs peuvent rencontrer un échec de requête. Assurez-vous que votre code d’application traite également toutes les requêtes ayant échoué.

## <a name="database-schema-and-queries"></a>Schéma de la base de données et requêtes
Voici quelques astuces à garder à l’esprit lorsque vous créez votre schéma de base de données et vos requêtes.

### <a name="use-bigint-or-uuid-for-primary-keys"></a>Utiliser BIGINT ou UUID pour les clés primaires
Lors de la création d’une application personnalisée ou de quelques frameworks, vous pouvez utiliser `INT` au lieu de `BIGINT` pour les clés primaires. Lorsque vous utilisez ```INT```, vous risquez de dépasser la capacité de stockage du type de données ```INT``` dans votre base de données. Cette modification apportée à une application de production existante peut s’avérer chronophage, avec un plus grand coût en temps de développement. Une autre option consiste à utiliser [UUID](https://www.postgresql.org/docs/current/datatype-uuid.html) pour les clés primaires. Cet identificateur utilise une chaîne 128 bits générée automatiquement, telle que ```a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11```. Apprenez-en davantage sur les [types de données PostgreSQL](https://www.postgresql.org/docs/8.1/datatype.html).

### <a name="use-indexes"></a>Utiliser des index

Il existe de nombreux types d’[index](https://www.postgresql.org/docs/9.1/indexes.html) dans Postgres, qui peuvent être utilisés de diverses façons. L’utilisation d’un index permet au serveur de rechercher et de récupérer des lignes spécifiques beaucoup plus rapidement qu’il peut le faire sans index. Toutefois, les index ajoutent également une surcharge au serveur de base de données, de sorte qu’il vaut mieux éviter la présence d’un trop grand nombre d’index.

### <a name="use-autovacuum"></a>Utiliser le nettoyage automatique
Vous pouvez optimiser votre serveur avec le nettoyage automatique sur un serveur Azure Database pour PostgreSQL. PostgreSQL autorise un meilleur accès concurrentiel aux bases de données, mais chaque mise à jour entraîne une insertion et une suppression. Chaque suppression entraîne le marquage logiciel des enregistrements qui seront supprimés ultérieurement. Pour exécuter ces tâches, PostgreSQL exécute une tâche de nettoyage. Si vous n’effectuez pas de nettoyage régulièrement, les tuples morts qui s’accumulent peuvent avoir les conséquences suivantes :

- Ballonnement des données, par exemple, des tables et des bases de données plus volumineuses
- Accroissement de la taille des index non optimaux
- Augmentation des E/S

Apprenez-en davantage sur l’[optimisation du nettoyage automatique](howto-optimize-autovacuum.md).

### <a name="use-pg_stats_statements"></a>Utiliser pg_stats_statements
Pg_stat_stat_stat_statements est une extension PostgreSQL qui est activée par défaut dans Azure Database pour PostgreSQL. L’extension fournit un moyen de suivre les statistiques d’exécution pour toutes les instructions SQL exécutées par un serveur. Consultez [Utilisation de pg_statement](howto-optimize-query-stats-collection.md).


### <a name="use-the-query-store"></a>Utiliser le magasin de données des requêtes
La fonctionnalité [Magasin des requêtes](./concepts-query-store.md) dans Azure Database pour PostgreSQL fournit un moyen plus efficace de suivre les performances des requêtes. Nous recommandons cette fonctionnalité comme alternative à l’utilisation de pg_stats_statements.

### <a name="optimize-bulk-inserts-and-use-transient-data"></a>Optimiser les insertions en bloc et utiliser des données temporaires
Si vos opérations de charge de travail impliquent des données temporaires ou insèrent des jeux de données volumineux en bloc, envisagez d’utiliser des tables non journalisées. Elle offre l’atomicité et la durabilité, par défaut. L’atomicité, la cohérence, l’isolation et la durabilité sont les propriétés d’ACID. Consultez la [procédure d’optimisation des insertions en bloc](howto-optimize-bulk-inserts.md).

## <a name="next-steps"></a>Étapes suivantes
[Guide Postgres](http://postgresguide.com/)
