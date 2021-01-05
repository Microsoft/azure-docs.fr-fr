---
title: Fonctionnalités Apache Cassandra prises en charge par l’API Cassandra Azure Cosmos DB
description: Découvrir la prise en charge des fonctionnalités Apache Cassandra dans l’API Cassandra Azure Cosmos DB
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 8c51450fb6ce5c381784e6aaf9b1a66c3c4ff153
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188545"
---
# <a name="apache-cassandra-features-supported-by-azure-cosmos-db-cassandra-api"></a>Fonctionnalités Apache Cassandra prises en charge par l’API Cassandra Azure Cosmos DB 
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Vous pouvez communiquer avec l’API Cassandra Azure Cosmos DB par le biais des [pilotes](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver) clients Cassandra open source et conformes au [protocole de transmission](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec) CQL Binary Protocol v4. 

En utilisant l’API Cassandra Azure Cosmos DB, vous pouvez bénéficier des avantages des API Cassandra Apache et des fonctionnalités d’entreprise que fournit Azure Cosmos DB. Parmi les fonctionnalités d’entreprise, citons la [distribution mondiale](distribute-data-globally.md), le [partitionnement automatique de scale-out](cassandra-partitioning.md), des garanties de disponibilité et de latence, le chiffrement au repos, les sauvegardes et bien plus encore.

## <a name="cassandra-protocol"></a>Protocole Cassandra 

L’API Cassandra Azure Cosmos DB est compatible avec l’API CQL (Cassandra Query Language) v3.11 (compatibilité descendante avec la version 2.x). Les commandes CQL, les outils, les limitations et les exceptions pris en charge sont listés ci-dessous. Les pilotes clients comprenant ces protocoles doivent pouvoir se connecter à l’API Cassandra Azure Cosmos DB.

## <a name="cassandra-driver"></a>Pilote Cassandra

Les versions suivantes des pilotes Cassandra sont prises en charge par l’API Cassandra Azure Cosmos DB :

* [Java 3.5+](https://github.com/datastax/java-driver)  
* [C# 3.5+](https://github.com/datastax/csharp-driver)  
* [Nodejs 3.5+](https://github.com/datastax/nodejs-driver)  
* [Python 3.15+](https://github.com/datastax/python-driver)  
* [C++ 2.9](https://github.com/datastax/cpp-driver)   
* [PHP 1.3](https://github.com/datastax/php-driver)  
* [Gocql](https://github.com/gocql/gocql)  
 

## <a name="cql-data-types"></a>Types de données CQL 

L’API Cassandra Azure Cosmos DB prend en charge les types de données CQL suivants :

|Commande  |Pris en charge. |
|---------|---------|
| ascii  | Oui |
| bigint  | Oui |
| objet BLOB  | Oui |
| boolean  | Oui |
| counter  | Oui |
| Date  | Oui |
| Décimal  | Oui |
| double  | Oui |
| float  | Oui |
| frozen  | Oui |
| inet  | Oui |
| int  | Oui |
| list  | Oui |
| set  | Oui |
| SMALLINT  | Oui |
| text  | Oui |
| time  | Oui |
| timestamp  | Oui |
| timeuuid  | Oui |
| TINYINT  | Oui |
| tuple  | Oui |
| uuid  | Oui |
| varchar  | Oui |
| varint  | Oui |
| tuples | Oui | 
| udts  | Oui |
| map | Oui |

Static est pris en charge pour la déclaration du type de données.

## <a name="cql-functions"></a>Fonctions CQL

L’API Cassandra Azure Cosmos DB prend en charge les fonctions CQL suivantes :

|Commande  |Pris en charge. |
|---------|---------|
| Jeton * | Oui |
| ttl | Oui |
| writetime | Oui |
| Caster | Non |

> [!NOTE]
> L’API Cassandra \* prend en charge le jeton en tant que projection/sélecteur et autorise uniquement token(pk) du côté gauche d’une clause Where. Par exemple, `WHERE token(pk) > 1024` est pris en charge, **contrairement à `WHERE token(pk) > token(100)`** .



Fonctions d’agrégation :

|Commande  |Pris en charge. |
|---------|---------|
| avg | Oui |
| count | Oui |
| min | Oui |
| max | Oui |
| Sum | Oui |

> [!NOTE]
> Les fonctions d’agrégation fonctionnent sur les colonnes habituelles. Toutefois, les agrégats de colonnes de clustering **ne sont pas** pris en charge.


Fonctions de conversion blob :
 
|Commande  |Pris en charge. |
|---------|---------|
| typeAsBlob(value)   | Oui |
| blobAsType(value) | Oui |


Fonctions UUID et timeuuid :
 
|Commande  |Pris en charge. |
|---------|---------|
| dateOf()  | Oui |
| now()  | Oui |
| minTimeuuid()  | Oui |
| unixTimestampOf()  | Oui |
| toDate(timeuuid)  | Oui |
| toTimestamp(timeuuid)  | Oui |
| toUnixTimestamp(timeuuid)  | Oui |
| toDate(timestamp)  | Oui |
| toUnixTimestamp(timestamp)  | Oui |
| toTimestamp(date)  | Oui |
| toUnixTimestamp(date) | Oui |


  
## <a name="cql-commands"></a>Commandes CQL

Azure Cosmos DB prend en charge les commandes de base de données suivantes sur les comptes de l’API Cassandra.

|Commande  |Pris en charge. |
|---------|---------|
| ALLOW FILTERING | Oui |
| ALTER KEYSPACE | N/A (service PaaS, réplication gérée en interne)|
| ALTER MATERIALIZED VIEW | Non |
| ALTER ROLE | Non |
| ALTER TABLE | Oui |
| ALTER TYPE | Non |
| ALTER_USER | Non |
| BATCH | Oui (lot non journalisé uniquement)|
| COMPACT STORAGE | N/A (service PaaS) |
| CREATE AGGREGATE | Non | 
| CREATE CUSTOM INDEX (SASI) | Non |
| CREATE INDEX | Oui (sans [spécifier de nom d’index](cassandra-secondary-index.md) et sans prise en charge des index sur les clés de clustering ou la collection FROZEN complète) |
| CREATE FUNCTION | Non |
| CREATE KEYSPACE (paramètres de réplication ignorés) | Oui |
| CREATE MATERIALIZED VIEW | Non |
| CREATE TABLE | Oui |
| CREATE TRIGGER | Non |
| CREATE TYPE | Oui |
| CREATE ROLE | Non |
| CREATE USER (déprécié en Apache Cassandra natif) | Non |
| Suppression | Oui |
| DELETE (transactions légères avec condition IF)| Oui |
| DISTINCT | Non |
| DROP AGGREGATE | Non |
| .DROP FUNCTION | Non |
| DROP INDEX | Oui |
| DROP KEYSPACE | Oui |
| DROP MATERIALIZED VIEW | Non |
| DROP ROLE | Non |
| DROP TABLE | Oui |
| DROP_TRIGGER | Non | 
| DROP TYPE | Oui |
| DROP USER (déprécié en Apache Cassandra natif) | Non |
| GRANT | Non |
| INSERT | Oui |
| INSERT (transactions légères avec condition IF)| Oui |
| LIST PERMISSIONS | Non |
| LIST ROLES | Non |
| LIST USERS (déprécié en Apache Cassandra natif) | Non |
| REVOKE | Non |
| SELECT | Oui |
| SELECT (transactions légères avec condition IF)| Non |
| UPDATE | Oui |
| UPDATE (transactions légères avec condition IF)| Non |
| TRUNCATE | Non |
| USE | Oui |

## <a name="json-support"></a>Prise en charge de JSON
|Commande  |Pris en charge. |
|---------|---------|
| SELECT JSON | Oui |
| INSERT JSON | Oui |
| fromJson() | Non |
| toJson() | Non |


## <a name="cassandra-api-limits"></a>Limites de l’API Cassandra

L’API Cassandra Azure Cosmos DB n’a aucune limite sur la taille des données stockées dans une table. Des centaines de téraoctets ou pétaoctets de données peuvent être stockées tout en garantissant le respect des limites de clé de partition. De même, chaque entité ou ligne équivalente n’a aucune limite de nombre de colonnes. Toutefois, la taille totale de l’entité ne doit pas dépasser 2 Mo. Les données par clé de partition ne peuvent pas dépasser 20 Go, comme dans toutes les autres API.

## <a name="tools"></a>Outils 

L’API Cassandra Azure Cosmos DB est une plateforme de services managés. Elle ne nécessite aucune surcharge de gestion ni aucun utilitaire, comme le Garbage Collector, Java Virtual Machine (JVM) et nodetool, pour gérer le cluster. Elle prend en charge des outils tels que cqlsh qui utilise la compatibilité de CQLv4 binaire. 

* L’Explorateur de données du portail Azure, les métriques, les diagnostics de journaux, PowerShell et CLI sont d’autres mécanismes pris en charge pour gérer le compte.

## <a name="hosted-cql-shell-preview"></a>Interpréteur de commandes CQL hébergé (préversion)

Vous pouvez ouvrir un interpréteur de commandes Cassandra natif hébergé (CQLSH v5.0.1) directement à partir de l’Explorateur de données dans le [portail Azure](data-explorer.md) ou de l’[Explorateur Azure Cosmos DB](https://cosmos.azure.com/). Avant l’interpréteur de commandes CQL, vous devez [activer la fonctionnalité Notebooks](enable-notebooks.md) dans votre compte (si ce n’est déjà fait, vous serez invité à cliquer sur `Open Cassandra Shell`). Consultez la note en surbrillance dans [Activer les notebooks pour un compte Azure Cosmos DB](enable-notebooks.md) pour les régions Azure prises en charge.

:::image type="content" source="./media/cassandra-support/cqlsh.png" alt-text="Ouvrir CQLSH":::

Vous pouvez également vous connecter à l’API Cassandra dans Azure Cosmos DB à l’aide de l’interpréteur CQLSH installé sur un ordinateur local. Il est fourni avec Apache Cassandra 3.1.1 et prêt à fonctionner moyennant la définition des variables d’environnement. Les sections suivantes incluent les instructions d’installation, de configuration et de connexion à l’API Cassandra dans Azure Cosmos DB sur Windows ou Linux à l’aide de CQLSH.

> [!NOTE]
> Les connexions à l’API Cassandra Azure Cosmos DB ne fonctionnent pas avec les versions DataStax Enterprise (DSE) de CQLSH. Veillez à utiliser uniquement les versions Apache Cassandra de CQLSH open source lors de la connexion à l’API Cassandra. 

**Windows :**

Si vous utilisez Windows, nous vous recommandons d’activer le [système de fichiers Windows pour Linux](/windows/wsl/install-win10#install-the-windows-subsystem-for-linux). Vous pouvez ensuite suivre les commandes Linux ci-dessous.

**Unix/Linux/Mac :**

```bash
# Install default-jre and default-jdk
sudo apt install default-jre
sudo apt-get update
sudo apt install default-jdk

# Import the Baltimore CyberTrust root certificate:
curl https://cacert.omniroot.com/bc2025.crt > bc2025.crt
keytool -importcert -alias bc2025ca -file bc2025.crt

# Install the Cassandra libraries in order to get CQLSH:
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# Export the SSL variables:
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false

# Connect to Azure Cosmos DB API for Cassandra:
cqlsh <YOUR_ACCOUNT_NAME>.cassandra.cosmosdb.azure.com 10350 -u <YOUR_ACCOUNT_NAME> -p <YOUR_ACCOUNT_PASSWORD> --ssl

```

Toutes les opérations CRUD exécutées par le biais d’un Kit de développement logiciel (SDK) compatible CQL v4 retournent des informations supplémentaires sur l’erreur et les unités de demande consommées. Les commandes DELETE et UPDATE doivent être gérées en tenant compte de la gouvernance des ressources, afin de garantir l’utilisation la plus efficace du débit approvisionné.

* Notez que la valeur gc_grace_seconds doit être égale à zéro si elle est spécifiée.

```csharp
var tableInsertStatement = table.Insert(sampleEntity); 
var insertResult = await tableInsertStatement.ExecuteAsync(); 
 
foreach (string key in insertResult.Info.IncomingPayload) 
        { 
            byte[] valueInBytes = customPayload[key]; 
            double value = Encoding.UTF8.GetString(valueInBytes); 
            Console.WriteLine($"CustomPayload:  {key}: {value}"); 
        } 
```

## <a name="consistency-mapping"></a>Mappage de cohérence 

L’API Cassandra Azure Cosmos DB offre un choix de cohérence pour les opérations de lecture.  Le mappage de cohérence est détaillé [ici](./cassandra-consistency.md#mapping-consistency-levels).

## <a name="permission-and-role-management"></a>Gestion des rôles et des autorisations

Azure Cosmos DB prend en charge le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour le provisionnement, la rotation de clés, l’affichage des métriques et les mots de passe/clés en lecture-écriture et en lecture seule, qui peuvent être obtenus via le [portail Azure](https://portal.azure.com). Azure Cosmos DB ne prend pas en charge les rôles pour les activités CRUD.

## <a name="keyspace-and-table-options"></a>Options d’espace de clé et de table

Les options pour le nom de région, la classe, replication_factor et le centre de centres dans la commande « CREATE KEYSPACE » sont actuellement ignorées. Le système utilise la méthode de réplication par [distribution globale](global-dist-under-the-hood.md) d’Azure Cosmos DB sous-jacente pour ajouter les régions. Si vous avez besoin de la présence de données inter-régions, vous pouvez l’activer au niveau du compte avec PowerShell, CLI ou le portail. Pour en savoir plus, consultez l’article expliquant [comment ajouter des régions](how-to-manage-database-account.md#addremove-regions-from-your-database-account). Durable_writes ne peut pas être désactivé, car Azure Cosmos DB garantit la durabilité de chaque écriture. Dans chaque région, Azure Cosmos DB réplique les données dans le jeu de réplicas composé de 4 réplicas, et cette [configuration](global-dist-under-the-hood.md) de jeu de réplicas ne peut pas être modifiée.
 
Toutes les options sont ignorées lors de la création de la table, sauf gc_grace_seconds, qui doit être définie sur zéro.
L’espace de clé et la table ont une option supplémentaire nommée « cosmosdb_provisioned_throughput » avec une valeur minimale de 400 RU/s. Le débit d’espace de clé permet de partager le débit entre plusieurs tables, et il est utile dans les scénarios où toutes les tables n’utilisent pas le débit provisionné. La commande ALTER TABLE permet de modifier le débit provisionné dans les régions. 

```
CREATE  KEYSPACE  sampleks WITH REPLICATION = {  'class' : 'SimpleStrategy'}   AND cosmosdb_provisioned_throughput=2000;  

CREATE TABLE sampleks.t1(user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=2000; 

ALTER TABLE gks1.t1 WITH cosmosdb_provisioned_throughput=10000 ;

```
## <a name="secondary-index"></a>Index secondaire
L’API Cassandra prend en charge les index secondaires sur tous les types de données, à l’exception des types de collections figés, et des types decimal et variant. 

## <a name="usage-of-cassandra-retry-connection-policy"></a>Utilisation de la stratégie de connexion de nouvelle tentative Cassandra

Azure Cosmos DB est un système régi par les ressources. Cela signifie que vous pouvez effectuer un certain nombre d’opérations durant une seconde donnée, en fonction des unités de requête consommées par les opérations. Si une application dépasse cette limite durant une seconde donnée, le taux des requêtes est limité et des exceptions sont levées. L’API Cassandra dans Azure Cosmos DB traduit ces exceptions en erreurs surchargées sur le protocole natif Cassandra. Pour vous assurer que votre application peut intercepter et effectuer de nouvelles tentatives de requête en cas de limitation du taux, les extensions [spark](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper) et [Java](https://github.com/Azure/azure-cosmos-cassandra-extensions) sont fournies. Consultez également les exemples de code Java pour les pilotes Datastax [version 3](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) et [version 4](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample-v4) lors de la connexion à l’API Cassandra dans Azure Cosmos DB. Si vous utilisez d’autres kits SDK pour accéder à l’API Cassandra dans Azure Cosmos DB, créez une stratégie de connexion pour effectuer une nouvelle tentative quand ces exceptions se produisent.

## <a name="next-steps"></a>Étapes suivantes

- Bien démarrer avec la [création d’un compte d’API Cassandra, d’une base de données et d’une table](create-cassandra-api-account-java.md) à l’aide d’une application Java
