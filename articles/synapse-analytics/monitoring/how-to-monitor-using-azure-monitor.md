---
title: Comment analyser Azure Synapse Analytics à l’aide d’Azure Monitor
description: Découvrez comment surveiller votre espace de travail Synapse Analytics à l’aide de métriques, d’alertes et de journaux Azure Monitor
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 9032fcaf35265c791913f5b69fb0972bada6885f
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602459"
---
# <a name="use-azure-monitor-with-your-azure-synapse-analytics-workspace"></a>Utilisez Azure Monitor avec votre espace de travail Azure Synapse Analytics

Les applications cloud sont complexes, et comprennent de nombreux éléments mobiles. Les analyses fournissent des données pour vous aider à garantir que votre application reste opérationnelle et soit exécutée en toute intégrité. Les analyses vous aident également à éviter des problèmes potentiels et à résoudre les problèmes précédents. Vous pouvez utiliser les données de surveillance pour obtenir des informations détaillées sur votre application. Cette connaissance vous aide à améliorer les performances et la maintenabilité de l’application. Elle vous aide également à automatiser des actions qui nécessitent autrement une intervention manuelle.

Azure Monitor fournit des métriques, des alertes et des journaux d’activité de niveau de base d’infrastructure pour la plupart des services Azure. Les journaux de diagnostic Azure sont générés par une ressource et fournissent des informations complètes et fréquentes sur le fonctionnement de cette ressource. Azure Synapse Analytics peut écrire les journaux de diagnostic dans Azure Monitor.

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Monitor](../../azure-monitor/overview.md).

## <a name="metrics"></a>Mesures

Monitor vous apporte une visibilité sur les performances et l’intégrité de vos charges de travail Azure. Le type de données d’analyse le plus important de Monitor est la métrique, également appelée compteur de performances. Des métriques sont émises par la plupart des ressources Azure. Monitor propose plusieurs façons de configurer et d’utiliser ces métriques à des fins d’analyse et de résolution des problèmes.

Pour accéder à ces métriques, suivez les instructions fournies dans [Plateforme de données Azure Monitor](../../azure-monitor/platform/data-platform.md).

### <a name="workspace-level-metrics"></a>Métriques au niveau de l’espace de travail

Voici quelques-unes des métriques émises par les espaces de travail :

| **Mesure**                           | **Catégorie de métrique, nom d’affichage**                  | **Unité** | **Types d’agrégation** | **Description**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| IntegrationActivityRunsEnded         | Intégration, métrique d’exécution d’activité                     | Nombre    | Sum (default), Count                | Nombre total d’exécutions d’activité qui se sont produites/terminées dans une fenêtre de 1 minute. </br></br> Utilisez la dimension de résultat de cette métrique pour filtrer par état final réussi, échec ou annulé.|
| IntegrationPipelineRunsEnded         | Intégration, métrique des exécutions de pipeline                     | Nombre    | Sum (default), Count                | Nombre total d’exécutions de pipeline qui se sont produites/terminées dans une fenêtre de 1 minute. </br></br> Utilisez la dimension de résultat de cette métrique pour filtrer par état final réussi, échec ou annulé. |
| IntegrationTriggerRunsEnded          | Intégration, métrique d’exécution de déclencheur                      | Nombre    | Sum (default), Count                | Nombre total d’exécutions de déclencheur qui se sont produites/terminées dans une fenêtre de 1 minute. </br></br> Utilisez la dimension de résultat de cette métrique pour filtrer par état final réussi, échec ou annulé. |
| BuiltinSqlPoolDataProcessedBytes     | Pool SQL intégré, données traitées (octets) | Byte | Sum (default) | Quantité de données traitées par le pool SQL serverless intégré. |
| BuiltinSqlPoolLoginAttempts          | Pool SQL intégré, tentatives de connexion | Nombre | Sum (default) | Nombre de tentatives de connexion pour le pool SQL serverless intégré. |
| BuiltinSqlPoolDataRequestsEnded      | Pool SQL intégré, demandes terminées (octets) | Nombre | Sum (default) | Nombre de demandes SQL terminées pour le pool SQL serverless intégré. </br></br> Utilisez la dimension de résultat de cette métrique pour filtrer par état final. |

### <a name="dedicated-sql-pool-metrics"></a>Métriques du pool SQL dédié

Voici quelques-unes des métriques émises par les pools SQL dédiés :

| **Mesure**                           | **Nom complet**                  | **Unité** | **Types d’agrégation** | **Description**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| DWULimit                            | Limite DWU                       | Count   | Max (default), Min, Avg | Taille configurée du pool SQL |
| DWUUsed                             | DWU utilisé                        | Count   | Max (default), Min, Avg | Constitue une représentation générale de l’utilisation dans le pool SQL. Mesurée par Limite DWU * Pourcentage DWU |
| DWUUsedPercent                      | Pourcentage d’utilisation de DWU             | Pourcentage | Max (default), Min, Avg | Constitue une représentation générale de l’utilisation dans le pool SQL. Mesurée en prenant la valeur maximale entre le pourcentage de processeur et le pourcentage d’E/S des données |
| ConnectionsBlockedByFirewall        | Connexions bloquées par le pare-feu | Count   | Sum (default)   | Nombre de connexions bloquées par les règles de pare-feu. Revisitez les stratégies de contrôle d’accès pour votre pool SQL et supervisez ces connexions si le nombre est élevé |
| AdaptiveCacheHitPercent             | Pourcentage d’accès au cache adaptatif   | Pourcentage | Max (default), Min, Avg | Mesure avec quelle efficacité les charges de travail utilisent le cache adaptatif. Utilisez cette métrique avec celle du pourcentage d’accès au cache pour déterminer s’il convient d’effectuer une mise à l’échelle pour une capacité supplémentaire ou de réexécuter les charges de travail pour alimenter le cache |
| AdaptiveCacheUsedPercent            | Pourcentage d’utilisation du cache adaptatif  | Pourcentage | Max (default), Min, Avg | Mesure avec quelle efficacité les charges de travail utilisent le cache adaptatif. Utilisez cette métrique avec celle du pourcentage d’utilisation du cache pour déterminer s’il convient d’effectuer une mise à l’échelle pour une capacité supplémentaire ou de réexécuter les charges de travail pour alimenter le cache |
| LocalTempDBUsedPercent              | Pourcentage utilisé de tempdb locale    | Pourcentage | Max (default), Min, Avg | Utilisation de la tempdb locale sur tous les nœuds de calcul. Des valeurs sont émises toutes les cinq minutes |
| MemoryUsedPercent                   | Pourcentage de mémoire utilisé          | Pourcentage | Max (default), Min, Avg | Utilisation de la mémoire sur tous les nœuds du pool SQL |
| CPUPercent                          | Pourcentage d’utilisation de l’UC             | Pourcentage | Max (default), Min, Avg | Utilisation de l’UC sur tous les nœuds du pool SQL |
| Connexions                         | Connexions                     | Count   | Sum (default)  | Nombre total de connexions au pool SQL |
| ActiveQueries                      | Requêtes actives                 | Count   | Sum (default)   | Requêtes actives. L’utilisation de cette métrique non filtrée et non fractionnée affiche toutes les requêtes actives en cours d’exécution sur le système |
| QueuedQueries                      | Requêtes mises en file d’attente                  | Count   | Sum (default)   | Nombre cumulatif de demandes mises en file d’attente une fois la limite de concurrence maximale atteinte |
| WLGActiveQueries                   | Requêtes actives du groupe de charge de travail   | Count   | Sum (default)   | Requêtes actives dans le groupe de charges de travail. L’utilisation de cette métrique non filtrée et non fractionnée affiche toutes les requêtes actives en cours d’exécution sur le système |
| WLGActiveQueriesTimeouts           | Délais d’expiration des requêtes du groupe de charge de travail   | Count   | Sum (default)   | Requêtes pour le groupe de charge de travail dont le délai a expiré. Les délais d’expiration des requêtes signalés par cette métrique démarrent uniquement une fois que l’exécution de la requête a commencé (cela n’inclut pas le temps d’attente dû au verrouillage ou à l’attente des ressources) |
| WLGQueuedQueries                   | Requêtes en file d’attente du groupe de charge de travail   | Count   | Sum (default)   | Nombre cumulatif de demandes mises en file d’attente une fois la limite de concurrence maximale atteinte |
| WLGAllocationBySystemPercent        | Allocation du groupe de charge de travail par pourcentage système | Pourcentage | Max (default), Min, Avg, Sum | Allocation de pourcentage de ressources par rapport à l’ensemble du système |
| WLGAllocationByEffectiveCapResourcePercent   | Allocation du groupe de charge de travail par pourcentage maximal de ressources | Pourcentage | Max (default), Min, Avg | Affiche l’allocation de pourcentage des ressources par rapport au pourcentage de ressources limitées réelles par groupe de charges de travail. Cette métrique fournit l’utilisation effective du groupe de charges de travail |
| WLGEffectiveCapResourcePercent      | Pourcentage de ressources limitées réelles  | Pourcentage | Max (default), Min, Avg | Pourcentage de ressources limitées réelles pour le groupe de charges de travail. S’il existe d’autres groupes de charges de travail avec min_percentage_resource > 0, effective_cap_percentage_resource est abaissé proportionnellement |
| WLGEffectiveMinResourcePercent      | Pourcentage minimal de ressources réelles  | Pourcentage | Max (default), Min, Avg, Sum | Paramètre de pourcentage minimal de ressources réelles autorisé en tenant compte du niveau de service et des paramètres de groupe de charges de travail. Le min_percentage_resource effectif peut être ajusté à une valeur supérieure sur des niveaux de service inférieurs |

### <a name="apache-spark-pool-metrics"></a>Métriques du pool Apache Spark

Voici quelques-unes des métriques émises par les pools Apache Spark :

| **Mesure**                           | **Catégorie de métrique, nom d’affichage**                  | **Unité** | **Types d’agrégation** | **Description**                |
|--------------------------------------|------------------------------------------|----------|----------------------|--------------------------------|
| BigDataPoolApplicationsEnded  | Applications Apache Spark terminées  | Nombre | Sum (default) | Nombre d’applications de pool Apache Spark terminées |
| BigDataPoolAllocatedCores     | Nombre de vCores alloués au pool Apache Spark                 | Nombre | Max (default), Min, Avg | VCores alloués pour un pool Apache Spark |
| BigDataPoolAllocatedMemory    | Quantité de mémoire (Go) allouée au pool Apache Spark            | Nombre | Max (default), Min, Avg | Mémoire allouée au pool Apache Spark (Go) |
| BigDataPoolApplicationsActive | Applications Apache Spark actives | Nombre | Max (default), Min, Avg | Nombre d’applications actives du pool Apache Spark |

## <a name="alerts"></a>Alertes

Connectez-vous au portail Azure, puis sélectionnez **Monitor** > **Alertes** pour créer des alertes.

### <a name="create-alerts"></a>Créer des alertes

1. Sélectionnez **+ Nouvelle règle d’alerte** pour créer une nouvelle alerte.

1. Définissez la **condition d’alerte** pour spécifier quand l’alerte doit être déclenchée.

    > [!NOTE]
    > Veillez à sélectionner **Toutes** dans la liste déroulante **Filtrer par type de ressource**.

1. Définissez les **détails de l’alerte** pour spécifier plus précisément le mode de configuration de l’alerte.

1. Définissez le **groupe d’actions** pour déterminer qui doit recevoir les alertes (et comment).

## <a name="logs"></a>Journaux d’activité

### <a name="workspace-level-logs"></a>Journaux au niveau de l’espace de travail

Voici les journaux émis par les espaces de travail Azure Synapse Analytics :

| Schéma de la table Log Analytics | Nom de la catégorie de journal                 | Description |
|-------------------------------|-------------------------------------------------|-------------|
| SynapseGatewayApiRequests     | GatewayApiRequests             | Demandes d’API de passerelle Azure Synapse. |
| SynapseRbacOperations         | SynapseRbacOperations          | Opérations de contrôle d’accès en fonction du rôle (SRBAC) Azure Synapse. |

### <a name="dedicated-sql-pool-logs"></a>Journaux des pools SQL dédiés

Voici les journaux émis par les pools SQL dédiés :

| Schéma de la table Log Analytics        | Nom de la catégorie de journal             | Description |
|----------------------|--------------------------------------|-------------|
| SynapseSqlPoolExecRequests  | ExecRequests | Informations sur les demandes/requêtes SQL dans un pool SQL Azure Synapse dédié.
| SynapseSqlPoolDmsWorkers    | DmsWorkers   | Informations sur les Workers effectuant les étapes DMS dans un pool SQL Azure Synapse dédié.
| SynapseSqlPoolRequestSteps  | RequestSteps | Informations sur les étapes de demande qui composent une demande/requête SQL donnée dans un pool SQL dédié Azure Synapse.
| SynapseSqlPoolSqlRequests   | SqlRequests  | Informations sur les distributions de requêtes des étapes des demandes/requêtes SQL dans un pool SQL dédié Azure Synapse.
| SynapseSqlPoolWaits         | Attend        | Informations sur les états d’attente rencontrés lors de l’exécution d’une demande/requête SQL dans un pool SQL dédié Azure Synapse, y compris les verrous et les attentes sur les files d’attente de transmission.

Pour plus d’informations sur ces journaux, voir les informations suivantes :
- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

### <a name="apache-spark-pool-log"></a>Journal du pool Apache Spark

Voici le journal émis par les pools Apache Spark :

| Schéma de la table Log Analytics               | Nom de la catégorie de journal              | Description                 |
|-----------------------------|---------------------------------------|-----------------------------|
| SynapseBigDataPoolApplicationsEnded | BigDataPoolAppsEnded | Informations sur les applications Apache Spark terminées |

### <a name="diagnostic-settings"></a>Paramètres de diagnostic

Utilisez des paramètres de diagnostic pour configurer les journaux de diagnostic des ressources autres que de calcul. Les paramètres d’un contrôle de ressource présentent les caractéristiques suivantes :

* Ils spécifient l’emplacement d’envoi des journaux de diagnostic. Il peut s’agir, part exemple, d’un compte de stockage Azure, d’un Event Hub Azure ou de journaux Azure Monitor.
* Ils spécifient les catégories de journal qui sont envoyées.
* Ils spécifient la durée pendant laquelle chaque catégorie de journal doit être conservée dans un compte de stockage.
* Une durée de rétention de zéro jour signifie que les journaux d’activité sont conservés indéfiniment. Autrement, la valeur peut également être n’importe quel nombre de jours, compris entre 1 et 2 147 483 647.
* Si des stratégies de rétention sont définies, mais que le stockage des journaux d’activité dans un compte de stockage est désactivé, les stratégies de rétention n’ont aucun effet. Par exemple, cette condition peut se produire lorsque seules les options Event Hubs ou journaux Azure Monitor sont sélectionnées.
* Les stratégies de rétention sont appliquées par jour. La séparation entre les jours se produit à minuit en temps universel coordonné (UTC). À la fin de la journée, les journaux des jours qui dépassent la stratégie de rétention sont supprimés. Par exemple, si vous avez une stratégie de rétention d’une journée, au début de la journée les journaux antérieurs à la veille sont supprimés.

Avec les paramètres de diagnostic Azure Monitor, vous pouvez acheminer les journaux de diagnostic à des fins d’analyse vers plusieurs cibles différentes.

* **Compte de stockage** : Enregistrez vos journaux de diagnostic dans un compte de stockage à des fins d’audit ou d’inspection manuelle. Vous pouvez utiliser les paramètres de diagnostic pour spécifier la durée de rétention en jours.
* **Event Hub** : Diffusez en continu les journaux vers Azure Event Hubs. Les journaux deviennent des entrées pour un service partenaire ou une solution d’analyse personnalisée telle que Power BI.
* **Espace de travail Log Analytics** : Analysez les journaux avec Log Analytics. L’intégration d’Azure Synapse à Log Analytics est utile dans les scénarios suivants :
  * Vous souhaitez écrire des requêtes complexes sur un ensemble varié de mesures publiées par Azure Synapse vers Log Analytics. Vous pouvez créer des alertes personnalisées pour ces requêtes via Azure Monitor.
  * Vous souhaitez analyser l’ensemble des espaces de travail. Vous pouvez acheminer des données de plusieurs espaces de travail vers un seul espace de travail Log Analytics.

Vous pouvez également utiliser un compte de stockage ou un espace de noms Event Hub qui ne se trouve pas dans le même abonnement que la ressource émettrice des journaux. L'utilisateur qui configure le paramètre doit disposer d'un accès Azure RBAC (Contrôle d'accès en fonction du rôle Azure) approprié pour les deux abonnements.

#### <a name="configure-diagnostic-settings"></a>Configurer les paramètres de diagnostic

Créez ou ajoutez des paramètres de diagnostic pour votre espace de travail, un pool SQL dédié ou un pool Apache Spark.

1. Dans le portail, accédez à Monitor. Sélectionnez **Paramètres** > **Paramètres de diagnostic**.

1. Sélectionnez l’espace de travail Synapse, le pool SQL dédié ou le pool Apache Spark pour lequel vous souhaitez créer un paramètre de diagnostic.

1. S’il n’existe aucun paramètre de diagnostic dans l’espace de travail sélectionné, vous êtes invité à en créer un. Sélectionnez **Activer les diagnostics**.

   S’il existe des paramètres de diagnostic existants sur l’espace de travail, vous verrez une liste de paramètres déjà configurés sur la ressource. Sélectionnez **Ajouter le paramètre de diagnostic**.

1. Nommez votre paramètre, sélectionnez **Envoyer à Log Analytics**, puis sélectionnez un espace de travail dans **Espace de travail Log Analytics**.

    > [!NOTE]
    > Étant donné qu’une table de journal Azure ne peut pas comporter plus de 500 colonnes, nous vous **recommandons vivement** de sélectionner le _mode spécifique de la ressource_. Pour plus d’informations, consultez [Limitations connues de Log Analytics](../../azure-monitor/platform/resource-logs.md#column-limit-in-azurediagnostics).

1. Sélectionnez **Enregistrer**.

Après quelques instants, le nouveau paramètre apparaît dans la liste des paramètres de votre espace de travail, de votre pool SQL dédié ou du pool Apache Spark. Les journaux de diagnostic sont diffusés en continu vers cet espace de travail dès que de nouvelles données d’événement sont générées. Jusqu’à 15 minutes peuvent s’écouler entre l’émission d’un événement et son affichage dans Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline dans Synapse Studio](how-to-monitor-pipeline-runs.md). 

Pour plus d’informations sur la surveillance d’applications Apache Spark, consultez l’article [Superviser les applications Apache Spark dans Synapse Studio](apache-spark-applications.md).

Pour plus d’informations sur la surveillance des requêtes SQL, consultez l’article [Surveiller les requêtes SQL dans Synapse Studio](how-to-monitor-sql-requests.md).
