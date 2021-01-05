---
title: Installer des applications tierces sur Azure HDInsight
description: Découvrez comment installer des applications Apache Hadoop tierces sur Azure HDInsight.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/17/2019
ms.author: hrasheed
ms.openlocfilehash: 3c1c7ac8fe8b7adf287bcde30a054df5ebaa63d7
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94337438"
---
# <a name="install-third-party-apache-hadoop-applications-on-azure-hdinsight"></a>Installer des applications Apache Hadoop tierces sur Azure HDInsight

Découvrez comment installer une application [Apache Hadoop](https://hadoop.apache.org/) tierce sur Azure HDInsight. Pour des instructions d’installation de votre propre application, consultez la page [Installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md).

Une application HDInsight est une application que les utilisateurs peuvent installer sur un cluster HDInsight. Ces applications peuvent être développées par Microsoft, par des éditeurs de logiciels indépendants (ISV) ou par vous-même.  

La liste suivante affiche les applications publiées :

|Application |Type(s) de cluster(s) | Description |
|---|---|---|
|[AtScale Intelligence Platform](https://azuremarketplace.microsoft.com/marketplace/apps/atscaleinc.atscale) |Hadoop |AtScale transforme votre cluster HDInsight en serveur OLAP scale-out, qui vous permet d’interroger des milliards de lignes de données de manière interactive à l’aide des outils décisionnels que vous connaissez et utilisez déjà, tels que Microsoft Excel, Power BI, Tableau Software et QlikView. |
|[CDAP for HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/cask.cdap-for-hdinsight) |hbase |CDAP est la première plateforme d’intégration unifiée pour le Big Data à offrir un retour sur investissement accéléré pour Hadoop et à permettre aux équipes informatiques de proposer des données en libre-service. Disponible en open source et extensible, CDAP élimine les obstacles à l’innovation. Conditions requises : 4 nœuds de région, D3 v2 minimum. |
|[Datameer](https://azuremarketplace.microsoft.com/marketplace/apps/datameer.datameer) |Hadoop |La plateforme évolutive en libre-service de Datameer pour la préparation, l’exploration et la gouvernance des données à des fins d’analytique accélère la transformation des données complexes issues de plusieurs sources en informations utiles prêtes à l’emploi. Vous bénéficiez ainsi d’insights plus rapides et pertinents à l’échelle de l’entreprise. |
|[Dataiku DSS on HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/dataiku.dss-on-hdi) |Hadoop, Spark |Dataiku DSS est une plateforme de science des données d’entreprise qui permet aux scientifiques des données et aux analyses de données de collaborer plus efficacement sur la conception et l’exécution de nouveaux produits et services de données, convertissant ainsi les données brutes en prédictions percutantes. |
|[WANdisco Fusion HDI App](https://community.wandisco.com/s/article/Use-WANdisco-Fusion-for-parallel-operation-of-ADLS-Gen1-and-Gen2) |Hadoop, Spark, HBase, Storm, Kafka |Parvenir à garantir la cohérence des données dans un environnement distribué représente un véritable défi au niveau des opérations de données volumineuses. WANdisco Fusion, une plateforme logicielle d’entreprise, résout ce problème en assurant la cohérence des données non structurées au sein de n’importe quel environnement. |
|[H2O SparklingWater for HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/h2o-ai.h2o-sparklingwater) |Spark |H2O Sparkling Water prend en charge les algorithmes distribués suivants : GLM, Naïve Bayes, Distributed Random Forest, Gradient Boosting Machine, Deep Neural Networks, Deep learning, K-means, PCA, Generalized Low Rank Models, Anomaly Detection, Autoencoders. |
|[Striim for Real-Time Data Integration to HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/striim.striimbyol) |Hadoop, HBase, Storm, Spark, Kafka |Striim (prononcé « stream ») est une plateforme de bout en bout d’intégration et d’intelligence qui permet l’ingestion, le traitement et l’analyse en continu de flux de données disparates. |
|[Jumbune Enterprise-Accelerating BigData Analytics](https://azuremarketplace.microsoft.com/marketplace/apps/impetus-infotech-india-pvt-ltd.impetus_jumbune) |Hadoop, Spark |Globalement, Jumbune aide les entreprises en : 1. accélérant les performances de charge de travail Hive, Java, Scala basées sur le moteur Tez, MapReduce et Spark ; 2. offrant une surveillance proactive des clusters Hadoop ; 3. mettant en place une gestion de la qualité des données sur le système de fichiers DFS. |
|[Kyligence Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/kyligence.kyligence) |Hadoop, HBase, Spark |Optimisé par Apache Kylin, Kyligence Enterprise applique le décisionnel au Big Data. En tant que moteur OLAP d’entreprise sur Hadoop, Kyligence Enterprise permet aux analystes métier de concevoir l’architecture du décisionnel sur Hadoop avec des méthodologies de décisionnel et d’entrepôt de données standard. |
|[Starburst Presto for Azure HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/starburstdatainc1579800938563.starburst-presto?tab=Overview) |Hadoop |Presto est un moteur de requête SQL distribué rapide et évolutif. Conçu pour séparer le calcul du stockage, Presto est parfait pour interroger des données dans les bases de données Azure Data Lake Storage, Stockage Blob Azure, SQL et NoSQL et autres sources de données. |
|[StreamSets Data Collector for HDInsight Cloud](https://azuremarketplace.microsoft.com/marketplace/apps/streamsets.streamsets-data-collector-hdinsight) |Hadoop, HBase, Spark, Kafka |StreamSets Data Collector est un puissant moteur léger qui diffuse les données en temps réel. Utilisez Data Collector pour acheminer et traiter les données dans vos flux de données. Cette application est fournie avec une licence d’essai de 30 jours. |
|[Trifacta Wrangler Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/trifacta.trifacta-db?tab=Overview) |Hadoop, Spark, HBase |Trifacta Wrangler Enterprise pour HDInsight prend en charge le data wrangling dans l’ensemble de l’entreprise pour n’importe quelle échelle de données. Le coût d’exécution de Trifacta sur Azure combine les coûts d’abonnement Trifacta et les coûts d’infrastructure Azure pour les machines virtuelles. |
|[Unifi Data Platform](https://unifisoftware.com/platform/) |Hadoop, HBase, Storm, Spark |Unifi Data Platform est une suite intégrée d’outils de données en libre-service conçus pour permettre aux utilisateurs professionnels de faire face aux défis liés aux données afin d’accroître progressivement les revenus et de réduire les coûts ou la complexité opérationnelle. |
|[Unraveldata APM](https://azuremarketplace.microsoft.com/marketplace/apps/unravel-data.unravel-app) |Spark |Application pour cluster HDInsight Spark. |
|[Waterline AI-Driven Data Catalog](https://azuremarketplace.microsoft.com/marketplace/apps/waterline_data.waterline_data) |Spark |Waterline catalogue, organise et régit les données à l’aide de l’intelligence artificielle afin d’identifier automatiquement les données contenant des termes métier. Le catalogue Waterline est un composant important pour la réussite de l’analytique en libre-service, de la conformité et de la gouvernance et des différentes initiatives de gestion informatique. |

Les instructions fournies dans cet article utilisent le portail Azure. Vous pouvez également exporter le modèle Azure Resource Manager à partir du portail ou obtenir une copie de ce modèle auprès des fournisseurs et utiliser Azure PowerShell et Azure Classic CLI pour déployer le modèle.  Consultez [Créer des clusters Apache Hadoop dans HDInsight avec des modèles Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).

## <a name="prerequisites"></a>Prérequis
Pour installer des applications HDInsight sur un cluster HDInsight existant, vous devez disposer d’un cluster HDInsight. Pour en créer un, consultez [Créer des clusters](hadoop/apache-hadoop-linux-tutorial-get-started.md). Vous pouvez également installer des applications HDInsight lorsque vous créez un cluster HDInsight.

## <a name="install-applications-to-existing-clusters"></a>Installer des applications dans des clusters existants
La procédure suivante montre comment installer des applications HDInsight dans un cluster HDInsight existant.

**Installer une application HDInsight**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, accédez à **Tous les services** > **Analytique** > **Clusters HDInsight**.
3. Sélectionnez un cluster HDInsight dans la liste.  Si vous ne disposez pas d’un tel cluster, vous devez d’abord en créer un.  Consultez la page [Créer des clusters](hadoop/apache-hadoop-linux-tutorial-get-started.md).
4. Sous la catégorie **Paramètres** , sélectionnez **Applications**. Vous pouvez consulter la liste des applications installées dans la fenêtre principale. 
   
    ![Applications HDInsight - menu du portail](./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png)
5. Dans le menu, sélectionnez **+Ajouter**. La liste des applications disponibles s’affiche.  Si l’option **+Ajouter** est grisée, cela signifie qu’il n’existe aucune application pour cette version du cluster HDInsight.
   
    ![Applications HDInsight - applications disponibles](./media/hdinsight-apps-install-applications/hdinsight-apps-list1.png)
6. Sélectionnez l’une des applications disponibles, puis suivez les instructions pour accepter les conditions légales.

Vous pouvez vérifier l’état de l’installation de l’application dans les notifications du portail (sélectionnez l’icône en forme de cloche en haut du portail). L’application installée s’affiche dans la liste Applications installées.

## <a name="install-applications-during-cluster-creation"></a>Installer des applications lors de la création du cluster

Vous pouvez également installer des applications HDInsight lorsque vous créez un cluster. Au cours du processus, les applications HDInsight sont installées une fois le cluster créé et en cours d’exécution. Pour installer des applications pendant la création du cluster à l’aide du portail Azure, à partir de l’onglet **Configuration + prix** , sélectionnez **+ Ajouter une application**.

![Portail Azure - Configuration des clusters - Applications](./media/hdinsight-apps-install-applications/azure-portal-cluster-configuration-applications.png)

## <a name="list-installed-hdinsight-apps-and-properties"></a>Répertorier les applications HDInsight installées et les propriétés
Le portail affiche une liste des applications HDInsight installées pour un cluster et les propriétés de chaque application installée.

**Répertorier une application HDInsight et afficher des propriétés**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, accédez à **Tous les services** > **Analytique** > **Clusters HDInsight**.
3. Sélectionnez un cluster HDInsight dans la liste.
4. Sous la catégorie **Paramètres** , sélectionnez **Applications**. Vous pouvez consulter la liste des applications installées dans la fenêtre principale. 
   
    ![Applications HDInsight - applications installées](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png)
5. Sélectionnez l’une des applications installées pour en afficher la propriété. Listes des propriétés :

    |Propriété | Description |
    |---|---|
    |Nom de l’application |Nom de l’application. |
    |Statut |État de l’application. |
    |Page web |L’URL de l’application web que vous avez déployée sur le nœud de périphérie. Les informations d’identification sont les mêmes que celles de l’utilisateur HTTP configurées pour le cluster. |
    |Point de terminaison SSH |Vous pouvez utiliser SSH pour vous connecter au nœud de périphérie. Les informations d’identification sont les mêmes que celles de l’utilisateur SSH configurées pour le cluster. Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md). |
    |Description | Description de l’application. |

6. Pour supprimer une application, cliquez avec le bouton droit sur l’application, puis sur **Supprimer** dans le menu contextuel.

## <a name="connect-to-the-edge-node"></a>Connexion au nœud de périmètre
Vous pouvez vous connecter au nœud de périmètre à l’aide de HTTP et SSH. Vous trouverez les informations de point de terminaison sur le [portail](#list-installed-hdinsight-apps-and-properties). Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

Les informations d’identification du point de terminaison HTTP sont les informations d’identification de l’utilisateur HTTP configurées pour le cluster HDInsight ; les informations d’identification du point de terminaison SSH sont les informations d’identification SSH configurées pour le cluster HDInsight.

## <a name="troubleshoot"></a>Dépanner
Consultez [Dépanner l’installation](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation).

## <a name="next-steps"></a>Étapes suivantes
* [Installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md): découvrez comment déployer des applications HDInsight inédites vers HDInsight.
* [Publier des applications HDInsight](hdinsight-apps-publish-applications.md) : découvrez comment publier vos applications HDInsight personnalisées sur la Place de marché Azure.
* [MSDN : Installer une application HDInsight](/rest/api/hdinsight/hdinsight-application) : découvrez comment définir les applications HDInsight.
* [Personnalisation de clusters HDInsight basés sur Linux à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md): apprenez à utiliser l’action de script pour installer des applications supplémentaires.
* [Créer des clusters Apache Hadoop sous Linux dans HDInsight à l’aide de modèles Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md) : apprenez à appeler des modèles Resource Manager pour la création de clusters HDInsight.
* [Utiliser des nœuds de périmètre vides dans HDInsight](hdinsight-apps-use-edge-node.md): apprenez à utiliser un nœud de périmètre vide pour accéder au cluster HDInsight, tester des applications HDInsight et héberger des applications HDInsight.