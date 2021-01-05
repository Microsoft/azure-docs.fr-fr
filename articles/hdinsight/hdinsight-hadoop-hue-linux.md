---
title: Hue avec Hadoop sur les clusters HDInsight basés sur Linux - Azure
description: Apprenez comment installer Hue sur les clusters HDInsight et utiliser le tunnel pour acheminer les demandes de Hue. Utilisez Hue pour parcourir le stockage et exécuter Hive ou Pig.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 03/31/2020
ms.openlocfilehash: e80df5d1c3d2b2195e76622964406cc65c933a63
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546192"
---
# <a name="install-and-use-hue-on-hdinsight-hadoop-clusters"></a>Installation et utilisation de Hue sur des clusters HDInsight Hadoop

Apprenez comment installer Hue sur les clusters HDInsight et utiliser le tunnel pour acheminer les demandes de Hue.

## <a name="what-is-hue"></a>Qu’est-ce que Hue ?

Hue est un ensemble d’applications web permettant d’interagir avec un cluster Apache Hadoop. Vous pouvez utiliser Hue pour parcourir le stockage associé à un cluster Hadoop (WASB, dans le cas de clusters HDInsight), exécuter les travaux Hive et les scripts Pig, etc. Les composants suivants sont disponibles avec l’installation de Hue sur un cluster HDInsight Hadoop.

* Éditeur Beeswax Hive
* Apache Pig
* Gestionnaire de Metastore
* Apache Oozie
* FileBrowser (qui communique avec le conteneur WASB par défaut)
* Explorateur de travaux

> [!WARNING]  
> Les composants fournis avec le cluster HDInsight bénéficient d’une prise en charge totale, et le support Microsoft vous aidera à identifier et à résoudre les problèmes liés à ces composants.
>
> Les composants personnalisés bénéficient d’un support commercialement raisonnable pour vous aider à résoudre le problème. Cela signifie SOIT que le problème pourra être résolu, SOIT que vous serez invité à affecter les ressources disponibles pour les technologies Open Source. Il existe par exemple de nombreux sites communautaires, comme : [Page de questions Microsoft Q&A pour HDInsight](/answers/topics/azure-hdinsight.html),[https://stackoverflow.com](https://stackoverflow.com). Par ailleurs, les projets Apache ont des sites de projet sur [https://apache.org](https://apache.org), par exemple : [Hadoop](https://hadoop.apache.org/).

## <a name="install-hue-using-script-actions"></a>Installer Hue à l’aide d’actions de script

Utilisez les informations du tableau ci-dessous pour votre action de script. Pour obtenir des instructions sur l’utilisation des actions de script, consultez [Personnaliser des clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md).

> [!NOTE]  
> Pour installer Hue sur des clusters HDInsight, il est recommandé d’utiliser une taille de nœud principal égale ou supérieure à A4 (8 cœurs, 14 Go de mémoire).

|Propriété |Valeur |
|---|---|
|Type de script :|- Personnalisé|
|Nom|Installer Hue|
|URI de script bash|`https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh`|
|Types de nœud :|Head|

## <a name="use-hue-with-hdinsight-clusters"></a>Utilisez Hue avec les clusters HDInsight

Vous ne pouvez avoir qu’un seul compte d’utilisateur Hue sur des clusters standard. Pour l’accès multi-utilisateur, activez [Pack Sécurité Entreprise](./domain-joined/hdinsight-security-overview.md) sur le cluster. Un tunneling SSH est le seul moyen d’accéder à Hue sur le cluster une fois celui-ci en cours d’exécution. Le tunneling via SSH autorise le trafic à atteindre directement le nœud principal du cluster exécutant Hue. Une fois que le cluster a terminé le provisionnement, effectuez les étapes suivantes pour utiliser Hue sur un cluster HDInsight.

> [!NOTE]  
> Nous vous recommandons de suivre les instructions ci-dessous sur le navigateur web Firefox.

1. Utilisez les informations de la section [Utilisation de SSH Tunneling pour accéder à l’interface web Apache Ambari, ResourceManager, JobHistory, NameNode, Oozie et d’autres interfaces web](hdinsight-linux-ambari-ssh-tunnel.md) pour créer un tunnel SSH entre votre système client et le cluster HDInsight, puis configurer votre navigateur web pour utiliser le tunnel SSH en tant que proxy.

1. Utilisez la [commande ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Une fois connecté, utilisez la commande suivante pour obtenir le nom de domaine complet du nœud principal primaire :

    ```bash
    hostname -f
    ```

    Cette commande renvoie un nom similaire à ce qui suit :

    ```output
    myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.cloudapp.net
    ```

    Il s’agit du nom d’hôte du nœud principal primaire sur lequel se trouve le site web Hue.

1. Utilisez le navigateur pour ouvrir le portail Hue à l’adresse `http://HOSTNAME:8888`. Remplacez HOSTNAME par le nom que vous avez obtenu à l’étape précédente.

   > [!NOTE]  
   > Lorsque vous vous connectez pour la première fois, vous êtes invité à créer un compte pour vous connecter au portail Hue. Les informations d’identification que vous spécifiez ici seront limitées au portail et ne sont pas liées à l’administrateur ou aux informations d’identification de l’utilisateur SSH spécifié lors de la configuration du cluster.

    ![Portail Hue sur HDInsight - Fenêtre de connexion](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-login.png "Spécifier les informations d’identification pour le portail Hue")

### <a name="run-a-hive-query"></a>Exécution d'une tâche Hive

1. À partir du portail Hue, sélectionnez **Éditeurs de requête** , puis sélectionnez **Hive** pour ouvrir l’éditeur Hive.

    ![Portail Hue sur HDInsight - Utiliser l’éditeur Hive](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-use-hive.png "Utiliser Hive")

2. Sous l’onglet **Aide** , sous **Base de données** , vous devez voir **hivesampletable** . Il s’agit d’une table d’échantillon qui est livrée avec tous les clusters Hadoop sur HDInsight. Saisissez un exemple de requête dans le volet de droite et vérifiez la sortie sur l’onglet **Résultats** dans le volet inférieur ci-dessous, comme illustré dans la capture d’écran.

    ![Portail Hue sur HDInsight - Requête Hive](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-hive-query.png "Exécuter une requête Hive")

    Vous pouvez également utiliser l’onglet **Graphique** pour afficher une représentation visuelle du résultat.

### <a name="browse-the-cluster-storage"></a>Parcourir le stockage de cluster

1. Dans le portail Hue,sélectionnez **Explorateur de fichiers** dans l’angle supérieur droit de la barre de menus.
2. Par défaut, l’explorateur de fichiers s’ouvre sur le répertoire **/user/myuser** . Sélectionnez la barre oblique située juste avant le répertoire de l’utilisateur dans le chemin pour accéder à la racine du conteneur de stockage Azure associé au cluster.

    ![Portail Hue sur HDInsight - Explorateur de fichiers](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-file-browser.png "Utiliser l’Explorateur de fichiers")

3. Cliquez avec le bouton droit sur un fichier ou un dossier pour afficher les opérations disponibles. Utilisez le bouton **Télécharger** situé dans le coin droit pour télécharger des fichiers dans le répertoire actif. Utilisez le bouton **Nouveau** pour créer des fichiers ou répertoires.

> [!NOTE]  
> L’Explorateur de fichiers Hue peut afficher uniquement le contenu du conteneur par défaut associé au cluster HDInsight. Les comptes/conteneurs de stockage supplémentaires que vous avez peut-être associés au cluster ne seront pas accessibles via l’Explorateur de fichiers. Toutefois, les autres conteneurs associés au cluster seront toujours accessibles pour les travaux Hive. Par exemple, si vous saisissez la commande `dfs -ls wasbs://newcontainer@mystore.blob.core.windows.net` dans l’éditeur Hive, vous pouvez voir le contenu des conteneurs supplémentaires. Dans cette commande, **newcontainer** n’est pas le conteneur par défaut associé à un cluster.

## <a name="important-considerations"></a>Points importants à prendre en compte

1. Le script utilisé pour installer Hue exécute l’opération uniquement sur le nœud principal primaire du cluster.

1. Pendant l’installation, plusieurs services Hadoop (HDFS, fils, RM2, Oozie) sont redémarrés pour mettre à jour la configuration. Une fois que le script a terminé l’installation de Hue, le démarrage d’autres services Hadoop peut prendre du temps. Cela peut affecter dans un premier temps les performances de Hue. Une fois que tous les services ont démarré, Hue est complètement fonctionnel.

1. Hue ne comprend pas les travaux Apache Tez, qui est le paramétrage par défaut actuel pour Hive. Si vous souhaitez utiliser MapReduce comme moteur d’exécution de Hive, mettez à jour le script pour utiliser la commande suivante dans votre script :

   `set hive.execution.engine=mr;`

1. Avec les clusters Linux, vous pouvez avoir un scénario dans lequel vos services fonctionnent sur le nœud principal primaire alors que le Gestionnaire de ressources s’exécute sur le nœud principal secondaire. Un tel scénario peut entraîner des erreurs (illustrées ci-dessous) lors de l’utilisation de Hue pour afficher les détails des travaux EN COURS sur le cluster. Toutefois, vous pouvez afficher les détails du travail lorsque la tâche est terminée.

   ![Portail Hue - Exemple de message d’erreur](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-error.png "Erreur de portail Hue")

   Il s’agit d’un problème connu. Pour résoudre ce problème, modifiez Ambari afin que le Gestionnaire de ressources actif s’exécute également sur le nœud principal primaire.

1. Hue connaît WebHDFS, tandis que les clusters HDInsight utilisent le stockage Azure à l’aide de `wasbs://`. Par conséquent, le script personnalisé utilisé avec l’action de script installe WebWasb, qui est un service compatible WebHDFS permettant de communiquer avec WASB. Donc, bien que le portail Hue indique HDFS à certains endroits (comme lorsque vous déplacez votre souris sur l’ **Explorateur de fichiers** ), il doit être compris comme WASB.

## <a name="next-steps"></a>Étapes suivantes

[Installation de R sur des clusters HDInsight](./r-server/r-server-overview.md). Utilisez la personnalisation de clusters pour installer R sur des clusters HDInsight Hadoop. R se compose d'un langage et d'un environnement open source destinés au calcul de statistiques. Il intègre des centaines de fonctions statistiques et possède son propre langage de programmation qui regroupe des aspects de la programmation fonctionnelle et de la programmation orientée objet. Il offre également des fonctionnalités graphiques étendues.