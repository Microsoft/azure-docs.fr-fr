---
title: Blocs-notes Zeppelin et cluster Apache Spark - Azure HDInsight
description: Instructions détaillées sur l’utilisation des blocs-notes Zeppelin avec les clusters Apache Spark sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: a692f4dd86d110f7f0a91a862a7b16ac28345de5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084526"
---
# <a name="use-apache-zeppelin-notebooks-with-apache-spark-cluster-on-azure-hdinsight"></a>Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight

Les clusters HDInsight Spark incluent des blocs-notes [Apache Zeppelin](https://zeppelin.apache.org/). Utilisez les notebooks pour exécuter des travaux Apache Spark. Dans cet article, vous allez apprendre à utiliser le bloc-notes Zeppelin sur un cluster HDInsight.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Spark sur HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).
* Le schéma d'URI de votre principal espace de stockage de clusters. Il peut s’agir de `wasb://` pour Stockage Blob Azure, de `abfs://` pour Azure Data Lake Storage Gen2 ou de `adl://` pour Azure Data Lake Storage Gen1. Si le transfert sécurisé est activé pour le stockage Blob, l’URI sera `wasbs://`.  Pour plus d’informations, voir [Exiger un transfert sécurisé dans Stockage Azure](../../storage/common/storage-require-secure-transfer.md).

## <a name="launch-an-apache-zeppelin-notebook"></a>Lancer un bloc-notes Apache Zeppelin

1. Dans la section **Vue d'ensemble** du cluster Spark, accédez à **Tableaux de bord du cluster** et sélectionnez **Bloc-notes Zeppelin**. Entrez les informations d'identification de l'administrateur du cluster.  

   > [!NOTE]  
   > Vous pouvez également atteindre le bloc-notes Zeppelin pour votre cluster en ouvrant l'URL suivante dans votre navigateur. Remplacez **CLUSTERNAME** par le nom de votre cluster.
   >
   > `https://CLUSTERNAME.azurehdinsight.net/zeppelin`

2. Créer un nouveau bloc-notes. Dans le volet d'en-tête, accédez à **Bloc-notes** > **Créer une note**.

    ![Créer un notebook Zeppelin](./media/apache-spark-zeppelin-notebook/hdinsight-create-zeppelin-notebook.png "Créer un nouveau bloc-notes Zeppelin")

    Entrez un nom pour le bloc-notes, puis sélectionnez **Créer une note**.

3. Vérifiez que l'en-tête du bloc-notes indique un état connecté. Il est indiqué par un point vert dans le coin supérieur droit.

    ![État du notebook Zeppelin](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-connected.png "État du bloc-notes Zeppelin")

4. Chargez un exemple de données dans une table temporaire. Lorsque vous créez un cluster Spark dans HDInsight, l'exemple de fichier de données `hvac.csv` est copié sur le compte de stockage associé, sous `\HdiSamples\SensorSampleData\hvac`.

    Collez l’extrait suivant dans le paragraphe vide créé par défaut dans le nouveau bloc-notes.

    ```scala
    %livy2.spark
    //The above magic instructs Zeppelin to use the Livy Scala interpreter

    // Create an RDD using the default Spark context, sc
    val hvacText = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

    // Define a schema
    case class Hvac(date: String, time: String, targettemp: Integer, actualtemp: Integer, buildingID: String)

    // Map the values in the .csv file to the schema
    val hvac = hvacText.map(s => s.split(",")).filter(s => s(0) != "Date").map(
        s => Hvac(s(0),
                s(1),
                s(2).toInt,
                s(3).toInt,
                s(6)
        )
    ).toDF()

    // Register as a temporary table called "hvac"
    hvac.registerTempTable("hvac")
    ```

    Appuyez sur **MAJ + ENTRÉE** ou sélectionnez le bouton **Lire** pour que le paragraphe exécute l’extrait de code. L’état indiqué dans le coin supérieur droit du paragraphe doit progresser de READY, PENDING, RUNNING à FINISHED. Le résultat s’affiche au bas du même paragraphe. La capture d’écran ressemble à l’image suivante :

    ![Créer une table temporaire à partir de données brutes](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-load-data.png "Créer une table temporaire à partir de données brutes")

    Vous pouvez également indiquer un titre pour chaque paragraphe. Dans le coin droit du paragraphe, sélectionnez l'icône **Paramètres** (pignon), puis sélectionnez **Afficher le titre**.  

    > [!NOTE]  
    > L’interpréteur %spark2 n’est pas pris en charge dans les bloc-notes Zeppelin sur toutes les versions HDInsight et l’interpréteur %sh ne sera pas pris en charge à partir de HDInsight 4.0 et versions ultérieures.

5. Vous pouvez maintenant exécuter des instructions Spark SQL dans la table `hvac`. Collez la requête suivante dans un nouveau paragraphe. La requête récupère l’ID de bâtiment. Elle récupère également la différence entre les températures cible et réelle pour chaque bâtiment à une date donnée. Appuyez sur **MAJ + ENTRÉE**.

    ```sql
    %sql
    select buildingID, (targettemp - actualtemp) as temp_diff, date from hvac where date = "6/1/13"
    ```  

    L’instruction **%sql** du début demande au bloc-notes d’utiliser l’interpréteur Livy Scala.

6. Sélectionnez l'icône **Graphique à barres** pour modifier l'affichage.  **paramètres**, apparaît une fois que vous avez sélectionné **Graphique à barres**, vous permet de choisir des **Clés** et des **Valeurs**.  La capture d’écran qui suit présente le résultat.

    ![Exécuter une instruction Spark SQL à l’aide du notebook1](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-1.png "Exécuter une instruction Spark SQL à l’aide du notebook1")

7. Vous pouvez également exécuter des instructions Spark SQL à l’aide de variables dans la requête. L'extrait suivant montre comment définir la variable `Temp` dans la requête avec les valeurs d'interrogation possibles. Lors de la première exécution de la requête, une liste déroulante est automatiquement renseignée avec les valeurs que vous avez spécifiées pour la variable.

    ```sql
    %sql  
    select buildingID, date, targettemp, (targettemp - actualtemp) as temp_diff from hvac where targettemp > "${Temp = 65,65|75|85}"
    ```

    Collez cet extrait dans un nouveau paragraphe, puis appuyez sur **MAJ + ENTRÉE**. Sélectionnez ensuite **65** dans la liste déroulante **Temp**.

8. Sélectionnez l'icône **Graphique à barres** pour modifier l'affichage.  Sélectionnez ensuite **paramètres** et apportez les modifications suivantes :

   * **Groupes :**  Ajoutez **targettemp**.  
   * **Valeurs :** 1. Supprimez **date**.  2. Ajoutez **temp_diff**.  3.  Remplacez l'agrégateur **SUM** par **AVG**.  

     La capture d’écran qui suit présente le résultat.

     ![Exécuter une instruction Spark SQL à l’aide du notebook2](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-spark-query-2.png "Exécuter une instruction Spark SQL à l’aide du notebook2")

## <a name="how-do-i-use-external-packages-with-the-notebook"></a>Comment utiliser des packages externes avec le bloc-notes ?

Un bloc-notes Zeppelin dans un cluster Apache Spark sur HDInsight peut utiliser des packages externes bénéficiant de la contribution de la communauté, qui ne sont pas inclus dans le cluster. Recherchez le [référentiel Maven](https://search.maven.org/) pour obtenir la liste complète des packages disponibles. Vous pouvez également obtenir une liste des packages disponibles à partir d’autres sources. Par exemple, une liste complète des packages bénéficiant de la contribution de la communauté est disponible sur le site [Spark Packages](https://spark-packages.org/)(Packages Spark).

Dans cet article, vous allez apprendre à utiliser le package [spark-csv](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) avec le bloc-notes Jupyter.

1. Ouvrez les paramètres de l’interpréteur. Dans le coin supérieur droit, sélectionnez le nom de l'utilisateur connecté, puis **Interpréteur**.

    ![Lancer l’interpréteur](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Sortie Hive")

2. Accédez à **livy2**, puis sélectionnez **modifier**.

    ![Modifier les paramètres de l’interpréteur1](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-1.png "Modifier les paramètres de l’interpréteur1")

3. Accédez à la clé `livy.spark.jars.packages` et définissez sa valeur au format `group:id:version`. Par conséquent, si vous souhaitez utiliser le package [spark-csv](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar), vous devez définir la valeur de la clé sur `com.databricks:spark-csv_2.10:1.4.0`.

    ![Modifier les paramètres de l’interpréteur2](./media/apache-spark-zeppelin-notebook/zeppelin-use-external-package-2.png "Modifier les paramètres de l’interpréteur2")

    Sélectionnez **Enregistrer**, puis **OK** pour redémarrer l’interpréteur Livy.

4. si vous souhaitez savoir comment atteindre la valeur de la clé entrée ci-dessus, lisez ce qui suit.

    a. Recherchez le package dans le référentiel Maven. Dans cet article, nous avons utilisé [spark-csv](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar).

    b. À partir du référentiel, rassemblez les valeurs pour **GroupId**, **ArtifactId** et **Version**.

    ![Utiliser des packages externes avec les notebooks Jupyter](./media/apache-spark-zeppelin-notebook/use-external-packages-with-jupyter.png "Utiliser des packages externes avec les blocs-notes Jupyter")

    c. Concaténez les trois valeurs séparées par deux-points ( **:** ).

    ```
    com.databricks:spark-csv_2.10:1.4.0
    ```

## <a name="where-are-the-zeppelin-notebooks-saved"></a>Où les blocs-notes Zeppelin sont-ils enregistrés ?

Les blocs-notes Zeppelin sont enregistrés dans les nœuds principaux du cluster. Par conséquent, si vous supprimez le cluster, les blocs-notes seront aussi supprimés. Si vous souhaitez conserver vos blocs-notes pour une utilisation ultérieure sur les autres clusters, vous devez les exporter après avoir terminé l’exécution des tâches. Pour exporter un bloc-notes, sélectionnez l'icône **Exporter** comme illustré sur l'image ci-dessous.

![Télécharger le notebook](./media/apache-spark-zeppelin-notebook/zeppelin-download-notebook.png "Télécharger le notebook")

Cette action enregistre le bloc-notes en tant que fichier JSON dans votre emplacement de téléchargement.

## <a name="use-shiro-to-configure-access-to-zeppelin-interpreters-in-enterprise-security-package-esp-clusters"></a>Utiliser `Shiro` pour configurer l’accès aux interpréteurs Zeppelin dans les clusters Pack Sécurité Entreprise (ESP)

Comme indiqué ci-dessus, l’interpréteur `%sh` n’est plus pris en charge à partir de HDInsight 4.0. En outre, puisque l’interpréteur `%sh` introduit des problèmes de sécurité potentiels, tels que l’accès à des keytabs à l’aide de commandes shell, il a également été supprimé des clusters ESP de HDInsight 3.6. Cela signifie que l’interpréteur `%sh` n’est pas disponible lorsque vous cliquez sur **Créer une note** ni dans l’interface utilisateur par défaut de l’interpréteur.

Les utilisateurs privilégiés de domaine peuvent utiliser le fichier `Shiro.ini` pour contrôler l’accès à l’interface utilisateur de l’interpréteur. Seuls ces utilisateurs peuvent créer des interpréteurs `%sh` et définir des autorisations sur chaque nouvel interpréteur `%sh`. Pour contrôler l’accès à l’aide du fichier `shiro.ini`, procédez comme suit :

1. Définissez un nouveau rôle à l’aide d’un nom de groupe de domaine existant. Dans l’exemple suivant, `adminGroupName` est un groupe d’utilisateurs privilégiés dans AAD. N’utilisez pas de caractères spéciaux ni d’espaces blancs dans le nom de groupe. Les caractères situés après `=` accordent les autorisations pour ce rôle. `*` signifie que le groupe dispose de toutes les autorisations.

    ```
    [roles]
    adminGroupName = *
    ```

2. Ajoutez le nouveau rôle pour l’accès aux interpréteurs Zeppelin. Dans l’exemple suivant, tous les utilisateurs de `adminGroupName` ont accès aux interpréteurs Zeppelin et peuvent créer des interpréteurs. Vous pouvez placer plusieurs rôles entre les crochets du paramètre `roles[]`, séparés par des virgules. Ensuite, les utilisateurs disposant des autorisations nécessaires peuvent accéder aux interpréteurs Zeppelin.

    ```
    [urls]
    /api/interpreter/** = authc, roles[adminGroupName]
    ```

## <a name="livy-session-management"></a>Gestion des sessions Livy

Le premier paragraphe de code de votre bloc-notes Zeppelin crée une session livy dans votre cluster. Cette session est partagée entre tous les blocs-notes Zeppelin que vous créez par la suite. Si la session livy est supprimée pour une raison quelconque, les travaux ne sont pas exécutés à partir du bloc-notes Zeppelin.

Dans ce cas, vous devez suivre les étapes suivantes avant de commencer à exécuter des tâches à partir d’un bloc-notes Zeppelin.  

1. Redémarrez l’interpréteur Livy à partir du bloc-notes Zeppelin. Pour ce faire, ouvrez les paramètres de l'interpréteur en sélectionnant le nom de l'utilisateur connecté dans le coin supérieur droit, puis sélectionnez **Interpréteur**.

    ![Lancer l’interpréteur](./media/apache-spark-zeppelin-notebook/zeppelin-launch-interpreter.png "Sortie Hive")

2. Accédez à **livy2**, puis sélectionnez **redémarrer**.

    ![Redémarrer l’interpréteur Livy](./media/apache-spark-zeppelin-notebook/hdinsight-zeppelin-restart-interpreter.png "Redémarrer l’interpréteur Zeppelin")

3. Exécutez une cellule de code à partir d’un bloc-notes Zeppelin existant. Ce code crée une session Livy dans le cluster HDInsight.

## <a name="general-information"></a>Informations générales

### <a name="validate-service"></a>Valider le service

Pour valider le service à partir d’Ambari, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/ZEPPELIN/summary` où CLUSTERNAME est le nom de votre cluster.

Pour valider le service à partir d’une ligne de commande, utilisez le protocole SSH pour accéder au nœud principal. Basculez l’utilisateur sur Zeppelin à l’aide de la commande `sudo su zeppelin`. Commandes relatives à l’état :

|Commande |Description |
|---|---|
|`/usr/hdp/current/zeppelin-server/bin/zeppelin-daemon.sh status`|État du service.|
|`/usr/hdp/current/zeppelin-server/bin/zeppelin-daemon.sh --version`|Version du service.|
|`ps -aux | grep zeppelin`|Identifier le PID.|

### <a name="log-locations"></a>Emplacements des journaux

|Service |Path |
|---|---|
|zeppelin-server|/usr/hdp/current/zeppelin-server/|
|Journaux d’activité du serveur|/var/log/zeppelin|
|Interpréteur de configuration, `Shiro`, site.xml, log4j|/usr/hdp/current/zeppelin-server/conf or /etc/zeppelin/conf|
|Répertoire PID|/var/run/zeppelin|

### <a name="enable-debug-logging"></a>Activer l’enregistrement du débogage

1. Accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/ZEPPELIN/summary`, où CLUSTERNAME est le nom de votre cluster.

1. Accédez à **CONFIGS** > **Advanced zeppelin-log4j-properties** > **log4j_properties_content**.

1. Remplacez `log4j.appender.dailyfile.Threshold = INFO` par `log4j.appender.dailyfile.Threshold = DEBUG`.

1. Ajoutez `log4j.logger.org.apache.zeppelin.realm=DEBUG`.

1. Enregistrez les modifications et redémarrez le service.

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)
* [Noyaux accessibles à Jupyter Notebook dans le cluster Apache Spark pour HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)
