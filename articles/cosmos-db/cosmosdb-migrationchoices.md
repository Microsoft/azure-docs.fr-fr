---
title: Options de migration Cosmos DB
description: Ce document décrit les différentes options de migration de vos données locales ou cloud vers Azure Cosmos DB
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: 8721c0eb728f568521e86baecb658dc9c869a7f6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097581"
---
# <a name="options-to-migrate-your-on-premises-or-cloud-data-to-azure-cosmos-db"></a>Options de migration de vos données locales ou cloud vers Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Vous pouvez charger des données à partir de différentes sources de données dans Azure Cosmos DB. Comme Azure Cosmos DB prend en charge plusieurs API, les cibles peuvent être des API existantes quelconques. Voici quelques scénarios dans lesquels vous migrez des données vers Azure Cosmos DB :

* Déplacer des données d’un conteneur Azure Cosmos vers un autre conteneur situé dans la même base de données ou dans une autre base de données.
* Déplacer des données entre conteneurs dédiés vers des conteneurs de base de données partagée.
* Déplacer des données depuis un compte Azure Cosmos situé dans une région vers un autre compte Azure Cosmos dans la même région ou dans une autre région.
* Déplacer des données depuis une source telle que le stockage Blob Azure, un fichier JSON, une base de données Oracle, Couchbase ou DynamoDB vers Azure Cosmos DB.

Pour prendre en charge les chemins de migration à partir des différentes sources vers les différentes API Azure Cosmos DB, plusieurs solutions fournissent une gestion spécialisée de chaque chemin de migration. Ce document liste les solutions disponibles et décrit leurs avantages et leurs limites.

## <a name="factors-affecting-the-choice-of-migration-tool"></a>Facteurs affectant le choix de l’outil de migration

Les facteurs suivants déterminent le choix de l’outil de migration :

* **Migration des données en ligne et hors connexion**  : De nombreux outils de migration fournissent un chemin pour effectuer une migration unique. Cela signifie que les applications qui accèdent à la base de données peuvent subir un temps d’arrêt. Certaines solutions de migration offrent un moyen d’effectuer une migration dynamique dans laquelle un pipeline de réplication est configuré entre la source et la cible.

* **Source de données**  : Les données existantes peuvent se trouver dans différentes sources de données comme Oracle DB2, Datastax Cassanda, Azure SQL Database, PostgreSQL, etc. Les données peuvent également figurer dans un compte Azure Cosmos DB existant et l’objectif de la migration peut être de changer le modèle de données ou de repartitionner les données dans un conteneur avec une clé de partition différente.

* **API Azure Cosmos DB**  : Pour l’API SQL dans Azure Cosmos DB, il existe un large éventail d’outils développés par l’équipe Azure Cosmos DB, qui peuvent aider dans les différents scénarios de migration. Toutes les autres API ont leur propre ensemble spécialisé d’outils développés et gérés par la communauté. Comme Azure Cosmos DB prend en charge ces API à un niveau de protocole filaire, ces outils doivent fonctionner en l’état également pendant la migration des données vers Azure Cosmos DB. Toutefois, ils peuvent nécessiter une gestion personnalisée des limitations, car ce concept est spécifique à Azure Cosmos DB.

* **Taille des données**  : La plupart des outils de migration fonctionnent très bien pour de plus petits jeux de données. Lorsque le jeu de données dépasse quelques centaines de gigaoctets, le choix des outils de migration est limité. 

* **Durée de migration attendue**  : Les migrations peuvent être configurées pour s’exécuter à un rythme lent et incrémentiel, qui consomme moins de débit ou qui peut consommer l’intégralité du débit provisionné sur le conteneur Azure Cosmos DB cible et effectuer la migration en un temps réduit.

## <a name="azure-cosmos-db-sql-api"></a>API SQL Azure Cosmos DB

|Type de migration|Solution|Sources prises en charge|Cibles prises en charge|Considérations|
|---------|---------|---------|---------|---------|
|Hors connexion|[Outil de migration de données](import-data.md)| &bull;Fichiers JSON/CSV<br/>&bull;API SQL Azure Cosmos DB<br/>&bull;MongoDB<br/>&bull;SQL Server<br/>&bull;Stockage Table<br/>&bull;AWS DynamoDB<br/>&bull;Stockage Blob Azure|&bull;API SQL Azure Cosmos DB<br/>&bull;API Tables Azure Cosmos DB<br/>&bull;Fichiers JSON |&bull; Facile à configurer et prend en charge plusieurs sources. <br/>&bull; Non adapté aux jeux de données volumineux|
|Hors connexion|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).| &bull;Fichiers JSON/CSV<br/>&bull;API SQL Azure Cosmos DB<br/>&bull;API Azure Cosmos DB pour MongoDB<br/>&bull;MongoDB <br/>&bull;SQL Server<br/>&bull;Stockage Table<br/>&bull;Stockage Blob Azure <br/> <br/>Pour d’autres sources prises en charge, consultez l’article [Azure Data Factory](../data-factory/connector-overview.md).|&bull;API SQL Azure Cosmos DB<br/>&bull;API Azure Cosmos DB pour MongoDB<br/>&bull;Fichiers JSON <br/><br/> Pour d’autres cibles prises en charge, consultez l’article [Azure Data Factory](../data-factory/connector-overview.md). |&bull; Facile à configurer et prend en charge plusieurs sources.<br/>&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB. <br/>&bull; Adapté aux jeux de données volumineux. <br/>&bull; Absence de points de contrôle – Si un problème se produit au cours de la migration, vous devez redémarrer l’ensemble du processus de migration.<br/>&bull; Absence de file d’attente de lettres mortes – Quelques fichiers erronés peuvent arrêter l’ensemble du processus de migration.|
|Hors connexion|[Connecteur Spark Azure Cosmos DB](spark-connector.md)|API SQL Azure Cosmos DB. <br/><br/>Vous pouvez utiliser d’autres sources avec des connecteurs supplémentaires à partir de l’écosystème Spark.| API SQL Azure Cosmos DB. <br/><br/>Vous pouvez utiliser d’autres cibles avec des connecteurs supplémentaires à partir de l’écosystème Spark.| &bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB. <br/>&bull; Adapté aux jeux de données volumineux. <br/>&bull; Nécessite une configuration Spark personnalisée. <br/>&bull; Spark est sensible aux incohérences de schéma et cela peut être un problème lors de la migration. |
|Hors connexion|[Outil personnalisé avec la bibliothèque d’exécuteur en bloc Cosmos DB](migrate-cosmosdb-data.md)| La source dépend de votre code personnalisé | API SQL Azure Cosmos DB| &bull; Fournit des fonctionnalités de points de contrôle et de lettres mortes qui renforcent la résilience de la migration. <br/>&bull; Adapté aux jeux de données très volumineux (plus de 10 To).  <br/>&bull; Nécessite l’installation personnalisée de cet outil exécuté en tant qu’App Service. |
|En ligne|[Fonctions Cosmos DB + API ChangeFeed](change-feed-functions.md)| API SQL Azure Cosmos DB | API SQL Azure Cosmos DB| &bull; Facile à configurer. <br/>&bull; Fonctionne uniquement si la source est un conteneur Azure Cosmos DB. <br/>&bull; Non adapté aux jeux de données volumineux <br/>&bull; Ne capture pas les suppressions à partir du conteneur source. |
|En ligne|[Service de migration personnalisé utilisant ChangeFeed](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator)| API SQL Azure Cosmos DB | API SQL Azure Cosmos DB| &bull; Assure le suivi de la progression. <br/>&bull; Fonctionne uniquement si la source est un conteneur Azure Cosmos DB. <br/>&bull; Fonctionne également pour les jeux de données plus volumineux.<br/>&bull; Exige de l’utilisateur qu’il configure un App Service pour héberger le processeur de flux de modification. <br/>&bull; Ne capture pas les suppressions à partir du conteneur source.|
|En ligne|[Striim](cosmosdb-sql-api-migrate-data-striim.md)| &bull;Oracle <br/>&bull;Apache cassandra<br/><br/> Consultez le [site web Striim](https://www.striim.com/sources-and-targets/) pour les autres sources prises en charge. |&bull;API SQL Azure Cosmos DB <br/>&bull; API Cassandra Azure Cosmos DB<br/><br/> Consultez le [site web Striim](https://www.striim.com/sources-and-targets/) pour les autres cibles prises en charge. | &bull; Fonctionne avec un vaste éventail de sources comme Oracle, DB2, SQL Server.<br/>&bull; Facilite la génération de pipelines ETL et fournit un tableau de bord pour la surveillance. <br/>&bull; Prend en charge des jeux de données plus volumineux. <br/>&bull; Comme il s’agit d’un outil tiers, il doit être acheté sur la place de marché et installé dans l’environnement de l’utilisateur.|

## <a name="azure-cosmos-db-mongo-api"></a>API Mongo d’Azure Cosmos DB

|Type de migration|Solution|Sources prises en charge|Cibles prises en charge|Considérations|
|---------|---------|---------|---------|---------|
|En ligne|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)| MongoDB|API Azure Cosmos DB pour MongoDB |&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB. <br/>&bull; Adapté aux jeux de données volumineux et chargé de la réplication des modifications dynamiques. <br/>&bull; Fonctionne uniquement avec d’autres sources MongoDB.|
|Hors connexion|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)| MongoDB| API Azure Cosmos DB pour MongoDB| &bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB. <br/>&bull; Adapté aux jeux de données volumineux et chargé de la réplication des modifications dynamiques. <br/>&bull; Fonctionne uniquement avec d’autres sources MongoDB.|
|Hors connexion|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).| &bull;Fichiers JSON/CSV<br/>&bull;API SQL Azure Cosmos DB<br/>&bull;API Azure Cosmos DB pour MongoDB <br/>&bull;MongoDB<br/>&bull;SQL Server<br/>&bull;Stockage Table<br/>&bull;Stockage Blob Azure <br/><br/> Pour d’autres sources prises en charge, consultez l’article [Azure Data Factory](../data-factory/connector-overview.md). | &bull;API SQL Azure Cosmos DB<br/>&bull;API Azure Cosmos DB pour MongoDB <br/>&bull; Fichiers JSON <br/><br/> Pour d’autres cibles prises en charge, consultez l’article [Azure Data Factory](../data-factory/connector-overview.md).| &bull; Facile à configurer et prend en charge plusieurs sources. <br/>&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB. <br/>&bull; Adapté aux jeux de données volumineux. <br/>&bull; L’absence de points de contrôle signifie que tout problème au cours de la migration impose le redémarrage de l’ensemble du processus de migration.<br/>&bull; L’absence de file d’attente de lettres mortes signifie que quelques fichiers erronés peuvent arrêter l’ensemble du processus de migration. <br/>&bull; Nécessite du code personnalisé pour augmenter le débit de lecture pour certaines sources de données.|
|Hors connexion|[Outils Mongo existants (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|MongoDB | API Azure Cosmos DB pour MongoDB| &bull; Facile à configurer et à intégrer. <br/>&bull; Nécessite une gestion personnalisée des limitations.|

## <a name="azure-cosmos-db-cassandra-api"></a>API Cassandra Azure Cosmos DB

|Type de migration|Solution|Sources prises en charge|Cibles prises en charge|Considérations|
|---------|---------|---------|---------|---------|
|Hors connexion|[Commande cqlsh COPY](cassandra-import-data.md#migrate-data-using-cqlsh-copy-command)|Fichiers CSV | API Cassandra Azure Cosmos DB| &bull; Facile à configurer. <br/>&bull; Non adapté aux jeux de données volumineux <br/>&bull; Fonctionne uniquement lorsque la source est une table Cassandra.|
|Hors connexion|[Copier la table avec Spark](cassandra-import-data.md#migrate-data-using-spark) | &bull;Apache cassandra<br/>&bull;API Cassandra Azure Cosmos DB| API Cassandra Azure Cosmos DB | &bull; Peut tirer profit des fonctionnalités Spark pour paralléliser la transformation et l’ingestion. <br/>&bull; Nécessite une configuration avec une stratégie personnalisée de nouvelles tentatives pour gérer les limitations.|
|En ligne|[Striim (à partir d’Oracle DB/Apache Cassandra)](cosmosdb-cassandra-api-migrate-data-striim.md)| &bull;Oracle<br/>&bull;Apache cassandra<br/><br/> Consultez le [site web Striim](https://www.striim.com/sources-and-targets/) pour les autres sources prises en charge.|&bull;API SQL Azure Cosmos DB<br/>&bull;API Cassandra Azure Cosmos DB <br/><br/> Consultez le [site web Striim](https://www.striim.com/sources-and-targets/) pour les autres cibles prises en charge.| &bull; Fonctionne avec un vaste éventail de sources comme Oracle, DB2, SQL Server. <br/>&bull; Facilite la génération de pipelines ETL et fournit un tableau de bord pour la surveillance. <br/>&bull; Prend en charge des jeux de données plus volumineux. <br/>&bull; Comme il s’agit d’un outil tiers, il doit être acheté sur la place de marché et installé dans l’environnement de l’utilisateur.|
|En ligne|[ (à partir d’Oracle DB/Apache Cassandra)](oracle-migrate-cosmos-db-blitzz.md)|&bull;Oracle<br/>&bull;Apache cassandra<br/><br/>Consultez le [site web Blitzz](https://www.blitzz.io/) pour les autres sources prises en charge. |API Cassandra Azure Cosmos DB. <br/><br/>Consultez le [site web Blitzz](https://www.blitzz.io/) pour les autres cibles prises en charge. | &bull; Prend en charge des jeux de données plus volumineux. <br/>&bull; Comme il s’agit d’un outil tiers, il doit être acheté sur la place de marché et installé dans l’environnement de l’utilisateur.|

## <a name="other-apis"></a>Autres API

Pour les API autres que l’API SQL, l’API Mongo et l’API Cassandra, différents outils sont pris en charge par les écosystèmes existants de chaque API. 

**API de table** 

* [Outil de migration de données](table-import.md#data-migration-tool)
* [AZCopy](table-import.md#migrate-data-by-using-azcopy)

**API Gremlin**

* [Bibliothèque d’exécuteurs en bloc Graph](bulk-executor-graph-dotnet.md)
* [Gremlin Spark](https://github.com/Azure/azure-cosmosdb-spark/blob/2.4/samples/graphframes/main.scala) 

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus, essayez les exemples d'applications utilisant la bibliothèque d'Exécuteurs en bloc en [.NET](bulk-executor-dot-net.md) et [Java](bulk-executor-java.md). 
* La bibliothèque d'Exécuteurs en bloc est intégrée au connecteur Spark Cosmos DB. Pour plus d'informations, consultez [Connecteur Spark Azure Cosmos DB](spark-connector.md).  
* Pour contacter l’équipe Azure Cosmos DB en ouvrant un ticket de support sous le type de problème « Conseils généraux », sous-type de problème « Grandes (To+) migrations » pour obtenir une aide supplémentaire lors des migrations à grande échelle.
