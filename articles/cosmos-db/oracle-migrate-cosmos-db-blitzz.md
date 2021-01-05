---
title: Migrer des données depuis Oracle vers l’API Cassandra Azure Cosmos DB à l’aide de Blitzz
description: Découvrez comment migrer des données depuis une base de données Oracle vers l’API Cassandra Azure Cosmos DB à l’aide de Blitzz.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 08/21/2019
ms.author: sngun
ms.reviewer: sngun
ms.openlocfilehash: 15bcd0c54fc5f6614f4d1925759704309048acae
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336438"
---
# <a name="migrate-data-from-oracle-to-azure-cosmos-db-cassandra-api-account-using-blitzz"></a>Migrer des données depuis un Oracle vers un compte d’API Cassandra Azure Cosmos DB à l’aide de Blitzz
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

L’API Cassandra d’Azure Cosmos DB est devenue un excellent choix pour les charges de travail d’entreprise exécutées sur Oracle, notamment pour les raisons suivantes :

* **Amélioration de l’extensibilité et de la disponibilité :** Cela élimine les points de défaillance uniques, améliore l’évolutivité et la disponibilité de vos applications.

* **Réduction significative des coûts :** vous pouvez réaliser des économies grâce à Azure Cosmos DB, notamment sur les machines virtuelles, la bande passante et les licences Oracle applicables. En outre, vous n'avez pas à gérer les coûts liés aux centres de données, aux serveurs, au stockage SSD, à la mise en réseau et à la consommation électrique.

* **Aucune surcharge liée à la gestion et à la surveillance :** Comme il s’agit d’un service cloud complètement managé, Azure Cosmos DB élimine la surcharge liée à la gestion et à la supervision d’une multitude de paramètres.

Il existe plusieurs façons de migrer des charges de travail de base de données d'une plateforme vers une autre. [Blitzz](https://www.blitzz.io) est un outil sûr et fiable qui permet de procéder à une migration sans interruption de différentes bases de données vers Azure Cosmos DB. Cet article décrit les étapes requises pour migrer des données à partir d’Oracle Database vers l’API Cassandra Azure Cosmos DB à l’aide de Blitzz.

## <a name="benefits-using-blitzz-for-migration"></a>Avantages de l'utilisation de Blitzz pour la migration

La solution de migration de Blitzz suit une approche étape par étape pour migrer les charges de travail opérationnelles complexes. Voici quelques-uns des aspects clés du plan de migration sans interruption de Blitzz :

* Il offre une migration automatique de la logique métier (tables, index, vues) de la base de données Oracle vers Azure Cosmos DB. Vous n’êtes pas tenu de créer des schémas manuellement.

* Blitzz offre une réplication de base de données parallèle et à haut volume. Il permet de synchroniser les plateformes source et cible pendant la migration à l'aide d'une technique appelée « Change-Data-Capture » (CDC). En utilisant CDC, Blitzz extrait en continu un flux de modifications de la base de données source (Oracle) et l’applique à la base de données de destination (Azure Cosmos DB).

* Il tolère les pannes et garantit une livraison des données en une seule fois, même en cas de défaillance matérielle ou logicielle du système.

* Il sécurise les données pendant le transit grâce à différentes méthodologies de sécurité, telles que TLS/SSL et le chiffrement.

* Il offre des services de conversion de logique métier complexe écrite en PL/SQL en logique métier équivalente dans Azure Cosmos DB.

## <a name="steps-to-migrate-data"></a>Étapes de migration des données

Cette section décrit les étapes requises pour configurer Blitzz et migre entre une base de données Oracle et Azure Cosmos DB.

1. À partir de l’ordinateur sur lequel vous envisagez d’installer le replicant Blitzz, ajoutez un certificat de sécurité. Ce certificat est requis par l’outil de réplication Blitzz pour établir une connexion TLS avec le compte Azure Cosmos DB spécifié. Pour ajouter le certificat, procédez comme suit :

   ```bash
   wget https://cacert.omniroot.com/bc2025.crt
   mv bc2025.crt bc2025.cer
   keytool -keystore $JAVA_HOME/lib/security/cacerts -importcert -alias bc2025ca -file bc2025.cer
   ```

1. Vous pouvez obtenir l’installation de Blitzz et les fichiers binaires soit en demandant une démo sur le [site web Blitzz](https://www.blitzz.io). ou en envoyant un [e-mail](mailto:success@blitzz.io) à l'équipe.

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/blitzz-replicant-download.png" alt-text="Téléchargement de l'outil de réplication Blitzz":::

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/replicant-files.png" alt-text="Fichiers de l'outil de réplication Blitzz":::

1. Depuis le terminal CLI, procédez à la configuration de la base de données source. Ouvrez le fichier de configuration à l’aide de la commande **`vi conf/conn/oracle.yml`** et ajoutez une liste délimitée des adresses IP des nœuds Oracle, du numéro de port, du nom d’utilisateur, du mot de passe et de tout autre détail requis. Le code suivant montre un exemple de fichier de configuration :

   ```bash
   type: ORACLE

   host: localhost
   port: 53546

   service-name: IO

   username: '<Username of your Oracle database>'
   password: '<Password of your Oracle database>'

   conn-cnt: 30
   use-ssl: false
   ```

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/open-connection-editor-oracle.png" alt-text="Ouvrir l’éditeur de connexion Oracle":::

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/oracle-connection-configuration.png" alt-text="Configuration de la connexion Oracle":::

   Après avoir renseigné les détails de la configuration, enregistrez et fermez le fichier.

1. Si vous le souhaitez, vous pouvez configurer le fichier de filtre de la base de données source. Le fichier de filtre spécifie les schémas ou les tables à migrer. Ouvrez le fichier de configuration à l'aide de la commande **`vi filter/oracle_filter.yml`** et entrez les informations de configuration suivantes :

   ```bash

   allow:
   -    schema: “io_blitzz”
   Types: [TABLE]
   ```
 
   Après avoir renseigné les détails du filtre de la base de données, enregistrez et fermez le fichier.

1. Ensuite, vous allez configurer la configuration de la base de données de destination. Avant de définir la configuration, [créez un compte API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account). [Choisissez la clé de partition appropriée](partitioning-overview.md#choose-partitionkey) à partir de vos données et créez ensuite un espace clé et une table pour stocker les données migrées.

1. Avant de migrer les données, augmentez le débit du conteneur jusqu'à la quantité requise pour que votre application migre rapidement. Par exemple, vous pouvez augmenter le débit jusqu'à 100 000 RU. La mise à l’échelle du débit avant le début de la migration vous aidera à migrer vos données plus rapidement. 

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/scale-throughput.png" alt-text="Mettre à l'échelle le débit d'un conteneur Azure Cosmos":::

   Vous devez réduire le débit une fois la migration terminée. En fonction de la quantité de données stockées et des RU requises pour chaque opération, vous pouvez estimer le débit requis après la migration des données. Pour en savoir plus sur la façon d’estimer les RU requises, voir [Approvisionner le débit sur les conteneurs et les bases de données](set-throughput.md)et [Estimer le nombre d’unités de requête/seconde à l’aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md).

1. Procurez-vous le **point de contact, le port, le nom d’utilisateur** et le **mot de passe principal** de votre compte Azure Cosmos dans le volet **Chaîne de connexion**. Vous utiliserez ces valeurs dans le fichier de configuration.

1. Depuis le terminal CLI, procédez à la configuration de la base de données de destination. Ouvrez le fichier de configuration à l’aide de la commande **`vi conf/conn/cosmosdb.yml`** et ajoutez une liste délimitée contenant l’URI hôte, le numéro de port, le nom d’utilisateur, le mot de passe et les autres paramètres requis. Voici un exemple de contenu du fichier de configuration :

   ```bash
   type: COSMOSDB

   host: `<Azure Cosmos account’s Contact point>`
   port: 10350

   username: 'blitzzdemo'
   password: `<Your Azure Cosmos account’s primary password>'

   max-connections: 30
   use-ssl: false
   ```

1. Migrez ensuite les données à l'aide de Blitzz. Vous pouvez exécuter le replicant Blizz en mode **complet** ou **instantané**  :

   * **Mode complet** - Dans ce mode, le replicant continue à s’exécuter après la migration et identifie tout changement sur le système Oracle source. S’il détecte des changements, ceux-ci sont répliqués sur le compte cible Azure Cosmos en temps réel.

   * **Mode instantané** - Dans ce mode, vous pouvez effectuer une migration de schéma et une réplication de données à usage unique. La réplication en temps réel n'est pas prise en charge avec cette option.


   En utilisant les deux modes ci-dessus, la migration peut être effectuée sans temps d’arrêt.

1. Pour migrer les données, à partir du terminal CLI du replicant Blitzz, exécutez la commande suivante :

   ```bash
   ./bin/replicant full conf/conn/oracle.yaml conf/conn/cosmosdb.yaml --filter filter/oracle_filter.yaml --replace-existing
   ```

   L'interface utilisateur de l'outil de réplication affiche la progression de la réplication. Une fois la migration du schéma et l’opération de capture instantanée terminées, la progression est affiche 100 %. Au terme de la migration, vous pouvez valider les données sur la base de données Azure Cosmos cible.

   :::image type="content" source="./media/oracle-migrate-cosmos-db-blitzz/oracle-data-migration-output.png" alt-text="Sortie de la migration des données Oracle":::

1. Comme vous avez utilisé le mode complet pour la migration, vous pouvez effectuer des opérations telles que l’insertion, la mise à jour ou la suppression de données dans la base de données Oracle source. Ultérieurement, vous pourrez constater qu’ils sont répliqués en temps réel sur la base de données cible Azure Cosmos. Après la migration, veillez à diminuer le débit configuré pour votre conteneur Azure Cosmos.

1. Vous pouvez arrêter le replicant à n’importe quel point et le redémarrer avec le commutateur **--resume**. La réplication reprend à partir du moment où elle s’est arrêtée sans compromettre la cohérence des données. La commande suivante montre comment utiliser le commutateur --resume.

   ```bash
   ./bin/replicant full conf/conn/oracle.yaml conf/conn/cosmosdb.yaml --filter filter/oracle_filter.yaml --replace-existing --resume
   ```

Pour en savoir plus sur la migration des données vers la destination et sur la migration en temps réel, consultez la [démo de l'outil de réplication Blitzz](https://www.youtube.com/watch?v=y5ZeRK5A-MI).

## <a name="next-steps"></a>Étapes suivantes

* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md) 
* [Meilleures pratiques en matière de clés de partition](partitioning-overview.md#choose-partitionkey)
* Articles [Estimer le nombre d'unités de requête/seconde à l'aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
