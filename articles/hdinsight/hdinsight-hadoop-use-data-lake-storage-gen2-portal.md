---
title: Créer Azure HDInsight – Azure Data Lake Storage Gen2 – Portail
description: Découvrez comment utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight à partir du portail.
author: guyhay
ms.author: guyhay
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 09/17/2020
ms.openlocfilehash: 104424c1e3bd1df69106db7da45b744755b51e82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91858720"
---
# <a name="create-a-cluster-with-data-lake-storage-gen2-using-the-azure-portal"></a>Créer un cluster avec Data Lake Storage Gen2 à partir du portail Azure

Le portail Azure est un outil web qui permet de gérer les services et les ressources hébergés dans le cloud Microsoft Azure. Cet article vous montre comment créer des clusters Azure HDInsight Linux par le biais du portail. Des informations supplémentaires sont disponibles dans [Créer des clusters HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Pour créer un cluster HDInsight qui utilise Data Lake Storage Gen2 comme stockage, effectuez les étapes suivantes pour configurer un compte qui dispose d’un espace de noms hiérarchique.

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur

Créez une identité managée affectée à l’utilisateur, si vous n’en avez pas encore.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le coin supérieur gauche, cliquez sur **Créer une ressource**.
1. Dans la zone de recherche, tapez **user assigned (utilisateur affecté)** , puis cliquez sur **Identité managée affectée par l'utilisateur**.
1. Cliquez sur **Créer**.
1. Entrez un nom pour votre identité managée, sélectionnez l'abonnement, le groupe de ressources et l'emplacement correspondants.
1. Cliquez sur **Créer**.

Pour plus d’informations sur le fonctionnement des identités managées dans Azure HDInsight, consultez [Identités managées dans Azure HDInsight](hdinsight-managed-identities.md).

![Créer une identité managée attribuée par l’utilisateur](./media/hdinsight-hadoop-use-data-lake-storage-gen2/create-user-assigned-managed-identity-portal.png)

## <a name="create-a-storage-account-to-use-with-data-lake-storage-gen2"></a>Créer un compte de stockage à utiliser avec Data Lake Storage Gen2

Créez un compte de stockage à utiliser avec Azure Data Lake Storage Gen2.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le coin supérieur gauche, cliquez sur **Créer une ressource**.
1. Dans la zone de recherche, tapez **stockage**, puis cliquez sur **compte de stockage**.
1. Cliquez sur **Créer**.
1. Sur l'écran **Créer un compte de stockage** :
    1. Sélectionnez l’abonnement et le groupe de ressources correspondants.
    1. Attribuez un nom à votre compte de stockage avec Data Lake Storage Gen2.
    1. Cliquez sur l’onglet **Avancé**.
    1. Cliquez sur **Activé** en regard de **Espace de noms hiérarchique** sous **Data Lake Storage Gen2**.
    1. Cliquez sur **Vérifier + créer**.
    1. Cliquez sur **Créer**

Pour plus d'informations sur les autres options lors de la création d'un compte de stockage, voir [Démarrage rapide : Créez un compte de stockage pour Azure Data Lake Storage Gen2](../storage/blobs/create-data-lake-storage-account.md).

![Capture d’écran montrant la création du compte de stockage dans le portail Azure](./media/hdinsight-hadoop-use-data-lake-storage-gen2/azure-data-lake-storage-account-create-advanced.png)

## <a name="set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2"></a>Définir des autorisations pour l’identité managée sur Data Lake Storage Gen2

Affectez l’identité managée au rôle **Propriétaire des données Blob du stockage** sur le compte de stockage.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage.
1. Sélectionnez votre compte de stockage, puis sélectionnez **Contrôle d’accès (IAM)** pour afficher les paramètres de contrôle d’accès du compte. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles.

    ![Capture d’écran montrant les paramètres de contrôle d’accès du stockage](./media/hdinsight-hadoop-use-data-lake-storage-gen2/portal-access-control.png)

1. Sélectionnez le bouton **+ Ajouter une attribution de rôle** pour ajouter un nouveau rôle.
1. Dans la fenêtre **Ajouter une attribution de rôle**, sélectionnez le rôle **Propriétaire des données Blob du stockage**. Ensuite, sélectionnez l’abonnement qui contient le compte de stockage et l’identité managée. Ensuite, recherchez l’identité managée affectée par l’utilisateur que vous avez créée. Pour finir, sélectionnez l’identité managée pour qu’elle soit listée sous **Membres sélectionnés**.

    ![Capture d’écran montrant comment attribuer un rôle Azure](./media/hdinsight-hadoop-use-data-lake-storage-gen2/add-rbac-role3-window.png)

1. Sélectionnez **Enregistrer**. L’identité affectée par l’utilisateur que vous avez sélectionnée est maintenant listée sous le rôle sélectionné.
1. Une fois cette configuration initiale terminée, vous pouvez créer un cluster par le biais du portail. Le cluster doit être dans la même région Azure que le compte de stockage. Sous l’onglet **Stockage** du menu de création du cluster, sélectionnez les options suivantes :

    * Comme **Type de stockage principal**, sélectionnez **Azure Data Lake Storage Gen2**.
    * Sous **Compte de stockage principal**, recherchez et sélectionnez le compte de stockage nouvellement créé avec le stockage Data Lake Storage Gen2.

    * Sous **Identité**, sélectionnez l’identité managée affectée par l’utilisateur nouvellement créée.

        ![Paramètres de stockage pour l’utilisation de Data Lake Storage Gen2 avec Azure HDInsight](./media/hdinsight-hadoop-use-data-lake-storage-gen2/azure-portal-cluster-storage-gentwo.png)

    > [!NOTE]
    > * Pour ajouter un compte de stockage secondaire avec Data Lake Storage Gen2, au niveau du compte de stockage, attribuez simplement l’identité managée créée précédemment au nouveau Data Lake Storage Gen2 que vous voulez ajouter. Sachez que l’ajout d’un compte de stockage secondaire avec Data Lake Storage Gen2 via le panneau « Comptes de stockage supplémentaires » de HDInsight n’est pas pris en charge.
    > * Vous pouvez activer RA-GRS ou RA-ZRS sur le compte Stockage Blob Azure utilisé par HDInsight. La création d’un cluster sur le point de terminaison secondaire RA-GRS ou RA-ZRS n’est toutefois pas prise en charge.

## <a name="delete-the-cluster"></a>Supprimer le cluster

Consultez [Suppression d’un cluster HDInsight avec votre navigateur, PowerShell ou l’interface de ligne de commande Azure](./hdinsight-delete-cluster.md).

## <a name="troubleshoot"></a>Dépanner

Si vous rencontrez des problèmes lors de la création de clusters HDInsight, reportez-vous aux [exigences de contrôle d’accès](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un cluster HDInsight. Apprenez maintenant à l’utiliser.

### <a name="apache-spark-clusters"></a>Clusters Apache Spark

* [Personnaliser des clusters HDInsight Linux à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md)
* [Créer une application autonome avec Scala](spark/apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](spark/apache-spark-livy-rest-interface.md)
* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark sur HDInsight avec des outils décisionnels](spark/apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](spark/apache-spark-machine-learning-mllib-ipython.md)

### <a name="apache-hadoop-clusters"></a>Clusters Apache Hadoop

* [Utilisation d’Apache Hive avec HDInsight](hadoop/hdinsight-use-hive.md)
* [Utilisation de MapReduce avec HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Clusters Apache HBase

* [Bien démarrer avec Apache HBase sur HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Développer des applications Java pour Apache HBase sur HDInsight](hbase/apache-hbase-build-java-maven-linux.md)
