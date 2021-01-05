---
title: Migrer des données de Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide de Blitzz
description: Apprenez à migrer des données d'une base de données Apache Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide de Blitzz.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 08/21/2019
ms.author: sngun
ms.reviewer: sngun
ms.openlocfilehash: c26d21e74e9808fe65890b7f4eba31ee742552a4
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339986"
---
# <a name="migrate-data-from-cassandra-to-azure-cosmos-db-cassandra-api-account-using-blitzz"></a>Migrer des données de Cassandra vers un compte API Cassandra Azure Cosmos DB à l'aide de Blitzz
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

L'API Cassandra d'Azure Cosmos DB est devenue un excellent choix pour les charges de travail d'entreprise exécutées sur Apache Cassandra, notamment pour les raisons suivantes : 

* **Aucune surcharge liée à la gestion et à la surveillance :** l'API élimine les surcharges liées à la gestion et à la surveillance d'une multitude de paramètres dans les fichiers du système d'exploitation, les fichiers JVM et les fichiers yaml, ainsi qu'à leurs interactions.

* **Réduction significative des coûts :** vous pouvez réaliser des économies grâce à Azure Cosmos DB, notamment sur les machines virtuelles, la bande passante et les licences applicables. En outre, vous n'avez pas à gérer les coûts liés aux centres de données, aux serveurs, au stockage SSD, à la mise en réseau et à la consommation électrique. 

* **Possibilité d'utiliser le code et les outils existants**  : Azure Cosmos DB fournit une compatibilité au niveau du protocole filaire avec les SDK et outils Cassandra existants. Cette compatibilité garantit la possibilité d’utiliser votre code base de code avec l’API Cassandra Azure Cosmos DB sans changements majeurs.

Il existe plusieurs façons de migrer des charges de travail de base de données d'une plateforme vers une autre. [Blitzz](https://www.blitzz.io) est un outil sûr et fiable qui permet de procéder à une migration sans interruption de différentes bases de données vers Azure Cosmos DB. Cet article décrit les étapes à suivre pour migrer des données d'une base de données Apache Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide de Blitzz.

## <a name="benefits-using-blitzz-for-migration"></a>Avantages de l'utilisation de Blitzz pour la migration

La solution de migration de Blitzz suit une approche étape par étape pour migrer les charges de travail opérationnelles complexes. Voici quelques-uns des aspects clés du plan de migration sans interruption de Blitzz :

* Il offre une migration automatique de la logique métier (tables, index, vues) de la base de données Apache Cassandra vers Azure Cosmos DB. Vous n’êtes pas tenu de créer des schémas manuellement.

* Blitzz offre une réplication de base de données parallèle et à haut volume. Il permet de synchroniser les plateformes source et cible pendant la migration à l'aide d'une technique appelée « Change-Data-Capture » (CDC). En utilisant CDC, Blitzz extrait en continu un flux de modifications de la base de données source (Apache Cassandra) et l'applique à la base de données de destination (Azure Cosmos DB).

* Il tolère les pannes et fournit une livraison des données en une seule fois, même en cas de défaillance matérielle ou logicielle du système.

* Il sécurise les données pendant le transit grâce à différentes méthodologies de sécurité, telles que TLS et le chiffrement.

## <a name="steps-to-migrate-data"></a>Étapes de migration des données

Cette section décrit les étapes à suivre pour configurer Blitzz et migrer les données d'une base de données Apache Cassandra vers Azure Cosmos DB.

1. À partir de l’ordinateur sur lequel vous envisagez d’installer le replicant Blitzz, ajoutez un certificat de sécurité. Ce certificat est requis par l’outil de réplication Blitzz pour établir une connexion TLS avec le compte Azure Cosmos DB spécifié. Pour ajouter le certificat, procédez comme suit :

   ```bash
   wget https://cacert.omniroot.com/bc2025.crt
   mv bc2025.crt bc2025.cer
   keytool -keystore $JAVA_HOME/lib/security/cacerts -importcert -alias bc2025ca -file bc2025.cer
   ```

1. Vous pouvez obtenir les fichiers d'installation de Blitzz et les fichiers binaires en demandant une démo sur le [site web de Blitzz](https://www.blitzz.io) ou en envoyant un [e-mail](mailto:success@blitzz.io) à l'équipe.

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/blitzz-replicant-download.png" alt-text="Téléchargement de l'outil de réplication Blitzz":::

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/replicant-files.png" alt-text="Fichiers de l'outil de réplication Blitzz":::

1. Depuis le terminal CLI, procédez à la configuration de la base de données source. Ouvrez le fichier de configuration à l'aide de la commande **`vi conf/conn/cassandra.yml`** et ajoutez une liste séparée par des virgules contenant les adresses IP des nœuds Cassandra, le numéro de port, le nom d'utilisateur, le mot de passe et toute autre information requise. Voici un exemple de contenu du fichier de configuration :

   ```bash
   type: CASSANDRA
  
   host: 172.17.0.2
   port: 9042

   username: 'cassandra'
   password: 'cassandra'

   max-connections: 30

   ```

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/open-connection-editor-cassandra.png" alt-text="Ouvrez l'éditeur de connexion Cassandra":::

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/cassandra-connection-configuration.png" alt-text="Configuration de la connexion Cassandra":::

   Après avoir renseigné les détails de la configuration, enregistrez et fermez le fichier.

1. Si vous le souhaitez, vous pouvez configurer le fichier de filtre de la base de données source. Le fichier de filtre spécifie les schémas ou les tables à migrer. Ouvrez le fichier de configuration à l'aide de la commande **`vi filter/cassandra_filter.yml`** et entrez les informations de configuration suivantes :

   ```bash

   allow:
   -    schema: “io_blitzz”
   Types: [TABLE]
   ```

   Après avoir renseigné les détails du filtre de la base de données, enregistrez et fermez le fichier.

1. Vous allez ensuite procéder à la configuration de la base de données de destination. Avant d'entamer la configuration, [créez un compte API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account), puis créez un espace de clés et une table pour stocker les données migrées. Comme vous procédez à une migration d'Apache Cassandra vers l'API Cassandra Azure Cosmos DB, vous pouvez utiliser la même clé de partition que celle utilisée avec Apache Cassandra.

1. Avant de migrer les données, augmentez le débit du conteneur jusqu'à la quantité requise pour que votre application migre rapidement. Par exemple, vous pouvez augmenter le débit jusqu'à 100 000 RU. La mise à l’échelle du débit avant le début de la migration vous aidera à migrer vos données plus rapidement.

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/scale-throughput.png" alt-text="Mettez à l'échelle le débit d'un conteneur Azure Cosmos":::

   Au terme de la migration, réduisez le débit. En fonction de la quantité de données stockées et des RU requises pour chaque opération, vous pouvez estimer le débit requis après la migration des données. Pour en savoir plus sur la façon d’estimer les RU requises, voir [Approvisionner le débit sur les conteneurs et les bases de données](set-throughput.md)et [Estimer le nombre d’unités de requête/seconde à l’aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md).

1. Procurez-vous le **point de contact, le port, le nom d’utilisateur** et le **mot de passe principal** de votre compte Azure Cosmos dans le volet **Chaîne de connexion**. Vous utiliserez ces valeurs dans le fichier de configuration.

1. Depuis le terminal CLI, procédez à la configuration de la base de données de destination. Ouvrez le fichier de configuration à l’aide de la commande **`vi conf/conn/cosmosdb.yml`** et ajoutez une liste délimitée contenant l’URI hôte, le numéro de port, le nom d’utilisateur, le mot de passe et les autres paramètres requis. L'exemple suivant présente le contenu du fichier de configuration :

   ```bash
   type: COSMOSDB

   host: '<Azure Cosmos account’s Contact point>'
   port: 10350

   username: 'blitzzdemo'
   password: '<Your Azure Cosmos account’s primary password>'

   max-connections: 30
   ```

1. Migrez ensuite les données à l'aide de Blitzz. Vous pouvez exécuter le replicant Blizz en mode **complet** ou **instantané**  :

   * **Mode complet** - Dans ce mode, l'outil de réplication continue de s'exécuter après la migration et surveille les modifications éventuelles sur le système Apache Cassandra source. S’il détecte des changements, ceux-ci sont répliqués sur le compte cible Azure Cosmos en temps réel.

   * **Mode instantané** - Dans ce mode, vous pouvez effectuer une migration de schéma et une réplication ponctuelle de données. La réplication en temps réel n'est pas prise en charge avec cette option.

   En utilisant les deux modes ci-dessus, la migration peut être effectuée sans temps d’arrêt. 

1. Pour migrer les données, à partir du terminal CLI du replicant Blitzz, exécutez la commande suivante :

   ```bash
   ./bin/replicant full conf/conn/cassandra.yaml conf/conn/cosmosdb.yaml --filter filter/cassandra_filter.yaml --replace-existing
   ```

   L'interface utilisateur de l'outil de réplication affiche la progression de la réplication. Une fois la migration du schéma et l’opération de capture instantanée terminées, la progression est affiche 100 %. Au terme de la migration, vous pouvez valider les données sur la base de données Azure Cosmos cible.

   :::image type="content" source="./media/cassandra-migrate-cosmos-db-blitzz/cassandra-data-migration-output.png" alt-text="Sortie de la migration des données Cassandra":::


1. Comme vous avez utilisé le mode complet pour la migration, vous pouvez effectuer des opérations telles que l'insertion, la mise à jour ou la suppression de données dans la base de données Apache Cassandra source. Par la suite, vous constaterez leur réplication en temps réel sur la base de données Azure Cosmos cible. Après la migration, veillez à diminuer le débit configuré pour votre conteneur Azure Cosmos.

1. Vous pouvez arrêter le replicant à n’importe quel point et le redémarrer avec le commutateur **--resume**. La réplication reprend à partir du moment où elle s’est arrêtée sans compromettre la cohérence des données. La commande suivante montre comment utiliser le commutateur --resume.

   ```bash
   ./bin/replicant full conf/conn/cassandra.yaml conf/conn/cosmosdb.yaml --filter filter/cassandra_filter.yaml --replace-existing --resume
   ```

Pour en savoir plus sur la migration des données vers la destination et sur la migration en temps réel, consultez la [démo de l'outil de réplication Blitzz](https://www.youtube.com/watch?v=fsUhF9LUZmM).

## <a name="next-steps"></a>Étapes suivantes

* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md) 
* [Meilleures pratiques en matière de clés de partition](partitioning-overview.md#choose-partitionkey)
* Articles [Estimer le nombre d'unités de requête/seconde à l'aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
