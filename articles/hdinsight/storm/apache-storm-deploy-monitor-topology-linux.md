---
title: Déploiement et gestion des topologies Apache Storm sur Azure HDInsight
description: Apprenez à déployer, surveiller et gérer des topologies Apache Storm à l’aide du tableau de bord Storm sur HDInsight basé sur Linux. Utilisez les outils Hadoop pour Visual Studio.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/18/2019
ms.openlocfilehash: 9b190b5847c7412344e2bb09fd4000985816219b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995588"
---
# <a name="deploy-and-manage-apache-storm-topologies-on-azure-hdinsight"></a>Déploiement et gestion des topologies Apache Storm sur Azure HDInsight

Ce document présente les principes fondamentaux de la gestion et du monitoring des topologies [Apache Storm](https://storm.apache.org/) qui s’exécutent sur des clusters Storm sur HDInsight.

## <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Storm sur HDInsight. Consultez la section [Création de clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Storm** dans le champ **Type de cluster**.

* (facultatif) Familiarité avec Secure Shell (SSH) et Secure Copy (SCP). Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* (Facultatif) Pour Visual Studio, le Kit de développement logiciel (SDK) Azure 2.5.1 ou une version ultérieure et Data Lake Tools pour Visual Studio. Pour plus d’informations, consultez [Apache Hadoop & Outils Visual Studio Data Lake](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

## <a name="submit-a-topology-using-visual-studio"></a>Soumettre une topologie en utilisant Visual Studio

Vous pouvez utiliser les outils de Data Lake pour Visual Studio pour envoyer du C# ou des topologies hybrides à votre cluster Storm. La procédure suivante utilise un exemple d’application. Pour plus d’informations sur la création de topologies à l’aide des outils de Data Lake, consultez [Topologies Apache Storm avec Visual Studio et C#](apache-storm-develop-csharp-visual-studio-topology.md).

1. Si vous n’avez pas encore installé la dernière version de Data Lake Tools pour Visual Studio, consultez [Utiliser Data Lake Tools pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

    > [!NOTE]  
    > Azure Data Lake et Stream Analytics Tools est le nouveau nom des outils HDInsight pour Visual Studio.
    >
    > Azure Data Lake et Stream Analytics Tools pour Visual Studio sont inclus dans la **charge de travail de développement Azure** pour Visual Studio 2019.

1. Démarrez Visual Studio.

1. Dans la fenêtre **Démarrer**, sélectionnez **Créer un projet**.

1. Dans la fenêtre **Créer un nouveau projet**, sélectionnez la zone de recherche, puis entrez `Storm`. Choisissez ensuite **Exemple Storm** dans la liste des résultats, puis sélectionnez **Suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, entrez le **Nom de projet**, puis accédez à ou créez un **Emplacement** auquel enregistrer le nouveau projet. Sélectionnez ensuite **Créer**.

    ![Configurer votre fenêtre de nouveau projet, Visual Studio](./media/apache-storm-deploy-monitor-topology-linux/apache-storm-sample1.png)

1. À partir de l’**Explorateur de serveurs**, cliquez avec le bouton droit sur **Azure**, sélectionnez **Se connecter à un abonnement Microsoft Azure...** , puis effectuez le processus de connexion.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet et choisissez **Envoyer à Storm sur HDInsight**.

    > [!NOTE]  
    > Si vous y êtes invité, entrez les informations d’identification de connexion pour votre abonnement Azure. Si vous disposez de plusieurs abonnements, connectez-vous à celui contenant votre cluster Storm sur HDInsight.

1. Dans la boîte de dialogue **Envoyer la topologie**, dans la liste déroulante **Cluster Storm**, choisissez votre cluster Storm sur HDInsight, puis sélectionnez **Envoyer**. Vous pouvez vérifier si l’envoi a été effectué correctement en examinant le volet **Sortie**.

## <a name="submit-a-topology-using-ssh-and-the-storm-command"></a>Soumettre une topologie avec SSH et la commande Storm

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Dans votre session ssh, utilisez la commande suivante pour démarrer l’exemple de topologie **WordCount** :

    ```bash
    storm jar /usr/hdp/current/storm-client/contrib/storm-starter/storm-starter-topologies-*.jar org.apache.storm.starter.WordCountTopology WordCount
    ```

    Cette commande démarre l’exemple de topologie WordCount sur le cluster. Cette topologie va générer des phrases de manière aléatoire, puis compter les occurrences de chaque mot dans ces phrases.

    > [!NOTE]  
    > Pendant l’envoi de la topologie au cluster, vous devez d’abord copier le fichier .jar contenant le cluster avant d’utiliser la commande `storm`. Vous pouvez utiliser la commande `scp` pour copier le fichier sur le cluster. Par exemple, entrez `scp FILENAME.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.jar`.
    >
    > L’exemple *WordCount* et d’autres exemples storm-starter sont déjà inclus dans votre cluster sous `/usr/hdp/current/storm-client/contrib/storm-starter/`.

## <a name="submit-a-topology-programmatically"></a>Soumettre une topologie par programme

Vous pouvez déployer par programme une topologie à l’aide du service Nimbus. [https://github.com/Azure-Samples/hdinsight-java-deploy-storm-topology](https://github.com/Azure-Samples/hdinsight-java-deploy-storm-topology) fournit un exemple d’application Java qui montre comment déployer et démarrer une topologie via le service Nimbus.

## <a name="monitor-and-manage-a-topology-in-visual-studio"></a>Surveiller et gérer une topologie dans Visual Studio

Lorsque vous soumettez une topologie à l’aide de Visual Studio, la fenêtre **Vue des topologies Storm** s’affiche. Sélectionnez la topologie à partir de la liste pour afficher des informations sur la topologie en cours d’exécution.

![Topologie de surveillance, fenêtre Vue des topologies Storm, Visual Studio](./media/apache-storm-deploy-monitor-topology-linux/visual-studio-monitor.png)

> [!NOTE]  
> Vous pouvez également afficher les **Topologies Storm** dans l’**Explorateur de serveurs**. Développez **Azure** > **HDInsight**, cliquez avec le bouton droit sur un Storm sur le cluster HDInsight, puis sélectionnez **Afficher les topologies Storm**.

Sélectionnez la forme des spouts et bolts pour afficher des informations sur ces composants. Une info-bulle avec des informations sur les composants s’affiche pour l’élément sélectionné.

### <a name="deactivate-and-reactivate-a-topology"></a>Désactiver et réactiver une topologie

La désactivation d’une topologie la met en pause jusqu’à ce qu’elle soit arrêtée ou réactivée. Pour effectuer ces opérations, utilisez les boutons **Désactiver** et **Réactiver** dans la zone **Actions** en haut de la fenêtre **Vue des topologies Storm**.

### <a name="rebalance-a-topology"></a>Rééquilibrer une topologie

Le rééquilibrage d’une topologie permet au système de réviser le parallélisme de la topologie. Par exemple, si vous avez redimensionné le cluster pour ajouter plus de nœuds, le rééquilibrage permet à une topologie d’identifier les nouveaux nœuds.

Pour rééquilibrer une topologie, utilisez le bouton **Rééquilibrer** dans la zone **Actions** de la fenêtre **Vue des topologies Storm**.

> [!WARNING]  
> Le rééquilibrage d’une topologie désactive la topologie, puis redistribue les workers uniformément sur le cluster avant de rétablir la topologie dans l’état où elle se trouvait avant le rééquilibrage. Si la topologie était active, elle redevient active. Si la topologie était désactivée, elle reste désactivée.

### <a name="kill-a-running-topology"></a>Arrêt d’une topologie en cours d’exécution

Les topologies Storm poursuivent leur exécution jusqu'à ce qu'elles soient arrêtées ou que le cluster soit supprimé. Pour arrêter une topologie, utilisez le bouton **Tuer** dans la zone **Actions**.

## <a name="monitor-and-manage-a-topology-using-ssh-and-the-storm-command"></a>Surveiller et gérer une topologie avec SSH et la commande Storm

L’utilitaire `storm` vous permet d’utiliser des topologies en cours d’exécution à partir de la ligne de commande. Pour obtenir la liste complète des commandes, utilisez `storm -h` .

### <a name="list-topologies"></a>Liste de toutes les topologies

Utilisez la commande suivante pour répertorier toutes les topologies en cours d’exécution :

```shell
storm list
```

Cette commande retourne des informations semblables au texte suivant :

```shell
Topology_name        Status     Num_tasks  Num_workers  Uptime_secs
-------------------------------------------------------------------
WordCount            ACTIVE     29         2            263
```

### <a name="deactivate-and-reactivate-a-topology"></a>Désactiver et réactiver une topologie

La désactivation d’une topologie la met en pause jusqu’à ce qu’elle soit arrêtée ou réactivée. Utilisez les commandes suivantes pour désactiver ou réactiver une topologie :

```shell
storm Deactivate TOPOLOGYNAME
```

```shell
storm Activate TOPOLOGYNAME
```

### <a name="kill-a-running-topology"></a>Arrêt d’une topologie en cours d’exécution

Les topologies Storm, une fois démarrées, continuent leur exécution jusqu’à ce qu’elles soient arrêtées. Pour ce faire, utilisez la commande suivante :

```shell
storm kill TOPOLOGYNAME
```

### <a name="rebalance-a-topology"></a>Rééquilibrer une topologie

Le rééquilibrage d’une topologie permet au système de réviser le parallélisme de la topologie. Par exemple, si vous avez redimensionné le cluster pour ajouter plus de nœuds, le rééquilibrage permet à une topologie d’identifier les nouveaux nœuds.

> [!WARNING]  
> Le rééquilibrage d’une topologie désactive la topologie, puis redistribue les workers uniformément sur le cluster avant de rétablir la topologie dans l’état où elle se trouvait avant le rééquilibrage. Si la topologie était active, elle redevient active. Si elle était désactivée, elle reste désactivée.

```shell
storm rebalance TOPOLOGYNAME
```

## <a name="monitor-and-manage-a-topology-using-the-storm-ui"></a>Surveiller et gérer une topologie avec l’interface utilisateur de Storm

L’interface utilisateur Storm fournit une interface web incluse dans votre cluster HDInsight pour utiliser les topologies en cours d’exécution. Pour afficher l’interface utilisateur Storm, utilisez un navigateur web pour ouvrir `https://CLUSTERNAME.azurehdinsight.net/stormui`, où *CLUSTERNAME* est le nom de votre cluster.

> [!NOTE]  
> Si vous êtes invité à fournir un nom d’utilisateur et un mot de passe, entrez le nom d'utilisateur et le mot de passe de l’administrateur que vous avez utilisé pour la création du cluster.

### <a name="storm-ui-main-page"></a>Page principale de l’interface utilisateur de Storm

La page principale de l’interface utilisateur de Storm fournit les informations suivantes :

| Section | Description |
| --- | --- |
| Résumé du cluster| informations de base sur le cluster Storm. |
| Résumé Nimbus | Une liste d’informations Nimbus de base. |
| Résumé de la topologie | liste des topologies en cours d’exécution. Pour afficher plus d’informations sur une topologie spécifique, sélectionnez le lien correspondant dans la colonne **Nom**. |
| Résumé du superviseur | informations sur le superviseur Storm. Pour afficher les ressources de travail associées à un superviseur spécifique, sélectionnez le lien correspondant dans la colonne **Hôte** ou **Id**. |
| Configuration Nimbus | configuration Nimbus du cluster. |

La page principale de l’interface utilisateur de Storm ressemble à cette page web :

![Page principale, interface utilisateur de Storm, topologies Apache Storm, Azure](./media/apache-storm-deploy-monitor-topology-linux/apache-storm-web-ui-main-page.png)

#### <a name="topology-summary"></a>Résumé de la topologie

La sélection d’un lien de la section **Résumé de la topologie** affiche les informations suivantes sur la topologie :

| Section | Description |
| --- | --- |
| Résumé de la topologie | informations de base sur la topologie. |
| Actions de topologie| actions de gestion que vous pouvez effectuer sur la topologie. Les actions disponibles sont décrites plus loin dans cette section. |
| Statistiques de topologie | statistiques relatives à la topologie. Pour définir le laps de temps pour une entrée dans cette section, sélectionnez son lien dans la colonne **Fenêtre**. |
| Spouts *(délai d’exécution)* | spouts utilisés par la topologie. Pour afficher plus d’informations sur un spout spécifique, sélectionnez le lien correspondant dans la colonne **Id**. |
| Bolts *(délai d’exécution)* | bolts utilisés par la topologie. Pour afficher plus d’informations sur un bolt spécifique, sélectionnez le lien correspondant dans la colonne **Id**. |
| Ressources Worker | Une liste de ressources Worker. Pour afficher plus d’informations sur une ressource Worker spécifique, sélectionnez le lien correspondant dans la colonne **Hôte**. |
| Visualisation de la topologie | Un bouton **Afficher la visualisation** qui affiche une visualisation de la topologie. |
| Configuration de la topologie | configuration de la topologie sélectionnée. |

La page Résumé de la topologie de Storm ressemble à cette page web :

![Page Résumé de la topologie, interface utilisateur de Storm, Apache Storm, Azure](./media/apache-storm-deploy-monitor-topology-linux/apache-storm-web-ui-topology-summary.png)

Dans la section **Actions de topologie**, vous pouvez sélectionner les boutons suivants pour effectuer une action :

| Bouton | Description |
| --- | --- |
| Activer | reprend le traitement d’une topologie désactivée. |
| Désactivation | interrompt une topologie en cours d’exécution. |
| Rééquilibrage | ajuste le parallélisme de la topologie. Il convient de rééquilibrer les topologies en cours d’exécution après avoir modifié le nombre de nœuds dans le cluster. Cette opération permet à la topologie d’ajuster le parallélisme pour compenser l’augmentation ou la diminution du nombre de nœuds du cluster.<br/><br/>Pour plus d’informations, voir <a href="https://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html" target="_blank">Comprendre le parallélisme d’une topologie Apache Storm</a>.
| Tuer | met fin à une topologie Storm après expiration du délai spécifié. |
| Débogage | Commence une session de débogage pour la topologie en cours d’exécution. |
| Arrêter le débogage | Met fin à la session de débogage pour la topologie en cours d’exécution. |
| Modifier le niveau de journalisation | Modifie le niveau de journalisation du débogage. |

##### <a name="spout-and-bolt-summary"></a>Résumé relatif aux spouts et aux bolts

La sélection d’un spout à partir de la section **Spouts** ou **Bolts** affiche les informations suivantes sur l’élément sélectionné :

| Section | Description |
| --- | --- |
| Résumé du composant | informations de base sur le spout ou le bolt. |
| Actions du composant | Boutons **Déboguer** et **Arrêter le débogage**. |
| Statistiques du spout ou statistiques du bolt | statistiques relatives au spout ou au bolt. Pour définir le laps de temps pour une entrée dans cette section, sélectionnez son lien dans la colonne **Fenêtre**. |
| (Bolt uniquement)<br/>Statistiques d’entrée *(intervalle de temps)* | informations sur les flux d’entrée consommés par le bolt. |
| Statistiques de sortie *(intervalle de temps)* | informations sur les flux de données émis par le spout ou le bolt. |
| Profilage et débogage | Contrôles pour le profilage et le débogage des composants sur cette page. Vous pouvez définir la valeur **État/délai d’expiration (minutes)** , et vous pouvez sélectionner les boutons pour **JStack**, **Redémarrer le Worker** et **Tas**. |
| Executors *(délai d’exécution)* | informations sur les instances du spout ou du bolt. Sélectionnez l’entrée **Port** d’un exécuteur spécifique afin d’afficher le journal des informations de diagnostic généré pour cette instance. Vous pouvez également voir les ressources Worker associées à un exécuteur spécifique en sélectionnant son lien dans la colonne **Hôte**. |
| Erreurs | informations d’erreur pour le spout ou le bolt. |

La page de résumé du bolt de Storm ressemble à cette page web :

![Page Résumé du bolt, interface utilisateur de Storm, Apache Storm, Azure](./media/apache-storm-deploy-monitor-topology-linux/apache-storm-web-ui-bolt-summary.png)

## <a name="monitor-and-manage-the-topology-using-the-rest-api"></a>Surveiller et gérer la topologie à l’aide de l’API REST

L’interface utilisateur Storm repose sur l’API REST, ce qui vous permet de profiter de tâches de gestion et de surveillance similaires à l’aide de l’API REST. À l'aide de l'API REST, vous pouvez créer des outils personnalisés pour gérer et surveiller les topologies Storm.

Pour plus d’informations, voir [API REST de l’interface utilisateur Apache Storm](https://storm.apache.org/releases/current/STORM-UI-REST-API.html). Les informations suivantes sont spécifiques à l’utilisation de l’API REST avec Apache Storm sur HDInsight.

> [!IMPORTANT]  
> L’API REST Storm n’est pas disponible publiquement sur Internet. Vous devez y accéder à l’aide d’un tunnel SSH vers le nœud principal du cluster HDInsight. Pour plus d’informations sur la création et l’utilisation d’un tunnel SSH, consultez [Utiliser le tunneling SSH pour accéder à Azure HDInsight](../hdinsight-linux-ambari-ssh-tunnel.md).

### <a name="base-uri"></a>URI de base

L’URI de base pour l’API REST sur les clusters HDInsight basés sur Linux est disponible à l’URL `https://HEADNODEFQDN:8744/api/v1/`, où vous remplacez *HEADNODEFQDN* par le nœud principal. Le nom de domaine du nœud principal est généré lors de la création du cluster et n’est pas statique.

Vous trouverez le nom de domaine complet (FQDN) du nœud principal du cluster de plusieurs façons :

| Méthode de détection FQDN | Description |
| --- | --- |
| Session SSH | utilisez la commande `headnode -f` à partir d’une session SSH vers le cluster. |
| Ambari Web | Sur la page web du cluster Ambari (`https://CLUSTERNAME.azurehdinsight.net`), sélectionnez **Services** en haut de la page, puis sélectionnez **Storm**. Sous l’onglet **Résumé**, sélectionnez **Serveur de l’interface utilisateur de Storm**. Le nom de domaine complet du nœud qui héberge l’interface utilisateur de Storm et l’API REST figure en haut de la page. |
| API Ambari REST | utilisez la commande `curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/STORM/components/STORM_UI_SERVER"` pour extraire des informations sur le nœud sur lequel l’interface utilisateur de Storm et l’API REST s’exécutent. Remplacez les deux instances de *CLUSTERNAME* par le nom du cluster. Quand vous y êtes invité, entrez le mot de passe du compte (administrateur) du cluster. Dans la réponse, l’entrée « host_name » de la sortie JSON contient le nom de domaine complet du nœud. |

### <a name="authentication"></a>Authentication

Les requêtes à l’API REST doivent utiliser l’ *authentification de base*, vous devez donc utiliser le nom et le mot de passe de l’administrateur du cluster HDInsight.

> [!NOTE]  
> Étant donné que l’authentification de base est envoyée en texte clair, vous devez *toujours* utiliser HTTPS pour sécuriser les communications avec le cluster.

### <a name="return-values"></a>Valeurs retournées

Les informations renvoyées depuis l’API REST peuvent être uniquement utilisables depuis le cluster. Par exemple, le nom de domaine complet (FQDN) retourné pour les serveurs [Apache ZooKeeper](https://zookeeper.apache.org/) n’est pas accessible sur Internet.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Développer des topologies Java avec Apache Maven](apache-storm-develop-java-topology.md).

Pour d’autres exemples de topologies, voir [Exemples de topologies Apache Storm dans Azure HDInsight](apache-storm-example-topology.md).
