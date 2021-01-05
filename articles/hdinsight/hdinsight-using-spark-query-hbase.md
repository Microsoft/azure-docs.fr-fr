---
title: Utiliser Spark pour lire et écrire des données HBase - Azure HDInsight
description: Utilisez le connecteur HBase Spark pour lire et écrire des données d’un cluster Spark sur un cluster HBase.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 08/12/2020
ms.openlocfilehash: eb62cf099d7ccc133a207a843a8be3debf5c5454
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91308416"
---
# <a name="use-apache-spark-to-read-and-write-apache-hbase-data"></a>Utiliser Apache Spark pour lire et écrire des données Apache HBase

Apache HBase est généralement interrogé soit avec son API de bas niveau (analyses, obtentions et insertions), soit avec une syntaxe SQL en utilisant Apache Phoenix. Apache fournit également le connecteur Apache Spark HBase. Le connecteur constitue une alternative pratique et efficace pour interroger et modifier des données stockées par HBase.

## <a name="prerequisites"></a>Prérequis

* Deux clusters HDInsight distincts déployés dans le même [réseau virtuel](./hdinsight-plan-virtual-network-deployment.md). Un cluster HBase et un cluster Spark avec Spark 2.1 (HDInsight 3.6) minimum installé. Pour plus d’informations, consultez [Créer des clusters Linux dans HDInsight à l’aide du portail Azure](hdinsight-hadoop-create-linux-clusters-portal.md).

* Le schéma d'URI de votre principal espace de stockage de clusters. Ce schéma serait wasb:// pour le Stockage Blob Azure, `abfs://` pour Azure Data Lake Storage Gen2 ou adl:// pour Azure Data Lake Storage Gen1. Si le transfert sécurisé est activé pour le stockage Blob, l’URI sera `wasbs://`.  Voir aussi [transfert sécurisé](../storage/common/storage-require-secure-transfer.md).

## <a name="overall-process"></a>Procédure générale

La procédure à suivre pour permettre à votre cluster Spark d’interroger votre cluster HBase est la suivante :

1. Préparer des exemples de données dans HBase.
2. Procurez-vous le fichier hbase-site.xml à partir de votre dossier de configuration de cluster HBase (/etc/HBase/conf), puis placez une copie de hbase-site.xml dans votre dossier de configuration Spark 2 (/etc/spark2/conf). (FACULTATIF : utilisez le script fourni par l’équipe HDInsight pour automatiser ce processus)
4. Exécuter `spark-shell` en référençant le connecteur HBase Spark par ses coordonnées Maven dans l’option `packages`.
5. Définir un catalogue qui mappe le schéma de Spark vers HBase.
6. Interagir avec les données HBase à l’aide des API RDD ou DataFrame.

## <a name="prepare-sample-data-in-apache-hbase"></a>Préparer des exemples de données dans Apache HBase

Dans cette étape, vous créez et vous remplissez un tableau dans Apache HBase, que vous pouvez ensuite interroger avec Spark.

1. Utilisez la commande `ssh` pour vous connecter à votre cluster HBase. Modifiez la commande ci-dessous en remplaçant `HBASECLUSTER` par le nom de votre cluster HBase, puis entrez la commande :

    ```cmd
    ssh sshuser@HBASECLUSTER-ssh.azurehdinsight.net
    ```

2. Utilisez la commande `hbase shell` pour démarrer l’interpréteur de commandes interactif HBase. Entrez la commande suivante dans votre connexion SSH :

    ```bash
    hbase shell
    ```

3. Utilisez la commande `create` pour créer une table HBase avec deux familles de colonnes. Entrez la commande suivante :

    ```hbase
    create 'Contacts', 'Personal', 'Office'
    ```

4. Utilisez la commande `put` pour insérer des valeurs dans une colonne et sur une ligne spécifiées d’une table particulière. Entrez la commande suivante :

    ```hbase
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
    put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'
    ```

5. Utilisez la commande `exit` pour arrêter l’interpréteur de commandes interactif HBase. Entrez la commande suivante :

    ```hbase
    exit
    ```
    
## <a name="run-scripts-to-set-up-connection-between-clusters"></a>Exécuter des scripts pour configurer la connexion entre les clusters

Pour configurer la communication entre les clusters, suivez les étapes ci-dessous pour exécuter deux scripts sur vos clusters. Ces scripts automatisent le processus de copie des fichiers décrit dans la section « Configurer la communication manuellement » ci-dessous. 

* Le script que vous exécutez à partir du cluster HBase charge `hbase-site.xml` et les informations de mappage IP HBase sur le stockage par défaut attaché à votre cluster Spark. 
* Le script que vous exécutez à partir du cluster Spark configure deux tâches cron pour exécuter régulièrement deux scripts d’assistance :  
    1.  Tâche cron HBase : télécharger de nouveaux fichiers `hbase-site.xml` et le mappage IP HBase à partir du compte de stockage par défaut Spark vers le nœud local
    2.  Tâche cron Spark : vérifie si une mise à l’échelle Spark s’est produite et si le cluster est sécurisé. Dans ce cas, modifiez `/etc/hosts` pour inclure le mappage IP HBase stocké localement

__REMARQUE__ : Avant de continuer, vérifiez que vous avez ajouté le compte de stockage du cluster Spark à votre cluster HBase en tant que compte de stockage secondaire. Assurez-vous que les scripts sont dans l’ordre indiqué ci-dessous.


1. Utilisez une [Action de script](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) sur votre cluster HBase pour appliquer les modifications avec les considérations suivantes : 


    |Propriété | Valeur |
    |---|---|
    |URI de script bash|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-hbase.sh`|
    |Type(s) de nœud|Région|
    |Paramètres|`-s SECONDARYS_STORAGE_URL`|
    |Persistant|Oui|

    * `SECONDARYS_STORAGE_URL` est l’URL du stockage par défaut côté Spark. Exemple de paramètre : `-s wasb://sparkcon-2020-08-03t18-17-37-853z@sparkconhdistorage.blob.core.windows.net`


2.  Utilisez une Action de script sur votre cluster Spark pour appliquer les modifications avec les considérations suivantes :

    |Propriété | Valeur |
    |---|---|
    |URI de script bash|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-spark.sh`|
    |Type(s) de nœud|Head, Worker, Zookeeper|
    |Paramètres|`-s "SPARK-CRON-SCHEDULE"` (facultatif) `-h "HBASE-CRON-SCHEDULE"` (facultatif)|
    |Persistant|Oui|


    * Vous pouvez spécifier la fréquence à laquelle ce cluster vérifie automatiquement si la mise à jour est nécessaire. Par défaut : -s “*/1 * * * *” -h 0 (dans cet exemple, le cron Spark s’exécute toutes les minutes, tandis que le cron HBase ne s’exécute pas)
    * Étant donné que le cron HBase n’est pas configuré par défaut, vous devez exécuter à nouveau ce script lors de l’exécution de la mise à l’échelle de votre cluster HBase. Si votre cluster HBase est souvent mis à l’échelle, vous pouvez choisir de configurer la tâche cron HBase automatiquement. Par exemple : `-h "*/30 * * * *"` configure le script pour effectuer des vérifications toutes les 30 minutes. Cette opération exécute régulièrement la planification cron HBase pour automatiser le téléchargement des nouvelles informations de HBase sur le compte de stockage commun vers le nœud local.
    
    

## <a name="set-up-communication-manually-optional-if-provided-script-in-above-step-fails"></a>Configurer la communication manuellement (facultatif, si le script fourni dans l’étape ci-dessus échoue)

__REMARQUE :__ Ces étapes doivent être effectuées chaque fois que l’un des clusters subit une activité de mise à l’échelle.

1. Copiez le fichier hbase-site.xml du stockage local vers la racine du stockage par défaut de votre cluster Spark.  Modifiez la commande ci-dessous pour l’adapter à votre configuration.  Ensuite, entrez la commande suivante depuis votre session SSH ouverte dans le cluster HBase :

    | Valeur de la syntaxe | Nouvelle valeur|
    |---|---|
    |[Schéma d’URI](hdinsight-hadoop-linux-information.md#URI-and-scheme) | Modifiez-la pour l’adapter à votre stockage.  La syntaxe ci-dessous est pour le stockage d’objets blob doté du transfert sécurisé.|
    |`SPARK_STORAGE_CONTAINER`|Remplacez par le nom du conteneur de stockage par défaut utilisé pour le cluster Spark.|
    |`SPARK_STORAGE_ACCOUNT`|Remplacez par le nom du compte de stockage par défaut utilisé pour le cluster Spark.|

    ```bash
    hdfs dfs -copyFromLocal /etc/hbase/conf/hbase-site.xml wasbs://SPARK_STORAGE_CONTAINER@SPARK_STORAGE_ACCOUNT.blob.core.windows.net/
    ```

2. Mettez ensuite fin à votre connexion SSH à votre cluster HBase.

    ```bash
    exit
    ```


3. Connectez-vous au nœud principal de votre cluster Spark à l’aide de SSH. Modifiez la commande ci-dessous en remplaçant `SPARKCLUSTER` par le nom de votre cluster Spark, puis entrez la commande :

    ```cmd
    ssh sshuser@SPARKCLUSTER-ssh.azurehdinsight.net
    ```

4. Entrez la commande ci-dessous pour copier `hbase-site.xml` du stockage par défaut de votre cluster Spark vers le dossier de configuration Spark 2 sur le stockage local du cluster :

    ```bash
    sudo hdfs dfs -copyToLocal /hbase-site.xml /etc/spark2/conf
    ```

## <a name="run-spark-shell-referencing-the-spark-hbase-connector"></a>Exécuter l’interpréteur de commandes Spark en référençant le connecteur HBase Spark

Une fois l’étape précédente terminée, vous devriez être en mesure d’exécuter Spark Shell, en référençant la version appropriée du connecteur Spark HBase. Pour trouver la version principale la plus récente du connecteur Spark HBase qui soit adaptée à votre scénario de cluster, consultez [SHC Core Repository](https://repo.hortonworks.com/content/groups/public/com/hortonworks/shc/shc-core/).

Par exemple, le tableau suivant liste deux versions, ainsi que les commandes correspondantes actuellement utilisées par l’équipe HDInsight. Vous pouvez utiliser les mêmes versions pour vos clusters si les versions de HBase et Spark sont identiques à celles indiquées dans le tableau. 


1. Dans votre session SSH ouverte sur le cluster Spark, entrez la commande suivante pour démarrer un interpréteur de commandes Spark :

    |Version de Spark| Version HDI HBase  | Version SHC    |  Commande  |
    | :-----------:| :----------: | :-----------: |:----------- |
    |      2.1    | HDI 3.6 (HBase 1.1) | 1.1.1-2.1-s_2.11    | `spark-shell --packages com.hortonworks:shc-core:1.1.1-2.1-s_2.11 --repositories https://repo.hortonworks.com/content/groups/public/` |
    |      2.4    | HDI 4.0 (HBase 2.0) | 1.1.0.3.1.2.2-1  | `spark-shell --packages com.hortonworks.shc:shc-core:1.1.0.3.1.2.2-1 --repositories http://repo.hortonworks.com/content/groups/public/` |

2. Laissez cette instance de l'interpréteur de commandes Spark ouverte et passez à l'étape [Définir un catalogue et l'interroger](#define-a-catalog-and-query). Si vous ne trouvez pas les fichiers .jar correspondant à vos versions dans le référentiel SHC Core, poursuivez votre lecture. 

Vous pouvez créer les fichiers .jar à partir de la branche GitHub [spark-hbase-connector](https://github.com/hortonworks-spark/shc). Par exemple, si vous utilisez Spark 2.3 et HBase 1.1, procédez comme suit :

1. Clonez le référentiel :

    ```bash
    git clone https://github.com/hortonworks-spark/shc
    ```
    
2. Accédez à la branche-2.3 :

    ```bash
    git checkout branch-2.3
    ```

3. Créez à partir de la branche (crée un fichier .jar) :

    ```bash
    mvn clean package -DskipTests
    ```
    
3. Exécutez la commande suivante (veillez à modifier le nom .jar qui correspond au fichier .jar que vous avez créé) :

    ```bash
    spark-shell --jars <path to your jar>,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar,/usr/hdp/current/hbase-client/lib/hbase-client.jar,/usr/hdp/current/hbase-client/lib/hbase-common.jar,/usr/hdp/current/hbase-client/lib/hbase-server.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol.jar,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar
    ```
    
4. Laissez cette instance de l'interpréteur de commandes Spark ouverte et passez à la section suivante. 



## <a name="define-a-catalog-and-query"></a>Définir un catalogue et l'interroger

Dans cette étape, vous définissez un objet de catalogue qui mappe le schéma depuis Apache Spark vers Apache HBase.  

1. Dans votre interpréteur de commandes Spark ouvert, entrez les instructions `import` suivantes :

    ```scala
    import org.apache.spark.sql.{SQLContext, _}
    import org.apache.spark.sql.execution.datasources.hbase._
    import org.apache.spark.{SparkConf, SparkContext}
    import spark.sqlContext.implicits._
    ```  

1. Entrez la commande ci-dessous pour définir un catalogue pour la table Contacts que vous avez créée dans HBase :

    ```scala
    def catalog = s"""{
        |"table":{"namespace":"default", "name":"Contacts"},
        |"rowkey":"key",
        |"columns":{
        |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
        |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
        |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
        |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
        |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
        |}
    |}""".stripMargin
    ```

    Le code :  

    1. définit un schéma de catalogue pour la table HBase nommée `Contacts`.  
    1. Identifie `key` comme RowKey et mappe les noms de colonnes utilisés dans Spark à la famille de colonne, au nom de colonne et au type de colonne utilisés dans HBase.  
    1. Définit le RowKey en détail comme une colonne nommée (`rowkey`), qui a une famille de colonne `cf` de `rowkey`.  

1. Entrez la commande ci-dessous pour définir une méthode qui fournit un DataFrame autour de votre table `Contacts` dans HBase :

    ```scala
    def withCatalog(cat: String): DataFrame = {
        spark.sqlContext
        .read
        .options(Map(HBaseTableCatalog.tableCatalog->cat))
        .format("org.apache.spark.sql.execution.datasources.hbase")
        .load()
     }
    ```

1. Créez une instance du DataFrame :

    ```scala
    val df = withCatalog(catalog)
    ```  

1. Interrogez le DataFrame :

    ```scala
    df.show()
    ```

    Vous devez voir deux lignes de données :

    ```output
    +------+--------------------+--------------+-------------+--------------+
    |rowkey|       officeAddress|   officePhone| personalName| personalPhone|
    +------+--------------------+--------------+-------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|    John Dole|1-425-000-0001|
    |  8396|5415 San Gabriel Dr.|  230-555-0191|  Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+-------------+--------------+
    ```

1. Inscrivez une table temporaire afin de pouvoir interroger la table HBase à l’aide de Spark SQL :

    ```scala
    df.createTempView("contacts")
    ```

1. Émettez une requête SQL par rapport à la table `contacts` :

    ```scala
    spark.sqlContext.sql("select personalName, officeAddress from contacts").show
    ```

    Les résultats doivent ressembler à ceci :

    ```output
    +-------------+--------------------+
    | personalName|       officeAddress|
    +-------------+--------------------+
    |    John Dole|1111 San Gabriel Dr.|
    |  Calvin Raji|5415 San Gabriel Dr.|
    +-------------+--------------------+
    ```

## <a name="insert-new-data"></a>Insérer de nouvelles données

1. Pour insérer un nouvel enregistrement Contact, définissez une classe `ContactRecord` :

    ```scala
    case class ContactRecord(
        rowkey: String,
        officeAddress: String,
        officePhone: String,
        personalName: String,
        personalPhone: String
        )
    ```

1. Créez une instance de `ContactRecord` et placez-la dans un tableau :

    ```scala
    val newContact = ContactRecord("16891", "40 Ellis St.", "674-555-0110", "John Jackson","230-555-0194")

    var newData = new Array[ContactRecord](1)
    newData(0) = newContact
    ```

1. Enregistrez le tableau de nouvelles données dans HBase :

    ```scala
    sc.parallelize(newData).toDF.write.options(Map(HBaseTableCatalog.tableCatalog -> catalog, HBaseTableCatalog.newTable -> "5")).format("org.apache.spark.sql.execution.datasources.hbase").save()
    ```

1. Examinez les résultats :

    ```scala  
    df.show()
    ```

    Un résultat similaire à ceci s’affiche normalement :

    ```output
    +------+--------------------+--------------+------------+--------------+
    |rowkey|       officeAddress|   officePhone|personalName| personalPhone|
    +------+--------------------+--------------+------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
    | 16891|        40 Ellis St.|  674-555-0110|John Jackson|  230-555-0194|
    |  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+------------+--------------+
    ```

1. Fermez l’interpréteur de commandes Spark en entrant la commande suivante :

    ```scala
    :q
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Connecteur Apache Spark HBase](https://github.com/hortonworks-spark/shc)
