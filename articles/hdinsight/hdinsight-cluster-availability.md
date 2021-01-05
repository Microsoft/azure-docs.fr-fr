---
title: Superviser la disponibilité des clusters avec Apache Ambari dans Azure HDInsight
description: Découvrez comment utiliser Apache Ambari pour superviser la disponibilité et l’intégrité des clusters.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/01/2020
ms.openlocfilehash: 5cfc2abad828a3974c04074a4cde062a479f673f
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533561"
---
# <a name="how-to-monitor-cluster-availability-with-apache-ambari-in-azure-hdinsight"></a>Superviser la disponibilité des clusters avec Apache Ambari dans Azure HDInsight

Les clusters HDInsight incluent Apache Ambari, qui affiche des informations instantanées sur l’intégrité des clusters et fournit des alertes prédéfinies.

Cet article montre comment utiliser Ambari pour superviser votre cluster. À travers plusieurs exemples, il explique pas à pas comment configurer une alerte Ambari, superviser le taux de disponibilité des nœuds, et créer une alerte Azure Monitor qui se déclenche quand aucune pulsation n’a été reçue d’un ou de plusieurs nœuds durant une période de cinq heures.

## <a name="dashboard"></a>tableau de bord

Vous pouvez accéder au tableau de bord Ambari en sélectionnant le lien **Accueil Ambari** dans la section **Tableaux de bord du cluster** du panneau HDInsight Overview (Vue d’ensemble de HDInsight) dans le portail Azure, comme illustré ci-dessous. Vous pouvez également y accéder à partir d’un navigateur, à l’adresse `https://CLUSTERNAME.azurehdinsight.net` (où CLUSTERNAME est le nom de votre cluster).

![Affichage du portail de ressources HDInsight](media/hdinsight-cluster-availability/azure-portal-dashboard-ambari.png)

Vous êtes ensuite invité à entrer un nom d’utilisateur et un mot de passe pour vous connecter au cluster. Saisissez les informations d’identification que vous avez choisies lors de la création du cluster.

Vous êtes alors redirigé vers le tableau de bord Ambari, qui contient des widgets affichant diverses métriques qui vous donnent un aperçu rapide de l’état d’intégrité de votre cluster HDInsight. Ces widgets affichent des métriques telles que le nombre de DataNodes (nœuds Worker) et de JournalNodes (nœud Zookeeper) en direct, la durée de fonctionnement des NameNodes (nœuds principaux), ainsi que des mesures spécifiques à certains types de cluster, tels que la durée de fonctionnement de YARN ResourceManager pour les clusters Spark et Hadoop.

![Apache Ambari - Affichage du tableau de bord](media/hdinsight-cluster-availability/apache-ambari-dashboard.png)

## <a name="hosts--view-individual-node-status"></a>Hôtes - afficher l’état de chaque nœud

Vous pouvez également afficher des informations d’état pour chaque nœud. Sélectionnez l’onglet **Hôtes** pour afficher une liste de tous les nœuds de votre cluster et consulter des informations de base sur chaque nœud. La coche verte à gauche de chaque nom de nœud indique que tous les composants du nœud sont opérationnels. Si un composant est arrêté sur un nœud, vous verrez un triangle d’alerte rouge au lieu de la coche verte.

![HDInsight Apache Ambari - Affichage des hôtes](media/hdinsight-cluster-availability/apache-ambari-hosts1.png)

Vous pouvez ensuite sélectionner le **nom** d’un nœud pour afficher des métriques d’hôtes plus détaillées pour ce nœud. Cet affichage indique l’état/la disponibilité de chaque composant.

![Affichage de nœud unique pour les hôtes Apache Ambari](media/hdinsight-cluster-availability/apache-ambari-hosts-node.png)

## <a name="ambari-alerts"></a>Alertes Ambari

Ambari offre également plusieurs alertes configurables qui peuvent fournir des notifications pour certains événements. Les alertes déclenchées sont signalées dans le coin supérieur gauche d’Ambari, sous la forme d’un badge rouge indiquant le nombre d’alertes. Sélectionnez ce badge pour voir une liste des alertes actives.

![Apache Ambari - Nombre d’alertes actuelles](media/hdinsight-cluster-availability/apache-ambari-alerts.png)

Pour afficher une liste de définitions d’alerte et leurs états, sélectionnez l’onglet **Alertes** , comme ci-dessous.

![Affichage des définitions d’alertes Ambari](media/hdinsight-cluster-availability/ambari-alerts-definitions.png)

Ambari offre de nombreuses alertes prédéfinies relatives à la disponibilité, notamment :

| Nom de l’alerte                        | Description   |
|---|---|
| DataNode Health Summary (Récapitulatif d’intégrité DataNode)           | Cette alerte de niveau de service est déclenchée si des DataNodes ne sont pas sains|
| NameNode High Availability Health (Intégrité de la haute disponibilité de NameNode) | Cette alerte de niveau de service est déclenchée si Active NameNode ou Standby NameNode ne sont pas exécutés.|
| Percent JournalNodes Available (Pourcentage de JournalNodes disponibles)    | Cette alerte est déclenchée si le nombre de JournalNodes en panne dans le cluster est supérieur au seuil critique configuré. Elle agrège les résultats des vérifications de processus JournalNode. |
| Percent DataNodes Available (Pourcentage de DataNodes disponibles)       | Cette alerte est déclenchée si le nombre de DataNodes en panne dans le cluster est supérieur au seuil critique configuré. Elle agrège les résultats des vérifications de processus DataNode.|


Pour afficher les détails d’une alerte ou en modifier les critères, sélectionnez le **nom** de l’alerte. Prenez **DataNode Health Summary** (Récapitulatif d’intégrité DataNode) comme exemple. Vous voyez la description de l’alerte, les critères spécifiques qui déclencheront une alerte de type « avertissement » ou « critique » ainsi que l’intervalle de vérification des critères. Pour modifier la configuration, sélectionnez le bouton **Modifier** dans le coin supérieur droit de la zone Configuration.

![Configuration de l’alerte Apache Ambari](media/hdinsight-cluster-availability/ambari-alert-configuration.png)

Ici, vous pouvez modifier la description et, plus important encore, l’intervalle de vérification et les seuils des alertes de type avertissement ou critique.

![Vue pour modifier les configurations d’alertes Ambari](media/hdinsight-cluster-availability/ambari-alert-configuration-edit.png)

Dans cet exemple, vous pouvez faire en sorte que 2 DataNodes non sains déclenchent une alerte critique et qu’1 DataNode non sain déclenche uniquement un avertissement. Sélectionnez **Enregistrer** quand vous avez terminé les modifications.

## <a name="email-notifications"></a>Notifications par e-mail

Vous pouvez également configurer les notifications par e-mail pour les alertes Ambari. Pour ce faire, dans l’onglet **Alertes** , cliquez sur le bouton **Actions** dans le coin supérieur gauche, puis sur **Gérer les modifications** .

![Action de gestion des notifications Ambari](media/hdinsight-cluster-availability/ambari-manage-notifications.png)

Une boîte de dialogue pour la gestion des notifications d’alerte s’ouvre. Sélectionnez **+** en bas de la boîte de dialogue et renseignez les champs obligatoires pour indiquer à Ambari les détails du serveur de messagerie à partir duquel envoyer des e-mails.

> [!TIP]
> Configurer les notifications par e-mail Ambari peut être un bon moyen pour centraliser la réception d’alertes lors de la gestion de nombreux clusters HDInsight.

## <a name="next-steps"></a>Étapes suivantes

- [Availability and reliability of Apache Hadoop clusters in HDInsight](./hdinsight-business-continuity.md) (Disponibilité et fiabilité des clusters Apache Hadoop dans HDInsight)
- [Disponibilité des clusters - Journaux Azure Monitor](./cluster-availability-monitor-logs.md)
- [Utiliser les journaux Azure Monitor](hdinsight-hadoop-oms-log-analytics-tutorial.md)
- [Notifications par e-mail Apache Ambari](apache-ambari-email.md)
