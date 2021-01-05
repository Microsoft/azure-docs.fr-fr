---
title: Présentation d’Apache Hive et HiveQL - Azure HDInsight
description: Apache Hive est un système d’entrepôt de données pour Apache Hadoop. Vous pouvez interroger les données stockées dans Hive à l’aide de HiveQL, qui est similaire à Transact-SQL. Dans ce document, découvrez comment utiliser Hive et HiveQL avec Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 02/28/2020
ms.openlocfilehash: d2e59b35a30bd838eab2b05dcacf83d8b2c21236
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92540395"
---
# <a name="what-is-apache-hive-and-hiveql-on-azure-hdinsight"></a>Présentation d’Apache Hive et HiveQL sur Azure HDInsight

[Apache Hive](https://hive.apache.org/) est un système d’entrepôt de données pour Apache Hadoop. Hive permet la synthèse, l’interrogation et l’analyse des données. Les requêtes Hive sont écrites dans le langage HiveQL, qui est un langage de requête semblable à SQL.

Hive vous permet de concevoir une structure sur des données largement non structurées. Une fois que vous avez défini la structure, vous pouvez utiliser HiveQL pour interroger les données sans connaître Java ou MapReduce.

HDInsight fournit plusieurs types de cluster adaptés à des charges de travail spécifiques. Voici les types de cluster les plus souvent utilisés pour les requêtes Hive :

|Type de cluster |Description|
|---|---|
|Interactive Query|Cluster Hadoop qui offre une fonctionnalité [LLAP (Low Latency Analytical Processing)](https://cwiki.apache.org/confluence/display/Hive/LLAP) afin d’améliorer les temps de réponse des requêtes interactives. Pour plus d’informations, consultez le document [Démarrer avec Interactive Query dans HDInsight](../interactive-query/apache-interactive-query-get-started.md).|
|Hadoop|Cluster Hadoop adapté aux charges de travail de traitement par lots. Pour plus d’informations, consultez le document [Démarrer avec Apache Hadoop dans HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md).|
|Spark|Apache Spark intègre une fonctionnalité permettant d’utiliser Hive. Pour plus d’informations, consultez le document [Démarrer avec Apache Spark dans HDInsight](../spark/apache-spark-jupyter-spark-sql.md).|
|hbase|HiveQL peut être utilisé pour interroger les données stockées dans Apache HBase. Pour plus d’informations, consultez le document [Démarrer avec Apache HBase dans HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md).|

## <a name="how-to-use-hive"></a>Utilisation de Hive

Consultez le tableau suivant pour découvrir les différentes façon d’utiliser Hive avec HDInsight :

| **Utilisez cette méthode** si vous souhaitez... | ... des requêtes **interactives** | ... un traitement par **lots** | ...depuis ce **système d’exploitation cluster** |
|:--- |:---:|:---:|:--- |:--- |
| [HDInsight Tools pour Visual Studio Code](../hdinsight-for-vscode.md) |✔ |✔ | Linux, Unix, Mac OS X ou Windows |
| [Outils HDInsight pour Visual Studio](../hadoop/apache-hadoop-use-hive-visual-studio.md) |✔ |✔ |Windows |
| [Affichage Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md) |✔ |✔ |N’importe lequel (basé sur le navigateur) |
| [Client Beeline](../hadoop/apache-hadoop-use-hive-beeline.md) |✔ |✔ |Linux, Unix, Mac OS X ou Windows |
| [REST API](../hadoop/apache-hadoop-use-hive-curl.md) |&nbsp; |✔ |Linux, Unix, Mac OS X ou Windows |
| [Windows PowerShell](../hadoop/apache-hadoop-use-hive-powershell.md) |&nbsp; |✔ |Windows |

## <a name="hiveql-language-reference"></a>Référence du langage HiveQL

Les informations de référence sur le langage HiveQL sont disponibles dans le [manuel du langage](https://cwiki.apache.org/confluence/display/Hive/LanguageManual).

## <a name="hive-and-data-structure"></a>Hive et structure des données

Hive est capable de travailler avec des données structurées et semi-structurées. Par exemple, les fichiers texte dans lesquels les champs sont délimités par des caractères spécifiques. L’instruction HiveQL suivante crée une table à partir de données délimitées par des espaces :

```hiveql
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
```

Hive prend également en charge un **sérialiseur/ désérialiseur (SerDe)** personnalisé pour des données structurées de manière irrégulière ou complexes. Pour plus d’informations, consultez le document [Utilisation d’un SerDe JSON personnalisé avec HDInsight](https://web.archive.org/web/20190217104719/https://blogs.msdn.microsoft.com/bigdatasupport/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight/).

Pour plus d’informations sur les formats de fichier pris en charge par Hive, consultez le [manuel de langage(https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

### <a name="hive-internal-tables-vs-external-tables"></a>Tables internes et externes Hive

Hive vous permet de créer deux types de tables :

* __Interne__  : Les données sont stockées dans l’entrepôt de données Hive. L’entrepôt de données se trouve dans `/hive/warehouse/` sur le stockage par défaut du cluster.

    Utilisez des tables internes lorsque l’une des conditions suivantes s’applique :

    * les données sont temporaires.
    * Vous voulez que Hive gère le cycle de vie de la table et des données.

* __Externe__  : Les données sont stockées en dehors de l’entrepôt de données. Les données peuvent être stockées sur tout stockage accessible par le cluster.

    Utilisez des tables externes lorsque l’une des conditions suivantes s’applique :

    * Les données sont également utilisées en dehors de Hive. Par exemple, les fichiers de données sont mis à jour par un autre processus (qui ne verrouille pas les fichiers).
    * Les données doivent rester dans l’emplacement sous-jacent, même après suppression de la table.
    * Vous avez besoin d’un emplacement personnalisé, par exemple un compte de stockage non sélectionné par défaut.
    * Un programme autre que Hive gère le format de données, l’emplacement, etc.

Pour plus d’informations, consultez le billet de blog [Introduction aux tables interne et externe Hive](/archive/blogs/cindygross/hdinsight-hive-internal-and-external-tables-intro).

## <a name="user-defined-functions-udf"></a>Fonctions définies par l’utilisateur (UDF)

Hive peut également être étendu via des **fonctions définies par l'utilisateur (UDF)** . Une fonction UDF vous permet d'implémenter une fonctionnalité ou une logique qui n'est pas facilement modelée en HiveQL. Pour obtenir un exemple d’utilisation des fonctions définies par l’utilisateur (UDF) avec Hive, consultez les documents suivants :

* [Utiliser une fonction Java définie par l’utilisateur avec Apache Hive](../hadoop/apache-hadoop-hive-java-udf.md)

* [Utiliser une fonction Python définie par l’utilisateur avec Apache Hive](../hadoop/python-udf-hdinsight.md)

* [Utiliser une fonction C# définie par l’utilisateur avec Apache Hive](../hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [Guide pratique pour ajouter à HDInsight une fonction Apache Hive définie par l’utilisateur](/archive/blogs/bigdatasupport/how-to-add-custom-hive-udfs-to-hdinsight)

* [Exemple de fonction Apache Hive définie par l’utilisateur pour convertir les formats date/heure en horodatage Hive](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## <a name="example-data"></a>Exemple de données

Hive sur HDInsight est préchargé avec une table interne nommée `hivesampletable`. HDInsight fournit également des exemples de jeux de données pouvant être utilisés avec Hive. Ces jeux de données sont stockés dans les répertoires `/example/data` et `/HdiSamples`. Ces répertoires sont disponibles dans le stockage par défaut de votre cluster.

## <a name="example-hive-query"></a>Exemple de requête Hive

Les instructions HiveQL suivantes projettent des colonnes sur le fichier `/example/data/sample.log` :

```hiveql
DROP TABLE log4jLogs;
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs
    WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log'
    GROUP BY t4;
```

Dans l’exemple précédent, les instructions HiveQL effectuent les opérations suivantes :

|. |Description |
|---|---|
|DROP TABLE|Si la table existe déjà, supprimez-la.|
|CREATE EXTERNAL TABLE|Crée une table **externe** dans Hive. Les tables externes stockent uniquement la définition de table dans Hive. Les données restent à l’emplacement d’origine, dans le format d’origine.|
|ROW FORMAT|indique à Hive la façon dont les données sont mises en forme. Dans ce cas, les champs de chaque journal sont séparés par un espace.|
|STORED AS TEXTFILE LOCATION|Indique à Hive où sont stockées les données (répertoire `example/data`) et qu’elles sont stockées sous forme de texte. Les données peuvent être dans un seul fichier ou réparties sur plusieurs fichiers dans le répertoire.|
|SELECT|Sélectionne toutes les lignes où la colonne **t4** contient la valeur **[ERROR]** . Cette instruction renvoie la valeur **3** , car trois lignes contiennent cette valeur.|
|INPUT__FILE__NAME LIKE '%.log'|Hive tente d’appliquer le schéma à tous les fichiers dans le répertoire. Dans ce cas, le répertoire contient des fichiers qui ne correspondent pas au schéma. Pour éviter que des données incorrectes n’apparaissent dans les résultats, cette instruction indique à Hive de retourner uniquement des données provenant de fichiers se terminant par .log.|

> [!NOTE]  
> Les tables externes doivent être utilisées lorsque vous vous attendez à ce que les données sous-jacentes soient mises à jour par une source externe, Par exemple, un processus de chargement de données automatisé ou une opération MapReduce.
>
> La suppression d'une table externe ne supprime **pas** les données, mais seulement la définition de la table.

Pour créer une table **interne** plutôt qu’externe, utilisez le code HiveQL suivant :

```hiveql
CREATE TABLE IF NOT EXISTS errorLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
STORED AS ORC;
INSERT OVERWRITE TABLE errorLogs
SELECT t1, t2, t3, t4, t5, t6, t7 
    FROM log4jLogs WHERE t4 = '[ERROR]';
```

Ces instructions effectuent les opérations suivantes :

|. |Description |
|---|---|
|CREATE TABLE IF NOT EXISTS|Si la table n’existe pas, créez-la. Étant donné que le mot clé **EXTERNAL** n’est pas utilisé, cette instruction crée une table interne. La table est stockée dans l’entrepôt de données Hive et gérée intégralement par Hive.|
|STORED AS ORC|Stocke les données dans un format ORC (Optimized Row Columnar). ORC est un format particulièrement efficace et optimisé pour le stockage de données Hive.|
|INSERT OVERWRITE ... SELECT|Sélectionne des lignes de la table **log4jLogs** qui contient **[ERROR]** , puis insère les données dans la table **errorLogs** .|

> [!NOTE]  
> Contrairement aux tables externes, la suppression d’une table interne entraîne également la suppression des données sous-jacentes.

## <a name="improve-hive-query-performance"></a>Améliorer les performances des requêtes Hive

### <a name="apache-tez"></a>Apache Tez

[Apache Tez](https://tez.apache.org) est une infrastructure qui permet une exécution à l'échelle beaucoup plus efficace pour les applications, telles que Hive, qui manipulent de grandes quantités de données. Tez est activé par défaut.  Les [documents de conception Apache Hive sur Tez](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez) contiennent des informations détaillées sur les options d’implémentation et les configurations du réglage.

### <a name="low-latency-analytical-processing-llap"></a>Low Latency Analytical Processing (LLAP)

[LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP) (parfois appelé Live Long and Process) est une nouvelle fonctionnalité de Hive 2.0 qui permet la mise en cache en mémoire des requêtes. LLAP accélère considérablement les requêtes Hive, avec dans certains cas des vitesses jusqu’à [26 fois plus rapides qu’avec Hive 1.x](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/).

HDInsight fournit la fonctionnalité LLAP dans le cluster de type Interactive Query. Pour plus d’informations, consultez le document [Démarrer avec Interactive Query](../interactive-query/apache-interactive-query-get-started.md).

## <a name="scheduling-hive-queries"></a>Planification des requêtes Hive

Plusieurs services peuvent être utilisés pour exécuter des requêtes Hive dans le cadre d’un workflow à la demande ou planifié.

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory permet d’utiliser HDInsight dans le cadre d’un pipeline Data Factory. Pour plus d’informations sur l’utilisation de Hive à partir d’un pipeline, consultez le document [Transformer des données à l’aide d’une activité Hive dans Azure Data Factory](../../data-factory/transform-data-using-hadoop-hive.md).

### <a name="hive-jobs-and-sql-server-integration-services"></a>Tâches Hive et SQL Server Integration Services

Vous pouvez utiliser les services SQL Server Integration Services (SSIS) pour exécuter un travail Hive. Le pack de fonctionnalités Azure pour SSIS fournit les composants suivants, compatibles avec les tâches Hive sur HDInsight.

* [Tâche Hive d’Azure HDInsight](/sql/integration-services/control-flow/azure-hdinsight-hive-task)

* [Gestionnaire de connexions d’abonnement Azure](/sql/integration-services/connection-manager/azure-subscription-connection-manager)

Pour plus d’informations, consultez la documentation relative au [Feature Pack Azure](/sql/integration-services/azure-feature-pack-for-integration-services-ssis).

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie est un workflow et un système de coordination qui gère les tâches Hadoop. Pour plus d’informations sur l’utilisation d’Oozie avec Hive, consultez le document [Utiliser Apache Oozie pour définir et exécuter un workflow](../hdinsight-use-oozie-linux-mac.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez Hive et que vous avez vu comment l’utiliser avec Hadoop dans HDInsight, utilisez les liens suivants pour découvrir d'autres façons d'utiliser Azure HDInsight.

* [Téléchargement de données vers HDInsight](../hdinsight-upload-data.md)
* [Utiliser des fonctions définies par l’utilisateur (UDF) Python avec Apache Hive et Apache Pig dans HDInsight](./python-udf-hdinsight.md)
* [Utilisation des tâches MapReduce avec HDInsight](hdinsight-use-mapreduce.md)