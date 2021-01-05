---
title: Réplication logique et décodage logique - Serveur flexible Azure Database pour PostgreSQL
description: Découvrez comment utiliser la réplication logique et le décodage logique dans le serveur flexible Azure Database pour PostgreSQL
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: b6689220873aaeb65337ba480e346e5d2c8020ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91707861"
---
# <a name="logical-replication-and-logical-decoding-in-azure-database-for-postgresql---flexible-server"></a>Réplication logique et décodage logique dans le serveur flexible Azure Database pour PostgreSQL

> [!IMPORTANT]
> Le serveur flexible Azure Database pour PostgreSQL est en préversion

La réplication logique et le décodage logique de PostgreSQL sont pris en charge dans le serveur flexible Azure Database pour PostgreSQL version 11.

## <a name="comparing-logical-replication-and-logical-decoding"></a>Comparaison de la réplication logique et du décodage logique
La réplication logique et le décodage logique ont plusieurs similitudes. Les deux
* vous permettent de répliquer des données à partir de Postgres ;
* utilisent le [journal WAL (write-ahead log)](https://www.postgresql.org/docs/current/wal.html) comme source des modifications ;
* utilisent des [emplacements de réplication logique](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html#LOGICALDECODING-REPLICATION-SLOTS) pour envoyer des données. Un emplacement représente un flux de modifications ;
* utilisent la propriété [IDENTITÉ DE RÉPLICA](https://www.postgresql.org/docs/current/sql-altertable.html#SQL-CREATETABLE-REPLICA-IDENTITY) d’une table pour déterminer les modifications qui peuvent être envoyées ;
* ne répliquent pas les modifications de DDL.


Les deux technologies présentent les différences suivantes : Réplication logique 
* vous permet de spécifier une table ou un ensemble de tables à répliquer ;
* réplique des données entre des instances PostgreSQL.

Décodage logique 
* extrait des modifications dans toutes les tables d’une base de données ; 
* ne peut pas envoyer directement des données entre des instances PostgreSQL.


## <a name="pre-requisites-for-logical-replication-and-logical-decoding"></a>Conditions préalables pour la réplication logique et le décodage logique

1. Définissez le paramètre du serveur `wal_level` sur `logical`.
2. Redémarrez le serveur pour appliquer les modifications `wal_level`.
3. Vérifiez que votre instance PostgreSQL autorise le trafic réseau à partir de votre ressource de connexion.
4. Accordez les autorisations de réplication de l’utilisateur administrateur.
   ```SQL
   ALTER ROLE <adminname> WITH REPLICATION;
   ```


## <a name="using-logical-replication-and-logical-decoding"></a>Utilisation de la réplication logique et du décodage logique

### <a name="logical-replication"></a>Réplication logique
La réplication logique utilise les termes « éditeur » et « abonné ». 
* L’éditeur est la base de données PostgreSQL **à partir de laquelle** vous envoyez des données. 
* L’abonné est la base de données PostgreSQL **vers laquelle** vous envoyez des données.

Voici quelques exemples de code que vous pouvez utiliser pour tester la réplication logique.

1. Connectez-vous à la base de données de publication. Créez une table et ajoutez des données.
   ```SQL
   CREATE TABLE basic(id SERIAL, name varchar(40));
   INSERT INTO basic(name) VALUES ('apple');
   INSERT INTO basic(name) VALUES ('banana');
   ```

2. Créez une publication pour la table.
   ```SQL
   CREATE PUBLICATION pub FOR TABLE basic;
   ```

3. Connectez-vous à la base de données de l’abonné. Créez une table avec le même schéma que sur l’éditeur.
   ```SQL
   CREATE TABLE basic(id SERIAL, name varchar(40));
   ```

4. Créez un abonnement qui se connectera à la publication que vous avez créée plut tôt.
   ```SQL
   CREATE SUBSCRIPTION sub CONNECTION 'host=<server>.postgres.database.azure.com user=<admin> dbname=<dbname> password=<password>' PUBLICATION pub;
   ```

5. Vous pouvez maintenant interroger la table sur l’abonné. Vous verrez qu’il a reçu des données de l’éditeur.
   ```SQL
   SELECT * FROM basic;
   ```

Vous pouvez ajouter des lignes à la table de l’éditeur et afficher les modifications sur l’abonné.

Consultez la documentation PostgreSQL pour en savoir plus sur la [réplication logique](https://www.postgresql.org/docs/current/logical-replication.html).

### <a name="logical-decoding"></a>Décodage logique
Le décodage logique peut être utilisé via le protocole de diffusion en continu ou une interface SQL. 

#### <a name="streaming-protocol"></a>Protocole de diffusion en continu
Il est souvent préférable de consommer les modifications à l’aide du protocole de diffusion en continu. Vous pouvez créer votre propre connecteur ou contrôle serveur consommateur, ou utiliser un service tiers comme [Debezium](https://debezium.io/). 

Consultez la documentation wal2json pour obtenir [un exemple utilisant le protocole de diffusion en continu avec pg_recvlogical](https://github.com/eulerto/wal2json#pg_recvlogical).

#### <a name="sql-interface"></a>Interface SQL 
Dans l’exemple ci-dessous, nous utilisons l’interface SQL avec le plug-in wal2json.
 
1. Créez un emplacement.
   ```SQL
   SELECT * FROM pg_create_logical_replication_slot('test_slot', 'wal2json');
   ```
 
2. Émettez des commandes SQL. Par exemple :
   ```SQL
   CREATE TABLE a_table (
      id varchar(40) NOT NULL,
      item varchar(40),
      PRIMARY KEY (id)
   );
   
   INSERT INTO a_table (id, item) VALUES ('id1', 'item1');
   DELETE FROM a_table WHERE id='id1';
   ```

3. Consommez les modifications.
   ```SQL
   SELECT data FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'pretty-print', '1');
   ```

   La sortie se présente ainsi :
   ```
   {
         "change": [
         ]
   }
   {
         "change": [
                  {
                           "kind": "insert",
                           "schema": "public",
                           "table": "a_table",
                           "columnnames": ["id", "item"],
                           "columntypes": ["character varying(40)", "character varying(40)"],
                           "columnvalues": ["id1", "item1"]
                  }
         ]
   }
   {
         "change": [
                  {
                           "kind": "delete",
                           "schema": "public",
                           "table": "a_table",
                           "oldkeys": {
                                 "keynames": ["id"],
                                 "keytypes": ["character varying(40)"],
                                 "keyvalues": ["id1"]
                           }
                  }
         ]
   }
   ```

4. Supprimez l’emplacement une fois que vous avez fini de l’utiliser.
   ```SQL
   SELECT pg_drop_replication_slot('test_slot'); 
   ```

Consultez la documentation PostgreSQL pour en savoir plus sur le [décodage logique](https://www.postgresql.org/docs/current/logicaldecoding.html).


## <a name="monitoring"></a>Surveillance
Vous devez surveiller le décodage logique. Tout emplacement de réplication non utilisé doit être supprimé. Les emplacements sont bloqués dans les journaux WAL Postgres et les catalogues système appropriés jusqu’à ce que les modifications aient été lues. Si votre abonné ou contrôle serveur consommateur échoue ou n’a pas été correctement configuré, les journaux non consommés s’accumulent et remplissent votre stockage. En outre, les journaux non consommés augmentent le risque d’un bouclage des ID de transaction. Les deux situations peuvent entraîner l’indisponibilité du serveur. Par conséquent, il est essentiel que les emplacements de réplication logique soient consommés en permanence. Si un emplacement de réplication logique n’est plus utilisé, supprimez-le immédiatement.

La colonne « active » de l’affichage pg_replication_slots indique si un contrôle serveur consommateur est connecté à un emplacement.
```SQL
SELECT * FROM pg_replication_slots;
```

[Définissez les alertes](howto-alert-on-metrics.md) sur le **nombre maximal d’ID de transactions utilisés** et le **stockage utilisé** des métriques de serveur flexible pour vous avertir lorsque les valeurs augmentent au-delà des seuils normaux. 

## <a name="limitations"></a>Limites
* **Réplicas en lecture** - Les réplicas en lecture d’Azure Database pour PostgreSQL ne sont pas pris en charge pour les serveurs flexibles.
* **Emplacements et basculement à haute disponibilité** -Les emplacements de réplication logique sur le serveur principal ne sont pas disponibles sur le serveur de secours de votre serveur secondaire AZ. Cela s’applique si votre serveur utilise l’option haute disponibilité redondante interzone. En cas de basculement vers le serveur de secours, les emplacements de réplication logique ne sont pas disponibles sur le serveur de secours.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [options de mise en réseau](concepts-networking.md)
* En savoir plus sur les [extensions](concepts-extensions.md) disponibles dans le serveur flexible
* En savoir plus sur la [haute disponibilité](concepts-high-availability.md)

