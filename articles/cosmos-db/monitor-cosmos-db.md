---
title: Superviser Azure Cosmos DB | Microsoft Docs
description: Découvrez comment surveiller les performances et la disponibilité d’Azure Cosmos DB.
author: SnehaGunda
services: cosmos-db
ms.service: cosmos-db
ms.topic: how-to
ms.date: 12/01/2020
ms.author: sngun
ms.custom: subject-monitoring
ms.openlocfilehash: f7cc9d4062ad9a49ff84db6ac035ff53934ae0a0
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532055"
---
# <a name="monitor-azure-cosmos-db"></a>Surveiller Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Lorsque vous avez des applications critiques et des processus métier basés sur des ressources Azure, vous voulez superviser ces ressources pour connaître leur disponibilité, leurs performances et leur fonctionnement. Cet article décrit les données de supervision générées par les bases de données Azure Cosmos et comment vous pouvez utiliser les fonctionnalités d’Azure Monitor pour analyser ces données et créer des alertes.

Vous pouvez superviser vos données avec des métriques côté client et côté serveur. Quand vous utilisez des métriques côté serveur, vous pouvez superviser les données stockées dans Azure Cosmos DB avec les options suivantes :

* **Supervision avec le portail Azure Cosmos DB :** vous pouvez effectuer la supervision avec les métriques disponibles dans l’onglet **Métriques** du compte Azure Cosmos. Les métriques de cet onglet incluent le débit, le stockage, la disponibilité, la latence, la cohérence et les métriques au niveau du système. Par défaut, la période de conservation de ces métriques est de 7 jours. Pour plus d’informations, consultez la section [Superviser les données collectées à partir du portail Azure Cosmos DB](#monitoring-from-azure-cosmos-db) de cet article.

* **Supervision à l’aide de métriques dans Azure Monitor :** vous pouvez superviser les métriques de votre compte Azure Cosmos et créer des tableaux de bord à partir d’Azure Monitor. Azure Monitor collecte les métriques d’Azure Cosmos DB par défaut. Vous n’avez pas à configurer quoi que ce soit de manière explicite. Ces métriques sont collectées avec une précision d’une minute, qui peut varier en fonction de la métrique que vous choisissez. Par défaut, la période de conservation de ces métriques est de 30 jours. La plupart des métriques disponibles à partir des options précédentes sont également disponibles dans ces métriques. Les valeurs de dimension des métriques, comme le nom du conteneur, ne respectent pas la casse. Vous devez donc utiliser une comparaison ne respectant pas la casse lorsque vous comparez les chaînes sur ces valeurs de dimension. Pour plus d’informations, consultez la section [Analyser les données de métriques](#analyze-metric-data) de cet article.

* **Supervision à l’aide des journaux de diagnostic dans Azure Monitor :** vous pouvez superviser les journaux de votre compte Azure Cosmos et créer des tableaux de bord à partir d’Azure Monitor. Les événements, traces et autres données de télémétrie avec une précision d’une seconde sont stockés sous forme de journaux. Par exemple, si le débit d’un conteneur est modifié, les propriétés d’un compte Cosmos sont modifiées et ces événements sont capturés dans les journaux. Vous pouvez analyser ces journaux en exécutant des requêtes sur les données collectées. Pour plus d’informations, consultez la section [Analyser les données de journal](#analyze-log-data) de cet article.

* **Supervision par programmation avec des SDK :** vous pouvez superviser votre compte Azure Cosmos par programmation en utilisant les SDK .NET, Java, Python et Node.js et les en-têtes de l’API REST. Pour plus d’informations, consultez la section [Superviser Azure Cosmos DB par programmation](#monitor-cosmosdb-programmatically) de cet article.

L’illustration suivante montre différentes options disponibles pour superviser un compte Azure Cosmos DB à l’aide du portail Azure :

:::image type="content" source="media/monitor-cosmos-db/monitoring-options-portal.png" alt-text="Options de supervision disponibles dans le portail Azure" border="false":::

Quand vous utilisez Azure Cosmos DB, côté client, vous pouvez collecter des informations détaillées sur les frais de requête, l’ID d’activité, des informations sur l’arborescence des appels de procédure et sur les traces des exceptions, le code d’état/sous-état HTTP et la chaîne de diagnostic permettant de déboguer tout problème potentiel. Ces informations sont également nécessaires si vous devez contacter l’équipe du support technique Azure Cosmos DB. 

## <a name="monitor-overview"></a>Présentation de Monitor

La page **Vue d’ensemble** du portail Azure pour chaque compte Azure Cosmos DB comprend un bref aperçu de l’utilisation des ressources, comme le nombre total de demandes, les requêtes ayant généré un code d’état HTTP spécifique et la facturation horaire. Ces informations sont utiles. Toutefois, elles ne constituent qu’une petite quantité des données de supervision disponibles dans ce volet. Certaines de ces données sont collectées automatiquement et peuvent être analysées dès que vous créez la ressource. Vous pouvez activer d’autres types de collecte de données avec une certaine configuration.

## <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?

Azure Cosmos DB crée la surveillance des données avec [Azure Monitor](../azure-monitor/overview.md), un service de supervision de pile complète dans Azure qui fournit un ensemble complet de fonctionnalités pour superviser vos ressources Azure en plus des ressources figurant dans d’autres clouds et en local.

Si vous n’êtes pas encore familiarisé avec la supervision des services Azure, commencez avec l’article [Supervision de ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md) qui décrit les concepts suivants :

* Qu’est-ce qu’Azure Monitor ?
* Coûts associés à la supervision
* Analyse des données collectées dans Azure
* Configuration de la collecte des données
* Outils standard dans Azure pour l’analyse et la génération d’alertes sur les données analysées

Les sections suivantes s’appuient sur cet article en décrivant les données spécifiques collectées à partir d’Azure Cosmos DB et en fournissant des exemples de configuration de collecte de données et d’analyse de ces données avec les outils Azure.

## <a name="azure-monitor-for-azure-cosmos-db"></a>Azure Monitor pour Azure Cosmos DB

Azure Monitor pour Azure Cosmos DB est basé sur la [fonctionnalité de classeurs d’Azure Monitor](../azure-monitor/platform/workbooks-overview.md) et utilise les mêmes données de supervision collectées pour Azure Cosmos DB que celles décrites dans les sections ci-dessous. Utilisez Azure Monitor pour une vue des performances globales, des défaillances, de la capacité et de l’intégrité opérationnelle de toutes vos ressources Azure Cosmos DB dans une expérience interactive unifiée, et tirez parti des autres fonctionnalités d’Azure Monitor pour l’analyse détaillée et les alertes. Plus d’informations sur l’article [Explorer Azure Monitor pour Azure Cosmos DB](../azure-monitor/insights/cosmosdb-insights-overview.md).

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="monitoring-data"></a><a id="monitoring-from-azure-cosmos-db"></a> Données de supervision 

Azure Cosmos DB collecte les mêmes types de données de supervision que les autres ressources Azure, qui sont décrites dans [Supervision de des données à partir de ressources Azure](../azure-monitor/insights/monitor-azure-resource.md#monitoring-data). Pour obtenir une référence détaillée des journaux et des métriques créés par Azure Cosmos DB, consultez [Informations de référence sur les données de supervision Azure Cosmos DB](monitor-cosmos-db-reference.md).

La page **Vue d’ensemble** du portail Azure pour chaque base de données Azure Cosmos comprend un bref aperçu de l’utilisation de la base de données, y compris l’utilisation des requêtes et la facturation horaire. Il s’agit d’informations utiles, mais qui ne constituent qu’une petite quantité des données de surveillance disponibles. Certaines de ces données sont collectées automatiquement et disponibles pour analyse dès que vous créez la base de données, tandis que vous pouvez activer la collecte de données supplémentaires par configuration.

:::image type="content" source="media/monitor-cosmos-db/overview-page.png" alt-text="Page de présentation":::

## <a name="collection-and-routing"></a>Collecte et routage

Les métriques de plateforme et le journal d’activité sont collectés et stockés automatiquement, mais ils peuvent être acheminés vers d’autres emplacements à l’aide d’un paramètre de diagnostic.  

Les journaux de ressources ne sont pas collectés ni stockés tant que vous n’avez pas créé un paramètre de diagnostic et que vous ne les acheminez pas vers un ou plusieurs emplacements.

Pour connaître le processus détaillé de la création d’un paramètre de diagnostic à l’aide du portail Azure et obtenir des exemples de requêtes de diagnostic, consultez [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](cosmosdb-monitor-resource-logs.md). Lorsque vous créez un paramètre de diagnostic, vous spécifiez les catégories de journaux à collecter.

Les métriques et les journaux que vous pouvez collecter sont décrits dans les sections suivantes.

## <a name="analyzing-metrics"></a><a id="analyze-metric-data"></a> Analyse des métriques

Azure Cosmos DB fournit une expérience personnalisée pour l’utilisation des métriques. Vous pouvez analyser les métriques d’Azure Cosmos DB avec les métriques d’autres services Azure à l’aide de Metrics Explorer en ouvrant **Métriques** dans le menu **Azure Monitor**. Pour plus d’informations sur l’utilisation de cet outil, consultez [Prise en main d’Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md). Vous pouvez également découvrir comment superviser la [latence côté serveur](monitor-server-side-latency.md), l’[utilisation des unités de requête](monitor-request-unit-usage.md) et l’[utilisation des unités de requêtes normalisées](monitor-normalized-request-units.md) pour vos ressources Azure Cosmos DB.

Pour obtenir la liste des métriques de plateforme collectées pour Azure Cosmos DB, consultez l’article [Analyse des métriques de référence de données Cosmos DB](monitor-cosmos-db-reference.md#metrics).

Toutes les métriques d’Azure Cosmos DB se trouvent dans l’espace de noms **Métriques standard Cosmos DB**. Vous pouvez utiliser les dimensions suivantes avec ces métriques lorsque vous ajoutez un filtre à un graphique :

* CollectionName
* nom_base_de_données
* OperationType
* Région
* StatusCode

Pour référence, vous pouvez voir une liste de [toutes les métriques de ressources prises en charge dans Azure Monitor](../azure-monitor/platform/metrics-supported.md).

### <a name="view-operation-level-metrics-for-azure-cosmos-db"></a>Afficher les métriques de niveau opération pour Azure Cosmos DB

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Surveillance** dans la barre de navigation gauche, puis sélectionnez **Métriques**.

   :::image type="content" source="./media/monitor-cosmos-db/monitor-metrics-blade.png" alt-text="Volet Métriques dans Azure Monitor":::

1. À partir du volet **Métriques** > **Sélectionner une ressource** > choisissez l’**abonnement** exigé, puis **Groupe de ressources**. Pour le **type de ressource**, sélectionnez **Comptes Azure Cosmos DB**, choisissez une de vos comptes Azure Cosmos existants, puis sélectionnez **Appliquer**.

   :::image type="content" source="./media/monitor-cosmos-db/select-cosmosdb-account.png" alt-text="Choisir un compte Cosmos DB pour voir les métriques":::

1. Ensuite, vous pouvez sélectionner une métrique dans la liste des métriques disponibles. Vous pouvez sélectionner des métriques propres aux unités de requête, au stockage, à la latence, à la disponibilité, à Cassandra, etc. Pour découvrir de plus près toutes les métriques disponibles dans cette liste, consultez l’article [Métriques par catégorie](monitor-cosmos-db-reference.md). Dans cet exemple, nous allons sélectionner **Unités de requête** et **Moy** comme valeur d’agrégation.

   En plus de ces détails, vous pouvez également sélectionner l’**intervalle de temps** et la **granularité temporelle** des métriques. Au maximum, vous pouvez voir les métriques des 30 derniers jours.  Une fois que vous avez appliqué le filtre, un graphique s’affiche. Vous pouvez voir le nombre moyen d’unités de requête consommées par minute pendant la période sélectionnée.  

   :::image type="content" source="./media/monitor-cosmos-db/metric-types.png" alt-text="Choisir une métrique à partir du portail Azure":::

### <a name="add-filters-to-metrics"></a>Ajouter des filtres aux métriques

Vous pouvez également filtrer les métriques et le graphique affiché par une valeur **CollectionName**, **DatabaseName**, **OperationType**, **Region** et **StatusCode** spécifique. Pour filtrer les métriques, sélectionnez **Ajouter un filtre** et choisissez la propriété nécessaire comme **OperationType**, puis sélectionnez une valeur comme **Requête**. Le graphique affiche alors les unités de requête consommées pour l’opération de requête pendant la période sélectionnée. Les opérations exécutées par procédure stockée ne sont pas journalisées si bien qu’elles ne sont pas disponibles sous la métrique OperationType.

:::image type="content" source="./media/monitor-cosmos-db/add-metrics-filter.png" alt-text="Ajouter un filtre pour sélectionner la granularité des métriques":::

Vous pouvez regrouper des métriques à l’aide de l’option **Appliquer la division**. Par exemple, vous pouvez regrouper les unités de requête par type d’opération et voir le graphique pour toutes les opérations comme illustré dans l’image suivante :

:::image type="content" source="./media/monitor-cosmos-db/apply-metrics-splitting.png" alt-text="Ajouter un filtre d’application de la division":::

## <a name="analyzing-logs"></a><a id="analyze-log-data"></a> Analyse des journaux

Les données des journaux Azure Monitor sont stockées dans des tables, chacune ayant son propre ensemble de propriétés uniques.

Tous les journaux de ressources dans Azure Monitor ont les mêmes champs suivis de champs spécifiques au service. Le schéma commun est décrit dans [Schéma des journaux des ressources Azure Monitor](../azure-monitor/platform/resource-logs-schema.md#top-level-common-schema). Pour obtenir la liste des types de journaux de ressources collectés pour Azure Cosmos DB, consultez l’article [Analyse de la référence des données Azure Cosmos DB](monitor-cosmos-db-reference.md#resource-logs).

Le [journal d’activité](/azure/azure-monitor/platform/activity-log) est un journal de plateforme dans Azure qui fournit des insights sur les événements de niveau abonnement. Vous pouvez l’afficher indépendamment ou le router vers Azure Monitor Logs, où vous pouvez effectuer des requêtes bien plus complexes à l’aide de Log Analytics.  

Azure Cosmos DB stocke les données dans les tables suivantes.

| Table de charge de travail | Description |
|:---|:---|
| AzureDiagnostics | Table commune utilisée par plusieurs services pour stocker les journaux de ressources. Les journaux de ressources d’Azure Cosmos DB peuvent être identifiés avec `MICROSOFT.DOCUMENTDB`.   |
| AzureActivity    | Table commune qui stocke tous les enregistrements du journal d’activité.

### <a name="sample-kusto-queries"></a>Exemples de requêtes Kusto

> [!IMPORTANT]
> Quand vous sélectionnez **Journaux** dans le menu Azure Cosmos DB, Log Analytics est ouvert avec l’étendue de requête définie sur le compte Azure Cosmos DB. Cela signifie que les requêtes de journal n’incluront que les données de cette ressource. Si vous voulez exécuter une requête qui inclut des données provenant d’autres comptes ou d’autres services Azure, sélectionnez **Journaux** dans le menu **Azure Monitor**. Pour plus d’informations, consultez [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](../azure-monitor/log-query/scope.md).

Voici quelques requêtes que vous pouvez entrer dans la barre de recherche **Recherche dans les journaux** pour superviser plus facilement vos ressources Azure Cosmos. Ces requêtes fonctionnent avec le [nouveau langage](../azure-monitor/log-query/log-query-overview.md).

* Pour rechercher tous les journaux de diagnostic issus d’Azure Cosmos DB pour la période spécifiée :

    ```Kusto
    AzureDiagnostics 
    | where ResourceProvider=="Microsoft.DocumentDb" and Category=="DataPlaneRequests"

    ```

* Pour rechercher toutes les opérations regroupées par ressource :

    ```Kusto
    AzureActivity 
    | where ResourceProvider=="Microsoft.DocumentDb" and Category=="DataPlaneRequests" 
    | summarize count() by Resource

    ```

* Pour rechercher toutes les activités utilisateur regroupées par ressource :

    ```Kusto
    AzureActivity 
    | where Caller == "test@company.com" and ResourceProvider=="Microsoft.DocumentDb" and Category=="DataPlaneRequests" 
    | summarize count() by Resource
    ```

## <a name="alerts"></a>Alertes

Azure Monitor vous avertit de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Elles permettent d’identifier et de résoudre les problèmes affectant votre système avant que vos clients ne les remarquent. Vous pouvez définir des alertes sur des [métriques](../azure-monitor/platform/alerts-metric-overview.md), sur des [journaux](../azure-monitor/platform/alerts-unified-log.md) et sur le [journal d’activité](../azure-monitor/platform/activity-log-alerts.md). Les différents types d’alertes présentent des avantages et des inconvénients

Par exemple, le tableau suivant récapitule quelques règles d’alerte pour vos ressources. Une liste détaillée des règles d’alerte est disponible dans le portail Azure. Pour en savoir plus, consultez l’article [Guide pratique pour configurer des alertes](create-alerts.md).  

| Type d’alerte | Condition | Description  |
|:---|:---|:---|
|Limitation du taux sur les unités de requête (alerte de métrique) |Nom de la dimension : StatusCode, Opérateur : Égal à, Valeurs de dimension : 429  | Émet une alerte si le conteneur ou une base de données a dépassé la limite de débit provisionnée. |
|Région basculée |Opérateur : Supérieur à, Type d’agrégation : Nombre, Valeur du seuil : 1 | Quand une seule région est basculée. Cette alerte est utile si vous n’avez pas activé le basculement automatique. |
| Permuter des clés (alerte de journal d’activité)| Niveau de l’événement : Information, État : Démarré| Émet une alerte quand les clés de compte sont permutées. Vous pouvez mettre à jour votre application avec les nouvelles clés. |

## <a name="monitor-azure-cosmos-db-programmatically"></a><a id="monitor-cosmosdb-programmatically"></a> Superviser Azure Cosmos DB par programmation

Les métriques au niveau du compte disponibles dans le portail, par exemple les demandes d'utilisation du stockage du compte et de total, ne sont pas disponibles via les API SQL. Toutefois, vous pouvez extraire des données d'utilisation au niveau de la collection à l’aide des API SQL. Pour extraire des données au niveau de la collection, procédez comme suit :

* Pour utiliser l’API REST, [effectuez une opération GET sur la collection](/rest/api/cosmos-db/get-a-collection). Les informations de quota et d'utilisation de la collection sont retournées dans les en-têtes x-ms-resource-quota et x-ms-resource-usage de la réponse.

* Pour utiliser le Kit de développement logiciel (SDK) .NET, utilisez la méthode [DocumentClient.ReadDocumentCollectionAsync](/dotnet/api/microsoft.azure.documents.client.documentclient.readdocumentcollectionasync), qui retourne une valeur [ResourceResponse](/dotnet/api/microsoft.azure.documents.client.resourceresponse-1) contenant un nombre de propriétés d’utilisation, notamment **CollectionSizeUsage**, **DatabaseUsage**, **DocumentUsage** et plus encore.

Pour accéder à des mesures supplémentaires, utilisez le [Kit de développement logiciel (SDK) Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Insights). Les définitions de mesures disponibles peuvent être récupérées en appelant :

```http
https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/providers/microsoft.insights/metricDefinitions?api-version=2018-01-01
```

Pour récupérer des métriques individuelles, utilisez le format suivant :

```http
https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/providers/microsoft.insights/metrics?timespan={StartTime}/{EndTime}&interval={AggregationInterval}&metricnames={MetricName}&aggregation={AggregationType}&`$filter={Filter}&api-version=2018-01-01
```

Pour plus d’informations, consultez l’article [API REST de surveillance Azure](../azure-monitor/platform/rest-api-walkthrough.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir une référence des journaux et des métriques créés par Azure Cosmos DB, consultez [Informations de référence sur les données de supervision Azure Cosmos DB](monitor-cosmos-db-reference.md).
* Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md).
