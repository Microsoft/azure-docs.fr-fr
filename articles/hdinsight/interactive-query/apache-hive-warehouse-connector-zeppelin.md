---
title: Hive Warehouse Connector – Apache Zeppelin utilisant Livy – Azure HDInsight
description: Découvrez comment intégrer Hive Warehouse Connector avec Apache Zeppelin sur Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: bccf2b9a3dfe42ca439a45eb1e35cfaff58d0208
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426953"
---
# <a name="integrate-apache-zeppelin-with-hive-warehouse-connector-in-azure-hdinsight"></a>Intégrer Apache Zeppelin avec Hive Warehouse Connector dans Azure HDInsight

Les clusters HDInsight Spark incluent des blocs-notes Apache Zeppelin avec différents interpréteurs. Cet article se concentre uniquement sur l’interpréteur livy pour accéder aux tables Hive à partir de Spark à l’aide de Hive Warehouse Connector.

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

## <a name="prerequisite"></a>Configuration requise

Suivez les étapes de [Configuration de Hive Warehouse Connector](apache-hive-warehouse-connector.md#hive-warehouse-connector-setup).

## <a name="getting-started"></a>Prise en main

1. Utilisez une [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster Apache Spark. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. À partir de votre session SSH, exécutez la commande suivante afin de noter les versions pour `hive-warehouse-connector-assembly` et `pyspark_hwc` :

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

    Enregistrez la sortie en vue d’une utilisation ultérieure lors de la configuration d’Apache Zeppelin.

## <a name="configure-livy"></a>Configurer Livy

Les configurations suivantes sont requises pour accéder aux tables Hive à partir de Zeppelin avec l’interpréteur Livy.

### <a name="interactive-query-cluster"></a>Cluster Interactive Query

1. Dans un navigateur w, accédez à `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HDFS/configs`, où LLAPCLUSTERNAME est le nom de votre cluster interactive Query.

1. Accédez à **Avancé** > **Configuration core-site personnalisée**. Sélectionnez **ajouter une propriété...** pour ajouter les configurations suivantes :

    | Configuration                 | Valeur |
    | ----------------------------- |-------|
    | hadoop.proxyuser.livy.groups  | *     |
    | hadoop.proxyuser.livy.hosts   | *     |

1. Enregistrez les modifications, puis redémarrez tous les composants affectés.

### <a name="spark-cluster"></a>Cluster Spark

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, où CLUSTERNAME est le nom de votre cluster Apache Spark.

1. Développez **Custom livy2-conf**. Sélectionnez **Ajouter une propriété...** pour ajouter la configuration suivante :

    | Configuration                 | Valeur                                      |
    | ----------------------------- |------------------------------------------  |
    | livy.file.local-dir-whitelist | /usr/hdp/current/hive_warehouse_connector/ |

1. Enregistrez les modifications, puis redémarrez tous les composants affectés.

### <a name="configure-livy-interpreter-in-zeppelin-ui-spark-cluster"></a>Configurer l’interpréteur Livy dans l’interface utilisateur Zeppelin (cluster Spark)

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/zeppelin/#/interpreter`, où `CLUSTERNAME` est le nom de votre cluster Apache Spark.

1. Accédez à **livy2**.

1. Ajoutez les configurations suivantes :

    | Configuration                 | Valeur                                      |
    | ----------------------------- |:------------------------------------------:|
    | livy.spark.hadoop.hive.llap.daemon.service.hosts | @llap0 |
    | livy.spark.security.credentials.hiveserver2.enabled | true |
    | livy.spark.sql.hive.llap | true |
    | livy.spark.yarn.security.credentials.hiveserver2.enabled | true |
    | livy.superusers | livy,zeppelin |
    | livy.spark.jars | `file:///usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-VERSION.jar`.<br>Remplacez VERSION par la valeur que vous avez obtenue dans [Prise en main](#getting-started). |
    | livy.spark.submit.pyFiles | `file:///usr/hdp/current/hive_warehouse_connector/pyspark_hwc-VERSION.zip`.<br>Remplacez VERSION par la valeur que vous avez obtenue dans [Prise en main](#getting-started). |
    | livy.spark.sql.hive.hiveserver2.jdbc.url | Définissez l’URL JDBC interactive HiveServer2 du cluster interactive Query. |
    | spark.security.credentials.hiveserver2.enabled | true |

1. Pour les clusters ESP uniquement, ajoutez la configuration suivante :

    | Configuration| Valeur|
    |---|---|
    | livy.spark.sql.hive.hiveserver2.jdbc.url.principal | `hive/<llap-headnode>@<AAD-Domain>` |

    * À partir d’un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, où CLUSTERNAME est le nom de votre cluster Interactive Query. Cliquez sur **HiveServer2 Interactive**. Vous verrez le nom de domaine complet (FQDN) du nœud principal sur lequel LLAP s’exécute, comme indiqué dans la capture d’écran. Remplacez `<llap-headnode>` par cette valeur.

        ![Nœud principal du connecteur d’entrepôt Hive](./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png)

    * Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster Interactive Query. Recherchez le paramètre `default_realm` dans le fichier `/etc/krb5.conf`. Remplacez `<AAD-DOMAIN>` par cette valeur sous forme de chaîne en majuscules, sinon les informations d’identification ne seront pas trouvées.

        ![Domaine AAD du connecteur d’entrepôt Hive](./media/apache-hive-warehouse-connector/aad-domain.png)

    * Par exemple, `hive/hn0-ng36ll.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET`.

1. Enregistrez les modifications, puis redémarrez l’interpréteur Livy.

Si l’interpréteur livy n’est pas accessible, modifiez le fichier `shiro.ini` présent dans le composant Zeppelin dans Ambari. Pour plus d’informations, consultez [Configuration de la sécurité d’Apache Zeppelin](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.1/configuring-zeppelin-security/content/enabling_access_control_for_interpreter__configuration__and_credential_settings.html).  


## <a name="running-queries-in-zeppelin"></a>Exécution de requêtes dans Zeppelin 

Lancez un bloc-notes Zeppelin à l’aide de l’interpréteur Livy, puis exécutez la commande suivante

```python
%livy2

import com.hortonworks.hwc.HiveWarehouseSession
import com.hortonworks.hwc.HiveWarehouseSession._
import org.apache.spark.sql.SaveMode

# Initialize the hive context
val hive = HiveWarehouseSession.session(spark).build()

# Create a database
hive.createDatabase("hwc_db",true)
hive.setDatabase("hwc_db")

# Create a Hive table
hive.createTable("testers").ifNotExists().column("id", "bigint").column("name", "string").create()

val dataDF = Seq( (1, "foo"), (2, "bar"), (8, "john")).toDF("id", "name")

# Validate writes to the table
dataDF.write.format("com.hortonworks.spark.sql.hive.llap.HiveWarehouseConnector").mode("append").option("table", "hwc_db.testers").save()

# Validate reads
hive.executeQuery("select * from testers").show()

```

## <a name="next-steps"></a>Étapes suivantes

* [Opérations HWC et Apache Spark](./apache-hive-warehouse-connector-operations.md)
* [Intégration de HWC avec Apache Spark et Apache Hive](./apache-hive-warehouse-connector.md)
* [Utiliser Interactive Query avec HDInsight](./apache-interactive-query-get-started.md)
