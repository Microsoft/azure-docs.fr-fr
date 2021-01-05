---
title: Créer des fonctionnalités pour les données dans un cluster Azure HDInsight Hadoop - Team Data Science Process
description: Exemples de requêtes Hive qui génèrent des fonctionnalités dans les données stockées dans un cluster Azure HDInsight Hadoop.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 30c0a02c2cbc11002f8e0bf0295dab91de5d0365
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020583"
---
# <a name="create-features-for-data-in-a-hadoop-cluster-using-hive-queries"></a>Création de fonctionnalités pour les données dans un cluster Hadoop à l’aide de requêtes Hive
Ce document montre comment créer des fonctionnalités pour des données stockées dans un cluster Hadoop Azure HDInsight à l’aide de requêtes Hive. Ces requêtes Hive utilisent les FDU (fonctions définies par l’utilisateur) Hive, dont les scripts sont intégrés.

Les opérations nécessaires pour créer des fonctionnalités peuvent utiliser la mémoire de manière intensive. Les performances des requêtes Hive deviennent plus importantes dans ces cas-là et peuvent être améliorées en ajustant certains paramètres. Le réglage de ces paramètres est abordé dans la dernière section.

Des exemples de requêtes propres aux scénarios mettant en œuvre le jeu de données [NYC Taxi Trip](https://chriswhong.com/open-data/foil_nyc_taxi/) sont également disponibles dans le [référentiel Github](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts). Le schéma de données de ces requêtes est déjà spécifié et elles sont exécutables en l’état. La dernière section présente également les paramètres que les utilisateurs peuvent ajuster pour accélérer le traitement des requêtes Hive.

Cette tâche est une étape du [processus TDSP (Team Data Science Process)](./index.yml).

## <a name="prerequisites"></a>Prérequis
Cet article suppose que vous avez :

* Créé un compte de stockage Azure. Pour des instructions, voir [Créer un compte Stockage Azure](../../storage/common/storage-account-create.md).
* Approvisionné un cluster Hadoop personnalisé avec le service HDInsight.  Si vous avez besoin d'aide, consultez [Personnaliser des clusters Hadoop Azure HDInsight pour l'analyse avancée](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).
* Chargé les données dans les tables Hive de clusters Hadoop Azure HDInsight. Si tel n’est pas le cas, commencez par suivre la procédure [Créer et charger des données dans les tables Hive](move-hive-tables.md).
* Activé l’accès à distance au cluster. Si vous avez besoin d'aide, consultez [Accéder au nœud principal du cluster Hadoop](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).

## <a name="feature-generation"></a><a name="hive-featureengineering"></a>Génération de fonctionnalités
Dans cette section, plusieurs exemples de création de fonctionnalités à l'aide de requêtes Hive sont décrits. Lorsque vous avez généré des fonctionnalités supplémentaires, vous pouvez soit les ajouter sous la forme de colonnes à la table existante, soit créer une table avec les fonctionnalités supplémentaires et la clé principale, sur lesquelles vous pouvez créer une jointure à la table d’origine. Voici les exemples présentés :

1. [Génération de fonctionnalités basées sur la fréquence](#hive-frequencyfeature)
2. [Risques de variables catégorielles en classification binaire](#hive-riskfeature)
3. [Extraction de fonctionnalités à partir de champs d’horodatage](#hive-datefeatures)
4. [Extraction de fonctionnalités à partir de champs de texte](#hive-textfeatures)
5. [Calcul de la distance entre des coordonnées GPS](#hive-gpsdistance)

### <a name="frequency-based-feature-generation"></a><a name="hive-frequencyfeature"></a>Génération de fonctionnalités basées sur la fréquence
Parfois, il est intéressant de calculer les fréquences des niveaux d’une variable catégorielle ou des niveaux combinés de plusieurs variables catégorielles. Les utilisateurs peuvent utiliser les scripts suivants pour calculer ces fréquences :

```hiveql
select
    a.<column_name1>, a.<column_name2>, a.sub_count/sum(a.sub_count) over () as frequency
from
(
    select
        <column_name1>,<column_name2>, count(*) as sub_count
    from <databasename>.<tablename> group by <column_name1>, <column_name2>
)a
order by frequency desc;
```


### <a name="risks-of-categorical-variables-in-binary-classification"></a><a name="hive-riskfeature"></a>Risques de variables catégorielles en classification binaire
En classification binaire, les variables catégorielles non numériques doivent être converties en fonctionnalités numériques, lorsque les modèles utilisés n’acceptent que les fonctionnalités numériques. Pour ce faire, chaque niveau non numérique est remplacé par un risque numérique. Cette section présente certaines requêtes Hive génériques qui calculent les valeurs de risque (« log odds ») d’une variable catégorielle.

```hiveql
set smooth_param1=1;
set smooth_param2=20;
select
    <column_name1>,<column_name2>,
    ln((sum_target+${hiveconf:smooth_param1})/(record_count-sum_target+${hiveconf:smooth_param2}-${hiveconf:smooth_param1})) as risk
from
    (
    select
        <column_nam1>, <column_name2>, sum(binary_target) as sum_target, sum(1) as record_count
    from
        (
        select
            <column_name1>, <column_name2>, if(target_column>0,1,0) as binary_target
        from <databasename>.<tablename>
        )a
    group by <column_name1>, <column_name2>
    )b
```

Dans cet exemple, les variables `smooth_param1` et `smooth_param2` sont configurées pour lisser les valeurs de risque obtenues à partir des données. Les risques sont compris entre -Inf et Inf. Un risque supérieur à 0 signifie que la probabilité que la cible soit égale à 1 est supérieure à 0,5.

Une fois la table de risque calculée, les utilisateurs peuvent attribuer les valeurs de risque à une table en créant une jointure à la table de risque. La requête de jointure Hive est fournie dans la section précédente.

### <a name="extract-features-from-datetime-fields"></a><a name="hive-datefeatures"></a>Extraction de fonctionnalités à partir de champs d’horodatage
Hive est livré avec un ensemble de FDU pour traiter des champs d’horodatage. Dans Hive, le format d’horodatage par défaut est « aaaa-MM-jj 00:00:00 » (comme « 1970-01-01 12:21:32 »). Cette section montre comment extraire le jour du mois et le mois d’un champ d’horodatage, ainsi que des exemples qui convertissent une chaîne d’horodatage d’un format autre que le format par défaut en une chaîne d’horodatage au format par défaut.

```hiveql
select day(<datetime field>), month(<datetime field>)
from <databasename>.<tablename>;
```

Cette requête Hive suppose que *\<datetime field>* est au format d’horodatage par défaut.

Si un champ d’horodatage n’est pas au format par défaut, il faut d’abord le convertir en horodatage Unix puis en chaîne d’horodatage au format par défaut. Une fois l’horodatage au format par défaut, les utilisateurs peuvent appliquer les FDU d’horodatage intégrées pour extraire des fonctionnalités.

```hiveql
select from_unixtime(unix_timestamp(<datetime field>,'<pattern of the datetime field>'))
from <databasename>.<tablename>;
```

Dans cette requête, si *\<datetime field>* suit le modèle *03/26/2015 12:04:39*, *\<pattern of the datetime field>'* doit être `'MM/dd/yyyy HH:mm:ss'`. Pour le tester, les utilisateurs peuvent exécuter :

```hiveql
select from_unixtime(unix_timestamp('05/15/2015 09:32:10','MM/dd/yyyy HH:mm:ss'))
from hivesampletable limit 1;
```

Dans cette requête, la table *hivesampletable* est installée par défaut sur tous les clusters Hadoop Azure HDInsight lors de leur approvisionnement.

### <a name="extract-features-from-text-fields"></a><a name="hive-textfeatures"></a>Extraction de fonctionnalités à partir de champs de texte
Lorsque la table Hive a un champ de texte qui contient plusieurs mots séparés par un espace, la requête suivante extrait la longueur de la chaîne et le nombre de mots de celle-ci.

```hiveql
select length(<text field>) as str_len, size(split(<text field>,' ')) as word_num
from <databasename>.<tablename>;
```

### <a name="calculate-distances-between-sets-of-gps-coordinates"></a><a name="hive-gpsdistance"></a>Calcul de la distance entre des coordonnées GPS
La requête fournie dans cette section peut être directement appliquée aux données du jeu « NYC Taxi Trip ». Cette requête montre comment appliquer une fonction mathématique intégrée dans Hive pour générer des fonctionnalités.

Les champs utilisés dans cette requête sont des coordonnées GPS des emplacements de départ et d’arrivée, intitulés *pickup\_longitude*, *pickup\_latitude*, *dropoff\_longitude* *dropoff\_latitude*. Les requêtes permettant de calculer la distance directe entre les coordonnées de départ et d’arrivée sont :

```hiveql
set R=3959;
set pi=radians(180);
select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude,
    ${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
    *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
    *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
    /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
    +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
    pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance
from nyctaxi.trip
where pickup_longitude between -90 and 0
and pickup_latitude between 30 and 90
and dropoff_longitude between -90 and 0
and dropoff_latitude between 30 and 90
limit 10;
```

Les équations mathématiques calculant la distance entre deux coordonnées GPS sont disponibles sur le site <a href="http://www.movable-type.co.uk/scripts/latlong.html" target="_blank">Movable Type Scripts</a> de Peter Lapisu. Dans ce code Javascript, la fonction `toRad()` est simplement *lat_or_lon* pi/180, qui convertit les degrés en radians. Ici, *lat_or_lon* est la latitude ou la longitude. Comme Hive ne fournit pas la fonction `atan2`, mais fournit la fonction `atan`, la fonction `atan2` est implémentée par la fonction `atan` dans la requête Hive ci-dessus, selon sa définition dans <a href="https://en.wikipedia.org/wiki/Atan2" target="_blank">Wikipédia</a>.

![Créer un espace de travail](./media/create-features-hive/atan2new.png)

La liste complète des FDU Hive intégrées est disponible dans la section **Fonctions intégrées** du <a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-MathematicalFunctions" target="_blank">wiki Apache Hive</a>.  

## <a name="advanced-topics-tune-hive-parameters-to-improve-query-speed"></a><a name="tuning"></a> Rubriques avancées : Ajuster les paramètres Hive pour accélérer le traitement des requêtes
Les paramètres par défaut du cluster Hive peuvent ne pas convenir aux requêtes Hive et aux données qu’elles traitent. Cette section présente certains paramètres que les utilisateurs peuvent ajuster pour accélérer le traitement des requêtes Hive. Les requêtes d’ajustement des paramètres doivent précéder les requêtes de traitement des données.

1. **Espace de tas Java** : les requêtes impliquant la jointure de jeux de données volumineux ou le traitement d’enregistrements longs peuvent générer une erreur de type **espace du tas insuffisant**. Cette erreur peut être évitée en définissant les paramètres *mapreduce.map.java.opts* et *mapreduce.task.io.sort.mb* sur les valeurs souhaitées. Voici un exemple :
   
    ```hiveql
    set mapreduce.map.java.opts=-Xmx4096m;
    set mapreduce.task.io.sort.mb=-Xmx1024m;
    ```

    Ce paramètre alloue 4 Go de mémoire à l’espace de tas Java et accélère le tri en lui allouant davantage de mémoire. Ajuster ce paramètre est une bonne idée si des erreurs se produisent à cause d’un espace de tas insuffisant.

1. **Taille de bloc DFS** : ce paramètre définit l’unité élémentaire de donnée stockée dans le système de fichiers. Par exemple, si la taille du bloc DFS est de 128 Mo, alors les données dont la taille est inférieure ou égale à 128 Mo sont stockées dans un seul bloc. Des blocs supplémentaires sont affectés aux données dont la taille est supérieure à 128 Mo. 
2. Une taille de bloc petite génère des temps de traitement importants dans Hadoop, car le nœud de noms doit traiter beaucoup plus de requêtes pour trouver le bloc approprié au fichier. Quand vos données ont une taille supérieure ou égale à plusieurs gigaoctets, le paramètre suivant est recommandé :

    ```hiveql
    set dfs.block.size=128m;
    ```

2. **Optimisation de l’opération de jointure dans Hive** : dans le framework map/reduce, si les opérations de jointure s’exécutent en général pendant la phase de réduction, il est possible de gagner en efficacité en planifiant les jointures pendant la phase de mappage (également appelée « jointures Map »). Définissez cette option :
   
    ```hiveql
    set hive.auto.convert.join=true;
    ```

3. **Détermination du nombre de mappeurs dans Hive** : Hadoop permet à l’utilisateur de définir le nombre de réducteurs, mais pas le nombre de mappeurs. Pour contrôler ce nombre dans une certaine mesure, l’astuce consiste à choisir les variables Hadoop *mapred.min.split.size* et *mapred.max.split.size*, car la taille de chaque tâche de mappage est déterminée par :
   
    ```hiveql
    num_maps = max(mapred.min.split.size, min(mapred.max.split.size, dfs.block.size))
    ```
   
    En général, la valeur par défaut de :
    
   - *mapred.min.split.size* est 0, celle de
   - *mapred.max.split.size* est **Long.MAX** et celle de 
   - *dfs.block.size* est 64 Mo.

     Comme nous pouvons le constater, vu la taille des données, l’ajustement de ces paramètres permet d’optimiser le nombre de mappeurs utilisés.

4. Voici d’autres **options avancées** optimisant les performances de Hive. Ces options vous permettent de configurer la mémoire allouée aux tâches de mappage et de réduction, et d’ajuster au mieux les performances. Gardez à l’esprit que *mapreduce.reduce.memory.mb* ne peut pas être supérieur à la mémoire physique de chaque nœud de travail du cluster Hadoop.
   
    ```hiveql
    set mapreduce.map.memory.mb = 2048;
    set mapreduce.reduce.memory.mb=6144;
    set mapreduce.reduce.java.opts=-Xmx8192m;
    set mapred.reduce.tasks=128;
    set mapred.tasktracker.reduce.tasks.maximum=128;
    ```