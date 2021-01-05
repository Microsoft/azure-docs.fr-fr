---
title: 'Démarrage rapide : Apache HBase et Apache Phoenix - Azure HDInsight'
description: Dans ce guide de démarrage rapide, vous apprenez à utiliser Apache Phoenix dans HDInsight. Découvrez également comment installer et configurer SQLLine sur votre ordinateur pour vous connecter à un cluster HBase dans HDInsight.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: quickstart
ms.date: 06/12/2019
ms.author: hrasheed
ms.openlocfilehash: 1c400e41c4c10023d2595bde8c0d62e26184cf05
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79370319"
---
# <a name="quickstart-query-apache-hbase-in-azure-hdinsight-with-apache-phoenix"></a>Démarrage rapide : Exécuter des requêtes Apache HBase dans Azure HDInsight avec Apache Phoenix

Lors de ce démarrage rapide, vous allez apprendre à utiliser Apache Phoenix pour exécuter des requêtes HBase dans Azure HDInsight. Apache Phoenix est un moteur de requête SQL pour Apache HBase. Il est accessible en tant que pilote JDBC et permet d'interroger et de gérer les tables HBase au moyen de SQL. [SQLLine](http://sqlline.sourceforge.net/) est un utilitaire de ligne de commande pour exécuter SQL.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache HBase. Consultez [Créer un cluster](../hadoop/apache-hadoop-linux-tutorial-get-started.md) pour créer un cluster HDInsight.  Veillez à choisir le type de cluster **HBase**.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="identify-a-zookeeper-node"></a>Identifier un nœud ZooKeeper

Quand vous vous connectez à un cluster HBase, vous devez vous connecter à l’un des nœuds Apache ZooKeeper. Chaque cluster HDInsight a trois nœuds ZooKeeper. Curl peut être utilisé pour identifier rapidement un nœud ZooKeeper. Modifiez la commande curl ci-dessous en remplaçant `PASSWORD` et `CLUSTERNAME` par les valeurs appropriées, puis entrez la commande dans une invite de commandes :

```cmd
curl -u admin:PASSWORD -sS -G https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER
```

Une partie du résultat ressemble à ce qui suit :

```output
    {
      "href" : "http://hn1-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net:8080/api/v1/clusters/myCluster/hosts/zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net/host_components/ZOOKEEPER_SERVER",
      "HostRoles" : {
        "cluster_name" : "myCluster",
        "component_name" : "ZOOKEEPER_SERVER",
        "host_name" : "zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net"
      }
```

Notez la valeur de `host_name` pour une utilisation ultérieure.

## <a name="create-a-table-and-manipulate-data"></a>Créer une table et manipuler des données

Vous pouvez utiliser SSH pour vous connecter à des clusters HBase, puis utiliser Apache Phoenix pour créer des tables HBase, et insérer et interroger des données.

1. Utilisez la commande `ssh` pour vous connecter à votre cluster HBase. Modifiez la commande ci-dessous en remplaçant `CLUSTERNAME` par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Remplacez le répertoire par le client Phoenix. Entrez la commande suivante :

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

3. Lancer [SQLLine](http://sqlline.sourceforge.net/). Modifiez la commande ci-dessous en remplaçant `ZOOKEEPER` le nœud ZooKeeper identifié précédemment, puis entrez la commande :

    ```bash
    ./sqlline.py ZOOKEEPER:2181:/hbase-unsecure
    ```

4. Créez une table HBase. Entrez la commande suivante :

    ```sql
    CREATE TABLE Company (company_id INTEGER PRIMARY KEY, name VARCHAR(225));
    ```

5. Utiliser la commande SQLLine `!tables` pour répertorier toutes les tables contenues dans HBase. Entrez la commande suivante :

    ```sqlline
    !tables
    ```

6. Insérer les valeurs dans le tableau. Entrez la commande suivante :

    ```sql
    UPSERT INTO Company VALUES(1, 'Microsoft');
    UPSERT INTO Company VALUES(2, 'Apache');
    ```

7. Interroger la table. Entrez la commande suivante :

    ```sql
    SELECT * FROM Company;
    ```

8. Supprimer un enregistrement. Entrez la commande suivante :

    ```sql
    DELETE FROM Company WHERE COMPANY_ID=1;
    ```

9. Déposer la table. Entrez la commande suivante :

    ```hbase
    DROP TABLE Company;
    ```

10. Utiliser la commande SQLLine `!quit` pour quitter SQLLine. Entrez la commande suivante :

    ```sqlline
    !quit
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir suivi ce guide de démarrage rapide, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées Azure Storage, pour que vous puissiez supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même lorsque vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, économique, mieux vaut supprimer les clusters lorsqu’ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Lors de ce démarrage rapide, vous avez appris à utiliser Apache Phoenix pour exécuter des requêtes HBase dans Azure HDInsight. Pour en savoir plus sur Apache Phoenix, l’article suivant fournit une étude plus approfondie.

> [!div class="nextstepaction"]
> [Apache Phoenix dans HDInsight](../hdinsight-phoenix-in-hdinsight.md)
