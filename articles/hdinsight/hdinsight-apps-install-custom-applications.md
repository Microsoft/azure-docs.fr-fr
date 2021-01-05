---
title: Installer vos applications Apache Hadoop personnalisées sur Azure HDInsight
description: Découvrez comment installer des applications HDInsight sur les clusters Apache Hadoop dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: dca248d217f4fb90d41c0a98a42b5eda4783913f
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825204"
---
# <a name="install-custom-apache-hadoop-applications-on-azure-hdinsight"></a>Installer des applications Apache Hadoop personnalisées sur Azure HDInsight

Dans cet article, vous allez découvrir comment installer une application [Apache Hadoop](https://hadoop.apache.org/) sur Azure HDInsight qui n’a pas été publiée sur le portail Azure. Dans cet article, vous allez installer l’application [Hue](https://gethue.com/).

Une application HDInsight est une application que les utilisateurs peuvent installer sur un cluster HDInsight.  Ces applications peuvent être développées par Microsoft, par des éditeurs de logiciels indépendants (ISV) ou par vous-même.  

## <a name="prerequisites"></a>Prérequis

Pour installer des applications HDInsight sur un cluster HDInsight existant, vous devez disposer d’un cluster HDInsight. Pour en créer un, consultez [Créer des clusters](hadoop/apache-hadoop-linux-tutorial-get-started.md). Vous pouvez également installer des applications HDInsight lorsque vous créez un cluster HDInsight.

## <a name="install-hdinsight-applications"></a>Install custom HDInsight applications

Les applications HDInsight peuvent être installées sur un cluster HDInsight existant ou lorsque vous créez un cluster. Pour définir des modèles Azure Resource Manager, consultez [MSDN : installer une application HDInsight](/rest/api/hdinsight/hdinsight-application).

Fichiers nécessaires pour le déploiement de cette application (Hue) :

* [azuredeploy.json](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/azuredeploy.json) : modèle Azure Resource Manager pour installer l’application HDInsight. Voir [MSDN : installer une application HDInsight](/rest/api/hdinsight/hdinsight-application) pour développer votre propre modèle Resource Manager.
* [hue-install_v0.sh](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/scripts/Hue-install_v0.sh) : action de script appelée par le modèle Resource Manager pour la configuration du nœud de périphérie.
* [hue-binaries.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz) : fichier binaire hue appelé à partir de hui-install_v0.sh.
* [hue-binaries-14-04.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz) : fichier binaire hue appelé à partir de hui-install_v0.sh.
* [webwasb-tomcat.tar.gz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/webwasb-tomcat.tar.gz) : exemple d’application web (Tomcat) appelée à partir de hui-install_v0.sh.

### <a name="to-install-hue-to-an-existing-hdinsight-cluster"></a>Pour installer Hue sur un cluster HDInsight existant

1. Sélectionnez l’image suivante pour vous connecter à Azure et ouvrir le modèle Resource Manager dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FHue%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Le modèle Resource Manager se trouve à l’emplacement [https://github.com/hdinsight/Iaas-Applications/tree/master/Hue](https://github.com/hdinsight/Iaas-Applications/tree/master/Hue).  Pour découvrir comment écrire ce modèle Resource Manager, consultez [MSDN : installer une application HDInsight](/rest/api/hdinsight/hdinsight-application).

1. Sélectionnez le **groupe de ressources** existant qui contient votre cluster dans la liste déroulante. Vous devez utiliser le même groupe de ressources que le cluster.

1. entrez le nom du cluster où vous souhaitez installer l’application. Ce cluster doit être un cluster existant.

1. Cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.

1. Sélectionnez **Achat**.

Vous pouvez voir l’état de l’installation dans la vignette épinglée sur le tableau de bord du portail ainsi que la notification du portail (cliquez sur l’icône en forme de cloche en haut du portail).  L’installation de l’application prend environ 10 minutes.

### <a name="to-install-hue-while-creating-a-cluster"></a>Pour installer Hue lors de la création d’un cluster

1. Sélectionnez l’image suivante pour vous connecter à Azure et ouvrir le modèle Resource Manager dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhdinsightapps%2Fcreate-linux-based-hadoop-cluster-in-hdinsight.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Le modèle Resource Manager se trouve à l’emplacement [https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json](https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json).  Pour découvrir comment écrire ce modèle Resource Manager, consultez [MSDN : installer une application HDInsight](/rest/api/hdinsight/hdinsight-application).

2. Suivez les instructions pour créer le cluster et installer Hue. Pour plus d’informations sur la création de clusters HDInsight, consultez [Création de clusters Hadoop basés sur Linux dans HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

### <a name="other-installation-methods"></a>Autres méthodes d’installation

Outre le portail Azure, vous pouvez également utiliser [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-powershell) et [l’interface de ligne de commande Azure](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-azure-cli) pour appeler des modèles Resource Manager.

## <a name="validate-the-installation"></a>validation de l'installation

Vous pouvez vérifier l’état de l’application sur le portail Azure pour valider l’installation de l’application. Vous pouvez également valider la création appropriée de tous les points de terminaison HTTP et du site web, le cas échéant.

Pour **Hue**, suivez ces étapes :

### <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le cluster dans lequel vous avez installé l’application.
1. Dans le menu **Paramètres**, sélectionnez **Applications**.
1. Sélectionnez **hue** dans la liste pour voir les propriétés.  
1. Sélectionnez le lien vers la page web pour valider le site web.

### <a name="azure-cli"></a>Azure CLI

Remplacez `CLUSTERNAME` et `RESOURCEGROUP` par les valeurs pertinentes, puis entrez les commandes suivantes :

* Pour lister toutes les applications pour le cluster HDInsight.

    ```azurecli
    az hdinsight application list --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

* Pour récupérer les propriétés de l’application spécifiée.

    ```azurecli
    az hdinsight application show --name hue --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

## <a name="troubleshoot-the-installation"></a>Dépanner l’installation

Vous pouvez vérifier l’état de l’installation de l’application dans la notification du portail (cliquez sur l’icône en forme de cloche en haut du portail).

En cas d’échec de l’installation d’une application, des messages d’erreur et des informations de débogage s’affichent à trois emplacements différents :

* Applications HDInsight : informations générales relatives à l’erreur.

    Ouvrez le cluster à partir du portail, puis sélectionnez Applications dans Paramètres :

    ![Erreur d’installation des applications hdinsight](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-error.png)

* Action de script HDInsight : si le message d’erreur des applications HDInsight indique un échec d’action de script, le volet Actions de script affichera davantage de détails sur l’échec de script.

    Sélectionnez Action de script dans Paramètres. L’historique des actions de script affiche les messages d’erreur

    ![Erreur de l’action de script des applications hdinsight](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-script-action-error.png)

* Interface utilisateur web d’Apache Ambari : si le script d’installation était à l’origine de l’échec, utilisez l’interface utilisateur web d’Ambari pour consulter les journaux d’activité complets des scripts d’installation.

    Pour plus d’informations, consultez [Résoudre les problèmes liés aux actions de script](./troubleshoot-script-action.md).

## <a name="remove-hdinsight-applications"></a>Supprimer des applications HDInsight

### <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le cluster dans lequel vous avez installé l’application.
1. Dans le menu **Paramètres**, sélectionnez **Applications**.
1. Cliquez avec le bouton droit sur l’application à supprimer, puis sélectionnez **Supprimer**.
1. Sélectionnez **Oui** pour confirmer.

### <a name="azure-cli"></a>Azure CLI

Remplacez `NAME`, `CLUSTERNAME` et `RESOURCEGROUP` par les valeurs pertinentes, puis entrez la commande suivante :

```azurecli
az hdinsight application delete --name NAME --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
```

## <a name="next-steps"></a>Étapes suivantes

* [MSDN : installer une application HDInsight](/rest/api/hdinsight/hdinsight-application) : découvrez comment développer des applications Resource Manager pour le déploiement d’applications HDInsight.
* [Installer des applications HDInsight](hdinsight-apps-install-applications.md) : découvrez comment installer une application HDInsight sur vos clusters.
* [Publier des applications HDInsight](hdinsight-apps-publish-applications.md) : découvrez comment publier vos applications HDInsight personnalisées sur la Place de marché Azure.
* [Personnalisation de clusters HDInsight basés sur Linux à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md): apprenez à utiliser l’action de script pour installer des applications supplémentaires.
* [Créer des clusters Apache Hadoop sous Linux dans HDInsight à l’aide de modèles Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md) : apprenez à appeler des modèles Resource Manager pour la création de clusters HDInsight.
* [Utiliser des nœuds de périmètre vides dans HDInsight](hdinsight-apps-use-edge-node.md): apprenez à utiliser un nœud de périmètre vide pour accéder au cluster HDInsight, tester des applications HDInsight et héberger des applications HDInsight.
