---
title: Utiliser Livy Spark pour envoyer des travaux à un cluster Spark sur Azure HDInsight
description: Découvrez comment utiliser l’API REST Apache Spark pour envoyer des travaux Spark à distance à un cluster Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 02/28/2020
ms.openlocfilehash: fcb845904216fbe4cb05828877775ea2178c45e9
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539154"
---
# <a name="use-apache-spark-rest-api-to-submit-remote-jobs-to-an-hdinsight-spark-cluster"></a>Utiliser l’API REST Spark Apache pour envoyer des travaux à distance à un cluster Spark HDInsight

Découvrez comment utiliser [Apache Livy](https://livy.incubator.apache.org/), l’API REST Apache Spark servant à envoyer des tâches à distance à un cluster Azure HDInsight Spark. Consultez la documentation détaillée sur [Apache Livy](https://livy.incubator.apache.org/docs/latest/rest-api.html).

Vous pouvez utiliser Livy pour exécuter des interpréteurs de commandes Spark interactifs ou soumettre des traitements par lots à exécuter sur Spark. Cet article traite de l’utilisation de Livy pour soumettre des traitements par lots. Les extraits de code dans cet article utilisent cURL pour effectuer des appels d’API REST au point de terminaison Livy Spark.

## <a name="prerequisites"></a>Conditions préalables requises

Un cluster Apache Spark sur HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

## <a name="submit-an-apache-livy-spark-batch-job"></a>Envoyer un traitement par lots Apache Livy Spark

Avant de soumettre un traitement par lots, vous devez télécharger le fichier .jar d’application sur le stockage associé au cluster. Pour ce faire, vous pouvez utiliser l’utilitaire en ligne de commande [AzCopy](../../storage/common/storage-use-azcopy-v10.md). D’autres clients permettent également de charger des données. Pour en savoir plus à leur sujet, consultez [Chargement de données pour les tâches Apache Hadoop dans HDInsight](../hdinsight-upload-data.md).

```cmd
curl -k --user "admin:password" -v -H "Content-Type: application/json" -X POST -d '{ "file":"<path to application jar>", "className":"<classname in jar>" }' 'https://<spark_cluster_name>.azurehdinsight.net/livy/batches' -H "X-Requested-By: admin"
```

### <a name="examples"></a>Exemples

* Si le fichier .jar se trouve sur le stockage de cluster (WASBS)

    ```cmd  
    curl -k --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST -d '{ "file":"wasbs://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

* Si vous souhaitez transmettre le nom de fichier .jar et le nom de classe dans un fichier d’entrée (dans cet exemple, input.txt)

    ```cmd
    curl -k  --user "admin:mypassword1!" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://mysparkcluster.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

## <a name="get-information-on-livy-spark-batches-running-on-the-cluster"></a>Obtenir des informations sur des lots Livy Spark en cours d’exécution sur le cluster

Syntaxe :

```cmd
curl -k --user "admin:password" -v -X GET "https://<spark_cluster_name>.azurehdinsight.net/livy/batches"
```

### <a name="examples"></a>Exemples

* Si vous souhaitez récupérer tous les lots Livy Spark en cours d’exécution sur le cluster :

    ```cmd
    curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches"
    ```

* Si vous souhaitez récupérer un lot spécifique avec un ID de lot donné

    ```cmd
    curl -k --user "admin:mypassword1!" -v -X GET "https://mysparkcluster.azurehdinsight.net/livy/batches/{batchId}"
    ```

## <a name="delete-a-livy-spark-batch-job"></a>Supprimer un traitement par lots Livy Spark

```cmd
curl -k --user "admin:mypassword1!" -v -X DELETE "https://<spark_cluster_name>.azurehdinsight.net/livy/batches/{batchId}"
```

### <a name="example"></a>Exemple

Suppression d’un traitement par lots avec l’ID de lot `5`.

```cmd
curl -k --user "admin:mypassword1!" -v -X DELETE "https://mysparkcluster.azurehdinsight.net/livy/batches/5"
```

## <a name="livy-spark-and-high-availability"></a>Livy Spark et haute disponibilité

Livy assure une haute disponibilité des travaux Spark exécutés sur le cluster. Voici quelques exemples.

* Si le service Livy tombe en panne après avoir soumis un travail à distance à un cluster Spark, l’exécution du travail se poursuit en arrière-plan. La sauvegarde de Livy entraîne la restauration de l’état du travail et la création d’un rapport à ce sujet.
* Les blocs-notes Jupyter pour HDInsight sont alimentés par Livy sur le serveur principal. Si un bloc-notes exécute un travail Spark et que le service Livy est redémarré, le bloc-notes poursuit l’exécution des cellules de code.

## <a name="show-me-an-example"></a>Afficher un exemple

Dans cette section, nous examinons des exemples d’utilisation de Livy Spark pour envoyer un traitement par lots, surveiller l’avancement du travail, puis supprimer celui-ci. L’application que nous utilisons dans cet exemple est décrite dans l’article [Créer une application Scala autonome et l’exécuter sur un cluster HDInsight Spark](apache-spark-create-standalone-application.md). Les étapes indiquées ici supposent que :

* Vous avez déjà copié le fichier .jar de l’application dans le compte de stockage associé au cluster.
* CuRL est installé sur l’ordinateur sur lequel vous effectuez la procédure.

Procédez comme suit :

1. Pour faciliter l’utilisation, définissez les variables d’environnement. Cet exemple est fondé sur un environnement Windows ; vous devez modifier les variables en fonction des besoins de votre environnement. Remplacez `CLUSTERNAME` et `PASSWORD` par les valeurs appropriées.

    ```cmd
    set clustername=CLUSTERNAME
    set password=PASSWORD
    ```

1. Vérifiez tout d’abord que Livy Spark est en cours d’exécution sur le cluster. Pour ce faire, nous pouvons obtenir une liste des lots en cours d’exécution. Si vous exécutez un travail à l’aide de Livy pour la première fois, la valeur retournée doit être 0.

    ```cmd
    curl -k --user "admin:%password%" -v -X GET "https://%clustername%.azurehdinsight.net/livy/batches"
    ```

    La sortie doit ressembler à l’extrait de code suivant :

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:47:53 GMT
    < Content-Length: 34
    <
    {"from":0,"total":0,"sessions":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Notez la dernière ligne qui indique **total:0** , ce qui suggère qu’aucun lot n’est en cours d’exécution.

1. Soumettons à présent un traitement par lots. L’extrait de code suivant utilise un fichier d’entrée (input.txt) pour transmettre le nom du fichier .jar et le nom de classe en tant que paramètres. Si vous exécutez ces étapes sur un ordinateur Windows, utiliser un fichier d’entrée est l’approche recommandée.

    ```cmd
    curl -k --user "admin:%password%" -v -H "Content-Type: application/json" -X POST --data @C:\Temp\input.txt "https://%clustername%.azurehdinsight.net/livy/batches" -H "X-Requested-By: admin"
    ```

    Les paramètres du fichier **input.txt** sont définis comme suit :

    ```text
    { "file":"wasbs:///example/jars/SparkSimpleApp.jar", "className":"com.microsoft.spark.example.WasbIOTest" }
    ```

    Le résultat doit ressembler à l’extrait de code suivant :

    ```output
    < HTTP/1.1 201 Created
    < Content-Type: application/json; charset=UTF-8
    < Location: /0
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:51:30 GMT
    < Content-Length: 36
    <
    {"id":0,"state":"starting","log":[]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    Notez la dernière ligne de la sortie qui indique **state:starting** . Elle indique également **id:0** . Ici, **0** correspond à l’ID du lot.

1. Vous pouvez maintenant récupérer l’état de ce lot à l’aide de l’ID correspondant.

    ```cmd
    curl -k --user "admin:%password%" -v -X GET "https://%clustername%.azurehdinsight.net/livy/batches/0"
    ```

    Le résultat doit ressembler à l’extrait de code suivant :

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Fri, 20 Nov 2015 23:54:42 GMT
    < Content-Length: 509
    <
    {"id":0,"state":"success","log":["\t diagnostics: N/A","\t ApplicationMaster host: 10.0.0.4","\t ApplicationMaster RPC port: 0","\t queue: default","\t start time: 1448063505350","\t final status: SUCCEEDED","\t tracking URL: http://myspar.lpel.jx.internal.cloudapp.net:8088/proxy/application_1447984474852_0002/","\t user: root","15/11/20 23:52:47 INFO Utils: Shutdown hook called","15/11/20 23:52:47 INFO Utils: Deleting directory /tmp/spark-b72cd2bf-280b-4c57-8ceb-9e3e69ac7d0c"]}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    La sortie indique maintenant **state:success** , ce qui suggère que le travail a été exécuté correctement.

1. Si vous le souhaitez, vous pouvez maintenant supprimer le lot.

    ```cmd
    curl -k --user "admin:%password%" -v -X DELETE "https://%clustername%.azurehdinsight.net/livy/batches/0"
    ```

    Le résultat doit ressembler à l’extrait de code suivant :

    ```output
    < HTTP/1.1 200 OK
    < Content-Type: application/json; charset=UTF-8
    < Server: Microsoft-IIS/8.5
    < X-Powered-By: ARR/2.5
    < X-Powered-By: ASP.NET
    < Date: Sat, 21 Nov 2015 18:51:54 GMT
    < Content-Length: 17
    <
    {"msg":"deleted"}* Connection #0 to host mysparkcluster.azurehdinsight.net left intact
    ```

    La dernière ligne de la sortie indique que le lot a été supprimé. Supprimer un travail alors qu’il est en cours d’exécution tue également le travail. Si vous supprimez un travail qui s’est terminé, les informations du travail sont entièrement supprimées.

## <a name="updates-to-livy-configuration-starting-with-hdinsight-35-version"></a>Mises à jour de la configuration de Livy depuis la version 3.5 de HDInsight

Par défaut, un cluster HDInsight 3.5 ou version supérieure, désactive l’utilisation de chemins locaux pour accéder à des fichiers de données ou des fichiers JAR. Nous vous conseillons plutôt d'utiliser le chemin `wasbs://` pour accéder aux fichiers JAR ou aux exemples de fichiers de données à partir du cluster.

## <a name="submitting-livy-jobs-for-a-cluster-within-an-azure-virtual-network"></a>Envoi de travaux Livy pour un cluster dans un réseau virtuel Azure

Si vous vous connectez à un cluster HDInsight Spark à partir d’un réseau virtuel Azure, vous pouvez vous connecter directement à Livy sur le cluster. Dans ce cas, l’URL du point de terminaison Livy est `http://<IP address of the headnode>:8998/batches`. Ici, **8998** est le port sur lequel Livy s’exécute sur le nœud principal du cluster. Pour plus d’informations sur l’accès aux services sur des ports non publics, consultez [Ports utilisés par les services Apache Hadoop sur HDInsight](../hdinsight-hadoop-port-settings-for-services.md).

## <a name="next-steps"></a>Étapes suivantes

* [Documentation de l’API REST Apache Livy](https://livy.incubator.apache.org/docs/latest/rest-api.html)
* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Suivi et débogage des tâches en cours d’exécution sur un cluster Apache Spark dans HDInsight)](apache-spark-job-debugging.md)