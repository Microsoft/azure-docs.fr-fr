---
title: Présentation d’Azure HDInsight
description: Présentation de HDInsight ainsi que de la pile de technologies et des composants Apache Hadoop et Apache Spark, notamment Kafka, Hive, Storm, et HBase pour l’analyse de données volumineuses (Big Data).
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: contperf-fy21q1
ms.date: 08/21/2020
ms.openlocfilehash: 7f4d820b33f36c295c294000d33a0ca5177fc4d1
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028068"
---
# <a name="what-is-azure-hdinsight"></a>Qu’est-ce qu’Azure HDInsight ?

Azure HDInsight est un service cloud d’analyse managé, complet et open source pour les entreprises. Vous pouvez utiliser les infrastructures open source telles que Hadoop, Apache Spark, Apache Hive, LLAP, Apache Kafka, Apache Storm, R et bien plus encore.

## <a name="what-is-hdinsight-and-the-hadoop-technology-stack"></a>Qu’est-ce que HDInsight et la pile de technologies Hadoop ?

Azure HDInsight est une distribution cloud des composants Hadoop. Azure HDInsight rend facile, rapide et économique le traitement de volumes importants de données. Vous pouvez utiliser les infrastructures open source les plus populaires, telles que Hadoop, Spark, Hive, LLAP, Kafka, Storm, R et bien plus encore. Avec ces infrastructures, vous pouvez activer un large éventail de scénarios, tels que l’extraction, la transformation et le chargement (ETL) ; l’entreposage de données ; l’apprentissage automatique ; et IoT.

Pour voir les piles de composants de technologie Hadoop disponibles sur HDInsight, consultez [Composants et versions disponibles avec HDInsight](./hdinsight-component-versioning.md). Pour plus d’informations sur Hadoop dans HDInsight, consultez la rubrique [Page de fonctionnalités Azure pour HDInsight](https://azure.microsoft.com/services/hdinsight/).

## <a name="what-is-big-data"></a>Que sont les données volumineuses ?

Les données volumineuses sont collectées dans des volumes toujours plus importants, à des vitesses élevées et pour une variété de formats plus grande qu’auparavant. Elles peuvent être historiques (c'est-à-dire stockées) ou en temps réel (c'est-à-dire diffusées à partir de la source). Consultez [Scénarios d’utilisation d’ HDInsight](#scenarios-for-using-hdinsight) pour en savoir plus sur les cas d’usage courants pour les Big Data.

## <a name="why-should-i-use-azure-hdinsight"></a>Pourquoi dois-je utiliser Azure HDInsight ?

Cette section répertorie les fonctionnalités d’Azure HDInsight.

|Fonctionnalité  |Description  |
|---------|---------|
|Cloud natif     |     Azure HDInsight vous permet de créer des clusters optimisés pour Hadoop, Spark,  [Interactive query (LLAP)](./interactive-query/apache-interactive-query-get-started.md), Kafka, Storm, HBase et ML Services sur Azure. HDInsight fournit également un contrat SLA de bout en bout sur toutes vos charges de travail de production.  |
|Économique et évolutif     | HDInsight vous permet d’effectuer un scale-up ou un scale-down de charges de travail. Vous pouvez réduire les coûts en créant des clusters à la demande et payer seulement pour ce que vous utilisez. Vous pouvez également créer des pipelines de données pour faire fonctionner vos travaux. Le stockage et le calcul découplés améliorent les performances et la flexibilité. |
|Sécurité et conformité    | HDInsight vous permet de protéger les ressources de données de votre entreprise à l’aide du réseau virtuel Azure, du chiffrementet de l’intégration avec Azure Active Directory. HDInsight répond également aux normes de conformité du gouvernement et du secteur les plus populaires.        |
|Supervision    | Azure HDInsight s’intègre aux journaux d’activité Azure Monitor pour fournir une interface unique permettant de superviser tous vos clusters.        |
|Disponibilité générale | HDInsight est disponible dans plus de régions que n’importe quelle autre offre d’analytique Big Data. Azure HDInsight est également disponible dans Azure Government, en Chine, et en Allemagne, ce qui vous permet de répondre aux besoins de votre entreprise dans les principaux domaines souverains. |  
|Productivité     |  Azure HDInsight vous permet d’utiliser des outils de productivité enrichis pour Hadoop et Spark avec les environnements de développement de votre choix. Parmi ces environnements de développement figurent Visual Studio, VSCode, Eclipse et IntelliJ pour la prise en charge de Scala, Python, R, Java et .NET. Les scientifiques des données peuvent également collaborer à l’aide de blocs-notes populaires, tels que Jupyter et Zeppelin.    |
|Extensibilité     |  Vous pouvez étendre les clusters HDInsight avec des composants installés (Hue, Presto, etc.) à l’aide d’actions de script, par l’ajout de nœuds de périphérie, ou l’intégration à d’autres applications certifiées Big data. HDInsight permet une intégration transparente aux solutions Big Data les plus populaires à l’aide d’un déploiement en un clic.|

## <a name="scenarios-for-using-hdinsight"></a>Scénarios d’utilisation de HDInsight

Azure HDInsight peut être utilisé pour divers scénarios lors d’un traitement de données Big data. Il peut s’agir de données historiques (déjà collectées et stockées) ou de données en temps réel (transmises en continu directement à partir de la source). Les scénarios pour le traitement de ces données peuvent être classés dans les catégories suivantes :

### <a name="batch-processing-etl"></a>Traitement par lots (ETL)

Extraction, transformation et chargement (ETL) est un processus au cours duquel les données structurées ou non sont extraites à partir de sources de données hétérogènes. Elles sont ensuite converties dans un format structuré et chargées dans un magasin de données. Les données converties peuvent être utilisées pour la science des données ou l’entreposage de données.

### <a name="data-warehousing"></a>Entrepôt de données

HDInsight permet d’exécuter des requêtes interactives sur des pétaoctets de données structurées ou non dans n’importe quel format. Vous pouvez également créer des modèles en les connectant à des outils BI. Pour plus d’informations, [consultez ce témoignage client](https://customers.microsoft.com/story/milliman).

Architecture de HDInsight : entrepôt de données

### <a name="internet-of-things-iot"></a>Internet des objets (IoT)

Vous pouvez utiliser HDInsight pour traiter les données de streaming reçues en temps réel depuis différentes sortes d’appareils. Pour plus d’informations, [lire ce billet de blog Azure annonçant la version préliminaire publique de Apache Kafka sur HDInsight avec Azure Disques Managés](https://azure.microsoft.com/blog/announcing-public-preview-of-apache-kafka-on-hdinsight-with-azure-managed-disks/).

Architecture HDInsight : Internet des objets

### <a name="data-science"></a>Science des données

Vous pouvez utiliser HDInsight pour créer des applications qui extraient des informations critiques à partir des données. Vous pouvez également utiliser Azure Machine Learning pour prédire les tendances futures de votre activité. Pour plus d’informations, [consultez ce témoignage client](https://customers.microsoft.com/story/pros).

Architecture de HDInsight : science des données

### <a name="hybrid"></a>Hybride

HDInsight permet d’étendre votre infrastructure Big Data locale existante sur Azure pour exploiter les fonctionnalités d’analyse avancée du cloud.

Architecture HDInsight : hybride

## <a name="cluster-types-in-hdinsight"></a>Types de cluster dans HDInsight

HDInsight comprend des types de cluster spécifiques et des fonctionnalités de personnalisation de cluster, comme l’ajout de composants, d’utilitaires et de langages. HDInsight offre les types de clusters suivants :

|Type du cluster | Description |
|---|---|
|[Apache Hadoop](./hadoop/apache-hadoop-introduction.md)|infrastructure qui utilise HDFS, la gestion de ressources YARN et un modèle de programmation MapReduce simple pour traiter et analyser les lots de données en parallèle.|
|[Apache Spark](./spark/apache-spark-overview.md)|infrastructure de traitement parallèle open source qui prend en charge le traitement en mémoire pour améliorer les performances des applications d’analyse du Big Data. Consultez [Présentation d’Apache Spark dans HDInsight](./spark/apache-spark-overview.md).|
|[Apache HBase](./hbase/apache-hbase-overview.md)|base de données NoSQL basée sur Hadoop qui fournit un accès aléatoire et une forte cohérence pour de vastes quantités de données non structurées et semi-structurées (potentiellement, des milliards de lignes multipliées par des millions de colonnes). Consultez [Qu’est-ce que HBase sur HDInsight ?](./hbase/apache-hbase-overview.md)|
|[ML Services](./r-server/r-server-overview.md)|serveur d’hébergement et de gestion de processus R distribués en parallèle. Ce serveur offre aux experts en science des données, aux statisticiens et aux programmeurs R un accès à la demande à des méthodes d’analyse évolutives et distribuées sur HDInsight. Consultez [Vue d’ensemble de ML Services sur HDInsight](./r-server/r-server-overview.md).|
|[Apache Storm](./storm/apache-storm-overview.md)|système de calcul distribué et en temps réel permettant le traitement rapide de vastes flux de données. Storm est fourni en tant que cluster géré dans HDInsight. Consultez la rubrique [Analyse de données de capteur en temps réel au moyen de Storm et de Hadoop](./storm/apache-storm-overview.md).|
|[Requête interactive Apache](./interactive-query/apache-interactive-query-get-started.md)|mise en cache pour des requêtes Hive interactives et plus rapides. Consultez l’article [Utilisation d’Interactive Query dans HDInsight](./interactive-query/apache-interactive-query-get-started.md).|
|[Apache Kafka](./kafka/apache-kafka-introduction.md)|plateforme open source utilisée pour créer des applications et des pipelines de données de diffusion en continu. Kafka fournit également une fonctionnalité de file d’attente de messages qui vous permet de publier des flux de données et de vous abonner à ces derniers. Consultez l’article [Présentation d’Apache Kafka sur HDInsight](./kafka/apache-kafka-introduction.md).|

## <a name="open-source-components-in-hdinsight"></a>Composants open source dans HDInsight

Azure HDInsight permet de créer des clusters avec des infrastructures open source, comme Hadoop, Spark, Hive, LLAP, Kafka, Storm, HBase et R. Ces clusters sont fournis par défaut avec d’autres composants open source inclus sur le cluster, comme Apache Ambari5, Avro5, Apache Hive3, HCatalog2, Apache Mahout2, Apache Hadoop MapReduce3, Apache Hadoop YARN2, Apache Phoenix3, Apache Pig3, Apache Sqoop3, Apache Tez3, Apache Oozie2 et Apache ZooKeeper5.  

## <a name="programming-languages-in-hdinsight"></a>Langages de programmation dans HDInsight

Les clusters HDInsight (dont Spark, HBase, Kafka, Hadoop et d’autres) prennent en charge de nombreux langages de programmation. Certains langages de programmation ne sont pas installés par défaut. Pour les bibliothèques, modules ou packages non installés par défaut, utilisez une action de script pour installer le composant.

|Langage de programmation  |Information  |
|---------|---------|
|Prise en charge des langages de programmation par défaut     | Par défaut, HDInsight prend en charge :<ul><li>Java</li><li>Python</li><li>.NET</li><li>Go</li></ul>  |
|Langages de machines virtuelles Java (JVM)     | De nombreux langages autres que Java peuvent s’exécuter sur une machine virtuelle Java (JVM). Toutefois, si vous exécutez certains de ces langages, vous devrez peut-être installer des composants supplémentaires sur le cluster. Les langages JVM suivants sont pris en charge sur les clusters HDInsight : <ul><li>Clojure</li><li>Jython (Python pour Java)</li><li>Scala</li></ul>     |
|Langages spécifiques à Hadoop     | Les clusters HDInsight prennent en charge les langages suivants, spécifiques à la pile de technologies Hadoop : <ul><li>Pig Latin pour les travaux Pig</li><li>HiveQL pour les travaux Hive et SparkSQL</li></ul>        |

## <a name="development-tools-for-hdinsight"></a>Outils de gestion pour HDInsight

Vous pouvez utiliser les outils de développement HDInsight, y compris IntelliJ, Eclipse, Visual Studio Code et Visual Studio, pour créer et soumettre une tâche et une requête de données HDInsight avec une intégration transparente avec Azure.

* Kit de ressources Azure pour IntelliJ10
* Kit de ressources Azure pour Eclipse6
* Outils Azure HDInsight pour VS Code13
* Azure Data Lake Tools pour Visual Studio9

## <a name="business-intelligence-on-hdinsight"></a>Le décisionnel sur HDInsight

Les outils décisionnels courants permettent de récupérer des données intégrées à HDInsight, de les analyser et de générer des rapports à leur sujet via le complément Power Query ou le Pilote ODBC Microsoft Hive :

* [Apache Spark BI utilisant des outils de visualisation de données avec Azure HDInsight](./spark/apache-spark-use-bi-tools.md)

* [Visualiser des données Apache Hive à l’aide de Microsoft Power BI dans Azure HDInsight](./hadoop/apache-hadoop-connect-hive-power-bi.md)

* [Visualiser des données Interactive Query Hive à l’aide de Power BI dans Azure HDInsight](./interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md)

* [Connecter Excel à Apache Hadoop avec Power Query](./hadoop/apache-hadoop-connect-excel-power-query.md) (requiert Windows)

* [Connecter Excel à Apache Hadoop avec le Pilote ODBC Microsoft Hive](./hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md) (requiert Windows)


## <a name="in-region-data-residency"></a>Résidence des données dans la région 

Spark, Hadoop, LLAP, Storm et MLService ne stockent pas les données client, si bien que ces services répondent automatiquement aux conditions de résidence des données dans la région, notamment à celles spécifiées dans le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/). 

En revanche, Kafka et HBase stockent des données client. Ces données sont automatiquement stockées par Kafka et HBase dans une seule région, donc ces services répondent aux conditions de résidence des données dans la région, notamment à celles spécifiées dans le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/). 


Les outils décisionnels courants permettent de récupérer des données intégrées à HDInsight, de les analyser et de générer des rapports à leur sujet par le biais du complément Power Query ou du Pilote ODBC Microsoft Hive.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un cluster Apache Hadoop dans HDInsight](./hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* Créer un cluster Apache Spark - Portail
* [Sécurité d’entreprise dans Azure HDInsight](./domain-joined/hdinsight-security-overview.md)
