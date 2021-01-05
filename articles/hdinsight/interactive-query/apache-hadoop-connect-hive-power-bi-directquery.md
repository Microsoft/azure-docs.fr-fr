---
title: Visualiser des données Interactive Query Hive à l’aide de Power BI dans Azure HDInsight
description: Utiliser Microsoft Power BI pour visualiser des données interactives Query Hive provenant d’Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/17/2019
ms.openlocfilehash: 92e94c911acb701b1ccf8e39636d152cc5bfb575
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534802"
---
# <a name="visualize-interactive-query-apache-hive-data-with-microsoft-power-bi-using-direct-query-in-hdinsight"></a>Visualiser des données Interactive Query Apache Hive avec Microsoft Power BI à l’aide de requêtes directes dans HDInsight

Cet article décrit comment connecter Microsoft Power BI à des clusters de requête interactive Azure HDInsight, et comment visualiser des données Apache Hive à l’aide d’une requête directe. L’exemple fourni charge les données d’une table Hive `hivesampletable` dans Power BI. La table Hive `hivesampletable` contient des données sur l’utilisation des téléphones mobiles. Vous allez ensuite tracer ces données d’utilisation sur une carte du monde :

![rapport cartographique Power BI HDInsight](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-visualization.png)

Vous pouvez tirer parti du [pilote ODBC Apache Hive](../hadoop/apache-hadoop-connect-hive-power-bi.md) pour effectuer l’import par le biais du connecteur ODBC générique dans Power BI Desktop. Il n’est toutefois pas recommandé pour les charges de travail décisionnelles en raison de la nature non interactive du moteur d’interrogation Hive. Le [connecteur Interactive Query HDInsight](./apache-hadoop-connect-hive-power-bi-directquery.md) et le [connecteur Apache Spark HDInsight ](/power-bi/spark-on-hdinsight-with-direct-connect) sont des choix plus judicieux du fait de leurs performances.

## <a name="prerequisites"></a>Conditions préalables requises
Avant de poursuivre cet article, vérifiez que vous avez les éléments suivants :

* Un **cluster HDInsight** . Cela peut être un cluster HDInsight avec Apache Hive ou un cluster du nouveau type Interactive Query. Pour plus d’informations sur la création de clusters, consultez [Créer un cluster](../hadoop/apache-hadoop-linux-tutorial-get-started.md).
* **[Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/)** . Vous pouvez télécharger une copie à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=45331).

## <a name="load-data-from-hdinsight"></a>Chargement des données à partir de HDInsight

La table Hive `hivesampletable` est fournie avec tous les clusters HDInsight.

1. Démarrez Power BI Desktop.

2. À partir de la barre de menus, accédez à **Accueil** > **Obtenir des données** > **Plus...** .

    ![HDInsight Power BI - Obtenir plus de données](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-open-odbc.png)

3. À partir de la fenêtre **Obtenir des données** , tapez **hdinsight** dans la zone de recherche.  

4. Dans les résultats de la recherche, sélectionnez **HDInsight Interactive Query** , puis **Se connecter** .  Si vous ne voyez pas **HDInsight Interactive Query** , vous devez mettre à jour Power BI Desktop vers la dernière version.

5. Sélectionnez **Continuer** pour fermer la boîte de dialogue **Connexion à un service tiers** .

6. Dans la fenêtre **HDInsight Interactive Query** , entrez les informations suivantes et sélectionnez **OK** :

    |Propriété | Valeur |
    |---|---|
    |Serveur |Entrez le nom du cluster, par exemple *myiqcluster.azurehdinsight.net* .|
    |Base de données |Entrez **default** pour cet article.|
    |Data Connectivity mode (Mode de connectivité des données) |Sélectionnez **DirectQuery** pour cet article.|

    ![HDInsight interactive query Power BI DirectQuery connect](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-interactive-query-power-bi-connect.png)

7. Entrez les informations d’identification HTTP, puis sélectionnez **Se connecter** . Le nom d’utilisateur par défaut est **admin** .

8. Dans la fenêtre **Navigateur** du volet gauche, sélectionnez **hivesampletale** .

9. Sélectionnez **Charger** dans la fenêtre principale.

    ![HDInsight interactive query Power BI hivesampletable](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-interactive-query-power-bi-hivesampletable.png)

## <a name="visualize-data-on-a-map"></a>Visualiser les données sur une carte

Poursuivez la procédure précédente.

1. Dans le volet Visualisations, sélectionnez **Carte** (l’icône en forme de globe). Une carte générique s’affiche alors dans la fenêtre principale.

    ![personnalisation de rapport Power BI HDInsight](./media/apache-hadoop-connect-hive-power-bi-directquery/hdinsight-power-bi-customize.png)

2. Dans le volet Champs, sélectionnez **country** et **devicemake** . Une carte du monde avec les points de données s’affiche dans la fenêtre principale après un court instant.

3. Développez la carte.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à visualiser des données depuis HDInsight avec Microsoft Power BI.  Pour plus d’informations sur la visualisation des données, voir les articles suivants :

* [Visualiser des données Apache Hive à l’aide de Microsoft Power BI et ODBC dans Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md). 
* [Utiliser Apache Zeppelin pour exécuter des requêtes Apache Hive dans Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
* [Connecter Excel à HDInsight avec le pilote ODBC Microsoft Hive](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).
* [Connecter Excel à Apache Hadoop à l’aide de Power Query](../hadoop/apache-hadoop-connect-excel-power-query.md).
* [Se connecter à Azure HDInsight et exécuter des requêtes Apache Hive avec Data Lake Tools pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).
* [Utiliser Azure HDInsight Tools pour Visual Studio Code](../hdinsight-for-vscode.md).
* [Charger des données dans HDInsight](./../hdinsight-upload-data.md).