---
title: Surveiller et gérer des clusters Azure HDInsight à l’aide de l’interface utilisateur Web d’Ambari
description: Découvrez comment utiliser Apache Ambari pour gérer et surveiller des clusters HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/16/2020
ms.openlocfilehash: f21b72cd519c1615d8273bf316a8d0ccad039672
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546158"
---
# <a name="manage-hdinsight-clusters-by-using-the-apache-ambari-web-ui"></a>Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari

[!INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

Apache Ambari simplifie la gestion et la surveillance d’un cluster Apache Hadoop. Cette simplification s’effectue en fournissant une interface utilisateur web et une API REST faciles à utiliser. Ambari est inclus dans les clusters HDInsight, et sert à surveiller le cluster et à apporter des modifications de configuration.

Dans ce document, vous apprenez à utiliser l’interface utilisateur web d’Ambari avec un cluster HDInsight.

## <a name="what-is-apache-ambari"></a><a id="whatis"></a>Qu’est-ce qu’Apache Ambari ?

[Apache Ambari](https://ambari.apache.org) simplifie la gestion d’Hadoop grâce à une interface utilisateur web facile à utiliser. Vous pouvez utiliser Ambari pour gérer et surveiller les clusters Hadoop. Les développeurs peuvent intégrer ces fonctionnalités dans leurs applications à l’aide des [API REST Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md).

## <a name="connectivity"></a>Connectivité

L’interface utilisateur web d’Ambari est disponible sur votre cluster HDInsight à l’adresse `https://CLUSTERNAME.azurehdinsight.net`, où `CLUSTERNAME` correspond au nom de votre cluster.

> [!IMPORTANT]  
> Une connexion à Ambari sur HDInsight requiert HTTPS. Lorsque vous êtes invité à vous authentifier, utilisez le nom du compte administrateur et le mot de passe fournis lors de la création du cluster. Si vous n’êtes pas invité à entrer vos informations d’identification, vérifiez vos paramètres réseau pour vous assurer qu’il n’existe aucun problème de connectivité entre le client et les clusters Azure HDInsight.

## <a name="ssh-tunnel-proxy"></a>Tunnel SSH (proxy)

Bien qu’Ambari pour votre cluster soit accessible directement via Internet, certains liens de l’interface utilisateur web d’Ambari (comme celui vers le JobTracker) ne sont pas exposés sur Internet. Pour accéder à ces services, vous devez créer un tunnel SSH. Pour plus d’informations, consultez [Utilisation du tunneling SSH avec HDInsight](hdinsight-linux-ambari-ssh-tunnel.md).

## <a name="ambari-web-ui"></a>Interface utilisateur web d'Ambari

> [!WARNING]  
> Les fonctionnalités de l’interface utilisateur web Ambari ne sont pas toutes prises en charge dans HDInsight. Pour plus d’informations, consultez la section [Opérations non prises en charge](#unsupported-operations) de ce document.

Pendant la connexion à l’interface utilisateur web d’Ambari, vous devez vous authentifier auprès de la page. Utilisez le nom d’utilisateur et le mot de passe de l’administrateur du cluster (administrateur par défaut) utilisés lors de la création du cluster.

Lorsque la page s'ouvre, vérifiez la barre située en haut de l'écran. Cette barre contient les informations et les commandes suivantes :

![Apache Ambari - Présentation du tableau de bord](./media/hdinsight-hadoop-manage-ambari/apache-ambari-dashboard.png)

|Élément |Description |
|---|---|
|Logo Ambari|Ouvre le tableau de bord, qui peut être utilisé pour surveiller le cluster.|
|Nom du cluster # ops|Affiche le nombre d’opérations Ambari en cours. En sélectionnant le nom du cluster ou **ops #** , une liste des opérations effectuées en arrière-plan s’affiche.|
|# alertes|Affiche les avertissements ou alertes critiques, le cas échéant, pour le cluster.|
|tableau de bord|Affiche le tableau de bord.|
|Services|Informations et paramètres de configuration des services sur le cluster.|
|Hôtes|Informations et paramètres de configuration des nœuds sur le cluster.|
|Alertes|Journal contenant informations, avertissements et alertes critiques.|
|Admin|Pile logicielle/services installés sur le cluster, informations sur le compte de service et sécurité Kerberos.|
|Bouton Administrateur|Gestion d’Ambari, paramètres utilisateur et déconnexion.|

## <a name="monitoring"></a>Surveillance

### <a name="alerts"></a>Alertes

La liste suivante contient les statuts d’alerte courants utilisés par Ambari :

* **OK**
* **Avertissement**
* **CRITIQUE**
* **UNKNOWN**

Les alertes autres que **OK** font apparaître l’entrée **# alerts** en haut de la page pour afficher le nombre d’alertes. La sélection de cette entrée fait apparaître les alertes et leur état.

Les alertes sont organisées en plusieurs groupes par défaut, qui peuvent être consultés depuis la page **Alertes** .

![Apache Ambari - Page de résumé des alertes](./media/hdinsight-hadoop-manage-ambari/hdinsight-alerts-page.png)

Vous pouvez gérer les groupes à l'aide du menu **Actions** , en sélectionnant **Gérer les groupes d'alerte** .

![Apache Ambari - Gérer les groupes d’alertes](./media/hdinsight-hadoop-manage-ambari/ambari-manage-alerts.png)

Vous pouvez gérer les méthodes d’alerte et créer des notifications d’alerte à partir du menu **Actions** en sélectionnant __Gérer les notifications__ . Les notifications en cours sont affichées. Créez des notifications à partir d’ici. Les notifications peuvent être envoyées via **EMAIL** ou **SNMP** lorsque des combinaisons spécifiques alerte/gravité se produisent. Par exemple, vous pouvez envoyer un e-mail lorsque l’une des alertes du groupe **YARN par défaut** est définie comme **Critique** .

![Apache Ambari - Créer une notification d’alerte](./media/hdinsight-hadoop-manage-ambari/create-alert-notification.png)

Enfin, en sélectionnant __Gérer les paramètres d’alerte__ à partir du menu __Actions__ , vous pouvez définir le nombre d’occurrences d’une alerte avant l’envoi d’une notification. Ce paramètre peut être utilisé pour empêcher les notifications des erreurs temporaires.

Pour obtenir un tutoriel sur les notifications d’alerte à l’aide d’un compte [SendGrid](../sendgrid-dotnet-how-to-send-email.md) gratuit, consultez [Configurer des notifications par e-mail Apache Ambari dans Azure HDInsight](./apache-ambari-email.md).

### <a name="cluster"></a>Cluster

L'onglet **Mesures** du tableau de bord contient une série de widgets qui permettent de surveiller facilement l'état de votre cluster d'un seul coup d'œil. Plusieurs widgets, tels que **Utilisation du processeur** , fournissent des informations supplémentaires lorsque vous cliquez dessus.

![Apache Ambari - Tableau de bord avec métriques](./media/hdinsight-hadoop-manage-ambari/hdi-metrics-dashboard.png)

L'onglet **Cartes thermiques** affiche les paramètres sous forme de cartes thermiques colorées, allant du vert au rouge.

![Apache Ambari - Tableau de bord avec cartes thermiques](./media/hdinsight-hadoop-manage-ambari/hdi-heatmap-dashboard.png)

Pour plus d’informations sur les nœuds du cluster, sélectionnez **Hôtes** . Puis sélectionnez le nœud spécifique qui vous intéresse.

![Apache Ambari - Résumé détaillé de l’hôte](./media/hdinsight-hadoop-manage-ambari/ambari-host-details1.png)

### <a name="services"></a>Services

La barre latérale du tableau de bord, intitulée **Services** , fournit un aperçu rapide de l'état des services exécutés sur le cluster. Différentes icônes sont utilisées pour indiquer l’état ou les actions à entreprendre. Par exemple, un symbole de recyclage jaune s’affiche si un service doit être recyclé.


![Apache Ambari - Barre latérale des services](./media/hdinsight-hadoop-manage-ambari/apache-ambari-service-bar.png)

> [!NOTE]  
> Les services affichés diffèrent selon les versions et les types de cluster HDInsight. Les services affichés ici peuvent être différents de ceux affichés pour votre cluster.

La sélection d’un service fait apparaître des informations détaillées sur le service.

![Apache Ambari - Informations résumées sur le service](./media/hdinsight-hadoop-manage-ambari/ambari-service-details.png)

#### <a name="quick-links"></a>Liens rapides

Certains services affichent un lien **Liens rapides** en haut de la page. Ce lien peut être utilisé pour accéder à des interfaces utilisateur web spécifiques des services, tels que :

* **Historique des travaux** : historique des travaux MapReduce.
* **Gestionnaire des ressources** : interface utilisateur du gestionnaire des ressources YARN.
* **NameNode** : interface utilisateur NameNode HDFS.
* **Interface utilisateur web Oozie** : interface utilisateur Oozie.

La sélection de l’un de ces liens ouvre un nouvel onglet dans votre navigateur qui affiche la page sélectionnée.

> [!NOTE]  
> Si vous sélectionnez **Liens rapides** pour un service, une erreur « serveur introuvable » peut s’afficher. Si vous rencontrez cette erreur, vous devez utiliser un tunnel SSH lorsque vous utilisez **Liens rapides** pour ce service. Pour plus d’informations, consultez [Utilisation du tunneling SSH avec HDInsight](hdinsight-linux-ambari-ssh-tunnel.md).

## <a name="management"></a>Gestion

### <a name="ambari-users-groups-and-permissions"></a>Utilisateurs d'Ambari, groupes et autorisations

L’utilisation des utilisateurs, des groupes et des autorisations est prise en charge. Pour l’administration locale, voir [Autoriser les utilisateurs à accéder à Apache Ambari Views](./hdinsight-authorize-users-to-ambari.md). Pour les clusters joints à un domaine, consultez l’article [Gestion des clusters HDInsight joints à un domaine](./domain-joined/hdinsight-security-overview.md).

> [!WARNING]  
> Ne modifiez pas le mot de passe pour l’agent de surveillance Ambari (hdinsightwatchdog) sur votre cluster HDInsight basé sur Linux. La modification du mot de passe élimine la possibilité d’utiliser les actions de script ou d’effectuer des opérations de mise à l’échelle sur votre cluster.

### <a name="hosts"></a>Hôtes

La page **Hôtes** répertorie tous les hôtes du cluster. Pour gérer des hôtes, procédez comme suit :

![Apache Ambari - Page de vue d’ensemble des hôtes](./media/hdinsight-hadoop-manage-ambari/hdinsight-hosts-page.png)

> [!NOTE]  
> L’ajout, la désactivation et la remise en service d’un hôte ne doivent pas être utilisés avec des clusters HDInsight.

1. Sélectionnez l’hôte que vous souhaitez gérer.

2. Utilisez le menu **Actions** pour sélectionner l'action que vous souhaitez effectuer :

    |Élément |Description |
    |---|---|
    |Démarrer tous les composants|Démarre tous les composants sur l’hôte.|
    |Arrêter tous les composants|Arrête tous les composants sur l’hôte.|
    |Redémarrer tous les composants|Redémarre tous les composants sur l’hôte.|
    |Activer le mode Maintenance|Supprime les alertes de l’hôte. Ce mode doit être activé si vous effectuez des actions qui génèrent des alertes. Par exemple, l’arrêt et le démarrage d’un service.|
    |Désactiver le mode Maintenance|Rétablit les alertes normales de l’hôte.|
    |Arrêter|Arrête DataNode ou NodeManagers sur l’hôte.|
    |Démarrer|Démarre DataNode ou NodeManagers sur l’hôte.|
    |Redémarrer|Arrête et redémarre DataNode ou NodeManagers sur l’hôte.|
    |Désactiver|Supprime un hôte du cluster. **N’utilisez pas cette action sur les clusters HDInsight.**|
    |Réactiver|Ajoute un hôte préalablement désactivé au cluster. **N’utilisez pas cette action sur les clusters HDInsight.**|

### <a name="services"></a><a id="service"></a>Services

Dans la page **Tableau de bord** ou **Services** , utilisez le bouton **Actions** situé au bas de la liste des services pour arrêter et démarrer tous les services.

:::image type="content" source="./media/hdinsight-hadoop-manage-ambari/ambari-service-actions.png" alt-text="Apache Ambari - Liste des actions du service." border="true":::

> [!WARNING]  
> Les nouveaux services doivent être ajoutés à l'aide d'une action de script lors de l’approvisionnement du cluster. Pour plus d’informations sur l’utilisation des actions de script, consultez [Personnaliser des clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md).

Bien que le bouton **Actions** permette de redémarrer tous les services, vous souhaitez souvent démarrer, arrêter ou redémarrer un service spécifique. Pour effectuer des actions sur un service individuel, procédez comme suit :

1. Depuis la page **Tableau de bord** ou **Services** , sélectionnez un service.

2. En haut de l'onglet **Résumé** , utilisez le bouton **Actions de service** et sélectionnez l'action que vous souhaitez effectuer. Cette action redémarre le service sur tous les nœuds.

    ![Apache Ambari - Actions sur un service individuel](./media/hdinsight-hadoop-manage-ambari/individual-service-actions.png)

   > [!NOTE]  
   > Le redémarrage de certains services lorsque le cluster fonctionne peut générer des alertes. Pour éviter ces alertes, vous pouvez utiliser le bouton **Actions de service** afin d’activer le **Mode Maintenance** avant de lancer le redémarrage.

3. Lorsqu’une action est sélectionnée, l’entrée **# op** située en haut de la page s’incrémente pour indiquer qu’une opération s’exécute en arrière-plan. Si leur affichage est configuré, une liste des opérations exécutées en arrière-plan apparaît.

   > [!NOTE]  
   > Si vous avez activé **Mode Maintenance** pour le service, n'oubliez pas de le désactiver à l'aide du bouton **Actions de service** une fois l'opération terminée.

Pour configurer un service, procédez comme suit :

1. Depuis la page **Tableau de bord** ou **Services** , sélectionnez un service.

2. Sélectionnez l'onglet **Configurations** . La configuration actuelle apparaît. Une liste des configurations précédentes est également affichée.

    ![Apache Ambari - Configuration du service](./media/hdinsight-hadoop-manage-ambari/ambari-service-configs.png)

3. Utilisez les champs affichés pour modifier la configuration, puis sélectionnez **Enregistrer** . Ou sélectionnez une configuration antérieure, puis sélectionnez **Définir comme actuelle** pour rétablir des paramètres utilisés précédemment.

## <a name="ambari-views"></a>Affichages Ambari

Les affichages Ambari permettent aux développeurs d’incorporer des éléments d’interface utilisateur dans l’interface utilisateur web Ambari à l’aide de l’infrastructure des affichages Apache Ambari. HDInsight propose les affichages suivants avec les types de clusters Hadoop :

* Affichage Hive : L’affichage Hive vous permet d’exécuter des requêtes Hive directement à partir du navigateur web. Vous pouvez enregistrer des requêtes, afficher les résultats, enregistrer les résultats dans le stockage de cluster ou les télécharger sur votre système local. Pour plus d’informations sur l’utilisation des affichages Hive, consultez [Utiliser des affichages Apache Hive avec HDInsight](hadoop/apache-hadoop-use-hive-ambari-view.md).

* Affichage Tez : L’affichage Tez vous permet de mieux comprendre et optimiser les tâches. Vous pouvez afficher des informations sur l’exécution des tâches Tez et les ressources utilisées.

## <a name="unsupported-operations"></a>Opérations non prises en charge

Les opérations Ambari suivantes ne sont pas prises en charge dans HDInsight :

* __Déplacement du service Collecteur de métriques__ . Quand vous affichez des informations sur le service Collecteur de métriques, l’une des actions disponibles à partir du menu Service Actions est __Move Metrics collector__ . Cette action n’est pas prise en charge avec HDInsight.

## <a name="next-steps"></a>Étapes suivantes

* [L’API REST Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md) avec HDInsight.
* [Utiliser Apache Ambari pour optimiser les configurations de cluster HDInsight](./hdinsight-changing-configs-via-ambari.md)
* [Mettre à l’échelle des clusters Azure HDInsight](./hdinsight-scaling-best-practices.md)