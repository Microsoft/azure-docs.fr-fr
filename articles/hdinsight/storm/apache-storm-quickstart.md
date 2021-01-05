---
title: 'Démarrage rapide : Créer/Gérer une topologie Apache Storm - Azure HDInsight'
description: Dans ce guide de démarrage rapide, découvrez comment créer et surveiller une topologie Apache Storm dans Azure HDInsight.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.date: 06/14/2019
ms.author: hrasheed
ms.custom: mvc
ms.openlocfilehash: 8f13c813eee9bcfde578ba210c5aa2f2fe1cac2c
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "86045023"
---
# <a name="quickstart-create-and-monitor-an-apache-storm-topology-in-azure-hdinsight"></a>Démarrage rapide : Créer et surveiller une topologie Apache Storm dans Azure HDInsight

Apache Storm est un système de calcul en temps réel, évolutif, distribué, à tolérance de panne, qui permet de traiter des flux de données. Avec Storm sur Azure HDInsight, vous pouvez créer un cluster Storm basé sur le cloud qui effectue l’analyse de données volumineuses en temps réel.

Dans ce guide de démarrage rapide, vous utilisez un exemple du projet Apache [storm-starter](https://github.com/apache/storm/tree/v2.0.0/examples/storm-starter) pour créer et surveiller une topologie Apache Storm sur un cluster Apache Storm existant.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Storm sur HDInsight. Consultez la section [Création de clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Storm** dans le champ **Type de cluster**.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="create-the-topology"></a>Création de la topologie

1. Connectez-vous à votre cluster Storm. Modifiez la commande ci-dessous en remplaçant `CLUSTERNAME` par le nom de votre cluster Storm, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. L’exemple **WordCount** est inclus dans votre cluster HDInsight sous `/usr/hdp/current/storm-client/contrib/storm-starter/`. Cette topologie génère des phrases aléatoires et compte le nombre d’occurrences. Utilisez la commande suivante pour démarrer la topologie **wordcount** sur le cluster :

    ```bash
    storm jar /usr/hdp/current/storm-client/contrib/storm-starter/storm-starter-topologies-*.jar org.apache.storm.starter.WordCountTopology wordcount
    ```

## <a name="monitor-the-topology"></a>Analyse de la topologie

Storm fournit une interface web incluse dans votre cluster HDInsight pour utiliser les topologies en cours d’exécution.

Suivez la procédure ci-après pour surveiller la topologie à l’aide de l’interface utilisateur de Storm :

1. Pour afficher l’interface utilisateur Storm, ouvrez un navigateur web et accédez à l’adresse `https://CLUSTERNAME.azurehdinsight.net/stormui`. Remplacez `CLUSTERNAME` par le nom de votre cluster.

2. Sous **Résumé de la topologie**, sélectionnez l’entrée **Statistiques** située dans la colonne **Nom**. Vous obtenez plus d’informations sur la topologie.

    ![Tableau de bord Storm avec les informations sur la topologie WordCount storm-starter.](./media/apache-storm-quickstart/hdi-topology-summary.png)

    Cette nouvelle page fournit les informations suivantes :

    |Propriété | Description |
    |---|---|
    |Statistiques de topologie|Informations de base sur les performances de la topologie, organisées dans des fenêtres de temps. La sélection d’une fenêtre de temps spécifique modifie la fenêtre de temps pour les informations affichées dans d’autres sections de la page.|
    |Spouts|Informations de base sur les spouts, y compris la dernière erreur retournée par chaque spout.|
    |Bolts|Informations de base sur les bolts.|
    |Configuration de la topologie|Obtenez des informations détaillées sur la configuration de la topologie.|
    |Activer|reprend le traitement d’une topologie désactivée.|
    |Désactivation|interrompt une topologie en cours d’exécution.|
    |Rééquilibrage|ajuste le parallélisme de la topologie. Il convient de rééquilibrer les topologies en cours d’exécution après avoir modifié le nombre de nœuds dans le cluster. Le rééquilibrage ajuste le parallélisme pour compenser l’augmentation/la réduction du nombre de nœuds du cluster. Pour plus d’informations, voir [Comprendre le parallélisme d’une topologie Apache Storm](https://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html).|
    |Tuer|met fin à une topologie Storm après expiration du délai spécifié.|

3. À partir de cette page, sélectionnez une entrée dans la section **Spouts** ou **Bolts**. Vous obtenez des informations relatives au composant sélectionné.

    ![Tableau de bord Storm avec des informations sur les composants sélectionnés.](./media/apache-storm-quickstart/hdi-component-summary.png)

    Cette nouvelle page affiche les informations suivantes :

    |Propriété | Description |
    |---|---|
    |Statistiques du spout/bolt|Informations de base sur les performances de la topologie, organisées dans des fenêtres de temps. La sélection d’une fenêtre de temps spécifique modifie la fenêtre de temps pour les informations affichées dans d’autres sections de la page.|
    |Statistiques d’entrée (bolt uniquement)|Informations sur les composants qui produisent des données consommées par le bolt.|
    |Statistiques de sortie|Informations sur les données émises par ce bolt.|
    |Exécuteurs|Informations sur les instances de ce composant.|
    |Erreurs|Erreurs générées par ce composant.|

4. Lorsque vous affichez les détails d’un spout ou d’un bolt, sélectionnez une entrée depuis la colonne **Port** située dans la section **Exécuteurs** pour afficher les détails d’une instance spécifique du composant.

```output
2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["with"]
2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["nature"]
2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [snow]
2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [snow, 747293]
2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [white]
2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [white, 747293]
2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [seven]
2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [seven, 1493957]
```

Dans cet exemple, le mot **sept** compte 1 493 957 occurrences. C’est le nombre de fois où le mot a été détecté depuis le démarrage de cette topologie.

## <a name="stop-the-topology"></a>Arrêt de la topologie

Retournez à la page **Résumé de la topologie**, puis sélectionnez le bouton **Supprimer** de la section **Actions de topologie**. Lorsque vous êtes invité à entrer le nombre de secondes à attendre avant l’arrêt de la topologie, entrez le nombre 10. Après le délai d’expiration, la topologie n’apparaît plus quand vous vous rendez dans la section **Interface utilisateur Storm** du tableau de bord.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir suivi ce guide de démarrage rapide, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées Azure Storage, pour que vous puissiez supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même lorsque vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, économique, mieux vaut supprimer les clusters lorsqu’ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé un exemple du projet Apache [storm-starter](https://github.com/apache/storm/tree/v2.0.0/examples/storm-starter) pour créer et surveiller une topologie Apache Storm sur un cluster Apache Storm existant. Passez à l’article suivant pour découvrir les notions de gestion et de surveillance des topologies Apache Storm.

> [!div class="nextstepaction"]
>[Déploiement et gestion des topologies Apache Storm sur Azure HDInsight](./apache-storm-deploy-monitor-topology-linux.md)