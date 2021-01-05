---
title: Utiliser Apache Beeline avec Apache Hive - Azure HDInsight
description: Découvrez comment utiliser le client Beeline pour exécuter des requêtes Hive avec Hadoop sur HDInsight. Beeline est un utilitaire qui permet d’utiliser HiveServer2 sur JDBC.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 10/28/2020
ms.custom: contperf-fy21q1, contperf-fy21q2
ms.openlocfilehash: 70fbbdfc5d8f1bac5fa27175ea25be1503a77594
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031893"
---
# <a name="use-the-apache-beeline-client-with-apache-hive"></a>Utiliser le client Apache Beeline avec Apache Hive

Cet article explique comment utiliser le client [Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) de ligne de commande pour créer et exécuter des requêtes Apache Hive sur une connexion SSH.

## <a name="background"></a>Arrière-plan

Beeline est un client Hive inclus dans les nœuds principaux de votre cluster HDInsight. Pour vous connecter au client Beeline installé sur votre cluster HDInsight ou pour installer Beeline localement, consultez [Installer Apache Beeline ou s’y connecter](connect-install-beeline.md). Beeline utilise JDBC pour se connecter à HiveServer2, un service hébergé sur votre cluster HDInsight. Vous pouvez également utiliser Beeline pour accéder à Hive sur HDInsight à distance via internet. Les exemples suivants présentent les chaînes de connexion les plus courantes utilisées pour se connecter à HDInsight à partir de Beeline.

## <a name="prerequisites-for-examples"></a>Configuration requise pour les exemples

* Un cluster Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Notez le schéma d’URI du stockage principal de votre cluster. Par exemple, `wasb://` pour Stockage Azure, `abfs://` pour Azure Data Lake Storage Gen2 ou `adl://` pour Azure Data Lake Storage Gen1. Si le transfert sécurisé est activé pour le stockage Azure, l’URI sera `wasbs://`. Pour plus d’informations, consultez l’article dédié au [transfert sécurisé](../../storage/common/storage-require-secure-transfer.md).

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md). La plupart des étapes décrites dans ce document supposent que vous utilisez Beeline à partir d’une session SSH sur le cluster. Vous pouvez également utiliser un client Beeline local, mais ces étapes ne sont pas traitées dans cet article.

## <a name="run-a-hive-query"></a>Exécution d'une tâche Hive

Cet exemple est basé sur l’utilisation du client Beeline à partir d’une connexion SSH.

1. Ouvrez une connexion SSH au cluster avec le code ci-dessous. Remplacez `sshuser` par l’utilisateur SSH de votre cluster, puis remplacez `CLUSTERNAME` par le nom de votre cluster. Lorsque vous y êtes invité, entrez le mot de passe du compte d’utilisateur SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Connectez-vous à HiveServer2 avec votre client Beeline à partir de votre session SSH ouverte en entrant la commande suivante :

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
    ```

3. Les commandes Beeline commencent par un caractère `!`. Par exemple, `!help` affiche l’aide. Toutefois, `!` peut être omis pour certaines commandes. Par exemple, `help` fonctionne également.

    Il y a `!sql`, qui est utilisé pour exécuter des instructions HiveQL. Cependant, HiveQL est si souvent utilisé que vous pouvez omettre le `!sql`qui précède. Les deux instructions suivantes sont équivalentes :

    ```hiveql
    !sql show tables;
    show tables;
    ```

    Sur un nouveau cluster, une seule table est listée : **hivesampletable**.

4. Utilisez la commande suivante pour afficher le schéma correspondant à hivesampletable :

    ```hiveql
    describe hivesampletable;
    ```

    Cette commande renvoie les informations suivantes :

    ```output
    +-----------------------+------------+----------+--+
    |       col_name        | data_type  | comment  |
    +-----------------------+------------+----------+--+
    | clientid              | string     |          |
    | querytime             | string     |          |
    | market                | string     |          |
    | deviceplatform        | string     |          |
    | devicemake            | string     |          |
    | devicemodel           | string     |          |
    | state                 | string     |          |
    | country               | string     |          |
    | querydwelltime        | double     |          |
    | sessionid             | bigint     |          |
    | sessionpagevieworder  | bigint     |          |
    +-----------------------+------------+----------+--+
    ```

    Ces informations décrivent les colonnes de la table.

5. Saisissez les instructions suivantes pour créer une table nommée **log4jLogs** avec les exemples de données fournies avec le cluster HDInsight : (Modifiez-les si nécessaire en fonction de votre schéma d’URI.)

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
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs
        WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log'
        GROUP BY t4;
    ```

    Ces instructions effectuent les opérations suivantes :

    |. |Description |
    |---|---|
    |DROP TABLE|Si la table existe, elle est supprimée.|
    |CREATE EXTERNAL TABLE|Crée une table **externe** dans Hive. Les tables externes stockent uniquement la définition de table dans Hive. Les données restent à l'emplacement d'origine.|
    |ROW FORMAT|Formatage des données. Dans ce cas, les champs de chaque journal sont séparés par un espace.|
    |STORED AS TEXTFILE LOCATION|Emplacement de stockage des données et format de fichier.|
    |SELECT|sélectionne toutes les lignes où la colonne **t4** contient la valeur **[ERROR]** . Cette requête renvoie la valeur **3**, car trois lignes contiennent cette valeur.|
    |INPUT__FILE__NAME LIKE '%.log'|Hive tente d’appliquer le schéma à tous les fichiers dans le répertoire. Dans ce cas, le répertoire contient des fichiers qui ne correspondent pas au schéma. Pour éviter que des données incorrectes n’apparaissent dans les résultats, cette instruction indique à Hive de retourner uniquement des données provenant de fichiers se terminant par .log.|

   > [!NOTE]  
   > Les tables externes doivent être utilisées lorsque vous vous attendez à ce que les données sous-jacentes soient mises à jour par une source externe, Par exemple, un processus de chargement de données automatisé ou une opération MapReduce.
   >
   > La suppression d'une table externe ne supprime **pas** les données, mais seulement la définition de la table.

    Le résultat de la commande se présente ainsi :

    ```output
    INFO  : Tez session hasn't been created yet. Opening session
    INFO  :

    INFO  : Status: Running (Executing on YARN cluster with App id application_1443698635933_0001)

    INFO  : Map 1: -/-      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0/1      Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 0(+1)/1  Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0/1
    INFO  : Map 1: 1/1      Reducer 2: 0(+1)/1
    INFO  : Map 1: 1/1      Reducer 2: 1/1
    +----------+--------+--+
    |   sev    | count  |
    +----------+--------+--+
    | [ERROR]  | 3      |
    +----------+--------+--+
    1 row selected (47.351 seconds)
    ```

6. Quittez Beeline :

    ```bash
    !exit
    ```

## <a name="run-a-hiveql-file"></a>Exécuter un fichier HiveQL

Cet exemple constitue la suite du précédent. Utilisez les étapes suivantes pour créer un fichier, puis exécutez-le à l’aide de Beeline.

1. Utilisez la commande suivante pour créer un fichier nommé **query.hql**:

    ```bash
    nano query.hql
    ```

1. Utilisez le texte ci-après comme contenu du fichier. Cette requête crée une table « interne » nommée **errorLogs** :

    ```hiveql
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    Ces instructions effectuent les opérations suivantes :

    |. |Description |
    |---|---|
    |CREATE TABLE IF NOT EXISTS|Si la table n’existe pas, elle est créée. Étant donné que le mot-clé **EXTERNAL** n’est pas utilisé, cette instruction crée une table interne. Les tables internes sont stockées dans l’entrepôt de données Hive et entièrement gérées par Hive.|
    |STORED AS ORC|Stocke les données dans un format ORC (Optimized Row Columnar). Le format ORC est particulièrement efficace et optimisé pour le stockage de données Hive.|
    |INSERT OVERWRITE ... SELECT|sélectionne les lignes de la table **log4jLogs** qui contiennent **[ERROR]** , puis insère les données dans la table **errorLogs**.|

    > [!NOTE]  
    > Contrairement aux tables externes, la suppression d’une table interne entraîne également la suppression des données sous-jacentes.

1. Pour enregistrer le fichier, utilisez **Ctrl**+**X**, puis, entrez **Y** et enfin appuyez sur **Entrée**.

1. Pour exécuter le fichier à l’aide de Beeline, utilisez les éléments suivants :

    ```bash
    beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i query.hql
    ```

    > [!NOTE]  
    > Le paramètre `-i` démarre Beeline et exécute les instructions dans le fichier `query.hql`. Une fois la requête terminée, vous accédez à l’invite `jdbc:hive2://headnodehost:10001/>`. Vous pouvez également exécuter un fichier avec le paramètre `-f`, qui ferme Beeline une fois la requête terminée.

1. Pour vérifier que la table **errorLogs** a été créée, utilisez l’instruction suivante pour renvoyer toutes les lignes à partir **d’errorLogs** :

    ```hiveql
    SELECT * from errorLogs;
    ```

    Trois lignes de données doivent normalement être renvoyées. Elles contiennent toutes **[ERROR]** dans la colonne t4 :

    ```output
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | errorlogs.t1  | errorlogs.t2  | errorlogs.t3  | errorlogs.t4  | errorlogs.t5  | errorlogs.t6  | errorlogs.t7  |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    | 2012-02-03    | 18:35:34      | SampleClass0  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 18:55:54      | SampleClass1  | [ERROR]       | incorrect     | id            |               |
    | 2012-02-03    | 19:25:27      | SampleClass4  | [ERROR]       | incorrect     | id            |               |
    +---------------+---------------+---------------+---------------+---------------+---------------+---------------+--+
    3 rows selected (0.813 seconds)
    ```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Hive dans HDInsight, consultez [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)

* Pour plus d’informations sur les autres façons de travailler avec Hadoop sur HDInsight, consultez [Utiliser MapReduce avec Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)
