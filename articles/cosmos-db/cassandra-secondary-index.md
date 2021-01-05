---
title: Indexation dans un compte d’API Cassandra Azure Cosmos DB
description: Découvrez comment fonctionne l’indexation secondaire dans un compte d’API Cassandra Azure Cosmos DB.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: conceptual
ms.date: 04/04/2020
ms.author: thvankra
ms.reviewer: sngun
ms.openlocfilehash: 98e8f713ad2e4eef47e40d89a23dbf49a98ad67c
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339888"
---
# <a name="secondary-indexing-in-azure-cosmos-db-cassandra-api"></a>Indexation secondaire dans l’API Cassandra Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

L’API Cassandra dans Azure Cosmos DB tire parti de l’infrastructure d’indexation sous-jacente pour exposer la force d’indexation inhérente à la plateforme. Toutefois, contrairement à l’API SQL principale, l’API Cassandra dans Azure Cosmos DB n’indexe pas tous les attributs par défaut. Au lieu de cela, elle prend en charge l’indexation secondaire pour créer un index sur certains attributs, qui se comporte de la même façon qu’Apache Cassandra.  

En général, il n’est pas recommandé d’exécuter des requêtes de filtre sur les colonnes qui ne sont pas partitionnées. Vous devez utiliser la syntaxe ALLOW FILTERING de manière explicite, ce qui aboutit à une opération qui peut ne pas s’exécuter correctement. Dans Azure Cosmos DB vous pouvez exécuter de telles requêtes sur des attributs de faible cardinalité, car elles effectuent une distribution ramifiée dans des partitions pour récupérer les résultats.

Il n’est pas recommandé de créer un index sur une colonne fréquemment mise à jour. Il est prudent de créer un index lorsque vous définissez la table. Cela garantit que les données et les index sont dans un état cohérent. Si vous créez un nouvel index sur les données existantes, vous ne pouvez actuellement pas suivre la modification de la progression de l’index pour la table. Si vous avez besoin de suivre la progression de cette opération, vous devez demander la modification de la progression via un [ticket de support]( https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).


> [!NOTE]
> L’index secondaire n’est pas pris en charge sur les objets suivants :
> - types de données tels que des types de collections gelées, de décimaux et de variantes.
> - Colonnes statiques
> - Clés de clustering

## <a name="indexing-example"></a>Exemple d’indexation

Commencez par créer un exemple d’espace de clés et une table en exécutant les commandes suivantes à l’invite de l’interpréteur de commandes CQL :

```shell
CREATE KEYSPACE sampleks WITH REPLICATION = {'class' : 'SimpleStrategy'};
CREATE TABLE sampleks.t1(user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=400;
``` 

Ensuite, insérez les exemples de données utilisateur avec les commandes suivantes :

```shell
insert into sampleks.t1(user_id,lastname) values (1, 'nishu');
insert into sampleks.t1(user_id,lastname) values (2, 'vinod');
insert into sampleks.t1(user_id,lastname) values (3, 'bat');
insert into sampleks.t1(user_id,lastname) values (5, 'vivek');
insert into sampleks.t1(user_id,lastname) values (6, 'siddhesh');
insert into sampleks.t1(user_id,lastname) values (7, 'akash');
insert into sampleks.t1(user_id,lastname) values (8, 'Theo');
insert into sampleks.t1(user_id,lastname) values (9, 'jagan');
```

Si vous essayez d’exécuter l’instruction suivante, vous allez rencontrer une erreur vous demandant d’utiliser `ALLOW FILTERING` : 

```shell
select user_id, lastname from sampleks.t1 where lastname='nishu';
``` 

Même si l’API Cassandra prend en charge la syntaxe ALLOW FILTERING, comme mentionné dans la section précédente, elle n’est pas recommandée. Au lieu de cela, vous devez créer un index comme indiqué dans l’exemple suivant :

```shell
CREATE INDEX ON sampleks.t1 (lastname);
```
Après avoir créé un index sur le champ « lastname », vous pouvez exécuter la requête précédente avec succès. Avec l’API Cassandra dans Azure Cosmos DB, vous n’avez pas à fournir de nom d’index. Un index par défaut avec le format `tablename_columnname_idx` est utilisé. Par exemple, ` t1_lastname_idx` est le nom d’index pour le tableau précédent.

## <a name="dropping-the-index"></a>Suppression de l’index 
Pour supprimer l’index, vous devez connaître son nom. Exécutez la commande `desc schema` pour afficher la description de votre table. La sortie de cette commande comprend le nom de l’index au format `CREATE INDEX tablename_columnname_idx ON keyspacename.tablename(columnname)`. Vous pouvez ensuite utiliser le nom de l’index pour supprimer celui-ci, comme dans l’exemple suivant :

```shell
drop index sampleks.t1_lastname_idx;
```

## <a name="next-steps"></a>Étapes suivantes
* Découvrir le fonctionnement de l’[indexation automatique](index-overview.md) dans Azure Cosmos DB
* [Fonctionnalités Apache Cassandra prises en charge par l’API Cassandra Azure Cosmos DB](cassandra-support.md)
