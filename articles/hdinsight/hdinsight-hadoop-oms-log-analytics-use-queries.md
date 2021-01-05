---
title: Interroger les journaux Azure Monitor pour surveiller les clusters Azure HDInsight
description: Apprenez à interroger les journaux Azure Monitor pour surveiller les travaux exécutés dans un cluster HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/02/2019
ms.openlocfilehash: c2ddcd8ea3524b4afdfa7f70d21f0cba96975f72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86085359"
---
# <a name="query-azure-monitor-logs-to-monitor-hdinsight-clusters"></a>Interroger les journaux Azure Monitor pour surveiller les clusters HDInsight

Découvrez des scénarios de base sur l'utilisation des journaux Azure Monitor pour surveiller les clusters Azure HDInsight :

* [Analyser les métriques du cluster HDInsight](#analyze-hdinsight-cluster-metrics)
* [Créer des alertes d’événement](#create-alerts-for-tracking-events)

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Vous devez avoir configuré un cluster HDInsight pour utiliser les journaux Azure Monitor, et ajouté les solutions de supervision des journaux Azure Monitor propres au cluster HDInsight à l'espace de travail. Pour obtenir des instructions, consultez [Utiliser les journaux Azure Monitor avec des clusters HDInsight](hdinsight-hadoop-oms-log-analytics-tutorial.md).

## <a name="analyze-hdinsight-cluster-metrics"></a>Analyser les métriques du cluster HDInsight

Découvrez comment rechercher des métriques spécifiques pour votre cluster HDInsight.

1. Ouvrez l’espace de travail Log Analytics qui est associé à votre cluster HDInsight à partir du portail Azure.
1. Sous **Général**, sélectionnez **Journaux**.
1. Tapez la requête suivante dans la zone de recherche afin de rechercher toutes les métriques disponibles dans tous les clusters HDInsight configurés pour utiliser les journaux Azure Monitor, puis sélectionnez **Exécuter**. Passez en revue les résultats.

    ```kusto
    search *
    ```

    ![Analytiques Apache Ambari - Rechercher toutes les métriques](./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-search-all-metrics.png "Rechercher toutes les métriques")

1. Dans le menu de gauche, sélectionnez l’onglet **Filtrer**.

1. Sous **Type**, sélectionnez **Pulsation**. Sélectionnez ensuite **Appliquer et exécuter**.

    ![Log Analytics - Rechercher des métriques spécifiques](./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-search-specific-metrics.png "Rechercher des métriques spécifiques")

1. Notez que la requête dans la zone de texte devient :

    ```kusto
    search *
    | where Type == "Heartbeat"
    ```

1. Vous pouvez afficher plus de détails en utilisant les options disponibles dans le volet gauche. Par exemple :

    - Pour voir les journaux d’un nœud spécifique :

        ![Rechercher des erreurs spécifiques - Sortie 1](./media/hdinsight-hadoop-oms-log-analytics-use-queries/log-analytics-specific-node.png "Rechercher des erreurs spécifiques - Sortie 1")

    - Pour voir les journaux à certains moments :

        ![Rechercher des erreurs spécifiques - Sortie 2](./media/hdinsight-hadoop-oms-log-analytics-use-queries/log-analytics-specific-time.png "Rechercher des erreurs spécifiques - Sortie 2")

1. Sélectionnez **Appliquer et exécuter**, et examinez les résultats. Notez également que la requête a été mise à jour :

    ```kusto
    search *
    | where Type == "Heartbeat"
    | where (Computer == "zk2-myhado") and (TimeGenerated == "2019-12-02T23:15:02.69Z" or TimeGenerated == "2019-12-02T23:15:08.07Z" or TimeGenerated == "2019-12-02T21:09:34.787Z")
    ```

### <a name="additional-sample-queries"></a>Autres exemples de requêtes

Exemple de requête basée sur la moyenne des ressources utilisées dans un intervalle de 10 minutes, catégorisées par nom de cluster :

```kusto
search in (metrics_resourcemanager_queue_root_default_CL) * 
| summarize AggregatedValue = avg(UsedAMResourceMB_d) by ClusterName_s, bin(TimeGenerated, 10m)
```

Au lieu d’affiner les résultats en fonction de la moyenne des ressources utilisées, vous pouvez utiliser la requête suivante pour affiner les résultats en fonction du moment où les ressources maximales ont été utilisées (ainsi qu’aux 90e et 95e centiles) dans une fenêtre de 10 minutes :

```kusto
search in (metrics_resourcemanager_queue_root_default_CL) * 
| summarize ["max(UsedAMResourceMB_d)"] = max(UsedAMResourceMB_d), ["pct95(UsedAMResourceMB_d)"] = percentile(UsedAMResourceMB_d, 95), ["pct90(UsedAMResourceMB_d)"] = percentile(UsedAMResourceMB_d, 90) by ClusterName_s, bin(TimeGenerated, 10m)
```

## <a name="create-alerts-for-tracking-events"></a>Créer des alertes pour des événements de suivi

La première étape pour créer une alerte consiste à définir une requête qui déclenche l’alerte. Vous pouvez utiliser la requête de votre choix pour créer une alerte.

1. Ouvrez l’espace de travail Log Analytics qui est associé à votre cluster HDInsight à partir du portail Azure.
1. Sous **Général**, sélectionnez **Journaux**.
1. Exécutez la requête suivante là où vous souhaitez créer une alerte, puis sélectionnez **Exécuter**.

    ```kusto
    metrics_resourcemanager_queue_root_default_CL | where AppsFailed_d > 0
    ```

    La requête fournit la liste des applications en échec exécutées sur des clusters HDInsight.

1. Sélectionnez **Nouvelle règle d’alerte** en haut de la page.

    ![Entrer une requête pour créer une alerte 1](./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-create-alert-query.png "Entrer une requête pour créer une alerte 1")

1. Dans la fenêtre **Créer une règle**, saisissez la requête et d’autres détails pour créer une alerte, puis sélectionnez **Créer une règle d'alerte**.

    ![Entrer une requête pour créer une alerte 2](./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-create-alert.png "Entrer une requête pour créer une alerte 2")

### <a name="edit-or-delete-an-existing-alert"></a>Modifier ou supprimer une alerte existante

1. Ouvrez l’espace de travail Log Analytics à partir du portail Azure.

1. Dans le menu de gauche, sous **Supervision**, sélectionnez **Alertes**.

1. Vers le haut, sélectionnez **Gérer les règles d’alerte**.

1. Sélectionnez l’alerte à modifier ou à supprimer.

1. Vous disposez des options suivantes : **Enregistrer**, **Ignorer**, **Désactiver** et **Supprimer**.

    ![HDInsight, journaux Azure Monitor, alerte, supprimer, modifier](media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-edit-alert.png)

Pour plus d'informations, consultez [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](../azure-monitor/platform/alerts-metric.md).

## <a name="see-also"></a>Voir aussi

* [Bien démarrer avec les requêtes de journal Azure Monitor](../azure-monitor/log-query/get-started-queries.md).
* [Créer des vues personnalisées à l'aide du Concepteur de vues d'Azure Monitor](../azure-monitor/platform/view-designer.md)
