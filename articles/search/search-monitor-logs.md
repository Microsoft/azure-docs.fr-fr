---
title: Collecter des données de journal
titleSuffix: Azure Cognitive Search
description: Collectez et analysez des données de journal en activant un paramètre de diagnostic, puis utilisez le langage de requête Kusto pour explorer les résultats.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: e6fcf5980cf64b5fc088dfa295ef6221ffda6de9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499932"
---
# <a name="collect-and-analyze-log-data-for-azure-cognitive-search"></a>Collecter et analyser des données de journal pour Recherche cognitive Azure

Les journaux de diagnostic ou opérationnels fournissent des informations sur les opérations détaillées de Recherche cognitive Azure et sont utiles pour surveiller les processus de charge de travail et de service. En interne, des informations système existent sur le back-end pendant une courte période, suffisante pour l’investigation et l’analyse si vous émettez un ticket de support. Toutefois, si vous souhaitez utiliser la direction automatique sur les données opérationnelles, vous devez configurer un paramètre de diagnostic pour spécifier l’emplacement où les informations de journalisation sont collectées.

La journalisation des diagnostics est activée via l’intégration à [Azure Monitor](../azure-monitor/index.yml). 

Quand vous configurez la journalisation des diagnostics, vous êtes invité à spécifier un mécanisme de stockage. Le tableau suivant énumère les options de collecte et de persistance des données.

| Ressource | Utilisé pour |
|----------|----------|
| [Envoyer à l’espace de travail Log Analytics](../azure-monitor/learn/tutorial-resource-logs.md) | Les événements et les mesures sont envoyés à un espace de travail Log Analytics, lequel peut être interrogé dans le portail pour retourner des informations détaillées. Pour une introduction sur le sujet, consultez [Prise en main des journaux d’activité Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md). |
| [Archiver avec le stockage Blob](../storage/blobs/storage-blobs-overview.md) | Les événements et les mesures sont archivés dans un conteneur de blobs et stockés dans des fichiers JSON. Les journaux d’activité peuvent être très granulaires (par heure/minute), ce qui est utile pour la recherche d’un incident spécifique, mais pas pour une investigation ouverte. Utilisez un éditeur JSON pour afficher un fichier journal brut ou Power BI pour agréger et visualiser les données du journal.|
| [Diffuser vers un Event Hub](../event-hubs/index.yml) | Les événements et les mesures sont diffusés vers un service Azure Event Hubs. Choisissez cette option comme autre service de collecte de données pour les journaux d’activité très volumineux. |

## <a name="prerequisites"></a>Prérequis

Créez des ressources à l’avance pour pouvoir en sélectionner une ou plusieurs lors de la configuration de la journalisation des diagnostics.

+ [Créer un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md)

+ [Créez un compte de stockage](../storage/common/storage-account-create.md)

+ [Créer un hub d’événements](../event-hubs/event-hubs-create.md)

## <a name="enable-data-collection"></a>Activer la collecte des données

Les paramètres de diagnostic spécifient la façon dont les événements et les mesures journalisés sont collectés.

1. Sous **Supervision**, sélectionnez **Paramètres de diagnostic**.

   ![Paramètres de diagnostic](./media/search-monitor-usage/diagnostic-settings.png "Paramètres de diagnostic")

1. Sélectionnez **+ Ajouter un paramètre de diagnostic**.

1. Cochez **Log Analytics**, sélectionnez votre espace de travail, puis sélectionnez **OperationLogs** et **AllMetrics**.

   ![Configurer la collecte de données](./media/search-monitor-usage/configure-storage.png "Configurer la collecte de données")

1. Enregistrez le paramètre.

1. Une fois la journalisation activée, utilisez votre service de recherche pour commencer à générer des journaux d’activité et des mesures. Cela prendra du temps avant que les mesures et les événements journalisés ne deviennent disponibles.

Pour Log Analytics, il faudra plusieurs minutes avant que les données ne soient disponibles, après quoi vous pourrez exécuter des requêtes Kusto pour retourner des données. Pour plus d’informations, consultez [Surveiller les demandes de requête](search-monitor-logs.md).

Pour le Stockage Blob, une heure est nécessaire pour que les conteneurs s’y affichent. Il y a un seul objet blob par heure et par conteneur. Les conteneurs sont créés uniquement quand il existe une activité à journaliser ou à mesurer. Quand les données sont copiées dans un compte de stockage, les données sont au format JSON et sont placées dans deux conteneurs :

+ journaux d’activité-Insights-operationlogs : pour les journaux d’activité de trafic de recherche
+ mesures-Insights-pt1m : pour les mesures

## <a name="query-log-information"></a>Informations sur le journal des requêtes

Deux tables contiennent des journaux d’activité et des mesures pour Recherche cognitive Azure : **AzureDiagnostics** et **AzureMetrics**.

1. Sous **Supervision**, sélectionnez **Journaux d’activité**.

1. Entrez **AzureMetrics** dans la fenêtre de requête. Exécutez cette requête simple pour vous familiariser avec les données collectées dans cette table. Faites défiler la table pour afficher les mesures et les valeurs. Notez le nombre d’enregistrements indiqué en haut et, si votre service a collecté des mesures pendant un certain temps, vous souhaiterez peut-être ajuster l’intervalle de temps pour obtenir un jeu de données gérable.

   ![Table AzureMetrics](./media/search-monitor-usage/azuremetrics-table.png "Table AzureMetrics")

1. Entrez la requête suivante pour retourner un jeu de résultats tabulaire.

   ```
   AzureMetrics
    | project MetricName, Total, Count, Maximum, Minimum, Average
   ```

1. Répétez les étapes précédentes, en commençant par **AzureDiagnostics** pour retourner toutes les colonnes à titre d’information, suivies d’une requête plus sélective qui extrait des informations plus intéressantes.

   ```
   AzureDiagnostics
   | project OperationName, resultSignature_d, DurationMs, Query_s, Documents_d, IndexName_s
   | where OperationName == "Query.Search" 
   ```

   ![Table AzureDiagnostics](./media/search-monitor-usage/azurediagnostics-table.png "Table AzureDiagnostics")

## <a name="kusto-query-examples"></a>Exemples de requêtes Kusto

Si vous avez activé la journalisation des diagnostics, vous pouvez interroger **AzureDiagnostics** pour obtenir la liste des opérations exécutées sur votre service et à quel moment. Vous pouvez également mettre en corrélation l’activité pour examiner les changements de performances.

#### <a name="example-list-operations"></a>Exemple : Lister les opérations 

Retournez une liste d’opérations et un nombre de chacune d’elles.

```
AzureDiagnostics
| summarize count() by OperationName
```

#### <a name="example-correlate-operations"></a>Exemple : Mettre en corrélation les opérations

Associez une demande de requête à des opérations d’indexation et restituez les points de données sur un graphique de temps pour voir si les opérations coïncident.

```
AzureDiagnostics
| summarize OperationName, Count=count()
| where OperationName in ('Query.Search', 'Indexing.Index')
| summarize Count=count(), AvgLatency=avg(DurationMs) by bin(TimeGenerated, 1h), OperationName
| render timechart
```

## <a name="logged-operations"></a>Opérations journalisées

Les événements journalisés capturés par Azure Monitor incluent ceux qui sont liés à l’indexation et aux requêtes. La table **AzureDiagnostics** dans Log Analytics collecte les données opérationnelles relatives aux requêtes et à l’indexation.

| NomOpération | Description |
|---------------|-------------|
| ServiceStats | Cette opération est un appel de routine à l’API [GET Service Statistics](/rest/api/searchservice/get-service-statistics), appelée directement ou implicitement pour remplir une page de présentation du portail lors de son chargement ou de son actualisation. |
| Query.Search |  Demandes de requêtes par rapport à un index. Consultez [Superviser les requêtes](search-monitor-queries.md) pour plus d’informations sur les requêtes journalisées.|
| Indexing.Index  | Cette opération est un appel à [Ajout, mise à jour ou suppression de documents](/rest/api/searchservice/addupdate-or-delete-documents). |
| indexes.Prototype | Il s’agit d’un index créé par l’Assistant Importation de données. |
| Indexers.Create | Créez un indexeur de manière explicite ou implicite à l’aide de l’Assistant Importation de données. |
| Indexers.Get | Retourne le nom d’un indexeur chaque fois que l’indexeur est exécuté. |
| Indexers.Status | Retourne l’état d’un indexeur chaque fois que l’indexeur est exécuté. |
| DataSources.Get | Retourne le nom de la source de données chaque fois qu’un indexeur est exécuté.|
| Indexes.Get | Retourne le nom d’un index chaque fois qu’un indexeur est exécuté. |

## <a name="log-schema"></a>Schéma du journal

Si vous créez des rapports personnalisés, les structures de données qui contiennent des données de journal Recherche cognitive Azure sont conformes au schéma ci-dessous. Pour le Stockage Blob, chaque blob a un objet racine appelé **records** contenant un tableau d’objets de journal. Chaque objet blob contient des enregistrements de toutes les opérations qui ont eu lieu au cours de la même heure.

Le tableau suivant est une liste partielle des champs communs à la journalisation des ressources.

| Nom | Type | Exemple | Notes |
| --- | --- | --- | --- |
| timeGenerated |DATETIME |"2018-12-07T00:00:43.6872559Z" |Horodatage de l’opération |
| resourceId |string |«/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/> MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE » |Votre ID de ressource |
| operationName |string |« Query.Search » |Nom de l’opération |
| operationVersion |string |"30/06/2020" |Version d’API utilisée |
| catégorie |string |« OperationLogs » |constant |
| resultType |string |« Success » |Valeurs possibles : Réussite ou Échec |
| resultSignature |int |200 |Code de résultat HTTP |
| durationMS |int |50 |Durée de l’opération en millisecondes |
| properties |object |consultez le tableau suivant |Objet contenant des données propres à l’opération |

### <a name="properties-schema"></a>Schéma de propriétés

Les propriétés ci-dessous sont spécifiques à Recherche cognitive Azure.

| Nom | Type | Exemple | Notes |
| --- | --- | --- | --- |
| Description_s |string |« GET /indexes(’content’)/docs » |Point de terminaison de l’opération |
| Documents_d |int |42 |Nombre de documents traités |
| IndexName_s |string |"test-index" |Nom de l’index associé à l’opération |
| Query_s |string |"?search=AzureSearch&$count=true&api-version=2020-06-30" |Paramètres de requête |

## <a name="metrics-schema"></a>Schéma de mesures

Les mesures sont capturées pour les demandes de requête et mesurées à des intervalles d’une minute. Chaque mesure expose des valeurs minimales, maximales et moyennes par minute. Pour plus d’informations, consultez [Surveiller les demandes de requête](search-monitor-queries.md).

| Nom | Type | Exemple | Notes |
| --- | --- | --- | --- |
| resourceId |string |«/SUBSCRIPTIONS/11111111-1111-1111-1111-111111111111/<br/>RESOURCEGROUPS/DEFAULT/PROVIDERS/<br/>MICROSOFT.SEARCH/SEARCHSERVICES/SEARCHSERVICE » |votre ID de ressource |
| metricName |string |« Latency » |Nom de la mesure |
| time |DATETIME |"2018-12-07T00:00:43.6872559Z" |Horodatage de l’opération |
| average |int |64 |Valeur moyenne des échantillons bruts dans l’intervalle de temps de la mesure, en secondes ou en pourcentage, en fonction de la mesure. |
| minimum |int |37 |Valeur minimale des échantillons bruts dans l’intervalle de temps de la mesure, en secondes. |
| maximum |int |78 |Valeur maximale des échantillons bruts dans l’intervalle de temps de la mesure, en secondes.  |
| total |int |258 |Valeur totale des échantillons bruts dans l’intervalle de temps de la mesure, en secondes.  |
| count |int |4 |Nombre de mesures émises à partir d’un nœud dans le journal pendant l’intervalle d’une minute.  |
| timegrain |string |« PT1M » |Fragment de temps de la mesure au format ISO 8601. |

Les requêtes sont souvent exécutées en quelques millisecondes, par conséquent seules les requêtes mesurées en secondes, QPS par exemple, apparaissent dans les métriques.

Dans le cas de la mesure **Search Queries Per Second**, la valeur minimale correspondra à la valeur la plus faible des requêtes de recherche par seconde qui a été enregistrée pendant cette minute. Il en va de même pour la valeur maximale. La moyenne représentera l’agrégat de ces valeurs pour toute la minute. Par exemple, sur une minute, le schéma peut être le suivant : une seconde de charge élevée, qui représente votre valeur SearchQueriesPerSecond maximale, puis 58 secondes de charge moyenne, et enfin une seconde avec une seule requête, qui représente la valeur minimale.

Pour **Pourcentage de requêtes de recherche limitées**, les valeurs minimales, maximales, moyennes et totales seront identiques : il s’agit du pourcentage de requêtes de recherche qui ont été limitées, en fonction du nombre total de requêtes de recherche pendant une minute.

## <a name="view-raw-log-files"></a>Afficher les fichiers journaux bruts

Le Stockage Blob est utilisé pour l’archivage des fichiers journaux. Vous pouvez utiliser n’importe quel éditeur JSON pour voir le fichier journal. Si vous n’en avez pas, nous vous recommandons [Visual Studio Code](https://code.visualstudio.com/download).

1. Dans le portail Azure, ouvrez votre compte de stockage. 

2. Dans le volet de navigation de gauche, cliquez sur **Objets blob**. **insights-logs-operationlogs** et **insights-metrics-pt1m** doivent s’afficher. Ces conteneurs sont créés par Recherche cognitive Azure quand les données de journal sont exportées vers le stockage Blob.

3. Faites défiler l’arborescence des dossiers vers le bas jusqu’à atteindre le fichier .json.  Utilisez le menu contextuel pour télécharger le fichier.

Une fois le fichier téléchargé, ouvrez-le dans un éditeur JSON pour en voir le contenu.

## <a name="next-steps"></a>Étapes suivantes

Si ce n’est déjà fait, passez en revue les principes de base de la surveillance des services de recherche pour en savoir plus sur les différentes capacités de surveillance disponibles.

> [!div class="nextstepaction"]
> [Surveiller les opérations et l’activité dans Recherche cognitive Azure](search-monitor-usage.md)