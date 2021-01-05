---
title: 'Démarrage rapide : Exécuter des requêtes Apache HBase dans Azure HDInsight - HBase Shell'
description: Dans ce guide de démarrage rapide, vous apprenez à utiliser Apache HBase Shell pour exécuter des requêtes Apache HBase.
keywords: hdinsight,hadoop,HBase
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: quickstart
ms.date: 06/12/2019
ms.author: hrasheed
ms.openlocfilehash: 572262cbece26171f9a67bf073906fa2dfd4d8e1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79371067"
---
# <a name="quickstart-query-apache-hbase-in-azure-hdinsight-with-hbase-shell"></a>Démarrage rapide : Exécuter des requêtes Apache HBase dans Azure HDInsight à l'aide de HBase Shell

Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Apache HBase Shell pour créer une table HBase, insérer des données, puis interroger la table.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache HBase. Consultez [Créer un cluster](../hadoop/apache-hadoop-linux-tutorial-get-started.md) pour créer un cluster HDInsight.  Veillez à choisir le type de cluster **HBase**.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="create-a-table-and-manipulate-data"></a>Créer une table et manipuler des données

Pour la plupart des utilisateurs, les données s’affichent sous la forme tabulaire :

![Données tabulaires Apache HBase HDInsight](./media/query-hbase-with-hbase-shell/hdinsight-hbase-contacts-tabular.png)

Dans HBase (une implémentation de [Cloud BigTable](https://cloud.google.com/bigtable/)), certaines données ont l’aspect suivant :

![Données BigTable Apache HBase HDInsight](./media/query-hbase-with-hbase-shell/hdinsight-hbase-contacts-bigtable.png)

Vous pouvez utiliser SSH pour vous connecter à des clusters HBase, puis utiliser Apache HBase Shell pour créer des tables HBase, et insérer et interroger des données.

1. Utilisez la commande `ssh` pour vous connecter à votre cluster HBase. Modifiez la commande ci-dessous en remplaçant `CLUSTERNAME` par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Utilisez la commande `hbase shell` pour démarrer l'interpréteur de commandes interactif HBase. Entrez la commande suivante dans votre connexion SSH :

    ```bash
    hbase shell
    ```

3. Utilisez la commande `create` pour créer une table HBase avec deux familles de colonnes. Entrez la commande suivante :

    ```hbase
    create 'Contacts', 'Personal', 'Office'
    ```

4. Utilisez la commande `list` pour répertorier toutes les tables contenues dans HBase. Entrez la commande suivante :

    ```hbase
    list
    ```

5. Utilisez la commande `put` pour insérer des valeurs dans une colonne et sur une ligne spécifiées d'une table particulière. Entrez la commande suivante :

    ```hbase
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    ```

6. Utilisez la commande `scan` pour analyser et renvoyer les données de la table `Contacts`. Entrez la commande suivante :

    ```hbase
    scan 'Contacts'
    ```

7. Utilisez la commande `get` pour extraire le contenu d'une ligne. Entrez la commande suivante :

    ```hbase
    get 'Contacts', '1000'
    ```

    Comme il n'y a qu'une seule ligne, vous obtenez des résultats semblables à ceux obtenus avec la commande `scan`.

8. Utilisez la commande `delete` pour supprimer la valeur d'une cellule dans une table. Entrez la commande suivante :

    ```hbase
    delete 'Contacts', '1000', 'Office:Address'
    ```

9. Utilisez la commande `disable` pour désactiver la table. Entrez la commande suivante :

    ```hbase
    disable 'Contacts'
    ```

10. Utilisez la commande `drop` pour déposer une table de HBase. Entrez la commande suivante :

    ```hbase
    drop 'Contacts'
    ```

11. Utilisez la commande `exit` pour arrêter l'interpréteur de commandes interactif HBase. Entrez la commande suivante :

    ```hbase
    exit
    ```

Pour plus d'informations sur le schéma de la table HBase, consultez [Introduction à la conception de schémas Apache HBase](http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf). Pour plus de commandes HBase, consultez le [Guide de référence Apache HBase](https://hbase.apache.org/book.html#quickstart).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir suivi ce guide de démarrage rapide, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées Azure Storage, pour que vous puissiez supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même lorsque vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, économique, mieux vaut supprimer les clusters lorsqu’ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à utiliser Apache HBase Shell pour créer une table HBase, insérer des données, puis interroger la table. Pour en savoir plus sur les données stockées dans HBase, consultez l'article suivant qui explique comment exécuter des requêtes avec Apache Spark.

> [!div class="nextstepaction"]
> [Utiliser Apache Spark pour lire et écrire des données Apache HBase](../hdinsight-using-spark-query-hbase.md)