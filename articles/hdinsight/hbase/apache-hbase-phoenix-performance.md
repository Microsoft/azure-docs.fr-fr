---
title: Performances de Phoenix dans Azure HDInsight
description: Bonnes pratiques pour optimiser les performances d’Apache Phoenix sur les clusters Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/27/2019
ms.openlocfilehash: 0dfb93db1af807459c37653189a90b754c933aa4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89504789"
---
# <a name="apache-phoenix-performance-best-practices"></a>Bonnes pratiques pour les performances d’Apache Phoenix

L’aspect le plus important des performances de [Apache Phoenix](https://phoenix.apache.org/) est d’optimiser l’instance [Apache HBase](https://hbase.apache.org/) sous-jacente. Phoenix crée un modèle de données relationnelles sur HBase qui convertit les requêtes SQL en opérations HBase, telles que des analyses. La conception de votre schéma de table, la sélection et le classement des champs dans votre clé primaire et votre utilisation des index affectent les performances de Phoenix.

## <a name="table-schema-design"></a>Conception de schéma de table

Lorsque vous créez une table dans Phoenix, celle-ci est stockée dans une table HBase. La table HBase contient des groupes de colonnes (familles de colonnes) qui sont accessibles ensemble. Une ligne dans la table Phoenix est une ligne dans la table HBase, où chaque ligne se compose de cellules avec version associées à une ou plusieurs colonnes. Logiquement, une seule ligne HBase est une collection de paires clé-valeur, ayant chacune la même valeur d’attribut rowkey. Autrement dit, chaque paire clé-valeur a un attribut rowkey, et la valeur de cet attribut rowkey est la même pour une ligne spécifique.

La conception de schéma d’une table Phoenix inclut la conception de la clé primaire, la conception de la famille de colonnes, la conception de la colonne individuelle et la façon dont les données sont partitionnées.

### <a name="primary-key-design"></a>Conception de clé primaire

La clé primaire définie sur une table Phoenix détermine comment les données sont stockées dans l’attribut rowkey de la table HBase sous-jacente. Dans HBase, la seule façon d’accéder à une ligne particulière est d’utiliser l’attribut rowkey. En outre, les données stockées dans une table HBase sont triées avec l’attribut rowkey. Phoenix génère la valeur de l’attribut rowkey par concaténation des valeurs de chaque colonne dans la ligne, dans l’ordre où elles sont définies dans la clé primaire.

Par exemple, une table pour les contacts possède le prénom, le nom, le numéro de téléphone et l’adresse, dans la même famille de colonnes. Vous pouvez définir une clé primaire basée sur un nombre de séquence croissant :

|rowkey|       address|   phone| firstName| lastName|
|------|--------------------|--------------|-------------|--------------|
|  1 000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole|
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji|

Toutefois, si vous interrogez fréquemment à l’aide de lastName, cette clé primaire peut ne pas s’avérer performante, étant donné que chaque requête nécessite une analyse complète de la table pour lire la valeur de chaque lastName. Au lieu de cela, vous pouvez définir une clé primaire sur les colonnes lastName, firstName et social security number. Cette dernière colonne permet de lever l’ambiguïté entre deux personnes résidant à la même adresse avec le même nom, comme un père et son fils.

|rowkey|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  1 000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

Avec cette nouvelle clé primaire, les clés de ligne générées par Phoenix seraient les suivantes :

|rowkey|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

Dans la première ligne ci-dessus, les données de l’attribut rowkey sont représentées comme indiqué :

|rowkey|       key|   value|
|------|--------------------|---|
|  Dole-John-111|address |1111 San Gabriel Dr.|  
|  Dole-John-111|phone |1-425-000-0002|  
|  Dole-John-111|firstName |John|  
|  Dole-John-111|lastName |Dole|  
|  Dole-John-111|socialSecurityNum |111|

Maintenant, cet attribut rowkey stocke une copie des données. Prenez en compte la taille et le nombre de colonnes que vous incluez dans votre clé primaire, car cette valeur est incluse dans chaque cellule de la table HBase sous-jacente.

En outre, si la clé primaire comporte des valeurs qui augmentent de manière monotone, vous devez créer la table avec des *salt buckets* pour éviter de créer des zones réactives d’écriture, voir [Partitionner les données](#partition-data).

### <a name="column-family-design"></a>Conception de famille de colonnes

Si certaines colonnes font l’objet d’un accès plus souvent que d’autres, vous devez créer plusieurs familles de colonnes pour séparer les colonnes fréquemment sollicitées de celles qui le sont rarement.

En outre, si certaines colonnes ont tendance à faire l’objet d’un accès ensemble, placez-les dans la même famille de colonnes.

### <a name="column-design"></a>Conception de colonne

* Les colonnes VARCHAR doivent avoir une taille inférieure à 1 Mo en raison des coûts d’E/S des colonnes volumineuses. Lors du traitement des requêtes, HBase matérialise les cellules entièrement avant de les transmettre au client et le client les reçoit entièrement avant de les transmettre au code de l’application.
* Stockez les valeurs de colonne à l’aide d’un format compact comme protobuf, Avro, msgpack ou BSON. JSON n’est pas recommandé, car il est plus volumineux.
* Envisagez de compresser les données avant le stockage pour réduire la latence et les coûts d’E/S.

### <a name="partition-data"></a>Données de partition

Phoenix vous permet de contrôler le nombre de régions où vos données sont distribuées, ce qui peut augmenter considérablement les performances en lecture/écriture. Lorsque vous créez une table Phoenix, vous pouvez utiliser la chaîne salt ou diviser vos données au préalable.

Pour utiliser la chaîne salt sur une table lors de la création, spécifiez le nombre de salt buckets :

```sql
CREATE TABLE CONTACTS (...) SALT_BUCKETS = 16
```

La chaîne salt fractionne la table sur les valeurs de clés primaires, en choisissant les valeurs automatiquement. 

Pour contrôler où les divisions de la table se produisent, vous pouvez préalablement fractionner la table en fournissant la plage de valeurs pour le fractionnement. Par exemple, pour créer une table fractionnée selon trois régions :

```sql
CREATE TABLE CONTACTS (...) SPLIT ON ('CS','EU','NA')
```

## <a name="index-design"></a>Conception d’index

Un index Phoenix est une table HBase qui stocke une copie de tout ou partie des données de la table indexée. Un index améliore les performances pour des types de requêtes spécifiques.

Lorsque vous avez plusieurs index définis et que vous interrogez ensuite une table, Phoenix sélectionne automatiquement le meilleur index pour la requête. L’index primaire est créé automatiquement, selon les clés primaires que vous sélectionnez.

Pour les requêtes anticipées, vous pouvez également créer des index secondaires en spécifiant leurs colonnes.

Lorsque vous concevez vos index :

* Créez uniquement les index dont vous avez besoin.
* Limitez le nombre d’index sur les tables fréquemment mises à jour. Les mises à jour d’une table se traduisent par des écritures dans la table principale et les tables d’index.

## <a name="create-secondary-indexes"></a>Créer des index secondaires

Les index secondaires peuvent améliorer les performances de lecture en transformant une analyse complète de la table en recherche de point, au détriment de l’espace de stockage et de la vitesse d’écriture. Les index secondaires peuvent être ajoutés ou supprimés après la création de la table et ne requièrent pas de modifications des requêtes existantes ; les requêtes s’exécutent plus rapidement. Selon vos besoins, envisagez de créer des index couverts, des index fonctionnels ou les deux.

### <a name="use-covered-indexes"></a>Utiliser des index couverts

Les index couverts sont des index qui incluent des données de la ligne en plus des valeurs qui sont indexées. Après avoir trouvé l’entrée d’index souhaitée, il est inutile d’accéder à la table primaire.

Dans l’exemple de table de contacts, vous pouvez créer un index secondaire uniquement sur la colonne socialSecurityNum. Cet index secondaire accélère les requêtes filtrées à l’aide des valeurs socialSecurityNum, mais la récupération des valeurs d’autres champs nécessitera une autre lecture de la table principale.

|rowkey|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

Toutefois, si vous souhaitez généralement rechercher les valeurs firstName et lastName en fonction de la valeur socialSecurityNum, vous pouvez créer un index couvert qui inclut les valeurs firstName et lastName en tant que données réelles dans la table d’index :

```sql
CREATE INDEX ssn_idx ON CONTACTS (socialSecurityNum) INCLUDE(firstName, lastName);
```

Cet index couvert permet à la requête suivante d’acquérir toutes les données simplement à l’aide d’une lecture de la table contenant l’index secondaire :

```sql
SELECT socialSecurityNum, firstName, lastName FROM CONTACTS WHERE socialSecurityNum > 100;
```

### <a name="use-functional-indexes"></a>Utiliser des index fonctionnels

Les index fonctionnels permettent de créer un index sur une expression arbitraire qui, selon vous, sera utilisée dans les requêtes. Dès que vous avez un index fonctionnel en place et qu’une requête utilise cette expression, l’index peut être utilisé pour récupérer les résultats, plutôt que la table de données.

Par exemple, vous pouvez créer un index pour vous permettent d’effectuer des recherches qui ne respectent pas la casse sur le prénom et le nom d’une personne :

```sql
CREATE INDEX FULLNAME_UPPER_IDX ON "Contacts" (UPPER("firstName"||' '||"lastName"));
```

## <a name="query-design"></a>Conception de requête

Les principales considérations pour la conception d’une requête sont les suivantes :

* Comprendre le plan de la requête et vérifier son comportement attendu.
* Joindre efficacement.

### <a name="understand-the-query-plan"></a>Comprendre le plan de la requête

Dans [SQLLine](http://sqlline.sourceforge.net/), utilisez EXPLAIN suivi de votre requête SQL pour afficher le plan des opérations qui seront effectuées par Phoenix. Vérifiez que le plan :

* Utilise votre clé primaire lorsque cela est approprié.
* Utilise les index secondaires appropriés, plutôt que la table de données.
* Utilise RANGE SCAN ou SKIP SCAN lorsque cela est possible, plutôt que TABLE SCAN.

#### <a name="plan-examples"></a>Exemples de plans

Par exemple, supposons que vous avez une table appelée FLIGHTS qui stocke les informations de retard de vol.

Pour sélectionner tous les vols avec un airlineid `19805`, où airlineid est un champ qui n’est pas dans la clé primaire, ni dans un index :

```sql
select * from "FLIGHTS" where airlineid = '19805';
```

Exécutez la commande explain comme suit :

```sql
explain select * from "FLIGHTS" where airlineid = '19805';
```

Le plan de la requête ressemble à ceci :

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER FLIGHTS
   SERVER FILTER BY AIRLINEID = '19805'
```

Dans ce plan, notez la phrase FULL SCAN OVER FLIGHTS. Cette phrase indique l’exécution de TABLE SCAN sur toutes les lignes de la table, plutôt que d’utiliser l’option RANGE SCAN ou SKIP SCAN plus efficace.

Maintenant, supposons que vous souhaitiez interroger la table concernant les vols du 2 janvier 2014 pour le transporteur `AA` avec un flightnum supérieur à 1. Supposons que les colonnes year, month, dayofmonth, carrier et flightnum existent dans l’exemple de table, et qu’elles font toutes partie de la clé primaire composite. La requête devrait se présenter comme suit :

```sql
select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;
```

Examinons le plan de cette requête avec :

```sql
explain select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;
```

Le plan obtenu est le suivant :

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER FLIGHTS [2014,1,2,'AA',2] - [2014,1,2,'AA',*]
```

Les valeurs dans les crochets correspondent à la plage de valeurs pour les clés primaires. Dans ce cas, les valeurs de la plage sont fixées sur year 2014, month 1 et dayofmonth 2, mais autorisent les valeurs pour flightnum à partir de 2 et valeurs supérieures (`*`). Ce plan de requête confirme que la clé primaire est utilisée comme prévu.

Ensuite, créez un index sur la table FLIGHTS nommé `carrier2_idx` sur le champ carrier uniquement. Cet index inclut également flightdate, tailnum, origin et flightnum en tant que colonnes couvertes dont les données sont aussi stockées dans l’index.

```sql
CREATE INDEX carrier2_idx ON FLIGHTS (carrier) INCLUDE(FLIGHTDATE,TAILNUM,ORIGIN,FLIGHTNUM);
```

Supposons que vous souhaitiez obtenir le transporteur (carrier), ainsi que les valeurs flightdate et tailnum, comme dans la requête suivante :

```sql
explain select carrier,flightdate,tailnum from "FLIGHTS" where carrier = 'AA';
```

L’index suivant devrait être utilisé :

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER CARRIER2_IDX ['AA']
```

Pour une liste complète des éléments qui peuvent apparaître dans les résultats du plan explain, consultez la section Plans Explain dans le [Guide d’optimisation de Phoenix Apache](https://phoenix.apache.org/tuning_guide.html).

### <a name="join-efficiently"></a>Joindre efficacement

En règle générale, les utilisateurs préfèrent éviter les jointures, sauf si un côté est de petite taille, en particulier sur les requêtes fréquentes.

Si nécessaire, vous pouvez effectuer des jointures volumineuses avec l’indicateur `/*+ USE_SORT_MERGE_JOIN */`, mais une jointure volumineuse est une opération coûteuse sur un grand nombre de lignes. Si la taille globale de toutes les tables du droit côté surpasse la mémoire disponible, utilisez l’indicateur `/*+ NO_STAR_JOIN */`.

## <a name="scenarios"></a>Scénarios

Les instructions suivantes décrivent certains modèles courants.

### <a name="read-heavy-workloads"></a>Charges de travail nécessitant beaucoup de lectures

Pour les cas d’usage nécessitant beaucoup de lectures, assurez-vous d’utiliser des index. En outre, pour éviter une surcharge de temps de lecture, envisagez de créer des index couverts.

### <a name="write-heavy-workloads"></a>Charges de travail nécessitant beaucoup d’écritures

Pour les charges de travail nécessitant beaucoup d’écritures où la clé primaire augmente de manière monotone, créez des salt buckets afin d’éviter les zones réactives d’écriture, au détriment du débit de lecture global en raison des analyses supplémentaires nécessaires. En outre, lorsque vous utilisez UPSERT pour écrire un grand nombre d’enregistrements, désactivez autoCommit et regroupez les enregistrements par lots.

### <a name="bulk-deletes"></a>Suppressions en bloc

Lorsque vous supprimez un jeu de données volumineux, activez autoCommit avant d’émettre la requête DELETE, afin que le client n’ait pas besoin de se souvenir des clés de ligne pour toutes les lignes supprimées. AutoCommit empêche le client de mettre en mémoire tampon les lignes affectées par l’opération DELETE, afin que Phoenix puisse les supprimer directement sur les serveurs de la région sans avoir à les retourner au client.

### <a name="immutable-and-append-only"></a>Immutable (Immuable) et Append-only (Ajouter uniquement)

Si votre scénario priorise la vitesse d’écriture par rapport à l’intégrité des données, pensez à désactiver le journal à écriture anticipée lors de la création de vos tables :

```sql
CREATE TABLE CONTACTS (...) DISABLE_WAL=true;
```

Pour en savoir plus sur le sujet et sur d’autres options, consultez [Apache Phoenix Grammar](https://phoenix.apache.org/language/index.html#options).

## <a name="next-steps"></a>Étapes suivantes

* [Guide d’optimisation d’Apache Phoenix](https://phoenix.apache.org/tuning_guide.html)
* [Index secondaires](https://phoenix.apache.org/secondary_indexing.html)
