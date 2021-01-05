---
title: 'Démarrage rapide : Apache Hive dans Azure HDInsight avec Apache Zeppelin'
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Apache Zeppelin pour exécuter des requêtes Apache Hive.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.custom: hdinsightactive
ms.date: 12/03/2019
ms.openlocfilehash: 49b576fd511d17616880e5d981fd3f649de797df
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79367922"
---
# <a name="quickstart-execute-apache-hive-queries-in-azure-hdinsight-with-apache-zeppelin"></a>Démarrage rapide : Exécuter des requêtes Apache Hive dans Azure HDInsight avec Apache Zeppelin

Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Apache Zeppelin pour exécuter des requêtes [Apache Hive](https://hive.apache.org/) dans Azure HDInsight. Les clusters Interactive Query HDInsight incluent les blocs-notes [Apache Zeppelin](https://zeppelin.apache.org/) que vous pouvez utiliser pour exécuter des requêtes Hive interactives.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Un cluster HDInsight Interactive Query. Consultez [Créer un cluster](../hadoop/apache-hadoop-linux-tutorial-get-started.md) pour créer un cluster HDInsight.  Vous devez choisir **Interactive Query** comme type de cluster.

## <a name="create-an-apache-zeppelin-note"></a>Créer une note Apache Zeppelin

1. Remplacez `CLUSTERNAME` par le nom de votre cluster dans l’URL suivante `https://CLUSTERNAME.azurehdinsight.net/zeppelin`. Entrez ensuite l’URL dans un navigateur web.

2. Entrez le nom d’utilisateur et le mot de passe de connexion pour votre cluster. Dans la page Zeppelin, choisissez d’ouvrir une note existante ou d’en créer une. **HiveSample** contient plusieurs exemples de requêtes Hive.  

    ![Zeppelin avec Interactive Query HDInsight](./media/hdinsight-connect-hive-zeppelin/hdinsight-hive-zeppelin.png)

3. Sélectionnez **Créer une note**.

4. Dans la boîte de dialogue **Créer une note**, entrez ou sélectionnez les valeurs suivantes :

    - Nom de la note : entrez un nom pour la note.
    - Interpréteur par défaut : sélectionnez **jdbc** dans la liste déroulante.

5. Sélectionnez **Créer une note**.

6. Entrez la requête Hive suivante dans la section de code, puis appuyez sur **Maj+Entrée** :

    ```hive
    %jdbc(hive)
    show tables
    ```

    ![exécution de la requête Interactive Query HDInsight avec Zeppelin](./media/hdinsight-connect-hive-zeppelin/hdinsight-hive-zeppelin-query.png)

    L’instruction **%jdbc(hive)** dans la première ligne indique au notebook d’utiliser l’interpréteur Hive JDBC.

    La requête retourne une table Hive appelée **hivesampletable**.

    Vous pouvez exécuter les deux autres requêtes Hive suivantes sur la table **hivesampletable** :

    ```hive
    %jdbc(hive)
    select * from hivesampletable limit 10

    %jdbc(hive)
    select ${group_name}, count(*) as total_count
    from hivesampletable
    group by ${group_name=market,market|deviceplatform|devicemake}
    limit ${total_count=10}
    ```

    Les résultats de la requête sont retournés plus rapidement qu’avec une requête Hive standard.

### <a name="additional-examples"></a>Exemples supplémentaires

1. Créez une table. Exécutez le code ci-dessous dans le notebook Zeppelin :

    ```hql
    %jdbc(hive)
    CREATE EXTERNAL TABLE log4jLogs (
        t1 string,
        t2 string,
        t3 string,
        t4 string,
        t5 string,
        t6 string,
        t7 string)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE;
    ```

1. Chargez des données dans la nouvelle table. Exécutez le code ci-dessous dans le notebook Zeppelin :

    ```hql
    %jdbc(hive)
    LOAD DATA
    INPATH 'wasbs:///example/data/sample.log'
    INTO TABLE log4jLogs;
    ```

1. Insérez un enregistrement unique. Exécutez le code ci-dessous dans le notebook Zeppelin :

    ```hql
    %jdbc(hive)
    INSERT INTO TABLE log4jLogs2
    VALUES ('A', 'B', 'C', 'D', 'E', 'F', 'G');
    ```

Passez en revue le [manuel du langage Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) pour découvrir d’autres syntaxes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir suivi ce guide de démarrage rapide, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées dans le stockage Azure. Vous pouvez ainsi supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser Apache Zeppelin pour exécuter des requêtes Apache Hive dans Azure HDInsight. Pour en savoir plus sur les requêtes Hive, consultez l’article suivant qui explique comment exécuter des requêtes avec Visual Studio.

> [!div class="nextstepaction"]
> [Se connecter à Azure HDInsight et exécuter des requêtes Apache Hive avec Data Lake Tools pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)
