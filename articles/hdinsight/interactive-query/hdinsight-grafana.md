---
title: Utiliser Grafana sur Azure HDInsight
description: Découvrez comment accéder au tableau de bord Grafana avec des clusters Apache Hadoop dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/27/2019
ms.openlocfilehash: 7750544367044cab2a0243577c200025bb9ff693
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86083030"
---
# <a name="access-grafana-in-azure-hdinsight"></a>Accéder à Grafana dans Azure HDInsight

[Grafana](https://grafana.com/) est un générateur open source de tableaux de bord et de graphes qui est couramment utilisé. Il est doté de nombreuses fonctionnalités : les utilisateurs peuvent créer des tableaux de bord personnalisables et pouvant être partagés, mais aussi se servir de tableaux de bord basés sur un script/modèle, de l’intégration LDAP, de multiples sources de données, etc.

Dans Azure HDInsight, Grafana est actuellement pris en charge avec les types de cluster Spark, Hbase, Kafka et Interactive Query. Il n’est pas pris en charge pour les clusters avec le Pack Sécurité Entreprise activé.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-apache-hadoop-cluster"></a>Créer un cluster Apache Hadoop

Consultez [Créer des clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md). Pour le **Type de cluster**, sélectionnez **Spark**, **Kafka**, **HBase** ou **Interactive Query**.

## <a name="access-the-grafana-dashboard"></a>Accéder au tableau de bord Grafana

1. Dans un navigateur Web, accédez à `https://CLUSTERNAME.azurehdinsight.net/grafana/` où CLUSTERNAME est le nom de votre cluster.

1. Entrez les informations d’identification utilisateur du cluster Hadoop.

1. Le tableau de bord Grafana s’affiche et se présente comme dans cet exemple :

    ![Tableau de bord web Grafana de HDInsight](./media/hdinsight-grafana/hdinsight-grafana-dashboard.png "Tableau de bord Grafana de HDInsight")

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, effectuez les étapes suivantes pour supprimer le cluster que vous avez créé :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Dans la zone **Recherche** située en haut, tapez **HDInsight**.

1. Sous **Services**, sélectionnez **Clusters HDInsight**.

1. Dans la liste des clusters HDInsight qui s’affiche, sélectionnez le **...** en regard du cluster que vous avez créé.

1. Sélectionnez **Supprimer**. Sélectionnez **Oui**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’analyse des données avec HDInsight, consultez les articles suivants :

* [Utilisation d’Apache Hive avec HDInsight](../hadoop/hdinsight-use-hive.md).

* [Utilisation de MapReduce avec HDInsight](../hadoop/hdinsight-use-mapreduce.md).

* [Prise en main de Visual Studio Hadoop Tools pour HDInsight](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).
