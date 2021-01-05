---
title: Foire aux questions sur l’API Cassandra pour Azure Cosmos DB
description: Obtenez des réponses aux questions fréquemment posées sur l’API Cassandra pour Azure Cosmos DB.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: conceptual
ms.date: 08/12/2020
ms.author: thvankra
ms.openlocfilehash: 1368a3174af08f557b6d08f298fba015601d568c
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030830"
---
# <a name="frequently-asked-questions-about-the-cassandra-api-in-azure-cosmos-db"></a>Forum aux questions sur l’API Cassandra dans Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article décrit les différences de fonctionnalités entre Apache Cassandra et l’API Cassandra dans Azure Cosmos DB. Il fournit également des réponses aux questions fréquemment posées sur l’API Cassandra pour Azure Cosmos DB.

## <a name="key-differences-between-apache-cassandra-and-the-cassandra-api"></a>Principales différences entre Apache Cassandra et l’API Cassandra

- Apache Cassandra recommande de limiter à 100 Mo la taille de clé de partition. L’API Cassandra pour Azure Cosmos DB autorise jusqu’à 20 Go par partition.
- Apache Cassandra permet de désactiver les validations durables. Vous pouvez ignorer l’écriture dans le journal de validation et accéder directement aux memtables. Cela peut entraîner une perte de données si le nœud cesse de fonctionner avant que les memtables soient vidées dans des SStables sur disque. Azure Cosmos DB effectue toujours des validations durables pour prévenir la perte de données.
- Les performances d’Apache Cassandra peuvent baisser si la charge de travail implique un grand nombre de remplacements ou suppressions. Cela est dû aux enregistrements désactivés que la charge de travail de lecture doit ignorer pour extraire les données les plus récentes. Les performances de lecture de l’API Cassandra ne baissent pas lorsque la charge de travail inclut un grand nombre de remplacements ou suppressions.
- Dans les scénarios de charges de travail de remplacement lourdes, une compression s’impose pour fusionner les SSTables sur le disque (une fusion est nécessaire parce que les écritures d’Apache Cassandra se font uniquement par ajout ; plusieurs mises à jour sont stockées en tant qu’entrées SSTable individuelles qui doivent être fusionnées périodiquement). Cette situation peut également entraîner une baisse des performances de lecture pendant le compactage. Cet impact sur les performances ne se produit pas dans l’API Cassandra parce que celle-ci n’implémente pas de compactage.
- La définition d’un facteur de réplication de 1 est possible avec Apache Cassandra. Toutefois, cela aboutit à une faible disponibilité si le seul nœud avec les données tombe en panne. Ce n’est pas un problème avec l’API Cassandra pour Azure Cosmos DB parce qu’il existe toujours un facteur de réplication de 4 (quorum de 3).
- L’ajout ou la suppression de nœuds dans Apache Cassandra nécessite une intervention manuelle, ainsi qu’une utilisation élevée du processeur sur le nouveau nœud lorsque les nœuds existants déplacent certaines de leurs plages de jetons vers le nouveau nœud. Cette situation est la même lorsque vous désaffectez un nœud existant. Quoi qu’il en soit, l’API Cassandra effectue un scale-out sans occasionner le moindre problème dans le service ou l’application.
- Il n’est pas nécessaire de définir **num_tokens** sur chaque nœud du cluster comme dans Apache Cassandra. Azure Cosmos DB gère entièrement les nœuds et les plages de jetons.
- L’API Cassandra est entièrement gérée. Vous n’avez pas besoin des commandes **nodetool**, telles que la réparation et la désaffectation, qui sont utilisées dans Apache Cassandra.

## <a name="other-frequently-asked-questions"></a>Autres questions fréquentes

### <a name="what-protocol-version-does-the-cassandra-api-support"></a>Quelle version de protocole l’API Cassandra prend-elle en charge ?

L’API Cassandra pour Azure Cosmos DB prend en charge CQL version 3. Sa compatibilité avec CQL est basée sur le [référentiel GitHub Apache Cassandra](https://github.com/apache/cassandra/blob/trunk/doc/cql3/CQL.textile) public. Si vous avez des commentaires sur la prise en charge d’autres protocoles, faites-le nous savoir via les [commentaires dans User Voice](https://feedback.azure.com/forums/263030-azure-cosmos-db) ou en nous envoyant un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="why-is-choosing-throughput-for-a-table-a-requirement"></a>Pourquoi faut-il obligatoirement choisir un débit pour une table ?

Azure Cosmos DB définit le débit par défaut pour votre conteneur en fonction de l’emplacement à partir duquel vous créez la table : portail Azure ou CQL.

Azure Cosmos DB fournit des garanties de performances et de latence avec des limites supérieures pour les opérations. Ces garanties sont possibles lorsque le moteur peut assurer la gouvernance sur les opérations du client. La définition du débit est pour vous l’assurance de bénéficier du débit et de la latence garantis, car la plateforme réserve cette capacité et garantit le succès des opérations.
Vous avez la possibilité de [modifier le débit de façon élastique](manage-scale-cassandra.md) pour profiter de la saisonnalité de votre application et réaliser des économies.

Le concept de débit est expliqué dans l’article [Unités de requête dans Azure Cosmos DB](request-units.md). Le débit d’une table est réparti uniformément entre les partitions physiques sous-jacentes.

### <a name="what-is-the-throughput-of-a-table-thats-created-through-cql"></a>Quel est le débit d’une table créée via CQL ?

Azure Cosmos DB utilise les unités de requête par seconde (RU/s) comme devise pour offrir le débit. Par défaut, les tables créées via CQL contiennent 400 unités de requête. Vous pouvez modifier les unités de requête à partir du portail.

CQL

```shell
CREATE TABLE keyspaceName.tablename (user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=1200
```

.NET

```csharp
int provisionedThroughput = 400;
var simpleStatement = new SimpleStatement($"CREATE TABLE {keyspaceName}.{tableName} (user_id int PRIMARY KEY, lastname text)");
var outgoingPayload = new Dictionary<string, byte[]>();
outgoingPayload["cosmosdb_provisioned_throughput"] = Encoding.UTF8.GetBytes(provisionedThroughput.ToString());
simpleStatement.SetOutgoingPayload(outgoingPayload);
```

### <a name="what-happens-when-throughput-is-used-up"></a>Que se passe-t-il quand tout le débit a été utilisé ?

Azure Cosmos DB fournit des garanties de performances et de latence avec des limites supérieures pour les opérations. Ces garanties sont possibles lorsque le moteur peut assurer la gouvernance sur les opérations du client. La définition du débit est pour vous l’assurance de bénéficier du débit et de la latence garantis, car la plateforme réserve cette capacité et garantit le succès des opérations.

Lorsque vous dépassez cette capacité, vous recevez le message d’erreur suivant, qui indique que votre capacité est épuisée :

**Surcharge 0x1001 : impossible de traiter la requête, car « Le taux de demandes est élevé »** 

Il est essentiel d’identifier les opérations et les volumes à l’origine de ce problème. Vous pouvez avoir une idée de la capacité utilisée au-delà de la capacité approvisionnée grâce aux métriques du portail Azure. Vous devez ensuite vérifier que cette capacité est répartie presque uniformément entre toutes les partitions sous-jacentes. Si vous constatez qu’une partition utilise la majeure partie du débit, vous avez une asymétrie de la charge de travail.

Des indicateurs de performance sont disponibles pour vous montrer l’utilisation du débit au fil des heures, jours et semaines, sur les différentes partitions, ou sous forme agrégée. Pour plus d’informations, consultez la section [Surveillance et débogage à l’aide de métriques dans Azure Cosmos DB](use-metrics.md).

Les journaux de diagnostic sont expliquées dans l’article [Journalisation des diagnostics Azure Cosmos DB](./monitor-cosmos-db.md).

### <a name="does-the-primary-key-map-to-the-partition-key-concept-of-azure-cosmos-db"></a>La clé primaire est-elle mappée sur le concept de clé de partition d’Azure Cosmos DB ?

Oui. La clé de partition est utilisée pour placer l’entité au bon endroit. Dans Azure Cosmos DB, elle est utilisée pour trouver la bonne partition logique qui est stockée sur une partition physique. Le concept de partitionnement est également expliqué dans l’article [Partitionner et mettre à l’échelle dans Azure Cosmos DB](partitioning-overview.md). Ce qu’il faut retenir, c’est qu’une partition logique ne doit pas dépasser la limite de 20 Go.

### <a name="what-happens-when-i-get-a-notification-that-a-partition-is-full"></a>Que se passe-t-il lorsque j’obtiens une notification indiquant qu’une partition est saturée ?

Azure Cosmos DB est un système basé sur un contrat de niveau de service (SLA). Il offre une mise à l’échelle illimitée, avec des garanties en matière de latence, de débit, de disponibilité et de cohérence. Cet espace de stockage illimité est basé sur un scale-out horizontal des données utilisant le partitionnement comme concept clé. Le concept de partitionnement est également expliqué dans l’article [Partitionner et mettre à l’échelle dans Azure Cosmos DB](partitioning-overview.md).

Vous devez respecter la limite de 20 Go sur le nombre d’entités ou d’éléments par partition logique. Pour vous assurer que la mise à l’échelle de votre application fonctionne correctement, nous vous recommandons de ne *pas* créer de partition sensible en stockant toutes les informations dans une seule partition et en interrogeant celle-ci. Cette erreur ne peut se produire qu’en présence d’une asymétrie des données, c’est-à-dire si vous en avez trop pour une clé de partition (plus de 20 Go). La répartition des données est visible sur le portail de stockage. Pour corriger cette erreur, recréez la table et choisissez une clé principale granulaire (clé de partition), qui permet une meilleure répartition des données.

### <a name="can-i-use-the-cassandra-api-as-a-key-value-store-with-millions-or-billions-of-partition-keys"></a>Puis-je utiliser l’API Cassandra en tant que magasin de valeurs de clé contenant des millions ou milliards de clés de partition ?

Azure Cosmos DB peut stocker des données illimitées en effectuant un scale-out du stockage. Ce stockage est indépendant du débit. Oui vous pouvez toujours utiliser l’API Cassandra pour stocker et récupérer des clés et valeurs en spécifiant la bonne clé primaire/de partition. Ces clés individuelles obtiennent leur propre partition logique et résident sur une partition physique sans problème.

### <a name="can-i-create-more-than-one-table-with-the-cassandra-api"></a>Puis-je créer plusieurs tables avec l’API Cassandra ?

Oui, l’API Cassandra permet de créer plusieurs tables. Chacune de ces tables est traitée en tant qu’unité de débit et de stockage.

### <a name="can-i-create-more-than-one-table-in-succession"></a>Puis-je créer plusieurs tables à la suite ?

Azure Cosmos DB est un système régi par les ressources pour les activités liées aux données et au plan de contrôle. Les conteneurs comme les collections et les tables sont des entités de runtime qui sont approvisionnées pour une capacité de débit déterminées. La création de ces conteneurs dans une succession rapide est une activité non spécifiquement prévue et potentiellement limitée. Si vous avez des tests qui suppriment ou créent des tables immédiatement, essayez de les espacer.

### <a name="what-is-the-maximum-number-of-tables-that-i-can-create"></a>Quel est le nombre maximal de tables que je peux créer ?

Il n’existe aucune limite physique au nombre de tables. Si vous avez un grand nombre de tables (dont la taille totale constante dépasse 10 To de données) à créer au lieu des dizaines ou centaines de tables habituelles, envoyez un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="what-is-the-maximum-number-of-keyspaces-that-i-can-create"></a>Quel est le nombre maximal d’espaces de clés que je peux créer ?

Il n’existe aucune limite physique au nombre d’espaces de clés, car il s’agit de conteneurs de métadonnées. Si vous avez un grand nombre d’espaces de clés, envoyez un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="can-i-bring-in-a-lot-of-data-after-starting-from-a-normal-table"></a>Puis-je importer une grande quantité de données si j’ai commencé à partir d’une table normale ?

Oui. En supposant que les partitions sont distribuées de manière uniforme, la capacité de stockage est gérée automatiquement et augmente à mesure que vous envoyez des données supplémentaires. Par conséquent, vous pouvez importer en toute confiance autant de données que vous le souhaitez sans avoir à gérer et approvisionner des nœuds ou d’autres ressources. En revanche, si vous anticipez une croissance immédiate des données, il est plus judicieux d’[effectuer l’approvisionnement en fonction du débit anticipé](set-throughput.md) directement au lieu de commencer à un niveau plus bas et devoir rehausser celui-ci immédiatement.

### <a name="can-i-use-yaml-file-settings-to-configure-api-behavior"></a>Puis-je utiliser des paramètres de fichier YAML pour configurer le comportement de l’API ?

L’API Cassandra pour Azure Cosmos DB assure la compatibilité au niveau du protocole pour l’exécution des opérations. Elle masque la complexité des opérations de gestion, de supervision et de configuration. En tant que développeur/utilisateur, vous n’avez pas à vous soucier de la disponibilité, des enregistrements désactivés, du cache de clés, du cache de lignes, des filtres de Bloom et d’une multitude d’autres paramètres. L’API Cassandra s’attache à fournir les performances de lecture et d’écriture dont vous avez besoin, sans la surcharge associée à la configuration et à la gestion.

### <a name="will-the-cassandra-api-support-node-addition-cluster-status-and-node-status-commands"></a>L’API Cassandra prend-elle en charge les commandes d’ajout de nœud, d’état de cluster et d’état de nœud ?

L’API Cassandra simplifie les processus de planification de la capacité et de réponse aux demandes d’élasticité pour le débit et le stockage. Azure Cosmos DB vous permet d’approvisionner le débit dont vous avez besoin. Vous pouvez ensuite augmenter ou réduire le débit autant de fois par jour que vous le souhaitez, sans vous soucier des tâches d’ajout, de suppression ou de gestion des nœuds. Vous n’avez pas besoin d’outils pour la gestion des nœuds et des clusters.

### <a name="what-happens-with-various-configuration-settings-for-keyspace-creation-like-simplenetwork"></a>Qu’en est-il des différents paramètres de configuration pour la création d’espaces clés, par exemple simple ou réseau ?

Azure Cosmos DB offre une fonctionnalité de distribution mondiale prête à l’emploi pour répondre aux besoins de haute disponibilité et de faible latence. Vous n’avez pas besoin de configurer des réplicas ou quoi que ce soit. Les écritures sont toujours validées durablement par un quorum dans toute région où vous écrivez des données, tout en offrant des garanties de performances.

### <a name="what-happens-with-various-settings-for-table-metadata"></a>Que se passe-t-il avec les différents paramètres pour les métadonnées de table ?

Azure Cosmos DB offre des garanties de performances pour les lectures, les écritures et le débit. Vous ne devez pas craindre de toucher aux paramètres de configuration et de les manipuler accidentellement. Ces paramètres sont les suivants : filtre de Bloom, mise en cache, changement de réparation des lectures, période de grâce du garbage collection et période de vidage des memtables pour la compression.

### <a name="is-time-to-live-supported-for-cassandra-tables"></a>Les tables Cassandra prennent-elles en charge la durée de vie (TTL) ?

Oui, TTL est pris en charge.

### <a name="how-can-i-monitor-infrastructure-along-with-throughput"></a>Comment puis-je surveiller l’infrastructure en même temps que le débit ?

Azure Cosmos DB est un service de plateforme qui vous permet d’augmenter votre productivité sans vous soucier des opérations de gestion et de surveillance de l’infrastructure. Par exemple, vous n’avez pas à surveiller l’état du nœud, l’état du réplica, le garbage collection et les paramètres du système d’exploitation en amont avec différents outils. Vous devez simplement surveiller le débit relevé dans les métriques accessibles sur le portail pour voir si vous approchez de la limite, et augmenter ou réduire ce débit en conséquence. Vous pouvez :

- Analyser des [contrats de niveau de service](./monitor-cosmos-db.md)
- Utiliser des [métriques](use-metrics.md)
- Utiliser des [journaux de diagnostic](./monitor-cosmos-db.md)

### <a name="which-client-sdks-can-work-with-the-cassandra-api"></a>Quels kits de développement logiciel (SDK) client peuvent fonctionner avec l’API Cassandra ?

Les pilotes clients du Kit de développement logiciel (SDK) Apache Cassandra reposant sur CQLv3 ont été utilisés pour les programmes clients. Si vous utilisez d’autres pilotes ou si vous rencontrez des difficultés, envoyez un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="are-composite-partition-keys-supported"></a>Les clés de partition composites sont-elles prises en charge ?

Oui, vous pouvez utiliser une syntaxe ordinaire pour créer des clés de partition composites.

### <a name="can-i-use-sstableloader-for-data-loading"></a>Puis-je utiliser sstableloader pour le chargement des données ?

Non, sstableloader n’est pas pris en charge.

### <a name="can-i-pair-an-on-premises-apache-cassandra-cluster-with-the-cassandra-api"></a>Puis-je associer un cluster Apache Cassandra local à l’API Cassandra ?

Actuellement, Azure Cosmos DB offre une expérience optimisée pour environnement cloud, sans surcharge liée aux opérations. Si vous avez besoin de procéder à une association, envoyez un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) avec une description de votre scénario. Nous préparons une offre pour vous aider à coupler le cluster Cassandra local ou cloud avec l’API Cassandra pour Azure Cosmos DB.

### <a name="does-the-cassandra-api-provide-full-backups"></a>L’API Cassandra fournit-elle des sauvegardes complètes ?

Azure Cosmos DB réalise deux sauvegardes complètes gratuites toutes les quatre heures, sur toutes les API. Vous n’avez donc pas besoin de planifier les sauvegardes. 

Si vous souhaitez modifier la rétention et la fréquence, envoyez un e-mail à l’adresse [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) ou ouvrez un cas de support. Pour en savoir plus sur la fonctionnalité de sauvegarde, consultez l’article [Sauvegarde et restauration en ligne automatiques avec Azure Cosmos DB](online-backup-and-restore.md).

### <a name="how-does-the-cassandra-api-account-handle-failover-if-a-region-goes-down"></a>Comment le compte d’API Cassandra gère-t-il le basculement en cas de défaillance d’une région ?

L’API Cassandra s’appuie sur la plateforme distribuée mondialement d’Azure Cosmos DB. Pour vérifier que votre application peut tolérer des temps d’arrêt de centre de données, activez au moins une région supplémentaire pour le compte dans le portail Azure. Pour plus d’informations, consultez [Haute disponibilité avec Azure Cosmos DB](high-availability.md).

Vous pouvez ajouter autant de régions que vous le souhaitez pour le compte, et contrôler l’emplacement de basculement en indiquant une priorité. Pour utiliser la base de données, vous devez également fournir une application à cet emplacement. Si vous procédez de la sorte, vos clients ne subissent aucun temps d’arrêt.

### <a name="does-the-cassandra-api-index-all-attributes-of-an-entity-by-default"></a>L’API Apache Cassandra indexe-t-elle par défaut tous les attributs d’une entité ?

Non. L’API Cassandra prend en charge les [index secondaires](cassandra-secondary-index.md), qui se comportent comme dans Apache Cassandra. Par défaut, elle n’indexe pas chaque attribut.  


### <a name="can-i-use-the-new-cassandra-api-sdk-locally-with-the-emulator"></a>Puis-je utiliser le nouveau kit SDK de l’API Cassandra localement avec l’émulateur ?

Oui, cette méthode est prise en charge. Pour plus d’informations sur la façon de l’activer, consultez l’article [Utiliser l’émulateur Azure Cosmos DB à des fins de développement et de test en local](local-emulator.md#cassandra-api).


### <a name="how-can-i-migrate-data-from-apache-cassandra-clusters-to-azure-cosmos-db"></a>Comment puis-je migrer des données de leurs clusters Apache Cassandra vers Azure Cosmos DB ?

Pour plus d’informations sur les options de migration, consultez le didacticiel [Migrer vos données vers un compte d’API Cassandra dans Azure Cosmos DB](cassandra-import-data.md).


### <a name="where-can-i-give-feedback-on-cassandra-api-features"></a>Où puis-je formuler des commentaires sur les fonctionnalités de l’API Cassandra ?

Partagez vos commentaires via les [commentaires user voice](https://feedback.azure.com/forums/263030-azure-cosmos-db).

[azure-portal]: https://portal.azure.com
[query]: ./sql-query-getting-started.md

## <a name="next-steps"></a>Étapes suivantes

- Prenez en main la [mise à l’échelle de manière élastique d’un compte d’API Cassandra Azure Cosmos DB](manage-scale-cassandra.md).
