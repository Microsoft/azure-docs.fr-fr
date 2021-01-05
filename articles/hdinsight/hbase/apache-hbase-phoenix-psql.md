---
title: 'Chargement en bloc dans Apache Phoenix à l’aide de psql : Azure HDInsight'
description: Utiliser l’outil psql pour charger des données en bloc dans des tables Apache Phoenix sur Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: 513d48114d73b2c91ebecf06f9492a6ebf0ba1cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89504704"
---
# <a name="bulk-load-data-into-apache-phoenix-using-psql"></a>Charger des données en bloc dans Apache Phoenix à l’aide de psql

[Apache Phoenix](https://phoenix.apache.org/) est une couche de base de données relationnelle massivement parallèle open source basée sur [Apache HBase](../hbase/apache-hbase-overview.md). Phoenix fournit des requêtes de type SQL sur HBase. Phoenix utilise des pilotes JDBC pour permettre aux utilisateurs de créer, supprimer et modifier des tables SQL, index, vues et séquences, ainsi que des lignes d’upsert individuellement et en bloc. Phoenix utilise une compilation native noSQL au lieu de MapReduce pour compiler les requêtes, pour créer des applications à faible latence sur HBase. Phoenix ajoute des co-processeurs pour prendre en charge le code fourni par le client en cours d’exécution dans l’espace d’adressage du serveur, en exécutant le code colocalisé avec les données. Cela réduit le transfert de données client/serveur.  Pour utiliser des données à l’aide de Phoenix dans HDInsight, créez tout d’abord des tables, puis chargez des données dans celles-ci.

## <a name="bulk-loading-with-apache-phoenix"></a>Chargement en masse avec Apache Phoenix

Plusieurs méthodes permettent d’obtenir des données dans HBase, notamment à l’aide d’API clientes, une tâche MapReduce avec TableOutputFormat ou en entrant les données manuellement à l’aide du shell HBase. Phoenix propose deux méthodes pour charger des données CSV dans des tables Phoenix : un outil de chargement client nommé `psql`et un outil de chargement en bloc basé sur MapReduce.

L’outil `psql` est monothread et parfaitement adapté au chargement de mégaoctets ou de gigaoctets de données. Tous les fichiers CSV à charger doivent avoir l’extension de fichier « .csv ».  Vous pouvez également spécifier des fichiers de script SQL dans la ligne de commande `psql` avec l’extension de fichier « .sql ».

Le chargement en bloc avec MapReduce est utilisé pour des volumes plus importants de données, généralement dans des scénarios de production, car MapReduce utilise plusieurs threads.

Avant de commencer à charger des données, vérifiez que Phoenix est activé et que les paramètres de délai d’expiration de requête sont tel que prévu.  Accédez au tableau de bord [Apache Ambari](https://ambari.apache.org/) de votre cluster HDInsight, sélectionnez HBase, puis sélectionnez l’onglet Configuration.  Faites défiler vers le bas pour vérifier si Apache Phoenix est défini sur `enabled` comme illustré :

![Paramètres de cluster HDInsight Apache Phoenix](./media/apache-hbase-phoenix-psql/apache-ambari-phoenix.png)

### <a name="use-psql-to-bulk-load-tables"></a>Utiliser `psql` pour charger des tables en bloc

1. Créez un fichier appelé `createCustomersTable.sql`et copiez le code ci-dessous dedans. Ensuite, enregistrez et fermez le fichier.

    ```sql
    CREATE TABLE Customers (
        ID varchar NOT NULL PRIMARY KEY,
        Name varchar,
        Income decimal,
        Age INTEGER,
        Country varchar);
    ```

1. Créez un fichier appelé `listCustomers.sql`et copiez le code ci-dessous dedans. Ensuite, enregistrez et fermez le fichier.

    ```sql
    SELECT * from Customers;
    ```

1. Créez un fichier appelé `customers.csv`et copiez le code ci-dessous dedans. Ensuite, enregistrez et fermez le fichier.

    ```txt
    1,Samantha,260000.0,18,US
    2,Sam,10000.5,56,US
    3,Anton,550150.0,42,Norway
    ```

1. Créez un fichier appelé `customers2.csv`et copiez le code ci-dessous dedans. Ensuite, enregistrez et fermez le fichier.

    ```txt
    4,Nicolle,180000.0,22,US
    5,Kate,210000.5,24,Canada
    6,Ben,45000.0,32,Poland
    ```

1. Ouvrez une invite de commandes et accédez au répertoire dans lequel se trouve le fichier nouvellement créé. Ci-dessous, remplacez CLUSTERNAME par le nom réel de votre cluster HBase. Ensuite, exécutez le code pour charger les fichiers vers le nœud principal de votre cluster :

    ```cmd
    scp customers.csv customers2.csv createCustomersTable.sql listCustomers.sql sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/tmp
    ```

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. À partir de votre session SSH, accédez à l’emplacement de l’outil **psql**. Exécutez les commandes ci-dessous :

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

1. Chargez les données en masse. Le code ci-dessous crée la table **Customers**, puis charge les données.

    ```bash
    python psql.py /tmp/createCustomersTable.sql /tmp/customers.csv
    ```

    Lorsque l’opération `psql` est terminée, vous devriez voir un message semblable à celui-ci :

    ```output
    csv columns from database.
    CSV Upsert complete. 3 rows upserted
    Time: 0.081 sec(s)
    ```

1. Vous pouvez continuer à utiliser `psql` pour afficher les contenus de la table Customers. Exécutez le code ci-dessous :

    ```bash
    python psql.py /tmp/listCustomers.sql
    ```

    Vous pouvez également utiliser l’[interpréteur de commandes HBase](./query-hbase-with-hbase-shell.md) ou [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md) pour interroger les données.

1. Charger des données supplémentaires. Maintenant que la table existe déjà, la commande la spécifie. Exécutez les commandes ci-dessous :

    ```bash
    python psql.py -t CUSTOMERS /tmp/customers2.csv
    ```

## <a name="use-mapreduce-to-bulk-load-tables"></a>Utiliser MapReduce pour charger des tables en bloc

Pour un chargement à un débit supérieur distribué sur le cluster, utilisez l’outil de chargement MapReduce. Ce chargeur convertit d’abord toutes les données dans HFiles, puis fournit le HFiles créé à HBase.

1. Cette section se poursuit avec la session SSH et les objets créés précédemment. En fonction de vos besoins, créez la table **Customers** et le fichier **customers.csv** en suivant des étapes décrites ci-dessus. Si nécessaire, rétablissez votre connexion SSH.

1. Tronquer les contenu de la table **Customers**. À partir de votre session SSH ouverte, exécutez les commandes suivantes :

    ```bash
    hbase shell
    truncate 'CUSTOMERS'
    exit
    ```

1. Copiez le fichier `customers.csv` de votre nœud principal dans le stockage Azure.

    ```bash
    hdfs dfs -put /tmp/customers.csv wasbs:///tmp/customers.csv
    ```

1. Accédez au répertoire d’exécution de la commande de chargement en bloc MapReduce :

    ```bash
    cd /usr/hdp/current/phoenix-client
    ```

1. Lancez le chargeur CSV MapReduce à l’aide de la commande `hadoop` avec le jar client Phoenix :

    ```bash
    HADOOP_CLASSPATH=/usr/hdp/current/hbase-client/lib/hbase-protocol.jar:/etc/hbase/conf hadoop jar phoenix-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input /tmp/customers.csv
    ```

    Une fois le chargement terminé, un message semblable à celui-ci devrait s’afficher :

    ```output
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x26f15dcceff02c3
    19/12/18 18:30:57 INFO zookeeper.ZooKeeper: Session: 0x26f15dcceff02c3 closed
    19/12/18 18:30:57 INFO zookeeper.ClientCnxn: EventThread shut down
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Incremental load complete for table=CUSTOMERS
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Removing output directory /tmp/50254426-aba6-400e-88eb-8086d3dddb6
    ```

1. Pour utiliser MapReduce avec Azure Data Lake Storage, recherchez le répertoire racine Data Lake Storage, qui correspond à la valeur `hbase.rootdir` dans `hbase-site.xml`. Dans la commande suivante, le répertoire racine Data Lake Storage est `adl://hdinsightconf1.azuredatalakestore.net:443/hbase1`. Dans cette commande, spécifiez les dossiers d’entrée et de sortie Data Lake Storage en tant que paramètres :

    ```bash
    cd /usr/hdp/current/phoenix-client

    $HADOOP_CLASSPATH=$(hbase mapredcp):/etc/hbase/conf hadoop jar /usr/hdp/2.4.2.0-258/phoenix/phoenix-4.4.0.2.4.2.0-258-client.jar

    org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/temp/input/customers.csv –zookeeper ZookeeperQuorum:2181:/hbase-unsecure --output  adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/output1
    ```

1. Pour interroger et afficher les données, vous pouvez utiliser **psql** comme décrit plus haut. Vous pouvez également utiliser l’[interpréteur de commandes HBase](./query-hbase-with-hbase-shell.md) ou [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md).

## <a name="recommendations"></a>Recommandations

* Utilisez le même support de stockage pour les dossiers d’entrée et de sortie, Stockage Azure (WASB) ou Azure Data Lake Storage (ADL). Pour transférer des données entre Stockage Azure et Data Lake Storage, utilisez la commande `distcp` :

    ```bash
    hadoop distcp wasb://@.blob.core.windows.net/example/data/gutenberg adl://.azuredatalakestore.net:443/myfolder
    ```

* Utilisez des nœuds de travail de plus grande taille. Les processus de mappage de la copie en bloc MapReduce produisent de grandes quantités de sortie temporaire qui saturent l’espace non DFS disponible. Pour une grande quantité de chargement en bloc, utilisez un plus grand nombre de nœuds de travail et de plus grande taille. Le nombre de nœuds de travail que vous allouez à votre cluster affecte directement la vitesse de traitement.

* Fractionnez les fichiers d’entrée en segments de 10 Go environ. Le chargement en bloc est une opération grande consommatrice du stockage, le fractionnement de vos fichiers d’entrée en plusieurs segments offre donc de meilleures performances.

* Évitez les zones réactives du serveur de la région. Si votre clé de ligne augmente de manière monolithique, les écritures séquentielles HBase peuvent entraîner un « hotspotting » du serveur de la région. Un *Salting* de la clé de ligne réduit les écritures séquentielles. Phoenix fournit une méthode transparente de « salting » de la clé de ligne avec un octet de « salting » pour une table particulière, comme indiqué ci-dessous.

## <a name="next-steps"></a>Étapes suivantes

* [Chargement de données en bloc avec Apache Phoenix](https://phoenix.apache.org/bulk_dataload.html)
* [Utiliser Apache Phoenix avec les clusters Apache HBase basés sur Linux dans HDInsight](../hbase/apache-hbase-query-with-phoenix.md)
* [Tables « salted »](https://phoenix.apache.org/salted.html)
* [Grammaire Apache Phoenix](https://phoenix.apache.org/language/index.html)
