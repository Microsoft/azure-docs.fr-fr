---
title: Utiliser la vue Apache Ambari Hive avec Apache Hadoop dans Azure HDInsight
description: Découvrez comment utiliser la vue Hive à partir de votre navigateur web pour envoyer des requêtes Hive. La vue Hive fait partie de l’interface utilisateur web Ambari fournie avec votre cluster HDInsight sous Linux.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/23/2020
ms.openlocfilehash: cc04a891bb32ede6c7bb72a339b728642cf343ed
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000519"
---
# <a name="use-apache-ambari-hive-view-with-apache-hadoop-in-hdinsight"></a>Utiliser la vue Apache Ambari Hive avec Apache Hadoop dans HDInsight

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Découvrez comment exécuter des requêtes Hive avec la vue Apache Ambari Hive. L’affichage Hive vous permet de créer, d’optimiser et d’exécuter des requêtes Hive à partir du navigateur web.

## <a name="prerequisites"></a>Conditions préalables requises

Un cluster Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).

## <a name="run-a-hive-query"></a>Exécution d'une tâche Hive

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre cluster.  Pour obtenir des instructions, consultez la page [Énumération et affichage des clusters](../hdinsight-administer-use-portal-linux.md#showClusters). Le cluster est ouvert dans un nouvel affichage du portail.

1. Dans **Tableaux de bord du cluster**, sélectionnez **Vues Ambari**. Lorsque vous êtes invité à vous authentifier, utilisez le nom de compte et le mot de passe de connexion de cluster (`admin` par défaut) que vous avez fournis lors de la création du cluster. Vous pouvez également accéder à `https://CLUSTERNAME.azurehdinsight.net/#/main/views` dans votre navigateur, où `CLUSTERNAME` est le nom de votre cluster.

1. Dans la liste des vues, sélectionnez __Affichage Hive__.

    ![Apache Ambari - Sélectionner la vue Apache Hive](./media/apache-hadoop-use-hive-ambari-view/select-apache-hive-view.png)

    L’affiche Hive est similaire à ceci :

    ![Image de la feuille de calcul de requête pour l’affichage Hive](./media/apache-hadoop-use-hive-ambari-view/ambari-worksheet-view.png)

1. Dans l’onglet __Requête__ , collez les instructions HiveQL suivantes dans la feuille de calcul :

    ```hiveql
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs(
        t1 string,
        t2 string,
        t3 string,
        t4 string,
        t5 string,
        t6 string,
        t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION '/example/data/';
    SELECT t4 AS loglevel, COUNT(*) AS count FROM log4jLogs
        WHERE t4 = '[ERROR]'
        GROUP BY t4;
    ```

    Ces instructions effectuent les opérations suivantes :

    |. | Description |
    |---|---|
    |DROP TABLE|Supprime la table et le fichier de données, si la table existe déjà.|
    |CREATE EXTERNAL TABLE|Crée une nouvelle table « externe » dans Hive. Les tables externes stockent uniquement la définition de table dans Hive. Les données restent à l'emplacement d'origine.|
    |ROW FORMAT|Montre la mise en forme des données. Dans ce cas, les champs de chaque journal sont séparés par un espace.|
    |STORED AS TEXTFILE LOCATION|Montre où sont stockées les données, et qu’elles sont stockées sous forme de texte.|
    |SELECT|Sélectionne toutes les lignes où la colonne t4 contient la valeur [ERROR].|

   > [!IMPORTANT]  
   > Conservez la sélection __Base de données__ par __défaut__. Les exemples de ce document utilisent la base de données par défaut incluse avec HDInsight.

1. Pour démarrer la requête, sélectionnez **Exécuter** au-dessous de la feuille de calcul. Ce bouton devient orange et le texte affiche **Stop** (Arrêter).

1. Lorsque la requête est terminée, l’onglet **Results** (Résultats) affiche les résultats de l’opération. Le texte suivant est le résultat de la requête :

    ```output
    loglevel       count
    [ERROR]        3
    ```

    Vous pouvez utiliser l’onglet **JOURNAL** pour afficher les informations de journalisation que le travail a créées.

   > [!TIP]  
   > Téléchargez ou enregistrez les résultats de la boîte de dialogue à liste déroulante **Actions** dans l’onglet **Résultats**.

### <a name="visual-explain"></a>Visual Explain

Pour afficher une visualisation du plan de requête, sélectionnez l’onglet **Visual Explain** au-dessous de la feuille de calcul.

La vue **Visual Explain** de la requête peut être utile pour comprendre le déroulement de requêtes complexes.

### <a name="tez-ui"></a>Interface utilisateur Tez

Sélectionnez l’onglet **Tez UI** (IU Tez) au-dessous de la feuille de calcul afin d’afficher l’interface utilisateur Tez pour la requête.

> [!IMPORTANT]  
> Tez n’est pas utilisé pour résoudre toutes les requêtes. Vous pouvez résoudre de nombreuses requêtes sans utiliser Tez.

## <a name="view-job-history"></a>Afficher l’historique des travaux

L’onglet __Travaux__ affiche un historique des requêtes Hive.

![Apache Hive - Afficher l’onglet d’historique des tâches](./media/apache-hadoop-use-hive-ambari-view/apache-hive-job-history.png)

## <a name="database-tables"></a>Tables de base de données

Vous pouvez utiliser l’onglet __Tables__ onglet pour travailler avec des tables dans une base de données Hive.

![Image de l’onglet Tables d’Apache Hive](./media/apache-hadoop-use-hive-ambari-view/hdinsight-tables-tab.png)

## <a name="saved-queries"></a>Requêtes enregistrées

À partir de l’onglet **Query** (Requête), vous pouvez éventuellement enregistrer des requêtes. Après avoir enregistré une requête, vous pouvez la réutiliser à partir de l’onglet __Saved Queries__ (Requêtes enregistrées).

![Onglet Affichage des requêtes Apache Hive enregistrées](./media/apache-hadoop-use-hive-ambari-view/ambari-saved-queries.png)

> [!TIP]  
> Les requêtes enregistrées sont stockées dans le stockage en cluster par défaut. Vous pouvez rechercher les requêtes enregistrées sous le chemin `/user/<username>/hive/scripts`. Ces fichiers sont stockés en tant que fichiers `.hql` en texte brut.
>
> Si vous supprimez le cluster, tout en conservant le stockage, vous pouvez employer un utilitaire tel que [l’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) ou l’Explorateur de stockage Data Lake (à partir du [portail Azure](https://portal.azure.com)) pour récupérer les requêtes.

## <a name="user-defined-functions"></a>Fonctions définies par l’utilisateur

Vous pouvez étendre Hive par l’intermédiaire de fonctions définies par l’utilisateur. Utilisez une fonction définie par l’utilisateur pour implémenter une fonctionnalité ou une logique qui n’est pas facilement modélisée en HiveQL.

Déclarez et enregistrez un ensemble de fonctions définies par l’utilisateur en utilisant l’onglet **UDF** en haut de l’affichage Hive. Ces UDF peuvent être utilisés avec les **l’éditeur de requête**.

![Affichage de l’onglet UDF d’Apache Hive](./media/apache-hadoop-use-hive-ambari-view/user-defined-functions.png)

Un bouton **Insérer des UDF** apparaît au bas de **l’éditeur de requête**. Cette entrée affiche une liste déroulante des UDF définies dans l’affichage Hive. La sélection d’une fonction UDF ajoute des instructions HiveQL à votre requête pour activer l’UDF.

Par exemple, si vous avez défini une fonction UDF avec les propriétés suivantes :

* Nom de ressource : myudfs

* Chemin d’accès à la ressource : /myudfs.jar

* Nom de la fonction UDF : myawesomeudf

* Nom de la classe UDF : com.myudfs.Awesome

L’utilisation du bouton **Insert udfs** (Insérer des fonctions définies par l’utilisateur) affiche une entrée nommée **myudfs** avec une autre liste déroulante pour chaque fonction définie par l’utilisateur qui est paramétrée pour cette ressource. Dans le cas présent, il s’agit de **myawesomeudf**. La sélection de cette entrée ajoute le code suivant au début de la requête :

```hiveql
add jar /myudfs.jar;
create temporary function myawesomeudf as 'com.myudfs.Awesome';
```

Vous pouvez ensuite utiliser la fonction UDF dans votre requête. Par exemple : `SELECT myawesomeudf(name) FROM people;`.

Pour plus d’informations sur l’utilisation des fonctions définies par l’utilisateur avec Hive sur HDInsight, consultez les articles suivants :

* [Utilisation de Python UDF avec Apache Hive et Apache Pig dans HDInsight](python-udf-hdinsight.md)
* [Utiliser une fonction UDF Java avec Apache Hive dans HDInsight](./apache-hadoop-hive-java-udf.md)

## <a name="hive-settings"></a>Paramètres Hive

Vous pouvez modifier différents paramètres Hive, par exemple pour passer du moteur d’exécution Tez (par défaut) à MapReduce pour Hive.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des informations générales sur Hive dans HDInsight :

* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)
* [Utiliser le client Apache Beeline avec Apache Hive](apache-hadoop-use-hive-beeline.md)
