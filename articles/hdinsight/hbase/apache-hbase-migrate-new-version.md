---
title: Migrer un cluster HBase vers une nouvelle version - Azure HDInsight
description: Explique comment effectuer la migration de clusters Apache HBase vers une version plus récente dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/02/2020
ms.openlocfilehash: 8ce25780e197c26e0e5b102670e093031e1a2582
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697660"
---
# <a name="migrate-an-apache-hbase-cluster-to-a-new-version"></a>Effectuer la migration d’un cluster Apache HBase vers une nouvelle version

Cet article décrit les étapes requises pour mettre à jour de votre cluster Apache HBase sur Azure HDInsight vers une version plus récente.

Le temps d’arrêt lors de la mise à niveau doit être minime, de l’ordre de quelques minutes. Ce temps d’arrêt est dû à la procédure pour vider toutes les données en mémoire, puis au temps nécessaire à la configuration et au redémarrage des services sur le nouveau cluster. Vos résultats peuvent varier en fonction du nombre de nœuds, de la quantité de données et d’autres variables.

## <a name="review-apache-hbase-compatibility"></a>Vérifier la compatibilité d’Apache HBase

Avant de mettre à niveau Apache HBase, vérifiez que la version de HBase présente sur le cluster source est compatible avec celle du cluster de destination. Pour plus d’informations, consultez [Composants et versions d’Apache Hadoop disponibles avec HDInsight](../hdinsight-component-versioning.md).

> [!NOTE]  
> Nous vous recommandons vivement d’examiner la matrice de compatibilité de versions dans le [guide sur HBase](https://hbase.apache.org/book.html#upgrading). Les dernières incompatibilités doivent être décrites dans les notes de publication de version HBase.

Voici un exemple de matrice de compatibilité de versions. O indique une compatibilité et N une incompatibilité potentielle :

| Type de compatibilité | Version principale| Version secondaire | Correctif |
| --- | --- | --- | --- |
| Compatibilité communication client-serveur | N | O | O |
| Compatibilité serveur-serveur | N | O | O |
| Compatibilité format de fichier | N | O | O |
| Compatibilité API client | N | O | O |
| Compatibilité fichier binaire client | N | N | O |
| **Compatibilité API limitée côté serveur** |  |  |  |
| Stable | N | O | O |
| En cours d’évolution | N | N | O |
| Instable | N | N | N |
| Compatibilité dépendance | N | O | O |
| Compatibilité opérationnelle | N | N | O |

## <a name="upgrade-with-same-apache-hbase-major-version"></a>Mettre à niveau vers la même version principale d’Apache HBase

Pour mettre à niveau votre cluster Apache HBase sur Azure HDInsight, procédez comme suit :

1. Vérifiez que votre application est compatible avec la nouvelle version, comme indiqué dans les notes de version et la matrice de compatibilité HBase. Testez votre application dans un cluster exécutant la version cible de HDInsight et HBase.

1. [Configurez un nouveau cluster HDInsight de destination](../hdinsight-hadoop-provision-linux-clusters.md) avec le même compte de stockage, mais avec un nom de conteneur différent :

   ![Utiliser le même compte de stockage, mais créer un autre conteneur](./media/apache-hbase-migrate-new-version/same-storage-different-container.png)

1. Videz votre cluster HBase source, qui est le cluster que vous mettez à niveau. HBase écrit les données entrantes dans un magasin en mémoire, appelé _memstore_. Une fois que le memstore atteint une certaine taille, HBase le vide sur le disque pour le stockage à long terme dans le compte de stockage du cluster. Lorsque vous supprimez l’ancien cluster, les memstores sont recyclés, ce qui entraîne potentiellement une perte de données. Pour vider manuellement le memstore de chaque table sur le disque, exécutez le script suivant. La version la plus récente de ce script se trouve sur [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/flush_all_tables.sh) Azure.

    ```bash
    #!/bin/bash
    
    #-------------------------------------------------------------------------------#
    # SCRIPT TO FLUSH ALL HBASE TABLES.
    #-------------------------------------------------------------------------------#
    
    LIST_OF_TABLES=/tmp/tables.txt
    HBASE_SCRIPT=/tmp/hbase_script.txt
    TARGET_HOST=$1
    
    usage ()
    {
        if [[ "$1" == "-h" ]] || [[ "$1" == "--help" ]]
        then
            cat << ...
    
    Usage: 
    
    $0 [hostname]
    
    Providing hostname is optional and not required when the script is executed within HDInsight cluster with access to 'hbase shell'.
    
    However hostname should be provided when executing the script as a script-action from HDInsight portal.
    
    For Example:
    
        1.  Executing script inside HDInsight cluster (where 'hbase shell' is 
            accessible):
    
            $0 
    
            [No need to provide hostname]
    
        2.  Executing script from HDinsight Azure portal:
    
            Provide Script URL.
    
            Provide hostname as a parameter (i.e. hn0, hn1, hn2.. or wn2 etc.).
    ...
            exit
        fi
    }
    
    validate_machine ()
    {
        THIS_HOST=`hostname`
    
        if [[ ! -z "$TARGET_HOST" ]] && [[ $THIS_HOST  != $TARGET_HOST* ]]
        then
            echo "[INFO] This machine '$THIS_HOST' is not the right machine ($TARGET_HOST) to execute the script."
            exit 0
        fi
    }
    
    get_tables_list ()
    {
    hbase shell << ... > $LIST_OF_TABLES 2> /dev/null
        list
        exit
    ...
    }
    
    add_table_for_flush ()
    {
        TABLE_NAME=$1
        echo "[INFO] Adding table '$TABLE_NAME' to flush list..."
        cat << ... >> $HBASE_SCRIPT
            flush '$TABLE_NAME'
    ...
    }
    
    clean_up ()
    {
        rm -f $LIST_OF_TABLES
        rm -f $HBASE_SCRIPT
    }
    
    ########
    # MAIN #
    ########
    
    usage $1
    
    validate_machine
    
    clean_up
    
    get_tables_list
    
    START=false
    
    while read LINE 
    do 
        if [[ $LINE == TABLE ]] 
        then
            START=true
            continue
        elif [[ $LINE == *row*in*seconds ]]
        then
            break
        elif [[ $START == true ]]
        then
            add_table_for_flush $LINE
        fi
    
    done < $LIST_OF_TABLES
    
    cat $HBASE_SCRIPT
    
    hbase shell $HBASE_SCRIPT << ... 2> /dev/null
    exit
    ...
    
    ```

1. Arrêtez l’ingestion des données vers l’ancien cluster HBase.

1. Pour vous assurer que toutes les données récentes dans le memstore sont vidées, exécutez à nouveau le script précédent.

1. Connectez-vous à [Apache Ambari](https://ambari.apache.org/) sur l’ancien cluster (`https://OLDCLUSTERNAME.azurehdidnsight.net`) et interrompez les services HBase. Lorsque vous êtes invité à confirmer que vous souhaitez arrêter les services, cochez la case pour activer le mode de maintenance pour HBase. Pour plus d’informations sur la connexion à Ambari et sur l’utilisation d’Ambari, consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur Web d’Ambari](../hdinsight-hadoop-manage-ambari.md).

    ![Dans Ambari, cliquez sur Services > HBase > Arrêter sous Action du service](./media/apache-hbase-migrate-new-version/stop-hbase-services1.png)

    ![Cocher la case pour activer le mode de maintenance pour HBase, puis confirmer](./media/apache-hbase-migrate-new-version/turn-on-maintenance-mode.png)

1. Si vous n’utilisez pas de clusters HBase avec la fonctionnalité Écritures améliorées, ignorez cette étape. Elle n’est requise que pour les clusters HBase avec la fonctionnalité Écritures améliorées.

   Sauvegardez le répertoire WAL sous HDFS en exécutant les commandes ci-dessous dans une session SSH sur l’un des nœuds ZooKeeper ou Worker du cluster d’origine.
   
   ```bash
   hdfs dfs -mkdir /hbase-wal-backup**
   hdfs dfs -cp hdfs://mycluster/hbasewal /hbase-wal-backup**
   ```
    
1. Connectez-vous à Ambari sur le nouveau cluster HDInsight. Modifiez le paramètre HDFS `fs.defaultFS` pour pointer vers le nom du conteneur utilisé par le cluster d’origine. Ce paramètre se trouve sous **HDFS > Configurations > Avancé > Advanced core-site (Site principal avancé)** .

   ![Dans Ambari, cliquez sur Services > HDFS > Configurations > Avancé](./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png)

   ![Dans Ambari, modifier le nom du conteneur](./media/apache-hbase-migrate-new-version/change-container-name.png)

1. Si vous n’utilisez pas de clusters HBase avec la fonctionnalité Écritures améliorées, ignorez cette étape. Elle n’est requise que pour les clusters HBase avec la fonctionnalité Écritures améliorées.

   Modifiez le chemin d’accès `hbase.rootdir` pour qu’il pointe vers le conteneur du cluster d’origine.

   ![Dans Ambari, modifier le nom du conteneur pour HBase rootdir](./media/apache-hbase-migrate-new-version/change-container-name-for-hbase-rootdir.png)
    
1. Si vous n’utilisez pas de clusters HBase avec la fonctionnalité Écritures améliorées, ignorez cette étape. Elle n’est nécessaire que pour les clusters HBase dotés de la fonctionnalité Écritures améliorées, et uniquement dans le cas où le cluster d’origine était un cluster HBase doté de la fonctionnalité Écritures améliorées.

   Nettoyez les données FS ZooKeeper et WAL de ce nouveau cluster. Émettez les commandes suivantes dans l’un des nœuds ZooKeeper ou Worker :

   ```bash
   hbase zkcli
   rmr /hbase-unsecure
   quit

   hdfs dfs -rm -r hdfs://mycluster/hbasewal**
   ```

1. Si vous n’utilisez pas de clusters HBase avec la fonctionnalité Écritures améliorées, ignorez cette étape. Elle n’est requise que pour les clusters HBase avec la fonctionnalité Écritures améliorées.
   
   Restaurez le répertoire WAL sur le nouveau HDFS du cluster dans une session SSH sur l’un des nœuds ZooKeeper ou Worker du nouveau cluster.
   
   ```bash
   hdfs dfs -cp /hbase-wal-backup/hbasewal hdfs://mycluster/**
   ```
   
1. Si vous mettez à niveau HDInsight 3.6 vers la version 4.0, suivez les étapes ci-dessous, sinon passez à l’étape 13 :

    1. Redémarrez tous les services requis dans Ambari en sélectionnant **Services** > **Restart All Required** (Redémarrer tous les services requis).
    1. Arrêtez le service HBase.
    1. Ouvrez une connexion SSH sur le nœud Zookeeper et exécutez la commande [zkCli](https://github.com/go-zkcli/zkcli)`rmr /hbase-unsecure` pour supprimer le znode racine HBase de Zookeeper.
    1. Redémarrez HBase.

1. Si vous mettez à niveau vers une version de HDInsight autre que 4.0, procédez comme suit :
    1. Enregistrez vos modifications.
    1. Redémarrez tous les services requis, comme indiqué par Ambari.

1. Pointez votre application vers le nouveau cluster.

    > [!NOTE]  
    > Le DNS statique de votre application est modifié lors de la mise à niveau. Au lieu de coder en dur ce DNS, vous pouvez configurer un CNAME dans les paramètres DNS de votre nom de domaine qui pointe vers le nom du cluster. Une autre option consiste à utiliser un fichier config pour votre application, que vous pouvez mettre à jour sans nouveau déploiement.

1. Lancez l’ingestion des données pour voir si tout fonctionne comme prévu.

1. Si le nouveau cluster vous convient, supprimez le cluster d’origine.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur [Apache HBase](https://hbase.apache.org/) et la mise à niveau de clusters HDInsight, consultez les articles suivants :

* [Mettre à niveau le cluster HDInsight](../hdinsight-upgrade-cluster.md)
* [Superviser et gérer Azure HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](../hdinsight-hadoop-manage-ambari.md)
* [Composants et versions d’Apache Hadoop](../hdinsight-component-versioning.md)
* [Optimiser Apache HBase](../optimize-hbase-ambari.md)
