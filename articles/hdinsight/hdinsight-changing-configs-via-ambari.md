---
title: Optimiser les clusters à l’aide d’Apache Ambari dans Azure HDInsight
description: Utilisez l’interface utilisateur web d’Apache Ambari pour configurer et optimiser les clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 05/04/2020
ms.openlocfilehash: 52eb1b6c89ff02cb44fe731c2463ab02c284f26c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86086447"
---
# <a name="optimize-clusters-with-apache-ambari-in-azure-hdinsight"></a>Optimiser les clusters à l’aide d’Apache Ambari dans Azure HDInsight

HDInsight fournit des clusters Apache Hadoop pour les applications de traitement de données à grande échelle. La gestion, la surveillance et l’optimisation de ces clusters complexes à nœuds multiples peuvent être difficiles. Apache Ambari est une interface web qui permet de gérer et de superviser les clusters Linux HDInsight.

Pour une présentation de l’interface utilisateur web d’Ambari, voir [Gérer des clusters HDInsight avec l’interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md).

Connectez-vous à Ambari à l’adresse `https://CLUSTERNAME.azurehdidnsight.net` avec les informations d’identification du cluster. L’écran initial affiche un tableau de bord de présentation.

![Apache Ambari - Tableau de bord utilisateur affiché](./media/hdinsight-changing-configs-via-ambari/apache-ambari-dashboard.png)

L’interface utilisateur Web d’Ambari permet de gérer les hôtes, les services, les alertes, les configurations et les vues. Ambari ne peut être utilisé pour créer un cluster HDInsight, ni mettre à niveau des services. De même, il ne peut gérer les piles et versions, mettre hors service ou réactiver des hôtes, ou ajouter des services au cluster.

## <a name="manage-your-clusters-configuration"></a>Gérer la configuration de votre cluster

Les paramètres de configuration permettent de paramétrer un service particulier. Pour modifier les paramètres de configuration d’un service, sélectionnez le service dans la barre latérale **Services** (à gauche). Accédez ensuite à l’onglet **Configurations** dans la page de détails du service.

![Apache Ambari - Barre latérale des services](./media/hdinsight-changing-configs-via-ambari/ambari-services-sidebar.png)

## <a name="modify-namenode-java-heap-size"></a>Modifier la taille du tas Java NameNode

La taille du tas Java NameNode dépend de nombreux facteurs tels que la charge du cluster, de même que le nombre de fichiers et le nombre de blocs. La taille par défaut de 1 Go fonctionne bien pour la plupart des clusters, mais certaines charges de travail peuvent nécessiter plus ou moins de mémoire.

Pour modifier la taille du tas Java NameNode :

1. Sélectionnez **HDFS (HDFS)** dans la barre latérale Services (Services) et accédez à l’onglet **Configs (Configurations)** .

    ![Apache Ambari - Configuration HDFS](./media/hdinsight-changing-configs-via-ambari/ambari-apache-hdfs-config.png)

1. Recherchez le paramètre **NameNode Java heap size (Taille du tas Java NameNode)** . Vous pouvez également utiliser la zone de texte **Filter (Filtrer)** pour saisir et trouver un paramètre. Sélectionnez l’icône de **crayon** en regard du nom de paramètre.

    ![Apache Ambari - Taille du tas Java NameNode](./media/hdinsight-changing-configs-via-ambari/ambari-java-heap-size.png)

1. Tapez la nouvelle valeur dans la zone de texte, puis appuyez sur **Entrée** pour enregistrer la modification.

    ![Ambari - Modifier la taille du tas Java NameNode - 1](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edit1.png)

1. La taille du tas Java NameNode passe de 2 Go à 1 Go.

    ![Taille du tas Java NameNode modifiée - 2](./media/hdinsight-changing-configs-via-ambari/java-heap-size-edited.png)

1. Enregistrez vos modifications en cliquant sur le bouton **Save (Enregistrer)** vert en haut de l’écran de configuration.

    ![Apache Ambari - Enregistrement de la configuration](./media/hdinsight-changing-configs-via-ambari/ambari-save-changes1.png)

## <a name="next-steps"></a>Étapes suivantes

* [Gérer des clusters HDInsight avec l’interface utilisateur web d’Ambari](hdinsight-hadoop-manage-ambari.md)
* [API REST Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md)
* [Optimiser Apache HBase](./optimize-hbase-ambari.md)
* [Optimiser Apache Hive](./optimize-hive-ambari.md)
* [Optimiser Apache Pig](./optimize-pig-ambari.md)
