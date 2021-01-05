---
title: Apache Spark et Hive - Hive Warehouse Connector - Azure HDInsight
description: Découvrez comment intégrer Apache Spark et Apache Hive au connecteur d’entrepôt Hive sur Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: 24968511d038b2cea41a59187c0a361684c6720e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86511889"
---
# <a name="integrate-apache-spark-and-apache-hive-with-hive-warehouse-connector-in-azure-hdinsight"></a>Intégrer Apache Spark et Apache Hive à Hive Warehouse Connector dans Azure HDInsight

Apache Hive Warehouse Connector (HWC) est une bibliothèque qui vous permet de travailler plus facilement avec Apache Spark et Apache Hive, Elle prend en charge certaines tâches comme le déplacement de données entre les DataFrames Spark et les tables Hive. De même, elle achemine les données de streaming Spark vers les tables Hive. Le connecteur d’entrepôt Hive fonctionne comme un pont entre Spark et Hive. Cette bibliothèque prend aussi en charge l’utilisation des langages de programmation Scala, Java et Python à des fins de développement.

Le connecteur d’entrepôt Hive vous permet de profiter des fonctionnalités uniques de Hive et de Spark afin de créer de puissantes applications Big Data.

Apache Hive prend en charge les transactions de base de données ACID (Atomiques, Cohérentes, Isolées et Durables). Pour plus d'informations sur ACID et les transactions dans Hive, voir [Transactions Hive](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions). Hive offre également des contrôles de sécurité détaillés via Apache Ranger et Low Latency Analytical Processing (LLAP), des options non disponibles dans Apache Spark.

Apache Spark dispose d'une API de flux structuré qui offre des capacités de diffusion en continu non disponibles dans Apache Hive. À partir de HDInsight 4.0, Apache Spark 2.3.1 et Apache Hive 3.1.0 ont des metastores distincts. Cette disparité de metastores peut nuire à l’interopérabilité. Le connecteur d'entrepôt Hive facilite l'utilisation simultanée de Spark et de Hive. La bibliothèque HWC charge les données de démons LLAP dans des exécuteurs Spark en parallèle. Ce processus la rend plus efficace et plus adaptable qu’une connexion JDBC standard de Spark à Hive.

![Architecture du connecteur d’entrepôt Hive](./media/apache-hive-warehouse-connector/hive-warehouse-connector-architecture.png)

Voici quelques-unes des opérations prises en charge par le connecteur d'entrepôt Hive :

* Description d’une table
* Création d'une table pour les données au format ORC
* Sélection de données Hive et récupération d'un DataFrame
* Écriture par lots d'un DataFrame vers Hive
* Exécution d'une instruction de mise à jour Hive
* Lecture des données d’une table à partir de Hive, transformation de ces données dans Spark, puis leur écriture dans une nouvelle table Hive
* Écriture d'un flux DataFrame ou Spark vers Hive à l'aide de HiveStreaming

## <a name="hive-warehouse-connector-setup"></a>Configuration du connecteur d’entrepôt Hive

> [!IMPORTANT]
> L’instance HiveServer2 Interactive installée sur les clusters Pack Sécurité Entreprise Spark 2.4 n’est pas prise en charge pour une utilisation avec Hive Warehouse Connector. Vous devez configurer un cluster HiveServer2 Interactive distinct pour héberger vos charges de travail HiveServer2 Interactive. Une configuration Hive Warehouse Connector utilisant un cluster Spark 2.4 unique n’est pas prise en charge.

Hive Warehouse Connector a besoin de clusters distincts pour les charges de travail Spark et Interactive Query. Suivez les étapes ci-dessous pour configurer ces clusters dans Azure HDInsight.

### <a name="create-clusters"></a>Créer des clusters

1. Créez un cluster HDInsight Spark **4.0** avec un compte de stockage et un réseau virtuel Azure personnalisé. Pour plus d'informations sur la création d'un cluster dans un réseau virtuel Azure, voir [Ajouter HDInsight à un réseau virtuel existant](../../hdinsight/hdinsight-plan-virtual-network-deployment.md#existingvnet).

1. Créez un cluster HDInsight Interactive Query (LLAP) **4.0** avec le même compte de stockage et le même réseau virtuel Azure que pour le cluster Spark.

### <a name="configure-hwc-settings"></a>Configurer les paramètres HWC

#### <a name="gather-preliminary-information"></a>Recueillir des informations préliminaires

1. À partir d’un navigateur web, accédez à `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HIVE`, où LLAPCLUSTERNAME est le nom de votre cluster Interactive Query.

1. Accédez à **Summary** > **HiveServer2 Interactive JDBC URL** et notez la valeur. La valeur peut être similaire à : `jdbc:hive2://zk0-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181,zk1-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181,zk4-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2-interactive`.

1. Accédez à **Configs** > **Advanced** > **Advanced hive-site** > **hive.zookeeper.quorum** et notez la valeur. La valeur peut être similaire à : `zk0-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,zk1-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,zk4-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181`.

1. Accédez à **Configs** > **Advanced** > **General** > **hive.metastore.uris** et notez la valeur. La valeur peut être similaire à : `thrift://iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083,thrift://hn1-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083`.

1. Accédez à **Configs** > **Advanced** > **Advanced hive-interactive-site** > **hive.llap.daemon.service.hosts** et notez la valeur. La valeur peut être similaire à : `@llap0`.

#### <a name="configure-spark-cluster-settings"></a>Configurer les paramètres du cluster Spark

1. À partir d’un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, où CLUSTERNAME est le nom de votre cluster Apache Spark.

1. Développez **Custom spark2-defaults**.

    ![Apache Ambari - Configuration Spark2](./media/apache-hive-warehouse-connector/hive-warehouse-connector-spark2-ambari.png)

1. Sélectionnez **Add Property...** (Ajouter une propriété...) pour ajouter les configurations suivantes :

    | Configuration | Valeur |
    |----|----|
    |`spark.datasource.hive.warehouse.load.staging.dir`|`wasbs://STORAGE_CONTAINER_NAME@STORAGE_ACCOUNT_NAME.blob.core.windows.net/tmp`. <br> Définir sur un répertoire de préproduction compatible HDFS approprié. Si vous utilisez deux clusters différents, ce répertoire doit être un dossier dans le répertoire de préproduction du compte de stockage du cluster LLAP afin que HiveServer2 puisse y accéder.  Remplacez `STORAGE_ACCOUNT_NAME` par le nom du compte de stockage utilisé par le cluster et `STORAGE_CONTAINER_NAME` par le nom du conteneur de stockage. |
    |`spark.sql.hive.hiveserver2.jdbc.url`| Valeur que vous avez obtenue précédemment à partir de **HiveServer2 Interactive JDBC URL** |
    |`spark.datasource.hive.warehouse.metastoreUri`| La valeur que vous avez obtenue précédemment à partir de **hive.metastore.uris**. |
    |`spark.security.credentials.hiveserver2.enabled`|`true` pour le mode de cluster YARN et `false` pour le mode client YARN. |
    |`spark.hadoop.hive.zookeeper.quorum`| La valeur que vous avez obtenue précédemment à partir de **hive.zookeeper.quorum**. |
    |`spark.hadoop.hive.llap.daemon.service.hosts`| La valeur que vous avez obtenue précédemment à partir de **hive.llap.daemon.service.hosts**. |

1. Enregistrez les modifications, puis redémarrez tous les composants concernés.

### <a name="configure-hwc-for-enterprise-security-package-esp-clusters"></a>Configurer HWC pour des clusters Pack Sécurité Entreprise (ESP)

Le Pack Sécurité Entreprise (ESP) vous fournit des fonctionnalités de qualité professionnelle, notamment l’authentification basée sur Active Directory, la prise en charge multi-utilisateur et le contrôle d’accès en fonction du rôle pour les clusters Apache Hadoop dans Azure HDInsight. Pour plus d’informations sur le pack ESP, consultez [Utiliser le Pack Sécurité Entreprise dans HDInsight](../domain-joined/apache-domain-joined-architecture.md).

En dehors des configurations mentionnées dans la section précédente, ajoutez la configuration suivante pour utiliser HWC sur les clusters ESP.

1. À partir de l’interface utilisateur web Ambari du cluster Spark, accédez à **Spark2** > **CONFIGS** > **Custom spark2-defaults**.

1. Mettez à jour la propriété suivante.

    | Configuration | Valeur |
    |----|----|
    | `spark.sql.hive.hiveserver2.jdbc.url.principal`    | `hive/<llap-headnode>@<AAD-Domain>` |
    
    * À partir d’un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, où CLUSTERNAME est le nom de votre cluster Interactive Query. Cliquez sur **HiveServer2 Interactive**. Vous verrez le nom de domaine complet (FQDN) du nœud principal sur lequel LLAP s’exécute, comme indiqué dans la capture d’écran. Remplacez `<llap-headnode>` par cette valeur.

        ![Nœud principal du connecteur d’entrepôt Hive](./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png)

    * Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster Interactive Query. Recherchez le paramètre `default_realm` dans le fichier `/etc/krb5.conf`. Remplacez `<AAD-DOMAIN>` par cette valeur sous forme de chaîne en majuscules, sinon les informations d’identification ne seront pas trouvées.

        ![Domaine AAD du connecteur d’entrepôt Hive](./media/apache-hive-warehouse-connector/aad-domain.png)

    * Par exemple, `hive/hn0-ng36ll.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET`.
    
1. Enregistrez les modifications et redémarrez les composants si nécessaire.

## <a name="hive-warehouse-connector-usage"></a>Utilisation de Hive Warehouse Connector

Vous pouvez choisir entre différentes méthodes pour vous connecter à votre cluster Interactive Query et exécuter des requêtes à l'aide du connecteur d’entrepôt Hive. Les méthodes prises en charge incluent les outils suivants :

* [Spark-shell / PySpark](../spark/apache-spark-shell.md)
* [Spark-submit](#spark-submit)
* [Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)


Vous trouverez ci-dessous des exemples de connexion à HWC à partir de Spark.

### <a name="spark-shell"></a>Spark-shell

1. Utilisez une [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster Apache Spark. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. À partir de votre session ssh, exécutez la commande suivante pour noter la version de `hive-warehouse-connector-assembly` :

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

1. Modifiez le code ci-dessous avec la version de `hive-warehouse-connector-assembly` identifiée ci-dessus. Exécutez ensuite la commande pour démarrer l’interpréteur de commandes Spark :

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. Après avoir démarré l’interpréteur de commandes Spark, une instance de Hive Warehouse Connector peut être démarrée à l’aide des commandes suivantes :

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

### <a name="spark-submit"></a>Spark-submit

Une fois que vous avez créé le code scala/java avec les dépendances dans un fichier jar d’assembly, utilisez la commande ci-dessous pour lancer une application Spark. Remplacez `<VERSION>` et `<APP_JAR_PATH>` par les valeurs réelles.

* Mode client YARN
    
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode client \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

* Mode de cluster YARN
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode cluster \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=true
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

Pour Python, ajoutez également la configuration suivante. 

```python
--py-files /usr/hdp/current/hive_warehouse_connector/pyspark_hwc-<VERSION>.zip
```
    
## <a name="run-queries-on-enterprise-security-package-esp-clusters"></a>Exécuter des requêtes sur des clusters Pack Sécurité Entreprise (ESP)

Utilisez `kinit` avant de lancer spark-shell ou spark-submit. Remplacez USERNAME par le nom d’un compte de domaine doté d’autorisations d’accès au cluster, puis exécutez la commande suivante :

```bash
kinit USERNAME
```

### <a name="securing-data-on-spark-esp-clusters"></a>Sécurisation des données sur les clusters ESP Spark

1. Créez une table `demo` avec quelques exemples de données en entrant les commandes suivantes :

    ```scala
    create table demo (name string);
    INSERT INTO demo VALUES ('HDinsight');
    INSERT INTO demo VALUES ('Microsoft');
    INSERT INTO demo VALUES ('InteractiveQuery');
    ```

1. Visualisez le contenu de la table avec la commande suivante. Avant l’application de la stratégie, la table `demo` affiche la colonne complète.

    ```scala
    hive.executeQuery("SELECT * FROM demo").show()
    ```

    ![table de démonstration avant application d’une stratégie Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-before-ranger-policy.png)

1. Appliquez une stratégie de masquage des colonnes qui n'affiche que les quatre derniers caractères de la colonne.  
    1. Accédez à l’interface utilisateur de l’administrateur Ranger à l’adresse `https://LLAPCLUSTERNAME.azurehdinsight.net/ranger/`.
    1. Cliquez sur le service Hive pour votre cluster sous **Hive**.
        ![Gestionnaire de service Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-service-manager.png)
    1. Cliquez sur l’onglet **Masquage**, puis sur **Ajouter une nouvelle stratégie**

        ![Connecteur de l’entrepôt Hive - Liste des stratégies Ranger Hive](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-hive-policy-list.png)

    1. Nommez la stratégie. Sélectionnez la base de données : **Default**, Table Hive : **demo**, Colonne Hive : **name**, Utilisateur : **rsadmin2**, Types d’accès : **select**, et **Partial mask: show last 4** dans le menu **Select Masking Option** (Sélectionner l’option de masquage). Cliquez sur **Add**.
                ![Créer une stratégie](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-create-policy.png)
1. Affichez à nouveau le contenu de la table. Après avoir appliqué la stratégie Ranger, nous ne voyons que les quatre derniers caractères de la colonne.

    ![table de démonstration après application d’une stratégie Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-after-ranger-policy.png)

## <a name="next-steps"></a>Étapes suivantes

* [Opérations HWC et Apache Spark](./apache-hive-warehouse-connector-operations.md)
* [Utiliser Interactive Query avec HDInsight](./apache-interactive-query-get-started.md)
* [Intégration de HWC à Apache Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)
* [Exemples d'interaction avec le connecteur d'entrepôt Hive en utilisant Zeppelin, Livy, spark-submit et pyspark](https://community.hortonworks.com/articles/223626/integrating-apache-hive-with-apache-spark-hive-war.html)
