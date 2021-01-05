---
title: Rendre opérationnel ML Services sur HDInsight - Azure
description: Découvrez comment faire fonctionner votre modèle de données pour faire des prédictions avec ML Services dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/27/2018
ms.openlocfilehash: 20159cf911670eb70fd5757991c07b63b3f1776b
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536264"
---
# <a name="operationalize-ml-services-cluster-on-azure-hdinsight"></a>Rendre opérationnel un cluster ML Services sur Azure HDInsight

Après avoir utilisé un cluster ML Services dans HDInsight pour effectuer votre modélisation de données, vous pouvez rendre opérationnel le modèle pour élaborer des prédictions. Cet article explique comment accomplir cette tâche.

## <a name="prerequisites"></a>Prérequis

* Un cluster ML Services sur HDInsight. Consultez [Création clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **ML Services** pour le **Type de cluster** .

* Un client Secure Shell (SSH) : un client SSH est utilisé pour se connecter à distance au cluster HDInsight et exécuter des commandes directement sur celui-ci. Pour en savoir plus, voir [Utilisation de SSH avec Hadoop Linux sur HDInsight depuis Linux, Unix ou OS X](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="operationalize-ml-services-cluster-with-one-box-configuration"></a>Rendre opérationnel un cluster ML Services avec une configuration à boîtier unique

> [!NOTE]  
> Les étapes ci-dessous s’appliquent à R Server 9.0 et ML Server 9.1. Pour ML Server 9.3, consultez [Utiliser l’outil d’administration pour gérer la configuration d’opérationnalisation](/machine-learning-server/operationalize/configure-admin-cli-launch).

1. Utilisez SSH au sein du nœud de périmètre.

    ```bash
    ssh USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net
    ```

    Pour savoir comment utiliser SSH avec Azure HDInsight, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. Basculez du répertoire vers la version appropriée et exécutez la commande sudo sur la DLL dotnet : 

    - Pour Microsoft ML Server 9.1 :

        ```bash
        cd /usr/lib64/microsoft-r/rserver/o16n/9.1.0
        sudo dotnet Microsoft.RServer.Utils.AdminUtil/Microsoft.RServer.Utils.AdminUtil.dll
        ```

    - Pour Microsoft R Server 9.0 :

        ```bash
        cd /usr/lib64/microsoft-deployr/9.0.1
        sudo dotnet Microsoft.DeployR.Utils.AdminUtil/Microsoft.DeployR.Utils.AdminUtil.dll
        ```

1. Les options disponibles s’affichent. Choisissez la première option, comme illustré dans la capture d’écran suivante, pour **configurer ML Server pour l’opérationnalisation** .

    ![Sélection dans l’utilitaire d’administration R Server](./media/r-server-operationalize/admin-util-one-box-1.png)

1. Vous devez maintenant choisir la configuration souhaitée pour rendre opérationnel ML Server. Sélectionnez la première option en entrant **A** .

    ![Rendre opérationnel ML Server dans l’utilitaire d’administration R Server](./media/r-server-operationalize/admin-util-one-box-2.png)

1. Quand vous y êtes invité, saisissez et saisissez à nouveau le mot de passe d’un utilisateur administrateur local.

1. Vous devriez voir des sorties suggérant que l’opération a réussi. Vous êtes également invité à sélectionner une autre option dans le menu. Entrez E pour revenir au menu principal.

    ![Opération réussie dans l’utilitaire d’administration R Server](./media/r-server-operationalize/admin-util-one-box-3.png)

1. Vous pouvez éventuellement effectuer des vérifications de diagnostic en exécutant un test de diagnostic, comme suit :

    a. Dans le menu principal, sélectionnez **6** pour exécuter les tests de diagnostic.

    ![Diagnostics dans l’utilitaire d’administration R Server](./media/r-server-operationalize/hdinsight-diagnostic1.png)

    b. Dans le menu Diagnostic Tests (Tests de diagnostic), sélectionnez **A** . Quand vous y êtes invité, entrez le mot de passe que vous avez fourni pour l’utilisateur administrateur local.

    ![Test dans l’utilitaire d’administration R Server](./media/r-server-operationalize/hdinsight-diagnostic2.png)

    c. Vérifiez que la sortie indique que l’intégrité globale est satisfaisante, comme illustré ci-dessous.

    ![Niveau d’intégrité satisfaisant dans l’utilitaire d’administration R Server](./media/r-server-operationalize/hdinsight-diagnostic3.png)

    d. Dans les options de menu présentées, entrez **E** pour revenir au menu principal, puis **8** pour quitter l’utilitaire d’administration.

### <a name="long-delays-when-consuming-web-service-on-apache-spark"></a>Retards importants lors de l’utilisation d’un service web sur Apache Spark

Si vous rencontrez d’importants retards lorsque vous utilisez un service web créé avec des fonctions mrsdeploy dans un contexte d’exécution Apache Spark, vous devrez peut-être ajouter des dossiers manquants. L’application Spark appartient à un utilisateur nommé «  *rserve2*  » à chaque fois qu’elle est appelée depuis un service web à l’aide de fonctions mrsdeploy. Pour contourner ce problème :

```r
# Create these required folders for user 'rserve2' in local and hdfs:

hadoop fs -mkdir /user/RevoShare/rserve2
hadoop fs -chmod 777 /user/RevoShare/rserve2

mkdir /var/RevoShare/rserve2
chmod 777 /var/RevoShare/rserve2


# Next, create a new Spark compute context:

rxSparkConnect(reset = TRUE)
```

À ce stade, la configuration de l’opérationnalisation est terminée. Vous pouvez désormais utiliser le package `mrsdeploy` sur votre RClient pour vous connecter à l’opérationnalisation sur le nœud de périphérie et commencer à utiliser ses fonctionnalités, telles que l’[exécution à distance](/machine-learning-server/r/how-to-execute-code-remotely) et les [services web](/machine-learning-server/operationalize/concept-what-are-web-services). Selon que votre cluster est configuré sur un réseau virtuel ou non, vous devrez peut-être configurer le tunneling de réacheminement du port via une connexion SSH. Les sections suivantes expliquent comment configurer ce tunnel.

### <a name="ml-services-cluster-on-virtual-network"></a>Cluster ML Services sur un réseau virtuel

Assurez-vous que vous autorisez le trafic via le port 12800 vers le nœud de périmètre. De cette façon, vous pouvez utiliser le nœud de périmètre pour vous connecter à la fonctionnalité d’opérationnalisation.

```r
library(mrsdeploy)

remoteLogin(
    deployr_endpoint = "http://[your-cluster-name]-ed-ssh.azurehdinsight.net:12800",
    username = "admin",
    password = "xxxxxxx"
)
```

Si `remoteLogin()` ne peut pas se connecter au nœud de périmètre, mais si vous pouvez exécuter SSH sur ce dernier, vous devez alors vérifier si la règle permettant d’autoriser le trafic sur le port 12800 a été configurée correctement ou non. Si vous continuez à rencontrer ce problème, configurez le tunneling de réacheminement du port via SSH pour le contourner. Pour connaître la marche à suivre, consultez la section suivante :

### <a name="ml-services-cluster-not-set-up-on-virtual-network"></a>Cluster ML Services non configuré sur un réseau virtuel

Si votre cluster n’est pas configuré sur un réseau virtuel ou si vous rencontrez des problèmes de connectivité via un réseau virtuel, vous pouvez utiliser le tunneling de réacheminement du port SSH :

```bash
ssh -L localhost:12800:localhost:12800 USERNAME@CLUSTERNAME-ed-ssh.azurehdinsight.net
```

Une fois votre session SSH active, le trafic à partir du port 12800 de votre machine locale est transféré vers le port du nœud de périmètre 12800, via une session SSH. Vérifiez que vous utilisez `127.0.0.1:12800` dans votre méthode `remoteLogin()`. Cela vous permet de vous connecter à l’opérationnalisation du nœud de périphérie via un réacheminement de port.

```r
library(mrsdeploy)

remoteLogin(
    deployr_endpoint = "http://127.0.0.1:12800",
    username = "admin",
    password = "xxxxxxx"
)
```

## <a name="scale-operationalized-compute-nodes-on-hdinsight-worker-nodes"></a>Mise à l’échelle des nœuds de calcul opérationnalisés sur les nœuds Worker HDInsight

Pour mettre à l’échelle les nœuds de calcul, vous devez d’abord désactiver les nœuds Worker, puis configurer les nœuds de calcul sur les nœuds de travail désactivés.

### <a name="step-1-decommission-the-worker-nodes"></a>Étape 1 : Désactiver les nœuds Worker

Le cluster ML Services n’est pas géré via [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html). Si les nœuds Worker ne sont pas désactivés, le gestionnaire de ressources YARN ne fonctionne pas comme prévu, car il n’a pas connaissance des ressources prises en charge par le serveur. Afin d’éviter ce problème, nous vous recommandons de désactiver les nœuds Worker avant d’effectuer un scale-out des nœuds de calcul.

Pour désactiver les nœuds Worker, procédez comme suit :

1. Connectez-vous à la console Ambari du cluster et cliquez sur l’onglet **Hôtes** .

1. Sélectionnez les nœuds Worker (à désactiver).

1. Cliquez sur **Actions** > **Hôtes sélectionnés** > **Hôtes** > **Activer le mode de maintenance** . Par exemple, dans l’image suivante, nous avons sélectionné wn3 et wn4 pour la désactivation.  

   ![Apache Ambari - Activer le mode Maintenance](./media/r-server-operationalize/get-started-operationalization.png)  

* Sélectionnez **Actions** > **Hôtes sélectionnés** > **DataNodes** > cliquez sur **Désactiver** .
* Sélectionnez **Actions** > **Hôtes sélectionnés** > **NodeManagers** > cliquez sur **Désactiver** .
* Sélectionnez **Actions** > **Hôtes sélectionnés** > **DataNodes** > cliquez sur **Arrêter** .
* Sélectionnez **Actions** > **Hôtes sélectionnés** > **NodeManagers** > cliquez sur **Arrêter** .
* Sélectionnez **Actions** > **Hôtes sélectionnés** > **Hôtes** > cliquez sur **Arrêter tous les composants** .
* Désélectionnez les nœuds Worker et sélectionnez les nœuds principaux.
* Sélectionnez **Actions** > **Hôtes sélectionnés** > **Hôtes** > **Redémarrer tous les composants** .

### <a name="step-2-configure-compute-nodes-on-each-decommissioned-worker-nodes"></a>Étape 2 : Configurer les nœuds de calcul sur chaque nœud Worker désactivé

1. Utilisez SSH dans chaque nœud Worker désactivé.

1. Exécutez l’utilitaire d’administration en utilisant la DLL adaptée au cluster ML Services dont vous disposez. Pour ML Server 9.1, exécutez la commande suivante :

    ```bash
    dotnet /usr/lib64/microsoft-deployr/9.0.1/Microsoft.DeployR.Utils.AdminUtil/Microsoft.DeployR.Utils.AdminUtil.dll
    ```

1. Entrez **1** pour sélectionner l’option **Configurer ML Server pour l’opérationnalisation** .

1. Entrez **C** pour sélectionner l’option `C. Compute node`. Cette opération permet de configurer un nœud de calcul sur le nœud Worker.

1. Quittez l’utilitaire d’administration.

### <a name="step-3-add-compute-nodes-details-on-web-node"></a>Étape 3 : Ajouter des détails sur les nœuds de calcul sur le nœud web

Une fois que tous les nœuds Worker désactivés sont configurés pour exécuter les nœuds de calcul, revenez au nœud de périphérie et ajoutez les adresses IP des nœuds Worker désactivés dans la configuration du nœud web ML Server :

1. Utilisez SSH au sein du nœud de périmètre.

1. Exécutez `vi /usr/lib64/microsoft-deployr/9.0.1/Microsoft.DeployR.Server.WebAPI/appsettings.json`.

1. Recherchez la section « Uris » et ajoutez l’adresse IP du nœud Worker ainsi que les détails du port.

    ```json
    "Uris": {
        "Description": "Update 'Values' section to point to your backend machines. Using HTTPS is highly recommended",
        "Values": [
            "http://localhost:12805", "http://[worker-node1-ip]:12805", "http://[workder-node2-ip]:12805"
        ]
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Gérer un cluster ML Services sur HDInsight](r-server-hdinsight-manage.md)
* [Options de contexte de calcul pour un cluster ML Services sur HDInsight](r-server-compute-contexts.md)
* [Options de stockage Azure pour un cluster ML Services sur HDInsight](r-server-storage.md)