---
title: Apache Hive et outils Data Lake pour Visual Studio – Azure HDInsight
description: Découvrez comment utiliser les outils Data Lake pour Visual Studio pour exécuter des requêtes Apache Hive avec Apache Hadoop sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: c8645ae9cb901b9fc95f00665d73e223a24fda63
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86076451"
---
# <a name="run-apache-hive-queries-using-the-data-lake-tools-for-visual-studio"></a>Exécution de requêtes Apache Hive à l’aide des outils Data Lake pour Visual Studio

Découvrez comment utiliser les outils Data Lake pour Visual Studio pour interroger Apache Hive. Les outils Data Lake vous permettent de facilement créer, soumettre et surveiller les requêtes Hive à Apache Hadoop sur Azure HDInsight.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Hadoop sur HDInsight. Pour plus d’informations sur la création de cet élément, consultez [Créer un cluster Apache Hadoop dans Azure HDInsight avec un modèle Resource Manager](./apache-hadoop-linux-tutorial-get-started.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/). Dans le cadre de cet article, Visual Studio 2019 a été utilisé.

* Outils HDInsight pour Visual Studio ou Outils Azure Data Lake pour Visual Studio. Pour plus d’informations sur l’installation et la configuration des outils, consultez [Installer les outils Data Lake pour Visual Studio](apache-hadoop-visual-studio-tools-get-started.md#install-data-lake-tools-for-visual-studio).

## <a name="run-apache-hive-queries-using-the-visual-studio"></a>Exécuter des requêtes Apache Hive avec Visual Studio

Vous pouvez créer et exécuter des requêtes Hive de deux façons :

* Créer des requêtes ad hoc.
* Créer une application Hive.

### <a name="create-an-ad-hoc-hive-query"></a>Créer une requête Hive ad-hoc.

Les requêtes ad hoc peuvent être exécutées dans le mode **Batch** ou **Interactive**.

1. Lancez **Visual Studio** et sélectionnez **Continuer sans code**.

2. À partir de l’**Explorateur de serveurs**, faites un clic droit sur **Azure**, sélectionnez **Se connecter à un abonnement Microsoft Azure...** , puis terminez le processus de connexion.

3. Développez **HDInsight**, cliquez avec le bouton droit sur le cluster dans lequel vous souhaitez exécuter la requête, puis sélectionnez **Écrire une requête Hive**.

4. Exécutez la requête Hive suivante :

    ```hql
    SELECT * FROM hivesampletable;
    ```

5. Sélectionnez **Exécuter**. Le mode d’exécution par défaut est **Interactive**.

    ![Exécuter une requête Hive interactive, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/vs-execute-hive-query.png)

6. Pour exécuter la même requête dans le mode **Batch**, utilisez la liste déroulante pour passer de **Interactive** à **Batch**. Le bouton d’exécution **Exécuter** devient **Envoyer**.

    ![Envoyer une requête Hive par lot, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/visual-studio-batch-query.png)

    L’éditeur Hive prend en charge IntelliSense. Data Lake Tools pour Visual Studio prend en charge le chargement des métadonnées distantes pendant la modification d’un script Hive. Par exemple, si vous tapez `SELECT * FROM`, IntelliSense répertorie tous les noms de table suggérés. Lorsqu’un nom de table est spécifié, IntelliSense répertorie les noms de colonne. Les outils prennent en charge la plupart des instructions DML, sous-requêtes et fonctions définies par l’utilisateur intégrées de Hive. IntelliSense propose uniquement les métadonnées du cluster sélectionné dans la barre d’outils HDInsight.

7. Dans la barre d’outils de la requête (la zone située sous l’onglet de la requête et au-dessus du texte de la requête), sélectionnez **Envoyer** ou sélectionnez la flèche déroulante à côté d’**Envoyer**, puis choisissez **Avancé** dans la liste déroulante. Si vous sélectionnez cette dernière option,

8. Si vous avez sélectionné l’option d’envoi avancé, configurez les éléments **Nom du travail**, **Arguments**, **Configurations supplémentaires** et **Répertoire d’état** dans la boîte de dialogue **Envoyer le script**. Ensuite, sélectionnez **Envoyer**.

    ![Boîte de dialogue Envoyer le script, requête Hive HDInsight Hadoop](./media/apache-hadoop-use-hive-visual-studio/vs-tools-submit-jobs-advanced.png)

### <a name="create-a-hive-application"></a>Création d’une application Hive

Pour exécuter une requête Hive en créant une application Hive, procédez comme suit :

1. Ouvrez **Visual Studio**.

2. Dans la fenêtre **Démarrer**, sélectionnez **Créer un projet**.

3. Dans la fenêtre **Créer un nouveau projet**, dans la zone **Rechercher des modèles**, entrez *Hive*. Choisissez ensuite **Application Hive**, puis sélectionnez **Suivant**.

4. Dans la fenêtre **Configurer votre nouveau projet**, entrez un **Nom de projet**, sélectionnez ou créez un **Emplacement** pour le nouveau projet, puis sélectionnez **Créer**.

5. Ouvrez le fichier **Script.hql** créé avec ce projet et collez les instructions HiveQL suivantes :

    ```hql
    set hive.execution.engine=tez;
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND  INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

    Ces instructions effectuent les opérations suivantes :

    * `DROP TABLE`: Supprime la table si elle existe.

    * `CREATE EXTERNAL TABLE`: crée une nouvelle table externe dans Hive. Les tables externes stockent uniquement la définition de table dans Hive. (Les données restent à l’emplacement d’origine.)

        > [!NOTE]  
        > Les tables externes doivent être utilisées lorsque vous vous attendez à ce que les données sous-jacentes soient mises à jour par une source externe, telles qu’un travail MapReduce ou un service Azure.
        >
        > La suppression d'une table externe ne supprime **pas** les données, mais seulement la définition de la table.

    * `ROW FORMAT`: indique à Hive la façon dont les données sont mises en forme. Dans ce cas, les champs de chaque journal sont séparés par un espace.

    * `STORED AS TEXTFILE LOCATION`: Indique à Hive que les données sont stockées dans le répertoire *example/data* sous forme de texte.

    * `SELECT`: Sélectionne toutes les lignes dont la colonne `t4` contient la valeur `[ERROR]`. Cette instruction renvoie la valeur `3`, car trois lignes contiennent cette valeur.

    * `INPUT__FILE__NAME LIKE '%.log'`: Indique à Hive de retourner uniquement des données provenant de fichiers se terminant par .log. Cette clause limite la recherche au fichier *sample.log* qui contient les données.

6. À partir de la barre d’outils du fichier de requête (ressemble à la barre d’outils de requête ad hoc), sélectionnez le cluster HDInsight que vous souhaitez utiliser pour cette requête. Modifiez ensuite **Interactive** en **Batch** (le cas échéant), puis sélectionnez **Envoyer** pour exécuter les instructions en tant que travail Hive.

   Le **résumé de tâche Hive** apparaît et affiche des informations sur la tâche en cours d’exécution. Utilisez le lien **Actualiser** pour actualiser les informations sur la tâche, jusqu’à ce que l’**état de la tâche** passe à **Terminé**.

   ![Résumé du travail Hive terminé, application Hive, Visual Studio](./media/apache-hadoop-use-hive-visual-studio/hdinsight-job-summary.png)

7. Sélectionnez **Sortie du travail** pour afficher la sortie de ce travail. Il affiche `[ERROR] 3`, soit la valeur retournée par cette requête.

### <a name="additional-example"></a>Exemple supplémentaire

L’exemple suivant s’appuie sur la table `log4jLogs` créée lors de la procédure précédente, [Créer une application Hive](#create-a-hive-application).

1. À partir de l’**Explorateur de serveurs**, faites un clic droit sur votre cluster et sélectionnez **Écrire une requête Hive**.

2. Exécutez la requête Hive suivante :

    ```hql
    set hive.execution.engine=tez;
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    Ces instructions effectuent les opérations suivantes :

    * `CREATE TABLE IF NOT EXISTS`: Crée une table, si elle n’existe pas déjà. Étant donné que le mot clé `EXTERNAL` n’est pas utilisé, cette instruction crée une table interne. Les tables internes sont stockées dans l’entrepôt de données Hive et gérées par Hive.

        > [!NOTE]  
        > Contrairement aux tables `EXTERNAL`, la suppression d’une table interne entraîne également la suppression des données sous-jacentes.

    * `STORED AS ORC`: Stocke les données dans un format *ORC* (Optimized Row Columnar). ORC est un format particulièrement efficace et optimisé pour le stockage de données Hive.

    * `INSERT OVERWRITE ... SELECT`: Sélectionne des lignes de la table `log4jLogs` qui contiennent `[ERROR]`, puis insère les données dans la table `errorLogs`.

3. Modifiez **Interactive** en **Batch** le cas échéant, puis sélectionnez **Envoyer**.

4. Pour vérifier que le travail a créé la table, accédez à **Explorateur de serveurs** et développez **Azure** > **HDInsight**. Développez votre cluster HDInsight, puis **Bases de données Hive** > **par défaut**. La table **errorLogs** et la table **log4jLogs** sont répertoriées.

## <a name="next-steps"></a>Étapes suivantes

Comme vous pouvez le voir, les outils HDInsight pour Visual Studio fournissent un moyen simple de travailler avec des requêtes Hive sur HDInsight.

* Pour des informations générales sur Hive dans HDInsight, consultez [Présentation d’Apache Hive et HiveQL sur Azure HDInsight](hdinsight-use-hive.md)

* Pour des informations sur les autres façons de travailler avec Hadoop sur HDInsight, consultez [Utiliser MapReduce dans Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)

* Pour plus d’informations sur les outils HDInsight pour Visual Studio, consultez [Utiliser les outils Data Lake pour Visual Studio afin de se connecter à Azure HDInsight et exécuter des requêtes Apache Hive](apache-hadoop-visual-studio-tools-get-started.md)
